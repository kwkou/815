<properties
    pageTitle="用于 .NET 的 Azure SDK 2.9 发行说明"
    description="用于 .NET 的 Azure SDK 2.9 发行说明"
    services="app-service\web"
    documentationcenter=".net"
    author="chrissfanos"
    editor=""
    translationtype="Human Translation" />
<tags
    ms.assetid="c83d815b-fc19-4260-821e-7d2a7206dffc"
    ms.service="app-service"
    ms.devlang="multiple"
    ms.topic="article"
    ms.tgt_pltfrm="na"
    ms.workload="integration"
    ms.date="02/24/2017"
    wacn.date="04/24/2017"
    ms.author="juliako;mikhegn"
    ms.sourcegitcommit="a114d832e9c5320e9a109c9020fcaa2f2fdd43a9"
    ms.openlocfilehash="c844f50026936ecf7beec863c99be82d1a078e3d"
    ms.lasthandoff="04/14/2017" />

# <a name="azure-sdk-for-net-29-release-notes"></a>用于 .NET 的 Azure SDK 2.9 发行说明

本主题包括用于 .NET 的 Azure SDK 版本 2.9 和 2.9.6 的发行说明。

##<a name="azure-sdk-for-net-296-release-summary"></a>用于 .NET 的 Azure SDK 2.9.6 版本摘要

发布日期：2016 年 11 月 16 日

在此版本中未引入对 Azure SDK 2.9 的任何重大更改。 将此 SDK 用于现有的云服务项目时，也不需要任何升级过程。

### <a name="visual-studio-2017-release-candidate"></a>Visual Studio 2017 候选发布

- 在 Visual Studio 2017 RC 中，此版本的用于 .NET 的 Azure SDK 内置于 Azure 工作负荷中。 从目前开始，进行 Azure 开发所需的所有工具都将成为 Visual Studio 2017 RC 的一部分。 对于 Visual Studio 2015 和 Visual Studio 2013，仍将可通过 WebPI 获得 SDK。 Visual Studio 2017 作为最终产品发布时，将不再发布 Visual Studio 2013 的 用于 .NET 的 Azure SDK 版本。 通过此链接可下载 Visual Studio 2017 RC：https://www.visualstudio.com/vs/visual-studio-2017-rc/

### <a name="azure-diagnostics"></a>Azure 诊断

- 已更改该行为，以仅存储部分连接字符串，并将密钥替换为云服务诊断存储连接字符串的令牌。 实际的存储密钥现在存储在用户配置文件文件夹中，因此可以控制其访问权限。 Visual Studio 将在本地调试和发布过程中从用户配置文件文件夹中读取存储密钥。 
- 为响应上述更改，Visual Studio Online 团队已增强 Azure 云服务部署任务模板，以便用户在连接集成和部署中发布到 Azure 时可以指定用于设置诊断扩展的存储密钥。
- 我们已实现存储 Azure 诊断 (WAD) 的安全连接字符串和词汇切分，以帮助用户跨环境解决配置问题。

### <a name="windows-server-2016-virtual-machines"></a>Windows Server 2016 虚拟机

- Visual Studio 现在支持将云服务部署到 OS 系列 5 (Windows Server 2016) 虚拟机。 对于现有的云服务，可以更改设置以针对新的 OS 系列。 创建新的云服务时，如果选择使用 .net 4.6 或更高版本创建服务，则服务将默认使用 OS 系列 5。  有关详细信息，可查看[来宾 OS 系列支持表](/documentation/articles/cloud-services-guestos-update-matrix/)。

#### <a name="known-issues"></a>已知问题

- Azure .NET SDK 2.9.6 引入了一个限制，阻止用户使用不受支持的 .NET 框架（例如 .NET 4.6）将项目部署到任何 OS 系列 (< 5)。 [此处](https://github.com/MicrosoftDocs/azure-cloud-services-files/tree/master/Azure%20Targets%20SDK%202.9)通过了一个解决方法。

### <a name="azure-in-role-cache"></a>Azure 角色中缓存 

- 对 Azure 角色中缓存的支持于 2016 年 11 月 30 日结束。 有关更多详细信息，请单击 [此处](https://azure.microsoft.com/zh-cn/blog/azure-managed-cache-and-in-role-cache-services-to-be-retired-on-11-30-2016/)。

### <a name="azure-resource-manager-templates-for-azure-stack"></a>Azure Stack 的 Azure Resource Manager 模板

- 我们已引入 Azure Stack 作为 Azure Resource Manager 模板的部署目标。

## <a name="azure-sdk-for-net-29-summary"></a>用于 .NET 的 Azure SDK 2.9 摘要

## <a name="overview"></a>概述
本文档包含 Azure SDK for .NET 2.9 版本的发行说明。 

有关此版本中的更新的详细信息，请参阅 [Azure SDK 2.9 公告文章](https://azure.microsoft.com/zh-cn/blog/announcing-visual-studio-azure-tools-and-sdk-2-9/)。

## <a name="azure-sdk-29-for-visual-studio-2015-update-2-and-visual-studio-15-preview"></a>Azure SDK 2.9 for Visual Studio 2015 Update 2 和 Visual Studio“15”预览版
此更新包含下列 bug 修复：

* 与 REST API 客户端生成有关的问题，在此问题中，字符串“未知类型”会作为代码生成文件夹的名称和/或放入所生成代码的命名空间的名称出现。
* 与计划 Web 作业有关的问题，在此问题中，验证信息无法传递给计划程序预配进程。

此更新包含下列新功能：

* 在“应用服务预配”对话框的“服务”选项卡中支持辅助应用服务。 

## <a name="azure-data-lake-tools-for-visual-studio-2015-update-2"></a>Azure Data Lake Tools for Visual Studio 2015 Update 2
此更新包含下列工具：

* **Azure Data Lake 工具** 现在已合并到用于 .NET 的 Azure SDK 发行版中。 当你安装 Azure SDK 时，便会自动安装此工具。 

    此工具会经常更新，请转到 [此处](http://aka.ms/datalaketool) 获取更新。
* **服务器资源管理器** 可以查看所有 U-SQL 元数据实体和创建一些 U-SQL 元数据实体。 

## <a name="hdinsight-tools"></a>HDInsight 工具
**HDInsight 工具** 现在支持 HDInsight 3.3 版，包括显示 Tez 图形和其他语言修复。

## <a name="azure-resource-manager"></a>Azure 资源管理器
此版本添加了对 Resource Manager 模板的[密钥保管库](/documentation/articles/resource-manager-keyvault-parameter/)支持。

## <a name="see-also"></a>另请参阅
[Azure SDK 2.9 通知文章](https://azure.microsoft.com/zh-cn/blog/announcing-visual-studio-azure-tools-and-sdk-2-9/)
<!--Update_Description: add a known issue-->