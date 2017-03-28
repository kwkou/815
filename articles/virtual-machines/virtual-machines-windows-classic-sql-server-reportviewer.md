<properties
    pageTitle="在网站中使用 ReportViewer | Azure"
    description="本主题介绍如何使用 Visual Studio ReportViewer 控件构建 Azure 网站，该控件用于显示 Azure 虚拟机上存储的报表。"
    services="virtual-machines-windows"
    documentationcenter="na"
    author="guyinacube"
    manager="erikre"
    editor="monicar"
    tags="azure-service-management" />
<tags
    ms.assetid="78b76318-d9bf-48ef-9d9e-d1b7d8cf3042"
    ms.service="virtual-machines-windows"
    ms.devlang="na"
    ms.topic="article"
    ms.tgt_pltfrm="vm-windows-sql-server"
    ms.workload="infrastructure-services"
    ms.date="10/04/2016"
    wacn.date="03/28/2017"
    ms.author="asaxton" />  


# 在 Azure 中托管的网站中使用 ReportViewer
> [AZURE.IMPORTANT] 
Azure 提供两个不同的部署模型用于创建和处理资源：[Resource Manager 模型和经典模型](/documentation/articles/resource-manager-deployment-model/)。本文介绍如何使用经典部署模型。Azure 建议大多数新部署使用 Resource Manager 模型。

可使用 Visual Studio ReportViewer 控件构建 Azure 网站，该控件用于显示 Azure 虚拟机上存储的报表。ReportViewer 控件位于使用 ASP.NET Web 应用程序模板生成的 Web 应用程序中。

> [AZURE.IMPORTANT]
ASP.NET MVC Web 应用程序模板不支持 ReportViewer 控件。
> 
> 

需要完成以下任务，才可将 ReportViewer 整合到 Azure 网站，。

* **添加**程序集到部署包
* **配置**身份验证和授权
* **发布** ASP.NET Web 应用程序到 Azure

## 先决条件
请查看 [Azure 虚拟机的 SQL Server Business Intelligence](/documentation/articles/virtual-machines-windows-classic-ps-sql-bi/) 中的“常规建议和最佳实践”部分。

> [AZURE.NOTE]
ReportViewer 控件随 Visual Studio Standard Edition 或更高版本提供。如果你使用的是 Web Developer Express Edition，则必须安装 [MICROSOFT REPORT VIEWER 2012 RUNTIME](https://www.microsoft.com/download/details.aspx?id=35747) 才能使用 ReportViewer 运行时功能。
><p> 
><p>Azure 中不支持在本地处理模式下配置的 ReportViewer。
> 
> 

## 将程序集添加到部署包
当在本地托管 ASP.NET 应用程序时，在 Visual Studio 安装过程中 ReportViewer 程序集通常直接安装在 IIS 服务器的全局程序集缓存 \(GAC\) 中，可以由应用程序直接访问。但在云中托管 ASP.NET 应用程序时，Azure 不允许将任何内容安装到 GAC 中，因此必须确保 ReportViewer 程序集可供应用程序本地使用。你可以通过在你的项目中添加它们的引用并将它们配置为以本地方式复制来实现此操作。

在远程处理模式下，ReportViewer 控件使用以下程序集：

* **Microsoft.ReportViewer.WebForms.dll**：包含 ReportViewer 代码，你需要在页面中使用 ReportViewer。当你将 ReportViewer 控件拖到项目中的一个 ASP.NET 页时，此程序集的引用将被添加到该项目中。
* **Microsoft.ReportViewer.Common.dll**：包含 ReportViewer 控件在运行时使用的类。它不会自动添加到你的项目。

### 添加对 Microsoft.ReportViewer.Common 的引用
* 右键单击项目的“引用”节点，选择“添加引用”，再在“.NET”选项卡中选择程序集并单击“确定”。

### 使程序集可由 ASP.NET 应用程序从本地访问
1. 在 **References** 文件夹中，单击 Microsoft.ReportViewer.Common 程序集，使其属性显示在“属性”窗格中。
2. 在属性窗格中，将“复制本地”设置为 True。
3. 为 Microsoft.ReportViewer.WebForms 重复步骤 1 和 2。

### 获取 ReportViewer 语言包
1. 安装 [Microsoft Download Center](http://go.microsoft.com/fwlink/?LinkId=317386) 中的适当 Microsoft Report Viewer 2012 Runtime 可再发行组件包。
2. 从下拉列表中选择语言，页面将重定向到相应的下载中心页面。
3. 单击“下载”以开始下载 ReportViewerLP.exe。
4. 下载 ReportViewerLP.exe 后，单击“运行”立即安装，或单击“保存”将其保存到你的计算机。如果你单击“保存”，请记住保存该文件所在的文件夹的名称。
5. 找到保存该文件的文件夹。右键单击 ReportViewerLP.exe，单击“以管理员身份运行”，然后单击“是”。
6. 在运行 ReportViewerLP.exe 之后, 你将看到 c:\\windows\\assembly 具有资源文件 **Microsoft.ReportViewer.Webforms.Resources** 和 **Microsoft.ReportViewer.Common.Resources**。

### 为本地化 ReportViewer 控件进行配置
1. 按照上面的指定说明下载并安装 Microsoft Report Viewer 2012 Runtime 可再发行组件包。
2. 在项目中创建 \<语言\> 文件夹，并将关联的资源程序集文件复制到此处。要复制的资源程序集文件为：**Microsoft.ReportViewer.Webforms.Resources.dll** 和 **Microsoft.ReportViewer.Common.Resources.dll**。选择资源程序集文件，并在“属性”窗格中将“复制到输出目录”设置为“始终复制”。
3. 为 Web 项目设置“区域性和 UI区域性”。有关如何为 ASP.NET 网页设置“区域性和 UI 区域性”的详细信息，请参阅[如何：为 ASP.NET 网页全球化设置区域性和 UI 区域性](https://msdn.microsoft.com/zh-cn/library/bz9tc508.aspx)。

## 配置身份验证和授权
ReportViewer 需要使用正确的凭据向报表服务器进行身份验证，并且凭据必须经报表服务器授权才能访问所需的报表。有关身份验证的信息，请参阅白皮书 [Reporting Services 报表查看器控件和基于 Azure 虚拟机的报表服务器](https://msdn.microsoft.com/zh-cn/library/azure/dn753698.aspx)。

## 发布 ASP.NET Web 应用程序到 Azure
有关将 ASP.NET Web 应用程序发布到 Azure 的说明，请参阅[如何：从 Visual Studio 将 Web 应用程序迁移和发布到 Azure](/documentation/articles/vs-azure-tools-migrate-publish-web-app-to-cloud-service/) 和 [Web Apps 和 ASP.NET 入门](/documentation/articles/web-sites-dotnet-get-started/)。

> [AZURE.IMPORTANT]
如果在解决方案资源管理器中的快捷菜单中未显示添加Azure 部署项目或添加 Azure 云服务项目命令，你可能需要将该项目的目标框架更改为 .NET Framework 4。
> <p>
> 两个命令提供基本相同的功能。其中一个命令将在快捷菜单中显示，具体取决于安装的 Azure SDK 版本。
> 
> 

## 资源
[Microsoft 报表](https://msdn.microsoft.com/zh-cn/library/bb885185.aspx)

[Azure 虚拟机中的 SQL Server Business Intelligence](/documentation/articles/virtual-machines-windows-classic-ps-sql-bi/)

[使用 PowerShell 创建运行本机模式报表服务器的 Azure VM](/documentation/articles/virtual-machines-windows-classic-ps-sql-report/)

[Reporting Services 报表查看器控件和基于 Azure 虚拟机的报表服务器](http://download.microsoft.com/download/2/2/0/220DE2F1-8AB3-474D-8F8B-C998F7C56B5D/Reporting%20Services%20report%20viewer%20control%20and%20Azure%20VM%20based%20report%20servers.docx)

<!---HONumber=Mooncake_0220_2017-->
<!--Update_Description: wording update-->