<properties linkid="develop-mobile-tutorials-handle-conflcits-offline-data-dotnet" urlDisplayName="Handle Conflicts with Offline Data" pageTitle="Handle Conflicts with offline data in Mobile Services (Windows Phone) | Mobile Dev Center" metaKeywords="" description="Learn how to handle conflicts with offline data in your Windows Phone application." metaCanonical="" disqusComments="1" umbracoNaviHide="1" documentationCenter="Mobile" title="Handling conflicts with offline data in Mobile Services" authors="wesmc" />

# 使用移动服务中的脱机数据处理冲突

<div class="dev-center-tutorial-selector sublanding">
<a href="/zh-cn/documentation/articles/mobile-services-windows-store-dotnet-handling-conflicts-offline-data" title="Windows Store C#">Windows 应用商店 C\#</a>
<a href="/zh-cn/documentation/articles/mobile-services-windows-phone-handling-conflicts-offline-data" title="Windows Phone" class="current">Windows Phone</a>
</div>

<p>本主题演示在使用 Azure 移动服务的脱机功能时如何同步数据和处理冲突。在本教程中，你将下载一个同时支持脱机和联机数据的应用程序，将移动服务与该应用程序集成，然后在运行该应用程序时登录到 Azure 管理门户以查看和更新数据库。</p>

本教程以前一教程[脱机数据入门][]中的步骤和示例应用程序为基础。在开始本教程之前，必须先完成[脱机数据入门][]。

本教程将指导你完成以下基本步骤：

1.  [下载 Windows Phone 项目][]
2.  [为数据库添加截止日期列][]

-   [更新 .NET 后端移动服务的数据库][]
-   [更新 JavaScript 移动服务的数据库][]

1.  [针对移动服务测试应用程序][]
2.  [手动更新后端中的数据以制造冲突][]

本教程需要 Visual Studio 2012 和 [Windows Phone 8 SDK][]。

<a name="download-app"></a>
## 下载示例项目

本教程基于[处理冲突代码示例][]，后者是用于 Visual Studio 2012 的 Windows Phone 8 项目。
此应用程序的 UI 类似于教程[脱机数据入门][]中的应用程序，不同的是，每个 TodoItem 有一个新的日期列。

![][]

1.  下载[处理冲突代码示例][]的 Windows Phone 版本。

2.  安装 [SQLite for Windows Phone 8][]（如果尚未安装）。

3.  在 Visual Studio 2012 中，打开已下载的项目。在“Windows Phone” \>“扩展” 下添加对“SQLite for Windows Phone” 的引用。

4.  在 Visual Studio 2012 中，按 F5  键在调试器中生成并运行应用程序。

5.  在应用程序中，为一些新的 Todo 项目键入一些文本，然后单击“保存” 以保存每个项目。你还可以修改所添加的 Todo 项目的截止日期。

请注意，此应用程序尚未连接到任何移动服务，因此“推送” 和“拉取” 按钮将引发异常。

<a name="add-column"></a>
## 向数据模型添加列

在本节中，你将更新移动服务的数据库，使之包括具有截止日期列的 TodoItem 表。此应用程序允许你在运行时更改项目的截止日期，以便在本教程的后面部分制造同步冲突。

此示例中的 `TodoItem` 类是在 MainPage.xaml.cs 中定义的。请注意，该类具有以下特性，这些特性将针对该表定向同步操作。

        [DataTable("TodoWithDate")]

更新数据库，使之包括该表。

<a name="dotnet-backend"></a>
### 更新 .NET 后端移动服务的数据库

如果你使用的是移动服务的 .NET 后端，请执行以下步骤来更新数据库的架构。

1.  在 Visual Studio 中打开 .NET 后端移动服务项目。
2.  在 Visual Studio 的解决方案资源管理器中，从你的服务项目展开“模型” 文件夹并打开 ToDoItem.cs。添加 `DueDate` 属性，如下所示。

          public class TodoItem :EntityData
          {
        public string Text { get; set; }
        public bool Complete { get; set; }
        public System.DateTime DueDate { get; set; }
          }

3.  在 Visual Studio 的解决方案资源管理器中，展开“App\_Start” 文件夹，然后打开 WebApiConfig.cs 文件。

    请注意，在 WebApiConfig.cs 文件中，默认数据库初始值设定项类是从 `DropCreateDatabaseIfModelChanges` 类派生的。这意味着，对该模型的任何更改将导致表被删除并重新创建，以适应新模型。因此表中的数据将丢失，并且表将重新植入。修改数据库初始值设定项的 Seed 方法，使下述 `Seed()` 初始化函数可以初始化新的 DueDate 列。保存 WebApiConfig.cs 文件。

    > [WACOM.NOTE] 使用默认数据库初始值设定项时，只要实体框架在代码优先模型定义中检测到数据模型更改，它就会删除并重新创建数据库。若要进行此数据模型更改并维护数据库中的现有数据，必须使用代码优先迁移。有关详细信息，请参阅[如何使用代码优先迁移更新数据模型][]。

        new TodoItem { Id = "1", Text = "First item", Complete = false, DueDate = DateTime.Today },
        new TodoItem { Id = "2", Text = "Second item", Complete = false, DueDate = DateTime.Today },

4.  在 Visual Studio 的解决方案资源管理器中，展开“控制器” 文件夹并打开 ToDoItemController.cs。将 `TodoItemController` 类重命名为 `TodoWithDateController`。这将更改表操作的 REST 终结点。

        public class TodoWithDateController :TableController<TodoItem>

5.  在 Visual Studio 的解决方案资源管理器中，右键单击你的 NET 后端移动服务项目，然后单击“发布” 以发布你的更改。

<a name="javascript-backend"></a>
### 更新 JavaScript 后端移动服务的数据库

对于 JavaScript 后端移动服务，你将添加一个名为“TodoWithDate” 的新表。若要为 JavaScript 后端移动服务添加 TodoWithDate  表，请执行以下步骤。

1.  登录到 [Azure 管理门户][]。

2.  导航到移动服务的“数据” 选项卡。

3.  单击页面底部的“创建” ，创建一个名为“TodoWithDate” 的新表。

<a name="test-app"></a>
## 针对移动服务测试应用程序

现在可以针对移动服务测试应用程序了。

1.  在 Azure 管理门户中，通过单击“仪表板” 选项卡的命令栏上的“管理密钥” 找到移动服务的应用程序密钥。复制应用程序密钥 。

2.  在 Visual Studio 的解决方案资源管理器中，打开客户端示例项目中的 App.xaml.cs 文件。更改 "MobileServiceClient" 的初始化以使用你的移动服务 URL 和应用程序密钥：

         public static MobileServiceClient MobileService = new MobileServiceClient(
        "https://your-mobile-service.azure-mobile.net/",
        "Your AppKey"
        );

3.  在 Visual Studio 中，按 F5  键生成并运行应用程序。

4.  和以前一样，在文本框中键入文本，然后单击“保存” 以保存一些新的 Todo 项目。这会将数据保存到本地同步表中，而不是保存到服务器。

    ![][]

5.  若要查看数据库的当前状态，请登录到 [Azure 管理门户][]，单击“移动服务” ，然后单击你的移动服务。

-   如果你使用的是移动服务的 JavaScript 后端，请单击“数据” 选项卡，然后单击“TodoWithDate” 表。单击“浏览” 可看到该表将仍是空的，因为我们尚未将应用程序中的更改推送到服务器。

        ![][1]

-   如果你使用的是移动服务的 .NET 后端，请单击“配置” 选项卡，然后单击你的 SQL 数据库。单击屏幕底部的“管理” 以登录到 SQL Azure 管理门户，通过运行类似以下语句的 SQL 查询来查看你的数据库。

            SELECT * FROM todolist.todowithdate

        ![][2]

1.  回到应用程序，单击“推送” 。

2.  在管理门户中，单击“TodoItem” 表上的“刷新” 。现在，你应该看到你在应用程序中输入的数据。

    ![][1]

3.  让模拟器 WVGA 512MB  启动并运行，为下一节做准备。在下一节中，你将在两个模拟器中运行此应用程序以制造冲突。

<a name="handle-conflict"></a>
## 更新后端中的数据以制造冲突

在实际情况中，当一个应用程序将更新推送到数据库中的一条记录，然后另一个应用程序尝试使用该记录中过时的版本字段将更改推送到同一条记录时，会发生同步冲突。如果应用程序的一个实例在未拉取已更新记录的情况下尝试更新同一条记录，则会发生冲突，并且冲突将在应用程序中作为 `MobileServicePreconditionFailedException` 被捕获。

在本节中，你将同时运行应用程序的两个实例以制造冲突。

1.  如果模拟器 WVGA 512MB  仍未启动和运行，请按 Ctrl+F5  以重新启动它。

2.  在 Visual Studio 中，将输出设备更改为“模拟器 WVGA” ，然后按 F5  在新的模拟器中运行应用程序的第二个实例。

    ![][2]

3.  在应用程序的第二个实例中，单击“拉取” 让本地存储与移动服务数据库同步。此时，应用程序的这两个实例应具有相同的数据。

    ![][3]

4.  在应用程序的第二个实例中，单击复选框以完成其中一个项目，然后单击“推送” 将你的更改推送到远程数据库。在下面的屏幕快照中，“提取 James” 已完成，说明已提取 James。应用程序的第一个实例现在有一条过时的记录。

    ![][4]

5.  在应用程序的第一个实例中，尝试更改过期记录的日期，然后单击“推送” 以使用过时记录更新远程数据库。在下面的屏幕快照中，我们计划在 "5/10/2014" 提取 James。

    ![][5]

6.  当你单击“推送” 按钮提交日期更改时，将显示一个对话框，指示已检测到冲突。系统将询问你如何解决冲突。请选择其中一个选项来解决冲突。

    在如下所示的情况下，已提取 James。因此，无需在 "5/10/2014" 计划提取他。将选择“使用服务器版本” 选项，让应用程序的第一个实例使用服务器中的记录更新该记录。

    ![][6]

## 查看处理同步冲突的代码

若要设置脱机功能以检测冲突，必须在本地数据库和数据传输对象中包括版本列。`TodoItem` 类具有以下成员：

        [Version]
    public string Version { get; set; }

`__version` 列也在 `OnNavigatedTo()` 方法设置的本地数据库中指定。

若要使用代码处理脱机同步冲突，请创建一个实现 `IMobileServiceSyncHandler` 的类。调用 `InitializeAsync` 时传递这种类型的对象：

     await App.MobileService.SyncContext.InitializeAsync(store, new SyncHandler(App.MobileService));

"MainPage.xaml.cs" 中的 `SyncHandler` 类实现了 `IMobileServiceSyncHandler`。将每个推送操作发送到服务器时均调用方法 `ExecuteTableOperationAsync`。如果引发了 `MobileServicePreconditionFailedException` 类型的异常，则意味着某个项目的本地版本和远程版本之间存在冲突。

若要在解决冲突时考虑到本地项目，则只需重试该操作。发生冲突后，需更新本地项目版本，使之与服务器版本匹配，因此重新执行该操作将使用本地更改覆盖服务器更改：

    await operation.ExecuteAsync(); 

若要在解决冲突时考虑到服务器项目，则只需从 `ExecuteTableOperationAsync` 返回。将放弃该对象的本地版本，将其替换为服务器中的值。

若要停止推送操作（但保留已排队的更改），请使用方法 `AbortPush()`：

    operation.AbortPush();

如果从 `ExecuteTableOperationAsync` 调用 `AbortPush`，则会停止当前的推送操作，但保留所有挂起的更改（包括当前操作）。下次调用 `PushAsync()` 时，这些更改将发送到服务器。

取消推送时，`PushAsync` 将引发 `MobileServicePushFailedException`，而异常属性 `PushResult.Status` 的值将为 `MobileServicePushStatus.CancelledByOperation`。

  [Windows 应用商店 C\#]: /zh-cn/documentation/articles/mobile-services-windows-store-dotnet-handling-conflicts-offline-data "Windows 应用商店 C#"
  [Windows Phone]: /zh-cn/documentation/articles/mobile-services-windows-phone-handling-conflicts-offline-data "Windows Phone"
  [脱机数据入门]: /zh-cn/documentation/articles/mobile-services-windows-phone-get-started-offline-data
  [下载 Windows Phone 项目]: #download-app
  [为数据库添加截止日期列]: #add-column
  [更新 .NET 后端移动服务的数据库]: #dotnet-backend
  [更新 JavaScript 移动服务的数据库]: #javascript-backend
  [针对移动服务测试应用程序]: #test-app
  [手动更新后端中的数据以制造冲突]: #handle-conflict
  [Windows Phone 8 SDK]: http://go.microsoft.com/fwlink/p/?linkid=268374
  [处理冲突代码示例]: http://go.microsoft.com/fwlink/?LinkId=398257
  [0]: ./media/mobile-services-windows-phone-handling-conflicts-offline-data/mobile-services-handling-conflicts-app-run1.png
  [SQLite for Windows Phone 8]: http://go.microsoft.com/fwlink/?LinkId=397953
  [如何使用代码优先迁移更新数据模型]: /zh-cn/documentation/articles/mobile-services-dotnet-backend-how-to-use-code-first-migrations
  [Azure 管理门户]: https://manage.windowsazure.cn/
  [1]: ./media/mobile-services-windows-phone-handling-conflicts-offline-data/mobile-services-todowithdate-push1.png
  [2]: ./media/mobile-services-windows-phone-handling-conflicts-offline-data/vs-emulator-wvga.png
  [3]: ./media/mobile-services-windows-phone-handling-conflicts-offline-data/two-emulators-synced.png
  [4]: ./media/mobile-services-windows-phone-handling-conflicts-offline-data/two-emulators-item-completed.png
  [5]: ./media/mobile-services-windows-phone-handling-conflicts-offline-data/two-emulators-date-change.png
  [6]: ./media/mobile-services-windows-phone-handling-conflicts-offline-data/two-emulators-conflict-detected.png
