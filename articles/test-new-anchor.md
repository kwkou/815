<properties 
	pageTitle="测试文档" 
	description="This tutorial walks you through the steps of implementing a Video-on-Demand (VoD) content delivery application with Azure Media Services using REST APIs." 
	services="media-services" 
	documentationCenter=".net" 
	authors="" 
	manager="Tiffena" 
	editor="Eric Chen"/>

<tags ms.service="test" ms.date="03/12/2015" wacn.date="06/09/2015"/>


# 这是一个测试文档用于测试新标签的网页渲染支持 

V.0625.1

> [AZURE.SELECTOR]
- [Configure VPN connectivity](/documentation/articles/hdinsight-hbase-geo-replication-configure-VNets/)
- [Configure DNS](/documentation/articles/hdinsight-hbase-geo-replication-configure-DNS/)
- [portal](http://www.azure.cn)
 

##test direct reference of md file

> [AZURE.TIP] Subscribe to the [Guest OS Update RSS Feed][rss] to receive the most timely notification on all Guest OS changes. Changes mentioned on that feed will be integrated into this page approximately every week.

> [AZURE.IMPORTANT] To ensure that your service works as expected, you must deploy it to Guest OS release that is compatible with the version of the Azure SDK used to develop your service. If you do not, the deployed service may exhibit errors in the cloud that were not apparent in the development environment.

> [AZURE.WARNING] The retirement of Guest OS family 1 began June 1, 2013 and is scheduled to complete soon. Do not create new installations and upgrade older ones using this Guest OS family. For more information see [Azure Guest OS Family 1 Retirement Information][fam1retire]

##测试Learning Map


<object type="image/svg+xml" data="https://wacn.blob.core.chinacloudapi.cn/documents/WacnElasticScaleMapcoded.svg" width="100%" height="100%">
</object>
0000

<!--
<object type="image/svg+xml" data="http://wacnstoragestaging.blob.core.chinacloudapi.cn/tech-content/articles/media/test-new-anchor/WacnElasticScaleMapcoded.svg" width="100%" height="100%">
</object>
-->

1111
<object type="image/svg+xml" data="./media/test-new-anchor/WacnElasticScaleMapcoded.svg" width="100%" height="100%">
</object>

>[AZURE.NOTE]
> To complete this tutorial, you need an Azure account. If you don't have an account, you can create a free trial account in just a couple of minutes. For details, see <a href="/pricing/1rmb-trial/" target="_blank">Azure Trial</a>.


>[AZURE.NOTE] From now on the new URI will be used in this tutorial.

## <a id="upload"></a>Create a new asset and upload a video file with REST API

In Media Services, you upload your digital files into an asset. The **Asset** entity can contain video, audio, images, thumbnail collections, text tracks and closed caption files (and the metadata about these files.)  Once the files are uploaded into the asset, your content is stored securely in the cloud for further processing and streaming. 

One of the values that you have to provide when creating an asset is asset creation options. The **Options** property is an enumeration value that describes the encryption options that an Asset can be created with. A valid value is one of the values from the list below, not a combination of values from this list:

 
- **None** = **0** - No encryption is used. Note that when using this option your content is not protected in transit or at rest in storage.

	If you plan to deliver an MP4 using progressive download, use this option. 
- **StorageEncrypted** = **1** - Encrypts your clear content locally using AES-256 bit encryption and then uploads it to Azure Storage where it is stored encrypted at rest. Assets protected with Storage Encryption are automatically unencrypted and placed in an encrypted file system prior to encoding, and optionally re-encrypted prior to uploading back as a new output asset. The primary use case for Storage Encryption is when you want to secure your high quality input media files with strong encryption at rest on disk.
- **CommonEncryption** = **2** - Use this option if you are uploading content that has already been encrypted and protected with Common Encryption or PlayReady DRM (for example, Smooth Streaming protected with PlayReady DRM).
- **EnvelopeEncrypted** = **4** – Use this option if you are uploading HLS encrypted with AES. Note that the files must have been encoded and encrypted by Transform Manager.

### Create an asset

An asset is a container for multiple types or sets of objects in Media Services, including video, audio, images, thumbnail collections, text tracks, and closed caption files. In the REST API, creating an Asset requires sending POST request to Media Services and placing any property information about your asset in the request body.

The following example shows how to create an asset.

**HTTP Request**

	POST https://wamsbayclus001rest-hs.cloudapp.net/api/Assets HTTP/1.1
	Content-Type: application/json
	DataServiceVersion: 1.0;NetFx
	MaxDataServiceVersion: 3.0;NetFx
	Accept: application/json
	Accept-Charset: UTF-8
	Authorization: Bearer http%3a%2f%2fschemas.xmlsoap.org%2fws%2f2005%2f05%2fidentity%2fclaims%2fnameidentifier=amstestaccount001&urn%3aSubscriptionId=f7f09258-6753-4ca2-b1ae-193798e2c9d8&http%3a%2f%2fschemas.microsoft.com%2faccesscontrolservice%2f2010%2f07%2fclaims%2fidentityprovider=https%3a%2f%2fwamsprodglobal001acs.accesscontrol.windows.net%2f&Audience=urn%3aWindowsAzureMediaServices&ExpiresOn=1421640053&Issuer=https%3a%2f%2fwamsprodglobal001acs.accesscontrol.windows.net%2f&HMACSHA256=vlG%2fPYdFDMS1zKc36qcFVWnaNh07UCkhYj3B71%2fk1YA%3d
	x-ms-version: 2.8
	x-ms-client-request-id: c59de965-bc89-4295-9a57-75d897e5221e
	Host: wamsbayclus001rest-hs.cloudapp.net
	Content-Length: 45
	
	{"Name":"BigBuckBunny.mp4", "Options":"0"}
	

**HTTP Response**

If successful, the following is returned:
	
	HTP/1.1 201 Created
	Cache-Control: no-cache
	Content-Length: 452
	Content-Type: application/json;odata=minimalmetadata;streaming=true;charset=utf-8
	Location: https://wamsbayclus001rest-hs.cloudapp.net/api/Assets('nb%3Acid%3AUUID%3A9bc8ff20-24fb-4fdb-9d7c-b04c7ee573a1')
	Server: Microsoft-IIS/8.5
	x-ms-client-request-id: c59de965-bc89-4295-9a57-75d897e5221e
	request-id: e98be122-ae09-473a-8072-0ccd234a0657
	x-ms-request-id: e98be122-ae09-473a-8072-0ccd234a0657
	X-Content-Type-Options: nosniff
	DataServiceVersion: 3.0;
	X-Powered-By: ASP.NET
	Strict-Transport-Security: max-age=31536000; includeSubDomains
	Date: Sun, 18 Jan 2015 22:06:40 GMT
		
	{  
	   "odata.metadata":"https://wamsbayclus001rest-hs.cloudapp.net/api/$metadata#Assets/@Element",
	   "Id":"nb:cid:UUID:9bc8ff20-24fb-4fdb-9d7c-b04c7ee573a1",
	   "State":0,
	   "Created":"2015-01-18T22:06:40.6010903Z",
	   "LastModified":"2015-01-18T22:06:40.6010903Z",
	   "AlternateId":null,
	   "Name":"BigBuckBunny2.mp4",
	   "Options":0,
	   "Uri":"https://storagetestaccount001.blob.core.windows.net/asset-9bc8ff20-24fb-4fdb-9d7c-b04c7ee573a1",
	   "StorageAccountName":"storagetestaccount001"
	}
	
### Create an AssetFile

The [AssetFile](https://msdn.microsoft.com/zh-CN/library/azure/hh974275.aspx) entity represents a video or audio file that is stored in a blob container. An asset file is always associated with an asset, and an asset may contain one or many AssetFiles. The Media Services Encoder task fails if an asset file object is not associated with a digital file in a blob container.

After you upload your digital media file into a blob container, you will use the **MERGE** HTTP request to update the AssetFile with information about your media file (as shown later in the topic). 

**HTTP Request**

	POST https://wamsbayclus001rest-hs.cloudapp.net/api/Files HTTP/1.1
	Content-Type: application/json
	DataServiceVersion: 1.0;NetFx
	MaxDataServiceVersion: 3.0;NetFx
	Accept: application/json
	Accept-Charset: UTF-8
	Authorization: Bearer http%3a%2f%2fschemas.xmlsoap.org%2fws%2f2005%2f05%2fidentity%2fclaims%2fnameidentifier=amstestaccount001&urn%3aSubscriptionId=f7f09258-6753-4ca2-b1ae-193798e2c9d8&http%3a%2f%2fschemas.microsoft.com%2faccesscontrolservice%2f2010%2f07%2fclaims%2fidentityprovider=https%3a%2f%2fwamsprodglobal001acs.accesscontrol.windows.net%2f&Audience=urn%3aWindowsAzureMediaServices&ExpiresOn=1421640053&Issuer=https%3a%2f%2fwamsprodglobal001acs.accesscontrol.windows.net%2f&HMACSHA256=vlG%2fPYdFDMS1zKc36qcFVWnaNh07UCkhYj3B71%2fk1YA%3d
	x-ms-version: 2.8
	Host: wamsbayclus001rest-hs.cloudapp.net
	Content-Length: 164
	
	{  
	   "IsEncrypted":"false",
	   "IsPrimary":"false",
	   "MimeType":"video/mp4",
	   "Name":"BigBuckBunny.mp4",
	   "ParentAssetId":"nb:cid:UUID:9bc8ff20-24fb-4fdb-9d7c-b04c7ee573a1"
	}


**HTTP Response**

	HTTP/1.1 201 Created
	Cache-Control: no-cache
	Content-Length: 535
	Content-Type: application/json;odata=minimalmetadata;streaming=true;charset=utf-8
	Location: https://wamsbayclus001rest-hs.cloudapp.net/api/Files('nb%3Acid%3AUUID%3Af13a0137-0a62-9d4c-b3b9-ca944b5142c5')
	Server: Microsoft-IIS/8.5
	request-id: 98a30e2d-f379-4495-988e-0b79edc9b80e
	x-ms-request-id: 98a30e2d-f379-4495-988e-0b79edc9b80e
	X-Content-Type-Options: nosniff
	DataServiceVersion: 3.0;
	X-Powered-By: ASP.NET
	Strict-Transport-Security: max-age=31536000; includeSubDomains
	Date: Mon, 19 Jan 2015 00:34:07 GMT
	
	{  
	   "odata.metadata":"https://wamsbayclus001rest-hs.cloudapp.net/api/$metadata#Files/@Element",
	   "Id":"nb:cid:UUID:f13a0137-0a62-9d4c-b3b9-ca944b5142c5",
	   "Name":"BigBuckBunny.mp4",
	   "ContentFileSize":"0",
	   "ParentAssetId":"nb:cid:UUID:9bc8ff20-24fb-4fdb-9d7c-b04c7ee573a1",
	   "EncryptionVersion":null,
	   "EncryptionScheme":null,
	   "IsEncrypted":false,
	   "EncryptionKeyId":null,
	   "InitializationVector":null,
	   "IsPrimary":false,
	   "LastModified":"2015-01-19T00:34:08.1934137Z",
	   "Created":"2015-01-19T00:34:08.1934137Z",
	   "MimeType":"video/mp4",
	   "ContentChecksum":null
	}


### Creating the AccessPolicy with write permission. 

Before uploading any files into blob storage, set the access policy rights for writing to an asset. To do that, POST an HTTP request to the AccessPolicies entity set. Define a DurationInMinutes value upon creation or you will receive a 500 Internal Server error message back in response. For more information on AccessPolicies, see [AccessPolicy](https://msdn.microsoft.com/zh-CN/library/azure/hh974297.aspx).

The following example shows how to create an AccessPolicy:
		
**HTTP Request**

	POST https://wamsbayclus001rest-hs.cloudapp.net/api/AccessPolicies HTTP/1.1
	Content-Type: application/json
	DataServiceVersion: 1.0;NetFx
	MaxDataServiceVersion: 3.0;NetFx
	Accept: application/json
	Accept-Charset: UTF-8
	Authorization: Bearer http%3a%2f%2fschemas.xmlsoap.org%2fws%2f2005%2f05%2fidentity%2fclaims%2fnameidentifier=amstestaccount001&urn%3aSubscriptionId=f7f09258-6753-4ca2-b1ae-193798e2c9d8&http%3a%2f%2fschemas.microsoft.com%2faccesscontrolservice%2f2010%2f07%2fclaims%2fidentityprovider=https%3a%2f%2fwamsprodglobal001acs.accesscontrol.windows.net%2f&Audience=urn%3aWindowsAzureMediaServices&ExpiresOn=1421640053&Issuer=https%3a%2f%2fwamsprodglobal001acs.accesscontrol.windows.net%2f&HMACSHA256=vlG%2fPYdFDMS1zKc36qcFVWnaNh07UCkhYj3B71%2fk1YA%3d
	x-ms-version: 2.8
	Host: wamsbayclus001rest-hs.cloudapp.net
	Content-Length: 74
	
	{"Name":"NewUploadPolicy", "DurationInMinutes":"440", "Permissions":"2"} 

**HTTP Request**

	If successful, the following response is returned:
	
	HTTP/1.1 201 Created
	Cache-Control: no-cache
	Content-Length: 312
	Content-Type: application/json;odata=minimalmetadata;streaming=true;charset=utf-8
	Location: https://wamsbayclus001rest-hs.cloudapp.net/api/AccessPolicies('nb%3Apid%3AUUID%3Abe0ac48d-af7d-4877-9d60-1805d68bffae')
	Server: Microsoft-IIS/8.5
	request-id: 74c74545-7e0a-4cd6-a440-c1c48074a970
	x-ms-request-id: 74c74545-7e0a-4cd6-a440-c1c48074a970
	X-Content-Type-Options: nosniff
	DataServiceVersion: 3.0;
	X-Powered-By: ASP.NET
	Strict-Transport-Security: max-age=31536000; includeSubDomains
	Date: Sun, 18 Jan 2015 22:18:06 GMT

	{  
	   "odata.metadata":"https://wamsbayclus001rest-hs.cloudapp.net/api/$metadata#AccessPolicies/@Element",
	   "Id":"nb:pid:UUID:be0ac48d-af7d-4877-9d60-1805d68bffae",
	   "Created":"2015-01-18T22:18:06.6370575Z",
	   "LastModified":"2015-01-18T22:18:06.6370575Z",
	   "Name":"NewUploadPolicy",
	   "DurationInMinutes":440.0,
	   "Permissions":2
	}

### Get the Upload URL

To receive the actual upload URL, create a SAS Locator. Locators define the start time and type of connection endpoint for clients that want to access Files in an Asset. You can create multiple Locator entities for a given AccessPolicy and Asset pair to handle different client requests and needs. Each of these Locators use the StartTime value plus the DurationInMinutes value of the AccessPolicy to determine the length of time a URL can be used. For more information, see [Locator](https://msdn.microsoft.com/zh-CN/library/azure/hh974308.aspx).


A SAS URL has the following format:

	{https://myaccount.blob.core.windows.net}/{asset name}/{video file name}?{SAS signature}

Some considerations apply:

- You cannot have more than five unique Locators associated with a given Asset at one time. For more information, see Locator.
- If you need to upload your files immediately, you should set your StartTime value to five minutes before the current time. This is because there may be clock skew between your client machine and Media Services. Also, your StartTime value must be in the following DateTime format: YYYY-MM-DDTHH:mm:ssZ (for example, "2014-05-23T17:53:50Z").	
- There may be a 30-40 second delay after a Locator is created to when it is available for use. This issue applies to both SAS URL and Origin Locators.

The following example shows how to create a SAS URL Locator, as defined by the Type property in the request body ("1" for a SAS locator and "2" for an On-Demand origin locator). The **Path** property returned contains the URL that you must use to upload your file.
	
**HTTP Request**
	
	POST https://wamsbayclus001rest-hs.cloudapp.net/api/Locators HTTP/1.1
	Content-Type: application/json
	DataServiceVersion: 1.0;NetFx
	MaxDataServiceVersion: 3.0;NetFx
	Accept: application/json
	Accept-Charset: UTF-8
	Authorization: Bearer http%3a%2f%2fschemas.xmlsoap.org%2fws%2f2005%2f05%2fidentity%2fclaims%2fnameidentifier=amstestaccount001&urn%3aSubscriptionId=f7f09258-6753-4ca2-b1ae-193798e2c9d8&http%3a%2f%2fschemas.microsoft.com%2faccesscontrolservice%2f2010%2f07%2fclaims%2fidentityprovider=https%3a%2f%2fwamsprodglobal001acs.accesscontrol.windows.net%2f&Audience=urn%3aWindowsAzureMediaServices&ExpiresOn=1421640053&Issuer=https%3a%2f%2fwamsprodglobal001acs.accesscontrol.windows.net%2f&HMACSHA256=vlG%2fPYdFDMS1zKc36qcFVWnaNh07UCkhYj3B71%2fk1YA%3d
	x-ms-version: 2.8
	Host: wamsbayclus001rest-hs.cloudapp.net
	Content-Length: 178
	
	{  
	   "AccessPolicyId":"nb:pid:UUID:be0ac48d-af7d-4877-9d60-1805d68bffae",
	   "AssetId":"nb:cid:UUID:9bc8ff20-24fb-4fdb-9d7c-b04c7ee573a1",
	   "StartTime":"2015-02-18T16:45:53",
	   "Type":1
	}


**HTTP Response**

If successful, the following response is returned:
		
	HTTP/1.1 201 Created
	Cache-Control: no-cache
	Content-Length: 949
	Content-Type: application/json;odata=minimalmetadata;streaming=true;charset=utf-8
	Location: https://wamsbayclus001rest-hs.cloudapp.net/api/Locators('nb%3Alid%3AUUID%3Aaf57bdd8-6751-4e84-b403-f3c140444b54')
	Server: Microsoft-IIS/8.5
	request-id: 2adeb1f8-89c5-4cc8-aa4f-08cdfef33ae0
	x-ms-request-id: 2adeb1f8-89c5-4cc8-aa4f-08cdfef33ae0
	X-Content-Type-Options: nosniff
	DataServiceVersion: 3.0;
	X-Powered-By: ASP.NET
	Strict-Transport-Security: max-age=31536000; includeSubDomains
	Date: Mon, 19 Jan 2015 03:01:29 GMT
	
	{  
	   "odata.metadata":"https://wamsbayclus001rest-hs.cloudapp.net/api/$metadata#Locators/@Element",
	   "Id":"nb:lid:UUID:af57bdd8-6751-4e84-b403-f3c140444b54",
	   "ExpirationDateTime":"2015-02-19T00:05:53",
	   "Type":1,
	   "Path":"https://storagetestaccount001.blob.core.windows.net/asset-f438649c-313c-46e2-8d68-7d2550288247?sv=2012-02-12&sr=c&si=af57bdd8-6751-4e84-b403-f3c140444b54&sig=fE4btwEfZtVQFeC0Wh3Kwks2OFPQfzl5qTMW5YytiuY%3D&st=2015-02-18T16%3A45%3A53Z&se=2015-02-19T00%3A05%3A53Z",
	   "BaseUri":"https://storagetestaccount001.blob.core.windows.net/asset-f438649c-313c-46e2-8d68-7d2550288247",
	   "ContentAccessComponent":"?sv=2012-02-12&sr=c&si=af57bdd8-6751-4e84-b403-f3c140444b54&sig=fE4btwEfZtVQFeC0Wh3Kwks2OFPQfzl5qTMW5YytiuY%3D&st=2015-02-18T16%3A45%3A53Z&se=2015-02-19T00%3A05%3A53Z",
	   "AccessPolicyId":"nb:pid:UUID:be0ac48d-af7d-4877-9d60-1805d68bffae",
	   "AssetId":"nb:cid:UUID:9bc8ff20-24fb-4fdb-9d7c-b04c7ee573a1",
	   "StartTime":"2015-02-18T16:45:53",
	   "Name":null
	}

### Upload a file into a blob storage container
	
Once you have the AccessPolicy and Locator set, the actual file is uploaded to an Azure Blob Storage container using the Azure Storage REST APIs. You can either upload in page or block blobs. 

>[AZURE.NOTE] You must add the file name for the file you want to upload to the Locator **Path** value received in the previous section. For example, https://storagetestaccount001.blob.core.windows.net/asset-e7b02da4-5a69-40e7-a8db-e8f4f697aac0/BigBuckBunny.mp4? . . . 

For more information on working with Azure storage blobs, see [Blob Service REST API](https://msdn.microsoft.com/zh-CN/library/azure/dd135733.aspx).


### Update the AssetFile 

Now that you've uploaded your file, update the FileAsset size (and other) information. For example:
	
	MERGE https://wamsbayclus001rest-hs.cloudapp.net/api/Files('nb%3Acid%3AUUID%3Af13a0137-0a62-9d4c-b3b9-ca944b5142c5') HTTP/1.1
	Content-Type: application/json
	DataServiceVersion: 1.0;NetFx
	MaxDataServiceVersion: 3.0;NetFx
	Accept: application/json
	Accept-Charset: UTF-8
	Authorization: Bearer http%3a%2f%2fschemas.xmlsoap.org%2fws%2f2005%2f05%2fidentity%2fclaims%2fnameidentifier=amstestaccount001&urn%3aSubscriptionId=f7f09258-6753-4ca2-b1ae-193798e2c9d8&http%3a%2f%2fschemas.microsoft.com%2faccesscontrolservice%2f2010%2f07%2fclaims%2fidentityprovider=https%3a%2f%2fwamsprodglobal001acs.accesscontrol.windows.net%2f&Audience=urn%3aWindowsAzureMediaServices&ExpiresOn=1421662918&Issuer=https%3a%2f%2fwamsprodglobal001acs.accesscontrol.windows.net%2f&HMACSHA256=utmoXXbm9Q7j4tW1yJuMVA3egRiQy5FPygwadkmPeaY%3d
	x-ms-version: 2.8
	Host: wamsbayclus001rest-hs.cloudapp.net
	
	{  
	   "ContentFileSize":"1186540",
	   "Id":"nb:cid:UUID:f13a0137-0a62-9d4c-b3b9-ca944b5142c5",
	   "MimeType":"video/mp4",
	   "Name":"BigBuckBunny.mp4",
	   "ParentAssetId":"nb:cid:UUID:9bc8ff20-24fb-4fdb-9d7c-b04c7ee573a1"
	}


**HTTP Response**

If successful, the following is returned:
	HTTP/1.1 204 No Content

## Delete the Locator and AccessPolicy 

**HTTP Request**


	DELETE https://wamsbayclus001rest-hs.cloudapp.net/api/Locators('nb%3Alid%3AUUID%3Aaf57bdd8-6751-4e84-b403-f3c140444b54') HTTP/1.1
	DataServiceVersion: 1.0;NetFx
	MaxDataServiceVersion: 3.0;NetFx
	Accept: application/json
	Accept-Charset: UTF-8
	Authorization: Bearer http%3a%2f%2fschemas.xmlsoap.org%2fws%2f2005%2f05%2fidentity%2fclaims%2fnameidentifier=amstestaccount001&urn%3aSubscriptionId=f7f09258-6753-4ca2-b1ae-193798e2c9d8&http%3a%2f%2fschemas.microsoft.com%2faccesscontrolservice%2f2010%2f07%2fclaims%2fidentityprovider=https%3a%2f%2fwamsprodglobal001acs.accesscontrol.windows.net%2f&Audience=urn%3aWindowsAzureMediaServices&ExpiresOn=1421662918&Issuer=https%3a%2f%2fwamsprodglobal001acs.accesscontrol.windows.net%2f&HMACSHA256=utmoXXbm9Q7j4tW1yJuMVA3egRiQy5FPygwadkmPeaY%3d
	x-ms-version: 2.8
	Host: wamsbayclus001rest-hs.cloudapp.net

	
**HTTP Response**

If successful, the following is returned:

	HTTP/1.1 204 No Content 
	...

**HTTP Request**

	DELETE https://wamsbayclus001rest-hs.cloudapp.net/api/AccessPolicies('nb%3Apid%3AUUID%3Abe0ac48d-af7d-4877-9d60-1805d68bffae') HTTP/1.1
	DataServiceVersion: 1.0;NetFx
	MaxDataServiceVersion: 3.0;NetFx
	Accept: application/json
	Accept-Charset: UTF-8
	Authorization: Bearer http%3a%2f%2fschemas.xmlsoap.org%2fws%2f2005%2f05%2fidentity%2fclaims%2fnameidentifier=amstestaccount001&urn%3aSubscriptionId=f7f09258-6753-4ca2-b1ae-193798e2c9d8&http%3a%2f%2fschemas.microsoft.com%2faccesscontrolservice%2f2010%2f07%2fclaims%2fidentityprovider=https%3a%2f%2fwamsprodglobal001acs.accesscontrol.windows.net%2f&Audience=urn%3aWindowsAzureMediaServices&ExpiresOn=1421662918&Issuer=https%3a%2f%2fwamsprodglobal001acs.accesscontrol.windows.net%2f&HMACSHA256=utmoXXbm9Q7j4tW1yJuMVA3egRiQy5FPygwadkmPeaY%3d
	x-ms-version: 2.8
	Host: wamsbayclus001rest-hs.cloudapp.net

**HTTP Response**

If successful, the following is returned:

	HTTP/1.1 204 No Content 
	...

 
## <a id="configure_streaming_units"></a>Configure streaming units with REST API

When working with Azure Media Services one of the most common scenarios is delivering adaptive bitrate streaming to your clients. With adaptive bitrate streaming, the client can switch to a higher or lower bitrate stream as the video is displayed based on the current network bandwidth, CPU utilization, and other factors. Media Services supports the following adaptive bitrate streaming technologies: HTTP Live Streaming (HLS), Smooth Streaming, MPEG DASH, and HDS (for Adobe PrimeTime/Access licensees only). 

Media Services provides dynamic packaging which allows you to deliver your adaptive bitrate MP4 or Smooth Streaming encoded content in streaming formats supported by Media Services (MPEG DASH, HLS, Smooth Streaming, HDS) without you having to re-package into these streaming formats. 

To take advantage of dynamic packaging, you need to do the following:

- get at least one On-demand streaming unit for the streaming endpoint from which you plan to delivery your content (described in this section).
- encode or transcode your mezzanine (source) file into a set of adaptive bitrate MP4 files or adaptive bitrate Smooth Streaming files (the encoding steps are demonstrated later in this tutorial),  

With dynamic packaging you only need to store and pay for the files in single storage format and Media Services will build and serve the appropriate response based on requests from a client. 

Note that in addition to being able to use the dynamic packaging capabilities, On-Demand Streaming reserved units provide you with dedicated egress capacity that can be purchased in increments of 200 Mbps. By default, on-demand streaming is configured in a shared-instance model for which server resources (for example, compute, egress capacity, etc.) are shared with all other users. To improve an on-demand streaming throughput, it is recommended to purchase On-Demand Streaming reserved units.

>[AZURE.NOTE] For information about pricing details, see [Media Services Pricing Details](http://go.microsoft.com/fwlink/?LinkId=275107).

To change the number of on-demand streaming reserved units, do the following:
	
### Get the streaming endpoint you want to update

For example, let's get the first streaming endpoint in your account (you can have up to 2 streaming endpoints in running state at the same time.)

**HTTP Request**:

	GET https://wamsbayclus001rest-hs.cloudapp.net/api/StreamingEndpoints()?$top=1 HTTP/1.1
	DataServiceVersion: 1.0;NetFx
	MaxDataServiceVersion: 3.0;NetFx
	Accept: application/json;odata=verbose
	Accept-Charset: UTF-8
	Authorization: Bearer http%3a%2f%2fschemas.xmlsoap.org%2fws%2f2005%2f05%2fidentity%2fclaims%2fnameidentifier=amstestaccount001&urn%3aSubscriptionId=f7f09258-6753-4ca2-b1ae-193798e2c9d8&http%3a%2f%2fschemas.microsoft.com%2faccesscontrolservice%2f2010%2f07%2fclaims%2fidentityprovider=https%3a%2f%2fwamsprodglobal001acs.accesscontrol.windows.net%2f&Audience=urn%3aWindowsAzureMediaServices&ExpiresOn=1421466122&Issuer=https%3a%2f%2fwamsprodglobal001acs.accesscontrol.windows.net%2f&HMACSHA256=TiKGEOTporft4pFGU24sSZRZk5GRAWszFXldl5NXAhY%3d
	x-ms-version: 2.8
	Host: wamsbayclus001rest-hs.cloudapp.net

**HTTP Response**
	
If successful, the following is returned:
	
	HTTP/1.1 200 OK
	. . . 
	
### Scale the streaming endpoint
 
**HTTP Request**:

	POST https://wamsbayclus001rest-hs.cloudapp.net/api/StreamingEndpoints('nb:oid:UUID:cd57670d-cc1c-0f86-16d8-3ad478bf9486')/Scale HTTP/1.1
	Content-Type: application/json;odata=verbose
	MaxDataServiceVersion: 3.0;NetFx
	Accept: application/json;odata=verbose
	Accept-Charset: UTF-8
	Authorization: Bearer http%3a%2f%2fschemas.xmlsoap.org%2fws%2f2005%2f05%2fidentity%2fclaims%2fnameidentifier=amstestaccount001&urn%3aSubscriptionId=f7f09258-6753-4ca2-b1ae-193798e2c9d8&http%3a%2f%2fschemas.microsoft.com%2faccesscontrolservice%2f2010%2f07%2fclaims%2fidentityprovider=https%3a%2f%2fwamsprodglobal001acs.accesscontrol.windows.net%2f&Audience=urn%3aWindowsAzureMediaServices&ExpiresOn=1421466122&Issuer=https%3a%2f%2fwamsprodglobal001acs.accesscontrol.windows.net%2f&HMACSHA256=TiKGEOTporft4pFGU24sSZRZk5GRAWszFXldl5NXAhY%3d
	x-ms-version: 2.8
	x-ms-client-request-id: 39f96c93-a4b1-43ce-b97e-b2aaa44ee2dd
	Host: wamsbayclus001rest-hs.cloudapp.net
	
	{"scaleUnits":1}

**HTTP Response**

	HTTP/1.1 202 Accepted
	Cache-Control: no-cache
	Server: Microsoft-IIS/8.5
	x-ms-client-request-id: 39f96c93-a4b1-43ce-b97e-b2aaa44ee2dd
	request-id: 3c1ba1c7-281c-4b2d-a898-09cb70a3a424
	x-ms-request-id: 3c1ba1c7-281c-4b2d-a898-09cb70a3a424
	operation-id: nb:opid:UUID:1853bcbf-b71f-4ed5-a4c7-a581d4f45ae7
	X-Content-Type-Options: nosniff
	DataServiceVersion: 1.0;
	Strict-Transport-Security: max-age=31536000; includeSubDomains
	Date: Fri, 16 Jan 2015 22:16:43 GMT
	Content-Length: 0

	
### <a id="long_running_op_status"></a> Check on the status of a long-running operation

The allocation of any new units of on-demand streaming takes around 20 minutes to complete. To check the status of the operation use the **Operations** method and specify the Id of the operation. The operation Id was returned in the response to the **Scale** request.

	operation-id: nb:opid:UUID:1853bcbf-b71f-4ed5-a4c7-a581d4f45ae7
 
**HTTP Request**:
	
	GET https://wamsbayclus001rest-hs.cloudapp.net/api/Operations('nb:opid:UUID:1853bcbf-b71f-4ed5-a4c7-a581d4f45ae7') HTTP/1.1
	MaxDataServiceVersion: 3.0;NetFx
	Accept: application/json;odata=verbose
	Accept-Charset: UTF-8
	Authorization: Bearer http%3a%2f%2fschemas.xmlsoap.org%2fws%2f2005%2f05%2fidentity%2fclaims%2fnameidentifier=amstestaccount001&urn%3aSubscriptionId=f7f09258-6753-4ca2-b1ae-193798e2c9d8&http%3a%2f%2fschemas.microsoft.com%2faccesscontrolservice%2f2010%2f07%2fclaims%2fidentityprovider=https%3a%2f%2fwamsprodglobal001acs.accesscontrol.windows.net%2f&Audience=urn%3aWindowsAzureMediaServices&ExpiresOn=1421466122&Issuer=https%3a%2f%2fwamsprodglobal001acs.accesscontrol.windows.net%2f&HMACSHA256=TiKGEOTporft4pFGU24sSZRZk5GRAWszFXldl5NXAhY%3d
	x-ms-version: 2.8
	Host: wamsbayclus001rest-hs.cloudapp.net
	
**HTTP Response**
	
	HTTP/1.1 200 OK
	Cache-Control: no-cache
	Content-Length: 515
	Content-Type: application/json;odata=verbose;charset=utf-8
	Server: Microsoft-IIS/8.5
	x-ms-client-request-id: 829e1a89-3ec2-4836-a04d-802b5aeff5e8
	request-id: f7ae8a78-af8d-4881-b9ea-ca072cfe0b60
	x-ms-request-id: f7ae8a78-af8d-4881-b9ea-ca072cfe0b60
	X-Content-Type-Options: nosniff
	DataServiceVersion: 1.0;
	Strict-Transport-Security: max-age=31536000; includeSubDomains
	Date: Fri, 16 Jan 2015 22:57:39 GMT
	
	{  
	   "d":{  
	      "__metadata":{  
	         "id":"https://wamsbayclus001rest-hs.cloudapp.net/api/Operations('nb%3Aopid%3AUUID%3Acc339c28-6bba-4f7d-bee5-91ea4a0a907e')",
	         "uri":"https://wamsbayclus001rest-hs.cloudapp.net/api/Operations('nb%3Aopid%3AUUID%3Acc339c28-6bba-4f7d-bee5-91ea4a0a907e')",
	         "type":"Microsoft.Cloud.Media.Vod.Rest.Data.Models.Operation"
	      },
	      "Id":"nb:opid:UUID:cc339c28-6bba-4f7d-bee5-91ea4a0a907e",
	      "State":"Succeeded",
	      "TargetEntityId":"nb:oid:UUID:cd57670d-cc1c-0f86-16d8-3ad478bf9486",
	      "ErrorCode":null,
	      "ErrorMessage":null
	   }
	}


## <a id="encode"></a>Encode the source file into a set of adaptive bitrate MP4 files

After ingesting Assets into Media Services, media can be encoded, transmuxed, watermarked, and so on, before it is delivered to clients. These activities are scheduled and run against multiple background role instances to ensure high performance and availability. These activities are called Jobs and each [Job](https://msdn.microsoft.com/zh-CN/library/azure/hh974289.aspx) is comprised of atomic Tasks that do the actual work on the Asset file. 

As was mentioned earlier, when working with Azure Media Services one of the most common scenarios is delivering adaptive bitrate streaming to your clients. Media Services can dynamically package a set of adaptive bitrate MP4 files into one of the following formats: HTTP Live Streaming (HLS), Smooth Streaming, MPEG DASH, and HDS (for Adobe PrimeTime/Access licensees only). 

To take advantage of dynamic packaging, you need to do the following:

- encode or transcode your mezzanine (source) file into a set of adaptive bitrate MP4 files or adaptive bitrate Smooth Streaming files,  
- get at least one On-demand streaming unit for the streaming endpoint from which you plan to delivery your content. 

The following section shows how to create a job that contains one encoding task. The task specifies to transcode the mezzanine file into a set of adaptive bitrate MP4s using **Azure Media Encoder**. The section also shows how to monitor the job processing progress. When the job is complete you you would be able to create locators that are needed to get access to your assets. 

### Get a media processor

In Media Services, a media processor is a component that handles a specific processing task, such as encoding, format conversion, encrypting, or decrypting media content. For the encoding task shown in this tutorial we are going to use the Azure Media Encoder.

The following code requests the encoder's id. 

**HTTP Request**

	GET https://wamsbayclus001rest-hs.cloudapp.net/api/MediaProcessors()?$filter=Name%20eq%20'Azure%20Media%20Encoder' HTTP/1.1
	DataServiceVersion: 1.0;NetFx
	MaxDataServiceVersion: 3.0;NetFx
	Accept: application/json
	Accept-Charset: UTF-8
	Authorization: Bearer http%3a%2f%2fschemas.xmlsoap.org%2fws%2f2005%2f05%2fidentity%2fclaims%2fnameidentifier=amstestaccount001&urn%3aSubscriptionId=f7f09258-6753-4ca2-b1ae-193798e2c9d8&http%3a%2f%2fschemas.microsoft.com%2faccesscontrolservice%2f2010%2f07%2fclaims%2fidentityprovider=https%3a%2f%2fwamsprodglobal001acs.accesscontrol.windows.net%2f&Audience=urn%3aWindowsAzureMediaServices&ExpiresOn=1421675491&Issuer=https%3a%2f%2fwamsprodglobal001acs.accesscontrol.windows.net%2f&HMACSHA256=9hUudHYnATpi5hN3cvTfgw%2bL4N3tL0fdsRnQnm6ZYIU%3d
	x-ms-version: 2.8
	Host: wamsbayclus001rest-hs.cloudapp.net
	

**HTTP Response**

	HTTP/1.1 200 OK
	Cache-Control: no-cache
	Content-Length: 273
	Content-Type: application/json;odata=minimalmetadata;streaming=true;charset=utf-8
	Server: Microsoft-IIS/8.5
	request-id: 6beb04b4-55a7-480d-8aa8-e5c5d59ffa1f
	x-ms-request-id: 6beb04b4-55a7-480d-8aa8-e5c5d59ffa1f
	X-Content-Type-Options: nosniff
	DataServiceVersion: 3.0;
	X-Powered-By: ASP.NET
	Strict-Transport-Security: max-age=31536000; includeSubDomains
	Date: Mon, 19 Jan 2015 07:54:09 GMT
	
	{  
	   "odata.metadata":"https://wamsbayclus001rest-hs.cloudapp.net/api/$metadata#MediaProcessors",
	   "value":[  
	      {  
	         "Id":"nb:mpid:UUID:1b1da727-93ae-4e46-a8a1-268828765609",
	         "Description":"Azure Media Encoder",
	         "Name":"Azure Media Encoder",
	         "Sku":"",
	         "Vendor":"Microsoft",
	         "Version":"4.4"
	      }
	   ]
	}

### Create a job

Each Job can have one or more Tasks depending on the type of processing that you want to accomplish. Through the REST API, you can create Jobs and their related Tasks in one of two ways: Tasks can be defined inline through the Tasks navigation property on Job entities, or through OData batch processing. The Media Services SDK uses batch processing; however, for the readability of the code examples in this topic, tasks are defined inline. For information on batch processing, see [Open Data Protocol (OData) Batch Processing](http://www.odata.org/documentation/odata-version-3-0/batch-processing).

The following example shows you how to create and post a Job with one Task set to encode a video at a specific resolution and quality. The following documentation section contains the list of all the [task presets](https://msdn.microsoft.com/zh-CN/library/azure/dn619392.aspx) supported by the Azure Media Processor.  

**HTTP Request**
	
	POST https://wamsbayclus001rest-hs.cloudapp.net/api/Jobs HTTP/1.1
	DataServiceVersion: 1.0;NetFx
	MaxDataServiceVersion: 3.0;NetFx
	Content-Type: application/json
	Accept: application/json;odata=verbose
	Accept-Charset: UTF-8
	Authorization: Bearer http%3a%2f%2fschemas.xmlsoap.org%2fws%2f2005%2f05%2fidentity%2fclaims%2fnameidentifier=amstestaccount001&urn%3aSubscriptionId=f7f09258-6753-4ca2-b1ae-193798e2c9d8&http%3a%2f%2fschemas.microsoft.com%2faccesscontrolservice%2f2010%2f07%2fclaims%2fidentityprovider=https%3a%2f%2fwamsprodglobal001acs.accesscontrol.windows.net%2f&Audience=urn%3aWindowsAzureMediaServices&ExpiresOn=1421675491&Issuer=https%3a%2f%2fwamsprodglobal001acs.accesscontrol.windows.net%2f&HMACSHA256=9hUudHYnATpi5hN3cvTfgw%2bL4N3tL0fdsRnQnm6ZYIU%3d
	x-ms-version: 2.8
	Host: wamsbayclus001rest-hs.cloudapp.net
	Content-Length: 482
	
	{  
	   "Name":"NewTestJob",
	   "InputMediaAssets":[  
	      {  
	         "__metadata":{  
	            "uri":"https://wamsbayclus001rest-hs.net/api/Assets('nb%3Acid%3AUUID%3A9bc8ff20-24fb-4fdb-9d7c-b04c7ee573a1')"
	         }
	      }
	   ],
	   "Tasks":[  
	      {  
	         "Configuration":"H264 Adaptive Bitrate MP4 Set 720p",
	         "MediaProcessorId":"nb:mpid:UUID:1b1da727-93ae-4e46-a8a1-268828765609",
	         "TaskBody":"<?xml version=\"1.0\" encoding=\"utf-8\"?><taskBody><inputAsset>JobInputAsset(0)</inputAsset>
				<outputAsset>JobOutputAsset(0)</outputAsset></taskBody>"
	      }
	   ]
	}

**HTTP Response**

If successful, the following response is returned:

	HTTP/1.1 201 Created
	Cache-Control: no-cache
	Content-Length: 1215
	Content-Type: application/json;odata=verbose;charset=utf-8
	Location: https://wamsbayclus001rest-hs.cloudapp.net/api/Jobs('nb%3Ajid%3AUUID%3A71d2dd33-efdf-ec43-8ea1-136a110bd42c')
	Server: Microsoft-IIS/8.5
	request-id: 532ac1ec-a475-4dce-b2d5-7c8ce94ac87c
	x-ms-request-id: 532ac1ec-a475-4dce-b2d5-7c8ce94ac87c
	X-Content-Type-Options: nosniff
	DataServiceVersion: 3.0;
	X-Powered-By: ASP.NET
	Strict-Transport-Security: max-age=31536000; includeSubDomains
	Date: Mon, 19 Jan 2015 08:04:35 GMT
		
	{  
	   "d":{  
	      "__metadata":{  
	         "id":"https://wamsbayclus001rest-hs.cloudapp.net/api/Jobs('nb%3Ajid%3AUUID%3A71d2dd33-efdf-ec43-8ea1-136a110bd42c')",
	         "uri":"https://wamsbayclus001rest-hs.cloudapp.net/api/Jobs('nb%3Ajid%3AUUID%3A71d2dd33-efdf-ec43-8ea1-136a110bd42c')",
	         "type":"Microsoft.Cloud.Media.Vod.Rest.Data.Models.Job"
	      },
	      "Tasks":{  
	         "__deferred":{  
	            "uri":"https://wamsbayclus001rest-hs.cloudapp.net/api/Jobs('nb%3Ajid%3AUUID%3A71d2dd33-efdf-ec43-8ea1-136a110bd42c')/Tasks"
	         }
	      },
	      "OutputMediaAssets":{  
	         "__deferred":{  
	            "uri":"https://wamsbayclus001rest-hs.cloudapp.net/api/Jobs('nb%3Ajid%3AUUID%3A71d2dd33-efdf-ec43-8ea1-136a110bd42c')/OutputMediaAssets"
	         }
	      },
	      "InputMediaAssets":{  
	         "__deferred":{  
	            "uri":"https://wamsbayclus001rest-hs.cloudapp.net/api/Jobs('nb%3Ajid%3AUUID%3A9bc8ff20-24fb-4fdb-9d7c-b04c7ee573a1')/InputMediaAssets"
	         }
	      },
	      "Id":"nb:jid:UUID:71d2dd33-efdf-ec43-8ea1-136a110bd42c",
	      "Name":"NewTestJob",
	      "Created":"2015-01-19T08:04:34.3287057Z",
	      "LastModified":"2015-01-19T08:04:34.3287057Z",
	      "EndTime":null,
	      "Priority":0,
	      "RunningDuration":0,
	      "StartTime":null,
	      "State":0,
	      "TemplateId":null,
	      "JobNotificationSubscriptions":{  
	         "__metadata":{  
	            "type":"Collection(Microsoft.Cloud.Media.Vod.Rest.Data.Models.JobNotificationSubscription)"
	         },
	         "results":[  
	
	         ]
	      }
	   }
	}


There are a few important things to note in any Job request:

- TaskBody properties MUST use literal XML to define the number of input, or output assets that will be used by the Task. The Task topic contains the XML Schema Definition for the XML.
- In the TaskBody definition, each inner value for <inputAsset> and <outputAsset> must be set as JobInputAsset(value) or JobOutputAsset(value).
- A task can have multiple output assets. One JobOutputAsset(x) can only be used once as an output of a task in a job.
- You can specify JobInputAsset or JobOutputAsset as an input asset of a task.
- Tasks must not form a cycle.
- The value parameter that you pass to JobInputAsset or JobOutputAsset represents the index value for an Asset. The actual Assets are defined in the InputMediaAssets and OutputMediaAssets navigation properties on the Job entity definition. 

>[AZURE.NOTE] Because Media Services is built on OData v3, the individual assets in InputMediaAssets and OutputMediaAssets navigation property collections are referenced through a "__metadata : uri" name-value pair. 

- InputMediaAssets maps to one or more Assets you have created in Media Services. OutputMediaAssets are created by the system. They do not reference an existing asset.
- OutputMediaAssets can be named using the assetName attribute. If this attribute is not present, then the name of the OutputMediaAsset will be whatever the inner text value of the <outputAsset> element is with a suffix of either the Job Name value, or the Job Id value (in the case where the Name property isn't defined). For example, if you set a value for assetName to "Sample", then the OutputMediaAsset Name property would be set to "Sample". However, if you did not set a value for assetName, but did set the job name to "NewJob", then the OutputMediaAsset Name would be "JobOutputAsset(value)_NewJob". 

	The following example shows how to set the assetName attribute:
	
		"<?xml version=\"1.0\" encoding=\"utf-8\"?><taskBody><inputAsset>JobInputAsset(0)</inputAsset><outputAsset assetName=\"CustomOutputAssetName\">JobOutputAsset(0)</outputAsset></taskBody>"


- To enable task chaining:

	- A job must have at least 2 tasks
	- There must be at least one task whose input is output of another task in the job.

For more information see, [Creating an Encoding Job with the Media Services REST API](https://msdn.microsoft.com/zh-CN/library/azure/jj129574.aspx).

### Monitor Processing Progress

You can retrieve the Job status by using the State property, as shown in the following example. 

**HTTP Request**

	GET https://wamsbayclus001rest-hs.net/api/Jobs('nb%3Ajid%3AUUID%3A71d2dd33-efdf-ec43-8ea1-136a110bd42c')/State HTTP/1.1
	Content-Type: application/json;odata=verbose
	Accept: application/json;odata=verbose
	DataServiceVersion: 3.0
	MaxDataServiceVersion: 3.0
	x-ms-version: 2.8
	Authorization: Bearer http%3a%2f%2fschemas.xmlsoap.org%2fws%2f2005%2f05%2fidentity%2fclaims%2fnameidentifier=youraccountname&urn%3aSubscriptionId=2f84471d-b1ae-4e75-aa09-010f0fc0cf5b&http%3a%2f%2fschemas.microsoft.com%2faccesscontrolservice%2f2010%2f07%2fclaims%2fidentityprovider=https%3a%2f%2fwamsprodglobal001acs.accesscontrol.windows.net%2f&Audience=urn%3aWindowsAzureMediaServices&ExpiresOn=1336908022&Issuer=https%3a%2f%2fwamsprodglobal001acs.accesscontrol.windows.net%2f&HMACSHA256=RYXOraO6Z%2f7l9whWZQN%2bypeijgHwIk8XyikA01Kx1%2bk%3d
	Host: wamsbayclus001rest-hs.net
	Content-Length: 0


**HTTP Response**

If successful, the following response is returned:

	HTTP/1.1 200 OK
	Cache-Control: no-cache
	Content-Length: 17
	Content-Type: application/json;odata=verbose;charset=utf-8
	Server: Microsoft-IIS/7.5
	x-ms-request-id: 01324d81-18e2-4493-a3e5-c6186209f0c8
	X-Content-Type-Options: nosniff
	DataServiceVersion: 1.0;
	X-AspNet-Version: 4.0.30319
	X-Powered-By: ASP.NET
	Date: Sun, 13 May 2012 05:16:53 GMT
	
	{"d":{"State":2}}


### Cancel a job

Media Services allows you to cancel running jobs through the CancelJob function. This call will return a 400 error code if you try to cancel a Job when its state is canceled, canceling, error, or finished.

The following example shows how to call CancelJob.


**HTTP Request**


	GET https://wamsbayclus001rest-hs.net/API/CancelJob?jobid='nb%3ajid%3aUUID%3a71d2dd33-efdf-ec43-8ea1-136a110bd42c' HTTP/1.1
	Content-Type: application/json;odata=verbose
	Accept: application/json;odata=verbose
	DataServiceVersion: 3.0
	MaxDataServiceVersion: 3.0
	x-ms-version: 2.2
	Authorization: Bearer http%3a%2f%2fschemas.xmlsoap.org%2fws%2f2005%2f05%2fidentity%2fclaims%2fnameidentifier=youraccountname&urn%3aSubscriptionId=2f84471d-b1ae-4e75-aa09-010f0fc0cf5b&http%3a%2f%2fschemas.microsoft.com%2faccesscontrolservice%2f2010%2f07%2fclaims%2fidentityprovider=https%3a%2f%2fwamsprodglobal001acs.accesscontrol.windows.net%2f&Audience=urn%3aWindowsAzureMediaServices&ExpiresOn=1336908753&Issuer=https%3a%2f%2fwamsprodglobal001acs.accesscontrol.windows.net%2f&HMACSHA256=kolAgnFfbQIeRv4lWxKSFa4USyjWXRm15Kd%2bNd5g8eA%3d
	Host: wamsbayclus001rest-hs.net


If successful, a 204 response code is returned with no message body.

>[AZURE.NOTE] You must URL-encode the job id (normally nb:jid:UUID: somevalue) when passing it in as a parameter to CancelJob.


### Get the output asset 

In the next section we are going to configure the delivery policy for the job's output asset (the encoded asset). The following code shows how to request the output asset Id. 


**HTTP Request**

	GET https://wamsbayclus001rest-hs.cloudapp.net/api/Jobs('nb%3Ajid%3AUUID%3A71d2dd33-efdf-ec43-8ea1-136a110bd42c')/OutputMediaAssets() HTTP/1.1
	DataServiceVersion: 1.0;NetFx
	MaxDataServiceVersion: 3.0;NetFx
	Accept: application/json
	Accept-Charset: UTF-8
	User-Agent: Microsoft ADO.NET Data Services
	Authorization: Bearer http%3a%2f%2fschemas.xmlsoap.org%2fws%2f2005%2f05%2fidentity%2fclaims%2fnameidentifier=amstestaccount001&urn%3aSubscriptionId=f7f09258-6753-4ca2-b1ae-193798e2c9d8&http%3a%2f%2fschemas.microsoft.com%2faccesscontrolservice%2f2010%2f07%2fclaims%2fidentityprovider=https%3a%2f%2fwamsprodglobal001acs.accesscontrol.windows.net%2f&Audience=urn%3aWindowsAzureMediaServices&ExpiresOn=1421675491&Issuer=https%3a%2f%2fwamsprodglobal001acs.accesscontrol.windows.net%2f&HMACSHA256=9hUudHYnATpi5hN3cvTfgw%2bL4N3tL0fdsRnQnm6ZYIU%3d
	x-ms-version: 2.8
	Host: wamsbayclus001rest-hs.cloudapp.net
	

**HTTP Response**

	HTTP/1.1 200 OK
	Cache-Control: no-cache
	Content-Length: 445
	Content-Type: application/json;odata=minimalmetadata;streaming=true;charset=utf-8
	Server: Microsoft-IIS/8.5
	request-id: 73cd605d-066a-46f1-8358-f4bd25a9220a
	x-ms-request-id: 73cd605d-066a-46f1-8358-f4bd25a9220a
	X-Content-Type-Options: nosniff
	DataServiceVersion: 3.0;
	X-Powered-By: ASP.NET
	Strict-Transport-Security: max-age=31536000; includeSubDomains
	Date: Mon, 19 Jan 2015 08:28:13 GMT
		
	{  
	   "odata.metadata":"https://wamsbayclus001rest-hs.cloudapp.net/api/$metadata#Assets",
	   "value":[  
	      {  
	         "Id":"nb:cid:UUID:71d2dd33-efdf-ec43-8ea1-136a110bd42c",
	         "State":0,
	         "Created":"2015-01-19T07:52:15.603",
	         "LastModified":"2015-01-19T07:52:15.603",
	         "AlternateId":null,
	         "Name":"Multibitrate MP4s",
	         "Options":0,
	         "Uri":"https://storagetestaccount001.blob.core.windows.net/asset-71d2dd33-efdf-ec43-8ea1-136a110bd42c",
	         "StorageAccountName":"storagetestaccount001"
	      }
	   ]
	}


## <a id="configure_delivery_method"></a>Configure delivery policy for the encoded asset

One of the steps in the Media Services content delivery workflow is configuring asset delivery policies. Some things that the asset delivery policy configuration includes: what protocols can be used to deliver the asset (for example, MPEG DASH, HLS, HDS, Smooth Streaming or all), whether you want to dynamically encrypt your asset and how (envelope or common encryption). 

The following **AssetDeliveryPolicies** HTTP Request specifies not to apply dynamic encryption (AssetDeliveryPolicyType could be one of these values: None = 0, Blocked = 1, NoDynamicEncryption = 2, DynamicEnvelopeEncryption = 3, DynamicCommonEncryption = 4) and to deliver the stream in any of the following protocols:  MPEG DASH, HLS, and Smooth Streaming protocols (AssetDeliveryProtocol could be a combination of the following values: None = 0,        SmoothStreaming = 1, Dash = 2, HLS = 4, Hds = 8, All = 65535). 


As a result of this delivery configuration you would be able to request a Smooth, HLS, or MPEG DASH streams using the following formats:

Smooth Streaming:

	{streaming endpoint name-media services account name}.streaming.mediaservices.windows.net/{locator ID}/{filename}.ism/Manifest

HLS:

	{streaming endpoint name-media services account name}.streaming.mediaservices.windows.net/{locator ID}/{filename}.ism/Manifest(format=m3u8-aapl)

MPEG DASH

	{streaming endpoint name-media services account name}.streaming.mediaservices.windows.net/{locator ID}/{filename}.ism/Manifest(format=mpd-time-csf) 


### Create AssetDeliveryPolicies


**HTTP Request**
		
	POST https://wamsbayclus001rest-hs.cloudapp.net/api/AssetDeliveryPolicies HTTP/1.1
	Content-Type: application/json
	DataServiceVersion: 1.0;NetFx
	MaxDataServiceVersion: 3.0;NetFx
	Accept: application/json
	Accept-Charset: UTF-8
	Authorization: Bearer http%3a%2f%2fschemas.xmlsoap.org%2fws%2f2005%2f05%2fidentity%2fclaims%2fnameidentifier=amstestaccount001&urn%3aSubscriptionId=f7f09258-6753-4ca2-b1ae-193798e2c9d8&http%3a%2f%2fschemas.microsoft.com%2faccesscontrolservice%2f2010%2f07%2fclaims%2fidentityprovider=https%3a%2f%2fwamsprodglobal001acs.accesscontrol.windows.net%2f&Audience=urn%3aWindowsAzureMediaServices&ExpiresOn=1421679198&Issuer=https%3a%2f%2fwamsprodglobal001acs.accesscontrol.windows.net%2f&HMACSHA256=aUvBcDwRAFk1JLxceWu%2bf9dVrCZM7PrTRbZd0TtoKvU%3d
	x-ms-version: 2.8
	Host: wamsbayclus001rest-hs.cloudapp.net
	Content-Length: 83
	
	{"Name":"Clear Policy", "AssetDeliveryPolicyType":"2","AssetDeliveryProtocol":"7"} 


**HTTP Response**
	
	HTTP/1.1 201 Created
	Cache-Control: no-cache
	Content-Length: 361
	Content-Type: application/json;odata=minimalmetadata;streaming=true;charset=utf-8
	Location: https://wamsbayclus001rest-hs.cloudapp.net/api/AssetDeliveryPolicies('nb%3Aadpid%3AUUID%3Aef97ae36-b898-4b8a-b427-7819ee726276')
	Server: Microsoft-IIS/8.5
	request-id: 7391a4b2-995d-4fc5-aca5-a398786ea38e
	x-ms-request-id: 7391a4b2-995d-4fc5-aca5-a398786ea38e
	X-Content-Type-Options: nosniff
	DataServiceVersion: 3.0;
	X-Powered-By: ASP.NET
	Strict-Transport-Security: max-age=31536000; includeSubDomains
	Date: Mon, 19 Jan 2015 09:13:18 GMT

	{  
	   "odata.metadata":"https://wamsbayclus001rest-hs.cloudapp.net/api/$metadata#AssetDeliveryPolicies/@Element",
	   "Id":"nb:adpid:UUID:ef97ae36-b898-4b8a-b427-7819ee726276",
	   "Name":"Clear Policy",
	   "AssetDeliveryProtocol":7,
	   "AssetDeliveryPolicyType":2,
	   "AssetDeliveryConfiguration":null,
	   "Created":"2015-01-19T09:13:18.911615Z",
	   "LastModified":"2015-01-19T09:13:18.911615Z"
	}
    

### Link the asset delivery policy with an asset

The following HTTP request associates the specified delivery policy with the specified asset.  

**HTTP Request**

	POST https://wamsbayclus001rest-hs.cloudapp.net/api/Assets('nb%3Acid%3AUUID%3A71d2dd33-efdf-ec43-8ea1-136a110bd42c')/$links/DeliveryPolicies HTTP/1.1
	DataServiceVersion: 1.0;NetFx
	MaxDataServiceVersion: 3.0;NetFx
	Accept: application/json
	Accept-Charset: UTF-8
	Content-Type: application/json
	Authorization: Bearer http%3a%2f%2fschemas.xmlsoap.org%2fws%2f2005%2f05%2fidentity%2fclaims%2fnameidentifier=amstestaccount001&urn%3aSubscriptionId=f7f09258-6753-4ca2-b1ae-193798e2c9d8&http%3a%2f%2fschemas.microsoft.com%2faccesscontrolservice%2f2010%2f07%2fclaims%2fidentityprovider=https%3a%2f%2fwamsprodglobal001acs.accesscontrol.windows.net%2f&Audience=urn%3aWindowsAzureMediaServices&ExpiresOn=1421679198&Issuer=https%3a%2f%2fwamsprodglobal001acs.accesscontrol.windows.net%2f&HMACSHA256=aUvBcDwRAFk1JLxceWu%2bf9dVrCZM7PrTRbZd0TtoKvU%3d
	x-ms-version: 2.8
	Host: wamsbayclus001rest-hs.cloudapp.net
	Content-Length: 140
	
	{ "uri":"https://wamsbayclus001rest-hs.cloudapp.net/api/AssetDeliveryPolicies('nb%3Aadpid%3AUUID%3Aef97ae36-b898-4b8a-b427-7819ee726276')" }


**HTTP Response**

	HTTP/1.1 204 No Content
    . . . 


## <a id="publish_get_urls"></a>Publish the asset and get streaming and progressive download URLs with REST API

To stream or download an asset you first need to "publish" it by creating a locator. Locators provide access to files contained in the asset. Media Services supports two types of locators: OnDemandOrigin locators, used to stream media (for example, MPEG DASH, HLS, or Smooth Streaming) and Access Signature (SAS) locators, used to download media files.

Once you create the locators, you can build the URLs that are used to stream or download your files. 


An on-demand URL for Smooth Streaming has the following format:

	{streaming endpoint name-media services account name}.streaming.mediaservices.windows.net/{locator ID}/{filename}.ism/Manifest

An on-demand URL for HLS has the following format:

	{streaming endpoint name-media services account name}.streaming.mediaservices.windows.net/{locator ID}/{filename}.ism/Manifest(format=m3u8-aapl)

An on-demand URL for MPEG DASH has the following format:

	{streaming endpoint name-media services account name}.streaming.mediaservices.windows.net/{locator ID}/{filename}.ism/Manifest(format=mpd-time-csf)


A SAS URL used to download files has the following format:

	{blob container name}/{asset name}/{file name}/{SAS signature}

This sections shows how to perform the following tasks necessary to "publish" your assets.  

- Creating the AccessPolicy with read permission 
- Creating a SAS URL for downloading content 
- Creating an origin URL for streaming content 

###Creating the AccessPolicy with read permission

Before downloading or streaming any media content, first define an AccessPolicy with read permissions and create the appropriate Locator entity that specifies the type of delivery mechanism you want to enable for your clients. For more information on the properties available, see [AccessPolicy Entity Properties](https://msdn.microsoft.com/zh-CN/library/azure/hh974297.aspx#accesspolicy_properties).

The following example shows how to specify the AccessPolicy for read permissions for a given Asset.

	POST https://wamsbayclus001rest-hs.net/API/AccessPolicies HTTP/1.1
	Content-Type: application/json
	Accept: application/json
	DataServiceVersion: 3.0
	MaxDataServiceVersion: 3.0
	x-ms-version: 2.8
	Authorization: Bearer http%3a%2f%2fschemas.xmlsoap.org%2fws%2f2005%2f05%2fidentity%2fclaims%2fnameidentifier=youraccountname&urn%3aSubscriptionId=2f84471d-b1ae-4e75-aa09-010f0fc0cf5b&http%3a%2f%2fschemas.microsoft.com%2faccesscontrolservice%2f2010%2f07%2fclaims%2fidentityprovider=https%3a%2f%2fwamsprodglobal001acs.accesscontrol.windows.net%2f&Audience=urn%3aWindowsAzureMediaServices&ExpiresOn=1337067658&Issuer=https%3a%2f%2fwamsprodglobal001acs.accesscontrol.windows.net%2f&HMACSHA256=dithjGvlXR9HlyAf5DE99N5OCYkPAxsHIcsTSjm9%2fVE%3d
	Host: wamsbayclus001rest-hs.net
	Content-Length: 74
	Expect: 100-continue
	
	{"Name": "DownloadPolicy", "DurationInMinutes" : "300", "Permissions" : 1}

If successful, a 201 success code is returned describing the AccessPolicy entity that you created. You will then use the AccessPolicy Id along with the Asset Id of the asset that contains the file you want to deliver(such as an output asset) to create the Locator entity.

>[AZURE.NOTE]
This basic workflow is the same as uploading a file when ingesting an Asset (as was discussed earlier in this topic). Also, like uploading files, if you (or your clients) need to access your files immediately, set your StartTime value to five minutes before the current time. This action is necessary because there may be clock skew between the client and Media Services. The StartTime value must be in the following DateTime format: YYYY-MM-DDTHH:mm:ssZ (for example, "2014-05-23T17:53:50Z").


###Creating a SAS URL for downloading content 

The following code shows how to get a URL that can be used to download a media file created and uploaded previously. The AccessPolicy has read permissions set and the Locator path refers to a SAS download URL.

	POST https://wamsbayclus001rest-hs.net/API/Locators HTTP/1.1
	Content-Type: application/json
	Accept: application/json
	DataServiceVersion: 3.0
	MaxDataServiceVersion: 3.0
	x-ms-version: 2.8
	Authorization: Bearer http%3a%2f%2fschemas.xmlsoap.org%2fws%2f2005%2f05%2fidentity%2fclaims%2fnameidentifier=youraccountname&urn%3aSubscriptionId=2f84471d-b1ae-4e75-aa09-010f0fc0cf5b&http%3a%2f%2fschemas.microsoft.com%2faccesscontrolservice%2f2010%2f07%2fclaims%2fidentityprovider=https%3a%2f%2fwamsprodglobal001acs.accesscontrol.windows.net%2f&Audience=urn%3aWindowsAzureMediaServices&ExpiresOn=1337067658&Issuer=https%3a%2f%2fwamsprodglobal001acs.accesscontrol.windows.net%2f&HMACSHA256=dithjGvlXR9HlyAf5DE99N5OCYkPAxsHIcsTSjm9%2fVE%3d
	Host: wamsbayclus001rest-hs.net
	Content-Length: 182
	Expect: 100-continue
	
	{"AccessPolicyId": "nb:pid:UUID:38c71dd0-44c5-4c5f-8418-08bb6fbf7bf8", "AssetId" : "nb:cid:UUID:71d2dd33-efdf-ec43-8ea1-136a110bd42c", "StartTime" : "2014-05-17T16:45:53", "Type":1}

If successful, the following response is returned:

	
	HTTP/1.1 201 Created
	Cache-Control: no-cache
	Content-Length: 1150
	Content-Type: application/json;odata=verbose;charset=utf-8
	Location: https://wamsbayclus001rest-hs.net/api/Locators('nb%3Alid%3AUUID%3A8e5a821d-2194-4d00-8884-adf979856874')
	Server: Microsoft-IIS/7.5
	x-ms-request-id: 8cfd8556-3064-416a-97f2-67d9e35f0911
	X-Content-Type-Options: nosniff
	DataServiceVersion: 1.0;
	X-AspNet-Version: 4.0.30319
	X-Powered-By: ASP.NET
	Date: Mon, 14 May 2012 21:41:32 GMT
		
	{  
	   "d":{  
	      "__metadata":{  
	         "id":"https://wamsbayclus001rest-hs.net/api/Locators('nb%3Alid%3AUUID%3A8e5a821d-2194-4d00-8884-adf979856874')",
	         "uri":"https://wamsbayclus001rest-hs.net/api/Locators('nb%3Alid%3AUUID%3A8e5a821d-2194-4d00-8884-adf979856874')",
	         "type":"Microsoft.Cloud.Media.Vod.Rest.Data.Models.Locator"
	      },
	      "AccessPolicy":{  
	         "__deferred":{  
	            "uri":"https://wamsbayclus001rest-hs.net/api/Locators('nb%3Alid%3AUUID%3A8e5a821d-2194-4d00-8884-adf979856874')/AccessPolicy"
	         }
	      },
	      "Asset":{  
	         "__deferred":{  
	            "uri":"https://wamsbayclus001rest-hs.net/api/Locators('nb%3Alid%3AUUID%3A71d2dd33-efdf-ec43-8ea1-136a110bd42c')/Asset"
	         }
	      },
	      "Id":"nb:lid:UUID:8e5a821d-2194-4d00-8884-adf979856874",
	      "ExpirationDateTime":"\/Date(1337049393000)\/",
	      "Type":1,
	      "Path":"https://storagetestaccount001.blob.core.windows.net/asset-71d2dd33-efdf-ec43-8ea1-136a110bd42c?st=2012-05-14T21%3A36%3A33Z&se=2012-05-15T02%3A36%3A33Z&sr=c&si=8e5a821d-2194-4d00-8884-adf979856874&sig=y75dViDpC5V8WutrXM%2B%2FGpR3uOtqmlISiNlHU1YUBOg%3D",
	      "AccessPolicyId":"nb:pid:UUID:38c71dd0-44c5-4c5f-8418-08bb6fbf7bf8",
	      "AssetId":"nb:cid:UUID:71d2dd33-efdf-ec43-8ea1-136a110bd42c",
	      "StartTime":"\/Date(1337031393000)\/"
	   }
	}


The returned **Path** property contains the SAS URL.

>[AZURE.NOTE]
If you download storage encrypted content, you must manually decrypt it before rendering it, or use the Storage Decryption MediaProcessor in a processing task to output processed files in the clear to an OutputAsset and then download from that Asset. For more information on processing, see Creating an Encoding Job with the Media Services REST API. Also, SAS URL Locators cannot be updated after they have been created. For example, you cannot reuse the same Locator with an updated StartTime value. This is because of the way SAS URLs are created. If you want to access an asset for download after a Locator has expired, then you must create a new one with a new StartTime.

###Download files

Once you have the AccessPolicy and Locator set, you can download files using the Azure Storage REST APIs.  

>[AZURE.NOTE] You must add the file name for the file you want to download to the Locator **Path** value received in the previous section. For example, https://storagetestaccount001.blob.core.windows.net/asset-e7b02da4-5a69-40e7-a8db-e8f4f697aac0/BigBuckBunny.mp4? . . . 

For more information on working with Azure storage blobs, see [Blob Service REST API](https://msdn.microsoft.com/zh-CN/library/azure/dd135733.aspx).

As a result of the encoding job that you performed earlier (encoding into Adaptive MP4 set), you have multiple MP4 files that you can progressively download. For example:    
	
	https://storagetestaccount001.blob.core.windows.net/asset-38058602-a4b8-4b33-b9f0-6880dc1490ea/BigBuckBunny_H264_650kbps_AAC_und_ch2_96kbps.mp4?sv=2012-02-12&sr=c&si=166d5154-b801-410b-a226-ee2f8eac1929&sig=P2iNZJAvAWpp%2Bj9yV6TQjoz5DIIaj7ve8ARynmEM6Xk%3D&se=2015-02-14T01:13:05Z
	
	https://storagetestaccount001.blob.core.windows.net/asset-38058602-a4b8-4b33-b9f0-6880dc1490ea/BigBuckBunny_H264_400kbps_AAC_und_ch2_96kbps.mp4?sv=2012-02-12&sr=c&si=166d5154-b801-410b-a226-ee2f8eac1929&sig=P2iNZJAvAWpp%2Bj9yV6TQjoz5DIIaj7ve8ARynmEM6Xk%3D&se=2015-02-14T01:13:05Z
	
	https://storagetestaccount001.blob.core.windows.net/asset-38058602-a4b8-4b33-b9f0-6880dc1490ea/BigBuckBunny_H264_3400kbps_AAC_und_ch2_96kbps.mp4?sv=2012-02-12&sr=c&si=166d5154-b801-410b-a226-ee2f8eac1929&sig=P2iNZJAvAWpp%2Bj9yV6TQjoz5DIIaj7ve8ARynmEM6Xk%3D&se=2015-02-14T01:13:05Z
	
	https://storagetestaccount001.blob.core.windows.net/asset-38058602-a4b8-4b33-b9f0-6880dc1490ea/BigBuckBunny_H264_2250kbps_AAC_und_ch2_96kbps.mp4?sv=2012-02-12&sr=c&si=166d5154-b801-410b-a226-ee2f8eac1929&sig=P2iNZJAvAWpp%2Bj9yV6TQjoz5DIIaj7ve8ARynmEM6Xk%3D&se=2015-02-14T01:13:05Z
	
	https://storagetestaccount001.blob.core.windows.net/asset-38058602-a4b8-4b33-b9f0-6880dc1490ea/BigBuckBunny_H264_1500kbps_AAC_und_ch2_96kbps.mp4?sv=2012-02-12&sr=c&si=166d5154-b801-410b-a226-ee2f8eac1929&sig=P2iNZJAvAWpp%2Bj9yV6TQjoz5DIIaj7ve8ARynmEM6Xk%3D&se=2015-02-14T01:13:05Z
	
	https://storagetestaccount001.blob.core.windows.net/asset-38058602-a4b8-4b33-b9f0-6880dc1490ea/BigBuckBunny_H264_1000kbps_AAC_und_ch2_96kbps.mp4?sv=2012-02-12&sr=c&si=166d5154-b801-410b-a226-ee2f8eac1929&sig=P2iNZJAvAWpp%2Bj9yV6TQjoz5DIIaj7ve8ARynmEM6Xk%3D&se=2015-02-14T01:13:05Z
	
	https://storagetestaccount001.blob.core.windows.net/asset-38058602-a4b8-4b33-b9f0-6880dc1490ea/BigBuckBunny_AAC_und_ch2_96kbps.mp4?sv=2012-02-12&sr=c&si=166d5154-b801-410b-a226-ee2f8eac1929&sig=P2iNZJAvAWpp%2Bj9yV6TQjoz5DIIaj7ve8ARynmEM6Xk%3D&se=2015-02-14T01:13:05Z
	
	https://storagetestaccount001.blob.core.windows.net/asset-38058602-a4b8-4b33-b9f0-6880dc1490ea/BigBuckBunny_AAC_und_ch2_56kbps.mp4?sv=2012-02-12&sr=c&si=166d5154-b801-410b-a226-ee2f8eac1929&sig=P2iNZJAvAWpp%2Bj9yV6TQjoz5DIIaj7ve8ARynmEM6Xk%3D&se=2015-02-14T01:13:05Z


###Creating an on-demand origin URL for streaming content


The following code shows how to create an origin URL Locator:

	POST https://wamsbayclus001rest-hs/API/Locators HTTP/1.1
	Content-Type: application/json
	Accept: application/json
	DataServiceVersion: 3.0
	MaxDataServiceVersion: 3.0
	x-ms-version: 2.8
	Authorization: Bearer http%3a%2f%2fschemas.xmlsoap.org%2fws%2f2005%2f05%2fidentity%2fclaims%2fnameidentifier=youraccountname&urn%3aSubscriptionId=2f84471d-b1ae-4e75-aa09-010f0fc0cf5b&http%3a%2f%2fschemas.microsoft.com%2faccesscontrolservice%2f2010%2f07%2fclaims%2fidentityprovider=https%3a%2f%2fwamsprodglobal001acs.accesscontrol.windows.net%2f&Audience=urn%3aWindowsAzureMediaServices&ExpiresOn=1337067658&Issuer=https%3a%2f%2fwamsprodglobal001acs.accesscontrol.windows.net%2f&HMACSHA256=dithjGvlXR9HlyAf5DE99N5OCYkPAxsHIcsTSjm9%2fVE%3d
	Host: wamsbayclus001rest-hs
	Content-Length: 182
	Expect: 100-continue
	
	{"AccessPolicyId": "nb:pid:UUID:38c71dd0-44c5-4c5f-8418-08bb6fbf7bf8", "AssetId" : "nb:cid:UUID:eb5540a2-116e-4d36-b084-7e9958f7f3c3", "StartTime" : "2014-05-17T16:45:53",, "Type":2}

If successful, the following response is returned:

	HTTP/1.1 201 Created
	Cache-Control: no-cache
	Content-Length: 981
	Content-Type: application/json;odata=verbose;charset=utf-8
	Location: https://wamsbayclus001rest-hs.net/api/Locators('nb%3Alid%3AUUID%3A52034bf6-dfae-4d83-aad3-3bd87dcb1a5d')
	Server: Microsoft-IIS/7.5
	x-ms-request-id: 2eac4158-fc78-4fa2-81ee-c9f582dc385f
	X-Content-Type-Options: nosniff
	DataServiceVersion: 1.0;
	X-AspNet-Version: 4.0.30319
	X-Powered-By: ASP.NET
	Date: Mon, 14 May 2012 21:41:39 GMT
		
	{  
	   "d":{  
	      "__metadata":{  
	         "id":"https://wamsbayclus001rest-hs.net/api/Locators('nb%3Alid%3AUUID%3A52034bf6-dfae-4d83-aad3-3bd87dcb1a5d')",
	         "uri":"https://wamsbayclus001rest-hs.net/api/Locators('nb%3Alid%3AUUID%3A52034bf6-dfae-4d83-aad3-3bd87dcb1a5d')",
	         "type":"Microsoft.Cloud.Media.Vod.Rest.Data.Models.Locator"
	      },
	      "AccessPolicy":{  
	         "__deferred":{  
	            "uri":"https://wamsbayclus001rest-hs.net/api/Locators('nb%3Alid%3AUUID%3A52034bf6-dfae-4d83-aad3-3bd87dcb1a5d')/AccessPolicy"
	         }
	      },
	      "Asset":{  
	         "__deferred":{  
	            "uri":"https://wamsbayclus001rest-hs.net/api/Locators('nb%3Alid%3AUUID%3A52034bf6-dfae-4d83-aad3-3bd87dcb1a5d')/Asset"
	         }
	      },
	      "Id":"nb:lid:UUID:52034bf6-dfae-4d83-aad3-3bd87dcb1a5d",
	      "ExpirationDateTime":"\/Date(1337049395000)\/",
	      "Type":2,
	      "Path":"http://wamsbayclus001rest-hs.net/52034bf6-dfae-4d83-aad3-3bd87dcb1a5d/",
	      "AccessPolicyId":"nb:pid:UUID:38c71dd0-44c5-4c5f-8418-08bb6fbf7bf8",
	      "AssetId":"nb:cid:UUID:eb5540a2-116e-4d36-b084-7e9958f7f3c3",
	      "StartTime":"\/Date(1337031395000)\/"
	   }
	}

To stream a Smooth Streaming origin URL in a streaming media player, you must append the Path property with the name of the Smooth Streaming manifest file, followed by "/manifest".

	http://amstestaccount001.streaming.mediaservices.windows.net/ebf733c4-3e2e-4a68-b67b-cc5159d1d7f2/BigBuckBunny.ism/manifest

To stream HLS, append (format=m3u8-aapl) after the "/manifest".

	http://amstestaccount001.streaming.mediaservices.windows.net/ebf733c4-3e2e-4a68-b67b-cc5159d1d7f2/BigBuckBunny.ism/manifest(format=m3u8-aapl)

To stream MPEG DASH, append (format=mpd-time-csf) after the "/manifest".

	http://amstestaccount001.streaming.mediaservices.windows.net/ebf733c4-3e2e-4a68-b67b-cc5159d1d7f2/BigBuckBunny.ism/manifest(format=mpd-time-csf)


## <a id="play"></a>Play your content  

Once you have SAS URLs, you can progressively download your MP4 files by pasting a URL into a browser (for example, IE, Chrome, Safari).

To test adaptive streaming URLs use the following players:

To test smooth streaming, use [http://amsplayer.azurewebsites.net/](http://amsplayer.azurewebsites.net), or [http://smf.cloudapp.net/healthmonitor](http://smf.cloudapp.net/healthmonitor).

To test MPEG DASH, use [http://dashif.org](http://dashif.org/reference/players/javascript).

To test HLS, use iOS or Safari devices, or [3ivx-hls-player](http://apps.microsoft.com/windows/zh-CN/app/3ivx-hls-player/f79ce7d0-2993-4658-bc4e-83dc182a0614). 



<h2>Additional Resources</h2>
- <a href="http://channel9.msdn.com/Shows/Azure-Friday/Azure-Media-Services-101-Get-your-video-online-now-">Azure Media Services 101 - Get your video online now!</a>
- <a href="http://channel9.msdn.com/Shows/Azure-Friday/Azure-Media-Services-102-Dynamic-Packaging-and-Mobile-Devices">Azure Media Services 102 - Dynamic Packaging and Mobile Devices</a>


<!-- Anchors. -->


<!-- URLs. -->
  [Web Platform Installer]: http://go.microsoft.com/fwlink/?linkid=255386
  [Management Portal]: http://manage.windowsazure.cn/


