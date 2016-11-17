<properties
    pageTitle="在 Xamarin.Forms 应用中连接到 Azure 存储"
    description="连接到 Azure blob 存储向待办事项列表 Xamarin.Forms 移动应用添加图像"
    documentationCenter="xamarin"
    authors="lindydonna"
    manager="erikre"
    editor=""
    services="app-service\mobile"/>

<tags
	ms.service="app-service-mobile"
	ms.date="05/10/2016"
	wacn.date="09/26/2016"/>

#在 Xamarin.Forms 应用中连接到 Azure 存储

## 概述

Azure 移动应用客户端和服务器 SDK 支持对结构化数据（包含对 /tables 终结点进行的 CRUD 操作）进行脱机同步。通常，此数据存储在数据库或类似的存储中，并且这些数据存储通常不能有效地存储大型二进制数据。此外，某些应用程序有相关数据存储在其他位置（例如，Blob 存储、文件共享），若能在 /tables 终结点中的记录与其他数据之间创建关联，会很有用。

本主题演示如何向移动应用待办事项列表快速入门项目中添加图像支持。必须先完成教程[创建 Xamarin.Forms 应用]。

在本教程中，将创建一个存储帐户，并将连接字符串添加到移动应用后端。然后将从新的移动应用类型 `StorageController<T>` 将新继承项添加到服务器项目。

>[AZURE.TIP] 本教程提供了可用的[配套示例](https://azure.microsoft.com/documentation/samples/app-service-mobile-dotnet-todo-list-files/)，用户可将其部署到自己的 Azure 帐户。

## 先决条件

* 完成[创建 Xamarin.Forms 应用]教程，其中列出了其他先决条件。本文使用该教程中完成的应用。

>[AZURE.NOTE] 如果要在注册 Azure 帐户之前就开始使用 Azure 应用服务，请转到 [Try App Service](https://tryappservice.azure.com/?appServiceName=mobile)（试用应用服务）。在那里，可以立即在应用服务中创建短期的入门级移动应用 - 无需信用卡，也无需做出承诺。

## 创建存储帐户

1. 按照教程[创建 Azure 存储帐户]创建存储帐户。

2. 在 Azure 门户中，导航到新创建的存储帐户并单击“密钥”图标。复制**主连接字符串**。

3. 导航到移动应用后端。在“所有设置”->“应用程序设置”->“连接字符串”下，创建名为 `MS_AzureStorageAccountConnectionString` 的新键并使用从存储帐户复制的值。使用“自定义”作为键类型。

## 向服务器添加存储控制器

需要将一个新的控制器添加到服务器项目，以响应 Azure 存储的 SAS 令牌请求，以及返回与记录对应的文件列表：

- [向服务器项目添加存储控制器](#add-controller-code)
- [存储控制器注册的路由](#routes-registered)
- [客户端和服务器通信](#client-communication)

###<a name="add-controller-code"></a>向服务器项目添加存储控制器

1. 在 Visual Studio 中，打开 .NET 服务器项目。添加 NuGet 包 [Microsoft.Azure.Mobile.Server.Files]。请务必选择“包括预发行版”。

2. 在 Visual Studio 中，打开 .NET 服务器项目。右键单击“控制器”文件夹，然后选择“添加”->“控制器”->“Web API 2 控制器 - 空”。将该控制器命名为 `TodoItemStorageController`。

3. 添加以下 using 语句：

        using Microsoft.Azure.Mobile.Server.Files;
        using Microsoft.Azure.Mobile.Server.Files.Controllers;

4. 将基类更改为 `StorageController`：
    
        public class TodoItemStorageController : StorageController<TodoItem>

5. 将以下方法添加到类：

        [HttpPost]
        [Route("tables/TodoItem/{id}/StorageToken")]
        public async Task<HttpResponseMessage> PostStorageTokenRequest(string id, StorageTokenRequest value)
        {
            StorageToken token = await GetStorageTokenAsync(id, value);

            return Request.CreateResponse(token);
        }

        // Get the files associated with this record
        [HttpGet]
        [Route("tables/TodoItem/{id}/MobileServiceFiles")]
        public async Task<HttpResponseMessage> GetFiles(string id)
        {
            IEnumerable<MobileServiceFile> files = await GetRecordFilesAsync(id);

            return Request.CreateResponse(files);
        }

        [HttpDelete]
        [Route("tables/TodoItem/{id}/MobileServiceFiles/{name}")]
        public Task Delete(string id, string name)
        {
            return base.DeleteFileAsync(id, name);
        }

6. 更新 Web API 配置，设置属性路由。在 **Startup.MobileApp.cs** 中，将以下代码行添加到 `ConfigureMobileApp()` 方法，在 `config` 变量的定义后：

        config.MapHttpAttributeRoutes();

7. 将服务器项目发布到移动应用后端。

###<a name="routes-registered"></a>存储控制器注册的路由

新 `TodoItemStorageController` 在它管理的记录下公开了两个子资源：

- StorageToken

    + HTTP POST：创建存储令牌
    
        `/tables/TodoItem/{id}/MobileServiceFiles`
    
- MobileServiceFiles

    + HTTP GET：检索与记录关联的文件列表
    
        `/tables/TodoItem/{id}/MobileServiceFiles`

    + HTTP DELETE：删除文件资源标识符中指定的文件
    
        `/tables/TodoItem/{id}/MobileServiceFiles/{fileid}`

###<a name="client-communication"></a>客户端和服务器通信

请注意，`TodoItemStorageController` *未* 提供用于上载或下载 blob 的路由。这是因为移动客户端在先获取 SAS 令牌（共享访问签名）以安全地访问特定 blob 或容器后， *直接* 与 blob 存储交互以执行这些操作。这是重要的体系结构设计，因为以其他方式访问存储会受到移动后端的可伸缩性和可用性的限制。移动客户端通过直接连接到 Azure 存储，可以充分利用其功能，如自动分区和地理分布。

共享访问签名对存储帐户中的资源提供委托访问。这意味着你可以授权客户端在指定时间段内，以一组指定权限有限地访问你的存储帐户中的对象，而不必共享你的帐户访问密钥。若要了解详细信息，请参阅[了解共享访问签名]。

下图显示了客户端和服务器交互。上载文件之前，客户端通过服务请求 SAS 令牌。服务使用存储连接字符串生成新的 SAS，然后将其返回到客户端。SAS 有时间限制，并将权限仅限于特定文件或容器。然后，移动客户端使用此 SAS 和 Azure 存储客户端 SDK 将文件上载到 blob 存储。

![请求 SAS 令牌](./media/app-service-mobile-xamarin-forms-blob-storage/storage-token-diagram.png)

## 更新客户端应用，添加图像支持

在 Visual Studio 或 Xamarin Studio 中打开 Xamarin.Forms 快速入门项目。将安装 NuGet 包并更新可移植库项目以及 iOS、Android 和 Windows 客户端项目：

- [添加 NuGet 包](#add-nuget)
- [添加 IPlatform 接口](#add-iplatform)
- [添加 FileHelper 类](#add-filehelper)
- [添加文件同步处理程序](#file-sync-handler)
- [更新 TodoItemManager](#update-todoitemmanager)
- [添加详细信息视图](#add-details-view)
- [更新主视图](#update-main-view)
- [更新 Android 项目](#update-android)、[iOS 项目](#update-ios)、[Windows 项目](#update-windows)

>[AZURE.NOTE] 本教程中仅包含有关 Android、iOS 和 Windows 应用商店平台的说明，而不包含有关 Windows Phone 的说明。

###<a name="add-nuget"></a>添加 NuGet 包

右键单击解决方案，然后选择“管理解决方案的 NuGet 程序包”。将以下 NuGet 包添加到解决方案中的**所有**项目。请务必选中“包括预发行版”。

  - [Microsoft.Azure.Mobile.Client.Files]

  - [Microsoft.Azure.Mobile.Client.SQLiteStore]

  - [PCLStorage]

为方便起见，本示例使用 [PCLStorage] 库，但它并不是 Azure 移动应用客户端 SDK 所必需的。

[PCLStorage]: https://www.nuget.org/packages/PCLStorage/

###<a name="add-iplatform"></a>添加 IPlatform 接口

在主要的可移植库项目中创建新接口 `IPlatform`。它使用 [Xamarin.Forms DependencyService] 模式，在运行时加载特定于相应平台的类。以后将在每个客户端项目中添加平台特定的实现。

1. 添加以下 using 语句：

        using Microsoft.WindowsAzure.MobileServices.Files;
        using Microsoft.WindowsAzure.MobileServices.Files.Metadata;
        using Microsoft.WindowsAzure.MobileServices.Sync;

2. 使用以下代码替换实现：

        public interface IPlatform
        {
            Task <string> GetTodoFilesPathAsync();

            Task<IMobileServiceFileDataSource> GetFileDataSource(MobileServiceFileMetadata metadata);

            Task<string> TakePhotoAsync(object context);

            Task DownloadFileAsync<T>(IMobileServiceSyncTable<T> table, MobileServiceFile file, string filename);
        }

###<a name="add-filehelper"></a>添加 FileHelper 类

1. 在主要的可移植库项目中创建新类 `FileHelper`。添加以下 using 语句：

        using System.IO;
        using PCLStorage;
        using System.Threading.Tasks;
        using Xamarin.Forms;

2. 添加类定义：

        public class FileHelper
        {
            public static async Task<string> CopyTodoItemFileAsync(string itemId, string filePath)
            {
                IFolder localStorage = FileSystem.Current.LocalStorage;

                string fileName = Path.GetFileName(filePath);
                string targetPath = await GetLocalFilePathAsync(itemId, fileName);

                var sourceFile = await localStorage.GetFileAsync(filePath);
                var sourceStream = await sourceFile.OpenAsync(FileAccess.Read);

                var targetFile = await localStorage.CreateFileAsync(targetPath, CreationCollisionOption.ReplaceExisting);

                using (var targetStream = await targetFile.OpenAsync(FileAccess.ReadAndWrite)) {
                    await sourceStream.CopyToAsync(targetStream);
                }

                return targetPath;
            }

            public static async Task<string> GetLocalFilePathAsync(string itemId, string fileName)
            {
                IPlatform platform = DependencyService.Get<IPlatform>();

                string recordFilesPath = Path.Combine(await platform.GetTodoFilesPathAsync(), itemId);

                    var checkExists = await FileSystem.Current.LocalStorage.CheckExistsAsync(recordFilesPath);
                    if (checkExists == ExistenceCheckResult.NotFound) {
                        await FileSystem.Current.LocalStorage.CreateFolderAsync(recordFilesPath, CreationCollisionOption.ReplaceExisting);
                    }

                return Path.Combine(recordFilesPath, fileName);
            }

            public static async Task DeleteLocalFileAsync(Microsoft.WindowsAzure.MobileServices.Files.MobileServiceFile fileName)
            {
                string localPath = await GetLocalFilePathAsync(fileName.ParentId, fileName.Name);
                var checkExists = await FileSystem.Current.LocalStorage.CheckExistsAsync(localPath);

                if (checkExists == ExistenceCheckResult.FileExists) {
                    var file = await FileSystem.Current.LocalStorage.GetFileAsync(localPath);
                    await file.DeleteAsync();
                }
            }
        }

###<a name="file-sync-handler"></a> 添加文件同步处理程序

在主要的可移植库项目中创建新类 `TodoItemFileSyncHandler`。此类包含 Azure SDK 中的回调，以便在添加或删除文件时通知代码。

Azure 移动客户端 SDK 不实际存储任何文件数据：客户端 SDK 调用 `IFileSyncHandler` 的实现，该实现确定是否将文件存储在本地设备上以及存储方式。

1. 添加以下 using 语句：

        using System.Threading.Tasks;
        using Microsoft.WindowsAzure.MobileServices.Files.Sync;
        using Microsoft.WindowsAzure.MobileServices.Files;
        using Microsoft.WindowsAzure.MobileServices.Files.Metadata;
        using Xamarin.Forms;

2. 将类定义替换为以下代码：

        public class TodoItemFileSyncHandler : IFileSyncHandler
        {
            private readonly TodoItemManager todoItemManager;

            public TodoItemFileSyncHandler(TodoItemManager itemManager)
            {
                this.todoItemManager = itemManager;
            }

            public Task<IMobileServiceFileDataSource> GetDataSource(MobileServiceFileMetadata metadata)
            {
                IPlatform platform = DependencyService.Get<IPlatform>();
                return platform.GetFileDataSource(metadata);
            }

            public async Task ProcessFileSynchronizationAction(MobileServiceFile file, FileSynchronizationAction action)
            {
                if (action == FileSynchronizationAction.Delete) {
                    await FileHelper.DeleteLocalFileAsync(file);
                }
                else { // Create or update. We're aggressively downloading all files.
                    await this.todoItemManager.DownloadFileAsync(file);
                }
            }
        }

###<a name="update-todoitemmanager"></a>更新 TodoItemManager

1. 在 **TodoItemManager.cs** 中，取消注释行 `#define OFFLINE_SYNC_ENABLED`。

2. 在 **TodoItemManager.cs** 中，添加以下 using 语句：

        using System.IO;
        using Xamarin.Forms;
        using Microsoft.WindowsAzure.MobileServices.Files;
        using Microsoft.WindowsAzure.MobileServices.Files.Sync;
        using Microsoft.WindowsAzure.MobileServices.Eventing;

3. 在 `TodoItemManager` 的构造函数中调用 `DefineTable()` 的后面添加以下代码：

        // Initialize file sync
        this.client.InitializeFileSyncContext(new TodoItemFileSyncHandler(this), store);

4. 在构造函数中，将调用 `InitializeAsync` 替换为以下代码。这可确保在本地存储中修改记录时没有回调。文件同步功能使用这些回调来触发文件同步处理程序。

        this.client.SyncContext.InitializeAsync(store, StoreTrackingOptions.NotifyLocalAndServerOperations);

5. 在 `SyncAsync()` 中调用 `PushAsync()` 的后面添加以下代码：

        await this.todoTable.PushFileChangesAsync();

6. 将以下方法添加到 `TodoItemManager`：

        internal async Task DownloadFileAsync(MobileServiceFile file)
        {
            var todoItem = await todoTable.LookupAsync(file.ParentId);
            IPlatform platform = DependencyService.Get<IPlatform>();

            string filePath = await FileHelper.GetLocalFilePathAsync(file.ParentId, file.Name); 
            await platform.DownloadFileAsync(this.todoTable, file, filePath);
        }

        internal async Task<MobileServiceFile> AddImage(TodoItem todoItem, string imagePath)
        {
            string targetPath = await FileHelper.CopyTodoItemFileAsync(todoItem.Id, imagePath);
            return await this.todoTable.AddFileAsync(todoItem, Path.GetFileName(targetPath));
        }

        internal async Task DeleteImage(TodoItem todoItem, MobileServiceFile file)
        {
            await this.todoTable.DeleteFileAsync(file);
        }

        internal async Task<IEnumerable<MobileServiceFile>> GetImageFilesAsync(TodoItem todoItem)
        {
            return await this.todoTable.GetFilesAsync(todoItem);
        }

###<a name="add-details-view"></a>添加详细信息视图

在本部分中，将为待办事项添加新的详细信息视图。该视图在用户选择待办事项时创建，并且它允许将新图像添加到待办事项。

1. 将新类 **TodoItemImage** 添加到可移植库项目，该类具有以下实现：

        public class TodoItemImage : INotifyPropertyChanged
        {
            private string name;
            private string uri;

            public MobileServiceFile File { get; private set; }

            public string Name
            {
                get { return name; }
                set
                {
                    name = value;
                    OnPropertyChanged(nameof(Name));
                }
            }

            public string Uri
            {
                get { return uri; }      
                set
                {
                    uri = value;
                    OnPropertyChanged(nameof(Uri));
                }
            }

            public TodoItemImage(MobileServiceFile file, TodoItem todoItem)
            {
                Name = file.Name;
                File = file;

                FileHelper.GetLocalFilePathAsync(todoItem.Id, file.Name).ContinueWith(x => this.Uri = x.Result);
            }

            public event PropertyChangedEventHandler PropertyChanged;

            private void OnPropertyChanged(string propertyName)
            {
                PropertyChanged?.Invoke(this, new PropertyChangedEventArgs(propertyName));
            }
        }

2. 编辑 **App.cs**。将 `MainPage` 的初始化替换为以下代码：
    
        MainPage = new NavigationPage(new TodoList());

3. 在 **App.cs** 中，添加以下属性：

        public static object UIContext { get; set; }

4. 右键单击可移植库项目并选择“添加”->“新建项”->“跨平台”->“Forms Xaml 页”。将视图命名为 `TodoItemDetailsView`。

5. 打开 **TodoItemDetailsView.xaml**，将 ContentPage 的正文替换为以下内容：

          <Grid>
            <Grid.RowDefinitions>
              <RowDefinition Height="Auto"/>
              <RowDefinition Height="Auto"/>
              <RowDefinition Height="*"/>
            </Grid.RowDefinitions>
            <Button Clicked="OnAdd" Text="Add image"></Button>
            <ListView x:Name="imagesList"
                      ItemsSource="{Binding Images}"
                      IsPullToRefreshEnabled="false"
                      Grid.Row="2">
              <ListView.ItemTemplate>
                <DataTemplate>
                  <ImageCell ImageSource="{Binding Uri}"
                             Text="{Binding Name}">
                  </ImageCell>
                </DataTemplate>
              </ListView.ItemTemplate>
            </ListView>
          </Grid>

6. 编辑 **TodoItemDetailsView.xaml.cs**，添加以下 using 语句：

        using System.Collections.ObjectModel;
        using Microsoft.WindowsAzure.MobileServices.Files;

7. 将 `TodoItemDetailsView` 的实现替换为以下内容：

        public partial class TodoItemDetailsView : ContentPage
        {
            private TodoItemManager manager;

            public TodoItem TodoItem { get; set; }        
            public ObservableCollection<TodoItemImage> Images { get; set; }

            public TodoItemDetailsView(TodoItem todoItem, TodoItemManager manager)
            {
                InitializeComponent();
                this.Title = todoItem.Name;

                this.TodoItem = todoItem;
                this.manager = manager;

                this.Images = new ObservableCollection<TodoItemImage>();
                this.BindingContext = this;
            }

            public async Task LoadImagesAsync()
            {
                IEnumerable<MobileServiceFile> files = await this.manager.GetImageFilesAsync(TodoItem);
                this.Images.Clear();

                foreach (var f in files) {
                    var todoImage = new TodoItemImage(f, this.TodoItem);
                    this.Images.Add(todoImage);
                }
            }

            public async void OnAdd(object sender, EventArgs e)
            {
                IPlatform mediaProvider = DependencyService.Get<IPlatform>();
                string sourceImagePath = await mediaProvider.TakePhotoAsync(App.UIContext);

                if (sourceImagePath != null) {
                    MobileServiceFile file = await this.manager.AddImage(this.TodoItem, sourceImagePath);

                    var image = new TodoItemImage(file, this.TodoItem);
                    this.Images.Add(image);
                }
            }
        }

###<a name="update-main-view"></a>更新主视图 

更新主视图，以便在用户选择某个待办事项时打开详细信息视图。

在 **TodoList.xaml.cs** 中，将 `OnSelected` 的实现替换为以下内容：

    public async void OnSelected(object sender, SelectedItemChangedEventArgs e)
    {
        var todo = e.SelectedItem as TodoItem;

        if (todo != null) {
            var detailsView = new TodoItemDetailsView(todo, manager);
            await detailsView.LoadImagesAsync();
            await Navigation.PushAsync(detailsView);
        }

        todoList.SelectedItem = null;
    }

###<a name="update-android"></a>更新 Android 项目

将平台特定的代码添加到 Android 项目，包括用于下载文件和使用摄像头捕获新图像的代码。

此代码使用 Xamarin.Forms [DependencyService](https://developer.xamarin.com/guides/xamarin-forms/dependency-service/)，在运行时加载特定于相应平台的类。

1. 将组件 **Xamarin.Mobile** 添加到 Android 项目。

2. 添加具有以下实现的新类 `DroidPlatform`。将“YourNamespace”替换为项目的主命名空间。

        using System;
        using System.IO;
        using System.Threading.Tasks;
        using Android.Content;
        using Microsoft.WindowsAzure.MobileServices.Files;
        using Microsoft.WindowsAzure.MobileServices.Files.Metadata;
        using Microsoft.WindowsAzure.MobileServices.Files.Sync;
        using Microsoft.WindowsAzure.MobileServices.Sync;
        using Xamarin.Media;

        [assembly: Xamarin.Forms.Dependency(typeof(YourNamespace.Droid.DroidPlatform))]
        namespace YourNamespace.Droid
        {
            public class DroidPlatform : IPlatform
            {
                public async Task DownloadFileAsync<T>(IMobileServiceSyncTable<T> table, MobileServiceFile file, string filename)
                {
                    var path = await FileHelper.GetLocalFilePathAsync(file.ParentId, file.Name);
                    await table.DownloadFileAsync(file, path);
                }

                public async Task<IMobileServiceFileDataSource> GetFileDataSource(MobileServiceFileMetadata metadata)
                {
                    var filePath = await FileHelper.GetLocalFilePathAsync(metadata.ParentDataItemId, metadata.FileName);
                    return new PathMobileServiceFileDataSource(filePath);
                }

                public async Task<string> TakePhotoAsync(object context)
                {
                    try {
                        var uiContext = context as Context;
                        if (uiContext != null) {
                            var mediaPicker = new MediaPicker(uiContext);
                            var photo = await mediaPicker.TakePhotoAsync(new StoreCameraMediaOptions());

                            return photo.Path;
                        }
                    }
                    catch (TaskCanceledException) {
                    }

                    return null;
                }

                public Task<string> GetTodoFilesPathAsync()
                {
                    string appData = Environment.GetFolderPath(Environment.SpecialFolder.MyDocuments);
                    string filesPath = Path.Combine(appData, "TodoItemFiles");

                    if (!Directory.Exists(filesPath)) {
                        Directory.CreateDirectory(filesPath);
                    }

                    return Task.FromResult(filesPath);
                }
            }
        }

3. 编辑 **MainActivity.cs**。在 `OnCreate` 中调用 `LoadApplication()` 的前面添加以下代码：

        App.UIContext = this;

###<a name="update-ios"></a>更新 iOS 项目

将平台特定的代码添加到 iOS 项目。

1. 将组件 **Xamarin.Mobile** 添加到 iOS 项目。

2. 添加具有以下实现的新类 `TouchPlatform`。将“YourNamespace”替换为项目的主命名空间。

        using System;
        using System.Collections.Generic;
        using System.IO;
        using System.Text;
        using System.Threading.Tasks;
        using Microsoft.WindowsAzure.MobileServices.Files;
        using Microsoft.WindowsAzure.MobileServices.Files.Metadata;
        using Microsoft.WindowsAzure.MobileServices.Files.Sync;
        using Microsoft.WindowsAzure.MobileServices.Sync;
        using Xamarin.Media;

        [assembly: Xamarin.Forms.Dependency(typeof(YourNamespace.iOS.TouchPlatform))]
        namespace YourNamespace.iOS
        {
            class TouchPlatform : IPlatform
            {
                public async Task DownloadFileAsync<T>(IMobileServiceSyncTable<T> table, MobileServiceFile file, string filename)
                {
                    var path = await FileHelper.GetLocalFilePathAsync(file.ParentId, file.Name);
                    await table.DownloadFileAsync(file, path);
                }

                public async Task<IMobileServiceFileDataSource> GetFileDataSource(MobileServiceFileMetadata metadata)
                {
                    var filePath = await FileHelper.GetLocalFilePathAsync(metadata.ParentDataItemId, metadata.FileName);
                    return new PathMobileServiceFileDataSource(filePath);
                }

                public async Task<string> TakePhotoAsync(object context)
                {
                    try {
                        var mediaPicker = new MediaPicker();
                        var mediaFile = await mediaPicker.PickPhotoAsync();
                        return mediaFile.Path;
                    }
                    catch (TaskCanceledException) {
                        return null;
                    }
                }

                public Task<string> GetTodoFilesPathAsync()
                {
                    string filesPath = Path.Combine(Environment.GetFolderPath(Environment.SpecialFolder.MyDocuments), "TodoItemFiles");

                    if (!Directory.Exists(filesPath)) {
                        Directory.CreateDirectory(filesPath);
                    }

                    return Task.FromResult(filesPath);
                }
            }
        }

3. 编辑 **AppDelegate.cs**，取消调用 `SQLitePCL.CurrentPlatform.Init()` 的注释。

###<a name="update-windows"></a>更新 Windows 项目

1. 安装 Visual Studio 扩展 [SQLite for Windows 8.1](http://go.microsoft.com/fwlink/?LinkID=716919)。有关详细信息，请参阅教程[为 Windows 应用启用脱机同步](/documentation/articles/app-service-mobile-windows-store-dotnet-get-started-offline-data/)。

2. 编辑 **Package.appxmanifest**，检查**网络摄像头**功能。

3. 添加具有以下实现的新类 `WindowsStorePlatform`。将“YourNamespace”替换为项目的主命名空间。

        using System;
        using System.Threading.Tasks;
        using Microsoft.WindowsAzure.MobileServices.Files;
        using Microsoft.WindowsAzure.MobileServices.Files.Metadata;
        using Microsoft.WindowsAzure.MobileServices.Files.Sync;
        using Microsoft.WindowsAzure.MobileServices.Sync;
        using Windows.Foundation;
        using Windows.Media.Capture;
        using Windows.Storage;
        using YourNamespace;

        [assembly: Xamarin.Forms.Dependency(typeof(WinApp.WindowsStorePlatform))]
        namespace WinApp
        {
            public class WindowsStorePlatform : IPlatform
            {
                public async Task DownloadFileAsync<T>(IMobileServiceSyncTable<T> table, MobileServiceFile file, string filename)
                {
                    var path = await FileHelper.GetLocalFilePathAsync(file.ParentId, file.Name);
                    await table.DownloadFileAsync(file, path);
                }

                public async Task<IMobileServiceFileDataSource> GetFileDataSource(MobileServiceFileMetadata metadata)
                {
                    var filePath = await FileHelper.GetLocalFilePathAsync(metadata.ParentDataItemId, metadata.FileName);
                    return new PathMobileServiceFileDataSource(filePath);
                }

                public async Task<string> GetTodoFilesPathAsync()
                {
                    var storageFolder = ApplicationData.Current.LocalFolder;
                    var filePath = "TodoItemFiles";

                    var result = await storageFolder.TryGetItemAsync(filePath);

                    if (result == null) {
                        result = await storageFolder.CreateFolderAsync(filePath);
                    }

                    return result.Name; // later operations will use relative paths
                }

                public async Task<string> TakePhotoAsync(object context)
                {
                    try {
                        CameraCaptureUI dialog = new CameraCaptureUI();
                        Size aspectRatio = new Size(16, 9);
                        dialog.PhotoSettings.CroppedAspectRatio = aspectRatio;

                        StorageFile file = await dialog.CaptureFileAsync(CameraCaptureUIMode.Photo);
                        return file.Path;
                    }
                    catch (TaskCanceledException) {
                        return null;
                    }
                }
            }
        }

##摘要

本文介绍了如何将 Azure 移动客户端和服务器 SDK 中的新文件支持用于 Azure 存储。

- 创建一个存储帐户，并将连接字符串添加到移动应用后端。只有后端具有 Azure 存储的密钥：每当移动客户端需要访问 Azure 存储时，它都会请求 SAS 令牌（共享访问签名）。若要了解有关 Azure 存储中 SAS 令牌的详细信息，请参阅[了解共享访问签名]。

- 创建一个控制器作为 `StorageController` 的子类，以处理 SAS 令牌请求并获取与记录关联的文件。默认情况下，文件使用记录 ID 作为容器名称的一部分来与记录关联；指定 `IContainerNameResolver` 的实现即可自定义该行为。还可以自定义 SAS 令牌策略。

- Azure 移动客户端 SDK 不实际存储任何文件数据。而是由客户端 SDK 调用 `IFileSyncHandler`，然后后者决定如何（以及是否）将文件存储在本地设备上。按如下所示注册同步处理程序：

        client.InitializeFileSync(new MyFileSyncHandler(), store);

      + Azure 移动客户端 SDK 需要文件数据时（例如，在上载过程中），调用 `IFileSyncHandler.GetDataSource`。这使用户能够管理如何（以及是否）将文件存储在本地设备上，并在需要时返回该信息。

      + 在文件同步流中调用 `IFileSyncHandler.ProcessFileSynchronizationAction`。将提供文件引用和 FileSynchronizationAction 枚举值，以便用户可以确定应用程序应如何处理该事件（例如，在创建或更新文件时自动下载该文件，在服务器上删除文件时也从本地设备中删除该文件）。

- 在联机或脱机模式下均可使用 `MobileServiceFile`（分别使用 `IMobileServiceTable` 或 `IMobileServiceSyncTable`）。在脱机情况下，应用调用 `PushFileChangesAsync` 时，将进行上载。这将导致处理脱机操作队列；对于每个文件操作，Azure 移动客户端 SDK 将调用 `IFileSyncHandler` 实例的 `GetDataSource` 方法检索要上载的文件内容。

- 若要检索项目的文件，请调用 `IMobileServiceTable<T>` 或 IMobileServiceSyncTable<T> 实例的 `GetFilesAsync` 方法。此方法返回与所提供的数据项关联的文件列表。（请注意：这是 *本地* 操作，将根据上次同步时对象的状态返回文件。若要从服务器获取已更新的文件列表，应先启动同步操作。）

        IEnumerable<MobileServiceFile> files = await myTable.GetFilesAsync(myItem);

- 文件同步功能使用本地存储中的记录更改通知，检索客户端已在推送或拉取操作过程中收到的记录。使用 `StoreTrackingOptions` 参数为同步上下文打开本地和服务器通知即可实现此功能。

        this.client.SyncContext.InitializeAsync(store, StoreTrackingOptions.NotifyLocalAndServerOperations);

      + 其他存储跟踪选项也可用，如仅本地或仅服务器通知。使用 `IMobileServiceClient` 的 `EventManager` 属性可添加或拥有自定义回调：

            jobService.MobileService.EventManager.Subscribe<StoreOperationCompletedEvent>(StoreOperationEventHandler);

- 直接修改 blob 存储即可从记录添加或删除文件，因为关联是通过命名约定实现的。但是，在这种情况下，应始终**在修改关联的 blob 时更新记录时间戳**。添加或删除文件时，Azure 移动客户端 SDK 始终更新记录。

    此要求的原因是某些移动客户端将已在本地存储中有记录。这些客户端执行增量拉取时，不会返回此记录，并且客户端将不会查询新的关联文件。若要避免此问题，未使用 Azure 移动客户端 SDK 执行任何 blob 存储更改时，建议更新记录时间戳。

- 客户端项目使用 [Xamarin.Forms DependencyService] 模式，在运行时加载特定于相应平台的类。在此示例中，定义了一个接口 `IPlatform`，并在每个特定于平台的项目中对其进行了实现。

<!-- URLs. -->

[Visual Studio Community 2013]: https://go.microsoft.com/fwLink/p/?LinkID=534203
[创建 Xamarin.Forms 应用]: /documentation/articles/app-service-mobile-xamarin-forms-get-started/
[Xamarin.Forms DependencyService]: https://developer.xamarin.com/guides/xamarin-forms/dependency-service/
[Microsoft.Azure.Mobile.Client.Files]: https://www.nuget.org/packages/Microsoft.Azure.Mobile.Client.Files/
[Microsoft.Azure.Mobile.Client.SQLiteStore]: https://www.nuget.org/packages/Microsoft.Azure.Mobile.Client.SQLiteStore/
[Microsoft.Azure.Mobile.Server.Files]: https://www.nuget.org/packages/Microsoft.Azure.Mobile.Server.Files/
[了解共享访问签名]: /documentation/articles/storage-dotnet-shared-access-signature-part-1/
[创建 Azure 存储帐户]: /documentation/articles/storage-create-storage-account/#create-a-storage-account

<!---HONumber=Mooncake_0919_2016-->