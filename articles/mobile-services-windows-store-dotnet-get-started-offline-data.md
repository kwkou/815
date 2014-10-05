<properties linkid="develop-mobile-tutorials-get-started-offline-data-dotnet" urlDisplayName="Getting Started with Offline Data" pageTitle="Get started with offline data in Mobile Services (Windows Store) | Mobile Dev Center" metaKeywords="" description="Learn how to use offline data in your Windows Store application." metaCanonical="" disqusComments="1" umbracoNaviHide="1" documentationCenter="Mobile" title="Get started with offline data in Mobile Services" authors="wesmc" />

# 移动服务中的脱机数据入门

<div class="dev-center-tutorial-selector sublanding">
<a href="/zh-cn/documentation/articles/mobile-services-windows-store-dotnet-get-started-offline-data" title="Windows Store C#" class="current">Windows 应用商店 C\#</a>
<a href="/zh-cn/documentation/articles/mobile-services-windows-phone-get-started-offline-data" title="Windows Phone">Windows Phone</a>
</div>

本主题演示如何使用 Azure 移动服务的脱机功能。当你的移动服务处于脱机情况时，可使用 Azure 移动服务脱机功能与本地数据库交互。当你重新联机时，脱机功能允许你通过移动服务同步本地更改。

在本教程中，你将更新[移动服务入门][]或[数据入门][]教程中的应用程序，以支持 Azure 移动服务的脱机功能。随后，你将在断开连接的脱机情况下添加数据，将这些项目同步到联机数据库，然后登录到 Azure 管理门户，查看在运行应用程序时对数据所做的更改。

> [WACOM.NOTE] 本教程旨在帮助你更好地了解如何使用移动服务通过 Azure 在 Windows 应用商店应用程序中存储和检索数据。因此，本主题指导你完成的许多步骤已在移动服务快速入门中代你完成。如果这是你第一次体验移动服务，请考虑首先完成[移动服务入门][]教程。

本教程将指导你完成以下基本步骤：

1.  [更新应用程序以支持脱机功能][]
2.  [在脱机情况下测试应用程序][]
3.  [更新应用程序以重新连接移动服务][]
4.  [测试已连接到移动服务的应用程序][]

本教程需要的内容如下：

-   运行在 Windows 8.1 上的 Visual Studio 2013。
-   完成[移动服务入门][]或[数据入门][]教程。
-   Windows Azure 移动服务 SDK NuGet 包版本 1.3.0-alpha2
-   Windows Azure 移动服务 SQLite Store NuGet 包 1.0.0-alpha
-   SQLite for Windows 8.1

> [WACOM.NOTE] 若要完成本教程，你需要一个 Azure 帐户。如果你没有帐户，只需花费几分钟就能创建一个免费试用帐户。有关详细信息，请参阅 [Azure 免费试用][]。

<a name="enable-offline-app"></a>
## 更新应用程序以支持脱机功能

当你的移动服务处于脱机情况时，可使用 Azure 移动服务脱机功能与本地数据库交互。若要在你的应用程序中使用这些功能，请将 `MobileServiceClient.SyncContext` 初始化到本地存储。然后，通过 `IMobileServiceSyncTable` 接口引用表。

本节使用 SQLite 作为脱机功能的本地存储。

> [WACOM.NOTE] 可以跳过本节，而只下载具有脱机支持的“入门”项目版本。若要下载启用了脱机支持的项目，请参阅[脱机示例入门][]。

1.  安装 SQLite。可以从此链接 ([SQLite for Windows 8.1][]) 安装它。

    > [WACOM.NOTE] 如果你使用的是 Internet Explorer，则单击用于安装 SQLite 的链接可能会提示你下载 .zip 文件形式的 .vsix。使用 .vsix 扩展名而不是 .zip 将文件保存到硬盘上的某一位置。在 Windows 资源管理器中双击 .vsix 文件以运行安装程序。

2.  在 Visual Studio 中，打开你在[移动服务入门][]或[数据入门][]教程中完成的项目。在解决方案资源管理器中，右键单击项目下的“引用” ，并在“Windows” \>“扩展” 下添加对“SQLite for Windows Runtime” 的引用。

    ![][]

3.  SQLite 运行时要求你将所生成的项目的处理器体系结构更改为“x86” 、“x64” 或“ARM” 。不支持“任何 CPU” 。请将处理器体系结构更改为要测试的受支持设置之一。

4.  在 Visual Studio 的解决方案资源管理器中，右键单击客户端应用程序项目，然后单击“管理 Nuget 包” 以运行 NuGet 包管理器。搜索“SQLiteStore” 以安装 WindowsAzure.MobileServices.SQLiteStore 包 。

    ![][1]

5.  在 Visual Studio 的解决方案资源管理器中，打开 MainPage.xaml.cs 文件。将下列 using 语句添加到文件顶部。

        using Microsoft.WindowsAzure.MobileServices.SQLiteStore;
        using Microsoft.WindowsAzure.MobileServices.Sync;
        using Newtonsoft.Json.Linq;

6.  在 Mainpage.xaml.cs 中，将 `todoTable` 的声明替换为通过调用 `MobileServicesClient.GetSyncTable()` 来初始化的类型为 `IMobileServicesSyncTable` 的声明。

        //private IMobileServiceTable<TodoItem> todoTable = App.MobileService.GetTable<TodoItem>();
        private IMobileServiceSyncTable<TodoItem> todoTable = App.MobileService.GetSyncTable<TodoItem>();

7.  在 MainPage.xaml.cs 中，更新 `TodoItem` 类，使该类包括 "Version" 系统属性，如下所示。

        public class TodoItem
        {
        public string Id { get; set; }
        [JsonProperty(PropertyName = "text")]
        public string Text { get; set; }
        [JsonProperty(PropertyName = "complete")]
        public bool Complete { get; set; }
        [Version]
        public string Version { get; set; }
        }

8.  在 MainPage.xaml.cs 中，更新 `OnNavigatedTo` 事件处理程序，使之成为 `async` 方法，并使用 SQLite 存储初始化客户端同步上下文。SQLite 存储是使用与移动服务表的架构匹配的表创建的，但它必须包含上一步添加的 "Version" 系统属性。

        protected async override void OnNavigatedTo(NavigationEventArgs e)
        {
        if (!App.MobileService.SyncContext.IsInitialized)
            {
        var store = new MobileServiceSQLiteStore("localsync12.db");
        store.DefineTable<TodoItem>();
        await App.MobileService.SyncContext.InitializeAsync(store, new MobileServiceSyncHandler());
            }
        RefreshTodoItems();
        }

9.  在 Visual Studio 的解决方案资源管理器中，打开 MainPage.xaml 文件。找到包含标题为 "Query and Update Data" 的 StackPanel 的 Grid 元素。添加以下 UI 代码以替换从行定义向下到 ListView 的开始标记之间的元素。

    此代码添加用于元素布局网格的行定义和列定义。它还添加两个按钮控件，其中包含“推送” 和“拉取” 操作的单击事件处理程序。这些按钮就在名为 ListItems 的 `ListView` 的上面。保存文件。

        <Grid.RowDefinitions>
        <RowDefinition Height="Auto" />
        <RowDefinition Height="Auto" />
        <RowDefinition Height="*" />
        </Grid.RowDefinitions>
        <Grid.ColumnDefinitions>
        <ColumnDefinition Width="Auto" />
        <ColumnDefinition Width="*" />
        </Grid.ColumnDefinitions>
        <StackPanel Grid.Row="0" Grid.ColumnSpan="2">
        <local:QuickStartTask Number="2" Title="Query, Update, and Synchronize Data" 
        Description="Use the Pull and Push buttons to synchronize the local store with the server" />
        </StackPanel>
        <Button Grid.Row="1" Grid.Column="0" Margin="5,5,0,0" Name="ButtonPush"
        Click="ButtonPush_Click" Width="80" Height="34">?Push</Button>
        <Button Grid.Row="1" Grid.Column="1" Margin="5,5,0,0"  Name="ButtonPull" 
        Click="ButtonPull_Click" Width="80" Height="34">?Pull</Button>
        <ListView Name="ListItems" SelectionMode="None" Margin="0,10,0,0" Grid.ColumnSpan="2" Grid.Row="2">

10. 在 MainPage.xaml.cs 中，添加“推送” 和“拉取” 按钮的按钮单击事件处理程序，并保存该文件。

        private async void ButtonPull_Click(object sender, RoutedEventArgs e)
        {
        Exception pullException = null;
        try
            {
        await todoTable.PullAsync();
        RefreshTodoItems();
            }
        catch (Exception ex)
            {
        pullException = ex;
            }
        if (pullException != null) {
        MessageDialog d = new MessageDialog("Pull failed:" + pullException.Message +
        "\n\nIf you are in an offline scenario, " + 
        "try your Pull again when connected with your Mobile Serice.");
        await d.ShowAsync();
            }
        }
        private async void ButtonPush_Click(object sender, RoutedEventArgs e)
        {
        string errorString = null;
        try
            {
        await App.MobileService.SyncContext.PushAsync();
        RefreshTodoItems();
            }
        catch (MobileServicePushFailedException ex)
            {
        errorString = "Push failed because of sync errors: " + 
        ex.PushResult.Errors.Count() + ", message:" + ex.Message;
            }
        catch (Exception ex)
            {
        errorString = "Push failed:" + ex.Message;
            }
        if (errorString != null) {
        MessageDialog d = new MessageDialog(errorString + 
        "\n\nIf you are in an offline scenario, " + 
        "try your Push again when connected with your Mobile Serice.");
        await d.ShowAsync();
            }
        }

11. 此时请不要运行该应用程序。按 F7  键以重新生成项目。确保没有生成错误发生。

<a name="test-offline-app"></a>
## 在脱机情况下测试应用程序

在本节中，你将断开应用程序与移动服务的连接以模拟脱机情况。然后，你将添加一些数据项，这些项将保存到本地存储中。

请注意，在本节中，应用程序不应连接到任何移动服务。因此，如果你测试“推送” 和“拉取” 按钮，它们将引发异常。在下一节中，你会将此客户端应用程序重新连接到移动服务来测试“推送” 和“拉取” 操作，以便将存储与移动服务数据库进行同步。

1.  在 Visual Studio 的解决方案资源管理器中，打开 App.xaml.cs。通过将你的 URL 的“azure-mobile.net” 替换为“azure-mobile.xxx” ，将 MobileServiceClient  的初始化更改为一个无效的地址。然后，保存文件。

         public static MobileServiceClient MobileService = new MobileServiceClient(
        "https://your-mobile-service.azure-mobile.xxx/",
        "AppKey"
        );

2.  在 Visual Studio 中，按 F5  生成并运行应用程序。输入新的 Todo 项目，然后单击“保存” 。新的 Todo 项目在推送到移动服务之前，只存在于本地存储中。客户端应用程序的行为就像它已连接到支持所有创建、读取、更新、删除 (CRUD) 操作的移动服务一样。

    ![][2]

3.  关闭应用程序并重新启动它，以验证你创建的新项目是否已永久保存到本地存储中。

<a name="update-online-app"></a>
## 更新应用程序以重新连接移动服务

在本节中，你会将应用程序重新连接到移动服务。这模拟的是通过移动服务从脱机状态转为联机状态的应用程序。

1.  在 Visual Studio 的解决方案资源管理器中，打开 App.xaml.cs。通过将你的 URL 的“azure-mobile.xxx” 替换为“azure-mobile.net” ，将 MobileServiceClient  的初始化更改回正确的地址。然后，保存文件。

         public static MobileServiceClient MobileService = new MobileServiceClient(
        "https://your-mobile-service.azure-mobile.net/",
        "Your AppKey"
        );

<a name="test-online-app"></a>
## 测试已连接到移动服务的应用程序

在本节中，你将测试推送和拉取操作，以便将本地存储与移动服务数据库同步。

1.  在 Visual Studio 中，按 F5  键重新生成并运行应用程序。请注意，数据看上去与脱机情况下相同，即使应用程序现已连接到移动服务。这是因为此应用程序始终使用 `IMobileServiceSyncTable`，后者指向本地存储。

    ![][3]

2.  登录到 Microsoft Azure 管理门户，查看你的移动服务数据库。如果你的服务使用移动服务的 JavaScript 后端，则可以浏览移动服务的“数据” 选项卡中的数据。如果你使用的是移动服务的 .NET 后端，则可以单击 SQL Azure 扩展中数据库的“管理” 按钮，以便对表执行查询。

    请注意，数据尚未在数据库和本地存储之间进行同步。

    ![][4]

3.  在应用程序中，按“推送” 按钮。这会导致应用程序依次调用 `MobileServiceClient.SyncContext.PushAsync` 和 `RefreshTodoItems`，以使用本地存储中的项目刷新应用程序。此推送操作将导致移动服务数据库从存储接收数据。但是，本地存储不会从移动服务数据库接收项目。

    推送操作从 `MobileServiceClient.SyncContext` 而非 `IMobileServicesSyncTable` 执行，并将更改推送到与该同步上下文关联的所有表中。这是为了应对表之间存在关系的情况。

    ![][5]

4.  在应用程序中将几个新项目添加到本地存储中。

    ![][6]

5.  这次在应用程序中按“拉取” 按钮。应用程序只调用 `IMobileServiceSyncTable.PullAsync()` 和 `RefreshTodoItems`。请注意，移动服务数据库中的所有数据均已拉入本地存储，并显示在应用程序中。但另请注意，本地存储中的所有数据仍推送到了移动服务数据库中。这是因为"拉取时始终先执行推送操作"。

    ![][7]

    ![][8]

## 摘要

为了支持移动服务的脱机功能，我们使用了 `IMobileServiceSyncTable` 接口，并使用本地存储初始化了 `MobileServiceClient.SyncContext`。在这种情况下，本地存储是一个 SQLite 数据库。

移动服务的正常 CRUD 操作执行起来就像应用程序仍处于连接状态一样，但所有操作都针对本地存储进行。

当我们想要将本地存储与服务器同步时，我们使用了 `IMobileServiceSyncTable.PullAsync` 和 `MobileServiceClient.SyncContext.PushAsync` 方法。

-   为了将更改推送到服务器中，我们调用了 `IMobileServiceSyncContext.PushAsync()`。此方法属于 `IMobileServicesSyncContext` 而不是同步表，因为它会将更改推送到所有表中：

    只有已在本地以某种方式修改（通过 CRUD 操作来完成）的记录才会发送到服务器。

-   为了从服务器上的表中将数据拉取到应用程序，我们调用了 `IMobileServiceSyncTable.PullAsync`。

    拉取时始终先发出推送操作。

    此外还有 "PullAsync()" 的重载，它允许指定查询。请注意，在移动服务的脱机支持的预览版中，"PullAsync" 将读取相应表（或查询）中的所有行 - 例如，它并不会只读取比上次同步更新的行。如果行已存在于本地同步表中，则这些行会保持不变。

## 后续步骤

-   [使用移动服务脱机支持处理冲突][]

  [Windows 应用商店 C\#]: /zh-cn/documentation/articles/mobile-services-windows-store-dotnet-get-started-offline-data "Windows 应用商店 C#"
  [Windows Phone]: /zh-cn/documentation/articles/mobile-services-windows-phone-get-started-offline-data "Windows Phone"
  [移动服务入门]: /zh-cn/documentation/articles/mobile-services-windows-store-get-started/
  [数据入门]: /zh-cn/documentation/articles/mobile-services-dotnet-backend-windows-store-dotnet-get-started-data/
  [更新应用程序以支持脱机功能]: #enable-offline-app
  [在脱机情况下测试应用程序]: #test-offline-app
  [更新应用程序以重新连接移动服务]: #update-online-app
  [测试已连接到移动服务的应用程序]: #test-online-app
  [Azure 免费试用]: http://www.windowsazure.com/zh-cn/pricing/free-trial/?WT.mc_id=AE564AB28
  [脱机示例入门]: http://go.microsoft.com/fwlink/?LinkId=394777
  [SQLite for Windows 8.1]: http://go.microsoft.com/fwlink/?LinkId=394776
  []: ./media/mobile-services-windows-store-dotnet-get-started-offline-data/mobile-services-add-reference-sqlite-dialog.png
  [1]: ./media/mobile-services-windows-store-dotnet-get-started-offline-data/mobile-services-sqlitestore-nuget.png
  [2]: ./media/mobile-services-windows-store-dotnet-get-started-offline-data/mobile-services-offline-app-run1.png
  [3]: ./media/mobile-services-windows-store-dotnet-get-started-offline-data/mobile-services-online-app-run1.png
  [4]: ./media/mobile-services-windows-store-dotnet-get-started-offline-data/mobile-data-browse.png
  [5]: ./media/mobile-services-windows-store-dotnet-get-started-offline-data/mobile-data-browse2.png
  [6]: ./media/mobile-services-windows-store-dotnet-get-started-offline-data/mobile-services-online-app-run2.png
  [7]: ./media/mobile-services-windows-store-dotnet-get-started-offline-data/mobile-services-online-app-run3.png
  [8]: ./media/mobile-services-windows-store-dotnet-get-started-offline-data/mobile-data-browse3.png
  [使用移动服务脱机支持处理冲突]: /zh-cn/documentation/articles/mobile-services-windows-store-dotnet-handling-conflicts-offline-data/
