<properties
	pageTitle="SLA for SQL Server Stretch Database"
    description="We guarantee at least 99.9% of the time customers will have connectivity between their SQL Server Stretch Database and our Internet gateway."
    services=""
    documentationCenter=""
    authors=""
    manager=""
    editor=""
    tags=""/>

<tags ms.service="legal" ms.date="07/2016" wacn.date="07/2016" wacn.lang="en"/>

> [AZURE.LANGUAGE]
- [中文](/support/legal/sql-server-stretch-database/)
- [English](/support/legal/sql-server-stretch-database-en/)

# SLA for SQL Server Stretch Database

We guarantee at least 99.9% of the time customers will have connectivity between their SQL Server Stretch Database and our Internet gateway.

## SLA details 
Additional Definitions

**"Database"** means one instance of SQL Server Stretch database.

Monthly Uptime Calculation and Service Levels for SQL Server Stretch Database Service

**"Maximum Available Minutes"** is the total number of minutes that a given Database has been deployed in Azure for a given Azure subscription during a billing month.

**"Downtime"** is the total accumulated minutes across all Databases deployed by Customer in a given Azure subscription during which the Database is unavailable. A minute is considered unavailable for a given Database if all continuous attempts by Customer to establish a connection to the Database within the minute fail.

**"Monthly Uptime Percentage"** for the SQL Server Stretch Database Service is calculated as Maximum Available Minutes less Downtime divided by Maximum Available Minutes in a billing month for a given Azure subscription. Monthly Uptime Percentage is represented by the following formula:

    Monthly Uptime % = (Maximum Available Minutes-Downtime) / Maximum Available Minutes

The following Service Levels and Service Credits are applicable to Customer’s use of the Azure SQL Stretch Database Service:

MONTHLY UPTIME PERCENTAGE	|SERVICE CREDIT
---|---
< 99.9%	|10%
< 99%	|25%
