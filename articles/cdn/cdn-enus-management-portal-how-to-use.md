<properties linkid="dev-net-common-tasks-cdn" urlDisplayName="CDN" pageTitle="How to use the Azure Content Delivery Network portal advanced features - Azure feature guide" metaKeywords="Azure CDN, Azure CDN, Azure blobs, Azure caching, Azure add-ons, cache refresh, content prefetch, log download, cache rules, CDN help files, CDN technical documentation, CDN" description="Learn how to use the advanced features of the Azure portal to manage Content Delivery Network endpoints" metaCanonical="" services="" documentationCenter=".NET" title="" authors="" solutions="" manager="" editor="" />
<tags ms.service="cdn_en"
    ms.date="7/7/2016"
    wacn.date="7/7/2016"
    wacn.lang="en"
    />
> [AZURE.LANGUAGE]
- [中文](/documentation/articles/cdn-management-portal-how-to-use/)
- [English](/documentation/articles/cdn-enus-management-portal-how-to-use/) 
# Azure Content Delivery Network portal user guide


The Azure CDN caches static content in storage blobs, cloud services, and websites on the Azure platform by using large numbers of physical nodes distributed across Mainland China. These nodes provide developers with a solution for delivering high-bandwidth content. This service also currently supports the use of source stations that have not been deployed on the Azure platform.

For more details and pricing for Content Delivery Network, see [Introduction to the Azure Content Delivery Network service](/home/features/cdn/).

+ [Overview](#step1)
+ [Domain name management](#step2)
+ [Traffic reports](#step3)
+ [Bandwidth reports](#step4)
+ [Cache refresh](#step5)
+ [Content prefetch](#step6)
+ [Log download](#step7)
+ [Service check](#step8)

## **Overview of the Azure CDN management portal**<a id="step1"></a>

In Advance management page, you will see overview of then endpoints you created under selected subscription ID. Parameters: Total Domain, Total Enabled Domain, Total Usage in Current Month and Bandwidths in Current Months.

![][1]
![][2]

### **Total Domain**

Total domains that were already created under the current Azure subscriptions.

### **Total Enabled Domains**

Statistics on accelerated domain names that are currently enabled under the current Azure subscriptions.

### **Total usage for the current month**

Total traffic in gigabytes (GB) that were used by all enabled domain names under the current Azure subscriptions during the current month.

### **Usages in current month**

Details of traffic in GB for each day in the current month under the current Azure subscriptions. If you need more detailed traffic information, you can click “Usage Chart” in the left navigation pane.


### **Bandwidth in current month**
Details of the peak bandwidth in megabytes per second (Mb/s) for each day in the current month. If you need more detailed bandwidth information, click “Bandwidth Chart” in the left navigation pane.

## **Domain name management**<a id="step2"></a>

Clicking “Domain Management” in the left navigation pane displays a list view of all domains that were created under the current Azure subscriptions. You can select different subscriptions from the subscriptions drop-down list to see details of acceleration domain names under different subscriptions. You can perform operations like Modify Configurations, Configure Caching Rules and Access Control Management for selected domain.

### **Accelerated domain list view**

![][3]

#### **The domain list view includes:**

-   Accelerated Domain, which are used to access CDN cached content. These domains must be accompanied by the corresponding ICP record details.
-   CDN Domain, which are provided by the CDN platform and all end with **.mschcdn.com**.
-   Original Domain, which are the origin location for content cached on the CDN nodes.
-   Acceleration type, which includes the acceleration types that are currently supported: “website acceleration”, “download acceleration”, “HTTP VoD acceleration”, and “live streaming acceleration”.
-   Status, which may be either enabled or disabled (including verifying ICP, requires CNAME configuration, disabled).

>**Note** that you need to CNAME your accelerated domain to CDN domain ending with **.mschcdn.com** provided by Azure CDN after the CDN service takes effect (within 60 minutes). 

>**Note** that only domain names with an **enabled** status can use the CDN service.

#### **Modify Configurations**

You can modify Original domain and host header here.
 
![][4]

#### **Cache rule configuration**

Azure CDN has default caching rules. You can also configure caching rules, which will be applied first. If customized rules don't match, it will apply default caching rules. 

![][5]

You can click “Cache rule configuration” and then set cache rules for selected domain, including:

- Folder-based configuration.

	Folders must begin with “/”, for example, /pic, /doc, or /htdoc/data. The back end matches all files within the designated folders, **including subfolders**.

- File extension-based configuration.

	The back end matches common file extensions, such as .jpg, .png, .gif, .txt, .m4v, or .mp3 **within all folders**.

- Full path-based configuration

	These are used to specify **a single file**, and must start with a forward slash (/). For example, "/sites/doc/example.doc". **Note**: If the path entered by the user is “/”, it will match the homepage.

>**Note** that the character strings entered by the user when configuring rules must not include special characters such as “{”, “}”, “(”, “)”, “[”, “]”, “.”, “?”, “*”, “\\”, “^”, or “$”.

>**Note** that entering a time of “0” means that caching is prohibited.

#### **Cache rules order**

The system match cache rules on the basis of the order of configuration, with the rule configured first given the highest priority level. Once a rule is matched, subsequent rules are **no longer** matched.

#### **Pre-defined templates**

Users can use “Apply pre-defined template” to quickly create cache configuration rules. 

#### **As forbidden**

If the user checks the “As forbidden” option, the accelerated domain name is not cached.

#### **Access control management**

You can use access control management to set up and configure referer blacklists and whitelists, and thereby implement an anti-theft chain.

![][6]

Once the anti theft chain is enabled, you can edit the external link rules. Each rule is made up of a path and a filename, for example, / *.png means all png files in the root directory.

Each rule is made up of a path and a filename, for example, / *.png means all png files in the root directory.

- If you set up a blacklist, access is denied if the referer is in the blacklist, but is otherwise permitted.
- If you set up a whitelist, access is permitted only if the referer is one of the domain names in the whitelist.

Once you have clicked “Submit” and waited for the operation to finish, the interface shows whether the operation was successful. If you click “Submit and close”, then the dialog box closes immediately. The next time you open the dialog box, you see the status of the last operation.

## **Usage Chart**<a id="step3"></a>

Select the subscription (choose one), time range, and accelerated domain(individual domain names or all of them) you wish to check, then click **Refresh**. The interface displays usage chart that meet the conditions you specified. Refer to the explanations on the actual interface page for data format details and implications.

![][7]

**Usage chart has the following functions:**

-   If you have multiple Azure subscriptions, you can check usage chart for individual subscriptions, as well as return-to-source usage.
-   If you have Azure subscriptions with multiple accelerated domains, you can view individual accelerated domain name traffic (and return-to-source traffic) and summaries of all accelerated domains traffic (and return-to-source traffic).
-   Granularity for usage is up to hour.
-   There are multiple time range types to check usage, including the last 24 hours, yesterday, today, the last 7 days, the last 15 days, the last 30 days, or the last month. You can also check traffic and return-to-source traffic for a specific time period, which cannot exceed 90 days.

## **Bandwidth chart**<a id="step4"></a>

Select the subscription (choose one), time range, and accelerated domains (individual domain names or all of them) you wish to check, then click **Refresh**. The interface displays bandwidth charts that meet the conditions you specified. Refer to the explanations on the actual interface page for data format details and implications.

![][8]

**Bandwidth chart has the following functions:**

-   If you have multiple Azure subscriptions, you can check bandwidth chart for individual subscriptions, as well as return-to-source bandwidth.
-   If you have Azure subscriptions with multiple accelerated domain names, you can view individual accelerated domain bandwidth (and return-to-source traffic) and summaries of all accelerated domain bandwidth (and return-to-source traffic) information.
-   Granularity for usage is up to hour.
-   There are multiple time range types to check bandwidth, including the last 24 hours, yesterday, today, the last 7 days, the last 15 days, the last 30 days, or the last month. You can also check bandwidth and return-to-source bandwidt for a specific time period, which cannot exceed 90 days.
-   Bandwidth chart displays the time at which peak bandwidth levels occur.


## **Cache refresh**<a id="step5"></a>

Clicking “Cache Refresh” in the bottom-left navigation pane performs a manual refresh of the specified files or folders.

### **Cache refresh list view**

Select the subscription (choose one), status, time range, and accelerated domains (choose one) you wish to check. The interface displays cache refresh records that meet the conditions you specified. Refer to the explanations on the actual interface page for data format details and implications.

![][9]

#### **Cache refresh list view includes:**

-   Accelerated domain names, which are URLs that are used to access CDN cache content.
-   Status (common statuses: successful, failed, or refreshing)
-   Submit time
-   Check time

If you successfully submitted the cache refresh rules, the status column shows the word “Successful”. If you did not successfully submit them, you must check that the cache refresh rules were correct. If there are any problems, you must re-create and re-submit the cache refresh rules.

### **Configure cache refresh rules**

You can configure cache refresh rules for individual files or for all files within a folder.

#### **Submit File Refresh**

![][10]

This task includes the steps listed below:

1. Click “Submit File Refresh” to enter the file refresh view.
2. In the “File Refreshing” dialog box, select the domain you wish to configure, and then enter the corresponding file path.
3. Click “+” to add a new rule or “x” to delete the corresponding file.
4. Click “Submit”.

The newly created file cache rules then appear in the cache refresh list view.

>**Note** that until the accelerated domain has passed the ICP verification process, it is not possible to perform a file refresh operation and the accelerated domain name drop-down list appears empty. Wait until it has passed ICP verification.

#### **Submit folder refresh**

![][11]

This task includes the steps listed below:

1. click “Submit Folder Refresh” to enter the folder refreshing view.
2. In the “Folder Refreshing” dialog box, select the domain you wish to configure, and then enter the corresponding folder path.
3. Click “+” to add a new rule or “x” to delete the corresponding directory.
4. Click “Submit”.

The newly created folder cache rules then appear in the cache refresh list view.

## **Content prefetch**<a id="step6"></a>

Click “Content Prefetch” in the left navigation pane to perform content prefetch  operations and check prefetch status for specific files.

Content prefetch means caching the content of a designated URL from the source station to the CDN nodes. This eliminates the waiting time the first time that you access the resource. Content prefetching is generally used in scenarios that involve the delivery of large files, where it can effectively improve the user access experience.

### **Content prefetch list view**

Select the subscription (choose one), status, time range, and accelerated domain (choose one) you wish to check. The interface displays precaching records that meet the conditions you specified. Refer to the explanations on the actual interface page for data format details and implications.

![][12]

#### **The content prefetch list view includes:**

-   Accelerated domain, which are URLs that are used to access CDN cache content.
-   Status (common statuses: successful, failed, or in progress)
-   Submit time


If you successfully submitted the precache rules, the status column shows the word “Successful”. If you did not successfully submit them, you must check that the precache rules were correct. If there are any problems, you must re-create and re-submit the precache rules.

### **Submit Precache**

You can configure precaching for individual files or multiple files.

![][13]

This task includes the steps listed below:

1. Click on “Submit Prefetch”.
2. In the “Content Prefetch” dialog box, select the domain you wish to configure, and then enter the corresponding file path.
3. Click “+” to add a new rule or “x” to delete the corresponding file.
4. Click “Submit”.

The newly created prefetch rules then appear in the Content Prefetch list view.

## **Log download**<a id="step7"></a>
Click “Log Download” in the left navigation pane to set CDN log download parameters for specific domains.

### **Log download view**

To download logs, you must first provide an Azure storage account that is used to save CDN logs. Click “Download Settings” to set this up.

![][14]

### **Download settings view**

You can set the storage account and the domain names for which log downloads are required in the “Download Settings” view. Once the setup is complete, the system automatically saves the logs that it finds to the designated storage account. You can delete the storage account to cancel log downloads.

![][15]

### **Log format details**
Logs are saved in blob format with a container called "cdn-access-logs". Every blob consists of a .csv file that is compressed with GZip. The meaning of each column within the log is given below:

- c-ip: Client IP address
- timestamp: Access time
- cs-method: HTTP request actions, such as GET/HEAD
- cs-uri-stem: Requested URI 
- http-ver: HTTP protocol version number
- sc-status: HTTP status code 
- sc-bytes: Number of bytes sent to the client by the server
- c-referer: Client-side referer URI
- c-user-agent: Client user agent identification
- rs-duration (ms): Time taken to complete the request (in milliseconds)
- hit-miss: Content Delivery Network cache hit and miss identification
- s-ip: IP address of the Content Delivery Network edge node that generated the log

>**Note** that if the CDN log does not contain content for a particular column, the corresponding record, for example the “c-referer” record, will be marked “-”. Also, depending on the edge node log configuration, the “rs-duration”, “hit-miss” and “s-ip” columns may also be empty.

>**Note** that once a website has been accelerated by CDN, the majority of its access records come from CDN edge nodes. When the network returns to the source, it enters the originating IP address in HTTP Header X-Forwarded-For, and the source station’s web server can edit the log to configure this information. If you need to find out the originating IP address of the client, you can refer to the information below.

>If we take NGINX as an example, you can add the following information to the configuration file:

>log\_format logCDN '$remote\_addr forwarded for $http\_x\_forwarded\_for - $remote\_user [$time\_local] ' '"$request" $status $body\_bytes\_sent ' '"$http\_referer" "$http\_user\_agent"';

>access\_log /var/log/nginx/access.log logCDN;

## **Service check**<a id="step8"></a>

Once you have created a CDN endpoint, you can perform some basic checks in the “Service Check” view. We strongly recommend that you perform service checks before you carry out CNAME operations.

![][16]

As shown in the view above, you must select the domain name to be checked, provide a resource that the source station can access, and then click “Check”.

he original domain is OK (HTTP status code is 200) - means the domain works
2. CDN service is deployed - means the CDN service is deployed
3. The CDN cache is OK - means the contents accessed from both origin and CDN endpoints are the same (by HTTP host header：HTTP Status Code，Last Modified Time，Content Length).

>**Note** that using the service check function does not guarantee that there are no anomalies in any of the CDN edge servers where the domain is located.

 

[1]: ./media/cdn-en-unified-portal/007.png
[2]: ./media/cdn-en-unified-portal/008.png
[3]: ./media/cdn-en-unified-portal/009.png
[4]: ./media/cdn-en-unified-portal/010.png
[5]: ./media/cdn-en-unified-portal/011.png
[6]: ./media/cdn-en-unified-portal/012.png
[7]: ./media/cdn-en-unified-portal/013.png
[8]: ./media/cdn-en-unified-portal/014.png
[9]: ./media/cdn-en-unified-portal/015.png
[10]: ./media/cdn-en-unified-portal/016.png
[11]: ./media/cdn-en-unified-portal/017.png
[12]: ./media/cdn-en-unified-portal/018.png
[13]: ./media/cdn-en-unified-portal/019.png
[14]: ./media/cdn-en-unified-portal/020.png
[15]: ./media/cdn-en-unified-portal/021.png
[16]: ./media/cdn-en-unified-portal/service-check.png