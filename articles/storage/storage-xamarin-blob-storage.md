<properties 
	pageTitle="如何通过 Xamarin（预览版）使用 Blob 存储 | Azure" 
	description="用于 Xamarin 的 Azure 存储客户端库预览版使开发人员能够使用其本机用户界面创建 iOS、Android 和 Windows 应用商店应用程序。本教程演示了如何通过 Xamarin 来创建使用 Azure Blob 存储的 Android 应用程序。" 
	services="storage" 
	documentationCenter="xamarin" 
	authors="micurd" 
	manager="" 
	editor=""/>

<tags 
	ms.service="storage" 
	ms.date="02/21/2016"
	wacn.date="04/18/2016"/>

# 如何通过 Xamarin（预览版）使用 Blob 存储

[AZURE.INCLUDE [storage-selector-blob-include](../includes/storage-selector-blob-include.md)]

## 概述

Xamarin 使开发人员能够通过共享的 C# 代码库来使用其本机用户界面创建 iOS、Android 和 Windows 应用商店应用程序。用于 Xamarin 的 Azure 存储客户端库是一个预览库；请注意，该库在将来可能会更改。

本教程演示了如何将 Azure Blob 存储用于 Xamarin Android 应用程序。若要在深入分析代码之前了解有关 Azure 存储空间的更多内容，请参阅本文档末尾的[后续步骤](#next-steps)。

[AZURE.INCLUDE [storage-create-account-include](../includes/storage-create-account-include.md)]

## 生成共享访问签名

使用用于 Xamarin 的 Azure 存储客户端库进行开发时，你不能使用帐户访问密钥对 Azure 存储空间帐户的访问权限进行身份验证。这是为了防止将你的帐户凭据分发给可以下载你的应用程序的用户。与之相反，我们鼓励使用共享访问签名 (SAS)，这样就不会暴露你的帐户凭据。

在本指南中，我们将使用 Azure PowerShell 来生成 SAS 令牌。然后，我们将创建一个使用生成的 SAS 的 Xamarin 应用程序。

首先，你需要安装 Azure PowerShell请查看[如何安装和配置 Azure PowerShell](/documentation/articles/powershell-install-configure/#Install) 以获取相关说明。

接下来，请打开 Azure PowerShell 并运行以下命令：请记住将 `ACCOUNT_NAME` 和 `ACCOUNT_KEY== ` 替换为你的存储帐户凭据。将 `CONTAINER_NAME` 替换为你选择的名称。

    PS C:\> $context = New-AzureStorageContext -Environment AzureChinaCloud -StorageAccountName "ACCOUNT_NAME" -StorageAccountKey "ACCOUNT_KEY=="
	PS C:\> New-AzureStorageContainer CONTAINER_NAME -Permission Off -Context $context
	PS C:\> $now = Get-Date
	PS C:\> New-AzureStorageContainerSASToken -Name CONTAINER_NAME -Permission rwdl -ExpiryTime $now.AddDays(1.0) -Context $context -FullUri

新容器的共享访问签名 URI 应如下所示：

	https://storageaccount.blob.core.chinacloudapi.cn/sascontainer?sv=2012-02-12&se=2013-04-13T00%3A12%3A08Z&sr=c&sp=wl&sig=t%2BbzU9%2B7ry4okULN9S0wst%2F8MCUhTjrHyV9rDNLSe8g%3Dsss

你在容器上创建的共享访问签名将在接下来的一天内有效。该签名将授予对容器内 blob 的完整权限（例如，读取、写入、删除和列表）。

有关共享访问签名的详细信息，请参阅[共享访问签名：创建 SAS 并将 SAS 用于 Blob 存储](/documentation/articles/storage-dotnet-shared-access-signature-part-2/)。

## 创建新的 Xamarin 应用程序

就本教程来说，我们将在 Visual Studio 中创建 Xamarin 应用程序。请按以下步骤创建该应用程序：

1. 下载并安装 [Visual Studio](https://www.visualstudio.com/zh-cn)。
2. 下载并安装 [Xamarin](http://xamarin.com/platform)。
3. 打开 Visual Studio，然后选择“文件”>“新建”>“项目”>“Android”>“空应用程序(Android)”。
4. 右键单击“解决方案资源管理器”窗格中的项目，然后选择“管理 NuGet 包”。然后搜索**“Azure 存储空间”**并安装 **Azure 存储空间 4.4.0-预览版**。

现在，你应该有了这样一个应用程序：单击某个按钮即可让计数器递增。

## 使用共享访问签名执行容器操作

接下来，请使用生成的 SAS URI，通过添加代码来执行一系列容器操作。

首先，添加以下 **using** 语句：

	using System.IO;
	using System.Text;
	using System.Threading.Tasks;
	using Microsoft.WindowsAzure.Storage.Blob;


接下来，添加 SAS 令牌的行。将 `"SAS_URI"` 字符串替换为你在 Azure PowerShell 中生成的 SAS URI。然后再添加一行，以便调用我们将在下面创建的 `UseContainerSAS` 方法。请注意，**async** 关键字已添加到了 delegate 前面。


	public class MainActivity : Activity
	{
    	int count = 1;
    	string sas = "SAS_URI";
    	protected override void OnCreate(Bundle bundle)
    	{
        	base.OnCreate(bundle);

        	// Set our view from the "main" layout resource
        	SetContentView(Resource.Layout.Main);

        	// Get our button from the layout resource, and attach an event to it
        	Button button = FindViewById<Button>(Resource.Id.MyButton);

        	button.Click += async delegate	{
             	button.Text = string.Format("{0} clicks!", count++);
             	await UseContainerSAS(sas);
         	};
     }

在 `OnCreate` 方法下添加新方法 `UseContainerSAS`。

	static async Task UseContainerSAS(string sas)
	{
    	//Try performing container operations with the SAS provided.

    	//Return a reference to the container using the SAS URI.
    	CloudBlobContainer container = new CloudBlobContainer(new Uri(sas));
    	string date = DateTime.Now.ToString();
    	try
    	{
        	//Write operation: write a new blob to the container.
        	CloudBlockBlob blob = container.GetBlockBlobReference("sasblob_" + date + ".txt");

        	string blobContent = "This blob was created with a shared access signature granting write permissions to the container. ";
        	MemoryStream msWrite = new
        	MemoryStream(Encoding.UTF8.GetBytes(blobContent));
        	msWrite.Position = 0;
        	using (msWrite)
         	{
             	await blob.UploadFromStreamAsync(msWrite);
         	}
         	Console.WriteLine("Write operation succeeded for SAS " + sas);
         	Console.WriteLine();
     	}
     	catch (Exception e)
     	{
        	Console.WriteLine("Write operation failed for SAS " + sas);
        	Console.WriteLine("Additional error information: " + e.Message);
        	Console.WriteLine();
     	}
     	try
     	{
        	//Read operation: Get a reference to one of the blobs in the container and read it.
        	CloudBlockBlob blob = container.GetBlockBlobReference("sasblob_” + date + “.txt");
        	string data = await blob.DownloadTextAsync();

        	Console.WriteLine("Read operation succeeded for SAS " + sas);
        	Console.WriteLine("Blob contents: " + data);
     	}
     	catch (Exception e)
     	{
        	Console.WriteLine("Additional error information: " + e.Message);
       		Console.WriteLine("Read operation failed for SAS " + sas);
        	Console.WriteLine();
     	}
     	Console.WriteLine();
     	try
     	{
        	//Delete operation: Delete a blob in the container.
         	CloudBlockBlob blob = container.GetBlockBlobReference("sasblob_” + date + “.txt");
         	await blob.DeleteAsync();

         	Console.WriteLine("Delete operation succeeded for SAS " + sas);
         	Console.WriteLine();
     	}
     	catch (Exception e)
     	{
        	Console.WriteLine("Delete operation failed for SAS " + sas);
        	Console.WriteLine("Additional error information: " + e.Message);
        	Console.WriteLine();
     	}
	}

## 运行应用程序

你现在可以在仿真程序或 Android 设备中运行此应用程序。

虽然此入门教程侧重于 Android，但你也可以将 `UseContainerSAS` 方法用于 iOS 或 Windows 应用商店应用程序。Xamarin 还允许开发人员创建 Windows Phone 应用程序，不过，我们的库目前尚不支持此功能。

## 后续步骤

在本教程中，你学习了如何将 Azure Blob 存储和 SAS 用于 Xamarin 应用程序。在进一步的练习中，可以应用相似的模式来生成用于 Azure 表或队列的 SAS 令牌。

查看以下链接，了解有关 blob、表和队列的详细信息：

[Azure 存储空间简介](/documentation/articles/storage-introduction/)  

- [通过 .NET 开始使用 Azure Blob 存储](/documentation/articles/storage-dotnet-how-to-use-blobs/)
- [通过 .NET 开始使用 Azure 表存储](/documentation/articles/storage-dotnet-how-to-use-tables/)
- [通过 .NET 开始使用 Azure 队列存储](/documentation/articles/storage-dotnet-how-to-use-queues/)
- [在 Windows 上开始使用 Azure 文件存储](/documentation/articles/storage-dotnet-how-to-use-files/)
- [使用 AzCopy 命令行实用程序传输数据](/documentation/articles/storage-use-azcopy/)

<!---HONumber=Mooncake_0411_2016-->