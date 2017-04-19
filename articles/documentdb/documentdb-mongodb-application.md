<properties
    pageTitle="使用 MongoDB API 生成 DocumentDB Web 应用 | Azure"
    description="使用 MongoDB 的 DocumentDB API 创建联机数据库 Web 应用的 NoSQL 教程。"
    keywords="mongodb 示例"
    services="documentdb"
    author="AndrewHoh"
    manager="jhubbard"
    editor=""
    documentationcenter=""
    translationtype="Human Translation" />
    
<tags
    ms.assetid="61a2ab3a-2fc3-4d49-a263-ed87c66628f6"
    ms.service="documentdb"
    ms.workload="data-services"
    ms.tgt_pltfrm="na"
    ms.devlang="na"
    ms.topic="article"
    ms.date="03/06/2017"
    wacn.date="04/17/2017"
    ms.author="anhoh"
    ms.sourcegitcommit="7cc8d7b9c616d399509cd9dbdd155b0e9a7987a8"
    ms.openlocfilehash="8e1a8a407347a1c7e6a4967555561f5135838d49"
    ms.lasthandoff="04/07/2017" />

# <a name="web-application-development-with-documentdb-api-for-mongodb"></a>使用 DocumentDB：MongoDB 的 API 开发 Web 应用程序
> [AZURE.SELECTOR]
- [.NET](/documentation/articles/documentdb-dotnet-application/)
- [适用于 MongoDB 的 .NET](/documentation/articles/documentdb-mongodb-application/)
- [Node.js](/documentation/articles/documentdb-nodejs-application/)
- [Java](/documentation/articles/documentdb-java-application/)
- [Python](/documentation/articles/documentdb-python-application/)

此示例说明如何使用 .NET 生成 MongoDB Web 应用的 DocumentDB: API。

若要使用此示例，必须：

- [创建](/documentation/articles/documentdb-create-mongodb-account/) Azure DocumentDB：MongoDB 帐户的 API。
- 检索 MongoDB [连接字符串](/documentation/articles/documentdb-connect-mongodb-account/)信息。

可以参考[在 Azure 中创建连接到虚拟机上运行的 MongoDB 的 Web 应用](/documentation/articles/web-sites-dotnet-store-data-mongodb-vm/)教程（需做出少量的修改），快速设置一个连接到 DocumentDB：MongoDB 帐户的 API 的 MongoDB 应用程序（在本地或发布到 Azure Web 应用）。  

1. 请遵循该教程，不过需要做出一项修改。  将 Dal.cs 代码替换为以下内容：

        using System;
        using System.Collections.Generic;
        using System.Linq;
        using System.Web;
        using MyTaskListApp.Models;
        using MongoDB.Driver;
        using MongoDB.Bson;
        using System.Configuration;
        using System.Security.Authentication;

        namespace MyTaskListApp
        {
            public class Dal : IDisposable
            {
                //private MongoServer mongoServer = null;
                private bool disposed = false;

                // To do: update the connection string with the DNS name
                // or IP address of your server. 
                //For example, "mongodb://testlinux.chinacloudapp.cn
                private string connectionString = "mongodb://localhost:27017";
                private string userName = "<your user name>";
                private string host = "<your host>";
                private string password = "<your password>";

                // This sample uses a database named "Tasks" and a 
                //collection named "TasksList".  The database and collection 
                //will be automatically created if they don't already exist.
                private string dbName = "Tasks";
                private string collectionName = "TasksList";

                // Default constructor.        
                public Dal()
                {
                }

                // Gets all Task items from the MongoDB server.        
                public List<MyTask> GetAllTasks()
                {
                    try
                    {
                        var collection = GetTasksCollection();
                        return collection.Find(new BsonDocument()).ToList();
                    }
                    catch (MongoConnectionException)
                    {
                        return new List<MyTask>();
                    }
                }

                // Creates a Task and inserts it into the collection in MongoDB.
                public void CreateTask(MyTask task)
                {
                    var collection = GetTasksCollectionForEdit();
                    try
                    {
                        collection.InsertOne(task);
                    }
                    catch (MongoCommandException ex)
                    {
                        string msg = ex.Message;
                    }
                }

                private IMongoCollection<MyTask> GetTasksCollection()
                {
                    MongoClientSettings settings = new MongoClientSettings();
                    settings.Server = new MongoServerAddress(host, 10250);
                    settings.UseSsl = true;
                    settings.SslSettings = new SslSettings();
                    settings.SslSettings.EnabledSslProtocols = SslProtocols.Tls12;

                    MongoIdentity identity = new MongoInternalIdentity(dbName, userName);
                    MongoIdentityEvidence evidence = new PasswordEvidence(password);

                    settings.Credentials = new List<MongoCredential>()
                    {
                        new MongoCredential("SCRAM-SHA-1", identity, evidence)
                    };

                    MongoClient client = new MongoClient(settings);
                    var database = client.GetDatabase(dbName);
                    var todoTaskCollection = database.GetCollection<MyTask>(collectionName);
                    return todoTaskCollection;
                }

                private IMongoCollection<MyTask> GetTasksCollectionForEdit()
                {
                    MongoClientSettings settings = new MongoClientSettings();
                    settings.Server = new MongoServerAddress(host, 10250);
                    settings.UseSsl = true;
                    settings.SslSettings = new SslSettings();
                    settings.SslSettings.EnabledSslProtocols = SslProtocols.Tls12;

                    MongoIdentity identity = new MongoInternalIdentity(dbName, userName);
                    MongoIdentityEvidence evidence = new PasswordEvidence(password);

                    settings.Credentials = new List<MongoCredential>()
                    {
                        new MongoCredential("SCRAM-SHA-1", identity, evidence)
                    };
                    MongoClient client = new MongoClient(settings);
                    var database = client.GetDatabase(dbName);
                    var todoTaskCollection = database.GetCollection<MyTask>(collectionName);
                    return todoTaskCollection;
                }

                # region IDisposable

                public void Dispose()
                {
                    this.Dispose(true);
                    GC.SuppressFinalize(this);
                }

                protected virtual void Dispose(bool disposing)
                {
                    if (!this.disposed)
                    {
                        if (disposing)
                        {
                        }
                    }

                    this.disposed = true;
                }

                # endregion
            }
        }

2. 根据你的帐户设置在 Dal.cs 文件中修改以下变量：

        private string userName = "<your user name>";
        private string host = "<your host>";
        private string password = "<your password>";

3. 应用可供使用！

## <a name="next-steps"></a>后续步骤
- 了解如何配合 DocumentDB：MongoDB 帐户的 API [使用 MongoChef](/documentation/articles/documentdb-mongodb-mongochef/) 和[使用 RoboMongo](/documentation/articles/documentdb-mongodb-robomongo/)。

