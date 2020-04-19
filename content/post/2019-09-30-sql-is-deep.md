+++
tags = []
categories = []
title = "SQL is deep - a personal journey"
date = "2019-10-01T09:58:11+02:00"
+++

Some time ago I was trying to advise a junior colleague who said he “knew” SQL: SQL is a bit like chess. You can learn the rules quickly; that doesn’t mean you know how to play well.

I love SQL. I’ve been using it professionally since 2005. A great thing about it is a counterpoint to the previous remark on depth: you can start getting useful and correct results with just a little knowledge.

For many people, `SELECT` / `WHERE` statements are more than enough. The `GROUP BY` clause gives you a bit more expressive power. Common Table Expressions (the `WITH` statement) will allow you to modularize your queries, and so on and so forth.

I’m not done with my learning, but it’s been enough to make me the go-to guy for SQL in some of the places I’ve worked at. \
I thought I might trace my stages of learning SQL. I’ve felt my skills revolutionized more times in SQL than in all other languages combined.

* `SELECT` / `JOIN` / `GROUP BY`. 2005. My first few months employed in Wall Street, in the now defunct *Bear Stearns*. I had not taken Databases in university, and had almost no exposure to SQL. This thing was fun, and a bit tricky to get right. Cool!

* Views and triggers. At our group in *Bear Stearns* we had a nightly update of of data for all the bonds in our fund. Lots of integrity checks and alert conditions managed with triggers. Interesting, though triggers do get overused.

* Cursors. The Ruby driver for SQL was trying to get all results from the DB at once, and things improved so much when I started using cursors for everything.

* Indexes and `EXPLAIN ANALYZE`. Wow, indexes make such a difference. And what a wonderful way to spend a couple of hours studying performance. B-trees, man, B-trees.

* Transactions: 2010. I knew about transactions but had not had that much of a call to use them at Bear. Most of our updates were from a single-thread process. At Amazon I led the design of a piece of infrastructure that was used by multiple people, with requests that were composed of smaller units. I learned the hard way that it is hard to reason about success/failure on composite operations without transactions.

* `GROUPING SETS`, `ROLLUP`. 2015. At *Guy Carpenter*, writing software to study catastrophe models, I need to populate choice menus with a set of parameters that combined into huge data sets. `GROUPING SETs` allowed me to boil the datasets down to these components - How cool is that?

* PL/pgSQL. Dataset importing into the DB is so much better if you just dump your data into an `UNLOGGED` staging table, and from there on do your transformations in SQL or PL/pgSQL, sever-side.

* `WITH` (aka Common Table Expressions). At Guy Carpenter. Wait, so you can modularize your gigantic multi-JOIN SQL query into named parts, and control the flow of execution? How did I not know this!

* *PostGIS*. Wow, a simple extension, and I can compute Gutenberg-Richter curves for earthquakes across the whole US in 20-seconds, outperforming several of the tools used by the analysts at Guy Carpenter. Thank you *Postgres/PostGIS*, for making me look so good.

* Recursive CTE’s to handle a graph data structure to represent addresses. This might be a crossing of the Rubicon, for me.

* Window functions, `LATERAL JOIN`. Woah, these transform some of my clunky SQL clauses into short and pithy poems.

* MySQL procedures. 2019. At *Consentio*, we use MySQL. Miss PostgreSQL. But, server-side procedures work quite well in this thing.

* Transaction Isolation levels. At Consentio, our parallel test suites were running into strange deadlocks where they should not. I read up on Transaction Isolation levels in Martin Kleppmann’s great book. MySQL 5.7 does some strange stuff to get Repeatable Reads. The whole thing is solved by putting the tests on Read-Committed level.
