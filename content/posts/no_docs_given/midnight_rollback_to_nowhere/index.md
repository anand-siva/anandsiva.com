+++
title = "Midnight Rollback to Nowhere: MySQL Crash Recovery and Rollback Hell"
date = "2026-04-08T21:13:40-04:00"
author = "Anand Siva"

cover = "mysql_shutdown_cover.png"

tags = ["mysql", "database", "innodb", "troubleshooting", "devops"]
categories = ["devops"]
publications = ["No Docs Given"]

keywords = [
  "mysql rollback stuck",
  "innodb crash recovery",
  "mysql oom kill recovery",
  "show engine innodb status rolling back",
  "information_schema innodb_trx rollback",
  "large transaction mysql import"
]

description = "A practical look at MySQL crash recovery, massive transaction rollbacks, and why an OOM-killed database can get stuck in rollback hell."

showFullContent = false
readingTime = true
hideComments = false
+++

## The Night Everything Stops

There is something strange that happens to MySQL instances when things go bad. If you are lucky and in the middle of just normal operations and the memory gives out and crashes, you will be able to restart it and get back to normal bussiness. 

But sometimes something strange happens. Stop me if you have heard this story before. You are doing some data loading into a mysql server, you walk away from the computer and just assume in an hour or two the mass upload of data in your mysql instance will be completed. It is worked many times before, so why would it change now. You get back to your computer and mysql is frozen. You can't even ssh into the box because memory is tapped out. You finally get to the box and see that mysql server is restarting, most probably OOM killed. You think to yourself, well that sucked but looks like mysql is recovering. You wait, you wait you wait. The startup is taking sooo long, over 15 mins. 

So you do the thing you have been trained to do. Looks like this box does not have enough memory. You try and shut the box down....nothing. Getting tired of this game, you decide to force stop the instance in AWS. Alright fine it took 5 mins but it is shutdown now. Go into the console, up the instance size, up the iops and  throughput of the EBS disk. Looks like we are ready to go! Startup the instance. WHAT?! Startup is still taking 15+ mins. Ohhh wait you forgot to change your my.cnf file to create a bigger buffer pool. First step, shutdown the database. Simple right, you got more memory, it should shutdown quickish. 5 mins goes by, 10 mins goes by, 15, 20. What the hell is going on?! 

## Is MySQL Actually Stuck?

Small side quest, how do you even know mysql is doing anything when it is stuck in this startup or shutdown loop. iostat of course. 

iostat (Input/Output statistics) is a Linux command that shows how busy your disks are and how much I/O your system is doing.

It tells you whether your system is waiting on disk… or actually doing work.

```
iostat -x 1

avg-cpu:  %user   %nice %system %iowait  %steal   %idle
          3.21    0.00    2.14   65.87    0.00   28.78

Device            r/s     w/s   rkB/s   wkB/s  rrqm/s wrqm/s  %util
nvme0n1          5.00  1200.00   80.00 45000.00   0.00   0.00  99.80
```

* %iowait is high (60%+) → CPU is waiting on disk
* w/s is huge → tons of writes (undo activity 👀)
* %util ~100% → disk is saturated

MySQL isn’t “stuck”. It’s hammering the disk trying to undo your life choices

## How a Giant Transaction Turns Into Rollback Hell

To understand what is going on let me give you a scenario on how this happens. 

Imagine a bulk import job loading a massive CSV into MySQL inside a single transaction. Maybe the goal was to keep the load atomic: either all the rows land, or none of them do. That sounds safe on paper. But if the server runs out of memory or gets killed halfway through, InnoDB does not just shrug and move on. It has to roll that transaction back.

```sql
LOAD DATA INFILE '/data/imports/customer_events_2026_04_01.csv'
INTO TABLE customer_events
FIELDS TERMINATED BY ','
OPTIONALLY ENCLOSED BY '"'
LINES TERMINATED BY '\n'
IGNORE 1 LINES
(
  account_id,
  event_id,
  event_type,
  event_time,
  campaign_id,
  source_number,
  destination_number,
  duration_seconds,
  revenue_cents,
  created_at
);
```

The problem is not the CSV itself. The problem is that this entire import is wrapped in one giant transaction. If that file contains 40 million rows, MySQL may spend a very long time inserting changes into InnoDB, updating indexes, writing undo records, and consuming memory along the way. If the server is OOM-killed before COMMIT, MySQL has to recover by rolling the whole thing back on startup. That rollback can take just as long, or sometimes longer, than the original load.

In theory, this is clean and transactional. In practice, if the instance dies after ingesting millions of rows but before the commit finishes, you are now trapped in the world’s least satisfying progress bar: crash recovery and rollback.

This is not very clear to most people, including myself, even after years of mysql admin experience. 

Crash recovery itself sucks because MySQL cannot just restart and hope for the best. It has to replay what was durable, figure out what was unfinished, and make the database consistent again. To make the mysql instance usable again, you will need to make the database consistent. Guess what needs to be done? The database needs to walk through each undo log during the rollback. When you are loading all that data what happens is MySQL writes two logs. A redo log, for committed work that needs to be replayed incase of a crash. Also it writes an undo log for EACH change, which basically says how to undo this row insert.

--- 

## Wait transactions are written to disk?

At this point, your disks are being hammered because MySQL is trying to undo all of those transaction writes. What looks like a frozen database is actually InnoDB working through a massive rollback, one change at a time.

This is an internal aspect of mysql that confused the hell out of me. I had assumed that while this transaction was being done, it was basically saving it into memory and then at the end writes it to disk. Boy was I wrong. 

InnoDB is constantly writing to disk during the transaction.

Even before commit:

* Data pages get flushed to disk (buffer pool eviction, checkpoints)
* Index pages get written
* Redo logs are definitely written to disk

👉 So your “uncommitted” transaction is already partially persisted

* Undo logs are on disk too
* Undo records live in undo tablespaces (disk)
* Not just memory

They can be HUGE for big transactions

When a crash happens: 
Now MySQL restarts and sees:

“This transaction was never committed”
“But I already wrote a ton of its data to disk”

👉 It must undo it

Rollback is not a memory operation. It is a disk-heavy, write-heavy process that re-touches the same data you just wrote—again.

Your transaction wasn’t in memory—it was halfway written to disk, and now MySQL has to unwrite it.

--- 

## Check rollback status

To even inspect what was going on, I had to force kill MySQL and bring it back up cleanly. During normal operation, it was stuck in a state where it wouldn’t fully start or stop, making it nearly impossible to introspect.

Once MySQL was starting up again, I could finally catch it in the act during crash recovery.

Check innodb stats and check for this line 

```
SHOW ENGINE INNODB STATUS\G

Trx id counter 123456
Purge done for trx's n:o < 123400 undo n:o < 0 state: running but idle

ROLLING BACK 38294721 rows
```

Here is the query to run to monitor the rollback 

```
SELECT 
    trx_id,
    trx_state,
    trx_started,
    trx_rows_modified,
    trx_rows_locked,
    trx_query
FROM information_schema.innodb_trx\G

*************************** 1. row ***************************
trx_id: 874562391
trx_state: ROLLING BACK
trx_started: 2026-04-08 19:42:11
trx_rows_modified: 38472951
trx_rows_locked: 0
trx_query: LOAD DATA INFILE '/data/imports/customer_events_2026_04_01.csv' INTO TABLE customer_events

```

Keep running that last SQL query, you will see this number go down trx_rows_modified: 38472951. Eventually it will just return no data.

---

## Databases Never Say Die

After `innodb_trx` returned nothing and InnoDB status showed no rollback, I figured I’d try shutting it down one more time.

How long would it take this time?

I braced for another 5–10 minute wait.

It shut down in 10 seconds.

10… seconds.

After everything—crashes, forced restarts, endless waiting—that’s when it clicked. This is why I fell in love with databases in the first place. No matter how bad things get, they fight their way back to a consistent, working state.

Battle-tested technology.

The moral of the story: if your MySQL instance is taking 30+ minutes to shut down, something is wrong. Don’t just throw more resources at it—check if you’re stuck in rollback hell.

{{< figure src="db_no_die.png"  style="max-width:70%;" >}}
