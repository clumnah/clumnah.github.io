---
layout: post
title: Enabling AlwaysOn for SSISDB
date: 2017-05-09 10:37:23.000000000 -04:00
type: post
parent_id: '0'
published: true
password: ''
status: publish
categories:
- SQL Server
tags: []
# meta:
#   _publicize_done_external: a:1:{s:7:"twitter";a:1:{i:15852853;s:52:"https://twitter.com/lumnah/status/861953271916834816";}}
#   _rest_api_published: '1'
#   _rest_api_client_id: "-1"
#   _publicize_job_id: '4864219113'
#   _publicize_done_16023606: '1'
#   _wpas_done_15852853: '1'
#   publicize_twitter_user: lumnah
#   publicize_linkedin_url: https://www.linkedin.com/updates?discuss=&scope=17000093&stype=M&topic=6267718965846372352&type=U&a=XBNn
#   _publicize_done_16023607: '1'
#   _wpas_done_15852854: '1'
# author:
#   login: clumnah
#   email: clumnah@gmail.com
#   display_name: Chris Lumnah
#   first_name: Chris
#   last_name: Lumnah
permalink: "/2017/05/09/enabling-alwayson-for-ssisdb/"
---
I am at client where I get to do some fun work. I get to work on a brand new environment that is dedicated to SQL Server. It has some pretty cool specs. Biggest thing is that they are going all 2016. Both the OS and SQL Server will be 2016. This has let to a lot of fun and interesting discussions. One of those being how do we place the SSIS Catalog into an availability group.

This led me to ponder the possibilities. When AlwaysOn first came out, this was not supported. Sure, you could find clever work-arounds for this, but at the end of the day, they were not sanctioned by Microsoft. This would then leave the client in a bit of a bind if they ran into issues. However, starting in SQL Server 2016, this is now supported. Microsoft has made it possible to officially place the SSISDB into an availability group.

Which group you put the SSISDB in, will be a design question that you will need to answer for yourself. In short, it should go into a group that makes sense. If all your databases are in one group, then placing this in DB into the same group makes sense. If you have multiple groups, you may need to make a choice that best suits your application needs.

In this example, I will assume you already have an availability group set up with a listener. In my lab, I have a very simple setup. my availability group is called Group1 and my listener is called Listener1.

This video shows the steps I took to add the SSISDB into an availability group. I even show publishing a SSIS package to the listener name instead of a SQL Server Name.

<figure class="video_container">
  <video controls="true" allowfullscreen="true" poster="">
    <source src="/assets/videos/2017/05/adding-ssisdb-into-an-availability-group_dvd.mp4" type="video/mp4">
  </video>
</figure>

In case you do not want to watch the video, below is the process I did to add the SSISDB into the availability group.

1. On the Primary Replica, open SQL Server Management Studio
2. Right click on Integration Services Catalog, click Create Catalog  
 ![img1](/assets/images/2017/05/img12.png)
3. On the Create Catalog Dialog,  
 ![img2](/assets/images/2017/05/img21.png)  
Check the box to enable automatic execution of Integration Services stored procedures at SQL Server startup. Then enter a strong password.
4. Open the Availability Group folder and navigate to the group you want to add the SSISDB to. Right click on Availability Database and click Add Database.  
 ![img3](/assets/images/2017/05/img31.png)
5. Due to the encryption that the SSISDB uses, you will need to provide the password you used when creating the catalog. Put that password into the password field, click refresh and then click the check box next to SSISDB. Click Next  
 ![img4](/assets/images/2017/05/img41.png)
6. On the Add Database to Availability Group dialog, ensure that Automatic Seeding is selected. Click Next.  
 ![img5](/assets/images/2017/05/img51.png)
7. You will be presented with a list of the secondary replicas, connect to each one listed. Then click Next  
 ![img6](/assets/images/2017/05/img61.png)
8. The information will be validated. As long as there are no failures, click Next.  
 ![img7](/assets/images/2017/05/img71.png)
9. On the summary dialog, click Finish. SSMS will add the database to the availability group  
 ![img8](/assets/images/2017/05/img81.png)
10. Click Close  
 ![img9](/assets/images/2017/05/img91.png)
11. Once the database has been added to the availability group, go to the Integration Services Catalog folder, right click and click Enable AlwaysOn Support. (you may need to refresh this folder before the option is available to you)  
 ![img10](/assets/images/2017/05/img101.png)
12. Click connect next to each secondary replica. Then click OK.  
 ![img11](/assets/images/2017/05/img111.png)

The SSISDB is now established in an availability group and will failover based on the group rules.

