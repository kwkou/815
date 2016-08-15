<properties linkid="dev-net-common-tasks-cdn" urlDisplayName="CDN" pageTitle="Azure Content Delivery Network FAQs: Azure feature guide" metaKeywords="Azure CDN, Azure CDN, Azure blobs, Azure caching, Azure add-ons, CDN FAQ, CDN FAQs, CDN acceleration, CDN service, configuring CNAME, CNAME, CNAME record, cache refresh, cache rules, CDN edge node, CDN technical documentation, CDN help files" description="Find answers to service configuration questions related to Azure Content Delivery Network" metaCanonical="" services="" documentationCenter=".NET" title="" authors="" solutions="" manager="" editor="" />
<tags ms.service="cdn_en"
    ms.date="7/7/2016"
    wacn.date="7/7/2016"
    wacn.lang="en"
    />
> [AZURE.LANGUAGE]
- [中文](/documentation/articles/cdn-faq-service-config/)
- [English](/documentation/articles/cdn-enus-faq-service-config/) 
#FAQs: Service configuration


+ [Configure CNAME](#step1)
+ [How do I confirm that my CNAME record has taken effect?](#step2)
+ [How do I ensure that content is synchronized with the source station after setting up Azure Content Delivery Network?](#step3)
+ [How do I change the origin domain/IP?](#step4)
+ [How do I set up cache refresh?](#step5)
+ [How do I obtain a visitor’s origin IP address from the origin server log?](#step6)
+ [How to test your CDN before go live?](#step7)

## **Configure CNAME**<a id="step1"></a>
Go to your domain management company and find the parsing manager for the domain. Delete the A record for the domain, and add a CNAME record with the CNAME domain that we provide.

## **How do I confirm that my CNAME record has taken effect?**<a id="step2"></a>
The time at which Domain Name System (DNS) changes take effect varies between regions, and depends on the time to live (TTL) of the original record corresponding to the domain name takes effect. If pinging (or digging) the domain name no longer resolves your origin IP, the CNAME record has already taken effect.

## **How do I ensure that content is synchronized with the source station after setting up Azure Content Delivery Network?**<a id="step3"></a>

- When you set cache rules, you should set different cache refresh rules for different content. You can set shorter cache times for frequently updated content, and longer cache times for content that is not regularly updated, to reduce pressure on the source station.
      
- If the cache refresh period that you set does not expire, but new content is published or some of the content is deleted, you can use the Cache Refresh features provided by Azure CDN management portal to manually force it to refresh.

## **How do I change the origin domain/IP?**<a id="step4"></a>

First, make sure that the new origin server is working normally. Then go to the Domain Management section in Azure CDN management portal, change the origin to the new address, and save.

## **How do I set up cache refresh?**<a id="step5"></a>

- Setting cache rules: You can set different cache refresh rules for different content on the Azure CDN unified portal. You should set shorter cache times for frequently updated content and longer cache times for content that is not regularly updated, to reduce pressure on the source station.
   
- Manual refresh: If the cache refresh period you set does not expire, but new content is published or some of the content is deleted, you can use the Cache Refresh features provided by the Azure CDN management portal to set a file refresh and directory refresh as required, to perform a manual refresh.

## **How do I obtain a visitor’s origin IP address from the origin server log?**<a id="step6"></a>

Once a website is accelerated by using Content Delivery Network, the vast majority of visits will come from the CDN cache nodes. When CDN returns to the source, it will enter the originating IP address in the HTTP Header **X-Forwarded-For**. The origin web server log configuration can be edited so that this information is recorded.

If we take NGINX as an example, you can add the following information to the configuration file:

log\_format logCDN '$remote\_addr forwarded for $http\_x\_forwarded\_for - $remote\_user [$time\_local] '

'"$request" $status $body\_bytes\_sent '

'"$http\_referer" "$http\_user\_agent"';
      
access\_log /var/log/nginx/access.log logCDN;

##**How to test your CDN before go live?**<a id="step7"></a>
 
After you have successfully created CDN for your origin domain _www.abc.com_ on Azure CDN by using the custom domain name _cdn.abc.com_, you will receive the CDN domain _cdn.abc.com.mschcdn.com_. You can adjust the hosts file to perform basic troubleshooting. This is done by using the following procedure:

1. Ping cdn.mydomain.com.mschcdn.com to obtain the IP address of the edge node, for example, a.b.c.d.

2. Adjust the local hosts file and add the record “a.b.c.d www.abc.com.”
     
Then visit _www.abc.com_ in a browser. If the website appears correctly, then there are no problems CDN domain. If it is impossible to access the website, but the website can be successfully accessed after the IP address in the hosts file is changed to the origin IP address, then there is a problem with the CDN service.

>**Note** that in Windows, the path for the hosts file is C:\\Windows\\System32\\drivers\\etc\\hosts. In UNIX-like operating systems such as Linux or BSD, the path for the hosts file is /etc/hosts. Administrator privileges are required to edit this file. Also note that, for the Azure Blob service and Cloud Services, you will get a 404 error message if you directly access the domain name. In such cases, you can troubleshoot by visiting a valid URI.



<!---HONumber=CDN_1201_2015-->
