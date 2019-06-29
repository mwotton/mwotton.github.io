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

  it "fails when we pound it hard" $ do
    let f incr x = (x+incr, x)
        iterations = 50
        threads = 20
    ref1 <- newIORef 1
    ref2 <- newIORef 1
    let withThreadsAndIterations f = forConcurrently [1..threads] $ replicateM iterations . f

    actualAtomic    <- withThreadsAndIterations $ atomicModifyIORef ref1 . f
    notReallyAtomic <- withThreadsAndIterations $ notReallyAtomicModifyIORef ref2 . f
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
    property $ \(Iterations iterations, ConcurrencyLevel threads) -> do
      let f incr x = (x+incr, x)
      let withThreadsAndIterations f = forConcurrently [1..threads] $ replicateM iterations . f

      ref1 <- newIORef 1
      ref2 <- newIORef 1

      actualAtomic    <- withThreadsAndIterations $ atomicModifyIORef ref1 . f
      notReallyAtomic <- withThreadsAndIterations $ notReallyAtomicModifyIORef ref2 . f
      notReallyAtomic `shouldBe` actualAtomic

```

This works too, but we still have a wall of text from the test!

```
simple tests fails when we pound it hard, but with a property
       Falsifiable (after 42 tests):
         (Iterations 37,ConcurrencyLevel 32)
       expected: [[2,4,6,8,...
```

What's going on here? Well, I've deliberately left out the `shrink` operation on the `Iterations` and
`ConcurrencyLevel` newtypes, mostly to show what happens. Let's add the shrink operation back in.

```
instance Arbitrary Iterations where
  arbitrary = Iterations . abs <$> arbitrary
  shrink (Iterations i) = fmap Iterations (shrink i)

instance Arbitrary ConcurrencyLevel where
  arbitrary = ConcurrencyLevel . abs <$> arbitrary
  shrink (ConcurrencyLevel i) = fmap ConcurrencyLevel (shrink i)
```

Now we get an error much more quickly:

```
  test/Main.hs:62:7:
  2) simple tests fails when we pound it hard, but with a property
       Falsifiable (after 49 tests and 7 shrinks):
         (Iterations 33,ConcurrencyLevel 5)

```

Final thing we might want to check: concurrency errors tend to be quite flaky. However, if we
run the whole test 10 times at a given number of iterations and concurrency and it fails every
time, that's probably a reasonable concrete test case to start investigating.

```
  it "property fails all of 5 attempts" $ do
    property $ \(Iterations iterations, ConcurrencyLevel threads) -> do
      let f incr x = (x+incr, x)
      let trials = 10
      let withThreadsAndIterations f = forConcurrently [1..threads] $ replicateM iterations . f

      ref1 <- newIORef 1
      ref2 <- newIORef 1

      actualAtomic    <- forM [1..trials] $ \_ ->
        withThreadsAndIterations $ atomicModifyIORef ref1 . f
      notReallyAtomic <- forM [1..trials] $ \_ ->
        withThreadsAndIterations $ notReallyAtomicModifyIORef ref2 . f

      -- we fail only if _every_ run fails.
      (head actualAtomic, or (zipWith (==) actualAtomic notReallyAtomic))
        `shouldBe` (head notReallyAtomic, True)
```

which gives us

```
3) simple tests property fails all of 5 attempts
       Falsifiable (after 39 tests and 2 shrinks):
         (Iterations 34,ConcurrencyLevel 15)
```

So: now we know that we can fairly reliably reproduce the bug at that number of iterations and concurrency
level. You can push this in lots of directions - for instance, maybe it's more important to limit the
concurrency than the runtime, so you might fix the iterations to something high and see how low the
concurrency level can go (at 100 iterations, I can provoke the bug at concurrency level 2!).

The point of all this is that you can use your test suite as a way to explore the dynamic state space of
your program and make it easy to find bugs. This is a very simple example, but I've used a similar technique
to find database connection pooling bugs.
