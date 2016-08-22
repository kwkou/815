<properties
	pageTitle="为 Azure 中使用流量管理器进行负载平衡的 Web 应用配置自定义域名"
    description="对 Azure 中包含流量管理器（用于负载平衡）的 Web 应用使用自定义域名。"
	services="app-service\web"
	documentationCenter=""
	authors="rmcmurray"
	manager="wpickett"
	editor=""/>  


<tags
	ms.service="app-service-web"
	ms.date="06/24/2016"
	wacn.date="08/22/2016"/>  


#使用流量管理器为 Azure 中的 Web 应用配置自定义域名

[AZURE.INCLUDE [web-selector](../../includes/websites-custom-domain-selector.md)]

[AZURE.INCLUDE [intro](../../includes/custom-dns-web-site-intro-traffic-manager.md)]

本文提供了将自定义域名与通过流量管理器进行负载平衡的 Azure 配合使用的一般说明。

[AZURE.INCLUDE [tmwebsitefooter](../../includes/custom-dns-web-site-traffic-manager-notes.md)]

[AZURE.INCLUDE [introfooter](../../includes/custom-dns-web-site-intro-notes.md)]

<a name="understanding-records"></a>
## 了解 DNS 记录

[AZURE.INCLUDE [understandingdns](../../includes/custom-dns-web-site-understanding-dns-traffic-manager.md)]

<a name="bkmk_configsharedmode"></a>
## 将 Web 应用配置为标准模式

[AZURE.INCLUDE [模式](../../includes/custom-dns-web-site-modes-traffic-manager.md)]

<a name="bkmk_configurecname"></a>
## 为自定义域添加 DNS 记录


要将自定义域与 Azure 中的 Web 应用关联，必须通过使用向你销售域名的域注册机构提供的工具在 DNS 表中为自定义域添加新条目。使用以下步骤找到并使用这些 DNS 工具。

1. 登录到你在域注册机构注册的帐户，然后查找用于管理 DNS 记录的页面。查找标为**域名**、**DNS** 或**名称服务器管理**的站点链接或区域。通过查看你的帐户信息，然后找一个“我的域”这样的链接，通常可以找到访问此页面的链接。

4. 找到域名管理页后，再找允许编辑 DNS 记录的链接。此链接可能作为“区域文件”、“DNS 记录”或“高级”配置链接列出。

	* 该页面将很可能有几个已经创建的记录，例如将 '**@**' 或 '*' 与“域驻留”页面关联的条目。它还可能包含用于常见子域（例如 **www**）的记录。
	* 该页将涉及 **CNAME 记录**，或者提供用于选择记录类型的下拉列表。它还可能涉及其他记录，例如 **A 记录**和 **MX 记录**。在有些情况下，CNAME 记录将通过其他名称（例如**别名记录**）调用。
	* 该页还会有字段允许你从**主机名**或**域名****映射**到另一域名。

5. 由于各个注册机构的具体情况不同，一般而言，你是从 自定义域名（例如 **contoso.com**）映射到用于 Web 应用的流量管理器域名 (**contoso.trafficmanager.cn**)。

> [AZURE.NOTE] 或者，如果某条记录已被使用并且你需要提前将应用绑定到该记录，请将 **awverify.contoso.com** 映射到 **contoso.trafficmanager.cn**。

6. 在注册机构添加或修改完 DNS 记录后，请保存这些更改。

<a name="enabledomain"></a>
## 启用流量管理器

[AZURE.INCLUDE [模式](../../includes/custom-dns-web-site-enable-on-traffic-manager.md)]

<!---HONumber=Mooncake_0815_2016-->