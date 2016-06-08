

接下来，您将创建一个新移动服务以替换数据存储的内存中列表。按照下列步骤操作以创建新的移动服务。

1. 登录到 [Azure 管理门户](https://manage.windowsazure.cn/)。 
2.	在导航窗格的底部，单击“+新建”：

	![plus-new](./media/mobile-services-create-new-service-data/plus-new.png)

3.	展开“计算”和“移动服务”，然后单击“创建”。

	![mobile-create](./media/mobile-services-create-new-service-data/mobile-create.png)

    此时将显示“新建移动服务”对话框。

4.	在“创建移动服务”页上，选择“创建免费的 20 MB SQL 数据库”，然后在“URL”文本框中键入新移动服务的子域名称，并等待名称验证。名称验证完成后，单击右箭头按钮以转到下一页。

	![mobile-create-page1](./media/mobile-services-create-new-service-data/mobile-create-page1.png)

    此时将显示“指定数据库设置”页。

    
	> [AZURE.NOTE] 在本教程中，您将创建新的 SQL 数据库实例和服务器。您可以重用此新数据库，并对其进行管理，如同任何其他 SQL 数据库实例一样。如果您在新移动服务的同一区域已经有了数据库，则可选择“使用现有数据库”，然后再选择该数据库。由于额外的带宽成本和更高的延迟，不建议使用位于不同区域的数据库。

5.	在“名称”中，键入新数据库的名称，然后键入“登录名”，也就是新的 SQL 数据库服务器的管理员登录名，键入并确认密码，然后单击勾选按钮完成该过程。

	![mobile-create-page2](./media/mobile-services-create-new-service-data/mobile-create-page2.png)

    
	> [AZURE.NOTE]如果您提供的密码不符合最低要求或不匹配，将显示警告。
	>
	> 我们建议您记下自己指定的管理员登录名和密码；今后您将需要使用这些信息来重用 SQL 数据库实例或服务器。

您现在已经创建了您的移动应用可以使用的新移动服务。接下来，您将添加一个在其中存储应用数据的新表。应用将使用此表而不是内存中集合。

<!---HONumber=Mooncake_0118_2016-->