<properties
    pageTitle="适用于 DocumentDB 的 DocumentDB API 的 Node.js 教程 | Azure"
    description="本 Node.js 教程将创建采用 DocumentDB API 的 DocumentDB。"
    keywords="node.js 教程, 节点数据库"
    services="documentdb"
    documentationcenter="node.js"
    author="AndrewHoh"
    manager="jhubbard"
    editor="monicar" />
<tags
    ms.assetid="14d52110-1dce-4ac0-9dd9-f936afccd550"
    ms.service="documentdb"
    ms.workload="data-services"
    ms.tgt_pltfrm="na"
    ms.devlang="node"
    ms.topic="hero-article"
    ms.date="12/25/2016"
    wacn.date="05/31/2017"
    ms.author="anhoh"
    ms.translationtype="Human Translation"
    ms.sourcegitcommit="4a18b6116e37e365e2d4c4e2d144d7588310292e"
    ms.openlocfilehash="a2ad41441ae81efb465d68ce8e9aec9f23b95cc1"
    ms.contentlocale="zh-cn"
    ms.lasthandoff="05/19/2017" />

# <a name="nodejs-tutorial-documentdb-nodejs-console-application"></a>Node.js 教程：DocumentDB Node.js 控制台应用程序
> [AZURE.SELECTOR]
- [.NET](/documentation/articles/documentdb-get-started/)
- [.NET Core](/documentation/articles/documentdb-dotnetcore-get-started/)
- [用于 MongoDB 的 Node.js](/documentation/articles/documentdb-mongodb-samples/)
- [Node.js](/documentation/articles/documentdb-nodejs-get-started/)
- [Java](/documentation/articles/documentdb-java-get-started/)
- [C++](/documentation/articles/documentdb-cpp-get-started/)

欢迎使用 DocumentDB Node.js SDK 的 Node.js 教程！ 学习本教程后，你将拥有一个可创建并查询 DocumentDB 资源的控制台应用程序。

我们将介绍：

- 创建并连接到 DocumentDB 帐户
- 设置应用程序
- 创建节点数据库
- 创建集合
- 创建 JSON 文档
- 查询集合
- 替换文档
- 删除文档
- 删除节点数据库

没有时间？ 不必担心！ 可在 [GitHub](https://github.com/Azure-Samples/documentdb-node-getting-started)上获取完整的解决方案。 有关快速说明，请参阅 [获取完整解决方案](#GetSolution) 。

完成 Node.js 教程后，请使用位于本页顶部和底部的投票按钮向我们提供反馈。 如果你希望我们直接与你联系，欢迎将你的电子邮件地址附在评论中。

现在，让我们开始吧！

## <a name="prerequisites-for-the-nodejs-tutorial"></a>Node.js 教程的先决条件
请确保你具有以下内容：

- 有效的 Azure 帐户。 如果没有，可以注册 [Azure 试用版](/pricing/1rmb-trial/)。
    - 另外，对于本教程，也可以使用 [DocumentDB 模拟器](/documentation/articles/documentdb-nosql-local-emulator/)。
- [Node.js](https://nodejs.org/) 版本 v0.10.29 或更高版本。

## <a name="step-1-create-an-azure-documentdb-account"></a>步骤 1：创建 DocumentDB 帐户
创建一个 DocumentDB 帐户。 如果已经有想要使用的帐户，可以跳到 [安装 Node.js 应用程序](#SetupNode)。 如果使用 DocumentDB 模拟器，请遵循 [DocumentDB 模拟器](/documentation/articles/documentdb-nosql-local-emulator/)中的步骤设置该模拟器，然后直接跳到[安装 Node.js 应用程序](#SetupNode)。

[AZURE.INCLUDE [documentdb-create-dbaccount](../../includes/documentdb-create-dbaccount.md)]

## <a id="SetupNode"></a>步骤 2：安装 Node.js 应用程序
1. 打开你最爱的终端。
2. 找到你想要在其中保存 Node.js 应用程序的文件夹或目录。
3. 使用以下命令创建两个空的 JavaScript 文件：
   - Windows:
     - ```fsutil file createnew app.js 0```
     - ```fsutil file createnew config.js 0```
   - Linux/OS X：
     - ```touch app.js```
     - ```touch config.js```
4. 通过 npm 安装 documentdb 模块。 请使用以下命令：
   - ```npm install documentdb --save```

很好！ 既然已完成安装设置，让我们开始编写一些代码。

## <a id="Config"></a>步骤 3：设置应用的配置
在喜爱的文本编辑器中打开 ```config.js``` 。

然后，复制并粘贴以下代码片段，并将属性 ```config.endpoint``` 和 ```config.primaryKey``` 设置为 DocumentDB 终结点 URI 和主密钥。 这两项配置都可以在 [Azure 门户](https://portal.azure.cn)中找到。

![Node.js 教程 - Azure 门户的屏幕截图，显示了一个 DocumentDB 帐户，在“DocumentDB 帐户”边栏选项卡上突出显示了“ACTIVE”中心、“密钥”按钮，在“密钥”边栏选项卡上突出显示了 URI、主密钥、辅助密钥的值 - Node 数据库][keys]

    // ADD THIS PART TO YOUR CODE
    var config = {}

    config.endpoint = "~your DocumentDB endpoint uri here~";
    config.primaryKey = "~your primary key here~";

复制 ```database id```、```collection id``` 和 ```JSON documents``` 并将其粘贴到在其中设置了 ```config.endpoint``` 和 ```config.authKey``` 属性的 ```config``` 对象下面。 如果已有要在数据库中存储的数据，则可以使用 DocumentDB 的[数据迁移工具](/documentation/articles/documentdb-import-data/)而不是添加文档定义。

    config.endpoint = "~your DocumentDB endpoint uri here~";
    config.primaryKey = "~your primary key here~";

    // ADD THIS PART TO YOUR CODE
    config.database = {
        "id": "FamilyDB"
    };

    config.collection = {
        "id": "FamilyColl"
    };

    config.documents = {
        "Andersen": {
            "id": "Anderson.1",
            "lastName": "Andersen",
            "parents": [{
                "firstName": "Thomas"
            }, {
                    "firstName": "Mary Kay"
                }],
            "children": [{
                "firstName": "Henriette Thaulow",
                "gender": "female",
                "grade": 5,
                "pets": [{
                    "givenName": "Fluffy"
                }]
            }],
            "address": {
                "state": "WA",
                "county": "King",
                "city": "Seattle"
            }
        },
        "Wakefield": {
            "id": "Wakefield.7",
            "parents": [{
                "familyName": "Wakefield",
                "firstName": "Robin"
            }, {
                    "familyName": "Miller",
                    "firstName": "Ben"
                }],
            "children": [{
                "familyName": "Merriam",
                "firstName": "Jesse",
                "gender": "female",
                "grade": 8,
                "pets": [{
                    "givenName": "Goofy"
                }, {
                        "givenName": "Shadow"
                    }]
            }, {
                    "familyName": "Miller",
                    "firstName": "Lisa",
                    "gender": "female",
                    "grade": 1
                }],
            "address": {
                "state": "NY",
                "county": "Manhattan",
                "city": "NY"
            },
            "isRegistered": false
        }
    };


数据库、集合和文档定义将充当 DocumentDB ```database id```、```collection id``` 和文档的数据。

最后，导出 ```config``` 对象，以便在 ```app.js``` 文件中引用。

            },
            "isRegistered": false
        }
    };

    // ADD THIS PART TO YOUR CODE
    module.exports = config;

## <a id="Connect"></a> 步骤 4：连接到 DocumentDB 帐户
在文本编辑器中打开空的 ```app.js``` 文件。 复制并粘贴以下代码，以导入 ```documentdb``` 模块和新建的 ```config``` 模块。

    // ADD THIS PART TO YOUR CODE
    "use strict";

    var documentClient = require("documentdb").DocumentClient;
    var config = require("./config");
    var url = require('url');

复制并粘贴以下代码，以使用前面保存的 ```config.endpoint``` 和 ```config.primaryKey``` 来创建新的 DocumentClient。

    var config = require("./config");
    var url = require('url');

    // ADD THIS PART TO YOUR CODE
    var client = new documentClient(config.endpoint, { "masterKey": config.primaryKey });

现已获得用于初始化 documentdb 客户端的代码，接下来让我们看看如何使用 DocumentDB 资源。

## <a name="step-5-create-a-node-database"></a>步骤 5：创建节点数据库
复制并粘贴以下代码，以设置“找不到”消息的 HTTP 状态、数据库 URL 和集合 URL。 这些 URL 可让 DocumentDB 客户端查找正确的数据库和集合。

    var client = new documentClient(config.endpoint, { "masterKey": config.primaryKey });

    // ADD THIS PART TO YOUR CODE
    var HttpStatusCodes = { NOTFOUND: 404 };
    var databaseUrl = `dbs/${config.database.id}`;
    var collectionUrl = `${databaseUrl}/colls/${config.collection.id}`;

可以使用 **DocumentClient** 类的 [createDatabase](https://azure.github.io/azure-documentdb-node/DocumentClient.html) 函数创建[数据库](/documentation/articles/documentdb-resources/#databases/)。 数据库是跨集合分区的文档存储的逻辑容器。

复制并粘贴 **getDatabase** 函数，以便使用 ```config``` 对象中指定的 ```id``` 在 app.js 文件中创建新数据库。 该函数将检查是否不存在具有相同 ```FamilyRegistry``` ID 的数据库。 如果确实存在，我们将返回该数据库而不是创建新的。

    var collectionUrl = `${databaseUrl}/colls/${config.collection.id}`;

    // ADD THIS PART TO YOUR CODE
    function getDatabase() {
        console.log(`Getting database:\n${config.database.id}\n`);

        return new Promise((resolve, reject) => {
            client.readDatabase(databaseUrl, (err, result) => {
                if (err) {
                    if (err.code == HttpStatusCodes.NOTFOUND) {
                        client.createDatabase(config.database, (err, created) => {
                            if (err) reject(err)
                            else resolve(created);
                        });
                    } else {
                        reject(err);
                    }
                } else {
                    resolve(result);
                }
            });
        });
    }

在设置 **getDatabase** 函数的位置下面复制并粘贴以下代码以添加帮助器函数 **exit**，该函数将列显退出消息并调用 **getDatabase** 函数。

                } else {
                    resolve(result);
                }
            });
        });
    }

    // ADD THIS PART TO YOUR CODE
    function exit(message) {
        console.log(message);
        console.log('Press any key to exit');
        process.stdin.setRawMode(true);
        process.stdin.resume();
        process.stdin.on('data', process.exit.bind(process, 0));
    }

    getDatabase()
    .then(() => { exit(`Completed successfully`); })
    .catch((error) => { exit(`Completed with error ${JSON.stringify(error)}`) });

在终端中，找到 ```app.js``` 文件并运行命令：```node app.js```

祝贺你！ 你已成功创建了 DocumentDB 数据库。

## <a id="CreateColl"></a>步骤 6：创建集合
> [AZURE.WARNING]
> **CreateDocumentCollectionAsync** 将创建新的集合，它牵涉定价。 有关详细信息，请访问 [定价页](/pricing/details/documentdb/)。
> 
> 

可以使用 **DocumentClient** 类的 [createCollection](https://azure.github.io/azure-documentdb-node/DocumentClient.html) 函数创建[集合](/documentation/articles/documentdb-resources/#collections/)。 集合是 JSON 文档和相关联的 JavaScript 应用程序逻辑的容器。

将 **getCollection** 函数复制并粘贴到 app.js 文件中 **getDatabase** 函数下面，以便使用 ```config``` 对象中指定的 ```id``` 创建新集合。 同样，我们将首先检查以确保不存在具有相同 ```FamilyCollection``` ID 的集合。 如果确实存在，我们将返回该集合而不是创建新的。

                } else {
                    resolve(result);
                }
            });
        });
    }

    // ADD THIS PART TO YOUR CODE
    function getCollection() {
        console.log(`Getting collection:\n${config.collection.id}\n`);

        return new Promise((resolve, reject) => {
            client.readCollection(collectionUrl, (err, result) => {
                if (err) {
                    if (err.code == HttpStatusCodes.NOTFOUND) {
                        client.createCollection(databaseUrl, config.collection, { offerThroughput: 400 }, (err, created) => {
                            if (err) reject(err)
                            else resolve(created);
                        });
                    } else {
                        reject(err);
                    }
                } else {
                    resolve(result);
                }
            });
        });
    }

将以下代码复制并粘贴到对 **getDatabase** 的调用下面，以执行 **getCollection** 函数。

    getDatabase()

    // ADD THIS PART TO YOUR CODE
    .then(() => getCollection())
    // ENDS HERE

    .then(() => { exit(`Completed successfully`); })
    .catch((error) => { exit(`Completed with error ${JSON.stringify(error)}`) });

在终端中，找到 ```app.js``` 文件并运行命令：```node app.js```

祝贺你！ 你已成功创建 DocumentDB 集合。

## <a id="CreateDoc"></a>步骤 7：创建文档
可以使用 **DocumentClient** 类的 [createDocument](https://azure.github.io/azure-documentdb-node/DocumentClient.html) 函数创建[文档](/documentation/articles/documentdb-resources/#documents/)。 文档为用户定义的（任意）JSON 内容。 现在，你可以将文档插入 DocumentDB。

将 **getFamilyDocument** 函数复制并粘贴到 **getCollection** 函数下面，创建包含 ```config``` 对象中保存的 JSON 数据的文档。 同样，我们将首先检查以确保不存在具有相同 ID 的文档。

                } else {
                    resolve(result);
                }
            });
        });
    }

    // ADD THIS PART TO YOUR CODE
    function getFamilyDocument(document) {
        let documentUrl = `${collectionUrl}/docs/${document.id}`;
        console.log(`Getting document:\n${document.id}\n`);

        return new Promise((resolve, reject) => {
            client.readDocument(documentUrl, { partitionKey: document.district }, (err, result) => {
                if (err) {
                    if (err.code == HttpStatusCodes.NOTFOUND) {
                        client.createDocument(collectionUrl, document, (err, created) => {
                            if (err) reject(err)
                            else resolve(created);
                        });
                    } else {
                        reject(err);
                    }
                } else {
                    resolve(result);
                }
            });
        });
    };

将以下代码复制并粘贴到对 **getCollection** 的调用下面，以执行 **getFamilyDocument** 函数。

    getDatabase()
    .then(() => getCollection())

    // ADD THIS PART TO YOUR CODE
    .then(() => getFamilyDocument(config.documents.Andersen))
    .then(() => getFamilyDocument(config.documents.Wakefield))
    // ENDS HERE

    .then(() => { exit(`Completed successfully`); })
    .catch((error) => { exit(`Completed with error ${JSON.stringify(error)}`) });

在终端中，找到 ```app.js``` 文件并运行命令：```node app.js```

祝贺你！ 你已成功创建 DocumentDB 文档。

![Node.js 教程 - 说明帐户、数据库、集合和文档间层次关系的关系图 - 节点数据库](./media/documentdb-nodejs-get-started/node-js-tutorial-account-database.png)

## <a id="Query"></a>步骤 8：查询 DocumentDB 资源
DocumentDB 支持对存储在每个集合中的 JSON 文档进行[各种查询](/documentation/articles/documentdb-sql-query/)。 下面的示例代码演示你可针对集合中文档运行的查询。

将 **queryCollection** 函数复制并粘贴到 app.js 文件中的 **getFamilyDocument** 函数下面。 DocumentDB 支持类似 SQL 的查询，如下所示。 有关构建复杂查询的详细信息，请参阅 [Query Playground](https://www.documentdb.com/sql/demo)（查询演练）和[查询文档](/documentation/articles/documentdb-sql-query/)。

                } else {
                    resolve(result);
                }
            });
        });
    }

    // ADD THIS PART TO YOUR CODE
    function queryCollection() {
        console.log(`Querying collection through index:\n${config.collection.id}`);

        return new Promise((resolve, reject) => {
            client.queryDocuments(
                collectionUrl,
                'SELECT VALUE r.children FROM root r WHERE r.lastName = "Andersen"'
            ).toArray((err, results) => {
                if (err) reject(err)
                else {
                    for (var queryResult of results) {
                        let resultString = JSON.stringify(queryResult);
                        console.log(`\tQuery returned ${resultString}`);
                    }
                    console.log();
                    resolve(results);
                }
            });
        });
    };


下图说明了如何对你创建的集合调用 DocumentDB SQL 查询语法。

![Node.js 教程 - 说明查询的范围和含义的关系图 - 节点数据库](./media/documentdb-nodejs-get-started/node-js-tutorial-collection-documents.png)

查询中的 [FROM](/documentation/articles/documentdb-sql-query/#FromClause/) 关键字是可选的，因为 DocumentDB 查询的范围已限制为单个集合。 因此，“FROM Families f”可与“FROM root r”或者任何其他所选变量名进行交换。 默认情况下，DocumentDB 将推断你选择的 Families、root 或变量名，并默认引用当前集合。

将以下代码复制并粘贴到对 **getFamilyDocument** 的调用下面，以执行 **queryCollection** 函数。

    .then(() => getFamilyDocument(config.documents.Andersen))
    .then(() => getFamilyDocument(config.documents.Wakefield))

    // ADD THIS PART TO YOUR CODE
    .then(() => queryCollection())
    // ENDS HERE

    .then(() => { exit(`Completed successfully`); })
    .catch((error) => { exit(`Completed with error ${JSON.stringify(error)}`) });

在终端中，找到 ```app.js``` 文件并运行命令：```node app.js```

祝贺你！ 你已成功查询了 DocumentDB 文档。

## <a id="ReplaceDocument"></a>步骤 9：替换文档
DocumentDB 支持替换 JSON 文档。

将 **replaceFamilyDocument** 函数复制并粘贴到 app.js 文件中的 **queryCollection** 函数下面。

                    }
                    console.log();
                    resolve(result);
                }
            });
        });
    }

    // ADD THIS PART TO YOUR CODE
    function replaceFamilyDocument(document) {
        let documentUrl = `${collectionUrl}/docs/${document.id}`;
        console.log(`Replacing document:\n${document.id}\n`);
        document.children[0].grade = 6;

        return new Promise((resolve, reject) => {
            client.replaceDocument(documentUrl, document, (err, result) => {
                if (err) reject(err);
                else {
                    resolve(result);
                }
            });
        });
    };

将以下代码复制并粘贴到对 **queryCollection** 的调用下面，以执行 **replaceDocument** 函数。 此外，再次将此代码添加到对 **queryCollection** 的调用，以验证是否已成功更改文档。

    .then(() => getFamilyDocument(config.documents.Andersen))
    .then(() => getFamilyDocument(config.documents.Wakefield))
    .then(() => queryCollection())

    // ADD THIS PART TO YOUR CODE
    .then(() => replaceFamilyDocument(config.documents.Andersen))
    .then(() => queryCollection())
    // ENDS HERE

    .then(() => { exit(`Completed successfully`); })
    .catch((error) => { exit(`Completed with error ${JSON.stringify(error)}`) });

在终端中，找到 ```app.js``` 文件并运行命令：```node app.js```

祝贺你！ 你已成功替换了 DocumentDB 文档。

## <a id="DeleteDocument"></a>步骤 10：删除文档
DocumentDB 支持删除 JSON 文档。

将 **deleteFamilyDocument** 函数复制并粘贴到 **replaceFamilyDocument** 函数下面。

                else {
                    resolve(result);
                }
            });
        });
    };

    // ADD THIS PART TO YOUR CODE
    function deleteFamilyDocument(document) {
        let documentUrl = `${collectionUrl}/docs/${document.id}`;
        console.log(`Deleting document:\n${document.id}\n`);

        return new Promise((resolve, reject) => {
            client.deleteDocument(documentUrl, (err, result) => {
                if (err) reject(err);
                else {
                    resolve(result);
                }
            });
        });
    };

将以下代码复制并粘贴到对第二个 **queryCollection** 的调用下面，以执行 **deleteDocument** 函数。

    .then(() => queryCollection())
    .then(() => replaceFamilyDocument(config.documents.Andersen))
    .then(() => queryCollection())

    // ADD THIS PART TO YOUR CODE
    .then(() => deleteFamilyDocument(config.documents.Andersen))
    // ENDS HERE

    .then(() => { exit(`Completed successfully`); })
    .catch((error) => { exit(`Completed with error ${JSON.stringify(error)}`) });

在终端中，找到 ```app.js``` 文件并运行命令：```node app.js```

祝贺你！ 你已成功删除了 DocumentDB 文档。

## <a id="DeleteDatabase"></a>步骤 11：删除 Node 数据库
删除已创建的数据库将删除该数据库及其所有子资源（集合、文档等）。

将 **cleanup** 函数复制并粘贴到 **deleteFamilyDocument** 函数下面，删除数据库和所有子资源。

                else {
                    resolve(result);
                }
            });
        });
    };

    // ADD THIS PART TO YOUR CODE
    function cleanup() {
        console.log(`Cleaning up by deleting database ${config.database.id}`);

        return new Promise((resolve, reject) => {
            client.deleteDatabase(databaseUrl, (err) => {
                if (err) reject(err)
                else resolve(null);
            });
        });
    }

将以下代码复制并粘贴到对 **deleteFamilyDocument** 的调用下面，以执行 **cleanup** 函数。

    .then(() => deleteFamilyDocument(config.documents.Andersen))

    // ADD THIS PART TO YOUR CODE
    .then(() => cleanup())
    // ENDS HERE

    .then(() => { exit(`Completed successfully`); })
    .catch((error) => { exit(`Completed with error ${JSON.stringify(error)}`) });

## <a id="Run"></a>步骤 12：运行整个 Node.js 应用程序！
总体而言，调用函数的顺序应如下所示：

    getDatabase()
    .then(() => getCollection())
    .then(() => getFamilyDocument(config.documents.Andersen))
    .then(() => getFamilyDocument(config.documents.Wakefield))
    .then(() => queryCollection())
    .then(() => replaceFamilyDocument(config.documents.Andersen))
    .then(() => queryCollection())
    .then(() => deleteFamilyDocument(config.documents.Andersen))
    .then(() => cleanup())
    .then(() => { exit(`Completed successfully`); })
    .catch((error) => { exit(`Completed with error ${JSON.stringify(error)}`) });

在终端中，找到 ```app.js``` 文件并运行命令：```node app.js```

你应该看到已启动应用的输出。 输出应该匹配下面的示例文本。

    Getting database:
    FamilyDB

    Getting collection:
    FamilyColl

    Getting document:
    Anderson.1

    Getting document:
    Wakefield.7

    Querying collection through index:
    FamilyColl
        Query returned [{"firstName":"Henriette Thaulow","gender":"female","grade":5,"pets":[{"givenName":"Fluffy"}]}]

    Replacing document:
    Anderson.1

    Querying collection through index:
    FamilyColl
        Query returned [{"firstName":"Henriette Thaulow","gender":"female","grade":6,"pets":[{"givenName":"Fluffy"}]}]

    Deleting document:
    Anderson.1

    Cleaning up by deleting database FamilyDB
    Completed successfully
    Press any key to exit

祝贺你！ 你已完成 Node.js 教程，并成功创建了你的第一个 DocumentDB 控制台应用程序！

## <a id="GetSolution"></a>获取完整的 Node.js 教程解决方案
如果没有时间完成本教程中的步骤，或者只需下载代码，则可从 [GitHub](https://github.com/Azure-Samples/documentdb-node-getting-started) 获取。

若要生成包含本文所有示例的 GetStarted 解决方案，需要以下内容：

- [DocumentDB 帐户][documentdb-create-account]。
- GitHub 上提供的 [GetStarted](https://github.com/Azure-Samples/documentdb-node-getting-started) 解决方案。

通过 npm 安装 **documentdb** 模块。 请使用以下命令：

- ```npm install documentdb --save```

如[步骤 3：设置应用的配置](#Config)中所述，接下来在 ```config.js``` 文件中更新 config.endpoint 和 config.authKey 的值。 

然后在终端中找到 ```app.js``` 文件并运行命令：```node app.js```。

就这么简单，生成以后即可开始操作！ 

## <a name="next-steps"></a>后续步骤
- 想要更复杂的 Node.js 示例？ 请参阅[使用 DocumentDB 生成 Node.js Web 应用程序](/documentation/articles/documentdb-nodejs-application/)。
- 了解如何[监视 DocumentDB 帐户](/documentation/articles/documentdb-monitor-accounts/)。
- 在 [Query Playground](https://www.documentdb.com/sql/demo)中对示例数据集运行查询。
- 在 [DocumentDB 文档页](/documentation/services/documentdb/)的“开发”部分了解有关编程模型的详细信息。

[documentdb-create-account]: /documentation/articles/documentdb-create-account/
[keys]: ./media/documentdb-nodejs-get-started/node-js-tutorial-keys.png

<!--Update_Description: link and wording update-->