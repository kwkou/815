<properties
   pageTitle="使用 Azure Resource Manager 模板设置 Service Fabric 群集 | Azure"
   description="使用 Azure Resource Manager 模板设置 Service Fabric 群集。"
   services="service-fabric"
   documentationCenter=".net"
   authors="ChackDan"
   manager="timlt"
   editor=""/>

<tags
   ms.service="service-fabric"
   ms.date="05/02/2016"
   wacn.date="07/07/2016"/>


# 使用 Azure Resource Manager 模板设置 Service Fabric 群集

本页面将帮助你使用 Azure Resource Manager 模板来设置 Service Fabric 群集。为此，你的订阅必须有足够的核心用于部署将构成此群集的 IaaS VM。

## 先决条件

- 若要设置安全群集，必须先将 X.509 证书上载到你的密钥保管库。你将需要源保管库 URL、证书 URL，以及证书指纹。
- 有关如何设置安全群集的详细信息，请参阅 [Service Fabric 群集安全性](/documentation/articles/service-fabric-cluster-security/)。

## 获取示例 Resource Manager 模板

可以从 [GitHub 上的 Azure 快速入门模板库](https://github.com/Azure/azure-quickstart-templates)获取示例 Resource Manager 模板。所有 Service Fabric 模板的名称都以“service-fabric...”开头。可以在存储库中搜索“fabric”，或向下滚动到示例模板集。为了帮助你快速找到所需的模板，模板已按以下方式命名：

- [service-fabric-unsecure-cluster-5-node-1-nodetype](https://github.com/Azure/azure-quickstart-templates/tree/master/service-fabric-unsecure-cluster-5-node-1-nodetype)：表示包含五个节点的单节点类型不安全群集模板。

- [service-fabric-secure-cluster-5-node-1-nodetype-wad](https://github.com/Azure/azure-quickstart-templates/tree/master/service-fabric-secure-cluster-5-node-1-nodetype-wad)：表示已启用 WAD、包含五个节点的单节点类型安全群集模板。

- [service-fabric-secure-cluster-10-node-2-nodetype-wad](https://github.com/Azure/azure-quickstart-templates/tree/master/service-fabric-secure-cluster-5-node-1-nodetype-wad)：表示已启用 WAD、包含十个节点的双节点类型安全群集模板。

## 创建自定义 Resource Manager 模板

可以从 [GitHub 上的 Azure 快速入门模板库](https://github.com/Azure/azure-quickstart-templates)获取示例模板，然后对其进行更改。
<!--
2. 可以登录到 Azure 门户预览，然后使用 Service Fabric 门户页来生成可自定义的模板。为此，请执行以下步骤：

    a.登录到 [Azure 门户预览](https://portal.azure.cn/)。

    b.完成[通过 Azure 门户预览创建 Service Fabric 群集](/documentation/articles/service-fabric-cluster-creation-via-portal/)一文中所述的群集创建过程，但不要单击“创建”，而是转到“摘要”并下载模板，如以下屏幕截图所示。

 ![Service Fabric 群集页的屏幕截图，其中显示了用于下载 Resource Manager 模板的链接][DownloadTemplate]
-->
## 使用 Azure PowerShell 将 Resource Manager 模板部署到 Azure

有关如何使用 PowerShell 部署模板的详细说明，请参阅[使用 PowerShell 部署 Resource Manager 模板](/documentation/articles/resource-group-template-deploy/)。

>[AZURE.NOTE] Service Fabric 群集需要有一定数量的节点可随时启动，以保持可用性和状态 - 称为“维持仲裁”。因此，除非你已事先执行[状态的完整备份](/documentation/articles/service-fabric-reliable-services-backup-restore/)，否则关闭群集中的所有计算机通常是不安全的做法。

<!--Every topic should have next steps and links to the next logical set of content to keep the customer engaged-->
## 后续步骤
- [Service Fabric 群集安全性](/documentation/articles/service-fabric-cluster-security/)
- [在 Visual Studio 中管理 Service Fabric 应用程序](/documentation/articles/service-fabric-manage-application-in-visual-studio/)。
- [Service Fabric 运行状况模型简介](/documentation/articles/service-fabric-health-introduction/)

<!--Image references-->
[DownloadTemplate]: ./media/service-fabric-cluster-creation-via-arm/DownloadTemplate.png

<!---HONumber=Mooncake_0523_2016-->