<properties linkid="" urlDisplayName="" pageTitle="定制MySQL 数据库 on Azure服务器参数 - Azure 微软云" metaKeywords="Azure 云,技术文档,文档与资源,MySQL,数据库,参数,定制,Azure MySQL, MySQL PaaS,Azure MySQL PaaS, Azure MySQL Service, Azure RDS" description="MySQL 数据库 on Azure支持您根据需求定制化服务器部分参数,帮您了解不同参数的设置范围和区间。" metaCanonical="" services="MySQL" documentationCenter="Services" title="" authors="" solutions="" manager="" editor="" />

<tags ms.service="mysql" ms.date="07/05/2016" wacn.date="07/05/2016" wacn.lang="cn" />

> [AZURE.LANGUAGE]
- [中文](/documentation/articles/mysql-database-advanced-settings/)
- [English](/documentation/articles/mysql-database-enus-advanced-settings/)

#定制MySQL 数据库 on Azure服务器参数

MySQL 数据库 on Azure支持您对服务器部分参数进行自定义设置，下表中列出可配置的参数，默认值，以及可选范围。

[了解更多MySQL参数信息](http://dev.mysql.com/doc/refman/5.5/en/server-system-variables.html)。

<table width="100%" border="1" cellspacing="0" cellpadding="0">
  <th align="left"><strong>参数</strong>
    </td>
  <th align="left"><strong>默认值</strong>
    </td>
  <th align="left"><strong>范围</strong>
    </td>
  
  <tr>
    <td>Event_scheduler</td>
    <td>OFF</td>
    <td>ON|OFF|DISABLED</td>
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
    <td>ON|OFF</td>
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
    <td>[1000 - 4294967295]</td>
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
  <tr>
    <td >time_zone</td>
    <td>System</td>
    <td>System, [-12:00 to +12:00] </td>
  </tr>
</table>
>[AZURE.NOTE] **考虑到流量管理器的限制，我们将wait_timeout的默认值调整为120s，可选范围为60-240s，但上述调整只对10月后创建的实例生效。对于以前的实例，请您手动将wait_timeout值设置为60-240s之间的任意数值，推荐120s。**

>[AZURE.NOTE] **关于时区的配置，可详细参考[MySQL on Azure上的时区配置](/documentation/articles/mysql-database-timezone-config/).**