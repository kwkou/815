<properties
    pageTitle="Azure 服务终结点"
    description="介绍 Azure Toolkit for Eclipse 中的 Azure 服务终结点设置。"
    services=""
    documentationCenter="java"
    authors="rmcmurray"
    manager="wpickett"
    editor=""/>

<tags
    ms.service="multiple"
    ms.date="05/04/2016" 
    wacn.date="06/13/2016"/>

<!-- Legacy MSDN URL = https://msdn.microsoft.com/library/azure/dn268600.aspx -->

# Azure 服务终结点 #

Azure 服务终结点确定你的应用程序是在全球 Azure 平台管理、中国 21Vianet 运营的 Azure 还是私有 Azure 平台中部署和管理。“服务终结点”对话框允许你指定要使用的服务终结点。若要打开“服务终结点”对话框，请在 Eclipse 中单击“窗口”，单击“首选项”，展开“Azure”，然后单击“服务终结点”。设置“活动集”字段可以确定要用于当前工作区中 Azure 项目的 Azure 服务终结点。

下面显示了“服务终结点”对话框。

![][ic719493]

## 设置服务终结点 ##

在“服务终结点”对话框中，执行以下操作之一：

* 如果要使用全球 Azure 平台，请从“活动集”下拉列表中选择“windowsazure.com”并单击“确定”。
* 如果要使用中国 21Vianet 运营的 Azure，请从“活动集”下拉列表中选择“windowsazure.cn (中国)”并单击“确定”。
* 如果要使用私有 Azure 平台：
    1. 单击“编辑”。
    2. 此时将打开一个对话框，通知你“服务终结点”对话框将会关闭，但首选项集文件将保持打开。单击**“确定”**。
    3. 在 preferencesets.xml 文件中创建一个新的 `preferenceset` 元素。对于此新元素，请创建 `name`、`blob`、`management`、`portalURL` 和 `publishsettings` 属性，并为其添加对应于私有 Azure 平台的值。可以将提供给现有 `preferenceset` 元素的值作为模板。**注意**：用于 `blob` 属性的值必须在 URL 中包含文本“blob”。
    4. 保存并关闭 preferencesets.xml。
    5. 重新打开“服务终结点”对话框。
    6. 从“活动集”下拉列表中，选择创建的活动集并单击“确定”。
    7. 创建私有 Azure 平台 `preferenceset` 元素后，可以通过在“服务终结点”对话框中单击“编辑”按钮来更改分配给它的值。你还可以根据需要创建多个私有 Azure 平台 `preferenceset` 元素。

## 另请参阅 ##

[适用于 Eclipse 的 Azure 工具包][]

[安装 Azure Toolkit for Eclipse][]

[在 Eclipse 中为 Azure 创建 Hello World 应用程序][]

有关将 Azure 与 Java 配合使用的详细信息，请参阅 [Azure Java 开发人员中心][]。

<!-- URL List -->

[Azure Java 开发人员中心]: /develop/java/
[适用于 Eclipse 的 Azure 工具包]: /documentation/articles/azure-toolkit-for-eclipse/
[在 Eclipse 中为 Azure 创建 Hello World 应用程序]: /documentation/articles/azure-toolkit-for-eclipse-creating-a-hello-world-application/
[安装 Azure Toolkit for Eclipse]: /documentation/articles/azure-toolkit-for-eclipse-installation/

<!-- IMG List -->

[ic719493]: ./media/azure-toolkit-for-eclipse-azure-service-endpoints/ic719493.png

<!---HONumber=Mooncake_0606_2016-->