<!-- Ibiza portal: tested -->

<properties 
   pageTitle="如何使用 Azure 门户预览在经典模式下设置静态专用 IP | Azure"
   description="了解静态专用 IP 以及如何使用 Azure 门户预览在经典模式下管理它们"
   services="virtual-network"
   documentationCenter="na"
   authors="telmosampaio"
   manager="carmonm"
   editor="tysonn"
   tags="azure-service-management"
/>
<tags
	ms.service="virtual-network"
	ms.date="02/04/2016"
	wacn.date="07/04/2016"/>

# 如何在 Azure 门户预览中设置静态专用 IP 地址（经典）

[AZURE.INCLUDE [virtual-networks-static-private-ip-selectors-classic-include](../includes/virtual-networks-static-private-ip-selectors-classic-include.md)]

[AZURE.INCLUDE [virtual-networks-static-private-ip-intro-include](../includes/virtual-networks-static-private-ip-intro-include.md)]

[AZURE.INCLUDE [azure-arm-classic-important-include](../includes/azure-arm-classic-important-include.md)]本文介绍经典部署模型。你还可以[管理资源管理器部署模型中的静态专用 IP 地址](/documentation/articles/virtual-networks-static-private-ip-arm-pportal/)。

[AZURE.INCLUDE [virtual-networks-static-ip-scenario-include](../includes/virtual-networks-static-ip-scenario-include.md)]

下面的示例步骤需要已创建简单的环境。如果你想要运行本文档中所显示的步骤，首先需要生成[创建 Vnet](/documentation/articles/virtual-networks-create-vnet-classic-pportal/) 中所述的测试环境。

## 如何在创建 VM 时指定静态专用 IP 地址
若要在名为 TestVNet 的 VNet 的 FrontEnd 子网中使用静态专用 IP 192.168.1.101 创建名为 DNS01 的 VM，请按照以下步骤进行操作：

1. 从浏览器导航到 http://portal.azure.cn, 如有必要，请使用 Azure 帐户登录。
2. 单击“新建”>“计算”>“Windows Server 2012 R2 数据中心”（注意“选择部署模型”列表已显示“经典”），然后单击“创建”。

	![在 Azure 门户预览中创建 VM](./media/virtual-networks-static-ip-classic-pportal/figure01.png)

3. 在“创建 VM”边栏选项卡中，输入要创建的 VM 的名称（在我们的方案中为 DNS01）、本地管理员帐户和密码。

	![在 Azure 门户预览中创建 VM](./media/virtual-networks-static-ip-classic-pportal/figure02.png)

4. 单击“可选配置”>“网络”>“虚拟网络”，然后单击“TestVNet”。如果 **TestVNet** 不可用，请确保你使用的位置为“中国北部”，并已创建本文开头所述的测试环境。

	![在 Azure 门户预览中创建 VM](./media/virtual-networks-static-ip-classic-pportal/figure03.png)

5. 在“网络”边栏选项卡中，确保当前选定的子网为“前端”，然后单击“IP 地址”，在“IP 地址分配”下单击“静态”，然后输入 192.168.1.101 作为“IP 地址”，如下所示。

	![在 Azure 门户预览中创建 VM](./media/virtual-networks-static-ip-classic-pportal/figure04.png)

6. 单击“IP 地址”边栏选项卡中的“确定”，然后单击“网络”边栏选项卡中的“确定”，再单击“可选配置”边栏选项卡中的“确定”。
7. 在“创建 VM”边栏选项卡中，单击“创建”。注意，以下磁贴将显示在仪表板中。

	![在 Azure 门户预览中创建 VM](./media/virtual-networks-static-ip-classic-pportal/figure05.png)

## 如何检索 VM 的静态专用 IP 地址信息

若要查看使用以上步骤创建的 VM 的静态专用 IP 地址信息，请执行以下步骤。

1. 在 Azure 门户预览中，单击“浏览全部”>“虚拟机（经典）”>“DNS01”>“所有设置”>“IP 地址”，然后注意到如下所示的 IP 地址分配和 IP 地址。

	![在 Azure 门户预览中创建 VM](./media/virtual-networks-static-ip-classic-pportal/figure06.png)

## 如何从 VM 中删除静态专用 IP 地址
若要从上面创建的 VM 中删除静态专用 IP 地址，请按照以下步骤操作。
	
1. 从上面所示的“IP 地址”边栏选项卡中，单击“IP 地址分配”右侧的“动态”，然后单击“保存”，然后单击“是”。

	![在 Azure 门户预览中创建 VM](./media/virtual-networks-static-ip-classic-pportal/figure07.png)

## 如何将静态专用 IP 地址添加到现有 VM
若要将静态专用 IP 地址添加到使用上面步骤创建的 VM 中，请按照以下步骤操作：

1. 从上面所示的“IP 地址”边栏选项卡中，单击“IP 地址分配”右侧的“静态”。
2. 键入 192.168.1.101 作为“IP 地址”，然后单击“保存”，然后单击“是”。

## 后续步骤

- 了解[保留公共 IP](/documentation/articles/virtual-networks-reserved-public-ip/) 地址。
- 了解[实例层级公共 IP (ILPIP)](/documentation/articles/virtual-networks-instance-level-public-ip/) 地址。
- 查阅[保留 IP REST API](https://msdn.microsoft.com/zh-cn/library/azure/dn722420.aspx)。


<!---HONumber=Mooncake_0418_2016-->