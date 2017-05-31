### <a name="noconnection"></a>修改本地网关 IP 地址前缀 - 无网关连接

如果没有网关连接且需要添加或删除 IP 地址前缀，则可使用 [az network local-gateway create](https://docs.microsoft.com/zh-cn/cli/azure/network/local-gateway#create) 命令，该命令也是用来创建本地网关的。 也可使用该命令来更新 VPN 设备的网关 IP 地址。 若要覆盖当前设置，请使用本地网关的现有名称。 如果使用其他名称，请创建一个新的本地网关，而不是覆盖现有本地网关。

每次进行更改时，必须指定前缀的完整列表，不能仅指定要更改的前缀。 仅指定需要保留的前缀。 在此示例中，为 10.0.0.0/24 和 20.0.0.0/24

    az network local-gateway create --gateway-ip-address 23.99.221.164 --name Site2 --connection-name TestRG1 --local-address-prefixes 10.0.0.0/24 20.0.0.0/24

### <a name="withconnection"></a>修改本地网关 IP 地址前缀 - 现有网关连接

如果有网关连接且需要添加或删除 IP 地址前缀，可使用 [az network local-gateway update](https://docs.microsoft.com/zh-cn/cli/azure/network/local-gateway#update) 更新前缀。 这将导致 VPN 连接中断一段时间。 修改 IP 地址前缀时，不需删除 VPN 网关。

每次进行更改时，必须指定前缀的完整列表，不能仅指定要更改的前缀。 在此示例中，10.0.0.0/24 和 20.0.0.0/24 已存在。 我们会添加前缀 30.0.0.0/24 和 40.0.0.0/24，并在更新时指定所有 4 个前缀。

    az network local-gateway update --local-address-prefixes 10.0.0.0/24 20.0.0.0/24 30.0.0.0/24 40.0.0.0/24 --name VNet1toSite2 --connection-name TestRG1