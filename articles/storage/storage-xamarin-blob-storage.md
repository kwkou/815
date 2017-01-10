<properties
    pageTitle="如何通过 Xamarin 使用 Blob 存储 | Azure"
    description="通过用于 Xamarin 的 Azure 存储客户端库，开发人员可以使用其本机用户界面创建 iOS、Android 和 Windows 应用商店应用。本教程演示了如何通过 Xamarin 来创建使用 Azure Blob 存储的应用程序。"
    services="storage"
    documentationcenter="xamarin"
    author="micurd"
    manager="jahogg"
    editor="tysonn" />
<tags
    ms.assetid="44cb845d-cf78-4942-95b8-952da4f9a2c2"
    ms.service="storage"
    ms.workload="storage"
    ms.tgt_pltfrm="na"
    ms.devlang="na"
    ms.topic="article"
    ms.date="11/28/2016"
    wacn.date="01/06/2017"
    ms.author="micurd" />

# 如何通过 Xamarin 使用 Blob 存储

[AZURE.INCLUDE [storage-selector-blob-include](../../includes/storage-selector-blob-include.md)]

## 概述

Xamarin 使开发人员能够通过共享的 C# 代码库来使用其本机用户界面创建 iOS、Android 和 Windows 应用商店应用程序。本教程演示了如何将 Azure Blob 存储用于 Xamarin 应用程序。如果要先详细了解 Azure 存储再深入分析代码，请参阅 [Azure 存储简介](/documentation/articles/storage-introduction/)。

[AZURE.INCLUDE [storage-create-account-include](../../includes/storage-create-account-include.md)]

[AZURE.INCLUDE [存储移动身份验证指南](../../includes/storage-mobile-authentication-guidance.md)]

## 创建新的 Xamarin 应用程序

对于本入门教程，我们将创建面向 Android、iOS 和 Windows 的应用。此应用将仅创建一个容器，并将 Blob 上传到此容器中。我们将通过 Windows 上 Visual Studio 开始入门，这些知识同样适用于通过 Mac OS 上的 Xamarin Studio 创建应用。

请按以下步骤创建应用程序：

1. 下载并安装 [Xamarin for Visual Studio](https://www.xamarin.com/download)（如果尚未这样做）。
2. 打开 Visual Studio ，创建空白应用（本机共享）：“文件”>“新建”>“项目”>“跨平台”>“空白应用（本机共享）”。
3. 右键单击“解决方案资源管理器”窗格中的解决方案，然后选择“为解决方案管理 NuGet 包”。搜索 **WindowsAzure.Storage**，并将最新稳定版本安装到解决方案中的所有项目。
4. 生成并运行项目。

现在，应该有了这样一个应用程序，单击其中某个按钮将使计数器递增。

> [AZURE.NOTE] 用于 Xamarin 的 Azure 存储客户端库当前支持以下项目类型：本机共享、Xamarin.Forms 共享、Xamarin.Android 和 Xamarin.iOS。

## 创建容器并上传 Blob

接下来，将一些代码添加到共享类 `MyClass.cs` 以创建容器，然后将 Blob 上传到此容器。`MyClass.cs` 应如下所示：

	using Microsoft.WindowsAzure.Storage;
	using Microsoft.WindowsAzure.Storage.Blob;
	using System.Threading.Tasks;

	namespace XamarinApp
	{
		public class MyClass
		{
			public MyClass ()
			{
			}

		    public static async Task createContainerAndUpload()
		    {
		        // Retrieve storage account from connection string.
		        CloudStorageAccount storageAccount = CloudStorageAccount.Parse("DefaultEndpointsProtocol=https;AccountName=your_account_name_here;AccountKey=your_account_key_here;EndpointSuffix=core.chinacloudapi.cn");

		        // Create the blob client.
		        CloudBlobClient blobClient = storageAccount.CreateCloudBlobClient();

		        // Retrieve reference to a previously created container.
		        CloudBlobContainer container = blobClient.GetContainerReference("mycontainer");

				// Create the container if it doesn't already exist.
            	await container.CreateIfNotExistsAsync();

		        // Retrieve reference to a blob named "myblob".
		        CloudBlockBlob blockBlob = container.GetBlockBlobReference("myblob");

		        // Create the "myblob" blob with the text "Hello, world!"
		        await blockBlob.UploadTextAsync("Hello, world!");
		    }
		}
	}

确保将“your\_account\_name\_here”和“your\_account\_key\_here”替换为实际帐户名和帐户密钥。然后就可以在 iOS、Android 和 Windows Phone 应用程序中使用此共享类。可将 `MyClass.createContainerAndUpload()` 添加到每个项目。例如：

### XamarinApp.Droid > MainActivity.cs

	using Android.App;
	using Android.Widget;
	using Android.OS;

	namespace XamarinApp.Droid
	{
		[Activity (Label = "XamarinApp.Droid", MainLauncher = true, Icon = "@drawable/icon")]
		public class MainActivity : Activity
		{
			int count = 1;

			protected override async void OnCreate (Bundle bundle)
			{
				base.OnCreate (bundle);

				// Set our view from the "main" layout resource
				SetContentView (Resource.Layout.Main);

				// Get our button from the layout resource,
				// and attach an event to it
				Button button = FindViewById<Button> (Resource.Id.myButton);

				button.Click += delegate {
					button.Text = string.Format ("{0} clicks!", count++);
				};

	      	  await MyClass.createContainerAndUpload();
			}
		}
	}

### XamarinApp.iOS > ViewController.cs

	using System;
	using UIKit;

	namespace XamarinApp.iOS
	{
		public partial class ViewController : UIViewController
		{
			int count = 1;

			public ViewController (IntPtr handle) : base (handle)
			{
			}

			public override async void ViewDidLoad ()
			{
				base.ViewDidLoad ();
				// Perform any additional setup after loading the view, typically from a nib.
				Button.AccessibilityIdentifier = "myButton";
				Button.TouchUpInside += delegate {
					var title = string.Format ("{0} clicks!", count++);
					Button.SetTitle (title, UIControlState.Normal);
				};

	            await MyClass.createContainerAndUpload();
	    	}

			public override void DidReceiveMemoryWarning ()
			{
				base.DidReceiveMemoryWarning ();
				// Release any cached data, images, etc that aren't in use.
			}
		}
	}

### XamarinApp.WinPhone > MainPage.xaml > MainPage.xaml.cs

	using Windows.UI.Xaml.Controls;
	using Windows.UI.Xaml.Navigation;

	// The Blank Page item template is documented at http://go.microsoft.com/fwlink/?LinkId=391641

	namespace XamarinApp.WinPhone
	{
	    /// <summary>
	    /// An empty page that can be used on its own or navigated to within a Frame.
	    /// </summary>
	    public sealed partial class MainPage : Page
	    {
	        int count = 1;

	        public MainPage()
	        {
	            this.InitializeComponent();

	            this.NavigationCacheMode = NavigationCacheMode.Required;
	        }

	        /// <summary>
	        /// Invoked when this page is about to be displayed in a Frame.
	        /// </summary>
	        /// <param name="e">Event data that describes how this page was reached.
	        /// This parameter is typically used to configure the page.</param>
	        protected override async void OnNavigatedTo(NavigationEventArgs e)
	        {
	            // TODO: Prepare page for display here.

	            // TODO: If your application contains multiple pages, ensure that you are
	            // handling the hardware Back button by registering for the
	            // Windows.Phone.UI.Input.HardwareButtons.BackPressed event.
	            // If you are using the NavigationHelper provided by some templates,
	            // this event is handled for you.
	            Button.Click += delegate {
	                var title = string.Format("{0} clicks!", count++);
	                Button.Content = title;
	            };

	            await MyClass.createContainerAndUpload();
	        }
	    }
	}


## 运行应用程序

现在可以在 Android 或 Windows Phone 仿真程序中运行此应用程序。也可在 iOS 仿真程序中运行此应用程序，但需要使用 Mac。有关如何执行此操作的具体说明，请阅读 [connecting Visual Studio to a Mac](https://developer.xamarin.com/guides/ios/getting_started/installation/windows/connecting-to-mac/)（将 Visual Studio 连接到 Mac）文档

运行应用后，会在存储帐户中创建容器 `mycontainer`。它应该包含 Blob `myblob`，其中包含文本 `Hello, world!`。可以使用 [Azure 存储资源管理器](http://storageexplorer.com/)对此进行验证。

## 后续步骤

在本入门指南中，你学习了如何使用 Azure 存储在 Xamarin 中创建跨平台应用程序。本入门指南着重介绍 Blob 存储的情况。但是，还对 Blob 存储、表存储、文件存储和队列存储进行更多操作。请参阅以下文章以了解更多信息：
- [通过 .NET 开始使用 Azure Blob 存储](/documentation/articles/storage-dotnet-how-to-use-blobs/)
- [通过 .NET 开始使用 Azure 表存储](/documentation/articles/storage-dotnet-how-to-use-tables/)
- [通过 .NET 开始使用 Azure 队列存储](/documentation/articles/storage-dotnet-how-to-use-queues/)
- [在 Windows 上开始使用 Azure 文件存储](/documentation/articles/storage-dotnet-how-to-use-files/)

<!---HONumber=Mooncake_0103_2017-->