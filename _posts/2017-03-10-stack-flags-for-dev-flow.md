---
layout: post
title: "cabal flags for dev flow"
description: ""
category:
tags: [haskell]
---


It's good practice to make sure your code is warning-free before
committing or releasing. In the throes of editing, though, it can be
extremely annoying to have to fix a swathe of trivial warnings when
you really just want to see if things basically make sense (especially
if you're fond of undefined-driven development, and are using a
prelude which marks undefined as deprecated).

I used to just edit the cabal file to add and delete "-Werror"
according to what mode I was in, but this was both annoying to do, and
potentially incorrect: if you add -Werror in when your code is already
built, cabal will not automatically rebuild local modules.

Thankfully, cabal flags _do_ force a rebuild. This means you just need
this at the top level in mycoolproject.cabal:

```
flag lib-Werror
  default: True
  manual: True
```

and this in your executable & library stanzas:

```
  if flag(lib-Werror)
    ghc-options: -Werror
```

Now, when you want to run cajun style, you can edit your command line
to something like

```
stack build --flag mycoolproject:-lib-Werror
```

turning this into a convenient makefile action is left as an exercise
to the reader - extending to hpack-style is similar.

(This tiny but pleasant tip is due
to [Michael Xavier](https://twitter.com/mxavier).)
