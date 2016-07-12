<properties 
	pageTitle="使用 REST 创建 ContentKey" 
	description="了解如何创建提供对资产进行安全访问的内容密钥。" 
	services="media-services" 
	documentationCenter="" 
	authors="Juliako" 
	manager="dwrede" 
	editor=""/>

<tags
	ms.service="media-services"
 	ms.date="04/18/2016"   
	wacn.date="06/27/2016"/>


#使用 REST 创建 ContentKey


> [AZURE.SELECTOR]
- [REST](/documentation/articles/media-services-rest-create-contentkey/)
- [.NET](/documentation/articles/media-services-dotnet-create-contentkey/)


媒体服务允许你创建新资产和传送加密的资产。**ContentKey** 提供对**资产**的安全访问。

创建新资产时（例如，[上载文件](/documentation/articles/media-services-rest-upload-files/)之前），可以指定以下加密选项：**StorageEncrypted**、**CommonEncryptionProtected** 或 **EnvelopeEncryptionProtected**。

将资产传送到你的客户端时，可以使用以下两个加密选项之一[将资产配置为动态加密](/documentation/articles/media-services-rest-configure-asset-delivery-policy/)：**DynamicEnvelopeEncryption** 或 **DynamicCommonEncryption**。

加密的资产必须与 **ContentKey** 关联。本文介绍如何创建内容密钥。

以下是用于生成内容密钥的常规步骤，你会将这些内容密钥与你想要进行加密的资产关联。

1. 随机生成一个 16 字节 AES 密钥（用于常规和信封加密）或 32 字节 AES 密钥（用于存储加密）。 

	这将成为你资产的内容密钥，这意味着该资产的所有关联文件在解密过程中需要使用同一内容密钥。 
2.	调用 [GetProtectionKeyId](https://msdn.microsoft.com/zh-cn/library/azure/jj683097.aspx#getprotectionkeyid) 和 [GetProtectionKey](https://msdn.microsoft.com/zh-cn/library/azure/jj683097.aspx#getprotectionkey) 方法来获取正确的 X.509 证书，必须使用该证书加密你的内容密钥。
3.	使用 X.509 证书的公钥来加密你的内容密钥。 

	媒体服务 .NET SDK 在加密时使用 RSA 和 OAEP。你可以参阅 [EncryptSymmetricKeyData 函数](https://github.com/Azure/azure-sdk-for-media-services/blob/dev/src/net/Client/Common/Common.FileEncryption/EncryptionUtils.cs)中的示例。
4.	创建一个使用密钥标识符和内容密钥计算得出的校验和值（基于 PlayReady AES 密钥校验和算法）。有关详细信息，请参阅位于[此处](http://www.microsoft.com/playready/documents/)的 PlayReady 标头对象文档的“PlayReady AES 密钥校验和算法”部分。

	下面的 .NET 示例将使用密钥标识符和明文内容密钥的 GUID 部分计算校验和。
	
		public static string CalculateChecksum(byte[] contentKey, Guid keyId)
		{
		    byte[] array = null;
		    using (AesCryptoServiceProvider aesCryptoServiceProvider = new AesCryptoServiceProvider())
		    {
		        aesCryptoServiceProvider.Mode = CipherMode.ECB;
		        aesCryptoServiceProvider.Key = contentKey;
		        aesCryptoServiceProvider.Padding = PaddingMode.None;
		        ICryptoTransform cryptoTransform = aesCryptoServiceProvider.CreateEncryptor();
		        array = new byte[16];
		        cryptoTransform.TransformBlock(keyId.ToByteArray(), 0, 16, array, 0);
		    }
		    byte[] array2 = new byte[8];
		    Array.Copy(array, array2, 8);
		    return Convert.ToBase64String(array2);
		}

5. 创建具有 **EncryptedContentKey**（转换为 base64 编码的字符串）、**ProtectionKeyId**、**ProtectionKeyType**、**ContentKeyType** 和在前面步骤中收到的**校验和**值的内容密钥。
6. 通过 $links 操作将 **ContentKey** 实体与你的 **资产**实体相关联。

请注意，本主题中省略了生成 AES 密钥、加密密钥以及计算校验和的示例。仅提供了演示如何与媒体服务进行交互的示例。


>[AZURE.NOTE] 使用媒体服务 REST API 时，需注意以下事项：
>
>访问媒体服务中的实体时，必须在 HTTP 请求中设置特定标头字段和值。有关详细信息，请参阅[媒体服务 REST API 开发的设置](/documentation/articles/media-services-rest-how-to-use/)。

>在成功连接到 https://media.chinacloudapi.cn 之后，你将接收到指定另一个媒体服务 URI 的 301 重定向。必须根据[使用 REST API 连接到媒体服务](/documentation/articles/media-services-rest-connect_programmatically/)中所述对新的 URI 执行后续调用。

##检索 ProtectionKeyId 
 

以下示例演示了如何检索证书的证书指纹 ProtectionKeyId，你在加密内容密钥时必须使用此指纹。执行此步骤以确保你的计算机已具备适当的证书。


请求：
	
	
	GET https://media.chinacloudapi.cn/api/GetProtectionKeyId?contentKeyType=0 HTTP/1.1
	MaxDataServiceVersion: 3.0;NetFx
	Accept: application/json
	Accept-Charset: UTF-8
	User-Agent: Microsoft ADO.NET Data Services
	Authorization: Bearer http%3a%2f%2fschemas.xmlsoap.org%2fws%2f2005%2f05%2fidentity%2fclaims%2fnameidentifier=juliakoams1&urn%3aSubscriptionId=zbbef702-2233-477b-9f16-bc4d3aa97387&http%3a%2f%2fschemas.microsoft.com%2faccesscontrolservice%2f2010%2f07%2fclaims%2fidentityprovider=https%3a%2f%2fwamsprodglobal001acs.accesscontrol.chinacloudapi.cn%2f&Audience=urn%3aWindowsAzureMediaServices&ExpiresOn=1423034908&Issuer=https%3a%2f%2fwamsprodglobal001acs.accesscontrol.chinacloudapi.cn%2f&HMACSHA256=7eSLe1GHnxgilr3F2FPCGxdL2%2bwy%2f39XhMPGY9IizfU%3d
	x-ms-version: 2.11
	Host: media.chinacloudapi.cn
	

响应：
	
	HTTP/1.1 200 OK
	Cache-Control: no-cache
	Content-Length: 139
	Content-Type: application/json;odata=minimalmetadata;streaming=true;charset=utf-8
	Server: Microsoft-IIS/8.5
	request-id: 2b6aa7a4-3a09-4b08-b581-26b55667f817
	x-ms-request-id: 2b6aa7a4-3a09-4b08-b581-26b55667f817
	X-Content-Type-Options: nosniff
	DataServiceVersion: 3.0;
	X-Powered-By: ASP.NET
	Strict-Transport-Security: max-age=31536000; includeSubDomains
	Date: Wed, 04 Feb 2015 02:42:52 GMT
	
	{"odata.metadata":"https://wamsbayclus001rest-hs.chinacloudapp.cn/api/$metadata#Edm.String","value":"7D9BB04D9D0A4A24800CADBFEF232689E048F69C"}

##检索 ProtectionKeyId 的 ProtectionKey

以下示例演示如何使用在上一步中收到的 ProtectionKeyId 来检索 X.509 证书。

请求：
		
	GET https://media.chinacloudapi.cn/api/GetProtectionKey?ProtectionKeyId='7D9BB04D9D0A4A24800CADBFEF232689E048F69C' HTTP/1.1
	MaxDataServiceVersion: 3.0;NetFx
	Accept: application/json
	Accept-Charset: UTF-8
	User-Agent: Microsoft ADO.NET Data Services
	Authorization: Bearer http%3a%2f%2fschemas.xmlsoap.org%2fws%2f2005%2f05%2fidentity%2fclaims%2fnameidentifier=juliakoams1&urn%3aSubscriptionId=zbbef702-e769-2233-9f16-bc4d3aa97387&http%3a%2f%2fschemas.microsoft.com%2faccesscontrolservice%2f2010%2f07%2fclaims%2fidentityprovider=https%3a%2f%2fwamsprodglobal001acs.accesscontrol.chinacloudapi.cn%2f&Audience=urn%3aWindowsAzureMediaServices&ExpiresOn=1423141026&Issuer=https%3a%2f%2fwamsprodglobal001acs.accesscontrol.chinacloudapi.cn%2f&HMACSHA256=lDBz5YXKiWe5L7eXOHsLHc9kKEUcUiFJvrNFFSksgkM%3d
	x-ms-version: 2.11
	x-ms-client-request-id: 78d1247a-58d7-40e5-96cc-70ff0dfa7382
	Host: media.chinacloudapi.cn
	


响应：
	
	HTTP/1.1 200 OK
	Cache-Control: no-cache
	Content-Length: 1227
	Content-Type: application/json;odata=minimalmetadata;streaming=true;charset=utf-8
	Server: Microsoft-IIS/8.5
	x-ms-client-request-id: 78d1247a-58d7-40e5-96cc-70ff0dfa7382
	request-id: 1523e8f3-8ed2-40fe-8a9a-5d81eb572cc8
	x-ms-request-id: 1523e8f3-8ed2-40fe-8a9a-5d81eb572cc8
	X-Content-Type-Options: nosniff
	DataServiceVersion: 3.0;
	X-Powered-By: ASP.NET
	Strict-Transport-Security: max-age=31536000; includeSubDomains
	Date: Thu, 05 Feb 2015 07:52:30 GMT
	
	{"odata.metadata":"https://wamsbayclus001rest-hs.chinacloudapp.cn/api/$metadata#Edm.String",
	"value":"MIIDSTCCAjGgAwIBAgIQqf92wku/HLJGCbMAU8GEnDANBgkqhkiG9w0BAQQFADAuMSwwKgYDVQQDEyN3YW1zYmx1cmVnMDAxZW5jcnlwdGFsbHNlY3JldHMtY2VydDAeFw0xMjA1MjkwNzAwMDBaFw0zMjA1MjkwNzAwMDBaMC4xLDAqBgNVBAMTI3dhbXNibHVyZWcwMDFlbmNyeXB0YWxsc2VjcmV0cy1jZXJ0MIIBIjANBgkqhkiG9w0BAQEFAAOCAQ8AMIIBCgKCAQEAzR0SEbXefvUjb9wCUfkEiKtGQ5Gc328qFPrhMjSo+YHe0AVviZ9YaxPPb0m1AaaRV4dqWpST2+JtDhLOmGpWmmA60tbATJDdmRzKi2eYAyhhE76MgJgL3myCQLP42jDusWXWSMabui3/tMDQs+zfi1sJ4Ch/lm5EvksYsu6o8sCv29VRwxfDLJPBy2NlbV4GbWz5Qxp2tAmHoROnfaRhwp6WIbquk69tEtu2U50CpPN2goLAqx2PpXAqA+prxCZYGTHqfmFJEKtZHhizVBTFPGS3ncfnQC9QIEwFbPw6E5PO5yNaB68radWsp5uvDg33G1i8IT39GstMW6zaaG7cNQIDAQABo2MwYTBfBgNVHQEEWDBWgBCOGT2hPhsvQioZimw8M+jOoTAwLjEsMCoGA1UEAxMjd2Ftc2JsdXJlZzAwMWVuY3J5cHRhbGxzZWNyZXRzLWNlcnSCEKn/dsJLvxyyRgmzAFPBhJwwDQYJKoZIhvcNAQEEBQADggEBABcrQPma2ekNS3Wc5wGXL/aHyQaQRwFGymnUJ+VR8jVUZaC/U/f6lR98eTlwycjVwRL7D15BfClGEHw66QdHejaViJCjbEIJJ3p2c9fzBKhjLhzB3VVNiLIaH6RSI1bMPd2eddSCqhDIn3VBN605GcYXMzhYp+YA6g9+YMNeS1b+LxX3fqixMQIxSHOLFZ1G/H2xfNawv0VikH3djNui3EKT1w/8aRkUv/AAV0b3rYkP/jA1I0CPn0XFk7STYoiJ3gJoKq9EMXhit+Iwfz0sMkfhWG12/XO+TAWqsK1ZxEjuC9OzrY7pFnNxs4Mu4S8iinehduSpY+9mDd3dHynNwT4="}

##创建 ContentKey 

检索到 X.509 证书并使用其公钥加密你的内容密钥后，请创建一个 **ContentKey** 实体并相应地设置其属性值。

创建内容密钥时必须设置的值之一是内容密钥类型。选择以下值之一。

    public enum ContentKeyType
    {
        /// <summary>
        /// Specifies a content key for common encryption.
        /// </summary>
        /// <remarks>This is the default value.</remarks>
        CommonEncryption = 0,

        /// <summary>
        /// Specifies a content key for storage encryption.
        /// </summary>
        StorageEncryption = 1,

        /// <summary>
        /// Specifies a content key for configuration encryption.
        /// </summary>
        ConfigurationEncryption = 2,

        /// <summary>
        /// Specifies a content key for Envelope encryption.  Only used internally.
        /// </summary>
        EnvelopeEncryption = 4
    }


以下示例演示了如何创建一个 **ContentKey**，其中它的 **ContentKeyType** 设置为存储加密 ("1") 且 **ProtectionKeyType** 设置为“0”，以指示保护密钥 ID 是 X.509 证书指纹。


请求

	POST https://media.chinacloudapi.cn/api/ContentKeys HTTP/1.1
	Content-Type: application/json
	DataServiceVersion: 1.0;NetFx
	MaxDataServiceVersion: 3.0;NetFx
	Accept: application/json
	Accept-Charset: UTF-8
	User-Agent: Microsoft ADO.NET Data Services
	Authorization: Bearer http%3a%2f%2fschemas.xmlsoap.org%2fws%2f2005%2f05%2fidentity%2fclaims%2fnameidentifier=juliakoams1&urn%3aSubscriptionId=zbbef702-2233-477b-9f16-bc4d3aa97387&http%3a%2f%2fschemas.microsoft.com%2faccesscontrolservice%2f2010%2f07%2fclaims%2fidentityprovider=https%3a%2f%2fwamsprodglobal001acs.accesscontrol.chinacloudapi.cn%2f&Audience=urn%3aWindowsAzureMediaServices&ExpiresOn=1423034908&Issuer=https%3a%2f%2fwamsprodglobal001acs.accesscontrol.chinacloudapi.cn%2f&HMACSHA256=7eSLe1GHnxgilr3F2FPCGxdL2%2bwy%2f39XhMPGY9IizfU%3d
	x-ms-version: 2.11
	Host: media.chinacloudapi.cn
	{
	"Name":"ContentKey",
	"ProtectionKeyId":"7D9BB04D9D0A4A24800CADBFEF232689E048F69C", 
	"ContentKeyType":"1", 
	"ProtectionKeyType":"0",
	"EncryptedContentKey":"your encrypted content key",
	"Checksum":"calculated checksum"
	}


响应：
	
	HTTP/1.1 201 Created
	Cache-Control: no-cache
	Content-Length: 777
	Content-Type: application/json;odata=minimalmetadata;streaming=true;charset=utf-8
	Location: https://media.chinacloudapi.cn/api/ContentKeys('nb%3Akid%3AUUID%3A9c8ea9c6-52bd-4232-8a43-8e43d8564a99')
	Server: Microsoft-IIS/8.5
	request-id: 76e85e0f-5cf1-44cb-b689-b3455888682c
	x-ms-request-id: 76e85e0f-5cf1-44cb-b689-b3455888682c
	X-Content-Type-Options: nosniff
	DataServiceVersion: 3.0;
	X-Powered-By: ASP.NET
	Strict-Transport-Security: max-age=31536000; includeSubDomains
	Date: Wed, 04 Feb 2015 02:37:46 GMT
	
	{"odata.metadata":"https://wamsbayclus001rest-hs.chinacloudapp.cn/api/$metadata#ContentKeys/@Element",
	"Id":"nb:kid:UUID:9c8ea9c6-52bd-4232-8a43-8e43d8564a99","Created":"2015-02-04T02:37:46.9684379Z",
	"LastModified":"2015-02-04T02:37:46.9684379Z",
	"ContentKeyType":1,
	"EncryptedContentKey":"your encrypted content key",
	"Name":"ContentKey",
	"ProtectionKeyId":"7D9BB04D9D0A4A24800CADBFEF232689E048F69C",
	"ProtectionKeyType":0,
	"Checksum":"calculated checksum"}

##将 ContentKey 与资产关联

创建 ContentKey 后，使用 $links 操作将其与你的资产关联，如以下示例所示：
	
请求：
	
	POST https://media.chinacloudapi.cn/api/Assets('nb%3Acid%3AUUID%3Afbd7ce05-1087-401b-aaae-29f16383c801')/$links/ContentKeys HTTP/1.1
	DataServiceVersion: 1.0;NetFx
	MaxDataServiceVersion: 3.0;NetFx
	Accept: application/json
	Accept-Charset: UTF-8
	Content-Type: application/json
	Authorization: Bearer http%3a%2f%2fschemas.xmlsoap.org%2fws%2f2005%2f05%2fidentity%2fclaims%2fnameidentifier=juliakoams1&urn%3aSubscriptionId=zbbef702-2233-477b-9f16-bc4d3aa97387&http%3a%2f%2fschemas.microsoft.com%2faccesscontrolservice%2f2010%2f07%2fclaims%2fidentityprovider=https%3a%2f%2fwamsprodglobal001acs.accesscontrol.chinacloudapi.cn%2f&Audience=urn%3aWindowsAzureMediaServices&ExpiresOn=1423141026&Issuer=https%3a%2f%2fwamsprodglobal001acs.accesscontrol.chinacloudapi.cn%2f&HMACSHA256=lDBz5YXKiWe5L7eXOHsLHc9kKEUcUiFJvrNFFSksgkM%3d
	x-ms-version: 2.11
	Host: media.chinacloudapi.cn

	
	{"uri":"https://wamsbayclus001rest-hs.chinacloudapp.cn/api/ContentKeys('nb%3Akid%3AUUID%3A01e6ea36-2285-4562-91f1-82c45736047c')"}

响应：

	HTTP/1.1 204 No Content 



<!---HONumber=Mooncake_0620_2016-->