1.	登录在线 [Azure 门户预览](https://portal.azure.cn/)。
2.	在跳转栏中，依次单击“New”、“Databases”、“DocumentDB”。

	![创建数据库时的 Azure 门户预览屏幕截图，其中突出显示了“新建”按钮、“创建”边栏选项卡中的“数据 + 存储器”以及“数据 + 存储器”边栏选项卡中的“Azure DocumentDB”](./media/documentdb-create-dbaccount/create-nosql-db-databases-json-tutorial-1.png)

3. 在“新建 DocumentDB 帐户”边栏选项卡中，为 DocumentDB 帐户指定所需配置。

	![“新建 DocumentDB”边栏选项卡的屏幕截图](./media/documentdb-create-dbaccount/create-nosql-db-databases-json-tutorial-2.png)


	- 在“ID”框中，输入一个名称用于标识 DocumentDB 帐户。对“ID”进行验证后，“ID”框中会出现一个绿色的复选标记。该“ID”值将成为 URI 中的主机名。“ID”只能包含小写字母、数字及“-”字符，且长度必须为 3 到 50 个字符。请注意，documents.azure.cn 会追加到所选择的终结点名称，其结果将成为 DocumentDB 帐户终结点。

	- 对于“订阅”，请选择要用于 DocumentDB 帐户的 Azure 订阅。如果帐户只有一个订阅，则默认情况下会选择该帐户。

	- 在“资源组”中，为 DocumentDB 帐户选择或创建资源组。默认创建新的资源组。有关详细信息，请参阅[使用 Azure 门户预览管理 Azure 资源](/documentation/articles/resource-group-portal/)。

	- 使用“位置”指定在其中托管 DocumentDB 帐户的地理位置。
	
    - 若要方便访问帐户和以后将创建的资源，请选中“固定到仪表板”。

4.	配置了新的 DocumentDB 帐户后，单击“创建”。要检查部署的状态，可在“启动板”监视进度。![启动板上的“创建”磁贴的屏幕截图 — 在线数据库创建者](./media/documentdb-create-dbaccount/create-nosql-db-databases-json-tutorial-3.png)

	或者，可以从通知中心监视进度。

	![快速创建数据库 — 通知中心的屏幕截图，其中显示正在创建 DocumentDB 帐户](./media/documentdb-create-dbaccount/create-nosql-db-databases-json-tutorial-4.png)  


	![通知中心的屏幕截图，其中显示 DocumentDB 帐户已成功创建并且部署到资源组 — 在线数据库创建者通知](./media/documentdb-create-dbaccount/create-nosql-db-databases-json-tutorial-5.png)

5.	创建 DocumentDB 帐户之后，它便已准备就绪，可与在线门户中的默认设置结合使用。请注意，DocumentDB 帐户的默认一致性设置为“会话”。可单击菜单中的“默认一致性”调整默认一致性设置。若要了解有关 DocumentDB 提供的一致性级别的详细信息，请参阅 [DocumentDB 中的一致性级别](/documentation/articles/resource-group-portal/)

    ![“资源组”边栏选项卡的屏幕截图 — 开始应用程序开发](./media/documentdb-create-dbaccount/create-nosql-db-databases-json-tutorial-6.png)  


    ![“一致性级别”边栏选项卡的屏幕截图 — 会话一致性](./media/documentdb-create-dbaccount/create-nosql-db-databases-json-tutorial-7.png)

[How to: Create a DocumentDB account]: #Howto
[Next steps]: #NextSteps
[documentdb-manage]: /documentation/articles/documentdb/documentdb-manage

<!---HONumber=Mooncake_0829_2016-->