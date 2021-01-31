+++
tags = [
]
categories = [
]
date = "2018-09-25T01:36:11+01:00"
title = "SQL over time, or, Ditch your ORM"

+++

Here I'm telling a story of PostgreSQL performance over time, and the point
of it, for me, is that given the richness of SQL, and the steady improvement of
implementations, it is foolish
to trust your querying to an ORM.
SQL is very much worth learning, and learning
well. There are many gains to be made, in efficiency and in code quality.

---

Years ago, I worked at a financial firm in New York City. The system we were
building had a Postgres database &mdash; version 8, I think.

There was a particular situation that made many of our queries slow:
we had a large
pool of bonds, each of which would be updated, often daily, with new *factors*.
We held all bond  factors in one table, which was append-only,
i.e. we did not `UPDATE` or `DELETE`,
we only added new factors. This is generally good design, and especially so in
contexts like the financial industry that may be audited, and held to the
Sarbanes-Oxley act.

We often needed to know the most recent factor for each of our bonds.
Conceptually, this involved some kind of a self-join.

First, let’s create a dummy table for our data, with daily factors for
5000 bonds over 5 years, which I think gives an idea of our old work set:

``` sql
create table factors as
with dates as (
      SELECT generate_series as date
      FROM generate_series('2013-01-01 00:00'::timestamp,
            '2018-01-01 00:00', '24 hours')
), bonds as (
      select 'bn_' || generate_series as bond
      from generate_series(1, 5000)
)
select bond, date, random() as factor
from bonds cross join dates;
```

To get the latest factor, we can think of first getting the latest time
each bond was updated:

``` sql
select bond, max(date) as date
from factors
group by bond;
```

Unfortunately, we can’t get the factor from the same row
that had the `max date`.
We need an extra join:

``` sql
select bond, date, factor
from (
      select bond, max(date) as date
      from factors
      group by bond
) latest
join factors using (bond, date);
```

Let’s do an `explain analyze`:

``` sql
Hash Join  (cost=535294.32..668778.20 rows=5000 width=23) […]
Hash Cond: ((factors_1.bond = factors.bond) AND  […]
->  HashAggregate  (cost=195209.37..195259.25 rows=4988 width=15) […]
      Group Key: factors_1.bond
      ->  Seq Scan on factors factors_1   […]
->  Hash  (cost=149534.58..149534.58 rows=9134958 width=23)  […]
      Buckets: 65536  Batches: 256  Memory Usage: 2456kB
      ->  Seq Scan on factors  […]
Planning time: 0.516 ms
Execution time: 6432.132 ms
```

This was a core query for us &mdash; many queries depended on the latest value of
our bonds. My friend Tom and I would agonize over how this operation slowed
down all else. We had a system of triggers and keys that didn’t work too
well. Materialized views were still years away.

The above query time of 6 seconds is on better hardware, and way more RAM than
we had available in 2005, and also, on a more modern version of Postgres (9.6).
So, the core query, which was a subquery in many other queries, would compound,
and we ended with multi-minute queries.

We wanted to do better, and Tom found some advice to use another query,
which both of us found un-intuitive:

``` sql
select f1.bond, f1.date, f1.factor
from factors f1
left join factors f2 on f1.bond = f2.bond and f1.date < f2.date
where f2.date is NULL;
```

The query was better than our first try, by quite a bit. We ended up using
that self-join idiom whenever we had time-series, which was often.

Let’s `explain analyze`:

``` sql
Hash Anti Join  (cost=308326.56..735885.05 rows=6089972 width=23) […]
Hash Cond: (f1.bond = f2.bond)
Join Filter: (f1.date < f2.date)
Rows Removed by Join Filter: 9135000
->  Seq Scan on factors f1  […]
->  Hash  (cost=149534.58..149534.58 rows=9134958 width=15) […]
      Buckets: 131072  Batches: 256  Memory Usage: 2823kB
      ->  Seq Scan on factors f2  […]
Planning time: 0.799 ms
Execution time: 8273.339 ms
```

Interestingly, in my current Linux machine running PostgreSQL 9.6, this second
query is slower, and uses more disk for the internal hashing previous to the `JOIN`.

Now let's see another approach using window functions:

``` sql
select bond, date, factor
from (
      select bond, rank() over wd as rank,
            first_value(date) over wd as date,
            first_value(factor) over wd as factor
      from factors
      window wd as (partition by bond order by date desc)
) as latest where rank = 1;
```

In a way, this is conceptually the cleanest approach. We partition our table
by bond, and rank members of each partition by date. We then just need to
get the top ranked entry in each partition.

However, performance is not very good.

Up to now, we have not used indexing, and so all our `JOIN`s have required
sorting/hashing. Let's create an index:

``` sql
create index idx_bond_date on factors (bond, date);
```

The index speeds up the first query very much. As you can see below,
the query plan
switches to use index scanning when joining the `latest` sub-query with the
main table.

``` sql
Nested Loop  (cost=195210.43..235393.18 rows=5000 width=23) […]
->  HashAggregate  (cost=195210.00..195259.88 rows=4988 width=15) […]
      Group Key: factors_1.bond
      ->  Seq Scan on factors factors_1  […]
->  Index Scan using idx_bond_date on factors  […]
      Index Cond: ((bond = factors_1.bond) AND (date = (max […]
Planning time: 0.785 ms
Execution time: 2703.610 ms
```

For the other methods, the index does not improve the query plan.

The Postgres query planner is improved release after release, and some
version-to-version comparisons have shown significant speedups over time.

But to reap the benefits of the improved planner, it's good to write idiomatic
SQL. Idiomaitic SQL is what planner improvements target. Leaving your SQL
to an ORM, what control do you hope to exert? If you want to get good performance,
you are likely to need to learn the internals of the ORM, on top of SQL, and
negate the economy of effort you were hoping for.

Now, about the query for latest factors: today I would use a materialized view,
and refresh it daily. Looking for the latest factors would be a simple table
scan. Your ORM won't tell you that!
