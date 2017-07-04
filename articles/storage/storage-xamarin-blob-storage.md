<properties
    pageTitle="如何通过 Xamarin 使用 Blob 存储 | Azure"
    description="通过用于 Xamarin 的 Azure 存储客户端库，开发人员可以使用其本机用户界面创建 iOS、Android 和 Windows 应用商店应用。本教程演示了如何通过 Xamarin 来创建使用 Azure Blob 存储的应用程序。"
    services="storage"
    documentationcenter="xamarin"
    author="michaelhauss"
    manager="vamshik"
    editor="tysonn" />
<tags
    ms.assetid="44cb845d-cf78-4942-95b8-952da4f9a2c2"
    ms.service="storage"
    ms.workload="storage"
    ms.tgt_pltfrm="na"
    ms.devlang="dotnet"
    ms.topic="article"
    ms.date="05/11/2017"
    wacn.date="05/31/2017"
    ms.author="michaelhauss"
    ms.translationtype="Human Translation"
    ms.sourcegitcommit="4a18b6116e37e365e2d4c4e2d144d7588310292e"
    ms.openlocfilehash="0712690e2e96a074cdca5703a4e035a0519871da"
    ms.contentlocale="zh-cn"
    ms.lasthandoff="05/19/2017" />

# <a name="how-to-use-blob-storage-from-xamarin"></a>如何通过 Xamarin 使用 Blob 存储
[AZURE.INCLUDE [storage-selector-blob-include](../../includes/storage-selector-blob-include.md)]

## <a name="overview"></a>概述
Xamarin 使开发人员能够通过共享的 C# 代码库来使用其本机用户界面创建 iOS、Android 和 Windows 应用商店应用。 本教程演示了如何将 Azure Blob 存储用于 Xamarin 应用程序。 如果要先详细了解 Azure 存储再深入分析代码，请参阅 [Azure 存储简介](/documentation/articles/storage-introduction/)。

[AZURE.INCLUDE [storage-create-account-include](../../includes/storage-create-account-include.md)]

[AZURE.INCLUDE [storage-mobile-authentication-guidance](../../includes/storage-mobile-authentication-guidance.md)]

## <a name="create-a-new-xamarin-application"></a>创建新的 Xamarin 应用程序
本教程将创建面向 Android、iOS 和 Windows 的应用。 此应用将仅创建一个容器，并将 Blob 上传到此容器中。 我们将使用 Windows 上的 Visual Studio，这些知识同样适用于通过 macOS 上的 Xamarin Studio 创建应用的情况。

请按以下步骤创建应用程序：

1. 下载并安装 [Xamarin for Visual Studio](https://www.xamarin.com/download)（如果尚未这样做）。
2. 打开 Visual Studio，创建空白应用（本机可移植）：“文件”>“新建”>“项目”>“跨平台”>“空白应用(本机可移植)”。
3. 右键单击“解决方案资源管理器”窗格中的解决方案，然后选择“为解决方案管理 NuGet 包”。 搜索 **WindowsAzure.Storage**，并将最新稳定版本安装到解决方案中的所有项目。
4. 生成并运行项目。

现在，应该有了这样一个应用程序，单击其中某个按钮将使计数器递增。

## <a name="create-container-and-upload-blob"></a>创建容器并上传 Blob
接下来，需要在 `(Portable)` 项目中，向 `MyClass.cs` 添加一些代码。 此代码将创建一个容器并在其中上传 Blob。 `MyClass.cs` 应如下所示：

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

	            public static async Task performBlobOperation()
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

请确保将“your_account_name_here”和“your_account_key_here”替换为实际帐户名和帐户密钥。 

用户的 iOS、Android 和 Windows Phone 项目全都引用可移植项目 - 这意味着，用户在一个位置编写所有共享代码即可跨所有项目使用。现在可以向每个项目添加以下代码行，进行充分利用：`MyClass.performBlobOperation()`

### <a name="xamarinappdroid--mainactivitycs"></a>XamarinApp.Droid > MainActivity.cs

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

            			await MyClass.performBlobOperation();
			}
		}
	}
    }

### <a name="xamarinappios--viewcontrollercs"></a>XamarinApp.iOS > ViewController.cs

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

		                await MyClass.performBlobOperation();
		            }

			public override void DidReceiveMemoryWarning ()
			{
				base.DidReceiveMemoryWarning ();
				// Release any cached data, images, etc that aren't in use.
			}
		}
	}
    }

### <a name="xamarinappwinphone--mainpagexaml--mainpagexamlcs"></a>XamarinApp.WinPhone > MainPage.xaml > MainPage.xaml.cs

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

                	await MyClass.performBlobOperation();
	        }
	    }
	}
	}

## <a name="run-the-application"></a>运行应用程序
现在可以在 Android 或 Windows Phone 仿真程序中运行此应用程序。 也可在 iOS 仿真程序中运行此应用程序，但需要使用 Mac。 有关如何执行此操作的具体说明，请阅读 [connecting Visual Studio to a Mac](https://developer.xamarin.com/guides/ios/getting_started/installation/windows/connecting-to-mac/)

运行应用后，会在存储帐户中创建容器 `mycontainer`。它应该包含 Blob `myblob`，Blob 中包含文本 `Hello, world!`。可以使用 [Azure 存储资源管理器](http://storageexplorer.com/)对此进行验证。

## <a name="next-steps"></a>后续步骤
本教程介绍如何在 Xamarin 中创建使用 Azure 存储的跨平台应用程序，特别介绍了 Blob 存储中的一个方案。 但是，还可以进行更多操作，不光是使用 Blob 存储，还可以使用表、文件和队列存储。 请参阅以下文章以了解更多信息：

* [通过 .NET 开始使用 Azure Blob 存储](/documentation/articles/storage-dotnet-how-to-use-blobs/)
* [通过 .NET 开始使用 Azure 表存储](/documentation/articles/storage-dotnet-how-to-use-tables/)
* [通过 .NET 开始使用 Azure 队列存储](/documentation/articles/storage-dotnet-how-to-use-queues/)
* [在 Windows 上开始使用 Azure 文件存储](/documentation/articles/storage-dotnet-how-to-use-files/)

[AZURE.INCLUDE [storage-try-azure-tools-blobs](../../includes/storage-try-azure-tools-blobs.md)]
<!--Update_Description: wording update;add anchors to subtitles-->