---
layout: post
title: "a field guide to persistent template"
description: ""
category:
tags: [haskell]
---
{% include JB/setup %}

Template Haskell is often considered a smell in Haskell, responsible
for everything from slow compile times to impenetrable error messages
to the cow's milk turning sour. Some of this is warranted, but if you
need to check anything based on compile-time information, it is
also the only game in town.

Persistent is a good example of where TH is arguably justified.
Defining our database tables and our types in the same format means
that there is a single source of truth, with no way for the code to
fall out of touch with the database.

Nonetheless, sometimes you do want to break this model; in this case,
I have a type (Status from twitter-types) that I'd like to persist in
the database. I can't include this definition in the standard
quasiquoted file: the type already exists, I can't define it again.
This leaves me with two options:

1. use a type with the same structure as the Status I get from
   twitter-types, and write some conversion functions for converting
   between persisted and original statuses. Probably about twenty
   lines and ten minutes' work.

2. read the source of persistent-template and work out a way of
   controlling the declaration of datatypes on a table-by-table basis.

If you're reading this and I actually succeeded at getting the damn
thing working, it's pretty obvious I chose #2: The Hard Way.
