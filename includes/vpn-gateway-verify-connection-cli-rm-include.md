可使用 [az network vpn-connection show](https://docs.microsoft.com/zh-cn/cli/azure/network/vpn-connection#show) 命令来验证连接是否成功。 将值配置为与自己的值相符。 在此示例中，“ --Name”是指所创建的需要测试的连接的名称。

    az network vpn-connection show --name VNet1toSite2 --resource-group TestRG1

当连接仍在建立过程中时，连接状态会显示“正在连接”。 建立连接后，状态将更改为“已连接”，且可以看到入口和出口字节数。