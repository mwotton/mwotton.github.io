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
Vietnamese: hot toc can't be analysed in terms of its parts. "hot toc"
has to be recognised as a single chunk, but we have to be able to
match "hot" on its own too.

Another example is searching for possible domain names in unstructured
text (you can imagine how useful this might be to a webscraping
company). My initial attempts at this were constructing monstrously
large regular expressions from lists of permitted subdomains and tlds.
(Pro tip: if you find yourself writing regular expressions to write
regular expressions, you may have missed a useful abstraction
somewhere.)

Once you view both these problems through the lens of compiler tech
rather than trying to hack it from the command line with
stringly-typed tools, it becomes obvious: this problem of recognising
a sequence of individual tokens has been studied extensively in
computer science. We call it lexing, and while it is more commonly
applied to computer languages, it works remarkably well on natural
languages too.

My first attempt was with CTKL, Manuel Chakravarty's compiler toolkit.
It worked reasonably well, but the runtime is dominated by the expense
of building the initial lexer: with a dictionary of 99k words and a
600 kbyte input file, it took 10s (unprofiled) to churn through. There
is a quadratic cost in construction of the lexer that is undetectable
when the dictionary is small (ie, in its usual use case). This could
be optimised, I just haven't done it.

While I was building the CTKL-based version,
[Wren Romano](http://winterkoninkje.dreamwidth.org/) got back to me on
some questions about her bytestring-trie library.

(Short digression: if you haven't come across them before, tries are a
very cool data structure that encodes a set of strings as shared
prefixes in a tree. This means that you can incrementally follow
strings, and easily find the longest possible match. As soon as we
fail to match, we bomb out, so we can be confident that we are doing
close to a constant amount of work per character inspected)


Anyway, after I moaned a bit that there was no way to get any
information about how much the trie had matched, she made a
[new release](https://hackage.haskell.org/package/bytestring-trie-0.2.4)
out that included functions that kept track of how much of the string
was matched. Now, I can simply build a trie out of the wordlist, and
take the longest match: when no match is found, we consider that
character burned and start again one character along. This would be
inefficient if we were doing string search, but because we expect most
of the text to be actual words, the overhead should be minimal.

{insert benchmark chatter here}

Anyway, I hope I've convinced you that compiler tools aren't just for
compilers. They're great for dealing with unstructured, messy input too.


[code](http://github.com/mwotton/lexy)

* wibble 1: yes, arguably - and ' as well. but this is a blog post,
  not a dissertation.
