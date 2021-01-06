---
layout: post
title: Automatic Seeding Error
date: 2017-05-16 09:00:24.000000000 -04:00
type: post
parent_id: '0'
published: true
password: ''
status: publish
categories:
- SQL Server
tags:
- AlwaysOn
# meta:
#   _rest_api_published: '1'
#   _rest_api_client_id: "-1"
#   _publicize_done_external: a:1:{s:7:"twitter";a:1:{i:15852853;s:52:"https://twitter.com/lumnah/status/864470154943623168";}}
#   _publicize_job_id: '5106965305'
#   _publicize_done_16023606: '1'
#   _wpas_done_15852853: '1'
#   publicize_twitter_user: lumnah
#   publicize_linkedin_url: https://www.linkedin.com/updates?discuss=&scope=17000093&stype=M&topic=6270235847417696256&type=U&a=DGAL
#   _publicize_done_16023607: '1'
#   _wpas_done_15852854: '1'
#   _g_feedback_shortcode: "[contact-field label='Name' type='name' required='1'/][contact-field
#     label='Email' type='email' required='1'/][contact-field label='Website' type='url'/][contact-field
#     label='Comment' type='textarea' required='1'/]"
# author:
#   login: clumnah
#   email: clumnah@gmail.com
#   display_name: Chris Lumnah
#   first_name: Chris
#   last_name: Lumnah
permalink: "/2017/05/16/automatic-seeding-error/"
---
In my lab, I decided to play around with the automatic seeding functionality that is part of Availability Groups. This was sparked by my last post about putting SSISDB into an AG. I wanted to see how it would work for a regular database. When I attempted to do so, I received the following error:

![img1](/assets/2017/05/automatic-seeding-error.png)

<span style="color:#ff0000;">Cannot alter the availability group 'Group1', because it does not exist or you do not have permission. (Microsoft SQL Server, Error: 15151)</span></p>
This seemed odd to me. What would cause this error? What changed in my environment?
First, I realized that I did break down my AG and recreate it in management studio. When I created the group, I did not put any databases into the group, so I cold test automatic seeding.

Next, I decided to go through the process and instead of using the GUI, I scripted out the commands and would run manually. This way I could pinpoint where the process was failing.
```
---
YOU MUST EXECUTE THE FOLLOWING SCRIPT IN SQLCMD MODE.  
:Connect sql2016n1

USE [master]

GO

ALTER AVAILABILITY GROUP [Group1]  
MODIFY REPLICA ON N'SQL2016N2' WITH (SEEDING\_MODE = AUTOMATIC)

GO

USE [master]

GO

ALTER AVAILABILITY GROUP [Group1]  
ADD DATABASE [WWI\_Preparation];

GO

:Connect sql2016n2

ALTER AVAILABILITY GROUP [Group1] GRANT CREATE ANY DATABASE;

GO  
```

When I ran the above code, the steps that were to run on SQL2016N1, succeeded, but those that were to run on SQL2016N2, failed. Additionally, I received the below message:

Msg 15151, Level 16, State 1, Line 24  
Cannot alter the availability group 'Group1', because it does not exist or you do not have permission.

I went and looked at the AlwaysOn High Availability folder in SSMS on SQL2016n2 and found nothing.

![2017-05-15_17-07-58](/assets/images/2017/05/2017-05-15_17-07-58.png)

So, the group cannot be given permissions to create any database, because the group did not exist. Then it dawned on me as to why.

Creating an availability group, without databases using the GUI, only creates the group on the primary. It does not reach out to the secondary replicas to join those nodes to the group. This must be done manually. So to resolve the error, I had to run this code on SQL2016N2:

```
USE [master]  
GO  
ALTER AVAILABILITY GROUP Group1 JOIN  
GO
```

This allowed the secondary replica to join the AG that was established on SQL2016N1. Then I was able to proceed with the next step of running

```
USE [master]  
GO  
ALTER AVAILABILITY GROUP [Group1] GRANT CREATE ANY DATABASE;  
GO
```

Once this was completed, the database started seeding from one node to the other.

I think the biggest takeaway from this is the importance of using a script to do DBA actions instead of the GUI. It allows for a more focused and deliberate execution, where you can see things happen in order. This, in turn, teaches you and allows for greater flexibility, when things go bump.
