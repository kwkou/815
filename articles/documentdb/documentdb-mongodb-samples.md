<properties
    pageTitle="使用 MongoDB API 生成 DocumentDB 应用 | Azure"
    description="使用 MongoDB 的 DocumentDB API 创建联机数据库的 NoSQL 教程。"
    keywords="mongodb 示例"
    services="documentdb"
    author="AndrewHoh"
    manager="jhubbard"
    editor=""
    documentationcenter=""
    translationtype="Human Translation" />
    
<tags
    ms.assetid="fb38bc53-3561-487d-9e03-20f232319a87"
    ms.service="documentdb"
    ms.workload="data-services"
    ms.tgt_pltfrm="na"
    ms.devlang="na"
    ms.topic="article"
    ms.date="03/06/2017"
    wacn.date="04/17/2017"
    ms.author="anhoh"
    ms.sourcegitcommit="7cc8d7b9c616d399509cd9dbdd155b0e9a7987a8"
    ms.openlocfilehash="de12bc89e5334d64b9dc8bab58eb9df167a6d60d"
    ms.lasthandoff="04/07/2017" />

# <a name="build-a-documentdb-api-for-mongodb-app-using-nodejs"></a>使用 Node.js 生成 DocumentDB：MongoDB 应用的 API
> [AZURE.SELECTOR]
- [.NET](/documentation/articles/documentdb-get-started/)
- [.NET Core](/documentation/articles/documentdb-dotnetcore-get-started/)
- [Java](/documentation/articles/documentdb-java-get-started/)
- [用于 MongoDB 的 Node.js](/documentation/articles/documentdb-mongodb-samples/)
- [Node.js](/documentation/articles/documentdb-nodejs-get-started/)
- [C++](/documentation/articles/documentdb-cpp-get-started/)

此示例说明如何使用 Node.js 生成 DocumentDB：MongoDB 控制台应用的 API。

若要使用此示例，必须：

- [创建](/documentation/articles/documentdb-create-mongodb-account/) Azure DocumentDB：MongoDB 帐户的 API。
- 检索 MongoDB [连接字符串](/documentation/articles/documentdb-connect-mongodb-account/)信息。

## <a name="create-the-app"></a>创建应用程序

1. 创建 app.js 文件，并复制粘贴以下代码。

        var MongoClient = require('mongodb').MongoClient;
        var assert = require('assert');
        var ObjectId = require('mongodb').ObjectID;
        var url = 'mongodb://<endpoint>:<password>@<endpoint>.documents.azure.com:10250/?ssl=true';

        var insertDocument = function(db, callback) {
        db.collection('families').insertOne( {
                "id": "AndersenFamily",
                "lastName": "Andersen",
                "parents": [
                    { "firstName": "Thomas" },
                    { "firstName": "Mary Kay" }
                ],
                "children": [
                    { "firstName": "John", "gender": "male", "grade": 7 }
                ],
                "pets": [
                    { "givenName": "Fluffy" }
                ],
                "address": { "country": "USA", "state": "WA", "city": "Seattle" }
            }, function(err, result) {
            assert.equal(err, null);
            console.log("Inserted a document into the families collection.");
            callback();
        });
        };

        var findFamilies = function(db, callback) {
        var cursor =db.collection('families').find( );
        cursor.each(function(err, doc) {
            assert.equal(err, null);
            if (doc != null) {
                console.dir(doc);
            } else {
                callback();
            }
        });
        };

        var updateFamilies = function(db, callback) {
        db.collection('families').updateOne(
            { "lastName" : "Andersen" },
            {
                $set: { "pets": [
                    { "givenName": "Fluffy" },
                    { "givenName": "Rocky"}
                ] },
                $currentDate: { "lastModified": true }
            }, function(err, results) {
            console.log(results);
            callback();
        });
        };

        var removeFamilies = function(db, callback) {
        db.collection('families').deleteMany(
            { "lastName": "Andersen" },
            function(err, results) {
                console.log(results);
                callback();
            }
        );
        };

        MongoClient.connect(url, function(err, db) {
        assert.equal(null, err);
        insertDocument(db, function() {
            findFamilies(db, function() {
            updateFamilies(db, function() {
                removeFamilies(db, function() {
                    db.close();
                });
            });
            });
        });
        });

2. 按照帐户设置修改 app.js 文件中的以下变量（了解如何查找[连接字符串](/documentation/articles/documentdb-connect-mongodb-account/)）：

        var url = 'mongodb://<endpoint>:<password>@<endpoint>.documents.azure.com:10250/?ssl=true';

3. 打开偏爱的终端，运行 **npm install mongodb --save**，然后使用 **node app.js** 运行应用程序

## <a name="next-steps"></a>后续步骤
- 了解如何配合 DocumentDB：MongoDB 帐户的 API [使用 MongoChef](/documentation/articles/documentdb-mongodb-mongochef/)。

<!-- Update_Description: wording update -->