---
layout: post
title: DSC Install of SQL Server
date: 2017-03-07 09:00:29.000000000 -05:00
type: post
parent_id: '0'
published: true
password: ''
status: publish
categories:
- General
tags: []
# meta:
#   _rest_api_published: '1'
#   _rest_api_client_id: "-1"
#   publicize_linkedin_url: https://www.linkedin.com/updates?discuss=&scope=17000093&stype=M&topic=6244901916657086464&type=U&a=oEPF
#   _publicize_job_id: '2602235521'
#   _publicize_done_external: a:1:{s:7:"twitter";a:1:{i:15852853;s:52:"https://twitter.com/lumnah/status/839136224627556352";}}
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
permalink: "/2017/03/07/dsc-install-of-sql-server/"
---
It has been a while since I have made a post and figured it was long over due. I figured for my first post in a while, it would be about something I have been working on lately. The automation of installing and configuring of SQL Server.

So the installation of SQL Server is now fairly straightforward. The wizard does a nice job of guiding you along the way. 2016 even includes best practice suggestions for tempdb and instance file initialization. Along the way, Microsoft as given us ways to automate the installation of SQL Server. You can sysprep an instance, but this does not really automate the installation. It just helps create a template of an instance. At the end of the day, you still need to do things manually. You can also use a configuration file to assist here. This is a great step forward, but it does not allow for all of the things you need to do to configure a SQL server.

Powershell does. Desired State Configuration (DSC) is functionality built into Powershell that allows for the installation and configuration of a SQL Server.

First, to get started, you need to have a base understanding of DSC. Without this, the rest of this post will be hard to follow, as DSC can be difficult to follow. The link below will take you to the videos on the Microsoft Virtual Academy site that Jeffrey Snover put together that explains how DSC works.

[Getting Started with Powershell Desired State Configuation (DSC)](https://mva.microsoft.com/en-us/training-courses/getting-started-with-powershell-desired-state-configuration-dsc--8672?l=ZwHuclG1_2504984382)

Also, the modules you will need to make DSC work in your environment can all be downloaded from the [Powershell Gallery](http://www.powershellgallery.com/). I have also found that searching for the module on GitHub, has returned back better documentation on how each module works.This has been great in my learning of how to get DSC going.

For the example what I have put together, we will use a Push mode for our DSC script.

The snippet that is below, uses a Powershell section called Configuration. This is similar to a Powershell Function in construction and how it works.

```
Configuration SQLServerInstall{  
 param(  
    [string]$ComputerName,  
    [int32]$MAXDOP,  
    [int32]$MAXMemory  
 )  
 Import-DscResource –Module PSDesiredStateConfiguration  
 Import-DscResource -Module xSQLServer  
 Import-DscResource -Module SecurityPolicyDsc  
 Import-DscResource -Module xPendingReboot

  Node $ComputerName{  
    WindowsFeature NET-Framework-Core{  
      Name = "NET-Framework-Core"  
      Ensure = "Present"  
      IncludeAllSubFeature = $true
    }  
    xSqlServerSetup InstallSQL{  
      DependsOn = '[WindowsFeature]NET-Framework-Core'  
      Features = $Configuration.InstallSQL.Features  
      InstanceName = $Configuration.InstallSQL.InstanceName  
      SQLCollation = $Configuration.InstallSQL.SQLCollation  
      SQLSysAdminAccounts = $Configuration.InstallSQL.SQLSysAdminAccounts  
      InstallSQLDataDir = $Configuration.InstallSQL.InstallSQLDataDir  
      SQLUserDBDir = $Configuration.InstallSQL.SQLUserDBDir  
      SQLUserDBLogDir = $Configuration.InstallSQL.SQLUserDBLogDir  
      SQLTempDBDir = $Configuration.InstallSQL.SQLTempDBDir  
      SQLTempDBLogDir = $Configuration.InstallSQL.SQLTempDBLogDir  
      SQLBackupDir = $Configuration.InstallSQL.SQLBackupDir

      SourcePath = $Configuration.InstallSQL.SourcePath  
      SetupCredential = $Node.InstallerServiceAccount  
    }  
    xSQLServerNetwork ConfigureSQLNetwork{  
      DependsOn = "[xSqlServerSetup]InstallSQL"  
      InstanceName = $Configuration.InstallSQL.InstanceName  
      ProtocolName = "tcp"  
      IsEnabled = $true  
      TCPPort = 1433  
      RestartService = $true  
    }  
    xSQLServerConfiguration DisableRemoveAccess{  
      SQLServer = $ComputerName  
      SQLInstanceName = $Configuration.InstallSQL.InstanceName  
      DependsOn = "[xSqlServerSetup]InstallSQL"  
      OptionName = "Remote access"  
      OptionValue = 0  
    }  
    UserRightsAssignment PerformVolumeMaintenanceTasks{  
      Policy = "Perform\_volume\_maintenance\_tasks"  
      Identity = "Builtin\Administrators"  
    }  
    UserRightsAssignment LockPagesInMemory{  
      Policy = "Lock\_pages\_in\_memory"  
      Identity = "Builtin\Administrators"  
    }  
    xPendingReboot PendingReboot{  
      Name = $ComputerName  
    }  
    LocalConfigurationManager{  
      RebootNodeIfNeeded = $True  
    }  
    xSQLServerAlwaysOnService EnableAlwaysOn{  
      SQLServer = $ComputerName  
      SQLInstanceName = $Configuration.InstallSQL.InstanceName  
      DependsOn = "[xSqlServerSetup]InstallSQL"  
      Ensure = "Present"  
    }  
    xSQLServerMaxDop SetMAXDOP{  
      SQLInstanceName = $Configuration.InstallSQL.InstanceName  
      DependsOn = "[xSqlServerSetup]InstallSQL"  
      MaxDop = $MAXDOP  
    }  
    xSQLServerMemory SetMAXDOP{  
      SQLInstanceName = $Configuration.InstallSQL.InstanceName  
      DependsOn = "[xSqlServerSetup]InstallSQL"  
      MaxMemory = $MAXMemory  
      DynamicAlloc = $False  
    }  
  }  
}  
```

To break down the script and the areas in the Configuration function, the names are thankfully fairly self explanatory.  
- WindowsFeature - Ensures that Windows features are present. This is a function that will initiate the installation if needed  
- xSqlServerSetup - This section will install SQL Server. If you look at the properties it is asking for, it will mirroring most of the properties that are in a configuration file  
- xSQLServerNetwork - This section is used to enable protocols, like TCP and configure what port SQL Server should use.  
- xSQLServerConfiguration - Is used to set values that you would normally set via sp_configure  
- UserRightsAssignment - Will set values in the Local Security Policy. In this case, I am setting who can Lock Pages in Memory and Perform Volume Maintenance Tasks  
- xPendingReboot - Will check to see if a reboot is required and reboot if needed  
- xSQLServerAlwaysOnService - Used to enable the Always On Service  
- xSQLServerMaxDop - Used to set MAXDOP  
- xSQLServerMemory - Used to set MIN and MAX memory values

Any place you see $Configuration.InstallSQL.* this is a value from a JSON file. I use this a parameter file that will feed this script. This allows me to easily reuse this script as many times as I need.

When the Powershell Script runs, the DSC will output a text file with an extension of MOF. DSC will then push this configuration file to the server and then proceed to configure the server as instructed. Then I use scripts that I have written or found along the years to finish the configuration. Ideally, using Powershell, I can configure the SQL Server any way I would like.


I have posted my entire set of scripts for installing and configuring SQL Server on Githib. You can find the scripts [here](https://github.com/clumnah/Configure-SQL-Server-with-DSC). Here you can see the full example. Enjoy.

