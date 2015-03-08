

以下步骤在 Azure 中创建新的移动服务，并将代码添加到您的项目，此代码将您的应用连接到此新服务。Visual Studio 2013 使用您提供的凭证来代表您连接到 Azure 以创建新的移动服务。创建新的移动服务时，您必须指定该移动服务用于存储应用数据的 Azure SQL Database。 

1. 在 Visual Studio 2013 中，打开 Solution Explorer，右键单击 Windows Store 应用项目，单击**添加**，然后单击**连接的服务...**。 

2. 在"服务管理器"对话框中，单击**创建服务...**，然后从"创建移动服务"对话框中的"订阅"选择**&lt;管理...&gt;**。  

	![create service manage subscriptions](./media/mobile-services-dotnet-backend-create-new-service-vs2013/mobile-create-service-from-vs2013.png)

3. 在"管理 Microsoft Azure Subscriptions"中，单击**登录...**以登录到您的 Azure 帐户（如果需要），选择可用的订阅，然后单击**关闭**。

	您的订阅已经具有一个或多个现有移动服务时，会显示服务名称。 

5. 返回**创建移动服务**对话框，为您的移动服务选择**订阅**、**运行时**中的 **.NET Framework** 后端和**区域**，然后为您的移动服务键入**名称**。

	>[WACOM.NOTE]移动服务名称必须是唯一的。当您提供的名称不可用时，"名称"****旁边将显示红色的 X。 

6. 在**数据库**中，选择**&lt;创建免费的 SQL Database&gt;**，提供**服务器用户名**、**服务器密码**和**服务器密码确认**，然后单击**创建**。

  	![create new mobile service in VS 2013](./media/mobile-services-dotnet-backend-create-new-service-vs2013/mobile-create-service-from-vs2013-2.png)

	> [WACOM.NOTE]
	> 在本教程中，您将创建新的免费 SQL Database 实例和服务器。您可以重用此新数据库，并对其进行管理，如同任何其他 SQL Database 实例一样。您只能有一个免费数据库实例。如果您在新移动服务所在的同一地区已经有了数据库，则可选择使用现有数据库。当您选择现有数据库时，请确认您提供了正确的登录凭据。如果您提供了错误的登录凭据，则会在不正常的状态下创建移动服务。

7. 创建移动服务后，从服务管理器中的列表选择新建的移动服务，然后单击**确定**。
 
   	在向导完成后，移动服务项目添加到您的解决方案，安装了所需的 NuGet 软件包，对移动服务客户端库的引用添加到项目中，然后您的项目源代码已更新。

