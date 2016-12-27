<properties
    pageTitle="虚拟机的多个 IP 地址 - Azure CLI | Azure"
    description="了解如何使用 Azure CLI 将多个 IP 地址分配给虚拟机 | Resource Manager。"
    services="virtual-network"
    documentationcenter="na"
    author="anavinahar"
    manager="narayan"
    editor=""
    tags="azure-resource-manager" />  

<tags
    ms.assetid="ms.service: virtual-network"
    ms.devlang="na"
    ms.topic="article"
    ms.tgt_pltfrm="na"
    ms.workload="infrastructure-services"
    ms.date="11/17/2016"
    wacn.date="12/26/2016"
    ms.author="annahar" />  


# 使用 Azure CLI 将多个 IP 地址分配给虚拟机
> [AZURE.SELECTOR]
- [Azure 门户预览](/documentation/articles/virtual-network-multiple-ip-addresses-portal/)
- [PowerShell](/documentation/articles/virtual-network-multiple-ip-addresses-powershell/)
- [CLI](/documentation/articles/virtual-network-multiple-ip-addresses-cli/)

一个 Azure 虚拟机 (VM) 具有一个或多个附加的网络接口 (NIC)。可为任何 NIC 分配一个或多个静态/动态的公共和专用 IP 地址。将多个 IP 地址分配给 VM 可实现以下功能：

* 在一台服务器上托管具有不同 IP 地址和 SSL 证书的多个网站或服务。
* 用作网络虚拟设备，例如防火墙或负载均衡器。
* 能够将任何 NIC 的任意专用 IP 地址添加到 Azure Load Balancer 后端池。过去，只有主 NIC 的主 IP 地址才能添加到后端池。若要了解有关如何对多个 IP 配置进行负载均衡的详细信息，请阅读[对多个 IP 配置进行负载均衡](/documentation/articles/load-balancer-multiple-ip/)一文。

每个附加到 VM 的 NIC 都具有一个或多个与之关联的 IP 配置。系统会为每个配置分配一个静态或动态专用 IP 地址。每个配置还可拥有一个与之关联的公共 IP 地址资源。系统会为公共 IP 地址资源分配一个动态或静态 IP 地址。如果你不熟悉 Azure 中的 IP 地址，请参阅 [IP addresses in Azure](/documentation/articles/virtual-network-ip-addresses-overview-arm/)（Azure 中的 IP 地址）一文获取详细信息。

本文说明如何使用 PowerShell 将多个 IP 地址分配给通过 Azure Resource Manager 部署模型创建的 VM。无法将多个 IP 地址分配给通过经典部署模型创建的资源。若要了解有关 Azure 部署模型的详细信息，请阅读[了解部署模型](/documentation/articles/resource-manager-deployment-model/)一文。

[AZURE.INCLUDE [virtual-network-preview](../../includes/virtual-network-preview.md)]

## 方案
创建具有一个 NIC 的 VM，并连接到虚拟网络。VM 需要三个不同的 *专用* IP 地址和两个 *公共* IP 地址。IP 地址会分配给以下 IP 配置：

* **IPConfig-1**：分配一个 *动态* 专用 IP 地址（默认）和一个 *静态* 公共 IP 地址。
* **IPConfig-2**：分配一个 *静态* 专用 IP 地址和一个 *静态* 公共 IP 地址。
* **IPConfig-3**：分配一个 *动态* 专用 IP 地址，不分配公共 IP 地址。
  
	![多个 IP 地址](./media/virtual-network-multiple-ip-addresses-powershell/OneNIC-3IP.png)  


IP 配置在创建 NIC 时关联到 NIC，NIC 在创建 VM 时附加到 VM。本方案使用的 IP 地址类型用于演示目的。可分配需要的任何 IP 地址和分配类型。

## <a name = "create"></a>创建具有多个 IP 地址的 VM

以下步骤说明如何根据本方案所述，创建具有多个 IP 地址的示例 VM。根据实现的需要，更改变量名称和 IP 地址类型。

1. 按照[安装和配置 Azure CLI](/documentation/articles/xplat-cli-install/) 一文中的步骤安装和配置 Azure CLI，并登录 Azure 帐户。

3. [创建资源组](/documentation/articles/virtual-machines-linux-create-cli-complete/#create-resource-groups-and-choose-deployment-locations)，然后创建[虚拟网络和子网](/documentation/articles/virtual-machines-linux-create-cli-complete/#create-a-virtual-network-and-subnet)。按如下所示，更改 ``` --address-prefixes ``` 和 ```--address-prefix``` 字段，满足本文列出的具体方案要求：

        --address-prefixes 10.0.0.0/16
        --address-prefix 10.0.0.0/24

	>[AZURE.NOTE] 
	上面引用的文章使用“西欧”作为创建资源的位置，但本文使用“中国西北部”。相应更改位置。

4. 为 VM [创建存储帐户](/documentation/articles/virtual-machines-linux-create-cli-complete/#create-a-storage-account)。


5. 创建要分配给 NIC 的 NIC 和 IP 配置。可根据需要添加、删除或更改配置。方案所述的配置如下所示：

	**IPConfig-1**

	输入以下命令，创建：

	- 一个具有静态公共 IP 地址的公共 IP 地址资源
	- 一个具有公共 IP 地址资源和动态专用 IP 地址的 IP 配置

        azure network public-ip create --resource-group myResourceGroup --location westchinaeast --name myPublicIP --domain-name-label mypublicdns --allocation-method Static

	> [AZURE.NOTE]
	公共 IP 地址会产生少许费用。有关 IP 地址定价的详细信息，请阅读 [IP address pricing](/pricing/details/reserved-ip-addresses/)（IP 地址定价）页。可在一个订阅中使用的公共 IP 地址数有限制。若要了解有关限制的详细信息，请阅读 [Azure 限制](/documentation/articles/azure-subscription-service-limits/#networking-limits)一文。

        azure network nic create --resource-group myResourceGroup --location westchinaeast --subnet-vnet-name myVnet --subnet-name mySubnet --name myNic1 --public-ip-name myPublicIP

	**IPConfig-2**

	 输入以下命令，创建具有一个静态公共 IP 地址和一个静态专用 IP 地址的新公共 IP 地址资源和新 IP 配置：

        azure network public-ip create --resource-group myResourceGroup --location westchinaeast --name myPublicIP2 --domain-name-label mypublicdns2 --allocation-method Static
    
        azure network nic ip-config create --resource-group myResourceGroup --nic-name myNic1 --name IPConfig-2 --private-ip-address 10.0.0.5 --public-ip-name myPublicIP2

	**IPConfig-3**

	输入以下命令，创建具有一个动态专用 IP 地址且没有公共 IP 地址的 IP 配置：

        azure network nic ip-config create --resource-group myResourceGroup --nic-name myNic1 --name IPConfig-3

	>[AZURE.NOTE] 
	尽管本文将所有 IP 配置都分配给一个 NIC，但还可将多个 IP 配置分配给 VM 中的任何 NIC。若要了解如何创建具有多个 NIC 的 VM，请阅读“创建具有多个 NIC 的 VM”一文。

6. [创建 Linux VM](/documentation/articles/virtual-machines-linux-create-cli-complete/#create-the-linux-vms)。请务必删除 ```  --availset-name myAvailabilitySet \ ``` 属性，此方案不需要它。根据方案使用相应的位置。

	>[AZURE.WARNING] 
	如果选择的位置不支持 VM 大小，“创建 VM”一文中的步骤 6 会失败。例如，运行以下命令，获取美国中西部的 VM 的完整列表。此位置名称可根据方案更改。
	> 
	> 		azure vm sizes --location westchinaeast

	例如，若要将 VM 大小更改为标准 DS2 v2，只需在步骤 6 中将以下属性 ```  --vm-size Standard_DS3_v2``` 到添加 ``` azure vm create ``` 命令。

7. 输入以下命令，查看 NIC 和关联的 IP 配置：

        azure network nic show --resource-group myResourceGroup --name myNic1

## <a name="OsConfig"></a>将 IP 地址添加到 VM 操作系统

连接并登录所创建的具有多个专用 IP 地址的 VM。必须手动添加所有添加到 VM 的专用 IP 地址（包括主地址）。针对 VM 操作系统完成以下步骤：

### Windows

1. 在命令提示符下键入 *ipconfig /all* 。只能看到 *Primary* 专用 IP 地址（通过 DHCP）。
2. 在命令提示符窗口中键入 *ncpa.cpl* ，打开“网络连接”窗口。
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
6. 在命令提示符下键入 *ipconfig /all* 。此时将显示添加的所有 IP 地址，DHCP 已关闭。
	
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
4. 打开该文件：vi *文件名* 。

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
	如果使用远程连接，请在同一行中同时运行 ifdown 和 ifup。
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

	应会看到其中一个文件是 *ifcfg-eth0* 。

5. 使用以下命令复制 *ifcfg-eth0* 文件并将它命名为 *ifcfg-eth0:0* ：

        cp ifcfg-eth0 ifcfg-eth0:0

6. 使用以下命令编辑 *ifcfg-eth0:0* 文件：

        vi ifcfg-eth1

7. 使用以下命令在文件中将设备更改为适当的名称，在本例中为 *eth0:0* ：

        DEVICE=eth0:0

8. 更改 *IPADDR = YourPrivateIPAddress* 行以反映 IP 地址。
9. 使用以下命令保存该文件：

        :wq

10. 运行以下命令重新启动网络服务，确保更改成功：

        /etc/init.d/network restart
        Ipconfig

应会在返回的列表中看到添加的 IP 地址 *eth0:0* 。

## <a name="add"></a>将 IP 地址添加到 VM

完成以下步骤，可将其他专用和公共 IP 地址添加到现有 NIC。示例根据本文所述的[方案](#Scenario)生成。

1. 打开 Azure CLI，在单个 CLI 会话中完成本部分的剩余步骤。如果还没有安装和配置 Azure CLI，请完成[安装和配置 Azure CLI](/documentation/articles/xplat-cli-install/) 一文中的步骤，并登录 Azure 帐户。

2. 若要注册预览版，请向[多个 IP](mailto:MultipleIPsPreview@microsoft.com?subject=Request%20to%20enable%20subscription%20%3csubscription%20id%3e) 发送电子邮件，其中包含订阅 ID 和目标用途。如果要尝试完成剩余步骤，需要满足以下条件：
	- 收到告知允许使用预览版的电子邮件
	- 遵守收到的电子邮件中的说明


3. 根据要求，完成以下任一部分中的步骤：

	**添加专用 IP 地址**
	
	若要将专用 IP 地址添加到 NIC，必须使用以下命令创建 IP 配置。如果想要添加动态专用 IP 地址，请在输入命令前删除 ```-PrivateIpAddress 10.0.0.7```。指定的静态 IP 地址必须是子网的未使用地址。

        azure network nic ip-config create --resource-group myResourceGroup --nic-name myNic1 --private-ip-address 10.0.0.7 --name IPConfig-4

	使用唯一配置名称和专用 IP 地址（用于具有静态 IP 地址的配置），根据需要创建多个配置。

	**添加公共 IP 地址**
	
	将公共 IP 地址关联到新 IP 配置或现有 IP 配置即可添加它。根据需要，完成以下任一部分中的步骤。

	> [AZURE.NOTE]
	公共 IP 地址会产生少许费用。有关 IP 地址定价的详细信息，请阅读 [IP address pricing](/pricing/details/reserved-ip-addresses/)（IP 地址定价）页。可在一个订阅中使用的公共 IP 地址数有限制。若要了解有关限制的详细信息，请阅读 [Azure 限制](/documentation/articles/azure-subscription-service-limits/#networking-limits)一文。
	>

	**将资源关联到新 IP 配置**
	
	每次在新 IP 配置中添加公共 IP 地址时，还必须添加专用 IP 地址，因为所有 IP 配置都必须具有专用 IP 地址。可添加现有公共 IP 地址资源，也可创建新的公共 IP 地址资源。若要新建，请输入以下命令：

          azure network public-ip create --resource-group myResourceGroup --location westchinaeast --name myPublicIP3 --domain-name-label mypublicdns3

 	若要创建具有动态专用 IP 地址和关联的 *myPublicIP3* 公共 IP 地址资源的新 IP 配置，请输入以下命令：

        azure network nic ip-config create --resource-group myResourceGroup --nic-name myNic --name IPConfig-4 --public-ip-name myPublicIP3

	**将资源关联到现有 IP 配置**公共 IP 地址资源仅可关联到尚未关联公共 IP 地址资源的 IP 配置。可输入以下命令，确定某个 IP 配置是否具有关联的公共 IP 地址：

        azure network nic ip-config list --resource-group myResourceGroup --nic-name myNic1

	在返回的输出中查找类似如下所示的行：
	
		Name               Provisioning state  Primary  Private IP allocation  Private IP version  Private IP address  Subnet    Public IP
		-----------------  ------------------  -------  ---------------------  ------------------  ------------------  --------  -----------
		default-ip-config  Succeeded           true     Dynamic                IPv4                10.0.0.4            mySubnet  myPublicIP
		IPConfig-2         Succeeded           false    Static                 IPv4                10.0.0.5            mySubnet  myPublicIP2
		IPConfig-3         Succeeded           false    Dynamic                IPv4                10.0.0.6            mySubnet
	 
	*IpConfig-3* 的**公共 IP** 列为空白，因此，当前没有公共 IP 地址资源与其关联。可将现有公共 IP 地址资源添加到 IpConfig-3，或输入以下命令进行创建：

        azure network public-ip create --resource-group  myResourceGroup --location westchinaeast --name myPublicIP3 --domain-name-label mypublicdns3 --allocation-method Static

	输入以下命令，将公共 IP 地址资源关联到名为 *IPConfig-3* 的现有 IP 配置：

        azure network nic ip-config set --resource-group myResourceGroup --nic-name myNic1 --name IPConfig-3 --public-ip-name myPublicIP3

7. 输入以下命令，查看分配给 NIC 的专用 IP 地址和公共 IP 地址资源：

        azure network nic ip-config list --resource-group myResourceGroup --nic-name myNic1

	您应该会看到与下面类似的输出：
	
		Name               Provisioning state  Primary  Private IP allocation  Private IP version  Private IP address  Subnet    Public IP
		-----------------  ------------------  -------  ---------------------  ------------------  ------------------  --------  -----------
		default-ip-config  Succeeded           true     Dynamic                IPv4                10.0.0.4            mySubnet  myPublicIP
		IPConfig-2         Succeeded           false    Static                 IPv4                10.0.0.5            mySubnet  myPublicIP2
		IPConfig-3         Succeeded           false    Dynamic                IPv4                10.0.0.6            mySubnet  myPublicIP3
	 
9. 根据本文[将 IP 地址添加到 VM 操作系统](#OsConfig)部分中的说明，将添加到 NIC 的 IP 地址添加到 VM 操作系统。

<!---HONumber=Mooncake_1219_2016-->