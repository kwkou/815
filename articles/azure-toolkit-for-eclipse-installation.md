<!-- Remove intelij, hello world for temp -->
<properties
	pageTitle="安装 Azure Toolkit for Eclipse | Azure"
	description="了解如何安装 Azure Toolkit for Eclipse。"
	services=""
	documentationCenter="java"
	authors="rmcmurray"
	manager="wpickett"
	editor=""/>

<tags
	ms.service="multiple"
	ms.date="06/24/2016" 
	wacn.date="08/01/2016"/>

<!-- Legacy MSDN URL = https://msdn.microsoft.com/library/azure/hh690946.aspx -->

# 安装 Azure Toolkit for Eclipse

使用 Azure Toolkit for Eclipse 提供的模板和功能，可轻松地利用 Eclipse 开发环境创建、开发、测试和部署 Azure 应用程序。Azure Toolkit for Eclipse 是一个开放源代码项目，其源代码可从 GitHub 上项目站点的 MIT 许可证下获取，URL 为：

<https://github.com/microsoft/azure-tools-for-java>

以下步骤说明如何安装 Azure Toolkit for Eclipse。

[AZURE.INCLUDE [azure-toolkit-for-eclipse-prerequisites](../../includes/azure-toolkit-for-eclipse-prerequisites.md)]

## 安装 Azure Toolkit for Eclipse

1. 启动 Eclipse。

1. Eclipse 打开后，单击 “帮助”菜单，然后单击“安装新软件”，如下图所示。

    ![安装 Azure Toolkit for Eclipse][01]

1. 在“可用软件”对话框的“使用”文本框中，键入**http://dl.microsoft.com/eclipse**，然后按 **Enter **键。

1. 在“名称”窗格中，选中“Azure Toolkit for Eclipse”，并取消选中“在安装过程中访问所有更新站点以查找所需的软件”。你的屏幕应与下图中所示类似：

    ![安装 Azure Toolkit for Eclipse][02]

1. 如果展开“Azure Toolkit for Eclipse”，你将看到以下项：

    * **适用于 Java 的 Application Insights 插件**：此组件允许你将 Azure 的遥测日志记录和分析服务用于应用程序和服务器实例。
    * **Azure 访问控制服务筛选器**：此组件对以下情况提供支持：向 Azure ACS 验证应用程序用户的身份，启用单一登录方案，以及从应用程序具体化标识逻辑。
    * **Azure 常用插件**：此组件提供其他工具包组件所需的常见功能。
    * **Azure Explorer for Eclipse**：此组件提供其他工具包组件所需的常见功能。
    * **Azure Plugin for Eclipse with Java**：此组件对以下情况提供支持：在 Eclipse 中，通过命令行开发可帮助构建、测试和部署适用于 Azure 云的 Java 应用程序的项目。
    * **Azure Web Apps Plugin with Java**：此组件在将 Java Web 应用程序部署到 Azure Web 应用容器时提供支持。
    * **Microsoft JDBC Driver 4.2 for SQL Server**：此组件提供适用于 SQL Server 的 JDBC API 以及适用于 Java Platform Enterprise Edition 8 的 Azure SQL 数据库。
    * **Apache Qpid JMS 客户端库包**：此组件提供 Apache Qpid 项目中的 JMS 客户端组件，以允许应用程序在 Azure 中使用 AMQP 消息传送。
    * **Azure Java 库包**：此组件提供用于访问 Azure 服务（例如存储空间、服务总线、服务运行时等等）的 API。

1. 单击**“下一步”**。（如果在安装该工具包时遇到不正常的延迟，请确保未选中“在安装过程中访问所有更新站点以查找所需的软件”。）

1. 在“安装详细信息”对话框中，单击“下一步”。

    ![查看安装详细信息][03]

1. 在“查看许可证”对话框中，查看许可协议条款。如果接受许可协议条款，请单击“我接受许可协议条款”，然后单击“完成”。（剩余步骤假定你接受许可协议条款。如果你不接受许可协议条款，请退出安装过程。）

    ![查看许可证][04]

    Eclipse 将下载并安装必要的包。

    ![安装进度][05]

1. 如果系统提示你重新启动 Eclipse 以完成安装，请单击“是”。

    ![重新启动提示][06]

## 另请参阅

有关 Azure Toolkits for Java IDE 的详细信息，请参阅以下链接：

- [适用于 Eclipse 的 Azure 工具包]
  - 安装 Azure Toolkit for Eclipse（本文）
  - [Azure Toolkit for Eclipse 的新增功能]

- [Azure Toolkit for IntelliJ]
  - [安装 Azure Toolkit for IntelliJ]

有关将 Azure 与 Java 配合使用的详细信息，请参阅 [Azure Java 开发人员中心]。

<!-- URL List -->

[适用于 Eclipse 的 Azure 工具包]: /documentation/articles/azure-toolkit-for-eclipse/
[Azure Toolkit for IntelliJ]: /documentation/articles/azure-toolkit-for-intellij/
[在 Eclipse 中创建 Azure 的 Hello World Web 应用]: /documentation/articles/app-service-web-eclipse-create-hello-world-web-app/
[在 IntelliJ 中创建 Azure 的 Hello World Web 应用]: /documentation/articles/app-service-web-intellij-create-hello-world-web-app/
[Installing the Azure Toolkit for Eclipse]: /documentation/articles/azure-toolkit-for-eclipse-installation
[安装 Azure Toolkit for IntelliJ]: /documentation/articles/azure-toolkit-for-intellij-installation/
[Azure Toolkit for Eclipse 的新增功能]: /documentation/articles/azure-toolkit-for-eclipse-whats-new/
[Azure Toolkit for IntelliJ 中的新增功能]: /documentation/articles/azure-toolkit-for-intellij-whats-new

[Azure Java 开发人员中心]: /develop/java/

<!-- IMG List -->

[01]: ./media/azure-toolkit-for-eclipse-installation/eclipse-installation-01.png
[02]: ./media/azure-toolkit-for-eclipse-installation/eclipse-installation-02.png
[03]: ./media/azure-toolkit-for-eclipse-installation/eclipse-installation-03.png
[04]: ./media/azure-toolkit-for-eclipse-installation/eclipse-installation-04.png
[05]: ./media/azure-toolkit-for-eclipse-installation/eclipse-installation-05.png
[06]: ./media/azure-toolkit-for-eclipse-installation/eclipse-installation-06.png

<!---HONumber=Mooncake_0725_2016-->
