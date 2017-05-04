<properties
    pageTitle="将适用于 MongoDB 的 Robomongo 与 Azure DocumentDB 配合使用 | Azure"
    description="了解如何将 Robomongo 与 DocumentDB: API for MongoDB 帐户配合使用"
    keywords="robomongo"
    services="documentdb"
    author="AndrewHoh"
    manager="jhubbard"
    editor=""
    documentationcenter=""
    translationtype="Human Translation" />
<tags
    ms.assetid="352c5fb9-8772-4c5f-87ac-74885e63ecac"
    ms.service="documentdb"
    ms.workload="data-services"
    ms.tgt_pltfrm="na"
    ms.devlang="na"
    ms.topic="article"
    ms.date="03/06/2017"
    wacn.date="05/02/2017"
    ms.author="anhoh"
    ms.sourcegitcommit="75890c3ffb1d1757de64a8b8344e9f2569f26273"
    ms.openlocfilehash="a71a30e7e1ead2d13f316b09a1bdab7cd970f2c4"
    ms.lasthandoff="04/25/2017" />

# <a name="use-robomongo-with-a-documentdb-api-for-mongodb-account"></a>将 Robomongo 与 DocumentDB: API for MongoDB 帐户配合使用
若要使用 Robomongo 连接到 Azure DocumentDB: API for MongoDB 帐户，必须：

- 下载并安装 [Robomongo](https://robomongo.org/)
- 具有 DocumentDB：MongoDB 帐户的 API [连接字符串](/documentation/articles/documentdb-connect-mongodb-account/)信息

## <a name="connect-using-robomongo"></a>使用 Robomongo 进行连接
若要将 DocumentDB: API for MongoDB 帐户添加到 Robomongo MongoDB 连接，请执行以下步骤。

1. 使用[此处](/documentation/articles/documentdb-connect-mongodb-account/)的指令检索 DocumentDB: API for MongoDB 帐户连接信息。

    ![连接字符串边栏选项卡的屏幕截图](./media/documentdb-mongodb-robomongo/connectionstringblade.png)
2. 运行 *Robomongo.exe*

3. 单击“文件”下的“连接”按钮以管理连接。 然后，在“MongoDB 连接”窗口中单击“创建”，这会打开“连接设置”窗口。

4. 在“连接设置”窗口中，选择名称。 然后，从步骤 1 的连接信息中找到**主机**和**端口**，并将其分别输入到“地址”和“端口”中。

    ![Robomongo 管理连接的屏幕截图](./media/documentdb-mongodb-robomongo/manageconnections.png)
5. 在“身份验证”选项卡上，单击“执行身份验证”。 然后，输入数据库（默认值为 Admin）、**用户名**和**密码**。
**用户名**和**密码**可以在步骤 1 的连接信息中找到。

    ![Robomongo 身份验证选项卡的屏幕截图](./media/documentdb-mongodb-robomongo/authentication.png)
6. 在“SSL”选项卡上，选中“使用 SSL 协议”，然后将“身份验证方法”更改为“自签名证书”。

    ![Robomongo SSL 选项卡的屏幕截图](./media/documentdb-mongodb-robomongo/SSL.png)
7. 最后，单击“测试”验证是否能够连接，然后单击“保存”。

## <a name="next-steps"></a>后续步骤
- 浏览 MongoDB [示例](/documentation/articles/documentdb-mongodb-samples/)的 DocumentDB: API。

<!---Update_Description: wording update -->