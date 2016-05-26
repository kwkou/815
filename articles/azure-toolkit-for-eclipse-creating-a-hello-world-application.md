<properties
    pageTitle="在 Eclipse 中为 Azure 创建 Hello World 应用程序"
    description="了解如何使用 Azure Toolkit for Eclipse 创建一个简单的 Hello World 应用程序。"
    services=""
    documentationCenter="java"
    authors="rmcmurray"
    manager="wpickett"
    editor=""/>

<tags
    ms.service="multiple"
    ms.date="03/28/2016" 
    wacn.date="05/23/2016"/>

<!-- Legacy MSDN URL = https://msdn.microsoft.com/library/azure/hh690944.aspx -->

# 在 Eclipse 中为 Azure 创建 Hello World 应用程序 #

以下步骤说明如何使用 Azure Toolkit for Eclipse 创建基本 JSP 应用程序并将其部署到 Azure。所述 JSP 示例是一个简化的示例，但就 Azure 部署而言，非常相似的步骤也适用于 Java servlet。

该应用程序类似于：

![][ic600360]

## 先决条件 ##

* Java 开发人员工具包 (JDK) 1.7 或更高版本。
* Eclipse IDE for Java EE Developers, Indigo 或更高版本。可以从 <http://www.eclipse.org/downloads/> 下载。
* 基于 Java 的 Web 服务器或应用程序服务器的分发版，如 Apache Tomcat、GlassFish、JBoss 应用程序服务器、Jetty 或 IBM® WebSphere® Application Server Liberty Core。
* Azure 订阅，可以从 </pricing/purchase-options/> 获取。
* Azure Toolkit for Eclipse。有关详细信息，请参阅[安装 Azure Toolkit for Eclipse][]。

## 创建 Hello World 应用程序 ##

首先，我们将从创建 Java 项目开始。

*  启动 Eclipse。在 Eclipse 中的菜单上，依次单击“文件”、新建和“动态 Web 项目”。（如果在单击“文件”、“新建”后未看到“动态 Web 项目”作为可用项目列出，则执行以下操作：依次单击“文件”、“新建”、“项目...”，展开“Web”，单击“动态 Web 项目”，然后单击“下一步”。）
*  在本教程中，项目命名为 **MyHelloWorld**。（请务必使用此名称，因为本教程中的后续步骤会将你的 WAR 文件命名为 MyHelloWorld）。你的屏幕应与下图中所示类似：![][ic589576]
* 单击“完成”。
* 在 Eclipse 的项目资源管理器视图中，展开 **MyHelloWorld**。右键单击“WebContent”，单击“新建”，然后单击“JSP 文件”。
* 在“新建 JSP 文件”对话框中，将文件命名为 **index.jsp**。将父文件夹保留为 **MyHelloWorld/WebContent**，如下所示：![][ic659262]
* 对于本教程，请在“选择 JSP 模板”对话框中选择“新建 JSP 文件(html)”，然后单击“完成”。
* 在 Eclipse 中打开 index.jsp 文件后，添加文本以便在现有 `<body>` 元素中动态显示 **Hello World!**。更新后的 `<body>` 内容应与下图中所示类似：
```
    <body>
    <b><% out.println("Hello World!"); %></b>
    </body>
```
* 保存 index.jsp。

## 以快速简单的方式将应用程序部署到 Azure ##

当你的 Java Web 应用程序准备好进行测试时，可立即使用以下快捷方式直接在 Azure 云上试用一下。

1. 在 Eclipse 的项目资源管理器中，单击“MyHelloWorld”。
1. 在 Eclipse 工具栏中，单击“发布到 Azure 云”按钮 ![][ic710882]。
1. 如果你是首次将此应用程序发布到 Azure，并且以前尚未为此应用程序创建 Azure 部署项目，则将自动为你创建 Azure 部署项目。你将看到以下提示，其中还列出了将自动部署的用于运行你的应用程序的 JDK 包和应用程序服务器。![][ic789598] 使用此快捷方法可快速简便地在 Azure 中测试你的应用程序，而无需配置不同于默认设置的特定服务器或 JDK。如果你对默认设置感到满意，则可以单击“确定”以继续执行后续步骤。但是，如果你要更改要用于应用程序的 JDK 或应用程序服务器，则可以稍后通过编辑已为你自动创建的 Azure 部署项目来执行该操作，也可以立即单击“取消”并阅读本教程的“关于 Azure 部署项目”部分。
1. 在“发布到 Azure”对话框中执行以下操作：
    1. 如果“订阅”列表中还没有任何订阅可供选择，请按照下列步骤来导入订阅信息：
        1. 单击“从发布设置文件导入”。
        1. 在“导入订阅信息”对话框中，单击“下载发布设置文件”。如果你尚未登录到 Azure 帐户，则系统将提示你登录。然后系统将提示你保存 Azure 发布设置文件。请将该文件保存到本地计算机。
        1. 仍在“导入订阅信息”对话框中，单击“浏览”按钮，选择在上一步中保存到本地的发布设置文件，然后单击“打开”。你的屏幕应与下图中所示类似：![][ic644267]
        1. 单击**“确定”**。
    1. 对于“订阅”，请选择要用于部署的订阅。
    1. 对于“存储帐户”，请选择要使用的存储帐户，或者单击“新建”以创建新的存储帐户。
    1. 对于“服务名称”，请选择要使用的云服务，或者单击“新建”以创建新的云服务。
    1. 对于“目标 OS”，请选择要用于部署的操作系统版本。
    1. 在本教程中，对于“目标环境”，请选择“过渡”。（如果你已准备好部署到生产站点，则将该选项更改为“生产”。）
    1. 可选：如果你希望新部署自动覆盖以前的部署，请确保选中“覆盖以前的部署”。启用了此选项后，在发布到同一位置时将不会遇到“409 冲突”问题。请注意，“发布到 Azure”对话框包含“远程访问”部分。默认情况下，未启用远程访问，我们将不会为此示例启用它。若要启用远程访问，需输入远程登录时要使用的用户名和密码。有关远程访问的详细信息，请参阅[在 Eclipse 中为 Azure 部署启用远程访问][]。“发布到 Azure”对话框将如下所示：![][ic719488]
1. 单击“发布”以发布到过渡环境。当系统提示你执行完全生成时，单击“是”。第一次生成时可能需要花费几分钟时间。“Azure 活动日志”将显示在 Eclipse 的选项卡式视图部分中。![][ic719489] 你可以使用此日志以及**控制台**视图来查看部署进度。一种替代方法是登录到 [Azure 管理门户][]，然后使用“云服务”部分来监视状态。
1. 当你的部署已成功部署时，“Azure 活动日志”将显示状态为“已发布”。单击“已发布”（如下图所示），浏览器将打开你的部署的实例。![][ic719490]

由于这是部署到过渡环境，DNS 名称将采用 http://&lt;*guid*&gt;.cloudapp.net 的格式，并且 URL 将包含 DNS 名称加上你的应用程序的后缀。例如，http://447564652c20426f6220526f636b7321.cloudapp.net/MyHelloWorld。 （**MyHelloWorld** 部分区分大小写。） 如果你在 Azure 平台管理门户中（在管理门户的“云服务”部分中）单击部署名称，也可以看到 DNS 名称。

尽管本演练针对的是到过渡环境的部署，但到生产环境的部署也遵循相同的步骤，不同的是，在“发布到 Azure”对话框中，对于“目标环境”，选择“生产”而不是“过渡”。部署到生产环境会生成基于所选 DNS 名称的 URL，而不是用于过渡的 GUID。

>[AZURE.WARNING] 此时，你已将你的 Azure 应用程序部署到云中。但在继续进行之前，你应认识到，即使已部署的应用程序没有运行，也会继续累加订阅的计费时间。因此，从 Azure 订阅中删除不需要的部署极为重要。

## 关于 Azure 部署项目 ##

若要将一个或多个 Java 应用程序部署到 Azure，需要 Azure 部署项目。它充当你的应用程序要发布到 Azure 所需包装到的“包”的角色。

除了有关你的应用程序的信息外，Azure 部署项目还包含有关部署的其他关键组件的信息，其中最重要的是：用于运行 Web 应用程序的应用程序服务器容器和 Java 运行时。Azure 支持大量可选择的 Java 运行时和 Java 应用程序服务器可供你从中选择。

尽管出于教学目的此处使用的示例已极大地简化，但 Azure 部署项目还可以包含其他重要的配置信息，让你可以使用你的应用程序创建几乎任意复杂的可缩放、高度可用的多层云服务。你可以启用**会话相关性（“粘滞会话”）**、**快速缓存**、**远程调试**、**SSL 卸载**、**防火墙/端口路由**、**远程访问**以及其他大量的强大功能。

如果你已完成本教程上一节的内容（“以快速简单的方式将应用程序部署到 Azure”），你现在将在项目资源管理器中看到自动为你生成的名为“**MyHelloWorld\_onAzure**”的新 Azure 部署项目。

你也可以通过自己先创建一个空的 Azure 部署项目，然后向其添加到你的应用程序来开始学习本教程。这是较长的过程，但可让你从一开始就更好地控制初始配置。

若要从头开始创建新的 Azure 部署项目，请单击“新建 Azure 部署项目”按钮 ![][ic710876]。

无论你是使用现有的 Azure 部署项目，还是从头开始创建一个 Azure 部署项目，你都能够随时同样轻松地更改该项目的配置设置和组件（如 JDK 或应用程序服务器）。

若要在现有 Azure 部署项目中更改 JDK、应用程序服务器或应用程序列表，请执行以下操作：

1. 在项目资源管理器中展开项目节点（例如 **MyHelloWorld\_onAzure**）
2. 右键单击“WorkerRole1”
3. 展开上下文菜单中的“Azure”子菜单
4. 单击“服务器配置”

无论你是通过编辑现有的 Azure 部署项目（如上所示）还是通过从头开始创建新的 Azure 部署项目来开始这些服务器配置步骤，你都将看到相同类型的对话框，让你可以配置 JDK、服务器和应用程序组件。若要详细了解如何更改这些对话框中的设置（例如，更改 JDK、应用程序服务器，以及在部署中添加或删除应用程序），请参阅[服务器配置属性][]一文。

## 仅限 Windows：将应用程序部署到计算模拟器 ##

>[AZURE.NOTE] Azure 模拟器仅在 Windows 上可用。如果你使用的是非 Windows 操作系统，请跳过本部分。

如果已按前面所述步骤创建新的 Azure 部署项目（即，隐式地通过将应用程序发布到 Azure），则已为云（而非本地模拟）配置 JDK 和应用程序服务器。若要准备在本地模拟器中测试你的项目，请执行以下步骤：

1. 在 Eclipse 的项目资源管理器中，单击“MyHelloWorld\_onAzure”。
1. 右键单击“WorkerRole1”。
1. 展开上下文菜单中的“Azure”子菜单。
1. 单击“服务器配置”。
1. 在“JDK”选项卡上，检查是否为工具包预先配置了默认的本地 JDK。如果没有预先配置，或者你想要更改假定的默认值，请确保选中“使用此文件路径中的 JDK 进行本地测试”复选框，并指定你要使用的 JDK 安装位置。如果你想要更改此位置，请单击“浏览”按钮，然后使用浏览控件选择要使用的 JDK 的目录位置。
1. 单击“服务器”选项卡。
1. 在对话框底部的“本地服务器路径”文本框中，输入与对话框顶部“部署此类型的服务器”复选框下面选择的服务器类型和主要版本号匹配的本地安装服务器的路径。如果你想要使用不同的应用程序服务器类型或主要版本，请先更改该复选框下面的选定内容。
1. 单击**“确定”**。
1. 在 Eclipse 工具栏中，单击“在 Azure 模拟器中运行”按钮 ![][ic710879]。如果“在 Azure 模拟器中运行”按钮未启用，请确保在 Eclipse 的项目资源管理器中选择了 **MyHelloWorld\_onAzure**，并确保 Eclipse 的项目资源管理器作为当前窗口具有焦点。这将首先开始完全生成你的项目，然后在计算模拟器中启动 Java Web 应用程序。（请注意，根据你的计算机的性能特征，第一次生成可能需要几秒钟到几分钟时间，但后续生成的速度将变快。） 在完成第一个生成步骤后，Windows 用户帐户控制 (UAC) 将提示你是否允许此命令对你的计算机进行更改。单击**“是”**。

>[AZURE.IMPORTANT] 如果未看到 UAC 提示，请检查 Windows 任务栏是否有 UAC 图标，然后先单击该图标。有时 UAC 提示未显示为最前面的窗口，但只显示为任务栏图标。

1. 检查计算模拟器 UI 的输出，以确定你的项目是否存在任何问题。根据部署的内容，你的应用程序可能需要几分钟时间才能在计算模拟器中完成启动。
1. 启动浏览器并使用 URL `http://localhost:8080/MyHelloWorld` 作为地址（此 URL 的 `MyHelloWorld` 部分区分大小写）。你应看到 MyHelloWorld 应用程序（index.jsp 的输出）与下图类似：![][ic589579]

当你准备使应用程序停止在计算模拟器中运行时，请在 Eclipse 工具栏中，单击“重置 Azure 模拟器”按钮 ![][ic710880]。

## 删除部署 ##

若要在 Azure Toolkit for Eclipse 中删除你的部署，请确保在 Eclipse 的项目资源管理器中选择“MyHelloWorld\_onAzure”，确保 Eclipse 项目资源管理器具有当前窗口焦点，然后单击 Eclipse 工具栏中的“取消发布”按钮 ![][ic710883]。（可以通过在 Eclipse 的项目资源管理器中右键单击“MyHelloWorld\_onAzure”，单击 “Azure”，然后单击“从 Azure 云取消部署”来执行相同的操作。） 这将显示“取消发布 Azure 项目”对话框。

![][ic719491]

选择包含你的部署的订阅和云服务，选择要删除的部署，然后单击“取消发布”。

（使用工具包来删除部署的替代方法是使用 Azure 管理门户的“云服务”部分：导航到你的部署，选择它，然后单击“删除”按钮。这将停止并删除该部署。如果你只需要停止该部署但不删除它，请单击“停止”按钮而非“删除”按钮，但如上所述，如果未删除该部署，则会继续针对该部署累加计费，即使该部署已停止）。

## 另请参阅 ##

[适用于 Eclipse 的 Azure 工具包][]

[安装 Azure Toolkit for Eclipse][]

[Azure Toolkit for Eclipse 的新增功能][]

有关将 Azure 与 Java 配合使用的详细信息，请参阅 [Azure Java 开发人员中心][]。

<!-- URL List -->

[Azure Java 开发人员中心]: /develop/java/
[Azure 管理门户]: http://manage.windowsazure.cn
[Azure Role Properties]: /documentation/articles/azure-toolkit-for-eclipse-azure-role-properties/
[适用于 Eclipse 的 Azure 工具包]: /documentation/articles/azure-toolkit-for-eclipse/
[在 Eclipse 中为 Azure 部署启用远程访问]: /documentation/articles/azure-toolkit-for-eclipse-enabling-remote-access-for-azure-deployments/
[安装 Azure Toolkit for Eclipse]: /documentation/articles/azure-toolkit-for-eclipse-installation/
[服务器配置属性]: /documentation/articles/azure-toolkit-for-eclipse-azure-role-properties/#server_configuration_properties
[Azure Toolkit for Eclipse 的新增功能]: /documentation/articles/azure-toolkit-for-eclipse-whats-new/

<!-- IMG List -->

[ic589576]: ./media/azure-toolkit-for-eclipse-creating-a-hello-world-application/ic589576.png
[ic589579]: ./media/azure-toolkit-for-eclipse-creating-a-hello-world-application/ic589579.png
[ic600360]: ./media/azure-toolkit-for-eclipse-creating-a-hello-world-application/ic600360.png
[ic644267]: ./media/azure-toolkit-for-eclipse-creating-a-hello-world-application/ic644267.png
[ic659262]: ./media/azure-toolkit-for-eclipse-creating-a-hello-world-application/ic659262.png
[ic710876]: ./media/azure-toolkit-for-eclipse-creating-a-hello-world-application/ic710876.png
[ic710879]: ./media/azure-toolkit-for-eclipse-creating-a-hello-world-application/ic710879.png
[ic710880]: ./media/azure-toolkit-for-eclipse-creating-a-hello-world-application/ic710880.png
[ic710882]: ./media/azure-toolkit-for-eclipse-creating-a-hello-world-application/ic710882.png
[ic710883]: ./media/azure-toolkit-for-eclipse-creating-a-hello-world-application/ic710883.png
[ic719488]: ./media/azure-toolkit-for-eclipse-creating-a-hello-world-application/ic719488.png
[ic719489]: ./media/azure-toolkit-for-eclipse-creating-a-hello-world-application/ic719489.png
[ic719490]: ./media/azure-toolkit-for-eclipse-creating-a-hello-world-application/ic719490.png
[ic719491]: ./media/azure-toolkit-for-eclipse-creating-a-hello-world-application/ic719491.png
[ic789598]: ./media/azure-toolkit-for-eclipse-creating-a-hello-world-application/ic789598.png

<!---HONumber=Mooncake_0215_2016-->