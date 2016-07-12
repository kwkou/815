<properties
	pageTitle="监视、诊断和排查存储空间问题 | Azure"
	description="使用存储分析、客户端日志记录等功能及其他第三方工具来确定、诊断和排查与 Azure 存储空间相关的问题。"
	services="storage"
	documentationCenter=""
	authors="jasonnewyork"
	manager="tadb"
	editor=""/>

<tags
	ms.service="storage"
	ms.date="04/29/2016"
	wacn.date="06/06/2016"/>

# 监视、诊断和排查 Azure 存储空间问题

[AZURE.INCLUDE [storage-selector-portal-monitoring-diagnosing-troubleshooting](../includes/storage-selector-portal-monitoring-diagnosing-troubleshooting.md)]

## 概述

诊断和排查在云环境中托管的分布式应用程序中的问题可能会比在传统环境中更复杂。应用程序可以部署在 PaaS 或 IaaS 基础结构、本地、移动设备，或这些项的某种组合中。通常，应用程序的网络流量可能会经过公用和专用网络，你的应用程序可以使用多种存储技术（如 Azure 存储表、Blob、队列或文件）以及其他数据存储（如关系数据库和文档数据库）。

若要成功管理此类应用程序，应主动监视这些应用程序，并了解如何诊断和排查这些应用程序及其相关技术的所有方面的问题。作为 Azure 存储服务的用户，你应持续监视你的应用程序所用的存储服务是否出现任何意外的行为更改（如比正常响应时间慢），并使用日志记录收集更详细的数据并深入地分析问题。从监视和日志记录获取的诊断信息将有助于确定应用程序所遇到问题的根本原因。然后，你可以排查该问题，并确定可以执行以更正该问题的相应步骤。Azure 存储空间是一项核心 Azure 服务，它是客户部署到 Azure 基础结构的大多数解决方案的重要组成部分。Azure 存储空间提供的功能可以简化监视、诊断和排查基于云的应用程序中的存储问题的过程。

有关 Azure 存储空间应用程序中的端到端故障排除的动手指导，请参阅[端到端故障排除 - 使用 Azure 存储空间指标和日志记录、AzCopy 和 Message Analyzer](/documentation/articles/storage-e2e-troubleshooting/)。

+ [介绍]
	+ [本指南的组织方式]
+ [监视存储服务]
	+ [监视服务运行状况]
	+ [监视容量]
	+ [监视可用性]
	+ [监视性能]
+ [诊断存储空间问题]
	+ [服务运行状况问题]
	+ [性能问题]
	+ [诊断错误]
	+ [存储模拟器问题]
	+ [存储日志记录工具]
	+ [使用网络日志记录工具]
+ [端到端跟踪]
	+ [关联日志数据]
	+ [客户端请求 ID]
	+ [服务器请求 ID]
	+ [时间戳]
+ [故障排除指南]
	+ [度量值显示高 AverageE2ELatency 和低 AverageServerLatency]
	+ [度量值显示低 AverageE2ELatency 和低 AverageServerLatency，但客户端遇到高延迟]
	+ [度量值显示高 AverageServerLatency]
	+ [队列上的消息传递出现意外的延迟]
	+ [度量值显示 PercentThrottlingError 增加]
	+ [度量值显示 PercentTimeoutError 增加]
	+ [度量值显示 PercentNetworkError 增加]
	+ [客户端正在接收“HTTP 403 (禁止访问)”消息]
	+ [客户端正在接收“HTTP 404 (未找到)”消息]
	+ [客户端正在接收“HTTP 409 (冲突)”消息]
	+ [度量值显示低 PercentSuccess，或者分析日志项包含事务状态为 ClientOtherErrors 的操作]
	+ [容量度量值显示存储容量使用量意外增加]
	+ [遇到具有大量附加 VHD 的虚拟机意外重新启动]
	+ [你的问题是由于使用存储模拟器进行开发或测试而导致]
	+ [安装 Azure SDK for .NET 时遇到问题]
	+ [你遇到了其他存储服务问题]
+ [附录]
	+ [附录 1：使用 Fiddler 捕获 HTTP 和 HTTPS 通信]
	+ [附录 2：使用 Wireshark 捕获网络流量]
	+ [附录 3：使用 Microsoft Message Analyzer 捕获网络流量]
	+ [附录 4：使用 Excel 查看度量值和日志数据]
	+ [附录 5：使用 Application Insights for Visual Studio Team Services 进行监视]

## <a name="introduction"></a>介绍

本指南演示如何使用 Azure 存储客户端库中的 Azure 存储分析、客户端日志记录等功能及其他第三方工具来确定、诊断和排查与 Azure 存储空间相关的问题。

![][1]

*图 1：监视、诊断和故障排除*

本指南的主要目标受众是开发使用 Azure 存储服务的联机服务的开发人员以及负责管理此类联机服务的 IT 专业人员。本指南的目标是：

- 帮助你维护 Azure 存储帐户的运行状况和性能。
- 为你提供必要的过程和工具来帮助你确定应用程序中的问题是否与 Azure 存储空间有关。
- 为你提供用于解决与 Azure 存储空间相关的问题的可操作指南。

### <a name="how-this-guide-is-organized"></a>本指南的组织方式

“[监视存储服务]”一节介绍如何使用 Azure 存储分析度量值（存储度量值）监视 Azure 存储服务的运行状况和性能。

“[诊断存储空间问题]”一节介绍如何使用 Azure 存储分析日志记录（存储日志记录）诊断问题。它还介绍了如何使用其中一个客户端库（如 .NET 存储客户端库或 Azure SDK for Java）中的工具启用客户端日志记录。

“[端到端跟踪]”一节介绍了如何关联各种日志文件中包含的信息和指标数据。

“[故障排除指南]”一节针对您可能会遇到的一些与存储相关的常见问题，提供了故障排除指南。

“[附录]”提供了有关如何使用其他工具的信息，例如如何使用 Wireshark 和 Netmon 分析网络数据包数据，如何使用 Fiddler 分析 HTTP/HTTPS 消息，以及如何使用 Microsoft Message Analyzer 关联日志数据等。


## <a name="monitoring-your-storage-service"></a>监视存储服务

如果你熟悉 Windows 性能监视，则可以将存储度量值视为 Windows 性能监视器计数器的 Azure 存储空间等效项。在存储度量值中您将找到一组综合度量值（相当于 Windows 性能监视器术语中的计数器），例如服务可用性、向服务发送的请求总数或向服务发出的成功请求的百分比（有关可用度量值的完整列表，请参阅 MSDN 上的<a href="http://msdn.microsoft.com/zh-cn/library/azure/hh343264.aspx" target="_blank">存储分析度量表架构</a>）。你可以指定希望存储服务每隔一小时还是每隔一分钟收集和聚合一次度量值。有关如何启用度量值和监视存储帐户的详细信息，请参阅 MSDN 上的<a href="https://msdn.microsoft.com/zh-cn/library/dn782843.aspx" target="_blank">启用存储度量值</a>。

你可以选择要将哪些每小时度量值显示在 Azure 经典门户中，并配置规则以便在每小时度量值超过特定阈值时，通过电子邮件通知管理员（有关详细信息，请参阅页面<a href="http://msdn.microsoft.com/zh-cn/library/azure/dn306638.aspx" target="_blank">如何：在 Azure 中接收警报通知和管理警报规则</a>）。存储服务将尽最大努力收集度量值，但可能无法记录每个存储操作。

下面的图 2 显示了 Azure 经典门户中的“监视”页，你可以在其中查看存储帐户的度量值，如可用性、请求总数和平均延迟数。也已设置通知规则，以便在可用性下降到低于某个级别时向管理员发出警报。通过查看此数据，一个可能的调查方面是表服务成功百分比低于 100%（有关详细信息，请参阅“[度量值显示低 PercentSuccess，或者分析日志项包含事务状态为 ClientOtherErrors 的操作]”一节）。

![][2]

*图 2 在 Azure 经典门户中查看存储度量值*

你应通过以下方式持续监视 Azure 应用程序以确保它们正常运行并按预期执行操作：

- 为应用程序建立一些基准指标，以用于比较当前数据和识别 Azure 存储空间和你的应用程序的任何重大行为更改。在许多情况下，基准指标的值将特定于应用程序，因此应在对应用程序进行性能测试时建立这些指标。
- 记录每分钟度量值，并使用这些度量值来主动监视意外的错误和异常，例如错误计数或请求速率达到峰值。
- 记录每小时度量值，并使用这些度量值来监视平均值，例如平均错误计数和请求速率。
- 使用诊断工具调查潜在问题，如稍后在“[诊断存储空间问题]”中所述。

下面的图 3 中的图表说明了对每小时度量值进行的求平均值操作如何可能隐藏活动达到峰值。每小时度量值似乎显示稳定的请求速率，而每分钟度量值却显示了实际发生的波动。

![][3]

本节的剩余部分将介绍应监视哪些度量值以及监视原因。

### <a name="monitoring-service-health"></a>监视服务运行状况

可以使用 [Azure 经典门户](https://manage.windowsazure.cn)查看全球所有 Azure 区域中存储服务（及其他 Azure 服务）的运行状况。这使你可以立即了解是否有不受你控制的问题正在影响你的应用程序所使用的区域中的存储服务。

此外，Azure 管理门户还可以提供影响各种 Azure 服务的事件的通知。注意：此信息以前已在 Azure 服务仪表板（网址为：<a href="/support/service-dashboard/" target="_blank">https://www.azure.cn/support/service-dashboard/</a>）上与历史数据一起提供。

虽然 Azure 经典门户从 Azure 数据中心内部收集运行状况信息（由内而外监视），但你也可以考虑采用由外而内方法来生成定期从多个位置访问 Azure 托管的 Web 应用程序的综合事务。<a href="http://www.keynote.com/solutions/monitoring/web-monitoring" target="_blank">Keynote</a>、<a href="https://www.gomeznetworks.com/?g=1" target="_blank">Gomez</a> 和 Application Insights for Visual Studio Team Services 提供的服务是这种由外而内方法的示例。有关 Application Insights for Visual Studio Team Services 的详细信息，请参阅附录“[附录 5：使用 Application Insights for Visual Studio Team Services 进行监视]”。

### <a name="monitoring-capacity"></a>监视容量

存储度量值仅存储 Blob 服务的容量度量值，因为 Blob 通常占所存储数据的最大比例（撰写本文时，尚不能使用存储度量值来监视表和队列的容量）。如果您已为 Blob 服务启用监视，则可以在 **$MetricsCapacityBlob** 表中找到此数据。存储度量值每天记录一次此数据，然后您可以使用 **RowKey** 的值来确定某行是否包含与用户数据（值 **data**）或分析数据（值 **analytics**）相关的实体。每个存储的实体均包含有关所用的存储量（**Capacity**，以字节为单位）、当前的容器数 (**ContainerCount**) 以及存储帐户中正在使用的 Blob 数 (**ObjectCount**) 的信息。有关 **$MetricsCapacityBlob** 表中存储的容量度量值的详细信息，请参阅 MSDN 上的<a href="http://msdn.microsoft.com/zh-cn/library/azure/hh343264.aspx" target="_blank">存储分析度量表架构</a>。

> [AZURE.NOTE] 你应监视这些值以便获取“你已接近存储帐户的容量限制”的早期警告。在 Azure 经典门户中你的存储帐户的“监视”页上，你可以添加警报规则，以便在聚合存储使用量超过或低于你指定的阈值时通知你。

若要帮助估算各种存储对象（如 Blob）的大小，请参阅博客文章<a href="http://blogs.msdn.com/b/windowsazurestorage/archive/2010/07/09/understanding-windows-azure-storage-billing-bandwidth-transactions-and-capacity.aspx" target="_blank">了解 Azure 存储空间计费 - 带宽、事务和容量</a>。

### <a name="monitoring-availability"></a>监视可用性

您应通过监视以下每小时或每分钟度量值表中的 **Availability** 列中的值来监视存储帐户中存储服务的可用性：**$MetricsHourPrimaryTransactionsBlob**、**$MetricsHourPrimaryTransactionsTable**、**$MetricsHourPrimaryTransactionsQueue**、**$MetricsMinutePrimaryTransactionsBlob**、**$MetricsMinutePrimaryTransactionsTable**、**$MetricsMinutePrimaryTransactionsQueue**、**$MetricsCapacityBlob**。**Availability** 列包含一个百分比值，指示该服务的可用性或该行所表示的 API 操作的可用性（**RowKey** 显示行是包含整体服务的度量值还是包含特定 API 操作的度量值）。

任何小于 100% 的值指示某些存储请求将失败。您可以通过检查度量值数据中显示具有不同错误类型（如 **ServerTimeoutError**）的请求数的其他列来了解失败原因。由于以下原因您应该会看到 **Availability** 暂时低于 100%：比如在该服务移动分区以更好地负载平衡请求时，出现暂时性服务器超时；客户端应用程序中的重试逻辑应处理此类间歇性情况。页面 <a href="http://msdn.microsoft.com/zh-cn/library/azure/hh343260.aspx" target="_blank"></a> 列出了存储度量值在其“可用性”计算中包括的事务类型。

在 Azure 经典门户中你的存储帐户的“监视”页上，你可以添加警报规则，以便在服务的“可用性”低于你指定的阈值时通知你。

本指南的“[故障排除指南]”一节将介绍与可用性相关的一些常见存储服务问题。

### <a name="monitoring-performance"></a>监视性能

若要监视存储服务的性能，可以使用每小时和每分钟度量值表中的以下度量值。

- **AverageE2ELatency** 和 **AverageServerLatency** 中的值显示存储服务或 API 操作类型处理请求所需的平均时间。**AverageE2ELatency** 是端到端延迟的度量值，除了包括处理请求所需的时间外，还包括读取请求和发送响应所需的时间（因此包括请求到达存储服务后的网络延迟）；**AverageServerLatency** 只是处理时间的度量值，因此不包括与客户端通信相关的任何网络延迟。有关这两个值之间可能存在显著区别的原因的讨论，请参阅本指南后面的“[度量值显示高 AverageE2ELatency 和低 AverageServerLatency]”一节。
- **TotalIngress** 和 **TotalEgress** 列中的值显示进出存储服务或通过特定 API 操作类型的数据总量（以字节为单位）。
- **TotalRequests** 列中的值显示存储服务的 API 操作正在接收的请求总数。**TotalRequests** 是存储服务收到的请求总数。

通常，对于作为出现需要调查的问题的指示器的任何这些值，你将监视其意外更改。

在 Azure 经典门户中你的存储帐户的“监视”页上，你可以添加警报规则，以便在此服务的任何性能度量值低于或超过你指定的阈值时通知你。

本指南的“[故障排除指南]”一节将介绍与性能相关的一些常见存储服务问题。


## <a name="diagnosing-storage-issues"></a>诊断存储空间问题

有多种方式可能让你意识到你的应用程序有问题，这包括：

- 导致应用程序崩溃或停止工作的严重故障。
- 正在监视的度量值与基准值相比发生重大更改，如上一节“[监视存储服务]”中所述。
- 根据应用程序的用户报告，某个特定操作未按预期完成，或者某项功能无法正常工作。
- 在日志文件中或通过某种其他通知方法显示应用程序中生成的错误。

通常，与 Azure 存储服务相关的问题分为以下四大类之一：

- 你的应用程序出现性能问题，该问题由你的用户报告或由性能度量值的更改显示。
- 一个或多个区域中的 Azure 存储基础结构存在问题。
- 你的应用程序遇到错误，该错误由你的用户报告或由你监视的其中一个错误计数度量值增加显示。
- 在开发和测试期间，你可能使用的是本地存储模拟器；你可能会遇到一些与使用存储模拟器特别相关的问题。

以下各节概述了诊断和排查这四大类中的每一类问题时应遵循的步骤。在本指南后面的“[故障排除指南]”一节将提供有关您可能会遇到的一些常见问题的更多详细信息。

### <a name="service-health-issues"></a>服务运行状况问题

服务运行状况问题通常不受你控制。Azure 经典门户提供了有关 Azure 服务（包括存储服务）当前存在的任何问题的信息。如果你在创建存储帐户时选择了启用读取访问地域冗余存储，则在主位置中的数据不可用时，你的应用程序可以暂时切换到辅助位置中的只读副本。为此，您的应用程序必须能够在使用主存储位置和辅助存储位置之间切换，并能够在降低的功能模式下使用只读数据。使用 Azure 存储客户端库，可以定义重试策略，以便在从主存储读取失败时可以从辅助存储读取。你的应用程序还需要注意辅助位置中的数据是否最终一致。有关详细信息，请参阅博客文章<a href="http://blogs.msdn.com/b/windowsazurestorage/archive/2013/12/04/introducing-read-access-geo-replicated-storage-ra-grs-for-windows-azure-storage.aspx" target="_blank">Azure 存储冗余选项和读取访问地域冗余存储</a>。

### <a name="performance-issues"></a>性能问题

应用程序的性能可能是主观判定的，尤其是从用户角度来看，更是如此。因此，请务必设置可用于帮助你确定是否可能存在性能问题的基准指标。从客户端应用程序角度来看，有许多因素可能会影响 Azure 存储服务的性能。这些因素可能在存储服务、客户端或网络基础结构中运作；因此必须设置策略来确定性能问题的源。

在通过度量值确定性能问题的可能根源位置后，可以使用日志文件查找详细信息以便进一步诊断并排查该问题。

在本指南后面的“[故障排除指南]”一节将提供有关您可能会遇到的一些常见性能相关问题的详细信息。

### <a name="diagnosing-errors"></a>诊断错误

应用程序用户可能会向你通知客户端应用程序报告的错误。存储度量值还会记录来自存储服务的不同错误类型（如 **NetworkError**、**ClientTimeoutError** 或 **AuthorizationError**）的计数。虽然存储度量值仅记录不同错误类型的计数，但你可以通过检查服务器端日志、客户端日志和网络日志来获取有关单个请求的详细信息。通常，存储服务返回的 HTTP 状态代码将指示请求失败的原因。

> [AZURE.NOTE] 请记住，你应该会看到一些间歇性错误：例如，因暂时性的网络状况导致的错误或应用程序错误。

MSDN 上的以下资源对了解与存储相关的状态和错误代码很有帮助：

- <a href="http://msdn.microsoft.com/zh-cn/library/azure/dd179357.aspx" target="_blank">常见的 REST API 错误代码</a>
- <a href="http://msdn.microsoft.com/zh-cn/library/azure/dd179439.aspx" target="_blank">Blob 服务错误代码</a>
- <a href="http://msdn.microsoft.com/zh-cn/library/azure/dd179446.aspx" target="_blank">队列服务错误代码</a>
- <a href="http://msdn.microsoft.com/zh-cn/library/azure/dd179438.aspx" target="_blank">表服务错误代码</a>

### <a name="storage-emulator-issues"></a>存储模拟器问题

Azure SDK 提供了一个存储模拟器，你可以在开发工作站上运行它。该模拟器可模拟 Azure 存储服务的大多数行为，因此在开发和测试期间很有用，使你能够无需 Azure 订阅和 Azure 存储帐户即可运行使用 Azure 存储服务的应用程序。

本指南的“[故障排除指南]”一节将介绍使用存储模拟器时遇到的一些常见问题。

### <a name="storage-logging-tools"></a>存储日志记录工具

存储日志记录为 Azure 存储帐户中的存储请求提供服务器端日志记录功能。有关如何启用服务器端日志记录和访问日志数据的详细信息，请参阅 MSDN 上的<a href="https://msdn.microsoft.com/zh-cn/library/dn782840.aspx" target="_blank">使用服务器端日志记录</a>。

使用 .NET 存储客户端库可以收集与应用程序执行的存储操作相关的客户端日志数据。有关如何启用客户端日志记录和访问日志数据的详细信息，请参阅 MSDN 上的<a href="https://msdn.microsoft.com/zh-cn/library/dn782839.aspx" target="_blank">使用存储客户端库进行客户端日志记录</a>。

> [AZURE.NOTE] 在某些情况下（如 SAS 授权失败），用户可能会报告一个错误，而你可能在服务器端存储日志中未找到该错误所对应的请求数据。你可以使用存储客户端库的日志记录功能来调查该问题的原因是否出在客户端上，或者使用网络监视工具来调查网络。

### <a name="using-network-logging-tools"></a>使用网络日志记录工具

你可以捕获客户端和服务器之间的流量，以便提供有关客户端和服务器正在交换的数据以及底层网络状况的详细信息。有用的网络日志记录工具包括：

- Fiddler (<a href="http://www.telerik.com/fiddler" target="_blank">http://www.telerik.com/fiddler</a>) 是一个免费 Web 调试代理，使用它可以检查 HTTP 和 HTTPS 请求和响应消息的标头和负载数据。有关详细信息，请参阅“[附录 1：使用 Fiddler 捕获 HTTP 和 HTTPS 通信]”。
- Microsoft 网络监视器 (Netmon) (<a href="http://www.microsoft.com/download/details.aspx?id=4865" target="_blank">http://www.microsoft.com/download/details.aspx?id=4865</a>) 和 Wireshark (<a href="http://www.wireshark.org/" target="_blank">http://www.wireshark.org/</a>) 是免费的网络协议分析器，使用它们可以查看各种网络协议的详细数据包信息。有关 Wireshark 的详细信息，请参阅“[附录 2：使用 Wireshark 捕获网络流量]”。
- Microsoft Message Analyzer 是 Microsoft 提供的用于取代 Netmon 的工具，它除了捕获网络数据包数据外，还可帮助你查看和分析其他工具捕获的日志数据。有关详细信息，请参阅“[附录 3：使用 Microsoft Message Analyzer 捕获网络流量]”。
- 如果您要执行基本连接测试，以检查客户端计算机是否能够通过网络连接到 Azure 存储服务，您不能在客户端上使用是标准 **ping** 工具来执行此操作。但是，您可以使用 **tcping** 工具来检查连接性。**Tcping** 在 <a href="http://www.elifulkerson.com/projects/tcping.php" target="_blank">http://www.elifulkerson.com/projects/tcping.php</a> 上提供下载。

在许多情况下，通过存储日志记录和存储客户端库记录的日志数据已足以诊断问题，但在某些情况下，你可能需要更详细的信息，而这些网络日志记录工具可以提供这些信息。例如，使用 Fiddler 查看 HTTP 和 HTTPS 消息时，你可以查看发往和来自存储服务的标头和负载数据，这使你能够检查客户端应用程序如何重试存储操作。协议分析器（例如 Wireshark）运行在数据包级别，这使你能够查看 TCP 数据，从而可以排查丢失的数据包和连接问题。Message Analyzer 可以在 HTTP 和 TCP 层上运行。

## <a name="end-to-end-tracing"></a>端到端跟踪

使用各种日志文件的端到端跟踪是一项有用的技术，用于调查潜在的问题。可以使用度量数据中的日期/时间信息来指示在日志文件中查找有助于排查问题的详细信息的起始位置。

### <a name="correlating-log-data"></a>关联日志数据

查看客户端应用程序、网络跟踪、服务器端存储日志记录中的日志时，能够关联不同日志文件中的请求至关重要。日志文件包含许多不同的字段，这些字段可用作关联标识符。客户端请求 ID 是最有用的字段，用于关联不同日志中的条目。但是,有时，使用服务器请求 ID 或时间戳会很有用。以下各节提供了有关这些选项的更多详细信息。

### <a name="client-request-id"></a>客户端请求 ID

存储客户端库会自动为每个请求生成唯一的客户端请求 ID。

- 在存储客户端库创建的客户端日志中，客户端请求 ID 将出现在与请求相关的每个日志项的“客户端请求 ID”字段中。
- 在网络跟踪（如 Fiddler 捕获的网络跟踪）中，客户端请求 ID 在请求消息中作为 **x-ms-client-request-id** HTTP 标头值可见。
- 在服务器端存储日志记录日志中，客户端请求 ID 将出现在“客户端请求 ID”列中。

> [AZURE.NOTE] 多个请求可以共享同一客户端请求 ID，因为客户端可以分配此值（尽管存储客户端库会自动分配一个新值）。在从客户端的重试的情况下的所有尝试都共享相同的客户端请求 ID。如果从客户端发送批处理，批处理有单个客户端请求 ID。


### <a name="server-request-id"></a>服务器请求 ID

存储服务会自动生成服务器请求 ID。

- 在服务器端存储日志记录日志中，服务器请求 ID 将出现在“请求 ID 标头”列中。
- 在网络跟踪（如 Fiddler 捕获的网络跟踪）中，服务器请求 ID 将作为 **x-ms-request-id** 标头值出现在响应消息中。
- 在存储客户端库创建的客户端日志中，服务器请求 ID 将出现在显示服务器响应详细信息的日志项的“操作文本”列中。

> [AZURE.NOTE] 存储服务始终为它接收的每个请求分配唯一的服务器请求 ID，l因此客户端进行的每次重试尝试和批处理中包含的每个操作均使用唯一的服务器请求 ID。

如果存储客户端库在客户端上引发 **StorageException**，则 **RequestInformation** 属性将包含 **RequestResult** 对象（其中包含 **ServiceRequestID** 属性）。您也可以通过 **OperationContext** 实例访问 **RequestResult** 对象。

下面的代码示例演示如何通过附加 **OperationContext** 对象（向存储服务发出的请求）设置自定义 **ClientRequestId** 值。它还演示了如何从响应消息中检索 **ServerRequestId** 值。

    CloudStorageAccount storageAccount = CloudStorageAccount.Parse(CloudConfigurationManager.GetSetting("StorageConnectionString"));
    CloudBlobClient blobClient = storageAccount.CreateCloudBlobClient();

    // Create an Operation Context that includes custom ClientRequestId string based on constants defined within the application along with a Guid.
    OperationContext oc = new OperationContext();
    oc.ClientRequestID = String.Format("{0} {1} {2} {3}", HOSTNAME, APPNAME, USERID, Guid.NewGuid().ToString());

    try
    {
        CloudBlobContainer container = blobClient.GetContainerReference("democontainer");
        ICloudBlob blob = container.GetBlobReferenceFromServer("testImage.jpg", null, null, oc);  
        var downloadToPath = string.Format("./{0}", blob.Name);
        using (var fs = File.OpenWrite(downloadToPath))
        {
            blob.DownloadToStream(fs, null, null, oc);
            Console.WriteLine("\t Blob downloaded to file: {0}", downloadToPath);
        }
    }
    catch (StorageException storageException)
    {
        Console.WriteLine("Storage exception {0} occurred", storageException.Message);
        // Multiple results may exist due to client side retry logic - each retried operation will have a unique ServiceRequestId
        foreach (var result in oc.RequestResults)
        {
                Console.WriteLine("HttpStatus: {0}, ServiceRequestId {1}", result.HttpStatusCode, result.ServiceRequestID);
        }
    }


### <a name="timestamps"></a>时间戳

你还可以使用时间戳来查找相关日志项，但应注意客户端和服务器之间可能存在的时钟偏差。在客户端上基于时间戳搜索服务器端的相应条目时，应加/减 15 分钟。请记住，包含度量值的 Blob 的 Blob 元数据指示 Blob 中存储的度量值的时间范围；如果你在同一分钟或小时内使用多个度量值 Blob，则此信息会很有用。

## <a name="troubleshooting-guidance"></a>故障排除指南

本节将帮助你诊断和排查在使用 Azure 存储服务时你的应用程序可能会遇到的一些常见问题。使用下面的列表来查找与你的具体问题相关的信息。

**故障排除决策树**

----------

你的问题是否与其中一个存储服务的性能相关？

- [度量值显示高 AverageE2ELatency 和低 AverageServerLatency]
- [度量值显示低 AverageE2ELatency 和低 AverageServerLatency，但客户端遇到高延迟]
- [度量值显示高 AverageServerLatency]
- [队列上的消息传递出现意外的延迟]

----------

你的问题是否与其中一个存储服务的可用性相关？

- [度量值显示 PercentThrottlingError 增加]
- [度量值显示 PercentTimeoutError 增加]
- [度量值显示 PercentNetworkError 增加]

----------

客户端应用程序是否从存储服务收到 HTTP 4XX（如 404）响应？

- [客户端正在接收“HTTP 403 (禁止访问)”消息]
- [客户端正在接收“HTTP 404 (未找到)”消息]
- [客户端正在接收“HTTP 409 (冲突)”消息]

----------

[度量值显示低 PercentSuccess，或者分析日志项包含事务状态为 ClientOtherErrors 的操作]

----------

[容量度量值显示存储容量使用量意外增加]

----------

[遇到具有大量附加 VHD 的虚拟机意外重新启动]

----------

[你的问题是由于使用存储模拟器进行开发或测试而导致]

----------

[安装 Azure SDK for .NET 时遇到问题]

----------

[你遇到了其他存储服务问题]

----------

### <a name="metrics-show-high-AverageE2ELatency-and-low-AverageServerLatency"></a>度量值显示高 AverageE2ELatency 和低 AverageServerLatency

下面的来自 Azure 经典门户监视工具的插图显示了一个示例，其中 **AverageE2ELatency** 明显高于 **AverageServerLatency**。

![][4]

请注意，存储服务仅对成功的请求计算度量值 **AverageE2ELatency**，与 **AverageServerLatency** 不同，它包括客户端发送数据及从存储服务接收确认所需的时间。因此，**AverageE2ELatency** 和 **AverageServerLatency** 之间的差异可能是由于客户端应用程序响应速度慢，或者是由网络上的情况导致的。

> [AZURE.NOTE]您还可以在存储日志记录日志数据中查看单个存储操作的 **E2ELatency** 和 **ServerLatency**。

#### 调查客户端的性能问题

客户端响应速度慢的可能原因包括：可用连接数或可用线程数有限。你可以通过以下方式解决此问题：修改客户端代码使其更高效（例如，对存储服务使用异步调用），或者使用更大的虚拟机（包含更多内核和更多内存）。

对于表和队列服务，Nagle 算法也可能会导致高 **AverageE2ELatency**（与 **AverageServerLatency** 相比）：有关详细信息，请参阅 Azure 存储空间团队博客上的文章 <a href="http://blogs.msdn.com/b/windowsazurestorage/archive/2010/06/25/nagle-s-algorithm-is-not-friendly-towards-small-requests.aspx" target="_blank">Nagle 算法对小请求不友好</a>。您可以通过使用 **System.Net** 命名空间中的 **ServicePointManager** 类在代码中禁用 Nagle 算法。应在应用程序中调用表或队列服务之前执行此操作，因为这样做不会影响已打开的连接。下面的示例来自辅助角色中的 **Application\_Start** 方法。

    var storageAccount = CloudStorageAccount.Parse(connStr);
    ServicePoint tableServicePoint = ServicePointManager.FindServicePoint(storageAccount.TableEndpoint);
    tableServicePoint.UseNagleAlgorithm = false;
    ServicePoint queueServicePoint = ServicePointManager.FindServicePoint(storageAccount.QueueEndpoint);
    queueServicePoint.UseNagleAlgorithm = false;

您应查看客户端日志以了解客户端应用程序正在提交多少个请求，并检查客户端中一般与 .NET 相关的性能瓶颈（如 CPU、.NET 垃圾回收、网络利用率或内存（作为排查 .NET 客户端应用程序问题的起点，请参阅 MSDN 上的<a href="http://msdn.microsoft.com/zh-cn/library/7fe0dd2y(v=vs.110).aspx" target="_blank">调试、跟踪和分析</a>）。

#### 调查网络延迟问题

通常，因网络导致的高端到端延迟是由暂时状况导致的。你可以使用工具（如 Wireshark 或 Microsoft Message Analyzer）调查临时和持久网络问题，例如丢弃数据包。

有关使用 Wireshark 排查网络问题的详细信息，请参阅“[附录 2：使用 Wireshark 捕获网络流量]”。

有关使用 Microsoft Message Analyzer 排查网络问题的详细信息，请参阅“[附录 3：使用 Microsoft Message Analyzer 捕获网络流量]”。

### <a name="metrics-show-low-AverageE2ELatency-and-low-AverageServerLatency"></a>度量值显示低 AverageE2ELatency 和低 AverageServerLatency，但客户端遇到高延迟

在这种情况下，最可能的原因是到达存储服务的存储请求出现延迟。你应调查来自客户端的请求为什么未到达 Blob 服务。

客户端延迟发送请求的可能原因包括：可用连接数或可用线程数有限。你还应检查客户端是否正在执行多次重试，如果是这种情况，请调查原因。您可以通过查找与请求关联的 **OperationContext** 对象并检索 **ServerRequestId** 值来以编程方式执行此操作。有关详细信息，请参阅“[服务器请求 ID]”一节中的代码示例。

如果客户端没有问题，则应调查潜在的网络问题，例如数据包丢失。可以使用工具（如 Wireshark 或 Microsoft Message Analyzer）调查网络问题。

有关使用 Wireshark 排查网络问题的详细信息，请参阅“[附录 2：使用 Wireshark 捕获网络流量]”。

有关使用 Microsoft Message Analyzer 排查网络问题的详细信息，请参阅“[附录 3：使用 Microsoft Message Analyzer 捕获网络流量]”。

### <a name="metrics-show-high-AverageServerLatency"></a>度量值显示高 AverageServerLatency

如果 Blob 下载请求出现高 **AverageServerLatency**，您应使用存储日志记录日志来了解对于同一 Blob（或一组 Blob）是否存在重复的请求。对于 Blob 上载请求，应调查客户端正在使用的数据块大小（例如，小于 64k 的数据块大小可能会导致开销，除非读取操作也在小于 64k 的区块中进行），以及是否有多个客户端正在并行将数据块上载到同一 Blob。您还应检查每分钟度量值以了解导致超出每秒可伸缩性目标的请求数峰值：另请参阅“[度量值显示 PercentTimeoutError 增加]”。

如果当对于同一 Blob（或一组 Blob）存在重复的请求时，您看到 Blob 下载请求显示高 **AverageServerLatency**，则应考虑使用 Azure 缓存或 Azure 内容交付网络 (CDN) 缓存这些 Blob。对于上载请求，你可以通过使用较大的数据块大小来提高吞吐量。对于表查询，也可以在执行相同的查询操作并且数据不会频繁更改的客户端上实施客户端缓存。

较高的 **AverageServerLatency** 值也可能是设计欠佳的表或查询的症状，它会导致扫描操作或执行追加/前面预置反模式。有关详细信息，请参阅“[度量值显示 PercentThrottlingError 增加]”。

> [AZURE.NOTE] 你可以在此处找到一份全面的性能清单：[Azure 存储空间性能和可伸缩性清单](/documentation/articles/storage-performance-checklist/)。

### <a name="you-are-experiencing-unexpected-delays-in-message-delivery"></a>队列上的消息传递出现意外的延迟

如果在应用程序将某一消息添加到队列的时间与可从队列中读取该消息的时间之间出现延迟，你应采取以下步骤来诊断此问题：

- 验证应用程序是否成功地将该消息添加到队列。检查应用程序在成功添加前是否未多次重试 **AddMessage** 方法。存储客户端库日志将显示存储操作的任何重复重试。
- 验证将消息添加到队列的辅助角色与从队列读取该消息的辅助角色之间不存在任何时钟偏差，使得处理看起来就像出现延迟。
- 检查从队列中读取该消息的辅助角色是否出现故障。如果队列客户端调用了 **GetMessage** 方法，但无法使用确认进行响应，则该消息将一直在队列中保持不可见，直到 **invisibilityTimeout** 期限过期。此时，该消息可供再次处理。
- 检查队列长度是否随着时间的推移不断增长。如果没有足够多的工作线程可用于处理其他工作线程放入队列的所有消息，会出现这种情况。此外，还应检查度量值以了解删除请求是否失败，并应查看消息的出队计数，该计数可能指示删除消息的重复失败尝试次数。
- 检查存储日志记录日志以查找在长于平常的时间段内具有高于预期的 **E2ELatency** 和 **ServerLatency** 值的任何队列操作。


### <a name="metrics-show-an-increase-in-PercentThrottlingError"></a>度量值显示 PercentThrottlingError 增加

当超出存储服务的可伸缩性目标时，会发生限制错误。存储服务这样做是为了确保没有单个客户端或租户可以在损害其他客户端或租户的情况下使用服务。有关存储帐户的可伸缩性目标和存储帐户中的分区的性能目标的详细信息，请参阅 <a href="http://msdn.microsoft.com/zh-cn/library/azure/dn249410.aspx" target="_blank">Azure 存储空间可伸缩性和性能目标</a>。

如果 **PercentThrottlingError** 度量值显示失败并出现限制错误的请求百分比增加，则需要调查以下两种情况之一：

- [PercentThrottlingError 暂时增加]
- [PercentThrottlingError 错误永久增加]

通常在存储请求数增加时，或者在最初对应用程序进行负载测试时，会出现 **PercentThrottlingError** 增加的情况。这也可能表现为在客户端中进行存储操作时出现“503 服务器忙”或“500 操作超时”HTTP 状态消息。

#### <a name="transient-increase-in-PercentThrottlingError"></a>PercentThrottlingError 暂时增加

如果您看到 **PercentThrottlingError** 的值达到峰值的时间与应用程序活动的高峰期保持一致，则应在客户端中对重试实施指数（而非线性）退让策略：这将减少分区上的即时负载，并帮助您的应用程序消除流量峰值。有关如何实现使用存储客户端库实现重试策略的详细信息，请参阅 MSDN 上的 <a href="http://msdn.microsoft.com/zh-cn/library/azure/microsoft.windowsazure.storage.retrypolicies.aspx" target="_blank">Microsoft.WindowsAzure.Storage.RetryPolicies 命名空间</a>。

> [AZURE.NOTE]您可能也会看到 **PercentThrottlingError** 的值达到峰值的时间与应用程序活动的高峰期不一致：这种情况最可能的原因是存储服务正在移动分区以改进负载平衡。

#### <a name="permanent-increase-in-PercentThrottlingError"></a>PercentThrottlingError 错误永久增加

在事务量永久增加后，或者在对应用程序进行初始负载测试时，如果您看到 **PercentThrottlingError** 的值一直很高，则需要评估您的应用程序使用存储分区的方式，以及它是否接近存储帐户的可伸缩性目标。例如，如果你在一个队列（它视为单个分区）上看到限制错误，则应考虑使用其他队列以将事务分布到多个分区上。如果你在表上看到限制错误，则需要考虑使用不同的分区方案，以便使用更多种分区键值将你的事务分布到多个分区。此问题的一个常见原因是由于“前面预置/追加”反模式，在该模式下你选择日期作为分区键，然后将某一天的所有数据都写入到一个分区：负载过轻，这可能会导致写入瓶颈。你应考虑使用不同的分区设计，或者评估使用 Blob 存储是否可能是更好的解决方案。你还应该检查出现限制是否是由于流量达到峰值而导致的，并应调查平滑处理请求模式的方式。

如果你将事务分布到多个分区中，必须仍要注意为存储帐户设置的可伸缩性限制。例如，如果你使用 10 个队列（每个队列每秒最多处理 2000 条 1KB 消息），你将达到存储帐户每秒 20,000 条消息的总体限制。如果你每秒需要处理的实体数超过 20,000 个，则应考虑使用多个存储帐户。你还应牢记，请求和实体的大小对存储服务何时限制你的客户端有影响：如果你使用较大的请求和实体，则可能会更快地受到限制。

低效的查询设计也会导致你达到表分区的可伸缩性限制。例如，一个使用筛选器的查询仅选择分区中百分之一的实体，但却扫描该分区中的所有实体，从而将需要访问每个实体。读取的每个实体均将计入该分区中的事务总数；因此，很容易就可以达到可伸缩性目标。

> [AZURE.NOTE]性能测试应显示你的应用程序中的任何低效查询设计。

### <a name="metrics-show-an-increase-in-PercentTimeoutError"></a>度量值显示 PercentTimeoutError 增加

您的度量值显示其中一个存储服务的 **PercentTimeoutError** 增加。同时，客户端将收到存储操作发出的大量“500 操作超时”HTTP 状态消息。

> [AZURE.NOTE]当存储服务通过将分区移到新服务器来对请求进行负载平衡时，你可能会临时地看到超时错误。

**PercentTimeoutError** 度量值是以下度量值的聚合：**ClientTimeoutError**、**AnonymousClientTimeoutError**、**SASClientTimeoutError**、**ServerTimeoutError**、**AnonymousServerTimeoutError** 和 **SASServerTimeoutError**。

服务器超时是由于服务器上的错误导致的。客户端超时之所以发生是因为服务器上的操作已超出客户端指定的超时值；例如，使用存储客户端库的客户端可以使用 **QueueRequestOptions** 类的 **ServerTimeout** 属性为操作设置超时值。

服务器超时指示存储服务存在需要进一步调查的问题。你可以使用度量值来了解是否已达到该服务的可伸缩性限制，并确定可能会导致此问题的流量峰值。如果问题是间歇性的，则可能是由于服务中的负载平衡活动导致的。如果问题是持久性的并且不是由于应用程序达到服务的可伸缩性限制导致的，则你应发送支持问题。对于客户端超时，你必须确定在客户端中超时是否设为适当的值，可更改客户端中设置的超时值，或者调查如何改善存储服务中的操作性能，例如通过优化表查询或缩小消息的大小。

### <a name="metrics-show-an-increase-in-PercentNetworkError"></a>度量值显示 PercentNetworkError 增加

您的度量值显示其中一个存储服务的 **PercentNetworkError** 增加。**PercentNetworkError** 度量值是以下度量值的聚合：**NetworkError**、**AnonymousNetworkError** 和 **SASNetworkError**。如果存储服务在客户端发出存储请求时检测到网络错误，则会出现这些错误。

出现此错误的最常见原因是客户端在存储服务超时到期之前断开连接。你应调查客户端中的代码，以了解客户端断开与存储服务的连接的原因和时间。你可以还使用 Wireshark、Microsoft Message Analyzer 或 Tcping 调查客户端的网络连接问题。这些工具在[附录]中进行了说明。

### <a name="the-client-is-receiving-403-messages"></a>客户端正在接收“HTTP 403 (禁止访问)”消息

如果客户端应用程序引发“HTTP 403 (禁止访问)”错误，则可能的原因是客户端在发送存储请求时使用了过期的共享访问签名 (SAS)（虽然其他可能的原因包括时钟偏差、无效密钥和空标头）。如果已过期的 SAS 密钥是原因，则你将不会在服务器端存储日志记录日志数据中看到任何条目。下表显示了存储客户端库生成的客户端日志的示例，它说明了如何出现此问题：

源|详细程度|详细程度|客户端请求 ID|操作文本
---|---|---|---|---
Microsoft.WindowsAzure.Storage|信息|3|85d077ab-...|正在按位置模式 PrimaryOnly 使用主位置启动操作。
Microsoft.WindowsAzure.Storage|信息|3|85d077ab -…|开始发出同步请求到 https://domemaildist.blob.core.chinacloudapi.cnazureimblobcontainer/blobCreatedViaSAS.txt?sv=2014-02-14&amp;sr=c&amp;si=mypolicy&amp;sig=OFnd4Rd7z01fIvh%2BmcR6zbudIH2F5Ikm%2FyhNYZEmJNQ%3D&amp;api-version=2014-02-14。
Microsoft.WindowsAzure.Storage|信息|3|85d077ab -…|正在等待响应。
Microsoft.WindowsAzure.Storage|警告|2|85d077ab -…|在等待响应时引发的异常：远程服务器返回了错误：(403) 禁止访问...
Microsoft.WindowsAzure.Storage|信息|3|85d077ab -…|收到响应。状态代码 = 403，请求 ID = 9d67c64a-64ed-4b0d-9515-3b14bbcdc63d，Content-MD5 =，ETag = 。
Microsoft.WindowsAzure.Storage|警告|2|85d077ab -…|在操作期间引发的异常：远程服务器返回了错误：(403) 禁止访问...
Microsoft.WindowsAzure.Storage|信息|3 |85d077ab -…|正在检查是否应重试该操作。重试次数 = 0，HTTP 状态代码 = 403，异常 = 远程服务器返回了一个错误：(403) 禁止访问...
Microsoft.WindowsAzure.Storage|信息|3|85d077ab -…|已根据位置模式将下一个位置设为主位置。
Microsoft.WindowsAzure.Storage|错误|1|85d077ab -…|重试策略不允许重试。操作失败，远程服务器返回了一个错误：(403) 禁止访问。

在此方案中，你应调查在客户端将该令牌发送到服务器之前 SAS 令牌即将到期的原因：

- 通常，当你创建 SAS 供客户端立即使用时，不应设置开始时间。如果使用当前时间生成 SAS 的主机与存储服务之间存在较小的时钟差异，则存储服务有可能收到尚未生效的 SAS。
- 不应在 SAS 上设置太短的到期时间。同样，生成 SAS 的主机与存储服务之间的较小时钟差异可能会导致 SAS 似乎早于预期到期。
- SAS 密钥中的版本参数（例如 **sv=2012-02-12**）是否与您正在使用的存储客户端库的版本匹配。应始终使用最新版的存储客户端库。有关 SAS 令牌版本控制的详细信息，请参阅 [Azure 存储空间的新增功能](http://blogs.msdn.com/b/windowsazurestorage/archive/2014/05/14/what-s-new-for-microsoft-azure-storage-at-teched-2014.aspx)。
- 
- 如果你重新生成存储访问密钥（在 Azure 经典门户中存储帐户的任意页上单击“管理访问密钥”），这可能会使任何现有的 SAS 令牌无效。如果你生成的 SAS 令牌具有较长的到期时间供客户端应用程序缓存，这可能是个问题。

如果你使用存储客户端库生成 SAS 令牌，则可轻松生成有效令牌。但是，如果您使用的是存储 REST API 并手动构造 SAS 令牌，则应仔细阅读 MSDN 上的主题<a href="http://msdn.microsoft.com/zh-cn/library/azure/ee395415.aspx" target="_blank">使用共享访问签名委托访问</a>。

### <a name="the-client-is-receiving-404-messages"></a>客户端正在接收“HTTP 404 (未找到)”消息
如果客户端应用程序从服务器收到“HTTP 404 (未找到)”消息，这意味着客户端正在尝试使用的对象（如实体、表、Blob、容器或队列）在存储服务中不存在。有多种原因可能会导致此问题，例如：

- [客户端或其他进程以前删除了该对象]
- [共享访问签名 (SAS) 授权问题]
- [客户端 JavaScript 代码无权访问该对象]
- [网络故障]

#### <a name="client-previously-deleted-the-object"></a>客户端或其他进程以前删除了该对象
在客户端正在尝试读取、更新或删除存储服务中的数据的情况下，通常很容易在服务器端日志中找到以前从存储服务中删除有问题对象的操作。通常，日志数据显示其他用户或进程删除了该对象。当客户端删除了该对象时，在服务器端存储日志记录日志中，“操作类型”和“请求的对象键”列会显示相关信息。

在客户端正在尝试插入对象的情况下，可能无法立即找到导致“HTTP 404 (未找到)”响应的明显原因，因为客户端正在创建新对象。但是，如果客户端正在创建 Blob，则必须能够找到 Blob 容器；如果客户端正在创建消息，则必须能够找到队列；如果客户端正在添加行，则必须能够找到表。

可以使用存储客户端库生成的客户端日志来更详细地了解客户端将特定请求发送到存储服务时的信息。

存储客户端库生成的以下客户端日志说明了客户端找不到它创建的 Blob 的容器时的问题。此日志包含以下存储操作的详细信息：

请求 ID|操作
---|---
07b26a5d-...|**DeleteIfExists** 方法，用于删除 Blob 容器。请注意，此操作包括 **HEAD** 请求以检查该容器是否存在。
e2d06d78-...|**CreateIfNotExists** 方法，用于创建 Blob 容器。请注意，此操作包括 **HEAD** 请求，用于检查该容器是否存在。**HEAD** 返回了 404 消息，但将继续执行。
de8b1c3c-...|**UploadFromStream** 方法，用于创建 Blob。**PUT** 请求失败，显示 404 消息

日志项：

请求 ID | 操作文本
---|---
07b26a5d-...|开始发出同步请求到 https://domemaildist.blob.core.chinacloudapi.cn/azuremmblobcontainer。
07b26a5d-...|StringToSign = HEAD............x-ms-client-request-id:07b26a5d-....x-ms-date:Tue, 03 Jun 2014 10:33:11 GMT.x-ms-version:2014-02-14./domemaildist/azuremmblobcontainer.restype:container.
07b26a5d-...|正在等待响应。
07b26a5d-... | 收到响应。状态代码 = 200，请求 ID = eeead849-...Content-MD5 = ，ETag =";0x8D14D2DC63D059B";。
07b26a5d-... | 响应标头已成功处理，继续执行该操作的剩余部分。
07b26a5d-... | 正在下载响应正文。
07b26a5d-... | 操作已成功完成。
07b26a5d-... | 开始发出同步请求到 https://domemaildist.blob.core.chinacloudapi.cn/azuremmblobcontainer。
07b26a5d-... | StringToSign = DELETE............x-ms-client-request-id:07b26a5d-....x-ms-date:Tue, 03 Jun 2014 10:33:12 GMT.x-ms-version:2014-02-14./domemaildist/azuremmblobcontainer.restype:container.
07b26a5d-... | 正在等待响应。
07b26a5d-... | 收到响应。状态代码 = 202，请求 ID = 6ab2a4cf-...，Content-MD5 = ，ETag = 。
07b26a5d-... | 响应标头已成功处理，继续执行该操作的剩余部分。
07b26a5d-... | 正在下载响应正文。
07b26a5d-... | 操作已成功完成。
e2d06d78-... | 开始发出异步请求到 https://domemaildist.blob.core.chinacloudapi.cn/azuremmblobcontainer.</td>
e2d06d78-... | StringToSign = HEAD............x-ms-client-request-id:e2d06d78-....x-ms-date:Tue, 03 Jun 2014 10:33:12 GMT.x-ms-version:2014-02-14./domemaildist/azuremmblobcontainer.restype:container.
e2d06d78-...| 正在等待响应。
de8b1c3c-... | 开始发出同步请求到 https://domemaildist.blob.core.chinacloudapi.cn/azuremmblobcontainer/blobCreated.txt。
de8b1c3c-... | StringToSign = PUT...64.qCmF+TQLPhq/YYK50mP9ZQ==........x-ms-blob-type:BlockBlob.x-ms-client-request-id:de8b1c3c-....x-ms-date:Tue, 03 Jun 2014 10:33:12 GMT.x-ms-version:2014-02-14./domemaildist/azuremmblobcontainer/blobCreated.txt.
de8b1c3c-... | 正在准备写入请求数据。
e2d06d78-... | 在等待响应时引发的异常：远程服务器返回了错误：(404) 未找到...
e2d06d78-... | 收到响应。状态代码 = 404，请求 ID = 353ae3bc-...，Content-MD5 = ，ETag = 。
e2d06d78-... | 响应标头已成功处理，继续执行该操作的剩余部分。
e2d06d78-... | 正在下载响应正文。
e2d06d78-... | 操作已成功完成。
e2d06d78-... | 开始发出异步请求到 https://domemaildist.blob.core.chinacloudapi.cn/azuremmblobcontainer。
e2d06d78-...|StringToSign = PUT...0.........x-ms-client-request-id:e2d06d78-....x-ms-date:Tue, 03 Jun 2014 10:33:12 GMT.x-ms-version:2014-02-14./domemaildist/azuremmblobcontainer.restype:container.
e2d06d78-... | 正在等待响应。
de8b1c3c-... | 正在写入请求数据。
de8b1c3c-... | 正在等待响应。
e2d06d78-... | 在等待响应时引发的异常：远程服务器返回了错误：(409) 冲突...
e2d06d78-... | 收到响应。状态代码 = 409，请求 ID = c27da20e-...，Content-MD5 = ，ETag = 。
e2d06d78-... | 正在下载错误响应正文。
de8b1c3c-... | 在等待响应时引发的异常：远程服务器返回了错误：(404) 未找到...
de8b1c3c-... | 收到响应。状态代码 = 404，请求 ID = 0eaeab3e-...，Content-MD5 = ，ETag = 。
de8b1c3c-...| 在操作期间引发的异常: 远程服务器返回了错误： (404) 未找到...
de8b1c3c-... | 重试策略不允许重试。操作失败，远程服务器返回了一个错误：(404) 未找到...
e2d06d78-... | 重试策略不允许重试。操作失败，远程服务器返回了一个错误 ：(409) 冲突...

在此示例中，该日志显示客户端正在交错执行 **CreateIfNotExists** 方法发出的请求（请求 ID e2d06d78…）与 **UploadFromStream** 方法发出的请求 (de8b1c3c-...)；之所以发生这种情况是因为客户端应用程序以异步方式调用了这两个方法。你应修改客户端中的异步代码，以确保客户端在尝试将任何数据上载到该容器中的 Blob 之前已创建该容器。理想情况下，你应该提前创建所有容器。

#### <a name="SAS-authorization-issue"></a>共享访问签名 (SAS) 授权问题

如果客户端应用程序尝试使用不包括必要的操作权限的 SAS 密钥，则存储服务向客户端返回“HTTP 404 (未找到)”消息。同时，您还会在度量值中看到 **SASAuthorizationError** 为非零值。

下表显示了存储日志记录日志文件中的示例服务器端日志消息：

<table>
  <tr>
    <td>请求开始时间</td>
    <td>2014-05-30T06:17:48.4473697Z</td>
  </tr>
  <tr>
    <td>操作类型</td>
    <td>GetBlobProperties</td>
  </tr>
  <tr>
    <td>请求状态</td>
    <td>SASAuthorizationError</td>
  </tr>
  <tr>
    <td>HTTP 状态代码</td>
    <td>404</td>
  </tr>
  <tr>
    <td>身份验证类型</td>
    <td>Sas</td>
  </tr>
  <tr>
    <td>服务类型</td>
    <td>Blob</td>
  </tr>
  <tr>
    <td>请求 URL</td>
    <td>
    https://domemaildist.blob.core.chinacloudapi.cn/azureimblobcontainer/blobCreatedViaSAS.txt?sv=2014-02-14&amp;amp;sr=c&amp;amp;si=mypolicy&amp;amp;sig=XXXXX&amp;amp;api-version=2014-02-14&amp;amp;</td>
  </tr>
  <tr>
    <td>请求 ID 标头</td>
    <td>a1f348d5-8032-4912-93ef-b393e5252a3b</td>
  </tr>
  <tr>
    <td>客户端请求 ID</td>
    <td>2d064953-8436-4ee0-aa0c-65cb874f7929</td>
  </tr>
</table>

你应调查客户端应用程序尝试执行尚未向其授予权限的操作的原因。

#### <a name="JavaScript-code-does-not-have-permission"></a>客户端 JavaScript 代码无权访问该对象

如果你使用的是 JavaScript 客户端并且存储服务返回 HTTP 404 消息，则应在浏览器中检查以下 JavaScript 错误：

    SEC7120: Origin http://localhost:56309 not found in Access-Control-Allow-Origin header.
    SCRIPT7002: XMLHttpRequest: Network Error 0x80070005, Access is denied.

> [AZURE.NOTE]在排查客户端 JavaScript 问题时，可以使用 Internet Explorer 中的 F12 开发人员工具来跟踪浏览器与存储服务之间交换的消息。

之所以发生这些错误是因为 Web 浏览器实施了“<a href="http://www.w3.org/Security/wiki/Same_Origin_Policy" target="_blank">同源策略</a>”安全限制，以防止网页调用与它来自的域不同的域中的 API。

若要解决此 JavaScript 问题，可以为客户端访问的存储服务配置跨域资源共享 (CORS)。有关详细信息，请参阅 MSDN 上的 <a href="http://msdn.microsoft.com/zh-cn/library/azure/dn535601.aspx" target="_blank">Azure 存储空间服务的跨域资源共享 (CORS) 支持</a>。

下面的代码示例演示如何配置 Blob 服务，以允许在 Contoso 域中运行的 JavaScript 访问 Blob 存储服务中的 Blob：

    CloudBlobClient client = new CloudBlobClient(blobEndpoint, new StorageCredentials(accountName, accountKey));
    // Set the service properties.
    ServiceProperties sp = client.GetServiceProperties();
    sp.DefaultServiceVersion = "2013-08-15";
    CorsRule cr = new CorsRule();
    cr.AllowedHeaders.Add("*");
    cr.AllowedMethods = CorsHttpMethods.Get | CorsHttpMethods.Put;
    cr.AllowedOrigins.Add("http://www.contoso.com");
    cr.ExposedHeaders.Add("x-ms-*");
    cr.MaxAgeInSeconds = 5;
    sp.Cors.CorsRules.Clear();
    sp.Cors.CorsRules.Add(cr);
    client.SetServiceProperties(sp);

#### <a name="network-failure"></a>网络故障

在某些情况下，丢失的网络数据包可能会导致存储服务向客户端返回 HTTP 404 消息。例如，当客户端应用程序从表服务中删除某个实体时，你看到客户端从表服务引发报告“HTTP 404 (未找到)”状态消息的存储异常。调查表存储服务中的表时，你看到该服务确实删除了请求的实体。

客户端中的异常详细信息包括表服务为请求分配的请求 ID (7e84f12d...)：您可以通过在日志文件的 **request-id-header** 列中搜索，使用此信息在服务器端存储日志中查找请求详细信息。此外，还可以使用度量值来确定此类失败何时发生，然后基于度量值记录此错误的时间搜索日志文件。此日志项显示删除失败并返回“HTTP (404) 客户端其他错误”状态消息。同一日志项还在 **client-request-id** 列中包括客户端生成的请求 ID (813ea74f…)。

服务器端日志文件还包含另一个具有同一 **client-request-id** 值 (813ea74f…) 的条目，该条目针对从同一客户端对同一实体进行的成功删除操作。此成功的删除操作在失败的删除请求之前很短的时间内发生。

此情况最有可能的原因是客户端将针对实体的删除请求发送到表服务，该请求成功，但未从服务器收到确认消息（可能是因为临时网络问题）。然后，客户端自动重试该操作（使用同一 **client-request-id**），但此重试失败，因为该实体已删除。

如果此问题频繁出现，你应该调查为什么客户端无法从表服务收到确认消息。如果此问题是间歇性的，则应捕获“HTTP (404) 未找到”错误并在客户端中记录它，但允许客户端继续执行。

### <a name="the-client-is-receiving-409-messages"></a>客户端正在接收“HTTP 409 (冲突)”消息

下表显示了服务器端日志中针对两个客户端操作的摘录：**DeleteIfExists** 后跟使用相同的 blob 容器名称的 **CreateIfNotExists**。请注意，每个客户端操作会导致将两个请求发送到服务器，先是 **GetContainerProperties** 请求（用于检查容器是否存在）,后跟 **DeleteContainer** 或 **CreateContainer** 请求。

Timestamp|操作|结果|容器名称|客户端请求 ID
---|---|---|---|---
05:10:13.7167225|GetContainerProperties|200|mmcont|c9f52c89-...
05:10:13.8167325|DeleteContainer|202|mmcont|c9f52c89-...
05:10:13.8987407|GetContainerProperties|404|mmcont|bc881924-...
05:10:14.2147723|CreateContainer|409|mmcont|bc881924-...

客户端应用程序中的代码先删除 Blob 容器，然后立即使用同一名称重新创建该容器：**CreateIfNotExists** 方法（客户端请求 ID bc881924-...）最终失败，显示“HTTP 409 (冲突)”错误。当客户端删除 Blob 容器、表或队列时，需要经过一段较短的时间，该名称才能再次变为可用。

客户端应用程序在创建新容器时应使用唯一的容器名称（如果“删除/重新创建”模式很常见）。

### <a name="metrics-show-low-percent-success"></a>度量值显示低 PercentSuccess，或者分析日志项包含事务状态为 ClientOtherErrors 的操作

**PercentSuccess** 度量值根据操作的 HTTP 状态代码捕获已成功的操作的百分比。状态代码为 2XX 的操作将计为成功，而状态代码在 3XX、4XX 和 5XX 范围内的操作将计为失败并降低 **PercentSucess** 度量值。在服务器端存储日志文件中，这些操作将使用事务状态 **ClientOtherErrors** 进行记录。

请务必注意，这些操作已成功完成，因此不会影响其他度量值，如可用性。成功执行但可能会导致失败的 HTTP 状态代码的一些操作示例包括：
- **ResourceNotFound**（未找到 404），例如，对不存在的 Blob 进行 GET 请求时生成。
- **ResouceAlreadyExists**（冲突 409），例如，在资源已存在的情况下进行 **CreateIfNotExist** 操作时生成。
- **ConditionNotMet**（未修改 304），例如，进行条件操作时生成，例如仅在自上次操作以来映像已更新时，客户端才会发送 **ETag** 值和一个 HTTP **If-None-Match** 标头来请求此映像。

可以在<a href="http://msdn.microsoft.com/zh-cn/library/azure/dd179357.aspx" target="_blank">常见的 REST API 错误代码</a>页上找到存储服务返回的常见 REST API 错误代码的列表。

### <a name="capacity-metrics-show-an-unexpected-increase"></a>容量度量值显示存储容量使用量意外增加


如果你在存储帐户中看到容量使用量突然地意外更改，则可以调查原因，具体方法是先查看可用性度量值；例如，失败的删除请求数增加可能会导致你所使用的 Blob 存储量增加，你本来希望应用程序特定的清理操作可以释放一些空间，但却未按预期正常工作（例如，因为用于释放空间的 SAS 令牌已过期）。

### <a name="you-are-experiencing-unexpected-reboots"></a>遇到具有大量附加 VHD 的 Azure 虚拟机意外重新启动

如果 Azure 虚拟机 (VM) 具有大量位于同一存储帐户的附加 VHD，则可能会超过单个存储帐户的可伸缩性目标，从而导致 VM 出现故障。您应查看存储帐户的每分钟度量值 (**TotalRequests**/**TotalIngress**/**TotalEgress**)，以获取超过一个存储帐户的可伸缩性目标的峰值。若要帮助确定是否已对您的存储帐户进行限制，请参阅“[度量值显示 PercentThrottlingError 增加]”一节。

通常，虚拟机对 VHD 进行的每个单独的输入或输出操作都会转换为对基础页 Blob 进行的“Get 页”或“Put 页”操作。因此，你可以根据应用程序的特定行为，对环境使用估计的 IOPS 以优化可以在单个存储帐户中设置的 VHD 数。我们不建议在单个存储帐户中设置超过 40 个的磁盘。有关存储帐户的当前可伸缩性目标的详细信息（尤其是您所用的存储帐户类型的总请求速率和总带宽），请参阅 <a href="http://msdn.microsoft.com/zh-cn/library/azure/dn249410.aspx" target="_blank">Azure 存储空间可伸缩性和性能目标</a>。如果你即将超过存储帐户的可伸缩性目标，则应将你的 VHD 放入多个不同的存储帐户中，以减少每个帐户中的活动。

### <a name="your-issue-arises-from-using-the-storage-emulator"></a>您的问题是由于使用存储模拟器进行开发或测试而导致

通常，在开发和测试过程中使用存储模拟器以避免需要 Azure 存储帐户。使用存储模拟器时可能发生的常见问题包括：

- [功能“X”在存储模拟器中无法正常工作]
- [使用存储模拟器时出现错误“其中一个 HTTP 标头的值的格式不正确”]
- [运行存储模拟器需要管理权限]

#### <a name="feature-X-is-not-working"></a>功能“X”在存储模拟器中无法正常工作

存储模拟器并非支持 Azure 存储服务的所有功能，如文件服务。有关详细信息，请参阅 MSDN 上的<a href="http://msdn.microsoft.com/zh-cn/library/azure/gg433135.aspx" target="_blank">存储模拟器与 Azure 存储服务之间的差异</a>。

对于存储模拟器不支持的这些功能，请使用云中的 Azure 存储服务。

#### <a name="error-HTTP-header-not-correct-format"></a>使用存储模拟器时出现错误“其中一个 HTTP 标头的值的格式不正确”

您正在针对本地存储模拟器测试使用存储客户端库的应用程序，方法调用（如 **CreateIfNotExists**）失败并显示错误消息“其中一个 HTTP 标头的值的格式不正确”。 这指示你所用的存储模拟器版本不支持你所用的存储客户端库版本。存储客户端库会为它发出的所有请求添加标头**x-ms-version**。如果存储模拟器无法识别 **x-ms-version** 标头中的值，则将拒绝该请求。

可以使用存储库客户端日志来查看它发送的 **x-ms-version header** 的值。如果您使用 Fiddler 跟踪客户端应用程序发出的请求，也可以查看 **x-ms-version header** 的值。

如果你安装并使用了存储客户端库的最新版本，但未更新存储模拟器，通常会出现这种情况。你应安装存储模拟器的最新版本，或者使用云存储而不是模拟器进行开发和测试。

#### <a name="storage-emulator-requires-administrative-privileges"></a>运行存储模拟器需要管理权限

当你运行存储模拟器时，系统提示你提供管理员凭据。仅当你首次初始化存储模拟器时，才会出现这种情况。初始化存储模拟器后，将无需管理权限即可再次运行。

有关详细信息，请参阅 MSDN 上的<a href="http://msdn.microsoft.com/zh-cn/library/azure/gg433132.aspx" target="_blank">使用命令行工具初始化存储模拟器</a>（也可以在 Visual Studio 中初始化存储模拟器，但这需要管理权限）。

### <a name="you-are-encountering-problems-installing-the-Windows-Azure-SDK"></a>安装 Azure SDK for .NET 时遇到问题

当你尝试安装 SDK 时，它尝试在本地计算机上安装存储模拟器时失败。安装日志包含以下消息之一：

- CAQuietExec：错误：无法访问 SQL 实例
- CAQuietExec：错误：无法创建数据库

原因是现有 LocalDB 安装有问题。默认情况下，存储模拟器在模拟 Azure 存储服务时使用 LocalDB 持久保存数据。你可以在尝试安装 SDK 之前，通过在命令提示符窗口中运行以下命令，来重置 LocalDB 实例。

    sqllocaldb stop v11.0
    sqllocaldb delete v11.0
    delete %USERPROFILE%\WAStorageEmulatorDb3*.*
    sqllocaldb create v11.0

**delete** 命令可从以前安装的存储模拟器中删除任何旧的数据库文件。

### <a name="you-have-a-different-issue-with-a-storage-service"></a>您遇到了其他存储服务问题

如果前面的故障排除章节未包括你遇到的存储服务问题，则应采用以下方法来诊断和排查你的问题。

- 检查你的度量值，以了解与预期的基准行为相比是否存在任何更改。从度量值，你可能能够确定此问题是暂时的还是永久性的，并可确定此问题影响哪些存储操作。
- 可以使用度量值信息来帮助你搜索服务器端日志数据，以获取有关发生的任何错误的更多详细信息。此信息可能会帮助你排查和解决该问题。
- 如果服务器端日志中的信息不足以成功排查此问题，则可以使用存储客户端库客户端日志来调查客户端应用程序和工具（如 Fiddler、Wireshark 和 Microsoft Message Analyzer）的行为以调查你的网络。

有关使用 Fiddler 的详细信息，请参阅“[附录 1：使用 Fiddler 捕获 HTTP 和 HTTPS 通信]”。

有关 Wireshark 的详细信息，请参阅“[附录 2：使用 Wireshark 捕获网络流量]”。

有关使用 Microsoft Message Analyzer 的详细信息，请参阅“[附录 3：使用 Microsoft Message Analyzer 捕获网络流量]”。

## <a name="appendices"></a>附录

附录介绍几种在诊断和排查 Azure 存储空间（及其他服务）问题时你可能会发现很有用的工具。这些工具不属于 Azure 存储空间，有些工具是第三方产品。因此，这些附录中介绍的工具可能在你与 Azure 或 Azure 存储空间签订的任何支持协议中均未涉及，因此，你在评估过程中，应查看这些工具的提供者提供的许可和支持选项。

### <a name="appendix-1"></a>附录 1：使用 Fiddler 捕获 HTTP 和 HTTPS 通信

Fiddler 是一个有用的工具，用于分析客户端应用程序与你所用的 Azure 存储服务之间的 HTTP 和 HTTPS 通信。您可以从 <a href="http://www.telerik.com/fiddler" target="_blank">http://www.telerik.com/fiddler</a> 下载 Fiddler。

> [AZURE.NOTE]Fiddler 可以解码 HTTPS 通信；你应仔细阅读 Fiddler 文档以了解它如何执行此操作，并了解安全隐患。

本附录提供了一个简要演练，介绍如何配置 Fiddler 以捕获已安装 Fiddler 的本地计算机与 Azure 存储服务之间的通信。

启动 Fiddler 后，它将开始捕获你的本地计算机上的 HTTP 和 HTTPS 通信。以下是一些用于控制 Fiddler 的有用命令：

- 停止和启动捕获流量。在主菜单上，转到“文件”，然后单击“捕获流量”可在打开和关闭捕获之间切换。
- 保存捕获的通信数据。在主菜单上，转到“文件”，单击“保存”，然后单击“所有会话”：这使您可以将流量保存在一个会话存档文件中。你以后可以重新加载会话存档以进行分析，或者将其发送到 Microsoft 技术支持（如果被要求）。

若要限制 Fiddler 捕获的通信量，可以使用在“筛选器”选项卡中配置的筛选器。下面的屏幕截图显示了只捕获发送到 **contosoemaildist.table.core.chinacloudapi.cn** 存储终结点的流量的筛选器：

![][5]

### <a name="appendix-2"></a>附录 2：使用 Wireshark 捕获网络流量

Wireshark 是一种网络协议分析器，可用于查看各种网络协议的详细数据包信息。您可以从 <a href="http://www.wireshark.org/" target="_blank">http://www.wireshark.org/</a> 下载 Wireshark。

以下过程演示，对于从安装 Wireshark 的本地计算机到 Azure 存储帐户中的表服务的流量，如何捕获其详细数据包信息。

1.	在本地计算机上启动 Wireshark。
2.	在“启动”部分中，选择本地网络接口或连接到 Internet 的接口。
3.	单击“捕获选项”。
4.	将一个筛选器添加到“捕获筛选器”文本框中。例如，**host contosoemaildist.table.core.chinacloudapi.cn** 会将 Wireshark 配置为只捕获发送到 **contosoemaildist** 存储帐户中的表服务终结点或从该终结点发送的数据包。捕获的筛选器的完整列表请参阅 <a href="http://wiki.wireshark.org/CaptureFilters" target="_blank">http://wiki.wireshark.org/CaptureFilters</a>。

    ![][6]

5.	单击“启动”。现在，当你在本地计算机上使用客户端应用程序时，Wireshark 将捕获发送到表服务终结点或从该终结点发送的所有数据包。
6.	完成后，在主菜单上，依次单击“捕获”、“停止”。
7.	若要将捕获的数据保存到 Wireshark 捕获文件中，请在主菜单上依次单击“文件”、“保存”。

WireShark 将在 **packetlist** 窗口中突出显示存在的任何错误。您还可以使用“专家信息”窗口（依次单击“分析”、“专家信息”）来查看错误和警告的摘要。

![][7]

您还可以选择查看 TCP 数据（如果应用程序层看到该数据），方法是右键单击 TCP 数据，然后选择“跟踪 TCP 流”。当你不使用捕获筛选器捕获转储时，此方法特别有用。有关详细信息，请参阅“此处”。<a href="http://www.wireshark.org/docs/wsug_html_chunked/ChAdvFollowTCPSection.html" target="_blank"></a>

![][8]

> [AZURE.NOTE]有关使用 Wireshark 的详细信息，请参阅 <a href="http://www.wireshark.org/docs/wsug_html_chunked/" target="_blank">Wireshark 用户指南</a>。

### <a name="appendix-3"></a>附录 3：使用 Microsoft Message Analyzer 捕获网络流量

可以使用 Microsoft Message Analyzer 以与 Fiddler 类似的方式捕获 HTTP 和 HTTPS 通信，并以与 Wireshark 类似的方式捕获网络流量。

#### 使用 Microsoft Message Analyzer 配置 Web 跟踪会话

若要使用 Microsoft Message Analyzer 为 HTTP 和 HTTPS 通信配置 Web 跟踪会话，请运行 Microsoft Message Analyzer 应用程序的，然后在“文件”菜单上单击“捕获/跟踪”。在可用的跟踪方案列表中，选择“Web 代理”。然后在“跟踪方案配置”面板的 **HostnameFilter** 文本框中，添加存储终结点的名称（你可以在 Azure 经典门户中查找这些名称）。例如，如果您的 Azure 存储帐户的名称是 **contosodata**，则应将以下内容添加到 **HostnameFilter** 文本框：

    contosodata.blob.core.chinacloudapi.cn contosodata.table.core.chinacloudapi.cn contosodata.queue.core.chinacloudapi.cn

> [AZURE.NOTE]空格字符分隔主机名。

当您准备好开始收集跟踪数据时，请单击“就此开始”按钮。

有关 Microsoft Message Analyzer “Web 代理”跟踪的详细信息，请参阅 TechNet 上的 <a href="http://technet.microsoft.com/zh-cn/library/jj674814.aspx" target="_blank">PEF-WebProxy 提供程序</a>。

Microsoft Message Analyzer 中内置的“Web 代理”跟踪基于 Fiddler；它可以捕获客户端 HTTPS 通信，并显示未加密的 HTTPS 消息。“Web 代理”跟踪的工作原理是通过为所有 HTTP 和 HTTPS 通信配置本地代理使其可以访问未加密的消息。

#### 使用 Microsoft Message Analyzer 诊断网络问题

除了使用 Microsoft Message Analyzer “Web 代理”跟踪来捕获客户端应用程序和存储服务之间的 HTTP/HTTPS 通信的详细信息外，您还可以使用内置的“本地链路层”跟踪来捕获网络数据包信息。这使你可以捕获类似于使用 Wireshark，可以捕获的数据，并诊断丢弃的数据包等网络问题。

下面的屏幕截图显示了“本地链路层”跟踪的一个示例，其中一些“信息性”消息显示在 **DiagnosisTypes** 列中。单击 **DiagnosisTypes** 列中的图标可显示消息的详细信息。在此示例中，服务器重新传输了消息 #305，因为它未收到来自客户端的确认消息：

![][9]

当你在 Microsoft Message Analyzer 中创建跟踪会话时，可以指定筛选器，以减少跟踪中的干扰项量。在定义跟踪的“捕获/跟踪”页上，单击 **Microsoft-Windows-NDIS-PacketCapture** 旁边的“配置”链接。下面的屏幕截图显示了筛选三个存储服务的 IP 地址的 TCP 通信的配置：

![][10]

有关 Microsoft Message Analyzer 本地链路层跟踪的详细信息，请参阅 TechNet 上的 <a href="http://technet.microsoft.com/zh-cn/library/jj659264.aspx" target="_blank">PEF-NDIS-PacketCapture 提供程序</a>。

### <a name="appendix-4"></a>附录 4：使用 Excel 查看度量值和日志数据

使用许多工具可以从 Azure 表存储中下载带分隔符格式的存储度量值数据，以便可以轻松地将这些数据加载到 Excel 中进行查看和分析。来自 Azure Blob 存储的存储日志记录数据已采用可以加载到 Excel 中的带分隔符格式。但是，您需要基于“<a href="http://msdn.microsoft.com/zh-cn/library/azure/hh343259.aspx" target="_blank">存储分析日志格式</a>”和“<a href="http://msdn.microsoft.com/zh-cn/library/azure/hh343264.aspx" target="_blank">存储分析度量表架构</a>”中的信息添加相应的列标题。

若要将存储日志记录数据导入 Excel（从 Blob 存储下载后），请执行以下操作：

- 在“数据”菜单上，单击“从文本”。
- 浏览到要查看的日志文件，然后单击“导入”。
- 在“文本导入向导”的第 1 步中，选择“带分隔符”。

在“文本导入向导”的第 1 步中，选择“分号”作为唯一的分隔符，然后选择双引号作为“文本限定符”。然后单击“完成”，并选择将数据放入工作簿中的位置。
<!--
### <a name="appendix-5"></a>附录 5：使用 Application Insights for Visual Studio Team Services 进行监视

在性能和可用性监视过程中，你还可以使用用于 Visual Studio Team Services 的 Application Insights 功能。此工具可以：

- 确保你的 Web 服务可用且响应迅速。无论您的应用程序是一个网站还是使用 Web 服务的设备应用程序，它都可以从世界各地的位置每隔几分钟测试一次您的 URL，并让您知道是否存在问题。
- 快速诊断你的 Web 服务中的任何性能问题或异常。查明 CPU 或其他资源是否过多使用，从异常获取堆栈跟踪并通过日志跟踪轻松地进行搜索。如果应用程序的性能降至低于可接受的限制，我们可以向你发送电子邮件。你可以同时监视 .NET 和 Java Web 服务。

在撰写本文时 Application Insights 处于预览状态。你可以在 <a href="http://msdn.microsoft.com/zh-cn/library/azure/dn481095.aspx" target="_blank">MSDN 上的 Application Insights for Visual Studio Team Services</a> 中找到详细信息。

-->
<!--Anchors-->
[介绍]: #introduction
[本指南的组织方式]: #how-this-guide-is-organized

[监视存储服务]: #monitoring-your-storage-service
[监视服务运行状况]: #monitoring-service-health
[监视容量]: #monitoring-capacity
[监视可用性]: #monitoring-availability
[监视性能]: #monitoring-performance

[诊断存储空间问题]: #diagnosing-storage-issues
[服务运行状况问题]: #service-health-issues
[性能问题]: #performance-issues
[诊断错误]: #diagnosing-errors
[存储模拟器问题]: #storage-emulator-issues
[存储日志记录工具]: #storage-logging-tools
[使用网络日志记录工具]: #using-network-logging-tools

[端到端跟踪]: #end-to-end-tracing
[关联日志数据]: #correlating-log-data
[客户端请求 ID]: #client-request-id
[服务器请求 ID]: #server-request-id
[时间戳]: #timestamps

[故障排除指南]: #troubleshooting-guidance
[度量值显示高 AverageE2ELatency 和低 AverageServerLatency]: #metrics-show-high-AverageE2ELatency-and-low-AverageServerLatency
[度量值显示低 AverageE2ELatency 和低 AverageServerLatency，但客户端遇到高延迟]: #metrics-show-low-AverageE2ELatency-and-low-AverageServerLatency
[度量值显示高 AverageServerLatency]: #metrics-show-high-AverageServerLatency
[队列上的消息传递出现意外的延迟]: #you-are-experiencing-unexpected-delays-in-message-delivery

[度量值显示 PercentThrottlingError 增加]: #metrics-show-an-increase-in-PercentThrottlingError
[PercentThrottlingError 暂时增加]: #transient-increase-in-PercentThrottlingError
[PercentThrottlingError 错误永久增加]: #permanent-increase-in-PercentThrottlingError
[度量值显示 PercentTimeoutError 增加]: #metrics-show-an-increase-in-PercentTimeoutError
[度量值显示 PercentNetworkError 增加]: #metrics-show-an-increase-in-PercentNetworkError

[客户端正在接收“HTTP 403 (禁止访问)”消息]: #the-client-is-receiving-403-messages
[客户端正在接收“HTTP 404 (未找到)”消息]: #the-client-is-receiving-404-messages
[客户端或其他进程以前删除了该对象]: #client-previously-deleted-the-object
[共享访问签名 (SAS) 授权问题]: #SAS-authorization-issue
[客户端 JavaScript 代码无权访问该对象]: #JavaScript-code-does-not-have-permission
[网络故障]: #network-failure
[客户端正在接收“HTTP 409 (冲突)”消息]: #the-client-is-receiving-409-messages

[度量值显示低 PercentSuccess，或者分析日志项包含事务状态为 ClientOtherErrors 的操作]: #metrics-show-low-percent-success
[容量度量值显示存储容量使用量意外增加]: #capacity-metrics-show-an-unexpected-increase
[遇到具有大量附加 VHD 的虚拟机意外重新启动]: #you-are-experiencing-unexpected-reboots
[你的问题是由于使用存储模拟器进行开发或测试而导致]: #your-issue-arises-from-using-the-storage-emulator
[功能“X”在存储模拟器中无法正常工作]: #feature-X-is-not-working
[使用存储模拟器时出现错误“其中一个 HTTP 标头的值的格式不正确”]: #error-HTTP-header-not-correct-format
[运行存储模拟器需要管理权限]: #storage-emulator-requires-administrative-privileges
[安装 Azure SDK for .NET 时遇到问题]: #you-are-encountering-problems-installing-the-Windows-Azure-SDK
[你遇到了其他存储服务问题]: #you-have-a-different-issue-with-a-storage-service

[附录]: #appendices
[附录 1：使用 Fiddler 捕获 HTTP 和 HTTPS 通信]: #appendix-1
[附录 2：使用 Wireshark 捕获网络流量]: #appendix-2
[附录 3：使用 Microsoft Message Analyzer 捕获网络流量]: #appendix-3
[附录 4：使用 Excel 查看度量值和日志数据]: #appendix-4


<!--Image references-->
[1]: ./media/storage-monitoring-diagnosing-troubleshooting-classic-portal/overview.png
[2]: ./media/storage-monitoring-diagnosing-troubleshooting-classic-portal/portal-screenshot.png
[3]: ./media/storage-monitoring-diagnosing-troubleshooting-classic-portal/hour-minute-metrics.png
[4]: ./media/storage-monitoring-diagnosing-troubleshooting-classic-portal/high-e2e-latency.png
[5]: ./media/storage-monitoring-diagnosing-troubleshooting-classic-portal/fiddler-screenshot.png
[6]: ./media/storage-monitoring-diagnosing-troubleshooting-classic-portal/wireshark-screenshot-1.png
[7]: ./media/storage-monitoring-diagnosing-troubleshooting-classic-portal/wireshark-screenshot-2.png
[8]: ./media/storage-monitoring-diagnosing-troubleshooting-classic-portal/wireshark-screenshot-3.png
[9]: ./media/storage-monitoring-diagnosing-troubleshooting-classic-portal/mma-screenshot-1.png
[10]: ./media/storage-monitoring-diagnosing-troubleshooting-classic-portal/mma-screenshot-2.png

<!---HONumber=Mooncake_0530_2016-->