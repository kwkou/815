<properties
	pageTitle=""
    description=""
    services=""
    documentationCenter=""
    authors=""
    manager=""
    editor=""
    tags=""/>

<tags ms.service="legal-en" ms.date="03/2016" wacn.date="03/2016" wacn.lang="en"/>

> [AZURE.LANGUAGE]
- [中文](/support/sla/media-services/)
- [English](/support/sla/media-services-en/)
#SLA for Media Services

1. For Media Services Encoding, we guarantee 99.9% availability of REST API transactions.

2. For Streaming, we will successfully service requests with a 99.9% availability guarantee for existing media content when at least one Streaming Unit is purchased.

3. For Live Channels, we guarantee that running Channels will have external connectivity at least 99.9% of the time.

4. For Content Protection, we guarantee that we will successfully fulfill key requests at least 99.9% of the time.

5. For Indexer, we will successfully service Indexer Task requests processed with an Encoding Reserved Unit 99.9% of the time.


##Introduction
 

This Service Level Agreement for Azure (this “SLA”) is made by 21Vianet in connection with, and is a part of, the agreement under which Customer has purchased Azure Services from 21Vianet (the “Agreement”).

We provide financial backing to our commitment to achieve and maintain Service Levels for our Services. If we do not achieve and maintain the Service Levels for each Service as described in this SLA, then you may be eligible for a credit towards a portion of your monthly service fees. These terms will be fixed for term of your Agreement. If a subscription is renewed, the version of this SLA that is current at the time the renewal term commences will apply throughout the renewal term. We will provide at least 90 days' notice for adverse material changes to this SLA. You can review the most current version of this SLA at any time by visiting [http://www.azure.cn/support/legal/sla/](/support/legal/sla/).


##General Terms
 

###1. Definitions 

1. "Claim" means a claim submitted by Customer to 21Vianet pursuant to this SLA that a Service Level has not been met and that a Service Credit may be due to Customer.

2. "Customer" refers to the organization that has entered into the Agreement.

3. "Customer Support" means the services by which 21Vianet may provide assistance to Customer to resolve issues with the Services.

4. "Error Code" means an indication that an operation has failed, such as an HTTP status code in the 5xx range.

5. "External Connectivity" is bi-directional network traffic over supported protocols such as HTTP and HTTPS that can be sent and received from a public IP address.

6. "Incident" means any set of circumstances resulting in a failure to meet a Service Level.

7. "Management Portal" means the web interface, provided by 21Vianet, through which customers may manage the Service.

8. "21Vianet" means the 21Vianet entity that appears on Customer's Agreement.

9. "Preview" refers to a preview, beta, or other pre-release version of a service or software offered to obtain customer feedback.

10. "Service” or “Services" refers to a Azure service provided to Customer pursuant to the Agreement for which an SLA is provided below.

11. "Service Credit" is the percentage of the monthly service fees for the affected Service or Service Resource that is credited to Customer for a validated Claim.

12. "Service Level" means standards 21Vianet chooses to adhere to and by which it measures the level of service it provides for each Service as specifically set forth below.

13. "Service Resource" means an individual resource available for use within a Service.

14. "Success Code" means an indication that an operation has succeeded, such as an HTTP status code in the 2xx range.

15. "Support Window" refers to the period of time during which a Service feature or compatibility with a separate product or service is supported. 

16. "Virtual Network" refers to a virtual private network that includes a collection of user-defined IP addresses and subnets that form a network boundary within Azure.

17. "Virtual Network Gateway" refers to a gateway that facilitates cross-premises connectivity between a Virtual Network and a customer on-premises network.

###2. Service Credit Claims

1. In order for 21Vianet to consider a Claim, Customer must submit the Claim to Customer Support within two months of the end of the billing month in which the Incident that is the subject of the Claim occurs. Customer must provide to Customer Support all information necessary for 21Vianet to validate the Claim, including but not limited to detailed descriptions of the Incident, the time and duration of the Incident, the affected resources or operations, and any attempts made by Customer to resolve the Incident

2. 21Vianet will use all information reasonably available to it to validate the Claim and to determine whether any Service Credits are due.

3. In the event that more than one Service Level for a particular Service is not met because of the same Incident, Customer must choose only one Service Level under which a Claim may be made based on the Incident.

4. Service Credits apply only to fees paid for the particular Service, Service Resource, or Service tier for which a Service Level has not been met. In cases where Service Levels apply to individual Service Resources or to separate Service tiers, Service Credits apply only to fees paid for the affected Service Resource or Service tier, as applicable.

###3. SLA Exclusions


This SLA and any applicable Service Levels do not apply to any performance or availability issues:

1. Due to factors outside 21Vianet’s reasonable control (for example, a network or device failure external to 21Vianet’s data centers, including at Customer's site or between Customer's site and 21Vianet’s data center);

2. That resulted from Customer's use of hardware, software, or services not provided by 21Vianet as part of the Services (for example, third-party software or services purchased from the Azure Store or other non-Azure services provided by 21Vianet);

3. Due to Customer's use of the Service in a manner inconsistent with the features and functionality of the Service (for example, attempts to perform operations that are not supported) or inconsistent with published documentation or guidance;

4. That resulted from faulty input, instructions, or arguments (for example, requests to access files that do not exist);

5. Caused by Customer's use of the Service after 21Vianet advised Customer to modify its use of the Service, if Customer did not modify its use as advised;

6. During or with respect to Previews or to purchases made using 21Vianet subscription credits;

7. That resulted from Customer's attempts to perform operations that exceed prescribed quotas or that resulted from throttling of suspected abusive behavior;

8. Due to Customer's use Service features that are outside of associated Support Windows; or

9. Attributable to acts by persons gaining unauthorized access to 21Vianet’s Service by means of Customer's passwords or equipment or otherwise resulting from Customer's failure to follow appropriate security practices.

###4. Service Credits

1. The amount and method of calculation of Service Credits is described below in connection with each Service. 

2. Service Credits are Customer's sole and exclusive remedy for any failure to meet any Service Level.

3. The Service Credits awarded in any billing month for a particular Service or Service Resource will not, under any circumstance, exceed Customer's monthly service fees that Service or Service Resource, as applicable, in the billing month.

4. For Services purchased as part of a suite, the Service Credit will be based on the pro-rata portion of the cost of the Service, as determined by 21Vianet in its reasonable discretion. In cases where Customer has purchased Services from a reseller, the Service Credit will be based on the estimated retail price for the applicable Service, as determined by 21Vianet in its reasonable discretion.


##The SLA details
 

###Additional Definitions
1. "**Allocated Egress Bandwidth**" is the amount of bandwidth configured by Customer in the Management Portal for a Media Service. Allocated Egress Bandwidth may be labeled "Streaming Units" or a similar name in the Management Portal.

2. "**Channel**" means an end point within a Media Service that is configured to receive media data.

3. "**Encoding**" means the processing of media files per subscription as configured in the Media Services Tasks.

4. "**Encoding Reserved Unit**" means encoding reserved units purchased by the customer in an Azure Media Services account.

5. "**Indexer Task**" means a Media Services Task that is configured to index an MP3 input file with a minimum five-minute duration.

6. "**Media Service**" means an Azure Media Services account, created in the Management Portal, associated with Customer's Azure subscription. Each Azure subscription may have more than one associated Media Service.

7. "**Media Service Request**" means a request issued to Customer's Media Service.

8. "**Media Services Task**" means an individual operation of media processing work as configured by Customer. Media processing operations involve encoding, converting, or indexing media files.

9. "**Streaming Unit**" means a unit of reserved egress capacity purchased by Customer for a Media Service.

10. "**Valid Key Requests**" are all requests made to the Content Protection Service for existing content keys in a Customer's Media Service.

11. "**Valid Media Services Requests**" are all qualifying Media Service Requests for existing media content in a Customer's Azure Storage account associated with its Media Service when at least one Streaming Unit has been purchased and allocated to that Media Service. Valid Media Services Requests do not include Media Service Requests for which total throughput exceeds 80% of the Allocated Bandwidth.

###Monthly Uptime Calculation and Service Levels for Encoding Service
1. "**Total Transaction Attempts**" is the total number of authenticated REST API requests with respect to a Media Service made by Customer during a billing month for a subscription. Total Transaction Attempts does not include REST API requests that return an Error Code that are continuously repeated within a five-minute window after the first Error Code is received.

2. "**Failed Transactions**" is the set of all requests within Total Transaction Attempts that do not return a Success Code within 30 seconds from 21Vianet's receipt of the request.

3. "**Monthly Uptime Percentage**" for the Azure Media Services Encoding Service is calculated as Total Transaction Attempts less Failed Transactions divided by Total Transaction Attempts in a billing month for a given Azure subscription. Monthly Uptime Percentage is represented by the following formula:

		Monthly Uptime % = (Total Transaction Attempts - Failed Transactions) / Total Transaction Attempts

4. The following Service Levels and Service Credits are applicable to Customer's use of the Azure Media Services Encoding Service:

Monthly Uptime Percentage | Service Credit  
---|---  
<99.9% | 10%   
<99% | 25% 

###Monthly Uptime Calculation and Service Levels for Indexer Service
1. "**Total Transaction Attempts**" is the total number of Indexer Tasks attempted to be executed using an available Encoding Reserved Unit by Customer during a billing month for a subscription.

2. "**Failed Transactions**" is the set of Indexer Tasks within Total Transaction Attempts that either 

	1) Do not complete within a time period that is 3 times the duration of the input file; or

	2) Do not start processing within 5 minutes of the time that an Encoding Reserved Unit becomes available for use by the Indexer Task.

3. "**Monthly Uptime Percentage**" for the Azure Media Services Indexer Service is calculated as Total Transaction Attempts less Failed Transactions divided by Total Transaction Attempts in a billing month for a given Azure subscription. Monthly Uptime Percentage is represented by the following formula:

		Monthly Uptime % = (Total Transaction Attempts - Failed Transactions) / Total Transaction Attempts

4. The following Service Levels and Service Credits are applicable to Customer's use of the Azure Media Services Indexer Service:

Monthly Uptime Percentage | Service Credit  
---|---  
<99.9% | 10%   
<99% | 25% 

###Monthly Uptime Calculation and Service Levels for Streaming Service
1. "**Deployment Minutes**" is the total number of minutes that a given Streaming Unit has been purchased and allocated to a Media Service during a billing month.

2. "**Maximum Available Minutes**" is the sum of all Deployment Minutes across all Streaming Units purchased and allocated to a Media Service during a billing month.

3. "**Downtime**" is the total accumulated Deployment Minutes when the Streaming Service is unavailable. A minute is considered unavailable for a given Streaming Unit if all continuous Valid Media Service Requests made to the Streaming Unit throughout the minute result in an Error Code.

4. "**Monthly Uptime Percentage**" for the Azure Media Services Streaming Service is calculated as Maximum Available Minutes less Downtime divided by Maximum Available Minutes in a billing month for a given Azure subscription. Monthly Uptime Percentage is represented by the following formula:

		Monthly Uptime % = (Maximum Available Minutes - Downtime) / Maximum Available Minutes

5. The following Service Levels and Service Credits are applicable to Customer's use of the Azure Media Services On-Demand Streaming Service:

Monthly Uptime Percentage | Service Credit  
---|---  
<99.9% | 10%  
<99% | 25% 

###Monthly Uptime Calculation and Service Levels for Live Channels
1. "**Deployment Minutes**" is the total number of minutes that a given Channel has been purchased and allocated to a Media Service and is in a running state during a billing month.

2. "**Maximum Available Minutes**" is the sum of all Deployment Minutes across all Channels purchased and allocated to a Media Service during a billing month.

3. "**Downtime**" is the total accumulated Deployment Minutes when the Live Channels Service is unavailable. A minute is considered unavailable for a given Channel if the Channel has no External Connectivity during the minute.

4. "**Monthly Uptime Percentage**" for the Live Channels Service is calculated as Maximum Available Minutes less Downtime divided by Maximum Available Minutes in a billing month for a given Azure subscription. Monthly Uptime Percentage is represented by the following formula:

		Monthly Uptime % = (Maximum Available Minutes - Downtime) / Maximum Available Minutes

5. The following Service Levels and Service Credits are applicable to Customer's use of the Azure Media Services Live Channels Service:

Monthly Uptime Percentage | Service Credit  
---|---  
<99.9% | 10%  
<99% | 25% 

###Monthly Uptime Calculation and Service Levels for Content Protection Service
1. "**Total Transaction Attempts**" are all Valid Key Requests made by Customer during a billing month for a given Azure subscription.

2. "**Failed Transactions**" are all Valid Key Requests included in Total Transaction Attempts that result in an Error Code or otherwise do not return a Success Code within 30 seconds after receipt by the Content Protection Service.

3. "**Monthly Uptime Percentage**" for Azure Media Services is calculated as Total Transaction Attempts less Failed Transactions divided by Total Transaction Attempts in a billing month for a given Azure subscription. Monthly Uptime Percentage is represented by the following formula:

		Monthly Uptime % = (Total Transaction Attempts - Failed Transactions) / Total Transaction Attempts

4. The following Service Levels and Service Credits are applicable to Customer's use of the Azure Media Services Content Protection Service:

Monthly Uptime Percentage | Service Credit  
---|---  
<99.9% | 10%  
<99% | 25% 
