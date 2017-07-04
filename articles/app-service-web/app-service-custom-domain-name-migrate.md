<properties
    pageTitle="将活动的自定义域迁移到 Azure 应用服务 | Azure"
    description="了解如何在不停机的情况下，将已分配给实时站点的自定义域迁移到 Azure 应用服务中的应用。"
    services="app-service"
    documentationcenter=""
    author="cephalin"
    manager="wpickett"
    editor="jimbe"
    tags="top-support-issue" />
<tags
    ms.assetid="10da5b8a-1823-41a3-a2ff-a0717c2b5c2d"
    ms.service="app-service"
    ms.workload="na"
    ms.tgt_pltfrm="na"
    ms.devlang="na"
    ms.topic="article"
    ms.date="01/30/2017"
    wacn.date="03/03/2017"
    ms.author="cephalin" />  


# 将活动的自定义域迁移到 Azure 应用服务

本文介绍如何在不停机的情况下，将活动的自定义域迁移到 [Azure 应用服务](/documentation/articles/app-service-value-prop-what-is/)。

将实时站点及其域名迁移到应用服务时，该域名已在进行实时通信，而你不希望 DNS 解析在迁移过程中出现任何停机。在这种情况下，请提前将域名绑定到用于域验证的 Azure 应用。

## 先决条件

本文假定用户已了解如何[手动将自定义域映射到应用服务](/documentation/articles/web-sites-custom-domain-name/)。

## 提前绑定域名

提前绑定自定义域时，请先完成下面的两项操作，然后再对 DNS 记录进行更改：

- 验证域所有权
- 为应用启用域名

最后将 DNS 记录更改为指向应用服务应用时，客户端会从旧站点重定向到应用服务应用，不会造成 DNS 解析暂停。

请遵循以下步骤操作：

1. 首先，请执行[创建 DNS 记录](/documentation/articles/web-sites-custom-domain-name/#createdns)中的步骤，通过 DNS 注册表创建验证 TXT 记录。其他 TXT 记录遵循从 &lt;*子域*>.&lt;*根域*> 映射到 &lt;*应用名称*>.chinacloudsites.cn 的约定。有关示例，请参见下表：
 
    <table cellspacing="0" border="1">
    <tr>
    <th>FQDN 示例</th>
    <th>TXT 主机</th>
    <th>TXT 值</th>
    </tr>
    <tr>
    <td>contoso.com (root)</td>
    <td>awverify.contoso.com</td>
    <td>&lt;<i>appname</i>>.chinacloudsites.cn</td>
    </tr>
    <tr>
    <td>www.contoso.com (sub)</td>
    <td>awverify.www.contoso.com</td>
    <td>&lt;<i>appname</i>>.chinacloudsites.cn</td>
    </tr>
    <tr>
    <td>\*.contoso.com (wildcard)</td>
    <td>awverify.\*.contoso.com</td>
    <td>&lt;<i>appname</i>>.chinacloudsites.cn</td>
    </tr>
    </table>

2. 然后，请执行[为应用启用自定义域名](/documentation/articles/web-sites-custom-domain-name/#enable)中的步骤，将自定义域名添加到 Azure 应用。

    自定义域现已在 Azure 应用中启用。只需通过域注册机构更新 DNS 记录。

3. 最后，更新域的 DNS 记录，使之指向 Azure 应用，如[创建 DNS 记录](/documentation/articles/web-sites-custom-domain-name/#createdns)中所示。

    DNS 传播开始后，应立即将用户流量重定向至 Azure 应用。

## 后续步骤
请参阅 [using an SSL certificate from elsewhere](/documentation/articles/web-sites-configure-ssl-certificate/)（使用在其他地方购买的 SSL 证书），了解如何使用 HTTPS 保护自定义域名。

<!---HONumber=Mooncake_0227_2017-->