---
layout: post
title: "you're holding LLMs wrong"
description: ""
category:
tags: [LLM]
---

LLM wrappers are bad and you shouldn't build them. There are two main reasons:

- prompt leakage techniques means that the prompt you put all that effort into honing is no moat at all to a determined attacker with fifteen minutes to spare.
  You can't reasonably defend against this: as Manuel puts it, LLMs are complex machines and complex machines can be hacked. https://twitter.com/TacticalGrace/status/1684864520919891968?s=20
- even if you manage to guard your prompt, you still have no way to make sure that your app is not saying something offensive, wrong, or libellous to the user.

Reinforcement learning  isn't enough to get around either of these problems. There exist multiple techniques for phrasing requests that the underlying model understands but the RL part doesn't (street slang, base64 encoding etc). If your output is unconstrained text, what possible tools do you have for quality control of the output, except layering more LLMs and hoping there isn't an input that fools all of them? Making systems more complex is rarely a reliable path to safety.

Ok, then. We have an exciting new technology, but we can't build things on it directly because it is unreliable. What options do we have?

One big clue can be gleaned from working with humans: they build useful things all the time, despite being biased, limited, spiteful and misinformed. We do this by using a
variety of gating techniques: automated tests, type systems and proofs, benchmarks, and even informal human feedback. Essentially, if we can constrain the space of the
answer sufficiently, any candidate answer will have just two possibilities: rejected, or accurate. Incidentally, this probably explains why tools like Copilot and the use
of ChatGPT for coding seems to be the most useful current use of LLMs: it's piggybacking on the existing systems of validation that have been set up to try to extract
working code out of us fallible humans. Even just a light manual test, or a quick scan of the code in a PR is usually enough to validate that it's at least directionally
correct, and not embedding a bitcoin miner in a survey page. (Parenthetically, I think this also changes the economics of using static types for coding quite significantly.
It can sometimes take a person a long time to demonstrate a particular property in the type system, to the extent that validating with a property test or unit test is a better
approach. If you can formulate the constraint in the types, though, it doesn't really matter if it takes the LLM ten attempts to get it right, so long as it gets there.)

This is also an echo of a correctness technique used extensively in computational complexity proofs. It's very common for checking an answer to be significantly cheaper
and simpler than coming up with it in the first place.

The common thread here is that we change the failure modes from potentially exposing core IP or giving the user potentially dangerous advice, to simply not providing an answer if it isn't accepted
by a ruthless, simple-minded decision procedure. Failure to progress is annoying, of course, and there are plenty of places where you couldn't deploy such a fallible system, but _using_
such a system to produce other systems that _do_ pass the stringent correctness requirements is perfectly acceptable.


The answer (at least in a programming context) is to use correctness techniques like tests, benchmarks, types and other-human feedback to ensure that the answer we are getting out has some value and isn't malicious. Similarly, it's a pretty common split in computational complexity proofs for checking the validity of an answer to be significantly easier than coming up with it in the first place. Therefore, I think the best way to use LLMs currently is to set up a strict correctness checking routine, ideally one that can provide contextual feedback to the LLM. Type checkers are one easy model here, especially since the output formatting is likely already in the LLM's training set, but anything that can render an error in sensible text will do.
