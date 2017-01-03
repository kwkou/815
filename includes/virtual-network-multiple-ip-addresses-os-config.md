## <a name="os-config"></a>将 IP 地址添加到 VM 操作系统

连接并登录所创建的具有多个专用 IP 地址的 VM。必须手动添加所有添加到 VM 的专用 IP 地址（包括主地址）。针对 VM 操作系统完成以下步骤：

### Windows

1. 在命令提示符下键入 *ipconfig /all*。只能看到 *Primary* 专用 IP 地址（通过 DHCP）。
2. 在命令提示符窗口中键入 *ncpa.cpl*，打开“网络连接”窗口。
3. 打开“本地连接”的属性。
4. 双击“Internet 协议版本 4 (IPv4)”。
5. 单击“使用下面的 IP 地址”并输入以下值：

	* **IP 地址**：输入 *Primary* 专用 IP 地址
	* **子网掩码**：根据子网设置此值。例如，如果子网为 /24 子网，则子网掩码为 255.255.255.0。
	* **默认网关**：子网中的第一个 IP 地址。如果子网为 10.0.0.0/24，则网关 IP 地址为 10.0.0.1。
	* 单击“使用下面的 DNS 服务器地址”并输入以下值：
		* **首选 DNS 服务器**：如果不使用自己的 DNS 服务器，请输入 168.63.129.16。如果使用自己的 DNS 服务器，请输入服务器的 IP 地址。
	* 单击“高级”按钮，然后添加其他 IP 地址。使用为主 IP 地址指定的相同子网，为 NIC 添加步骤 8 中列出的每个辅助专用 IP 地址。
	* 单击“确定”关闭“TCP/IP 设置”，然后再次单击“确定”关闭适配器设置。此时会重新建立 RDP 连接。
6. 在命令提示符下键入 *ipconfig /all*。此时将显示添加的所有 IP 地址，DHCP 已关闭。
	
### Linux (Ubuntu)

1. 打开终端窗口。
2. 请确保以 root 用户身份操作。如果不是，请输入以下命令：

        sudo -i

3. 更新网络接口（假设为“eth0”）的配置文件。

    * 保留 dhcp 的现有行项。主 IP 地址会保留之前的配置。
    * 使用以下命令添加其他静态 IP 地址的配置：

            cd /etc/network/interfaces.d/
            ls

    应会看到一个 .cfg 文件。
4. 打开该文件：vi *文件名*。

    该文件的末尾应会显示以下命令行：

        auto eth0
        iface eth0 inet dhcp

5. 在此文件包含的命令行后面添加以下命令行：

        iface eth0 inet static
        address <your private IP address here>

6. 使用以下命令保存该文件：

        :wq

7. 使用以下命令重置网络接口：

        sudo ifdown eth0 && sudo ifup eth0

    > [AZURE.IMPORTANT]
    > 如果使用远程连接，请在同一行中同时运行 ifdown 和 ifup。
    >

8. 使用以下命令验证 IP 地址是否已添加到网络接口：

        Ip addr list eth0

应会在列表中看到添加的 IP 地址。
	
### Linux（Redhat、CentOS 和其他操作系统）

1. 打开终端窗口。
2. 请确保以 root 用户身份操作。如果不是，请输入以下命令：

        sudo -i

3. 输入密码，根据提示的说明操作。切换为 root 用户后，使用以下命令导航到网络脚本文件夹：

        cd /etc/sysconfig/network-scripts

4. 使用以下命令列出相关的 ifcfg 文件：

        ls ifcfg-*

    应会看到其中一个文件是 *ifcfg-eth0*。

5. 使用以下命令复制 *ifcfg-eth0* 文件并将它命名为 *ifcfg-eth0:0*：

        cp ifcfg-eth0 ifcfg-eth0:0

6. 使用以下命令编辑 *ifcfg-eth0:0* 文件：

        vi ifcfg-eth1

7. 使用以下命令在文件中将设备更改为适当的名称，在本例中为 *eth0:0*：

        DEVICE=eth0:0

8. 更改 *IPADDR = YourPrivateIPAddress* 行以反映 IP 地址。
9. 使用以下命令保存该文件：

        :wq

10. 运行以下命令重新启动网络服务，确保更改成功：

        /etc/init.d/network restart
        Ipconfig

应会在返回的列表中看到添加的 IP 地址 *eth0:0*。

<!---HONumber=Mooncake_1226_2016-->