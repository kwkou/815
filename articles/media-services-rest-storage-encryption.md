<properties 
	pageTitle="使用 AMS REST API 通过存储空间加密来加密内容" 
	description="了解如何使用 AMS REST API 通过存储空间加密来加密内容。" 
	services="media-services" 
	documentationCenter="" 
	authors="Juliako" 
	manager="erikre" 
	editor=""/>

<tags 
	ms.service="media-services" 
 	ms.date="05/02/2016"   
	wacn.date="6/20/2016"/>


#使用 AMS REST API 通过存储空间加密来加密内容

强烈建议通过 AES-256 位加密在本地加密内容，然后将其上传到 Azure 存储空间中以加密形式静态存储相关内容。

本文概述了 AMS 存储空间加密并演示了如何上传存储空间加密的内容：

- 创建内容密钥。
- 创建资产。创建资产时，请将 AssetCreationOption 设置为 StorageEncryption。

	加密的资产必须与内容密钥关联。
- 将内容密钥链接到资产。  
- 对 AssetFile 实体设置加密相关的参数。
 
>[AZURE.NOTE]如果要传送存储加密资产，则必须配置资产的传送策略。在流式传输资产之前，流式处理服务器会删除存储加密，然后再使用指定的传送策略流式传输你的内容。有关详细信息，请参阅[配置资产传送策略](/documentation/articles/media-services-rest-configure-asset-delivery-policy/)。


>[AZURE.NOTE] 使用媒体服务 REST API 时，需注意以下事项：
>
>访问媒体服务中的实体时，必须在 HTTP 请求中设置特定标头字段和值。有关详细信息，请参阅[媒体服务 REST API 开发的设置](/documentation/articles/media-services-rest-how-to-use/)。

>在成功连接到 https://media.chinacloudapi.cn 之后，你将接收到指定另一个媒体服务 URI 的 301 重定向。必须按[使用 REST API 连接到媒体服务](/documentation/articles/media-services-rest-connect-programmatically/)中所述对新的 URI 执行后续调用。

##存储空间加密概述 

AMS 存储空间加密将 **AES-CTR** 模式加密应用于整个文件。AES-CTR 模式是一分组加密，无需填充便可对任意长度的数据进行加密。它采用 AES 算法加密计数器分组，然后使用要加密或解密的数据对 AES 的输出执行异或运算。通过将 InitializationVector 的值复制到计数器值的 0 到 7 字节来构造所用的计数器分组，而计数器值的 8 到 15 字节设置为零。在长度为 16 字节的计数器分组中，8 到 15 字节（即，最少有效字节）用作简单的 64 位无符号整数，对于所处理数据的每个后续分组，该整数都会递增 1 并保留网络字节顺序。请注意，如果此整数达到最大值 (0xFFFFFFFFFFFFFFFF)，则递增会将分组计数器重置为零（8 到 15 字节），且不会影响其他 64 位计数器（即 0 到 7 字节）。为了维护 AES-CTR 模式加密的安全性，每个内容密钥的给定密钥标识符的 InitializationVector 值对每个文件必须是唯一的，且文件长度应小于 2^64 分组。这是为了确保计数器值永远不会重复用于给定密钥。有关 CTR 模式的详细信息，请参阅[此 wiki 页](https://en.wikipedia.org/wiki/Block_cipher_mode_of_operation#CTR)（wiki 文章使用术语“Nonce”取代“InitializationVector”）。

使用“存储空间加密”通过 AES-256 位加密在本地加密明文内容，然后将其上传到 Azure 存储空间中以加密形式静态存储相关内容。受存储加密保护的资产将在编码前自动解密并放入经过加密的文件系统中，并可选择在重新上载为新的输出资产前重新加密。存储加密的主要用例是在磁盘上通过静态增强加密来保护高品质的输入媒体文件。

若要传送存储加密资产，必须配置资产的传送策略，以使媒体服务了解要如何传送你的内容。在流式传输资产之前，流式处理服务器会删除存储加密，然后再使用指定的传传送策略（例如 AES、通用加密或无加密）流式传输你的内容。

##创建用于加密的 ContentKey

加密的资产必须与存储空间加密密钥关联。在创建资产文件前，必须创建用于加密的内容密钥。本节介绍如何创建内容密钥。

以下是用于生成内容密钥的常规步骤，你会将这些内容密钥与你想要进行加密的资产关联。

1. 对于存储空间加密，随机生成一个 32 字节的 AES 密钥。 

	这将成为你资产的内容密钥，这意味着该资产的所有关联文件在解密过程中需要使用同一内容密钥。 
2.	调用 [GetProtectionKeyId](https://msdn.microsoft.com/zh-cn/library/azure/jj683097.aspx#getprotectionkeyid) 和 [GetProtectionKey](https://msdn.microsoft.com/zh-cn/library/azure/jj683097.aspx#getprotectionkey) 方法来获取正确的 X.509 证书，必须使用该证书加密你的内容密钥。
3.	使用 X.509 证书的公钥来加密你的内容密钥。 

	媒体服务 .NET SDK 在加密时使用 RSA 和 OAEP。你可以参阅 [EncryptSymmetricKeyData 函数](https://github.com/Azure/azure-sdk-for-media-services/blob/dev/src/net/Client/Common/Common.FileEncryption/EncryptionUtils.cs)中的 .NET 示例。
4.	创建使用密钥标识符和内容密钥计算的校验和值。下面的 .NET 示例将使用密钥标识符和明文内容密钥的 GUID 部分计算校验和。
	

        public static string CalculateChecksum(byte[] contentKey, Guid keyId)
        {
            const int ChecksumLength = 8;
            const int KeyIdLength = 16;

            byte[] encryptedKeyId = null;

            // Checksum is computed by AES-ECB encrypting the KID
            // with the content key.
            using (AesCryptoServiceProvider rijndael = new AesCryptoServiceProvider())
            {
                rijndael.Mode = CipherMode.ECB;
                rijndael.Key = contentKey;
                rijndael.Padding = PaddingMode.None;

                ICryptoTransform encryptor = rijndael.CreateEncryptor();
                encryptedKeyId = new byte[KeyIdLength];
                encryptor.TransformBlock(keyId.ToByteArray(), 0, KeyIdLength, encryptedKeyId, 0);
            }

            byte[] retVal = new byte[ChecksumLength];
            Array.Copy(encryptedKeyId, retVal, ChecksumLength);

            return Convert.ToBase64String(retVal);
        }


5. 创建具有 **EncryptedContentKey**（转换为 base64 编码的字符串）、**ProtectionKeyId**、**ProtectionKeyType**、**ContentKeyType** 和在前面步骤中收到的**校验和**值的内容密钥。

	
	对于存储空间加密，应在请求正文中包括以下属性。
	 
	请求正文属性 | 说明
	---|---
	ID | 我们使用以下格式自行生成的 ContentKey ID：“nb:kid:UUID:<NEW GUID>”。
	ContentKeyType | 这是此内容密钥的内容密钥类型（为整数）。我们为存储加密传递了值 1。
	EncryptedContentKey | 我们创建一个新的内容密钥值，这是一个 256 位（32 字节）的值。该密钥通过使用存储空间加密 X.509 证书进行加密，我们通过执行 GetProtectionKeyId 和 GetProtectionKey 方法的 HTTP GET 请求从 Azure 媒体服务中检索该证书。有关示例，请参阅下面的 .NET 代码：[此处](https://github.com/Azure/azure-sdk-for-media-services/blob/dev/src/net/Client/Common/Common.FileEncryption/EncryptionUtils.cs)定义的 **EncryptSymmetricKeyData** 方法 。
	ProtectionKeyId | 这是存储空间加密 X.509 证书的保护密钥 ID，用于加密内容密钥。
	ProtectionKeyType | 这是用于加密内容密钥的保护密钥的加密类型。对于我们的示例，此值为 StorageEncryption(1)。
	校验和 |内容密钥的 MD5 计算的校验和。它通过使用内容密钥加密内容 ID 计算得出。此示例代码演示了如何计算校验和。
	

###检索 ProtectionKeyId 
 

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
	
	{"odata.metadata":"https://wamsbayclus001rest-hs.cloudapp.net/api/$metadata#Edm.String","value":"7D9BB04D9D0A4A24800CADBFEF232689E048F69C"}

###检索 ProtectionKeyId 的 ProtectionKey

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
	
	{"odata.metadata":"https://wamsbayclus001rest-hs.cloudapp.net/api/$metadata#Edm.String",
	"value":"MIIDSTCCAjGgAwIBAgIQqf92wku/HLJGCbMAU8GEnDANBgkqhkiG9w0BAQQFADAuMSwwKgYDVQQDEyN3YW1zYmx1cmVnMDAxZW5jcnlwdGFsbHNlY3JldHMtY2VydDAeFw0xMjA1MjkwNzAwMDBaFw0zMjA1MjkwNzAwMDBaMC4xLDAqBgNVBAMTI3dhbXNibHVyZWcwMDFlbmNyeXB0YWxsc2VjcmV0cy1jZXJ0MIIBIjANBgkqhkiG9w0BAQEFAAOCAQ8AMIIBCgKCAQEAzR0SEbXefvUjb9wCUfkEiKtGQ5Gc328qFPrhMjSo+YHe0AVviZ9YaxPPb0m1AaaRV4dqWpST2+JtDhLOmGpWmmA60tbATJDdmRzKi2eYAyhhE76MgJgL3myCQLP42jDusWXWSMabui3/tMDQs+zfi1sJ4Ch/lm5EvksYsu6o8sCv29VRwxfDLJPBy2NlbV4GbWz5Qxp2tAmHoROnfaRhwp6WIbquk69tEtu2U50CpPN2goLAqx2PpXAqA+prxCZYGTHqfmFJEKtZHhizVBTFPGS3ncfnQC9QIEwFbPw6E5PO5yNaB68radWsp5uvDg33G1i8IT39GstMW6zaaG7cNQIDAQABo2MwYTBfBgNVHQEEWDBWgBCOGT2hPhsvQioZimw8M+jOoTAwLjEsMCoGA1UEAxMjd2Ftc2JsdXJlZzAwMWVuY3J5cHRhbGxzZWNyZXRzLWNlcnSCEKn/dsJLvxyyRgmzAFPBhJwwDQYJKoZIhvcNAQEEBQADggEBABcrQPma2ekNS3Wc5wGXL/aHyQaQRwFGymnUJ+VR8jVUZaC/U/f6lR98eTlwycjVwRL7D15BfClGEHw66QdHejaViJCjbEIJJ3p2c9fzBKhjLhzB3VVNiLIaH6RSI1bMPd2eddSCqhDIn3VBN605GcYXMzhYp+YA6g9+YMNeS1b+LxX3fqixMQIxSHOLFZ1G/H2xfNawv0VikH3djNui3EKT1w/8aRkUv/AAV0b3rYkP/jA1I0CPn0XFk7STYoiJ3gJoKq9EMXhit+Iwfz0sMkfhWG12/XO+TAWqsK1ZxEjuC9OzrY7pFnNxs4Mu4S8iinehduSpY+9mDd3dHynNwT4="}

### 创建内容密钥 

检索到 X.509 证书并使用其公钥加密你的内容密钥后，请创建一个 **ContentKey** 实体并相应地设置其属性值。

创建内容密钥时必须设置的值之一是内容密钥类型。对于存储空间加密，值为“1”。

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
	
	{"odata.metadata":"https://wamsbayclus001rest-hs.cloudapp.net/api/$metadata#ContentKeys/@Element",
	"Id":"nb:kid:UUID:9c8ea9c6-52bd-4232-8a43-8e43d8564a99","Created":"2015-02-04T02:37:46.9684379Z",
	"LastModified":"2015-02-04T02:37:46.9684379Z",
	"ContentKeyType":1,
	"EncryptedContentKey":"your encrypted content key",
	"Name":"ContentKey",
	"ProtectionKeyId":"7D9BB04D9D0A4A24800CADBFEF232689E048F69C",
	"ProtectionKeyType":0,
	"Checksum":"calculated checksum"}

## 创建资产

以下示例说明了如何创建资产。

**HTTP 请求**

	POST https://media.chinacloudapi.cn/api/Assets HTTP/1.1
	Content-Type: application/json
	DataServiceVersion: 1.0;NetFx
	MaxDataServiceVersion: 3.0;NetFx
	Accept: application/json
	Accept-Charset: UTF-8
	Authorization: Bearer http%3a%2f%2fschemas.xmlsoap.org%2fws%2f2005%2f05%2fidentity%2fclaims%2fnameidentifier=amstestaccount001&urn%3aSubscriptionId=z7f09258-6753-2233-b1ae-193798e2c9d8&http%3a%2f%2fschemas.microsoft.com%2faccesscontrolservice%2f2010%2f07%2fclaims%2fidentityprovider=https%3a%2f%2fwamsprodglobal001acs.accesscontrol.chinacloudapi.cn%2f&Audience=urn%3aWindowsAzureMediaServices&ExpiresOn=1421640053&Issuer=https%3a%2f%2fwamsprodglobal001acs.accesscontrol.chinacloudapi.cn%2f&HMACSHA256=vlG%2fPYdFDMS1zKc36qcFVWnaNh07UCkhYj3B71%2fk1YA%3d
	x-ms-version: 2.11
	Host: media.chinacloudapi.cn
	
	{"Name":"BigBuckBunny" "Options":1}


**HTTP 响应**

如果成功，将返回以下响应：
	
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
	Strict-Transport-Security: max-age=31536000; includeSubDomains
	Date: Sun, 18 Jan 2015 22:06:40 GMT
	{  
	   "odata.metadata":"https://wamsbayclus001rest-hs.cloudapp.net/api/$metadata#Assets/@Element",
	   "Id":"nb:cid:UUID:9bc8ff20-24fb-4fdb-9d7c-b04c7ee573a1",
	   "State":0,
	   "Created":"2015-01-18T22:06:40.6010903Z",
	   "LastModified":"2015-01-18T22:06:40.6010903Z",
	   "AlternateId":null,
	   "Name":"BigBuckBunny.mp4",
	   "Options":1,
	   "Uri":"https://storagetestaccount001.blob.core.chinacloudapi.cn/asset-9bc8ff20-24fb-4fdb-9d7c-b04c7ee573a1",
	   "StorageAccountName":"storagetestaccount001"
	}
	
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

	
	{"uri":"https://wamsbayclus001rest-hs.cloudapp.net/api/ContentKeys('nb%3Akid%3AUUID%3A01e6ea36-2285-4562-91f1-82c45736047c')"}

响应：

	HTTP/1.1 204 No Content 

##创建 AssetFile

[AssetFile](http://msdn.microsoft.com/zh-cn/library/azure/hh974275.aspx) 实体表示 blob 容器中存储的视频或音频文件。一个资产文件始终与一个资产关联，而一个资产则可能包含一个或多个资产文件。如果资产文件对象未与 BLOB 容器中的数字文件关联，则媒体服务 Encoder 任务将失败。

请注意，**AssetFile** 实例和实际媒体文件是两个不同的对象。AssetFile 实例包含有关媒体文件的元数据，而媒体文件包含实际媒体内容。

将数字媒体文件上传到 blob 容器后，需要使用 **MERGE** HTTP 请求来更新 AssetFile 中有关媒体文件的信息（本主题中未展示）。

**HTTP 请求**

	POST https://media.chinacloudapi.cn/api/Files HTTP/1.1
	Content-Type: application/json
	DataServiceVersion: 1.0;NetFx
	MaxDataServiceVersion: 3.0;NetFx
	Accept: application/json
	Accept-Charset: UTF-8
	Authorization: Bearer http%3a%2f%2fschemas.xmlsoap.org%2fws%2f2005%2f05%2fidentity%2fclaims%2fnameidentifier=amstestaccount001&urn%3aSubscriptionId=z7f09258-6753-4ca2-2233-193798e2c9d8&http%3a%2f%2fschemas.microsoft.com%2faccesscontrolservice%2f2010%2f07%2fclaims%2fidentityprovider=https%3a%2f%2fwamsprodglobal001acs.accesscontrol.chinacloudapi.cn%2f&Audience=urn%3aWindowsAzureMediaServices&ExpiresOn=1421640053&Issuer=https%3a%2f%2fwamsprodglobal001acs.accesscontrol.chinacloudapi.cn%2f&HMACSHA256=vlG%2fPYdFDMS1zKc36qcFVWnaNh07UCkhYj3B71%2fk1YA%3d
	x-ms-version: 2.11
	Host: media.chinacloudapi.cn
	Content-Length: 164
	
	{  
	   "IsEncrypted":"true",
       "EncryptionScheme" : "StorageEncryption", 
       "EncryptionVersion" : "1.0",       
       "EncryptionKeyId" : "nb:kid:UUID:32e6efaf-5fba-4538-b115-9d1cefe43510",
       "InitializationVector" : "397304628502661816</d:InitializationVector",
       "Options":0,
	   "IsPrimary":"false",
	   "MimeType":"video/mp4",
	   "Name":"BigBuckBunny.mp4",
	   "ParentAssetId":"nb:cid:UUID:9bc8ff20-24fb-4fdb-9d7c-b04c7ee573a1"
	}

**HTTP 响应**

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
	   "EncryptionVersion": "1.0",
	   "EncryptionScheme": "StorageEncryption",
	   "IsEncrypted":true,
	   "EncryptionKeyId":"nb:kid:UUID:32e6efaf-5fba-4538-b115-9d1cefe43510",
	   "InitializationVector":"397304628502661816</d:InitializationVector",
	   "IsPrimary":false,
	   "LastModified":"2015-01-19T00:34:08.1934137Z",
	   "Created":"2015-01-19T00:34:08.1934137Z",
	   "MimeType":"video/mp4",
	   "ContentChecksum":null
	}

<!---HONumber=Mooncake_0613_2016-->