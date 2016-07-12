<properties
	pageTitle="使用混合连接从 .NET 后端移动服务连接到本地 SQL Server | Azure 移动服务"
	description="了解如何使用 Azure 混合连接从 .NET 后端移动服务连接到本地 SQL Server"
	services="mobile-services"
	documentationCenter=""
	authors="ggailey777"
	manager="dwrede"
	editor=""/>

<tags 
	ms.service="mobile-services" 
	ms.date="03/05/2016"
	wacn.date="04/18/2016"/>

  
# 使用混合连接从 Azure 移动服务连接到本地 SQL Server 


当企业在过渡到云环境时，可能无法立即就将所有资产迁移到 Azure。使用混合连接，Azure 移动服务可以安全地连接到本地资产。这样，移动客户端便可以使用 Azure 访问你的本地数据。支持的资产包括静态 TCP 端口上运行的任何资源，例如 Microsoft SQL Server、MySQL、HTTP Web API 和大多数自定义 Web 服务。混合连接使用共享访问签名 (SAS) 授权，确保从移动服务和本地混合连接管理器到混合连接的连接安全。

在本教程中，你将学习如何修改 .NET 后端移动服务，以使用本地 SQL Server 数据库，而不是随服务一起提供的默认 Azure SQL 数据库。如[此文](http://blogs.msdn.com/b/azuremobile/archive/2014/05/12/connecting-to-an-external-database-with-node-js-backend-in-azure-mobile-services.aspx)中所述，JavaScript 后端移动服务也支持混合连接。


##先决条件##

本教程要求做好以下准备：

- **现有的 .NET 后端移动服务**<br/>遵循[移动服务入门]教程，从 [Azure 管理门户]创建和下载新的 .NET 后端移动服务。

[AZURE.INCLUDE [hybrid-connections-prerequisites](../includes/hybrid-connections-prerequisites.md)]

## 安装 SQL Server Express，启用 TCP/IP 并在本地创建一个 SQL Server 数据库

[AZURE.INCLUDE [hybrid-connections-create-on-premises-database](../includes/hybrid-connections-create-on-premises-database.md)]

## 创建混合连接

[AZURE.INCLUDE [hybrid-connections-create-new](../includes/hybrid-connections-create-new.md)]


## 安装本地混合连接管理器以完成连接


[AZURE.INCLUDE [hybrid-connections-install-connection-manager](../includes/hybrid-connections-install-connection-manager.md)]

## 配置移动服务项目以连接到 SQL Server 数据库

在此步骤中，定义本地数据库的连接字符串，并修改移动服务以使用此连接。

1. 在 Visual Studio 2013 中，打开用于定义 .NET 后端移动服务的项目。 

	若要了解如何下载 .NET 后端项目，请参阅[移动服务入门](/documentation/articles/mobile-services-dotnet-backend-windows-store-dotnet-get-started/)。

2. 在“解决方案资源管理器”中打开 Web.config 文件，找到 **connectionStrings** 节，添加类似于以下内容的新 SqlClient 项目，此项目指向本地 SQL Server 数据库：
	
	    <add name="OnPremisesDBConnection" 
         connectionString="Data Source=OnPremisesServer,1433;
         Initial Catalog=OnPremisesDB;
         User ID=HybridConnectionLogin;
         Password=<**secure_password**>;
         MultipleActiveResultSets=True"
         providerName="System.Data.SqlClient" />

	请记得将这个字符串中的 `<**secure_password**>` 替换成你为 *HbyridConnectionLogin* 创建的密码。
	
3. 在 Visual Studio 中单击“保存”以保存 Web.config 文件。

	> [AZURE.NOTE]在本地计算机上运行时，使用此连接设置。在 Azure 中运行时，则以门户中定义的连接设置覆盖此设置。

4. 展开“Models”文件夹并打开文件名以 *Context.cs* 结尾的数据模型文件。

5. 修改 **DbContext** 实例构造函数，以将 `OnPremisesDBConnection` 值传递到类似于以下代码段的基本 **DbContext** 构造函数：

        public class hybridService1Context : DbContext
        {
            public hybridService1Context()
                : base("OnPremisesDBConnection")
            {
            }
        }

	现在，服务将使用新的 SQL Server 数据库连接。
##在本地测试数据库连接

在发布到 Azure 并使用混合连接之前，最好先确保数据库连接能够在本地运行时正常工作。这样，你就可以更轻松地诊断并更正任何连接问题，然后重新发布并开始使用混合连接。

[AZURE.INCLUDE [mobile-services-dotnet-backend-test-local-service-api-documentation](../includes/mobile-services-dotnet-backend-test-local-service-api-documentation.md)]

## 更新 Azure 以使用本地连接字符串

在验证数据库连接后，需要为这个新的连接字符串添加应用设置，以便能够从 Azure 使用该连接字符串，并且能够将移动服务发布到 Azure。

1. 在 [Azure 管理门户]中，浏览到你的移动服务。

1. 单击“配置”选项卡，然后找到“连接字符串”部分。

	![本地数据库的连接字符串](./media/mobile-services-dotnet-backend-hybrid-connections-get-started/11.png)

3. 添加名为 `OnPremisesDBConnection` 的新 **SQL Server** 连接字符串，其值如下：

		Server=OnPremisesServer,1433;Database=OnPremisesDB;User ID=HybridConnectionsLogin;Password=<**secure_password**>


	将 `<**secure_password**>` 替换为 *HybridConnectionLogin* 的安全密码。

4. 按“保存”以保存混合连接以及刚刚创建的连接字符串。

5. 使用 Visual Studio 将更新的移动服务项目发布到 Azure。

	此时将显示服务启动页。

6. 与前面一样，使用启动页上的“立即试用”按钮，或使用连接到移动服务的客户端应用程序，调用可生成数据库更改的某些操作。

	>[AZURE.NOTE]当你使用“立即试用”按钮来启动帮助 API 页时，请记得提供应用程序密钥作为密码（用户名为空白）。

7. 在 SQL Server Management Studio 中连接到 SQL Server 实例，打开对象资源管理器，然后依次展开“OnPremisesDB”数据库和“表”。

8. 右键单击“hybridService1.TodoItems”表，然后选择“选择前 1000 行”以查看结果。

	请注意，移动服务已使用混合连接将应用中生成的更改保存到本地数据库。

##另请参阅##
 
+ [混合连接网站](http://azure.microsoft.com/zh-cn/services/biztalk-services/)
<!--+ [BizTalk 服务：“仪表板”、“监视”、“缩放”、“配置”和“混合连接”选项卡](/documentation/articles/biztalk-dashboard-monitor-scale-tabs/)-->
+ [如何对 .NET 后端移动服务进行数据模型更改](/documentation/articles/mobile-services-dotnet-backend-how-to-use-code-first-migrations/)

<!-- IMAGES -->

<!-- Links -->
[Azure 经典门户]: http://manage.windowsazure.cn
[移动服务入门]: /documentation/articles/mobile-services-dotnet-backend-windows-store-dotnet-get-started/

<!---HONumber=Mooncake_0118_2016-->