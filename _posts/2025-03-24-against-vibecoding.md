---
layout: post
title: "Against Vibecoding (Sorta)"
description: ""
category:
tags: [llm]
---

I've been writing code professionally for something like 25 years, which I think classifies me as a greybeard if not an outright grognard at this point. As such, it's almost expected that I have a grumpy rant about Kids These Days and their Gee Pee Tees and Claudes. And I do, sorta.

The thing is, a hackable framework for using LLMs for development is easily the most transformative change for me since I discovered real type systems, and potentially might be even better. I am, as they say, forced to stan.

## So what's wrong with vibecoding, anyway?

The problem with a capable tool is that it lets you make bigger messes. Once you vibecode yourself up an app that's bigger than the model can handle, you will hit the same wall that every dev hits eventually: that you don't understand what's going on, and you're afraid of making changes because you might break it. You'll just hit it way later, at a point where the app is beyond the model, by definition, and certainly beyond its putative author, who has been lounging around eating grapes and barking occasional complaints.

## What's it good for?

Prototyping. Personal apps. Go nuts, the upside you get from these things is almost all gravy and the downside is minimal. If a personal app goes down there's exactly one person who cares. Vibecoding is an absolutely wonderful tool for exploring a problem space, but you really do have to throw the prototype away afterwards.

## What's the alternative?

This will sound terribly boring and square, but it's using LLMs and agent frameworks alongside standard software development practices.

### Types!

I expect typed languages to become even more popular with the advent of LLMs. The tradeoff has always been about how hard it can be to get a type-correct program together, but having a strict judge to check the output of a tireless LLM is all carrot, no stick. Whatever environment you're in, you can probably add some basic types: Python has type hints, JavaScript has throwing your app out and rewriting it in TypeScript. I'm a fan of more advanced type systems like Haskell's but there's a lot of bang-for-buck in the basic stuff.

### Tests!

One of the orthodox development practices more honoured in the breach than the observance is test-first development, but it doesn't have to be. It's easier than ever to do: just ask the agent to write the tests and a stub, and see them fail before you ask it to write code that fits them.

The great thing about having tests with descriptive error messages is that the agent can probably actually solve the problem itself. The payoff isn't some wispy idea of software virtue, and of eventual speed: the test you write now will help you in two minutes' time, and forever after.

In that vein, let me heavily recommend generative testing, a la [Hypothesis](https://hypothesis.works/). I use (and ported) [MiniTSis](https://github.com/lambdamechanic/miniTSis) but that's only because I was working in TypeScript at the time, if you're using Python then just use the progenitor. (Quick precis for the uninitiated: if you can state some properties and provide a generator for inputs, a generative testing tool can not only try thousands or millions of test cases against your code, it can reduce the test case to something you (or an LLM) might actually get some insight out of reading. Do not use QuickCheck-derived tools if you can avoid it, having an integrated shrinker and a test database like Hypothesis is a massive quality-of-life improvement.)

### Packaging!

Whatever environment you're in, set your project up as a proper packaged project from the start. You're going to add package scripts and dependencies and tests as you go along, you might as well have them there and not have to move them around later. (Aider-driven development can have a tendency to have trouble moving files around - it's not a primitive to say "rename this file", all the LLM can do is SEARCH/REPLACE the content to a new file, and SEARCH/REPLACE the old one with the empty string. Start it in the right place if you can.)

### For the love of God, source control.

Don't lose your code to a bad prompt. The tool I'm about to discuss will by default make each edit into a commit. If you're collaborating with others you may want to squash those commits but that's a detail.


## How I use LLMs for coding

The nitty gritty! My favourite tool at the moment is [Aider](https://aider.chat). It's not as flashy as Cursor, but I've found using a terminal-based tool far more hackable & usable, and it is completely open: you can use whatever model you want and the source is there if you want deep customisations.

### Use /architect mode

The only* model that seems any good at editing using Aider's SEARCH/REPLACE model is Sonnet 3.5, but some of the other models (Deepseek R1, grok-beta, o3-mini) are a bit smarter at actual code. If you're writing simple code then probably Sonnet 3.5 by itself is fine. You can do this by running `aider --architect --model openrouter/x-ai/grok-beta --editor-model openrouter/anthropic/claude-3.5-sonnet`, modify as appropriate.

* edit: several hours after I published this, deepseek v3 0324 came out, and it is about as good as Sonnet at editing, and much cheaper. Life moves pretty fast.

### Use openrouter.ai

You might have noticed that in the previous tip, my models have a suspicious prefix. Everyone and their dog has a model and a billing department: save yourself a little hassle and use [OpenRouter](https://openrouter.ai). It's the same price, for certain models there are actually multiple providers and you can choose on the openrouter site whether you care more about latency or cost.

Getting a single bill rather than four and being able to try models as soon as they come out is also great.

### Keep your context slim

Aider will only send the files you add, though it will occasionally use repository context to ask for more files to be added. You want to keep the chat context and the files in scope as minimal as you can: this keeps responses snappy and bills down.

In aid of that, I'll frequently run `/reset`, `/run exe command_to_run_my_tests`, and `/load .LOADCOMMANDS`. This drops the context and calls a little shell script called `exe` you can grab [here](https://gist.github.com/mwotton/bbd198f8ed76c231e0fb0eb644d07fb2) - all it does is look for editable files in the output of your command, and plunk them into `.LOADCOMMANDS` in the root folder of your project. `/load .LOADCOMMANDS` will then load any mentioned files into your context (so make sure your tests & compile outputs are explicit about what happened and where!)

This workflow is a little janky, and I think eventually if Aider doesn't offer some internal scripting, I'll switch to writing my own REPL using `aider --message` as the engine. It works for now, though.

### Don't spam "What's wrong? Fix" _too_ often

When your test run ends with a failure, Aider will frequently fill out "What's wrong? Fix" as a sample response.

It can be pretty tempting to just sit and pull the handle over and over, hoping you get a clean compile and test run. It even works often enough to be attractive! A big part of using these tools effectively, however, is noticing when they're stuck and jumping in to give some context. Often it's as easy as running the appropriate `/web` command, to give it access to a particular page in the documentation.

### Use /undo like it's going out of fashion

Frequently, you'll see the model merrily going down the wrong path. Don't try to repair the damage, just run `/undo` and provide a better prompt: the first rule of finding yourself in a hole is to stop digging.

### Keep an eye on the leaderboards & try out models

The [Aider](https://aider.chat/docs/leaderboards/) leaderboards are a really helpful tool for model selection. You probably won't change your models daily but they do have strengths and weaknesses. For simple code, Sonnet 3.5 is absolutely rock-solid. It's still good at the more complex stuff but I sometimes find grok-beta and R1 better. It can be quite personal, because it's going to respond to your prompting style, so play around.

### Voice, maybe?

I haven't used this as heavily as the rest of it, but voice coding is looking very promising. It doesn't really matter any more that you can't put in every weird character under the sun if you can just get the gist across to the model. This one's an area of active experimentation for me, play with `/voice` if you'd like more.

## So what's your point?

I don't actually expect the juniors to read this, and they don't need to anyway. There are a million articles and books on how to develop software systematically, and nobody reads them until they have a project blow up: Icarus has to lose his wings before he takes glue composition seriously.

I'm talking to my fellow grognards: this is not something you can safely ignore, like scrum or crypto. Coding agents are good today, and will only get better. The good news is that while some of your skills are obsolete, many are not: it's still critical to notice when it's going off track and to have the depth in the field to nudge it into a more promising path. All the software development principles and techniques you know like testing, modularity, interfaces and types still pay huge dividends, and multiplicatively so. Install aider, get an openrouter account, and go to town on a hobby project at the very least. Apart from anything else, it's just a huge amount of fun.
