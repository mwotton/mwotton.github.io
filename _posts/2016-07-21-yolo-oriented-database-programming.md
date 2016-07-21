---
layout: post
title: "yolo oriented database programming"
description: ""
category: Haskell
tags: [haskell postgresql persistent]
---
{% include JB/setup %}

I've been playing around a bit with a slightly different style of
database code lately. Usually, I'd carefully check that a given
insertion/update isn't going to violate any parameters, then actually
do it, all wrapped in the implicit surrounding transaction that
Persistent gives you. It works, but it never feels quite right; what
was the point of telling your database what the constraints were if
you're going to duplicate them anyway?

So, this. It's a bit of an experiment, but so far I quite like it. The
basic idea is that Persistent has the outermost transaction, but we
can kinda fake nested transactions using savepoints. It requires
Postgresql, but I never really expected to be able to swap in
another SQL database anyway, and frankly why would you want to.

Basic idea here is that we want to add an entry to a join table and
return whether or not it was there already. Lightly edited for
corporate compliance:

```
data Result
  = NotPresent
  | AlreadyAdded
  | NowAdded
  deriving (Eq,Show)

doAThing :: UUID -> UUID -> DB Result
doAThing barU fooU =
  handle uniqViolation $ do
    startTransaction
    inserted <-  insertSelectCount $
       from $
       \(foos,bars) ->
         do where_ (foos ^. FooUuid ==. val fooU)
            where_ (bars ^. BarUuid ==. val barU)
            return $ Baz <# (foos ^. FooId) <&> (bars ^. BarId)
    commitTransaction
    case inserted of
      0 -> return NotPresent
      1 -> return NowAdded
      n -> throwM (InvalidInsertedCount n)
  where
    uniqViolation
      :: SqlError -> DB Result
    uniqViolation _e = do
      rollbackTransaction
      return AlreadyAdded

    rollbackTransaction, startTransaction, commitTransaction :: DB ()
    startTransaction    = rawExecute "SAVEPOINT             savepointname" []
    commitTransaction   = rawExecute "RELEASE SAVEPOINT     savepointname" []
    rollbackTransaction = rawExecute "ROLLBACK TO SAVEPOINT savepointname" []
```


EDIT: extracting as a combinator to make the logic a bit clearer:

```

withViolation :: forall a . DB a -> DB a ->  DB a
withViolation def body = do
  handle violation $ do
    startTransaction
    result <- body
    commitTransaction
    return result
  where
    violation :: SqlError -> DB a
    violation _e = do
      rollbackTransaction
      def

    rollbackTransaction, startTransaction, commitTransaction :: DB ()
    startTransaction    = rawExecute "SAVEPOINT             violationSavepoint" []
    commitTransaction   = rawExecute "RELEASE SAVEPOINT     violationSavepoint" []
    rollbackTransaction = rawExecute "ROLLBACK TO SAVEPOINT violationSavepoint" []
```

I'm @mwotton on twitter, hit me up with opinions, recriminations, etc.
