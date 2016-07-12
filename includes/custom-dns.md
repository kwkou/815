# 为 Azure 云服务配置自定义域名

> [WACOM.NOTE]
> 快速入门 - 使用全新的 Azure [操作实例指南](https://support.microsoft.com/zh-CN/kb/2990804)!它可使自定义域名快速地与 Azure 云服务或 Azure Web 应用相关联，并确保通信安全 (SSL)。

当您在 Azure 中创建应用程序时，Azure 会在 chinacloudapp.cn 域上提供一个子域，以便您的用户可以使用以下 URL 访问您的应用程序，如 http://&lt;*myapp*>.chinacloudapp.cn. 但是，您还可以在自己的域名（例如 contoso.com）上公开应用程序。

> [WACOM.NOTE] 
> 本任务中的过程适用于 Azure 云服务。有关存储帐户，请参阅[为 Azure 存储帐户配置自定义域名](/documentation/articles/storage-custom-domain-name/)。有关 Web 应用，请参阅[为 Azure Web 应用配置自定义域名](/documentation/articles/web-sites-custom-domain-name/)。

本文内容：

-   [了解 CNAME 和 A 记录](#access-app)
-   [为自定义域添加 CNAME 记录](#add-cname)
-   [为自定义域添加 A 记录](#add-aname)

<h2><a name="access-app"></a>了解 CNAME 和 A 记录</h2>

CNAME（即别名记录）和 A 记录都允许您将域名与特定服务器（在此示例中为服务）进行关联，但是它们的工作方式不同。对 Azure 云服务使用 A 记录时，在确定使用哪个 A 记录之前，还有一些应该考虑的特定注意事项：

###CNAME 或别名记录

CNAME 记录将  *specific* 域（例如 **contoso.com** 或 **www.contoso.com**）映射到规范域名。在这种情况下，规范域名是您的 Azure 托管应用程序的 **&lt;myapp>.chinacloudapp.cn** 域名。创建映射后，CNAME 将为 **&lt;myapp>.chinacloudapp.cn** 创建一个别名。CNAME 条目将自动解析为您的 **&lt;myapp>.chinacloudapp.cn** 服务的 IP 地址，因此，如果该云服务的 IP 地址发生更改，您无需采取任何措施。

> [WACOM.NOTE] 
> 某些域注册机构只允许您在使用 CNAME 记录（例如 www.contoso.com）和非根名称（例如 contoso.com）时映射子域。有关 CNAME 记录的详细信息，请参阅由您的注册机构提供的文档、<a href="http://en.wikipedia.org/wiki/CNAME_record">关于 CNAME 记录的 Wikipedia 条目</a>或 <a href="http://tools.ietf.org/html/rfc1035">IETF 域名 - 实现和规范</a>文档。

###A 记录

A 记录将域（例如 **contoso.com** 或 **www.contoso.com**）、 *or a wildcard domain*（例如 **\*.contoso.com**）映射到 IP 地址。在 Azure 云服务案例中是映射到该服务的虚拟 IP。与 CNAME 记录相比，A 记录的主要优势是您可以有一个使用通配符的条目，例如 **\*.contoso.com**，它将处理多个子域（例如 **mail.contoso.com**、**login.contoso.com** 或 **www.contso.com**）的请求。

> [WACOM.NOTE]
> 由于 A 记录映射到静态 IP 地址，它无法动态解析您的云服务的 IP 地址的更改。将在您第一次部署到空槽（无论是生产还是临时）时分配由您的云服务所使用的 IP 地址。如果您删除针对该槽的部署，则 Azure 将释放该 IP 地址，并且可能为将来任何针对该槽的部署提供新的 IP 地址。
> 
> 为方便起见，在临时和生产部署之间切换或对现有部署执行就地升级时，将保留给定部署槽（生产或临时）的 IP 地址。有关执行这些操作的详细信息，请参阅[如何管理云服务](/documentation/articles/cloud-services-how-to-manage/)。


<h2><a name="add-cname"></a>为自定义域添加 CNAME 记录</h2>

若要创建 CNAME 记录，必须使用您的注册机构提供的工具在 DNS 表中为您的自定义域添加一个新条目。每个注册机构指定 CNAME 记录的方法类似但略有不同，但概念是相同的。

1. 使用下列方法之一查找分配给您的云服务的 **.chinacloudapp.cn** 域名。

  * 登录到 [Azure 管理门户]，依次选择您的云服务、"仪表板"，然后在"速览"部分中查找" Web 应用 URL"条目。

  		  ![显示 Web 应用 URL 的速览部分][csurl]

  * 安装和配置 [Azure Powershell](/documentation/articles/powershell-install-configure/)，然后使用以下命令：

    Get-AzureDeployment -ServiceName yourservicename | Select Url

  保存任一方法返回的 URL 中所使用的域名，因为您将在创建 CNAME 记录时需要它。

1.  登录到您的 DNS 注册机构的 Web 应用，然后转至用于管理 DNS 的页面。查找 Web 应用中标签为"域名"、"DNS"或"名称服务器管理"的链接或区域。

2.  现在找到您可以在其中选择或输入 CNAME 记录的位置。您可能需要从下拉列表中选择记录类型，或者需要转到高级设置页面。应当查找"CNAME"、"别名"或"子域"字样。

3.  如果希望为 **www.customdomain.com** 创建别名，还必须为 CNAME 提供域或子域别名，例如 **www**。如果希望为根域创建别名，它可能在注册机构的 DNS 工具中以符号"@"的形式列出。

4. 然后，您必须提供一个规范主机名，在此示例中它是您的应用程序的 **chinacloudapp.cn** 域。

例如，以下 CNAME 记录会将 **www.contoso.com** 的全部流量都转发至 **contoso.chinacloudapp.cn**（您已部署的应用程序的自定义域名）：

<table border="1" cellspacing="0" cellpadding="5" style="border: 1px solid #000000;">
<tr>
<td><strong>别名/主机名/子域</strong></td>
<td><strong>规范域</strong></td>
</tr>
<tr>
<td>www</td>
<td>contoso.chinacloudapp.cn</td>
</tr>
</table>

**www.contoso.com** 的访问者将永远看不到真正的主机 
(contoso.chinacloudapp.cn)，因此转发过程对
最终用户不可见。

> [WACOM.NOTE]
> 上述示例仅适用于 <strong>www</strong> 子域的流量。因为无法为 CNAME 记录使用通配符，所以必须为每个域/子域创建一个 CNAME。如果希望将子域（例如 *.contoso.com）的流量定向到您的 chinacloudapp.cn 地址，则可以在 DNS 设置中配置"URL 重定向"<strong></strong>或"URL 转发"<strong></strong>条目，或者创建一个 A 记录。


<h2><a name="add-aname"></a>为自定义域添加记录</h2>

若要创建 A 记录，必须首先找到您的云服务的虚拟 IP 地址。然后，使用您的注册机构所提供的工具在 DNS 表中为您的自定义域名添加新条目。每个注册机构指定 A 记录的方法类似但略有不同，但概念是相同的。

1. 使用以下方法之一来获取您的云服务的 IP 地址。

  * 登录到 [Azure 管理门户]，依次选择您的云服务、"仪表板"，然后在"速览"部分中查找"公共虚拟 IP (VIP) 地址"条目。

   		 ![显示 VIP 的速览部分][vip]

  * 安装和配置 [Azure Powershell](/documentation/articles/powershell-install-configure/)，然后使用以下命令：

      get-azurevm -servicename yourservicename | get-azureendpoint -VM {$_.VM} | select Vip

    如果您具有与您的云服务相关联的多个终结点，则将收到包含 IP 地址的多个行，但是它们全都应该显示相同的地址。

  保存该 IP 地址，因为您将在创建 A 记录时需要它。

1.  登录到您的 DNS 注册机构的 Web 应用，然后转至用于管理 DNS 的页面。查找 Web 应用中标签为"域名"、"DNS"或"名称服务器管理"的链接或区域。

2.  现在找到您可以在其中选择或输入 A 记录的位置。您可能需要从下拉列表中选择记录类型，或者需要转到高级设置页面。

3. 选择或输入将使用此 A 记录的域或子域。例如，如果希望为 **www.customdomain.com** 创建别名，请选择 **www**。如果希望为所有子域创建通配符条目，请输入 '__*__'。这将涵盖所有子域，例如 **mail.customdomain.com**、**login.customdomain.com** 和 **www.customdomain.com**。

  如果希望为根域创建 A 记录，它可能在注册机构的 DNS 工具中以符号"@"的形式列出。

4. 在提供的字段中输入您的云服务的 IP 地址。这会将 A 记录中使用的域条目与您的云服务部署的 IP 地址相关联。

例如，以下 A 记录会将 **contoso.com** 的全部流量都转发至 **137.135.70.239**（您已部署的应用程序的 IP 地址）：

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

## 后续步骤

-   [如何管理云服务](/documentation/articles/cloud-services-how-to-manage/)
-   [如何将 CDN 内容映射到自定义域][]

  [在自定义域中公开应用程序]: #access-app
  [为自定义域添加 CNAME 记录]: #add-cname
  [在自定义域中公开应用程序]: #access-data
  [VIP 交换]: http://msdn.microsoft.com/zh-cn/library/ee517253.aspx
  [创建将子域与存储帐户相关联的 CNAME 记录]: #create-cname
  [Azure 管理门户]: https://manage.windowsazure.cn
  ["验证自定义域"对话框]: http://i.msdn.microsoft.com/dynimg/IC544437.jpg
  [如何将 CDN 内容映射到自定义域]: http://msdn.microsoft.com/zh-cn/library/windowsazure/gg680307.aspx
  [vip]: ./media/custom-dns/csvip.png
  [csurl]: ./media/custom-dns/csurl.png
  
  <!--HONumber=41-->
