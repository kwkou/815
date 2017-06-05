<properties
    linkid="dev-net-common-tasks-cdn"
    urlDisplayName="CDN"
    pageTitle="Azure China CDN API doc"
    metaKeywords="Azure CDN, Azure CDN, Azure blobs, Azure caching, Azure add-on, Live Streaming, Streaming media acceleration, CDN acceleration, CDN services, mainstream CDN, live streaming media acceleration, media services, Azure Media Service, cache rules, HLS, CDN technology files, CDN help files, live video acceleration, live broadcast acceleration"
    description="Learn How to Create Live Streaming Acceleration Type CDNs on Azure Management Portal and Default Caching Rules for Live Streaming CDNs"
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
    ms.date="5/4/2017"
    wacn.date="5/4/2017"
    wacn.lang="en" />

> [AZURE.LANGUAGE]
- [中文](/documentation/articles/cdn-api/)
- [English](/documentation/articles/cdn-enus-api/)

# <a name="cdn-api"></a>CDN API overview


##<a name=""></a>Overview

You can use this interface to manage Azure China CDN resources. You can obtain the corresponding results by sending an HTTPS request to the CDN API server and adding the corresponding parameters to the request in accordance with the interface instructions. The Azure CDN will authenticate each access request. Requests must be submitted using the HTTPS protocol, and the request must include the signature information. Please refer to the document signing mechanism for the specific operation process. Character encoding is UTF-8.

The API can be used to implement the following operations:

- Node management – create, delete, enable, and disable CDN nodes for the specified Azure subscription; query and update CDN node information, for example by editing the CDN source station; set cache rules
- Traffic query – view traffic information by month, day, hour, or minute for the subscription or individual domain names
- Content management – cache refresh and preloading

<!--HONumber=May17_HO3-->