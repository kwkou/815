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
- [中文](/support/sla/storage/)
- [English](/support/sla/storage-en/)
# SLA for Storage

We guarantee that at least 99.9% of the time, we will successfully process requests to append, update, read, or delete data (Provided that the format is correct), Storage Account may connect to our Internet gateway.


## Introduction
  

This Service Level Agreement for Azure (this “SLA”) is made by 21Vianet in connection with, and is a part of, the agreement under which Customer has purchased Azure Services from 21Vianet (the “Agreement”).

We provide financial backing to our commitment to achieve and maintain Service Levels for our Services. If we do not achieve and maintain the Service Levels for each Service as described in this SLA, then you may be eligible for a credit towards a portion of your monthly service fees. These terms will be fixed for term of your Agreement. If a subscription is renewed, the version of this SLA that is current at the time the renewal term commences will apply throughout the renewal term. We will provide at least 90 days' notice for adverse material changes to this SLA. You can review the most current version of this SLA at any time by visiting [http://www.azure.cn/support/legal/sla/](http://www.azure.cn/support/legal/sla/) 


## General Terms 
 

### 一、Definitions 
 
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



### 二、Service Credit Claims

1. In order for 21Vianet to consider a Claim, Customer must submit the Claim to Customer Support within two months of the end of the billing month in which the Incident that is the subject of the Claim occurs. Customer must provide to Customer Support all information necessary for 21Vianet to validate the Claim, including but not limited to detailed descriptions of the Incident, the time and duration of the Incident, the affected resources or operations, and any attempts made by Customer to resolve the Incident

2. 21Vianet will use all information reasonably available to it to validate the Claim and to determine whether any Service Credits are due.

3. In the event that more than one Service Level for a particular Service is not met because of the same Incident, Customer must choose only one Service Level under which a Claim may be made based on the Incident.

4. Service Credits apply only to fees paid for the particular Service, Service Resource, or Service tier for which a Service Level has not been met. In cases where Service Levels apply to individual Service Resources or to separate Service tiers, Service Credits apply only to fees paid for the affected Service Resource or Service tier, as applicable.

### 三、SLA Exclusions

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


### 四、Service Credits

1. The amount and method of calculation of Service Credits is described below in connection with each Service. 

2. Service Credits are Customer's sole and exclusive remedy for any failure to meet any Service Level.

3. The Service Credits awarded in any billing month for a particular Service or Service Resource will not, under any circumstance, exceed Customer's monthly service fees that Service or Service Resource, as applicable, in the billing month.

4. For Services purchased as part of a suite, the Service Credit will be based on the pro-rata portion of the cost of the Service, as determined by 21Vianet in its reasonable discretion. In cases where Customer has purchased Services from a reseller, the Service Credit will be based on the estimated retail price for the applicable Service, as determined by 21Vianet in its reasonable discretion.


## The SLA details
 
### Additional Definitions

1. "**Geo Replication Lag**" for GRS and RA-GRS Accounts is the time it takes for data stored in the Primary Region of the storage account to replicate to the Secondary Region of the storage account. Because GRS and RA-GRS Accounts are replicated asynchronously to the Secondary Region, data written to the Primary Region of the storage account will not be immediately available in the Secondary Region. Customers can query the Geo Replication Lag for a storage account, but 21Vianet does not provide any guarantees as to the length of any Geo Replication Lag under this SLA.

2. "**Geographically Redundant Storage (GRS) Account**" is a storage account for which data is replicated synchronously within a Primary Region and then replicated asynchronously to a Secondary Region. Customers cannot directly read data from or write data to the Secondary Region associated with GRS Accounts.

3. "**Locally Redundant Storage (LRS) Account**" is a storage account for which data is replicated synchronously only within a Primary Region.

4. "**Primary Region**" is a geographical region in which data within a storage account is located, as selected by Customer when creating the storage account. Customers may execute write requests only against data stored within the Primary Region associated with storage accounts. 

5. "**Read Access Geographically Redundant Storage (RA-GRS) Account**" is a storage account for which data is replicated synchronously within a Primary Region and then replicated asynchronously to a Secondary Region. Customers can directly read data from, but cannot write data to, the Secondary Region associated with RA-GRS Accounts.

6. "**Secondary Region**" is a geographical region in which data within a GRS or RA-GRS Account is replicated and stored, as assigned by Azure based on the Primary Region associated with the storage account. Customers cannot specify the Secondary Region associated with storage accounts.

7. "**Zone Redundant Storage (ZRS) Account**" is a storage account for which data is replicated across multiple facilities. These facilities may be within the same geographical region or across two geographical regions. 




### Monthly Uptime Calculation and Service Levels for Storage Service

1. "**Total Storage Transactions**" is the set of all storage transactions, other than Excluded Transactions, attempted within a one-hour interval across all storage accounts in the Storage Service in a given subscription.

2. "**Excluded Transactions**" are storage transactions that do not count toward either Total Storage Transactions or Failed Storage Transactions. Excluded Transactions include pre-authentication failures; authentication failures; attempted transactions for storage accounts over their prescribed quotas; creation or deletion of containers, tables, or queues; clearing of queues; and copying blobs between storage accounts.

3. "**Error Rate**" is the total number of Failed Storage Transactions divided by the Total Storage Transactions during a set time interval (currently set at one hour).

4. "**Failed Storage Transactions**" is the set of all storage transactions within Total Storage Transactions that are not completed within the Maximum Processing Time associated with their respective transaction type, as specified in the table below. Maximum Processing Time includes only the time spent processing a transaction request within the Storage Service and does not include any time spent transferring the request to or from the Storage Service.

<table>
 	  <tr>
      <th>Request Type</th>
      <th >Maximum Processing Time*</th>      
  	 </tr>  
  	<tr>
      <td>PutBlob and GetBlob (includes blocks and pages)Get Valid Page Blob Ranges</td>
      <td>Two (2) seconds multiplied by the number of MBs transferred in the course of processing the request</td>      
   	</tr>
  	<tr>
      <td>Copy Blob</td>
      <td>Ninety (90) seconds (where the source and destination blobs are within the same storage account)</td>
     </tr>
 	<tr>
      <td>PutBlockList <br/>
		  GetBlockList</td>
      <td>Sixty (60) seconds</td>      
 	 </tr>
    <tr>
      <td>Table Query <br/>
		  List Operations</td>
      <td>Ten (10) seconds (to complete processing or return a continuation)</td>      
 	 </tr>
	<tr>
      <td>Batch Table Operations</td>
      <td>Thirty (30) seconds</td>      
 	 </tr>
	<tr>
      <td>All Single Entity Table Operations <br/>
			All other Blob and Message Operations</td>
      <td>Two (2) seconds</td>      
 	 </tr>
</table>

*These figures represent maximum processing times. Actual and average times are expected to be much lower.

		Monthly Uptime % = (Maximum Available Minutes − Downtime) ÷ Maximum Available Minutes 

Failed Storage Transactions do not include:


1. Transaction requests that are throttled by the Storage Service due to a failure to obey appropriate back-off principles. 


2. Transaction requests having timeouts set lower than the respective Maximum Processing Times specified above. 


3. Read transactions requests to RA-GRS Accounts for which Customer did not attempt to execute the request against Secondary Region associated with the storage account if the request to the Primary Region was not successful. 


4. Read transaction requests to RA-GRS Accounts that fail due to Geo-Replication Lag.

5. “**Error Rate**”is the total number of Failed Storage Transactions divided by the Total Storage Transactions during a given one-hour interval. If the Total Storage Transactions in a given one-hour interval is zero, the error rate for that interval is 0%.

6. “**Monthly Uptime Percentage**”for the Storage Service is calculated by subtracting from 100% the Average Error Rate for the billing month for a given Azure subscription. The “Average Error Rate” for a billing month is the sum of Error Rates for each hour in the billing month divided by the total number of hours in the billing month. Monthly Uptime Percentage is represented by the following formula:

		Monthly Uptime % = 100% - Average Error Rate 

7. The following Service Levels and Service Credits are applicable to Customer's use of the Storage Service for all qualified transaction requests for LRS, ZRS, and GRS Accounts and write transaction requests for RA-GRS Accounts:


	Monthly Uptime Percentage	|Service Credit
	----------------------------|---------
	<99.95%				        |10% 
	<99%				        |25% 

8. The following Service Levels and Service Credits are applicable to Customer's use of the Storage Service for qualified read transaction requests for RA-GRS Accounts:


	Monthly Uptime Percentage	|Service Credit
	----------------------------|---------
	<99.95%				        |10% 
	<99%				        |25% 
