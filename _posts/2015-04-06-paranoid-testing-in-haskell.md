---
layout: post
title: "Paranoid Testing in Haskell"
description: ""
category:
tags: [haskell testing]
---
{% include JB/setup %}

How we know our code does the right thing?

The pat, Haskell-y answer is "types", but obviously this doesn't cover
everything, or we wouldn't bother with quickcheck and tests at all.

Obviously we write tests at the app level, but implicitly, we are also
depending on all the code that we import. The particular combination
of library versions Cabal has picked might be unique to your build,
and an equally unique bug might lurk in that combination.

This happened to me recently: a case-insensitive literal string parser
(stringCI) was [broken](https://github.com/bos/attoparsec/issues/99) in a few
versions, and my [robots-txt](https://github.com/meanpath/robots)
library was silently broken: I didn't find out until my CI ran for
[unrelated reasons](https://travis-ci.org/meanpath/robots/builds/56539308).
Stackage wouldn't have solved that problem - Attoparsec's tests still
passed, they just missed an important case. It only showed up in the
context of robots-txt's tests.

Upper bounds might have alleviated this slightly, but manually
managing the upper bounds down to a minor version and continually
updating them every time a fix comes in is tedious in the extreme, and
in any case, doesn't cover the case that a particular combination of
libraries triggers the bug. This is what makes it almost impossible to
test for at the library level: there are way too many ranges of
library versions you're expected to support. Combinatorial explosion
sets in very quickly. However, this is eminently possible at the app
level: for any given build, there's an explicit set of versioned
packages already chosen.

What I'd like to propose is a command (maybe integrated with cabal?)
that would take the current set of package versions (possibly as a
cabal sandbox), and for each dependency, install and run its tests in
a clean copy of that sandbox. A copy is necessary because while we
can expect the libraries to play well together in terms of building
cleanly, we can't expect their test dependencies to be so
well-behaved: no-one could cavil if you decided you wanted a particular version of
hspec or quickcheck.

In the concrete case of my app, depending on robots-txt and
transitively on attoparsec: if I had run
robots-txt's tests in the context of the version of attoparsec picked
by cabal, I would have found the problem immediately.

This wouldn't be a quick operation, but
computer time is cheaper than human time. It also wouldn't need
to be run every time you deploy - only when you update the frozen
dependencies (which we are all doing anyway, right?).

The output of this could also be fed into the anonymous build
reporting that's coming with [Hackage
2](https://github.com/haskell/hackage-server/issues/44) for a fairly
massive set of test data to augment the already-planned build data
collection.
