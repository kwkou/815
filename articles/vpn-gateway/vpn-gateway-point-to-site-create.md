<properties
    pageTitle="使用点到站点连接将计算机连接到 Azure 虚拟网络：经典管理门户 | Azure"
    description="使用经典管理门户创建点到站点 VPN 网关连接，安全连接到经典 Azure 虚拟网络。"
    services="vpn-gateway"
    documentationcenter="na"
    author="cherylmc"
    manager="timlt"
    editor=""
    tags="azure-service-management"
    translationtype="Human Translation" />
<tags
    ms.assetid="4f5668a5-9b3d-4d60-88bb-5d16524068e0"
    ms.service="vpn-gateway"
    ms.devlang="na"
    ms.topic="hero-article"
    ms.tgt_pltfrm="na"
    ms.workload="infrastructure-services"
    ms.date="03/20/2017"
    wacn.date="04/17/2017"
    ms.author="cherylmc"
    ms.sourcegitcommit="e0e6e13098e42358a7eaf3a810930af750e724dd"
    ms.openlocfilehash="9d2e536a0697a3cfdeac345f67c5faac23421954"
    ms.lasthandoff="04/06/2017" />

# <a name="configure-a-point-to-site-connection-to-a-vnet-using-the-classic-management-portal-classic"></a>使用经典管理门户配置与 VNet 的点到站点连接（经典）
> [AZURE.SELECTOR]
- [Resource Manager - Azure 门户预览](/documentation/articles/vpn-gateway-howto-point-to-site-resource-manager-portal/)
- [Resource Manager - PowerShell](/documentation/articles/vpn-gateway-howto-point-to-site-rm-ps/)
- [经典 - Azure 门户预览](/documentation/articles/vpn-gateway-howto-point-to-site-classic-azure-portal/)
- [经典 - 经典管理门户](/documentation/articles/vpn-gateway-point-to-site-create/)

使用点到站点 (P2S) 配置可以创建从单个客户端计算机到虚拟网络的安全连接。 P2S 是基于 SSTP（安全套接字隧道协议）的 VPN 连接。 如果要从远程位置（例如，从家里或会议室）连接到 VNet，或者只有少数几台客户端计算机需要连接到虚拟网络，点到站点连接将非常有用。 P2S 连接不需要 VPN 设备或面向公众的 IP 地址。 可从客户端计算机建立 VPN 连接。

本文逐步讲解如何使用经典管理门户，在经典部署模型中创建具有点到站点连接的 VNet。 有关点到站点连接的详细信息，请参阅本文末尾的 [点到站点常见问题解答](#faq) 。

### <a name="deployment-models-and-methods-for-p2s-connections"></a>P2S 连接的部署模型和方法
[AZURE.INCLUDE [deployment models](../../includes/vpn-gateway-deployment-models-include.md)]

下表显示了 P2S 配置的两种部署模型和可用的部署方法。 当有配置步骤相关的文章发布时，我们会直接从此表格链接到该文章。

[AZURE.INCLUDE [vpn-gateway-clasic-rm](../../includes/vpn-gateway-table-point-to-site-include.md)]

## <a name="basic-workflow"></a>基本工作流
![点到站点连接示意图](./media/vpn-gateway-point-to-site-create/p2sclassic.png "点到站点")

以下步骤逐步建立与虚拟网络的安全点到站点连接。 点到站点连接的配置分为四个部分。 必须遵循这些部分的先后顺序完成配置。 请不要跳过步骤。

* **第 1 部分** 创建虚拟网络和 VPN 网关。
* **第 2 部分** 创建并上载用于身份验证的证书。
* **第 3 部分** 导出并安装客户端证书。
* **第 4 部分** 配置 VPN 客户端。

## <a name="vnetvpn"></a>第 1 部分 - 创建虚拟网络和 VPN 网关

开始之前，请确保你拥有 Azure 订阅。 如果你还没有 Azure 订阅，可以注册获取[试用帐户](/pricing/1rmb-trial)。

### <a name="part-1-create-a-virtual-network"></a>第 1 部分：创建虚拟网络
1. 登录到 [Azure 经典管理门户](https://manage.windowsazure.cn)。 这些步骤使用经典管理门户而不是 Azure 门户预览。 目前无法使用 Azure 门户预览创建 P2S 连接。
2. 在屏幕左下角，单击“新建”。 在导航窗格中，单击“网络服务”，然后单击“虚拟网络”。 单击“自定义创建”以启动配置向导  。
3. 在“虚拟网络详细信息”  页上，输入以下信息，然后单击右下角的“下一步”箭头。

    * **名称**- 为虚拟网络命名。 例如“VNet1”。 将 VM 部署到此 VNet 时，需要引用此名称。
    * **位置**：位置直接与你想让资源 (VM) 驻留在的物理位置（区域）有关。 例如，如果你希望部署到此虚拟网络的 VM 的物理位置位于中国东部，请选择该位置。 创建虚拟网络后，将无法更改与虚拟网络关联的区域。
4. 在“DNS 服务器和 VPN 连接”页上，输入以下信息，然后单击右下角的“下一步”箭头  。

    * **DNS 服务器**：输入 DNS 服务器名称和 IP 地址，或从快捷菜单中选择一个以前注册的 DNS 服务器。 此设置不创建 DNS 服务器。 此设置允许指定要用于对此虚拟网络进行名称解析的 DNS 服务器。 如果你想要使用 Azure 默认名称解析服务，请将本部分留空。
    * **配置点到站点 VPN**：选中此复选框。
5. 在“点到站点连接”  页上，指定你的 VPN 客户端在连接后接收 IP 地址时的 IP 地址范围。 有几个与用户能够指定的地址范围相关的规则。 必须确保指定的范围与本地网络上的任何范围不重叠。
6. 请输入以下信息，然后单击“下一步”箭头。

    * **地址空间**：包括“起始 IP”和 CIDR（地址计数）。
    * **添加地址空间**：仅在网络设计需要时才添加地址空间。
7. 在“虚拟网络地址空间”页上，指定要用于虚拟网络的地址范围。 这些都是动态 IP 地址 (DIPS)，将分配给你部署到此虚拟网络的 VM 和其他角色实例。<br><br>所选范围不要与本地网络所用范围重叠，这一点尤其重要。 必须与网络管理员协调，他们可能需要从本地网络地址空间划分一个 IP 地址范围供虚拟网络使用。
8. 输入以下信息，然后单击复选标记即可创建虚拟网络。

    * **地址空间**：添加要用于此虚拟网络的内部 IP 地址范围，包括起始 IP 和计数。 所选范围不要与本地网络所用范围重叠，这一点非常重要。 
    * **添加子网**：附加的子网不是必需的，但你可能需要为具有静态 DIP 的 VM 创建一个单独的子网。 或者，你可能需要在子网中拥有与其他角色实例分开的 VM。
    * **添加网关子网**：网关子网是点到站点 VPN 所必需的。 单击此项可添加网关子网。 网关子网仅用于虚拟网络网关。
9. 创建虚拟网络后，可以在 Azure 经典管理门户中的“网络”页上，看到“状态”下面列出了“已创建”。 创建虚拟网络后，便可以创建动态路由网关。

### <a name="part-2-create-a-dynamic-routing-gateway"></a>第 2 部分：创建动态路由网关
必须将网关类型配置为动态。 静态路由网关无法使用此功能。

1. 在 Azure 经典管理门户的“网络”页上，单击创建的虚拟网络，然后导航到“仪表板”页。
2. 在“仪表板”页的底部，单击“创建网关”。 。 单击“是”  即可开始创建网关。 创建网关最多可能需要 45 分钟。

## <a name="generate"></a>第 2 部分 - 生成证书并上传
Azure 使用证书对点到站点 VPN 的 VPN 客户端进行身份验证。 创建根证书以后，请将公共证书数据（不是密钥）作为 Base-64 编码的 X.509 .cer 文件导出。 然后，将公共证书数据从根证书上载到 Azure。

在使用点到站点连接连接到 VNet 的每台客户端计算机上，必须安装客户端证书。 客户端证书从根证书生成，安装在每个客户端计算机上。 如果未安装有效的客户端证书，而客户端尝试连接到 VNet，则身份验证会失败。

在本部分中，需要执行以下操作：

* 获取根证书的 .cer 文件。 这可以是自签名根证书，也可以使用企业证书系统。
* 将 .cer 文件上传到 Azure。
* 生成客户端证书。

### <a name="root"></a>第 1 部分：获取根证书的 .cer 文件

如果要使用企业级解决方案，可以使用现有的证书链。 获取要使用的根证书的 .cer 文件。

如果使用的不是企业证书解决方案，则需要创建自签名根证书。 若要创建包含进行 P2S 身份验证所需的字段的自签名根证书，可以使用 PowerShell。 [使用 PowerShell 为点到站点连接创建自签名的根证书](/documentation/articles/vpn-gateway-certificates-point-to-site/)将引导用户完成相关步骤，以便创建自签名的根证书。

> [AZURE.NOTE]
> 以前，为点到站点连接创建自签名根证书和生成客户端证书的建议方法是使用 makecert。 现在，也可以使用 PowerShell 来创建这些证书。 使用 PowerShell 的优势之一是能够创建 SHA-2 证书。 请参阅[使用 PowerShell 为点到站点连接创建自签名根证书](/documentation/articles/vpn-gateway-certificates-point-to-site/)，了解所需的值。
>
>

#### <a name="to-export-the-public-key-for-a-self-signed-root-certificate"></a>导出自签名根证书的公钥。

点到站点连接要求将公钥 (.cer) 上载到 Azure。 以下步骤帮助你导出自签名根证书的 .cer 文件。

1. 若要获取证书 .cer 文件，请打开 **certmgr.msc**。 找到自签名根证书（通常位于“Certificates - Current User\Personal\Certificates”中），然后右键单击。 单击“所有任务”，然后单击“导出”。 此操作将打开“证书导出向导”。
2. 在向导中，单击“下一步”。 选择“否，不导出私钥”，然后单击“下一步”。
3. 在“导出文件格式”页上，选择“Base-64 编码的 X.509 (.CER)”，然后单击“下一步”。 
4. 在“要导出的文件”中，“浏览”到要将证书导出的目标位置。 在“文件名”中，为证书文件命名。 然后单击“下一步” 。
5. 单击“完成”以导出证书  。 会看到“导出已成功” 。 单击“确定”  关闭该向导。

### <a name="upload"></a>第 2 部分：将根证书 .cer 文件上载到 Azure 经典管理门户

将受信任的证书添加到 Azure。 在将 Base64 编码 X.509 (.cer) 文件添加到 Azure 时，则是在告诉 Azure 信任该文件所代表的根证书。 最多可以上载 20 个根证书的文件。 不要将根证书的私钥上载到 Azure。 上载 .Cer 文件后，Azure 将使用它来对连接到虚拟网络的客户端进行身份验证。

1. 在 Azure 经典管理门户的虚拟网络“证书”页上，单击“上载根证书”。
2. 在“上载证书”页上  ，浏览 .cer 根证书，然后单击复选标记。

### <a name="createclientcert"></a>第 3 部分：生成客户端证书
可以为要连接的每个客户端生成唯一证书，也可以在多台客户端上使用同一个证书。 生成唯一客户端证书的优势是能够根据需要吊销单个证书。 否则，如果每个人都使用相同的客户端证书，在需要吊销某个客户端的证书时，必须为所有使用该证书进行身份验证的客户端生成并安装新证书。

####<a name="enterprise-certificate"></a>企业证书
- 如果使用的是企业证书解决方案，请使用通用名称值格式“name@yourdomain.com”生成客户端证书，而不要使用“域名\用户名”格式。
- 请确保颁发的客户端证书基于“用户”证书模板，该模板使用“客户端身份验证”作为使用列表中的第一项，而不是智能卡登录等。可以通过双击客户端证书，然后查看“详细信息”>“增强型密钥用法”来检查证书。

####<a name="self-signed-root-root-certificate"></a>自签名根证书 
如果使用的是自签名根证书，请参阅[使用 PowerShell 生成客户端证书](/documentation/articles/vpn-gateway-certificates-point-to-site/#clientcert)，了解生成与点到站点连接兼容的客户端证书的步骤。

## <a name="installclientcert"></a>第 3 部分 - 导出并安装客户端证书

生成客户端证书以后，必须将该证书作为 .pfx 文件导出，然后将其安装在客户端计算机上。 每台客户端计算机都必须拥有客户端证书才能进行身份验证。 可以自动安装客户端证书，也可以手动安装该证书。 以下步骤将引导用户手动导出并安装客户端证书。

### <a name="export-the-client-certificate"></a>导出客户端证书

1. 若要导出客户端证书，请打开 **certmgr.msc**。 右键单击要导出的客户端证书，单击“所有任务”，然后单击“导出”。 此操作将打开“证书导出向导”。
2. 在向导中，单击“**下一步**”，选择“**是，导出私钥**”，然后单击“**下一步**”。
3. 在“导出文件格式”页上，保留选择默认值。 请务必选中“包括证书路径中的所有证书(如果可能)”。 。
4. 在“安全性”页上，必须保护私钥  。 如果选择使用密码，请务必记下或牢记为此证书设置的密码。 。
5. 在“要导出的文件”中，“浏览”到要将证书导出的目标位置。 在“文件名”中，为证书文件命名。 然后单击“下一步” 。
6. 单击“完成”导出证书。

### <a name="install-the-client-certificate"></a>安装客户端证书

安装客户端证书时，需要使用导出客户端证书时创建的密码。

1. 找到 *.pfx* 文件并将其复制到客户端计算机。 在客户端计算机上，双击 *.pfx* 文件以进行安装。 将“**存储位置**”保留为“**当前用户**”，然后单击“**下一步**”。
2. 在“要导入的**文件**”页上，不要进行任何更改。 单击“下一步”。
3. 在“**私钥保护**”页上，如果使用了密码，请输入证书的密码，或验证安装证书的安全主体是否正确，然后单击“**下一步**”。
4. 在“**证书存储**”页上，保留默认位置，然后单击“**下一步**”。
5. 单击“**完成**”。 在证书安装的“**安全警告**”上，单击“**是**”。 可随时单击“是”，因为证书已生成。 现已成功导入证书。

## <a name="vpnclientconfig"></a>第 4 部分 - 配置 VPN 客户端
若要连接到虚拟网络，还需配置 VPN 客户端。 客户端要求提供客户端证书和正确的 VPN 客户端配置包才能成功进行连接。 要配置 VPN 客户端，请按顺序执行以下步骤。

### <a name="part-1-create-the-vpn-client-configuration-package"></a>第 1 部分：创建 VPN 客户端配置包
1. 在 Azure 经典管理门户中虚拟网络的“仪表板”页上，导航到右上角的速览菜单。 VPN 客户端包中含有用于配置 Windows 内置 VPN 客户端软件的配置信息。 该程序包不安装额外的软件。 这些设置特定于要连接到的虚拟网络。 有关支持的客户端操作系统列表，请参阅本文末尾的[点到站点连接常见问题解答](#faq)。<br><br>选择与要在其中进行安装的客户端操作系统对应的下载包：

    * 对于 32 位客户端，请选择“下载 32 位客户端 VPN 程序包”。
    * 对于 64 位客户端，请选择“下载 64 位客户端 VPN 程序包”。
2. 创建客户端程序包需要几分钟的时间。 安装完程序包后，便可以下载文件。 你下载的 *.exe* 文件可以安全地存储在本地计算机上。
3. 在 Azure 经典管理门户中生成和下载 VPN 客户端程序包后，可以将客户端程序包安装在客户端计算机上，需要从该计算机连接到虚拟网络。 如果你计划将 VPN 客户端程序包安装到多台客户端计算机，请确保每台计算机都安装了客户端证书。

### <a name="part-2-install-the-vpn-configuration-package-on-the-client"></a>第 2 部分：在客户端上安装 VPN 配置包
1. 将配置文件通过本地方式复制到需要连接到虚拟网络的计算机上，然后双击 .exe 文件。 
2. 安装完程序包后，即可启动 VPN 连接。 Microsoft 没有对配置包进行签名。 可能需要使用组织的签名服务对程序包进行签名，也可以自行使用 [SignTool](http://go.microsoft.com/fwlink/p/?LinkId=699327)对其签名。 可以使用不签名的程序包。 但是，如果程序包未签名，那么在安装该程序包时，会显示一条警告。
3. 在客户端计算机上，导航到“网络设置”，然后单击“VPN”。 此时将看到列出的连接。 它将显示要连接到的虚拟网络的名称，如下所示： 

    ![VPN 客户端](./media/vpn-gateway-point-to-site-create/vpn.png "VPN 客户端")

### <a name="part-3-connect-to-azure"></a>第 3 部分：连接到 Azure
1. 若要连接到 VNet，请在客户端计算机上导航到 VPN 连接，找到创建的 VPN 连接。 其名称与虚拟网络的名称相同。 单击“连接”。 可能会出现与使用证书相关的弹出消息。 如果出现此消息，请单击“继续”  以使用提升的权限。 
2. 在“连接”状态页上，单击“连接”以启动连接。 如果看到“选择证书”屏幕，请确保所显示的客户端证书是要用来连接的证书。 如果不是，请使用下拉箭头选择正确的证书，然后单击“确定”。

    ![VPN 客户端 2](./media/vpn-gateway-point-to-site-create/clientconnect.png "VPN 客户端连接")
3. 现在应已建立连接。

    ![VPN 客户端 3](./media/vpn-gateway-point-to-site-create/connected.png "VPN 客户端连接 2")

> [AZURE.NOTE]
> 如果使用的是通过企业 CA 解决方案颁发的证书，并且无法进行身份验证，请检查客户端证书上的身份验证顺序。 可以通过双击客户端证书，并转到“详细信息”>“增强型密钥用法”来检查身份验证列表顺序。 请确保此列表显示的第一项是“客户端身份验证”。 如果不是，则需要基于将“客户端身份验证”作为列表中第一项的用户模板颁发客户端证书。 
>
>

### <a name="part-4-verify-the-vpn-connection"></a>第 4 部分：验证 VPN 连接
1. 若要验证你的 VPN 连接是否处于活动状态，请打开提升的命令提示符，然后运行 *ipconfig/all*。
2. 查看结果。 请注意，你收到的 IP 地址是点到站点连接地址范围中的一个地址，该范围是你在创建 VNet 时指定的。 结果应大致如下所示：

示例：

    PPP adapter VNet1:
        Connection-specific DNS Suffix .:
        Description.....................: VNet1
        Physical Address................:
        DHCP Enabled....................: No
        Autoconfiguration Enabled.......: Yes
        IPv4 Address....................: 192.168.130.2(Preferred)
        Subnet Mask.....................: 255.255.255.255
        Default Gateway.................:
        NetBIOS over Tcpip..............: Enabled

## <a name="faq"></a>点到站点常见问题解答

[AZURE.INCLUDE [Point-to-Site FAQ](../../includes/vpn-gateway-point-to-site-faq-include.md)]

## <a name="next-steps"></a>后续步骤

连接完成后，即可将虚拟机添加到虚拟网络。 有关详细信息，请参阅[虚拟机](/documentation/services/virtual-machines/)。 若要详细了解网络和虚拟机，请参阅 [Azure 和 Linux VM 网络概述](/documentation/articles/virtual-machines-linux-azure-vm-network-overview/)。
<!--Update_Description: wording update-->