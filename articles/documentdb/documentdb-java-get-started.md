<properties
    pageTitle="NoSQL 教程：Azure DocumentDB Java SDK | Azure"
    description="使用 DocumentDB Java SDK 创建联机数据库和 Java 控制台应用程序的 NoSQL 教程。Azure DocumentDB 是用于 JSON 的 NoSQL 数据库。"
    keywords="nosql 教程, 联机数据库, java 控制台应用程序"
    services="documentdb"
    documentationcenter="Java"
    author="arramac"
    manager="jhubbard"
    editor="monicar" />
<tags
    ms.assetid="75a9efa1-7edd-4fed-9882-c0177274cbb2"
    ms.service="documentdb"
    ms.workload="data-services"
    ms.tgt_pltfrm="na"
    ms.devlang="java"
    ms.topic="hero-article"
    ms.date="01/05/2017"
    wacn.date="02/27/2017"
    ms.author="arramac" />  


# NoSQL 教程：构建 DocumentDB Java 控制台应用程序
> [AZURE.SELECTOR]
- [.NET](/documentation/articles/documentdb-get-started/)
- [.NET Core](/documentation/articles/documentdb-dotnetcore-get-started/)
- [Java](/documentation/articles/documentdb-java-get-started/)
- [Node.js](/documentation/articles/documentdb-nodejs-get-started/)
- [C++](/documentation/articles/documentdb-cpp-get-started/)

欢迎使用 Azure DocumentDB Java SDK 的 NoSQL 教程！ 学习本教程后，你将拥有一个可创建并查询 DocumentDB 资源的控制台应用程序。

我们介绍：

- 创建并连接到 DocumentDB 帐户
- 配置 Visual Studio 解决方案
- 创建联机数据库
- 创建集合
- 创建 JSON 文档
- 查询集合
- 创建 JSON 文档
- 查询集合
- 替换文档
- 删除文档
- 删除数据库

现在，让我们开始吧！

## 先决条件
确保具有以下内容：

- 有效的 Azure 帐户。如果没有，可以注册[试用版](/pricing/1rmb-trial/)。或者，可以在本教程中使用 [Azure DocumentDB 模拟器](/documentation/articles/documentdb-nosql-local-emulator/)。
- [Git](https://git-scm.com/downloads)
- [Java 开发工具包 \(JDK\) 7+](http://www.oracle.com/technetwork/java/javase/downloads/index.html)。
- [Maven](http://maven.apache.org/download.cgi)。

## 第 1 步：创建 DocumentDB 帐户
让我们创建一个 DocumentDB 帐户。如果已经有想要使用的帐户，可以跳到[克隆 Github 项目](#GitClone)。如果要使用 DocumentDB 模拟器，请按照 [Azure DocumentDB 模拟器](/documentation/articles/documentdb-nosql-local-emulator/)中的步骤设置模拟器，并跳到[克隆 Github 项目](#GitClone)。

[AZURE.INCLUDE [documentdb-create-dbaccount](../../includes/documentdb-create-dbaccount.md)]

## <a id="GitClone"></a>步骤 2：克隆 Github 项目
入门时，可以克隆适用于 [Get Started with DocumentDB and Java](https://github.com/Azure-Samples/documentdb-java-getting-started)（DocumentDB 和 Java 入门）的 Github 存储库。例如，从本地目录运行以下命令，在本地检索示例项目。

    git clone git@github.com:Azure-Samples/documentdb-java-getting-started.git

    cd documentdb-java-getting-started

该目录包含适用于项目的 `pom.xml`，以及内含 Java 源代码（包括 `Program.java`，演示如何使用 Azure DocumentDB 执行简单的操作，例如创建文档和查询集合中的数据）的 `src` 文件夹。`pom.xml` 包括一个依赖项，依赖于 [Maven 上的 DocumentDB Java SDK](https://mvnrepository.com/artifact/com.microsoft.azure/azure-documentdb)。

    <dependency>
        <groupId>com.microsoft.azure</groupId>
        <artifactId>azure-documentdb</artifactId>
        <version>LATEST</version>
    </dependency>

## <a id="Connect"></a>第 3 步：连接到 DocumentDB 帐户
接下来，回到 [Azure 门户预览](https://portal.azure.cn)，检索终结点和主要主密钥。DocumentDB 终结点和主密钥是必需的，可让应用程序知道要连接的对象，让 DocumentDB 信任应用程序的连接。

在 Azure 门户预览中，导航到 DocumentDB 帐户，然后单击“密钥”。从门户复制 URI，并将其粘贴到 Program.java 文件的 `<your endpoint URI>` 中。然后从门户复制主密钥，并将其粘贴到 `<your key>` 中。

    this.client = new DocumentClient(
        "<your endpoint URI>",
        "<your key>"
        , new ConnectionPolicy(),
        ConsistencyLevel.Session);

![NoSQL 教程创建 Java 控制台应用程序时使用的 Azure 门户预览的屏幕截图。显示 DocumentDB 帐户，在“DocumentDB 帐户”边栏选项卡上突出显示“ACTIVE”中心、“键”按钮，在“键”边栏选项卡上突出显示 URI、主键、辅键的值][keys]  


## 第 4 步：创建数据库
可以通过使用 **DocumentClient** 类的 [createDatabase](http://azure.github.io/azure-documentdb-java/com/microsoft/azure/documentdb/DocumentClient.html#createDatabase-com.microsoft.azure.documentdb.Database-com.microsoft.azure.documentdb.RequestOptions-) 方法来创建 DocumentDB [数据库](/documentation/articles/documentdb-resources/#databases/)。数据库是跨集合分区的 JSON 文档存储的逻辑容器。

    Database database = new Database();
    database.setId("familydb");
    this.client.createDatabase(database, null);

## <a id="CreateColl"></a>步骤 5：创建集合
> [AZURE.WARNING]
**createCollection** 将创建一个具有保留吞吐量的新集合，它牵涉定价。有关更多详细信息，请访问[定价页](/pricing/details/documentdb/)。
> 
> 

可以通过使用 **DocumentClient** 类的 [createCollection](http://azure.github.io/azure-documentdb-java/com/microsoft/azure/documentdb/DocumentClient.html#createCollection-java.lang.String-com.microsoft.azure.documentdb.DocumentCollection-com.microsoft.azure.documentdb.RequestOptions-) 方法创建[集合](/documentation/articles/documentdb-resources/#collections/)。集合是 JSON 文档和相关联的 JavaScript 应用程序逻辑的容器。


    DocumentCollection collectionInfo = new DocumentCollection();
    collectionInfo.setId("familycoll");

    // DocumentDB collections can be reserved with throughput specified in request units/second. 
    // Here we create a collection with 400 RU/s.
    RequestOptions requestOptions = new RequestOptions();
    requestOptions.setOfferThroughput(400);

    this.client.createCollection("/dbs/familydb", collectionInfo, requestOptions);

## <a id="CreateDoc"></a>步骤 6：创建 JSON 文档
可以通过使用 **DocumentClient** 类的 [createDocument](http://azure.github.io/azure-documentdb-java/com/microsoft/azure/documentdb/DocumentClient.html#createDocument-java.lang.String-java.lang.Object-com.microsoft.azure.documentdb.RequestOptions-boolean-) 方法创建[文档](/documentation/articles/documentdb-resources/#documents/)。文档为用户定义的（任意）JSON 内容。现在，我们可以插入一个或多个文档。如果已有要在数据库中存储的数据，则可以使用 DocumentDB 的[数据迁移工具](/documentation/articles/documentdb-import-data/)，将数据导入数据库。

    // Insert your Java objects as documents 
    Family andersenFamily = new Family();
    andersenFamily.setId("Andersen.1");
    andersenFamily.setLastName("Andersen")

    // More initialization skipped for brevity. You can have nested references
    andersenFamily.setParents(new Parent[] { parent1, parent2 });
    andersenFamily.setDistrict("WA5");
    Address address = new Address();
    address.setCity("Seattle");
    address.setCounty("King");
    address.setState("WA");

    andersenFamily.setAddress(address);
    andersenFamily.setRegistered(true);

    this.client.createDocument("/dbs/familydb/colls/familycoll", family, new RequestOptions(), true);

![说明 NoSQL 教程创建 Java 控制台应用程序所用帐户、联机数据库、集合和文档的层次关系的图表](./media/documentdb-get-started/nosql-tutorial-account-database.png)  


## <a id="Query"></a>步骤 7：查询 DocumentDB 资源
DocumentDB 支持对存储在每个集合中的 JSON 文档进行各种[查询](/documentation/articles/documentdb-sql-query/)。以下示例代码演示如何使用 SQL 语法通过 [queryDocuments](http://azure.github.io/azure-documentdb-java/com/microsoft/azure/documentdb/DocumentClient.html#queryDocuments-java.lang.String-com.microsoft.azure.documentdb.SqlQuerySpec-com.microsoft.azure.documentdb.FeedOptions-) 方法查询 DocumentDB 中的文档。

    FeedResponse<Document> queryResults = this.client.queryDocuments(
        "/dbs/familydb/colls/familycoll",
        "SELECT * FROM Family WHERE Family.lastName = 'Andersen'", 
        null);

    System.out.println("Running SQL query...");
    for (Document family : queryResults.getQueryIterable()) {
        System.out.println(String.format("\tRead %s", family));
    }

## <a id="ReplaceDocument"></a>步骤 8：替换 JSON 文档
DocumentDB 支持使用 [replaceDocument](http://azure.github.io/azure-documentdb-java/com/microsoft/azure/documentdb/DocumentClient.html#replaceDocument-com.microsoft.azure.documentdb.Document-com.microsoft.azure.documentdb.RequestOptions-) 方法更新 JSON 文档。

    // Update a property
    andersenFamily.Children[0].Grade = 6;

    this.client.replaceDocument(
        "/dbs/familydb/colls/familycoll/docs/Andersen.1", 
        andersenFamily,
        null);

## <a id="DeleteDocument"></a>步骤 9：删除 JSON 文档
同样，DocumentDB 支持使用 [deleteDocument](http://azure.github.io/azure-documentdb-java/com/microsoft/azure/documentdb/DocumentClient.html#deleteDocument-java.lang.String-com.microsoft.azure.documentdb.RequestOptions-) 方法删除 JSON 文档。

    this.client.delete("/dbs/familydb/colls/familycoll/docs/Andersen.1", null);

## <a id="DeleteDatabase"></a>步骤 10：删除数据库
删除已创建的数据库将删除该数据库及其所有子资源（集合、文档等）。

    this.client.deleteDatabase("/dbs/familydb", null);

## <a id="Run"></a>步骤 11：运行整个 Java 控制台应用程序！
若要从控制台运行应用程序，请首先使用 Maven 进行编译：
    
    mvn package

运行 `mvn package` 即可从 Maven 下载最新的 DocumentDB 库并生成 `GetStarted-0.0.1-SNAPSHOT.jar`。然后，通过运行以下命令来运行该应用：

    mvn exec:java -D exec.mainClass=GetStarted.Program

祝贺你！ 你已经完成本 NoSQL 教程，并且获得了一个可正常使用的 Java 控制台应用程序！

## 后续步骤
- 需要 Java Web 应用教程？ 请参阅[使用 DocumentDB 通过 Java 构建 Web 应用程序](/documentation/articles/documentdb-java-application/)。
- 了解如何[监视 DocumentDB 帐户](/documentation/articles/documentdb-monitor-accounts/)。
- 在 [Query Playground](https://www.documentdb.com/sql/demo) 中对示例数据集运行查询。
- 在 [DocumentDB 文档页](/documentation/services/documentdb/)的“Develop”（开发）部分中了解有关编程模型的详细信息。

[documentdb-create-account]: /documentation/articles/documentdb-create-account/
[keys]: media/documentdb-get-started/nosql-tutorial-keys.png

<!---HONumber=Mooncake_0220_2017-->
