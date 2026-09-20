+++
title = "What the Duck? How DuckDB Queried 100 Million Records in 2 Seconds"
date = "2026-09-20T14:37:35-04:00"
#dateFormat = "2006-01-02" # This value can be configured for per-post date formatting
author = "Anand Siva"
authorTwitter = "" #do not include @
cover = "duckdb-cover.jpg"
tags = ["duckdb", "parquet", "data-engineering", "analytics", "python"]
categories = ["devops"]
publications = ["No Docs Given"]
keywords = [
  "DuckDB tutorial",
  "DuckDB vs Spark",
  "DuckDB Parquet performance",
  "NDJSON vs Parquet",
  "query 100 million records",
  "single-node analytics"
]
description = "Can one machine really compete with Spark? I put DuckDB to the test with 100 million records and discover why Parquet turns a 67-second query into a two-second one."
showFullContent = false
readingTime = true
hideComments = false
+++

Let's continue with our trend of visiting technologies that are talked about a lot, but I have no idea what they mean. I have been hearing a bunch of hype about DuckDB in the data engineering world. From what I gathered, it was some type of single-node database. This confused me quite a bit. In the current world, everything runs in parallel with multiple processes, à la Spark and, originally, Hadoop. How is a single-node process supposed to help with mass data manipulation?

{{< figure src="have-you-heard-about-duckdb.jpg" alt="Screaming bird asking if you have heard about DuckDB" style="max-width:70%;" >}}

This stick figure diagram shows what I thought was the premier way to deal with massive amounts of data. 

```
Huge Dataset
     |
     v
+---------+---------+---------+
| Node 1  | Node 2  | Node 3  |
+---------+---------+---------+
     \         |         /
      \        |        /
       +-------+-------+
               |
            Result

```
More data → more machines → more parallelism → faster processing

This is what I discussed in my article about Apache Spark. Spark Joy: How I Cut 100M Record Processing from 4.5 Minutes to 76 Seconds

https://anandsiva.com/posts/no_docs_given/spark_joy/

After writing the article, I had done it. I had conquered data engineering. Spark was all I needed, along with Apache Iceberg. Now there is this little duck mocking me, saying DuckDB is here to stay.

I will be using a lot of duck puns in this article. I was just watching the movie *The Beekeeper* with Jason Statham, and they never backed down from the bee puns until the very end.

## What is DuckDB??

They call it "your universal data wrangling tool." DuckDB is an in-process analytical database designed to run fast SQL queries over large datasets on a single machine. Unlike traditional databases, it can query formats like Parquet and CSV directly without requiring you to first load the data into a database.

I have seen some places where they say it is SQLite's cousin. But idk, the main thing they have in common is that they can both be small, single-node databases. SQLite still needs the data to be loaded into the database. SQLite runs everywhere because it’s embedded directly into applications rather than running as a separate database server.

You’ll find SQLite in phones, browsers, desktop apps, IoT devices, and operating systems—anywhere an application needs a lightweight local database. DuckDB borrowed that embedded model, but optimized it for analytics instead of transactional application data.

This is where my confusion comes in. I read their main page and looked up some examples, but I still did not have a mental model of it. Don't get me wrong: I understand its power to read different file formats—most importantly for data engineering, Parquet files. Many of the examples are not even loading data into DuckDB. Instead, they use DuckDB's engine as a processing engine to read and write Parquet.

DuckDB is playing duck, duck, goose with my mind (strap in there are more puns coming). 

DuckDB is written primarily in C and C++ and has bindings for many popular programming languages, including Python, R, Java, Go, Rust, and Node.js. In this way, your application can call DuckDB through the language of your choice while DuckDB's native engine handles the underlying query execution, memory management, file access, and parallel processing.

### Single Node ≠ Single Thread

In my mind, I thought the DuckDB mental model looked like this:

```
DuckDB

          DuckDB
             |
          one CPU
             |
            :(
```

Really, it is more like this:

```
             One Machine
+--------------------------------+
|                                |
| CPU 0   CPU 1   CPU 2   CPU 3 |
| CPU 4   CPU 5   CPU 6   CPU 7 |
|   \       |       |       /    |
|        DuckDB Engine            |
|              |                 |
|          NVMe / RAM             |
+--------------------------------+
```

DuckDB parallelizes work within that machine. For example, Parquet row groups can be processed in parallel, and DuckDB's own storage similarly uses row groups as a unit of parallelism.

## Why Not Just Use Spark?

I am going to start this section by making one thing clear: you can't just drop DuckDB everywhere as a direct replacement for Spark. Spark exists for a reason. It is designed to distribute massive amounts of work across multiple machines, but doing that requires quite a bit of machinery.

Look at some of the things you need to think about with Spark:

* Scheduler
* Networking
* Serialization
* Shuffle
* Executors
* Coordination
* Cluster startup
* Data movement

After writing an entire article on Spark, I still don't fully understand all of it. It is a completely different way of thinking about computation. There is a planner figuring out how the work should be performed, executors actually doing the work, data potentially moving between those executors, and a whole lot happening behind the scenes.

The funny thing is that if I just look at my Spark code, much of that complexity isn't immediately apparent.

Think about a much simpler problem. Say you have **18.3 GB of data stored as Parquet files in S3** and you want to process that data and get some answers from it.

You have a few choices. You could write some code that goes through each file, parses the data, and keeps a running total. You could reach for Spark and distribute those files across workers so they can be processed in parallel.

But now I know there is another option, you know what time it is. Time to Duck it up. Let's see how DuckDB can be the best way to process this data.

## Rubber Duckie to the rescue

As always, let's use a real example here. I am going to fork the GitHub project I originally created for Apache Spark.

https://github.com/anand-siva/pyspark_example

Lol, how did I not know this? I can't fork this repo into my own namespace. I just have to pull it down and put it into a new project. So I am going to do that.

https://github.com/anand-siva/duckdb_exmaple

Let's go there first and fast-forward through the example until we get the data loaded into Parquet.

The records in this dataset look like this 

```
{
  "transaction_id": "uuid",
  "customer_id": 123456,
  "state": "MD",
  "product_category": "electronics",
  "amount": 149.95,
  "created_at": "2026-06-12T12:34:56.000000+00:00"
}
```
Let's compare the time required to create and store NDJSON files, as I did in the prior project.

I will be generating 100,000,000 records across 1,000 NDJSON files.

For comparison, this is how long the original Python script took to create and store the NDJSON dataset. In this case, "loading" includes generating every synthetic record in Python, serializing it as JSON, and uploading all 1,000 objects to MinIO. It does not refer to the later analytical query.

Normal Python generation and upload: ~10 minutes

```
Done.
Total records: 100,000,000
Total files: 1,000
Total time: 619.79s
Average throughput: 161,345 records/sec
```

```
./run_seed_ndjson.sh

Done.
Total records: 100,000,000
Total files: 1,000
Total size: 15.69 GiB
Total time: 189.42s
Average throughput: 527,940 records/sec
```

This version asks DuckDB to generate the rows, encode them as uncompressed NDJSON, and write them directly to MinIO. The reported time covers that data-generation and writing phase; it does not include one-time tasks such as creating the Python environment, installing dependencies, or starting MinIO.

The optimized version created the same number of records with the same schema about 3.3× faster, cutting runtime from 10.3 minutes to just over 3.1 minutes. The generated values are not identical to those from the original Python script, so this is a comparison of the two approaches rather than a byte-for-byte benchmark.

* Time saved: 430.37 seconds = 7 min 10 sec
* Runtime reduction: 69.4%
* Speedup: 3.27× faster
* Throughput: 161,345 → 527,940 records/sec
* Throughput increase: 227.2%

Really, if you want to see the power of DuckDB, you can generate the data and load it directly into Parquet files instead of NDJSON files.

```
./setup_test_data.sh
```

```
Done.
Total records: 100,000,000
Total files: 1,000
Total size: 2.92 GiB
Total time: 77.97s
Average throughput: 1,282,609 records/sec

=====================================
Setup complete
=====================================

Run the analytical query with:
  .venv/bin/python query_transactions.py

MinIO console: http://localhost:9001
Username: admin
Password: password123
```

The 77.97-second result is specifically the Parquet seeding phase: DuckDB generated the rows, encoded and compressed them with Zstandard, and wrote 1,000 Parquet objects to MinIO. The setup script also creates a virtual environment, installs dependencies, and starts MinIO when needed, but those one-time setup steps are not included in the reported seed time.

Damn, that was just over one minute right there! Next, we will query both datasets and see how fast DuckDB can process this data.

NDJSON query first—transaction count by state.

Spark example:

```
Transaction count by state:
CA: 10,005,069
FL: 10,004,135
IL: 10,000,918
MD: 10,002,238
NC: 9,994,621
NY: 10,003,472
PA: 9,999,562
TX: 9,995,115
VA: 10,001,618
WA: 9,993,252

Processed 100,000,000 records in 76.5s
```

DuckDB example:

```
./run_query_ndjson.sh

 󰄛   ./run_query_ndjson.sh

Revenue by state:
CA: $2,522,827,429.38
FL: $2,524,847,752.78
IL: $2,523,726,664.71
MD: $2,525,819,822.60
NC: $2,525,673,736.57
NY: $2,525,841,033.07
PA: $2,526,939,531.28
TX: $2,524,688,833.08
VA: $2,525,106,825.29
WA: $2,524,849,603.58

Transaction count by state:
CA: 9,992,559
FL: 9,997,004
IL: 9,996,278
MD: 10,001,673
NC: 10,001,354
NY: 10,004,212
PA: 10,007,385
TX: 9,999,777
VA: 10,000,515
WA: 9,999,243

Processed 100,000,000 records in 67.3s
```

At first glance, 67.3 seconds looks like a modest improvement over Spark's 76.5 seconds. This is not a controlled Spark-versus-DuckDB benchmark, though. The Spark result came from the separately generated dataset in my earlier project and calculated only transaction counts, while the DuckDB query calculated both counts and revenue. The different state totals above confirm that the underlying records are not identical.

What this comparison does show is that DuckDB handled a similar 100-million-record scan on one machine with a much simpler script. Even so, it still took over a minute. Why did NDJSON continue to take so long?

NDJSON may be easy for us to read, but it makes the query engine do a lot of work. DuckDB still had to read all 15.69 GiB of text, split it into 100 million lines, parse every JSON object, find the fields by name, and convert values such as `amount` into the correct data type before it could perform the aggregation. JSON does not store useful column types or statistics, so there is not much work DuckDB can skip. At that point, parsing the input becomes a large part of the query rather than the aggregation itself.

Parquet is built for this kind of analytical query. The same dataset was only 2.92 GiB in Parquet, so there was far less data to read. It also stored the values in typed, compressed columns. Since this query only needed `state` and `amount`, DuckDB could read those columns without decoding fields such as `transaction_id`, `customer_id`, `product_category`, and `created_at`. Parquet's metadata and row-group statistics can also help an engine avoid unnecessary reads when a query includes filters.

This is why changing the query engine alone did not transform the NDJSON result. The file format was still the bottleneck. But this is also why I loaded the same data as Parquet.

{{< figure src="ndjson-vs-parquet-drake.jpg" alt="Drake rejecting processing NDJSON files and approving processing Parquet files" style="max-width:70%;" >}}

Let's see how quickly DuckDB can query that version.

```
./run_query_parquet.sh

Revenue by state:
CA: $2,522,827,429.38
FL: $2,524,847,752.78
IL: $2,523,726,664.71
MD: $2,525,819,822.60
NC: $2,525,673,736.57
NY: $2,525,841,033.07
PA: $2,526,939,531.28
TX: $2,524,688,833.08
VA: $2,525,106,825.29
WA: $2,524,849,603.58

Transaction count by state:
CA: 9,992,559
FL: 9,997,004
IL: 9,996,278
MD: 10,001,673
NC: 10,001,354
NY: 10,004,212
PA: 10,007,385
TX: 9,999,777
VA: 10,000,515
WA: 9,999,243

Processed 100,000,000 records in 2.2s
```

2 seconds!!! 2 seconds!! Wow, that is pretty impressive for 1,000 files and 100,000,000 records.

That is roughly 30 times faster than the NDJSON query. DuckDB did not need to repeatedly interpret a huge wall of JSON text; it could go almost directly to the two compact columns needed for the calculation. The database engine mattered, but pairing it with a format designed for analytics is what produced the dramatic result.

## Why some people still don't give a duck

After running this experiment, I was like, WHAT? Why aren't more people talking about this?! Then I made the mistake of going on Reddit, and I found out why.

* I used AI to summarize this for me because it is Sunday and I am trying to watch the Jet's lose another football game in overtime

Of course, Reddit had opinions. The criticism was not that DuckDB is slow. It was that seeing a two-second query makes it very easy to assume DuckDB should replace every other data platform. People who had pushed it beyond its sweet spot were quick to point out where that idea falls apart.

One commenter summed up the scale question nicely:

> "DuckDB is amazing, but for the right scale. Snowflakes and Bigqueries of the world still have their place."

[Source: a discussion about using DuckDB in production](https://www.reddit.com/r/dataengineering/comments/1qwd5of/comment/ojkfjot/)

DuckDB can process data larger than memory by spilling work to disk, but it is still limited to the CPU, memory, and storage available on one machine. Some people reported that larger-than-memory workloads were less predictable than the fast, in-memory examples. My favorite description was:

> "I'd say it has two speed: crazy fast and [killed]"

[Source: a discussion about DuckDB memory issues](https://www.reddit.com/r/dataengineering/comments/1gyf53h/duckdb_memory_issues_and_postgresql_migration/)

That is one person's experience, not a universal benchmark. Other replies in the same discussion reported better results after setting memory limits and configuring a temporary directory. Still, it highlights an important difference: Spark can spread a workload across machines, while DuckDB eventually runs into the ceiling of the one machine it is running on.

Concurrency across independent processes is another concern. DuckDB is embedded inside an application process rather than running as a traditional shared database server. That is great for local analytics and data pipelines, but it changes what it can safely replace:

> "The limitation is it's single-process—no concurrent write access, so anything with multiple users writing data simultaneously is a no-go."

[Source: another discussion about DuckDB in production](https://www.reddit.com/r/dataengineering/comments/1qwd5of/comment/o3osziz/)

That comment simplifies the situation a little. DuckDB supports concurrent writer threads inside one process. The limitation appears when independent processes need to write to the same native DuckDB database file; supporting that requires additional architecture instead of the normal embedded setup. The [DuckDB concurrency documentation](https://duckdb.org/docs/current/connect/concurrency) explains the available options.

So it is not that people simply do not like DuckDB. The real argument is about using it for the right job. It looks fantastic for local analysis, embedded analytics, and single-machine data pipelines. It looks much less like a direct replacement for a distributed compute engine, a shared warehouse, or a transactional database serving lots of concurrent writers.

## DuckDB for the win?

My personal take is that DuckDB is an interesting piece of technology, and I think it has its place. I also think that DuckDB is only as good as the storage format you give it. Working with Parquet instead of NDJSON? Hell yes!! For smaller-scale analytics, this tool seems amazing.

I am also slightly biased because you can use SQL to query data with DuckDB. Do I think it can replace Spark when you need to churn through 1 TB of data? Definitely not. But will it help in an application where you want to query a day's worth of data—around 100 GB—with less code than Spark? Most probably.

It is another tool to add to the toolkit. I will have to experiment with it some more to really see where it fits in this new world of data engineering.

I have reached the end of this article, and now, after writing about Apache Spark and DuckDB, *I have fully mastered modern data engineering* [/s]. I learned a lot about the duck today and why I should care about it. I will look for places to implement it and report back with better use cases. Until next time, data nerds!!!

{{< image src="duckdb-data-nerds.gif" alt="Me and my data nerds talking about DuckDB" style="width:100%; max-width:100%;" >}}
