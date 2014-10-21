<properties linkid="manage-services-how-to-monitor-websites" urlDisplayName="How to monitor" pageTitle="How to monitor web sites - Azure service management" metaKeywords="Azure monitoring web sites, Azure Management Portal Monitor, Azure monitoring" description="Learn how to monitor Azure web sites by using the Monitor page in the Management Portal." metaCanonical="" services="web-sites" documentationCenter="" title="How to Monitor Web Sites" authors="timamm" solutions="" manager="paulettm" editor="mollybos" />

# <a name="howtomonitor"></a>如何监视网站

网站通过监视管理页面提供监视功能。监视管理页面按如下所述提供网站的性能统计信息。

## 目录

-   [如何：添加网站度量值][如何：添加网站度量值]
-   [如何：接收来自网站度量值的警报][如何：接收来自网站度量值的警报]
-   [如何：查看网站的使用率配额][如何：查看网站的使用率配额]
-   [如何：减少资源使用率][如何：减少资源使用率]
-   [超出使用率配额时出现的情况][超出使用率配额时出现的情况]
-   [如何：为网站配置诊断和下载日志][如何：为网站配置诊断和下载日志]
-   [如何：监视 Web 终结点状态][如何：监视 Web 终结点状态]

## <a name="websitemetrics"></a>如何：添加网站度量值

1.  在 [Azure 管理门户][Azure 管理门户]中，从网站的管理页面上单击“监视器”选项卡，以显示“监视器”管理页面。默认情况下，“监视器”页面上的图表显示的度量值与“仪表板”页面上图表所显示的度量值是相同的。

2.  若要查看网站的其他度量值，请单击页面底部的“添加度量值”，以显示“选择度量值”对话框。

3.  单击以选择要在“监视器”页面上显示的其他度量值。

4.  选择了要添加到“监视器”页面的度量值之后，请单击“确定”。

5.  将度量值添加到“监视器”页面之后，单击以启用/禁用各度量值旁边的选项框来给页面顶部的图表添加/移除度量值。

6.  若要从“监视器”页移除度量值，请选中要移除的度量值，然后单击页面底部的“删除度量值”图标。

以下列表说明了你在“监视器”页面上的图表中可以查看的度量值：

-   **CPU 时间** - 网站的 CPU 使用情况的度量值。
-   **请求** - 客户端对网站的请求的计数。
-   **输出的数据** - 由网站发送到客户端的数据的度量值。
-   **输入的数据** - 由网站从客户端收到的数据的度量值。
-   **HTTP 客户端错误** - 发送的 Http“4xx 客户端错误”消息的数目。
-   **Http 服务器错误** - 已发送的 HTTP“5xx 服务器错误”消息数。
-   **HTTP 成功** - 发送的 Http“2xx 成功”消息的数目。
-   **HTTP 重定向** - 发送的 Http“3xx 重定向”消息的数目。
-   **HTTP 401 错误** - 发送的 Http“401 未授权”消息的数目。
-   **Http 403 错误** - 已发送的 HTTP“403 禁止访问”消息数。
-   **HTTP 404 错误** - 发送的 Http“404 找不到”消息的数目。
-   **HTTP 406 错误** - 发送的 Http“406 不可接受”消息的数目。

## <a name="howtoreceivealerts"></a>如何：接收来自网站度量值的警报

在“标准”网站模式中，可以收到基于网站监视度量值的警报。该警报功能要求你首先配置用于监视的 Web 终结点，你可以在“配置”页的“监视器”部分中进行此配置。然后，在 Azure 管理门户的**“设置”**页上，可以创建规则，当所选度量值达到指定的值时，触发警报。你还可以选择在触发警报时发送电子邮件。有关更多信息，请参见[如何：在 Azure 中接收警报通知和管理警报规则][如何：在 Azure 中接收警报通知和管理警报规则]。

## <a name="howtoviewusage"></a>如何：查看网站的使用率配额

从网站的“伸缩”管理页面可将网站配置为以“共享”或“标准”网站模式运行。每个 Azure 订阅均有权访问为在“共享”网站模式中每个区域运行最多 100 个网站所提供的一组资源。对于为此目的而向每个网站订阅提供的该组资源可由同一地理区域中配置为在“共享”模式中运行的其他网站共享。由于共享这些资源是为了供其他网站使用，因此所有订阅对这些资源的使用是受限的。订阅使用这些资源存在限制，该限制以各网站“仪表板”管理页面的使用率概述部分下列出的使用率配额形式表示。

**注意**
网站配置为以“标准”模式运行时，会被分配专用资源，资源大小等同于 [Azure 的虚拟机和云服务大小][Azure 的虚拟机和云服务大小]的表中的“小型”（默认值）、“中型”或“大型”虚拟机大小。对于可用于在“标准”模式下运行网站的订阅，没有针对资源的限制。但是，每个区域可创建的“标准”模式网站的数量是 500。

### 查看为“共享”网站模式配置的网站的使用率配额

若要确定网站对资源使用率配额的影响程度，请执行下列步骤：

1.  打开网站的“仪表板”管理页面。
2.  在“使用概览”部分下，会显示“输出的数据”、“CPU 时间”和“文件系统存储”的使用率配额。为每种资源显示的绿色条指明订阅的资源使用率配额由当前网站使用了多少，为每种资源显示的灰色条指明订阅的资源使用率配额由与你的网站订阅相关联的所有其他共享模式网站使用了多少。

资源使用率配额可帮助阻止过度使用下列资源：

-   **输出的数据** - 从以“共享”模式运行的网站按当前配额间隔（24 小时）发送到其客户端的数据量的度量值。
-   **CPU 时间** - 由以“共享”模式运行的网站在当前配额时间间隔中使用的 CPU 时间量。
-   **文件系统存储** - 以“共享”模式运行的网站使用的文件系统存储量。

当超出订阅的使用率配额时，Azure 会采取措施停止资源的过度使用。这样是为了防止订户耗尽资源进而损害其他订户。

## <a name="resourceusage"></a>如何：减少资源使用率

由于 Azure 通过度量 24 小时配额间隔期间订阅的共享模式网站所使用的资源来计算资源使用率配额，所以请考虑以下情况：

-   由于配置为以共享模式运行的网站的数量会不断增长，所以就有可能超出共享模式资源使用率配额。
    如果即将超出资源使用率配额，请考虑减少配置为以共享模式运行的网站的数量。
-   与此类似的是，由于以共享模式运行的任何网站的实例数量也会增加，所以超出共享模式资源使用率配额也是有可能的。
    如果即将超出资源使用率配额，请考虑回调共享模式网站的其他实例。

## <a name="exceeded"></a>超出使用率配额时出现的情况

如果在一个配额间隔（24 小时）中超出订阅的资源使用率配额，Azure 会采取以下措施：

-   **输出的数据** - 超出配置时，Azure 会在当前配额间隔的剩余时间内停止订阅的配置为以“共享”模式运行的所有网站。Azure 将在下一配额间隔开始时启动该网站。

-   **CPU 时间** - 超出配置时，Azure 会在当前配额间隔的剩余时间内停止订阅的配置为以“共享”模式运行的所有网站。Azure 将在下一配额间隔开始时启动该网站。

-   **文件系统存储** - 如果部署订阅的配置为以共享模式运行的任何网站将导致超出文件系统存储使用率配额，Azure 会阻止该部署。如果文件系统存储资源已增加到其配额允许的最大大小，则仍可访问文件系统存储来进行读取操作，但会阻止任何写入操作，包括常规网站活动所需的操作。这时，你可以配置以共享网站模式运行的一个或多个网站以标准网站模式运行，或将文件系统存储使用率减少到文件系统存储使用率配额之下。

## <a name="howtoconfigdiagnostics"></a>如何：为网站配置诊断和下载日志

网站的“配置”管理页面上启用了诊断。有两种类型的诊断：“应用程序诊断”和“网站诊断”。

#### 应用程序诊断

“配置”管理页面的“应用程序诊断”部分控制着应用程序产生的信息的日志记录，在记录应用程序内发生的事件时适用。例如，当你的应用程序中发生错误时，你可能会需要向用户提供友好的错误信息，同时将更详细的错误信息写入日志供以后分析。

你可以启用或禁用以下应用程序诊断：

-   **应用程序日志记录（文件系统）** - 启用对应用程序生成的信息的日志记录。“日志记录级别”字段确定是否记录“错误”、“警告”或“信息”级别信息。你也可以选择“详细”，这种模式将记录该应用程序产生的所有信息。

    此设置产生的日志存储在网站的文件系统中，可使用下面“下载网站的日志文件”中介绍的步骤来下载。

-   **应用程序日志记录（表系统）** - 启用对应用程序生成的信息的日志记录，类似于应用程序日志记录（文件系统）选项。但是，日志信息以表的形式存储在 Azure 存储帐户中。

    若要指定 Azure 存储帐户和表，请依次选择“打开”、“日志记录级别”和“管理表存储”。指定要使用的存储帐户和表，或者创建一个新表。

    可以使用 Azure 存储客户端来访问存储在表中的日志信息。

-   **应用程序日志记录（Blob 存储）** - 启用对应用程序生成的信息的日志记录，类似于应用程序日志记录（表存储）选项。但是，日志信息以 blob 的形式存储在 Azure 存储帐户中。

    若要指定 Azure 存储帐户和 blob，请依次选择“打开”、“日志记录级别”和“管理 Blob 存储”。指定要使用的存储帐户、Blob 容器和 Blob 名称，或者创建一个新容器和 Blob。

有关 Azure 存储帐户的详细信息，请参阅[如何管理存储帐户][如何管理存储帐户]。

<div class="dev-callout"> 
<b>说明</b> 
<p>只有 .NET 应用程序支持到表或 Blob 存储中的应用程序日志记录。</p> </div>

由于将应用程序日志记录到存储要求使用存储客户端来查看日志数据，所以在你计划使用了解如何从 Azure 表或 Blob 存储中直接读取和处理数据的服务或应用程序时，它是最有用的。将日志记录到文件系统会产生可使用 FTP 或其他工具下载到你本地计算机的文件，本节后面会有所说明。

<div class="dev-callout"> 
<b>说明</b> 
<p><b>应用程序诊断（文件系统）</b>、<b>应用程序诊断（表存储）</b>和<b>应用程序诊断（blob 存储）</b>可同时启用，但各自具有单独的日志级别配置。例如，作为长期日志记录解决方案，你可能需要将错误和警告记录到存储，而同时为了排解疑难问题，在检测应用程序代码之后需要启用详细级别的文件系统日志记录。</p> </div>

<div class="dev-callout"> 
<b>说明</b> 
<p>也可以从 Azure PowerShell 中使用 <b>Set-AzureWebsite</b> cmdlet 来启用诊断。</p><p>如果尚未安装 Azure PowerShell，或者尚未将其配置为使用 Azure 订阅，请参阅<a href="http://azure.microsoft.com/zh-cn/documentation/articles/install-configure-powershell/">如何使用 Azure PowerShell</a>。</p></div>

<div class="dev-callout"> 
<b>说明</b> 
<p>应用程序日志记录依赖于你的应用程序生成的日志信息。用于生成日志信息的方法以及这些信息的格式均特定于应用程序所采用的编写语言。有关使用应用程序日志记录的特定于语言的信息，请参阅以下文章：</p>
<ul>
<li><b>.NET</b> - <a href="/en-us/develop/net/common-tasks/diagnostics-logging-and-instrumentation/">为 Azure 网站启用诊断日志记录</a></li>
<li><b>Node.js</b> - <a href="/en-us/develop/nodejs/how-to-guides/Debug-Website/">如何在 Azure 网站中调试 Node.js 应用程序</a></li>
</ul>
<p>只有 .NET 应用程序支持到表或 Blob 存储中的应用程序日志记录。</p>
</div>

#### 网站诊断

“配置”管理页面的“网站诊断”部分控制着由 Web 服务器执行的日志记录，例如，针对 Web 请求、页面服务故障或页面服务所用时长的日志记录。你可以启用或禁用以下选项：

-   **Web 服务器日志记录** - 启用“Web 服务器日志记录”可使用 W3C 扩展日志文件格式保存网站日志。Web 服务器日志记录为对你的网站的所有传入请求生成一个记录，记录中包含客户端 IP 地址、请求的 URI、响应的 HTTP 状态代码以及客户端的用户代理字符串之类的信息。可将日志保存到 Azure 存储帐户或文件系统中。

    若要将 Web 服务器日志保存到 Azure 存储帐户，请选择“存储”，然后选择“管理存储”，以指定用来保留日志的存储帐户和 Azure Blob 容器。有关 Azure 存储帐户的详细信息，请参阅[如何管理存储帐户][如何管理存储帐户]。

    若要将 Web 服务器日志保存到文件系统，请选择“文件系统”。这将会启用“配额”框，在该框中，你可为日志文件设置最大磁盘空间量。最小大小为 25MB，最大为 100MB。默认大小是 35MB。

    默认情况下，从不删除 Web 服务器日志。若要指定在其之后将自动删除日志的时段，请选择“设置保持”并且在“保留期”框中输入保留日志的天数。“Azure 存储空间”和“文件系统”选项中都提供了这一设置。

-   **详细的错误信息** - 打开详细错误日志记录以记录有关 HTTP 错误的其他信息（状态代码大于 400）。

-   **失败请求跟踪** - 打开失败请求跟踪以捕获失败的客户端请求的信息，例如 400 系列 HTTP 状态代码。失败请求跟踪将会生成一个 XML 文档，其中包含对在 IIS 中通过其传递请求的模块的跟踪、模块返回的详细信息以及调用模块的时间。此信息可用于隔离其中发生了故障的组件。

为网站启用诊断后，请单击“配置”管理页底部的“保存”图标，以应用你设置的选项。

<div class="dev-callout"> 
<b>重要说明</b> 
<p>记录和跟踪会对网站提出诸多要求。建议在重现所排查的问题后关闭日志记录和跟踪功能。</p> 
</div>

### 高级配置

可通过将键/值对添加到“配置”管理页的“应用程序设置”部分，对诊断进行进一步的修改。可以从“应用程序设置”配置以下设置：

**DIAGNOSTICS\_TEXTTRACELOGDIRECTORY**

-   将用于保存应用程序日志的位置，该位置是相对于 Web 根的。

-   默认值：..\\..\\LogFiles\\Application

**DIAGNOSTICS\_TEXTTRACEMAXBUFFERSIZEBYTES**

-   在捕获应用程序日志时要使用的最大缓冲区大小。在刷新到文件或存储中之前，信息最初是写入缓冲区的。如果新信息在刷新前写入缓冲区，则可能会失去以前记录的信息。如果你的应用程序生成大量日志信息，则考虑增加缓冲区的大小。

-   默认值：10MB

**DIAGNOSTICS\_TEXTTRACEMAXLOGFOLDERSIZEBYTES**

-   存储写入文件的应用程序诊断的“应用程序”文件夹的最大大小。

-   默认值：1MB

### 下载网站的日志文件

可以使用 FTP、Azure PowerShell 或 Azure 命令行工具下载日志文件。

**FTP**

1.  打开网站的“仪表板”管理页，然后记下在“诊断日志”下列出的 FTP 站点以及在“部署/FTP 用户”下列出的帐户。FTP 站点是日志文件所在的位置，而“部署用户”下列出的帐户用于对 FTP 站点进行身份验证。
2.  如果尚未创建部署凭据，则“部署用户”下方列出的帐户将列为“未设置”。在此情况下，你必须创建部署凭据（如“仪表板”的“重置部署凭据”一节中所述），因为必须使用这些凭据对存储日志文件的 FTP 站点进行身份验证。Azure 不支持使用 Live ID 凭据对此 FTP 站点进行身份验证。
3.  请考虑使用 [FileZilla][FileZilla] 等 FTP 客户端连接到 FTP 站点。在指定凭据和查看 FTP 站点上的文件夹时，FTP 客户端通常比浏览器更容易使用。
4.  将 FTP 站点中的日志文件复制到本地计算机。

**Azure PowerShell**

1.  从“开始”屏幕或“开始”菜单，搜索“Azure PowerShell”。右键单击“Azure PowerShell”项，并选择“以管理员身份运行”。

    <div class="dev-callout"> 
<b>说明</b> 
<p>如果未安装 <b>Azure PowerShell</b>，请参阅 <a href="http://msdn.microsoft.com/zh-cn/library/azure/jj554332.aspx">Azure PowerShell Cmdlet 入门</a>以了解安装和配置信息。</p>
</div>

2.  从 Azure PowerShell 提示符处，使用以下命令下载日志文件：

        Save-AzureWebSiteLog -Name websitename

    这将下载 **websitename** 指定的网站的日志文件，并将这些文件保存到当前目录的 **log.zip** 文件中。

    还可以通过使用以下命令查看日志事件的实时流：

        Get-AzureWebSiteLog -Name websitename -Tail

    这会在 Azure PowerShell 提示符出现时将日志信息显示给提示符。

**Azure 命令行工具**

打开新命令提示符、PowerShell、bash 或终端会话，并且使用以下命令下载日志文件：

    azure site log download websitename

这将下载 **websitename** 指定的网站的日志文件，并将这些文件保存到当前目录的 **log.zip** 文件中。

还可以通过使用以下命令查看日志事件的实时流：

    azure site log tail websitename

这会将日志信息显示给从其运行命令的命令提示符、PowerShell、bash 或终端会话。

<div class="dev-callout"> 
<b>说明</b> 
<p>如果未安装 <b>azure</b> 命令，请参阅<a href="http://azure.microsoft.com/zh-cn/documentation/articles/xplat-cli/">如何使用 Azure 命令行工具</a>以了解安装和配置信息。</p>
</div>

### 读取日志文件

为网站启用日志记录和/或跟踪功能后生成的日志文件是不同的，具体取决于在网站的“配置”管理页上设置的日志记录/跟踪的级别。以下是日志文件的位置以及可用于分析日志文件的方法：

**日志文件类型：应用程序日志记录**

-   位置：/LogFiles/Application/。此文件夹包含一个或多个包含应用程序日志记录生成的信息的文本文件。记录的信息包括日期和时间、应用程序的进程 ID (PID) 和应用程序检测生成的值。

-   用于读取文件的工具：理解你的应用程序生成的值的文本编辑器或分析器

**日志文件类型：失败请求跟踪**

-   位置:/LogFiles/W3SVC\#\#\#\#\#\#\#\#\#/。此文件夹包含一个 XSL 文件和一个或多个 XML 文件。请确保将 XSL 文件下载到 XML 文件所在的目录中，因为 XSL 文件提供了在 Internet Explorer 中查看 XML 文件时格式化和筛选这些文件的内容的功能。

-   用于读取文件的工具：Internet Explorer

**日志文件类型：详细错误日志记录**

-   位置:/LogFiles/DetailedErrors/。/LogFiles/DetailedErrors/ 文件夹包含一个或多个 .htm 文件，这些文件提供已出现的所有 HTTP 错误的大量信息。

-   用于读取文件的工具：Web 浏览器

.htm 文件包括以下各部分：

-   **详细错误信息：**包括有关错误的信息，如“模块”、“处理程序”、“错误代码”和“请求的 URL”。

-   **最可能的原因：**列出导致错误的多个可能的原因。

-   **可以尝试的操作：**列出用于解决错误所报告的问题的可能的解决方案。

-   **链接和更多信息**：提供有关错误的其他摘要信息，并且还包括指向其他资源（如 Microsoft 知识库文章）的链接。

**日志文件类型：Web 服务器日志记录**

-   位置:/LogFiles/http/RawLogs。使用 [W3C 扩展日志格式][W3C 扩展日志格式]设置文件中存储的信息的格式。Azure 网站不使用 s-computername、s-ip 和 cs-version 字段。

-   用于读取文件的工具：日志分析程序。用于分析和查询 IIS 日志文件。可从 Microsoft 下载中心获得 Log Parser 2.2，网址为 <http://go.microsoft.com/fwlink/?LinkId=246619>。

## <a name="webendpointstatus"></a>如何：监视 Web 终结点状态

此功能仅在“标准”模式下提供，允许你从最多 3 个地理位置监视最多 2 个终结点。

终结点监视可从测试 Web URL 的响应时间和运行时间的分布式地理位置配置 Web 测试。该测试可对 Web URL 执行 HTTP get 操作，以从每个位置确定响应时间和运行时间。每个已配置位置每 5 分钟运行一次测试。

将使用 HTTP 响应代码监视运行时间，并且以毫秒为单位计算响应时间。当响应时间不到 30 秒且 HTTP 状态代码小于 400 时，可将运行时间视为 100%。当响应时间超过 30 秒或 HTTP 状态代码大于 400 时，运行时间为 0%。

在配置终结点监视后，你可逐层展开到各个终结点以从每个测试位置查看详监视时间间隔内响应时间和运行时间状态的详细信息。

**配置终结点监视：**

1.  打开“网站”。单击要配置的网站的名称。
2.  单击**“配置”**选项卡。
3.  转到“监视”部分以便输入你的终结点设置。
4.  输入终结点的名称。
5.  输入要监视的服务的 URL。例如，[][]<http://contoso.chinacloudapp.cn></a>。
6.  从列表中选择一个或多个地理位置。
7.  （可选）重复之前的步骤以便创建第二个终结点。
8.  单击**“保存”**。Web 终结点监视数据可能需要一段时间后才在“仪表板”和“监视器”选项卡上可用。

有关网站终结点监视的更多信息，请观看以下视频：

-   [Scott Guthrie 介绍 Azure 网站并设置终结点监视][Scott Guthrie 介绍 Azure 网站并设置终结点监视]

-   [保持 Azure 网站运行以及终结点监视 - Stefan Schackow][保持 Azure 网站运行以及终结点监视 - Stefan Schackow]

  [如何：添加网站度量值]: #websitemetrics
  [如何：接收来自网站度量值的警报]: #howtoreceivealerts
  [如何：查看网站的使用率配额]: #howtoviewusage
  [如何：减少资源使用率]: #resourceusage
  [超出使用率配额时出现的情况]: #exceeded
  [如何：为网站配置诊断和下载日志]: #howtoconfigdiagnostics
  [如何：监视 Web 终结点状态]: #webendpointstatus
  [Azure 管理门户]: http://manage.windowsazure.cn/
  [如何：在 Azure 中接收警报通知和管理警报规则]: http://go.microsoft.com/fwlink/?LinkId=309356
  [Azure 的虚拟机和云服务大小]: http://go.microsoft.com/fwlink/?LinkID=309169
  [如何管理存储帐户]: http://www.windowsazure.cn/zh-cn/manage/services/storage/how-to-manage-a-storage-account/
  [如何使用 Azure PowerShell]: http://azure.microsoft.com/zh-cn/documentation/articles/install-configure-powershell/
  [为 Azure 网站启用诊断日志记录]: /en-us/develop/net/common-tasks/diagnostics-logging-and-instrumentation/
  [如何在 Azure 网站中调试 Node.js 应用程序]: /en-us/develop/nodejs/how-to-guides/Debug-Website/
  [FileZilla]: http://go.microsoft.com/fwlink/?LinkId=247914
  [Azure PowerShell Cmdlet 入门]: http://msdn.microsoft.com/zh-cn/library/azure/jj554332.aspx
  [如何使用 Azure 命令行工具]: http://azure.microsoft.com/zh-cn/documentation/articles/xplat-cli/
  [W3C 扩展日志格式]: http://go.microsoft.com/fwlink/?LinkID=90561
  []: http://contoso.chinacloudapp.cn
  [Scott Guthrie 介绍 Azure 网站并设置终结点监视]: http://azure.microsoft.com/zh-cn/documentation/videos/websites-and-endpoint-monitoring-scottgu/
  [保持 Azure 网站运行以及终结点监视 - Stefan Schackow]: http://azure.microsoft.com/zh-cn/documentation/videos/azure-web-sites-endpoint-monitoring-and-staying-up/
