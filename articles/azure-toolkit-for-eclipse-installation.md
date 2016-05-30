<properties
	pageTitle="安装 Azure Toolkit for Eclipse"
	description="了解如何安装 Azure Toolkit for Eclipse。"
	services=""
	documentationCenter="java"
	authors="rmcmurray"
	manager="wpickett"
	editor=""/>

<tags
	ms.service="multiple"
    	ms.date="03/28/2016" 
	wacn.date="05/23/2016"/>

<!-- Legacy MSDN URL = https://msdn.microsoft.com/library/azure/hh690946.aspx -->

# 安装 Azure Toolkit for Eclipse #

使用 Azure Toolkit for Eclipse 提供的模板和功能，可轻松地利用 Eclipse 开发环境创建、开发、测试和部署 Azure 应用程序。它是一个开放源代码项目，其源代码可从 GitHub 上项目站点的 Apache 许可证 2.0 下获取，网址为：

<https://github.com/MSOpenTech/WindowsAzureToolkitForEclipseWithJava>。

以下步骤说明如何安装 Azure Toolkit for Eclipse。

[AZURE.INCLUDE [azure-toolkit-for-eclipse-prerequisites](../includes/azure-toolkit-for-eclipse-prerequisites.md)]

## 安装 Azure Toolkit for Eclipse ##

1. 启动 Eclipse。
2. 在 Eclipse 中的菜单上，依次单击“帮助”、“安装新软件”，如下图中所示。<strong></strong><strong></strong>![][ic590123]
3. 在“可用软件”对话框中的“使用”文本框中，键入 <strong>http://dl.msopentech.com/eclipse</strong>，然后按 <strong>Enter</strong> 键。<strong></strong><strong></strong>
4. 在“名称”窗格中，选中 Azure Toolkit for Eclipse，并取消选中“在安装过程中联系所有更新站点以查找所需的软件”。<strong></strong><strong></strong><strong></strong>你的屏幕应显示如下：![][ic719482]
5. 如果展开 <strong>Azure Toolkit for Eclipse</strong>，你将看到以下项：
    * **Azure 访问控制服务筛选器**：此组件为使用 Azure ACS 对应用程序用户进行身份验证提供支持。
    * **Azure 常用插件**：此组件包含其他组件依赖的共享功能。
    * **Azure Toolkit for Eclipse**：此组件包含项目配置逻辑、发布到云向导和用户界面。
    * **Microsoft JDBC Driver 4.0 for SQL Server**：此组件可简化使用 SQL 数据库的应用程序开发。
    * **Apache Qpid JMS 客户端库包**：此组件提供了 Apache Qpid 项目中的 JMS 客户端库，以允许应用程序在 Azure 中使用基于高级消息队列协议 (AMQP) 的消息传送
    * **Package for Azure Libraries for Java**：此组件允许你使用 Java 构建可利用 Azure 可缩放云计算资源的 Azure 应用程序。
    * **适用于 Java 的 Application Insights 插件**：此组件允许你将 Azure 的遥测日志记录和分析服务用于你的应用程序和服务器实例。
6. 单击**“下一步”**。（如果在安装该工具包时遇到不正常的延迟，请确保未选中“在安装过程中联系所有更新站点以查找所需的软件”。）
7. 在“安装详细信息”对话框中，单击“下一步”。
8. 在“查看许可证”对话框中，查看许可协议条款。如果你接受许可协议条款，请单击“我接受许可协议条款”，然后单击“完成”。（剩余步骤假定你接受许可协议条款。如果你不接受许可协议条款，请退出安装过程。）
9. 如果系统提示你重新启动 Eclipse 以完成安装，请单击“立即重新启动”。

## 另请参阅 ##

[适用于 Eclipse 的 Azure 工具包][]

[在 Eclipse 中为 Azure 创建 Hello World 应用程序][]

[Azure Toolkit for Eclipse 的新增功能][]

有关将 Azure 与 Java 配合使用的详细信息，请参阅 [Azure Java 开发人员中心][]。

<!-- URL List -->

[适用于 Eclipse 的 Azure 工具包]: /documentation/articles/azure-toolkit-for-eclipse/
[Azure Java 开发人员中心]: /develop/java/
[在 Eclipse 中为 Azure 创建 Hello World 应用程序]: /documentation/articles/azure-toolkit-for-eclipse-creating-a-hello-world-application/
[Installing the Azure Toolkit for Eclipse]: /documentation/articles/azure-toolkit-for-eclipse-installation/
[Web Platform Installer (WebPI)]: http://go.microsoft.com/fwlink/?LinkID=252838
[Azure Toolkit for Eclipse 的新增功能]: /documentation/articles/azure-toolkit-for-eclipse-whats-new/

<!-- IMG List -->

[ic590123]: ./media/azure-toolkit-for-eclipse-installation/ic590123.png
[ic719482]: ./media/azure-toolkit-for-eclipse-installation/ic719482.png

<!---HONumber=Mooncake_0215_2016-->