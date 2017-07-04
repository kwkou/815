<properties
    linkid="dev-net-common-tasks-cdn"
    urlDisplayName="CDN"
    pageTitle="User Guide for the new version of the Azure CDN Management Portal"
    metaKeywords="Azure CDN, Azure CDN, Azure blobs, Azure caching, Azure add-ons, Cache refresh, content prefetching, log download, cache rules, CDN help files, CDN technical files, CDN"
    description="Learn how to use advanced features of the Azure CDN Management Portal to manage CDN endpoints"
    metaCanonical=""
    services="cdn"
    documentationCenter=".NET"
    authors="v-jijes"
    solutions=""
    manager=""
    editor="" />
<tags
    ms.service="cdn_en"
    ms.author="v-jijes"
    ms.topic="article"
    ms.date="4/20/2017"
    wacn.date="4/20/2017"
    wacn.lang="en" />

> [AZURE.LANGUAGE]
- [中文](/documentation/articles/cdn-management-v2-portal-how-to-use/)
- [English](/documentation/articles/cdn-enus-management-v2-portal-how-to-use/)

# <a name="azure-cdn"></a>User Guide for the new version of the Azure CDN Management Portal


The Azure China CDN Management Portal has been redesigned so that function modules are categorized, and a number of new management functions have also been added. At the moment, clicking on the “Manage” button in the [Azure Portal Preview](https://portal.azure.cn/) will take you to the old Management Portal, but you can click on “Access the New Site” in the Overview interface to jump to the new version of the Management Portal.

![][16]

## <a name=""></a>**The functions of the new version of the Management Portal are shown below:**

+ [Overview](#step1)
+ [Domain name management](#step2)
+ [Monitoring and Analysis](#step3)
+ [Content Management](#step4)
+ [Self-service Tools](#step5)
+ [Security Management](#step6)

## **Overview of the Azure Content Delivery Network management page**<a id="step1"></a>

From the Overview page, you can view the basic details for your CDN subscription account, as well as key data such as the total number of accelerated domain names, enabled domain names, total traffic for the current month, and peak bandwidth for the current day. You can also view the traffic trend graph and bandwidth trend graph for the current month.

![][1]

 **Number of domain names**

Statistics on accelerated domain names that were already created under the current Azure subscriptions.

 **Accelerated domain names that are already enabled**

Statistics on accelerated domain names that are currently enabled under the current Azure subscriptions.

 **Total traffic for the current month**

Total traffic in megabytes (MB) that were used by all accelerated domain names under the current Azure subscriptions during the current month.

 **Traffic for the current month**

Details of traffic for each day in the current month under the current Azure subscriptions. If you need more detailed traffic information, you can click “Traffic Reports” in the left navigation pane to go to the traffic statistics report page.

**Monthly bandwidth** 
Details of the peak bandwidth in kilobits per second (Kb/s) for each day in the current month. If you need more detailed bandwidth information, click “Bandwidth Reports” in the left navigation pane to go to the bandwidth statistics report page.

## **Domain name management**<a id="step2"></a>

Clicking “Domain Name Management” in the left navigation pane displays a list view of all Content Delivery Network acceleration domain names that were created under the current Azure subscriptions. You can search for the domain name you want to manage from the search box. You can also select different subscriptions from the Azure subscriptions drop-down list to see details of CDN acceleration domain names under different subscriptions. You can also click on the corresponding domain name line to perform operations such as **“Edit Configuration”**, **“Cache Rule Configuration,”** or **“Access Control Management”** for the selected domain names.

### <a name=""></a>**Accelerated domain name list view**

![][3]

 **The domain name list view includes:**

-   Custom domain names, which are used to access Content Delivery Network cached content. These domain names must be accompanied by the corresponding ICP record details.
-   Source station addresses, which are the origin domain for content cached on the Content Delivery Network.
-   Status, which may be either enabled or disabled (including ICP approval, requires CNAME configuration, disabled, and other **non-enabled** statuses).
-   ICP record information corresponding to domain names.
-   Acceleration type, which includes the acceleration types that are currently supported: “website acceleration,” “download acceleration,” “HTTP VoD acceleration,” “live-streaming media acceleration,” “HTTPS acceleration,” and “image processing” acceleration).
-   The Azure CDN platform will provide CNAMEs, all of which have **.mschcdn.com** at the end, and the domain name provider must map its custom domain name to this CNAME.
-   CNAME status may be “Active” or “Inactive.”

>**Note:**
>You need to configure the CNAME mapping details for your accelerated domain name after the Content Delivery Network service takes effect (within 60 minutes). You can then map it to the network domain name provided by Microsoft.

>**Note:**
>Only domain names with an **enabled** status can use the network service normally.

### <a name=""></a>**Properties View**
When you select a domain name line, a view of the operations you can perform to change the domain names will appear on the right side. From the properties view, you can edit the source station domain name or host header.

![][17]

>**Note:**
>Value of the host string in the HTTP request header if the host header indicates the return-to-source CDN. This value is generally a character string in the form of a domain name. The source station uses this domain name to identify whether it is the same as the domain name that was configured on the source station.  **Cache rule configuration view**

![][4]

When you select a domain name line, a view of the operations you can perform to change the domain names will appear on the right side. The system sets default rules based on the cache rules. You can adjust these settings to your own requirements. User rules are given priority in matching. If there are no hits for user rules, the system default cache rules are implemented item by item.

>**Notes**

>- No-cache rules are prioritized
>- The cache rules are implemented from top to bottom

### <a name=""></a>**Cache rule configuration**

You can click “Cache rule configuration” and then set the required cache rules for domain names, including:

- Directory-based configuration.

    Directories must begin with “/”, for example, /pic, /doc, or /htdoc/data. The back-end matches all files within the designated directory, **including subdirectories**.

- File extension-based configuration.

    The back end matches common file extensions, such as .jpg, .png, .gif, .txt, .m4v, or .mp3 The back-end will match the specified file suffixes **within all folders**.

- Full path-based configuration

    These are used to specify **a single file**, and must start with a forward slash (/). For example, “/sites/doc/example.doc.”

>**Notes**

>- If the path entered by the user is “/” it will match the homepage.
>- The character strings entered by the user when configuring rules must not include any of the following special characters: { } ( ) [ ] . ? * \ ^ or $.
>- Entering a time of “0” means that caching is prohibited.

**Cache configuration order**

The system **matches each item consecutively** on the basis of the order of configuration, with the rule configured first given the highest priority level. Once a rule is matched, subsequent rules are **no longer** matched.

 **Custom templates**

Users can use “Apply a custom template” to quickly create cache configuration rules. The diagram above shows a rule created after the user goes into “Apply a custom template” and selects “Common files.” Users can edit the automatically created rules as required.

 **Prohibiting cache setup**

If the user checks the “Set to caching prohibited” option, the accelerated domain name is not cached.

### <a name=""></a>**Access control management**

You can use access control management to set up and configure referer blacklists and whitelists, and thereby implement an anti-theft chain.

![][5]

Once the anti theft chain is enabled, you can edit the external link rules. Each rule is made up of a path and a filename, For example, /*.png means all png files in the root directory.

Each rule is made up of a path and a filename, For example, /*.png means all png files in the root directory.

- If you set up a blacklist, access is denied if the referer is in the blacklist, but is otherwise permitted.
- If you set up a whitelist, access is permitted only if the referer is one of the domain names in the whitelist.

Once you have clicked “Submit” and waited for the operation to finish, the interface shows whether the operation was successful. If you click “Submit and close”, then the dialog box closes immediately. The next time you open the dialog box, you see the status of the last operation.

## **Monitoring and Analysis**<a id="step3"></a>

### <a name=""></a>**Traffic bandwidth**

Select the custom accelerated domain name (individual domain names or all of them) and time range you want to view, then click **Refresh**. The interface displays traffic statistics reports and bandwidth statistics reports that meet the conditions you specified. Refer to the explanations on the actual interface page for data format details and implications.

![][6]

**Traffic statistics reports have the following functions:**

-   If you have multiple Azure subscriptions, you can check traffic statistics for individual subscriptions, as well as return-to-source traffic statistics.
-   If you have Azure subscriptions with multiple accelerated domain names, you can view individual accelerated domain name traffic (and return-to-source traffic) statistics results and summaries of all accelerated domain name traffic (and return-to-source traffic).
-   The highest level of precision for queries is accurate to the minute.
-   Traffic statistics provide multiple time range types for queries. You can query traffic statistics for the last 24 hours, the last 7 days, and the previous month. You can also check traffic and return-to-source traffic for a specific time period, which cannot exceed 60 days.

**Bandwidth statistics reports have the following functions:**

-   If you have multiple Azure subscriptions, you can check bandwidth statistics for individual subscriptions, as well as return-to-source bandwidth statistics.
-   If you have Azure subscriptions with multiple accelerated domain names, you can view individual accelerated domain name bandwidth (and return-to-source traffic) statistics results and summaries of all accelerated domain name bandwidth (and return-to-source traffic) information.
-   The highest level of precision for queries is accurate to the minute.
-   Bandwidth statistics display the time at which peak bandwidth levels occur.
-   Bandwidth statistics provide flexible time range options for queries. You can query bandwidth statistics for the last 24 hours and the previous month. You can also check bandwidth statistics for a specific time period, which cannot exceed 60 days.

## **Content Management**<a id="step4"></a>

### <a name=""></a>**Cache refresh**

Clicking “Cache Refresh” in the bottom-left navigation pane performs a manual refresh of the specified files or directories.

 **Cache refresh list view**

In the “Query Previous Data” view on the right side of the cache refresh list view, select the domain name, time range, and status that you want to check, then click **Refresh**. The interface will display cache refresh records that match the conditions you entered. 
![][8]

 **Cache refresh list view includes:**

-   Custom domain names, which are URLs that are used to access Content Delivery Network cache content
-   Status (common statuses: successful, failed, or refreshing)
-   Submission time

If you successfully submitted the cache refresh rules, the status bar shows the word “Successful.” If you did not successfully submit them, you must check that the cache refresh rules were correct. If there are any problems, you must re-create and re-submit the cache refresh rules.

 **Submit cache refresh**

Press the “Submit cache refresh” button to configure cache refresh rules for individual files or for all files within a directory.

 **Submit file refresh**

![][9]

This task includes the steps listed below:

1. Select “File refresh.”
2. Click “Add.”
2. Select the domain name you wish to configure from the list of domain names, and then enter the corresponding file path.
3. Click “Add” to continue adding new rules or “x” to delete the corresponding file.
4. Click “Submit.”

The newly created file cache rules then appear in the cache refresh list view.

>**Note:**
>Until the accelerated domain name has passed the ICP verification process, it is not possible to perform a file refresh operation, and the accelerated domain name drop-down list will appear empty. Wait until it has passed back-end verification.

**Submit directory refresh**

![][10]

This task includes the steps listed below:

1. Select “File refresh.”
2. Click “Add.”
2. Select the domain name you wish to configure from the list of domain names, and then enter the corresponding file path.
3. Click “Add” to continue adding new rules or “x” to delete the corresponding directory path.
4. Click “Submit.”

The newly created directory cache rules then appear in the cache refresh list view.

### <a name=""></a>**Preloading**
Preloading means caching the content of a designated URL from the source station to the network nodes, to eliminate the waiting time when a user accesses the resource for the first time. Preloading is generally used in scenarios that involve the delivery of large files, where it can effectively improve the user access experience.

**Preload List View**

In the “Query Previous Data” view on the right side of the Preload view, select the domain name, time range, and status that you want to check, then click **Refresh**. The interface will display preload records that match the conditions you entered.

![][11]
**The content preload list view includes:**

-   Custom domain names, which are URLs that are used to access Content Delivery Network cache content
-   Status (common statuses: successful, failed, or in progress)
-   Submission time

If you successfully submitted the precache rules, the status bar shows the word “Successful.” If you did not successfully submit them, you must check that the precache rules were correct. If there are any problems, you must re-create and re-submit the precache rules.

**Submit Preloading**

Click “Submit Cache Refresh” to configure precaching for individual or multiple files.

![][12]

This task includes the steps listed below:

1. In the “Submit Preload” view, click “Add.”
2. Select the domain name you wish to configure from the list of custom domain names, and then enter the corresponding file path.
3. Click “Add” to continue adding new rules or “x” to delete the corresponding directory path.
4. Click “Submit.”

The newly created precache loading rules then appear in the precache loading list view.

## **Self-service Tools**<a id="step5"></a>

### <a name=""></a>**Service Check**
Once you have created a Content Delivery Network service endpoint, you can perform some basic checks in the “Service Check” view. We strongly recommend that you perform service checks before you carry out CNAME operations.

![][15]

As shown in the view above, you must select the domain name to be checked, provide a resource that the source station can access, and then click “Check.”

1. Source station normal indicates that the resource provided can be accessed.
2. Content Delivery Network deployment complete indicates that the network services corresponding to the domain name have been deployed.
3. Content Delivery Network cache normal indicates that the content that was accessed via the source station is consistent with the content that was accessed via Content Delivery Network (by comparing HTTP headers: HTTP Status Code, Last Modified Time, and Content Length).

>**Note**
>that using the service check function does not guarantee that there are no anomalies in any of the network edge servers where the domain name is located.

### <a name=""></a>**Log Download**
Click “Log Download” in the left navigation pane to set Content Delivery Network raw log download parameters for specific domain names.

 **Log Download View**
To download logs, you must first provide an Azure storage account that is used to save Content Delivery Network logs. Click “Settings” to set this up.

![][13]

 **Download settings view**

In “Settings” (as per the right side of the view below), you can set the storage account and the domain name that you wish to download logs for. You can also verify the accessibility of the storage account. Once the setup is complete, the system automatically saves the logs that it finds to the designated storage account. You can delete the storage account to cancel log downloads.

![][14]

 **Log format details**
Logs are saved in blob format with a container called “cdn-access-logs.” Every blob consists of a .csv file that is compressed with GZip. The meaning of each column within the log is given below:

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

>**Note:**
>If the Content Delivery Network log does not contain content for a particular column, the corresponding record, for example the “c-referrer” record, will be marked “-”. Also, depending on the edge node log configuration, the “rs-duration,” “hit-miss,” and “s-ip” columns may also be empty.

>**Note:**
>Once a website has been accelerated by the Content Delivery Network, the majority of its access records come from network edge nodes. When the network returns to the source, it enters the originating IP address in HTTP Header X-Forwarded-For, and the source station’s web server can edit the log to configure this information. If you need to find out the originating IP address of the client, you can refer to the information below.

>If we take NGINX as an example, you can add the following information to the configuration file:

>log_format logCDN '$remote_addr forwarded for $http_x_forwarded_for - $remote_user [$time_local]  '
                  '"$request" $status $body_bytes_sent '
                  '"$http_referer" "$http_user_agent"';

>access_log /var/log/nginx/access.log logCDN;

## Key Management<a id="step6"></a>

Click “Key Management” in the left navigation pane to generate, create, or delete access keys required by the Azure CDN API interface.

**Existing Key List View**

This view shows all of your keys.

![][18]

**The key list view includes:**

-   ID, the ID of the key
-   Name, the user-friendly name for the key
-   Permissions (including read-only and read-write)
-   Status (including active, canceled, deleting, deleted, and creating)

**Key Details**

Select a key to view details of the key. You can also make changes to the key, for example to update, disable, enable or delete the key.

![][19]

**Create a Key**

Create a new key.

-   Name, the user-friendly name for the key
-   Read-only or read-write permissions

[1]: ./media/cdn-new-portal/001.png
[2]: ./media/cdn-new-portal/002.png
[3]: ./media/cdn-new-portal/003.png
[4]: ./media/cdn-new-portal/cache-policy-2.png
[5]: ./media/cdn-new-portal/access-control.png
[6]: ./media/cdn-new-portal/004.png
[7]: ./media/cdn-new-portal/005.png
[8]: ./media/cdn-new-portal/006.png
[9]: ./media/cdn-new-portal/007.png
[10]: ./media/cdn-new-portal/008.png
[11]: ./media/cdn-new-portal/prefetch-1.png
[12]: ./media/cdn-new-portal/prefetch-2.png
[13]: ./media/cdn-new-portal/log-download-1.png
[14]: ./media/cdn-new-portal/log-download-2.png
[15]: ./media/cdn-new-portal/service-check.png
[16]: ./media/cdn-new-portal/version-change.png
[17]: ./media/cdn-new-portal/property.png
[18]: ./media/cdn-new-portal/key-1.png
[19]: ./media/cdn-new-portal/key-2.png
[20]: ./media/cdn-new-portal/key-3.png
[21]: ./media/cdn-new-portal/key-4.png

<!--HONumber=May17_HO3-->