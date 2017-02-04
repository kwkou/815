<properties
   pageTitle="Publish-WebApplicationWebSite（Windows PowerShell 脚本）| Azure"
   description="了解如何将 Web 项目发布到 Azure 网站。如果资源不存在，则此脚本将在你的 Azure 订阅中创建所需的资源。"
   services="visual-studio-online"
   documentationCenter="na"
   authors="TomArcher"
   manager="douge"
   editor="" />

<tags
    ms.assetid="63cfaa2d-f04d-40dc-8677-345385c278d5"
    ms.service="multiple"
    ms.devlang="dotnet"
    ms.topic="article"
    ms.tgt_pltfrm="na"
    ms.workload="multiple"
    ms.date="11/11/2016"
    wacn.date="02/04/2017"
    ms.author="tarcher" />

# Publish-WebApplicationWebSite（Windows PowerShell 脚本）

##语法

将 Web 项目发布到 Azure 网站。如果资源不存在，则脚本将在你的 Azure 订阅中创建所需的资源。

	Publish-WebApplicationWebSite
	–Configuration <configuration>
	-SubscriptionName <subscriptionName>
	-WebDeployPackage <packageName>
	-DatabaseServerPassword @{Name = "name"; Password = "password"}
	-SendHostMessagesToOutput
	-Verbose


## 配置

描述部署的详细信息的 JSON 配置文件的路径。

|参数|默认值|
|---|---|
|别名|无|
|必需？|是|
|位置|名为|
|默认值|无|
|接受管道输入？|false|
|接受通配符？|false|

## SubscriptionName

你要在其中创建网站的 Azure 订阅的名称。

|参数|默认值|
|---|---|
|别名|无|
|必需？|false|
|位置|名为|
|默认值|无|
|接受管道输入？|false|
|接受通配符？|false|

## WebDeployPackage

要发布到网站的 Web 部署包的路径。可以在 Visual Studio 中使用“发布 Web”向导来创建此包。有关详细信息，请参阅 [Azure 云服务和 ASP.NET 入门](/documentation/articles/vs-azure-tools-publish-webapplicationwebsite-windows-powershell-script/)。

|参数|默认值|
|---|---|
|别名|无|
|必需？|false|
|位置|名为|
|默认值|无|
|接受管道输入？|false|
|接受通配符？|false|

## DatabaseServerPassword

Azure 中的 SQL 数据库的用户名和密码。

|参数|默认值|
|---|---|
|别名|无|
|必需？|false|
|位置|名为|
|默认值|无|
|接受管道输入？|false|
|接受通配符？|false|

## SendHostMessagesToOutput

如果为 true，则将消息从脚本打印到输出流。

|参数|默认值|
|---|---|
|别名|无|
|必需？|false|
|位置|名为|
|默认值|false|
|接受管道输入？|false|
|接受通配符？|false|

## 备注

JSON 配置文件指定要部署的内容的详细信息。它包括当你创建项目时指定的信息，如网站的名称和用户名。它还包括要预配的数据库（如果有的话）。以下代码显示一个示例 JSON 配置文件：

	{
	    "environmentSettings": {
	        "webSite": {
	            "name": "WebApplication10554",
	            "location": "China East"
	        },
	        "databases": [
	            {
	                "connectionStringName": "DefaultConnection",
	                "databaseName": "WebApplication10554_db",
	                "serverName": "iss00brc88",
	                "user": "sqluser2",
	                "password": "",
	                "edition": "",
	                "size": "",
	                "collation": "",
	                "location": "China East"
	            }
	        ]
	    }
	}

你可以编辑 JSON 配置文件以更改部署的内容。网站部分是必需的，但数据库部分则是可选的。

## 后续步骤

有关详细信息，请参阅[Publish-WebApplicationVM（Windows PowerShell 脚本）](/documentation/articles/vs-azure-tools-publish-webapplicationvm/)

<!---HONumber=Mooncake_0509_2016-->