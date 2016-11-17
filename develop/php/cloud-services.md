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
    wacn.date="11/17/2016"
    />


<h1 id="menu-php-cloud">云服务</h1>
<h2 id="header-0">常规</h2>
<h3>指南： <a href="/documentation/articles/fundamentals-application-models/">什么是 Azure 云服务？</a></h3>
<p>Azure 提供了三种用于运行应用程序的计算模型：Azure 网站、Azure 虚拟机和 Azure 云服务。本主题概述了这三种模型，以及可帮助您确定适用于自己应用程序的模型的信息。</p>
<h2 id="header-1">创建</h2>
<h3>功能指南： <a href="/documentation/articles/cloud-services-php-create-web-role/">创建 PHP Web 角色或辅助角色</a></h3>
<p>了解如何在 Azure 中使用 Azure PowerShell cmdlet 创建 PHP 云服务。云服务（包括 Web 角色和辅助角色）提供了平台即服务 (PaaS)。在云服务中，Web 角色提供专用的 Internet Information Services (IIS) Web 服务器来承载前端 Web 应用程序，而辅助角色可独立于用户交互或输入运行异步任务、运行时间较长的任务或永久性任务。</p>
<h2 id="header-2">配置</h2>
<h3>功能指南： <a href="/documentation/articles/cloud-services-how-to-configure/">配置云服务</a></h3>
<p>您可以在 Azure 管理门户中配置最常用的云服务设置。或者，如果您希望直接更新配置文件，则可以下载要更新的服务配置文件，然后上载更新文件并使用配置更改更新云服务。无论使用哪种方法，配置更新都将应用于所有角色实例。</p>
<h3>如何： <a href="/documentation/articles/cloud-services-configure-ssl-certificate/">在 Azure 中为应用程序配置 SSL</a></h3>
<p>安全套接字层 (SSL) 加密是用于保护通过 Internet 发送的数据的最常见方法。此常见任务讨论了如何为 Web 角色指定 HTTPS 终结点以及如何上载 SSL 证书来保护您的应用程序。</p>

<h2 id="header-3">管理</h2>
<h3>功能指南： <a href="/documentation/articles/cloud-services-how-to-scale/">扩展云服务</a></h3>
<p>可以通过添加或删除角色实例来缩放云服务。如果将 SQL 数据库 链接到云服务，则还可以缩放数据库。</p>
<h3>功能指南： <a href="/documentation/articles/cloud-services-how-to-manage/">管理云服务</a></h3>
<p>本指南介绍如何更新云服务部署，如何交换部署以将过渡环境中的部署升级到生产环境中的部署，如何将资源链接到云服务，以及如何删除云服务。</p>

