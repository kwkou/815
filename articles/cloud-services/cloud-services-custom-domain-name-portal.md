<properties
	pageTitle="在云服务中配置自定义域名 | Azure"
	description="了解如何通过配置 DNS 设置在自定义域上向 Internet 公开 Azure 应用程序或数据。这些示例使用 Azure 门户。"
	services="cloud-services"
	documentationCenter=".net"
	authors="Thraka"
	manager="timlt"
	editor=""/>

<tags
	ms.service="cloud-services"
	ms.date="12/14/2016"
	wacn.date="03/31/2017"/>

# 为 Azure 云服务配置自定义域名

> [AZURE.SELECTOR]
- [Azure 门户](/documentation/articles/cloud-services-custom-domain-name-portal)
- [Azure 经典管理门户](/documentation/articles/cloud-services-custom-domain-name)

创建云服务时，Azure 会将其分配给 **chinacloudapp.cn** 的子域。例如，如果云服务名为“contoso”，用户将能够访问类似 http://contoso.chinacloudapp.cn 的 URL 上的应用程序。Azure 还会分配一个虚拟 IP 地址。

但是，你还可以在自己的域名（例如 **contoso.com** ）上公开应用程序。此文章介绍了如何保留或配置云服务 Web 角色的自定义域名称。

是否已了解什么是 CNAME 和 A 记录？ [跳过解释](#add-a-cname-record-for-your-custom-domain)。

> [AZURE.NOTE]
本任务中的过程适用于 Azure 云服务。对于应用程序服务，请参阅[此文章](/documentation/articles/web-sites-custom-domain-name)。对于存储帐户，请参阅[此文章](/documentation/articles/storage-custom-domain-name)。

<p/>

> [AZURE.TIP]
快速入门 - 使用全新的 Azure [操作实例指南](http://support.microsoft.com/zh-cn/kb/2990804)！ 它可使自定义域名快速地与 Azure 云服务或 Azure 网站相关联，并确保通信安全 (SSL)。

## 了解 CNAME 和 A 记录

CNAME（即别名记录）和 A 记录都支持将域名与特定服务器（在此示例中为服务）进行关联，但是它们的工作方式有所不同。对 Azure 云服务使用 A 记录时，在确定使用哪个 A 记录之前，还有一些应该考虑的特定注意事项：

### CNAME 或别名记录

CNAME 记录会将特定的域（例如 **contoso.com** 或 **www.contoso.com** ）映射到规范域名。在这种情况下，规范域名是 Azure 托管应用程序的 **[myapp].chinacloudapp.cn** 域名。CNAME 创建后，将为 **[myapp].chinacloudapp.cn** 创建一个别名。CNAME 条目会自动解析为 **[myapp].chinacloudapp.cn** 服务的 IP 地址，因此，如果该云服务的 IP 地址发生更改，你无需采取任何措施。

> [AZURE.NOTE]
某些域注册机构只允许在使用 CNAME 记录（例如 www.contoso.com）和非根名称（例如 contoso.com）时映射子域。有关 CNAME 记录的详细信息，请参阅注册机构提供的文档或 [IETF 域名 - 实现和规范文档](http://tools.ietf.org/html/rfc1035)。

### A 记录

*A* 记录将域（例如 **contoso.com** 或 **www.contoso.com** ）或通配符域（例如 **\*.contoso.com** ）映射到 IP 地址。对于 Azure 云服务案例，则映射到该服务的虚拟 IP。与 CNAME 记录相比，A 记录的主要优势是可以拥有一个使用通配符的条目，例如 **\*.contoso.com** ，它将处理多个子域（例如 **mail.contoso.com** 、 **login.contoso.com** 或 **www.contso.com** ）的请求。

> [AZURE.NOTE]
由于 A 记录映射到静态 IP 地址，它无法自动解析云服务 IP 地址的更改。将在第一次部署到空槽（无论是生产还是临时）时分配由云服务所使用的 IP 地址。 如果删除针对该槽的部署，则 Azure 将释放该 IP 地址，并且可能为将来任何针对该槽的部署提供新的 IP 地址。
>
> 为方便起见，在临时和生产部署之间切换或对现有部署执行就地升级时，将保留给定部署槽（生产或临时）的 IP 地址。有关执行这些操作的详细信息，请参阅[如何管理云服务](/documentation/articles/cloud-services-how-to-manage)。


## <a name="add-a-cname-record-for-your-custom-domain"></a> 为自定义域添加 CNAME 记录

若要创建 CNAME 记录，必须使用注册机构提供的工具在 DNS 表中为自定义域添加新条目。每个注册机构指定 CNAME 记录的方法类似，但略有不同，其概念是相同的。

1. 使用下列方法之一找到分配给云服务的 **.chinacloudapp.cn** 域名。

   * 登录到 [Azure 门户]，选择云服务，查看“概要”部分，然后找到“站点 URL”条目。

       ![显示站点 URL 的速览部分][csurl]
            
       **或者**
  
   * 安装并配置 [Azure Powershell](/documentation/articles/powershell-install-configure)，然后使用以下命令：


        Get-AzureDeployment -ServiceName yourservicename | Select Url

    
    保存任一方法返回的 URL 中所使用的域名，因为创建 CNAME 记录时需要它。

1.  登录到 DNS 注册机构的网站，然后转至管理 DNS 的页面。查找标为 **域名** 、 **DNS** 或 **名称服务器管理** 的站点链接或区域。

2.  现在找到可以在其中选择或输入 CNAME 记录的位置。可能需要从下拉列表中选择记录类型，或者转到高级设置页面。应该查找词 **CNAME** 、 **Alias** 或 **Subdomains** 。

3.  如果希望为 **www.customdomain.com** 创建别名，还必须为 CNAME 提供域或子域别名，例如 **www** 。如果希望为根域创建别名，它可能在注册机构的 DNS 工具中以符号“**@**”的形式列出。

4. 然后，必须提供一个规范主机名，在此示例中即应用程序的 **chinacloudapp.cn** 域。

例如，以下 CNAME 记录会将 **www.contoso.com** 的全部流量都转发至 **contoso.chinacloudapp.cn** （已部署的应用程序的自定义域名）：

| 别名/主机名/子域 | 规范域 |
| ------------------------- | -------------------- |
| www | contoso.chinacloudapp.cn |

> [AZURE.NOTE]
**www.contoso.com** 的访问者将不会看到真正的主机 (contoso.chinacloudapp.cn)，因此，转发过程对最终用户不可见。

> 上述示例仅适用于 **www** 子域的流量。因为无法为 CNAME 记录使用通配符，所以必须为每个域/子域创建一个 CNAME。如果希望将子域（例如 *.contoso.com）的流量定向到 chinacloudapp.cn 地址，则可以在 DNS 设置中配置“URL 重定向”或“URL 转发”条目，或者创建一个 A 记录。


## 为自定义域添加 A 记录

若要创建 A 记录，必须首先找到云服务的虚拟 IP 地址。然后，使用注册机构所提供的工具在 DNS 表中为自定义域名添加新条目。每个注册机构指定 A 记录的方法类似，但略有不同，其概念是相同的。

1. 使用以下方法之一来获取云服务的 IP 地址。

   * 登录到 [Azure 门户]，选择云服务，查看“概要”部分，然后找到“公共 IP 地址”条目。

       ![显示 VIP 的速览部分][vip]

       **或者**

   * 安装并配置 [Azure Powershell](/documentation/articles/powershell-install-configure)，然后使用以下命令：


        get-azurevm -servicename yourservicename | get-azureendpoint -VM {$_.VM} | select Vip

    
    保存该 IP 地址，因为创建 A 记录时需要它。

1.  登录到 DNS 注册机构的网站，然后转至管理 DNS 的页面。查找标为 **域名** 、 **DNS** 或 **名称服务器管理** 的站点链接或区域。

2.  现在找到可以在其中选择或输入 A 记录的位置。可能需要从下拉列表中选择记录类型，或者转到高级设置页面。

3. 选择或输入将使用此 A 记录的域或子域。例如，如果希望为 **www.customdomain.com** 创建别名，请选择“www”。如果希望为所有子域创建通配符条目，请输入 '\_\_\*\_\_'。这将涵盖所有子域，例如 **mail.customdomain.com** 、 **login.customdomain.com** 和  **www.customdomain.com** 。

    如果希望为根域创建 A 记录，它可能在注册机构的 DNS 工具中以符号“**@**”的形式列出。

4. 在提供的字段中输入云服务的 IP 地址。这会将 A 记录中使用的域条目与云服务部署的 IP 地址相关联。

例如，以下 A 记录会将 **contoso.com** 的全部流量都转发至 **137.135.70.239**（已部署的应用程序的 IP 地址）：

| 主机名/子域 | IP 地址 |
| ------------------- | -------------- |
| @ | 137\.135.70.239 |


此示例展示了如何为根域创建 A 记录。如果希望创建一个通配符条目来涵盖所有子域，则输入 '\_\_\*\_\_' 作为子域。

>[AZURE.WARNING]
Azure 中的 IP 地址默认为动态 IP 地址。你可能想使用[保留 IP 地址](/documentation/articles/virtual-networks-reserved-public-ip)以确保 IP 地址不会更改。

## 后续步骤

* [如何管理云服务](/documentation/articles/cloud-services-how-to-manage)
* [云服务的常规配置](/documentation/articles/cloud-services-how-to-configure-portal)。
* 了解如何[部署云服务](/documentation/articles/cloud-services-how-to-create-deploy-portal)。
* 配置 [SSL 证书](/documentation/articles/cloud-services-configure-ssl-certificate-portal)。

[Expose Your Application on a Custom Domain]: #access-app
[Add a CNAME Record for Your Custom Domain]: #add-cname
[Expose Your Data on a Custom Domain]: #access-data
[VIP swaps]: /documentation/articles/cloud-services-how-to-manage-portal/#how-to-swap-deployments-to-promote-a-staged-deployment-to-production
[Create a CNAME record that associates the subdomain with the storage account]: #create-cname
[Azure 门户]: https://portal.azure.cn
[vip]: ./media/cloud-services-custom-domain-name-portal/csvip.png
[csurl]: ./media/cloud-services-custom-domain-name-portal/csurl.png
 

<!---HONumber=Mooncake_Quality_Review_1202_2016-->