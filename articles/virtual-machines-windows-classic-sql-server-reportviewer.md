<properties 
	pageTitle="在 Web 应用中使用 ReportViewer | Azure"
	description="本主题介绍如何使用 Visual Studio ReportViewer 控件构建 Azure Web 应用，该控件用于显示存储在 Azure 虚拟机上的报表。"
	services="virtual-machines-windows"
	documentationCenter="na"
	authors="guyinacube"
	manager="jhubbard"
	editor="monicar" 
	tags="azure-service-management" />
<tags 
	ms.service="virtual-machines-windows"
	ms.date="04/14/2016"
	wacn.date="05/24/2016" />

# 在 Azure 中托管的 Web 应用中使用 ReportViewer

> [AZURE.IMPORTANT]Azure 具有用于创建和处理资源的两个不同的部署模型：[资源管理器和经典](/documentation/articles/resource-manager-deployment-model/)。本文介绍使用经典部署模型。Azure 建议大多数新部署使用资源管理器模型。

你可以使用 Visual Studio ReportViewer 控件构建 Azure Web 应用，该控件用于显示存储在 Azure 虚拟机上的报表。ReportViewer 控件位于使用 ASP.NET Web 应用模板生成的 Web 应用中。

>[AZURE.IMPORTANT]ASP.NET MVC Web 应用程序模板不支持 ReportViewer 控件。

若要将 ReportViewer 整合到你的 Azure Web 应用，需要完成以下任务。

- **添加**程序集到部署包

- **配置**身份验证和授权

- **发布** ASP.NET Web 应用程序到 Azure

## 先决条件

查看 [Azure 虚拟机中的 SQL Server Business Intelligence](/documentation/articles/virtual-machines-windows-classic-ps-sql-bi/) 中的“常规建议和最佳实践”部分。

>[AZURE.NOTE]ReportViewer 控件随 Visual Studio Standard Edition 或更高版本提供。如果你使用的是 Web Developer Express Edition，则必须安装 [MICROSOFT REPORT VIEWER 2012 RUNTIME](https://www.microsoft.com/download/details.aspx?id=35747) 才能使用 ReportViewer 运行时功能。
><p>在 Azure 中不支持在本地处理模式下配置的 ReportViewer。


## 将程序集添加到部署包

当在本地托管 ASP.NET 应用程序时，在 Visual Studio 安装过程中 ReportViewer 程序集通常直接安装在 IIS 服务器的全局程序集缓存 (GAC) 中，可以由应用程序直接访问。但是，当在云中托管 ASP.NET 应用程序时，Azure 不允许将任何内容安装到 GAC 中，因此你必须确保 ReportViewer 程序集在本地可供你的应用程序使用。你可以通过在你的项目中添加它们的引用并将它们配置为以本地方式复制来实现此操作。

在远程处理模式下，ReportViewer 控件使用以下程序集：

- **Microsoft.ReportViewer.WebForms.dll**：包含 ReportViewer 代码，你需要在页面中使用 ReportViewer。当你将 ReportViewer 控件拖到项目中的一个 ASP.NET 页时，此程序集的引用将被添加到该项目中。

- **Microsoft.ReportViewer.Common.dll**：包含 ReportViewer 控件在运行时使用的类。它不会自动添加到你的项目。

### 添加对 Microsoft.ReportViewer.Common 的引用

- 右键单击你的项目的“引用”节点，然后选择“添加引用”、在.NET 选项卡中选择程序集并单击“确定”。

### 使程序集可由 ASP.NET 应用程序从本地访问

1. 在 **References** 文件夹中，单击 Microsoft.ReportViewer.Common 程序集，使其属性显示在“属性”窗格中。

1. 在属性窗格中，将“复制本地”设置为 True。

1. 为 Microsoft.ReportViewer.WebForms 重复步骤 1 和 2。

### 获取 ReportViewer 语言包

1. 安装 [Microsoft Download Center](http://go.microsoft.com/fwlink/?LinkId=317386) 中的适当 Microsoft Report Viewer 2012 Runtime 可再发行组件包。

1. 从下拉列表中选择语言，页面将重定向到相应的下载中心页面。

1. 单击“下载”以开始下载 ReportViewerLP.exe。

1. 下载 ReportViewerLP.exe 后，单击“运行”立即安装，或单击“保存”将其保存到你的计算机。如果你单击“保存”，请记住保存该文件所在的文件夹的名称。

1. 找到保存该文件的文件夹。右键单击 ReportViewerLP.exe，单击“以管理员身份运行”，然后单击“是”。

1. 在运行 ReportViewerLP.exe 之后, 你将看到 c:\\windows\\assembly 具有资源文件 **Microsoft.ReportViewer.Webforms.Resources** 和 **Microsoft.ReportViewer.Common.Resources**。

### 为本地化 ReportViewer 控件进行配置

1. 按照上面的指定说明下载并安装 Microsoft Report Viewer 2012 Runtime 可再发行组件包。

1. 在项目中创建 <language> 文件夹并将关联的资源程序集文件复制到该位置。要复制的资源程序集文件为：**Microsoft.ReportViewer.Webforms.Resources.dll** 和 **Microsoft.ReportViewer.Common.Resources.dll**。选择资源程序集文件，并在属性窗格中将“复制到输出目录”设置为“**始终复制**”。

1. 为 Web 项目设置“区域性和 UI区域性”。有关如何为 ASP.NET 网页设置“区域性和 UI 区域性”的详细信息，请参阅[如何：为 ASP.NET 网页全球化设置区域性和 UI 区域性](https://msdn.microsoft.com/zh-cn/library/bz9tc508.aspx)。

## 配置身份验证和授权

ReportViewer 需要使用正确的凭据向报表服务器进行身份验证，并且凭据必须经报表服务器授权才能访问所需的报表。有关身份验证的信息，请查看白皮书 [Reporting Services 报表查看器控件和基于 Azure 虚拟机的报表服务器](https://msdn.microsoft.com/zh-cn/library/azure/dn753698.aspx)。

## 发布 ASP.NET Web 应用程序到 Azure

有关将 ASP.NET Web 应用程序发布到 Azure 的说明，请参阅 [Web Apps 和 ASP.NET 入门](/documentation/articles/web-sites-dotnet-get-started/)。

>[AZURE.IMPORTANT]如果在解决方案资源管理器中的快捷菜单中未显示添加Azure 部署项目或添加 Azure 云服务项目命令，你可能需要将该项目的目标框架更改为 .NET Framework 4。
><p>两个命令提供基本相同的功能。其中一个命令将显示在快捷菜单中，这取决于已安装的 Azure SDK 版本。

## 资源

[Microsoft 报表](https://msdn.microsoft.com/zh-cn/library/bb885185.aspx)

[Azure 虚拟机中的 SQL Server Business Intelligence](/documentation/articles/virtual-machines-windows-classic-ps-sql-bi/)

[使用 PowerShell 创建运行本机模式报表服务器的 Azure VM](/documentation/articles/virtual-machines-windows-classic-ps-sql-report/)

[Reporting Services 报表查看器控件和基于 Azure 虚拟机的报表服务器](http://download.microsoft.com/download/2/2/0/220DE2F1-8AB3-474D-8F8B-C998F7C56B5D/Reporting%20Services%20report%20viewer%20control%20and%20Azure%20VM%20based%20report%20servers.docx)

<!---HONumber=Mooncake_0104_2016-->