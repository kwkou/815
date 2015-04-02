<properties 
  pageTitle=".Net-网站 - Azure 微软云"
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
- 利用存储和 SQL 数据库 等托管服务扩展功能。
- 了解如何通过使用 Azure 网站、Web 作业、云服务和 VM 在云中运行 ASP.NET Web 应用程序和 .NET 项目。

##特色

####网站入门
####ASP.NET 入门
###[入门教程](/documentation/articles/web-sites-dotnet-get-started/)

<table width="100%" border="0" cellspacing="0" cellpadding="0">
      <tr>
        <th align="left" scope="col">开始使用</th>
        <th align="left" scope="col">API 参考</th>
      </tr>
      <tr>
        <td><a href="/documentation/articles/web-sites-dotnet-get-started/">Azure 网站和 ASP.NET 入门</a></td>
        <td>Blob 存储 <a href="/documentation/articles/storage-dotnet-how-to-use-blobs/">.NET</a> | <a href="http://msdn.microsoft.com/zh-cn/library/azure/dd135733.aspx">REST</a></td>
      </tr>
      <tr>
        <td><a href="/documentation/articles/dotnet-sdk/">Azure SDK for .NET 入门</a></td>
        <td>表存储 <a href="/documentation/articles/storage-dotnet-how-to-use-tables/">.NET</a> | <a href="http://msdn.microsoft.com/zh-cn/library/azure/dd179423.aspx">REST</a></td>
      </tr>
      <tr>
        <td><a href="/documentation/articles/fundamentals-introduction-to-azure/">Azure 简介</a></td>
        <td>服务管理 <a href="http://go.microsoft.com/fwlink/p/?linkid=327806&clcid=0x804">.NET</a> | <a href="http://msdn.microsoft.com/zh-cn/library/azure/ee460799.aspx">REST</a></td>
      </tr>
      <tr>
        <td><a href="/documentation/articles/choose-web-site-cloud-service-vm/">网站、云服务或虚拟机？</a></td>
        <td><a href="/documentation/api/">更多</a></td>
      </tr>
</table>

###[虚拟机](/develop/net/virtual-machines/) | 网站 | [云服务](/develop/net/cloud-services/) | [移动](/develop/net/mobile/) 

##构建你的应用程序

###计算
- [利用 Visual Studio 创建和部署网站](/documentation/articles/web-sites-dotnet-get-started/)

	了解如何只需几分钟即可从 Visual Studio 创建 ASP.NET MVC 网站并进行部署。然后，了解通过 Azure 管理门户和 Visual Studio 提供的高级管理功能，通过这些功能可轻松监视你的应用程序、查看日志和缩放应用以使用保留资源。
          
- [使用成员资格、OAuth 和 SQL 数据库 创建和部署安全的 ASP.NET MVC 5 应用程序](/documentation/articles/web-sites-dotnet-deploy-aspnet-mvc-app-membership-oauth-sql-database/)
            
	了解如何构建安全的 ASP.NET MVC Web 应用程序，以便存储和访问 SQL 数据库中的数据，并使用户能够使用 Yahoo 凭据进行登录。
	
- [使用成员资格、OAuth 和 SQL 数据库 创建和部署安全的 ASP.NET Web 表单应用程序](/documentation/articles/web-sites-dotnet-deploy-aspnet-webforms-app-membership-oauth-sql-database/)
            
	了解如何构建安全的 ASP.NET Web 表单 Web 应用程序，以便存储和访问 SQL 数据库 中的数据，并使用户能够使用 Yahoo 凭据进行登录。

- [使用 Web 作业处理后端](/documentation/articles/websites-dotnet-webjobs-sdk-get-started/)
            
	Web 作业 SDK 是一种框架，可简化将后台处理添加到 Azure 网站的任务。本教程概述了 SDK 中的功能，并指导你完成创建和运行简单 Hello World 后台进程。

- [WebJob 资源](http://go.microsoft.com/fwlink/?linkid=390226&clcid=0x804)

	查找指向有关如何使用 WebJob 的文章、博文、论坛会话。

- [使用 WebMatrix 创建和部署网站](/documentation/articles/web-sites-dotnet-using-webmatrix/)
            
	使用 WebMatrix 创建和部署托管 WebMatrix Bakery 示例应用程序的新 Azure 网站。
           
- [为网站配置定制域名](/documentation/articles/web-sites-custom-domain-name/)
            
	你可以将网站映射到自己的域名（例如 contoso.com），而不是在网站 URL 的 chinacloudsites.cn 域上使用友好子域。从受欢迎的注册服务中查找域名以及查找带流量管理器和不带流量管理器的站点。
            
- [为网站配置 SSL](/documentation/articles/web-sites-configure-ssl-certificate/)
            
	安全套接字层 (SSL) 加密是用于保护通过 Internet 发送的数据的最常见方法。此常见任务讨论了如何为网站指定 HTTPS 终结点以及如何上载 SSL 证书来保护你的应用程序。
            
查看服务： [网站](/documentation/services/websites/)
      
###部署和源代码管理
      
- [如何部署 Azure 网站](/documentation/articles/web-sites-deploy/)
            
	了解可用于部署网站的选项，包括通过 Visual Studio、WebMatrix 或 FTP 进行手动部署、通过 PowerShell 等工具实现自动部署，或者通过 TFS、Visual Studio Online、Git 和 Mercurial 等系统实现源控件自动化。
  
<!--li><a href="/documentation/articles/web-sites-dotnet-orchard-cms-gallery/">通过市场创建和部署网站</a>
            <div data-show-less-more-member="true">
              了解如何通过市场创建新网站并立即部署该网站。只需不到 5 分钟的时间，你即可启动并运行新的 Orchard 网站。
            
</li-->
          
- [从源代码控制发布到 Azure 网站](/documentation/articles/web-sites-publish-source-control/)

	了解如何使用 Git 从本地计算机直接发布到网站，并了解如何实现从BitBucket、CodePlex、GitHub 或 Mercurial 等存储库网站进行持续部署。
  
- [使用 Visual Studio Online 和 Git 向 Azure 进行持续交付](/documentation/articles/cloud-services-continuous-delivery-use-vso-git/)
            
	了解如何使用 Visual Studio Online 团队项目来托管源代码 Git 存储库，并在向存储库推送提交时自动构建和部署网站或云服务。
 
- [使用 Visual Studio Online 向 Azure 进行持续交付](/documentation/articles/cloud-services-continuous-delivery-use-vso/)
            
	了解如何使用 Visual Studio Online 自动执行 Azure 应用程序的持续构建、打包和部署。
  
- [如何将 Azure Web 作业部署到 Azure 网站](/documentation/articles/websites-dotnet-deploy-webjobs/)

	了解如何使用 Visual Studio 将控制台应用程序项目作为 Azure Web 作业部署到 Azure 网站。
          
###数据

- [Azure 数据管理和分析服务简介](/documentation/articles/fundamentals-data-management-business-analytics/)
            
	了解 Azure 中可帮助你使用关系和非关系数据的技术。
 

- [Blob 存储功能指南](/documentation/articles/storage-dotnet-how-to-use-blobs/)
            
	Blob 是存储大量非结构化文本或二进制数据（如视频、音频和图像）的最简单方式。Blob 是经 ISO 27001 认证的托管服务，可自动扩展以存储多达 100 TB 的数据。几乎可从任何位置通过 REST 和客户端 API 访问这些 Blob。

          
- [表存储功能指南](/documentation/articles/storage-dotnet-how-to-use-tables/)

	表为需要存储大量非结构化数据的应用程序提供 NoSQL 容量。表是经 ISO 27001 认证的托管服务，可自动扩展以存储多达 100 TB 的数据。几乎可从任何位置通过 REST 和托管 API 访问这些表。
            
          
- [队列存储功能指南](/documentation/articles/storage-dotnet-how-to-use-queues/)

	Azure 队列可存储大量消息，用户可以通过经验证的呼叫，使用 HTTP 或 HTTPS 从世界任何地方访问这些消息。队列存储的常见用途包括：创建积压工作以进行异步处理，从 Azure Web 角色向 Azure 辅助角色传递消息。
            
          
- [文件存储功能指南](/documentation/articles/storage-dotnet-how-to-use-files/)

	文件存储使用标准 SMB 2.1 协议为应用程序提供共享存储。Azure 虚拟机和云服务可通过安装的共享跨应用程序组件共享文件数据，而本地应用程序可通过文件存储 API 访问共享中的文件数据。
            
          
- [如何通过 Web 作业 SDK 使用 Azure 队列存储](/documentation/articles/websites-dotnet-webjobs-sdk-storage-queues-how-to/)

	了解如何为使用 Azure 队列存储服务和 Azure Web 作业 SDK 版本 1.0.0 的常见方案编写 C# 代码。
          
- [SQL 数据库 功能指南](/documentation/articles/sql-database-dotnet-how-to-use/)

	对于需要功能完备的关系型数据库即服务的应用程序，Azure 提供了 SQL 数据库（以前称为 SQL Azure 数据库）。SQL 数据库 提供高级别互操作性，允许客户利用众多主要开发框架来构建应用程序。
            
          
- [使用 SSMS 管理 SQL 数据库](/documentation/articles/sql-database-manage-azure-ssms/)

	你可以使用 SQL Server Management Studio 管理 SQL 数据库 逻辑服务器和数据库。本文包括有关创建和管理数据库、创建和管理登录，以及使用动态管理视图进行监视的详细信息。
            
          
- [使用成员资格、OAuth 和 SQL 数据库 创建和部署安全的 ASP.NET MVC 5 应用程序](/documentation/articles/web-sites-dotnet-deploy-aspnet-mvc-app-membership-oauth-sql-database/)

	了解如何构建安全的 ASP.NET MVC Web 应用程序，以便存储和访问 SQL 数据库中的数据，并使用户能够使用 Yahoo 凭据进行登录。
            
<!--
- [使用 MongoLab 通过 MongoDB 在 Azure 上创建 ASP.NET 应用程序](/documentation/articles/store-mongolab-web-sites-dotnet-store-data-mongodb/)

MongoDB 是面向常用文档的 NoSQL 解决方案。在本教程中，你将了解如何创建 C#&ldquo;任务列表&rdquo;型应用程序，以便在由 MongoLab 托管的 MongoDB 实例中存储数据。
-->
            
- [开发适用于 HDInsight 的 C# Hadoop Streaming 计划](/documentation/articles/hdinsight-hadoop-develop-deploy-streaming-jobs/)

	了解如何在 HDInsight Emulator 上开发和测试 Hadoop Streaming MapReduce 程序，并随后使用 PowerShell 脚本在 HDInsight 上运行该程序。
            
          
        
查看服务： [存储](/documentation/services/storage/) | [SQL 数据库](/documentation/services/sql-database/) | [HDInsight](/documentation/services/hdinsight/)    
      
###应用程序服务
      
      
        
- [利用 Azure Active Directory 向你的 Web 应用程序添加登录](http://msdn.microsoft.com/zh-cn/library/azure/dn151790.aspx)

	了解如何构建使用 Azure AD 实现单一登录的 Web 应用程序。
            
          
- [利用 Azure Active Directory 的其他案例与解决方案](http://msdn.microsoft.com/zh-cn/library/azure/dn151121.aspx)

	Microsoft Azure Active Directory 可为单一登录、身份验证和集成应用程序等许多常见任务提供解决方案。本节包含了关于这些任务的深入信息。
            
          
- [Service Bus 队列功能指南](/documentation/articles/service-bus-dotnet-how-to-use-queues)

	Service Bus 队列提供简单且有保证的先入先出消息传送策略，支持一系列标准协议（REST、AMQP 和 WS*）以及用于将消息放入/推出队列的 API。
            
          
- [Service Bus 主题功能指南](/documentation/articles/service-bus-dotnet-how-to-use-topics-subscriptions/)

	Service Bus 主题提供发布/订阅消息传送模型，以支持一对多通信。你可以选择以每个订阅为基础注册主题的筛选规则，这样就能限制哪些主题订阅接收某个主题下的哪些消息。
            
          
- [事件中心功能指南](http://msdn.microsoft.com/zh-cn/library/azure/dn789972.aspx)

	事件中心是消息传递实体，与队列和主题同级，可在高呑吐量的情况下实现来自一组不同的设备和服务的事件流的集合。事件中心可实现各种管理和监视方案。
            
          
- [Service Bus 中继功能指南](/documentation/articles/service-bus-dotnet-how-to-use-relay/)

	Service Bus 中继允许内部部署 Web 服务设计公共终结点，从而解决了内部部署应用程序和外界之间的通信难题。之后，系统可访问这些 Web 服务，这些服务将继续在世界各地的任何位置以本地方式运行。
            
          
- [Service Bus AMQP 功能指南](/documentation/articles/service-bus-dotnet-advanced-message-queuing/)

	此操作方法指南说明如何通过 .NET API 从 AMQP 1.0 使用 Service Bus 中转消息传递功能（队列和发布/订阅主题）。
            
          
- [利用媒体服务对媒体进行上载、编码和流传送](/documentation/articles/media-services-dotnet-get-started/)
	Azure 媒体服务在 Azure 上提供了一个可扩展的媒体平台。可使用媒体服务组件完成上传、存储、编码和流式传输内容等任务。可以利用系统端到端或将单个组件与你现有的工具和流程进行集成。
            
          
- [计划程序功能指南](http://msdn.microsoft.com/zh-cn/library/azure/dn479785.aspx)

	Azure 计划程序是一种多租户应用程序服务，可按周期性或日程表规律计划可靠操作，即使遇到网络、机器和数据中心故障时也能可靠执行。计划程序 REST API 帮助管理这些操作的通信。
            
          
- [使用 Blitline 处理图像](/documentation/articles/store-blitline-how-to-use/)

	Blitline 是一项基于云计算的图像处理服务。本指南介绍如何访问 Blitline 服务以及如何将作业提交到 Blitline。
                    
查看服务： [Active Directory](/documentation/services/active-directory/) | [多重身份验证](/documentation/services/multi-factor-authentication/) | [Service Bus](/documentation/services/service-bus/) | [媒体服务](/develop/media-services/) | [计划程序](/documentation/services/scheduler/)
      
###云就绪
            
- [构建实际云应用程序](http://go.microsoft.com/fwlink/p/?linkid=398598&clcid=0x804)

	本系列的文章将指导你通过基于模式的方法构建实际云解决方案。这些模式适用于开发流程以及体系结构和编码实践。
                      
- [云设计模式](/documentation/articles/architecture-overview/)

	了解如何在 Azure 中实现常见设计模式。
            
<!--          
<li><a href="/documentation/articles/cache-dotnet-how-to-use-service/">托管缓存功能指南</a>
            <div data-show-less-more-member="true">
              Azure 托管缓存是内存中的分布式可扩展解决方案，可帮助你通过对数据进行超高速访问来构建高度可扩展且高度可响应的应用程序。本指南显示了如何从 .NET 应用程序使用托管缓存。
            
</li>
<li><a href="/documentation/articles/cache-dotnet-how-to-use-azure-redis-cache/" ms.pgarea="content" ms.cmpgrp="body" ms.cmptyp="link list link" ms.cmpnm=" | Redis 缓存功能指南" ms.title="" km.title="" ms.interactiontype="1" ms.index="3">Redis 缓存功能指南</a>
            <div data-show-less-more-member="true">
              Microsoft Azure Redis 缓存建立在常用开放源 Redis 缓存的基础之上，目前提供预览版。这使你能够访问 Microsoft 管理的安全、专用的 Redis 缓存。本指南将帮助你开始在 .NET 中使用 Azure Redis 缓存。
            
          </li>
-->
          
- [CDN 功能指南](/documentation/articles/cdn-how-to-use/)

	Azure 内容交付网络 (CDN) 提供了一个全局解决方案，通过在全国各地的物理节点缓存内容来交付高带宽内容。本文介绍了如何开始使用 CDN。
            
          
- [缩放网站](/documentation/articles/web-sites-scale/)

	了解如何使用 Azure 管理门户来缩放网站，包括利用按度量或计划自动缩放等功能。
            
<!--
http://www.asp.net/identity/overview/features-api/best-practices-for-deploying-passwords-and-other-sensitive-data-to-aspnet-and-azure">Best practices for deploying passwords and other sensitive data</a>
            This tutorial shows how your code can securely access passwords and other secure information on ASP.NET and Azure websites.
-->
          
        
查看服务： <!--a href="/documentation/services/cache/">缓存</a> |--> [CDN](/documentation/services/cdn/)
      
##调试和管理你的应用程序
      
###调试
      
      
        
- [在 Visual Studio 中对网站进行故障排除](/documentation/articles/web-sites-dotnet-troubleshoot-visual-studio/)

	了解如何使用 Visual Studio 中内置的 Azure 工具对网站中托管的应用程序进行故障排除。你可以远程运行调试模式、查看应用程序和 Web 服务器日志以及直接在 Visual Studio 中查看或编辑网站文件。
            
          
- [启用诊断日志记录](/documentation/articles/web-sites-enable-diagnostic-log/)

	Azure 提供内置诊断功能以帮助调试 Azure 网站中托管的应用程序。本文介绍如何启用诊断日志记录功能并将检测功能添加到应用程序中，以及如何访问由 Azure 记录的信息。
            
          
- [Azure 网站上的远程调试](http://azure.microsoft.com/blog/2014/05/06/introduction-to-remote-debugging-on-azure-web-sites/)（[第 1 部分](http://azure.microsoft.com/blog/2014/05/06/introduction-to-remote-debugging-on-azure-web-sites/)、[第 2 部分](http://azure.microsoft.com/blog/2014/05/07/introduction-to-remote-debugging-azure-web-sites-part-2-inside-remote-debugging/)、[第 3 部分](http://azure.microsoft.com/blog/2014/05/08/introduction-to-remote-debugging-on-azure-web-sites-part-3-multi-instance-environment-and-git/)）

	了解如何通过 Visual Studio 连接到正在运行的网站并进行远程调试。
          
###监视
      
      
        
- [监视和遥测](http://www.asp.net/aspnet/overview/developing-apps-with-windows-azure/building-real-world-cloud-apps-with-windows-azure/monitoring-and-telemetry)（用 Azure 构建实际云应用程序）

	借助良好的遥测和日志记录系统，你可以随时了解应用程序使用情况，而当有应用程序出现故障时，你可以立即查明原因并能获取有帮助的故障排除信息。本章节介绍了如何选择并充分利用遥测系统。
            
          
- [了解 Azure 中的监视警告和通知](http://msdn.microsoft.com/library/azure/dn306639.aspx)

	了解如何理解并使用 Azure 管理门户中的内置警报和通知。
            
          
- [如何使用 Azure Management Portal 来监视网站](/documentation/articles/web-sites-monitor/)

	一旦你的网站启动并运行后，你就可以监视其它的性能了。根据显示的内容，你可以配置站点以输出诊断日志来帮助你解决性能问题。你也可以查看特定使用率级别的配额使用量情况。
            
          
- [使用 New Relic 监视网站](/documentation/articles/store-new-relic-web-sites-dotnet-application-performance-management/)

	New Relic 是以开发人员为主的工具，用于监视应用程序，并使用户能够深入了解应用程序的性能和可靠性。本指南介绍了如何向网站中添加 New Relic 的一流性能监视。
    
##工具和自动化
      
###工具
        
- [.NET 的 Azure SDK 发行说明](https://msdn.microsoft.com/zh-cn/library/azure/dn794167.aspx?f=255&MSPPError=-2147217396)

	了解 Azure SDK for .NET 中包含的最新更新。            
          
- [适用于 Visual Studio 的 Azure 工具入门](http://msdn.microsoft.com/zh-cn/library/azure/ff687127.aspx)

	了解如何使用 Azure Tools for Visual Studio 更高效地从 Visual Studio 开发、部署和调试 Azure 应用程序。
      
###自动化      
        
- [安装和配置 Azure PowerShell](/documentation/articles/install-configure-powershell/)

	Windows PowerShell for Azure 提供用于通过 Windows PowerShell cmdlet 开发和部署 Azure 应用程序的命令行环境。本节介绍如何使用 Windows PowerShell cmdlet 创建、部署和管理 Azure 资源。
          
- [从 Visual Studio 生成 PowerShell 脚本](http://msdn.microsoft.com/zh-cn/library/azure/dn642480.aspx)

	在 Visual Studio 中创建 Web 应用程序时，你可生成 PowerShell 脚本，以供日后用于将应用程序自动发布到 Azure 网站或虚拟机。你可以在 Visual Studio 编辑器中编辑和扩展 PowerShell 脚本以适应你的要求，或将脚本与现有构建、测试和发布脚本相集成。
          
- [使用 Azure Management Libraries for .NET](http://go.microsoft.com/fwlink/p/?linkid=327806&clcid=0x804)

	了解如何通过简单的 .NET API 来管理云服务、虚拟机、虚拟网络、网站、存储帐户等。
            
          
        
      
    
  


