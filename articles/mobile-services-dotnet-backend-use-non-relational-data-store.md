<properties 
	pageTitle="使用非关系数据存储构建服务 | Windows Azure" 
	description="了解如何使用非关系数据存储（如 MongoDB 或 Azure 表存储）用于基于 .NET 的移动服务" 
	services="mobile-services" 
	documentationCenter="" 
	authors="mattchenderson" 
	manager="dwrede" 
	editor="mollybos"/>

<tags 
	ms.service="mobile-services" 
	ms.date="08/08/2015" 
	wacn.date="10/03/2015"/>

# 生成使用 MongoDB 而非 SQL 数据库作为存储的 .NET 后端移动服务

本主题说明如何将非关系数据存储用于 .NET 后端移动服务。在此教程中，你将要修改移动服务快速入门项目，以使用 MongoDB 而不是默认的 Azure SQL 数据库作为数据存储。

只有在完成[移动服务入门]或[将移动服务添加到现有应用程序]教程之后，才可以完成本教程。还需要将 MongoLab 服务添加到你的订阅。

## <a name="create-store"></a>创建 MongoLab 非关系存储

1. 在 [Azure 管理门户]中，单击“新建”，然后单击“应用商店”。

2. 单击“MongoLab”外接程序，然后完成向导以注册一个 MongoLab 帐户。

	<!-- 有关 MongoLab 的详细信息，请参阅 [MongoLab 外接程序页]。 -->

3. 设置帐户后，单击“连接信息”并复制连接字符串。

4. 在移动服务中，单击“配置”选项卡，向下滚动到“连接字符串”并输入新的连接字符串（其“名称”为 `MongoConnectionString`，其“值”为 MongoDB 连接），然后单击“保存”。

	![添加 MongoDB 连接字符串](./media/mobile-services-dotnet-backend-use-non-relational-data-store/mongo-connection-string.png)

	存储帐户连接字符串将以加密形式存储在应用设置中。你可以在运行时在任何表控制器中访问此字符串。

5. 在 Visual Studio 的解决方案资源管理器中，打开移动服务项目的 Web.config 文件，并添加以下新连接字符串：

		<add name="MongoConnectionString" connectionString="<MONGODB_CONNECTION_STRING>" />

6. 将 `<MONGODB_CONNECTION_STRING>` 占位符替换为 MongoDB 连接字符串。

	移动服务在本地计算机上运行时将使用此连接字符串，使你可以在发布之前测试代码。在 Azure 中运行时，移动服务将改用门户中设置的连接字符串值，并忽略项目中的连接字符串。

## <a name="modify-service"></a>修改数据类型和表控制器

1. 安装 **WindowsAzure.MobileServices.Backend.Mongo** NuGet 包。

2. 将 **TodoItem** 修改为派生自 **DocumentData** 而不是 **EntityData**。

        public class TodoItem : DocumentData
        {
            public string Text { get; set; }

            public bool Complete { get; set; }
        }

3. 在 **TodoItemController** 中，将 **Initialize** 方法替换为以下代码：

        protected override async void Initialize(HttpControllerContext controllerContext)
        {
            base.Initialize(controllerContext);
            string connectionStringName = "MongoConnectionString";
            string databaseName = "<YOUR-DATABASE-NAME>";
            string collectionName = "todoItems";

            DomainManager = new MongoDomainManager<TodoItem>(connectionStringName, databaseName,
	                        collectionName, Request, Services);
        }

4. 在上述 **Initialize** 方法的代码中，将 `<YOUR-DATABASE-NAME>` 替换为你在预配 MongoLab 外接程序时选择的名称。

现在，可以测试应用。

## <a name="test-application"></a>测试应用程序

1. （可选）重新发布移动服务 .NET 后端项目。

	你也可以先在本地测试移动服务，然后将 .NET 后端项目发布到 Azure。无论是在本地还是在 Azure 中测试，移动服务都使用 MongoDB 作为存储。

2. 与前面一样，使用启动页上的“立即试用”按钮，或使用连接到移动应用的客户端应用程序，来查询数据库中的项。
 
	 请注意，你将看不到前面从快速入门教程存储在 SQL 数据库中的任何项。

	>[AZURE.NOTE]当你使用“立即试用”按钮来启动帮助 API 页时，请记得提供应用程序密钥作为密码（用户名为空白）。
	

3. 创建新项。

	应用和移动服务的行为应如同以往，不过，数据现在将存储在非关系存储而不是 SQL 数据库中。

##后续步骤

现在你已知道，为 .NET 后端使用表存储是如此简单，接下来建议你了解其他一些后端存储选项：

+ [生成使用表存储而非 SQL 数据库的 .NET 后端移动服务](/documentation/articles/mobile-services-dotnet-backend-store-data-table-storage)</br>与你刚刚完成的教程一样，本主题说明如何将非关系数据存储用于移动服务。在此教程中，你将要修改移动服务快速入门项目，以使用 Azure 存储空间而不是 SQL 数据库作为数据存储。
 
+ [使用混合连接来连接到本地 SQL Server](/documentation/articles/mobile-services-dotnet-backend-hybrid-connections-get-started)</br>混合连接可让移动服务安全地连接到本地资产。这样，移动客户端便可以使用 Azure 访问你的本地数据。支持的资产包括静态 TCP 端口上运行的任何资源，例如 Microsoft SQL Server、MySQL、HTTP Web API 和大多数自定义 Web 服务。

+ [使用移动服务将图像上载到 Azure 存储空间](/documentation/articles/mobile-services-dotnet-backend-windows-store-dotnet-upload-data-blob-storage)</br>说明如何扩展 TodoList 示例项目，以便将图像从应用上载到 Azure Blob 存储。


<!-- Anchors. -->
[创建非关系存储]: #create-store
[修改数据和控制器]: #modify-service
[测试应用程序]: #test-application


<!-- Images. -->
[0]: ./media/mobile-services-dotnet-backend-use-non-relational-data-store/create-mongo-lab.png
[1]: ./media/mobile-services-dotnet-backend-use-non-relational-data-store/mongo-connection-string.png


<!-- URLs. -->
[移动服务入门]: /zh-cn/documentation/articles/mobile-services-dotnet-backend-windows-store-dotnet-get-started
[数据处理入门]: /zh-cn/documentation/articles/mobile-services-dotnet-backend-windows-store-dotnet-get-started-data
[Azure 管理门户]: https://manage.windowsazure.cn/
[什么是表服务]: /zh-cn/documentation/articles/storage-dotnet-how-to-use-tables/#what-is
[MongoLab 外接程序页]: /zh-cn/gallery/store/mongolab/mongolab

<!---HONumber=71-->