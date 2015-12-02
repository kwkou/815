<properties linkid="" urlDisplayName="" pageTitle="Understanding Service Layers and Versions – Microsoft Azure Cloud" metaKeywords="Azure 云,技术文档,文档与资源,MySQL,数据库,性能,Azure MySQL, MySQL PaaS,Azure MySQL PaaS, Azure MySQL Service, Azure RDS,ASDB基准" description="Explains service tiers and the performance of different versions, and provides you with a detailed reference for choosing MySQL Database on Azure. Based on the ASDB benchmark, we have provided test data for different versions for your reference." metaCanonical="" services="MySQL" documentationCenter="Services" title="" authors="" solutions="" manager="" editor="" />

<tags ms.service="mysql" ms.date="" wacn.date="04/29/2015"/>

#Understanding Service Tiers and Versions

##ASDB (Azure SQL Database Benchmark) Test Result References 
ASDB (Azure SQL Database Benchmark) is a database benchmark designed by Microsoft to facilitate quantitative improvements in terms of translating cloud computing resource allocation into higher database performance. This benchmark includes virtually all basic operations that occur within the online transaction processing (OLTP) workload. While the design of the benchmark is based on cloud computing, the database schema and data population and transactions are representative of a wide range of basic elements commonly used in OLTP workloads.

##Features of ASDB 
The basic features of ASDB include:

- The database used for ASDB uses six tables. Two of these tables are fixed-size tables, namely meaning that the number of rows does not change. Three of these tables are scaling tables, such that the performance achieved by the database is proportional to the number of lines in the tables. The final table is a growing table, the size of which is subject to data insertions and deletions performed while the benchmark is running.

- The schema of the tables is composed of a mixture of data types, including integer, numeral, character, and date/time. It also includes primary and secondary keys, but does not include any foreign keys. For example, there are no referential integrity constraints between the tables.

- The maximum performance is proportional to the size of the database and the concurrent connections. For example, a database with 100 concurrent connections can achieve a maximum transaction rate of 100 TPS, because the benchmark application has pacing control to ensure that every connection on average completes a maximum of no more than one transaction per second. More concurrent connections and a larger database are required to facilitate a higher transaction (TPS) rate.

- ASDB includes nine different types of transaction. Each transaction type is focused on a specific set of system characteristics in the database engine and system hardware, and the different transaction types are focused on different areas. The transactions use four basic SQL DML operations (select, update, insert, and delete) to test the database engine and basic hardware resources such as CPU, memory, IO and networks.

For a more detailed explanation of ASDB, please refer to [Overview of the ASDB Benchmark](https://msdn.microsoft.com/zh-cn/library/azure/dn741327.aspx#Benchmark_summary).

##Benchmark test results from ASDB

The table below shows the level of stable performance that can be maintained by each version using the ASDB benchmark, as well as the corresponding number of concurrent connections. Each test was run for an hour or more under stable conditions.

<table border="1" cellspacing="0" cellpadding="0" width="477">
  <tr>
    <td width="47" valign="top"><p><strong>Version</strong><strong> </strong></p></td>
    <td width="195" valign="top"><p align="center"><strong>Transaction</strong><strong> </strong><strong>rate</strong><strong> (TPS)</strong></p></td>
    <td width="122" valign="top"><p align="center"><strong>Test database size</strong><strong> </strong></p></td>
    <td width="113" valign="top"><p align="center"><strong>Concurrent connections</strong><strong> </strong></p></td>
  </tr>
  <tr>
    <td width="47" valign="top"><p align="center"><strong>MS 1</strong></p></td>
    <td width="195" valign="top"><p align="center">7.5</p></td>
    <td width="122" valign="top"><p align="center">3GB</p></td>
    <td width="113" valign="top"><p align="center">10</p></td>
  </tr>
  <tr>
    <td width="47" valign="top"><p align="center"><strong>MS 2</strong></p></td>
    <td width="195" valign="top"><p align="center">15</p></td>
    <td width="122" valign="top"><p align="center">3GB</p></td>
    <td width="113" valign="top"><p align="center">17</p></td>
  </tr>
  <tr>
    <td width="47" valign="top"><p align="center"><strong>MS 3</strong></p></td>
    <td width="195" valign="top"><p align="center">40</p></td>
    <td width="122" valign="top"><p align="center">7.2GB</p></td>
    <td width="113" valign="top"><p align="center">43</p></td>
  </tr>
  <tr>
    <td width="47" valign="top"><p align="center"><strong>MS 4</strong></p></td>
    <td width="195" valign="top"><p align="center">120</p></td>
    <td width="122" valign="top"><p align="center">28GB</p></td>
    <td width="113" valign="top"><p align="center">140</p></td>
  </tr>
</table>

>[AZURE.NOTE] It is important to be aware that, like all benchmarks, ASDB can only provide representative and indicative results. The transaction rates achieved using the benchmark application will differ from those achieved using other applications. The benchmark includes a collection of different transaction types; these transaction types are run to target a schema that includes a range of table and data types. While the benchmark performs common basic operations that are shared by all OLTP workloads, it does not represent any specific type of database or application. The goal of this benchmark is to provide a reasonable reference for a database’s expected relative performance when increasing or decreasing performance levels. In reality, databases vary in terms of size and complexity, encounter different workload mixes, and respond in different ways. For example, an IO-intensive application might reach the IO threshold very quickly, while a memory-intensive application might rapidly reach the memory limit. There is no guarantee that any specific database will perform as indicated by the ASDB benchmark under increased load conditions.
