#为 Azure Web 应用配置自定义域名

当创建 Web 应用时，Azure 会在 chinacloudsites.cn 域上提供一个友好子域，以便您的用户可以使用以下 URL 访问您的 Web 应用，如 http://&lt;mysite>.chinacloudsites.cn. 但是，如果将 Web 应用配置为共享或标准模式，则可以将 Web 应用映射到您自己的域名。

另外，您可以使用 Azure Traffic Manager 对您的 Web 应用的传入流量实现负载平衡。有关 Traffic Manager 如何作用于 Web 应用的详细信息，请参阅[使用 Azure Traffic Manager 控制 Azure Web 应用流量][trafficmanager]。

> [WACOM.NOTE] 本任务中的步骤要求您将 Web 应用配置为共享或标准模式，这可能会更改对您的订阅的计费量。有关详细信息，请参阅<a href="/home/features/web-site/pricing/"> Web 应用定价详细信息</a>。

本文内容：

-   [了解 CNAME 和 A 记录](#understanding-records)
-   [将 Web 应用配置为共享或标准模式](#bkmk_configsharedmode)
-   [将 Web 应用添加到 Traffic Manager](#trafficmanager)
-   [为自定义域添加 CNAME](#bkmk_configurecname)
-   [为自定义域添加 A 记录](#bkmk_configurearecord)

<h2><a name="understanding-records"></a>了解 CNAME 和 A 记录</h2>

CNAME（即别名记录）和 A 记录都允许您将域名与 Web 应用进行关联，但是，它们的工作方式不同。

###CNAME 或别名记录

CNAME 记录将  *specific* 域（例如 **contoso.com** 或 **www.contoso.com**）映射到规范域名。在这种情况下，规范域名是您的 Azure Web 应用的 **&lt;myapp>.chinacloudsites.cn** 域名或您的 Traffic Manager 配置文件的 **&lt;myapp>.trafficmgr.com** 域名。创建映射后，CNAME 将为 **&lt;myapp>.chinacloudsites.cn** 或 **&lt;myapp>.trafficmgr.com** 域名创建一个别名。CNAME 条目将自动解析为您的 **&lt;myapp>.chinacloudsites.cn** 或 **&lt;myapp>.trafficmgr.com** 域名的 IP 地址，因此，如果 Web 应用的 IP 地址发生更改，您不必采取任何操作。

> [WACOM.NOTE] 某些域注册机构只允许您在使用 CNAME 记录（例如 www.contoso.com）和非根名称（例如 contoso.com）时映射子域。有关 CNAME 记录的详细信息，请参阅由您的注册机构提供的文档、<a href="http://en.wikipedia.org/wiki/CNAME_record">关于 CNAME 记录的 Wikipedia 条目</a>或 <a href="http://tools.ietf.org/html/rfc1035">IETF 域名 - 实现和规范</a>文档。

###A 记录

A 记录将域（例如 **contoso.com** 或 **www.contoso.com**）、 *or a wildcard domain*（例如 **\*.contoso.com**）映射到 IP 地址。对于 Azure Web 应用而言，这是服务的虚拟 IP 或者您为 Web 应用购买的具体 IP 地址。与 CNAME 记录相比，A 记录的主要优势是您可以有一个使用通配符的条目，例如 **\*.contoso.com**，它将处理多个子域（例如 **mail.contoso.com**、**login.contoso.com** 或 **www.contoso.com**）的请求。

> [WACOM.NOTE] 由于 A 记录映射到静态 IP 地址，因此它无法自动解析 Web 应用的 IP 地址的更改。用于 A 记录的 IP 地址是您为 Web 应用配置自定义域名设置时提供的；但是，如果您删除并重新创建了 Web 应用或者将 Web 应用模式改回了免费，则该值可能会更改。

> [WACOM.NOTE] A 记录不能用于使用 Traffic Manager 的负载平衡。有关详细信息，请参阅[使用 Azure Traffic Manager 控制 Azure Web 应用流量][trafficmanager]。
 
<a name="bkmk_configsharedmode"></a><h2>将 Web 应用配置为共享或标准模式</h2>

在 Web 应用上设置自定义域名只适用于 Azure Web 应用的共享和标准模式。将 Web 应用从免费 Web 应用模式切换到共享或标准 Web 应用模式之前，必须先取消 Web 应用订阅已有的支出上限。有关共享和标准模式定价的详细信息，请参阅[定价详细信息][PricingDetails]。

1. 在浏览器中，打开[经典管理门户][portal]。
2. 在" Web 应用"选项卡中，单击您的 Web 应用的名称。

	![][standardmode1]

3. 单击"规模"选项卡。

	![][standardmode2]

	
4. 在"常规"部分中，通过单击"共享"设置 Web 应用模式。

	![][standardmode3]

	> [WACOM.NOTE] 如果您将为此 Web 应用使用 Traffic Manager，则必须选择标准模式而非共享模式。

5. 单击"保存"。
6. 在系统提示您共享模式（如果您选择"标准"，则此处为标准模式）成本将增加时，如果您同意成本增加，则单击"是"。

	<!--![][standardmode4]-->

	**注意**<br /> 
	如果收到"为 Web 应用  'website name' 配置规模失败"错误，则可使用详细信息按钮获得详细信息。 

<a name="trafficmanager"></a><h2>（可选）将您的 Web 应用添加到 Traffic Manager</h2>

如果希望为 Web 应用使用 Traffic Manager，请执行下列步骤。

1. 如果还没有 Traffic Manager 配置文件，请根据[使用"快速创建"创建 Traffic Manager 配置文件][createprofile] 中的信息创建一个。记下与您的 Traffic Manager 配置文件关联的 **.trafficmgr.com** 域名。这将在后面的步骤中使用。

2. 根据[添加或删除终结点][addendpoint] 中的信息在 Traffic Manager 配置文件中将您的 Web 应用添加为终结点。

	> [WACOM.NOTE] 如果在添加终结点时您的 Web 应用未列出，请验证是否已将其配置为标准模式。必须将 Web 应用配置为标准模式，Traffic Manager 才起作用。

3. 登录到您的 DNS 注册机构的 Web 应用，然后转至用于管理 DNS 的页面。查找 Web 应用中标签为"域名"、"DNS"或"名称服务器管理"的链接或区域。

4. 现在找到您可以在其中选择或输入 CNAME 记录的位置。您可能需要从下拉列表中选择记录类型，或者需要转到高级设置页面。应当查找"CNAME"、"别名"或"子域"字样。

5. 还必须为 CNAME 提供域或子域别名。例如，如果希望为 **www.customdomain.com** 创建别名，则输入 **www**。

5. 还必须提供作为此 CNAME 别名的规范域名的主机名。这是您的 Web 应用的 **.trafficmgr.com** 名称。

例如，以下 CNAME 记录会将 **www.contoso.com** 的全部流量都转发至 **contoso.trafficmgr.com**（ Web 应用的域名）：

<table border="1" cellspacing="0" cellpadding="5" style="border: 1px solid #000000;">
<tr>
<td><strong>别名/主机名/子域</strong></td>
<td><strong>规范域</strong></td>
</tr>
<tr>
<td>www</td>
<td>contoso.trafficmgr.com</td>
</tr>
</table>

**www.contoso.com** 的访问者将永远看不到真正的主机
(contoso.azurewebsite.net)，因此，转发过程对最终用户不可见。

> [WACOM.NOTE] 如果您在为 Web 应用使用 Traffic Manager，则不需要执行下面的部分（"为自定义域添加 CNAME"和"为自定义域添加 A 记录"）中的步骤。在前面的步骤中创建的 CNAME 记录会将传入流量路由到 Traffic Manager，然后后者会将流量路由到 Web 应用终结点。

<a name="bkmk_configurecname"></a><h2>为自定义域添加 CNAME</h2>

若要创建 CNAME 记录，必须使用您的注册机构提供的工具在 DNS 表中为您的自定义域添加一个新条目。每个注册机构指定 CNAME 记录的方法类似但略有不同，但概念是相同的。

1. 使用下列方法之一查找分配给您的 Web 应用的 **.azurewebsite.net** 域名。

	* 登录到 [Azure 经典管理门户][portal]，依次选择您的 Web 应用、"仪表板"，然后在"速览"部分中查找" Web 应用 URL"条目。

	* 安装和配置 [Azure Powershell](/documentation/articles/powershell-install-configure/)，然后使用以下命令：

			get-azurewebsite yoursitename | select hostnames

	* 安装和配置 [Azure 跨平台命令行接口](/documentation/articles/xplat-cli-install/)，然后使用以下命令：

			azure site domain list yoursitename

	保存此 **.azurewebsite.net** 名称，因为在下面的步骤中将使用该名称。

3. 登录到您的 DNS 注册机构的 Web 应用，然后转至用于管理 DNS 的页面。查找 Web 应用中标签为"域名"、"DNS"或"名称服务器管理"的链接或区域。

4. 现在找到您可以在其中选择或输入 CNAME 记录的位置。您可能需要从下拉列表中选择记录类型，或者需要转到高级设置页面。应当查找"CNAME"、"别名"或"子域"字样。

5. 还必须为 CNAME 提供域或子域别名。例如，如果希望为 **www.customdomain.com** 创建别名，则输入 **www**。如果希望为根域创建别名，它可能在注册机构的 DNS 工具中以符号"@"的形式列出。

5. 还必须提供作为此 CNAME 别名的规范域名的主机名。这是您的 Web 应用的 **.azurewebsite.net** 名称。

例如，以下 CNAME 记录会将 **www.contoso.com** 的全部流量都转发至 **contoso.azurewebsite.net**（ Web 应用的域名）：

<table border="1" cellspacing="0" cellpadding="5" style="border: 1px solid #000000;">
<tr>
<td><strong>别名/主机名/子域</strong></td>
<td><strong>规范域</strong></td>
</tr>
<tr>
<td>www</td>
<td>contoso.azurewebsite.net</td>
</tr>
</table>

**www.contoso.com** 的访问者将永远看不到真正的主机
(contoso.azurewebsite.net)，因此，转发过程对
最终用户不可见。

> [WACOM.NOTE] 上述示例仅适用于 __www__ 子域的流量。因为无法为 CNAME 记录使用通配符，所以必须为每个域/子域创建一个 CNAME。如果希望将子域（例如 *.contoso.com）的流量定向到您的 azurewebsite.net 地址，则可以在 DNS 设置中配置"__URL 重定向_"或"__URL 转发__"条目，或者创建一个 A 记录。

> [WACOM.NOTE] CNAME 通过 DNS 系统向外传播可能需要一段时间。直到 CNAME 已传播出去，才能设置 Web 应用的 CNAME。可使用 <a href="http://www.digwebinterface.com/">http://www.digwebinterface.com/</a> 之类的服务确认该 CNAME 是否可用。

###将域名添加到 Web 应用

在域名的 CNAME 记录已传播后，必须将其与您的 Web 应用关联。您可以使用 Azure 跨平台命令行界面或 Azure 经典管理门户将 CNAME 记录定义的自定义域名添加到您的 Web 应用。

**使用命令行工具添加域名**

安装和配置 [Azure 跨平台命令行接口](/documentation/articles/xplat-cli-install/)，然后使用以下命令：

	azure site domain add customdomain yoursitename

例如，以下命令会将 **www.contoso.com** 的自定义域名添加到 **contoso.azurewebsite.net** Web 应用：

	azure site domain add www.contoso.com contoso

可以使用以下命令确认自定义域名是否已添加到 Web 应用：

	azure site domain list yoursitename

此命令返回的列表应当包含自定义域名以及默认的 **.azurewebsite.net** 条目。

**使用 Azure 经典管理门户添加域名**

1. 在浏览器中，打开 [Azure 经典管理门户][portal]。

2. 在" Web 应用"选项卡中，单击您的 Web 应用的名称、选择"仪表板"，然后从页面底部选择"管理域"。

	![][setcname2]

6. 在"域名"文本框中，键入您已配置的域名。 

	![][setcname3]

6. 单击复选标记以接受该域名。

在完成配置后，自定义域名将在您 Web 应用"配置"页的"域名"部分列出。 

<a name="bkmk_configurearecord"></a><h2>为自定义域添加 A 记录</h2>

若要创建 A 记录，必须首先找到您的 Web 应用的 IP 地址。然后，使用您的注册机构提供的工具在 DNS 表中为您的自定义域添加一个条目。每个注册机构指定 A 记录的方法类似但略有不同，但概念是相同的。除了创建 A 记录之外，还必须创建 Azure 用来验证 A 记录的 CNAME 记录。

1. 在浏览器中，打开 [Azure 经典管理门户][portal]。

2. 在" Web 应用"选项卡中，单击您的 Web 应用的名称、选择"仪表板"，然后从屏幕底部选择"管理域"。

	![][setcname2]

5. 在"管理自定义域"对话框上，找到"在配置 A 记录时要使用的 IP 地址"。复制 IP 地址。在创建 A 记录时将使用该地址。

5. 在"管理自定义域"对话框上，记下位于对话框顶部文本末尾的 awverify 域名。它应当是 **awverify.mysite.chinacloudsites.cn**，其中 **mysite** 是您的 Web 应用的名称。复制该域名，因为在创建验证用的 CNAME 记录时将使用该域名。

6. 登录到您的 DNS 注册机构的 Web 应用，然后转至用于管理 DNS 的页面。查找 Web 应用中标签为"域名"、"DNS"或"名称服务器管理"的链接或区域。

6. 找到您可以在其中选择或输入 A 记录和 CNAME 记录的位置。您可能需要从下拉列表中选择记录类型，或者需要转到高级设置页面。

7. 执行下列步骤以创建 A 记录：

	1. 选择或输入将使用 A 记录的域或子域。例如，如果希望为 **www.customdomain.com** 创建别名，请选择 **www**。如果希望为所有子域创建通配符条目，请输入 '__*__'。这将涵盖所有子域，例如 **mail.customdomain.com**、**login.customdomain.com** 和 **www.customdomain.com**。

	 如果希望为根域创建 A 记录，它可能在注册机构的 DNS 工具中以符号"@"的形式列出。

	2. 在提供的字段中输入您的云服务的 IP 地址。这会将 A 记录中使用的域条目与您的云服务部署的 IP 地址相关联。

	 例如，以下 A 记录会将 **contoso.com** 的全部流量都转发至 **137.135.70.239**（我们已部署的应用程序的 IP 地址）：

	 <table border="1" cellspacing="0" cellpadding="5" style="border: 1px solid #000000;">
	 <tr>
	 <td><strong>主机名/子域</strong></td>
	 <td><strong>IP 地址</strong></td>
	 </tr>
	 <tr>
	 <td>@</td>
	 <td>137.135.70.239</td>
	 </tr>
	 </table>

	 此示例展示了如何为根域创建 A 记录。如果希望创建一个通配符条目来涵盖所有子域，则输入 '__*__' 作为子域。

7. 接下来，创建一个别名为 **awverify** 且规范域为之前获得的 **awverify.mysite.chinacloudsites.cn** 的 CNAME 记录。

	> [WACOM.NOTE] 虽然别名 awverify 对某些注册机构而言可能有效，但是另一些注册机构可能需要完整的别名域名 awverify.www.customdomainname.com 或 awverify.customdomainname.com。

	例如，以下示例将创建 Azure 可以用来验证 A 记录配置的 CNAME 记录。

	<table border="1" cellspacing="0" cellpadding="5" style="border: 1px solid #000000;">
	<tr>
	<td><strong>别名/主机名/子域</strong></td>
	<td><strong>规范域</strong></td>
	</tr>
	<tr>
	<td>awverify</td>
	<td>awverify.contoso.chinacloudsites.cn</td>
	</tr>
	</table>

> [WACOM.NOTE] awverify CNAME 通过 DNS 系统向外传播可能需要一段时间。在 awverify CNAME 传播出去之前，您无法为 Web 应用设置 A 记录定义的自定义域名。可使用 <a href="http://www.digwebinterface.com/">http://www.digwebinterface.com/</a> 之类的服务确认该 CNAME 是否可用。

###将域名添加到 Web 应用

在域名的 **awverify** CNAME 记录已传播后，必须将 A 记录定义的自定义域与您的 Web 应用相关联。您可以使用 Azure 跨平台命令行界面或 Azure 经典管理门户将 A 记录定义的自定义域名添加到您的 Web 应用。

**使用命令行工具添加域名**

安装和配置 [Azure 跨平台命令行接口](/documentation/articles/xplat-cli-install/)，然后使用以下命令：

	azure site domain add customdomain yoursitename

例如，以下命令会将 **contoso.com** 的自定义域名添加到 **contoso.azurewebsite.net** Web 应用：

	azure site domain add contoso.com contoso

可以使用以下命令确认自定义域名是否已添加到 Web 应用：

	azure site domain list yoursitename

此命令返回的列表应当包含自定义域名以及默认的 **.azurewebsite.net** 条目。

**使用 Azure 经典管理门户添加域名**

1. 在浏览器中，打开 [Azure 经典管理门户][portal]。

2. 在" Web 应用"选项卡中，单击您的 Web 应用的名称、选择"仪表板"，然后从页面底部选择"管理域"。

	![][setcname2]

6. 在"域名"文本框中，键入您已配置的域名。 

	![][setcname3]

6. 单击复选标记以接受该域名。

在完成配置后，自定义域名将在您 Web 应用"配置"页的"域名"部分列出。

> [WACOM.NOTE] 在将 A 记录定义的自定义域名添加到您的 Web 应用后，可以使用您的注册机构提供的工具删除 awverify CNAME 记录。不过，如果您将来希望添加其他 A 记录，则必须重新创建 awverify 记录，然后才能将新的 A 记录定义的新域名与 Web 应用相关联。

## 后续步骤

-   [如何管理 Web 应用](/documentation/articles/web-sites-manage/)

-   [为 Web 应用配置 SSL 证书](/documentation/articles/web-sites-configure-ssl-certificate/)


<!-- Bookmarks -->

[将 Web 应用配置为共享模式]: #bkmk_configsharedmode
[配置您的域注册机构中的 CNAME]: #bkmk_configurecname
[在您的域注册机构上配置 CNAME 验证记录]: #bkmk_configurecname
[为域名配置 A 记录]:#bkmk_configurearecord
[在经典管理门户中设置域名]: #bkmk_setcname

<!-- Links -->

[PricingDetails]: /pricing/overview/
[portal]: http://manage.windowsazure.cn
[digweb]: http://www.digwebinterface.com/
[cloudservicedns]: /documentation/articles/custom-dns/
[trafficmanager]: /documentation/articles/web-sites-traffic-manager/
[addendpoint]: http://msdn.microsoft.com/zh-cn/library/azure/hh744839.aspx
[createprofile]: http://msdn.microsoft.com/zh-cn/library/azure/dn339012.aspx

<!-- images -->

[setcname1]: ../media/dncmntask-cname-5.png


<!-- images -->
[standardmode1]: ./media/custom-dns-web-site/dncmntask-cname-1.png
[standardmode2]: ./media/custom-dns-web-site/dncmntask-cname-2.png
[standardmode3]: ./media/custom-dns-web-site/dncmntask-cname-3.png
[standardmode4]: ./media/custom-dns-web-site/dncmntask-cname-4.png 


[setcname2]: ./media/custom-dns-web-site/dncmntask-cname-6.png
[setcname3]: ./media/custom-dns-web-site/dncmntask-cname-7.png
<!--HONumber=41-->
