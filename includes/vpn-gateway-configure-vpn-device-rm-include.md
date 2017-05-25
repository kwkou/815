通过站点到站点连接连接到本地网络需要 VPN 设备。 虽然我们提供的配置步骤不是针对所有 VPN 设备的，但是你可以访问以下链接，获取有用的信息：

- 有关兼容 VPN 设备的信息，请参阅 [VPN 设备](/documentation/articles/vpn-gateway-about-vpn-devices/)。 
- 有关设备配置设置的链接，请参阅[已验证的 VPN 设备](/documentation/articles/vpn-gateway-about-vpn-devices/#devicetable)。 我们会尽力提供各种设备配置链接。 如需最新的配置信息，最好是咨询设备制造商。
- 若要了解如何编辑设备配置示例，请参阅[编辑示例](/documentation/articles/vpn-gateway-about-vpn-devices/#editing)。
- 对于 IPsec/IKE 参数，请参阅[参数](/documentation/articles/vpn-gateway-about-vpn-devices/#ipsec)。
- 在配置 VPN 设备之前，对于要使用的 VPN 设备，请查看是否存在任何[已知的设备兼容性问题](/documentation/articles/vpn-gateway-about-vpn-devices/#known)。

配置 VPN 设备时，需要以下项：

- 虚拟网关的“公共 IP 地址”。

    -  若要使用 Azure 门户预览查找公共 IP 地址，请导航到“虚拟网关”，然后单击网关的名称。 
    - 若要使用 PowerShell 查找虚拟网关的公共 IP 地址，请使用以下示例，将相关值替换为自己的值。

            Get-AzureRmPublicIpAddress -Name GW1PublicIP -ResourceGroupName TestRG
- 共享密钥。 此共享密钥就是在创建站点到站点 VPN 连接时指定的共享密钥。 在示例中，我们使用很基本的共享密钥。 你应该生成更复杂的可用密钥。