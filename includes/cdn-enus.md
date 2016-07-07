> [AZURE.LANGUAGE]
- [中文](/documentation/articles/cdn-overview/)
- [English](/documentation/articles/cdn-overview/) 
# Use Azure Content Delivery Network

Azure Content Delivery Network caches static content in storage blobs, cloud services, and websites on the Azure platform by using large numbers of physical nodes that are distributed across mainland China to provide developers with a solution for delivering high-bandwidth content. This network also currently supports the use of origins that are not deployed on the Azure platform.

This task includes the steps listed below:

+ [Step 1: Create a storage account, cloud service, website, or media service](#step1)
+ [Step 2: Create a new CDN endpoint](#step2)
+ [Step 3: Access content on CDN](#step3)
+ [Step 4: Delete content from Content Delivery Network](#step4)
+ [Step 5: Use advanced management features](#step5)

The advantages of using Content Delivery Network to cache Azure data include:

- Better performance and user experience for end users who are far from a content source, and are using applications where many "internet trips" are required to load content
- Large distributed scale to better handle instantaneous high load, like at the start of a product launch event. 

Existing Azure customers in China can now use CDN in the [Azure CDN portal](https://manage.windowsazure.cn/).

## Step 1: Create a storage account, cloud service, website, or media service<a id="step1"></a>
You can create CDN endpoints for storage accounts, cloud services, websites, or media services in existing Azure subscriptions. You can also create new storage accounts, cloud services, or websites for use in Azure subscriptions by using the following method:

### Create a storage account for Azure subscriptions
Refer to [How to create storage accounts](/zh-cn/documentation/articles/storage-create-storage-account/)

### Create a cloud service for Azure subscriptions
Refer to [How to create and deploy cloud services](/zh-cn/documentation/articles/cloud-services-how-to-create-deploy/)

### Create a website for Azure subscriptions
Refer to [How to create and deploy websites](/zh-cn/documentation/articles/web-sites-create-deploy/)

### Create a media service for Azure subscriptions
Refer to [How to create and deploy media services](/documentation/articles/media-services-create-account/)

## Step 2: Create a new Content Delivery Network endpoint<a id="step2"></a>
Once a storage account, cloud services or websites is enabled, all publicly available objects are entitled to access the network edge high-speed caching. If you edit an object that is currently cached in CDN endpoint, the new content will be accessible only after the time to live (TTL) expires and the object’s content is updated (or manually refresh them using the advanced management features).

### Create a new Content Delivery Network endpoint
1. In the navigation pane of the [Azure portal](https://manage.windowsazure.cn/), click CDN.
2. In the Function area, click New. In the New dialog box, select App Services, CDN, and Quick Create.

    ![CDN Quick Create][1]
3. Select all Azure subscriptions you wish to use from the Subscriptions drop-down list (if there are multiple subscriptions).

    ![CDN Quick Create][2]
4. Select the acceleration type from the Acceleration Type drop-down list. The types of acceleration currently supported are Web Acceleration, Download Acceleration, HTTP VoD (Video on Demand) Acceleration, and Live Streaming (Video Direct Broadcast) Acceleration.
5. In the Origin Domain Type drop-down list, select Cloud Service, Storage Account, Web App, Media Service, or a custom origin domain.
6. In the Origin Domain drop-down list, select available cloud service, storage account, web app, or media service domain list. If the selected Origin Domain Type is Customized Origin Domain, input your own origin domain address here.
7. In Custom Domain, enter the custom domain name you wish to use, e.g., cdn.<yourcompany>.com.
8. In Origin Host Header, enter the return-to-source access host header that your origin accepted. Once you have entered the custom domain, the system will automatically fill in a default value based on the origin domain type that you selected. To be more specific, if your origin is on Azure, the default value will be the corresponding origin domain. If your origin is not on Azure, the default value will be the custom domain that you entered. You can also modify this based on the actual configuration of your origin.
9. In ICP Number, enter the corresponding **ICP record number** for the custom domain that you entered (e.g., 京ICP备XXXXXXXXHao-X).
10. Click Create to create the new endpoint.
11. Once the endpoint has been created, it will appear in the endpoint list for selected subid. The list view shows the custom domains that were used to access cached content, as well as the origin domains.

The origin domain is the source origin location of the content cached on Content Delivery Network. Custom domains are URLs that are used to access cache content.
> **Note** that configurations created for endpoints cannot be used immediately:

1. The ICP number must be valid and the custom domain name and ICP number must match. This process can take up to **one business day** to complete.
2. If the details do not pass the ICP verification, you must delete the Content Delivery Network endpoint you created and create a new endpoint by using the correct custom domain name and ICP number.
3. If the details pass the ICP verification, the service will be registered within **60 minutes** so that it can be propagated by the network. At the same time, you also need to configure the CNAME mapping details, as indicated by the notifications in UI, before the cache content can finally be accessed via the custom domain name.

## Step 3: Access content in Content Delivery Network<a id="step3"></a>
If you want to access content cached on the network, do so by using the custom domain name you provided in Step 2. The addresses of cached blobs are similar to the following address (using the example from Step 2):

`http://cdn.yourcompany.com/<myPublicContainer>/<BlobName>`

## Step 4: Delete content from the Content Delivery Network<a id="step4"></a>
If you don’t want to continue to cache objects on Content Delivery Network, you can use any one of the following procedures:

- For Azure blobs, delete the blob from the public container.
- Generate a private container to replace the public container. For more information on this, refer to [Restricting access to containers and blobs](http://msdn.microsoft.com/zh-cn/library/dd179354.aspx).
- You can use the Azure portal to ban or delete Content Delivery Network endpoints.
- You can change the cloud service to a request that no longer responds to this object.

Objects already cached in Content Delivery Network will remain in a cached state until the TTL for the object expires. Once the TTL expires, the network will check whether the endpoint is still valid and whether anonymous access to the object is still possible. If the object cannot be accessed, it will no longer be cached.


## Step 5: Use advanced management features<a id="step5"></a>
Once you have created a new Content Delivery Network endpoint, you can use the Azure portal to check the basic configuration details and perform other basic operations, such as disable/enable and delete network endpoints. You can also click the Management button to jump to another management page, where you can use the advanced management functions:

![Manage Button][3]
> **Note: You will be taken to another CDN management page that is not part of the Azure portal. (Ensure that you allow your browser to open the new window)**

![Adv Portal][4]

This management interface provides features including overview, Domain Management, Usage Chart, Bandwidth Chart, Cache Refresh, Content Prefetch, Log Download and Service Check. For more details, click to enter the corresponding function modules, and then read the corresponding help files. You can also open the help files directly via the left navigation bar.

<!--Image references-->
[1]: ./media/cdn/001.png
[2]: ./media/cdn/002.png
[3]: ./media/cdn/003.png
[4]: ./media/cdn/004.png

<!---HONumber=CDN_1201_2015-->