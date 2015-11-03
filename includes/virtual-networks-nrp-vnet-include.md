## 虚拟网络
虚拟网络 (VNET) 和子网可帮助定义 Azure 中运行的工作负载的安全边界。VNET 的特征包括一个地址空间（也称为 CIDR 块）的集合。

子网是 VNET 的子资源，可帮助定义使用 IP 地址前缀在 CIDR 块中定义地址空间的段。可以将 NIC 添加到子网，并连接到 VM，以便为各种工作负荷提供连接。

![单个 VM 上的 NIC](./media/resource-groups-networking/Figure4.png)

VNET 资源的关键属性包括：

- IP 地址空间（CIDR 块） 
- VNET 名称
- 子网
- DNS 服务器

还可以将 VNET 关联到下列网络资源：

- VPN 网关

### 子网

子网的关键属性包括：

- IP 地址前缀
- 子网名称

还可以将子网关联到下列网络资源：

- NSG

<!---HONumber=76-->