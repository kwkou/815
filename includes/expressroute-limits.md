#### ExpressRoute 限制

下列限制适用于每个订阅的 ExpressRoute 资源。

| 资源 | 默认限制 |
|---|---|
| 每个订阅的 ExpressRoute 线路数 | 10 |
| ARM 的每个订阅每个区域的 ExpressRoute 线路数 | 10 |
| 具有 ExpressRoute Standard 的 Azure 私用对等互连的最大路由数 | 4,000 |
| 具有 ExpressRoute Premium 附加设备的 Azure 私用对等互连的最大路由数 | 10,000 |
| 具有 ExpressRoute Standard 的 Azure 公共对等互连的最大路由数 | 200 |
| 具有 ExpressRoute Premium 附加设备的 Azure 公共对等互连的最大路由数 | 200 |
| 具有 ExpressRoute Standard 的 Azure Microsoft 对等互连的最大路由数 | 200 |
| 具有 ExpressRoute Premium 附加设备的 Azure Microsoft 对等互连的最大路由数 | 200 |
| 每个 ExpressRoute 线路允许的虚拟网络链接数 | 请参阅下表 |

#### 每个 ExpressRoute 线路的虚拟网络数

| **线路大小** | **针对 Standard 的 VNet 链接数** | **使用 Premium 附加设备时的 VNet 链接数** |
|---|---|---|
| 10 Mbps | 10 | 不支持 |
| 50 Mbps | 10 | 20 |
| 100 Mbps | 10 | 25 |
| 200 Mbps | 10 | 25 |
| 500 Mbps | 10 | 40 |
| 1 Gbps | 10 | 50 |
| 2 Gbps | 10 | 60 |
| 5 Gbps | 10 | 75 |
| 10 Gbps | 10 | 100 |

<!---HONumber=Mooncake_1207_2015-->