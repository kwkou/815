## <a name="os-config"></a>将 IP 地址添加到 VM 操作系统

连接并登录所创建的具有多个专用 IP 地址的 VM。必须手动添加所有添加到 VM 的专用 IP 地址（包括主地址）。针对 VM 操作系统完成以下步骤：

### Windows

1. 在命令提示符下键入 *ipconfig /all*。只能看到 *Primary* 专用 IP 地址（通过 DHCP）。
2. 在命令提示符窗口中键入 *ncpa.cpl*，打开“网络连接”窗口。
3. 打开相应的“本地连接”适配器的属性。
4. 双击“Internet 协议版本 4 (IPv4)”。
5. 单击“使用下面的 IP 地址”并输入以下值：

    * **IP 地址**：输入 *Primary* 专用 IP 地址
    * **子网掩码**：根据子网设置此值。例如，如果子网为 /24 子网，则子网掩码为 255.255.255.0。
    * **默认网关**：子网中的第一个 IP 地址。如果子网为 10.0.0.0/24，则网关 IP 地址为 10.0.0.1。
    * 单击“使用下面的 DNS 服务器地址”并输入以下值：
        * **首选 DNS 服务器**：如果不使用自己的 DNS 服务器，请输入 168.63.129.16。如果使用自己的 DNS 服务器，请输入服务器的 IP 地址。
    * 单击“高级”按钮，然后添加其他 IP 地址。使用为主 IP 地址指定的相同子网，为 NIC 添加步骤 8 中列出的每个辅助专用 IP 地址。
        >[AZURE.WARNING] 
        如果没有正确地遵循上述步骤，则可能会失去到 VM 的连接。在继续操作之前，请确保为步骤 5 输入的信息是准确的。

    * 单击“确定”关闭“TCP/IP 设置”，然后再次单击“确定”关闭适配器设置。此时会重新建立 RDP 连接。

6. 在命令提示符下键入 *ipconfig /all*。此时将显示添加的所有 IP 地址，DHCP 已关闭。

### 验证 (Windows)

若要确保你能够通过关联的公共 IP 从辅助 IP 配置连接到 Internet，请在使用上述步骤正确地添加它以后，执行以下命令：

    ping -S 10.0.0.5 hotmail.com

>[AZURE.NOTE]
仅当你所使用的上述专用 IP 地址具有与之关联的公共 IP 时，才能 ping 到 Internet。

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
4. 打开 文件。该文件的末尾应会显示以下命令行：

        auto eth0
        iface eth0 inet dhcp

5. 在此文件包含的命令行后面添加以下命令行：

        iface eth0 inet static
        address <your private IP address here>
        netmask <your subnet mask>

6. 使用以下命令保存该文件：

        :wq

7. 使用以下命令重置网络接口：

        sudo ifdown eth0 && sudo ifup eth0

    > [AZURE.IMPORTANT]
    如果使用远程连接，请在同一行中同时运行 ifdown 和 ifup。
    >

8. 使用以下命令验证 IP 地址是否已添加到网络接口：

        ip addr list eth0

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

5. 若要添加 IP 地址，请为其创建配置文件，如下所示。请注意，必须为每个 IP 配置创建一个文件。

        touch ifcfg-eth0:0

6. 使用以下命令打开 *ifcfg-eth0:0* 文件：

        vi ifcfg-eth0:0

7. 在此示例中，请使用以下命令向文件 *eth0:0* 添加内容。请务必根据 IP 地址更新信息。

        DEVICE=eth0:0
        BOOTPROTO=static
        ONBOOT=yes
        IPADDR=192.168.101.101
        NETMASK=255.255.255.0

8. 使用以下命令保存该文件：

        :wq

9. 运行以下命令重新启动网络服务，确保更改成功：

        /etc/init.d/network restart
        ifconfig

    应会在返回的列表中看到添加的 IP 地址 *eth0:0*。

### 验证 (Linux)

若要确保你能够通过关联的公共 IP 从辅助 IP 配置连接到 Internet，请执行以下命令：

    ping -I 10.0.0.5 hotmail.com

>[AZURE.NOTE]
仅当你所使用的上述专用 IP 地址具有与之关联的公共 IP 时，才能 ping 到 Internet。

对于 Linux VM，在尝试验证源自辅助 NIC 的出站连接时，可能需要添加适当的路由。可通过多种方式执行此操作。对于 Linux 分发版，请参阅相应的文档。下面是实现此目的的一种方法：

    echo 150 custom >> /etc/iproute2/rt_tables 

    ip rule add from 10.0.0.5 lookup custom
    ip route add default via 10.0.0.1 dev eth2 table custom

- 确保执行以下替换：
	- 将 **10.0.0.5** 替换为有关联的公共 IP 地址的专用 IP 地址
	- 将 **10.0.0.1** 替换为默认网关
	- 将 **eth2** 替换为辅助 NIC 的名称

<!---HONumber=Mooncake_0320_2017-->