<properties 
	pageTitle="连接到具有 MongoDB 协议支持的 DocumentDB 帐户 | Azure" 
	description="了解如何连接到具有 MongoDB 协议支持的 DocumentDB 帐户（目前以预览版提供）。使用 MongoDB 连接字符串进行连接。" 
	keywords="mongodb 连接字符串"
	services="documentdb" 
	authors="stephbaron" 
	manager="jhubbard" 
	editor="" 
	documentationCenter=""/>

<tags 
	ms.service="documentdb" 
	ms.workload="data-services" 
	ms.tgt_pltfrm="na" 
	ms.devlang="na" 
	ms.topic="article" 
	ms.date="08/23/2016" 
	ms.author="stbaro"
   	wacn.date="10/18/2016"/>  


# 如何连接到具有 MongoDB 协议支持的 DocumentDB 帐户

了解如何使用标准 MongoDB 连接字符串 URI 格式连接到具有 MongoDB 协议支持的 Azure DocumentDB 帐户。

## 获取帐户的连接字符串信息

1. 在新窗口中，登录到 [Azure 门户](https://portal.azure.cn)。
2. 在“帐户”边栏选项卡的**左导航**栏中，单击“连接字符串”。若要导航到“帐户”边栏选项卡，请在跳转栏上单击“更多服务”，单击“DocumentDB (NoSQL)”，然后选择具有 MongoDB 协议支持的 DocumentDB 帐户。

	![“所有设置”边栏选项卡的屏幕截图](./media/documentdb-connect-mongodb-account/SettingsBlade.png)  


3. 此时将打开“连接字符串信息”边栏选项卡，其中显示了使用 MongoDB 驱动程序连接到帐户所需的所有信息，包括预先构造的连接字符串。

	![连接字符串边栏选项卡的屏幕截图](./media/documentdb-connect-mongodb-account/ConnectionStringBlade.png)

## 连接字符串要求

请务必注意，DocumentDB 支持标准 MongoDB 连接字符串 URI 格式，但要满足几个具体的要求：DocumentDB 帐户需要通过 SSL 进行身份验证和安全通信。因此，连接字符串格式为：

	mongodb://username:password@host:port/[database]?ssl=true

上图显示了“连接字符串”边栏选项卡中提供此字符串值的位置。

- 用户名（必需）
	- DocumentDB 帐户名
- 密码（必需）
	- DocumentDB 帐户密码
- 主机（必需）
	- DocumentDB 帐户的 FQDN
- 端口（必需）
	- 10250
- 数据库（可选）
	- 连接使用的默认数据库
- ssl=true（必需）

以上面“连接字符串信息”中显示的帐户为例。有效的连接字符串为：
	
	mongodb://contoso123:<password@contoso123.documents.azure.com:10250/mydatabase?ssl=true

## 使用 MongoDB 的 C# 驱动程序进行连接
如前所述，所有 DocumentDB 帐户需要通过 SSL 进行身份验证和安全通信。尽管 MongoDB 连接字符串的 URI 格式支持 ssl=true 查询字符串参数，但使用 MongoDB C# 驱动程序需要在创建 MongoClient 时使用 MongoClientSettings 对象。以下代码片段演示在获取上述帐户信息后，如何连接到帐户并使用“任务”数据库。

	        MongoClientSettings settings = new MongoClientSettings();
            settings.Server = new MongoServerAddress("contoso123.documents.azure.com", 10250);
            settings.UseSsl = true;
            settings.SslSettings = new SslSettings();
            settings.SslSettings.EnabledSslProtocols = SslProtocols.Tls12;

            MongoIdentity identity = new MongoInternalIdentity("Tasks", "contoso123");
            MongoIdentityEvidence evidence = new PasswordEvidence("<password>");

            settings.Credentials = new List<MongoCredential>()
            {
                new MongoCredential("SCRAM-SHA-1", identity, evidence)
            };
            MongoClient client = new MongoClient(settings);
            var database = client.GetDatabase("Tasks",);
	

## 后续步骤


- 了解如何对具有 MongoDB 协议支持的 DocumentDB 帐户[使用 MongoChef](/documentation/articles/documentdb-mongodb-mongochef/)。
- 浏览具有 MongoDB 协议支持的 DocumentDB [示例](/documentation/articles/documentdb-mongodb-samples/)。

 

<!---HONumber=Mooncake_1010_2016-->
