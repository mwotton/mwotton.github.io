---
layout: post
title: "haskell profiling without pain"
description: ""
category:
tags: [haskell]
---

In general, stack is pretty good at caching build artifacts - the
first build might be glacial, but incremental builds are snappy. This
all goes out the window when you start playing with flags like
--profile, though - stack sees that something has changed and
dutifully rebuilds every-bloody-thing. This is a huge disincentive to
casually run profiling builds.

thankfully, stack keeps all its bits and bobs inside ./.stack-work, so
all you need to do is create a shadow directory like so:

```
mkdir foo-prof
cd foo-prof
lndir ../foo
cd foo-prof
rm -rf .stack-work
stack build --profile
```

(on ubuntu, you can get lndir from xutils-dev.)

now, you have a directory with a separate .stack-work, and can mess around
in that directory for all your profiling needs without slowing
everything else down to a shuddering halt.

(nb: this will not cope terribly gracefully with new files being
added. a workaround is to only symlink the top level files and
directory instead of a full shadow tree: that way, only top level
changes will break things.)

(another possibility would be to set STACK_WORK=.stack-work-profiling
whenever you run profiling commands, but you're going to screw it up
eventually - probably better to have totally separate implicit contexts.)
