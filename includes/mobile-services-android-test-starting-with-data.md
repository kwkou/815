更新应用程序以使用移动服务作为后端存储后，便可以使用 Android 模拟器或 Android 手机针对移动服务测试该应用程序。

1.  在“运行”菜单中，单击“运行”以启动项目。

    这将执行使用 Android SDK 构建的应用程序，该应用程序使用客户端库发送一个查询，该查询从你的移动服务返回项目。

2.  和前面一样，输入有意义的文本，然后单击“添加”。

    此时会将一个新项作为 insert 发送到移动服务。

    你可以重新启动应用程序，以查看更改是否已持久保存在 Azure 中的数据库内。你也可以使用 Azure 管理门户检查数据库：后面的两个步骤将会执行此操作以查看数据库中的更改。

3.  在 Azure 管理门户中，单击与你的移动服务关联的数据库对应的“管理”。

    ![][0]

4.  在管理门户中，执行一个查询以查看 Windows 应用商店应用程序所做的更改。你的查询应类似于以下查询，不过，请使用你的数据库名称而不是 `todolist`.

        SELECT * FROM [todolist].[todoitems]

    ![][1]

针对 Android 的**数据处理入门**教程到此结束。

  [0]: ./media/mobile-services-dotnet-backend-windows-store-dotnet-get-started-data/manage-sql-azure-database.png
  [1]: ./media/mobile-services-dotnet-backend-windows-store-dotnet-get-started-data/sql-azure-query.png
