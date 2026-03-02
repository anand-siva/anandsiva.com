+++
title = "Rise of the DBSentinel"
date = "2026-03-01T20:15:48-05:00"
#dateFormat = "2006-01-02" # This value can be configured for per-post date formatting
author = "Anand Siva"
authorTwitter = "" #do not include @
cover = "meme_1.png"
tags = ["devops", "database", "golang", "open-source"]
keywords = [
  "dbsentinel",
  "database monitoring",
  "dba automation",
  "database outages",
  "open source database tool",
  "golang side project"
]
description = "Starting an open-source database monitoring tool to bring DBA insights into one place."
showFullContent = false
readingTime = false
hideComments = false
+++

I always wanted to create some type of software project. I look online and I see some pretty good ideas. I look at all the different SaaS tools that exist and the fancy websites that I see people spinning up with AI. At the same time I always wanted to create an open source project also. But where do I start?! Like I said before I kept looking at learning React and creating a pretty looking dashboard that backed a simple CRUD application. I have helped some people along the way with some fun side projects, but I never started my own. Not that I did not try, I had many many projects that ended up in the graveyard. Every time it would be the same thing, I would start, use AI to build a bunch of stuff and then have an existential crisis and quit. 

The tools I researched online started to all blend together. AI tools on AI tools that would read your docs, analyze your data, etc etc. Looks like all the tools have been created at this point so it kind of killed my motivation to start building myself. I know I know I always talk about self-esteem and building for the love of the game. But looks like all the problems I wanted to solve have already been solved.

Lately I have been reading articles about SaaS ideas that were booorrringgg but needed. A bunch of ideas about niche SaaS websites that serve a subset of a bigger group. The examples that were given was a booking tool for tattoo studios, tracking software for small freelancers, tailored tools for non-profits, and the list goes on and on. These types of apps would really be another CRUD type application. Something I tried many times and then just stopped.

I did not know it at the time, but those articles subtly motivated me to start on this upcoming endeavour. Why was I trying to create in spaces I really did not know anything about. Maybe that was why I did not have any momentum. I should start with something I know and has been a thorn in my side for years. Something you should know about me is that even though I have done a variety of different IT jobs, my first love was always databases. Ever since undergrad I loved databases, I thought they were so powerful. All the world runs on data including all these new AI systems. You know what else is one of those tasks that is crucially needed but most times heavily underappreciated until it goes wrong....database management. 

Some of the biggest outages in the last few years have been because of database crashes. 

{{< figure src="meme_2.png"  style="max-width:70%;" >}}

GitHub - https://github.blog/news-insights/company-news/oct21-post-incident-analysis/
GitLab - https://about.gitlab.com/blog/postmortem-of-database-outage-of-january-31/

Those are some big companies with A LOT of smart people and plenty of monitoring tools. If they can be affected by database outages like this, we all don't stand a chance.

Now let's get down to the real problem. I can say from personal experience it is REALLY hard to be an expert in all things database. There is general database server tuning, db-specific query optimization, expert level data modeling, and the all important database administrator (DBA). I have been a DBA on and off for years now and it is quite an unforgiving field. The type of work that goes unnoticed. Really the only time that type of work is noticed is when bad things happen and applications crash because of the database. Now there are tools out there, lots of tools. All types of tools that will show you slow queries, bad indexes, stats drift, and database engine details. The only problem...it takes so long to put all those things together to figure out what is going wrong with your database. Worst yet, your database can be a ticking time bomb without you knowing it. Looking at DB problems over the years is FRUSTRATING!! I am looking at slow logs and trying to compare it with indexes and then I have to switch and look at the checkpoints in the database to see if that caused an issue.

Idk where the idea came from, but I was thinking I have not written a blog in a week or so and I wanted to write this week. On top of that I wanted for a long time to create my own open source tool and learn Golang in the process. Just an hour ago it dawned on me that I could kill 3 birds with one stone. 

I am going to go for it! I am going to make an open source tool that helps monitor databases that can act as an automated DBA. Hell maybe this tool already exists out there and maybe someone has already solved this problem. All I know is that there are a lot less products and tools that support the "boring" stuff like databases.

I am going to call this open source tool DBSentinel. I am not exactly sure what it will fully do yet, but I will probably start it with just some simple stuff to start understanding Go. My end goal is to have a tool or processes that can sit alongside your database that can look after it. Show bad trends, statistics drift, slow query analysis, and anything else a DBA would usually do.

{{< figure src="meme_3.png"  style="max-width:70%;" >}}

Maybe if I do a good job I will get some GitHub stars, maybe if I do an excellent job someone will fork (steal) my tool and make it their own. No idea where this adventure will end up to be honest.

Just to show I am semi-serious here is the GitHub link with literally just a README:
https://github.com/anand-siva/dbsentinel
