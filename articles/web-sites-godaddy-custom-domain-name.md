<properties title="Learn how to configure an Azure web site to use a domain name registered with GoDaddy" pageTitle="Configure a GoDaddy domain name for an Azure web site" metaKeywords="Azure, Azure Web Sites, domain name" description="" services="web-sites" documentationCenter="" authors="larryfr, jroth" />

# 为 Azure 网站配置自定义域名 (GoDaddy)

<div class="dev-center-tutorial-selector sublanding"><a href="/zh-cn/documentation/articles/web-sites-custom-domain-name" title="自定义域">自定义域</a><a href="/zh-cn/documentation/articles/web-sites-godaddy-custom-domain-name" title="GoDaddy" class="current">GoDaddy</a><a href="/zh-cn/documentation/articles/web-sites-network-solutions-custom-domain-name" title="Network Solutions">Network Solutions</a><a href="/zh-cn/documentation/articles/web-sites-registerdotcom-custom-domain-name" title="Register.com">Register.com</a><a href="/zh-cn/documentation/articles/web-sites-enom-custom-domain-name" title="Enom">Enom</a><a href="/zh-cn/documentation/articles/web-sites-moniker-custom-domain-name" title="Moniker">Moniker</a><a href="/zh-cn/documentation/articles/web-sites-dotster-custom-domain-name" title="Dotster">Dotster</a><a href="/zh-cn/documentation/articles/web-sites-domaindiscover-custom-domain-name" title="DomainDiscover">DomainDiscover</a><a href="/zh-cn/documentation/articles/web-sites-directnic-custom-domain-name" title="Directnic">Directnic</a></div>

<div class="dev-center-tutorial-subselector"><a href="/zh-cn/documentation/articles/web-sites-godaddy-custom-domain-name/" title="网站" class="current">网站</a> | <a href="/zh-cn/documentation/articles/web-sites-godaddy-traffic-manager-custom-domain-name/" title="使用流量管理器的网站">使用流量管理器的网站</a></div>

[WACOM.INCLUDE [介绍][介绍]]

本文提供了如何将从 [Go Daddy][Go Daddy] 购买的自定义域名用于 Azure 网站的说明。

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

## <a name="bkmk_configurecname"></a>为自定义域添加 DNS 记录

若要将自定义域与 Azure 网站关联，必须使用 GoDaddy 提供的工具在 DNS 表中为自定义域添加新条目。使用以下步骤找到用于 GoDaddy.com 的 DNS 工具

1.  通过 GoDaddy.com 登录你的帐户，选择“我的帐户”，然后选择“管理你的域”。最后，选择要用于你的 Azure 网站的域名。

    ![GoDaddy 的自定义域页面][GoDaddy 的自定义域页面]

2.  从“域详细信息”页面上，选择“DNS 区域文件”选项卡。此部分用于添加和修改用于你的域名的 DNS 记录。选择“编辑”按钮以显示“区域文件编辑器”。

    ![“DNS 区域文件”选项卡][“DNS 区域文件”选项卡]

3.  “区域文件编辑器”为各个记录类型划分为不同部分，从 A 记录开始（作为第一部分列为“A (主机)”，后面是 CNAME 记录（列为“CNAME (别名)”）。若要添加新条目，请使用对应部分下面的“快速添加”按钮。若要编辑现有条目，请选择该条目并修改现有信息。

    > [WACOM.NOTE] 在给区域文件添加条目之前，请注意，GoDaddy 已经为常用子域（在编辑器中被称为“主机”）创建了 DNS 记录，如“电子邮件”、“文件”、“邮件”，以及其他。如果你要使用的名称已经存在，请修改现有的记录，而不是创建新记录。

    -   添加 CNAME 记录时，必须将“主机”字段设置为你要使用的子域。例如，**www**。必须将“指向”字段设置为 Azure 网站的 **.azurewebsites.net** 域名。例如 **contoso.chinacloudsites.cn**。

        ![区域文件编辑器][区域文件编辑器]

        > [WACOM.NOTE] 如果你要使用 A 记录，则还必须使用以下配置之一添加 CNAME 记录：
        >
        > -   “指向”**\<yourwebsitename\>.chinacloudsites.cn** 值的“主机”值 **www**。
        >
        > 或者
        >
        > -   “指向”**awverify.\<yourwebsitename\>.chinacloudsites.cn** 值的“主机”值 **awverify.www**。
        >
        > 此 CNAME 记录由 Azure 用来验证你拥有 A 记录所描述的域

    -   在添加 A 记录时，必须将“主机”字段设置为 <**@*>\*（代表根域名，如 **contoso.com**）、\*（用于匹配多个子域的通配符），或者是你要使用的子域（例如 **www**）。必须将“指向”字段设置为你的 Azure 网站的 IP 地址。

        ![记录的区域文件编辑器][记录的区域文件编辑器]

4.  完成添加或修改记录之后，请单击“保存区域文件”，以保存这些更改。

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
  [网站]: /zh-cn/documentation/articles/web-sites-godaddy-custom-domain-name/ "网站"
  [使用流量管理器的网站]: /zh-cn/documentation/articles/web-sites-godaddy-traffic-manager-custom-domain-name/ "使用流量管理器的网站"
  [介绍]: ../includes/custom-dns-web-site-intro.md
  [Go Daddy]: https://godaddy.com
  [introfooter]: ../includes/custom-dns-web-site-intro-notes.md
  [了解 DNS 记录]: #understanding-records
  [为基本、共享或标准模式配置网站]: #bkmk_configsharedmode
  [为自定义域添加 DNS 记录]: #bkmk_configurecname
  [在网站上启用域]: #enabledomain
  [understandingdns]: ../includes/custom-dns-web-site-understanding-dns-raw.md
  [modes]: ../includes/custom-dns-web-site-modes.md
  [GoDaddy 的自定义域页面]: ./media/web-sites-custom-domain-name/godaddy-customdomain.png
  [“DNS 区域文件”选项卡]: ./media/web-sites-custom-domain-name/godaddy-zonetab.png
  [区域文件编辑器]: ./media/web-sites-custom-domain-name/godaddy-quickaddcname.png
  [记录的区域文件编辑器]: ./media/web-sites-custom-domain-name/godaddy-quickaddarecord.png
  [1]: ../includes/custom-dns-web-site-enable-on-web-site.md
