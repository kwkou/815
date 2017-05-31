如果无法通过 VPN 连接连接到虚拟机，请查看以下项目：

- 验证 VPN 连接是否成功。
- 验证是否已连接到 VM 的专用 IP 地址。
- 如果可以使用专用 IP 地址连接到 VM，但不能使用计算机名称进行连接，则请验证是否已正确配置 DNS。 若要详细了解如何对 VM 进行名称解析，请参阅[针对 VM 的名称解析](/documentation/articles/virtual-networks-name-resolution-for-vms-and-role-instances/)。

通过点到站点进行连接时，请检查下述其他项：

- 使用“ipconfig”检查分配给以太网适配器的 IPv4 地址，该适配器所在的计算机正是要从其进行连接的计算机。 如果该 IP 地址位于你要连接到的 VNet 的地址范围内，或者位于 VPNClientAddressPool 的地址范围内，则称为地址空间重叠。 当地址空间以这种方式重叠时，网络流量不会抵达 Azure，而是呆在本地网络中。
- 验证是否在为 VNet 指定 DNS 服务器 IP 地址之后，才生成 VPN 客户端配置包。 如果更新了 DNS 服务器 IP 地址，请生成并安装新的 VPN 客户端配置包。

若要详细了解如何排查 RDP 连接问题，请参阅[排查到 VM 的远程桌面连接问题](/documentation/articles/virtual-machines-windows-troubleshoot-rdp-connection/)。