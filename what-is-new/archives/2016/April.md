<properties
	pageTitle="历史公告 2016年4月 - Azure"
    description="历史公告 2016年4月"
    services=""
    documentationCenter=""
    authors=""
    manager=""
    editor=""
    tags=""/>

<tags ms.service="what-is-new_archives" ms.date="" wacn.date="" wacn.lang="cn"/>

# Azure 公告
## 2016年4月时事通讯

### 公告
### 2016年4月25日 互联网信息服务单位网络安全责任告知书 
为了保护网络信息安全，保障公民、法人和其他组织的合法权益，维护国家安全和社会公共利益，根据《中华人民共和国计算机信息系统安全保护条例》、《计算机信息网络国际联网安全保护管理办法》等相关法律法规规定，请您尽快登录全国公安机关互联网站安全管理服务平台 http://www.beian.gov.cn 提交公安备案申请。自通知之日起，超过 30 日未办理的，公安局将依法暂停接入。详细信息请访问[互联网信息服务单位网络安全责任告知书](/support/network-security-announcement)页面。

### 2016年4月16日 关于 Azure 云服务异常的通告 
事件总结：北京时间2016/4/16 11：45到15:10，使用中国东部和中国北部服务的用户在打开管理门户网站( https://manage.windowsazure.cn )时可能会遇到问题。用户也可能无法连接这些区域内的虚拟机， Redis 缓存以及媒体服务。使用流分析服务的用户可能会看到正在创建的工作流和已存在的工作流停止执行。使用 SQL 数据库的用户可能无法创建，删除或者导入导出数据库。使用 Azure 活动目录的用户可能无法执行服务管理操作。使用服务总线的用户可能看不到日志，并且不能读取服务总线资源。初步调查原因：由于负载均衡的程序问题，计算集群的负载均衡节点被移除，导致中国北部数据中心的计算集群的所有服务失去连接。恢复：工程师将软件负载均衡的节点进行了故障转移，恢复了所有的服务。下一步：后台工程师团队会继续跟进并确认最终原因。

SUMMARY OF IMPACT: Between 11:45 and 15:10 CST on 16 Apr 2016 customers using the Azure Management Portal in China East and China North may have experienced issues when trying to access the Azure Portal (https://manage.windowsazure.cn). Customers may also have experienced failures when trying to connect to the Virtual Machines, Redis Cache or Media Services hosted in these regions. Customers using Stream Analytics may have seen creating new jobs and existing jobs may have stalled. Customers using SQL Databases may have seen issues creating, deleting, importing or exporting Databases. Customers using Azure Active Directory may have seen issues performing service management operations. Customers using Service Bus in the region may not have been able to see logs or have been unable to read from service bus resources. PRELIMINARY ROOT CAUSE: Our Load Balancer endpoints for compute clusters were removed due to a Software Load Balancer (SLB) programming issue. This caused a connectivity loss to all services hosted in compute clusters in the China North datacenter. MITIGATION: Azure Engineers performed a failover of SLB endpoints, which recovered all services. NEXT STEPS: The Azure Engineering teams will follow up with a detailed root cause. 

### 2016年4月8日 SQL Server Stretch Database 功能预览版上线  
SQL Server Stretch Database 功能预览版上线。通过 SQL Server Stretch Database，您可以将冷热交互数据从 SQL Server 2016 动态拉伸到 Azure。与冷数据存储不同，您的数据随时可用。通过 Stretch Database, 您可以更长时间地保留数据而不会超时。根据您访问数据的频率选择一个级别，然后根据需要按比例扩展或缩小。使用 Stretch Database 不需要对应用程序作任何更改，并且您可以将 Always Encrypted 与 Stretch Database 搭配使用更安全地扩展数据。有关详细信息，请访问 [SQL Server Stretch Database](/home/features/sql-server-stretch-database) 页面，要全面了解定价情况，请访问 [SQL Server Stretch Database 价格](/pricing/details/sql-server-stretch-database/)页面，使用详情请参见 [SQL Server Stretch Database 服务](/documentation/services/sql-server-stretch-database/)文档。 


### 2016年4月8日  MySQL Database on Azure 现在已经支持任意时间点的回滚 
MySQL Database on Azure 现在允许用户恢复数据库到过去7天中的任意一个时间点，恢复的数据库会运行在用户新建的服务器实例上。对任意时间点的回滚的支持使得用户不用再担心数据遭到损坏或丢失的情况的发生，大大提高了用户检错数据和修复数据的能力。了解详情请访问 [MySQL Database on Azure](/home/features/mysql/)。 

### 2016年4月1日 MySQL 服务主从复制功能预览版上线
MySQL 的复制功能轻松实现弹性扩展，突破单个数据库实例的访问限制，增强了生产工作负载的可用性。复制功能自动将主实例的任何更改复制到所有只读副本中。您可以为指定的数据库实例创建一个或多个只读复制，将读取访问分流到这些只读实例，从而满足越来越大的数据库读取负载。除了读写分离，您也可以用专用的只读实例运行商业智能报告( BI )，或利用复制功能迁移数据库。
有关详细信息，请访问 [MySQL Database on Azure](/home/features/mysql/) 页面，要全面了解定价情况，请访问 [MySQL Database on Azure](/pricing/details/mysql/) 价格页面，使用详情请参见 [MySQL Database on Azure 服务](/documentation/articles/mysql-database-read-replica)文档。 