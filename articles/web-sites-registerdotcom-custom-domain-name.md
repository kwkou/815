<properties title="Learn how to configure an Azure web site to use a domain name registered with Register.com" pageTitle="Configure a Register.com domain name for an Azure web site" metaKeywords="Azure, Azure Web Sites, domain name" description="" services="web-sites" documentationCenter="" authors="larryfr, jroth" />

# 为 Azure 网站配置自定义域名 (Register.com)

<div class="dev-center-tutorial-selector sublanding"><a href="/zh-cn/documentation/articles/web-sites-custom-domain-name" title="自定义域">自定义域</a><a href="/zh-cn/documentation/articles/web-sites-godaddy-custom-domain-name" title="GoDaddy">GoDaddy</a><a href="/zh-cn/documentation/articles/web-sites-network-solutions-custom-domain-name" title="Network Solutions">Network Solutions</a><a href="/zh-cn/documentation/articles/web-sites-registerdotcom-custom-domain-name" title="Register.com" class="current">Register.com</a><a href="/zh-cn/documentation/articles/web-sites-enom-custom-domain-name" title="Enom">Enom</a><a href="/zh-cn/documentation/articles/web-sites-moniker-custom-domain-name" title="Moniker">Moniker</a><a href="/zh-cn/documentation/articles/web-sites-dotster-custom-domain-name" title="Dotster">Dotster</a><a href="/zh-cn/documentation/articles/web-sites-domaindiscover-custom-domain-name" title="DomainDiscover">DomainDiscover</a><a href="/zh-cn/documentation/articles/web-sites-directnic-custom-domain-name" title="Directnic">Directnic</a></div>

<div class="dev-center-tutorial-subselector"><a href="/zh-cn/documentation/articles/web-sites-registerdotcom-custom-domain-name/" title="网站" class="current">网站</a> | <a href="/zh-cn/documentation/articles/web-sites-registerdotcom-traffic-manager-custom-domain-name/" title="使用流量管理器的网站">使用流量管理器的网站</a></div>

[WACOM.INCLUDE [介绍][介绍]]

本文提供了如何将从 [Register.com][1] 购买的自定义域名用于 Azure 网站的说明。

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
要将自定义域与 Azure 网站关联，必须通过使用由 Register.com 提供的工具在 DNS 表中为自定义域添加新条目。请使用以下步骤找到并使用这些 DNS 工具。

1.  在 register.com 上登录到你的帐户，然后在右上角选择“你的帐户”以查看你的域，然后选择你的自定义域名。

    ![我的帐户页面][我的帐户页面]

2.  向下滚动页面直到显示“高级技术设置”。这部分中的链接可用于管理你的域的记录。

    -   对于 A 记录，请使用“编辑 IP 地址记录”链接。
    -   对于 CNAME 记录，请使用“编辑域别名记录”链接。

    ![高级技术设置][高级技术设置]

3.  当你单击“编辑”按钮时，你将看到一个窗体，可用于修改现有记录，或添加新记录。对于 CNAME 和 A 两种记录，窗体都是类似的。

    -   添加 CNAME 记录时，必须将“.mydomainname.com”字段设置为你要使用的子域。例如，**www**。必须将“指向”值选为你的 Azure 网站的 **.chinacloudsites.cn** 域名。例如 **contoso.chinacloudsites.cn**。将“引用主机名称”保留为“选择”，因为在创建用于 Azure 网站的 CNAME 记录时此字段不是必填字段。

        ![cname 窗体][cname 窗体]

        > [WACOM.NOTE] 如果你要使用 A 记录，则还必须使用以下配置之一添加 CNAME 记录：
        >
        > -   “别名”值 **www** 以及“其他主机”值 **\<yourwebsitename\>.chinacloudsites.cn**。
        >
        > 或者
        >
        > -   “别名”值 **awverify.www** 以及“其他主机”值 **awverify.\<yourwebsitename\>.chinacloudsites.cn**。
        >
        > 此 CNAME 记录由 Azure 用来验证你拥有 A 记录所描述的域。

    -   添加 A 记录时，必须将“.mydomainname.com”字段设置为你要使用的子域（例如 **www**）。该字段保留为空，以设置根域，或使用 \*\* 来创建通配符映射。**必须将“指向”字段设置为你的 Azure 网站的 IP 地址。**

        ![记录窗体][记录窗体]

4.  完成添加或修改记录之后，请单击“继续”，以查看这些更改。再次选择**“继续”**，以保存这些更改。

## <a name="enabledomain"></a>在网站上启用域名

[WACOM.INCLUDE [modes][2]]

  [自定义域]: /zh-cn/documentation/articles/web-sites-custom-domain-name "自定义域"
  [GoDaddy]: /zh-cn/documentation/articles/web-sites-godaddy-custom-domain-name "GoDaddy"
  [Network Solutions]: /zh-cn/documentation/articles/web-sites-network-solutions-custom-domain-name "Network Solutions"
  [Register.com]: /zh-cn/documentation/articles/web-sites-registerdotcom-custom-domain-name "Register.com"
  [Enom]: /zh-cn/documentation/articles/web-sites-enom-custom-domain-name "Enom"
  [Moniker]: /zh-cn/documentation/articles/web-sites-moniker-custom-domain-name "Moniker"
  [Dotster]: /zh-cn/documentation/articles/web-sites-dotster-custom-domain-name "Dotster"
  [DomainDiscover]: /zh-cn/documentation/articles/web-sites-domaindiscover-custom-domain-name "DomainDiscover"
  [Directnic]: /zh-cn/documentation/articles/web-sites-directnic-custom-domain-name "Directnic"
  [网站]: /zh-cn/documentation/articles/web-sites-registerdotcom-custom-domain-name/ "网站"
  [使用流量管理器的网站]: /zh-cn/documentation/articles/web-sites-registerdotcom-traffic-manager-custom-domain-name/ "使用流量管理器的网站"
  [介绍]: ../includes/custom-dns-web-site-intro.md
  [1]: https://register.com
  [introfooter]: ../includes/custom-dns-web-site-intro-notes.md
  [了解 DNS 记录]: #understanding-records
  [为基本、共享或标准模式配置网站]: #bkmk_configsharedmode
  [为自定义域添加 DNS 记录]: #bkmk_configurecname
  [在网站上启用域]: #enabledomain
  [understandingdns]: ../includes/custom-dns-web-site-understanding-dns-raw.md
  [modes]: ../includes/custom-dns-web-site-modes.md
  [我的帐户页面]: ./media/web-sites-custom-domain-name/rdotcom-myaccount.png
  [高级技术设置]: ./media/web-sites-custom-domain-name/rdotcom-advancedsettings.png
  [cname 窗体]: ./media/web-sites-custom-domain-name/rdotcom-editcnamerecord.png
  [记录窗体]: ./media/web-sites-custom-domain-name/rdotcom-editarecord.png
  [2]: ../includes/custom-dns-web-site-enable-on-web-site.md
