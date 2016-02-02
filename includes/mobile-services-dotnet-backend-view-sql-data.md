
本教程的最后可选步骤是在与移动服务关联的 SQL 数据库中检查和复审所存储的数据。

1. 在 [Azure 经典门户](https://manage.windowsazure.cn/)中，单击与移动服务关联的数据库对应的“管理”。
 
	![登录以管理 SQL 数据库](./media/mobile-services-dotnet-backend-view-sql-data/manage-sql-azure-database.png)

2. 在门户中，执行查询以查看 Windows Store 应用所做的更改。你的查询应类似于以下查询，不过，请使用你的数据库名称而不是 <code>todolist</code>。</p>

        SELECT * FROM [todolist].[todoitems]

    ![在 SQL 数据库中查询存储项](./media/mobile-services-dotnet-backend-view-sql-data/sql-azure-query.png)

	请注意，该表包含 Id、\_\_createdAt、\_\_updatedAt 和 \_\_Version 列。这些列支持脱机数据同步并在 [EntityData](http://msdn.microsoft.com/zh-cn/library/microsoft.windowsazure.mobile.service.entitydata.aspx) 基类中实现。有关详细信息，请参阅 [脱机数据同步入门]。

<!---HONumber=Mooncake_0118_2016-->