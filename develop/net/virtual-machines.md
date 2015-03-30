<properties 
  pageTitle=".Net-虚拟机 - Azure 微软云"
  metakeywords="" 
  description="" 
  services="" 
  documentationCenter=".net" 
  authors="" 
  manager="Tiffena" 
  editor="EricChen"/>

#.NET Azure 文档

- 使用 .NET 和 Visual Studio 在数秒内部署新的或现有的应用程序。
- 从 TFS 或源代码存储库（如 GitHub）自动部署。
- 利用存储和 SQL Database 等托管服务扩展功能。
- 了解如何通过使用 Azure 网站、Web 作业、云服务和 VM 在云中运行 ASP.NET Web 应用程序和 .NET 项目。

##特色

####网站入门
####ASP.NET 入门
###[入门教程](/zh-cn/documentation/articles/web-sites-dotnet-get-started/)

<table width="100%" border="0" cellspacing="0" cellpadding="0">
      <tr>
        <th align="left" scope="col">开始使用</th>
        <th align="left" scope="col">API 参考</th>
      </tr>
      <tr>
        <td><a href="/zh-cn/documentation/articles/web-sites-dotnet-get-started/">Azure 网站和 ASP.NET 入门</a></td>
        <td>Blob 存储 <a href="/zh-cn/documentation/articles/storage-dotnet-how-to-use-blobs/">.NET</a> | <a href="http://msdn.microsoft.com/zh-cn/library/azure/dd135733.aspx">REST</a></td>
      </tr>
      <tr>
        <td><a href="/zh-cn/documentation/articles/dotnet-sdk/">Azure SDK for .NET 入门</a></td>
        <td>表存储 <a href="/zh-cn/documentation/articles/storage-dotnet-how-to-use-tables/">.NET</a> | <a href="http://msdn.microsoft.com/zh-cn/library/azure/dd179423.aspx">REST</a></td>
      </tr>
      <tr>
        <td><a href="/zh-cn/documentation/articles/fundamentals-introduction-to-azure/">Azure 简介</a></td>
        <td>服务管理 <a href="http://go.microsoft.com/fwlink/p/?linkid=327806&clcid=0x804">.NET</a> | <a href="http://msdn.microsoft.com/zh-cn/library/azure/ee460799.aspx">REST</a></td>
      </tr>
      <tr>
        <td><a href="/zh-cn/documentation/articles/choose-web-site-cloud-service-vm/">网站、云服务或虚拟机？</a></td>
        <td><!--a href="/zh-cn/documentation/api/">更多</a--></td>
      </tr>
</table>

###虚拟机 | [网站](/zh-cn/develop/net/websites/) | [云服务](/zh-cn/develop/net/cloud-services/) | [移动](/zh-cn/develop/net/mobile/)    
    
##构建你的应用程序

###计算

- <a href="/zh-cn/documentation/articles/virtual-machines-windows-tutorial/">从市场创建 Windows 虚拟机</a>


	当你使用 Azure 管理门户中的映像市场时，创建运行 Windows Server 操作系统的虚拟机很容易。无需以前的经验，你就可以在云中创建运行 Windows Server 操作系统的虚拟机，并且可以访问和自定义该虚拟机。

- <a href="http://msdn.microsoft.com/zh-cn/library/azure/dn569263.aspx>在 Visual Studio 中创建虚拟机</a>

	了解如何通过 Visual Studio 中的服务器资源管理器来创建虚拟机。

- <a href="http://msdn.microsoft.com/zh-cn/library/azure/jj131259.aspx">从服务器资源管理器访问虚拟机</a>

	了解如何从 Visual Studio 中查看有关 Azure 虚拟机的信息。

- <a href="http://msdn.microsoft.com/zh-cn/library/azure/dn535788.aspx">利用 RDP 或 SSH 连接 Azure 虚拟机</a>

	本文简要总结了如何使用远程桌面协议 (RDP) 或安全外壳 (SSH) 客户端登录到 Microsoft Azure 虚拟机。文中还介绍了相关要求和故障排除提示，并包含了转向详细说明的链接。

- <a href="/zh-cn/documentation/articles/virtual-machines-dotnet-run-compute-intensive-task/">在虚拟机上的 .NET 中运行需要进行大量计算的任务</a>

	虚拟机是将现有应用程序快速移动到云中的一种有效方式。本教程演示如何创建 Azure 虚拟机，以及如何使用该虚拟机托管需要进行大量计算的 .NET 应用程序。然后演示如何通过使用服务总线队列远程监视计算任务，来使用 Azure 服务扩展该应用程序。

	查看服务： [虚拟机](/zh-cn/documentation/services/virtual-machines/)

###数据

- <a href="/zh-cn/documentation/articles/fundamentals-data-management-business-analytics/">Azure 数据管理和分析服务简介</a>

	了解 Azure 中可帮助你使用关系和非关系数据的技术。

- <a href="/zh-cn/documentation/articles/storage-dotnet-how-to-use-blobs/">Blob 存储功能指南</a>

	Blob 是存储大量非结构化文本或二进制数据（如视频、音频和图像）的最简单方式。Blob 是经 ISO 27001 认证的托管服务，可自动扩展以存储多达 100 TB 的数据。几乎可从任何位置通过 REST 和客户端 API 访问这些 Blob。

- <a href="/zh-cn/documentation/articles/storage-dotnet-how-to-use-tables/">表存储功能指南</a>

	表为需要存储大量非结构化数据的应用程序提供 NoSQL 容量。表是经 ISO 27001 认证的托管服务，可自动扩展以存储多达 100 TB 的数据。几乎可从任何位置通过 REST 和托管 API 访问这些表。

- <a href="/zh-cn/documentation/articles/storage-dotnet-how-to-use-queues/">队列存储功能指南</a>

	Azure 队列可存储大量消息，用户可以通过经验证的呼叫，使用 HTTP 或 HTTPS 从世界任何地方访问这些消息。队列存储的常见用途包括：创建积压工作以进行异步处理，从 Azure Web 角色向 Azure 辅助角色传递消息。

- <a href="/zh-cn/documentation/articles/storage-dotnet-how-to-use-files/">文件存储功能指南</a>

	文件存储使用标准 SMB 2.1 协议为应用程序提供共享存储。Azure 虚拟机和云服务可通过安装的共享跨应用程序组件共享文件数据，而本地应用程序可通过文件存储 API 访问共享中的文件数据。

- <a href="/zh-cn/documentation/articles/sql-database-dotnet-how-to-use/">SQL Database 功能指南</a>

	对于需要功能完备的关系型数据库即服务的应用程序，Azure 提供了 SQL Database（以前称为 SQL Azure 数据库）。SQL Database 提供高级别互操作性，允许客户利用众多主要开发框架来构建应用程序。

- <a href="/zh-cn/documentation/articles/sql-database-manage-azure-ssms/">使用 SSMS 管理 SQL Database</a>

	你可以使用 SQL Server Management Studio 管理 SQL Database 逻辑服务器和数据库。本文包括有关创建和管理数据库、创建和管理登录，以及使用动态管理视图进行监视的详细信息。

- <a href="/zh-cn/documentation/articles/hdinsight-hadoop-develop-deploy-streaming-jobs/">开发适用于 HDInsight 的 C# Hadoop Streaming 计划</a>

	了解如何在 HDInsight Emulator 上开发和测试 Hadoop Streaming MapReduce 程序，并随后使用 PowerShell 脚本在 HDInsight 上运行该程序。

	查看服务： [存储](/zh-cn/documentation/services/storage/) | [SQL Database](/zh-cn/documentation/services/sql-database/) | [HDInsight](/zh-cn/documentation/services/hdinsight/)

###应用程序服务

- [利用 Azure Active Directory 向你的 Web 应用程序添加登录](http://msdn.microsoft.com/zh-cn/library/azure/dn151790.aspx)

	了解如何构建使用 Azure AD 实现单一登录的 Web 应用程序。

- [利用 Azure Active Directory 的其他案例与解决方案](http://msdn.microsoft.com/en-us/library/azure/dn151121.aspx)

	Microsoft Azure Active Directory 可为单一登录、身份验证和集成应用程序等许多常见任务提供解决方案。本节包含了关于这些任务的深入信息。

- [Service Bus 队列功能指南](/zh-cn/documentation/articles/service-bus-dotnet-how-to-use-queues/)
	Service Bus 队列提供简单且有保证的先入先出消息传送策略，支持一系列标准协议（REST、AMQP 和 WS*）以及用于将消息放入/推出队列的 API。

- [Service Bus 主题功能指南](/zh-cn/documentation/articles/service-bus-dotnet-how-to-use-topics-subscriptions/)

	Service Bus 主题提供发布/订阅消息传送模型，以支持一对多通信。你可以选择以每个订阅为基础注册主题的筛选规则，这样就能限制哪些主题订阅接收某个主题下的哪些消息。

- [事件中心功能指南](http://msdn.microsoft.com/en-us/library/azure/dn789972.aspx)

	事件中心是消息传递实体，与队列和主题同级，可在高呑吐量的情况下实现来自一组不同的设备和服务的事件流的集合。事件中心可实现各种管理和监视方案。

- [Service Bus 中继功能指南](/zh-cn/documentation/articles/service-bus-dotnet-how-to-use-relay/)

	Service Bus 中继允许内部部署 Web 服务设计公共终结点，从而解决了内部部署应用程序和外界之间的通信难题。之后，系统可访问这些 Web 服务，这些服务将继续在世界各地的任何位置以本地方式运行。

- [Service Bus AMQP 功能指南](/zh-cn/documentation/articles/service-bus-dotnet-advanced-message-queuing/)

	此操作方法指南说明如何通过 .NET API 从 AMQP 1.0 使用 Service Bus 中转消息传递功能（队列和发布/订阅主题）。

- [利用媒体服务对媒体进行上载、编码和流传送](/zh-cn/documentation/articles/media-services-dotnet-get-started/)

	Azure 媒体服务在 Azure 上提供了一个可扩展的媒体平台。可使用媒体服务组件完成上传、存储、编码和流式传输内容等任务。可以利用系统端到端或将单个组件与你现有的工具和流程进行集成。

- [计划程序功能指南](http://msdn.microsoft.com/en-us/library/azure/dn479785.aspx)

	Azure 计划程序是一种多租户应用程序服务，可按周期性或日程表规律计划可靠操作，即使遇到网络、机器和数据中心故障时也能可靠执行。计划程序 REST API 帮助管理这些操作的通信。

- [使用 SendGrid 发送电子邮件](/zh-cn/documentation/articles/sendgrid-dotnet-how-to-send-email/)

	Azure 应用程序可以使用 SendGrid 来包括电子邮件功能。SendGrid 提供了可靠的电子邮件传递服务、实时分析和灵活的 API，使用户能够轻松地将服务合并到他们的 Azure 应用程序中。

- [将 Twilio 用于电话和 SMS](/zh-cn/documentation/articles/twilio-dotnet-how-to-use-for-voice-sms/)

	Azure 应用程序可以通过 Twilio 合并电话和短信服务 (SMS) 消息功能。可使用 Twilio API 拨打和接听电话，收发短信，以及通过现有互联网连接（包括移动连接）进行语音通信。

- [使用 Blitline 处理图像](/zh-cn/documentation/articles/store-blitline-how-to-use/)

	Blitline 是一项基于云计算的图像处理服务。本指南介绍如何访问 Blitline 服务以及如何将作业提交到 Blitline。

	查看服务： [Active Directory](/zh-cn/documentation/services/active-directory/) | [多重身份验证](/zh-cn/documentation/services/multi-factor-authentication/) | [Service Bus](/zh-cn/documentation/services/service-bus/) | [媒体服务](/zh-cn/develop/media-services/) | [计划程序](/zh-cn/documentation/services/scheduler/)

###云就绪

- [构建实际云应用程序](http://go.microsoft.com/fwlink/p/?linkid=398598&clcid=0x804)
            
	本系列的文章将指导你通过基于模式的方法构建实际云解决方案。这些模式适用于开发流程以及体系结构和编码实践。
            
- [云设计模式](/zh-cn/documentation/articles/architecture-overview/)
            
	了解如何在 Azure 中实现常见设计模式。
            
- [托管缓存功能指南](/zh-cn/documentation/articles/cache-dotnet-how-to-use-service/)
            
	Azure 托管缓存是内存中的分布式可扩展解决方案，可帮助你通过对数据进行超高速访问来构建高度可扩展且高度可响应的应用程序。本指南显示了如何从 .NET 应用程序使用托管缓存。
            
- [Redis 缓存功能指南](/zh-cn/documentation/articles/cache-dotnet-how-to-use-azure-redis-cache/)
            
	Microsoft Azure Redis 缓存建立在常用开放源 Redis 缓存的基础之上，目前提供预览版。这使你能够访问 Microsoft 管理的安全、专用的 Redis 缓存。本指南将帮助你开始在 .NET 中使用 Azure Redis 缓存。
            
- [CDN 功能指南](/zh-cn/documentation/articles/cdn-how-to-use/)
            
	Azure 内容交付网络 (CDN) 提供了一个全局解决方案，通过在世界各地的物理节点缓存内容来交付高带宽内容。本文介绍了如何开始使用 CDN。
            
- [管理虚拟机的可用性](/zh-cn/documentation/articles/manage-availability-virtual-machines/)
            
	了解如何在 Azure 中使用多台虚拟机，以确保在出现本地网络故障、本地磁盘硬件故障以及计划内停机时，你的应用程序仍然可用。
            
- [了解网络安全性的基本知识](http://go.microsoft.com/fwlink/p/?linkid=389558&clcid=0x804)
            
	本白皮书概述了在连接网络中的虚拟机时应如何使用 Azure 内置的可配置安全功能。
            
- [缩放云服务或虚拟机](/zh-cn/documentation/articles/cloud-services-how-to-scale/)
            
	在 Azure 管理门户的&ldquo;缩放&rdquo;页上，你可以手动缩放应用程序，也可以设置参数使其自动缩放。你可以缩放运行 Web 角色、辅助角色或虚拟机的应用程序。
            
	查看服务： <a href="/zh-cn/documentation/services/cache/">缓存</a> | <a href="/zh-cn/documentation/services/cdn/">CDN</a>
      
##调试和管理你的应用程序
      
###调试
      
- [在 Visual Studio 中调试云服务或虚拟机](http://msdn.microsoft.com/en-us/library/azure/ff683670.aspx)
            
	了解如何使用针对虚拟机的新远程调试功能和本机代码调试功能来远程诊断应用程序问题。
            
###监视
      
- [监视和遥测（用 Azure 构建实际云应用程序）](http://www.asp.net/aspnet/overview/developing-apps-with-windows-azure/building-real-world-cloud-apps-with-windows-azure/monitoring-and-telemetry)
            
	借助良好的遥测和日志记录系统，你可以随时了解应用程序使用情况，而当有应用程序出现故障时，你可以立即查明原因并能获取有帮助的故障排除信息。本章节介绍了如何选择并充分利用遥测系统。
            
- [了解 Azure 中的监视警告和通知](http://msdn.microsoft.com/zh-cn/library/azure/dn306639.aspx)
            
	了解如何理解并使用 Azure 管理门户中的内置警报和通知。
            
##工具和自动化
      
###工具
      
- [.NET 的 Azure SDK 发行说明](http://go.microsoft.com/fwlink/p/?linkid=296710&clcid=0x804)
            
	了解 Azure SDK for .NET 中包含的最新更新。
            
- [适用于 Visual Studio 的 Azure 工具入门](http://msdn.microsoft.com/zh-cn/library/azure/ff687127.aspx)
            
	了解如何使用 Azure Tools for Visual Studio 更高效地从 Visual Studio 开发、部署和调试 Azure 应用程序。
            
###自动化
      
- [安装和配置 Azure PowerShell](/zh-cn/documentation/articles/install-configure-powershell/)
            
	Windows PowerShell for Azure 提供用于通过 Windows PowerShell cmdlet 开发和部署 Azure 应用程序的命令行环境。本节介绍如何使用 Windows PowerShell cmdlet 创建、部署和管理 Azure 资源。
            
- [从 Visual Studio 生成 PowerShell 脚本](http://msdn.microsoft.com/zh-cn/library/azure/dn642480.aspx)
            
	在 Visual Studio 中创建 Web 应用程序时，你可生成 PowerShell 脚本，以供日后用于将应用程序自动发布到 Azure 网站或虚拟机。你可以在 Visual Studio 编辑器中编辑和扩展 PowerShell 脚本以适应你的要求，或将脚本与现有构建、测试和发布脚本相集成。
            
- [使用 Azure Management Libraries for .NET](http://go.microsoft.com/fwlink/p/?linkid=327806&clcid=0x804)
            
	了解如何通过简单的 .NET API 来管理云服务、虚拟机、虚拟网络、网站、存储帐户等。
	