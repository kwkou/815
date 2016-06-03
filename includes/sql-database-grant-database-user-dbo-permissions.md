## 使用 SSMS 创建新数据库用户

以下步骤假设你使用的是 SSMS、已连接到对象资源管理器中的 SQL 数据库，并以服务器级主体管理员或使用有权授予用户权限的用户帐户连接到 SQL 数据库逻辑服务器。此外，以下步骤假设数据库中存在一个你要授予其 dbo 权限的用户。

1. 在对象资源管理器中，展开“数据库”节点并选择包含你要为其授予 dbo 权限的用户的数据库。

     ![SQL Server Management Studio：连接到 SQL 数据库服务器](./media/sql-database-create-new-database-user/sql-database-create-new-database-user-1.png)

2. 右键单击选定的数据库，然后选择“查询”。

     ![SQL Server Management Studio：连接到 SQL 数据库服务器](./media/sql-database-create-new-database-user/sql-database-create-new-database-user-2.png)

3. 在查询窗口中，编辑并使用以下 Transact-SQL 语句向指定的用户授予 dbo 权限。

    ```ALTER ROLE db\_owner ADD MEMBER user1;```

     ![SQL Server Management Studio：连接到 SQL 数据库服务器](./media/sql-database-grant-database-user-dbo-permissions/sql-database-grant-database-user-dbo-permissions-1.png)

<!---HONumber=Mooncake_0503_2016-->
