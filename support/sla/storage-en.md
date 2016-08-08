<properties
	pageTitle=""
    description=""
    services=""
    documentationCenter=""
    authors=""
    manager=""
    editor=""
    tags=""/>

<tags ms.service="legal-en" ms.date="08/2016" wacn.date="08/2016" wacn.lang="en"/>

> [AZURE.LANGUAGE]
- [中文](/support/sla/storage/)
- [English](/support/sla/storage-en/)

# SLA for Storage

- We guarantee that at least 99.99% (99.9% for Cool Access Tier) of the time, we will successfully process requests to read data from Read Access-Geo Redundant Storage (RA-GRS) Accounts, provided that failed attempts to read data from the primary region are retried on the secondary region.
- We guarantee that at least 99.9% (99% for Cool Access Tier) of the time, we will successfully process requests to read data from Locally Redundant Storage (LRS) and Geo Redundant Storage (GRS) Accounts.
- We guarantee that at least 99.9% (99% for Cool Access Tier) of the time, we will successfully process requests to write data to Locally Redundant Storage (LRS) and Geo Redundant Storage (GRS) Accounts and Read Access-Geo Redundant Storage (RA-GRS) Accounts.

##SLA details
 
###Additional Definitions

"**Average Error Rate**" for a billing month is the sum of Error Rates for each hour in the billing month divided by the total number of hours in the billing month.

"**Blob Storage Account**" is a storage account specialized for storing data as blobs and provides the ability to specify an access tier indicating how frequently the data in that account is accessed.

"**Cool Access Tier**" is an attribute of a Blob Storage Account indicating that the data in the account is infrequently accessed and has a lower availability service level than data in other access tiers.

"**Excluded Transactions**" are storage transactions that do not count toward either Total Storage Transactions or Failed Storage Transactions. Excluded Transactions include pre-authentication failures; authentication failures; attempted transactions for storage accounts over their prescribed quotas; creation or deletion of containers, tables, or queues; clearing of queues; and copying blobs between storage accounts.

"**Error Rate**" is the total number of Failed Storage Transactions divided by the Total Storage Transactions during a set time interval (currently set at one hour). If the Total Storage Transactions in a given one-hour interval is zero, the error rate for that interval is 0%.

"**Failed Storage Transactions**" is the set of all storage transactions within Total Storage Transactions that are not completed within the Maximum Processing Time associated with their respective transaction type, as specified in the table below. Maximum Processing Time includes only the time spent processing a transaction request within the Storage Service and does not include any time spent transferring the request to or from the Storage Service.

|REQUEST TYPES	| MAXIMUM PROCESSING TIME|  
|---|---|  
|PutBlob and GetBlob (includes blocks and pages) <br /> Get Valid Page Blob Ranges |	Two (2) seconds multiplied by the number of MBs transferred in the course of processing the request |  
|Copy Blob	| Ninety (90) seconds (where the source and destination blobs are within the same storage account)|  
|PutBlockList |	Sixty (60) seconds|  
|GetBlockList |  |  
|Table Query <br /> List Operations | Ten (10) seconds (to complete processing or return a continuation)|  
|Batch Table Operations	| Thirty (30) seconds |  
|All Single Entity Table Operations <br /> All other Blob and Message Operations | Two (2) seconds |  

These figures represent maximum processing times. Actual and average times are expected to be much lower.

Failed Storage Transactions do not include:

1. Transaction requests that are throttled by the Storage Service due to a failure to obey appropriate back-off principles.
2. Transaction requests having timeouts set lower than the respective Maximum Processing Times specified above.
3. Read transactions requests to RA-GRS Accounts for which you did not attempt to execute the request against Secondary Region associated with the storage account if the request to the Primary Region was not successful.
4. Read transaction requests to RA-GRS Accounts that fail due to Geo-Replication Lag.

"**Geo Replication Lag**" for GRS and RA-GRS Accounts is the time it takes for data stored in the Primary Region of the storage account to replicate to the Secondary Region of the storage account. Because GRS and RA-GRS Accounts are replicated asynchronously to the Secondary Region, data written to the Primary Region of the storage account will not be immediately available in the Secondary Region. You can query the Geo Replication Lag for a storage account, but 21Vianet does not provide any guarantees as to the length of any Geo Replication Lag under this SLA.

"**Geographically Redundant Storage (GRS) Account**" is a storage account for which data is replicated synchronously within a Primary Region and then replicated asynchronously to a Secondary Region. You cannot directly read data from or write data to the Secondary Region associated with GRS Accounts.

"**Locally Redundant Storage (LRS) Account**" is a storage account for which data is replicated synchronously only within a Primary Region.

"**Primary Region**" is a geographical region in which data within a storage account is located, as selected by you when creating the storage account. You may execute write requests only against data stored within the Primary Region associated with storage accounts.

"**Read Access Geographically Redundant Storage (RA-GRS) Account**" is a storage account for which data is replicated synchronously within a Primary Region and then replicated asynchronously to a Secondary Region. You can directly read data from, but cannot write data to, the Secondary Region associated with RA-GRS Accounts.

"**Secondary Region**" is a geographical region in which data within a GRS or RA-GRS Account is replicated and stored, as assigned by Azure based on the Primary Region associated with the storage account. You cannot specify the Secondary Region associated with storage accounts.

"**Total Storage Transactions**" is the set of all storage transactions, other than Excluded Transactions, attempted within a one-hour interval across all storage accounts in the Storage Service in a given subscription.

Monthly Uptime Percentage: Monthly Uptime Percentage is calculated using the following formula:

	100% - Average Error Rate

Service Credit – LRS, GRS and RA-GRS (write requests) Accounts:

MONTHLY UPTIME PERCENTAGE |	SERVICE CREDIT  
---|---  
< 99.9%	| 10%  
< 99% |	25%

Service Credit – RA-GRS (read requests) Accounts:

MONTHLY UPTIME PERCENTAGE |	SERVICE CREDIT  
---|---  
< 99.99% | 10%  
< 99% |	25%  

Service Credit – LRS, GRS and RA-GRS (write requests) Blob Storage Accounts (Cool Access Tier):

MONTHLY UPTIME PERCENTAGE |	SERVICE CREDIT  
---|---  
< 99% |	10%  
< 98% |	25%  

Service Credit – RA-GRS (read requests) Blob Storage Accounts (Cool Access Tier):

MONTHLY UPTIME PERCENTAGE |	SERVICE CREDIT  
---|---  
< 99.9% | 10%  
< 98% |	25%

##Version History

[1.1](/support/sla/storage-en/)Last updated: August 2016

[1.0](/support/sla/storage-en-v1/)Last updated: March 2016