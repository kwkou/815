既然此应用已更新来将移动服务用作后端存储，您便可以使用 Android 模拟器或 Android 手机针对移动服务测试该应用。

1. 在**运行**菜单中，单击**运行**以启动项目。

	这将执行使用 Android SDK 构建的应用，该应用使用客户端库发送一个查询，该查询从您的移动服务返回项目。

5. 与以前一样，键入有意义的文本，然后单击**添加**。

   	此时会将一个新项作为 insert 发送到移动服务。

    您可以重新启动此应用，以查看更改是否已持久保存在 Azure 中的数据库内。您也可以使用 Azure 管理门户来检查数据库：后续两个步骤将会执行此操作以查看数据库中的更改。


4. 在 Azure 管理门户中，单击与移动服务关联的数据库对应的"管理"。

    ![](./media/mobile-services-dotnet-backend-windows-store-dotnet-get-started-data/manage-sql-azure-database.png)

5. 在管理门户中，执行查询以查看 Windows Store 应用所做的更改。您的查询类似于以下查询，但是使用数据库名称而不是 `todolist`。

        SELECT * FROM [todolist].[todoitems]

    ![](./media/mobile-services-dotnet-backend-windows-store-dotnet-get-started-data/sql-azure-query.png)

针对 Android 的**数据处理入门**教程到此结束。
