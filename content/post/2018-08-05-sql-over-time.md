+++
tags = [
]
categories = [
]
date = "2018-08-05T11:36:11+01:00"
title = "SQL over time"

+++

Years ago, I worked at a financial firm in New York City. The system we were
building had a Postgres database -- version 8, I think.

There was a particular situation that made many of our queries slow:
we had a large
pool of bonds, each of which would be updated, often daily, with new *factors*.
We held all bond  factors in one table, which was append-only,
i.e. we did not `UPDATE` or `DELETE`,
we only added new factors. This is generally good design, and especially so in
contexts like the financial industry that may be audited, and held to the
Sarbanes-Oxley act.

We often needed to know the most recent factor for each of our bonds.
Conceptually, this involved one or other kind of self-join.

Let’s see an example. First, let’s create a dummy table for our data:

``` SQL
create table factors as
with dates as (
	SELECT generate_series as date
	FROM generate_series('2018-01-01 00:00'::timestamp,
		'2018-08-04 12:00', '10 hours')
), bonds as (
	select 'bn_' || generate_series as bond
	from generate_series(1, 1000)
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
that had the `max_date`.
We need an extra join:

``` sql
with latest_dates as (
	select bond, max(date) as date
	from factors
	group by bond
)
select bond, date, factor
from latest_dates
join factors using (bond, date);
```

Let’s do an `explain analyze`:

```
Hash Join  (cost=30366.00..35525.50 rows=1000 width=48) (actual time=839.536..1260.912 rows=1000 loops=1)
   Hash Cond: ((latest_dates.bond = factors.bond) AND (latest_dates.date = factors.date))
   CTE latest_dates
     ->  HashAggregate  (cost=11070.00..11080.00 rows=1000 width=14) (actual time=353.348..353.988 rows=1000 loops=1)
           ->  Seq Scan on factors factors_1  (cost=0.00..8480.00 rows=518000 width=14) (actual time=0.025..75.878 rows=518000 loops=1)
   ->  CTE Scan on latest_dates  (cost=0.00..20.00 rows=1000 width=40) (actual time=353.353..355.030 rows=1000 loops=1)
   ->  Hash  (cost=8480.00..8480.00 rows=518000 width=22) (actual time=485.041..485.041 rows=518000 loops=1)
         Buckets: 2048  Batches: 32  Memory Usage: 902kB
         ->  Seq Scan on factors  (cost=0.00..8480.00 rows=518000 width=22) (actual time=0.008..166.135 rows=518000 loops=1)
 Total runtime: 1261.271 ms
 ```

Not great, given the data set isn’t that big.

```
bn_217  | 2018-08-04 10:00:00 |   0.527411319315434
 bn_964  | 2018-08-04 10:00:00 |  0.0373749709688127
```

My friend Tom and I would agonize over how this pretty core query was piled
on everything else. We had a system of triggers and keys that didn’t work too
well. Materialized views were still years away (Postgres 9.3).

Tom found some advice to use another query, which both of us found un-intutive:

``` SQL
select f1.bond, f1.date, f1.factor
from factors f1
left join factors f2 on f1.bond = f2.bond and f1.date < f2.date
where f2.date is NULL
```

Let’s `explain analyze`:

```
 Hash Anti Join  (cost=17485.00..40998.83 rows=345333 width=22) (actual time=394.663..2625.339 rows=1000 loops=1)
   Hash Cond: (f1.bond = f2.bond)
   Join Filter: (f1.date < f2.date)
   Rows Removed by Join Filter: 3329963
   ->  Seq Scan on factors f1  (cost=0.00..8480.00 rows=518000 width=22) (actual time=0.013..94.830 rows=518000 loops=1)
   ->  Hash  (cost=8480.00..8480.00 rows=518000 width=14) (actual time=393.446..393.446 rows=518000 loops=1)
         Buckets: 4096  Batches: 64 (originally 32)  Memory Usage: 1025kB
         ->  Seq Scan on factors f2  (cost=0.00..8480.00 rows=518000 width=14) (actual time=0.008..148.163 rows=518000 loops=1)
 Total runtime: 2625.760 ms
```

```
select bond, date, factor
from (
	select bond, rank() over wd as rank,
		first_value(date) over wd as date,
		first_value(factor) over wd as factor
	from factors
	window wd as (partition by bond order by date desc)
) as latest where rank = 1;
```

```
 Subquery Scan on latest  (cost=78896.91..98321.91 rows=2590 width=22) (actual time=1721.732..3085.822 rows=1000 loops=1)
   Filter: (latest.rank = 1)
   Rows Removed by Filter: 517000
   ->  WindowAgg  (cost=78896.91..91846.91 rows=518000 width=22) (actual time=1721.722..2997.214 rows=518000 loops=1)
         ->  Sort  (cost=78896.91..80191.91 rows=518000 width=22) (actual time=1721.707..1934.382 rows=518000 loops=1)
               Sort Key: factors.bond, factors.date
               Sort Method: external merge  Disk: 17208kB
               ->  Seq Scan on factors  (cost=0.00..8480.00 rows=518000 width=22) (actual time=0.013..83.520 rows=518000 loops=1)
 Total runtime: 3120.775 ms
 ```
