<properties linkid="develop-mobile-tutorials-dotnet-backend-get-started-with-data-wp8" urlDisplayName="Get Started with Data" pageTitle="Get started with data (Windows Phone) | Mobile Dev Center" metaKeywords="" description="Learn how to get started using Mobile Services to leverage data in your Windows Phone app." metaCanonical="" services="" documentationCenter="Mobile" title="Get started with data in Mobile Services" authors="wesmc" solutions="" manager="" editor="" />

# 移动服务中的数据处理入门

<div class="dev-center-tutorial-selector sublanding"><a href="/zh-cn/documentation/articles/mobile-services-dotnet-backend-windows-store-dotnet-get-started-data/" title="Windows Store C#">Windows 应用商店 C#</a><a href="/zh-cn/documentation/articles/mobile-services-dotnet-backend-windows-store-javascript-get-started-data/" title="Windows Store JavaScript">Windows 应用商店 JavaScript</a><a href="/zh-cn/documentation/articles/mobile-services-dotnet-backend-windows-phone-get-started-data/" title="Windows Phone" class="current">Windows Phone</a></div>
<div class="dev-center-tutorial-subselector"><a href="/zh-cn/documentation/articles/mobile-services-dotnet-backend-windows-phone-get-started-data/" title=".NET backend" class="current">.NET 后端</a> | <a href="/zh-cn/documentation/articles/mobile-services-windows-phone-get-started-data/"  title="JavaScript backend">JavaScript 后端</a></div>

本主题说明如何使用 Azure 移动服务作为 Windows Phone 8 应用程序的后端数据源。在本教程中，你将要为某个应用程序（该应用程序在内存中存储数据）下载一个 Visual Studio 2012 项目，创建一个新的移动服务，将该移动服务与该应用程序相集成，并查看运行该应用程序时对数据所做的更改。

在本教程中创建的移动服务支持移动服务中的 .NET 运行时。这样，你便可以将 .NET 语言和 Visual Studio 用于移动服务中的服务器端业务逻辑。若要创建允许以 JavaScript 编写服务器端业务逻辑的移动服务，请参阅本主题中的 [JavaScript 后端版本][]。

本教程将指导你完成以下基本步骤：

1.  [下载 Windows Phone 8 应用程序项目][]
2.  [创建新的移动服务][]
3.  [在本地下载移动服务][]
4.  [更新 Windows Phone 应用程序以使用移动服务][]
5.  [针对本地托管的服务测试 Windows Phone 应用程序][]
6.  [将移动服务发布到 Azure][]
7.  [针对 Azure 中托管的服务测试 Windows Phone 应用程序][]

本教程要求在 Windows 8 上运行 Visual Studio 2012 和 [Windows Phone 8 SDK][]。若要完成本教程以创建 Windows Phone 8.1 应用程序，必须使用 Visual Studio 2013 Update 2 或更高版本。

> [WACOM.NOTE]若要完成本教程，你需要一个 Azure 帐户。如果你没有帐户，只需花费几分钟就能创建一个免费试用帐户。有关详细信息，请参阅 [Azure 免费试用][]。

<a name="download-app"></a>
## 下载 GetStartedWithData 项目

本教程是在 [GetStartedWithMobileServices 应用程序][]的基础上制作的，该应用程序是一个 Windows Phone Silverlight 8 应用程序项目。此应用程序的 UI 与移动服务快速入门中生成的应用程序类似，不过，前者的一些新增项本地存储在内存中。

1.  从[开发人员代码示例站点][GetStartedWithMobileServices 应用程序]下载 GetStartedWithMobileServices 示例应用程序的 C\# 版本。

    ![][0]

    > [WACOM.NOTE]若要创建 Windows Phone Silverlght 8.1 应用程序，只需在下载的 Windows Phone Silverlight 8 应用程序项目中将目标操作系统更改为 Windows Phone 8.1。若要创建 Windows Phone 应用商店应用程序，请下载 GetStartedWithData 示例应用程序项目的 [Windows Phone 应用商店应用程序版本][]。

2.  右键单击 Visual Studio，然后单击“以管理员身份运行”，以使用管理特权运行 Visual Studio 。

3.  在 Visual Studio 中打开下载的项目，然后检查 MainPage.xaml.cs 文件。

    请注意添加的 "TodoItem" 对象存储在内存中的 "ObservableCollection\<TodoItem\>" 内。

4.  在 Visual Studio 中，选择应用程序的部署目标。你可以部署到 Windows Phone 设备，或者 Windows Phone SDK 随附的模拟器之一。在本教程中，我们将演示如何部署到模拟器。

    ![][1]

5.  按 "F5" 键。随后将生成、部署并启动应用程序以进行调试。

6.  在应用程序中的文本框内键入一些文本，然后单击“保存”以便在内存中保存该应用程序的一些项 。

    ![][2]

    请注意每个 `TodoItem` 的文本显示在刷新按钮的下面，另外还提供了一个复选框，让你将项标记为已完成。

<a name="create-service"></a>
## 创建新的移动服务创建新的移动服务

[WACOM.INCLUDE [mobile-services-dotnet-backend-create-new-service](../includes/mobile-services-dotnet-backend-create-new-service.md)]

<a name="download-the-service-locally"></a>
## 在本地下载服务下载移动服务项目并将其添加到解决方案

1.  在 [Azure 管理门户][]中，单击新建的移动服务或者其云图标选项卡，以转到概述页。

    ![][3]

2.  单击“Windows Phone 8” 平台。在“入门”部分下，展开“连接现有 Windows Phone 8 应用程序”，然后单击“下载”按钮以下载移动服务的个性化初学者项目 。

    ![][4]

3.  另外，请在该部分下，单击以下屏幕快照中所示的链接，以下载你刚刚下载的移动服务的发布配置文件。

    > [WACOM.NOTE] 请将该文件保存在安全位置，因为它包含与你的 Azure 帐户相关的敏感信息。在根据本教程后面的内容发布移动服务后，你将要删除此文件。

    ![][5]

4.  解压缩你下载的个性化服务初学者项目。将 zip 文件中的文件夹复制到“数据处理入门”解决方案文件 (.sln) 所在的同一个 "C\#" 目录。这样可以方便 NuGet 包管理器将所有程序包保持同步。

    ![][6]

5.  在 Visual Studio 的解决方案资源管理器中，右键单击“数据处理入门”Windows 应用商店应用程序对应的解决方案。单击“添加” ，然后单击“现有项目” 。

    ![][7]

6.  在“添加现有项目”对话框中，导航到你已移到 "C\#" 目录中的移动服务项目文件夹。在服务子目录中选择 C\# 项目文件 (.csproj)。单击“打开” 将该项目添加到你的解决方案。

    ![][8]

7.  在 Visual Studio 的解决方案资源管理器中，右键单击你刚添加的服务项目，然后单击“生成”以验证该项目是否能够生成且不出错 。在生成期间，NuGet 包管理器可能需要还原项目中引用的某些 NuGet 包。

    ![][9]

8.  再次右键单击该服务项目。这一次请单击“调试”上下文菜单下的“启动新实例” 。

    ![][10]

    Visual Studio 将打开服务的默认网页。你可以单击“立即尝试”以从默认网页测试移动服务中的方法 。

    ![][11]

    默认情况下，Visual Studio 在 IIS Express 本地托管你的移动服务。在任务栏中右键单击 IIS Express 的任务栏图标即可看到此信息。

    ![][12]

<a name="update-app"></a>
## 更新 Windows Phone 应用程序更新 Windows Phone 应用程序以使用移动服务

在本部分中，你将要更新 Windows Phone 应用程序，以将移动服务用作应用程序的后端服务。

1.  在 Visual Studio 的解决方案资源管理器中，右键单击 Windows Phone 应用程序项目，然后单击“管理 NuGet 包” 。

    ![][13]

2.  在“管理 NuGet 包”对话框中，搜索联机程序包集合中的 "WindowsAzure.MobileServices"，并单击它以安装 Azure 移动服务 Nuget 包。然后关闭该对话框。

    ![][14]

3.  返回到 Azure 管理门户，找到标签为“连接你的应用程序并存储服务中的数据”的步骤 。复制用于创建 `MobileServiceClient` 连接的代码段。

    ![][15]

4.  在 Visual Studio 中打开 App.xaml.cs。在 `App` 类定义的开头位置粘贴该代码段。另外，请将以下 `using` 语句添加到该文件的顶部，然后保存该文件。

        using Microsoft.WindowsAzure.MobileServices;

    ![][16]

5.  在 Visual Studio 中打开 MainPage.xaml.cs，然后在该文件的顶部添加以下 using 语句：

        using Microsoft.WindowsAzure.MobileServices;

6.  在 Visual Studio 中的 MainPage.xaml.cs 内，将 `MainPage` 类定义替换为以下定义，然后保存该文件。

    此代码借助移动服务 SDK 使应用程序将其数据存储在服务提供的表中，而不是本地存储在内存中。三个主要方法为 `InsertTodoItem`、`RefreshTodoItems` 和 `UpdateCheckedTodoItem`。通过这三个方法，你可以在 Azure 中的表内异步插入、查询和更新数据集合。

        public sealed partial class MainPage :PhoneApplicationPage
        {
        private MobileServiceCollection<TodoItem, TodoItem> items;
        private IMobileServiceTable<TodoItem> todoTable = 
        App.MobileService.GetTable<TodoItem>();            
        public MainPage()
            {
        this.InitializeComponent();
            }
        private async void InsertTodoItem(TodoItem todoItem)
            {
        await todoTable.InsertAsync(todoItem); 
        items.Add(todoItem);
            }
        private async void RefreshTodoItems()
            {
        items = await todoTable 
        .ToCollectionAsync(); 
        ListItems.ItemsSource = items;
            }
        private async void UpdateCheckedTodoItem(TodoItem item)
            {
        await todoTable.UpdateAsync(item);      
            }
        private void ButtonRefresh_Click(object sender, RoutedEventArgs e)
            {
        RefreshTodoItems();
            }
        private void ButtonSave_Click(object sender, RoutedEventArgs e)
            {
        var todoItem = new TodoItem { Text = InputText.Text };
        InsertTodoItem(todoItem);
            }
        private void CheckBoxComplete_Checked(object sender, RoutedEventArgs e)
            {
        CheckBox cb = (CheckBox)sender;
        TodoItem item = cb.DataContext as TodoItem;
        item.Complete = (bool)cb.IsChecked;
        UpdateCheckedTodoItem(item);
            }
        protected override void OnNavigatedTo(NavigationEventArgs e)
            {
        RefreshTodoItems();
            }
        }

<a name="test-locally-hosted"></a>
## 在本地测试 Windows Phone 应用程序针对本地托管的服务测试 Windows Phone 应用程序

在本部分中，你将要使用 Visual Studio 来测试位于开发工作站本地的应用程序和移动服务。若要从 Windows Phone 设备或者某个 Windows Phone 模拟器测试 IIS Express 本地托管的移动服务，你必须配置 IIS Express 和工作站，以允许与工作站的 IP 地址和端口建立连接。Windows Phone 设备和模拟器以非本地网络客户端的形式建立连接。

#### 配置 IIS Express 以允许建立远程连接

[WACOM.INCLUDE [mobile-services-how-to-configure-iis-express](../includes/mobile-services-how-to-configure-iis-express.md)]

#### 针对 IIS Express 中的移动服务测试应用程序

1.  在 Visual Studio 中打开 App.xaml.cs 文件，并注释掉你最近粘贴到该文件中的 `MobileService` 定义。添加新定义，以根据你在工作站上配置的 IP 地址和端口建立连接。然后，保存文件。你的代码应类似于...

         public static MobileServiceClient MobileService = new MobileServiceClient(
        "http://192.168.137.72:58203");

        //public static MobileServiceClient MobileService = new MobileServiceClient(
        //    "https://todolist.preview.azure-mobile-preview.net/",
        //    "XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX"
        //);        

2.  在 Visual Studio 中，按 F7 键或者在“生成”菜单中单击“生成解决方案”，以同时生成 Windows Phone 应用程序和移动服务 。在 Visual Studio 的输出窗口中确认是否已生成这两个项目且未出错

    ![][17]

3.  在 Visual Studio 中，按 F5 键或者在“调试”菜单中单击“启动调试”，以运行应用程序并将移动服务托管在 IIS Express 本地 。

    > [WACOM.NOTE] 确保使用“以管理员身份运行”选项运行 Visual Studio 。否则，IIS Express 可能不会加载 applicationhost.config 发生的更改。

    ![][18]

4.  输入新 todoitem 的文本。然后单击“保存” 。这样便会在 IIS Express 本地托管的移动服务所创建的数据库中插入一个新的 todoItem。单击某个项对应的复选框可将它标记为已完成。

    ![][19]

5.  在 Visual Studio 中停止调试应用程序。打开服务器资源管理器并展开“数据连接”即可查看针对后端服务创建的数据库中的更改。右键单击“MS\_TableConnectionString”下的 TodoItems 表，然后单击“显示表数据” 

    ![][20]

6.  测试完本地托管的移动服务后，请删除你创建的、用于打开工作站上端口的 Windows 防火墙规则。

<a name="publish-mobile-service"></a>
## 将移动服务发布到 Azure将移动服务发布到 Azure

[WACOM.INCLUDE [mobile-services-dotnet-backend-publish-service](../includes/mobile-services-dotnet-backend-publish-service.md)]

<a name="test-azure-hosted"></a>
## 测试 Azure 上的移动服务测试已发布到 Azure 的移动服务

1.  在 Visual Studio 中打开 App.xaml.cs。注释掉功能如下的代码：创建与本地托管移动服务建立连接的 `MobileServiceClient`。取消注释功能如下的代码：创建与 Azure 中的服务建立连接的 `MobileServiceClient`。保存对该文件所做的更改。

        sealed partial class App :Application
        {
        //public static MobileServiceClient MobileService = new MobileServiceClient(
        //          "http://192.168.137.72:58203");

        // Use this constructor instead after publishing to the cloud
        public static MobileServiceClient MobileService = new MobileServiceClient(
        "https://todolist.preview.azure-mobile-preview.net/",
        "XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX"
            );        
            ....

2.  在 Visual Studio 中，按 F5 键或者在“调试”菜单中单击“启动调试” 。这样，便会使用运行应用程序之前发生的更改重新生成应用程序，以连接到 Azure 中远程托管的移动服务。

    ![][18]

3.  输入一些新的 todoitem，然后单击“保存”以保存每个 todoitem 。单击复选框以完成其中的某些新项。每个新的 todoItem 将在你前面通过 Azure 管理门户为移动服务配置的 SQL 数据库中存储并更新。

    ![][21]

    你可以重新启动应用程序，以查看更改是否已持久保存在 Azure 中的数据库内。你还可以使用 Azure 管理门户或者 Visual Studio 的 SQL Server 对象资源管理器来检查数据库。后面两个步骤将使用 Azure 管理门户来查看数据库中的更改。

4.  在 Azure 管理门户中，单击与你的移动服务关联的数据库对应的“管理”。

    ![][22]

5.  在管理门户中，执行一个查询以查看应用程序所做的更改。你的查询应类似于以下查询，不过，请使用你的数据库名称而不是 `todolist`。

        SELECT * FROM [todolist].[todoitems]

    ![][23]

"数据处理入门"教程到此结束。

<a name="next-steps"> </a>
## 后续步骤

本教程所述的基础知识演示了如何使 Windows Phone 8 应用程序处理使用 .Net 运行时生成的移动服务中的数据。接下来，建议你完成下列教程之一，这些教程是基于本教程中创建的 GetStartedWithData 应用程序制作的：

-   [使用脚本验证和修改数据][]
    了解更多有关使用移动服务中的服务器脚本验证和更改从应用程序发送的数据的信息。

-   [使用分页优化查询][]
    了解如何使用查询中的分页控制单个请求中处理的数据量。

完成数据系列教程后，请试着学习下列教程之一：

-   [身份验证入门][]
    了解如何对应用程序用户进行身份验证。

-   [移动服务 .NET 操作方法概念性参考][]
    了解有关如何将移动服务与 .NET 一起使用的详细信息。

  [Windows 应用商店 C\#]: /zh-cn/documentation/articles/mobile-services-dotnet-backend-windows-store-dotnet-get-started-data/ "Windows 应用商店 C#"
  [Windows 应用商店 JavaScript]: /zh-cn/documentation/articles/mobile-services-dotnet-backend-windows-store-javascript-get-started-data/ "Windows 应用商店 JavaScript"
  [Windows Phone]: /zh-cn/documentation/articles/mobile-services-dotnet-backend-windows-phone-get-started-data/ "Windows Phone"
  [.NET 后端]: /zh-cn/documentation/articles/mobile-services-dotnet-backend-windows-phone-get-started-data/ ".NET 后端"
  [JavaScript 后端]: /zh-cn/documentation/articles/mobile-services-windows-phone-get-started-data/ "JavaScript 后端"
  [JavaScript 后端版本]: /zh-cn/develop/mobile/tutorials/get-started-with-data-wp8
  [下载 Windows Phone 8 应用程序项目]: #download-app
  [创建新的移动服务]: #create-service
  [在本地下载移动服务]: #download-the-service-locally
  [更新 Windows Phone 应用程序以使用移动服务]: #update-app
  [针对本地托管的服务测试 Windows Phone 应用程序]: #test-locally-hosted
  [将移动服务发布到 Azure]: #publish-mobile-service
  [针对 Azure 中托管的服务测试 Windows Phone 应用程序]: #test-azure-hosted
  [Windows Phone 8 SDK]: http://go.microsoft.com/fwlink/p/?linkid=268374
  [Azure 免费试用]: http://www.windowsazure.com/zh-cn/pricing/free-trial/?WT.mc_id=AE564AB28&returnurl=http%3A%2F%2Fwww.windowsazure.com%2Fen-us%2Fdocumentation%2Farticles%2Fmobile-services-dotnet-backend-windows-store-dotnet-get-started-data%2F
  [GetStartedWithMobileServices 应用程序]: http://go.microsoft.com/fwlink/p/?linkid=271146
  [0]: ./media/mobile-services-dotnet-backend-windows-phone-get-started-data/mobile-data-sample-download-wp8-vs12.png
  [Windows Phone 应用商店应用程序版本]: http://go.microsoft.com/fwlink/p/?LinkId=397372
  [1]: ./media/mobile-services-dotnet-backend-windows-phone-get-started-data/vs-deployment-target.png
  [2]: ./media/mobile-services-dotnet-backend-windows-phone-get-started-data/app-view.png
  [mobile-services-dotnet-backend-create-new-service]: ../includes/mobile-services-dotnet-backend-create-new-service.md
  [Azure 管理门户]: https://manage.windowsazure.cn/
  [3]: ./media/mobile-services-dotnet-backend-windows-phone-get-started-data/mobile-service-overview-page.png
  [4]: ./media/mobile-services-dotnet-backend-windows-phone-get-started-data/download-service-project.png
  [5]: ./media/mobile-services-dotnet-backend-windows-phone-get-started-data/download-publishing-profile.png
  [6]: ./media/mobile-services-dotnet-backend-windows-phone-get-started-data/copy-service-and-packages-folder.png
  [7]: ./media/mobile-services-dotnet-backend-windows-phone-get-started-data/add-service-project-to-solution.png
  [8]: ./media/mobile-services-dotnet-backend-windows-phone-get-started-data/add-existing-project-dialog.png
  [9]: ./media/mobile-services-dotnet-backend-windows-phone-get-started-data/vs-build-service-project.png
  [10]: ./media/mobile-services-dotnet-backend-windows-phone-get-started-data/vs-start-debug-service-project.png
  [11]: ./media/mobile-services-dotnet-backend-windows-phone-get-started-data/service-welcome-page.png
  [12]: ./media/mobile-services-dotnet-backend-windows-phone-get-started-data/iis-express-tray.png
  [13]: ./media/mobile-services-dotnet-backend-windows-phone-get-started-data/vs-manage-nuget-packages.png
  [14]: ./media/mobile-services-dotnet-backend-windows-phone-get-started-data/manage-nuget-packages.png
  [15]: ./media/mobile-services-dotnet-backend-windows-phone-get-started-data/copy-mobileserviceclient-snippet.png
  [16]: ./media/mobile-services-dotnet-backend-windows-phone-get-started-data/vs-pasted-mobileserviceclient.png
  [mobile-services-how-to-configure-iis-express]: ../includes/mobile-services-how-to-configure-iis-express.md
  [17]: ./media/mobile-services-dotnet-backend-windows-phone-get-started-data/vs-build-solution.png
  [18]: ./media/mobile-services-dotnet-backend-windows-phone-get-started-data/vs-run-solution.png
  [19]: ./media/mobile-services-dotnet-backend-windows-phone-get-started-data/local-item-checked.png
  [20]: ./media/mobile-services-dotnet-backend-windows-phone-get-started-data/vs-show-local-table-data.png
  [mobile-services-dotnet-backend-publish-service]: ../includes/mobile-services-dotnet-backend-publish-service.md
  [21]: ./media/mobile-services-dotnet-backend-windows-phone-get-started-data/azure-items.png
  [22]: ./media/mobile-services-dotnet-backend-windows-phone-get-started-data/manage-sql-azure-database.png
  [23]: ./media/mobile-services-dotnet-backend-windows-phone-get-started-data/sql-azure-query.png
  [使用脚本验证和修改数据]: /zh-cn/develop/mobile/tutorials/validate-modify-and-augment-data-wp8
  [使用分页优化查询]: /zh-cn/develop/mobile/tutorials/add-paging-to-data-wp8
  [身份验证入门]: /zh-cn/documentation/articles/mobile-services-dotnet-backend-windows-phone-get-started-users/
  [移动服务 .NET 操作方法概念性参考]: /zh-cn/develop/mobile/how-to-guides/work-with-net-client-library
