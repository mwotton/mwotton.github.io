---
layout: post
title: "Fixing cabal build failures"
description: ""
category:
tags: [haskell]
---

Fixing cabal build failures
===========================

It is relatively common for library maintainers to have strict upper
bounds on their dependencies. This prevents the nasty problems I talked
about
[yesterday](https://shimweasel.com/2015/04/06/paranoid-testing-in-haskell/).
Unfortunately it also means that when new versions of that dependency
come out, you can get some unnecessary build failures from cabal, if
you're depending on that new version but the maintainer for your other
dependency hasn't raised the upper bounds yet.

Before we had cabal sandboxes, this was a major hassle. I have a
swathe of libraries on hackage with -mwotton or -buildfix appended
just so I could do my work. Now we have a better alternative: cabal
sandbox add-source.

This came up again recently while talking about Arion
