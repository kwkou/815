<properties
    pageTitle="将 Blob 从存储帐户复制到 Azure 媒体服务资产 | Azure"
    description="本主题说明如何将现有 blob 复制到媒体服务资产。示例使用 Azure 媒体服务 .NET SDK 扩展。"
    services="media-services"
    documentationcenter=""
    author="Juliako"
    manager="erikre"
    editor="" />
<tags
    ms.assetid="6a63823f-f3c9-424c-91b8-566f70bec346"
    ms.service="media-services"
    ms.workload="media"
    ms.tgt_pltfrm="na"
    ms.devlang="ne"
    ms.topic="article"
    ms.date="01/31/2017"
    wacn.date="03/10/2017"
    ms.author="juliako" />  


# 将现有 Blob 复制到媒体服务资产
本主题介绍如何使用 [Azure 媒体服务 .NET SDK 扩展](https://github.com/Azure/azure-sdk-for-media-services-extensions/)将存储帐户中的 Blob 复制到新的 Azure 媒体服务 (AMS) 资产。

扩展方法适用于：

- 常规资产。
- 实时存档资产（FragBlob 格式）。
- 属于不同的媒体服务帐户（甚至不同的数据中心）的源资产和目标资产。但是，这样做可能会产生费用。有关定价的详细信息，请参阅[数据传输](/pricing/details/data-transfer/)。

>[AZURE.NOTE] 在不使用媒体服务 API 的情况下，不应该尝试更改媒体服务生成的 blob 容器内容。


本主题介绍两个代码示例：

1. 将 Blob 从一个 AMS 帐户中的资产复制到另一个 AMS 帐户中的新资产。
2. 将 Blob 从某个存储帐户复制到一个 AMS 帐户中的新资产。

## 在两个 AMS 帐户之间复制 Blob  

### 先决条件

两个媒体服务帐户。请参阅主题[如何创建媒体服务帐户](/documentation/articles/media-services-create-account/)。

### 下载示例
用户可以执行本文中的步骤，也可以单击[此处](https://azure.microsoft.com/documentation/samples/media-services-dotnet-copy-blob-into-asset/)下载包含本文所述代码的示例。

### 设置项目

1. 使用 Visual Studio 创建包含 C# 控制台应用程序项目的新解决方案。
3. 使用 [NuGet](https://www.nuget.org/packages/windowsazure.mediaservices.extensions) 安装和添加 Azure 媒体服务 .NET SDK 扩展 (windowsazure.mediaservices.extensions)。安装此包也会安装适用于 .NET 的媒体服务 SDK 并添加所有其他必需的依赖项。
4. 添加此项目所需的其他引用：System.Configuration。
6. 将 appSettings 节添加到 .config 文件，并根据媒体服务帐户、目标存储帐户和源资产 ID 更新值。
   
		<appSettings>
	          <add key="MediaServicesSourceAccountName" value="name"/>
	          <add key="MediaServicesSourceAccountKey" value="key"/>
	          <add key="MediaServicesDestAccountName" value="name"/>
	          <add key="MediaServicesDestAccountKey" value="key"/>
	          <add key="DestStorageAccountName" value="name"/>
	          <add key="DestStorageAccountKey" value="key"/>
	          <add key="SourceAssetID" value="nb:cid:UUID:assetID"/>       
		</appSettings>

### 将 Blob 从一个 AMS 帐户中的资产复制到另一个 AMS 帐户中的资产

以下代码使用扩展 **IAsset.Copy** 方法，通过单个扩展将源资产中的所有文件复制到目标资产。存在带异步支持的其他重载。

	using System;
	using System.Linq;
	using System.Configuration;
	using Microsoft.WindowsAzure.MediaServices.Client;
	using Microsoft.WindowsAzure.Storage.Auth;
	
	namespace CopyExistingBlobsIntoAsset
	{
	    class Program
	    {
	        static string _sourceAaccountName = ConfigurationManager.AppSettings["MediaServicesSourceAccountName"];
	        static string _sourceAccountKey = ConfigurationManager.AppSettings["MediaServicesSourceAccountKey"];
	        static string _destAccountName = ConfigurationManager.AppSettings["MediaServicesDestAccountName"];
	        static string _destAccountKey = ConfigurationManager.AppSettings["MediaServicesDestAccountKey"];
	        static string _destStorageAccountName = ConfigurationManager.AppSettings["DestStorageAccountName"];
	        static string _destStorageAccountKey = ConfigurationManager.AppSettings["DestStorageAccountKey"];
	        static string _sourceAssetID = ConfigurationManager.AppSettings["SourceAssetID"];
	        
		private static readonly String _defaultScope = "urn:WindowsAzureMediaServices";
		
		// Azure China uses a different API server and a different ACS Base Address from the Global.
		private static readonly String _chinaApiServerUrl = "https://wamsshaclus001rest-hs.chinacloudapp.cn/API/";
		private static readonly String _chinaAcsBaseAddressUrl = "https://wamsprodglobal001acs.accesscontrol.chinacloudapi.cn";
		
		private static Uri _apiServer = null;
	
	        private static CloudMediaContext _sourceContext = null;
	        private static CloudMediaContext _destContext = null;
	
	        static void Main(string[] args)
	        {
	            // Create the context for your source Media Services account.
	
	            // Use the cached credentials to create CloudMediaContext.
	                _sourceCachedCredentials = new MediaServicesCredentials(
	                                _sourceAaccountName,
	                                _sourceAccountKey,
									_defaultScope,
									_chinaAcsBaseAddressUrl);

					// Create the API server Uri
					_apiServer = new Uri(_chinaApiServerUrl);

	                // Used the chached credentials to create CloudMediaContext.
	                _sourceContext = new CloudMediaContext(_apiServer, _sourceCachedCredentials);


	            // Create the context for your destination Media Services account.
	                _destCachedCredentials = new MediaServicesCredentials(
	                                _destAccountName,
	                                _destAccountKey,
									_defaultScope,
									_chinaAcsBaseAddressUrl);

					// Create the API server Uri
					_apiServer = new Uri(_chinaApiServerUrl);

	                // Used the chached credentials to create CloudMediaContext.
	                _destContext = new CloudMediaContext(_apiServer, _destCachedCredentials);

	
	            // Get the credentials of the default Storage account bound to your destination Media Services account.
	            StorageCredentials destinationStorageCredentials =
	                new StorageCredentials(_destStorageAccountName, _destStorageAccountKey);
	
	            // Get a reference to the source asset in the source context.
	            IAsset sourceAsset = _sourceContext.Assets.Where(a => a.Id == _sourceAssetID).First();
	
	            // Create an empty destination asset in the destination context.
	            IAsset destinationAsset = _destContext.Assets.Create(sourceAsset.Name, AssetCreationOptions.None);
	
	            // Copy the files in the source asset instance into the destination asset instance.
				// There is an additional overload with async support.
	            sourceAsset.Copy(destinationAsset, destinationStorageCredentials);

				
	        }
	    }
	}


## 将 Blob 从存储帐户复制到 AMS 帐户 

### 先决条件

- 一个需要从其中复制 Blob 的存储帐户。
- 一个需要将 Blob 复制到其中的 AMS 帐户。

### 设置项目

1. 使用 Visual Studio 创建包含 C# 控制台应用程序项目的新解决方案。
3. 使用 [NuGet](https://www.nuget.org/packages/windowsazure.mediaservices.extensions) 安装和添加 Azure 媒体服务 .NET SDK 扩展 (windowsazure.mediaservices.extensions)。安装此包也会安装适用于 .NET 的媒体服务 SDK 并添加所有其他必需的依赖项。
4. 添加此项目所需的其他引用：System.Configuration。
6. 将 appSettings 节添加到 .config 文件，并根据源存储和目标 AMS 帐户更新值。
   
		  <appSettings>
		    <add key="MediaServicesAccountName" value="name" />
		    <add key="MediaServicesAccountKey" value="key" />
		    <add key="MediaServicesStorageAccountName" value="name" />
		    <add key="MediaServicesStorageAccountKey" value="key" />
		    <add key="SourceStorageAccountName" value="name" />
		    <add key="SourceStorageAccountKey" value="key" />
		  </appSettings>

### 将 Blob 从某个存储帐户复制到一个 AMS 帐户中的新资产

以下代码将 Blob 从存储帐户复制到媒体服务资产。

	using System;
	using System.Configuration;
	using System.Linq;
	using Microsoft.WindowsAzure.MediaServices.Client;
	using System.Threading;
	using Microsoft.WindowsAzure.Storage.Auth;
	using Microsoft.WindowsAzure.Storage;
	using Microsoft.WindowsAzure.Storage.Blob;
	using System.Threading.Tasks;
	
	namespace CopyExistingBlobsIntoAsset
	{
	    class Program
	    {
	        // Read values from the App.config file.
	        private static readonly string _mediaServicesAccountName =
	            ConfigurationManager.AppSettings["MediaServicesAccountName"];
	        private static readonly string _mediaServicesAccountKey =
	            ConfigurationManager.AppSettings["MediaServicesAccountKey"];
	        private static readonly string _mediaServicesStorageAccountName =
	            ConfigurationManager.AppSettings["MediaServicesStorageAccountName"];
	        private static readonly string _mediaServicesStorageAccountKey =
	            ConfigurationManager.AppSettings["MediaServicesStorageAccountKey"];
	        private static readonly string _sourceStorageAccountName =
	     ConfigurationManager.AppSettings["SourceStorageAccountName"];
	        private static readonly string _sourceStorageAccountKey =
	            ConfigurationManager.AppSettings["SourceStorageAccountKey"];
		    
		private static readonly String _defaultScope = "urn:WindowsAzureMediaServices";
		
		// Azure China uses a different API server and a different ACS Base Address from the Global.
		private static readonly String _chinaApiServerUrl = "https://wamsshaclus001rest-hs.chinacloudapp.cn/API/";
		private static readonly String _chinaAcsBaseAddressUrl = "https://wamsprodglobal001acs.accesscontrol.chinacloudapi.cn";
		
		private static Uri _apiServer = null;
	
	        // Field for service context.
	        private static CloudMediaContext _context = null;
	        private static CloudStorageAccount _sourceStorageAccount = null;
	        private static CloudStorageAccount _destinationStorageAccount = null;
	
	        static void Main(string[] args)
	        {
	
	            // Used the cached credentials to create CloudMediaContext.
	                _cachedCredentials = new MediaServicesCredentials(
	                                _mediaServicesAccountName,
	                                _mediaServicesAccountKey,
									_defaultScope,
									_chinaAcsBaseAddressUrl);

					// Create the API server Uri
					_apiServer = new Uri(_chinaApiServerUrl);

	                // Used the chached credentials to create CloudMediaContext.
	                _context = new CloudMediaContext(_apiServer, _cachedCredentials);

	
	            _sourceStorageAccount = 
					new CloudStorageAccount(new StorageCredentials(_sourceStorageAccountName, 
						_sourceStorageAccountKey), true);
	
	            _destinationStorageAccount = 
					new CloudStorageAccount(new StorageCredentials(_mediaServicesStorageAccountName, 
						_mediaServicesStorageAccountKey), true);
	            
	            CloudBlobClient sourceCloudBlobClient = 
					_sourceStorageAccount.CreateCloudBlobClient();
	            CloudBlobContainer sourceContainer = 
					sourceCloudBlobClient.GetContainerReference("NameOfBlobContainerYouWantToCopy");
	
	            CreateAssetFromExistingBlobs(sourceContainer);
	        }
	        
	        static public IAsset CreateAssetFromExistingBlobs(CloudBlobContainer sourceBlobContainer)
	        {
	            CloudBlobClient destBlobStorage = _destinationStorageAccount.CreateCloudBlobClient();
	
	            // Create a new asset. 
	            IAsset asset = _context.Assets.Create("NewAsset_" + Guid.NewGuid(), AssetCreationOptions.None);
	
	            IAccessPolicy writePolicy = _context.AccessPolicies.Create("writePolicy",
	                TimeSpan.FromHours(24), AccessPermissions.Write);
	
	            ILocator destinationLocator = 
					_context.Locators.CreateLocator(LocatorType.Sas, asset, writePolicy);
	
	            // Get the asset container URI and Blob copy from mediaContainer to assetContainer. 
	            CloudBlobContainer destAssetContainer =
	                destBlobStorage.GetContainerReference((new Uri(destinationLocator.Path)).Segments[1]);
	
	            if (destAssetContainer.CreateIfNotExists())
	            {
	                destAssetContainer.SetPermissions(new BlobContainerPermissions
	                {
	                    PublicAccess = BlobContainerPublicAccessType.Blob
	                });
	            }
	
	            var blobList = sourceBlobContainer.ListBlobs();
	
	            foreach (var sourceBlob in blobList)
	            {
	                var assetFile = asset.AssetFiles.Create((sourceBlob as ICloudBlob).Name);
	
	                ICloudBlob destinationBlob = destAssetContainer.GetBlockBlobReference(assetFile.Name);
	
	                // Call the CopyBlobHelpers.CopyBlobAsync extension method to copy blobs.
	                using (Task task = 
	                    CopyBlobHelpers.CopyBlobAsync((CloudBlockBlob)sourceBlob, 
	                        (CloudBlockBlob)destinationBlob, 
	                        new BlobRequestOptions(), 
	                        CancellationToken.None))
	                {
	                    task.Wait();
	                }
	
	                assetFile.ContentFileSize = (sourceBlob as ICloudBlob).Properties.Length;
	                assetFile.Update();
	                Console.WriteLine("File {0} is of {1} size", assetFile.Name, assetFile.ContentFileSize);
	            }
	
	            asset.Update();
	
	            destinationLocator.Delete();
	            writePolicy.Delete();
	
	            // Set the primary asset file.
	            // If, for example, we copied a set of Smooth Streaming files, 
	            // set the .ism file to be the primary file. 
	            // If we, for example, copied an .mp4, then the mp4 would be the primary file. 
	            var ismAssetFiles = asset.AssetFiles.ToList().
	                Where(f => f.Name.EndsWith(".ism", StringComparison.OrdinalIgnoreCase)).ToArray();
	
	            // The following code assigns the first .ism file as the primary file in the asset.
	            // An asset should have one .ism file.  
	            ismAssetFiles.First().IsPrimary = true;
	            ismAssetFiles.First().Update();
	
	            return asset;
	        }
	    }
	}
	
## 后续步骤

现即可编码已上传的资产。有关详细信息，请参阅[对资产进行编码](/documentation/articles/media-services-encode-asset/)。

<!---HONumber=Mooncake_0306_2017-->
<!--Update_Description: whole content update to new code samples-->