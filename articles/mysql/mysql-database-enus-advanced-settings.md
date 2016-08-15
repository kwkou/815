<properties linkid="" urlDisplayName="" pageTitle="Setting MySQL Database on Azure Server Parameters – Azure Cloud" metaKeywords="Azure Cloud, technical documentation, documents and resources, MySQL, database, parameters, customization, Azure MySQL, MySQL PaaS, Azure MySQL PaaS, Azure MySQL Service, Azure RDS" description="MySQL Database on Azure supports customizing some parameters to your own requirements. We will help you to understand the selectable ranges and intervals for different parameters." metaCanonical="" services="MySQL" documentationCenter="Services" title="" authors="" solutions="" manager="" editor="" />

<tags ms.service="mysql_en" ms.date="07/05/2016" wacn.date="07/05/2016" wacn.lang="en" />

> [AZURE.LANGUAGE]
- [中文](/documentation/articles/mysql-database-advanced-settings/)
- [English](/documentation/articles/mysql-database-enus-advanced-settings/)

#Setting MySQL Database on Azure server parameters

MySQL Database on Azure supports custom settings for some parameters. The following table lists these parameters, their default values, and their selectable ranges.

[Find out more about MySQL parameters](http://dev.mysql.com/doc/refman/5.5/en/server-system-variables.html).

<table width="100%" border="1" cellspacing="0" cellpadding="0">
  <th align="left"><strong>Parameter</strong>
    </td>
  <th align="left"><strong>Default value</strong>
    </td>
  <th align="left"><strong>Range</strong>
    </td>
  
  <tr>
    <td>Event_scheduler</td>
    <td>OFF</td>
    <td>[ON|OFF|DISABLED]</td>
  </tr>
  <tr>
    <td>div_precision_increment</td>
    <td>4</td>
    <td>[0-30]</td>
  </tr>
  <tr>
    <td>group_concat_max_len</td>
    <td>1024</td>
    <td>[4-16777216]</td>
  </tr>
  <tr>
    <td>Innodb_adaptive_hash_index</td>
    <td>ON</td>
    <td>[ON|OFF]</td>
  </tr>
  <tr>
    <td>innodb_lock_wait_timeout</td>
    <td>50</td>
    <td>[1-3600]</td>
  </tr>
  <tr>
    <td>interactive_timeout</td>
    <td>1800</td>
    <td>[10-1800]</td>
  </tr>
  <tr>
    <td>log_queries_not_using_indexes</td>
    <td>OFF</td>
    <td>[ON|OFF]</td>
  </tr>
  <tr>
    <td>Log_bin_trust_function_creators</td>
    <td>OFF</td>
    <td>[ON|OFF|FALSE]</td>
  </tr>
  <tr>
    <td>max_allowed_packet</td>
    <td>1048576</td>
    <td>[1024- 16777216]</td>
  </tr>
   <tr>
    <td>server-id</td>
    <td>Random Value</td>
    <td>[1000 – 4294967295]</td>
  </tr>
  <tr>
    <td>sql_mode</td>
    <td>Empty</td>
    <td>ALLOW_INVALID_DATES | ANSI_QUOTES
    | ERROR_FOR_DIVISION_BY_ZERO
    | HIGH_NOT_PRECEDENCE | IGNORE_SPACE 
    | NO_AUTO_CREATE_USER | NO_AUTO_VALUE_ON_ZERO 
    | NO_BACKSLASH_ESCAPES | NO_DIR_IN_CREATE
    | NO_ENGINE_SUBSTITUTION | NO_FIELD_OPTIONS
    | NO_KEY_OPTIONS | NO_TABLE_OPTIONS
    | NO_UNSIGNED_SUBTRACTION | NO_ZERO_DATE
    | NO_ZERO_IN_DATE | ONLY_FULL_GROUP_BY
    | PAD_CHAR_TO_FULL_LENGTH | PIPES_AS_CONCAT
    | REAL_AS_FLOAT | STRICT_ALL_TABLES
    | STRICT_TRANS_TABLES
    
<a href="http://dev.mysql.com/doc/refman/5.5/en/sql-mode.html">http://dev.mysql.com/doc/refman/5.5/en/sql-mode.html</a></td>
  </tr>
  <tr>
    <td >wait_timeout</td>
    <td>120</td>
    <td>[60-240] </td>
  </tr>
</table>
>[AZURE.NOTE]**In view of the limitations of Azure Traffic Manager, we have adjusted the default value for wait\_timeout to 120 seconds (s) and the selectable range to 60s to 240s, but this adjustment only works on instances created after October 2015. For earlier instances, please manually set the value of wait\_timeout to any number between 60s and 240s. We recommend 120s. **
<!--HONumber=81-->