<properties
    linkid="dev-net-common-tasks-cdn"
    urlDisplayName="CDN"
    pageTitle="Overview of Azure CDN in China – Azure feature guide"
    metaKeywords="Azure CDN, Azure CDN, Azure blobs, Azure caching, Azure add-ons, CDN, CDN acceleration, CDN services, mainstream CDN, multi-scenario acceleration, free CDN, CDN website acceleration, website acceleration, webpage acceleration, static acceleration, download acceleration, VOD acceleration, live streaming media acceleration, cloud services, storage account, cache refresh, return to source, cloud acceleration, acceleration effects, node, traffic, CNAME, bandwidth, network speed, anti-theft chain, https acceleration, low-cost bandwidth, access acceleration, small file acceleration, download acceleration, large file acceleration, streaming media acceleration, HTTPS secure acceleration, cache refresh, content preloading, anti-theft chain, log download, CDN technical files, CDN help files, CDN FAQ"
    description="Learn the overview of Azure CDN, advantages, typical scenarios, and key features."
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
    ms.date="4/7/2016"
    wacn.date="4/7/2016"
    wacn.lang="en" />

> [AZURE.LANGUAGE]
- [中文](/documentation/articles/cdn-security/)
- [English](/documentation/articles/cdn-enus-security/)

#<a name="azure-cdn"></a>Azure CDN Secure Transport Mechanism


Content hijacking means that an operator/link provider/hacker connects a bypass device to the router to detect HTTP requests passing through, identify target HTTP requests using a particular hijack strategy, and send a 302 response to the client before the source station feeds back data, so that the client is forced to jump to other server request content, i.e. the URL request is hijacked.

Criminals may replace the original client user request content with tempered content for nefarious purposes, such as to illegally profit, spread malicious information, or compete by improper means.

Content hijacking or content tampering causes end users to experience access errors or download the wrong files.

CDN-accelerated end-to-end access for end-user CDN node access -> CDN network file transfer -> CDN node back to source. The Azure CDN ensures that files within the CDN are securely transferred. Azure CDN also uses certificate encryption, intelligent routing, and URI encryption for transfers between end users accessing a CDN node and the return-to-source CDN node in order to prevent and handle content hijacking and tampering. The security of the source station itself depends upon the client taking the corresponding measures.

+ [Secure file transport in the Azure CDN](#step1)
+ [Secure transfers between end users accessing a CDN node and the return-to-source CDN node](#step2)

##**Secure file transport in the Azure CDN**<a id="step1"></a>
Azure CDN acceleration ensures that files within the CDN are securely transferred and CDN node files are securely stored.

###<a name="azure-cdn"></a>**Secure file transport in the Azure CDN**

Content obtained by the CDN is transferred within the CDN using the Azure CDN operator’s proprietary transport protocol, which encrypts data packets for transfer.

###<a name="cdn"></a>**CDN node file storage security**

1. CDN node file caching uses a special hash algorithm optimized by Azure CDN partners to map files to irregular directories. It is impossible to find the storage path for the file without the Azure CDN partner’s hash algorithm. This method ensures that documents are both secure and invisible, preventing illegitimate users from retrieving stored content or tampering with cached content.

2.  Azure CDN partners use encrypted storage for file content stored on nodes, so even if the file storage path is found, the stored files are still encrypted. Without knowing the file encryption algorithm, even if the file content is accessed, it will simply be rendered unrecognizable. This stops criminals from achieving their illegal objectives by means of content tampering.

##**Secure transfers between end users accessing a CDN node and the return-to-source CDN node**<a id="step2"></a>

###<a name="https"></a>**HTTPS acceleration**

The request encryption scheme is the HTTPS acceleration solution. HTTPS is a network protocol built using the SSL and HTTP protocols that can perform accelerated transfers and authentication. The creation of an encrypted communication channel between the server and the client ensures that data will not be intercepted or hacked during the online transfer process, and also makes hijacking impossible.

**HTTPS acceleration access modes**

Azure CDN partners apply for SSL certificates for clients and deploy them to the CDN node -> End-user initiates HTTPS connection request -> Request data is returned to the user if the SSL handshake is successful.

If you need to enable HTTPS acceleration, please see [Azure CDN HTTPS Acceleration Service](/documentation/articles/cdn-enus-https-how-to/).

###<a name=""></a>**Intelligent routing options to prevent hijacking**

Smaller operators generally use certain probabilities of interception for content hijacking, and generally perform hijacks using specific links. If the client has multiple sources, this is equivalent to having multiple return-to-source paths. Moreover, the probability of operator hijacks occurring simultaneously on multiple links is far lower than for a single link. Based on this feature, when the return-to-source CDN is hijacked, a corresponding return-to-source path is selected or the next return-to-source path tried by determining the legitimacy of the location address for the 302 jump. Intelligently determining return-to-source paths massively reduces the risk of content hijacking.

**Access mode selected for intelligent anti-hijack routing ->** Client initiates request for CDN node access -> CDN node not cached -> CDN node return-to-source request A -> return-to-source 302 jump occurs -> the legitimacy of the 302 jump is judged and determined to be a hijack -> the return-to-source request automatically tries the next source B -> source B responds normally.

###<a name="uri"></a>**URI encryption**

The Azure CDN’s URI encryption anti-hijacking features work by performing a certain form of encryption on URIs, so that the URI corresponding to a particular file is continually changing, making it impossible for operators to hijack, thereby effectively reducing the incidence of such activity.

In situations that require source station cooperation or where there is a [source character missing] side environment, the source station or app client encrypts request URIs using an encryption key agreed with the Azure CDN partner.

**Encryption principles**

1. Client-side encryption:

    Original URL: Http://<domain name>[:<port>]/URI?a=1&b=2

    Performing DES encryption on the URL: DES (current time + encryption key + path, encryption key) = DES encrypted string

    Encrypted URL: http://<domain name>[:<port>]/<DES encrypted string>?a=1&b=2

2. CDN reverse decryption

    The CDN reverse-decrypts the source station URI using the agreed encryption key, splices it to produce the original URL, and then continues with the subsequent processing based on the original URL.

>**Explanation**

1. The value of the encryption key must be agreed between the Azure CDN partner and the customer
2. As one of the encryption factors uses the time that the request was initiated, a different request URL will be sent for the same file at different times, thereby achieving customization of URLs.
3. If the customer’s file is in APK format, as mobile operating systems may verify the file’s extension name before downloading the APK, the extension name (e.g. APK) of the previous file must be added to the final encrypted URL (i.e. after the DES encrypted string), as well as any other query_string values (if necessary).

**Handling exceptions**

In exceptional circumstances, for example where the form of the request URL is not consistent with the encryption format, decryption fails, or the key obtained after decryption is different from the key used for decryption, the Azure CDN will by default regard this as an illegal request and return a 403 Forbidden error.

**Access modes**

The source station encrypts the requested URI with the agreed encryption algorithm and key -> the user-initiated request reaches the CDN node -> the CDN node performs encryption -> response is sent to the client

<!--HONumber=May17_HO3-->