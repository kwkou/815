<properties title="Learn how to configure an Azure web site that uses Traffic Manager to use a domain name registered with eNom" pageTitle="Configure an eNom domain name for an Azure web site that uses Traffic Manager" metaKeywords="Windows Azure, Windows Azure Web Sites, domain name" description="" services="web-sites" documentationCenter="" authors="larryfr, jroth" />

# 使用流量管理器为 Azure 网站配置自定义域名 (eNom)

<div class="dev-center-tutorial-selector sublanding"><a href="/en-us/documentation/articles/web-sites-custom-domain-name" title="自定义域">自定义域</a><a href="/en-us/documentation/articles/web-sites-godaddy-custom-domain-name" title="GoDaddy">GoDaddy</a><a href="/en-us/documentation/articles/web-sites-network-solutions-custom-domain-name" title="Network Solutions">Network Solutions</a><a href="/en-us/documentation/articles/web-sites-registerdotcom-custom-domain-name" title="Register.com">Register.com</a><a href="/en-us/documentation/articles/web-sites-enom-custom-domain-name" title="Enom" class="current">Enom</a><a href="/en-us/documentation/articles/web-sites-moniker-custom-domain-name" title="Moniker">Moniker</a><a href="/en-us/documentation/articles/web-sites-dotster-custom-domain-name" title="Dotster">Dotster</a><a href="/en-us/documentation/articles/web-sites-domaindiscover-custom-domain-name" title="DomainDiscover">DomainDiscover</a><a href="/en-us/documentation/articles/web-sites-directnic-custom-domain-name" title="Directnic">Directnic</a></div>

<div class="dev-center-tutorial-subselector"><a href="/en-us/documentation/articles/web-sites-enom-custom-domain-name/" title="网站">网站</a> | <a href="/en-us/documentation/articles/web-sites-enom-traffic-manager-custom-domain-name/" title="使用流量管理器的网站" class="current">使用流量管理器的网站</a></div>

[WACOM.INCLUDE [介绍][介绍]]

本文提供了如何将从 [eNom][eNom] 购买的自定义域名用于 Azure 网站的说明。

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
若要将自定义域与 Azure 网站关联，必须使用 eNom 提供的工具在 DNS 表中为自定义域添加新条目。使用以下步骤找到用于 eNom.com 的 DNS 工具

1.  通过 eNom 登录你的帐户，选择“域”，然后选择“我的域”。这样就会显示你的域名。

2.  从“我的域”页面上，使用“管理域”字段来选择“主机记录”。这样就会显示主机记录字段。

    ![“DNS 区域文件”选项卡][“DNS 区域文件”选项卡]

3.  利用“主机记录”编辑器，你可以使用“记录类型”字段选择特定的记录类型。对于使用流量管理器的 Azure 网站，只应选择使用“CNAME (别名)”，因为流量管理器只能用于 CNAME 记录。

    ![区域文件编辑器][区域文件编辑器]

    -   添加 CNAME 记录时，必须将“主机名称”字段设置为你要使用的子域。例如，**www**。必须将“地址”字段设置为用于 Azure 网站的流量管理器配置文件的“.trafficmanager.cn”域名。例如，**contoso.trafficmanager.cn**。

4.  完成添加或修改记录之后，请单击“保存”，以保存这些更改。

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
  [网站]: /en-us/documentation/articles/web-sites-enom-custom-domain-name/ "网站"
  [使用流量管理器的网站]: /en-us/documentation/articles/web-sites-enom-traffic-manager-custom-domain-name/ "使用流量管理器的网站"
  [介绍]: ../includes/custom-dns-web-site-intro-traffic-manager.md
  [eNom]: https://enom.com
  [tmwebsitefooter]: ../includes/custom-dns-web-site-traffic-manager-notes.md
  [introfooter]: ../includes/custom-dns-web-site-intro-notes.md
  [了解 DNS 记录]: #understanding-records
  [将网站配置为标准模式]: #bkmk_configsharedmode
  [为自定义域添加 DNS 记录]: #bkmk_configurecname
  [为网站启用流量管理器]: #enabledomain
  [understandingdns]: ../includes/custom-dns-web-site-understanding-dns-traffic-manager.md
  [modes]: ../includes/custom-dns-web-site-modes-traffic-manager.md
  [“DNS 区域文件”选项卡]: ./media/web-sites-custom-domain-name/e-hostrecords.png
  [区域文件编辑器]: ./media/web-sites-custom-domain-name/e-editrecordstm.png
  [1]: ../includes/custom-dns-web-site-enable-on-traffic-manager.md
