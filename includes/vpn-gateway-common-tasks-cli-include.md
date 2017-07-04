### <a name="to-view-local-network-gateways"></a>查看本地网关

若要查看本地网关的列表，请使用 [az network local-gateway list](https://docs.microsoft.com/zh-cn/cli/azure/network/local-gateway#list) 命令。

    az network local-gateway list --resource-group TestRG1

### <a name="noconnection"></a>修改本地网关 IP 地址前缀 - 无网关连接

如果没有网关连接且需要添加或删除 IP 地址前缀，则可使用 [az network local-gateway create](https://docs.microsoft.com/zh-cn/cli/azure/network/local-gateway#create) 命令，该命令也是用来创建本地网关的。 也可使用该命令来更新 VPN 设备的网关 IP 地址。 若要覆盖当前设置，请使用本地网关的现有名称。 如果使用其他名称，请创建一个新的本地网关，而不是覆盖现有本地网关。

每次进行更改时，必须指定前缀的完整列表，不能仅指定要更改的前缀。 仅指定需要保留的前缀。 在此示例中，为 10.0.0.0/24 和 20.0.0.0/24

    az network local-gateway create --gateway-ip-address 23.99.221.164 --name Site2 --connection-name TestRG1 --local-address-prefixes 10.0.0.0/24 20.0.0.0/24

### <a name="withconnection"></a>修改本地网关 IP 地址前缀 - 现有网关连接

如果有网关连接且需要添加或删除 IP 地址前缀，可使用 [az network local-gateway update](https://docs.microsoft.com/zh-cn/cli/azure/network/local-gateway#update) 更新前缀。 这将导致 VPN 连接中断一段时间。 修改 IP 地址前缀时，不需删除 VPN 网关。

每次进行更改时，必须指定前缀的完整列表，不能仅指定要更改的前缀。 在此示例中，10.0.0.0/24 和 20.0.0.0/24 已存在。 我们会添加前缀 30.0.0.0/24 和 40.0.0.0/24，并在更新时指定所有 4 个前缀。

    az network local-gateway update --local-address-prefixes 10.0.0.0/24 20.0.0.0/24 30.0.0.0/24 40.0.0.0/24 --name VNet1toSite2 --connection-name TestRG1

### <a name="to-modify-the-local-network-gateway-gatewayipaddress"></a>修改本地网关的“gatewayIpAddress”

如果要连接的 VPN 设备已更改其公共 IP 地址，则需根据该更改修改本地网关。 可以更改网关 IP 地址而不删除现有的 VPN 网关连接（如果有）。 若要修改网关 IP 地址，请使用 [az network local-gateway update](https://docs.microsoft.com/zh-cn/cli/azure/network/local-gateway#update) 命令将值“Site2”和“TestRG1”替换为自己的值。

    az network local-gateway update --gateway-ip-address 23.99.222.170 --name Site2 --resource-group TestRG1

验证输出中的 IP 地址是否正确：

    "gatewayIpAddress": "23.99.222.170",

### <a name="to-verify-the-shared-key-values"></a>验证共享密钥值

验证共享密钥值与用于 VPN 设备配置的值是否相同。 如果不同，请使用设备提供的值再次运行链接，或者使用返回的值更新设备。 值必须匹配。 若要查看共享密钥，请使用 [az network vpn-connection-list](https://docs.microsoft.com/zh-cn/cli/azure/network/vpn-connection#list) 命令。

    az network vpn-connection shared-key show --connection-name VNet1toSite2 --resource-group TestRG1

### <a name="to-view-the-vpn-gateway-public-ip-address"></a>查看 VPN 网关的公共 IP 地址

若要查找虚拟网关的公共 IP 地址，请使用 [az network public-ip list](https://docs.microsoft.com/zh-cn/cli/azure/network/public-ip#list) 命令。 为了方便阅读，对本示例的输出进行了格式化，以表格式显示一系列公共 IP。

    az network public-ip list --resource-group TestRG1 --output table