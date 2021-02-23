+++
tags = ["go", "golang", "sql", "database"]
categories = []
title = "Go tests and MySQL Transactions"
date = 2021-02-21T09:49:18+01:00
draft = false
+++

## Parallel testing

Go has good tooling for testing out of the box. To increase speed, it generally
runs tests in parallel.

At work, we noticed that with our growing codebase, our test suite would sometimes
have failed tests due to database deadlocks. Rerunning the failed tests
would fix this, and it wasn't happening that often anyway, so we didn't worry
too much.

Database deadlocks are not as bad as they sound, and databases have a good way
of dealing with them.

But the problem started to happen more and more, so we decided to investigate.
We were able to locate a single unit test that, if run in parallel with
itself, would lead to deadlock. Pretty unexpected.

Now a brief aside: in a database, for data integrity, each single datum is
protected against simultaneous modification by two parties (processes).
Think of a bank account: if both you and your partner try to withdraw money
from your joint account at the same time, the final account balance needs to
make sense, with no money created or destroyed.

Databases deal with this all the time, with the help of **Transactions**,
constructs which
allow us to aggregate a collection of operations we want to perform, and ensure
that either they all succeed, or they all get “rolled back”.
A transaction might be: *read the current balance from account A,
subtract X from it, add X to account B*.
You clearly don’t want this process to be half-done.

Deadlocks can happen in certain cases when two parties (transactions) both want
to modify two or more pieces of shared data at the same time.
Again, databases can cope well, and can enforce that the data remain consistent.

Back to our tests. As mentioned, there was a test that would lead to deadlocks
when run in parallel with itself. It looked something like this:

``` txt
Create an Organization object, org1, with a set of images
begin database transaction:
    1. store org1 in the Database
    2. delete any images in the DB associated with org1
    3. for each image of org1, store it in the Database
end transaction
Verify org1 can be retrieved from the Database
```

When this test runs in parallel with itself, it means two processes each create a
fake organization and store it in the Database using a transaction.

We said
> Deadlocks can happen in certain cases when two parties both want to
> modify two or more pieces of data at the same time.

However, in our case, each process will create **a different**
fake organization in the database. Why were they colliding?

## MySQL Transaction Isolation Levels

It turns out that the Transactions used most of the time in databases are not
perfect. Perfect transactions would not be affected by any other transaction
running concurrently.
Perfect transactions do exist, but are very expensive to compute, so databases
offer Transactions that are faster to compute, and are quite good, not perfect.
In fact, databases offer several levels of **Transaction Isolation** with different
cost–quality tradeoffs.

Our database, *MySQL*, uses **Repeatable Read** Transactions by default, unless
explicitly asked for some other type. MySQL produces Transactions by locking,
in comparison with *PostgreSQL*, which uses *Optimistic Concurrency*.
In order to offer Repeatable Read with locks, MySQL deploys *next-key locks* and
*gap locks*, when a query does not identify a single
row by **unique** index. See the
[MySQL documentation.](https://dev.mysql.com/doc/refman/5.7/en/innodb-transaction-isolation-levels.html)

In our case, in step 2 of the transaction, MySQL was putting a lock on the
`organization_images` table, for the `organization_id` from the DB that we got
in step 1. It was also locking adjacent rows with gap locks.

So, when two test processes, say A and B, run in parallel, in step 2, to delete rows,
A puts a lock on the `organization_images` table, for `organization_id`, say 245.
On Repeatable Read, this means it will hold gap locks for neighboring indexes.
Process B creates `organization_id` 246, tries to delete
the associated images in `organization_images` and places gap locks on neighbors.
Meanwhile, process A gets to step 3, and it can't progress because process B
holds a gap lock, while B itself can't progress because of A's gap lock ...
and that is how deadlock is reached.

Databases can detect a deadlock and will reject the transaction
that took the database into this state. Data will not be corrupted.

But our test suite was becoming more and more prone to deadlock failures. \
We changed our test suite to use an isolation level of **Read Committed**
that doesn't use next-key locks, and all our issues disappeared.

NOTE: Read Committed transactions in MySQL may suffer from
[Phantom Rows,](https://dev.mysql.com/doc/refman/5.7/en/innodb-next-key-locking.html)
so while we used this level for unit tests, running code was kept at the default
Repeatable Read level.

## Conclusion

Having our tests run in parallel uncovered a rather involved issue with
our database implementation.

And that's a good thing. The more similar our test environment is to real
life, the better. Headaches during testing are better than headaches
in production.

The concept of transactions, and its practical implementation in databases, is
a fascinating topic. The book
[*Designing Data-Intensive Applications*,](https://dataintensive.net/)
by [Martin Kleppmann](https://martin.kleppmann.com/)
is a great introduction.
