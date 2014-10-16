---
layout: post
title: "Lexers for NLP"
description: "using compiler technology for natural language analysis"
category:
tags: [haskell, NLP]
---
{% include JB/setup %}

I work for Leadstage on scraping the web. As such, I have a lot of
data flowing through our servers, and we often find it useful to run
some linguistic analysis - frequency analysis, language
identification, this sort of thing.

This means that there is a fair bit of relatively simple analysis that
needs to be done quickly, on large volumes of data.

Word boundary identification is one of these problems. This isn't a
tremendously difficult problem in English: often it's done with
regular expressions and ad-hoc tools, because a word is defined as a
non-empty string of alphabetic characters*. It's not so easy in
Vietnamese: "hot toc" can't be analysed in terms of its parts, it has
to be recognised as a single chunk. If it's not followed by "toc", we
should match it by itself.

Another example is searching for possible domain names in unstructured
text (you can imagine how useful this might be to a webscraping
company). My initial attempts at this were constructing monstrously
large regular expressions from lists of permitted subdomains and tlds.
(Pro tip: if you find yourself writing regular expressions to write
regular expressions, you may have missed a useful abstraction
somewhere.)

Once you view both these problems through the lens of compiler tech
rather than trying to hack it from the command line with
stringly-typed tools like the unix toolkit, it becomes obvious: this
problem of recognising a sequence of individual tokens has been
studied extensively in computer science. We call it lexing, and while
it is more commonly applied to computer languages, it works remarkably
well on natural languages too.

My first attempt was with CTKL, Manuel Chakravarty's compiler toolkit.
It worked reasonably well, but the runtime is dominated by the expense
of building the initial lexer, which is quadratic. When the dictionary
is small (ie, in its usual use case), this doesn't matter, and could
be optimised, I just haven't done it.


While I was building the CTKL-based version,
[Wren Romano](http://winterkoninkje.dreamwidth.org/) got back to me on
some questions about her bytestring-trie library.

(Short digression: if you haven't come across them before, tries are a
very cool data structure that encodes a set of strings as shared
prefixes in a tree. This means that you can incrementally follow
strings, and easily find the longest possible match. As soon as we
fail to match, we bomb out, so we can be confident that we are doing
close to a constant amount of work per character inspected.)

Anyway, after I moaned a bit that there was no way to get any
information about how much the trie had matched, she made a
[new release](https://hackage.haskell.org/package/bytestring-trie-0.2.4)
out that included functions that kept track of how much of the string
was matched. Now, I can simply build a trie out of the wordlist, and
take the longest match: when no match is found, we consider that
character burned and start again one character along. This would be
inefficient if we were doing string search, but because we expect most
of the text to be actual words, the overhead should be minimal.

Benchmarks
==========

benchmarking lexers/ctk
time                 313.6 ms   (311.4 ms .. 316.3 ms)
                     1.000 R²   (1.000 R² .. 1.000 R²)
mean                 311.0 ms   (310.1 ms .. 312.2 ms)
std dev              1.146 ms   (462.6 μs .. 1.474 ms)
variance introduced by outliers: 16% (moderately inflated)

benchmarking lexers/trie
time                 267.8 ms   (258.7 ms .. 275.9 ms)
                     1.000 R²   (0.999 R² .. 1.000 R²)
mean                 267.3 ms   (264.7 ms .. 269.3 ms)
std dev              2.523 ms   (1.291 ms .. 3.045 ms)
variance introduced by outliers: 16% (moderately inflated)

oddly, the real amount of time taken in practice is quite different.
Piping ~ 60mb from a cached file through the cli tool gives these results:

LEXER=ctk ./dist/build/english/english +RTS -p > /dev/null  74.80s user 5.14s system 87% cpu 1:30.86 total
LEXER=trie ./dist/build/english/english +RTS -p > /dev/null  40.10s user 2.08s system 99% cpu 42.423 total

A megabyte a second is acceptable, although with terabytes of data to
play with nothing's ever fast enough. My initial approach was with
unix tools:

  sed 's/^/^/;s/$/$/;' < /usr/share/dict/words > precise_patterns
  tr -c '[:alpha:]' '\n' | grep -f precise_patterns >/dev/null

but this blew out to 26gb before I killed it (and in any case, would
not work with non-English languages). You can use the fixed-strings
argument to grep (-F) which brings the runtime down to 15s, but that's
not quite precise, as it doesn't anchor at the beginning and end of
the line. It is possible to create one single regular expression that
looks a bit like


```
^((A)|(A's)|(AA's)|(AB's)|(ABM's)...)$

```

but it is extremely slow - it looks like grep isn't clever enough to
factor out all the strings that share the same prefix.

Anyway, I hope I've convinced you that compiler tools aren't just for
compilers. They're great for dealing with unstructured, messy input
too - just because the domain is fuzzy doesn't mean that your handling
of it should be.


[code](http://github.com/mwotton/lexy)

* wibble 1: yes, arguably - and ' as well. but this is a blog post,
  not a dissertation.
3
