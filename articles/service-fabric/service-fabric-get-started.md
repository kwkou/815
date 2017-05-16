<properties
    pageTitle="设置 Azure 微服务的开发环境 | Azure"
    description="安装运行时、SDK 和工具并创建本地开发群集。 完成此设置后，你就可以开始生成应用程序。"
    services="service-fabric"
    documentationcenter=".net"
    author="rwike77"
    manager="timlt"
    editor="" />
<tags
    ms.assetid="b94e2d2e-435c-474a-ae34-4adecd0e6f8f"
    ms.service="service-fabric"
    ms.devlang="dotNet"
    ms.topic="get-started-article"
    ms.tgt_pltfrm="NA"
    ms.workload="NA"
    ms.date="03/22/2017"
    wacn.date="05/15/2017"
    ms.author="ryanwi, mikhegn"
    ms.translationtype="Human Translation"
    ms.sourcegitcommit="457fc748a9a2d66d7a2906b988e127b09ee11e18"
    ms.openlocfilehash="57ec9a01c6d4a59d021691177663179954df25d9"
    ms.contentlocale="zh-cn"
    ms.lasthandoff="05/05/2017" />

# <a name="prepare-your-development-environment"></a>准备开发环境
[AZURE.INCLUDE [azure-sdk-developer-differences](../../includes/azure-sdk-developer-differences.md)]

> [AZURE.SELECTOR]
- [Windows](/documentation/articles/service-fabric-get-started/)
- [Linux](/documentation/articles/service-fabric-get-started-linux/)
- [OSX](/documentation/articles/service-fabric-get-started-mac/)

 若要在开发计算机上生成并运行 [Azure Service Fabric 应用程序][1] ，请安装运行时、SDK 和工具。 此外，还需执行 SDK 中包含的 Windows PowerShell 脚本。

## <a name="prerequisites"></a>先决条件
### <a name="supported-operating-system-versions"></a>支持的操作系统版本
支持使用以下操作系统版本进行开发：

* Windows 7
* Windows 8/Windows 8.1
* Windows Server 2012 R2
* Windows Server 2016
* Windows 10

> [AZURE.NOTE]
> 默认情况下，Windows 7 仅包含 Windows PowerShell 2.0。 Service Fabric PowerShell cmdlet 需要 PowerShell 3.0 或更高版本。 可以从 Microsoft 下载中心 [下载 Windows PowerShell 5.0][powershell5-download] 。
> 
> 

## <a name="install-the-sdk-and-tools"></a>安装 SDK 和工具
### <a name="to-use-visual-studio-2017"></a>使用 Visual Studio 2017
Service Fabric 工具是 Visual Studio 2017 中 Azure 开发和管理工作负荷的一部分。 在 Visual Studio 安装过程中启用此工作负荷。
此外，需要使用 Web 平台安装程序安装 Azure Service Fabric SDK。

* [安装 Azure Service Fabric SDK][core-sdk]

### <a name="to-use-visual-studio-2015-requires-visual-studio-2015-update-2-or-later"></a>使用 Visual Studio 2015（需要安装 Visual Studio 2015 Update 2 或更高版本）
对于 Visual Studio 2015，使用 Web 平台安装程序将 Service Fabric 工具和 SDK 一起安装：

* [安装 Azure Service Fabric SDK 和工具][full-bundle-vs2015]

### <a name="sdk-installation-only"></a>仅安装 SDK
如果只需要 SDK，则安装此包：
* [安装 Azure Service Fabric SDK][core-sdk]

> [AZURE.WARNING]
> 在安装过程中使用这些启动链接，或者在 Chrome 浏览器中使用这些链接时，客户会遇到报告的错误。 这些错误是 Web 平台安装程序中的已知问题，我们正在着手解决。  请尝试以下解决方法：
>- 在 Internet Explorer 或 Edge 浏览器中启动上述链接，或者
>- 在开始菜单中启动 Web 平台安装程序，搜索“Service Fabric”，然后安装 SDK
> 
> 对此给你带来的不便，我们深表歉意。 

当前版本包括：
* Service Fabric SDK 2.5.216
* Service Fabric 运行时 5.5.216
* Visual Studio 2015 工具 1.5.50311.1

有关支持的版本列表，请参阅 [Service Fabric 支持](/documentation/articles/service-fabric-support/)

## <a name="enable-powershell-script-execution"></a>启用 PowerShell 脚本执行
Service Fabric 使用 Windows PowerShell 脚本创建本地开发群集和部署 Visual Studio 中的应用程序。 默认情况下，Windows 会阻止这些脚本运行。 若要启用它们，你必须修改你的 PowerShell 执行策略。 以管理员身份打开 PowerShell 并输入以下命令：


	Set-ExecutionPolicy -ExecutionPolicy Unrestricted -Force -Scope CurrentUser

## <a name="next-steps"></a>后续步骤
完成设置开发环境之后，便可开始生成和运行应用。

* [在 Visual Studio 中创建你的第一个 Service Fabric 应用程序](/documentation/articles/service-fabric-create-your-first-application-in-visual-studio/)
* [了解如何在本地群集上部署和管理应用程序](/documentation/articles/service-fabric-get-started-with-a-local-cluster/)
* [了解编程模型：Reliable Services 和 Reliable Actors](/documentation/articles/service-fabric-choose-framework/)

* [使用 Service Fabric 资源管理器可视化群集](/documentation/articles/service-fabric-visualizing-your-cluster/)
* 了解 [Service Fabric 支持选项](/documentation/articles/service-fabric-support/)

[1]: /home/features/service-fabric "Service Fabric 活动页"
[2]: http://go.microsoft.com/fwlink/?LinkId=517106 "VS RC"
[full-bundle-vs2015]:http://www.microsoft.com/web/handlers/webpi.ashx?command=getinstallerredirect&appid=MicrosoftAzure-ServiceFabric-VS2015 "VS 2015 WebPI 链接"
[full-bundle-dev15]:http://www.microsoft.com/web/handlers/webpi.ashx?command=getinstallerredirect&appid=MicrosoftAzure-ServiceFabric-Dev15 "Dev15 WebPI 链接"
[core-sdk]:http://www.microsoft.com/web/handlers/webpi.ashx?command=getinstallerredirect&appid=MicrosoftAzure-ServiceFabric-CoreSDK "Core SDK WebPI 链接"
[powershell5-download]:https://www.microsoft.com/en-us/download/details.aspx?id=50395
<!--update: update SDK version-->