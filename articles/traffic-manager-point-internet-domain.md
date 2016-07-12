<properties
   pageTitle="将公司 Internet 域指向流量管理器域 | Azure"
   description="本文将帮助你将公司域名指向流量管理器域名。"
   services="traffic-manager"
   documentationCenter=""
   authors="joaoma"
   manager="carmonm"
   editor="tysonn" />
<tags
	ms.service="traffic-manager"
	ms.date="03/17/2016"
	wacn.date="04/26/2016"/>

# 将公司 Internet 域指向 Azure 流量管理器域

若要将公司域名指向流量管理器域名，请修改你的 Internet DNS 服务器上的 DNS 资源记录来使用 CNAME 记录类型，该记录类型将公司域名映射到流量管理器配置文件的域名。可以在流量管理器配置文件的“配置”页上的“常规”部分中查看流量管理器域名。

例如，若要将公司域名 www.contoso.com 指向流量管理器域名 contoso.trafficmanager.cn，需要将你的 DNS 资源记录更新为以下内容：

    www.contoso.com IN CNAME contoso.trafficmanager.cn

现在，对 *www.contoso.com* 的所有流量请求都将定向到 *contoso.trafficmanager.cn*。

>[AZURE.IMPORTANT]无法将第二级域（例如 *contoso.com*）指向流量管理器域。这是 DNS 协议的一个限制，它不允许第二级域名存在 CNAME 记录。

## 后续步骤

[流量管理器路由方法](/documentation/articles/traffic-manager-routing-methods/)

[流量管理器 - 禁用、启用或删除配置文件](/documentation/articles/disable-enable-or-delete-a-profile/)

[流量管理器 - 禁用或启用终结点](/documentation/articles/disable-or-enable-an-endpoint/)

<!---HONumber=Mooncake_1221_2015-->