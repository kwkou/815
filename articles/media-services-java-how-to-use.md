<properties 
	pageTitle="如何将 Azure 媒体服务与 Java 配合使用" 
	description="介绍如何使用 Azure 媒体服务执行编码、加密和流式处理资源等常见任务。" 
	services="media-services" 
	documentationCenter="java" 
	authors="rmcmurray" 
	manager="wpickett" 
	editor="jimbe"/>

<tags
	ms.service="media-services"
 	ms.date="04/18/2016"  
	wacn.date="06/27/2016"/>

#如何将媒体服务与 Java 结合使用

[AZURE.INCLUDE [media-services-selector-get-started](../includes/media-services-selector-get-started.md)]

##<a id="setup-account"></a>为媒体服务设置 Azure 帐户

若要设置你的媒体服务帐户，请使用 Azure 管理门户。请参阅主题[如何创建媒体服务帐户](/documentation/articles/media-services-create-account/)。在 Azure 管理门户中创建帐户后，便可以设置你的计算机以进行媒体服务开发。

##<a id="setup-dev"></a>完成设置以进行媒体服务开发

本部分介绍使用用于 Java 的媒体服务 SDK 进行媒体服务开发需要满足的一般性先决条件。

###先决条件

-   在新的或现有的 Azure 订阅中拥有一个媒体服务帐户。请参阅主题[如何创建媒体服务帐户](/documentation/articles/media-services-create-account/)。
-   适用于 Java 的 Azure 库，可以从 [Azure Java 开发人员中心][]安装。

##<a if="connect"></a>如何将媒体服务与 Java 结合使用

以下代码演示了如何创建一个资产、如何将媒体文件上载到该资产、如何使用任务运行某个作业以转换该资产，以及如何创建一个定位符用于流式传输你的视频。

在使用此代码之前，需要设置一个媒体服务帐户。有关设置帐户的信息，请参阅[如何创建媒体服务帐户](/documentation/articles/media-services-create-account/)。

将 `clientId` 和 `clientSecret` 变量替换为你的值。该代码还依赖于本地存储的文件。你需要提供自己的文件以供使用。
	
	import java.io.*;
	import java.security.NoSuchAlgorithmException;
	import java.util.EnumSet;
	
	import com.microsoft.windowsazure.Configuration;
	import com.microsoft.windowsazure.exception.ServiceException;
	import com.microsoft.windowsazure.services.media.MediaConfiguration;
	import com.microsoft.windowsazure.services.media.MediaContract;
	import com.microsoft.windowsazure.services.media.MediaService;
	import com.microsoft.windowsazure.services.media.WritableBlobContainerContract;
	import com.microsoft.windowsazure.services.media.models.AccessPolicy;
	import com.microsoft.windowsazure.services.media.models.AccessPolicyInfo;
	import com.microsoft.windowsazure.services.media.models.AccessPolicyPermission;
	import com.microsoft.windowsazure.services.media.models.Asset;
	import com.microsoft.windowsazure.services.media.models.AssetFile;
	import com.microsoft.windowsazure.services.media.models.AssetFileInfo;
	import com.microsoft.windowsazure.services.media.models.AssetInfo;
	import com.microsoft.windowsazure.services.media.models.Job;
	import com.microsoft.windowsazure.services.media.models.JobInfo;
	import com.microsoft.windowsazure.services.media.models.JobState;
	import com.microsoft.windowsazure.services.media.models.ListResult;
	import com.microsoft.windowsazure.services.media.models.Locator;
	import com.microsoft.windowsazure.services.media.models.LocatorInfo;
	import com.microsoft.windowsazure.services.media.models.LocatorType;
	import com.microsoft.windowsazure.services.media.models.MediaProcessor;
	import com.microsoft.windowsazure.services.media.models.MediaProcessorInfo;
	import com.microsoft.windowsazure.services.media.models.Task;
	
	
	public class HelloMediaServices
	{
	    // Media Services account credentials configuration
	    private static String mediaServiceUri = "https://media.windows.cn/API/";
	    private static String oAuthUri = "https://wamsprodglobal001acs.accesscontrol.chinacloudapi.cn/v2/OAuth2-13";
	    private static String clientId = "account name";
	    private static String clientSecret = "account key";
	    private static String scope = "urn:WindowsAzureMediaServices";
		private static MediaContract mediaService;
	    
	    // Encoder configuration
	    private static String preferedEncoder = "Media Encoder Standard";
	    private static String encodingPreset = "H264 Multiple Bitrate 720p";
		
		public static void main(String[] args)
		{
		
			try {
				// Set up the MediaContract object to call into the Media Services account
				Configuration configuration = MediaConfiguration.configureWithOAuthAuthentication(
				mediaServiceUri, oAuthUri, clientId, clientSecret, scope);
				mediaService = MediaService.create(configuration);
				
				
				// Upload a local file to an Asset
				AssetInfo uploadAsset = uploadFileAndCreateAsset("BigBuckBunny.mp4");
				System.out.println("Uploaded Asset Id: " + uploadAsset.getId());
				
				
				// Transform the Asset
				AssetInfo encodedAsset = encode(uploadAsset);
				System.out.println("Encoded Asset Id: " + encodedAsset.getId());
				
				// Create the Streaming Origin Locator
				String url = getStreamingOriginLocator(encodedAsset);
				
				System.out.println("Origin Locator URL: " + url);
				System.out.println("Sample completed!");
			
			} catch (ServiceException se) {
				System.out.println("ServiceException encountered.");
				System.out.println(se.toString());
			} catch (Exception e) {
				System.out.println("Exception encountered.");
				System.out.println(e.toString());
			}
		
		}
	
		private static AssetInfo uploadFileAndCreateAsset(String fileName)
			throws ServiceException, FileNotFoundException, NoSuchAlgorithmException {

			WritableBlobContainerContract uploader;
			AssetInfo resultAsset;
			AccessPolicyInfo uploadAccessPolicy;
			LocatorInfo uploadLocator = null;
			
			// Create an Asset
			resultAsset = mediaService.create(Asset.create().setName(fileName).setAlternateId("altId"));
			System.out.println("Created Asset " + fileName);
			
			// Create an AccessPolicy that provides Write access for 15 minutes
			uploadAccessPolicy = mediaService
				.create(AccessPolicy.create("uploadAccessPolicy", 15.0, EnumSet.of(AccessPolicyPermission.WRITE)));
			
			// Create a Locator using the AccessPolicy and Asset
			uploadLocator = mediaService
				.create(Locator.create(uploadAccessPolicy.getId(), resultAsset.getId(), LocatorType.SAS));
			
			// Create the Blob Writer using the Locator
			uploader = mediaService.createBlobWriter(uploadLocator);
			
			File file = new File("BigBuckBunny.mp4"); 
			
			// The local file that will be uploaded to your Media Services account
			InputStream input = new FileInputStream(file);
			
			System.out.println("Uploading " + fileName);
			
			// Upload the local file to the asset
			uploader.createBlockBlob(fileName, input);
			
			// Inform Media Services about the uploaded files
			mediaService.action(AssetFile.createFileInfos(resultAsset.getId()));
			System.out.println("Uploaded Asset File " + fileName);
			
			mediaService.delete(Locator.delete(uploadLocator.getId()));
			mediaService.delete(AccessPolicy.delete(uploadAccessPolicy.getId()));
			
			return resultAsset;
		}
	
		// Create a Job that contains a Task to transform the Asset
		private static AssetInfo encode(AssetInfo assetToEncode)
			throws ServiceException, InterruptedException {
	
			// Retrieve the list of Media Processors that match the name
			ListResult<MediaProcessorInfo> mediaProcessors = mediaService
			                .list(MediaProcessor.list().set("$filter", String.format("Name eq '%s'", preferedEncoder)));
	
	        // Use the latest version of the Media Processor
	        MediaProcessorInfo mediaProcessor = null;
	        for (MediaProcessorInfo info : mediaProcessors) {
	            if (null == mediaProcessor || info.getVersion().compareTo(mediaProcessor.getVersion()) > 0) {
	                mediaProcessor = info;
	            }
	        }
	
	        System.out.println("Using Media Processor: " + mediaProcessor.getName() + " " + mediaProcessor.getVersion());
	
	        // Create a task with the specified Media Processor
	        String outputAssetName = String.format("%s as %s", assetToEncode.getName(), encodingPreset);
	        String taskXml = "<taskBody><inputAsset>JobInputAsset(0)</inputAsset>"
	                + "<outputAsset assetCreationOptions="0"" // AssetCreationOptions.None
	                + " assetName="" + outputAssetName + "">JobOutputAsset(0)</outputAsset></taskBody>";
	
	        Task.CreateBatchOperation task = Task.create(mediaProcessor.getId(), taskXml)
	                .setConfiguration(encodingPreset).setName("Encoding");
	
	        // Create the Job; this automatically schedules and runs it.
	        Job.Creator jobCreator = Job.create()
	                .setName(String.format("Encoding %s to %s", assetToEncode.getName(), encodingPreset))
	                .addInputMediaAsset(assetToEncode.getId()).setPriority(2).addTaskCreator(task);
	        JobInfo job = mediaService.create(jobCreator);
	        
	        String jobId = job.getId();
	        System.out.println("Created Job with Id: " + jobId);
	
	        // Check to see if the Job has completed
	        checkJobStatus(jobId);
	        // Done with the Job
	
	        // Retrieve the output Asset
	        ListResult<AssetInfo> outputAssets = mediaService.list(Asset.list(job.getOutputAssetsLink()));
	        return outputAssets.get(0);
	    }
	    
	
	    public static String getStreamingOriginLocator(AssetInfo asset) throws ServiceException {
	        // Get the .ISM AssetFile
	        ListResult<AssetFileInfo> assetFiles = mediaService.list(AssetFile.list(asset.getAssetFilesLink()));
	        AssetFileInfo streamingAssetFile = null;
	        for (AssetFileInfo file : assetFiles) {
	            if (file.getName().toLowerCase().endsWith(".ism")) {
	                streamingAssetFile = file;
	                break;
	            }
	        }
	
	        AccessPolicyInfo originAccessPolicy;
	        LocatorInfo originLocator = null;
	
	        // Create a 30-day readonly AccessPolicy
	        double durationInMinutes = 60 * 24 * 30;
	        originAccessPolicy = mediaService.create(
	                AccessPolicy.create("Streaming policy", durationInMinutes, EnumSet.of(AccessPolicyPermission.READ)));
	
	        // Create a Locator using the AccessPolicy and Asset
	        originLocator = mediaService
	                .create(Locator.create(originAccessPolicy.getId(), asset.getId(), LocatorType.OnDemandOrigin));
	
	        // Create a Smooth Streaming base URL
	        return originLocator.getPath() + streamingAssetFile.getName() + "/manifest";
	    }
	
	    private static void checkJobStatus(String jobId) throws InterruptedException, ServiceException {
	        boolean done = false;
	        JobState jobState = null;
	        while (!done) {
	            // Sleep for 5 seconds
	            Thread.sleep(5000);
	            
	            // Query the updated Job state
	            jobState = mediaService.get(Job.get(jobId)).getState();
	            System.out.println("Job state: " + jobState);
	
	            if (jobState == JobState.Finished || jobState == JobState.Canceled || jobState == JobState.Error) {
	                done = true;
	            }
	        }
	    }
	
	}




##其他资源

有关媒体服务 Javadoc 文档，请参阅[适用于 Java 的 Azure 库文档][]。

<!-- URLs. -->

  [Azure Java 开发人员中心]: /develop/java/
  [适用于 Java 的 Azure 库文档]: http://azure.github.io/azure-sdk-for-java/
  [Media Services Client Development]: /documentation/articles/media-services-develop-video-players/

 
<!---HONumber=Mooncake_0620_2016-->