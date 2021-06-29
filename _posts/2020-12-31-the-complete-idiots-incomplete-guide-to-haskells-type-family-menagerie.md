---
layout: post
title: "The complete idiot's incomplete guide to Haskell's type family menagerie"
description: ""
category:
tags: [haskell]
---

If you're anything like me, you have avoided learning any more than
the bare minimum about Haskell's dependently typed features. This is
understandable: Haskell's basic features are smooth, elegant and
intuitive, a charge nobody has ever leveled against GADTs or type
families. You can go a long way using simple Haskell constructs.

However, it's becoming harder and harder to ignore, with libraries
like squeal and servant offering fascinating new abilities, like being
able to typecheck your queries against a database with squeal, or
generate HTTP clients without access to anything but a schema
expressed as a type that the server must also fit. It's not just about
finding more ways to eliminate sources of error: fancy types can also
allow you to write code that can operate over a much broader range of
inputs, making your codebase smaller and more understandable.

Anyway, enough pontification. Let's get into the zoo and explain what
each is good for.

1. GADTs

GADTs are a generalised version of sums. This means that, like sums,
they are closed. Everything about that GADT has to be in one place,
its declaration.

The classic use case for GADTs is in typesafe interpreters. To see
why, let's try to write one without them, and see where problems come
up.

```haskell
data Exp
  = IntLit Int
  | BinOp (Int -> Int) Exp Exp
  | IfMoreThan Exp Exp Exp Exp
```

Good enough for our purposes. Let's write an evaluator.


```haskell

eval :: Exp -> Int
eval (Lit i) = i
eval (BinOp f a b) = f (eval a) (eval b)
eval (IfMoreThan a b thenBranch elseBranch)
  | a' > b' = eval thenBranch
  | otherwise = eval elseBranch
  where a' = eval a
        b' = eval b
```

Cool, everything works. The language isn't terribly modular though, do
we really want to have to add a new `If` construct for every kind of
boolean comparison? Let's try to separate it out.


``` haskell
data Exp
  = IntLit Int
  | BinOp (Int -> Int) Exp Exp
  | If BoolExp Exp Exp

add = BinOp (+)

data BoolExp
  = BoolLit Bool
  | BoolBinArithmeticOp (Int -> Int -> Bool) Exp Exp
  | BoolBinOp (Bool -> Bool -> Bool) BoolExp BoolExp
  | IfBool BoolExp BoolExp BoolExp

eval = \case
  IntLit i -> i
  BinOp f a b -> f (eval a) (eval b)
  BoolBinOp f a b ->  f (evalBool a) (evalBool b)
  IfBool discr a b ->
    if (evalBool discr)
    then eval a
    else eval b

evalBool = \case
  BoolLit b -> b
  BoolBinArithmeticOp f a b -> f (eval a) (eval b)
  BoolBinOp f a b ->  f (evalBool a) (evalBool b)
  IfBool discr a b ->
    if (evalBool discr)
    then evalBool a
    else evalBool b


```

So what's going on here? We hit the problem that we are now modelling
Boolean values explicitly in the language, and need to be able to
represent expressions that return Booleans too.

This duplication is pretty unsatisfactory. We had to create two
variants of BinOps and If to handle the Bool case and the Int case, and the
duplication will repeat every time we add a new construct, like
lambdas. Perhaps GADTs help us do better?

``` haskell

data Exp a where
  Lit :: a -> Exp a
  BinOp :: (a -> a) -> Exp a -> Exp a -> Exp a
  If :: Exp Bool -> Exp a -> Exp a -> Exp a


eval = \case
  Lit a -> a
  BinOp f a b -> f (eval a) (eval b)
  If discr thenBranch elseBranch -> case eval discr fo
    True -> eval thenBranch
    False -> eval elseBranch
```

No ugly hacks, we can just write it in a direct fashion that makes sense.

2. Data Families

So, what can we do with data families that GADTs don't give us?

For a start, we can make things _open_. As a silly, contrived example,
let's say we want to represent things that have a length.

``` haskell
class Lengthable e
  data Length e
  len :: Length e -> Int

instance Lengthable [()] where
  -- very little worth tracking about a list of units
  data Length [()] = Int
  len i = i

instance Lengthable [a] where
  data Length [a] = [a]
  len = length

```



3. Data Families
