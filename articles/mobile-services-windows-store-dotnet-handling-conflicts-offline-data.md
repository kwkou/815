<properties 
	pageTitle="在通用 Windows 应用中处理与脱机数据的冲突 | Microsoft Azure" 
	description="了解在通用 Windows 应用程序中同步脱机数据时如何使用 Azure 移动服务处理冲突" 
	documentationCenter="windows" 
	authors="wesmc7777" 
	manager="dwrede" 
	editor="" 
	services="mobile-services"/>

<tags 
	ms.service="mobile-services" 
	ms.date="02/11/2016"
	wacn.date="03/21/2016"/>


# 在“移动服务”中处理与脱机数据同步的冲突

[AZURE.INCLUDE [mobile-services-selector-offline-conflicts](../includes/mobile-services-selector-offline-conflicts.md)]

##概述

本主题演示在使用 Azure 移动服务的脱机功能时如何同步数据和处理冲突。



在本教程中，你将为支持处理脱机同步冲突的应用下载通用 Windows C# 解决方案。您要将移动服务与应用程序集成，然后再运行的 Windows 应用商店 8.1 和 Windows Phone 8.1 客户端生成同步冲突，并解决冲突。

本教程以前一教程[脱机数据处理入门]中的步骤和示例应用程序为基础。在开始本教程之前，应先完成[脱机数据入门]。


##先决条件

本教程需要运行在 Windows 8.1 上的 Visual Studio 2013。


##下载示例项目

![][0]

本教程逐步介绍 [Todo 脱机移动服务示例]如何处理本地脱机存储与 Azure 中的移动服务数据库之间的冲突。

1. 下载[移动服务示例 GitHub 存储库]的 zip 文件，并将其解压缩到工作目录。 

2. 如果尚未安装[脱机数据入门]教程中所述的适用于 Windows 8.1 和 Windows Phone 8.1 的 SQLite，请安装这两个运行时。

3. 在 Visual Studio 2013 中，打开 *mobile-services-samples\\TodoOffline\\WindowsUniversal\\TodoOffline-Universal.sln* 解决方案文件。按 **F5** 键重新生成并运行项目。验证是否还原了 NuGet 包以及是否正确设置了引用。

    >[AZURE.NOTE]可能需要删除对 SQLite 运行时的任何旧引用，并将其替换为已更新的引用（如[脱机数据入门]教程中所述）。

4. 在应用中，在“插入 TodoItem”中键入一些文本，然后单击“保存”将某些 todo 项添加到本地存储中。然后关闭应用程序。

请注意，此应用程序尚未连接到任何移动服务，因此“推送”和“拉取”按钮将引发异常。




##针对移动服务测试应用程序

现在可以针对移动服务测试应用程序了。

1. 在 [Azure 管理门户]中，通过单击“仪表板”选项卡的命令栏上的“管理密钥”找到移动服务的应用程序密钥。复制“应用程序密钥”。

2. 在 Visual Studio 的解决方案资源管理器中，打开客户端示例项目中的 App.xaml.cs 文件。更改 **MobileServiceClient** 的初始化以使用你的移动服务 URL 和应用程序密钥：

         public static MobileServiceClient MobileService = new MobileServiceClient(
            "https://your-mobile-service.azure-mobile.net/",
            "Your AppKey"
        );

3. 在 Visual Studio 中，按 **F5** 键重新生成并运行应用。

    ![][0]


##更新后端中的数据以制造冲突

在实际情况中，当一个应用程序将更新推送到数据库中的一条记录，然后另一个应用程序尝试使用该记录中过时的版本字段将更新推送到同一条记录时，会发生同步冲突。如[脱机数据入门]中所述，要支持脱机同步功能需要版本系统属性。通过每次数据库更新检查此版本信息。如果应用的实例尝试使用过时版本更新记录，则将发生冲突，并且会在应用中捕获为 `MobileServicePreconditionFailedException`。如果应用未捕获 `MobileServicePreconditionFailedException`，则最终将引发 `MobileServicePushFailedException`，描述遇到了多少同步错误。

>[AZURE.NOTE]若要通过脱机数据同步支持同步已删除的记录，应启用“[软删除](/documentation/articles/mobile-services-using-soft-delete/)”。否则，必须手动删除本地存储中的记录，或者调用 `IMobileServiceSyncTable::PurgeAsync()` 以清除本地存储。


下面的步骤演示使用示例同时运行 Windows Phone 8.1 和 Windows 应用商店 8.1 客户端以引发冲突并解决冲突。

1. 在 Visual Studio 中，右键单击 Windows Phone 8.1 项目，然后单击“设为启动项目”。然后按 **Ctrl+F5** 键以运行 Windows Phone 8.1 客户端而不进行调试。在模拟器中运行 Windows Phone 8.1 客户端之后，单击“拉取”按钮将本地存储与数据库的当前状态同步。
 
    ![][3]
 
   
2. 在 Visual Studio 中，右键单击 Windows 8.1 运行时项目，然后单击“设为启动项目”以将其重设为启动项目。然后按 **F5** 运行该项目。在运行 Windows 应用商店 8.1 客户端之后，单击“拉取”按钮将本地存储与数据库的当前状态同步。

    ![][4]
 
3. 此时，两个客户端都将与数据库同步。这两个客户端的代码还使用增量同步，以便仅同步不完整的 todo 项。已完成的 todo 项将被忽略。选择其中的一个项，并在两个客户端中将同一个项的文本编辑为不同的值。单击“推送”按钮将这两个更改与服务器上的数据库同步。

    ![][5]

    ![][6]


4. 最后执行其推送的客户端会遇到冲突，并让用户决定要将哪个值提交到数据库。该异常提供了用于解决冲突的正确版本值。

    ![][7]



## 查看处理同步冲突的代码

若要使用移动服务脱机功能，必须在本地数据库和数据传输对象中都包括版本列。这是通过更新 `TodoItem` 类的以下成员实现的：

        [Version]
        public string Version { get; set; }

当使用 `TodoItem` 类来定义本地存储时，`__version` 列包括在本地数据库的 `OnNavigatedTo()` 方法中。

若要使用代码处理脱机同步冲突，请创建一个实现 `IMobileServiceSyncHandler` 的类。调用 `MobileServiceClient.SyncContext.InitializeAsync()` 时传递这种类型的对象。在示例的 `OnNavigatedTo()` 方法中也会发生这种情况。

     await App.MobileService.SyncContext.InitializeAsync(store, new SyncHandler(App.MobileService));

**SyncHandler.cs** 中的类 `SyncHandler` 实现了 `IMobileServiceSyncHandler`。将每个推送操作发送到服务器时均调用 `ExecuteTableOperationAsync` 方法。如果引发了 `MobileServicePreconditionFailedException` 类型的异常，则意味着某个项目的本地版本和远程版本之间存在冲突。

若要在解决冲突时考虑到本地项目，则只需重试该操作。发生冲突后，需更新本地项目版本，使之与服务器版本匹配，因此重新执行该操作将使用本地更改覆盖服务器更改：

    await operation.ExecuteAsync(); 

若要在解决冲突时考虑到服务器项目，则只需从 `ExecuteTableOperationAsync` 返回。将放弃该对象的本地版本，将其替换为服务器中的值。

若要停止推送操作（但保留已排队的更改），请使用方法 `AbortPush()`：

    operation.AbortPush();

这将停止当前的推送操作，但会保留所有挂起的更改，包括当前操作（如果从 `ExecuteTableOperationAsync` 调用了 `AbortPush`）。下次调用 `PushAsync()` 时，这些更改将发送到服务器。

在取消推送时，`PushAsync` 将引发 `MobileServicePushFailedException`，而异常属性 `PushResult.Status` 将具有值 `MobileServicePushStatus.CancelledByOperation`。



<!-- Images -->
[0]: ./media/mobile-services-windows-store-dotnet-handling-conflicts-offline-data/mobile-services-handling-conflicts-app-run1.png
[1]: ./media/mobile-services-windows-store-dotnet-handling-conflicts-offline-data/javascript-backend-database.png
[2]: ./media/mobile-services-windows-store-dotnet-handling-conflicts-offline-data/dotnet-backend-database.png
[3]: ./media/mobile-services-windows-store-dotnet-handling-conflicts-offline-data/wp81-view.png
[4]: ./media/mobile-services-windows-store-dotnet-handling-conflicts-offline-data/win81-view.png
[5]: ./media/mobile-services-windows-store-dotnet-handling-conflicts-offline-data/wp81-edit-text.png
[6]: ./media/mobile-services-windows-store-dotnet-handling-conflicts-offline-data/win81-edit-text.png
[7]: ./media/mobile-services-windows-store-dotnet-handling-conflicts-offline-data/conflict.png




<!-- URLs -->
[Handling conflicts code sample]: http://go.microsoft.com/fwlink/?LinkId=394787
[Get started with Mobile Services]: /documentation/articles/mobile-services-windows-store-get-started/
[脱机数据入门]: /documentation/articles/mobile-services-windows-store-dotnet-get-started-offline-data/
[脱机数据处理入门]: /documentation/articles/mobile-services-windows-store-dotnet-get-started-offline-data/
[SQLite for Windows 8.1]: http://go.microsoft.com/fwlink/?LinkId=394776
[Azure 经典门户]: https://manage.windowsazure.cn/
[Handling Database Conflicts]: /documentation/articles/mobile-services-windows-store-dotnet-handle-database-conflicts/#test-app
[移动服务示例 GitHub 存储库]: http://go.microsoft.com/fwlink/?LinkId=512865
[Todo 脱机移动服务示例]: http://go.microsoft.com/fwlink/?LinkId=512866

<!---HONumber=Mooncake_0118_2016-->