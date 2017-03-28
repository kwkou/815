<properties
    pageTitle="威胁检测 - Azure SQL 数据库 | Azure"
    description="威胁检测会检测异常的数据库活动，指出数据库有潜在的安全威胁。"
    services="sql-database"
    documentationcenter=""
    author="ronitr"
    manager="jhubbard"
    editor="v-romcal" />
<tags
    ms.assetid="b50d232a-4225-46ed-91e7-75288f55ee84"
    ms.service="sql-database"
    ms.custom="secure and protect"
    ms.devlang="NA"
    ms.topic="article"
    ms.tgt_pltfrm="NA"
    ms.workload="data-services"
    ms.date="07/10/2016"
    wacn.date="03/24/2017"
    ms.author="ronmat; ronitr" />  


# SQL 数据库威胁检测

威胁检测会检测异常的数据库活动，指出数据库有潜在的安全威胁。威胁检测目前为预览版。

## 概述

威胁检测提供新的安全层，在发生异常活动时会提供安全警报，让客户检测潜在威胁并做出响应。用户可以使用 [SQL 数据库审核](/documentation/articles/sql-database-auditing/)来探查可疑事件，判断这些可疑事件是否是因为有人尝试访问、破坏或利用数据库中的数据而生成的。无需成为安全专家，也不需要管理先进的安全监视系统，就能使用威胁检测轻松解决数据库的潜在威胁。

威胁检测会检测异常的数据库活动，指出潜在的 SQL 注入企图。SQL 注入是 Internet 上常见的 Web 应用程序安全问题之一，用于攻击数据驱动的应用程序。攻击者利用应用程序漏洞将恶意 SQL 语句注入应用程序入口字段，以破坏或修改数据库中的数据。

## 后续步骤

* 若要配置和管理威胁检测，请参阅[在 Azure 门户预览中配置和管理威胁检测](/documentation/articles/sql-database-threat-detection-portal/)。

<!---HONumber=Mooncake_0320_2017-->