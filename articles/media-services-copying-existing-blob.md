<properties 
	pageTitle="将现有 Blob 复制到媒体服务资产中" 
	description="本主题说明如何将现有 Blob 复制到媒体服务资产中。" 
	services="media-services" 
	documentationCenter="" 
	authors="Juliako" 
	manager="dwrede" 
	editor=""/>

<tags
	ms.service="media-services"
	ms.date="03/14/2016" 
	wacn.date="04/05/2016"/>

#将现有 Blob 复制到媒体服务资产中

本主题介绍如何将存储帐户中的 Blob 复制到新的 Azure 媒体服务资产中。

Blob 可以存在于与媒体服务帐户关联的存储帐户中，也可以存在于不与媒体服务帐户关联的存储帐户中。本主题演示如何将 Blob 从存储帐户复制到媒体服务资产中。请注意，你还可以跨数据中心复制。但是，这样做可能会产生费用。有关定价的详细信息，请参阅[数据传输](/pricing/#header-11)。

>[AZURE.NOTE] 在不使用媒体服务 API 的情况下，你不应该尝试更改媒体服务生成的 BLOB 容器内容。

##先决条件

- 在新的或现有的 Azure 订阅中拥有两个媒体服务帐户。请参阅主题[如何创建媒体服务帐户](/documentation/articles/media-services-create-account/)。
- 操作系统：Windows 10、Windows 7、Windows 2008 R2 或 Windows 8。
- .NET Framework 4.5。
- Visual Studio 2010 SP1（专业版、高级专业版、旗舰版或学习版）或更高版本。

##设置项目

在本部分中，你将创建和设置一个 C# 控制台应用程序项目。

1. 使用 Visual Studio 创建包含 C# 控制台应用程序项目的新解决方案。 
2. 为“名称”输入 CopyExistingBlobsIntoAsset，然后单击“确定”。
1. 使用 Nuget 添加对媒体服务相关 DLL 的引用。在 Visual Studio 主菜单中，选择“工具”->“库程序包管理器”->“程序包管理器控制台”。在控制台窗口中，键入 Install-Package windowsazure.mediaservices，然后按 Enter。
1. 添加此项目所需的其他引用：System.Configuration。
1. 将默认添加到 Programs.cs 文件中的 using 语句替换为以下语句：
		
		using System;
		using System.Linq;
		using System.Configuration;
		using System.IO;
		using System.Text;
		using System.Threading;
		using System.Threading.Tasks;
		using System.Collections.Generic;
		using Microsoft.WindowsAzure.Storage;
		using Microsoft.WindowsAzure.MediaServices.Client;
		using Microsoft.WindowsAzure;
		using System.Web;
		using Microsoft.WindowsAzure.Storage.Blob;
		using Microsoft.WindowsAzure.Storage.Auth;

1. 将 appSettings 节添加到 .config 文件，并根据媒体服务和 Storage 密钥与名称值更新值。

		<appSettings>
		  <add key="MediaServicesAccountName" value="Media-Services-Account-Name"/>
		  <add key="MediaServicesAccountKey" value="Media-Services-Account-Key"/>
		  <add key="MediaServicesStorageAccountName" value="Media-Services-Storage-Account-Name"/>
		  <add key="MediaServicesStorageAccountKey" value="Media-Services-Storage-Account-Key"/>
		  <add key="ExternalStorageAccountName" value="External-Storage-Account-Name"/>
		  <add key="ExternalStorageAccountKey" value="External-Storage-Account-Key"/>
		</appSettings>


##将 Blob 从存储帐户复制到媒体服务资产中

下面的代码示例执行以下任务：

1. 创建 CloudMediaContext 实例。 
1. 创建 CloudStorageAccount 实例：\_sourceStorageAccount 和\_destinationStorageAccount。
1. 将本地目录中的平滑流文件上载到位于 \_sourceStorageAccount 中的 Blob 容器内。
1. 创建一个新资产。为此资产创建的 Blob 容器位于 \_destinationStorageAccount 中。
1. 使用 Azure 存储 SDK 将指定的 Blob 复制到与此资产关联的容器中。

	>[AZURE.NOTE]如果定位符已过期，复制操作不会引发异常。

1. 由于在本示例中我们要复制平滑流文件，因此示例演示了如何将 .ism 文件设置为主文件。例如，如果我们复制了一个 .mp4 文件，则该 mp4 文件将设置为主文件。
1. 为与此资产关联的 OnDemandOrigin 定位符创建平滑流 URL。 
			
		class Program
		{
		    // Read values from the App.config file. 
		    static string _accountName = ConfigurationManager.AppSettings["MediaServicesAccountName"];
		    static string _accountKey = ConfigurationManager.AppSettings["MediaServicesAccountKey"];
		    static string _storageAccountName = ConfigurationManager.AppSettings["MediaServicesStorageAccountName"];
		    static string _storageAccountKey = ConfigurationManager.AppSettings["MediaServicesStorageAccountKey"];
		    static string _externalStorageAccountName = ConfigurationManager.AppSettings["ExternalStorageAccountName"];
		    static string _externalStorageAccountKey = ConfigurationManager.AppSettings["ExternalStorageAccountKey"];
		
			private static readonly String _defaultScope = "urn:WindowsAzureMediaServices";

			// Azure China uses a different API server and a different ACS Base Address from the Global.
			private static readonly String _chinaApiServerUrl = "https://wamsshaclus001rest-hs.chinacloudapp.cn/API/";
			private static readonly String _chinaAcsBaseAddressUrl = "https://wamsprodglobal001acs.accesscontrol.chinacloudapi.cn";

			// Azure China Storage Endpoint Suffix
			private static readonly String _chinaEndpointSuffix = "core.chinacloudapi.cn";

		    private static MediaServicesCredentials _cachedCredentials = null;
		    private static CloudMediaContext _context = null;
		
		    private static CloudStorageAccount _sourceStorageAccount = null;
		    private static CloudStorageAccount _destinationStorageAccount = null;
		
		    static void Main(string[] args)
		    {
		        _cachedCredentials = new MediaServicesCredentials(
		                        _accountName,
		                        _accountKey,
								_defaultScope,
								_chinaAcsBaseAddressUrl);

				// Create the API server Uri
				_apiServer = new Uri("https://wamsshaclus001rest-hs.chinacloudapp.cn/API/");

		        // Use the cached credentials to create CloudMediaContext.
		        _context = new CloudMediaContext(_apiServer, _cachedCredentials);
		
		        // In this example the storage account from which we copy blobs is not 
		        // associated with the Media Services account into which we copy blobs.
		        // But the same code will work for coping blobs from a storage account that is 
		        // associated with the Media Services account.
		        //
		        // Get a reference to a storage account that is not associated with a Media Services account
		        // (an external account).  
		        StorageCredentials externalStorageCredentials =
		            new StorageCredentials(_externalStorageAccountName, _externalStorageAccountKey);
		        _sourceStorageAccount = new CloudStorageAccount(externalStorageCredentials, _chinaEndpointSuffix, true);
		
		        //Get a reference to the storage account that is associated with a Media Services account. 
		        StorageCredentials mediaServicesStorageCredentials =
		            new StorageCredentials(_storageAccountName, _storageAccountKey);
		        _destinationStorageAccount = new CloudStorageAccount(mediaServicesStorageCredentials, _chinaEndpointSuffix, false);
		
		        // Upload Smooth Streaming files into a storage account.
		        string localMediaDir = @"C:\supportFiles\streamingfiles";
		        CloudBlobContainer blobContainer =
		            UploadContentToStorageAccount(localMediaDir);
		
		        // Create a new asset and copy the smooth streaming files into 
		        // the container that is associated with the asset.
		        IAsset asset = CreateAssetFromExistingBlobs(blobContainer);
		
		        // Get the streaming URL for the smooth streaming files 
		        // that were copied into the asset.   
		        string urlForClientStreaming = CreateStreamingLocator(asset);
		        Console.WriteLine("Smooth Streaming URL: " + urlForClientStreaming);
		
		        Console.ReadLine();
		    }
		
		    /// <summary>
		    /// Uploads content from a local directory into the specified storage account.
		    /// In this example the storage account is not associated with the Media Services account.
		    /// </summary>
		    /// <param name="localPath">The path from which to upload the files.</param>
		    /// <returns>The container that contains the uploaded files.</returns>
		    static public CloudBlobContainer UploadContentToStorageAccount(string localPath)
		    {
		        CloudBlobClient externalCloudBlobClient = _sourceStorageAccount.CreateCloudBlobClient();
		        CloudBlobContainer externalMediaBlobContainer = externalCloudBlobClient.GetContainerReference("streamingfiles");
		
		        externalMediaBlobContainer.CreateIfNotExists();
		
		        // Upload files to the blob container.  
		        DirectoryInfo uploadDirectory = new DirectoryInfo(localPath);
		        foreach (var file in uploadDirectory.EnumerateFiles())
		        {
		            CloudBlockBlob blob = externalMediaBlobContainer.GetBlockBlobReference(file.Name);
		
		            blob.UploadFromFile(file.FullName, FileMode.Open);
		        }
		
		        return externalMediaBlobContainer;
		    }
		
		    /// <summary>
		    /// Creates a new asset and copies blobs from the specifed storage account.
		    /// </summary>
		    /// <param name="mediaBlobContainer">The specified blob container.</param>
		    /// <returns>The new asset.</returns>
		    static public IAsset CreateAssetFromExistingBlobs(CloudBlobContainer mediaBlobContainer)
		    {
		        // Create a new asset. 
	                IAsset asset = _context.Assets.Create("NewAsset_" + Guid.NewGuid(), AssetCreationOptions.None);
		
		        IAccessPolicy writePolicy = _context.AccessPolicies.Create("writePolicy", TimeSpan.FromHours(24), AccessPermissions.Write);
		        ILocator destinationLocator = _context.Locators.CreateLocator(LocatorType.Sas, asset, writePolicy);
		
		        CloudBlobClient destBlobStorage = _destinationStorageAccount.CreateCloudBlobClient();
		
		        // Get the asset container URI and Blob copy from mediaContainer to assetContainer. 
		        string destinationContainerName = (new Uri(destinationLocator.Path)).Segments[1];
		
		        CloudBlobContainer assetContainer = destBlobStorage.GetContainerReference(destinationContainerName);
		
		        if (assetContainer.CreateIfNotExists())
		        {
		            assetContainer.SetPermissions(new BlobContainerPermissions
		            {
		                PublicAccess = BlobContainerPublicAccessType.Blob
		            });
		        }
		
		        var blobList = mediaBlobContainer.ListBlobs();
		        foreach (var sourceBlob in blobList)
		        {
		            var assetFile = asset.AssetFiles.Create((sourceBlob as ICloudBlob).Name);
		            CopyBlob(sourceBlob as ICloudBlob, assetContainer);
		            assetFile.ContentFileSize = (sourceBlob as ICloudBlob).Properties.Length;
		            assetFile.Update();
		        }
		
	                asset.Update();
	
	                destinationLocator.Delete();
	                writePolicy.Delete();
	
	                // Since we copied a set of Smooth Streaming files, 
	                // set the .ism file to be the primary file. 
	                // If we, for example, copied an .mp4, then the mp4 would be the primary file. 
	                SetISMFileAsPrimary(asset);
	
	                return asset;
	            }
	
	            /// <summary>
	            /// Creates the OnDemandOrigin locator in order to get the streaming URL.
	            /// </summary>
	            /// <param name="asset">The asset that contains the smooth streaming files.</param>
	            /// <returns>The streaming URL.</returns>
	            static public string CreateStreamingLocator(IAsset asset)
	            {
	                var ismAssetFile = asset.AssetFiles.ToList().
	                    Where(f => f.Name.EndsWith(".ism", StringComparison.OrdinalIgnoreCase)).First();
	
	                // Create a 30-day readonly access policy. 
	                IAccessPolicy policy = _context.AccessPolicies.Create("Streaming policy",
	                    TimeSpan.FromDays(30),
	                    AccessPermissions.Read);
	
	                // Create a locator to the streaming content on an origin. 
	                ILocator originLocator = _context.Locators.CreateLocator(LocatorType.OnDemandOrigin, asset,
	                    policy,
	                    DateTime.UtcNow.AddMinutes(-5));
	
	                return originLocator.Path + ismAssetFile.Name + "/manifest";
	            }
	
	            /// <summary>
	            /// Copies the specified blob into the specified container.
	            /// </summary>
	            /// <param name="sourceBlob">The source container.</param>
	            /// <param name="destinationContainer">The destination container.</param>
	            static private void CopyBlob(ICloudBlob sourceBlob, CloudBlobContainer destinationContainer)
	            {
	                var signature = sourceBlob.GetSharedAccessSignature(new SharedAccessBlobPolicy
	                {
	                    Permissions = SharedAccessBlobPermissions.Read,
	                    SharedAccessExpiryTime = DateTime.UtcNow.AddHours(24)
	                });
	
	                ICloudBlob destinationBlob = destinationContainer.GetBlockBlobReference(sourceBlob.Name);
	
	                if (destinationBlob.Exists())
	                {
	                    Console.WriteLine(string.Format("Destination blob '{0}' already exists. Skipping.", destinationBlob.Uri));
	                }
	                else
	                {
	
	                    // Display the size of the source blob.
	                    Console.WriteLine(sourceBlob.Properties.Length);
	
	                    Console.WriteLine(string.Format("Copy blob '{0}' to '{1}'", sourceBlob.Uri, destinationBlob.Uri));
	                    destinationBlob.StartCopyFromBlob(new Uri(sourceBlob.Uri.AbsoluteUri + signature));
	
	                    while (true)
	                    {
	                        // The StartCopyFromBlob is an async operation, 
	                        // so we want to check if the copy operation is completed before proceeding. 
	                        // To do that, we call FetchAttributes on the blob and check the CopyStatus. 
	                        destinationBlob.FetchAttributes();
	                        if (destinationBlob.CopyState.Status != CopyStatus.Pending)
	                        {
	                            break;
	                        }
	                        //It's still not completed. So wait for some time.
	                        System.Threading.Thread.Sleep(1000);
	                    }
	
	
	                    // Display the size of the destination blob.
	                    Console.WriteLine(destinationBlob.Properties.Length);
	
	                }
	            }
	
	            /// <summary>
	            /// Sets a file with the .ism extension as a primary file.
	            /// </summary>
	            /// <param name="asset">The asset that contains the smooth streaming files.</param>
	            static private void SetISMFileAsPrimary(IAsset asset)
	            {
	
	                //If you expect the asset to contain the .ism asset file, set the .ism file as the primary file.
	                var ismAssetFiles = asset.AssetFiles.ToList().
	                    Where(f => f.Name.EndsWith(".ism", StringComparison.OrdinalIgnoreCase)).ToArray();
	
	                // The following code assigns the first .ism file as the primary file in the asset.
	                // An asset should have one .ism file.  
	                ismAssetFiles.First().IsPrimary = true;
	                ismAssetFiles.First().Update();
	            }
	        }

 


<!---HONumber=Mooncake_0328_2016-->