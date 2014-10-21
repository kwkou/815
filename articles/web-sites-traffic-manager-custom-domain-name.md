<properties title="Learn how to configure an Azure web site that uses Traffic Manager to use a custom domain name" pageTitle="Configure a custom domain name for an Azure web site that uses Traffic Manager" metaKeywords="Windows Azure, Windows Azure Web Sites, domain name" description="" services="web-sites" documentationCenter="" authors="larryfr, jroth" />

# 使用流量管理器为 Azure 网站配置自定义域名

<div class="dev-center-tutorial-selector sublanding"><a href="/en-us/documentation/articles/web-sites-custom-domain-name" title="自定义域" class="current">自定义域</a><a href="/en-us/documentation/articles/web-sites-godaddy-custom-domain-name" title="GoDaddy">GoDaddy</a><a href="/en-us/documentation/articles/web-sites-network-solutions-custom-domain-name" title="Network Solutions">Network Solutions</a><a href="/en-us/documentation/articles/web-sites-registerdotcom-custom-domain-name" title="Register.com">Register.com</a><a href="/en-us/documentation/articles/web-sites-enom-custom-domain-name" title="Enom">Enom</a><a href="/en-us/documentation/articles/web-sites-moniker-custom-domain-name" title="Moniker">Moniker</a><a href="/en-us/documentation/articles/web-sites-dotster-custom-domain-name" title="Dotster">Dotster</a><a href="/en-us/documentation/articles/web-sites-domaindiscover-custom-domain-name" title="DomainDiscover">DomainDiscover</a><a href="/en-us/documentation/articles/web-sites-directnic-custom-domain-name" title="Directnic">Directnic</a></div>

<div class="dev-center-tutorial-subselector"><a href="/en-us/documentation/articles/web-sites-custom-domain-name/" title="网站">网站</a> | <a href="/en-us/documentation/articles/web-sites-traffic-manager-custom-domain-name/" title="使用流量管理器的网站" class="current">使用流量管理器的网站</a></div>

[WACOM.INCLUDE [介绍][介绍]]

本文提供了将自定义域名与通过流量管理器进行负载平衡的 Azure 网站配合使用的一般说明。请查看本文顶端的选项卡上是否列出了你的域注册机构。如果有，请选择该选项卡以了解特定于注册机构的步骤。

[WACOM.INCLUDE [tmwebsitefooter][tmwebsitefooter]]

[WACOM.INCLUDE [introfooter][introfooter]]

本文内容：

-   [了解 DNS 记录][了解 DNS 记录]
-   [将网站配置为标准模式][将网站配置为标准模式]
-   [为自定义域添加 DNS 记录][为自定义域添加 DNS 记录]
-   [为网站启用流量管理器][为网站启用流量管理器]

## <a name="understanding-records"></a>了解 DNS 记录

[WACOM.INCLUDE [understandingdns][understandingdns]]

## <a name="bkmk_configsharedmode"></a>将网站配置为标准模式

[WACOM.INCLUDE [modes][modes]]

<a name="bkmk_configurecname"></a>

## 为自定义域添加 DNS 记录

</p>
要将自定义域与 Azure 网站关联，必须通过使用向你销售域名的域注册机构提供的工具在 DNS 表中为自定义域添加新条目。使用以下步骤找到并使用这些 DNS 工具。

1.  登录到你在域注册机构注册的帐户，然后查找用于管理 DNS 记录的页面。查找网站中标签为**“域名”**、**“DNS”**或**“名称服务器管理”**的链接或区域。通过查看你的帐户信息，然后找一个“我的域”这样的链接，通常可以找到访问此页面的链接。

2.  找到域名管理页后，再找允许你编辑 DNS 记录的链接。此链接可能是作为“区域文件”、“DNS 记录”或“高级”配置链接列出。

    -   该页面将很可能有几个已经创建的记录，例如将 <'**@**>' 或 '\*' 与“域驻留”页面关联的条目。它还可能包含用于常见子域（例如 **www**）的记录。
    -   该页将涉及“CNAME 记录”，或者提供用于选择记录类型的下拉列表。它还可能涉及其他记录，例如“A 记录”和“MX 记录”。在有些情况下，CNAME 记录将通过其他名称（例如“别名记录”）调用。
    -   该页还会有字段允许你从“主机名”或“域名”“映射”到另一域名。

3.  由于各个注册机构的具体情况不同，一般而言，你是*从* 自定义域名（例如 **contoso.com**）映射*到*用于 Azure 网站的流量管理器域名（**contoso.trafficmanager.cn**）。

4.  在注册机构添加或修改完 DNS 记录后，请保存这些更改。

## <a name="enabledomain"></a>启用流量管理器网站

[WACOM.INCLUDE [modes][1]]

  [自定义域]: /en-us/documentation/articles/web-sites-custom-domain-name "自定义域"
  [GoDaddy]: /en-us/documentation/articles/web-sites-godaddy-custom-domain-name "GoDaddy"
  [Network Solutions]: /en-us/documentation/articles/web-sites-network-solutions-custom-domain-name "Network Solutions"
  [Register.com]: /en-us/documentation/articles/web-sites-registerdotcom-custom-domain-name "Register.com"
  [Enom]: /en-us/documentation/articles/web-sites-enom-custom-domain-name "Enom"
  [Moniker]: /en-us/documentation/articles/web-sites-moniker-custom-domain-name "Moniker"
  [Dotster]: /en-us/documentation/articles/web-sites-dotster-custom-domain-name "Dotster"
  [DomainDiscover]: /en-us/documentation/articles/web-sites-domaindiscover-custom-domain-name "DomainDiscover"
  [Directnic]: /en-us/documentation/articles/web-sites-directnic-custom-domain-name "Directnic"
  [网站]: /en-us/documentation/articles/web-sites-custom-domain-name/ "网站"
  [使用流量管理器的网站]: /en-us/documentation/articles/web-sites-traffic-manager-custom-domain-name/ "使用流量管理器的网站"
  [介绍]: ../includes/custom-dns-web-site-intro-traffic-manager.md
  [tmwebsitefooter]: ../includes/custom-dns-web-site-traffic-manager-notes.md
  [introfooter]: ../includes/custom-dns-web-site-intro-notes.md
  [了解 DNS 记录]: #understanding-records
  [将网站配置为标准模式]: #bkmk_configsharedmode
  [为自定义域添加 DNS 记录]: #bkmk_configurecname
  [为网站启用流量管理器]: #enabledomain
  [understandingdns]: ../includes/custom-dns-web-site-understanding-dns-traffic-manager.md
  [modes]: ../includes/custom-dns-web-site-modes-traffic-manager.md
  [1]: ../includes/custom-dns-web-site-enable-on-traffic-manager.md
