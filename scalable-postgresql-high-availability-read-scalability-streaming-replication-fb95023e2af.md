
# High availability and scalable reads in PostgreSQL

A detailed primer on scaling PostgreSQL via streaming replication (with performance measurements)

![](https://cdn-images-1.medium.com/max/5000/1*REgUBQRGJncpf2WiutgUQA.jpeg)

*Note: [TimescaleDB is hiring](http://www.timescale.com/careers)! C Developers, R&D Engineers, Marketing, Sales, Evangelism and more.*

Today PostgreSQL, [the fastest growing DBMS of 2017](https://db-engines.com/en/blog_post/76), is more popular than ever. Yet developers often still choose a non-relational (or “NoSQL”) system over PostgreSQL, typically because of one reason: scale.

Here’s how the thought process usually goes: PostgreSQL is a relational database; relational databases are hard to scale; non-relational databases scale more easily; let’s use a non-relational database.

These non-relational database systems may provide scale but also introduce significant costs: poor reliability, poor query performance (e.g., no secondary indexes), poor usability (e.g., custom query languages), cardinality problems, small ecosystems of compatible tools, etc.

Clearly, there are a lot of benefits if one can make PostgreSQL scale. And as popular as PostgreSQL is, most developers still underestimate its native scalability.

**Scalability.** That word consistently comes up in database evaluations, but often with inconsistent meanings. So before we continue, let’s unpack it.

When it comes to scalability, we’ve found that developers are typically looking for some combination of the following three requirements:

1. Higher insert performance

1. Higher read performance

1. High availability (technically not related to scale, yet still often cited as a reason)

PostgreSQL already natively supports two of those requirements, higher read performance and high-availability, via a feature called **streaming replication**. So if your workload peaks below 50,000 inserts a second (e.g., on a setup with 8 cores and 32GB memory), then you should have no problems scaling with PostgreSQL using streaming replication.

In this post, we discuss how to scale read throughput and provide high availability in PostgreSQL using streaming replication. We then dig deeper into the several supported replication modes. We conclude with some performance numbers, measuring the impact of each of the replication modes on insert and read performance.

As part of our analysis, we include numbers on [TimescaleDB](https://github.com/timescale/timescaledb), an open source time-series database that we have developed, which as a PostgreSQL extension supports streaming replication out of the box. An important part of our design is to ensure that TimescaleDB has a similar performance profile to vanilla PostgreSQL for core features like replication, so we ran extra benchmarks to confirm our design satisfies that requirement.

*(TimescaleDB is an open source time-series database that scales any PostgreSQL database for time-series data. It can be installed on an existing PostgreSQL instance or entirely from scratch. [More information here.](http://docs.timescale.com/v0.9/getting-started/installation/mac/installation-homebrew))*

*Note: This post will focus on metrics and high level concepts. For a hands-on walkthrough on how to set up streaming replication, please see our [Replication Tutorial](http://docs.timescale.com/latest/tutorials/replication).*

## How PostgreSQL streaming replication works

At a high level, PostgreSQL streaming replication works by streaming records of database modifications from the primary server to one or more replicas, which can then be used as read-only nodes (to scale queries) or as failovers (for HA).

![How an (N+1) node PostgreSQL cluster works](https://cdn-images-1.medium.com/max/2880/0*TeNEOpUK4Qjn9eYj.)*How an (N+1) node PostgreSQL cluster works*

PostgreSQL streaming replication leverages the [Write Ahead Log](https://www.postgresql.org/docs/current/static/wal-intro.html) (WAL). The replication works by continuously shipping segments of the WAL from the primary to any connected replicas. Each replica then applies the WAL changes and makes them available for querying.

### What Is The WAL Anyway?

Before we get too far into replication, let’s first understand what the WAL is and why we have it.

The Write Ahead Log, is an append-only series of instructions that captures every atomic database change (i.e., every transaction). Using a WAL is a common approach in database systems for ensuring atomicity and durability. In particular, durability is quite important: it is the notion that when a database commits a transaction, the resulting data is queryable by future transactions, even in the case of a server crash. This is where the WAL comes in.

When we run a query that modifies data (or makes schema changes), PostgreSQL first writes that data modification to memory in order to record the change quickly. Memory is very fast to access, but also volatile, meaning that if the server were to crash, our recent data changes would disappear after a server restart. So we also need to eventually write the changes to a persistent store. For this reason, there are periodic points in time (called “checkpoints”), where PostgreSQL writes any modified (or “dirty”) pages in memory out to disk.

However, before PostgreSQL writes to the main data files on disk, it first appends entries to the WAL (also stored on disk). Why utilize a separate structure and not write directly to the main data files? The answer lies in the speed difference of writing sequentially versus randomly. Writes to the main data directory could be spread across multiple files and indexes, causing the disk to jump around a lot. On the other hand, writes to the WAL are sequential, which is always faster (especially on spinning disks, but even on SSDs).

The transaction can then choose to return a commit after writing to the WAL, *but before writing to the main data files. *Now, if the server crashes, on restart it can replay all changes on the WAL *since the last checkpoint.*

![During normal operation PostgreSQL makes changes to queryable state and writes them to WAL concurrently. If there is a server crash, the server reads from the WAL and applies any changes that could have been lost due to the crash so that everything that was queryable before the crash is available when recovery finishes.](https://cdn-images-1.medium.com/max/2880/0*HJxkjXSuNOBA5tcd.)*During normal operation PostgreSQL makes changes to queryable state and writes them to WAL concurrently. If there is a server crash, the server reads from the WAL and applies any changes that could have been lost due to the crash so that everything that was queryable before the crash is available when recovery finishes.*

In other words, the WAL is the canonical record of all database changes, so that we can replay the changes *that were in memory but had not yet been written to the main data directory* in the case of a server crash.

### WAL and Replication

The WAL helps if the server crashes and then restarts (e.g., because of an out of memory error, or power outage). But it does have one glaring limitation: it cannot help if the disk becomes corrupted, or suffers another common unrecoverable issue, or is stomped on and beaten with a baseball bat:

![***This is for all you gif lovers out there. Now back to business.***](https://cdn-images-1.medium.com/max/2000/0*YzVvaddXKX699lEX.)****This is for all you gif lovers out there. Now back to business.****

All of our data was on that server and it’s not coming back. So we also need something to make sure we are resilient to unrecoverable failures. That is where replication comes in.

Here PostgreSQL (really, the generally under-appreciated core PostgreSQL developer community) uses a clever approach. Instead of building a separate infrastructure to support replication, PostgreSQL just uses the same WAL. It ships the WAL to other servers; the other servers replay the WAL as if they were recovering at a server restart; and *voila!*, we now have a complete replica of the database on a different server.

![The primary server handles all inserts, updates, deletes, schema changes and the reads pointed at it. Replicas receive instructions, written to the WAL by the primary server, via the WAL Sender, persisting the instructions to each replicas own copy of the WAL. The replicas, always in hot standby mode, use a recovery process to read from the WAL and apply its changes to the database. Only-read queries are allowed to these replicas.](https://cdn-images-1.medium.com/max/3200/1*vuOYiLxxUN02a7AyS6Y9AQ.jpeg)*The primary server handles all inserts, updates, deletes, schema changes and the reads pointed at it. Replicas receive instructions, written to the WAL by the primary server, via the WAL Sender, persisting the instructions to each replicas own copy of the WAL. The replicas, always in hot standby mode, use a recovery process to read from the WAL and apply its changes to the database. Only-read queries are allowed to these replicas.*

In the image above, the primary server, in read/write mode, handles all data manipulation changes (inserts, updates, deletes), data definition changes (schema changes), as well as any reads pointed at it. It plans each transaction and determines where to store, update, delete, or find data. Any resulting instructions for modifying the database are then written to the WAL, and a commit message is returned to the client so it knows that the data has been persisted. Replicas receive these instructions from the WAL Sender, persisting them to each replica’s own copy of the WAL. Again, each replica always exists in hot standby mode, where a startup (recovery) process reads from the WAL and applies its changes to the database, and read-only queries are allowed. Crucially, this means replicas are free to make efficient changes to their underlying disk based on information the primary has already determined.

Normally, when in recovery mode, reads and writes to the database are not allowed, but for a replica server one can place it in hot standby mode. Hot standby mode disallows writes and replays the WAL like recovery mode but allows read-only queries. One can then use the replica for read-throughput, all while data changes are being streamed in and applied in the background.

## Different Replication Modes For Different Situations

PostgreSQL supports three basic modes of replication, each of which dictates the amount of data replication that will occur before a write is considered complete by the client.

![[**Asynchronous Replication** ](#e1e2)returns to the client after data has been written to the primary’s WAL, [**Synchronous Write](#abbd)** returns when data has been written to the replica’s WAL, and [**Synchronous Apply](#cbe0)** returns when the replica’s WAL has been applied and made queryable.](https://cdn-images-1.medium.com/max/3200/1*mC4RiaYkqJj6wy4rVG2LiQ.jpeg)*[**Asynchronous Replication** ](#e1e2)returns to the client after data has been written to the primary’s WAL, [**Synchronous Write](#abbd)** returns when data has been written to the replica’s WAL, and [**Synchronous Apply](#cbe0)** returns when the replica’s WAL has been applied and made queryable.*

Within each replication mode you can make further adjustments to expectations around consistency, persistence, and when a transaction commits. This behavior is handled by the **synchronous_commit** and **synchronous_standby_names** settings (see our [tutorial](https://docs.timescale.com/v0.9/tutorials/replication) or the [PostgreSQL docs](https://www.postgresql.org/docs/current/static/warm-standby.html#SYNCHRONOUS-REPLICATION) for a deeper dive on those settings).

![](https://cdn-images-1.medium.com/max/4000/1*P9DZc0NBc6a6dpBplavZlQ.jpeg)

### 1. Asynchronous Replication

Asynchronous replication makes no guarantees about whether data has been sent to any connected replicas. Data is considered written once it has been written to the primary server’s WAL. The WAL sender will stream all WAL data to any connected replicas, but this will happen asynchronously after the WAL is written.

**Write Performance: **Asynchronous replication is the most performant replication mode for inserts. Clients only have to wait for the primary to write the WAL. Any latency between the primary and replicas is separate from the write time as seen by the client. Any latency between writes on the primary and writes on the replica (commonly called **replication lag**), however, will result in temporary data inconsistency until the replicas catch up.

**Read Consistency: **Since there are no guarantees that data has been streamed to the replicas upon a successful write, this mode can result in temporary data inconsistency between the primary and the replicas. Until the relevant WAL has been streamed to the replicas and applied to their databases, clients reading from the replicas won’t see the new data in their queries.

**Data Loss: **Data loss is a possibility with asynchronous replication if the primary server irreparably crashes before the WAL was streamed to the replicas [[1]](#b2ba). However, if the primary crashes and comes back up, the replicas will resume streaming from where they left off and eventually catch up with the primary server.

[[Back to Figure]](#2dfa)

### 2. Synchronous Write Replication

Synchronous write replication guarantees that all specified replicas [[2]](#b2a4) will write their data to the WAL before the primary returns success to the client.

**Write Performance: **Write performance for synchronous write replication is much slower than its asynchronous counterpart. Writes on the primary in this mode incur the additional overhead of communicating with the replicas over the network as well as waiting for the replicas to write to the WAL.

**Read Consistency: **Synchronous write ensures that the WAL has been written to, not that the data has been applied to the persistent database layer. While this is a stronger guarantee than with asynchronous replication, it is still not fully consistent. A client may read from a replica before the WAL segment has been applied to the replica’s database but after it has been applied to the primary’s. In general, the replication lag incurred in this mode is far lower since the write ensures that the WAL data has been sent and written to the replica.

**Data Loss: **Data loss is still a possibility with synchronous write commit, but it is much less likely than with asynchronous commit. In most synchronous write cases, both the primary as well as all specified replicas would have to irreparably crash [[3]](#dde2). In any other case, data can be recovered. Either the primary will come back online and the replicas can resume streaming, or a replica which is guaranteed to have the latest copy of the WAL can be promoted to the primary.

[[Back to Figure]](#2dfa)

### 3. Synchronous Apply Replication

Synchronous apply guarantees not only that the WAL will be written to all specified replicas, but also that the WAL segments will be fully applied to the database.

**Write Performance: **Since the client must wait for all write operations to finish on the primary and each specified replica, this is the slowest replication mode.

**Read Consistency: **Every specified replica is guaranteed to be fully consistent with the primary as a write will not be considered successful until it has been applied to the database.

**Data Loss: **Synchronous apply provides even stronger guarantees against data loss than synchronous write. In every possible configuration of synchronous apply mode, the databases of the primary and all specified replicas are guaranteed to be fully up to date. Data loss will only occur in the event that both the primary and all specified replicas are irrecoverably lost.

[[Back to Figure]](#2dfa)

### Fun fact: Choose trade-offs at transaction time, not server setup time

One of the most powerful parts about streaming replication in PostgreSQL is that one can set the replication mode on a *per transaction basis*.

For example, let’s suppose you have database a workload with important, yet infrequent, changes to relational data. But you also have another workload with less important, yet much more frequent time-series inserts. You may decide to require a strongly consistent replication mode for the relational data with a low insert rate, but an eventually consistent mode for the time-series data with a high insert rate.

You can then set **Synchronous Apply Replication **as your default mode, but when writing time-series data, change the setting to **Asynchronous Replication** *just for that transaction. *This would let you handle spikes in the write rate for time-series, while ensuring the relational transaction is fully consistent across all nodes, *all on a per transaction basis, all on the same database.*

## Measuring the performance impact of each replication mode

We’ve found that performance measurements are the best way to truly evaluate database configuration options (and databases in general). We take these sorts of benchmarks seriously and use them to measure the progress of our own work.

(For example, we recently discovered [limitations with PostgreSQL 10 when working with high numbers of partitions](https://blog.timescale.com/time-series-data-postgresql-10-vs-timescaledb-816ee808bac5), and showed [how to achieve better performance when working with time-series data](https://blog.timescale.com/timescaledb-vs-6a696248104e).)

Since the main performance impact of streaming replication is on writes, here we focus on the relative differences in write performance between each of the streaming replication modes against a database with no replication. We also duplicated the experiment on another PostgreSQL database with TimescaleDB installed to confirm that it would perform similarly.

Please note that the write performance for PostgreSQL (but not TimescaleDB) degrades significantly if the total size of a table exceeds main memory. Thus, we present performance for a 100M row dataset that does fit in memory. See [the last section](#0521) for how to deal with larger data sizes.

Additionally, we measured the change in read-throughput with and without replication, but only on a single TimescaleDB instance. [While raw queries are faster on TimescaleDB than vanilla PostgreSQL](https://blog.timescale.com/timescaledb-vs-6a696248104e), the delta between with/without replication is comparable in both setups.

### Experiment setup

Here is our setup and dataset:

* 3 nodes (1 primary and 2 replicas), each on their own Azure VM

* Azure VM: D8 Standard (8 cores, 32GB memory) with network attached SSDs

* All databases used 8 clients concurrently

* Dataset: 4,000 simulated devices generated 10 CPU metrics every 10 seconds for 3 full days. Metrics were modeled as a table row (or hypertable row for TimescaleDB) with 10 columns (1 for each metric) and indexes on time, hostname, and one of the metrics (cpu_usage)

* Inserts resulted in 1 table/hypertable with 100M rows of data

* For TimescaleDB, we set the chunk size to 12 hours, resulting in 6 total chunks ([more on TimescaleDB tuning](http://docs.timescale.com/v0.9/using-timescaledb/hypertables))

### Performance numbers

**Write performance
**The table below compares row inserts per second for PostgreSQL and TimescaleDB under four scenarios: no replication, and each of the three replication modes.

Each replication mode is ordered left-to-right by the level of consistency guarantee, from lowest (Asynchronous) to highest (Synchronous Apply).

![](https://cdn-images-1.medium.com/max/4488/1*OMuBKtaPq3tufsogswkBcQ.jpeg)

Here we see that both PostgreSQL and TimescaleDB experience a negligible drop in write performance with **Asynchronous Replication**; this is because in this case the primary does not wait for the logs to be persisted to disk, nor do the replicas wait for the data to be applied on the primary. Of course, this option includes the lowest consistency guarantee.

On the other end of the spectrum, we see that both experience a ~50% drop-off in write performance with **Synchronous Apply**; this is because the primary now has to wait for the transaction to be applied twice (once on the primary, and once again in parallel on all replicas). This option, however, does provide the strongest consistency guarantee.

**Synchronous Write**, as one would expect, is right in the middle, both in terms of insert performance and consistency guarantee.

**Read performance
**Our read performance experiment was much simpler, because regardless of which replication mode one chooses, queries perform similarly. We used the same dataset and VM setup as we did for the insert benchmarks. We’ve only included read performance numbers for TimescaleDB to keep things simple, but vanilla PostgreSQL enjoys similar performance gains from spreading reads across replicas.

Here we compare queries per second for a variety of queries under a single node vs. 3 nodes (1 primary and 2 replicas):

![TimescaleDB read performance improvements on a 3 node setup](https://cdn-images-1.medium.com/max/4488/1*sPFYjrxiz9SUs8R8bSzpdg.jpeg)*TimescaleDB read performance improvements on a 3 node setup*

With streaming replication with 2 replicas, we see an average of 2.5x faster queries and large improvements across the board.

The same experiment with a 5 node cluster (1 primary, 4 replicas) demonstrates that we can scale reads linearly as we add read nodes to the cluster, resulting in 4.8x faster queries:

![TimescaleDB read performance improvements on a 5 node setup](https://cdn-images-1.medium.com/max/4488/1*o_vLDqQLLSNXhvBWA1pdcA.jpeg)*TimescaleDB read performance improvements on a 5 node setup*

### Experiment summary

As we see above, the trade-offs between insert performance and consistency for each of the replication modes are not trivial, and require thought on the types of workloads you plan to support.

For example, for high volume time-series workloads, where some data loss would be acceptable in the case of irrecoverable loss of the primary, you might choose the performance gains from **Asynchronous Replication [[4]](#eaf6)**.

On the other hand, for users dealing with transactional data that cannot be lost under any circumstances, the absolute persistence guarantees provided by **Synchronous Apply** may be worth the additional overhead.

Or perhaps the best option for you is the in-the-middle (but closer to Synchronous Apply) **Synchronous Write**. The choice is yours.

Under any of these options, additional nodes leads to a much higher read-throughput (2.9x higher on average in our experiment with a 3 node cluster, 4.8x higher with 5 nodes), which should scale fairly linearly with the number of replica nodes.

When deciding which option to use, please be sure to weigh your requirements carefully versus the costs. For more information on how to set up your desired configuration, feel free to refer to our [Replication Tutorial](http://docs.timescale.com/latest/tutorials/replication#view-replication-diagnostics) for guidance (which also includes instructions for TimescaleDB).

## Considerations for large datasets

While higher read-throughput and high-availability are valuable, some workloads also require higher insert performance than what one typically gets with PostgreSQL, especially when considering that insert performance on PostgreSQL degrades dramatically as your dataset grows:

![**Insert performance on PostgreSQL degrades dramatically as your dataset grows.**](https://cdn-images-1.medium.com/max/2880/0*jN21vV8-NZiFC8gM.)***Insert performance on PostgreSQL degrades dramatically as your dataset grows.***

There are several non-native options for scaling insert performance, depending on your workload type; discussing them would perhaps warrant another blog post.

But if you have time-series data, then we suggest considering [TimescaleDB](https://github.com/timescale/timescaledb). Not only do we see 20x higher inserts at scale, but also 2000x faster deletes, 1.2x-14,000x faster queries, and a variety of usability improvements ([full benchmarks](https://blog.timescale.com/timescaledb-vs-6a696248104e)). And all of the replication options described above work with TimescaleDB out-of-the-box.

For more information on TimescaleDB, please refer to our [Github](https://github.com/timescale/timescaledb) and [developer docs](https://docs.timescale.com/latest/main).

*Like this post? Please recommend and/or share.*

*And if you’d like to learn more about [**TimescaleDB](http://www.timescale.com/)**, please check out our [GitHub](https://github.com/timescale/timescaledb) (stars always appreciated), and please [let us know](mailto:hello@timescale.com) how we can help.*

[***And we are hiring](http://www.timescale.com/careers)!***

## 👣🎵

[*[1]](#3293) Asynchronous replication can be achieved with **synchronous_commit** set to either **off** or **local** — in each case the primary will not wait for replicas to return success to the writer. However, **synchronous_commit** carries a higher possibility of data loss since it only guarantees that WAL data has been sent to the OS to write to disk. If the OS crashes before the WAL data has been written, the buffered data will be irretrievably lost. See [https://www.postgresql.org/docs/current/static/wal-async-commit.html](https://www.postgresql.org/docs/current/static/wal-async-commit.html) for more info.[^](#3293)*

[*[2]](#5d79) You can specify which replicas the primary cares about using the **synchronous_standby_names **setting. You can also use this setting to tune how many replicas the primary will wait for before returning success to a writer client. An empty **synchronous_standby_names** setting will cause the primary to only wait for writes to its own node, which is the equivalent of setting **synchronous_commit = local**. For more details, see our [tutorial](https://docs.timescale.com/v0.9/tutorials/replication).[^](#5d79)*

[*[3]](#86cd) The only case where this isn’t fully true is if **synchronous_commit **is set to **remote_write**. In this case the WAL data on replicas is guaranteed to be flushed to the OS for writing to disk, but could be lost if the OS crashes before it actually finishes writing the buffered data.[^](#86cd)*

[*[4]](#7920) We should note that this will mostly be beneficial in the case where the high write rates are not sustained consistently. If they are always higher than the replicas can keep up with, the replicas will fall further and further behind, eventually becoming useless. However, for bursty workloads where it is important to return quickly and consistently to the client, but the replicas are likely to have a relatively low write period at some point in the future, asynchronous replication can be highly beneficial.[^](#7920)*
