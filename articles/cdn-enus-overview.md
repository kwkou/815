<properties linkid="dev-net-common-tasks-cdn" urlDisplayName="CDN" pageTitle="Overview of Azure Content Delivery Network in China: Azure feature guide" metaKeywords="Azure CDN, Azure CDN, Azure blobs, Azure caching, Azure add-ons, CDN, CDN acceleration, CDN service, mainstream CDN, multi-scenario acceleration, free CDN, CDN website acceleration, website acceleration, webpage acceleration, static acceleration, download acceleration, VOD acceleration, streaming media webcast acceleration, cloud service,  storage account, cache refresh, return to origin, cloud acceleration, acceleration results, node, traffic, CNAME, bandwidth, network speed, anti-theft chain, https acceleration, low-cost bandwidth, access acceleration, small file acceleration, download acceleration, large file acceleration, streaming media acceleration, HTTPS secure acceleration, cache refresh, content pre-loading, anti-theft chain, log download, CDN technical documentation, CDN help files, CDN FAQs" description="Get an overview of Azure Content Delivery Network and its advantages, typical scenarios and key features." metaCanonical="" services="" documentationCenter=".NET" title="" authors="" solutions="" manager="" editor="" />
<tags ms.service="cdn_en"
    ms.date="7/7/2016"
    wacn.date="7/7/2016"
    wacn.lang="en"
    />
> [AZURE.LANGUAGE]
- [中文](/documentation/articles/cdn-overview/)
- [English](/documentation/articles/cdn-enus-overview/) 
# Overview of Azure Content Delivery Network

The Azure Content Delivery Network caches static content in storage blobs, cloud services, and websites on the Azure platform by using large numbers of physical nodes distributed across Mainland China. This provides media services with acceleration for streaming content delivery and offers developers solutions for delivering high-bandwidth content. This service also currently supports the use of origins that have not been deployed on the Azure platform.

For more details and pricing for Content Delivery Network, see [Introduction to the Azure Content Delivery Network service](/home/features/cdn/).

If you are an existing user of the CDN, visit the [Azure Content Delivery Network management portal](https://manage.windowsazure.cn) to manage your CDN domains. See [Using Azure Content Delivery Network](/documentation/articles/cdn-enus-how-to-use/) for a more specific user guide.

+ [What is a Content Delivery Network?](#step1)
+ [Advantages of Content Delivery Network](#step2)
+ [Features of Content Delivery Network](#step3)

## What is Content Delivery Network?<a id="step1"></a>

CDN stands for content delivery network. Virtually all large-scale websites use the CDN technology today, but the technology is by no means exclusive to large websites. The basic purpose of CDN is to avoid bottlenecks and other parts of the Internet that could affect data transfer rates and stability and make content transfers faster and more stable. By placing node servers in different places across the network and building a more intelligent layer of virtual networks above the internet infrastructure, CDN systems can redirect user requests in real time to the node closest to the user based on the status of network traffic and node connections and loads, as well as distance to the user and response times.
 
Taking the Microsoft website as an example, Microsoft may be an American company, but its customers are distributed across the globe. This means that there are users in every part of the world who need to access the Microsoft website or download product updates from Microsoft Update. If the website servers were deployed only in a single location in the US, users in the areas around this location could undoubtedly achieve satisfactory speeds when accessing the website, as they are close the server and network delays would also be minimal. However, a user in China who is downloading product updates from a server in this US location would require that all data packets were transferred back and forth between China and the US. As the total length of the route extends to thousands of kilometers, this would cause enormous delays.

However, this problem can be solved by placing CDN nodes in China. The nodes automatically cache data to data centers in the major cities throughout China, shortening the distance between users and content and thereby reducing the time required to transfer data. CDN caching makes it possible for users to obtain content from a nearby location, which resolves the issue of Internet network congestion and increases the response speed for users who are accessing websites and apps.


![][1]


## Advantages<a id="step2"></a>

### Built-in Azure services

Azure CDN supports a number of different Azure services, including storage blobs, cloud services, websites, and media services, providing the user with comprehensive, one-stop cloud service support.

![][2]


### Full self-service model

Traditional CDN services require a large amount of complex and tedious configuration processes. However, users can manage the life-cycle of CDN endpoints themselves from creating endpoints, check usage and bandwidth, download raw access logs, to configure various advanced functions (such as cache rule configuration, forced refreshing of cached content, content preload, and antitheft chains) by using the Azure portal and the dedicated CDN management portal. 

![][3]

![][4]

### Dynamic optimization of all network nodes

The CDN service integrates mainstream CDN providers in China to provide comprehensive acceleration types from static webpage acceleration, large files  download acceleration (such as software installation/update packages, game clients, apps and videos), to video on demand (VoD) and live-streaming acceleration for online video websites and online educational websites, thereby meeting the delivery needs of different types of resource. It also provides full network coverage that spans the entire region for mainstream telecoms carriers, including China Telecom, China Unicom, and China Mobile, as well as other ISPs, and allocates user requests to the optimal node by using load-balancing technology and intelligent dispatch strategies based on the real-time network status.


### Reduces costs

By cooperating with mainstream CDN providers in China, Azure CDN  can provide all Azure users, including business clients and web-direct users, with high-quality, low-cost CDN services.

## Acceleration for multiple scenarios

The Content Delivery Network service supports different acceleration scenarios described below, which creates a comprehensive and multidimensional acceleration service for users.

![][5]

### Website and small file acceleration

One classic scenario for CDN is the so-called “small files” (e.g., HTML webpage files, image files, JavaScript files, or CSS files) used by many websites. This allows the website to deliver a better user experience and thereby increase user visits and ultimately drive up revenues for the entire business. The typical user group would be small, medium, and large enterprises that provide website services for Internet users.

### Download delivery for large files

Another typical usage scenario for CDN is to facilitate multinode delivery of large file downloads, which ultimately ensures that the download experience proceeds smoothly. Without CDN, typical user scenarios such as operating system and firmware upgrades, publishing new games online (which require downloading client installation packages), and mobile app updates, would have a huge impact on source station bandwidth usage and could even cause the source server to stop working.

### Streaming media acceleration

As online video and media services has grown popular over the last few years, increasing numbers of people have become used to using Internet platforms to watch videos and listen to audio content. The limitations on the Internet environment in China places huge demands on the final delivery of audio and video content. The typical user group is all types of media website and mobile app clients that provide services to Internet users.

### HTTPS secure acceleration

Security will always be the issue that users are most concerned about. CDN not only provides comprehensive acceleration for HTTP protocol, but also offers a dedicated HTTPS acceleration service for scenarios that need to support HTTPS protocol. In terms of usage scenarios, this service could be categorized as a small file acceleration service, but it also provides certain optimization services that use dynamic routing optimization.


## Functions<a id="step3"></a>

### Full self-service creation and management of CDN endpoints

Azure CDN provides comprehensive, self-service management for the entire lifecycle of CDN endpoints, including creating, deleting, enabling, and disabling endpoints, as well as endpoints configurations (such as change origin domain, configure cache rules, cache refresh and content preload).

![][6]

### Visualized reports for traffic and bandwidth information

Azure CDN management portal allows easy, quick, and clear checking of traffic and bandwidth usage details for CDN domains.

![][7]

![][8]


### Cache rule configuration

The system already provides default cache rules for various acceleration types. Users can also customize and edit rules according to their actual requirements.

![][9]

### Antitheft chain

CDN provides content access control features to address the specific needs of users in China, including an antitheft chain. Users can use these features to take better and more effective control over their content, to achieve the goal of keeping content protected.

![][10]


### Cache refresh

Sometimes when users finish updating a particular file on the original server, they want to see the results of the update reflected on the CDN endpoints in real time. However, as the network has default or user-defined cache rules, the updated changes are not propagated to all network nodes in real time. It is at times like this that the “cache refresh” function can be used to force a cache refresh for individual files or batches of files. The goal is to purge the previously cached contents from all nodes, so that the next time users access one of these files they will obtain the updated file.

![][11]

### Content pre-fetch

Content pre-fetch means caching the content of a designated URL from the source station to CDＮ nodes　in advance, to eliminate the waiting time the first time that the user accesses the resource. Content pre-fetch is generally used in scenarios that involve the delivery of large files, where it can effectively improve the user access experience.

![][12]

### Log downloads

Users sometimes need to analyze CDN acceleration performance or raw access information. In such situations, they can obtain such raw access information by using the “log download” function. The user must provide an accessible Azure storage account to use this feature, as CDN will save the log access files for the corresponding customer(s) on this account.

![][13]

### Service checks

Once you have created a CDN endpoint, you can use the "Service Check" view to see whether it is possible to access the origin, whether propagation to all CDN nodes is complete, and whether CDN nodes are caching correctly.

![][14]

<!--Image references-->
[1]: ./media/cdn-overview/overview-en-001.png
[2]: ./media/cdn-overview/overview-en-002.png
[3]: ./media/cdn-overview/overview-en-003.png
[4]: ./media/cdn-overview/overview-en-004.png
[5]: ./media/cdn-overview/overview-en-005.png
[6]: ./media/cdn-overview/overview-en-006.png
[7]: ./media/cdn-overview/overview-en-007.png
[8]: ./media/cdn-overview/overview-en-008.png
[9]: ./media/cdn-overview/overview-en-009.png
[10]: ./media/cdn-overview/overview-en-010.png
[11]: ./media/cdn-overview/overview-en-011.png
[12]: ./media/cdn-overview/overview-en-012.png
[13]: ./media/cdn-overview/overview-en-013.png
[14]: ./media/cdn-overview/overview-en-014.png