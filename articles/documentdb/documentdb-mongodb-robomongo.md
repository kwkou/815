<properties
    pageTitle="将 Robomongo 与具有 MongoDB 协议支持的 Azure DocumentDB 帐户配合使用 | Azure"
    description="了解如何将 Robomongo 与具有 MongoDB 协议支持的 DocumentDB 帐户（目前以预览版提供）配合使用。"
    keywords="robomongo"
    services="documentdb"
    author="AndrewHoh"
    manager="jhubbard"
    editor=""
    documentationcenter="" />
<tags
    ms.assetid="352c5fb9-8772-4c5f-87ac-74885e63ecac"
    ms.service="documentdb"
    ms.workload="data-services"
    ms.tgt_pltfrm="na"
    ms.devlang="na"
    ms.topic="article"
    ms.date="11/29/2016"
    wacn.date="01/12/2017"
    ms.author="anhoh" />  


# 将 Robomongo 与具有 MongoDB 协议支持的 DocumentDB 帐户配合使用
若要使用 Robomongo 连接到具有 MongoDB 协议支持的 Azure DocumentDB 帐户，必须：

- 下载并安装 [Robomongo](https://robomongo.org/)
- 提供具有 MongoDB 协议支持的 DocumentDB 帐户的[连接字符串](/documentation/articles/documentdb-connect-mongodb-account/)信息

## 使用 Robomongo 进行连接
若要将具有 MongoDB 协议支持的 DocumentDB 帐户添加到 Robomongo MongoDB 连接，请执行以下步骤。

1. 使用[此处](/documentation/articles/documentdb-connect-mongodb-account/)的指令检索具有 MongoDB 协议支持的 DocumentDB 的连接信息。

    ![连接字符串边栏选项卡的屏幕截图](./media/documentdb-mongodb-robomongo/connectionstringblade.png)  

2. 运行 *Robomongo.exe*

3. 单击“文件”下的“连接”按钮以管理连接。然后，在“MongoDB 连接”窗口中单击“创建”，这会打开“连接设置”窗口。

4. 在“连接设置”窗口中，选择名称。然后，从步骤 1 的连接信息中找到**主机**和**端口**，并将其分别输入到**地址**和**端口**中。

    ![Robomongo 管理连接的屏幕截图](./media/documentdb-mongodb-robomongo/manageconnections.png)  

5. 在“身份验证”选项卡上，单击“执行身份验证”。然后，输入数据库（默认值为 *Admin*）、**用户名**和**密码**。**用户名**和**密码**可以在步骤 1 的连接信息中找到。

    ![Robomongo 身份验证选项卡的屏幕截图](./media/documentdb-mongodb-robomongo/authentication.png)  

6. 在“SSL”选项卡上，选中“使用 SSL 协议”，然后将“身份验证方法”更改为“自签名证书”。

    ![Robomongo SSL 选项卡的屏幕截图](./media/documentdb-mongodb-robomongo/SSL.png)  

7. 最后，单击“测试”验证是否能够连接，然后单击“保存”。

## 后续步骤
- 浏览具有 MongoDB 协议支持的 DocumentDB [示例](/documentation/articles/documentdb-mongodb-samples/)。

<!---HONumber=Mooncake_0103_2017-->
