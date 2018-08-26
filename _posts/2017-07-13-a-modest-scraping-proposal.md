---
layout: post
title: "A Modest Scraping Proposal"
description: ""
category:
tags: [haskell scraping]
---

# Why scraping libraries in Haskell aren't good enough

Every time I mention that Haskell webscraping libraries are a bit
lacking, somebody points to (http-conduit|wreq) and (tagsoup|taggy)
and suggests that it's just a matter of gluing them together. This is
akin to saying that your home-made car is just as good as any other
car, and pointing to a brake pad and a steering column.

A robust scraping pipeline needs to handle caching, respect
robots.txt, accept input and process output in a variety of formats,
gather metadata about the fetched data, handle quotas & rate limits,
and log sites/second, error rates and unexpected failures in real time
to a dashboard or similar. (Some of these are derived from scrapy, the
python scraping framework - others are things I've needed along the
way. As to why you shouldn't just use scrapy: it's slow, untyped, and
prone to running out of memory.)

I may actually write this library at some point, but would be just as
happy if someone read this and beat me to it.

## The shape of the problem

What I mean by scraping is something at least semi-adversarial.
Fetching items from the Twitter API is not scraping: it may be true
that the owner of the website doesn't mind you scraping them (which is
why we check robots.txt), but they will go to absolutely no effort to
avoid breaking your scrapers.

There are many ways to scrape. Sometimes you can just enumerate a list
of URLs: more commonly, each fetch gives you both a list of new URLs
to fetch and a list of results. This allows a recursive approach to
fetching, and also requires some kind of store so you don't fetch the
same URL twice and end up in an endless loop.

## Robots

While you can get away with quick and dirty scraping without checking
robots.txt (and let's face it, we've all run curl in a loop before),
it's pretty rude to do large scrapes without checking the original
provider is OK with it. Thankfully some uncommonly handsome and
benevolent developer has done the heavy lifting of parsing and
processing robots.txt files - any broad spectrum scraping solution
ought to incorporate https://hackage.haskell.org/package/robots-txt .

## Caching

Scraping is an inherently trial-and-error driven process. The web is
messy and inconsistent, and it isn't uncommon to discover your XPath
query (or equivalent) is not quite right hundreds of thousands of
pages in. Proper caching of both raw fetches and processed results
means you can fix small errors in code and selectively redo failing
parses - it also means that if the site under scrape has intermittent
failures, you can refetch missing pages without starting the scrape
from scratch.

More or less anything can be used as a persistent store, so long as
it's indexed on the URL. I used multiple sqlite databases as a backend
when I was scraping 160 million sites a day: for smaller scrapes, a
single PostgreSQL instance should be fine. Your bottleneck is going to
be the fetching, not local writes.

## Input

A good scraping library should
provide a way of querying common data formats like XML, HTML, JSON,
CSV and plain text. The standard way of dealing with these formats in
Haskell is to use Aeson to parse JSON to a datatype representing what
you expect to see: I like this method for normal HTTP interactions,
but scraping is a process of extracting a relatively tiny
amount of information from a large chunk of text. Something like
lens-aeson lets you dig into a json structure in an ad-hoc way

```
foo ^? key "hah" . nth 1 . key "hfasd"
```

Because you typically add scraped fields one at a time, this avoids
the necessity of adding both a record field and an aeson parser every
time you add a field.

Similar reasoning applies to the other formats.

## Output

It should provide a way of serialising results to CSV, XML or
line-oriented JSON (or standard JSON if you're feeling masochistic).

This implies that we should store our results in an output-neutral
format - in keeping with Scrapy's nomenclature, I'll call each result an
Item, and think of it as a map of column names to values.

## Metadata

It's often interesting to know metadata about the fetch process
itself: a scraping library ought to allow access to the path of URLs
that got you to the current one as well as the robots.txt that allowed
access and any headers from the server response.

## Rate limiting

Rate limiting is important for a few reasons. It's not at all uncommon
to accidentally overload a site by scraping too aggressively - I've
even accidentally taken down DNS servers in my time. A good scraping
library should let you limit global requests/second as well as
reqs/second to a particular domain.

It's also important to have configurable quotas for individual fetches.
I've had a PHP site on a fast server spew hundreds of megabytes of
error messages a second at me, all of which got dutifully loaded into
the database. Don't be like me, be smart.

## Logging

Scrapes often go wrong hours in. It's really useful to have something
like scrapinghub.com that gives you a visualisation of how many
requests/s you're getting, and items/s is pretty useful too, but
scrapinghub only works with python+scrapy, so that's no help.

We're getting a little out of scope here, logging is its own thing
that can get almost arbitrarily complicated, but at the very least,
you'd want something that can spit structured info to stderr, and to
be able to set a constraint that if items/s, requests/s or bytes/s
fall below a threshold over the last minute or so.

## Testing

You need at least two kinds of tests: individual offline tests for
each kind of page you want to fetch, as well as a rougher online test
that given a starting point, fetching provides about the number of
results you expect. This is necessarily hazy, but if you usually do 2
fetches for each full item, and that suddenly blows out to 100, it
usually indicates that the formatting has changed on some page you
rely on, and you need to fix it.

## Security

You might think there isn't much to say about security in a scraper -
after all, it's the one calling all the shots. However, it's important
to think about what implicit (eg. IP range) and explicit (password,
token, etc) credentials you're using when scraping. It's entirely
possible for a site that knows it's going to be scraped to redirect
you to a resource for which you have privileged access: if the result
of your scrape will become public later, you've just exposed private data.

The library should have a default configuration under which it won't
scrape private IPs, use creds, or even go off-domain (though
domain.com -> www.domain.com should be fine) - that way you only get
potentially dangerous behaviour if you explicitly ask for it.

(Redirects are a thorny problem in general: a library should catch
redirect loops and allow the user to set policies on off-site
redirects, maximum redirect chain lengths, etc)

## Distribution

I'll be a heretic here and say you probably don't need distribution.
I scraped the front page of 160 million domains every day with 13
machines: if you're not working at that scale, it really doesn't
matter.

If you really needed to extend the design, you'd want to set it up so
that you had a central blocking queue that started out with just the
initial URLs. Scrapers would connect to it to get URLs, and at each
step acknowledge that they've fetched that URL, along with the results
of the fetch - they'd never do more than one level of fetching. After
n minutes without an acknowledgment, the URL can be sent out again,
though you might want to return the history in the response, to avoid
redundant fetching.

I implemented this model over ZMQ, but HTTP would be fine and probably
simpler - it's going to be chatty no matter what, so avoid it if you can.

## Queue control

One of the problems I faced with Scrapy was that as it used an evented
framework rather than an explicit queue, it was extremely easy to blow out memory. In one example,
my source data was a set of gzipped files that listed other gzipped
files that _then_ contained links to the data I needed. Because I
didn't have control of the queue, it ended up loading every
single one into memory before it ever got to the juicy part with the
final data.

## Putting it into code

So, notionally, our scraper takes data in some known format and
extracts some of that information into two things: a (possibly empty)
list of new URLs to look at (along with a tag to indicate what kind of
URL it is) and a list of dictionaries that represents the actual
information you're interested in from that page. (Commonly this will
actually be a single element, but sometimes if you're scraping search
results, you'll get many notionally separate chunks of data on a single page.)

What might this look like?

```
-- user code
data PageType
  = InitialIndex
  | IntermediateIndex
  | ActualData
  deriving (Eq,Ord)
```

This is what defines the shape of the scrape, as seen by the user.
Separating out the different kinds of pages we see means that we can
parse them differently, without relying on fragile regular expressions
on the URL - it also means we can prioritise some fetches above
others, as well as restrict download slots on a page-type basis.

```
-- library code - this needs to be fleshed out.
-- something slightly less general than Aeson's Value type: just scalars.
data Column = ColText Text
            | ColInt  Integer
            | ...

type Item = [(Text,Column)]

-- Quotas, number of concurrent threads, logging, robots, etc
data Config a =
  Config
  { threads :: Int
  , downloadSlots :: a -> Int
  ...
  }

data Metadata = ...

-- here a is instantiated to PageType
scrape :: Config a
       -> [(a, URL)]
       -> (a -> Metadata -> ([(a,URL)], [Item]))
       -> ([Item] -> IO ())
       -> IO ()
```

(This ought to be in something like Conduit or Pipes for real code: it's
presented in this simpler form for clarity only.)

Dissecting the type of scrape, we give it some configuration
information, an initial list of URLs (each with an associated tag), an
extractor function, and a way to do something with each row.

Notice the downloadSlots tag in the Config record? That's there so
that we can control intermediate fetches, and avoid the memory blowout
I described earlier. You might think it's enough to have an Ord
instance for PageType and use that: unfortunately, what typically
happens is that the first few hundred pages will all be InitialIndex
and IntermediateIndex pages; while ActualData requests will be
prioritised, by the time any are actually processed, we might have
already blown out memory. The downloadSlots refers to how many of that
kind of download can be downloading at once: that combined with the
Ord instance on PageTypes means we can keep a queue of fetchable
things ordered in a sensible way that minimises the amount of memory required.

# Conclusion

One reviewer quite reasonably raised the concern that this is an
extremely un-Haskelly library. Our platonic ideal of a Haskell library
is something that exports a single, coherent concept in such a way
that it never needs to be reimplemented. This is not that: scraping is
a dirty, error-prone, highly contingent endeavour. The goal here is to
package up a pile of hard-won knowledge about where the facerakes are
and make them easier to avoid.

(In other news, if you actually need this or other Haskell work done,
I am available for hire: mwotton@gmail.com.)

(Thanks to @tureus, @jfischoff, @mxavier and @thumphriees for their
detailed feedback, and thanks to everybody else who read it, even if
you couldn't think of improvements.)

(discussion [here](https://www.reddit.com/r/haskell/comments/6nazqm/notes_on_design_of_webscrapers_in_haskell/))
