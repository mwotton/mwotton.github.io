---
layout: post
title: "Haskell Testing Desiderata"
description: ""
category:
tags: []
---

A few days ago I wrote about
[testing in Haskell](https://shimweasel.com/2016/10/24/fast-tests-and-static-languages).
It's not quite the whole story, though, and I wanted to dig into what
I'd like to see.

# Current solutions

I currently use Intero in Emacs, and switch between ```ghcid``` and ```stack build --file-watch
--test --fast```. None of them is quite perfect, as I'll describe.

# Desiderata

## Interpreter-based

We are aiming for subsecond response times. In a big project (such as
my current [betterteam](https://www.betterteam.com/) work project, we
have about 21000 lines of Haskell, we just don't have time to wait for
the compiler to reload almost everything. Intero and ghcid work well
here, stack build does not.

## Responsivity

By responsivity, I mean that when _anything_ that could affect the
semantics of the code changes, the relevant tests are fired. This is
really important, because you need to be able to trust that your
test-runner is telling you the full, up-to-date truth. Without this,
you need to keep the possibility in your mind that your tools have
started to lag the truth of the situation.

Unfortunately, the ghcid solution I mentioned yesterday doesn't
handle this case quite perfectly: if I have a large project with
sub-projects set up through [stack](https://www.haskellstack.org/) and
change something in a subproject, ghcid will not notice the problem.
(It has improved recently, in that it watches stack.yaml and the
appropriate cabal file, but widespread damage detection is a little
outside its remit.)

[Intero](https://github.com/commercialhaskell/intero/) has a similar
problem, and while I find it really useful, I frequently have to
restart it so it will notice added packages in the stack.yaml and cabal
file, or subproject changes.

In contrast, ```stack build --file-watch --test --fast```, my other go-to,
handles this flawlessly.

## Parsimony

We shouldn't do more work than is necessary. This also sounds pretty
obvious, but the file-based nature of Haskell code militates against
us here: if module A depends on module B, and we changed a line of
code in module B that module A does not in fact use, it's difficult to
tell that not only does module A not need to be recompiled, but its
tests do not need to be rerun. Barring the introduction of something
like [Unison's](https://unisonweb.org/)
addressing-by-hash-of-toplevel-value approach, we're going to have to
cop this slowdown, and just try to keep our files small & coherent
enough that a minimum of pointless work needs to be done.

As far as just loading the code goes, the interpreter approaches do
ok (unless you have significant Template Haskell, in which case heaven
help you.) The compilation approach is usually pretty slow, at least if we
keep the subsecond goal in mind.

I haven't seen anything capable of only running the tests that
have been affected. The closest is
[Hspec's](https://hspec.github.io/) --rerun flag, which keeps a running
tally of failing tests and only runs them until they all pass. This
helps a lot, especially if you haven't put the work in to make your
tests fast (we have a fair number of integration tests that interact
with the database, and currently it takes 180s to run everything).

(Quick plug: if you use the ```stack build --file-watch --test
--fast``` approach, --rerun doesn't work out of the box, as it relies on
environment variables. My little
[hspec-stack-rerun](https://github.com/mwotton/hspec-stack-rerun)
project works around this: with any luck hspec's upcoming support for
config files will make it unnecessary soon.)

# Fantasyland project

In the world I see when I shut my eyes, I have a process watching for
file changes that can compute a parsimonious set of code to be
reloaded and tests to be run. That would talk to a running ghci
process (probably something similar to ghcid or intero), instructing it first
which files to reload, and then which tests to run. It might even be
aware of subprojects to the extent that it would only run failing
tests in the lowest level subproject until it passes, to let you stay
drilled in, then run whatever tests have been possibly compromised at
higher levels until everything passes. I want my testrunner to be
Tenzing Norgay, helping me through the rough bits and making up for my
laziness, sloppiness and impatience.

Then I open my eyes, sigh, and hit M-x intero-restart.
