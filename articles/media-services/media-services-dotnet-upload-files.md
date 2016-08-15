<properties 
	pageTitle="使用 .NET 将文件上载到媒体服务帐户" 
	description="了解如何通过创建和上载资产将媒体内容加入媒体服务。" 
	services="media-services" 
	documentationCenter="" 
	authors="juliako" 
	manager="dwrede" 
	editor=""/>

<tags
	ms.service="media-services"
 	ms.date="04/18/2016" 
	wacn.date="06/20/2016"/>



#使用 .NET 将文件上载到媒体服务帐户

[AZURE.INCLUDE [media-services-selector-upload-files](../includes/media-services-selector-upload-files.md)]

在媒体服务中，可以将数字文件上载（引入）到资产中。**资产**实体可以包含视频、音频、图像、缩略图集合、图文轨迹和隐藏字幕文件（以及有关这些文件的元数据。） 上载文件完成后，相关内容即安全地存储在云中供后续处理和流式处理。

资产中的文件称为**资产文件**。**AssetFile** 实例和实际媒体文件是两个不同的对象。AssetFile 实例包含有关媒体文件的元数据，而媒体文件包含实际媒体内容。

在创建资产时，可以指定以下加密选项。

- **无** - 不使用加密。这是默认值。请注意，使用此选项时，你的内容在传送过程中或静态存储过程中都不会受到保护。如果计划使用渐进式下载交付 MP4，则使用此选项。 
- **CommonEncryption** - 上载经过常用加密或 PlayReady DRM 加密并受其保护的内容（例如，受 PlayReady DRM 保护的平滑流）时使用此选项。
- **EnvelopeEncrypted** – 如果要上载使用 AES 加密的 HLS，请使用此选项。请注意，Transform Manager 必须已对文件进行编码和加密。
- **StorageEncrypted** - 使用 AES-256 位加密在本地加密明文内容，然后将其上载到 Azure Storage 中以加密形式静态存储相关内容。受存储加密保护的资产将在编码前自动解密并放入经过加密的文件系统中，并可选择在重新上载为新的输出资产前重新加密。存储加密的主要用例是在磁盘上通过静态增强加密来保护高品质的输入媒体文件。

	媒体服务为资产提供磁盘上的存储加密，而不是通过数字权限管理器 (DRM) 等线路提供加密。

	如果你的资产已经过存储加密，则必须配置资产传送策略。有关详细信息，请参阅[配置资产传送策略](/documentation/articles/media-services-dotnet-configure-asset-delivery-policy/)。

如果指定使用 **CommonEncrypted** 选项或 **EnvelopeEncypted** 选项加密资产，则需要将资产关联到 **ContentKey**。有关详细信息，请参阅[如何创建 ContentKey](/documentation/articles/media-services-dotnet-create-contentkey/)。

如果指定使用 **StorageEncrypted** 选项加密资产，适用于 .NET 的媒体服务 SDK 将为资产创建 **StorateEncrypted** **ContentKey**。

>[AZURE.NOTE]构建流内容的 URL 时，媒体服务会使用 IAssetFile.Name 属性的值（如 http://{AMSAccount}.origin.mediaservices.chinacloudapi.cn/{GUID}/{IAssetFile.Name}/streamingParameters.）。出于这个原因，不允许使用百分号编码。**Name** 属性的值不能含有任何以下保留的[百分号编码字符](http://zh.wikipedia.org/wiki/百分号编码#.E4.BF.9D.E7.95.99.E5.AD.97.E7.AC.A6.E7.9A.84.E7.99.BE.E5.88.86.E5.8F.B7.E7.BC.96.E7.A0.81)：!*'();:@&=+$,/?%#"。此外，文件扩展名中只能含有一个“.”。

本主题说明如何使用媒体服务.NET SDK 以及媒体服务.NET SDK Extensions 将文件上载到媒体服务资产中。

 
## 使用媒体服务 .NET SDK 上载单个文件 

以下示例代码使用 .NET SDK 执行以下任务：

- 创建空资产。
- 创建要与资产关联的 AssetFile 实例。
- 创建用于定义权限以及资产访问持续时间的 AccessPolicy 实例。
- 创建用于提供资产访问权限的 Locator 实例。
- 将单个媒体文件上载到媒体服务。 

		
		static public IAsset CreateAssetAndUploadSingleFile(AssetCreationOptions assetCreationOptions, string singleFilePath)
		{
            if (!File.Exists(singleFilePath))
            {
                Console.WriteLine("File does not exist.");
                return null;
            }

            var assetName = Path.GetFileNameWithoutExtension(singleFilePath);
            IAsset inputAsset = _context.Assets.Create(assetName, assetCreationOptions); 

            var assetFile = inputAsset.AssetFiles.Create(Path.GetFileName(singleFilePath));

            Console.WriteLine("Created assetFile {0}", assetFile.Name);

            var policy = _context.AccessPolicies.Create(
                                    assetName,
                                    TimeSpan.FromDays(30),
                                    AccessPermissions.Write | AccessPermissions.List);

            var locator = _context.Locators.CreateLocator(LocatorType.Sas, inputAsset, policy);

            Console.WriteLine("Upload {0}", assetFile.Name);

            assetFile.Upload(singleFilePath);
            Console.WriteLine("Done uploading {0}", assetFile.Name);

            locator.Delete();
            policy.Delete();

            return inputAsset;
		}

##使用媒体服务 .NET SDK 上载多个文件 

以下代码演示如何创建资产及上载多个文件。

代码将执行以下操作：
	
- 	使用上一步中定义的 CreateEmptyAsset 方法创建一个空资产。
 	
- 	创建用于定义权限以及资产访问持续时间的 **AccessPolicy** 实例。
 	
- 	创建用于提供资产访问权限的 **Locator** 实例。
 	
- 	创建 **BlobTransferClient** 实例。此类型表示对 Azure Blob 进行操作的客户端。在此示例中，我们使用客户端来监视上载进度。
 	
- 	枚举指定目录下的所有文件，并为每个文件创建一个 **AssetFile** 实例。
 	
- 	使用 **UploadAsync** 方法将文件上载到媒体服务中。
 	
>[AZURE.NOTE] 使用 UploadAsync 方法可确保调用不会阻塞并且文件并行上载。
 	
 	
        static public IAsset CreateAssetAndUploadMultipleFiles(AssetCreationOptions assetCreationOptions, string folderPath)
        {
            var assetName = "UploadMultipleFiles_" + DateTime.UtcNow.ToString();

            IAsset asset = _context.Assets.Create(assetName, assetCreationOptions);

            var accessPolicy = _context.AccessPolicies.Create(assetName, TimeSpan.FromDays(30),
                                                                AccessPermissions.Write | AccessPermissions.List);

            var locator = _context.Locators.CreateLocator(LocatorType.Sas, asset, accessPolicy);

            var blobTransferClient = new BlobTransferClient();
            blobTransferClient.NumberOfConcurrentTransfers = 20;
            blobTransferClient.ParallelTransferThreadCount = 20;

            blobTransferClient.TransferProgressChanged += blobTransferClient_TransferProgressChanged;

            var filePaths = Directory.EnumerateFiles(folderPath);

            Console.WriteLine("There are {0} files in {1}", filePaths.Count(), folderPath);

            if (!filePaths.Any())
            {
                throw new FileNotFoundException(String.Format("No files in directory, check folderPath: {0}", folderPath));
            }

            var uploadTasks = new List<Task>();
            foreach (var filePath in filePaths)
            {
                var assetFile = asset.AssetFiles.Create(Path.GetFileName(filePath));
                Console.WriteLine("Created assetFile {0}", assetFile.Name);

                // It is recommended to validate AccestFiles before upload. 
                Console.WriteLine("Start uploading of {0}", assetFile.Name);
                uploadTasks.Add(assetFile.UploadAsync(filePath, blobTransferClient, locator, CancellationToken.None));
            }

            Task.WaitAll(uploadTasks.ToArray());
            Console.WriteLine("Done uploading the files");

            blobTransferClient.TransferProgressChanged -= blobTransferClient_TransferProgressChanged;

            locator.Delete();
            accessPolicy.Delete();

            return asset;
        }
	
	static void  blobTransferClient_TransferProgressChanged(object sender, BlobTransferProgressChangedEventArgs e)
	{
	    if (e.ProgressPercentage > 4) // Avoid startup jitter, as the upload tasks are added.
	    {
	        Console.WriteLine("{0}% upload competed for {1}.", e.ProgressPercentage, e.LocalFile);
	    }
	}



上载大量资产时，请注意以下事项。

- 每个线程创建一个新的 **CloudMediaContext** 对象。**CloudMediaContext** 类不是线程安全的。
 
- 将 NumberOfConcurrentTransfers 从默认值 2 增加到更高的值（如 5）。设置此属性将影响 **CloudMediaContext** 的所有实例。
 
- 将 ParallelTransferThreadCount 保留为默认值 10。
 
##<a id="ingest_in_bulk"></a>使用媒体服务 .NET SDK 批量引入资产 

上载大型资产文件可能在资产创建过程中形成瓶颈。批量引入资产（简称“批量引入”）涉及到将资产创建过程与上载过程分离。若要使用批量引入方法，请创建一个描述资产及其关联文件的清单 (IngestManifest)。然后，你可以使用所选上载方法将关联的文件上载到该清单的 Blob 容器。Azure 媒体服务将会监视与清单关联的 Blob 容器。文件上载到 Blob 容器后，Azure 媒体服务将基于清单 (IngestManifestAsset) 中资产的配置完成资产创建过程。


若要创建新的 IngestManifest，请调用通过 CloudMediaContext 中的 IngestManifests 集合公开的 Create 方法。此方法将使用你提供的清单名称创建一个新的 IngestManifest。

	IIngestManifest manifest = context.IngestManifests.Create(name);

创建将与批量 IngestManifest 关联的资产。在要批量引入的资产上配置所需的加密选项。

	// Create the assets that will be associated with this bulk ingest manifest
	IAsset destAsset1 = _context.Assets.Create(name + "_asset_1", AssetCreationOptions.None);
	IAsset destAsset2 = _context.Assets.Create(name + "_asset_2", AssetCreationOptions.None);

一个 IngestManifestAsset 将一个资产与一个用于批量引入的批量 IngestManifest 相关联。它还关联构成每个资产的 AssetFiles。若要创建 IngestManifestAsset，请使用服务器上下文中的 Create 方法。

以下示例演示如何添加两个新的 IngestManifestAssets，这两项将以前创建的两个资产关联到批量引入清单。每个 IngestManifestAsset 还关联一组将在批量引入期间为每个资产上载的文件。

	string filename1 = _singleInputMp4Path;
	string filename2 = _primaryFilePath;
	string filename3 = _singleInputFilePath;
	
	IIngestManifestAsset bulkAsset1 =  manifest.IngestManifestAssets.Create(destAsset1, new[] { filename1 });
	IIngestManifestAsset bulkAsset2 =  manifest.IngestManifestAssets.Create(destAsset2, new[] { filename2, filename3 });
	
可以使用任何能够将资产文件上载到 Blob 存储容器 URI（由 IngestManifest 的 **IIngestManifest.BlobStorageUriForUpload** 属性提供）的高速客户端应用程序。一个明显的高速上载服务就是[适用于 Azure 应用程序的点播 Aspera](https://datamarket.azure.com/application/2cdbc511-cb12-4715-9871-c7e7fbbb82a6)。你还可以编写代码来上载资产文件，如以下代码示例所示。
	
	static void UploadBlobFile(string destBlobURI, string filename)
	{
	    Task copytask = new Task(() =>
	    {
	        var storageaccount = new CloudStorageAccount(new StorageCredentials(_storageAccountName, _storageAccountKey), true);
	        CloudBlobClient blobClient = storageaccount.CreateCloudBlobClient();
	        CloudBlobContainer blobContainer = blobClient.GetContainerReference(destBlobURI);
	
	        string[] splitfilename = filename.Split('\\');
	        var blob = blobContainer.GetBlockBlobReference(splitfilename[splitfilename.Length - 1]);
	
	        using (var stream = System.IO.File.OpenRead(filename))
	            blob.UploadFromStream(stream);
	
	        lock (consoleWriteLock)
	        {
	            Console.WriteLine("Upload for {0} completed.", filename);
	        }
	    });
	
	    copytask.Start();
	}

以下代码示例中显示了用于上载本主题中使用的示例资源文件的代码。
	
	UploadBlobFile(manifest.BlobStorageUriForUpload, filename1);
	UploadBlobFile(manifest.BlobStorageUriForUpload, filename2);
	UploadBlobFile(manifest.BlobStorageUriForUpload, filename3);
	

可以通过轮询 **IngestManifest** 的 Statistics 属性来确定与 **IngestManifest** 关联的所有资产的批量引入进度。若要更新进度信息，每次轮询 Statistics 属性时，都必须使用新的 **CloudMediaContext**。

以下示例演示如何按 **ID** 轮询 IngestManifest。
	
	static void MonitorBulkManifest(string manifestID)
	{
	   bool bContinue = true;
	   while (bContinue)
	   {
	      CloudMediaContext context = GetContext();
	      IIngestManifest manifest = context.IngestManifests.Where(m => m.Id == manifestID).FirstOrDefault();
	
	      if (manifest != null)
	      {
	         lock(consoleWriteLock)
	         {
	            Console.WriteLine("\nWaiting on all file uploads.");
	            Console.WriteLine("PendingFilesCount  : {0}", manifest.Statistics.PendingFilesCount);
	            Console.WriteLine("FinishedFilesCount : {0}", manifest.Statistics.FinishedFilesCount);
	            Console.WriteLine("{0}% complete.\n", (float)manifest.Statistics.FinishedFilesCount / (float)(manifest.Statistics.FinishedFilesCount + manifest.Statistics.PendingFilesCount) * 100);
	
	            if (manifest.Statistics.PendingFilesCount == 0)
	            {
	               Console.WriteLine("Completed\n");
	               bContinue = false;
	            }
	         }
	
	         if (manifest.Statistics.FinishedFilesCount < manifest.Statistics.PendingFilesCount)
	            Thread.Sleep(60000);
	      }
	      else // Manifest is null
	         bContinue = false;
	   }
	}
	


##使用 .NET SDK Extensions 上载文件 

以下示例演示如何使用 .NET SDK Extensions 上载单个文件。在此情况下，将使用 **CreateFromFile** 方法，但也可以使用异步版本 (**CreateFromFileAsync**)。**CreateFromFile** 方法可让你指定文件名、加密选项和回调，以报告文件的上载进度。


	static public IAsset UploadFile(string fileName, AssetCreationOptions options)
	{
	    IAsset inputAsset = _context.Assets.CreateFromFile(
	        fileName,
	        options,
	        (af, p) =>
	        {
	            Console.WriteLine("Uploading '{0}' - Progress: {1:0.##}%", af.Name, p.Progress);
	        });
	
	    Console.WriteLine("Asset {0} created.", inputAsset.Id);
	
	    return inputAsset;
	}

以下示例将调用 UploadFile 函数，并指定存储加密作为资产创建选项。


	var asset = UploadFile(@"C:\VideoFiles\BigBuckBunny.mp4", AssetCreationOptions.StorageEncrypted);




##后续步骤
将资产上载到媒体服务后，请转到[如何获取媒体处理器][]主题。

[如何获取媒体处理器]: /documentation/articles/media-services-get-media-processor/
 

<!---HONumber=Mooncake_0613_2016-->