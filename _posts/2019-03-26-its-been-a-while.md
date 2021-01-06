---
layout: post
title: It's been a while...
date: 2019-03-26 09:00:54.000000000 -04:00
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
#   _edit_last: '111693662'
#   geo_public: '0'
#   _rest_api_published: '1'
#   _rest_api_client_id: "-1"
#   _publicize_done_external: a:1:{s:7:"twitter";a:1:{i:15852853;s:53:"https://twitter.com/lumnah/status/1110536032343281664";}}
#   timeline_notification: '1553607396'
#   _publicize_job_id: '29059955598'
#   _publicize_done_16023606: '1'
#   _wpas_done_15852853: '1'
#   publicize_twitter_user: lumnah
#   publicize_linkedin_url: ''
#   _publicize_done_21637047: '1'
#   _wpas_done_15852854: '1'
#   _g_feedback_shortcode_d6da1b1551a87d83bad768445f770fe7e72f1469: "[contact-field
#     label='Name' type='name' required='1'/][contact-field label='Email' type='email'
#     required='1'/][contact-field label='Website' type='url'/][contact-field label='Comment'
#     type='textarea' required='1'/]"
#   _g_feedback_shortcode_atts_d6da1b1551a87d83bad768445f770fe7e72f1469: a:10:{s:2:"to";s:17:"clumnah@gmail.com";s:7:"subject";s:58:"[Chris
#     Lumnah&#039;s SQL Server Blog] It's been a while...";s:12:"show_subject";s:2:"no";s:6:"widget";i:0;s:2:"id";i:527;s:18:"submit_button_text";s:6:"Submit";s:14:"customThankyou";s:0:"";s:21:"customThankyouMessage";s:30:"Thank
#     you for your submission!";s:22:"customThankyouRedirect";s:0:"";s:10:"jetpackCRM";b:1;}
# author:
#   login: clumnah
#   email: clumnah@gmail.com
#   display_name: Chris Lumnah
#   first_name: Chris
#   last_name: Lumnah
permalink: "/2019/03/26/its-been-a-while/"
excerpt: Configure Kerberos on Mac
---
So I know I have blogged in a while and that is bad on my part. I have been very busy with the new job that blogging just doesn't make the list. It has also been a struggle figuring out what I can share and what I should share. Well, I finally have found something I can share that has been plaguing me for a while and I think will be beneficial to share.

I run a Mac to do my daily work. I will run VMs or RDP sessions to Windows boxes so I can do SQL work. I have been struggling with trying to be more efficient and not having to use Windows if I do not have to. Plus, if I can stay on the Mac side, this will aid me in my current goal to learn Linux, containers, and other database platforms. To be able to use my Mac to do more SQL Server related work, I need to be able to connect to various different SQL Servers to run queries. I have an environment where I will connect to a development domain for most of my work and a production domain for things that I do that are customer facing. I also want to be able to use Windows Authentication to connect to these various SQL Servers. This has proven to be difficult on my Mac as I do not have my Mac joined to a domain.

I have heard of other members of the community doing this, but I kept struggling. I had read this blog post several times, but nothing worked.

Then it dawned on me and I realized my error. To use Windows Authentication, you must have Kerberos working. This was the missing link. To have Kerberos working you have to have a kerberos ticket and the server you want to connect to must have a Service Principal Name (SPN) created for it.

First, I made sure to run the Microsoft Kerberos Configuration Manager for SQL Server. You can download that [here](https://www.microsoft.com/en-us/download/details.aspx?id=39046). This should be run from a Windows box that is joined to the domain. When you run this against one of your SQL Servers, it will scan your Active Directory to see if an SPN exists for the SQL Server. If there is an issue, you will see the below

![](/assets/images/2019/03/screen-shot-2019-03-20-at-8.35.34-pm.png)

Under the status column, you can see that it says Missing. That means that an SPN is not created for this SQL Server. The Kerberos Configuration Manager will provide you a script to fix this for you. You will want to provide this script to your Domain Admin so they can run it and correct the issue. Once this has been run and an SPN has been created, you can use Kerberos for Windows Authentication. If you don't have an SPN, Windows will fall back to NTLM, but in the Mac world, you cannot fall back to that and thus, cannot use Windows Authentication.

Second, after the SPN is in place, open up a terminal and run
```
kinit username@DOMAIN.COM
```

username will be what you currently use to log in to your Active Directory Domain, while domain.com should be the domain name, but it must be in **ALL CAPS.**

You will get asked to authenticate by inputting your password. If everything is correct, you can run the next statement of
```
klist username@DOMAIN.COM
```

You should get similar output below  
```
Credentials cache: API:BA74140D-7B6E-49B5-93E1-EA069AC87B20
Principal: chris.lumnah@RANGERS.LAB`

Issued Expires Principal  
Mar 20 20:05:58 2019 Mar 21 06:05:52 2019 krbtgt/RANGERS.LAB@RANGERS.LAB

Now you can use Azure Data Studio to connect to your SQL Server on Windows using Windows Authentication.
```
