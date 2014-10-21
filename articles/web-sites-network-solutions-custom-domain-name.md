<properties title="Learn how to configure an Azure web site to use a domain name registered with Network Solutions" pageTitle="Configure a Network Solutions domain name for an Azure web site" metaKeywords="Azure, Azure Web Sites, domain name" description="" services="web-sites" documentationCenter="" authors="larryfr, jroth" />

# 为 Azure 网站配置自定义域名 (Network Solutions)

<div class="dev-center-tutorial-selector sublanding"><a href="/en-us/documentation/articles/web-sites-custom-domain-name" title="自定义域">自定义域</a><a href="/en-us/documentation/articles/web-sites-godaddy-custom-domain-name" title="GoDaddy">GoDaddy</a><a href="/en-us/documentation/articles/web-sites-network-solutions-custom-domain-name" title="Network Solutions" class="current">Network Solutions</a><a href="/en-us/documentation/articles/web-sites-registerdotcom-custom-domain-name" title="Register.com">Register.com</a><a href="/en-us/documentation/articles/web-sites-enom-custom-domain-name" title="Enom">Enom</a><a href="/en-us/documentation/articles/web-sites-moniker-custom-domain-name" title="Moniker">Moniker</a><a href="/en-us/documentation/articles/web-sites-dotster-custom-domain-name" title="Dotster">Dotster</a><a href="/en-us/documentation/articles/web-sites-domaindiscover-custom-domain-name" title="DomainDiscover">DomainDiscover</a><a href="/en-us/documentation/articles/web-sites-directnic-custom-domain-name" title="Directnic">Directnic</a></div>

<div class="dev-center-tutorial-subselector"><a href="/en-us/documentation/articles/web-sites-network-solutions-custom-domain-name/" title="网站" class="current">网站</a> | <a href="/en-us/documentation/articles/web-sites-network-solutions-traffic-manager-custom-domain-name/" title="使用流量管理器的网站">使用流量管理器的网站</a></div>

[WACOM.INCLUDE [介绍][介绍]]

本文提供了如何将从 [Network Solutions][1] 购买的自定义域名用于 Azure 网站的说明。

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
若要将自定义域与 Azure 网站关联，必须使用由 Network Solutions 提供的工具在 DNS 表中为自定义域添加新条目。使用以下步骤找到并使用 networksolutions.com 的 DNS 工具

1.  在 networksolutions.com 上登录到你的帐户，然后在右上角选择“我的帐户”。

2.  从“我的产品和服务”选项卡上，选择“编辑 DNS”。

    ![编辑 DNS 页面][编辑 DNS 页面]

3.  从“域名”页面上的“管理 <yourdomainname>”部分中选择“编辑高级 DNS 记录”。

    ![突出显示了编辑高级 DNS 记录的域名页][突出显示了编辑高级 DNS 记录的域名页]

4.  “更新高级 DNS”页面包含用于各个记录类型的部分，每个部分下都有一个“编辑”按钮。

    -   对于 A 记录，请使用“IP 地址(A 记录)”部分。
    -   对于 CNAME 记录，请使用“主机别名(CNAME 记录)”部分。

    ![更新高级 DNS 页面][更新高级 DNS 页面]

5.  当你单击“编辑”按钮时，你将看到一个窗体，可用于修改现有记录，或添加新记录。

    > [WACOM.NOTE] 添加新条目之间，请注意，Network Solutions 已经为根域 (<'@'>) 之类的内容和子域的通配符记录 ('\*') 创建了一些默认的 DNS 记录。如果你要使用的记录已经存在，请修改现有的记录，而不是创建新记录。

    -   添加 CNAME 记录时，必须将“别名”字段设置为你要使用的子域。例如，**www**。你必须选择“其他主机”字段旁边的环形字段，并将“其他主机”设置为你的 Azure 网站的 **.chinacloudsites.cn** 域名。例如 **contoso.chinacloudsites.cn**。将“引用主机名称”保留为“选择”，因为在创建用于 Azure 网站的 CNAME 记录时此字段不是必填字段。

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

    -   在添加 A 记录时，必须将“主机”字段设置为 <**@*>\*（代表根域名，如 **contoso.com**）、\*（用于匹配多个子域的通配符），或者是你要使用的子域（例如 **www**）。必须将“数值 IP”字段设置为你的 Azure 网站的 IP 地址。

        ![记录窗体][记录窗体]

6.  完成添加或修改记录之后，请单击“继续”，以查看这些更改。选择**“仅保存更改”**，保存这些更改。

## <a name="enabledomain"></a>在网站上启用域名

[WACOM.INCLUDE [modes][2]]

  [自定义域]: /en-us/documentation/articles/web-sites-custom-domain-name "自定义域"
  [GoDaddy]: /en-us/documentation/articles/web-sites-godaddy-custom-domain-name "GoDaddy"
  [Network Solutions]: /en-us/documentation/articles/web-sites-network-solutions-custom-domain-name "Network Solutions"
  [Register.com]: /en-us/documentation/articles/web-sites-registerdotcom-custom-domain-name "Register.com"
  [Enom]: /en-us/documentation/articles/web-sites-enom-custom-domain-name "Enom"
  [Moniker]: /en-us/documentation/articles/web-sites-moniker-custom-domain-name "Moniker"
  [Dotster]: /en-us/documentation/articles/web-sites-dotster-custom-domain-name "Dotster"
  [DomainDiscover]: /en-us/documentation/articles/web-sites-domaindiscover-custom-domain-name "DomainDiscover"
  [Directnic]: /en-us/documentation/articles/web-sites-directnic-custom-domain-name "Directnic"
  [网站]: /en-us/documentation/articles/web-sites-network-solutions-custom-domain-name/ "网站"
  [使用流量管理器的网站]: /en-us/documentation/articles/web-sites-network-solutions-traffic-manager-custom-domain-name/ "使用流量管理器的网站"
  [介绍]: ../includes/custom-dns-web-site-intro.md
  [1]: https://networksolutions.com
  [introfooter]: ../includes/custom-dns-web-site-intro-notes.md
  [了解 DNS 记录]: #understanding-records
  [为基本、共享或标准模式配置网站]: #bkmk_configsharedmode
  [为自定义域添加 DNS 记录]: #bkmk_configurecname
  [在网站上启用域]: #enabledomain
  [understandingdns]: ../includes/custom-dns-web-site-understanding-dns-raw.md
  [modes]: ../includes/custom-dns-web-site-modes.md
  [编辑 DNS 页面]: ./media/web-sites-custom-domain-name/ns-editdns.png
  [突出显示了编辑高级 DNS 记录的域名页]: ./media/web-sites-custom-domain-name/ns-editadvanced.png
  [更新高级 DNS 页面]: ./media/web-sites-custom-domain-name/ns-updateadvanced.png
  [cname 窗体]: ./media/web-sites-custom-domain-name/ns-cname.png
  [记录窗体]: ./media/web-sites-custom-domain-name/ns-arecord.png
  [2]: ../includes/custom-dns-web-site-enable-on-web-site.md
