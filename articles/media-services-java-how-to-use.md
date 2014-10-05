<properties linkid="develop-media-services-how-to-guides-media-services-java" urlDisplayName="Media Services" pageTitle="How to use Media Services (Java) - Azure feature guide" metaKeywords="Azure Media Services, Azure media, Azure streaming, azure media, azure streaming, azure encoding" description="Describes how to use Azure Media Services to perform common tasks including encoding, encrypting, and streaming resources." metaCanonical="" services="media-services" documentationCenter="Java" title="How to Use Media Services" authors="robmcm" solutions="" manager="wpickett" editor="mollybos" scriptId="" videoId="" />

如何使用 Media Services
=======================

本指南说明如何结合使用 Java 与 Azure Media Services 开始编程。本指南提供 Media Services 的技术概述、针对 Media Services 配置 Azure 帐户的步骤，以及演示如何完成典型编程任务的代码。

目录
----

-   [什么是 Media Services？](#what-are)
-   [针对 Media Services 设置 Azure 帐户](#setup-account)
-   [完成设置以进行 Media Services 开发](#setup-dev)
-   [如何：将 Media Services 与 Java 结合使用](#connect)
-   [其他资源](#additional-resources)

什么是 Media Services？什么是 Media Services？
----------------------------------------------

Azure Media Services 构成了一个可扩展的媒体平台，它在 Azure 中集成了 Microsoft Media Platform 和第三方媒体组件的精华。Media Services 在云中提供一个媒体管道，让行业合作伙伴扩展或取代组件技术。ISV 和媒体提供商可以使用 Media Services 来生成端到端媒体解决方案。本概述主题将介绍 Media Services 的一般体系结构和常见开发方案。

下图展示了 Media Services 的基本体系结构。

![Media Services 体系结构](./media/media-services-dotnet-how-to-use/wams-01.png)

### Media Services 功能支持

当前版本的 Media Services 提供以下功能集，用于在云中开发媒体应用程序。

-   **引入**。引入操作可将资产插入到系统，例如，先将其上载并加密，然后再将其放入 Azure 存储空间。随着 RTM 版本的发布，Media Services 将允许与合作伙伴组件相集成，以提供快速 UDP（用户数据报协议）上载解决方案。
-   **编码**。编码操作包括编码、变换和转换媒体资产。你可以使用 Media Services 随附的媒体编码器在云中运行编码任务。编码选项包括：
    -   使用 Azure 媒体编码器并操作一系列标准编解码器和格式，包括行业领先的 IIS 平滑流式处理和 MP4，以及将相关格式转换为 Apple HTTP 实时流。
    -   转换整个库或单个文件，并获取对输入和输出的全面控制权。
    -   受支持的文件类型、格式和编解码器众多（请参阅 [Media Services 支持的文件类型][]）。
    -   支持的格式转换。Media Services 允许你将 ISO MP4 (.mp4) 转换为平滑流式处理文件格式 (PIFF 1.3)（.ismv；.isma）。还可以将平滑流式处理文件格式 (PIFF) 转换为 Apple HTTP 实时流（.msu8、.ts）。
-   **保护**。保护内容意味着对实时流内容或点播内容进行加密，以安全地进行传输、存储和交付。Media Services 提供 DRM 技术感知的解决方案来保护内容。当前支持的 DRM 技术包括 Microsoft PlayReady 保护和 MPEG 通用加密。将来会提供对其他 DRM 技术的支持。
-   **流式处理**。流式处理内容涉及到将实时内容或点播内容发送到客户端，或者让你从云中检索或下载特定的媒体文件。Media Services 提供格式感知的解决方案来流式处理内容。Media Services 针对平滑流式处理、Apple HTTP 实时流和 MP4 格式提供流式来源支持。将来会提供对其他格式的支持。你也可以使用 Azure CDN 或第三方 CDN（启用相应的选项即可扩展为支持数百万个用户）无缝交付流式处理内容。

### Media Services 开发方案

Media Services 支持下表中所述的多种常见媒体开发方案。

<table data-morhtml="true" border="2" cellspacing="0" cellpadding="5" style="border: 2px solid #000000;">
  <thead data-morhtml="true">
    <tr data-morhtml="true">
<th data-morhtml="true">方案</th>
<th data-morhtml="true">说明</th>
    </tr>
  </thead>
  <tbody data-morhtml="true">
    <tr data-morhtml="true">
<td data-morhtml="true">生成端到端工作流</td>
<td data-morhtml="true">完全在云中生成综合性的媒体工作流。从上载媒体到分发内容，Media Services 提供一系列组件，将这些组件相结合可以处理特定的应用工作流。当前功能包括上载、存储、编码、格式转换、内容保护和点播流交付。</td>
    </tr>
    <tr data-morhtml="true">
<td data-morhtml="true">生成混合工作流</td>
<td data-morhtml="true">你可以将 Media Services 与现有工具和过程相集成。例如，在现场为内容编码，再将其上载到 Media Services 以转码为多种格式，然后通过 Azure CDN 或第三方 CDN 交付内容。可以通过标准 REST API 单独调用 Media Services，以将它与外部应用程序和服务相集成。</td>
    </tr>
    <tr data-morhtml="true">
<td data-morhtml="true">为媒体播放器提供云支持</td>
<td data-morhtml="true">你可以跨多个设备（包括 iOS、Android 和 Windows 设备）与平台创建、管理和交付媒体。</td>
    </tr>
  </tbody>
</table>

### Media Services 客户端开发

使用 SDK 和播放器框架扩展 Media Services 解决方案的功能范围，以生成媒体客户端应用程序。开发人员可以使用这些客户端来生成 Media Services 应用程序，在各种设备和平台上提供引人入胜的用户体验。根据要生成的客户端应用程序的目标设备，你可以选择 Microsoft 和其他第三方合作伙伴提供的多种 SDK 和播放器框架。

下面提供了可用客户端 SDK 和播放器框架的列表。有关此处所列和其他已计划的 SDK 与播放器框架及其支持的功能的详细信息，请参阅 [Media Services 客户端开发](http://msdn.microsoft.com/zh-cn/library/windowsazure/dn223283.aspx)。

#### Mac 和 PC 客户端支持

对于 PC 和 Mac，你可以使用 Microsoft Silverlight 或 Adobe Open Source Media Framework 来针对性地设计流式处理体验。

-   [适用于 Silverlight 的平滑流式处理客户端](http://www.iis.net/download/smoothclient)
-   [Microsoft Media Platform：适用于 Silverlight 的播放器框架](http://smf.codeplex.com/documentation)
-   [适用于 OSMF 2.0 的平滑流式处理插件](http://go.microsoft.com/fwlink/?LinkId=275022)。有关如何使用此插件的信息，请参阅[如何使用适用于 Adobe Open Source Media Framework 的平滑流式处理插件](http://go.microsoft.com/fwlink/?LinkId=275034)。

#### Windows 8 应用程序

对于 Windows 8，你可以使用支持的任一开发语言和构造（例如 HTML、Javascript、XAML、C\# 和 C+）生成 Windows 应用商店应用程序。

-   [适用于 Windows 8 的平滑流式处理客户端 SDK](http://go.microsoft.com/fwlink/?LinkID=246146)。有关如何使用此 SDK 创建 Windows 应用商店应用程序的详细信息，请参阅[如何生成平滑流式处理 Windows 应用商店应用程序](http://go.microsoft.com/fwlink/?LinkId=271647)。有关如何使用 HTML5 语言创建平滑流式处理播放器的信息，请参阅[演练：生成你的第一个 HTML5 平滑流式处理播放器](http://msdn.microsoft.com/zh-cn/library/jj573656.aspx)。

-   [Microsoft Media Platform：适用于 Windows 8 Windows 应用商店应用程序的播放器框架](http://playerframework.codeplex.com/wikipage?title=Player%20Framework%20for%20Windows%208%20Metro%20Style%20Apps&referringTitle=Home)

#### Xbox

Xbox 支持可使用平滑流式处理内容的 Xbox LIVE 应用程序。Xbox LIVE 应用程序开发工具包 (ADK) 包含：

-   适用于 Xbox LIVE ADK 的平滑流式处理客户端
-   Microsoft Media Platform：适用于 Xbox LIVE ADK 的播放器框架

#### 嵌入式设备或专用设备

联网的电视机、机顶盒、蓝光播放机、智能电视机顶盒等设备，以及带有自定义应用程序开发框架和自定义媒体管道的移动设备。Microsoft 提供以下可购买许可的移植工具包，并允许合作伙伴为平台移植平滑流式处理播放功能。

-   [平滑流式处理客户端移植工具包](http://www.microsoft.com/zh-cn/mediaplatform/sspk.aspx)
-   [Microsoft PlayReady 设备移植工具包](http://www.microsoft.com/PlayReady/Licensing/device_technology.mspx)

#### Windows Phone

Microsoft 提供可用于生成 Windows Phone 版高级视频应用程序的 SDK。

-   [适用于 Silverlight 的平滑流式处理客户端](http://www.iis.net/download/smoothclient)
-   [Microsoft Media Platform：适用于 Silverlight 的播放器框架](http://smf.codeplex.com/documentation)

#### iOS 设备

对于 iPhone、iPod 和 iPad 等 iOS 设备，Microsoft 随附了一个 SDK，让你针对这些平台生成应用程序，以交付高级视频内容：适用于具有 PlayReady 功能的 iOS 设备的平滑流式处理 SDK。该 SDK 仅向许可接受方提供，有关详细信息，请[向 Microsoft 发送电子邮件](mailto:askdrm@microsoft.com)。有关 iOS 开发的信息，请参阅 [iOS 开发人员中心](https://developer.apple.com/devcenter/ios/index.action)。

#### Android 设备

有许多 Microsoft 合作伙伴针对 Android 平台提供 SDK，用于在 Android 设备上添加播放平滑流式处理内容的功能。有关这些合作伙伴的更多详细信息，请[向 Microsoft 发送电子邮件](mailto:sspkinfo@microsoft.com?subject=Partner%20SDKs%20for%20Android%20Devices)。

设置帐户针对 Media Services 设置 Azure 帐户
-------------------------------------------

若要设置你的 Media Services 帐户，请使用 Azure 管理门户。请参阅主题[如何创建 Media Services 帐户](http://go.microsoft.com/fwlink/?linkid=256662)。在管理门户中创建帐户后，便可以设置你的计算机以进行 Media Services 开发。

完成设置以进行 Media Services 开发
----------------------------------

本部分介绍使用 Media Services SDK for Java 进行 Media Services 开发需要满足的一般性先决条件。

### 先决条件

-   在新的或现有的 Azure 订阅中拥有一个 Media Services 帐户。请参阅主题[如何创建 Media Services 帐户](http://go.microsoft.com/fwlink/?linkid=256662)。
-   适用于 Java 的 Azure 库，可以从 [Azure Java 开发人员中心](http://www.windowsazure.cn/zh-cn/develop/java/)安装。

将 Media Services 与 Java 结合使用如何：将 Media Services 与 Java 结合使用
--------------------------------------------------------------------------

以下代码演示了如何创建一个资产、如何将媒体文件上载到该资产、如何使用任务运行某个作业以转换该资产，以及如何下载转换后的资产的输出文件。

在使用此代码之前，需要设置一个 Media Services 帐户。有关设置帐户的信息，请参阅[如何创建 Media Services 帐户](http://www.windowsazure.com/zh-cn/manage/services/media-services/how-to-create-a-media-services-account/)。

将 `clientId` 和 `clientSecret` 变量替换为你自己的值。该代码还依赖于本地存储的文件 `c:/media/MPEG4-H264.mp4`。你需要提供自己的文件以供使用。该代码还需要一个输出文件夹 `c:/output`，输出文件将下载到其中。

    import java.io.*;
    import java.net.URI;
    import java.net.URISyntaxException;
    import java.security.NoSuchAlgorithmException;
    import java.util.EnumSet;
    import java.util.List;

    import com.microsoft.windowsazure.services.blob.client.*;
    import com.microsoft.windowsazure.services.core.Configuration;
    import com.microsoft.windowsazure.services.core.ServiceException;
    import com.microsoft.windowsazure.services.core.storage.StorageException;
    import com.microsoft.windowsazure.services.media.*;
    import com.microsoft.windowsazure.services.media.models.*;

    public class HelloMediaServices 
    {

    private static MediaContract mediaService;
    private static AssetInfo assetToEncode, encodedAsset;

    public static void main(String[] args) 
        {
    try 
            {

    // Set up the MediaContract object to call into the media services.
    Init();
                
    // Upload a local file to a media asset.
    Upload();

    // Transform the asset.
    Transform();

    // Retrieve the URL of the asset's transformed output.
    Download();

    // Delete all assets. 
    // When you want to delete the assets that have been uploaded, 
    // comment out the calls to Upload(), Transfer(), and Download(), 
    // and uncomment the following call to Cleanup().
    //Cleanup();

    System.out.println("Application completed.");
            }
    catch (ServiceException se) 
            {
    System.out.println("ServiceException encountered.");
    System.out.println(se.getMessage());
            }
    catch (Exception e) 
            {
    System.out.println("Exception encountered.");
    System.out.println(e.getMessage());
            }
        }

    // Initialize the server context to get programmatic access to the Media Services programming objects.
    // The media services URI, OAuth URI and scope can be used exactly as shown.
    // Substitute your media service account name and access key for the clientId and clientSecret variables.
    // You can obtain your media service account name and access key from the Media Services section
    // of the Azure Management portal, https://manage.windowsazure.com.
    private static void Init() throws ServiceException 
        {
    String mediaServiceUri = "https://media.windows.net/API/";
    String oAuthUri = "https://wamsprodglobal001acs.accesscontrol.windows.net/v2/OAuth2-13";
    String clientId = "your_client_id";  // Use your media service account name.
    String clientSecret = "your_client_secret"; // Use your media service access key. 
    String scope = "urn:WindowsAzureMediaServices";

    // Specify the configuration values to use with the MediaContract object.
    Configuration configuration = MediaConfiguration
    .configureWithOAuthAuthentication(mediaServiceUri, oAuthUri, clientId, clientSecret, scope);

    // Create the MediaContract object using the specified configuration.
    mediaService = MediaService.create(configuration);
            
        }

    // Upload a media file to your Media Services account.
    // This code creates an asset, an access policy (using Write access) and a locator, 
    // and uses those objects to upload a local file to the asset.
    private static void Upload() throws ServiceException, FileNotFoundException, NoSuchAlgorithmException 
        {
            
        WritableBlobContainerContract uploader;
            
        AccessPolicyInfo uploadAccessPolicy;
        LocatorInfo uploadLocator = null;
            
    // Create an asset.
        assetToEncode = mediaService.create(Asset.create().setName("myAsset").setAlternateId("altId"));
    System.out.println("Created asset with id:" + assetToEncode.getId());

    // Create an access policy that provides Write access for 15 minutes.
    uploadAccessPolicy = mediaService.create(AccessPolicy.create("uploadAccessPolicy", 
                                                                         15.0, 
                                                                 EnumSet.of(AccessPolicyPermission.WRITE)));
    System.out.println("Created upload access policy with id: "
    + uploadAccessPolicy.getId());

    // Create a locator using the access policy and asset.
    // This will provide the location information needed to add files to the asset.
    uploadLocator = mediaService.create(Locator.create(uploadAccessPolicy.getId(),
            assetToEncode.getId(), LocatorType.SAS));
    System.out.println("Created upload locator with id:" + uploadLocator.getId());

    // Create the blob writer using the locator.
    uploader = mediaService.createBlobWriter(uploadLocator);

    // The name of the file as it will exist in your Media Services account.
    String fileName = "MPEG4-H264.mp4";  

    // The local file that will be uploaded to your Media Services account.
    InputStream input = new FileInputStream(new File("c:/media/" + fileName));

    // Upload the local file to the asset.
    uploader.createBlockBlob(fileName, input);

    // Inform Media Services about the uploaded files.
    mediaService.action(AssetFile.createFileInfos(assetToEncode.getId()));
    System.out.println("File uploaded.");
            
           
    System.out.println("Deleting upload locator and access policy.");
    mediaService.delete(Locator.delete(uploadLocator.getId()));
    mediaService.delete(AccessPolicy.delete(uploadAccessPolicy.getId()));
            
        }

    // Create a job that contains a task to transform the asset.
    // In this example, the asset will be transformed using the Azure Media Encoder.
    private static void Transform() throws ServiceException, InterruptedException 
        {
    // Use the Azure Media Encoder, by specifying it by name.
    // Retrieve the list of media processors that match this name.      
        ListResult<MediaProcessorInfo> mediaProcessors = mediaService
                .list(MediaProcessor.list()
                .set("$filter", "Name eq 'Azure Media Encoder'"));
            
        // Use the latest version of the media processor.
        MediaProcessorInfo mediaProcessor = null;
        for (MediaProcessorInfo info :mediaProcessors)
            {
            if (null == mediaProcessor || info.getVersion().compareTo(mediaProcessor.getVersion()) > 0)
                {
                mediaProcessor = info;
                }
            }

        System.out.println("Using processor:" + mediaProcessor.getName() +
                " " + mediaProcessor.getVersion());

    // Create a task with the specified media processor, in this case to transform the original asset to the H264 Broadband 720p preset.
    // Information on the various configurations can be found at
    // http://msdn.microsoft.com/en-us/library/windowsazure/jj129582.aspx.
    // This example uses only one task, but others could be added.
    Task.CreateBatchOperation task = Task.create(
    mediaProcessor.getId(),
    "<taskBody><inputAsset>JobInputAsset(0)</inputAsset><outputAsset>JobOutputAsset(0)</outputAsset></taskBody>")
    .setConfiguration("H264 Broadband 720p").setName("MyTask");

    // Create a job creator that specifies the asset, priority and task for the job. 
    Job.Creator jobCreator = Job.create()
    .setName("myJob")
    .addInputMediaAsset(assetToEncode.getId())
    .setPriority(2)
    .addTaskCreator(task);

    // Create the job within your Media Services account.
    // Creating the job automatically schedules and runs it.
    JobInfo jobInfo = mediaService.create(jobCreator);
    String jobId = jobInfo.getId();
    System.out.println("Created job with id:" + jobId);
    // Check to see if the job has completed.
    CheckJobStatus(jobId);
    // Done with the job.

    // Retrieve the output asset.
    ListResult<AssetInfo> outputAssets = mediaService.list(Asset.list(jobInfo.getOutputAssetsLink()));
    encodedAsset = outputAssets.get(0);
        }

    // Download the output assets of the transformed asset.
    private static void Download() throws ServiceException, URISyntaxException, FileNotFoundException, StorageException, IOException 
        {
            
        AccessPolicyInfo downloadAccessPolicy = null;

    downloadAccessPolicy =
    mediaService.create(AccessPolicy.create("Download", 15.0, EnumSet.of(AccessPolicyPermission.READ)));
    System.out.println("Created download access policy with id: "
    + downloadAccessPolicy.getId());
            
        LocatorInfo downloadLocator = null;
    downloadLocator = mediaService.create(
            Locator.create(downloadAccessPolicy.getId(), encodedAsset.getId(), LocatorType.SAS));
    System.out.println("Created download locator with id:" + downloadLocator.getId());        

    System.out.println("Accessing the output files of the encoded asset.");
    // Iterate through the files associated with the encoded asset.
    for(AssetFileInfo assetFile:mediaService.list(AssetFile.list(encodedAsset.getAssetFilesLink())))
            {
    String fileName = assetFile.getName();
                
    System.out.print("Downloading file:" + fileName + ". ");
    String locatorPath = downloadLocator.getPath();
    int startOfSas = locatorPath.indexOf("
        ");

    String blobPath = locatorPath + fileName;
    if (startOfSas >= 0) 
                {
    blobPath = locatorPath.substring(0, startOfSas) + "/" + fileName + locatorPath.substring(startOfSas);
                }
    URI baseuri = new URI(blobPath);
    CloudBlobClient blobClient;
    blobClient = new CloudBlobClient(baseuri);
                
    // Ensure that you have a c:\output folder, or modify the path specified in the following statement.
    String localFileName = "c:/output/" + fileName;
                
    CloudBlockBlob sasBlob;
    sasBlob = new CloudBlockBlob(baseuri, blobClient);
    File fileTarget = new File(localFileName);
                
    sasBlob.download(new FileOutputStream(fileTarget));
    System.out.println("Download complete.");
                
            }

    System.out.println("Deleting download locator and access policy.");
    mediaService.delete(Locator.delete(downloadLocator.getId()));
    mediaService.delete(AccessPolicy.delete(downloadAccessPolicy.getId()));
          
        }
        
    // Remove all assets from your Media Services account.
    // You could instead remove assets by name or ID, etc., but for 
    // simplicity this example removes all of them.
    private static void Cleanup() throws ServiceException 
        {
    // Retrieve a list of all assets.
    List<AssetInfo> assets = mediaService.list(Asset.list());

    // Iterate through the list, deleting each asset.
    for (AssetInfo asset:assets)
            {
        System.out.println("Deleting asset named " + asset.getName() + " (" + asset.getId() + ")");
    mediaService.delete(Asset.delete(asset.getId()));
            }
        }

    // Helper function to check to on the status of the job.
    private static void CheckJobStatus(String jobId) throws InterruptedException, ServiceException
        {
    int maxRetries = 12; // Number of times to retry.Small jobs often take 2 minutes.
    JobState jobState = null;
    while (maxRetries > 0) 
            {
    Thread.sleep(10000);  // Sleep for 10 seconds, or use another interval.
    // Determine the job state.
    jobState = mediaService.get(Job.get(jobId)).getState();
    System.out.println("Job state is " + jobState);

    if (jobState == JobState.Finished || 
    jobState == JobState.Canceled || 
    jobState == JobState.Error) 
                {
    // The job is done.
    break;
                }
    // The job is not done.Sleep and loop if max retries 
    // has not been reached.
    maxRetries--;
            }
      
        }

    }

创建的资产存储在 Azure 存储空间中。但是，请只使用 Azure Media Services API（而不是 Azure 存储 API）来添加、更新或删除资产。

### 确定哪些媒体处理器可用

上面的代码使用了一个媒体处理器，可根据特定的媒体处理器名称来访问该处理器（如果存在多个版本，则会使用最新版本）。若要确定哪些媒体处理器可用，可以使用以下代码。

    for (MediaProcessorInfo mediaProcessor:mediaService.list(MediaProcessor.list()))
    {
    System.out.print(mediaProcessor.getName() + ", ");
    System.out.print(mediaProcessor.getId() + ", ");  
    System.out.print(mediaProcessor.getVendor() + ", ");  
    System.out.println(mediaProcessor.getVersion());  
    }

或者，按以下代码所示根据名称检索媒体处理器的 ID。

    String mediaProcessorName = "Storage Decryption"; 
    EntityListOperation<MediaProcessorInfo> operation;
    MediaProcessorInfo processor;

    operation = MediaProcessor.list();
    operation.getQueryParameters().putSingle("$filter", "Name eq '" + mediaProcessorName + "'");
    processor = mediaService.list(operation).get(0); 
    System.out.println("Processor named " + mediaProcessorName + 
    " has ID of " + processor.getId());

### 取消作业

如果需要取消一个尚未完成处理的作业，可以按以下代码所示根据作业 ID 取消该作业。

    mediaService.action(Job.cancel(jobId));

其他资源其他资源
----------------

有关 Media Services Javadoc 文档，请参阅[适用于 Java 的 Azure 库文档](http://dl.windowsazure.com/javadoc/)。

