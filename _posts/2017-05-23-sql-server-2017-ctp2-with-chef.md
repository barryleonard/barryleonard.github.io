---
layout: post
title:  "SQL Server 2017 CTP 2.0 with Chef"
date:   2017-05-23
description: Microsoft SQL Server 2017 CTP 2.0 automation with Chef
categories:
- Devops
permalink: 2017-05-23-sql-server-2017-ctp2-automation-with-chef
---

Quick post to demo the ease we can adopt new versions of Microsoft SQL server using Chef

---

### Introduction ###

In my previous post I explained how we're using automation to provision SQL servers to company standard.

In this post I want to explain how we adopt new versions of SQL server in a matter of hours

### SQL 2017 CTP 2.0 ###

SQL 2017 will be the next version from Microsoft and they're making builds available to test at a fast rate. We wanted to make sure we were ready for any team in the business that wanted to explore the benefits of adopting this new version.

### Adding support ###

First we add a new suite to the local test kitchen.yaml file:

    name: sqlserver2017-ctp2-sa
    run_list:
      - recipe[cookbook-fourth-sqlserver::service_accounts]
      - recipe[cookbook-fourth-sqlserver::service_rights]
      - recipe[cookbook-fourth-sqlserver::server]
      - recipe[cookbook-fourth-sqlserver::ssms_package]
    attributes:
      cookbook-fourth-sqlserver:
        version: '2017-CTP2'
        accept_eula: true
        update_enabled: false
        instance_name: 'MSSQLSERVER'
        feature_list: 'SQLENGINE,DQC,CONN,BC,SDK,BOL,SNAC_SDK'
        sql_collation: 'Latin1_General_CI_AS'
        sql_account: '.\SQL_SERVICE_USER'
        agent_account: '.\SQLAGENTUSER'
        agent_startup: 'automatic'
        sysadmins: 'vagrant'
        ssms:
          version: 'SSMS-Setup-ENU.17.0.RC3' 

We repeat the above for kitchen.cloud.yaml

Next we add the Microsoft SQL Server and Management Studio versions to our version helper libraries:

**SQL Server Installer:**

        '2017-CTP2' => {
          reg_version_string: 'MSSQL14.',
          install_dir_version: '140',
          sqlserver_url_x86_64: 'https://<our_file_server>2017_CTP2/SQLServerVnextCTP2.0-x64-ENU.iso',
          sqlserver_package_name_x86_64: 'Microsoft SQL Server vNext CTP2.0 (64-bit)',
          sqlserver_checksum_x86_64: 'DD4B0F6246E9A58525716E2B2F3964C57C794E83CB002AA4EF61AAF6390E68EF'
        }

**Management Studio:**

        'SSMS-Setup-ENU.17.0.RC3' => {
          ssms_installer_url_x86_64: 'https://<our_file_server>/SQL_Server/Tools/SSMS-Setup-ENU.17.0.RC3.exe',
          ssms_installer_checksum_x86_64: '19D8A132906CC8F36A158704272D521114523B51935AB66AB1A03163682A2FF4'
        }

That's it. Now we can install Microsoft SQL Server 2017 CTP 2.0 and Management Studio by passing in the following attributes to the environment in Chef: 

       cookbook-fourth-sqlserver:
        version: '2017-CTP2'
        ssms:
          version: 'SSMS-Setup-ENU.17.0.RC3'

The added bonus is that we can use all our existing cookbook unit and integration tests as well. 

The next challenge will be to add container support to our SQL server cookbook now that SQL supports it.