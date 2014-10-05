<properties linkid="develop-media-services-how-to-guides-create-assets" urlDisplayName="Create Encrypted Asset and Upload to Storage" pageTitle="Create Encrypted Asset and Upload to Storage Azure" metaKeywords="" description="Learn how to get media content into Media Services by creating and uploading an encrypted asset." metaCanonical="" services="media-services" documentationCenter="" title="How to: Create an encrypted Asset and upload to storage" authors="migree" solutions="" manager="" editor="" />

如何：创建加密的资产并上载到存储中
==================================

本文是介绍 Azure Media Services 编程的系列主题中的一篇。前一个主题是[针对 Media Services 设置计算机](http://go.microsoft.com/fwlink/?LinkID=301751&clcid=0x409)。

若要将媒体内容添加到 Media Services 中，请先创建一个资产并在其中添加文件，然后上载该资产。此过程称为引入内容。

创建资产时，可以指定三个不同的加密选项。

-   **AssetCreationOptions.None**：不加密。如果你想要创建不加密的资产，则必须设置此选项。
-   **AssetCreationOptions.CommonEncryptionProtected**：适用于通用加密保护 (CENC) 文件，例如，已进行 PlayReady 加密的一组文件。
-   **AssetCreationOptions.StorageEncrypted**：存储加密。将明文输入文件上载到 Azure 存储空间之前对其进行加密。

**注意**：Media Services 提供磁盘存储加密，而不通过数字版权管理器 (DRM) 等途径加密。

下面的示例代码将执行以下操作：

-   创建空资产。
-   创建要与资产关联的 AssetFile 实例。
-   创建用于定义权限以及资产访问持续时间的 AccessPolicy 实例。
-   创建用于提供资产访问权限的 Locator 实例。
-   将单个媒体文件上载到 Media Services。

``` {}
static private IAsset CreateEmptyAsset(string assetName, AssetCreationOptions assetCreationOptions)
{
var asset = _context.Assets.Create(assetName, assetCreationOptions);

Console.WriteLine("Asset name:" + asset.Name);
Console.WriteLine("Time created:" + asset.Created.Date.ToString());

return asset;
}

static public IAsset CreateAssetAndUploadSingleFile(AssetCreationOptions assetCreationOptions, string singleFilePath)
{
var assetName = "UploadSingleFile_" + DateTime.UtcNow.ToString();
var asset = CreateEmptyAsset(assetName, assetCreationOptions);

var fileName = Path.GetFileName(singleFilePath);

var assetFile = asset.AssetFiles.Create(fileName);

Console.WriteLine("Created assetFile {0}", assetFile.Name);
Console.WriteLine("Upload {0}", assetFile.Name);

assetFile.Upload(singleFilePath);
Console.WriteLine("Done uploading of {0} using Upload()", assetFile.Name);

return asset;
}
```

以下代码演示如何创建资产及上载多个文件。

``` {}
static public IAsset CreateAssetAndUploadMultipleFiles( AssetCreationOptions assetCreationOptions, string folderPath)
{
var assetName = "UploadMultipleFiles_" + DateTime.UtcNow.ToString();

var asset = CreateEmptyAsset(assetName, assetCreationOptions);

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
throw new FileNotFoundException(String.Format("No files in directory, check folderPath:{0}", folderPath));
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
```

后续步骤
--------

将资产上载到 Media Services 后，请转到[如何获取媒体处理器](http://go.microsoft.com/fwlink/?LinkID=301732&clcid=0x409)主题。

