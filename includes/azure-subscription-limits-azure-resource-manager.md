| 资源 | 默认限制 | 最大限制 |
| --- | --- | --- |
| 每个订阅的 VM 数 |每个区域 20 个<sup>1</sup> |每个区域 10,000 个 |
| 每个订阅的 VM 总核心数 |每个区域 20 个<sup>1</sup> |每个区域 10,000 个 |
| 每个订阅每个系列（Dv2、F 等）的 VM 核心数 |每个区域 20 个<sup>1</sup> |每个区域 10,000 个 |
| 每订阅的协同管理员数 |不受限制 |不受限制 |
| 每个订阅的[存储帐户数](/documentation/articles/storage-create-storage-account/) |200 |200<sup>2</sup> |
| 每个订阅的[资源组](/documentation/articles/resource-group-overview)数 |800 |800 |
| 每个订阅的[可用性集](/documentation/articles/virtual-machines-windows-manage-availability/#configure-multiple-virtual-machines-in-an-availability-set-for-redundancy)数 |每个区域 2000 个 |每个区域 2000 个 |
| 资源管理器 API 读取次数 |每小时 15000 次 |每小时 15000 次 |
| 资源管理器 API 写入次数 |每小时 1200 次 |每小时 1200 次 |
| 资源管理器 API 请求大小 |4194304 字节 |4194304 字节 |
| 每个订阅的[云服务数](/documentation/articles/cloud-services-choose-me/) |不适用<sup>3</sup> |不适用<sup>3</sup> |
| 每个订阅的[地缘组](/documentation/articles/virtual-networks-migrate-to-regional-vnet/)数 |不适用<sup>3</sup> |不适用<sup>3</sup> |

<sup>1</sup>默认限制因产品类别类型（例如试用、即用即付和系列）而有所不同，如 Dv2 等。

<sup>2</sup>这包括标准和高级存储帐户。如果需要的存储帐户超过 200 个，请通过 [Azure 支持](/support/faq/)提出请求。Azure 存储空间团队将评审你的业务案例，最多可以批准 250 个存储帐户。

<sup>3</sup>使用 Azure 资源组和 Azure 资源管理器时不再需要这些功能。

> [AZURE.NOTE]
请务必强调虚拟机核心数的区域总数限制，以及单独强制实施的每个大小系列（Dv2、F 等）的区域性限制。例如，假设某个订阅在中国东部的 VM 核心数限制为 30 个，即 A 系列的核心数限制为 30，D 系列的核心数限制也为 30。此订阅允许部署 30 个 A1 VM，或 30 个 D1 VM，或者两者的组合，但其总数不能超过 30 个核心（例如，10 个 A1 VM 和 20 个 D1 VM）。
> <!-- -->
> 
> 
>

<!---HONumber=Mooncake_0103_2017-->