<properties 
	pageTitle="如何使用 REST API 检查作业进度 | Azure" 
	description="了解如何跟踪作业进度。" 
	services="media-services" 
	documentationCenter="" 
	authors="Juliako" 
	manager="erikre" 
	editor=""/>  


<tags 
	ms.service="media-services" 
	ms.workload="media" 
	ms.tgt_pltfrm="na" 
	ms.devlang="na" 
	ms.topic="article" 
	ms.date="09/26/2016"   
	wacn.date="12/27/2016"   
	ms.author="juliako"/>

#如何：检查作业进度

> [AZURE.SELECTOR]
- [门户](/documentation/articles/media-services-portal-check-job-progress/)
- [.NET](/documentation/articles/media-services-check-job-progress/)
- [REST](/documentation/articles/media-services-rest-check-job-progress/)

当你运行作业时，通常需要采用某种方式跟踪作业进度。你可以使用作业的 State 属性查看该作业的状态。有关 State 属性的详细信息，请参阅[作业实体属性](https://docs.microsoft.com/zh-cn/rest/api/media/operations/job#a-namejobentitypropertiesa-job-entity-properties)。


请求：
	
	GET https://wamsshaclus001rest-hs.chinacloudapp.cn/api/Jobs()?$filter=Id%20eq%20'nb%3Ajid%3AUUID%3Af3c43f94-327f-2347-90bb-3bf79f8559f1'&$top=1 HTTP/1.1
	DataServiceVersion: 1.0;NetFx
	MaxDataServiceVersion: 3.0;NetFx
	Accept: application/json
	Accept-Charset: UTF-8
	Authorization: Bearer http%3a%2f%2fschemas.xmlsoap.org%2fws%2f2005%2f05%2fidentity%2fclaims%2fnameidentifier=juliakoams1&urn%3aSubscriptionId=zbbef702-2233-477b-9f16-bc4d3aa97387&http%3a%2f%2fschemas.microsoft.com%2faccesscontrolservice%2f2010%2f07%2fclaims%2fidentityprovider=https%3a%2f%2fwamsprodglobal001acs.accesscontrol.chinacloudapi.cn%2f&Audience=urn%3aWindowsAzureMediaServices&ExpiresOn=1423640758&Issuer=https%3a%2f%2fwamsprodglobal001acs.accesscontrol.chinacloudapi.cn%2f&HMACSHA256=z5yFIG%2bk8Z2G2aXABqM60P9smHNKD7P4BfSxXanwKFc%3d
	x-ms-version: 2.11
	Host: wamsshaclus001rest-hs.chinacloudapp.cn
	


响应：
	
	HTTP/1.1 200 OK
	Cache-Control: no-cache
	Content-Length: 450
	Content-Type: application/json;odata=minimalmetadata;streaming=true;charset=utf-8
	Server: Microsoft-IIS/8.5
	x-ms-client-request-id: d9b83c57-e26c-4d10-a20b-2be634c4b6a8
	request-id: 91d2be35-20ed-4e1c-a147-e82cd000c193
	x-ms-request-id: 91d2be35-20ed-4e1c-a147-e82cd000c193
	X-Content-Type-Options: nosniff
	DataServiceVersion: 3.0;
	Strict-Transport-Security: max-age=31536000; includeSubDomains
	
	{"odata.metadata":"https://wamsshaclus001rest-hs.chinacloudapp.cn/api/$metadata#Jobs","value":[{"Id":"nb:jid:UUID:f3c43f94-327f-2347-90bb-3bf79f8559f1","Name":"Encoding BigBuckBunny into to H264 Adaptive Bitrate MP4 Set 720p","Created":"2015-02-11T01:46:08.897","LastModified":"2015-02-11T01:46:08.897","EndTime":null,"Priority":0,"RunningDuration":0.0,"StartTime":"2015-02-11T01:46:16.58","State":2,"TemplateId":null,"JobNotificationSubscriptions":[]}]} 

<!---HONumber=Mooncake_Quality_Review_1215_2016-->