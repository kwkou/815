<properties
   pageTitle="设置开发环境 | Azure"
   description="安装运行时、SDK 和工具并创建本地开发群集。完成此设置后，你就可以开始生成应用程序。"
   services="service-fabric"
   documentationCenter=".net"
   authors="seanmck"
   manager="samgeo"
   editor=""/>

<tags
   ms.service="service-fabric"
   ms.date="06/15/2016"
   wacn.date="07/04/2016"/>

# 准备开发环境
 若要在开发计算机上生成并运行 [Azure Service Fabric 应用程序][1]，你需要安装运行时、SDK 和工具。此外，还需执行 SDK 中包含的 Windows PowerShell 脚本。

## 先决条件
### 支持的操作系统版本
支持使用以下操作系统版本进行开发：

- Windows 7
- Windows 8/Windows 8.1
- Windows Server 2012 R2
- Windows 10

>[AZURE.NOTE] 默认情况下，Windows 7 仅包含 Windows PowerShell 2.0。需要安装 PowerShell 3.0 或更高版本才能使用 Service Fabric PowerShell cmdlet。可以从 Microsoft 下载中心[下载 Windows PowerShell 5.0][powershell5-download]。

## 安装运行时、SDK 和工具

Web 平台安装程序为 Service Fabric 开发提供三种配置：

- [安装适用于 Visual Studio 2015 Update 2 的 Service Fabric 运行时、SDK 和工具][full-bundle-vs2015]
- [安装适用于 Visual Studio "15" 预览版的 Service Fabric 运行时、SDK 和工具][full-bundle-dev15]
- [仅安装 Service Fabric 运行时和 SDK（不安装 Visual Studio 工具）][core-sdk]


## 启用 PowerShell 脚本执行

Service Fabric 使用 Windows PowerShell 脚本创建本地开发群集和部署 Visual Studio 中的应用程序。默认情况下，Windows 会阻止这些脚本运行。若要启用它们，你必须修改你的 PowerShell 执行策略。以管理员身份打开 PowerShell 并输入以下命令：

```powershell
Set-ExecutionPolicy -ExecutionPolicy Unrestricted -Force -Scope CurrentUser
```

## 后续步骤
设置开发环境之后，你可以开始构建和运行应用。

- [在 Visual Studio 中创建你的第一个 Service Fabric 应用程序](/documentation/articles/service-fabric-create-your-first-application-in-visual-studio/)
- [了解如何在本地群集上部署和管理应用程序](/documentation/articles/service-fabric-get-started-with-a-local-cluster/)
- [了解编程模型：Reliable Services 和 Reliable Actors](/documentation/articles/service-fabric-choose-framework/)

- [使用 Service Fabric 资源管理器可视化群集](/documentation/articles/service-fabric-visualizing-your-cluster/)

[1]: /home/features/service-fabric "Service Fabric 活动页"
[2]: http://go.microsoft.com/fwlink/?LinkId=517106 "VS RC"
[full-bundle-vs2015]: http://www.microsoft.com/web/handlers/webpi.ashx?command=getinstallerredirect&appid=MicrosoftAzure-ServiceFabric-VS2015-2_1 "VS 2015 WebPI 链接"
[full-bundle-dev15]: http://www.microsoft.com/web/handlers/webpi.ashx?command=getinstallerredirect&appid=MicrosoftAzure-ServiceFabric-Dev15-2_1 "Dev15 WebPI 链接"
[core-sdk]: http://www.microsoft.com/web/handlers/webpi.ashx?command=getinstallerredirect&appid=ServiceFabricSDK_2_1 "Core SDK WebPI 链接"
[powershell5-download]: https://www.microsoft.com/zh-cn/download/details.aspx?id=50395

<!---HONumber=Mooncake_0627_2016-->