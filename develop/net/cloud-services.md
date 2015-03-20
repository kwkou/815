<properties linkid="dev-net-cloud-services" urlDisplayName=".NET开发人员中心|Azure" pageTitle="Windows Azure 开发人员中心：.NET" metaKeywords=".NET" description="" metaCanonical="" services=".NET" documentationCenter="Develop" title="" authors="" solutions="" manager="Tiffena" editor="Eric Chen" />

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
        <td><a href="/zh-cn/documentation/api/">更多</a></td>
      </tr>
</table>

###[虚拟机](/zh-cn/develop/net/virtual-machines/) | [网站](/zh-cn/develop/net/websites/) | 云服务 | [移动](/zh-cn/develop/net/mobile/) 

<div data-tab-panel-id="tab-panel">
  <div>
    <div>
      <h2>构建你的应用程序</h2>
      <div>
        <h3 data-jumpto-stop="true">计算</h3>
      </div>
      <div>
        <ul>
          <li><a href="http://azure.microsoft.com/zh-cn/documentation/articles/cloud-services-dotnet-get-started/" ms.pgarea="content" ms.cmpgrp="body" ms.cmptyp="link list link" ms.cmpnm=" | 云服务和 ASP.NET 入门" ms.title="" km.title="" ms.interactiontype="1" ms.index="0">云服务和 ASP.NET 入门</a>
            <div data-show-less-more-member="true">
              <p>了解如何创建含有 ASP.NET MVC 前端的多层 .NET 应用程序，并将其部署到 Azure 云服务。该应用程序使用 SQL Database、Blob 和队列。</p>
            </div>
          </li>
          <li><a href="http://azure.microsoft.com/zh-cn/documentation/articles/cloud-services-dotnet-multi-tier-app-using-service-bus-queues/" ms.pgarea="content" ms.cmpgrp="body" ms.cmptyp="link list link" ms.cmpnm=" | 使用 Service Bus 队列构建 .NET 多层应用程序" ms.title="" km.title="" ms.interactiontype="1" ms.index="1">使用 Service Bus 队列构建 .NET 多层应用程序</a>
            <div data-show-less-more-member="true">
              <p>构建一个前端 ASP.NET MVC Web 角色，该角色使用后端辅助角色来处理长时间运行的作业。你将学习如何创建和部署多角色解决方案，以及如何使用 Service Bus 队列和主题来实现角色间通信。</p>
            </div>
          </li>
          <li><a href="http://azure.microsoft.com/zh-cn/documentation/articles/cloud-services-configure-ssl-certificate/" ms.pgarea="content" ms.cmpgrp="body" ms.cmptyp="link list link" ms.cmpnm=" | 为云服务配置 SSL" ms.title="" km.title="" ms.interactiontype="1" ms.index="2">为云服务配置 SSL</a>
            <div data-show-less-more-member="true">
              <p>安全套接字层 (SSL) 加密是用于保护通过 Internet 发送的数据的最常见方法。此常见任务讨论了如何为 Web 角色指定 HTTPS 终结点以及如何上载 SSL 证书来保护你的应用程序。</p>
            </div>
          </li>
          <li><a href="http://azure.microsoft.com/zh-cn/documentation/articles/cloud-services-custom-domain-name/" ms.pgarea="content" ms.cmpgrp="body" ms.cmptyp="link list link" ms.cmpnm=" | 为云服务配置自定义域名" ms.title="" km.title="" ms.interactiontype="1" ms.index="3">为云服务配置自定义域名</a>
            <div data-show-less-more-member="true">
              <p>默认情况下，可以通过友好子域（例如 http://&lt;myapp&gt;.cloudapp.net 和 https://&lt;mydata&gt;.blob.core.windows.net）访问 Azure 应用程序和存储帐户。本文演示如何在你自己的自定义域（如 http://&lt;myapp&gt;.com）中公开你的应用程序和数据。</p>
            </div>
          </li>
          <li><a href="http://msdn.microsoft.com/library/azure/hh180152.aspx" ms.pgarea="content" ms.cmpgrp="body" ms.cmptyp="link list link" ms.cmpnm=" | 了解云服务角色" ms.title="" km.title="" ms.interactiontype="1" ms.index="4">了解云服务角色</a>
            <div data-show-less-more-member="true">
              <p>了解有关云服务 Web 角色和辅助角色的更多信息，以及如何有效地将二者结合以创建多层应用程序。</p>
            </div>
          </li>
        </ul>
        <p>查看服务： <a href="http://azure.microsoft.com/zh-cn/documentation/services/cloud-services/" ms.pgarea="content" ms.cmpgrp="body" ms.cmptyp="link" ms.cmpnm="云服务" ms.title="" km.title="" ms.interactiontype="1">云服务</a></p>
      </div>
    </div>
    <div>
      <div>
        <h3 data-jumpto-stop="true">部署和源代码管理</h3>
      </div>
      <div>
        <ul>
          <li><a href="http://msdn.microsoft.com/library/azure/gg433027.aspx" ms.pgarea="content" ms.cmpgrp="body" ms.cmptyp="link list link" ms.cmpnm=" | 管理云服务过渡部署和生产部署" ms.title="" km.title="" ms.interactiontype="1" ms.index="0">管理云服务过渡部署和生产部署</a>
            <div data-show-less-more-member="true">
              <p>Azure 提供生产和暂存环境，你可在其中创建服务部署。服务部署到生产或过渡环境后，会在该环境中为服务分配一个公共 IP 地址，叫做虚拟 IP 地址 (VIP)。VIP 用于与部署中角色相关的所有输入终结点。</p>
            </div>
          </li>
          <li><a href="http://msdn.microsoft.com/library/azure/ff683672.aspx" ms.pgarea="content" ms.cmpgrp="body" ms.cmptyp="link list link" ms.cmpnm=" | 从 Visual Studio 发布云服务" ms.title="" km.title="" ms.interactiontype="1" ms.index="1">从 Visual Studio 发布云服务</a>
            <div data-show-less-more-member="true">
              <p>通过使用 Azure Tools for Microsoft Visual Studio，你可直接从 Visual Studio 发布 Azure 应用程序。Visual Studio 支持以集成方式将云服务发布到暂存或生产环境中。</p>
            </div>
          </li>
          <li><a href="http://azure.microsoft.com/zh-cn/documentation/articles/cloud-services-continuous-delivery-use-vso-git/" ms.pgarea="content" ms.cmpgrp="body" ms.cmptyp="link list link" ms.cmpnm=" | 使用 Visual Studio Online 和 Git 向 Azure 进行持续交付" ms.title="" km.title="" ms.interactiontype="1" ms.index="2">使用 Visual Studio Online 和 Git 向 Azure 进行持续交付</a>
            <div data-show-less-more-member="true">
              <p>了解如何使用 Visual Studio Online 团队项目来托管源代码 Git 存储库，并在向存储库推送提交时自动构建和部署网站或云服务。</p>
            </div>
          </li>
          <li><a href="http://azure.microsoft.com/zh-cn/documentation/articles/cloud-services-continuous-delivery-use-vso/" ms.pgarea="content" ms.cmpgrp="body" ms.cmptyp="link list link" ms.cmpnm=" | 使用 Visual Studio Online 向 Azure 进行持续交付" ms.title="" km.title="" ms.interactiontype="1" ms.index="3">使用 Visual Studio Online 向 Azure 进行持续交付</a>
            <div data-show-less-more-member="true">
              <p>了解如何使用 Visual Studio Online 自动执行 Azure 应用程序的持续构建、打包和部署。</p>
            </div>
          </li>
          <li><a href="http://azure.microsoft.com/zh-cn/documentation/articles/cloud-services-dotnet-continuous-delivery/" ms.pgarea="content" ms.cmpgrp="body" ms.cmptyp="link list link" ms.cmpnm=" | 使用 Team Foundation Server 持续交付云服务" ms.title="" km.title="" ms.interactiontype="1" ms.index="4">使用 Team Foundation Server 持续交付云服务</a>
            <div data-show-less-more-member="true">
              <p>了解如何使用 Team Foundation Server 实现 Azure 云服务的持续交付。此过程使你能够在签入每个代码后，自动创建服务包并将其部署到 Azure。</p>
            </div>
          </li>
        </ul>
      </div>
    </div>
    <div>
      <div>
        <h3 data-jumpto-stop="true">数据</h3>
      </div>
      <div>
        <ul>
          <li><a href="http://azure.microsoft.com/zh-cn/documentation/articles/fundamentals-data-management-business-analytics/" ms.pgarea="content" ms.cmpgrp="body" ms.cmptyp="link list link" ms.cmpnm=" | Azure 数据管理和分析服务简介" ms.title="" km.title="" ms.interactiontype="1" ms.index="0">Azure 数据管理和分析服务简介</a>
            <div data-show-less-more-member="true">
              <p>了解 Azure 中可帮助你使用关系和非关系数据的技术。</p>
            </div>
          </li>
          <li><a href="http://azure.microsoft.com/zh-cn/documentation/articles/storage-dotnet-how-to-use-blobs/" ms.pgarea="content" ms.cmpgrp="body" ms.cmptyp="link list link" ms.cmpnm=" | Blob 存储功能指南" ms.title="" km.title="" ms.interactiontype="1" ms.index="1">Blob 存储功能指南</a>
            <div data-show-less-more-member="true">
              <p>Blob 是存储大量非结构化文本或二进制数据（如视频、音频和图像）的最简单方式。Blob 是经 ISO 27001 认证的托管服务，可自动扩展以存储多达 100 TB 的数据。几乎可从任何位置通过 REST 和客户端 API 访问这些 Blob。</p>
            </div>
          </li>
          <li><a href="http://azure.microsoft.com/zh-cn/documentation/articles/storage-dotnet-how-to-use-tables/" ms.pgarea="content" ms.cmpgrp="body" ms.cmptyp="link list link" ms.cmpnm=" | 表存储功能指南" ms.title="" km.title="" ms.interactiontype="1" ms.index="2">表存储功能指南</a>
            <div data-show-less-more-member="true">
              <p>表为需要存储大量非结构化数据的应用程序提供 NoSQL 容量。表是经 ISO 27001 认证的托管服务，可自动扩展以存储多达 100 TB 的数据。几乎可从任何位置通过 REST 和托管 API 访问这些表。</p>
            </div>
          </li>
          <li><a href="http://azure.microsoft.com/zh-cn/documentation/articles/storage-dotnet-how-to-use-queues/" ms.pgarea="content" ms.cmpgrp="body" ms.cmptyp="link list link" ms.cmpnm=" | 队列存储功能指南" ms.title="" km.title="" ms.interactiontype="1" ms.index="3">队列存储功能指南</a>
            <div data-show-less-more-member="true">
              <p>Azure 队列可存储大量消息，用户可以通过经验证的呼叫，使用 HTTP 或 HTTPS 从世界任何地方访问这些消息。队列存储的常见用途包括：创建积压工作以进行异步处理，从 Azure Web 角色向 Azure 辅助角色传递消息。</p>
            </div>
          </li>
          <li><a href="http://azure.microsoft.com/zh-cn/documentation/articles/storage-dotnet-how-to-use-files/" ms.pgarea="content" ms.cmpgrp="body" ms.cmptyp="link list link" ms.cmpnm=" | 文件存储功能指南" ms.title="" km.title="" ms.interactiontype="1" ms.index="4">文件存储功能指南</a>
            <div data-show-less-more-member="true">
              <p>文件存储使用标准 SMB 2.1 协议为应用程序提供共享存储。Azure 虚拟机和云服务可通过安装的共享跨应用程序组件共享文件数据，而本地应用程序可通过文件存储 API 访问共享中的文件数据。</p>
            </div>
          </li>
          <li><a href="http://azure.microsoft.com/zh-cn/documentation/articles/sql-database-dotnet-how-to-use/" ms.pgarea="content" ms.cmpgrp="body" ms.cmptyp="link list link" ms.cmpnm=" | SQL Database 功能指南" ms.title="" km.title="" ms.interactiontype="1" ms.index="5">SQL Database 功能指南</a>
            <div data-show-less-more-member="true">
              <p>对于需要功能完备的关系型数据库即服务的应用程序，Azure 提供了 SQL Database（以前称为 SQL Azure 数据库）。SQL Database 提供高级别互操作性，允许客户利用众多主要开发框架来构建应用程序。</p>
            </div>
          </li>
          <li><a href="http://azure.microsoft.com/zh-cn/documentation/articles/sql-database-manage-azure-ssms/" ms.pgarea="content" ms.cmpgrp="body" ms.cmptyp="link list link" ms.cmpnm=" | 使用 SSMS 管理 SQL Database" ms.title="" km.title="" ms.interactiontype="1" ms.index="6">使用 SSMS 管理 SQL Database</a>
            <div data-show-less-more-member="true">
              <p>你可以使用 SQL Server Management Studio 管理 SQL Database 逻辑服务器和数据库。本文包括有关创建和管理数据库、创建和管理登录，以及使用动态管理视图进行监视的详细信息。</p>
            </div>
          </li>
          <li><a href="http://azure.microsoft.com/zh-cn/documentation/articles/hdinsight-hadoop-develop-deploy-streaming-jobs/" ms.pgarea="content" ms.cmpgrp="body" ms.cmptyp="link list link" ms.cmpnm=" | 开发适用于 HDInsight 的 C# Hadoop Streaming 计划" ms.title="" km.title="" ms.interactiontype="1" ms.index="7">开发适用于 HDInsight 的 C# Hadoop Streaming 计划</a>
            <div data-show-less-more-member="true">
              <p>了解如何在 HDInsight Emulator 上开发和测试 Hadoop Streaming MapReduce 程序，并随后使用 PowerShell 脚本在 HDInsight 上运行该程序。</p>
            </div>
          </li>
        </ul>
        <p>查看服务： <a href="http://azure.microsoft.com/zh-cn/documentation/services/storage/" ms.pgarea="content" ms.cmpgrp="body" ms.cmptyp="link" ms.cmpnm="存储" ms.title="" km.title="" ms.interactiontype="1">存储</a> | <a href="http://azure.microsoft.com/zh-cn/documentation/services/sql-database/" ms.pgarea="content" ms.cmpgrp="body" ms.cmptyp="link" ms.cmpnm="SQL Database" ms.title="" km.title="" ms.interactiontype="1">SQL Database</a> | <a href="http://azure.microsoft.com/zh-cn/documentation/services/hdinsight/" ms.pgarea="content" ms.cmpgrp="body" ms.cmptyp="link" ms.cmpnm="HDInsight" ms.title="" km.title="" ms.interactiontype="1">HDInsight</a></p>
      </div>
    </div>
    <div>
      <div>
        <h3 data-jumpto-stop="true">应用程序服务</h3>
      </div>
      <div>
        <ul>
          <li><a href="http://msdn.microsoft.com/library/azure/dn151790.aspx" ms.pgarea="content" ms.cmpgrp="body" ms.cmptyp="link list link" ms.cmpnm=" | 利用 Azure Active Directory 向你的 Web 应用程序添加登录" ms.title="" km.title="" ms.interactiontype="1" ms.index="0">利用 Azure Active Directory 向你的 Web 应用程序添加登录</a>
            <div data-show-less-more-member="true">
              <p>了解如何构建使用 Azure AD 实现单一登录的 Web 应用程序。</p>
            </div>
          </li>
          <li><a href="http://msdn.microsoft.com/en-us/library/azure/dn151121.aspx" ms.pgarea="content" ms.cmpgrp="body" ms.cmptyp="link list link" ms.cmpnm=" | 利用 Azure Active Directory 的其他案例与解决方案" ms.title="" km.title="" ms.interactiontype="1" ms.index="1">利用 Azure Active Directory 的其他案例与解决方案</a>
            <div data-show-less-more-member="true">
              <p>Microsoft Azure Active Directory 可为单一登录、身份验证和集成应用程序等许多常见任务提供解决方案。本节包含了关于这些任务的深入信息。</p>
            </div>
          </li>
          <li><a href="http://azure.microsoft.com/zh-cn/documentation/articles/service-bus-dotnet-how-to-use-queues/" ms.pgarea="content" ms.cmpgrp="body" ms.cmptyp="link list link" ms.cmpnm=" | Service Bus 队列功能指南" ms.title="" km.title="" ms.interactiontype="1" ms.index="2">Service Bus 队列功能指南</a>
            <div data-show-less-more-member="true">
              <p>Service Bus 队列提供简单且有保证的先入先出消息传送策略，支持一系列标准协议（REST、AMQP 和 WS*）以及用于将消息放入/推出队列的 API。</p>
            </div>
          </li>
          <li><a href="http://azure.microsoft.com/zh-cn/documentation/articles/service-bus-dotnet-how-to-use-topics-subscriptions/" ms.pgarea="content" ms.cmpgrp="body" ms.cmptyp="link list link" ms.cmpnm=" | Service Bus 主题功能指南" ms.title="" km.title="" ms.interactiontype="1" ms.index="3">Service Bus 主题功能指南</a>
            <div data-show-less-more-member="true">
              <p>Service Bus 主题提供发布/订阅消息传送模型，以支持一对多通信。你可以选择以每个订阅为基础注册主题的筛选规则，这样就能限制哪些主题订阅接收某个主题下的哪些消息。</p>
            </div>
          </li>
          <li><a href="http://msdn.microsoft.com/en-us/library/azure/dn789972.aspx" ms.pgarea="content" ms.cmpgrp="body" ms.cmptyp="link list link" ms.cmpnm=" | 事件中心功能指南" ms.title="" km.title="" ms.interactiontype="1" ms.index="4">事件中心功能指南</a>
            <div data-show-less-more-member="true">
              <p>事件中心是消息传递实体，与队列和主题同级，可在高呑吐量的情况下实现来自一组不同的设备和服务的事件流的集合。事件中心可实现各种管理和监视方案。</p>
            </div>
          </li>
          <li><a href="http://azure.microsoft.com/zh-cn/documentation/articles/service-bus-dotnet-how-to-use-relay/" ms.pgarea="content" ms.cmpgrp="body" ms.cmptyp="link list link" ms.cmpnm=" | Service Bus 中继功能指南" ms.title="" km.title="" ms.interactiontype="1" ms.index="5">Service Bus 中继功能指南</a>
            <div data-show-less-more-member="true">
              <p>Service Bus 中继允许内部部署 Web 服务设计公共终结点，从而解决了内部部署应用程序和外界之间的通信难题。之后，系统可访问这些 Web 服务，这些服务将继续在世界各地的任何位置以本地方式运行。</p>
            </div>
          </li>
          <li><a href="http://azure.microsoft.com/zh-cn/documentation/articles/service-bus-dotnet-advanced-message-queuing/" ms.pgarea="content" ms.cmpgrp="body" ms.cmptyp="link list link" ms.cmpnm=" | Service Bus AMQP 功能指南" ms.title="" km.title="" ms.interactiontype="1" ms.index="6">Service Bus AMQP 功能指南</a>
            <div data-show-less-more-member="true">
              <p>此操作方法指南说明如何通过 .NET API 从 AMQP 1.0 使用 Service Bus 中转消息传递功能（队列和发布/订阅主题）。</p>
            </div>
          </li>
          <li><a href="http://azure.microsoft.com/zh-cn/documentation/articles/media-services-dotnet-get-started/" ms.pgarea="content" ms.cmpgrp="body" ms.cmptyp="link list link" ms.cmpnm=" | 利用媒体服务对媒体进行上载、编码和流传送" ms.title="" km.title="" ms.interactiontype="1" ms.index="7">利用媒体服务对媒体进行上载、编码和流传送</a>
            <div data-show-less-more-member="true">
              <p>Azure 媒体服务在 Azure 上提供了一个可扩展的媒体平台。可使用媒体服务组件完成上传、存储、编码和流式传输内容等任务。可以利用系统端到端或将单个组件与你现有的工具和流程进行集成。</p>
            </div>
          </li>
          <li><a href="http://msdn.microsoft.com/en-us/library/azure/dn479785.aspx" ms.pgarea="content" ms.cmpgrp="body" ms.cmptyp="link list link" ms.cmpnm=" | 计划程序功能指南" ms.title="" km.title="" ms.interactiontype="1" ms.index="8">计划程序功能指南</a>
            <div data-show-less-more-member="true">
              <p>Azure 计划程序是一种多租户应用程序服务，可按周期性或日程表规律计划可靠操作，即使遇到网络、机器和数据中心故障时也能可靠执行。计划程序 REST API 帮助管理这些操作的通信。</p>
            </div>
          </li>
          <li><a href="http://azure.microsoft.com/en-us/documentation/articles/sendgrid-dotnet-how-to-send-email/" ms.pgarea="content" ms.cmpgrp="body" ms.cmptyp="link list link" ms.cmpnm=" | 使用 SendGrid 发送电子邮件" ms.title="" km.title="" ms.interactiontype="1" ms.index="9">使用 SendGrid 发送电子邮件</a>
            <div data-show-less-more-member="true">
              <p>Azure 应用程序可以使用 SendGrid 来包括电子邮件功能。SendGrid 提供了可靠的电子邮件传递服务、实时分析和灵活的 API，使用户能够轻松地将服务合并到他们的 Azure 应用程序中。</p>
            </div>
          </li>
          <li><a href="http://azure.microsoft.com/en-us/documentation/articles/twilio-dotnet-how-to-use-for-voice-sms/" ms.pgarea="content" ms.cmpgrp="body" ms.cmptyp="link list link" ms.cmpnm=" | 将 Twilio 用于电话和 SMS" ms.title="" km.title="" ms.interactiontype="1" ms.index="10">将 Twilio 用于电话和 SMS</a>
            <div data-show-less-more-member="true">
              <p>Azure 应用程序可以通过 Twilio 合并电话和短信服务 (SMS) 消息功能。可使用 Twilio API 拨打和接听电话，收发短信，以及通过现有互联网连接（包括移动连接）进行语音通信。</p>
            </div>
          </li>
          <li><a href="http://azure.microsoft.com/zh-cn/documentation/articles/store-blitline-how-to-use/" ms.pgarea="content" ms.cmpgrp="body" ms.cmptyp="link list link" ms.cmpnm=" | 使用 Blitline 处理图像" ms.title="" km.title="" ms.interactiontype="1" ms.index="11">使用 Blitline 处理图像</a>
            <div data-show-less-more-member="true">
              <p>Blitline 是一项基于云计算的图像处理服务。本指南介绍如何访问 Blitline 服务以及如何将作业提交到 Blitline。</p>
            </div>
          </li>
        </ul>
        <p>查看服务： <a href="http://azure.microsoft.com/zh-cn/documentation/services/active-directory/" ms.pgarea="content" ms.cmpgrp="body" ms.cmptyp="link" ms.cmpnm="Active Directory" ms.title="" km.title="" ms.interactiontype="1">Active Directory</a> | <a href="http://azure.microsoft.com/zh-cn/documentation/services/multi-factor-authentication/" ms.pgarea="content" ms.cmpgrp="body" ms.cmptyp="link" ms.cmpnm="多重身份验证" ms.title="" km.title="" ms.interactiontype="1">多重身份验证</a> | <a href="http://azure.microsoft.com/zh-cn/documentation/services/service-bus/" ms.pgarea="content" ms.cmpgrp="body" ms.cmptyp="link" ms.cmpnm="Service Bus" ms.title="" km.title="" ms.interactiontype="1">Service Bus</a> | <a href="http://azure.microsoft.com/zh-cn/develop/media-services/" ms.pgarea="content" ms.cmpgrp="body" ms.cmptyp="link" ms.cmpnm="媒体服务" ms.title="" km.title="" ms.interactiontype="1">媒体服务</a> | <a href="http://azure.microsoft.com/zh-cn/documentation/services/scheduler/" ms.pgarea="content" ms.cmpgrp="body" ms.cmptyp="link" ms.cmpnm="计划程序" ms.title="" km.title="" ms.interactiontype="1">计划程序</a></p>
      </div>
    </div>
    <div>
      <div>
        <h3 data-jumpto-stop="true">云就绪</h3>
      </div>
      <div>
        <ul>
          <li><a href="http://go.microsoft.com/fwlink/p/?linkid=398598&clcid=0x804" ms.pgarea="content" ms.cmpgrp="body" ms.cmptyp="link list link" ms.cmpnm=" | 构建实际云应用程序" ms.title="" km.title="" ms.interactiontype="1" ms.index="0">构建实际云应用程序</a>
            <div data-show-less-more-member="true">
              <p>本系列的文章将指导你通过基于模式的方法构建实际云解决方案。这些模式适用于开发流程以及体系结构和编码实践。</p>
            </div>
          </li>
          <li><a href="http://azure.microsoft.com/zh-cn/documentation/articles/architecture-overview/" ms.pgarea="content" ms.cmpgrp="body" ms.cmptyp="link list link" ms.cmpnm=" | 云设计模式" ms.title="" km.title="" ms.interactiontype="1" ms.index="1">云设计模式</a>
            <div data-show-less-more-member="true">
              <p>了解如何在 Azure 中实现常见设计模式。</p>
            </div>
          </li>
          <li><a href="http://azure.microsoft.com/zh-cn/documentation/articles/cache-dotnet-how-to-use-service/" ms.pgarea="content" ms.cmpgrp="body" ms.cmptyp="link list link" ms.cmpnm=" | 托管缓存功能指南" ms.title="" km.title="" ms.interactiontype="1" ms.index="2">托管缓存功能指南</a>
            <div data-show-less-more-member="true">
              <p>Azure 托管缓存是内存中的分布式可扩展解决方案，可帮助你通过对数据进行超高速访问来构建高度可扩展且高度可响应的应用程序。本指南显示了如何从 .NET 应用程序使用托管缓存。</p>
            </div>
          </li>
          <li><a href="http://azure.microsoft.com/zh-cn/documentation/articles/cache-dotnet-how-to-use-azure-redis-cache/" ms.pgarea="content" ms.cmpgrp="body" ms.cmptyp="link list link" ms.cmpnm=" | Redis 缓存功能指南" ms.title="" km.title="" ms.interactiontype="1" ms.index="3">Redis 缓存功能指南</a>
            <div data-show-less-more-member="true">
              <p>Microsoft Azure Redis 缓存建立在常用开放源 Redis 缓存的基础之上，目前提供预览版。这使你能够访问 Microsoft 管理的安全、专用的 Redis 缓存。本指南将帮助你开始在 .NET 中使用 Azure Redis 缓存。</p>
            </div>
          </li>
          <li><a href="http://azure.microsoft.com/zh-cn/documentation/articles/cache-dotnet-how-to-use-in-role/" ms.pgarea="content" ms.cmpgrp="body" ms.cmptyp="link list link" ms.cmpnm=" | 角色中缓存功能指南" ms.title="" km.title="" ms.interactiontype="1" ms.index="4">角色中缓存功能指南</a>
            <div data-show-less-more-member="true">
              <p>了解如何在 Azure 中实现常见设计模式。</p>
            </div>
          </li>
          <li><a href="http://azure.microsoft.com/zh-cn/documentation/articles/cdn-how-to-use/" ms.pgarea="content" ms.cmpgrp="body" ms.cmptyp="link list link" ms.cmpnm=" | CDN 功能指南" ms.title="" km.title="" ms.interactiontype="1" ms.index="5">CDN 功能指南</a>
            <div data-show-less-more-member="true">
              <p>Azure 内容交付网络 (CDN) 提供了一个全局解决方案，通过在世界各地的物理节点缓存内容来交付高带宽内容。本文介绍了如何开始使用 CDN。</p>
            </div>
          </li>
          <li><a href="http://msdn.microsoft.com/en-us/library/windowsazure/jj717232.aspx" ms.pgarea="content" ms.cmpgrp="body" ms.cmptyp="link list link" ms.cmpnm=" | 在云服务上设计大规模服务的最佳实践" ms.title="" km.title="" ms.interactiontype="1" ms.index="6">在云服务上设计大规模服务的最佳实践</a>
            <div data-show-less-more-member="true">
              <p>云计算是分布式计算；无论选择在任何平台上操作，分布式计算都需要缜密的规划和交付。本文档的目的是基于实际客户应用场景提供详细指导，展示如何利用平台即服务 (PaaS) 方法在 Microsoft Azure 和 SQL Database 上构建可扩展应用程序（使用 Web 和辅助角色，此类应用程序构建为 Microsoft Azure 云服务）。</p>
            </div>
          </li>
          <li><a href="http://www.asp.net/identity/overview/features-api/best-practices-for-deploying-passwords-and-other-sensitive-data-to-aspnet-and-azure" ms.pgarea="content" ms.cmpgrp="body" ms.cmptyp="link list link" ms.cmpnm=" | Best practices for deploying passwords and other sensitive data" ms.title="" km.title="" ms.interactiontype="1" ms.index="7">Best practices for deploying passwords and other sensitive data</a>
            <div data-show-less-more-member="true">This tutorial shows how your code can securely access passwords and other secure information on ASP.NET and Azure websites.</div>
          </li>
        </ul>
        <p>查看服务： <a href="http://azure.microsoft.com/zh-cn/documentation/services/cache/" ms.pgarea="content" ms.cmpgrp="body" ms.cmptyp="link" ms.cmpnm="缓存" ms.title="" km.title="" ms.interactiontype="1">缓存</a> | <a href="http://azure.microsoft.com/zh-cn/documentation/services/cdn/" ms.pgarea="content" ms.cmpgrp="body" ms.cmptyp="link" ms.cmpnm="CDN" ms.title="" km.title="" ms.interactiontype="1">CDN</a></p>
      </div>
    </div>
  </div>
</div>
<div data-tab-panel-id="tab-panel">
  <div>
    <div>
      <h2>调试和管理你的应用程序</h2>
      <div>
        <h3 data-jumpto-stop="true">调试</h3>
      </div>
      <div>
        <ul>
          <li><a href="http://azure.microsoft.com/zh-cn/documentation/articles/cloud-services-dotnet-diagnostics/" ms.pgarea="content" ms.cmpgrp="body" ms.cmptyp="link list link" ms.cmpnm=" | 实现云服务诊断" ms.title="" km.title="" ms.interactiontype="1" ms.index="0">实现云服务诊断</a>
            <div data-show-less-more-member="true">
              <p>通过 Azure 诊断，可以从在 Azure 中运行的应用程序收集诊断数据。可以将诊断数据用于调试和故障排除、衡量性能、监视资源使用状况、进行流量分析和容量规划以及进行审核。</p>
            </div>
          </li>
          <li><a href="http://msdn.microsoft.com/library/azure/dn186185.aspx" ms.pgarea="content" ms.cmpgrp="body" ms.cmptyp="link list link" ms.cmpnm=" | 配置 Azure 诊断" ms.title="" km.title="" ms.interactiontype="1" ms.index="1">配置 Azure 诊断</a>
            <div data-show-less-more-member="true">
              <p>了解如何通过使用 Visual Studio 轻松地配置 Azure 诊断。</p>
            </div>
          </li>
          <li><a href="http://msdn.microsoft.com/en-us/library/azure/ff683670.aspx" ms.pgarea="content" ms.cmpgrp="body" ms.cmptyp="link list link" ms.cmpnm=" | 在 Visual Studio 中调试云服务或虚拟机" ms.title="" km.title="" ms.interactiontype="1" ms.index="2">在 Visual Studio 中调试云服务或虚拟机</a>
            <div data-show-less-more-member="true">
              <p>了解如何通过 Visual Studio 对云服务中的问题进行远程诊断和调试。</p>
            </div>
          </li>
        </ul>
      </div>
    </div>
    <div>
      <div>
        <h3 data-jumpto-stop="true">监视</h3>
      </div>
      <div>
        <ul>
          <li><a href="http://www.asp.net/aspnet/overview/developing-apps-with-windows-azure/building-real-world-cloud-apps-with-windows-azure/monitoring-and-telemetry" ms.pgarea="content" ms.cmpgrp="body" ms.cmptyp="link list link" ms.cmpnm=" | 监视和遥测（用 Azure 构建实际云应用程序）" ms.title="" km.title="" ms.interactiontype="1" ms.index="0">监视和遥测（用 Azure 构建实际云应用程序）</a>
            <div data-show-less-more-member="true">
              <p>借助良好的遥测和日志记录系统，你可以随时了解应用程序使用情况，而当有应用程序出现故障时，你可以立即查明原因并能获取有帮助的故障排除信息。本章节介绍了如何选择并充分利用遥测系统。</p>
            </div>
          </li>
          <li><a href="http://msdn.microsoft.com/library/azure/dn306639.aspx" ms.pgarea="content" ms.cmpgrp="body" ms.cmptyp="link list link" ms.cmpnm=" | 了解 Azure 中的监视警告和通知" ms.title="" km.title="" ms.interactiontype="1" ms.index="1">了解 Azure 中的监视警告和通知</a>
            <div data-show-less-more-member="true">
              <p>了解如何理解并使用 Azure 管理门户中的内置警报和通知。</p>
            </div>
          </li>
          <li><a href="http://azure.microsoft.com/zh-cn/documentation/articles/cloud-services-how-to-monitor/" ms.pgarea="content" ms.cmpgrp="body" ms.cmptyp="link list link" ms.cmpnm=" | 如何使用 Azure Management Portal 来监视网站" ms.title="" km.title="" ms.interactiontype="1" ms.index="2">如何使用 Azure Management Portal 来监视网站</a>
            <div data-show-less-more-member="true">
              <p>一旦你的网站启动并运行后，你就可以监视其它的性能了。根据显示的内容，你可以配置站点以输出诊断日志来帮助你解决性能问题。你也可以查看特定使用率级别的配额使用量情况。</p>
            </div>
          </li>
          <li><a href="http://azure.microsoft.com/zh-cn/documentation/articles/store-new-relic-cloud-services-dotnet-application-performance-management/" ms.pgarea="content" ms.cmpgrp="body" ms.cmptyp="link list link" ms.cmpnm=" | 使用 New Relic 监视云服务" ms.title="" km.title="" ms.interactiontype="1" ms.index="3">使用 New Relic 监视云服务</a>
            <div data-show-less-more-member="true">
              <p>New Relic 是以开发人员为主的工具，用于监视应用程序，并使用户能够深入了解应用程序的性能和可靠性。本指南介绍如何向你的云服务中添加 New Relic 的一流性能监视。</p>
            </div>
          </li>
          <li><a href="http://azure.microsoft.com/zh-cn/documentation/articles/cloud-services-how-to-app-dynamics/" ms.pgarea="content" ms.cmpgrp="body" ms.cmptyp="link list link" ms.cmpnm=" | 使用 AppDynamics 监视云服务" ms.title="" km.title="" ms.interactiontype="1" ms.index="4">使用 AppDynamics 监视云服务</a>
            <div data-show-less-more-member="true">
              <p>AppDynamics 应用程序性能监视解决方案可帮助你确定生产环境中的问题、对此类问题进行故障排除并隔离根本原因。本指南介绍了如何开始对 Azure 使用 AppDynamics。</p>
            </div>
          </li>
        </ul>
      </div>
    </div>
  </div>
</div>
<div data-tab-panel-id="tab-panel">
  <div>
    <div>
      <h2>工具和自动化</h2>
      <div>
        <h3 data-jumpto-stop="true">工具</h3>
      </div>
      <div>
        <ul>
          <li><a href="http://go.microsoft.com/fwlink/p/?linkid=296710&clcid=0x804" ms.pgarea="content" ms.cmpgrp="body" ms.cmptyp="link list link" ms.cmpnm=" | .NET 的 Azure SDK 发行说明" ms.title="" km.title="" ms.interactiontype="1" ms.index="0">.NET 的 Azure SDK 发行说明</a>
            <div data-show-less-more-member="true">
              <p>了解 Azure SDK for .NET 中包含的最新更新。</p>
            </div>
          </li>
          <li><a href="http://msdn.microsoft.com/library/azure/ff687127.aspx" ms.pgarea="content" ms.cmpgrp="body" ms.cmptyp="link list link" ms.cmpnm=" | 适用于 Visual Studio 的 Azure 工具入门" ms.title="" km.title="" ms.interactiontype="1" ms.index="1">适用于 Visual Studio 的 Azure 工具入门</a>
            <div data-show-less-more-member="true">
              <p>了解如何使用 Azure Tools for Visual Studio 更高效地从 Visual Studio 开发、部署和调试 Azure 应用程序。</p>
            </div>
          </li>
        </ul>
      </div>
    </div>
    <div>
      <div>
        <h3 data-jumpto-stop="true">自动化</h3>
      </div>
      <div>
        <ul>
          <li><a href="http://azure.microsoft.com/zh-cn/documentation/articles/install-configure-powershell/" ms.pgarea="content" ms.cmpgrp="body" ms.cmptyp="link list link" ms.cmpnm=" | 安装和配置 Azure PowerShell" ms.title="" km.title="" ms.interactiontype="1" ms.index="0">安装和配置 Azure PowerShell</a>
            <div data-show-less-more-member="true">
              <p>Windows PowerShell for Azure 提供用于通过 Windows PowerShell cmdlet 开发和部署 Azure 应用程序的命令行环境。本节介绍如何使用 Windows PowerShell cmdlet 创建、部署和管理 Azure 资源。</p>
            </div>
          </li>
          <li><a href="http://msdn.microsoft.com/library/azure/dn642480.aspx" ms.pgarea="content" ms.cmpgrp="body" ms.cmptyp="link list link" ms.cmpnm=" | 从 Visual Studio 生成 PowerShell 脚本" ms.title="" km.title="" ms.interactiontype="1" ms.index="1">从 Visual Studio 生成 PowerShell 脚本</a>
            <div data-show-less-more-member="true">
              <p>在 Visual Studio 中创建 Web 应用程序时，你可生成 PowerShell 脚本，以供日后用于将应用程序自动发布到 Azure 网站或虚拟机。你可以在 Visual Studio 编辑器中编辑和扩展 PowerShell 脚本以适应你的要求，或将脚本与现有构建、测试和发布脚本相集成。</p>
            </div>
          </li>
          <li><a href="http://go.microsoft.com/fwlink/p/?linkid=327806&clcid=0x804" ms.pgarea="content" ms.cmpgrp="body" ms.cmptyp="link list link" ms.cmpnm=" | 使用 Azure Management Libraries for .NET" ms.title="" km.title="" ms.interactiontype="1" ms.index="2">使用 Azure Management Libraries for .NET</a>
            <div data-show-less-more-member="true">
              <p>了解如何通过简单的 .NET API 来管理云服务、虚拟机、虚拟网络、网站、存储帐户等。</p>
            </div>
          </li>
        </ul>
      </div>
    </div>
  </div>
</div>

