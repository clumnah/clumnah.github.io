---
layout: post
title: New job...new thinking?
date: 2018-03-13 08:00:26.000000000 -04:00
type: post
parent_id: '0'
published: true
password: ''
status: publish
categories:
- General
- SQL Server
tags: []
# meta:
#   _rest_api_published: '1'
#   _rest_api_client_id: "-1"
#   publicize_twitter_user: lumnah
#   _publicize_job_id: '15680012670'
#   timeline_notification: '1520943149'
#   _publicize_done_external: a:1:{s:7:"twitter";a:1:{i:15852853;s:52:"https://twitter.com/lumnah/status/973532247142928384";}}
#   _publicize_done_16023606: '1'
#   _wpas_done_15852853: '1'
#   publicize_linkedin_url: https://www.linkedin.com/updates?discuss=&scope=17000093&stype=M&topic=6379297938870398976&type=U&a=3PQU
#   _publicize_done_16023607: '1'
#   _wpas_done_15852854: '1'
# author:
#   login: clumnah
#   email: clumnah@gmail.com
#   display_name: Chris Lumnah
#   first_name: Chris
#   last_name: Lumnah
permalink: "/2018/03/13/new-job-new-thinking/"
---
So I recently started a new position as a SQL Server Solutions Architect at [Rubrik Inc.](https://www.rubrik.com) We concentrate on data protection of your data center. I will be concentrating on SQL Server specific protection. Hit up the website or search YouTube for some videos if you want to know more.

Backup and recovery has always been something I am interested in. It was the first skill I learned when becoming a DBA. I wrote my own database backup process using SSIS before Powershell was a thing, that was highly dynamic, configurable, and most importantly simple to support. With the introduction of the Data Domain, I learned how to optimize the performance of a backup and restore. Over my career, I have learned that backup and recovery and data protection is probably the single biggest job the DBA has.

In my new role, it is expected of me to keep current and more importantly learn new things. SQL Server 2016 introduced a feature called Stretch Database. Essentially, this feature allows you to take data in a table or tables and put part of your data in Azure and part stays local. Conceptually, this works similarly to table partitioning but mechanically it is different. Table partitioning separates your table into partitions based on a query function. All of your data is in the table, it is just reorganized into those partitions so queries can run more efficiently against smaller subsets of data. Stretch is similar in that you have a filter query that says what data stays local and what goes to Azure. The big difference here is that the data going to Azure actually leaves the local database.

Now, why would you do this? A good example of why, may be a large sales order system. For the most part, your queries may run against the most recent data. By keeping that data local, queries can return fast. Data that needs to read that old data will have to reach into Azure to get it. Stretching a database allows you to place that old data upon cheaper storage. Yes, this data will probably take longer to query, but that is by design. Place the old, less used data on cheap storage to free up the local faster storage for more often used data.

I started to think about the implications of backup and recovery. How this affects RPO and RTO.

**Backup**

Thankfully, to back up a database that has been stretched to Azure existing native backup scripts do not need to change. This sounds great, I can offload a bunch of data to the cloud and nothing has to change from a backup and recovery perspective.

Well not so fast.

When you do a native SQL backup, SQL Server will perform what is called a shallow backup. This means that the backup will contain only the local data and eligible for migration at the time the backup runs. Eligible data is the data is not yet migrated, but will be migrated to Azure based on the query function.

So what happens to the data that has been moved to Azure? Azure will backup that data for you automatically with storage snapshots done every 8 hours.

See this [link](https://docs.microsoft.com/en-us/sql/sql-server/stretch-database/backup-stretch-enabled-databases-stretch-database) for more details.

**Restore**

So I have a backup or really backups of my data, how do I do a restore? Well it depends....Where is the failure?

If the failure you are recovering from is local,

1. Restore your local database as you normally would from a native SQL backup.
2. Run the stored procedure&nbsp; **sys.sp\_rda\_reauthorize\_db** to reestablish **&nbsp;** the connection between the local database and the Azure stretch database

If the failure is in Azure,

1. Restore the Azure database in the Azure portal
2. Run the stored procedure&nbsp; **sys.sp\_rda\_reauthorize\_db** to reestablish **&nbsp;** the connection between the local database and the Azure stretch database

While the above looks simple, it can be bit more involved as you walk through the details &nbsp;[here](https://docs.microsoft.com/en-us/sql/sql-server/stretch-database/restore-stretch-enabled-databases-stretch-database#reconnect).

Ultimately, stretch database is a cool piece of technology that makes you think differently. It forces you to think more of a recovery strategy than a backup strategy. You have to test out how to restore this type of database to understand the steps involved. You want to have a run book that shows what you need to do to recover a database that is involved with stretch. You want to plan out your backups to most effectively give you a successful recovery. Essentially, you need a recovery strategy, not a backup strategy.


Hmmm, sounds like something [Paul Randal](https://www.sqlskills.com/blogs/paul/the-accidental-dba-day-8-of-30-backups-planning-a-recovery-strategy/) has been saying for a while. So basically the same thinking, just reenforced.

