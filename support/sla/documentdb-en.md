<properties
	pageTitle=""
    description=""
    services=""
    documentationCenter=""
    authors=""
    manager=""
    editor=""
    tags=""/>

<tags ms.service="legal" ms.date="08/2016" wacn.date="08/2016" wacn.lang="en"/>

> [AZURE.LANGUAGE]
- [中文](/support/sla/documentdb/)
- [English](/support/sla/documentdb-en/)
# DocumentDB 的服务级别协议

We guarantee at least 99.99% of the time we will successfully process requests to perform operations against DocumentDB Resources.

##SLA details 
"**Average Error Rate**" for a billing month is the sum of Error Rates for each hour in the billing month divided by the total number of hours in the billing month.

"**Database Account**" is a DocumentDB account containing one or more databases.

"**Error Rate**" is the total number of Failed Requests divided by Total Requests, across all Resources in a given Azure subscription, during a given one-hour interval. If the Total Requests in a given one-hour interval is zero, the Error Rate for that interval is 0%.

"**Excluded Requests**" are requests within Total Requests that result in an HTTP 4xx status code, other than an HTTP 408 status code.

"**Failed Requests**" is the set of all requests within Total Requests that either return an Error Code or an HTTP 408 status code or fail to return a Success Code within 5 minutes for Database Account creation and deletion, 3 minutes for updating the Performance/Offer level, and 5 seconds for all other operations.

"**Resource**" is a set of URI addressable entities associated with a Database Account.

"**Total Request**" is the set of all requests, other than Excluded Requests, to perform operations issued against Resources attempted within a one-hour interval within a given Azure subscription during a billing month.

"**Monthly Uptime Percentage**" for the DocumentDB Service is calculated by subtracting from 100% the Average Error Rate for a given Azure subscription in a billing month. The "Average Error Rate" for a billing month is the sum of Error Rates for each hour in the billing month divided by the total number of hours in the billing month. Monthly Uptime Percentage is represented by the following formula:

	Monthly Uptime % = 100% - Average Error Rate

Service Credit:

Monthly Uptime Percentage |	Service Credit  
---|---  
< 99.99% | 10%  
< 99% |	25%
