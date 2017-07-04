<properties 
	pageTitle="PHP 调用认知服务证书认证问题" 
	description="PHP 调用认知服务证书认证问题" 
	services="cognitive-services" 
	documentationCenter="" 
	authors=""
	manager="" 
	editor=""/>
<tags 
	ms.service="cognitive-services-aog"
	ms.date="" 
	wacn.date="12/05/2016"/>
# PHP 调用认知服务证书认证问题 #

### 问题描述 ###

在使用 PHP 的 http 客户端工具（如 Guzzle）调用认知服务时出现证书认证问题，错误信息如下：

	Fatal error: Uncaught exception 'GuzzleHttp\Exception\RequestException' with message 'cURL error 60: SSL certificate problem: unable to get local issuer certificate

### 解决方法 ###

该问题的出现是由于没有配置信任的服务器 HTTPS 验证。默认 cURL 是被设为不信任任何 CAs 的，因此这就是无法访问相关服务器的原因。解决该问题的最好方法是指定一组默认受信任的 CAs。 

- CA 证书

	数字证书认证机构（Certificate Authority， 缩写为 CA），是负责发放和管理数字证书的权威机构，它要制定政策和具体步骤来验证、识别用户身份，并对用户证书进行签名，以确保证书持有者的身份和公钥的拥有权。证书通常包括所有者的公钥、 证书的到期时间、所有者的名称和其他关于所有者的信息。

所以可以通过下载证书的方式解决该问题，具体操作方法如下：

1.	下载证书保存到本地，下载地址：[https://curl.haxx.se/ca/cacert.pem](https://curl.haxx.se/ca/cacert.pem)
2.	配置 php.ini 文件：`curl.cainfo =<filepath>/cacert.pem`
3.	重启 Apache 服务器，问题即可解决。