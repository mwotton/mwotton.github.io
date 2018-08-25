---
layout: post
title: "Novelty Budgets"
description: ""
category:
tags: []
---
{% include JB/setup %}

---
layout: post
title: "You Need A Novelty Budget" description: ""
category:
tags: [software]
---
{% include JB/setup %}

# You need a novelty budget

We measure a lot of things in software engineering these days. Test
coverage, time-to-deploy, bugs per line - they're all good things to
keep an eye on. They're all proxies for risk of failure, either from
moving too slowly, or from bugs and downtime destroying the business.

Something that's not often explicitly controlled, however, is
_Novelty_. One of the dirty secrets of programming is that almost
every production codebase contains some dependency that the developers
have never used before. Perhaps they've written a trivial project to
play with it, but mostly they're relying on community feedback, or if
you want to be dismissive, fashion. Civil engineers have solid rules
for the way a bridge ought to be constructed: we switch web
application frameworks on an annual basis.

## Why are we indulging in so much novelty anyway?

There are reasons for this churn. The widespread availability of
libraries and frameworks is one of the reasons creating a startup is
cheaper and easier than ever before. Your app sits on top of a vast
pyramid of code you've never seen, and if you want to keep up, you
can't just ignore better options when they come along: your company
will suffer, and so will your career (unless you enjoying maintaining
pre-millennial line-of-business COBOL apps).

## Avoiding the Scylla of endless novelty and the Charybdis of stasis

My technique is to use something called a "novelty budget".
It's a rough, informal thing: in essence, you decide how much
tolerance you have for library code you've never deployed on a new
project before, and apportion it out in the ways that you think will
give you the biggest bang-for-buck. This means that you accept that
you'll pay a cost in missing documentation, unpolished tooling and
ungooglable errors for that part of the project, and you make up for
it by using the most solid, boring, dependable pieces you can for the
rest of it. The name of the game is reducing risk, not optimising for
the best possible case you can imagine.

The essence of this approach is that you are trying to balance average
and worst case time-to-completion.  Any novel component is going to
increase your worst case time: the hope is that it improves the
average, both now and over the evolution of your app in the coming
months and years.

## The default tech stack

Let's assume you've decided where you're spending the biggest chunk of
your budget and put that piece aside: I can't comment on why that's a
good expenditure for you. Whatever it is, every other piece of the project
has to tighten its belt to compensate. (Obviously, if one of these is
your primary expenditure, feel free to ignore me - this is just a set
of defaults.)

### Suggestions

- choose something boring like Ubuntu or Debian, rather than a fancy
  hardened BSD variant or exotic unikernel.

- Don't adopt a graph database or fancy distributed database until
  you've satisfied yourself that PostgreSQL or SQLite are really not
  sufficient. (My defunct startup MeanPath scraped 160 million
  websites a day using SQLite. The limits are probably higher than you
  think.)

- Try a serverside-rendered site before adopting a SPA framework - you
  might find that a limited amount of scripting is all you need.

- Devops automation is a real timesaver, but don't feel that you have
  to go full Docker/Kubernetes right off the bat.  While you
  definitely need a process to build a deployable artifact, it's
  entirely possible that artifact is something more akin to a git repo
  that gets pushed to heroku endpoint than full-on enterprise-grade
  automation.

## Process

When I can, I run my development processes on GitHub via pull reviews,
issues, and the rest of that stack: possibly there is a better
solution for parts of it, but the more you can stay in the boring
zone, the lower your chances of a catastrophic failure. Writing
your own process management software is unlikely to be a good idea
unless that's your company's reason to exist.

## Startups are different though, right?

There's a seductive argument that since the chances of any given
startup's success are small anyway, you might as well crank all the
knobs to 11 just to increase the variance. I think this is a bad idea:
contrary to the founder legend, most startups are not solving
difficult technical problems. The job of the founding engineers is to
get something out there that works well-enough, and to do so quickly
enough to test the hypothesis on which the startup was founded. Once
you get traction, you might hit scaling problems: at that point,
you'll have either revenue or funding to fix them.

## Haskell case study

If you don't care about Haskell, skip this section.

Concretely, my biggest novelty budget expenditure for a project is
frequently Haskell. Haskell has some obvious benefits for me: I know-
it well, and I can bake out a lot of potential flaws just by leaning
hard on the type system. It has a cost, though: IDE tooling is not as
seamless as something like Java or Ruby, sometimes libraries are
missing, sometimes you hit baffling type errors (especially with more
advanced libraries).

The principle of a novelty budget applies within Haskell too, both at
a language level and in your choice of libraries.

### Language

Haskell gives you a lot of rope to hang yourself, if you're so
inclined: there's a whole zoo of extensions, and the emphasis in
community blog posts on sexy types means that you can easily get the
impression that if your app doesn't have a type-indexed generic free
monad at its core, you might as well be writing Perl. It just isn't
true. Sum types, parametric polymorphism and typeclasses already put
the language way ahead of almost everything out there, and you can
build very solid apps without using any extensions. Treat it as an a
la carte menu, not a buffet. (I don't include things like
OverloadedStrings or LambdaCase here - they are minor syntactic
conveniences that don't add significantly to cognitive load.)

### Libraries

I tend to use the Yesod/Persistent+Esqueleto stack. It has problems
but almost any reasonable thing someone might do with a website has
been attempted in Yesod, and there's usually a solution. There are
fascinating database experiments out there with Ferry and Opaleye, but
they don't have the volume of use. Similarly, Servant is a brilliant
piece of work, but in my test projects, I inevitably hit some small
but critical thing that will not get fixed for days, or weeks, or
months, and I can't afford that time on a commercial project. (Yes,
one option is to fix it yourself or sponsor the author to do it: for
whatever reason, I've found that it's usually far harder to get
payments authorised than it is to spend the equivalent amount of dev
time. That's a company management problem I have no idea how to
solve.)

Another approach I've seen people have success with is to use WAI
directly or very minimal frameworks like Scotty. While they don't have
all the bells and whistles of Yesod, you are very unlikely to end up
hitting a hard stop.

## Conclusion

I hope I've convinced you that this is at least a concept worth
thinking about. I should mention that I've mostly worked at smallish
startups, and it's entirely possible it doesn't generalise to
AmaGooBookSoft - would love to hear from anyone in that environment.

Feel free to tell [me](https://twitter.com/mwotton) I'm completely off base.

PS. I'm not claiming to have invented this concept - it's been a term of
art in my development conversations for years. I was frankly surprised
to find nothing about it written down and would welcome hearing about
anything I've missed.

### References and apologies

Similar: http://mcfunley.com/choose-boring-technology
A contrary view from the enterprise side of the wordl: http://mattjolson.github.io/2016/12/04/new-toy-syndrome.html

Thanks to [Matt Olson](https://twitter.com/carnivorous8008) and [David Maciver](https://twitter.com/DRMacIver)
for thoughtful comments on a draft.

Thanks also to my other reviewers who I currently can't find because
Twitter search is terrible, and apologies that this took almost a year
to actually find time to polish and publish.
