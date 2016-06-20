<properties 
  pageTitle="应用服务 - Azure 微软云"
  metakeywords="" 
  description="" 
  services="" 
  documentationCenter="java-app-services" 
  authors="" 
  manager="Tiffena" 
  editor="EricChen"/>
<tags ms.service=""
    ms.date="10/23/2014"
    wacn.date="04/11/2015"
    />


<h1 id="menu-java-appservices">应用程序服务</h1>
<h2 id="header-0">证书管理</h2>
<h3>如何： <a href="/documentation/articles/java-add-certificate-ca-store/">将证书添加到 Java CA 证书存储</a></h3>
<p>了解如何将证书颁发机构 (CA) 证书添加到 Java CA 证书 (cacerts) 存储。本文提供了 Twilio 服务所需的 CA 证书以及 Azure Service Bus 的 CA 证书的特定示例。</p>
<h2 id="header-1">CDN</h2>
<h3>功能指南： <a href="/documentation/articles/cdn-how-to-use/">使用 CDN 提供高带宽内容</a></h3>
<p>通过在世界各地的物理节点缓存 blob 和静态内容，Azure 内容交付网络 (CDN) 为交付高带宽内容提供了一个全局解决方案。本文描述如何启用 CDN 以及添加、访问和删除内容。</p>
<!--
<h2 id="header-2">电子邮件和音频</h2>
<h3>功能指南： <a href="/documentation/articles/store-sendgrid-java-how-to-send-email/">SendGrid 电子邮件服务</a></h3>
<p>Azure 应用程序可以使用 SendGrid 来包括电子邮件功能。SendGrid 提供了可靠的电子邮件传递服务、实时分析和灵活的 API，使用户能够轻松地将服务合并到他们的 Azure 应用程序中。</p>
<h3>教程： <a href="/documentation/articles/store-sendgrid-java-how-to-send-email/" ms.pgarea="content" ms.cmpgrp="body" ms.cmptyp="link" ms.cmpnm="在 Azure 部署中通过 Java 使用 SendGrid 发送电子邮件" ms.title="" km.title="" ms.interactiontype="1">在 Azure 部署中通过 Java 使用 SendGrid 发送电子邮件</a></h3>
<p>本教程演示如何使用 SendGrid 通过 Azure 中承载的网页发送电子邮件。</p>
<h3>功能指南： <a href="/documentation/articles/partner-twilio-java-how-to-use-voice-sms/" ms.pgarea="content" ms.cmpgrp="body" ms.cmptyp="link" ms.cmpnm="Twilio 音频和 SMS 服务" ms.title="" km.title="" ms.interactiontype="1">Twilio 音频和 SMS 服务</a></h3>
<p>Azure 应用程序可以通过 Twilio 合并电话和短信服务 (SMS) 消息功能。可使用 Twilio API 拨打和接听电话，收发短信，以及通过现有互联网连接（包括移动连接）进行语音通信。</p>-->
<h2 id="header-3">标识</h2>
<h3>功能指南： <a href="/documentation/articles/active-directory-java-authenticate-users-access-control-eclipse/">访问控制</a></h3>
<p>Access Control 是一项 Azure 服务，可用于轻松对需要访问您的 Web 应用程序和服务的用户进行身份验证，而不必在代码中添加复杂的身份验证逻辑因素。该服务还提供与 Windows Identity Foundation (WIF) 的集成，支持常用 Web 身份提供程序（包括 Windows Live ID、Yahoo），并且支持 ADFS 2.0。</p>
<h3>如何： <a href="/documentation/articles/active-directory-java-view-saml-returned-by-access-control/">查看 Azure Access Control 服务返回的 SAML</a></h3>
<p>当用户成功登录时，Access Control 服务会向您的应用程序返回安全声明标记语言 (SAML)。</p>
<!--<h2 id="header-4">图像处理</h2>
<h3>功能指南： <a href="/documentation/articles/store-blitline-how-to-use/">Blitline 图像处理服务</a></h3>
<p>Blitline 是一项基于云计算的图像处理服务。本指南介绍如何访问 Blitline 服务以及如何将作业提交到 Blitline。</p>-->
<h2 id="header-5">消息传送和集成</h2>
<h3>功能指南： <a href="/documentation/articles/service-bus-java-how-to-use-queues/">Service Bus 队列</a></h3>
<p>Service Bus 队列提供简单且有保证的先入先出消息传送策略，支持一系列标准协议（REST、AMQP 和 WS*）以及用于将消息放入/推出队列的 API。</p>
<h3>功能指南： <a href="/documentation/articles/service-bus-java-how-to-use-topics-subscriptions/">Service Bus 主题</a></h3>
<p>Service Bus 主题提供发布/订阅消息传送模型，以支持一对多通信。您可以选择以每个订阅为基础注册主题的筛选规则，这样就能限制哪些主题订阅接收某个主题下的哪些消息。</p>
<h3>功能指南： <a href="/documentation/articles/storage-java-how-to-use-queue-storage/">Azure 队列服务</a></h3>
<p>Azure 队列可存储大量消息，用户可以通过经验证的呼叫，使用 HTTP 或 HTTPS 从世界任何地方访问这些消息。队列存储的常见用途包括：创建积压工作以进行异步处理，从 Azure Web 角色向 Azure 辅助角色传递消息。</p>
<h3>指南： <a href="/documentation/articles/service-bus-java-amqp-overview/">Azure Service Bus 中的 AMQP 1.0 支持</a></h3>
<p>高级消息队列协议 (AMQP) 1.0 是一个高效、可靠的线级消息传送协议，可用于构建强大、跨平台的消息传送应用程序。本文概述了 Service Bus 中的 AMQP 1.0 支持，该支持服务允许您使用开放标准协议构建跨平台的混合应用程序。</p>
<h3>功能指南： <a href="/documentation/articles/service-bus-java-amqp-overview/">Service Bus AMQP</a></h3>
<p>此操作方法指南说明如何通过 .NET API 从 AMQP 1.0 使用 Service Bus 中转消息传递功能（队列和发布/订阅主题）。</p>
