---
layout: post
title: "a fallible guide to persistent-template"
description: ""
category:
tags: [haskell negativeresult streamofconsciousness]
---

Template Haskell is often considered a smell in Haskell, responsible
for everything from slow compile times to impenetrable error messages
to the cow's milk turning sour. Some of this is warranted, but if you
need to check anything based on compile-time information, it is
also the only game in town.

Persistent is a good example of where TH is arguably justified.
Defining our database tables and our types in the same format means
that there is a single source of truth, with no way for the code to
fall out of touch with the database.

Nonetheless, sometimes you do want to break this model; in this case,
I have a type (Status from twitter-types) that I'd like to persist in
the database. I can't include this definition in the standard
quasiquoted file: the type already exists, I can't define it again.
This leaves me with two options:

1. use a type with the same structure as the Status I get from
   twitter-types, and write some conversion functions for converting
   between persisted and original statuses. Probably about twenty
   lines and ten minutes' work.

2. read the source of persistent-template and work out a way of
   controlling the declaration of datatypes on a table-by-table basis.

So! Let's dig into how it's currently set up.

Our top level entry point is "share".

```
share :: [[EntityDef] -> Q [Dec]]
      -> [EntityDef]
      -> Q [Dec]
```

In the first argument, we have a list of "builders", in a sense -
we'll pass a list of EntityDefs to each of them, and each will create
some declarations (Q [Dec]). In the normal case, we'll pass 'mkPersist
sqlSettings' and 'mkMigrate "migrateAll"'.

In the second, we have a list of EntityDefs, typically (but not necessarily) coming
from the 'persistLowerCase' quasiquoter.

Finally, our result type is a Q [Dec]. This is a little bit strange:
'share' is called at the top level, and it looks like a bare top-level
expression, not a binding declaration. Digging through the docs (and
https://stackoverflow.com/documentation/haskell/5216/template-haskell-quasiquotes#t=201705181813575250883)
, it turns out that when we have a Q [Dec] at the top level, we can
omit the standard $( ... ) syntax that we'd usually use for
introducing TH to normal Haskell code. Almost a little too convenient,
but ok.

This tells us that we are going to have to make at least two changes.
Our EntityDef will need to carry
the information of whether a datatype needs to be created for the
Entity or not. We can then monkey with mkPersist to look at that
information and only generate a datatype if we have requested one. (If all goes well,
mkMigrate ought to work unchanged.)

(For the moment, we'll change this in-place: the question of whether
this is to be a mergeable fork or an add-on can be dodged for now.)

So: first order of business is EntityDef. ```ag 'data EntityDef'```
shows us that it is defined in Database.Persist.Types.Base, meaning we'll have to edit persistent as
well as persistent-template. We'll add an 'isStandalone' field
to the EntityDef declaration that will default to False. When we do that, we try compiling to see
what broke:

```
    /home/mark/projects/persistent/persistent/Database/Persist/Quasi.hs:299:5: error:
        • Couldn't match expected type ‘EntityDef’
                      with actual type ‘Bool -> EntityDef’
        • Probable cause: ‘EntityDef’ is applied to too few arguments
          In the second argument of ‘($)’, namely
            ‘EntityDef ...
```

which makes sense. Looking at mkEntityDef, we can define a dummy value
for isStandalone, just to check everything else is hunky-dory, and it
compiles. Great. (Parenthetically, notice that we are never too far
from a type-correct system, even if it doesn't quite do what we want
yet. Get too far into the weeds and it is very difficult to extract
yourself.)

Now we want to see where we can stash a declaration that a particular
entity is standalone.


```
mkEntityDef :: PersistSettings
            -> Text -- ^ name
            -> [Attr] -- ^ entity attributes
            -> [Line] -- ^ indented lines
```

It's not the PersistSettings, they're global. You _could_ cram it into
the name of the model as some kind of godawful in-band signalling, but
this path always leads to pain. Let's keep looking. [Line] doesn't
seem right, they're the individual fields of the table, and we're
trying to find where to specify a feature of the entity as a whole. By
a process of elimination, it has to be [Attr]. Let's go grepping
again. (I should probably set tags up instead of dumb text search, but
grep/ag/ack are simple and dependable.)

Oh, there's a surprise - 'ag "data Attr"' and 'ag "newtype Attr"'
yield nothing. Turns out it's just a type synonym for Text. That's
unexpected - it'll mean we have to take a bit of attribute namespace,
but I suppose that was inevitable anyway. This means we can actually
back out our earlier changes and just pass "standalone" as a textual attribute.

With that, we just need to monkey with mkEntity a little.
'dataTypeDec' looks promising: if we check for "standalone" in the
attributes, we should be just about home free.

```
+    let inclDtd = if "standalone" `elem` (entityAttrs t)
+                  then []
+                  else [dtd]
     return $ addSyn $
-       dtd : mconcat fkc `mappend`
+       (mconcat (inclDtd:fkc)) `mappend`
```

Compiles, shipit!

We should probably check that this works the way we expect.

```
share [mkPersist sqlSettings] [persistLowerCase|
Bar standalone json
    name Text
|]
```

Gratifyingly and hopefully unsurprisingly, this fails in an obvious
way:

```

/home/mark/projects/persistent/persistent-template/test/main.hs:62:1: error:
    Not in scope: type constructor or class ‘Bar’
    Perhaps you meant ‘Baz’ (line 53)
```

Let's define it then!

```
data Bar = Bar { barName :: Text }
```

Clean compile, and all existing tests green.
We haven't actually tested this functionality yet, but inside the
persistent lib is a difficult place to do it - we'll add a more
thorough test in our app, given that we had to write that code anyway.

Checking against my app, I realise that I want to store a Status
(which fortuitously is named according to Persistent conventions - if
it weren't, I'd need to turn mpsPrefixFields off in sqlSettings.)

I define Status in the model quasiquoter, using all the same fields -
we want to get back to it working, and _then_ turn on standalone.

Now I get a bunch of errors about not having
PersistField defined for a range of types. Excellent! These are easy
to define with StandaloneDeriving (as Show is already defined by derivation)

```
derivePersistField "Entities"
deriving instance Read Entities
deriving instance Read HashTagEntity
deriving instance Read UserEntity
deriving instance Read x => Read (Entity x)
...
```

We continue doing this more or less mechanically until we get to this
odd complaint:

```
    • No instance for (persistent-2.7.0:Database.Persist.Sql.Class.PersistFieldSql
                         TW.Status)
```

We don't _particularly_ want to persist a Status as a field: the whole
point was to model it explicitly in a table. Why are we getting this?
Ah! The Status type has a self-reference in it! quotedStatus is a
Maybe Status!

It's at this point I'm inclined to say "ok, this was a bad idea, a
single isomorphism isn't so bad", but I want to finish this post, so
let's follow the rabbit down the hole and define ```derivePersistField
"Status"``` too. Might as well be hung for a sheep as a lamb.

Now we run into an odd problem:

```
/home/mark/projects/owlstacks.com/Model.hs:23:1: error:
    Duplicate instance declarations:
      instance PersistFieldSql Status -- Defined at Model.hs:23:1
      instance PersistFieldSql Status -- Defined at Orphans.hs:68:1
   |
23 | share
   | ^^^^^...

/home/mark/projects/owlstacks.com/Model.hs:23:1: error:
    Duplicate instance declarations:
      instance PersistField Status -- Defined at Model.hs:23:1
      instance PersistField Status -- Defined at Orphans.hs:68:1
   |
23 | share
```

Now it's starting to become a little clearer: we need the PersistField
instance in the Orphans file, which means we also have to be able to
disable instance generation in the quasiquoting code. One more time
unto the breach, dear friends: we're going back into TH.hs

inside mkPersist, we need to monkey with the defined instances


```
standalone t =  "standalone" `elem` (entityAttrs t)
```
and
```
filter (not . standalone)
```

inside mkPersist. Ok! Now we have "multiple declarations of StatusId",
and here we come to a bit of a grinding halt. Status is one type from
twitter-types, StatusId is another. Persistent expects to be able to
nab that piece of the namespace to describe the primary key of the
table in Haskell, and it isn't configurable.

I could write more code to get around this, but we have proceeded well
past the point where this exercise could be expected to yield useful
code, and I think I'm going to call it here. Persistent is a pretty
opinionated framework: if you have significantly different needs, or
want to serialise types that come from elsewhere to the database
without introducing an intermediary type, it
looks like a better idea to just use something else. Apologies for the
abrupt end, it's as much a surprise to me as anyone else.
