+++
title = "Let's Get Ready to Parquet: Why Columnar Storage Changed Data Engineering"
date = "2026-08-07T18:10:09-04:00"
#dateFormat = "2006-01-02" # This value can be configured for per-post date formatting
author = "Anand Siva"
authorTwitter = "" #do not include @
cover = "parquet-cover.png"
tags = ["parquet", "data-engineering", "spark", "analytics", "columnar-storage"]
categories = ["devops"]
publications = ["No Docs Given"]
keywords = [
  "apache parquet tutorial",
  "what is parquet file format",
  "parquet vs csv performance",
  "columnar storage explained",
  "parquet compression example",
  "data engineering parquet"
]
description = "A practical introduction to Apache Parquet, why columnar storage matters, and how Parquet compares to CSV in size and read performance."
showFullContent = false
readingTime = true
hideComments = false
+++

You know when you have been saying a word for a while and you think you absolutely nailed the pronunciation? That was me with "parquet." I kept pronouncing it as paar . ket. Well I was sorely mistaken because it is actually pronounced paar · kay.

But what exactly in the hell is Parquet? If you have been around data engineering the last few years, you have heard the term Parquet. Store the data in a Parquet file. Move the Parquet files around. Use Spark to process the Parquet files. For a while there I did not bother to look into what exactly that meant. All I knew was that it was some type of file format used for storing data.

It is so much more than that, it is the backbone of the lakehouse architecture and modern data querying. When I started getting back into data engineering, I definitely took it for granted. I just used AWS services to drop files into this format without thinking twice. I really just thought it was the trendy toy of the year. It is so much more than that!

To understand why Parquet became one of the most widely adopted data formats, we need to rewind to the Hadoop era. Hadoop made it possible to store enormous amounts of data cheaply, but analyzing that data was still expensive because every query often required scanning entire files, even when only a handful of columns were needed.

The data that was processed grew from gigabytes to terabytes to petabytes. Anyone that has dealt with this enormous amount of data understands this pain. As a former database developer, the biggest pain with aggregate queries was that the standard database formats were TERRIBLE at querying large swaths of data. In their defense, large-scale analytics was not what the storage was designed for. Also, most times with aggregates you do not need all the columns, most of the time you really only need to read 2-3 columns.

And this is where Parquet comes in.

## What is Parquet?

Think of Parquet as a file format built for asking questions about data rather than simply storing it. By organizing data into columns instead of rows, it enables tools like Spark, Athena, Trino, DuckDB, and Snowflake to scan less data, compress files more efficiently, and execute analytical queries much faster than traditional formats.

Now this is where most articles would go into an in-depth explanation of columnar storage and how the compression works and all that jazz. There are docs for all of that, the GitHub is right here:

https://github.com/apache/parquet-format/

When I first visited the Apache Parquet GitHub repository, I expected to find a utility or library that created Parquet files. Instead, the more I read, the more I realized that wasn't its purpose at all.

The repository primarily contains the Parquet file format specification. In other words, it defines how Parquet files are structured, how metadata is stored, and the rules that implementations follow when reading and writing Parquet data. It isn't a production library that you can simply import into your application.

If you're looking for an actual implementation that can read and write Parquet files, you'll want one of the language-specific libraries that implement the specification. Here are a few popular ones:

* Java: Apache Parquet Java (parquet-java)
* Python: PyArrow, Fastparquet
* C++: Apache Arrow C++
* Go: parquet-go
* Rust: Apache Arrow Rust
* .NET: Parquet.NET
* JavaScript: hyparquet

I was about to explain exactly what the format was like, but realized I have no idea. I write these articles while I am trying to learn this as well. I think it is a great way to learn and really motivates me instead of just reading through all the docs.

## Parquet format

Without getting too technical, here is how a Parquet file is formatted:

```
+-------------------+
| File Header       |
+-------------------+
| Row Group 1       |
|  ├─ Column A      |
|  ├─ Column B      |
|  └─ Column C      |
+-------------------+
| Row Group 2       |
|  ├─ Column A      |
|  ├─ Column B      |
|  └─ Column C      |
+-------------------+
| ...               |
+-------------------+
| File Metadata     |
+-------------------+
| Footer            |
+-------------------+
```

This type of format really threw me off. I have never seen an actual columnar format before. If you are anything like me, this looks strange as hell.

Let me try a small example to see if I even understand it.

Let's say I have a CSV like this:

| Name | Age | Color | Favorite Toy |
|------|-----|-------|--------------|
| Luna | 2 | Black | Feather Wand |
| Oliver | 5 | Orange Tabby | Catnip Mouse |
| Willow | 1 | Gray | Crinkle Ball |

In Parquet code it would look like this:

```
Name = [
  "Luna",
  "Oliver",
  "Willow"
]

Age = [
  2,
  5,
  1
]

Color = [
  "Black",
  "Orange Tabby",
  "Gray"
]

FavoriteToy = [
  "Feather Wand",
  "Catnip Mouse",
  "Crinkle Ball"
]
```

My first question was how does it know that rows match up, but it stores this data in that order. Each column stores its values in the exact same sequence.

So in our example:

* Index 0 represents Luna.
* Index 1 represents Oliver.
* Index 2 represents Willow.

Now I had to look this up because how in the hell does the file only read certain chunks in the overall file?

The answer is the file's metadata. Every Parquet file contains a footer that records the exact byte offsets of each column chunk. A query engine reads this metadata first, then jumps directly to the columns it needs instead of scanning the entire file.

What type of data is in these headers and footers? 

| Section | Example Metadata |
|---------|------------------|
| **Header** | `PAR1` (magic number identifying the file as Parquet) |
| **Footer** | **Schema:** `Name (STRING)`, `Age (INT)`, `Color (STRING)`, `Favorite Toy (STRING)` |
| **Footer** | **Row Count:** `3` |
| **Footer** | **Column Offsets:** `Name @ 1,024`, `Age @ 2,048`, `Color @ 3,072`, `Favorite Toy @ 4,096` |
| **Footer** | **Compression:** `Snappy` |
| **Footer** | **Statistics:** `Age (min=1, max=5)`, `Name (min="Luna", max="Willow")` |


I love the statistics stored in the metadata. Instead of reading every row, a query engine can first inspect the metadata—such as the minimum and maximum values for each row group—to determine whether it even needs to read that section of the file. In many cases, this allows it to skip large portions of the data entirely.

^ See those em dashes above, oh yeah I used AI to clean up my explanation. Side tangent: I think I find AI useful to help me explain specific technical issues. That is because when I write something that I do not know too much about, I double check it. If it is a new topic, more often than not, I am a little off with my first explanation.

{{< image src="parquet-em-dashes.png" alt="Meme joking that I am a liar for pretending I did not use AI to help with the explanation" style="width:100%; max-width:100%;" >}}

The next thing that I noticed is the compression. From the Parquet GitHub:

"Parquet is built to support very efficient compression and encoding schemes. Multiple projects have demonstrated the performance impact of applying the right compression and encoding scheme to the data. Parquet allows compression schemes to be specified on a per-column level, and is future-proofed to allow adding more encodings as they are invented and implemented."

Wow, how powerful and open-ended they made this specification. Instead of locking it in, the authors decided to leave the door open to newer encodings. This is what gives a tool a long shelf life, when the original schematic is made with enough flexibility it can keep up with the times.

## Let's try it out!!!

As always I think the best way to understand this concept is to try it out yourself. If you want to follow along with the examples, here is the sample GitHub repo.

https://github.com/anand-siva/parquet-demo

Let's start by running the startup script to create a virtual environment.

```
bash scripts/setup.sh

 󰄛   bash scripts/setup.sh
Creating virtual environment at /Users/amoney/anandsiva.com/parquet-demo/.venv
Activating virtual environment
Upgrading pip
Requirement already satisfied: pip in ./.venv/lib/python3.14/site-packages (26.1.2)
Collecting pip
  Downloading pip-26.2.1-py3-none-any.whl.metadata (4.6 kB)
Downloading pip-26.2.1-py3-none-any.whl (1.8 MB)
   ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ 1.8/1.8 MB 12.7 MB/s  0:00:00
Installing collected packages: pip
  Attempting uninstall: pip
    Found existing installation: pip 26.1.2
    Uninstalling pip-26.1.2:
      Successfully uninstalled pip-26.1.2
Successfully installed pip-26.2.1
Installing requirements

Setup complete.
Activate the environment with:
source /Users/amoney/anandsiva.com/parquet-demo/.venv/bin/activate

-- run this source command

source /Users/amoney/anandsiva.com/parquet-demo/.venv/bin/activate
```

Next let's create our CSV file:

```
scripts/generate_csv.py

python3 scripts/generate_csv.py
100,000 rows written - 3.08 MB
200,000 rows written - 6.16 MB
300,000 rows written - 9.23 MB
400,000 rows written - 12.31 MB
..............
32,500,000 rows written - 999.99 MB
32,600,000 rows written - 1003.07 MB
32,700,000 rows written - 1006.15 MB
32,800,000 rows written - 1009.23 MB
32,900,000 rows written - 1012.30 MB
33,000,000 rows written - 1015.38 MB
33,100,000 rows written - 1018.46 MB
33,200,000 rows written - 1021.54 MB

Done!
Rows: 33,280,375
File Size: 1.00 GB
```

Let's check out this CSV:

```
cat data/cats.csv| head -2
Name,Age,Color,FavoriteToy
Dfwlhxbippy,2,White,Catnip Mouse
```

Ok now I created a Python script to convert this into Parquet:

```
python3 scripts/convert_to_parquet.py

CSV written from: /Users/amoney/anandsiva.com/parquet-demo/data/cats.csv
Parquet written to: /Users/amoney/anandsiva.com/parquet-demo/parquet/cats.parquet
CSV size: 1024.01 MB
Parquet size: 375.85 MB
Size reduction: 63.30%
```

Wow that is a crazy size reduction. It is a little misleading though since I have a lot of repeating data, but you can see how it would help in columns that only have a few options.

Let's check out this parquet file!

```
 󰄛   cat parquet/cats.parquet| head -10
PAR1Ȝ���nL�
��@�
    DfwlhxbippyZpjrq
Utsurtcfcc
Yoaapfvkbr	XpbgfmbfOfspwdus	Dyugbstvu
HFpigxxbggAtyqwf                                 LpbsmnjeqfdSivhqkh	Uaolgsjql	EpbmowqrlQaizOygdbpecZhokzeyRqftzwZiiggk	HrtpcdmmkPpajl� Fpccybsuj
Mltsjd! Mzgwjhuvh�Rftfikqs!Ytxic!5(Jdgvfehaggw$WwilbeulGIaiaah ZcckruqgsTubigypsy?(Nbyzwqheucz?GptorxpkqmyqlwcdugZgecsmx%TWjebpbdeUsejmoja�(Pfcpqftbhws$Qjjwpfdqvj�Pxbpv�Khgqwpo�$JzbztrlmdfPaocrzk�Wychz	Piuyx	CpmcjW(Ygjjfpewtgd�Dkfqhqs� Onxyyowky\$Lltnawvibr KytqlygykMDnfbusAuvscu%i(RiabplvxclvXJbkxczoE0(Flnyhvwpofh�(Pnnsrrgplxg�
                                                                                                                                          SnfyE(Jneokewcuki\Fjrjehq#PnugjVEgfwbmfm!�Xapnfa�$Ihphwqjpz
```

haha I had no idea why I thought when I opened up this file I would see human readable stuff. Actually if you look close enough you can see some human words like my name "Siva" in the file. That's because string values and some metadata are stored in the binary format. You're just seeing tiny islands of readable text surrounded by binary data that only a Parquet reader knows how to interpret.

Now this next step is a benchmark on how you can query the CSV and then the Parquet. There is no way in real life you would ever need to do this. I like to think about this benchmark as a fun little test. In normal operations, I would compare this type of query between a MySQL database and an Iceberg data lake.

Let's run the benchmark next:

```
python3 scripts/benchmark.py

Read CSV: 11.5305s (33,280,375 rows)
Read Parquet: 1.4875s (33,280,375 rows)
Parquet speedup: 7.75x
```

This only shows how long it takes to load the dataset into memory, but it was cool to see that since it is smaller it can load faster.

This is just scratching the surface on the beauty and power of the Parquet file format. Interesting how a file format specification could change the course of data engineering.

I have personally run many queries in a data lake that would absolutely time out in a MySQL database. And when I say ran them I mean they ran in milliseconds and seconds instead of minutes or hours. Once you see the power of this file format you cannot look back. I was happy to write an article about the backbone of modern data engineering. It continues to be the unsung hero of the data era.
