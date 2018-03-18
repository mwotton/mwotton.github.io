---
layout: post
title: "notes on a better migration system for Persistent"
description: ""
category: programming
tags: [haskell databases]
---
{% include JB/setup %}

## Problems with Persistent

I use Persistent for a lot of my Haskell database work. It has some
great qualities - I really appreciate the type-based linkage between
the database types and my Haskell types, and while its query language
is anemic, Esqueleto does a good job of modelling most of SQL (or at
least enough that I can get useful work done without constant
frustration.)

The biggest problem I hit is with the migration system. It's computed
by comparing the current structure of tables with what the Haskell
model thinks they should be, and has some limited support for
detecting what changes need to be made: however, it doesn't allow you
to consider two versions of the same database in one program, which
makes using Haskell functions to populate new or changed fields &
tables impossible.

In previous codebases I've ended up serialising the SQL at development
time and creating ActiveRecord migrations (hat-tip to Chris Allen for
that trick), allowing me to at least use SQL to populate new columns,
but I frequently ended up needing to write functionality again in SQL
just for the migration.

(It's also possible that you can solve these problems by not using
Persistent, but so far every other database access library I've tried
either takes a strings-in/strings-out approach which leaves me worried
that the code and the database have fallen out of sync, or have used a
plethora of fancy types that I find hard to work with.)

## Desiderata for a migration library

1. Should work with Persistent's datatypes and migrations. It needs to
   know what DDL changes that persistent would try to make. This
   doesn't mean it has to do the same thing (and often won't - can't
   do triggers, constraints, etc): it just means that when complete,
   persistent should agree that there are no migrations to be run.

2. It should provide a predicate to that effect, which should be easy
   to run in a test suite as a sanity check, as well as on startup.

3. Migrations should be easy to run on startup, in the context of
   Persistent. this one is probably controversial, but fits my use
   case. If the migration fails, we should have a way of signalling to
   the environment that we are a failed deploy, and that the migration
   should be aborted and the last version of the code substituted back
   in. (Keter has health checks, though not terribly exhaustive ones -
   I _think_ it just checks that the given HTTP port is open, so
   killing that listener might be enough.)

   We also need to make sure that if multiple web backends try to run
   the same migration concurrently that only one makes it through, and
   that the others don't interpret their inability to run the
   already-run migration as evidence that they're broken.

4. DDL migrations should be in plain SQL. Data-changing migrations can
   be in Haskell or SQL: this allows us to back-populate columns using
   haskell functions with all the context we're used to. Because we
   aren't trying to talk about different structural states of the
   database at the level of types, this will usually require a two
   step migration process: add the fields in a nullable way and populate
   them using Haskell functions, then in the second step, make them
   non-nullable or foreign-keyed or whatever other database-level
   constraint needs to apply.

   This also implies that you need nested transactions: if the first
   step succeeds, but the populated fields do not satisfy the
   constraints you expected them to, the database needs to be rolled
   back to before the migration started. (There can actually be a
   third step, where you update your Haskell types to reflect the new
   constraints: this might be as simple as switching `Maybe a` to
   `a` - this can fail too, if your SQL migration wasn't correct, but
   it has no database effects and can be rolled back without having to
   worry about the database.)
