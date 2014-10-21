<properties title="Learn how to configure an Azure web site to use a domain name registered with DomainDiscover - TierraNet" pageTitle="Configure a DomainDiscover domain name for an Azure web site" metaKeywords="Azure, Azure Web Sites, DomainDiscover, TierraNet" description="Learn how to configure an Azure web site to use a domain name registered with DomainDiscover - TierraNet" services="web-sites" documentationCenter="" authors="larryfr,jroth" />

# 为 Azure 网站配置自定义域名 (DomainDiscover / TierraNet)

<div class="dev-center-tutorial-selector sublanding"><a href="/en-us/documentation/articles/web-sites-custom-domain-name" title="自定义域">自定义域</a><a href="/en-us/documentation/articles/web-sites-godaddy-custom-domain-name" title="GoDaddy">GoDaddy</a><a href="/en-us/documentation/articles/web-sites-network-solutions-custom-domain-name" title="Network Solutions">Network Solutions</a><a href="/en-us/documentation/articles/web-sites-registerdotcom-custom-domain-name" title="Register.com">Register.com</a><a href="/en-us/documentation/articles/web-sites-enom-custom-domain-name" title="Enom">Enom</a><a href="/en-us/documentation/articles/web-sites-moniker-custom-domain-name" title="Moniker">Moniker</a><a href="/en-us/documentation/articles/web-sites-dotster-custom-domain-name" title="Dotster">Dotster</a><a href="/en-us/documentation/articles/web-sites-domaindiscover-custom-domain-name" title="DomainDiscover" class="current">DomainDiscover</a><a href="/en-us/documentation/articles/web-sites-directnic-custom-domain-name" title="Directnic">Directnic</a></div>

<div class="dev-center-tutorial-subselector"><a href="/en-us/documentation/articles/web-sites-domaindiscover-custom-domain-name/" title="网站" class="current">网站</a> | <a href="/en-us/documentation/articles/web-sites-domaindiscover-traffic-manager-custom-domain-name/" title="使用流量管理器的网站">使用流量管理器的网站</a></div>

[WACOM.INCLUDE [介绍][介绍]]

本文提供了如何将从 [DomainDiscover.com][DomainDiscover.com] 购买的自定义域名用于 Azure 网站的说明。DomainDiscover.com 是 TierraNet 的组成部分。

[WACOM.INCLUDE [introfooter][introfooter]]

本文内容：

-   [了解 DNS 记录][了解 DNS 记录]
-   [为基本、共享或标准模式配置网站][为基本、共享或标准模式配置网站]
-   [为自定义域添加 DNS 记录][为自定义域添加 DNS 记录]
-   [在网站上启用域][在网站上启用域]

## <a name="understanding-records"></a>了解 DNS 记录

[WACOM.INCLUDE [understandingdns][understandingdns]]

## <a name="bkmk_configsharedmode"></a>为基本、共享或标准模式配置网站

[WACOM.INCLUDE [modes][modes]]

<a name="bkmk_configurecname"></a>

## 为自定义域添加 DNS 记录

</p>
若要将自定义域与 Azure 网站关联，必须使用 DomainDiscover 提供的工具在 DNS 表中为自定义域添加新条目。使用以下步骤找到用于 DomainDiscover.com 的 DNS 工具

1.  从“登录”菜单中选择“控制面板”，通过 DomainDiscover.com (TierraNet) 登录到你的帐户。

    ![DomainDiscover 登录菜单][DomainDiscover 登录菜单]

2.  在“域服务”页面上，选择要用于你的 Azure 网站的域。

    ![域管理页面][域管理页面]

3.  在域设置中，单击“DNS 服务”的“编辑”按钮。

    ![DNS 编辑按钮][DNS 编辑按钮]

4.  在“管理 DNS”窗口中，选择要添加到“添加记录”列表中的 DNS 记录的类型。然后单击“添加”按钮。

    ![DNS 编辑按钮][1]

5.  在下面的页面上，输入 DNS 记录值。然后单击“添加”按钮。

    ![DNS 编辑按钮][2]

    -   在添加 CNAME 记录时，必须先在“管理 DNS”页面上选择**“CNAME (别名)”**。然后将“主机”字段设置为你要使用的子域。例如，**www**。必须将“主机别名”字段设置为你的 Azure 网站的 **.chinacloudsites.cn** 域名。例如 **contoso.chinacloudsites.cn**。然后提供生存时间 (TTL) 值，如 1800 秒。

    -   在添加 A 记录时，必须先在“管理 DNS”页面上选择 **A**。然后将“主机”字段设置为 <**@*>\*（代表根域名，如 **contoso.com**）或者是你要使用的子域（例如 **www**）。必须将“IP 地址”字段设置为你的 Azure 网站的 IP 地址。然后提供生存时间 (TTL) 值，如 1800 秒。

        > [WACOM.NOTE] 如果你要使用 A 记录，则还必须使用以下配置之一添加 CNAME 记录：
        >
        > -   “主机”值 **www** 以及“主机别名”值 **\<yourwebsitename\>.chinacloudsites.cn**。
        >
        > 或者
        >
        > -   “主机”值 **awverify.www** 以及“主机别名”值 **awverify.\<yourwebsitename\>.chinacloudsites.cn**。
        >
        > 此 CNAME 记录由 Azure 用来验证你拥有 A 记录所描述的域

6.  如果添加了 A 记录，可能会出现警告信息，指出你的域的现有 A 记录不是不活动状态。它使用最近更改的记录，并且 DomainDirect 已经有一个用于根域名的默认 A 记录。你可以依靠此优先性，也可以通过选择“删除”按钮移除默认 A 记录。

## <a name="enabledomain"></a>在网站上启用域名

[WACOM.INCLUDE [modes][3]]

  [自定义域]: /en-us/documentation/articles/web-sites-custom-domain-name "自定义域"
  [GoDaddy]: /en-us/documentation/articles/web-sites-godaddy-custom-domain-name "GoDaddy"
  [Network Solutions]: /en-us/documentation/articles/web-sites-network-solutions-custom-domain-name "Network Solutions"
  [Register.com]: /en-us/documentation/articles/web-sites-registerdotcom-custom-domain-name "Register.com"
  [Enom]: /en-us/documentation/articles/web-sites-enom-custom-domain-name "Enom"
  [Moniker]: /en-us/documentation/articles/web-sites-moniker-custom-domain-name "Moniker"
  [Dotster]: /en-us/documentation/articles/web-sites-dotster-custom-domain-name "Dotster"
  [DomainDiscover]: /en-us/documentation/articles/web-sites-domaindiscover-custom-domain-name "DomainDiscover"
  [Directnic]: /en-us/documentation/articles/web-sites-directnic-custom-domain-name "Directnic"
  [网站]: /en-us/documentation/articles/web-sites-domaindiscover-custom-domain-name/ "网站"
  [使用流量管理器的网站]: /en-us/documentation/articles/web-sites-domaindiscover-traffic-manager-custom-domain-name/ "使用流量管理器的网站"
  [介绍]: ../includes/custom-dns-web-site-intro.md
  [DomainDiscover.com]: https://domaindiscover.com
  [introfooter]: ../includes/custom-dns-web-site-intro-notes.md
  [了解 DNS 记录]: #understanding-records
  [为基本、共享或标准模式配置网站]: #bkmk_configsharedmode
  [为自定义域添加 DNS 记录]: #bkmk_configurecname
  [在网站上启用域]: #enabledomain
  [understandingdns]: ../includes/custom-dns-web-site-understanding-dns-raw.md
  [modes]: ../includes/custom-dns-web-site-modes.md
  [DomainDiscover 登录菜单]: .\media\web-sites-domaindiscover-custom-domain-name\DomainDiscover_LoginMenu.png
  [域管理页面]: .\media\web-sites-domaindiscover-custom-domain-name\DomainDiscover_DomainManagement.png
  [DNS 编辑按钮]: .\media\web-sites-domaindiscover-custom-domain-name\DomainDiscover_DNSEditButton.png
  [1]: .\media\web-sites-domaindiscover-custom-domain-name\DomainDiscover_DNSAddRecords.png
  [2]: .\media\web-sites-domaindiscover-custom-domain-name\DomainDiscover_DNSRecords.png
  [3]: ../includes/custom-dns-web-site-enable-on-web-site.md
