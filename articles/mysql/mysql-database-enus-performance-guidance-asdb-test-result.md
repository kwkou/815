<properties linkid="" urlDisplayName="" pageTitle="Understanding Service Layers and Versions – Azure Cloud" metakeywords="Azure Cloud, technical documentation, documents and resources, MySQL, database, performance, Azure MySQL, MySQL PaaS, Azure MySQL PaaS, Azure MySQL Service, Azure RDS, ASDB benchmark" description="Explains service tiers and the performance of different versions, and provides you with a detailed reference for choosing MySQL Database on Azure. Based on the Azure SQL Database Benchmark, we have provided test data for different versions for your reference." metaCanonical="" services="MySQL" documentationCenter="Services" title="" authors="" solutions="" manager="" editor="" />

<tags ms.service="mysql_en" ms.date="07/05/2016" wacn.date="07/05/2016" wacn.lang="en" />

> [AZURE.LANGUAGE]
- [中文](/documentation/articles/mysql-database-performance-guidance-asdb-test-result/)
- [English](/documentation/articles/mysql-database-enus-performance-guidance-asdb-test-result/)

#Understanding Service Tiers and Versions

##Azure SQL Database Benchmark test result references 
Azure SQL Database Benchmark is a database benchmark designed by Microsoft to facilitate quantitative improvements for translating cloud computing resource allocation into higher database performance. This benchmark includes virtually all basic operations that occur within the online transaction processing (OLTP) workload. While the design of the benchmark is based on cloud computing, the database schema and data population and transactions are representative of a wide range of basic elements commonly used in OLTP workloads.

##Features of Azure SQL Database Benchmark 
The basic features of Azure SQL Database Benchmark include:

- The database used for Azure SQL Database Benchmark uses six tables. Two of these tables are fixed-size tables, meaning that the number of rows does not change. Three of these tables are scaling tables, such that the performance achieved by the database is proportional to the number of lines in the tables. The final table is a growing table, the size of which is subject to data insertions and deletions performed while the benchmark is running.

- The schema of the tables is composed of a mixture of data types, including integer, numeral, character, and date/time. It also includes primary and secondary keys, but does not include any foreign keys. For example, there are no referential integrity constraints among the tables.

- The maximum performance is proportional to the size of the database and the concurrent connections. For example, a database with 100 concurrent connections can achieve a maximum transaction rate of 100 transactions per second (TPS), because the benchmark application has pacing control to ensure that every connection on average completes no more than a maximum of one transaction per second. More concurrent connections and a larger database are required to facilitate a higher TPS rate.

- Azure SQL Database Benchmark includes nine different types of transaction. Each transaction type is focused on a specific set of system characteristics in the database engine and system hardware, and the different transaction types are focused on different areas. The transactions use four basic SQL data manipulation language (DML) operations (select, update, insert, and delete) to test the database engine and basic hardware resources such as CPU, memory, I/O, and networks.

For a more detailed explanation of Azure SQL Database Benchmark, refer to [Overview of the Azure SQL Database Benchmark](https://msdn.microsoft.com/zh-cn/library/azure/dn741327.aspx#Benchmark_summary).

##Benchmark test results from Azure SQL Database Benchmark

The table below shows the level of stable performance that can be maintained by each version that uses the Azure SQL Database Benchmark, as well as the corresponding number of concurrent connections. Each test was run for an hour or more under stable conditions.

<table border="1" cellspacing="0" cellpadding="0" width="477">
  <tr>
    <td width="47" valign="top"><p><strong>Version</strong><strong> </strong></p></td>
    <td width="195" valign="top"><p align="center"><strong>Transaction</strong><strong> </strong><strong>rate</strong><strong> (TPS)</strong></p></td>
    <td width="122" valign="top"><p align="center"><strong>Test database size</strong><strong> </strong></p></td>
    <td width="113" valign="top"><p align="center"><strong>Concurrent connections</strong><strong> </strong></p></td>
  </tr>
  <tr>
    <td width="47" valign="top"><p align="center"><strong>MS 1</strong></p></td>
    <td width="195" valign="top"><p align="center">10</p></td>
    <td width="122" valign="top"><p align="center">1.7GB</p></td>
    <td width="113" valign="top"><p align="center">12</p></td>
  </tr>
  <tr>
    <td width="47" valign="top"><p align="center"><strong>MS 2</strong></p></td>
    <td width="195" valign="top"><p align="center">20</p></td>
    <td width="122" valign="top"><p align="center">3.0GB</p></td>
    <td width="113" valign="top"><p align="center">22</p></td>
  </tr>
  <tr>
    <td width="47" valign="top"><p align="center"><strong>MS 3</strong></p></td>
    <td width="195" valign="top"><p align="center">60</p></td>
    <td width="122" valign="top"><p align="center">8.9GB</p></td>
    <td width="113" valign="top"><p align="center">65</p></td>
  </tr>
  <tr>
    <td width="47" valign="top"><p align="center"><strong>MS 4</strong></p></td>
    <td width="195" valign="top"><p align="center">200</p></td>
    <td width="122" valign="top"><p align="center">28.6GB</p></td>
    <td width="113" valign="top"><p align="center">210</p></td>
  </tr>
  <tr>
    <td width="47" valign="top"><p align="center"><strong>MS 5</strong></p></td>
    <td width="195" valign="top"><p align="center">300</p></td>
    <td width="122" valign="top"><p align="center">43.6GB</p></td>
    <td width="113" valign="top"><p align="center">320</p></td>
  </tr>
  <tr>
    <td width="47" valign="top"><p align="center"><strong>MS 6</strong></p></td>
    <td width="195" valign="top"><p align="center">450</p></td>
    <td width="122" valign="top"><p align="center">73.6GB</p></td>
    <td width="113" valign="top"><p align="center">470</p></td>
  </tr>
</table>

>[AZURE.NOTE]It is important to be aware that, like all benchmarks, Azure SQL Database Benchmark can provide only representative and indicative results. The transaction rates achieved by using the benchmark application will differ from those achieved by using other applications. The benchmark includes a collection of different transaction type. These transaction types are run to target a schema that includes a range of table and data types. While the benchmark performs common basic operations that are shared by all OLTP workloads, it does not represent any specific type of database or application. The goal of this benchmark is to provide a reasonable reference for a database’s expected relative performance when increasing or decreasing performance levels. In reality, databases vary in terms of size and complexity, encounter different workload mixes, and respond in different ways. For example, an I/O-intensive application might reach the I/O threshold very quickly, while a memory-intensive application might rapidly reach the memory limit. There is no guarantee that any specific database will perform as indicated by the Azure SQL Database Benchmark under increased load conditions.**

<!--HONumber=81-->