---
layout: post
title: "Tractable concurrency bug reproduction with property tests"
description: ""
category: haskell
tags: []
---

Concurrency bugs can be some of the hardest to figure out. Even once you've made it reproducible on demand,
it's often a very large, cumbersome test case, at which point you're faced with the choice of laboriously
whittling it down manually, or trying to debug it with a much larger dataset than would be preferred.

(Code will be interspersed: a full, buildable repo is at https://github.com/mwotton/qc-concurrency)

First, let's set up a bug:

```
notReallyAtomicModifyIORef ::  IORef a -> (a -> (a, b)) -> IO b
notReallyAtomicModifyIORef ref f = do
  r <- readIORef ref
  let (a,b) = f r
  writeIORef ref a
  pure b
```

While it's fairly clear that this is chock full of races, let's pretend we don't know that yet. We are
suspicious of it enough that we test against an oracle:

```
newtype Iterations = Iterations Int
  deriving Show

instance Arbitrary Iterations where
  arbitrary = Iterations . abs <$> arbitrary

spec = describe "simple tests" $ do
  it "gets the same result in a single thread" $ do
    property $ \(Iterations n) -> do
	  -- we'll use this increment function partially applied.
	  -- we want to be able to supply an increment so that in later
	  -- tests, different threads can have observably different effects.
      let f incr x = (x+incr, x)
      ref1 <- newIORef 1
      ref2 <- newIORef 1
      actualAtomic <- mapM (atomicModifyIORef ref1 . f) [1..n]
      notReallyAtomic <- mapM (notReallyAtomicModifyIORef ref2 . f) [1..n]
      notReallyAtomic `shouldBe` actualAtomic
```


This passes with flying colours. This shouldn't reassure us too much, of course: it's hard to get concurrency
bugs when you interact in a single-threaded way, but it at least tells us that it's not _completely_ wrong.

Let's try a more devious test:

```
withThreadsAndIterations threads iterations f = forConcurrently [1..threads] $ replicateM (fromIntegral iterations) . f

...

  it "fails when we pound it hard" $ do
    let f incr x = (x+incr, x)
        iterations = 50
        threads = 20
    ref1 <- newIORef 1
    ref2 <- newIORef 1
    let runConcurrently = withThreadsAndIterations threads iterations

    actualAtomic    <- runConcurrently $ atomicModifyIORef ref1 . f
    notReallyAtomic <- runConcurrently $ notReallyAtomicModifyIORef ref2 . f
    notReallyAtomic `shouldBe` actualAtomic
```

Hurrah, we have an observable failure! Less hurrah: it's a page of numbers, and I had to manually change the
thread and iteration count to get an error. But why do that when we have computers to do it for us?

```
newtype ConcurrencyLevel = ConcurrencyLevel Int
  deriving Show

instance Arbitrary ConcurrencyLevel where
  arbitrary = ConcurrencyLevel . abs <$> arbitrary

...

  it "fails when we pound it hard, but with a property" $ do
    property $ \(ConcurrencyLevel threads, Iterations iterations) -> do
      let f incr x = (x+incr, x)
      let runConcurrently = withThreadsAndIterations threads iterations

      ref1 <- newIORef 1
      ref2 <- newIORef 1

      actualAtomic    <- runConcurrently $ atomicModifyIORef ref1 . f
      notReallyAtomic <- runConcurrently $ notReallyAtomicModifyIORef ref2 . f
      notReallyAtomic `shouldBe` actualAtomic

```

This works too, but we still have a wall of text from the test!

```
  2) simple tests fails when we pound it hard, but with a property
       Falsifiable (after 20 tests):
         (ConcurrencyLevel 14,Iterations 12)
```

What's going on here? Well, I've deliberately left out the `shrink` operation on the `Iterations` and
`ConcurrencyLevel` newtypes, mostly to show what happens. Let's add the shrink operation back in.

```
instance Arbitrary Iterations where
  arbitrary = Iterations . abs <$> arbitrary
  shrink (Iterations i) = Iterations <$> shrink i

instance Arbitrary ConcurrencyLevel where
  arbitrary = ConcurrencyLevel . abs <$> arbitrary
  shrink (ConcurrencyLevel i) = ConcurrencyLevel <$> shrink i
```

Now we get a smaller error case.

```
 2) simple tests fails when we pound it hard, but with a property
       Falsifiable (after 12 tests):
         (ConcurrencyLevel 9,Iterations 7)

```

Can we do better? Yes! We care about a low concurrency level much more than we care about a low number of iterations,
but there's no direct way to express that to QuickCheck's shrinking algorithm: once it has found a failing
case, it will continue to shrink from that. This is a little awkward, because it might start at a relatively
low number of iterations and not be able to get an error without a higher than necessary concurrency value.

Thankfully, there's an easy fix: if we change the `Arbitrary` instance for `Iterations`, we can add 100 to it.
The shrinker is still happy to shrink below that (and this is a good reason not to rely on fancy definitions for
`Arbitrary` to maintain correctness: as a rule, your correctness properties should always be on your types,
not baked into the generators), but it will give the tests a better chance to settle on the lowest necessary
concurrency.

with this new definition
```

instance Arbitrary Iterations where
  arbitrary = Iterations . (+100) . abs <$> arbitrary
  shrink (Iterations i) = fmap Iterations (shrink i)
```

we get a much better concurrency level:

```
  2) simple tests fails when we pound it hard, but with a property
       Falsifiable (after 7 tests and 7 shrinks):
         (ConcurrencyLevel 2,Iterations 39)
```

Final thing we might want to check: concurrency errors tend to be quite flaky. However, if we
run the whole test 10 times at a given number of iterations and concurrency and it fails every
time, that's probably a reasonable concrete test case to start investigating.

```
  it "property fails all of 10 attempts" $ do
    property $ \(ConcurrencyLevel threads, Iterations iterations) -> do
      let f incr x = (x+incr, x)
      let trials = 10
      let runConcurrently = withThreadsAndIterations threads iterations

      ref1 <- newIORef 1
      ref2 <- newIORef 1

      actualAtomic    <- forM [1..trials] $ \_ ->
        runConcurrently $ atomicModifyIORef ref1 . f
      notReallyAtomic <- forM [1..trials] $ \_ ->
        runConcurrently $ notReallyAtomicModifyIORef ref2 . f

      -- we fail only if _every_ run fails.
      (head actualAtomic, or (zipWith (==) actualAtomic notReallyAtomic))
        `shouldBe` (head notReallyAtomic, True)
```

which gives us

```
 3) simple tests property fails all of 10 attempts
       Falsifiable (after 7 tests and 5 shrinks):
         (ConcurrencyLevel 2,Iterations 45)
```


# Conclusion

What can we take away from this?

1. This is a fairly efficient, low effort method for capturing nasty concurrency bugs.
2. Shrinking is often neglected, but is very important for producing tractable test cases.
3. Arbitrary instances should be as simple as possible, and encode no correctness requirements: the place
for that is in the datatype, whether through smart constructors or something like [refined](https://hackage.haskell.org/package/refined).
4. It is however totally ok to fiddle with the generator to guide search, rather than to avoid cases
where you know it's going to break.
5. Because QuickCheck will shrink your inputs in order, it's important to put the one you care about
minimising most first in the tuple.

and perhaps a meta-point:

6. This test might be very short-lived. Once you actually find the bug, there's a good chance you can
capture it in a much quicker unit test, and that might be how you want to prevent regressions. I used this
technique recently to find a [bug](https://github.com/morphismtech/squeal/pull/139) in Squeal, and the test
took a full minute to find the problem. This is more a way to explore the dynamic behaviour of your program
than it is a permanent test.
