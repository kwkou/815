## 方案

为了更好地说明如何创建 NSG，本文档将使用以下方案。

![VNet 方案](./media/virtual-networks-create-nsg-scenario-include/figure1.png)

在此方案中，你将为 **TestVNet** 虚拟网络中的每个子网创建 NSG，如下面所述：

- **NSG-FrontEnd**。前端 NSG 将应用于 *FrontEnd* 子网，并且包含以下两个规则：	
	- **rdp-rule**。此规则允许将 RDP 流量传输到 *FrontEnd* 子网。
	- **web-rule**。此规则允许将 HTTP 流量传输到 *FrontEnd* 子网。
- **NSG-BackEnd**。后端 NSG 将应用于 *BackEnd* 子网，并且包含以下两个规则：	
	- **sql-rule**。此规则只允许从 *FrontEnd* 子网传输的 SQL 流量。
	- **web-rule**。此规则将拒绝从 *BackEnd* 子网传输的所有 Internet 绑定流量。

将这些规则组合起来可创建一个与 DMZ 类似的方案，其中后端子网只能接收来自前端子网的 SQL 通信的传入流量且不能访问 Internet，而前端子网可以与 Internet 通信并只接收传入 HTTP 请求。
 

<!---HONumber=76-->