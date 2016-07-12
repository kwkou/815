
若要配置 VPN 设备，你需要使用虚拟网络网关的公共 IP 地址来配置本地 VPN 设备。请联系你的设备制造商以获得具体的配置信息并配置设备。有关可在 Azure 上正常工作的 VPN 设备的详细信息，请参阅 [VPN Devices](/documentation/articles/vpn-gateway-about-vpn-devices/)（VPN 设备）。

若要使用 PowerShell 查找虚拟网络网关的公共 IP 地址，请使用以下示例：

	Get-AzureRmPublicIpAddress -Name GW1PublicIP -ResourceGroupName TestRG

也可以使用 Azure 门户预览来查看虚拟网络网关的公共 IP 地址。导航到“虚拟网络网关”，然后单击网关的名称。

<!---HONumber=Mooncake_0425_2016-->
