<properties
    pageTitle="在 Eclipse 中显示 Azure Libraries Package for Java 的 Javadoc 内容"
    description="如何在 Eclipse 中显示 Azure Libraries 的 Javadoc 内容。"
    services=""
    documentationCenter="java"
    authors="rmcmurray"
    manager="wpickett"
    editor=""/>

<tags
    ms.service="multiple"
    ms.date="05/04/2016" 
    wacn.date="06/20/2016"/>

<!-- Legacy MSDN URL = https://msdn.microsoft.com/library/azure/hh698319.aspx -->

# 在 Eclipse 中显示 Azure Libraries Package for Java 的 Javadoc 内容 #

通过将 Javadoc 内容关联到 Azure Libraries for Java，可以在 Eclipse 环境中查看 Azure Libraries for Java 的 Javadoc 内容。以下步骤说明如何在 Eclipse 中使用此功能。

此过程假设你已在生成路径中添加了 Azure Libraries for Java。

## 在 Eclipse 中显示 Azure Libraries for Java 的 Javadoc 内容 ##

* 在 Eclipse 的项目资源管理器内，在项目的“引用库”部分中，打开 Azure Library for Java JAR 的上下文菜单。例如 **microsoft-windowsazure-api-0.1.0.jar**（根据你安装的版本，版本号可能不同）。
* 单击“属性”。
* 在“属性”对话框中的左窗格内，单击“Javadoc 位置”。此时将显示“Javadoc 位置”对话框。
* 你可以指定“Javadoc URL”或“存档中的 Javadoc”。
    * 如果你选择指定“Javadoc URL”，请使用类似于 **http://dl.windowsazure.com/javadoc** 或 **http://dl.windowsazure.com/storage/javadoc** 的 URL。
    * 如果你选择使用“存档中的 Javadoc”，则可以指定外部文件或工作区文件。
    做出选择并根据需要浏览/验证。以下示例会将 Azure Libraries for Java 关联到已本地下载至名为 **c:\\MyAzureJARs** 的文件夹的相应 Javadoc JAR。
    ![][ic553487]
* *可选*：单击“验证”。此处可能会显示 Javadoc JAR 的潜在问题。
* 单击**“确定”**。

与库关联后，Javadoc 内容应会显示在 Eclipse IDE 中。例如，如果 `blob` 在代码中定义为 `CloudBlockBlob` 类型，则当你在代码中键入 `blob.acquireLease` 时，会显示以下示例 Javadoc 内容：

![][ic553488]

## 另请参阅 ##

[适用于 Eclipse 的 Azure 工具包][]

[在 Eclipse 中为 Azure 创建 Hello World 应用程序][]

[安装 Azure Toolkit for Eclipse][]

有关将 Azure 与 Java 配合使用的详细信息，请参阅 [Azure Java 开发人员中心][]。

<!-- URL List -->

[Azure Java 开发人员中心]: /develop/java/
[适用于 Eclipse 的 Azure 工具包]: /documentation/articles/azure-toolkit-for-eclipse/
[在 Eclipse 中为 Azure 创建 Hello World 应用程序]: /documentation/articles/azure-toolkit-for-eclipse-creating-a-hello-world-application/
[安装 Azure Toolkit for Eclipse]: /documentation/articles/azure-toolkit-for-eclipse-installation/

<!-- IMG List -->

[ic553487]: ./media/azure-toolkit-for-eclipse-displaying-javadoc-content-for-azure-libraries/ic553487.png
[ic553488]: ./media/azure-toolkit-for-eclipse-displaying-javadoc-content-for-azure-libraries/ic553488.png

<!---HONumber=Mooncake_0215_2016-->