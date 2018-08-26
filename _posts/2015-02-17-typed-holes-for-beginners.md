---
layout: post
title: "Typed Holes for beginners"
description: ""
category:
tags: [haskell]
---

Typed holes are a feature GHC borrowed from Agda.
They are incredibly useful for incremental development, making it
really easy to go from correct (but incomplete) state to a more
complete version.

Unfortunately, Austin Seipp's
[tutorial](https://wiki.haskell.org/GHC/Typed_holes) assumes knowledge
of type classes and to some extent free monads. This isn't necessary,
and can be off-putting to beginners, which is a shame: I've found them
one of the easiest ways for newbie Haskellers to break down problems
iteratively. One of the worst feelings you can get learning a language
is getting stuck and not knowing how to unstick yourself: following
hole-driven dev when you aren't quite sure what you're doing is a
great way to avoid getting lost in the weeds.

In aid of that, I've taken a very easy exercise and shown how you
might solve it in microscopic detail. I deliberately don't use pattern
matching, user datatypes or anything beyond the absolute basics.

Here, we set up the problem - in practice, we would not name g,g2...
but just change in-place. I've named them all to make it clear.

In our first version, we'll just name the arguments to the function
and insert a hole for the result.

The problem we are trying to solve: given a function that takes an 'a'
to a 'b', and a pair with an 'a' and a 'c', return a 'b' and a 'c'.

<pre>
g,g2,g3,g4,g5,g6,g7 :: (a -> b) -> (a, c) -> (b, c)

g x y = _foo
</pre>


<pre>
src/Text/FastEdit.hs:56:9:
    Found hole ‘_foo’ with type: (b, c)
    Where: ‘b’ is a rigid type variable bound by
               the type signature for g :: (a -> b) -> (a, c) -> (b, c)
               at src/Text/FastEdit.hs:55:24
           ‘c’ is a rigid type variable bound by
               the type signature for g :: (a -> b) -> (a, c) -> (b, c)
               at src/Text/FastEdit.hs:55:24
    Relevant bindings include
      y :: (a, c) (bound at src/Text/FastEdit.hs:56:5)
      x :: a -> b (bound at src/Text/FastEdit.hs:56:3)
      g :: (a -> b) -> (a, c) -> (b, c)
        (bound at src/Text/FastEdit.hs:56:1)
    In the expression: _foo
    In an equation for ‘g’: g x y = _foo
</pre>

If we look at the type GHC has found for _foo, clearly our result is
going to be a pair: let's split our hole up.


<pre>
g2 x y = (_foo,_bar)
</pre>

<pre>
src/Text/FastEdit.hs:57:11:
    Found hole ‘_foo’ with type: b
    Where: ‘b’ is a rigid type variable bound by
               the type signature for g2 :: (a -> b) -> (a, c) -> (b, c)
               at src/Text/FastEdit.hs:55:24
    Relevant bindings include
      y :: (a, c) (bound at src/Text/FastEdit.hs:57:6)
      x :: a -> b (bound at src/Text/FastEdit.hs:57:4)
      g2 :: (a -> b) -> (a, c) -> (b, c)
        (bound at src/Text/FastEdit.hs:57:1)
    In the expression: _foo
    In the expression: (_foo, _bar)
    In an equation for ‘g2’: g2 x y = (_foo, _bar)


src/Text/FastEdit.hs:57:16:
    Found hole ‘_bar’ with type: c
    Where: ‘c’ is a rigid type variable bound by
               the type signature for g2 :: (a -> b) -> (a, c) -> (b, c)
               at src/Text/FastEdit.hs:55:24
    Relevant bindings include
      y :: (a, c) (bound at src/Text/FastEdit.hs:57:6)
      x :: a -> b (bound at src/Text/FastEdit.hs:57:4)
      g2 :: (a -> b) -> (a, c) -> (b, c)
        (bound at src/Text/FastEdit.hs:57:1)
    In the expression: _bar
    In the expression: (_foo, _bar)
    In an equation for ‘g2’: g2 x y = (_foo, _bar)

</pre>

Focusing on _bar first, we know that it's got type 'c'.
Now we have to look at the relevant bindings: we need to find some way
of getting a 'c' out of them. To do this, we look at the final return
type of each. x doesn't mention c, so we can ignore it. g2 does, but
it also needs to have a c passed in, so it can't be the ultimate
source. That leaves y. It's not quite the right type, but let's pick
that out as an argument and update the hole.

<pre>
g3 x y = (_foo,_bar y)
</pre>


<pre>
src/Text/FastEdit.hs:58:11:
    Found hole ‘_foo’ with type: b
    Where: ‘b’ is a rigid type variable bound by
               the type signature for g3 :: (a -> b) -> (a, c) -> (b, c)
               at src/Text/FastEdit.hs:55:24
    Relevant bindings include
      y :: (a, c) (bound at src/Text/FastEdit.hs:58:6)
      x :: a -> b (bound at src/Text/FastEdit.hs:58:4)
      g3 :: (a -> b) -> (a, c) -> (b, c)
        (bound at src/Text/FastEdit.hs:58:1)
    In the expression: _foo
    In the expression: (_foo, _bar y)
    In an equation for ‘g3’: g3 x y = (_foo, _bar y)

src/Text/FastEdit.hs:58:16:
    Found hole ‘_bar’ with type: (a, c) -> c
    Where: ‘a’ is a rigid type variable bound by
               the type signature for g3 :: (a -> b) -> (a, c) -> (b, c)
               at src/Text/FastEdit.hs:55:24
           ‘c’ is a rigid type variable bound by
               the type signature for g3 :: (a -> b) -> (a, c) -> (b, c)
               at src/Text/FastEdit.hs:55:24
    Relevant bindings include
      y :: (a, c) (bound at src/Text/FastEdit.hs:58:6)
      x :: a -> b (bound at src/Text/FastEdit.hs:58:4)
      g3 :: (a -> b) -> (a, c) -> (b, c)
        (bound at src/Text/FastEdit.hs:58:1)
    In the expression: _bar
    In the expression: _bar y
    In the expression: (_foo, _bar y)
</pre>

Progress! Again focusing on the second error message, we now need a
function from (a,c) to c. The relevant bindings don't seem to help us
here, so we look up (a,c) -> c on
[hoogle](https://www.haskell.org/hoogle/?hoogle=%28a%2Cc%29+-%3E+c) -  the first result is snd,
which seems about right.

<pre>
g4 x y = (_foo,snd y)
</pre>

<pre>
src/Text/FastEdit.hs:59:11:
    Found hole ‘_foo’ with type: b
    Where: ‘b’ is a rigid type variable bound by
               the type signature for g4 :: (a -> b) -> (a, c) -> (b, c)
               at src/Text/FastEdit.hs:55:24
    Relevant bindings include
      y :: (a, c) (bound at src/Text/FastEdit.hs:59:6)
      x :: a -> b (bound at src/Text/FastEdit.hs:59:4)
      g4 :: (a -> b) -> (a, c) -> (b, c)
        (bound at src/Text/FastEdit.hs:59:1)
    In the expression: _foo
    In the expression: (_foo, snd y)
    In an equation for ‘g4’: g4 x y = (_foo, snd y)
</pre>

We've finished the second hole, so we focus on _foo now.
Looking at the relevant bindings, we can dismiss y because it doesn't
return a 'b'. x looks like the most relevant piece, but it is clear we
need to pass something into it, because it's a function. We don't know
what yet, so we let the hole do the work.

<pre>
g5 x y = ( x _foo,snd y)
</pre>

<pre>
src/Text/FastEdit.hs:60:14:
    Found hole ‘_foo’ with type: a
    Where: ‘a’ is a rigid type variable bound by
               the type signature for g5 :: (a -> b) -> (a, c) -> (b, c)
               at src/Text/FastEdit.hs:55:24
    Relevant bindings include
      y :: (a, c) (bound at src/Text/FastEdit.hs:60:6)
      x :: a -> b (bound at src/Text/FastEdit.hs:60:4)
      g5 :: (a -> b) -> (a, c) -> (b, c)
        (bound at src/Text/FastEdit.hs:60:1)
    In the first argument of ‘x’, namely ‘_foo’
    In the expression: x _foo
    In the expression: (x _foo, snd y)
</pre>

On the home straight now! We have an 'a' hidden inside y:

<pre>
g6 x y = ( x (_foo y) ,snd y)
</pre>


<pre>
src/Text/FastEdit.hs:61:15:
    Found hole ‘_foo’ with type: (a, c) -> a
    Where: ‘a’ is a rigid type variable bound by
               the type signature for g6 :: (a -> b) -> (a, c) -> (b, c)
               at src/Text/FastEdit.hs:55:24
           ‘c’ is a rigid type variable bound by
               the type signature for g6 :: (a -> b) -> (a, c) -> (b, c)
               at src/Text/FastEdit.hs:55:24
    Relevant bindings include
      y :: (a, c) (bound at src/Text/FastEdit.hs:61:6)
      x :: a -> b (bound at src/Text/FastEdit.hs:61:4)
      g6 :: (a -> b) -> (a, c) -> (b, c)
        (bound at src/Text/FastEdit.hs:61:1)
    In the expression: _foo
    In the first argument of ‘x’, namely ‘(_foo y)’
    In the expression: x (_foo y)
</pre>

We could use hoogle again, but we happen to remember that 'fst' can be
used to pick out the first element of a pair:


<pre>
g7 x y = (x (fst y) ,snd y)
</pre>


and we're done.


I realise this may seem like a huge amount of work for a fairly
trivial function, but the important point is that we never got
off-track: if GHC gives us an error rather than information about a
hole, we don't even try to fix it: we just roll back to our last good
state and try again. We were never lost in the weeds; we moved
relentlessly forward. In practice, experienced Haskellers will have
something like [ghc-mod](http://www.mew.org/~kazu/proj/ghc-mod/en/)
running so they can get faster feedback in-editor, but we can still do
it with just an editor and an open GHCi.


ADDENDUM
========

As @benno37 pointed out, it's the underscore that makes it a typed
hole. This is quite clever really - it is _only_ considered a hole if
it would otherwise be a scope error, so if for some misbegotten reason
you decide to name a parameter to a function "_foo", it will remain a
normal variable.
