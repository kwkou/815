可以连接到已部署到 VNet 的 VM，方法是创建到 VM 的远程桌面连接。 若要通过初始验证来确认能否连接到 VM，最好的方式是使用其专用 IP 地址而不是计算机名称进行连接。 这种方式是测试能否进行连接，而不是测试名称解析是否已正确配置。

1. 定位专用 IP 地址。 查找 VM 的专用 IP 地址时，可以通过 Azure 门户或 PowerShell 查看 VM 的属性。

    - Azure 门户 - 在 Azure 门户中定位虚拟机。 查看 VM 的属性。 专用 IP 地址已列出。

    - PowerShell - 通过此示例查看资源组中的 VM 和专用 IP 地址的列表。 在使用此示例之前不需对其进行修改。

            $vms = get-azurermvm
            $nics = get-azurermnetworkinterface | where VirtualMachine -NE $null

            foreach($nic in $nics)
            {
              $vm = $vms | where-object -Property Id -EQ $nic.VirtualMachine.id
              $prv = $nic.IpConfigurations | select-object -ExpandProperty PrivateIpAddress
              $alloc = $nic.IpConfigurations | select-object -ExpandProperty PrivateIpAllocationMethod
              Write-Output "$($vm.Name): $prv,$alloc"
            }

2. 验证是否已使用 VPN 连接连接到 VNet。
3. 打开**远程桌面连接**，方法是：在任务栏的搜索框中键入“RDP”或“远程桌面连接”，然后选择“远程桌面连接”。 也可在 PowerShell 中使用“mstsc”命令打开远程桌面连接。 
4. 在远程桌面连接中，输入 VM 的专用 IP 地址。 可以通过单击“显示选项”来调整其他设置，然后进行连接。

### <a name="to-troubleshoot-an-rdp-connection-to-a-vm"></a>排查到 VM 的 RDP 连接的问题

如果无法通过 VPN 连接连接到虚拟机，请查看以下项目：

- 验证 VPN 连接是否成功。
- 验证是否已连接到 VM 的专用 IP 地址。
- 如果可以使用专用 IP 地址连接到 VM，但不能使用计算机名称进行连接，则请验证是否已正确配置 DNS。 若要详细了解如何对 VM 进行名称解析，请参阅[针对 VM 的名称解析](/documentation/articles/virtual-networks-name-resolution-for-vms-and-role-instances/)。
- 若要详细了解 RDP 连接，请参阅[排查到 VM 的远程桌面连接问题](/documentation/articles/virtual-machines-windows-troubleshoot-rdp-connection/)。