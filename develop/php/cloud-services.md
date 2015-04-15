<properties 
  pageTitle="Php-云服务 - Azure 微软云"
  metakeywords="" 
  description="" 
  services="" 
  documentationCenter="php" 
  authors="" 
  manager="Tiffena" 
  editor="EricChen"/>
<tags ms.service=""
    ms.date=""
    wacn.date="04/11/2015"
    />


<h1 id="menu-php-cloud">云服务</h1>
<h2 id="header-0">常规</h2>
<h3>指南： <a href="/documentation/articles/fundamentals-application-models/#CloudServices">什么是 Azure 云服务？</a></h3>
<p>Azure 提供了三种用于运行应用程序的计算模型：Azure 网站、Azure 虚拟机和 Azure 云服务。本主题概述了这三种模型，以及可帮助您确定适用于自己应用程序的模型的信息。</p>
<h2 id="header-1">创建</h2>
<h3>功能指南： <a href="/documentation/articles/cloud-services-php-create-web-role/">创建 PHP Web 角色或辅助角色</a></h3>
<p>了解如何在 Azure 中使用 Azure PowerShell cmdlet 创建 PHP 云服务。云服务（包括 Web 角色和辅助角色）提供了平台即服务 (PaaS)。在云服务中，Web 角色提供专用的 Internet Information Services (IIS) Web 服务器来承载前端 Web 应用程序，而辅助角色可独立于用户交互或输入运行异步任务、运行时间较长的任务或永久性任务。</p>
<h2 id="header-2">配置</h2>
<h3>功能指南： <a href="/documentation/articles/cloud-services-how-to-configure/">配置云服务</a></h3>
<p>您可以在 Azure 管理门户中配置最常用的云服务设置。或者，如果您希望直接更新配置文件，则可以下载要更新的服务配置文件，然后上载更新文件并使用配置更改更新云服务。无论使用哪种方法，配置更新都将应用于所有角色实例。</p>
<h3>如何： <a href="/documentation/articles/cloud-services-configure-ssl-certificate/">在 Azure 中为应用程序配置 SSL</a></h3>
<p>安全套接字层 (SSL) 加密是用于保护通过 Internet 发送的数据的最常见方法。此常见任务讨论了如何为 Web 角色指定 HTTPS 终结点以及如何上载 SSL 证书来保护您的应用程序。</p>
<h3>如何： <a href="/develop/php/common-tasks/enable-remote-desktop/">在 Azure 中启用远程桌面</a></h3>
<p>您可以通过远程桌面访问在 Azure 中运行的角色实例的桌面。您可以使用远程桌面连接配置虚拟机，或者排查应用程序问题。</p>
<h2 id="header-3">管理</h2>
<h3>功能指南： <a href="/documentation/articles/cloud-services-how-to-scale/">扩展云服务</a></h3>
<p>可以通过添加或删除角色实例来缩放云服务。如果将 SQL 数据库 链接到云服务，则还可以缩放数据库。</p>
<h3>功能指南： <a href="/documentation/articles/cloud-services-how-to-manage/">管理云服务</a></h3>
<p>本指南介绍如何更新云服务部署，如何交换部署以将过渡环境中的部署升级到生产环境中的部署，如何将资源链接到云服务，以及如何删除云服务。</p>
<h3>功能指南： <a href="/documentation/articles/cloud-services-php-how-to-use-service-management/">服务管理</a></h3>
<p>服务管理 API 是一个 REST API，可通过它以编程方式访问通过 Azure 管理门户获得的许多服务管理功能。所有 API 操作都是通过 SSL 执行的，并且使用 X.509 v3 证书互相进行身份验证。可以从在 Azure 中运行的服务访问管理服务，或直接通过 Internet 从可发送 HTTPS 请求和接收 HTTPS 响应的任意应用程序访问管理服务。</p>
