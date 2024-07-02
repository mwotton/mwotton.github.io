---
layout: post
title: "MiniTSis 6.0.1 release"
description: "first public release of MiniTSis"
category: tech
tags: [Typescript, MiniTSis, web]
---

# MiniTSis 6.0.1 release

I'm happy to announce the first public release of [MiniTSis](https://www.npmjs.com/package/minitsis), a property testing library for TypeScript with integrated shrinking and a test database.

## What is MiniTSis?

I recently worked on a large, complex client project and had to replicate some functionality in a different environment. Given that I had to match the existing implementation, a property test suite with an oracle seemed the right approach, so I built the first version using [fast-check](https://fast-check.dev/). Unfortunately, while it did work for finding bugs, the shrinking in fast-check is not integrated, which means you end up in the situation of knowing you have a problem in your implementation, but not being able to do anything about it without reading hundreds of lines of JSON.

After complaining vociferously, [David](drmaciver.com) gently suggested that I stop whinging and just port [Minithesis](https://github.com/DRMacIver/minithesis). This is the result of that nudge, and I don't think I'd have been able to finish the client project without it. Getting a fully-shrunk failing test case makes tracking down bugs far easier, and having a test database means that once MiniTSis has finished shrinking a failing test case, it will be automatically saved and tested until it passes again.

The shrinking algorithm is bitstream-based, which means that it can't directly take advantage of the structure of generators. I may implement boundary tracking at some point: this would be for performance rather than correctness, though.

Bug reports and PRs at the [GitHub repo](https://github.com/lambdamechanic/miniTSis)
