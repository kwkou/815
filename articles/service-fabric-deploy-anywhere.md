<properties
   pageTitle="在 Windows Server 和 Linux 上创建 Azure Service Fabric 群集 | Azure"
   description="Service Fabric 群集会在 Windows Server 或 Linux 上运行，这意味着你将能够在可以运行 Windows Server 和 Linux 的任何位置部署和承载 Service Fabric 应用程序。"
   services="service-fabric"
   documentationCenter=".net"
   authors="kunalds"
   manager="timlt"
   editor=""/>

<tags
   ms.service="service-fabric"
   ms.date="05/02/2016"
   wacn.date="07/04/2016"/>

# 在 Windows Server 或 Linux 上创建独立的 Service Fabric 群集
Azure Service Fabric 允许在运行 Windows Server 或 Linux 的任何 VM 或计算机上创建 Service Fabric 群集。这意味着你能够在具有一组相互连接的 Windows Server 或 Linux 计算机的任何环境（无论是本地环境还是任何云提供商所提供的）中部署和运行 Service Fabric 应用程序。

**注意**：应该通过 Azure 资源模型模板在 Azure 上创建群集。有关详细信息，请阅读 [Create a Service Fabric cluster by using an Azure Resource Manager template（使用 Azure Resource Manager 模板创建 Service Fabric 群集）](/documentation/articles/service-fabric-cluster-creation-via-arm/)

Service Fabric 提供了一个安装包用于在本地创建这些独立 Service Fabric 群集。此功能的一个主要优点是在使用 Service Fabric 构建应用程序时不存在供应商锁定，因为是由你选择这些应用程序的运行位置。此功能还会使你有更大的能力实现更广泛的客户群，因为客户对于想要在其中运行你的应用程序的环境可能具有不同的要求。例如，医疗保健和金融行业中的客户的需求可能与汽车或旅行行业中的客户不同。

## 支持的操作系统
你将能够在运行以下这些操作系统的 VM 或计算机上创建群集：

* Windows Server 2012 R2
* Windows Server 2016
* Linux

有关 Windows Server 的详细信息，请参阅 [Service Fabric cluster creation for Windows Server（创建适用于 Windows Server 的 Service Fabric 群集）](/documentation/articles/service-fabric-cluster-creation-for-windows-server/)

## 群集创建和配置
Service Fabric 提供可以下载的安装包。下载了此包之后，你便需要对 JSON 配置文件进行更改，以指定群集的设置。编辑了群集设置之后，你会运行安装程序脚本，该脚本会创建跨群集设置中指定的计算机的群集。还可以运行一个脚本来从一组计算机中删除群集。

## 任何云部署与本地部署
用于在本地创建 Service Fabric 群集的过程类似于在具有一组 VM 的任何所选云上创建群集的过程。用于设置 VM 的初始步骤由所使用的云提供程序或本地环境进行控制。具有一组在它们之间启用了网络连接的 VM 之后，随后用于设置 Service Fabric 包、编辑群集设置以及运行群集创建和管理脚本的步骤相同。这可在你选择面向新宿主环境时，确保你操作和管理 Service Fabric 群集的知识和经验可转移。

## 创建独立 Service Fabric 群集的优点
* 由于不存在供应商限制，你可以选择创建群集的位置。
* Service Fabric 应用程序在编写之后，可以在多个宿主环境中运行，只需进行最小程度的更改，甚至无需更改。
* 构建 Service Fabric 应用程序的知识可从一个宿主环境转移到另一个环境。
* 运行和管理 Service Fabric 群集的操作经验可从一个环境转移到另一个环境。
* 广泛的客户范围不会受宿主环境约束的限制。
* 存在针对大范围中断的额外一层可靠性和保护，因为你可以在数据中心或云提供商中断时将服务转移到其他部署环境。

## Azure 上的 Service Fabric 群集的优点胜于在本地创建的独立 Service Fabric 群集
在 Azure 上运行 Service Fabric 群集相对于本地选项具有一些优势，因此，如果你对于群集的运行位置没有特定需求，则我们建议在 Azure 上运行它们。在 Azure 上，我们提供与其他 Azure 功能和服务的集成，这样可使群集的操作和管理更容易且更可靠。

* **Azure 门户预览：**Azure 门户预览使群集易于创建和管理。

* **Azure Resource Manager：**使用 Azure Resource Manager 可以单元的形式方便地管理群集使用的所有资源，并简化了成本跟踪和计费。
* **用作 Azure 资源的 Service Fabric 群集：**Service Fabric 群集是一种 ARM 资源，你可以像在 Azure 中处理其他 ARM 资源一样为它建模。
* **与 Azure 基础结构集成：**Service Fabric 协调 OS 的 Azure 基础结构、网络和其他升级，以提高应用程序的可用性与可靠性。  
* **诊断：**在 Azure 中，我们提供与 Azure 诊断和操作见解的集成。
* **自动缩放：**对于 Azure 上的群集，我们借助虚拟机缩放集提供内置自动缩放功能。在本地和其他云环境中，你必须构建自己的自动缩放功能或使用 Service Fabric 为缩放群集而公开的 API 来手动缩放。

## 后续步骤
在运行 Windows Server 的 VM 或计算机上创建群集：[创建适用于 Windows Server 的 Service Fabric 群集](/documentation/articles/service-fabric-cluster-creation-for-windows-server/)

在运行 Linux 的 VM 或计算机上创建群集：[Linux 上的 Service Fabric](/documentation/articles/service-fabric-linux-overview/)

<!---HONumber=Mooncake_0523_2016-->