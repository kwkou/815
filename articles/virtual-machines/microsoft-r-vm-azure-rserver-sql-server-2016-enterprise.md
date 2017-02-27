<properties
    pageTitle="预配 Azure 上的 R Server Only SQL Server 2016 Enterprise VM"
    description="预配 Azure 上的 R Server Only SQL Server 2016 Enterprise VM"
    keywords="Microsoft R Server"
    services="virtual-machines-windows"
    documentationcenter=""
    tags=""
    author=""
    manager=""
    editor="" />
<tags
    ms.service="virtual-machines-windows"
    ms.workload=""
    ms.tgt_pltfrm="na"
    ms.devlang="na"
    ms.topic=""
    ms.date="01/22/2016"
    wacn.date=""
    ms.author="" />  


# 预配 Azure 上的 R Server Only SQL Server 2016 Enterprise VM

SQL Server 2016 和更高版本

更新时间：2016 年 7 月 22 日

适用于：SQL Server 2016

Azure 上的 R Server Only SQL Server 2016 Enterprise 虚拟机是可快速轻松地将服务器环境配置为支持 R 解决方案的一个新选项。此 Azure 虚拟机中已预配置 Microsoft R Server（独立版）。

此新虚拟机取代了以前在 Azure 应用商店中提供的适用于 Windows 虚拟机的 RRE。

此虚拟机还包含用于在应用程序和后端系统中部署 R 分析的 DeployR Enterprise。有关详细信息，请参阅 [About Deploy R](https://msdn.microsoft.com/microsoft-r/deployr-about)（关于 Deploy R）。

## 预配 R Server 虚拟机

如果你是 Azure VM 的新手，我们建议参阅本文，了解有关使用门户和配置虚拟机的详细信息。[虚拟机 - 入门](/documentation/services/virtual-machines/windows/)

从 Azure 应用商店创建 R Server VM

1. 单击“虚拟机”，然后在搜索框中键入 R Server。
2. 选择“R Server Only SQL Server 2016 Enterprise”
3. 根据以下文章中所述继续预配虚拟机：https://www.azure.cn/documentation/articles/virtual-machines-windows-hero-tutorial/
4. 创建并运行 VM 后，请单击“连接”按钮打开连接并登录到新计算机。
5. 连接时，默认会打开“服务器管理器”，但不需要进一步配置服务器。关闭“服务器管理器”转到桌面，然后继续执行后续步骤：
    * 安装其他 R 工具或开发工具
    * 配置 DeployR

在 Azure 经典管理门户中找到 R Server VM

1. 单击“虚拟机”，然后单击“新建”。
2. 在“新建”窗格中，“计算”和“虚拟机”应已选中。
3. 单击“从库中”查找 VM 映像。可在搜索框中键入 R Server；或者单击“Microsoft”并向下滚动，直到出现“R Server Only SQL Server 2016 Enterprise”。

## 安装 R 工具

默认情况下，Microsoft R Server 中包含随同基本 R 安装一起安装的所有 R 工具，其中包括 RTerm 和 RGui。桌面上已添加 RGui 的快捷方式，方便用户直接使用 R。

但是，你也可以安装其他 R 工具，例如 RStudio、用于 Visual Studio 的 R 工具或 Microsoft R Client。请参阅以下链接获取下载位置和说明：

* [用于 Visual Studio 的 R 工具](https://www.visualstudio.com/features/rtvs-vs.aspx)
* [Microsoft R Client](https://msdn.microsoft.com/microsoft-r/install-r-client-windows)
* [用于 Windows 的 RStudio](https://www.rstudio.com/products/rstudio/download/)

安装这些工具后，请确保将它们指向 R Server 库。

## 在虚拟机上使用 DeployR

需要执行一些附加步骤才能使用此 VM 上安装的 DeployR 实例。

若要配置 DeployR 实例，请执行以下操作：

1. 在 VM 上打开“DeployR 管理员实用工具”。
2. 根据 [Post Installation Steps](https://msdn.microsoft.com/microsoft-r/deployr-install-on-windows)（安装后步骤）中所述设置 DeployR 管理员密码
3. 设置 DeployR Web 上下文。有关详细信息，请参阅 [DeployR Admin Installation in the Cloud](https://msdn.microsoft.com/microsoft-r/deployr-admin-install-in-cloud)（云中的 DeployR 管理员安装）
4. 根据 [Configuring Azure Endpoints](https://msdn.microsoft.com/microsoft-r/deployr-admin-install-in-cloud#configuring-azure-endpoints)（配置 Azure 终结点）中所述，在 VM 上打开相应的端口
5. 根据 [Updating the firewall](https://msdn.microsoft.com/microsoft-r/deployr-admin-install-in-cloud#updating-the-firewall)（更新防火墙）中所述更新 Windows 防火墙

## 访问 Azure 存储帐户中的数据


需要使用 Azure 存储帐户中的数据时，可通过多个选项来访问或移动数据：

* 使用 [AzCopy](/documentation/articles/storage-use-azcopy/) 等实用工具将数据从存储帐户复制到本地文件系统。
* 将文件添加到存储帐户中的某个文件共享，然后将该文件共享装载为 VM 上的网络驱动器。有关详细信息，请参阅[装载 Azure 文件](/documentation/articles/storage-dotnet-how-to-use-files/)。

## 使用 Azure Data Lake Storage \(ADLS\) 帐户中的数据

可以使用 ScaleR 从 ADLS 存储中读取数据。为此，可以像使用 webHDFS 引用 HDFS 文件系统一样引用存储帐户。有关详细信息，请参阅此[设置指南](http://go.microsoft.com/fwlink/?LinkId=723452)。

## 资源

在 MSDSN 库中可以找到有关 Microsoft R 的其他文档：[R Server and Scale R](https://msdn.microsoft.com/microsoft-r)（R Server 和 Scale R）

请参阅以下附加资源，了解有关 R 的一般信息：

* [DataCamp](http://www.datacamp.com/)：提供有关 R 的免费临时简介课程，此外还包括一套有关使用 Revolution R 处理大数据的课程。
* [交叉验证](https://stats.stackexchange.com/)：可在该站点上针对机器学习中的统计问题提问。
* [R 帮助邮件列表存档](https://www.r-project.org/mail.html)：提供丰富的历史信息资源。
* [MRAN 网站](https://mran.microsoft.com/documents/getting-started/)：其他许多 R 资源。

## 另请参阅

[SQL Server R Services](https://msdn.microsoft.com/zh-cn/library/mt604845.aspx)

<!---HONumber=Mooncake_0213_2017-->