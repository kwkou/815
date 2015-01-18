<properties title="Learn how to configure an Azure web site to use a domain name registered with eNom" pageTitle="Configure an eNom domain name for an Azure web site" metaKeywords="Windows Azure, Windows Azure Web Sites, domain name" description="" services="web-sites" documentationCenter="" authors="larryfr, jroth" />

# 为 Azure 网站配置自定义域名 (eNom)

<div class="dev-center-tutorial-selector sublanding"><a href="/zh-cn/documentation/articles/web-sites-custom-domain-name" title="自定义域">自定义域</a><a href="/zh-cn/documentation/articles/web-sites-godaddy-custom-domain-name" title="GoDaddy">GoDaddy</a><a href="/zh-cn/documentation/articles/web-sites-network-solutions-custom-domain-name" title="Network Solutions">Network Solutions</a><a href="/zh-cn/documentation/articles/web-sites-registerdotcom-custom-domain-name" title="Register.com">Register.com</a><a href="/zh-cn/documentation/articles/web-sites-enom-custom-domain-name" title="Enom" class="current">Enom</a><a href="/zh-cn/documentation/articles/web-sites-moniker-custom-domain-name" title="Moniker">Moniker</a><a href="/zh-cn/documentation/articles/web-sites-dotster-custom-domain-name" title="Dotster">Dotster</a><a href="/zh-cn/documentation/articles/web-sites-domaindiscover-custom-domain-name" title="DomainDiscover">DomainDiscover</a><a href="/zh-cn/documentation/articles/web-sites-directnic-custom-domain-name" title="Directnic">Directnic</a></div>

<div class="dev-center-tutorial-subselector"><a href="/zh-cn/documentation/articles/web-sites-enom-custom-domain-name/" title="网站" class="current">网站</a> | <a href="/zh-cn/documentation/articles/web-sites-enom-traffic-manager-custom-domain-name/" title="使用流量管理器的网站">使用流量管理器的网站</a></div>

[WACOM.INCLUDE [介绍][介绍]]

本文提供了如何将从 [eNom][eNom] 购买的自定义域名用于 Azure 网站的说明。

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
若要将自定义域与 Azure 网站关联，必须使用 eNom 提供的工具在 DNS 表中为自定义域添加新条目。使用以下步骤找到用于 enom.com 的 DNS 工具

1.  通过 eNom 登录你的帐户，选择“域”，然后选择“我的域”。这样就会显示你的域名。

2.  从“我的域”页面上，使用“管理域”字段来选择“主机记录”。这样就会显示主机记录字段。

    ![“DNS 区域文件”选项卡][“DNS 区域文件”选项卡]

3.  利用“主机记录”编辑器，你可以使用“记录类型”字段选择特定的记录类型。对于 Azure 网站，你应该只选择使用“CNAME (别名)”或“A (地址)”。

    ![区域文件编辑器][区域文件编辑器]

    > [WACOM.NOTE] 在给区域文件添加条目之前，请注意，eNom 已经为根域创建了 DNS 记录 (<'@'>)，为子域创建了通配符 ('\*')。如果你希望将根域重定向到你的网站，或使用通配符 A 记录，则应修改这些条目，而不是创建新条目。

    -   添加 CNAME 记录时，必须将“主机名称”字段设置为你要使用的子域。例如，**www**。必须将“地址”字段设置为你的 Azure 网站的 **.chinacloudsites.cn** 域名。例如 **contoso.chinacloudsites.cn**。

        > [WACOM.NOTE] 如果你要使用 A 记录，则还必须使用以下配置之一添加 CNAME 记录：
        >
        > -   “别名”值 **www** 以及“其他主机”值 **\<yourwebsitename\>.chinacloudsites.cn**。
        >
        > 或者
        >
        > -   “别名”值 **awverify.www** 以及“其他主机”值 **awverify.\<yourwebsitename\>.chinacloudsites.cn**。
        >
        > 此 CNAME 记录由 Azure 用来验证你拥有 A 记录所描述的域。

    -   在添加 A 记录时，必须将“主机名称”字段设置为 <**@**>（代表根域名，如 **contoso.com**）、\*（用于匹配多个子域的通配符），或者是你要使用的某特定子域（例如 **www**）。必须将“地址”字段设置为你的 Azure 网站的 IP 地址。

        > [WACOM.NOTE] 在添加 A 记录时，也必须添加一个主机为 **awverify** 的 CNAME 记录，以及一个 \*\*awverify.\<yourwebsitename\>.chinacloudsites.cn 的“指向”。

4.  完成添加或修改记录之后，请单击“保存”，以保存这些更改。

## <a name="enabledomain"></a>在网站上启用域名

[WACOM.INCLUDE [modes][1]]

  [自定义域]: /zh-cn/documentation/articles/web-sites-custom-domain-name "自定义域"
  [GoDaddy]: /zh-cn/documentation/articles/web-sites-godaddy-custom-domain-name "GoDaddy"
  [Network Solutions]: /zh-cn/documentation/articles/web-sites-network-solutions-custom-domain-name "Network Solutions"
  [Register.com]: /zh-cn/documentation/articles/web-sites-registerdotcom-custom-domain-name "Register.com"
  [Enom]: /zh-cn/documentation/articles/web-sites-enom-custom-domain-name "Enom"
  [Moniker]: /zh-cn/documentation/articles/web-sites-moniker-custom-domain-name "Moniker"
  [Dotster]: /zh-cn/documentation/articles/web-sites-dotster-custom-domain-name "Dotster"
  [DomainDiscover]: /zh-cn/documentation/articles/web-sites-domaindiscover-custom-domain-name "DomainDiscover"
  [Directnic]: /zh-cn/documentation/articles/web-sites-directnic-custom-domain-name "Directnic"
  [网站]: /zh-cn/documentation/articles/web-sites-enom-custom-domain-name/ "网站"
  [使用流量管理器的网站]: /zh-cn/documentation/articles/web-sites-enom-traffic-manager-custom-domain-name/ "使用流量管理器的网站"
  [介绍]: ../includes/custom-dns-web-site-intro.md
  [eNom]: https://enom.com
  [introfooter]: ../includes/custom-dns-web-site-intro-notes.md
  [了解 DNS 记录]: #understanding-records
  [为基本、共享或标准模式配置网站]: #bkmk_configsharedmode
  [为自定义域添加 DNS 记录]: #bkmk_configurecname
  [在网站上启用域]: #enabledomain
  [understandingdns]: ../includes/custom-dns-web-site-understanding-dns-raw.md
  [modes]: ../includes/custom-dns-web-site-modes.md
  [“DNS 区域文件”选项卡]: ./media/web-sites-custom-domain-name/e-hostrecords.png
  [区域文件编辑器]: ./media/web-sites-custom-domain-name/e-editrecords.png
  [1]: ../includes/custom-dns-web-site-enable-on-web-site.md
