### <a name="to-modify-the-local-network-gateway-gatewayipaddress"></a>修改本地网关的“gatewayIpAddress”

如果要连接的 VPN 设备已更改其公共 IP 地址，则需根据该更改修改本地网关。 可以更改网关 IP 地址而不删除现有的 VPN 网关连接（如果有）。 若要修改网关 IP 地址，请使用 [az network local-gateway update](https://docs.microsoft.com/zh-cn/cli/azure/network/local-gateway#update) 命令将值“Site2”和“TestRG1”替换为自己的值。

    az network local-gateway update --gateway-ip-address 23.99.222.170 --name Site2 --resource-group TestRG1

验证输出中的 IP 地址是否正确：

    "gatewayIpAddress": "23.99.222.170",