<properties
   pageTitle="资源管理器和服务管理（经典）部署模型 | Azure"
   description="了解资源管理器与经典部署模型之间的差异。"
   services="virtual-network"
   documentationCenter=""
   authors="telmosampaio"
   manager="carolz"
   editor=""
   tags="azure-resource-manager,azure-service-management"/>

<tags
   ms.service="virtual-network"
   ms.date="02/11/2016"
   wacn.date="03/21/2016"/>

# Azure 部署模型

Azure 平台正在转换。不论你是 Azure 新手还是经验丰富的老手，都必须了解我们在此转换期间对此平台所做的一些重要更改。

所有 Azure 资源都支持以下一个或两个部署模型：

- **资源管理器：**这是 Azure 资源的最新部署模型。大部分较新的资源都已支持这种部署模型，最终所有资源都会支持它。   
 
- **经典：**目前大部分现有 Azure 资源都支持此模型。添加到 Azure 的新资源将不支持此模型。

每种 Azure 资源的文档详述了可用于创建该资源的服务模型。

## 为什么了解这一点很重要？ 

原因如下：

- 在这两种模型下使用的 Azure 平台功能有所不同。例如，使用资源管理器部署模型（或只是资源管理器）创建的资源可以通过 [Azure 资源管理器模板](/documentation/articles/resource-group-overview/#template-deployment)创建，而使用经典部署模型创建的资源则不能这么做。
- 单个 Azure 资源功能或行为在这两种模型下有所不同，或只存在于其中一种模型。例如，平衡使用经典部署模型创建的虚拟机的流量负载是*隐式的*，因为虚拟机是 Azure 云服务的成员，而云服务内的虚拟机会自动平衡负载。使用资源管理器创建的虚拟机不是云服务的成员，你必须*显式*创建不同的 Azure 负载平衡器资源，以平衡多个虚拟机的流量负载。  
- 在这两种模型中，创建、配置和管理 Azure 资源的方式有所不同。
- 使用一种部署模型所创建的资源，不一定能与使用不同部署模型创建的资源互操作。例如，使用一种部署模型创建的 Azure 虚拟机只能连接到使用相同部署模型创建的 Azure 虚拟网络。    

每种部署模型的基础是每个资源的应用程序编程接口 (API)。资源管理器部署模型有[资源管理器 API](https://msdn.microsoft.com/zh-cn/library/azure/dn948464.aspx)，经典部署模型有[服务管理 API](https://msdn.microsoft.com/zh-cn/library/azure/ee460799.aspx)。开发人员可以编写代码，以*直接*与这些 API 交互。

但是，IT 专业人员通常在 Web 浏览器中使用图形门户、在 Windows 计算机上使用 Azure PowerShell cmdlet，或在 Windows、OS X 或 Linux 计算机上使用 Azure 命令行界面 (CLI) 来与这些 API *间接*交互。IT 专业人员使用的这三种间接方法都能直接与 API 交互。这意味着 Azure 平台或资源引入新功能时，始终可以先通过 API 直接获取，而在 API 可供使用后通过间接方法获取新资源和功能的支持。

以下部分说明了如何通过三种间接方法，使用不同的部署模型来配置 Azure 资源。

## 门户

- **[Azure 门户](https://manage.windowsazure.cn)：**如果你已使用 Azure 一段时间，便已用过此门户。该门户用于创建和配置可支持经典部署模型的旧式 Azure 资源。你无法使用它来创建或配置仅支持资源管理器的资源。 

某些资源和功能只可以在其中一个门户中创建和配置。某些资源或功能还不能在任何一个门户中创建或配置，而只能通过 PowerShell 和/或 CLI 进行配置。每种 Azure 资源的文档详述了可用于创建该资源的方法。

## PowerShell
通过 [PowerShell](/documentation/articles/powershell-install-configure/)，可以使用命令行或编写脚本，从 Windows 计算机创建和配置 Azure 资源。每个 Azure 资源都有相应的[资源管理器 cmdlet](https://msdn.microsoft.com/zh-cn/library/azure/mt125356.aspx) 和/或[服务管理 cmdlet](https://msdn.microsoft.com/zh-cn/library/azure/dn708504.aspx)。某些资源和功能只能使用 PowerShell 或 CLI 来创建和配置。根据具体的资源，使用资源管理器 PowerShell cmdlet 时，你可以使用两个选项创建和配置 Azure 资源：

- **仅限 PowerShell cmdlet：**可以使用每个资源的 cmdlet 单独创建和配置每个 Azure 资源。你可以从命令行执行此操作，或者在 PowerShell 脚本中包含可存储和设置版本的多个命令。

- **将 PowerShell cmdlet 与 Azure 资源管理器模板配合使用：**你可以将 PowerShell 与 Azure 资源管理器模板配合使用来创建 Azure 资源。可以保存模板并设置其版本。<!--有关详细信息，请阅读[使用 Azure 资源管理器模板部署应用程序](/documentation/articles/resource-group-template-deploy/)一文。你还可以下载和修改常见解决方案的多个 [Azure 快速入门模板](http://azure.microsoft.com/documentation/templates/)。-->

## CLI
你可以使用 CLI 从 Windows、OS X 或 Linux 计算机创建和配置 Azure 资源。请阅读[安装 Azure CLI](/documentation/articles/xplat-cli-install/) 一文，以便在所选的操作系统上安装 CLI。

## 后续步骤

- 详细了解[资源管理器](/documentation/articles/resource-group-overview/)。

<!---HONumber=Mooncake_0314_2016-->