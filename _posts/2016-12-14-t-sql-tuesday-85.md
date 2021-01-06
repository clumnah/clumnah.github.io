---
layout: post
title: 'T-SQL Tuesday #85'
date: 2016-12-14 06:57:59.000000000 -05:00
type: post
parent_id: '0'
published: true
password: ''
status: publish
categories:
- General
- Powershell
- SQL Server
tags:
- T-SQL Tuesday
- tsql2sday
# meta:
#   _edit_last: '111693662'
#   geo_public: '0'
#   _rest_api_published: '1'
#   _rest_api_client_id: "-1"
#   _publicize_job_id: '29979518635'
#   _publicize_done_external: a:1:{s:7:"twitter";a:1:{i:15852853;s:52:"https://twitter.com/lumnah/status/809004535645687808";}}
#   _publicize_done_16023606: '1'
#   _wpas_done_15852853: '1'
#   publicize_twitter_user: lumnah
#   publicize_linkedin_url: https://www.linkedin.com/updates?discuss=&scope=17000093&stype=M&topic=6214770229285777408&type=U&a=t-JU
#   _publicize_done_16023607: '1'
#   _wpas_done_15852854: '1'
# author:
#   login: clumnah
#   email: clumnah@gmail.com
#   display_name: Chris Lumnah
#   first_name: Chris
#   last_name: Lumnah
permalink: "/2016/12/14/t-sql-tuesday-85/"
---
![tsql2sday](/assets/images/2016/11/tsql2sday150x150.jpg)

For the last T-SQL Tuesday of the year, this month is being hosted by Kenneth Fisher ([b](https://sqlstudies.com/2016/12/06/4169/)\|[t](https://twitter.com/sqlstudent144)) and is about backup and recovery.

Any DBA knows that backups are crucial to their job. Without good backups it is very difficult to ensure that the environment is protected from dangers that lurk all around. Any DBA knows the BACKUP and RESTORE commands fairly well. They have scripts handy that can do a backup or a restore of database ready to go at the slightest whisper of a problem. However there are new commands that are available via Powershell to assist a DBA in getting a backup or a restore completed. These can possible aid in doing those items faster, as you do not need to wait for SSMS to load. Lets take a look at these commands at their basic level. I am going to use WideWorldImporters and my SQL 2016 instance.

# Restore Database

## T-SQL

```sql
USE [master]  
RESTORE DATABASE [WideWorldImporters]  
FROM DISK = N'E:\Backup\WideWorldImporters-Full.bak'  
WITH FILE = 1,  
  MOVE N'WWI\_Primary' TO N'E:\SQLData\WideWorldImporters.mdf',  
  MOVE N'WWI\_UserData' TO N'E:\SQLData\WideWorldImporters\_UserData.ndf',  
  MOVE N'WWI\_Log' TO N'E:\SQLLog\WideWorldImporters.ldf',  
  MOVE N'WWI\_InMemory\_Data\_1' TO N'E:\SQLData\WideWorldImporters\_InMemory\_Data\_1',  
  NOUNLOAD,  
  STATS = 5  
GO  
```

## Powershell

```Powershell
$RelocateData1 = New-Object Microsoft.SqlServer.Management.Smo.RelocateFile('WWI_Primary', 'E:\SQLData\WideWorldImporters.mdf')  
$RelocateData2 = New-Object Microsoft.SqlServer.Management.Smo.RelocateFile('WWI_UserData', 'E:\SQLData\WideWorldImporters_UserData.ndf')  
$RelocateLog = New-Object Microsoft.SqlServer.Management.Smo.RelocateFile('WWI_Log', 'E:\SQLLog\WideWorldImporters.ldf')  
$RelocateData3 = New-Object Microsoft.SqlServer.Management.Smo.RelocateFile('WWI_InMemory_Data_1', 'E:\SQLData\WideWorldImporters_InMemory_Data_1')  
Restore-SqlDatabase -ServerInstance 'sql2016' -Database 'WideWorldImporters' -BackupFile 'E:\Backup\WideWorldImporters-Full.bak' -RelocateFile @($RelocateData1,$RelocateData2,$RelocateData3,$RelocateLog)  
```

# Backup Database

## T-SQL

```sql
BACKUP DATABASE [WideWorldImporters]  
TO DISK = N'E:\Backup\WideWorldImporters-Full.bak'  
WITH NOFORMAT,  
  NOINIT,  
  NAME = N'WideWorldImporters-Full Database Backup',  
  SKIP,  
  NOREWIND,  
  NOUNLOAD,  
  STATS = 10  
GO  
```

## Powershell

```Powershell
Backup-SqlDatabase -ServerInstance 'sql2016' -Database 'WideWorldImporters' -BackupFile&nbsp; 'E:\Backup\WideWorldImporters-Full.bak'  
```

As you can see, these basic commands are very similar. With some additional review and time, the possibilities to automate a backup or restore of a database, or a group of databases, becomes quite easy. Better yet, these commands are native T-SQL and Powershell statements.

What if you want to go deeper? What if you want to answer the following questions

1. When was my last database backup?
2. What is the list of the last backups that were done?
3. When was the database restored?

DBAs have scripts that will answer these questions relatively quickly using T-SQL. If they do not, they will spend time writing the code themselves or spend time searching online for a query that someone else has published already that answers the questions. What if you want to do this via Powershell? At this time, there is no native command provided by Microsoft to answer the above questions. However, the SQL Community has taken it upon themselves to create Powershell commands that will answer the above questions and do much more.

Chrissy LeMaire ([b](https://blog.netnerds.net/)\|[t](https://twitter.com/cl)), a Data Platform MVP out of Belgium, and few other DBAs and MVPs have teamed up to create a suite of Powershell commands aimed at DBA. I encourage you to check out their site, [https://dbatools.io/](https://dbatools.io/), and download their module. The suite covers a bunch of common DBA tasks as well as database migration tasks. Here I will highlight 3 commands that will answer the above questions.

## When was my last database backup?

This is a simple question that we as DBAs get asked, and ask a lot. With the dbatools Powershell module, this question can be answered in one line.

```
Get-DbaLastBackup -SqlServer sql2016 | Out-GridView  
```
The above simple command returns back the below data

![capture1](/assets/images/2016/12/capture1.png)

## What is the list of the last backups that were done?

This is another question we all need to answer from time to time. Generally, we will need to answer this question when we need to a restore and need to know what files have been created recently and where they exist.

```
get-dbabackuphistory -SqlServer sql2016 -Databases WideWorldImporters | Out-GridView  
```

This above one liner, returns back the below output. This tells me that my latest backups were a transaction log and a full backup. From here I can go and perform the restore if I so desired.

![capture2](/assets/images/2016/12/capture2.png?w=300)

What is great about a command like this is that I can take this and create subsequent restore commands in Powershell and automate the restore.

## When was the database restored?

Lastly, at times we need to know when a database was restored and from what file it was restored from. Again, a Powershell one liner from dbatools comes to our rescue to quickly and efficiently answer the question.

```
Get-DBARestoreHistory -SqlServer sql2016 -Databases WideWorldImporters -Detailed | Out-Grid  
```

Here is the output:

![capture3](/assets/images/2016/12/capture3.png?w=300)

Powershell is a great tool for the DBA to become more efficient in the normal day to day work. The above commands that I have highlighted above is just scraping the surface. To learn more in depth info about these commands, please read Chrissy’s [blog](https://blog.netnerds.net/2016/12/dbatools-to-the-backuprestore-rescue/) which explains this new functionality much deeper and with videos.

The team that is putting together the dbatools suite, is doing a great job coming up with commands that are useful now to all DBAs. I hope you explore what they have to offer and incorporate them into your daily work.

