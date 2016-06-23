<properties 
	pageTitle="下载 Azure SDK for Java" 
	description="了解如何下载 Azure SDK for Java，并提供 Maven 项目的示例代码，以及 Azure Tookit for Eclipse 的基本安装步骤。" 
	services="" 
	documentationCenter="java" 
	authors="rmcmurray" 
	manager="wpickett" 
	editor=""/>

<tags 
	ms.service="multiple" 
	ms.date="05/04/2016" 
	wacn.date="06/20/2016"/>

# 下载 Azure SDK for Java

本文包含有关下载和安装适用于 Java 的 Azure 库的说明。

**注意：**将根据 [Apache 许可证 2.0 版][license]分发 Java 版 Azure 客户端库。

## 适用于 Java 的 Azure 库 – 手动下载

若要手动安装适用于 Java 的 Azure 库，请单击 <http://go.microsoft.com/fwlink/?LinkId=690320> 下载包含所有库及所有依赖项的 ZIP 文件。

将 zip 文件下载到计算机之后，请提取其内容，然后使用以下选项之一将 JAR 文件添加你的项目中：

* 在 Eclipse 中将 JAR 文件导入你的 Java 项目。

* 在 Eclipe 中为项目配置**生成路径**，以将该路径包含在 JAR 文件的路径中。

有关在 Eclipse 中设置生成路径的详细信息，请参阅 Eclipse 网站上的 [Java 生成路径]一文。

**注意：**有关许可证和其他信息，请参阅 ZIP 文件中的 license.txt 和 ThirdPartyNotices.txt 文件。

## 适用于 Java 的 Azure 库 - 使用 Maven 生成

### 步骤 1 - 将项目设置为使用 Maven 来生成

若要在 Eclipse 中创建使用适用于 Java 的 Azure 库的 Maven 项目，请遵循[适用于 Java 的 Azure 管理库入门][maven-getting-started]一文。

### 步骤 2 - 使用必要的依赖项配置 Maven 设置

将项目配置为使用 Maven 生成之后，可以使用类似于以下示例的语法，将必要的依赖项添加到 pom.xml 文件中。请注意，你无需添加以下示例中所列的每个依赖项，而只需添加项目所需的特定依赖项。

> [AZURE.NOTE] 在以下示例中的每个 `<version>` 元素中，将此示例中的“n.n.n”占位符替换为有效版本号，可从 [Maven 上的 Azure 库存储库]获取此版本号。

    <dependency>
        <groupId>com.microsoft.azure</groupId>
        <artifactId>azure-svc-mgmt</artifactId>
        <version>n.n.n</version>
    </dependency>
    <dependency>
        <groupId>com.microsoft.azure</groupId>
        <artifactId>azure-svc-mgmt-compute</artifactId>
        <version>n.n.n</version>
    </dependency>
    <dependency>
        <groupId>com.microsoft.azure</groupId>
        <artifactId>azure-svc-mgmt-network</artifactId>
        <version>n.n.n</version>
    </dependency>
    <dependency>
        <groupId>com.microsoft.azure</groupId>
        <artifactId>azure-svc-mgmt-sql</artifactId>
        <version>n.n.n</version>
    </dependency>
    <dependency>
        <groupId>com.microsoft.azure</groupId>
        <artifactId>azure-svc-mgmt-storage</artifactId>
        <version>n.n.n</version>
    </dependency>
    <dependency>
        <groupId>com.microsoft.azure</groupId>
        <artifactId>azure-svc-mgmt-websites</artifactId>
        <version>n.n.n</version>
    </dependency>
    <dependency>
        <groupId>com.microsoft.azure</groupId>
        <artifactId>azure-svc-mgmt-media</artifactId>
        <version>n.n.n</version>
    </dependency>
    <dependency>
        <groupId>com.microsoft.azure</groupId>
        <artifactId>azure-servicebus</artifactId>
        <version>n.n.n</version>
    </dependency>
    <dependency>
        <groupId>com.microsoft.azure</groupId>
        <artifactId>azure-serviceruntime</artifactId>
        <version>n.n.n</version>
    </dependency>

## 安装 Azure Toolkit for Eclipse

本部分包含有关安装 Azure Toolkit for Eclipse 的基本说明；有关详细说明，请参阅[安装 Azure Toolkit for Eclipse]。

### 先决条件

1. [Azure Toolkit for Eclipse 的新增功能]一文中所列的 Windows 操作系统。
1. [Azure Toolkit for Eclipse 的新增功能]一文中所列的 Macintosh 或 Linux 操作系统。
1. Eclipse IDE for Java EE Developers, Indigo 或更高版本。可以从 <http://www.eclipse.org/downloads/> 下载。

### 基本安装步骤

1. 在 Eclipse 中，从“帮助”菜单中选择“安装新软件”。
1. 输入站点位置 <http://dl.microsoft.com/eclipse>，然后按 **Enter**。
1. 选择要安装的项目，然后单击“完成”。

Azure Toolkit for Eclipse 使用最新版本的 Azure SDK。可使用 Web 平台安装程序 (WebPI) 从以下地址下载 <http://go.microsoft.com/fwlink/?LinkID=252838>。但是，如果你尚未安装 Azure SDK，则在你创建第一个 Azure 部署项目时，适用于 Eclipse 的 Azure 工具包将自动安装相应版本的 Azure SDK。

## 另请参阅

[适用于 Eclipse 的 Azure 工具包]

[安装 Azure Toolkit for Eclipse]

[在 Eclipse 中为 Azure 创建 Hello World 应用程序]

有关将 Azure 与 Java 配合使用的详细信息，请参阅 [Azure Java 开发人员中心]。

<!-- URL List -->

[Azure Java 开发人员中心]: /develop/java
[Maven 上的 Azure 库存储库]: http://go.microsoft.com/fwlink/?LinkID=286274
[适用于 Eclipse 的 Azure 工具包]: /documentation/articles/azure-toolkit-for-eclipse/
[在 Eclipse 中为 Azure 创建 Hello World 应用程序]: /documentation/articles/azure-toolkit-for-eclipse-creating-a-hello-world-application/
[安装 Azure Toolkit for Eclipse]: /documentation/articles/azure-toolkit-for-eclipse-installation/
[Java 生成路径]: http://help.eclipse.org/luna/index.jsp?topic=%2Forg.eclipse.jdt.doc.user%2Freference%2Fref-properties-build-path.htm
[license]: http://www.apache.org/licenses/LICENSE-2.0.html
[maven-getting-started]: http://go.microsoft.com/fwlink/?LinkID=622998
[zip-download]: http://go.microsoft.com/fwlink/?LinkId=690320
[Azure Toolkit for Eclipse 的新增功能]: /documentation/articles/azure-toolkit-for-eclipse-whats-new/

<!---HONumber=Mooncake_0405_2016-->