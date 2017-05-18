---
layout: post
title: "startup engineering team basics"
description: ""
category:
tags: [haskell engineering]
---
{% include JB/setup %}

I was a bit hesitant about writing this one: it just encodes what I
think is more or less standard practice in healthy engineering teams
in startups. It doesn't include anything with serious legal
restrictions like medical devices, and engineering team structures in
bigger companies usually have years of accretion and complicated power
dynamics - I don't have anything useful to say there, so I'll shut up.

This is also biased towards remote teams. I think this is basically
the right default: what will work for remote teams will also work for
colocated teams, with the added bonus that more contextual information
tends to get recorded.

If you read this and think "well that was obvious", good! Please skip
to whatever site makes you happy instead. With that craven disclaimer
to the side, let's begin.

# Meta-point: You have a novelty budget

The name of the game here is to be as standard as possible. You have
some idea that's going to set the world on fire: great. Be innovative
on your main project, but unless you're a tools company, don't try to
innovate on tools. There is little to gain and an awful lot to lose.
(This doesn't mean the odd script here and there is bad, but they
should feed into the shared context of your main tools.)

# The Three Pillars

The absolute minimum tool setup you need:

    1. a bug/project tracker. Formal, structured, transactional
    2. a communications medium. Informal, unstructured, collaborative
    3. a continuous integration & deployment system

My usual approach is to get these basics set up, then add tools when
and if there are pain points. It's really important that extra systems be
set up only in response to a need: you want the absolute minimum
process feasible. Process builds up like scar tissue and is shockingly
hard to get rid of after the fact. (Logging and monitoring is a close
fourth, but it's not _absolutely_ essential right from the beginning
and is reasonably easy to add in later.)

## Bug Tracker/Source Control

This should be obvious, but if you don't know what's wrong with your
product, you don't know what you should be doing. It's critical that
you have a single source of truth for the current status
of the project, and most conversation
about explicit bugs & features should happen in the context of a bug
report. GitHub now has projects, so rough prioritisation of stories
is possible. Bitbucket is fine too - the important thing is to use
something that most other tools can talk to without fuss.

You can set up separate bug tracker and source control systems but I
wouldn't recommend it: you get all kinds of nice things like being
able to close issues with magic text in pull requests by having them
together. (Yes, you can set up webhooks to do this yourself, but why?
Occam's razor and novelty budget say no.)

## Comms

If we were robots and specs came fully-formed and perfect, we might
not need a centralised communications channel. As it is, we humans
like to feel like part of a communication.

This is a great place to display events like git commits, pull request
notifications, outage checks and CI results, but keep them segregated in a channel like #monitoring or
#botchatter: trying to talk about code in a channel that keeps
squawking hashes at you is like trying to think in a hurricane.
Depending on the volume you may want to split - again, do it in
response to pain.

Otherwise, have a channel for each broad area of the company. Minimum
is #dev - you may need #backend and #frontend as you grow. Split them
when it feels like >50% of the content in them is not relevant to
everybody in them. You'll probably have some channels for non-devs
too: I have no opinion about them. I think it's a mistake to have
goof-off channels like #random in a small startup - it tends to foster
a channel-cop attitude, and the odd joke in #dev is really not a
problem.

Slack is more or less standard. If you choose this, actually pay for
it: being able to search back to the beginning of the startup is
really important. I hear gitter and riot are ok too - make sure
you have integrations from your CI and source control systems for
whichever you choose.

(I like IRC but I recognise I'm in a dwindling minority.)

A bot can be useful. I haven't got deeply into it, but it's one way of
doing offline standups. (I'm torn on this: for dumb human reasons,
having at least a video call for standup every day seems to build team
spirit more than a botted questionnaire, however much more convenient
it is to be able to record follow-up questions in a searchable comms channel.)


## CI

CI is where your code gets built, tested and deployed. Your basic flow
should be to have a developer make a pull request on your source
control system, which notifies CI to pull that code and test it. (You
can have it be notified on every commit if you prefer: I like the
control of being able to make small commits without overloading the
build server.) CI should then make a webhook call back to GitHub to
let it know that it failed, and GitHub can then inform Slack. (I'm
going to go with the default tools, please mentally rename as
appropriate).

You'll also need a webhook for successful merges. This should do more
or less the same thing as PR builds, but also push to production. With
any luck, you'll be able to cache the products of builds so you don't
need to rebuild every dependency every time.

The choices here are a bit more difficult. If you're using Haskell,
you probably want to set up your own server: GHC build processes are
slow and CPU-heavy, and my experiences with hosted services has been
disappointingly slow. (The tempo of your build server is extremely
important: humans do not switch contexts well, so often your devs will
be waiting for a build to succeed before they can go on. Do not skimp
on this.)

I've tried drone and wouldn't recommend it: they currently have no way
of adding extra workers, which paints you into a corner if you outgrow
one machine. Jenkins is the grand old
dame of build servers and can do more or less anything given enough
configuration but is kind of a bear to set up in an automated
way. If you can get away with just using Travis or CircleCI, thank
your lucky stars and do that. I'm actively looking for a better way to
do hosted CI.

# Dev flow

Oof, that was a big wall of text. Don't despair, we'll edit the shit
out of this later.

It can be a bit hard to visualise what's going on here, so let's run
through a story from start to finish. Again, I'm going to assume
Slack and Github for the sake of concreteness.

Dramatis Personae:
    Bob, a non-dev with a problem
    Helen, a dev
    Fritz, another dev

    1. Bob has just got off
