---
layout: post
title: "trees and rectangles: reconciling SQL and recursive programming languages"
description: ""
category:
tags: [haskell, sql]
---

There's a contradiction at the heart of web development.

Most structures from HTML through to JSON calls are fundamentally
hierarchical. Relationships are expressed primarily through
embedding, and this means we mostly deal with trees.

On the other hand, traditional SQL databases talk about
rectangles. Data is essentially tabular, and even with complicated
joins and filters, the output is still a rectangle.

How do we resolve this?

### Object-Relational Mappers (or: teaching trees to think about rectangles)

ORMs were one stab at this problem. The underlying idea was that you
could teach the object system about the structure of a relational
database, and then make database queries lazily when the pattern of
calls required the data, thereby letting the developer stay primarily
in a hierarchical pattern of thought.

While this does sound appealing, it turns out to have many problems in
practice, from the
[n+1 query problem](https://medium.com/@bretdoucette/n-1-queries-and-how-to-avoid-them-a12f02345be5)
to nasty interactions between data cached in objects and the underlying database state. I won't go
into this too deeply, as [Ted Neward](http://blogs.tedneward.com/post/the-vietnam-of-computer-science/)
and [Mark Rickerby](https://maetl.net/talks/rise-and-fall-of-orm) have
already covered it admirably, but in essence: it isn't feasible to
ignore the structures and guarantees of the underlying relational
database if you want either correctness or performance. Rails still
uses this pattern, but even there the Arel package is used to allow
devs to talk about queries in a way that maps closely to the
database.


### NoSQL! (kill all the rectangles, talk trees)

To massively oversimplify, the term "NoSQL" became popular around 2009
to describe a disparate group of databases that did not rely primarily
on the relational view of data as rectangles. Arguably, the earlier object
database approach was a similar idea, the idea being in least at part
that if you couldn't easily map tree-based structures to rectangular
tables, why not get rid of the rectangular data store entirely? (Graph
databases aren't that far from this approach either.)

This isn't inherently a broken model. As long as the whole object
graph is maintained, it can even be faster - traversals can be done
via pointers rather than by consulting indexes on keys. (It also has
to be said that SQL appears to be difficult to scale to enormous
datasets, prompting the development of stores like BigTable.)

Frequently it's modelled in unfortunate ways, however. Sarah Mei's
[breakdown](http://www.sarahmei.com/blog/2013/11/11/why-you-should-never-use-mongodb/)
of why not to use Mongo is a good description of what can happen when
you serialise object data as text, rather than preserving the graph as
a whole.

Even assuming that you manage to avoid that particular pitfall,
however, if query patterns change you could still be in trouble. One
of SQL's big advantages is that it supports ad-hoc, unexpected
queries, and that the indices can be tweaked without changing anything
about the structure of the store itself.

### kill all the trees, talk rectangles

Logically, if you look at the 2x2, there should be a way to talk
rectangles all the way through. I have no idea at all what this would
look like. APL on the front end?

If you want to stick with anything resembling standard front-end
technology, this seems like a non-starter, though I'm open to being convinced.

### json_agg & friends (teaching rectangles to make trees)

By an obvious rhetorical trick, I'm going to talk about what I think is the best
option last.

Let's assume you've decided to use a tree-based language but a
rectangular, relational database. For concreteness, let's assume
Libraries, Books, Borrowers and Loans.

(Here, a Book represents a physical artifact, not the platonic notion of a
book that may have many copies.)

Let's say we wanted to display all of the Loans that overlap a given
date. In addition, we'd like the data about the Books that were loaned
out, the Borrowers who borrowed them, and the Libraries they came
from.

How would we model this? In a RESTful system, we might dodge the
question by providing a single parameterised query on Loans, and letting the
front-end ask about other entities as needed on an individual
basis. In a server-side ORM system, it'd be lazily loaded every time
we accessed a data field.

In either case, the default is a mosquito cloud of queries. Not only
is this inefficient, it can be actively inconsistent.

We can do a little better if we design the query ourself, but if we're
only using classical SQL, it's still going to be a series of fetches:
first we need to get the Loans, and then the Books and Borrowers
associated with them, and finally the Libraries associated with the
Books. Even in a single transaction, this can leave us with an
inconsistent view of the world: in `PostgreSQL`, for instance, `Read
Committed` is the default isolation level, which means that if a Book
is deleted concurrently, we might see a Loan in the first fetch for a
Book that has disappeared by the end of the transaction.

There is a way out of this, though. More recent releases of PostgreSQL
(FIXME, which release? which standard?) have aggregation operators
that let us construct trees in JSON.

```sql
WITH
  books_with_loans as
  (select books.*,json_agg(loans.*)
   from books inner join loans on loans.book_id=books.id
   where loans.start < #{date} and loans.end > #{date}
   group by books.id),
  borrowers_with_loans as
  (select borrowers.*,json_agg(loans.*)
   from borrowers inner join loans on loans.borrower_id=borrowers.id
   where loans.start < #{date} and loans.end > #{date}
   group by borrowers.id)

  select libraries.*,
         json_agg(borrowers_with_loans.*) filter (where borrowers_with_loans.id is not null) as borrowers,
		 json_agg(books_with_loans.*)     filter (where books_with_loans.id is not null) as books
   from libraries left join borrowers_with_loans on borrowers_with_loans.library_id=libraries.id
                  left join books_with_loans on books_with_loans.library_id=libraries.id
```

There are certainly things to complain about here. I'm aggregating
before I join in `books_with_loans` and `borrowers_with_loans` and the
filter syntax is a bit obscure. (FIXME, can do better. in particular,
can we prove that we will always be consistent at READ-COMMITTED
isolation level?).

It's also reasonable to argue that we are duplicating loans in the
books branch and the borrowers branch. This problem is a little more
fundamental, and is almost the same problem we saw with Mongo: I want
to represent an object graph, but I must serialise it from text. An
alternative representation might use a table of loans indexed by ids;
this could also be translated into JSON-POINTER format without too
much effort.

These concerns aside, though, this is at least a decent first stab at
convincing SQL to talk in terms of trees. Even if all our updates are
done at a more fine-grained level, it is pretty common to want to
fetch tree-shaped subsets of the data for display.

FIXME maybe graphql?
