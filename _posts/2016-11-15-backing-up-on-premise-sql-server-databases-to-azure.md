---
layout: post
title: Backing up on-premise SQL Server databases to Azure
date: 2016-11-15 09:00:00.000000000 -05:00
type: post
parent_id: '0'
published: true
password: ''
status: publish
categories:
- General
tags:
- Backup and Recovery
permalink: "/2016/11/15/backing-up-on-premise-sql-server-databases-to-azure/"
---
So SQL 2016 has been officially released in June and there are a boat load of new features. One of the first new features to review is the ability to backup your databases to Azure. Now, technically, this feature has been around since SQL 2012 SP1 CU2, but the main idea behind SQL Server 2016 is that its bigger, better, faster than previous versions. Going with that theme, 2016 has upgraded its functionality and the performance of this feature.

DBAs want backups. DBAs want as many backups as they can take and store. Every DBA knows that if you do not have backups, and not just backups, but good backups, when it comes time to restore and you can’t, it could be a resume generating event.

Having a limited amount of storage is a big challenge that DBAs face. The business and DBAs always want to store more backups than they can actually fit onto their storage arrays. Now, while some DBAs wear multiple hats, most are not storage admins. So when a disk failure happens, they pray to the storage deities that the SAN team can fix the issue with no data loss. Another big challenge that the DBA faces, is not having the backup on hand ready to restore when issues arise. While they are happy that the backups are stored at an offsite location, there is generally a good amount of time wasted while the backup files are brought in from the offsite location. This could be hours, with the business down waiting for the backup to arrive and then to start the process of a restore. This process could be repeated multiple times if the files brought back are not correct or bad. What is worse is I have seen some people use a safe deposit box to store their backups offsite. While this sounds good in theory, when disaster strikes in the middle of the night or on the weekend, you should be extending your outage by hours or days.

Now, Azure offers a interesting solution to this problem.

DBAs do not have to worry about storage limits. Azure offers you unlimited storage to store any and all backups in a secure, offsite location. Do not worry about the offsite storage limiting your access. Any file you write to Azure is available to you as soon as you finish writing the file. This means no more waiting for files to be brought in from your current offsite location. Another benefit to backing up to Azure is that you do not have to manage any hardware. When a failure happens Microsoft is already working to resolve the issue and replacing the drives. Better yet, your backups are automatically replicated to 2 other locations. Now your data sits on 3 different areas. This means that if a disk failure happens, your data is safe and still recoverable.

For an in depth tutorial on how to backup to Azure, see this [link](https://msdn.microsoft.com/library/dn466438.aspx). In this post will cover the first 3 lessons of the tutorial.

Let’s walk through what it takes to set this up and be able to use this functionality.

First, lets take a look at a simple backup statement for the WideWorldImporters database. This is a basic backup statement going to the D drive.

```sql
BACKUP DATABASE [WideWorldImporters]  
TO DISK = N'D:\WideWorldImporters-Full.bak'  
WITH NOFORMAT  
  , INIT  
  , NAME = N'WideWorldImporters-Full Database Backup'  
  , SKIP  
  , NOREWIND  
  , NOUNLOAD  
  , COMPRESSION  
  , STATS = 10  
GO  
```

Next, let’s create the necessary objects in Azure to write a backup. These are the basics that you need to be able to backup to Azure.

- Azure account. If you do not have one, you can open up a free trial [here](https://azure.microsoft.com/en-us/).
- Resource group is an organizational unit. This allows you to group various resources (VMs, storage, network, etc.) in Azure components into logical groups.
- Storage account is a secure account that gives you access to services in Azure Storage.
- Container is like a folder in a file system. It proves a useful way to assign security polices to groups of objects.

Once you have your Azure account established, sign into the Azure portal.

1. Go to the Resource groups section and click Add.  
![image](/assets/images/2016/11/image.png)  

2. Fill out the on screen form to create the storage account. In this case, i named my Resource group **Backup Vault,** you can name yours whatever you’d like. Select the appropriate subscription and resource group location. The resource group location, is the Azure Datacenter that the resource group will be created in.  
![image](/assets/images/2016/11/image_thumb1.png)
3. Once Azure tells you the group has been created you should see it in your Resource Groups list. If you do not, click refresh.  
![image](/assets/images/2016/11/image_thumb2.png)
4. Click on your newly created resource group  
![image](/assets/images/2016/11/image_thumb3.png)
5. Click Add. In the search field, type Storage Account. Select the result for storage, not VM extensions. Then click Create.  
![image](/assets/images/2016/11/image_thumb4.png)
6. Fill out the form
1. Give it a name. The name has to be unique and in all lower case letters.
2. Use the resource manager deployment model. This is the newer deployment model that Azure is using.
3. Use general purpose for account type. You need to choose General, as you will be writing files to this storage account. General is best suited for file storage.
4. Use standard for performance because this is for storing backup files. It will be rare access
5. Replication use Locally Redundant. Locally redundant will create 2 more copies of your data in the same data center but in different areas. If you choose Geo-Redundant, this will allow you place the additional copies of data in a separate data center hundreds of miles away. See this [link](https://azure.microsoft.com/en-us/documentation/articles/storage-redundancy/) for more information.
6. Select the resource group you created above
7. In the Location field, choose the location closest to you.
8. Click Create  
![image](/assets/images/2016/11/image_thumb5.png)
7. Now we need to create a container. In your new storage account, click on Blobs under services  
![image](/assets/images/2016/11/image_thumb6.png)
8. Click on Add Container
9. Give it a name and set the Access type to be Private
10. When the container is finished creating, select the container  
![image](/assets/images/2016/11/image_thumb7.png)
11. Click on Properties and then copy the contents of the URL field. You will use this later in Management Studio.
12. Go back to the storage account, and click on Shared access signature  
![image](/assets/images/2016/11/image_thumb8.png)
13. Unless, there is a need to change the above settings, do not change any and click Generate SAS
14. You will see a SAS Token, copy the contents of that field.
15. In Management Studio, you will want to modify the below code
```sql
CREATE CREDENTIAL [https://StorageAccountName.blob.core.windows.net/ContainerName]
WITH IDENTITY='Shared Access Signature'  
  , SECRET='SHARED ACCESS SIGNATURE FROM STORAGE ACCOUNT'
GO 
```

Replace [[https://StorageAccountName.blob.core.windows.net/ContainerName]](https://StorageAccountName.blob.core.windows.net/ContainerName]) with the URL from step 11

Replace 'SHARED ACCESS SIGNATURE FROM STORAGE ACCOUNT' with the SAS token created in step 14

Remove the first character from the SAS token which should be a ?

1. 
  1. Execute the modified code.
  2. Finally, lets create a backup of our database to the new location using the below sample code

[code language="sql"]  
BACKUP DATABASE [WideWorldImporters]  
TO DISK = '&lt;a href="https://clbackupstorage.blob.core.windows.net/crlbackuplocation/WideWorldImporters-Full.bak'"&gt;https://clbackupstorage.blob.core.windows.net/crlbackuplocation/WideWorldImporters-Full.bak'&lt;/a&gt;  
WITH NOFORMAT  
  , INIT  
  , NAME = N'WideWorldImporters-Full Database Backup'  
  , SKIP  
&nbsp;&nbsp;&nbsp; , NOREWIND  
&nbsp;&nbsp;&nbsp; , NOUNLOAD  
&nbsp;&nbsp;&nbsp; , COMPRESSION  
&nbsp;&nbsp;&nbsp; , STATS = 10  
GO  
[/code]

As far as pricing details, to learn more about the costs of writing your backups to Azure storage, I encourage you to utilize the Azure pricing calculator to better understand what the costs are. You can find that link [here](https://azure.microsoft.com/en-us/pricing/calculator/?service=storage).

While there is a lot to cover in doing a backup of a database to an Azure blob location, I hope the above gives you the steps to follow to create the objects you need to be able to write your database backups to Azure.

