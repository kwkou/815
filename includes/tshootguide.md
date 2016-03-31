#Troubleshooting in Azure

*Troubleshooting* refers to the general task of locating and
understanding unexpected application behavior and correcting it. When developing applications, developers test, run, and debug
their applications before deploying them into a production
environment. This fact is true whether the application runs on a desktop
computer or a server in the cloud. However, a widely distributed,
multi-instance application designed for scale-out can be hard to debug,
requiring more than standard tools and approaches.

For this reason, cloud applications should be designed with
troubleshooting in mind. How you design troubleshooting support into
your application depends first on where your application runs, and
second on which languages or runtimes your application is built with or
uses. 

If you are building an application that runs on an Azure
Virtual Machine, in many cases, you can approach troubleshooting design as well
as debugging as you would on the operating system if it were running on
your own servers.

Applications running on Azure are widely distributed,
multi-instance applications that can be hard to debug. These types of
applications require more than standard tools and approaches to troubleshooting. This topic
discusses some proven troubleshooting practices and contains links to
more intensive information on the practices described.

**Note**: This topic assumes you are either designing your application
or have successfully deployed your Azure application and that
something unexpected is now occurring. It does not discuss how to deploy
an application on Azure. For more information about developing
and deploying your Azure application, see <a href="/zh-cn/documentation/">/zh-cn/documentation/</a>.

This topic first describes some best practices that will help you design
your application so that you can troubleshoot effectively when problems
occur. (If you do not design your application to enable you to follow
the code flow in advance, it can be very hard to locate problems when
they do occur.) These best practices are valid for all types of
applications running on Azure, regardless of the application
model or language used.

The following sections describe specific approaches for developing
supportable Azure applications regardless of the type of
application: Web Sites, Cloud Services, or Virtual Machines.

Each section describes best practices at a high level, and contains
pointers to resources that demonstrate best practices in detail or
describe how to implement them.


##In this Document

* [Best Practices for Troubleshooting in Azure](#BestPractices)
* [Azure Web Sites](#Websites)
* [Azure Cloud Services](#CloudServices)
* [Azure Virtual Machines](#Vms)
* [Azure Services](#PlatformServices)
* [SQL数据库 Troubleshooting](#SQLTroubleshooting)

<h2><a id="BestPractices"></a>Best Practices for Troubleshooting in Azure</h2>

This section describes best practices that apply to Azure
applications no matter which hosting model or language you use. It
contains resources for more in-depth discussion of those practices.

To build the foundation for efficient troubleshooting in Azure,
concentrate your efforts in these three main areas:

-   Handling Failures Gracefully - *Each component service must be able
    to endure the failure of dependent services or infrastructure*.
-   Tracing, Logging and Monitoring - *Each component service must have the proper
    debugging, tracing, event, and error logging.*
-   Debug errors where you can - *Before promoting for production, but
    also at the component and network level when running.*

Designing an application with these ideas in mind allows the application
to provide you with the information necessary to track down unexpected
behavior when it occurs.

###Design to Handle Errors Gracefully

Applications should handle error conditions gracefully if they can. This
is even more important given the distributed nature of Azure.
Effective troubleshooting begins with a good transient-failure handling
design. Transient errors are one of the main areas in which cloud
applications fail to behave as expected due to transient error
conditions inherent to internet applications. 

Transient errors are failures related to latencies and intermittent
network connections inherent in shared resources on the internet. Some
examples are:

-   Shared computer resources such as Azure Cloud Services and
    SQL数据库 (to give two examples) can be slightly less or more
    responsive from moment to moment.
-   Responsiveness delays due to providing durability for services. For example, SQL
    Azure keeps multiple copies of databases consistent in order to
    provide durability, this has an impact on responsiveness.
-   Delays caused by HTTP or other protocol connections ending prior to
    completing work. For example, HTTP requests may not reach an
    endpoint and return prior to their timeout period.


To help alleviate the impact of transient errors, Azure
applications should:

-   Be loosely coupled so that components are not locally dependent upon
    services that fail more often than in on-premise environments.
-   Make asynchronous calls whenever possible to keep processes from
    becoming dependent on immediate responses.
-   Use a transient error handling approach that detects certain
    categories of failures and can implement retry behavior for those
    failures based on some configurable retry policy.

Calls to services should either build or use a transient error handling layer to detect common failure scenarios and retry the call based upon a configuration setting. For .NET developers, one recommended library is the [Microsoft Enterprise Library 5.0 Integration Pack for Azure].
Microsoft Enterprise Library is a collection of reusable application
blocks that address common cross-cutting concerns in enterprise software
development. The Microsoft Enterprise Library Integration Pack for
Azure is an extension to Microsoft Enterprise Library 5.0 that
can be used with Azure. It includes the
Autoscaling Application Block, Transient Fault Handling Application
Block, blob configuration source, protected configuration provider, and
learning materials. Another, simpler .NET library with fewer features is the
[Cloud Application Framework & Extensions (CloudFx)]. CloudFx offers a set
of production quality components and building blocks intended to
jump-start the implementation of feature-rich, reliable, and extensible
Azure-based solutions and services.

###Perform Appropriate Tracing and Logging

Because the complexity of distributed, scale-out applications,
traditional debuggers that work against one process are of much less use 
when investigating issues that occur while your application is running.
Therefore tracing and logging are of upmost importance. The execution of
your app and its data is shared across many services, which are hosted
on many different machines. In a large scale distributed application, it
is difficult, if not impossible to determine which service instance to
attach to and debug. Tracing and logging allow you to follow application
execution and data flow giving you a better understanding of the state
of your application.

Successful Azure applications have a logging and tracing
strategy designed in to the application from the beginning. This reduces
the time and effort required to locate any issues and repair them
quickly without having to call Microsoft for support.

**Note**: Writing traces and logging extensively has a performance
impact; doing so intensively has a more profound one. Therefore the
design of your application should include a configurable tracing and
logging policy that can be adjusted at need. Ideally, the level of
logging should be adjustable from the **ServiceConfiguration.cscfg** 
file so that it can be changed without having to redeploy.

Having volumes of logs does not guarantee speedy bug detection and
repair; a large amount of data takes a long time to decipher, and
collecting it impacts the performance of your application. Adjustable
logging controls both the size and storage cost of log data.

In the MSDN Magazine article [Take Control of Logging and Tracing in Azure], Mike Kelly describes four types of diagnostic output to
consider:

-   Debug Output - Only in debug build, includes asserts
-   Tracing - Tracks the flow of control during execution
-   Event Logging - Major events in program execution
-   Error Logging - Exceptional or dangerous situation

Other languages, application platforms, and operating systems
may have different terminology for tracing and logging. If you are using
one of these development platforms on Azure, use the equivalent 
strategy and tools for the language or the platform you are using.

Mixed mode applications are applications executing in a combination of
Azure Virtual Machines, Web Sites, and Cloud Services. When
building applications of this type, tracing and logging become even more
important because they are more widely distributed. To troubleshoot
these mixed-mode applications the overall data and execution flow must
be followed in order to identify any problems. The mechanics of tracing and logging a mixed mode application depends upon the hosting model of the component. 

###Monitor Your Application

Tracing and logging fit in to the bigger area of monitoring. Monitoring
allows you to:

-   Discover Azure applications
-   Determine status of each role instance
-   Collect and monitor performance information including latency and
    throughput
-   Collect and monitor events
-   Collect and monitor trace messages
-   Monitor resource usage
-   Monitor quality of service metrics
-   Perform capacity planning
-   Perform traffic analysis (users, views, peak times)
-   Estimate billing
-   Perform auditing

Monitoring is accomplished with a tool or set of tools. Which tool you
use depends on the platform and/or languages your application uses and on your monitoring goals and requirements.

**Microsoft System Center Monitoring Pack for Azure Applications**

This [Monitoring Pack] allows you to use [Microsoft System Center
Operations Manager] to monitor the availability and performance of
Azure Applications.

Using Microsoft System Center Operations Manager 2012 is the best way to
monitor the health of your Azure application.

**Azure Platform Management Tool**

Another tool is the [Azure Platform Management Tool (MMC)], which
enables you to manage your Azure hosted services and storage
accounts. This tool is provided as a sample with complete source code so
you can see how to perform various management and configuration tasks
using the Azure Management and Diagnostics APIs.

**Cerebrata Tools**

Cerebrata provides a number of tools that allow you to monitor and
manage your Azure applications. These include [Azure Diagnostics
Manager], [Cloud Storage Studio], and [Azure Management Cmdlets].

Azure Diagnostics Manager 2 is a Windows (WPF) based client for managing
Azure Diagnostics. It lets you view, download, and manage the
diagnostics data collected by applications running in Azure. See
[http://www.cerebrata.com](http://www.cerebrata.com) for more information on these products.

Cloud Storage Studio 2 is a Windows (WPF) based client for managing
Azure Storage.

Azure Management Cmdlets is a set of Windows PowerShell cmdlets for managing
Azure Storage, Hosted Services, SQL数据库 instances, and
Diagnostics. It also provides cmdlets to back up and restore storage
accounts.

**Network Monitoring: AlertBot, Gomez, Keynote, Pingdom**

Compuware's [Gomez] Application Performance Management, [Keynote], [Pingdom],
and [AlertBot] are solutions for monitoring your Azure application
remotely. They allow you to monitor the availability of your application
and optimize performance. Some services, such as Pingdom, enable
notification by email, SMS, or a desktop application when an error is
detected. This type of monitoring simulates what an end user does to
successfully monitor an application.

**Apica Tools**

Apica provides tools that monitors your Azure WEB application "from outside." For more information, see <a href="http://www.apicasystem.com/integration-partners/">http://www.apicasystem.com/integration-partners/</a>.


**AVIcode**

Microsoft purchased AVIcode and it is now part of Microsoft System
Center. [AVIcode] delivers .NET application performance monitoring
capabilities with a comprehensive suite of application monitoring
capabilities.

**Performance Profiling**

You can [profile] your Azure application when it runs in 
Azure to determine any performance issues. When you publish your 
Azure application from Visual Studio, you can choose to profile the
application and select the profiling settings that you require.

**Azure VM Assistant**

The [VM Assistant] tool is a CodePlex project that collects all the
relevant troubleshooting data in one place when you Remote Desktop into
a virtual machine instance. The **VM Health** button gives the current
status of the instance.

###Debugging Errors Where You Can

Before deploying an app to Azure, it is a best practice to debug
your application locally. The Azure SDK contains emulators that
mimic the Azure cloud environment, allowing you to run your app and
do rudimentary tests without having to deploying your application. The
debugging tools you use vary depending upon the programming language and the development tools
you are using.

After an application has been deployed, you can debug in the cloud using a debugger like Visual Studio. This requires creating a debug build and deploying it. In order to do this,you must connect to a specific role instance. If you have a complex application with multiple roles and role instances, it can be very difficult to determine to which role instance to connect. Visual Studio Ultimate supports IntelliTrace, which allows .NET roles to track debug information. When a problem occurs you can download this information and load it into Visual Studio. You can look at each role instance's IntelliTrace log to determine where the problem occurred. While there are some drawbacks to debugging in the cloud, there are some circumstances in which it is required. Not all Azure Services have an emulator (for example 服务总线) and not all supported development tools (for example Mac and Linux) come with emulators. 

Once you have debugged your application locally you will most likely
have to rely on the instrumentation built into your application to
determine where problems are occurring.

**Node.js Debugging**

For debugging Node.JS applications, you can use the Node-Inspector tool
which is available on [GitHub](https://github.com/dannycoates/node-inspector). Node-Inspector can be run locally on your
development machine using the Azure storage emulator. For more
information see Jim Wang's blog: [Debugging Node in the Azure Emulator].

If you are debugging your application on Azure, install the full version
of IISNode from [GitHub](https://github.com/windowsazure/iisnode/downloads) on your web role, worker role, or VM instance. As
discussed earlier, this is not a recommended way to debug your
application when it is in production and scaled to multiple instances because you may not know to which role instance or VM to
debug.

To use Node-Inspector on a web role, install the package in the web role
and use the Node-Inspector tool as you normally would. For more
information about debugging with Node-Inspector, see [Debugging with
Node-Inspector].

**IntelliTrace**

Microsoft Visual Studio Ultimate contains [IntelliTrace], which can
be enabled to debug applications before deployment into production.
IntelliTrace supports ASP.NET, and WCF applications. Intellitrace is not
supported when it is enabled in a production service, but can be used to
get exceptions for an application after deployment to Azure. Jim
Nakashima's blog post describes how to use [IntelliTrace to debug 
Azure Cloud Services].

**Fiddler**

[Fiddler] is a Web Debugging Proxy that logs all HTTP(S) traffic between
your computer and the Internet. Fiddler allows you to inspect traffic,
set breakpoints, and "fiddle" with incoming or outgoing data. Fiddler is
especially helpful for troubleshooting Azure Storage.

To use Fiddler against the local development fabric, use ipv4.fiddler
instead of 127.0.0.1:

-   Launch Fiddler.
-   Launch your service in the development fabric.
-   Browse to http://ipv4.fiddler:/. Fiddler should trace the request.

To use Fiddler against the local development storage, modify the service
configuration file to point to Fiddler:

Open the ServiceConfiguration.cscfg file and change the connection
string to:

	Value="UseDevelopmentStorage=true;DevelopmentStorageProxyUri=http://ipv4.fiddler"

-   Launch Fiddler.
-   Launch your service. Fiddler should trace any storage requests.

##Troubleshooting and the Azure Hosting Models##
This section discusses best practices for debugging applications using the different Azure hosting models.
<h2><a id="Websites"></a>Azure Web Sites</h2>

When designing a supportable Azure web site, follow the
recommendations made earlier in this document when possible. This
includes checking for and handling errors, applying transient fault
retry logic, tracing, and logging. Troubleshooting Azure
web sites is accomplished by configuring web sites to display application
errors, configuring diagnostics for a web site, collecting diagnostic
data and then analyzing the collected data to identify and resolve
problems.

Azure web sites enable configuration of the following diagnostic
options:

-   Web Server Logging
-   Detailed Error Messages
-   Failed Request Tracing

For more information on these topics see: [Troubleshooting an Azure web site].

When web server logs for Azure web sites are enabled, the web site
will record all HTTP transactions into a log file using the [W3C extended log file format]. You can then use [Log Parser] to query the log file. Some
example log parser queries are available on [Log Parser Plus] and [TechNet
Log Parser Examples]. If you want to generate the CHART output type on a
computer that is running Office 2007/2010, install [Office 2003 Web
Components] following the instructions on [Log Parser Plus](http://logparserplus.com/article/2).

Azure web sites uses the same failed request tracing
functionality that has been available with IIS 7.0 and later. It is not,
however, configurable like IIS failed request tracing. When you enable
failed request tracing for Azure web sites, **all** failed
requests are captured. 

<h2><a id="Cloudservices"></a>Azure Cloud Services</h2>

Because of the distributed nature of Azure Cloud Services, it's
important to defend your application by making calls asynchronously and
handling retries for transient failures, as described previously.

The debugging technique used for Azure Cloud Services depends on the type of problem you are experiencing. Problems involving a specific role or role instance, for example a role failing to start or cycling, are best investigated using Remote Desktop. In these cases you will know which role or role instance is problematic and you can connect to the affected role. When a problem occurs and you are not sure what role instance is causing the problem, tracing and logging is a better method for troubleshooting. Azure Diagnostics provides a mechanism to collect and manage trace and log information.

Some new debugging features have been added to the Azure SDK 1.7 including making it easier to find stack traces when exceptions occur and improvements in Remote Desktop connectivity. Stack traces are now included in the Windows Event Log, making it easier to see exactly what went wrong. In addition Remote Desktop connectivity has been improved. If your role is cycling or aborted you will be able to use Remote Desktop to connect to the problematic role and investigate the problem. 

The Azure Portal provides access to monitoring data that helps IT professionals and developers anticipate and diagnose problems in Azure Cloud Services. By default values such as "CPU Percentage", "Data In", "Data Out", "Disk Read Throughput" and "Disk Write Throughput" are collected by the host VM. There is no configuration needed to enable these metrics for role instances and there is no cost impact to customers. Additional performance information can also be collected. To collect verbose diagnostic information you must have a valid diagnostics connection string as this information will be stored in Azure Storage and will therefore incur additional storage costs. When user enables verbose monitoring, the portal will remotely configure role instances to collect the default set of performance counters. 

###Azure Diagnostics 

The original Azure SDK 1.0 included functionality to collect
diagnostics data and store them in Azure storage collectively known
as Azure Diagnostics. This software, built upon the Event
Tracing for Windows (ETW) framework, fulfills two design requirements
introduced by Azure scale-out architecture:

-   Save diagnostic data that would be lost during a reimaging of the
    instance.
-   Provide a central repository for diagnostics from multiple
    instances.

After configuring Diagnostics in the role, Diagnostics collects
diagnostic data from all the instances of that particular role. The
diagnostic data can be used for debugging and troubleshooting, measuring
performance, monitoring resource usage, traffic analysis and capacity
planning, and auditing. Transfers to Azure storage account for
persistence can either be scheduled or on-demand.

Azure Diagnostics changes the server paradigm in four important
ways:

-   Diagnostics must be enabled at application creation time.
-   Specific tools/steps are needed to visualize diagnostic results.
-   Crashes will cause the loss of diagnostic data unless it is written
    to durable storage (Azure Storage as opposed to being on
    each instance).
-   Diagnostic storage incurs a monthly cost when stored in Microsoft
    Azure storage.

Cost is of particular importance because one of the key benefits of
Azure is cost reduction. The only way to eliminate the
cost of using Diagnostics today is to leave the data on the virtual machine.
This may work in a small deployment, but is impractical where there are
many instances. Here are a few ways to minimize the financial impact:

-   Make sure that the storage account is in the same data center as
    your application. If for some reason they are not in the same data
    center, choose the interval of scheduled transfers wisely. Shorter
    transfer times will increase data relevance, but that trade off may
    not be great enough to justify the additional bandwidth and
    processing overhead.
-   Periodically copy and clear the diagnostic data from Azure
    Storage. The diagnostic data will transit through Azure
    storage, but not reside there unnecessarily. There are a number of
    tools to do this: System Center Monitoring Pack for Azure,
    Cerebrata's Azure Diagnostics Manager, and Azure PowerShell
    cmdlets.
-   Choose only the diagnostic data that you will need to troubleshoot
    and monitor your application. Capturing too much data may make it
    harder to troubleshoot in addition to costing significantly more.
-   Control the collection and extent of diagnostic data by implementing
    an on-demand switch in your application.
-   Utilize the logging level (Verbose, Info, Warning, Error) so that
    all information is available, then utilize the post-deploy Diagnostics
    config to selectively gather data.

<h2><a id="Vms"></a>Azure Virtual Machines</h2>

Troubleshooting applications running on Azure Virtual Machines
typically involve the same troubleshooting techniques that you would use
with the operating systems and platforms in use. For example:

-   Windows Server 2008 R2 x64 (RTM and SP1); Windows 8 Service x64
    (Datacenter and Core). There is a large amount of information
    available on the subject, and many tools available to help you do
    the job quickly.

-   For more information about the approaches, see [How to: Debug Windows Service Applications].

-   For a video illustrating the best approaches, see [Daniel Pearson's presentation Windows Debugging and Troubleshooting].

-   Open Suse Linux. There is also a tremendous amount of resources
    available for troubleshooting applications on Suse Linux, both in
    the [Suse Linux documentation] and on the internet.
-   Ubuntu Linux. Again, a large amount of information exists. To
    start with the product documentation, see [https://help.ubuntu.com/](https://help.ubuntu.com).
-   CentOS Linux. For more information, see [http://centos.org/](http://centos.org).

<h2><a id="PlatformServices"></a>Troubleshooting Azure Services</h2>

Many of the Azure services such as Azure SQL数据库, Azure
Active Directory, and Azure storage have troubleshooting advice
that is specific to their use, regardless whether the application is
executing on Azure, what programming language or libraries it was built
with, or executing on a non-Microsoft operating system. The following
information provides best practices specific to some of these services.

There are many supported libraries that implement best practices for
asynchronous calls, traces, and event logging as outlined in the design
portion of this document.

####Azure Storage Troubleshooting

The following links discuss either designs or practices to mitigate
problems requiring troubleshooting or locations in which you should add
tracing or logging work.

- Storage status and error codes: [http://msdn.microsoft.com/zh-cn/library/windowsazure/dd179382.aspx](http://msdn.microsoft.com/zh-cn/library/windowsazure/dd179382.aspx)

- Storage analytics: [http://blogs.msdn.com/b/windowsazurestorage/archive/2011/08/03/windows-azure-storage-analytics.aspx](http://blogs.msdn.com/b/windowsazurestorage/archive/2011/08/03/windows-azure-storage-analytics.aspx) (see the three "for more information" links at the bottom.)

- Overview of retry policies in the Azure Storage Client Library: [http://blogs.msdn.com/b/windowsazurestorage/archive/2011/02/03/overview-of-retry-policies-in-the-windows-azure-storage-client-library.aspx](http://blogs.msdn.com/b/windowsazurestorage/archive/2011/02/03/overview-of-retry-policies-in-the-windows-azure-storage-client-library.aspx)

- How to get the most out of Azure Tables: [http://blogs.msdn.com/b/windowsazurestorage/archive/2010/11/06/how-to-get-most-out-of-windows-azure-tables.aspx](http://blogs.msdn.com/b/windowsazurestorage/archive/2010/11/06/how-to-get-most-out-of-windows-azure-tables.aspx)

	- See these sections: Best Practices; Tips for Programming Azure; and Timeouts and ServerBusy - Is it normal?

- Protecting your Blobs against application errors: [http://blogs.msdn.com/b/windowsazurestorage/archive/2010/04/30/protecting-your-blobs-against-application-errors.aspx](http://blogs.msdn.com/b/windowsazurestorage/archive/2010/04/30/protecting-your-blobs-against-application-errors.aspx)

**Azure Storage Explorers**

There are a number of ways to explore Azure storage. The 
Azure Storage team came up with a [list of storage explorers]. Any of
these will allow you to see Diagnostics files and Azure Storage
Analytics files. Cloudberry Lab's [Explorer for Azure Blob Storage] provides a user interface to enable Storage Analytics directly in the
application by clicking **Storage Settings**.

<h2><a id="Misc"></a>Azure 服务总线 Troubleshooting</h2>

This section provides high-level guidance about how to develop an
application that uses the Azure 服务总线 in a robust and
maintainable way that will minimize common issues. It also provides
details about how to identify and address common 服务总线 errors.

###General Guidance

The 服务总线 is an internet-scale [enterprise 服务总线] that
supports relayed and brokered messaging capabilities. The 服务总线
implements quotas and thresholds at a system level for both types of
messaging. If your application exceeds these quotas, it will be
throttled or your requests or messages will be rejected. For full
details about 服务总线 quotas and the behavior you will see when they
are exceeded, see [Azure 服务总线 Quotas]. Some quotas are user
defined, for example the size of a queue or topic, which is defined when
the entity is created.

To get a view into the data in your 服务总线 messaging entity and how
it is being processed, you can use [服务总线 Explorer] or the Server
Explorer in the Azure Tools for Visual Studio (version 1.7 or higher,) to create, delete, and test queues, topics, subscriptions, and rules. This
is an excellent way to troubleshoot an application that is running but
not processing data the way you expect. These tools include
functionality that enables you to test queues, topics, and relay
entities, trace the operations performed by individual sender and
receiver tasks, monitor progress and performance of an ongoing test run,
and generate detailed logs of the results, including any error messages.

###服务总线 Relay

The 服务总线 "relay" service runs in the cloud and supports a
variety of different transport protocols and Web services standards,
including SOAP, WS-\*, and REST. You can use the 服务总线 relay as a
delegate to listen for incoming sessions and requests sent to a WCF
service. In the relayed messaging pattern, an on-premises or cloud-based
service connects to the relay service through an outbound port and
creates a bi-directional socket for communication tied to a particular
rendezvous address. The client does not have to know where the service
resides, and the on-premises service does not need any inbound ports
open on the firewall. However, depending on your network configuration,
you may encounter problems when connecting to the 服务总线 relay from
behind a firewall or through a proxy server. [Hosting Behind a Firewall
with the 服务总线] describes how to troubleshoot and resolve such
connection issues.

###服务总线 Queues and Topics

服务总线 queues and topics provide brokered messaging
functionality-messages are pushed to the 服务总线 queue or topic
where they are reliably retained until the receiver is ready to consume
them. Message senders and receivers do not have to be online at the same
time; the messaging infrastructure reliably stores messages until the
consuming party is ready to receive them. The messaging API can
encounter a variety of errors that might impact your application. These
can be broadly grouped into the following categories:

-   User error-for example, a code indicating an argument was invalid.
    Recommended action: Try to fix the code before proceeding.
-   Setup/configuration error-for example, a queue or topic associated
    with the action does not exist or has been deleted. Recommended
    action: Review your configuration and change it if necessary.
-   Transient error-for example, a response indicating that the service
    was not able to process the request at the current time. Recommended
    action: Retry the operation or notify users. For more information,
    see [Handling Transient Communication Errors].
-   Other errors-for example, timeout errors or errors indicating that a
    message lock was lost. Recommended action: You generally do not
    handle these exceptions to perform cleanups or aborts. They might be
    used for tracing.

[Messaging Exceptions] provides an overview of exceptions that users of
the 服务总线 client libraries for .NET might encounter, along with
recommendations about how to handle each type of exception. Because the
client libraries for .NET align closely to the structure of the Service
Bus libraries for other languages, this guidance may be useful even if
you are not programming in a .NET language. In some cases, for example
with transient errors, you can retry the action. You can follow the
transient error handling guidelines outlined earlier in this article to
efficiently handle transient errors. In addition, for more details, best
practices, and sample code that demonstrates how to handle transient
服务总线 errors in a .NET application, see the [Handling Transient Communication Errors] section in the Azure Developer Guidance
article [Best Practices for Leveraging Azure 服务总线 Brokered Messaging API].

Another area to focus on when developing an application that uses
brokered messaging is to ensure you implement reliable message receiving
logic that can accurately and efficiently handle anomalies in messages.
The "Implementing Reliable Message Receive Loops" section of the [Best Practices for Leveraging Azure 服务总线 Brokered Messaging API] article describes a number of techniques for using the **PeekLock** receive mode, which is the mode that supports multiple message
deliveries if the message isn't delivered successfully on the first try.
The article recommends best practices that will help ensure your
application does not process duplicate messages. It also helps avoid
problems that can occur due to lock timeouts, and improve overall
performance in **PeekLock** mode by ensuring that you process messages
promptly. The article also provides sample code that uses the Service
Bus client libraries for .NET.

###Additional Troubleshooting Resources

For additional details about common 服务总线 errors and ways to
investigate and address them, see [Troubleshooting the 服务总线].

<!--
##Azure Active Directory Access Control Service (ACS)
Troubleshooting

-   Error codes:
    [http://msdn.microsoft.com/zh-cn/library/windowsazure/gg185949.aspx](http://msdn.microsoft.com/zh-cn/library/windowsazure/gg185949.aspx)
-   ACS Service limitations:
    [http://msdn.microsoft.com/zh-cn/library/windowsazure/gg185909.aspx](http://msdn.microsoft.com/zh-cn/library/windowsazure/gg185909.aspx)

<h2><a id="SQLTroubleshooting"></a>Azure SQL数据库 Troubleshooting</h2>

When interacting with an Azure SQL数据库 extra care must be taken to
deal with the distributed nature of Azure SQL数据库 applications. This
section discusses several areas that warrant attention. This is by no
means an exhaustive list. The key to writing supportable Azure SQL数据库 code
is to examine the return codes and make sure that you have solid retry
code to handle failures.

Your application must handle login failures gracefully. SQL数据库
instances require the use of SQL Authentication. If you cannot
successfully log in, either your credentials are not valid or the
database you requested is not available.

Your application must handle the service being inaccessible. If the
server is already provisioned and the Azure SQL数据库 service is
available (you can check this using the [Azure Health Status] page), the
likely cause is configuration issues in your on-site installation. For
instance, you may be unable to resolve the name (which can be tested
with tools such as tracert), you may have a firewall blocking port 1433
that is used by SQL数据库, or you may be using a proxy server that is
not configured properly. Use the same techniques to troubleshoot these
difficulties that you would for SQL Server. For more information, see [SQL数据库 Connectivity Troubleshooting Guide] and [Troubleshooting SQL数据库].

Your application must handle general network errors. You may receive
general network errors because Azure SQL数据库 might disconnect users
under these circumstances:

-   When a connection is idle for an extended period of time
-   When a connection consumes an excessive amount of resources or holds
    onto a transaction for an extended period of time
-   If the server is too busy

To improve performance in Azure SQL数据库, use the same techniques
you would use with SQL Server. For more information, see the following
topics:

-   [Troubleshooting Queries] in SQL Server Books Online
-   [Troubleshoot and Optimize Queries with SQL数据库]
-   [SQL数据库 Performance Considerations and Troubleshooting]
-   [Improving Your I/O Performance]
-   [System Center Monitoring Pack for SQL数据库]

Azure SQL数据库 uses a subset of SQL Server error messages. For more
information about SQL Server errors, see [Errors and Events Reference
(Database Engine)] in SQL Server Books Online

If you need to recover login names or passwords, contact your service
administrator, who can grant you proper access to the server and
database. Service administrators can also reset their own passwords
using the Azure Management Portal.

SQL数据库 queries can fail for various reasons - a malformed query,
network issues, and so on. Some errors are transient, meaning the
problem often goes away by itself. For this subset of errors, it makes
sense to retry the query after a short interval. If the query still
fails after several retries, you would report an error. Of course, not
all errors are transient. SQL Error 102, "Incorrect syntax," won't go
away no matter how many times you submit the same query. In short,
implementing robust retry logic requires some thought. A tabular data
stream (TDS) error token is sent prior to disconnecting users, when
possible. To improve application experience, we recommend that you
implement the retry logic in your SQL数据库 applications to catch these
errors. When an error occurs, re-establish the connection, and then
re-execute the failed commands or the query. For more information, see
the following links:

-   [Retry Logic for Transient Failures in SQL数据库]
-   [SQL数据库 Retry Logic Sample]
-   [The Transient Fault Handling Application Block]

###Azure SQL Backup and Restore Strategy

Azure SQL数据库 requires its own backup-and-restore strategy because of the
environment and tools available. In many ways the risks have been
mediated by the database being in the Microsoft data centers. The tools
that we have today cover the other risk factors, however better tools
are coming to make the job much easier. Red-gate recently published a
free tool for SQL数据库 backup and restore which can be found here: [http://www.red-gate.com/products/dba/sql-azure-backup/](http://www.red-gate.com/products/dba/sql-azure-backup).

SQL Data Sync enables you to easily create and schedule
bi-directional synchronizations from within the Data Sync web site
without the need to write a single line of code. You can find more
information here:
[http://msdn.microsoft.com/zh-cn/library/windowsazure/hh456371.aspx](http://msdn.microsoft.com/zh-cn/library/windowsazure/hh456371.aspx).

For more information on SQL数据库 backup and Restore strategies see the
following articles:

-   This topic gives an overview of SQL数据库 Backup and Restore
    Strategies:
    [http://social.technet.microsoft.com/wiki/contents/articles/1792.sql-azure-backup-and-restore-strategy.aspx](http://social.technet.microsoft.com/wiki/contents/articles/1792.sql-azure-backup-and-restore-strategy.aspx)
-   This topic explains how to back up a database to another database on
    the same server:
    [http://msdn.microsoft.com/zh-cn/library/windowsazure/ff951631.aspx](http://msdn.microsoft.com/zh-cn/library/windowsazure/ff951631.aspx)
-   This topic explains how to export an existing SQL数据库 instance to
    a blob on a given storage account:
    [http://msdn.microsoft.com/zh-cn/library/windowsazure/hh335292.aspx](http://msdn.microsoft.com/zh-cn/library/windowsazure/hh335292.aspx)
-   This topic explains how to import an existing SQL数据库 instance
    from a bacpac file stored in a blob:
    [http://msdn.microsoft.com/zh-cn/library/windowsazure/hh335291.aspx](http://msdn.microsoft.com/zh-cn/library/windowsazure/hh335291.aspx)
-   This topic describes the business continuity capabilities provided
    by SQL数据库:
    [http://msdn.microsoft.com/zh-cn/library/windowsazure/hh852669.aspx](http://msdn.microsoft.com/zh-cn/library/windowsazure/hh852669.aspx)

<h2><a id="Cache"></a>Azure Caching</h2>
Azure Caching comes in two flavors: the Azure Shared Caching and role-based Azure Caching (Preview). Shared Caching is a multitenent Azure service that provides caching services. Azure Caching (Preview) hosts caching on a role by using a portion of the memory from the virtual machines that host your role instances. To troubleshoot Azure Caching, observe the behavior of the cache by checking error codes and catching exceptions. When using role-based Caching (Preview),you can also use performance counters. Caching problems generally fall into one of the following categories:

- 	Quota-related errors - a quota has been exceeded (Shared Caching)
- 	Throttling - occurs when there is not enough physical memory to support additional cached items
- 	Eviction - items are forcibly evicted to make room for new items in a way that hurts application performance
- 	Expiration - expiration times are set too short or long

For more information on quota-related errors, see [Understanding Quotas].
-->




















[Monitoring Pack]: http://www.microsoft.com/download/en/details.aspx?id=11324
[Microsoft System Center Operations Manager]: http://www.microsoft.com/zh-cn/server-cloud/products/system-center-2012-r2/default.aspx#fbid=9ZH5wGSDgf0
[Azure Platform Management Tool (MMC)]: http://wapmmc.codeplex.com/
[Azure Diagnostics Manager]: http://cerebrata.com/Products/AzureDiagnosticsManager/
[Cloud Storage Studio]: http://cerebrata.com/Products/CloudStorageStudio/
[Azure Management Cmdlets]: http://cerebrata.com/Products/AzureManagementCmdlets/
[Gomez]: http://www.compuware.com/application-performance-management/
[Keynote]: http://www.keynote.com/solutions/
[Pingdom]: http://pingdom.com/
[AlertBot]: http://www.alertbot.com/products/website-monitoring/default.aspx
[IntelliTrace]: http://msdn.microsoft.com/zh-cn/library/dd264915.aspx
[startup task]: http://msdn.microsoft.com/zh-cn/library/gg456327.aspx
[AVIcode]: http://www.microsoft.com/zh-cn/server-cloud/system-center/avicode.aspx
[profile]: http://msdn.microsoft.com/zh-cn/library/windowsazure/hh369930.aspx
[VM Assistant]: http://azurevmassist.codeplex.com/
[Debugging Node in the Azure Emulator]: http://weblogs.asp.net/jimwang/archive/2012/04/17/debugging-node-node-inspector-in-the-azure-emulator.aspx
[Debugging with Node-Inspector]: http://howtonode.org/debugging-with-node-inspector

[IntelliTrace to debug Azure Cloud Services]: http://blogs.msdn.com/b/jnak/archive/2010/06/07/using-intellitrace-to-debug-windows-azure-cloud-services.aspx

[Troubleshooting an Azure web site]: /develop/net/best-practices/troubleshooting-web-sites/
[W3C extended log file format]: http://go.microsoft.com/fwlink/?LinkID=90561
[Log Parser]: http://go.microsoft.com/fwlink/?LinkId=246619
[Log Parser Plus]: http://logparserplus.com/Examples
[TechNet Log Parser Examples]: http://technet.microsoft.com/zh-cn/library/ee692659.aspx
[Office 2003 Web Components]: http://www.microsoft.com/zh-cn/download/details.aspx?id=22276
[How to: Debug Windows Service Applications]: http://msdn.microsoft.com/zh-cn/library/7a50syb3%28v=vs.90%29.aspx
[Daniel Pearson's presentation Windows Debugging and Troubleshooting]: http://technet.microsoft.com/zh-cn/video/Video/hh867800
[Suse Linux documentation]: https://www.suse.com/documentation/
[list of storage explorers]: http://blogs.msdn.com/b/windowsazurestorage/archive/2010/04/17/windows-azure-storage-explorers.aspx
[Explorer for Azure Blob Storage]: http://www.cloudberrylab.com/free-microsoft-azure-explorer.aspx
[enterprise 服务总线]: http://en.wikipedia.org/wiki/Enterprise_service_bus
[Azure 服务总线 Quotas]: http://msdn.microsoft.com/zh-cn/library/windowsazure/ee732538.aspx
[服务总线 Explorer]: http://code.msdn.microsoft.com/Service-Bus-Explorer-f2abca5a
[Hosting Behind a Firewall with the 服务总线]: http://msdn.microsoft.com/zh-cn/library/windowsazure/ee706729.aspx
[Handling Transient Communication Errors]: http://msdn.microsoft.com/zh-cn/library/hh851746(VS.103).aspx
[Messaging Exceptions]: http://msdn.microsoft.com/zh-cn/library/hh418082.aspx
[Handling Transient Communication Errors]: http://msdn.microsoft.com/zh-cn/library/hh851746(VS.103).aspx
[Best Practices for Leveraging Azure 服务总线 Brokered Messaging API]: http://msdn.microsoft.com/zh-cn/library/windowsazure/hh545245.aspx
[Troubleshooting the 服务总线]: http://msdn.microsoft.com/zh-cn/library/windowsazure/ee706702.aspx
[Azure Health Status]: http://go.microsoft.com/fwlink/p/?LinkId=168847
[SQL数据库 Connectivity Troubleshooting Guide]: http://social.technet.microsoft.com/wiki/contents/articles/sql-azure-connectivity-troubleshooting-guide.aspx
[Troubleshooting SQL数据库]: http://msdn.microsoft.com/zh-cn/library/ee730906.aspx
[Troubleshooting Queries]: http://msdn.microsoft.com/zh-cn/library/ms186351(SQL.100).aspx
[Troubleshoot and Optimize Queries with SQL数据库]: http://social.technet.microsoft.com/wiki/contents/articles/1104.troubleshoot-and-optimize-queries-with-sql-azure.aspx
[SQL数据库 Performance Considerations and Troubleshooting]: http://channel9.msdn.com/Events/TechEd/NorthAmerica/2011/DBI314
[Improving Your I/O Performance]: http://blogs.msdn.com/b/sqlazure/archive/2010/07/27/10043069.aspx?PageIndex=2#comments
[System Center Monitoring Pack for SQL数据库]: http://www.microsoft.com/zh-cn/download/details.aspx?id=10631
[Errors and Events Reference (Database Engine)]: http://go.microsoft.com/fwlink/p/?LinkId=166622
[Retry Logic for Transient Failures in SQL数据库]: http://social.technet.microsoft.com/wiki/contents/articles/4235.retry-logic-for-transient-failures-in-sql-azure.aspx
[SQL数据库 Retry Logic Sample]: http://code.msdn.microsoft.com/windowsazure/SQL-Azure-Retry-Logic-2d0a8401
[The Transient Fault Handling Application Block]: http://msdn.microsoft.com/zh-cn/library/hh680934(PandP.50).aspx
[Microsoft Enterprise Library 5.0 Integration Pack for Azure]: http://msdn.microsoft.com/zh-cn/library/hh680918%28v=pandp.50%29.aspx
[Take Control of Logging and Tracing in Azure]: http://msdn.microsoft.com/zh-cn/magazine/ff714589.aspx
[Cloud Application Framework & Extensions (CloudFx)]: http://nuget.org/packages/Microsoft.Experience.CloudFx
[Fiddler]: http://www.fiddler2.com/fiddler2/
[Understanding Quotas]: http://msdn.microsoft.com/zh-cn/library/gg185683.aspx
[Troubleshooting Cache]: http://go.microsoft.com/fwlink/?LinkId=252730 
