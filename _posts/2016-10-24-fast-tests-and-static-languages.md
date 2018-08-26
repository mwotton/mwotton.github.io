---
layout: post
title: "Fast tests and static languages"
description: ""
category:
tags: [haskell]
---

This is to correct a small error in David R. Maciver's piece on [static
typing](http://www.drmaciver.com/2016/10/static-typing-will-not-save-us-from-broken-software/)

Most of the piece is sensible - I just wanted to dispel an apparently
widespread misunderstanding about build times.

```
There are absolutely statically typed languages where build times
are reasonable but this tends to be well correlated with them having
bad type systems. e.g. Go is obsessed with good build times, but Go is
also obsessed with having a type system straight out of the 70s which
fights against you at every step of the way. Java’s compile times are
sorta reasonable but the Java type system is also not particularly
powerful. Haskell, Scala or Rust all have interesting and powerful
type systems and horrible build times. There are counter-examples –
OCaml build times are reportedly pretty good – but by and large the
more advanced the type system the longer the build times.
```

Clearly David is setting up a tradeoff, but the tradeoff is not a real
one. Yes, if you run your tests by compiling your code then running
it, it can be slow: my [robots-txt library](https://github.com/meanpath/robots)
takes about 8 seconds on my machine to
build and run the 229 tests. However, every modern static language
worth its salt has an interpreter, same as every dynamic language, and
it is generally much faster to run your tests through that and skip
all the time taken by codegen and linking (unless the
tests themselves take a long time, which is something I can't help you
with.)

If i instead run ghcid, using the invocation from the makefile:

```
	ghcid --warnings --test=:main
```

it will watch my files, and run them whenever something changes.
If I change the actual code, it takes 0.18s or so: just changing the
test takes 0.04s. If I change robots.cabal or stack.yaml, it does take
a few seconds to reload everything: fiddling with those is rare enough
that I'm not too worried. This is if anything much faster than my
experience of running similar systems under Ruby.

None of this is intended to be dumping on David. There's no reason
he'd know about what is very much common practice in the community:
contrary to the common perception of Haskellers being unable to shut
up about the language, I think we have a bad habit of only discussing
the high-tech stuff (Free Monads! Phantom Types! Template Haskell!)
and ignoring the normal practical things that make developing
pleasant.

If this has been at all illuminating to you, please consider blogging
about your praxis too. Maybe I'll learn something.
