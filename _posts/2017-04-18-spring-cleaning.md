---
layout: post
title: Spring Cleaning
date: 2017-04-18 07:27:19.000000000 -04:00
type: post
parent_id: '0'
published: true
password: ''
status: publish
categories:
- Powershell
- SQL Server
tags: []
# meta:
#   _edit_last: '111693662'
#   geo_public: '0'
#   publicize_linkedin_url: https://www.linkedin.com/updates?discuss=&scope=17000093&stype=M&topic=6260094027631198210&type=U&a=4LSP
#   _publicize_job_id: '4123599760'
#   _publicize_done_external: a:1:{s:7:"twitter";a:1:{i:15852853;s:52:"https://twitter.com/lumnah/status/854328336398647300";}}
#   _publicize_done_16023606: '1'
#   _wpas_done_15852853: '1'
#   publicize_twitter_user: lumnah
#   _publicize_done_16023607: '1'
#   _wpas_done_15852854: '1'
# author:
#   login: clumnah
#   email: clumnah@gmail.com
#   display_name: Chris Lumnah
#   first_name: Chris
#   last_name: Lumnah
permalink: "/2017/04/18/spring-cleaning/"
---
The birds are singing, the grass is able to be seen, the air is warm, the sun is shining, more importantly, baseball has started. I finally get to go enjoy some down time at America's most beloved ballpark. Fenway Park for the non-New Englanders. Yes, Spring is finally here and with Spring generally, comes some cleaning up. I have a VM lab that I run on my Surface Pro 4. Nothing crazy, just a bunch of VMs to test out different versions of SQL in different scenarios. Recently I had to build up a 3 node cluster to test with Windows and SQL Server 2016 Always On Availability Groups for a client.The tests required me to tear down and install SQL Server multiple times while I tested out a set of scripts to install and configure SQL, and later test out failover scenarios. More on that later though.

As I was going through my environment, I realized I created a new domain controller for my tests. This DC has a new name and domain name which is different from my other VMs. I quickly realized that this will cause me issues later with authentication. No worries. I will just boot up the VMs and then and join them to the new domain. Easy-peasy. Now let met go test out my SQL Servers.

DOH!!

I received a login failure with access is denied. Using Windows Authentication with my new domain and recently joined server is not working. Why?.....Oh right, my new user id does not have access to SQL Server itself. As I sit there smacking myself in the head, I am also thinking about the amount of time it will take me to rebuild those VMs. Then it hit me!!!

Powershell to the rescue!!!

More specifically, dbatools to the rescue.

Now if you have not heard of dbatools, you have to check this out. Head over to [dbatools.io](https://dbatools.io/) and look around on the site. This set of Powershell cmdlets, started out as a project by Chrissy Lemaire to make database migrations super simple. Now with a couple of cmdlets, she can migrate an entire server and sit back enjoying a beer. Now, with the help of a few MVPs and input from the SQL Community, this has evolved into a fairly comprehensive set of cmdlets to aid DBAs everywhere in their day-to-day tasks. I highly recommend checking this out.

One of the cmdlets in dbatools is a command called Reset-SQLAdmin. This is a super useful script. You point this at a SQL Server and it will either reset the sa account or add an account to your SQL Server with sysadmin access. It is meant for you to use as a tool when you have lost access to your SQL Server. This fits my needs perfectly. The command is as easy as running the below command.

Reset-SqlAdmin -SqlServer sqlserver\sqlexpress -Login ad\administrator

That is it. It will go through its process and after a few seconds, you will have access to your SQL Server. Want to see it in action? Check out the video on the [dbatools.io site](https://dbatools.io/functions/other/reset-sqladmin/). One thing to note, this process does require an outage as it will stop and start your SQL Services. **So do not use this in production.&nbsp;**

So back to my issue....I ran this against my 6 VMs, and voila, I am in. I had access to my old VMs with little to no effort what so ever. This saved me hours of time and effort. Now I am able to reorganize the VMs in my lab and clean up old users and database.

Powershell and dbatools making things easy-peasy!
