<properties
    pageTitle="Azure SQL 数据库管理和开发工具 | Azure"
    description="介绍 Azure SQL 数据库的管理、开发工具和选项"
    services="sql-database"
    documentationcenter=""
    author="CarlRabeler"
    manager="jhubbard"
    editor=""
    translationtype="Human Translation" />
<tags
    ms.assetid="37767380-975f-4dee-a28d-80bc2036dda3"
    ms.service="sql-database"
    ms.custom="overview"
    ms.workload="data-management"
    ms.tgt_pltfrm="na"
    ms.devlang="na"
    ms.topic="article"
    ms.date="03/03/2017"
    wacn.date="04/17/2017"
    ms.author="carlrab"
    ms.sourcegitcommit="7cc8d7b9c616d399509cd9dbdd155b0e9a7987a8"
    ms.openlocfilehash="328aca5f9c08e2e7ebd9cd09b7a4f00c6c2ac7b7"
    ms.lasthandoff="04/07/2017" />

# <a name="overview-tools-to-manage--develop-with-azure-sql-database"></a>概述：Azure SQL 数据库管理和开发工具
本主题介绍用于 Azure SQL 数据库管理和开发的工具。

> [AZURE.IMPORTANT]
> 此文档集包括快速入门、示例及操作指南，展示如何使用以下各段中介绍的工具来管理和开发 Azure SQL 数据库。 使用左侧的导航窗格和筛选器框可查找 Azure 门户、PowerShell 和 T-SQL 的特定内容。
>

## <a name="azure-portal"></a>Azure 门户
[Azure 门户](https://portal.azure.cn)是一个基于 Web 的应用程序，可以在其中创建、更新和删除数据库及逻辑服务器并监视数据库活动。 如果刚开始使用 Azure（管理少量的数据库）或需要快速执行某些操作，该工具是理想之选。

## <a name="sql-server-management-studio-and-transact-sql"></a>SQL Server Management Studio 和 Transact-SQL
SQL Server Management Studio (SSMS) 是一种在计算机上运行的客户端工具，用于通过 Transact-SQL 管理云中的数据库。 许多数据库管理员都熟悉 SSMS（可用于 Azure SQL 数据库）。 [下载最新版本的 SSMS](https://msdn.microsoft.com/zh-cn/library/mt238290)，在使用 Azure SQL 数据库时始终使用该最新版本。 

> [AZURE.IMPORTANT]
> 请务必使用最新版本的 [SQL Server Management Studio](https://msdn.microsoft.com/zh-cn/library/mt238290)。
>  

## <a name="sql-server-data-tools-in-visual-studio"></a>Visual Studio 中的 SQL Server Data Tools
SQL Server Data Tools (SSDT) 是一种在计算机上运行的客户端工具，用于开发云中的数据库。 如果你是熟悉 Visual Studio 或其他集成开发环境 (IDE) 的应用程序开发人员，请[尝试使用 Visual Studio 中的 SSDT](https://msdn.microsoft.com/zh-cn/library/mt204009.aspx)。  

> [AZURE.IMPORTANT]
> 请务必使用最新版本的 [SQL Server Data Tools](https://msdn.microsoft.com/zh-cn/library/mt204009.aspx) 以保持与 Azure 和 SQL 数据库的更新同步。
>  
## <a name="powershell"></a>PowerShell
可以使用 PowerShell 管理数据库和弹性池，并自动执行 Azure 资源部署。 Azure 建议在生产环境中使用此工具来管理大量的数据库并自动进行部署和资源更改。

## <a name="elastic-database-tools"></a>弹性数据库工具
使用弹性数据库工具执行如下操作： 


* 使用[拆分-合并工具](/documentation/articles/sql-database-elastic-scale-overview-split-and-merge/)将多租户模型数据库移至单租户模型
* 使用[弹性扩展客户端库](/documentation/articles/sql-database-elastic-database-client-library/)管理单租户模型或多租户模型中的数据库。

## <a name="additional-resources"></a>其他资源
* [Azure 自动化](/documentation/services/automation/)
* [Azure 计划程序](/documentation/services/scheduler/)
<!--Update_Description: wording update-->
