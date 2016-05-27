<properties
   pageTitle="如何将项目升级到最新版本的 Azure Tools | Azure"
   description="了解如何在 Visual Studio 中将 Azure 项目升级到最新版本的 Azure Tools"
   services="visual-studio-online"
   documentationCenter="na"
   authors="TomArcher"
   manager="douge"
   editor="" />
<tags
   ms.service="multiple"
   ms.date="04/18/2016"
   wacn.date="05/23/2016" />

# 如何将项目升级到最新版本的 Azure Tools for Visual Studio

## 概述

安装最新版本的 Azure Tools（或是比 1.6 版更新的旧版）之后，任何使用 1.6 版（2011 年 11 月）以前的 Azure Tools 所创建的项目，在打开时立即自动升级。如果使用这些工具的 1.6 版（2011 年 11 月）创建项目，且仍然安装该版本，可以在较旧版本中打开这些项目，稍后再确定是否要将它们升级。

## 项目升级时有何更改

如果项目自动升级，或指定要将它升级，项目修改为使用最新版本的某些组件，某些属性也更改，如本部分所述。如果项目需要其他更改，才能与较新版本的工具兼容，必须手动进行这些更改。

- Web 角色的 Web.config 文件和辅助角色的 app.config 文件更新为引用较新版的 Microsoft.WindowsAzure.Diagnostics.DiagnosticMonitoirTraceListener.dll。

- Microsoft.WindowsAzure.StorageClient.dll、Microsoft.WindowsAzure.Diagnostics.dll 和 Microsoft.WindowsAzure.ServiceRuntime.dll 程序集升级到新的版本。

- 存储在 Azure 项目文件 (.ccproj) 中的发布配置文件将转到 **Publish** 子目录中的另一个文件，扩展名为 .azurePubXml。

- 发布配置文件中的某些属性更新为支持新的和更改的功能。**AllowUpgrade** 已替换为 **DeploymentReplacementMethod**，因为你可以同时或以递增方式更新已部署的云服务。

- 将添加属性 **UseIISExpressByDefault** 并将其设置为 false，以便用于调试的 Web 服务器不会自动从 Internet 信息服务 (IIS) 切换为 IIS Express。IIS Express 是较新版本的工具创建的项目的默认 Web 服务器。

- 如果 Azure Caching 托管在一个或多个项目的角色中，当项目升级时，服务配置（.cscfg 文件）和服务定义（.csdef 文件）中的某些属性将会更改。如果项目使用 Azure Caching NuGet 包，项目将升级到最新版本的包。你应该打开 Web.config 文件，并验证客户端配置在升级期间是否已得到适当维护。如果添加了对 Azure Caching 客户端程序集的引用而未使用 NuGet 包，则不会更新这些程序集；你必须手动更新对新版本的这些引用。

>[AZURE.IMPORTANT] 对于 F# 项目，必须手动更新对 Azure 程序集的引用，使项目引用这些程序集的较新版本。

### 如何将 Azure 项目升级到最新版本

1. 将最新版本的 Azure tools 安装到想要用于升级项目的 Visual Studio 安装，然后打开想要升级的项目。如果项目是使用 1.6 版（2011 年 11 月）以前的 Azure Tools 创建的，项目将自动升级到最新版本。如果项目是使用 2011 年 11 月版本创建的，且仍然安装该版本，则项目将在该版本中打开。

1. 在解决方案资源管理器中，打开项目节点的快捷菜单，选择“属性”，然后在出现的对话框中选择“应用程序”选项卡。

    “应用程序”选项卡将显示与项目关联的工具版本。如果显示了 Azure Tools 的最新版本，则表示项目已升级。如果已安装的工具版本比选项卡中显示的版本更高，则会出现“升级”按钮。

1. 选择“升级”按钮，将项目升级到最新版本的工具。

1. 生成项目，然后解决由于 API 更改而导致的任何错误。有关如何针对新版本修改代码的信息，请参阅特定 API 的文档。

<!---HONumber=Mooncake_0503_2016-->