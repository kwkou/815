<properties linkid="dev-net-common-tasks-cdn" urlDisplayName="CDN" pageTitle="Azure CDN how to create https endpoint-EN" metaKeywords="Azure CDN, Azure CDN, Azure blobs, Azure caching, Azure add-ons, CDN, CDN acceleration, CDN service, mainstream CDN, multi-scenario acceleration, free CDN, CDN website acceleration, website acceleration, webpage acceleration, static acceleration, download acceleration, VOD acceleration, streaming media webcast acceleration, cloud service,  storage account, cache refresh, return to origin, cloud acceleration, acceleration results, node, traffic, CNAME, bandwidth, network speed, anti-theft chain, https acceleration, low-cost bandwidth, access acceleration, small file acceleration, download acceleration, large file acceleration, streaming media acceleration, HTTPS secure acceleration, cache refresh, content pre-loading, anti-theft chain, log download, CDN technical documentation, CDN help files, CDN FAQs" description="How to create HTTPS CDN endpoint." metaCanonical="" services="" documentationCenter=".NET" title="" authors="" solutions="" manager="" editor="" />
<tags ms.service="cdn_en"
    ms.date="7/7/2016"
    wacn.date="7/7/2016"
    wacn.lang="en"
    />
> [AZURE.LANGUAGE]
- [中文](/documentation/articles/cdn-https-how-to/)
- [English](/documentation/articles/cdn-enus-https-how-to/) 

# Azure CDN HTTPS acceleration service


## Request access
The Azure CDN HTTPS acceleration service is only available to paid Azure users.

1. **To apply for access:** Please contact the [Azure technical support team](https://www.azure.cn/support/contact/). You will need to provide the Azure subscription ID that you want to use the HTTPS acceleration service for.


2. **Self-service creation:** Once the Azure CDN team receives and approves your access request, they will start the HTTPS acceleration service for the Azure subscription you provided. You can then log into the Azure Management Portal to complete the self-service creation process. Please refer to the instructions for the self-service creation process.

    ![][1]


3. **SSL certificate application and configuration:** Once your configuration request is received, the Azure CDN will apply for an SSL certificate on your behalf (see the notes for details of certificate types).
    > **Notes**

    > The application and configuration for this certificate takes around ***5 working days***. You will also need to cooperate during the application process, so that the certificate issuer can confirm ownership of the domain name. Details of this confirmation process are given later in this article.

4. **When the configuration is finished and takes effect:** Once the configuration process is complete, you can complete the final setup of the CNAME record as you would to create services with other types of acceleration. Finally, you can complete any other associated management and configuration tasks through the unified Azure CDN Management Portal.


## Self-service creation process
1. Once you have finished creating the HTTPS acceleration type on the Azure Management Portal, press the “Management” button shown in the image below to jump to the Azure CDN Management Portal, from where you can complete the subsequent access procedure.

	![][2]

	![][3]

2. In the Azure CDN Management Portal, under “Domain name management”, select the HTTPS acceleration domain name that you need to configure, and then press “View HTTPS configuration status”.

	![][4]

3. You will then be able to see the 5 steps that are required to complete the whole HTTPS access configuration process. This step (Step 2: “Submit certificate application”) requires you to provide the following additional information as prompted by the interface:

	**Estimated bandwidth**: The estimated peak bandwidth required.

	**Estimated average file size**: The estimated average size of the files for which cache acceleration is required.

	**Acceleration demand time**: Specify whether the demand for HTTPS acceleration is long term or short term.

	**Access port for client access to CDN node**: Specify how the client you wish to enable will access the CDN node.

	1) Only enable HTTPS access (HTTP access is forbidden)

	2) Enable HTTP and HTTPS access

	3) Always redirect HTTP access to HTTPS access

	**Return-to-source port for CDN node access to the source station**: Specify how the CDN node can return to origin source.

		1) Only use HTTP to return to source

		2) Only use HTTPS to return to source

		3) Use both HTTP and HTTPS to return to source

	**Test URL**: Enter a URL which can subsequently be used to check access. You must ensure that this URL on the source station is accessible.

4. Once you have filled out all the relevant information for the previous step, press the “Confirm” button to complete the operation. To view the HTTPS configuration status again later:

	![][5]

	Once the Azure CDN service provider’s back end confirms all the information submitted and also submitted the official certificate application to the certificate issuer, you can proceed to the interface for Step 3 as shown below:

	![][6]

5. The next step is the most critical step in the certificate application process, during which the user needs to confirm ownership of the domain name. As the Azure CDN service provider submits the certificate application to the certificate issuer on the user’s behalf, the certificate issuer will confirm domain name ownership with the end user during this process. In order to confirm, **the domain name owner must complete final confirmation in the manner specified in the email sent by the domain name issuer**.

	There are two methods of receiving the email:

	1) **Default method**: Once the certificate issuer receives the request, the domain name  ownership confirmation email will be sent by default to the email address associated with the acceleration domain name (see the screenshot above for details) as soon as possible. If you choose to obtain the confirmation email by this method, you can go directly to the next step by clicking on the “Confirm” button.
		
	

	2) **DNS TXT method**: If you are unable to sign in to the email account above to complete confirmation of the domain name ownership, you can complete the confirmation process **by creating a DNS TXT record** as shown in the image below. This method may take a little longer to complete.

	![][7]

    Once you have created the DNS TXT log, you can check it using the following commands in the Windows operating system : nslookup -qt=txt www.cdn.test.com

	You can then sign in to the email account specified in the DNS TXT log, just as you would for the default method, in order to complete the subsequent domain name ownership verification process.

	Once you have successfully created the corresponding DNS TXT log in this step, click on the **Confirm** button for this step and proceed to the next step.

	![][9]

	

6. The user can complete the domain name ownership verification process by accessing the email.
	
	Once the Azure CDN back end confirms which method of receiving the confirmation email you have chosen, you will see the following interface:

	![][11]

	At this point, you can proceed to the corresponding email account to finish verifying domain name ownership.

	Email subject: Please validate ownership of your domain www.cdn.test.com -- DigiCert order 00123456

	Email body:

	![][14]

	You need to click on the confirmation link in the email to complete confirmation of domain name ownership.

	After this, you need to click on “Complete” in the interface for Step 4 to finally complete confirmation of the entire domain name.

7. Next, the entire configuration process will move into the final step, as shown in the image below:

	![][12]

	In this step, after the Azure CDN back end receives the corresponding certificate from the certificate issuer, it will take a certain amount of time to complete the final configuration tasks. After this, you will see the final completion interface:

	![][13]

	Lastly, you can complete the final CNAME configuration process through your domain name service provider just as you would to create accelerated domain names with other acceleration types; direct the user-defined accelerated domain name to the CDN domain name provided by the Azure CDN platform, which should have an extension similar to **.mschcdn.com**, in order to enable the whole configuration to take effect.




## Notes

### About the SSL certificates used
The SSL certificate type used is a SAN multi-domain name certificate (SAN/UCC SSL):

Subject Alternative Name (SAN) certificates are also known as Unified Communication Certificates (UCC). SAN SSL certificates allow you to add multiple “domain names” or “server” names that need protection within the same certificate. This feature provides a huge amount of flexibility, as it lets you create an SSL certificate that is not only easy to use and install, but also more secure than wildcard SSL certificates, and perfectly suited to your server security requirements.

Certificate issuers: <https://www.digicert.com/>
	
The Azure CDN will apply for, install, and maintain the SSL certificate on your behalf.

### Information about charges
Information about charges for CDN HTTPS acceleration nodes, as for other CDN acceleration nodes, can be viewed through the corresponding [Azure account portal](https://account.windowsazure.cn) and is summarized below the corresponding Azure subscription.


### CDN HTTPS pricing
The Azure CDN HTTPS acceleration service is currently included in the premium version. See the [official Azure website](https://www.azure.cn/home/features/cdn/pricing/), for details of pricing and charges.


<!--Image references-->
[1]: ./media/cdn-https/he001.png
[2]: ./media/cdn-https/he002.png
[3]: ./media/cdn-https/he003.png
[4]: ./media/cdn-https/he004.png
[5]: ./media/cdn-https/he005.png
[6]: ./media/cdn-https/he006.png
[7]: ./media/cdn-https/he007.png
[8]: ./media/cdn-https/h008.png
[9]: ./media/cdn-https/he009.png
[10]: ./media/cdn-https/h010.png
[11]: ./media/cdn-https/he011.png
[12]: ./media/cdn-https/he012.png
[13]: ./media/cdn-https/he013.png
[14]: ./media/cdn-https/h007.png

<!---HONumber=Acom_0503_2016_CDN-->