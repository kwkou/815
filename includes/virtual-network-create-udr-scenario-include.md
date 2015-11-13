## 方案

为了更好地说明如何创建 UDR，本文档将使用以下方案。

![图像说明](./media/virtual-network-create-udr-scenario-include/figure1.png)

在此方案中你将为*前端子网*创建一个 UDR，并为*后端子网*创建另一个 UDR，如下所述：

- **UDR-FrontEnd**。前端 UDR 将应用于*前端*子网，并且包含一个路由：	
	- **RouteToBackend**。此路由将目标至后端子网的所有流量发送到 **FW1** 虚拟机。
- **UDR-BackEnd**。后端 UDR 将应用于*后端*子网，并且包含一个路由：	
	- **RouteToFrontend**。此路由将目标至前端子网的所有流量发送到 **FW1** 虚拟机。

这些路由的组合将确保从一个子网发送到另一个子网的所有流量都将路由到正用作虚拟设备的 **FW1** 虚拟机。你还需要打开该 VM 的 IP 转发，以确保它可以接收发送到其他虚拟机的流量。

<!---HONumber=79-->