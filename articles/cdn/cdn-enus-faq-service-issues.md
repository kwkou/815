<properties linkid="dev-net-common-tasks-cdn" urlDisplayName="CDN" pageTitle="Azure CDN FAQs – Azure Feature Guide" metaKeywords="Azure CDN, Azure CDN, Azure blobs, Azure caching, Azure add-ons, cannot cache, cannot CNAME, origin rate high, cache refresh failed, CDN FAQ, CDN FAQS, CDN use failed, CDN service failure, CDN configuration error, speed slow, cannot open website, login exception, CNAME, CDN technical documentation, CDN help files" description="Find answers to common service consulting questions or inquiries related to the Azure CDN" metaCanonical="" services="" documentationCenter=".NET" title="" authors="" solutions="" manager="" editor="" />
<tags ms.service="cdn_en"
    ms.date="7/7/2016"
    wacn.date="7/7/2016"
    wacn.lang="en"
    />
> [AZURE.LANGUAGE]
- [中文](/documentation/articles/cdn-faq-service-issues/)
- [English](/documentation/articles/cdn-enus-faq-service-issues/) 

# FAQs - Service Issues

+ [Why can’t cache my origin URL?](#step1)
+ [Why are custom domain resolved to origin?](#step2)
+ [Cannot Cname*](#step3)
+ [Why is return to origin traffic greater than CDN traffic?](#step4)
+ [Why are websites slower since I started using the CDN](#step5)
+ [The return to origin rate is relatively high](#step6)
+ [Websites will not open since I started using the CDN](#step7)
+ [There are issues with logging in to the website since I started using the CDN](#step8)
+ [Cache refresh failure](#step9)
+ [Why is CDN traffic used up when I haven’t Cnamed to the Azure CDN?](#step10)
 
## **Why can’t cache my origin URL?**<a id="step1"></a>

The inability to cache a URL is usually due to one of the following reasons:

1. The response header for the origin URL includes the following information:
   - Set-Cookie (and the Allow ignore Cookie option has not been checked in the Configure Cache Rules in Azure CDN management portal). Note: When Set-Cookie is used for user login and identification, the Allow Ignore Cookie option cannot be checked; otherwise, it could cause cache issues.
   - Cache-Control: no-store/no-cache/private (and the Allow ignore Cache-Control option has not been checked in the Configure Cache Rules).
   - The Expires time is a time in the past; Expires sets the point in time when the cache expires, so if it is a time in the past, it will make it impossible to cache.
   - The value of Max-age is very small; Max-age sets the length of the caching time in seconds, so if the value is too small, for example less than two digits, it will expire very quickly and make it impossible to cache.

2. The cache rules are not configured or are configured with errors such that the URL cannot hit any caching rules; for example, if the user accidentally enters the rule “[Any Character] (.gif|.jpg|.bmp) (.gif|.jpg|.bmp)”, then it will be impossible to hit the rule even for images, because the file extension is duplicated.
   
3. No users have so far accessed the URL, as some nodes will only cache the content after it has been accessed.

## **Why are custom domain resolved to origin?**<a id="step2"></a>

If the CDN service is served by endpoints in Mainland China, overseas Internet users or Chinese Internet users using non-Chinese DNS resolutions may be resolved to the origin. The Azure CDN platform’s default configuration for such situations is to use CDN nodes in China to serve overseas visits.

## **Cannot Cname**<a id="step3"></a>

Although we have not placed restrictions on CDN custom domains, it is possible that some domain management companies do not allow the use of CNAME for domains without host name. However, there are better ways to deal with this: either change the custom domain name, or change your domain name management company.

## **Why is return to origin traffic greater than CDN traffic?**<a id="step4"></a>

In general, return to origin traffic is less than or equal to CDN traffic, but in certain circumstances, return to origin traffic can also be greater than CDN traffic, e.g. if the visitor sends a request for a relatively large file. For example, if a 100MB file has not been cached by the nodes, the CDN nodes will retrieve it from origin, which for a file of this size inevitably takes some time; however, the visitor may not keep waiting at this time, with the result that the connection is disconnected. In such situation, the CDN nodes will still bring over the entire 100MB file. By the time that the CDN nodes realize that the visitor no longer wants the file, so the return to source traffic will reach 100MB, while the CDN traffic is zero.

## **Why are websites slower since I started using the CDN**<a id="step5"></a>

The cache hit rate is low. The cache hit rate can be affected by several factors:

- Cache configuration issues.
     
- The HTTP header causes inability to cache.
     
- The CDN custom domain has just been added, so the number of cached files is still small.
     
- The type of the origin has limited contents that can be cached.
     
- Website visit numbers are low and the expiration time is short, so the number of cached files is small.
- 
## **The return to origin rate is relatively high**<a id="step6"></a>

1. Cache rule configuration issues
    
2. The amount of resources that can be cached is small
    
3. The cache time is short

## **Websites will not open since I started using the CDN**<a id="step7"></a> 

Possible reasons:
 
1. Origin site issue.
     
2. The origin has a firewall that is preventing the CDN nodes from accessing origin.
    
3. The nodes are blocking the client IP.
     
4. If you have added a new domain or the domain name status has changed, you may need to wait for a while for them to take effect across all CDN edge nodes to, typically around 60 minutes.
     
5. CDN edge nodes failures
    
## **There are issues with logging in to the website since I started using the CDN**<a id="step8"></a>

It is highly likely that resources that shouldn’t be cached have been set up in the cache rules, in which case, you need to set the user back-end login folder to not cache (e.g. /user or /admin); if it is a website with no login or user session, then please enable Allow ignore cookie in Configure Caching Rules on Azure CDN management portal.

## **Cache refresh failure**<a id="step9"></a>

After a manual refresh is submitted, the system will identify the equipment that is working normally to allocate tasks, but if an anomaly occurs in a particular device during the allocation process, the system will automatically detect and disable it (clients will not be parsed to the relevant node after the node is closed, and it will not affect the CDN service). Other normal devices will continue to be allocated tasks until they are complete, at which time, the interface will display “Refresh failed.” At the same time, customer service and operations and maintenance will receive an email notification to deal with the issue as soon as possible, even if the refresh failure does not affect the CDN service.

## **Why is CDN traffic used up when I haven’t Cnamed to the Azure CDN?**<a id="step10"></a>

If the CDN service is enabled, it is possible that our detection equipment will still detect origin and CDN service even if you haven’t “CNAME-ed” to Azure CDN. This is also why we recommend that you choose smaller files when selecting URL monitoring. If you do not want domains with no CNAME to consume data traffic, you can switch the service to the disabled status.

<!---HONumber=CDN_1201_2015-->