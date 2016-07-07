<properties linkid="dev-net-common-tasks-cdn" urlDisplayName="CDN" pageTitle="Create Download Acceleration-Type CDNs – Azure Feature Guide" metaKeywords="Azure CDN, Azure CDN, Azure blobs, Azure caching, Azure add-ons, CDN acceleration, CDN service, cloud acceleration, download acceleration, download, cache rules, ICP, ICP record number, ICP number, technical documentation, help files, bandwidth, large file download, software upgrade installation package, game download acceleration, app download acceleration, mobile app update, firmware upgrade" description="Learn how to create Download Acceleration-type CDNs on Azure Management Portal, and learn about default caching rules for Download CDNs." metaCanonical="" services="" documentationCenter=".NET" title="" authors="" solutions="" manager="" editor="" />
<tags ms.service="cdn_en"
    ms.date="7/7/2016"
    wacn.date="7/7/2016"
    wacn.lang="en"
    />
> [AZURE.LANGUAGE]
- [中文](/documentation/articles/cdn-how-to-create-Download-CDN-endpoint/)
- [English](/documentation/articles/cdn-enus-how-to-create-Download-CDN-endpoint/) 
#Create download acceleration CDN nodes

Download Acceleration is primarily intended for use with downloads of large files of 20MB or more, for example with download delivery for software installation packages, game clients, apps, or videos. The Azure CDN caches the files to the CDN edge nodes, relieving bandwidth pressure on source station downloads and improving the user’s download experience.

Download Acceleration is suitable for use in scenarios such as downloading operating system (OS) and firmware upgrades, game clients, mobile app updates, and computer applications.

This article is about creating domain names for download acceleration. You can also refer to [Using Azure CDN](/documentation/articles/cdn-enus-how-to-use/) to find out about the basics of creating Azure CDN acceleration nodes.

###**Default cache rules for download acceleration**
The Azure CDN sets default cache rules (see below) for Download Acceleration. You can also set custom cache rules according to your own requirements. For specific details, see the Azure CDN Management Portal advanced management help file on “Domain Management.” If the source station content changes or is updated, but the cache time to live (TTL) has not yet expired, you can manually refresh the CDN cache files to synchronize the updated source station content in real time. For specific details, see the Azure CDN Management Portal advanced management help file on “Cache Refresh.”

**The system’s default cache rules for Download Acceleration**

1. Dynamic files—such as those with the extensions PHP, ASPX, ASP, JSP and DO—are not cached.
2. Files with the extensions 7Z, APK, WDF, CAB, DHP, EXE, FLV, GZ, IPA, ISO, MPK, MPQ, PBCV, PXL, QNP, R00, RAR, XY, XY2, ZIP, and CAB are cached for one month.

###**Create domain names for download acceleration**

1. In the navigation pane of the Azure Management Portal, click “CDN.”
2. In the function area, click “Create New.” In the “Create New” dialogue box, select “App Services,” “CDN,” and “Quick Create” in that order.
3. Select “Download Acceleration” from the “Acceleration Type” drop-down list.
4. In the “Origin Domain Type” drop-down list, select cloud service, storage account, web app, or a customized origin domain. **Note** that Download Acceleration does not support the “media service” origin domain type.
5. In the “Origin Domain” drop-down list, select one option from the list of available cloud services, storage accounts, or web apps for use in creating the CDN endpoint. 

    ![012](./media/cdn-doc/download-en-001.png)

    If the selected “Origin Domain Type” is “Customized Origin Domain,” input your own origin domain address under “Origin Domain.” You can enter one or multiple origin domain IP addresses (separate multiple addresses with semicolons, e.g. “126.1.1.1;172.1.1.1”), or an origin domain name such as “origin.azurechina.com.”

    ![008](./media/cdn-doc/download-en-002.png)

6. In “Custom Domain,” enter the custom domain name you wish to use, e.g. cdn1.azurechinatest.com. Custom domains support extensive domain name acceleration.
7. In “Origin Host Header,” enter the return to source access host header accepted by your source station. Once you have entered the “Custom Domain,” the system will automatically fill in a default value based on the “Origin Domain Type” you selected. To be more specific, if your source station is on Azure, the default value will be the corresponding source station address. If your source station is not on Azure, the default value will be the “Custom Domain” that you entered. Of course, you can also modify this based on the actual configuration of your source station.

    If the origin domain type is a storage account, the corresponding return to source host header is:

    ![007](./media/cdn-doc/download-en-003.png)  
    
    If the origin domain type is a custom origin domain, the corresponding return to source host header is:

    ![013](./media/cdn-doc/download-en-004.png)
    
      
8. In “ICP Number,” enter the corresponding ICP record number for the custom domain that you entered (e.g., Jing ICP Bei XXXXXXXX Hao-X).
     
    ![009](./media/cdn-doc/download-en-005.png)

9. Click “Create” to create the new endpoint.

Once the endpoint has been created, it will appear in the list of subscribed endpoints. 
The list view shows the custom domains used to access cached content, as well as the origin domains. The origin domain is the original location of the content cached on the CDN. Custom domains are URLs used to access CDN cache content.


> **Note** that configurations created for endpoints cannot be used immediately; they must first pass checks to confirm that the ICP custom domain name matches the ICP number. For more details, see the second half of Step 2: Create new CDN endpoints in [Using Azure CDN](/documentation/articles/cdn-enus-how-to-use/).

<!---HONumber=CDN_1201_2015-->