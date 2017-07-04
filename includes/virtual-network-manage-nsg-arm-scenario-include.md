<!-- Ibiza portal: tested -->

## 示例方案

为了更好地说明如何管理 NSG，本文将使用以下方案。

![VNet 方案](./media/virtual-networks-create-nsg-scenario-include/figure1.png)

在此方案中，你将为 **TestVNet** 虚拟网络中的每个子网创建 NSG，如下面所述：

- **NSG-FrontEnd**。前端 NSG 将应用于 *FrontEnd* 子网，并且包含以下两个规则：	
	- **rdp-rule**。此规则允许将 RDP 流量传输到 *FrontEnd* 子网。
	- **web-rule**。此规则允许将 HTTP 流量传输到 *FrontEnd* 子网。
- **NSG-BackEnd**。后端 NSG 将应用于 *BackEnd* 子网，并且包含以下两个规则：	
	- **sql-rule**。此规则只允许从 *FrontEnd* 子网传输的 SQL 流量。
	- **web-rule**。此规则将拒绝从 *BackEnd* 子网传输的所有 Internet 绑定流量。

将这些规则组合起来可创建一个与 DMZ 类似的方案，其中后端子网只能接收来自前端子网的 SQL 通信的传入流量且不能访问 Internet，而前端子网可以与 Internet 通信并只接收传入 HTTP 请求。

若要部署上述方案，请按照[此链接](http://github.com/telmosampaio/azure-templates/tree/master/201-IaaS-WebFrontEnd-SQLBackEnd-NSG)，下载该模板，并且做必要的修改，然后使用 PowerShell 或者 CLI 发布模板。

>[AZURE.NOTE] 你从 GitHub 仓库 "azure-quickstart-templates" 中下载的模板，需要做一些修改才能适用于 Azure 中国云环境。例如，替换一些终结点 -- "blob.core.windows.net" 替换成 "blob.core.chinacloudapi.cn"，"cloudapp.azure.com" 替换成 "chinacloudapp.cn"；改掉一些不支持的 VM 映像，还有，改掉一些不支持的 VM 大小。

<!---HONumber=Mooncake_0516_2016-->