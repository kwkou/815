<properties
   pageTitle="为 Azure 构建的命令行 | Azure"
   description="为 Azure 构建的命令行"
   services="visual-studio-online"
   documentationCenter="na"
   authors="TomArcher"
   manager="douge"
   editor="" />
<tags
   ms.service="multiple"
   ms.date="05/08/2016"
   wacn.date="05/23/2016" />

# 为 Azure 构建的命令行

## 概述

可以通过在命令提示符下运行 MSBuild 来为 Azure 部署创建包。除了自动化某些生成过程外，你还可以配置和定义用于调试、过渡和生产的生成。


## Microsoft 生成引擎 (MSBuild)

通过使用 Microsoft 生成引擎 (MSBuild)，你可以在未安装 Visual Studio 的生成实验室环境中生成产品。MSBuild 对可扩展且 Microsoft 完全支持的项目文件使用 XML 格式。在此文件格式中，你可以描述必须为一个或多个平台和配置生成的项目。

还可在命令提示符下运行 MSBuild，本主题描述了该方法。通过在命令提示符下设置属性，可以生成项目的特定配置。同样，还可以定义 MSBuild 命令将生成的目标。有关命令行参数和 MSBuild 的详细信息，请参阅 [MSBuild 命令行参考](https://msdn.microsoft.com/zh-cn/library/ms164311.aspx)。

## 安装

如以下过程所述，必须先在生成服务器上安装软件和工具，才能使用 MSBuild 创建 Azure 包：

1. 安装包括 MSBuild 的 .NET Framework 4 或更高版本。

1. 安装 [Azure 创作工具](/downloads)（查找 MicrosoftAzureAuthoringTools-x64.msi 或 MicrosoftAzureAuthoringTools-x86.msi）。

1. 安装[用于 .NET 的 Azure 库](/downloads)（查找 MicrosoftAzureLibsForNet-x64.msi 或 MicrosoftAzureLibs-x86.msi）。

1. 从另一台计算机上的 Visual Studio 复制 Microsoft.WebApplication.targets 文件。

    该文件位于目录 C:\\Program Files (x86)\\MSBuild\\Microsoft\\Visual Studio\\v12.0\\WebApplications (v11.0 for Visual Studio 2012) 中，应将其复制到生成服务器上的同一目录。

1. 安装 [Azure Tools for Visual Studio](http://go.microsoft.com/fwlink/?LinkId=394616)。

    查找 WindowsAzureTools.vs120.exe 以生成 Visual Studio 2013 项目。

## MSBuild 参数

创建包的最简方法是使用 `/t:Publish` 选项运行 MSBuild。默认情况下，此命令将创建与项目的根文件夹相关的目录，例如 ProjectDir\\bin\\Configuration\\app.publish。在生成 Azure 项目时，将生成两个文件，即包文件本身和附带的配置文件：

- Project.cspkg

- ServiceConfiguration.TargetProfile.cscfg

默认情况下，每个 Azure 项目均包含两个服务配置文件，这两个文件分别针对本地（调试）生成和云（过渡或生产）生成，你可根据需要添加或删除服务配置文件。在 Visual Studio 中生成包时，系统会询问你要将哪个服务配置文件与包一起包含。使用 MSBuild 生成包时，默认情况下将包含本地服务配置文件。若要包含其他服务配置文件，请设置 MSBuild 命令 (`MSBuild /t:Publish /p:TargetProfile=ProfileName`) 的 `TargetProfile` 属性。

如果想为存储的包和配置文件使用备用目录，请使用 `/p:PublishDir=Directory` 选项设置路径，包括尾部反斜杠分隔符。

## 部署

生成包后，即可将其部署到 Azure。有关演示该过程的教程，请参阅 Azure 网站。有关如何自动化该过程的信息，请参阅[在 Azure 中持续交付云服务](/documentation/articles/cloud-services-dotnet-continuous-delivery/)。

<!---HONumber=Mooncake_0516_2016-->