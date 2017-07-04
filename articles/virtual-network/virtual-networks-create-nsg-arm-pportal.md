<properties 
   pageTitle="如何使用门户在 ARM 模式下创建 NSG | Azure"
   description="了解如何使用门户在 ARM 下创建和部署 NSG"
   services="virtual-network"
   documentationCenter="na"
   authors="telmosampaio"
   manager="carolz"
   editor="tysonn"
   tags="azure-resource-manager"
/>
<tags 
   ms.service="virtual-network"
   ms.devlang="na"
   ms.topic="article"
   ms.tgt_pltfrm="na"
   ms.workload="infrastructure-services"
   ms.date="02/04/2016"
   wacn.date="12/16/2016"
   ms.author="jdial" />

# 如何使用门户管理 NSG

[AZURE.INCLUDE [virtual-networks-create-nsg-selectors-arm-include](../../includes/virtual-networks-create-nsg-selectors-arm-include.md)]

[AZURE.INCLUDE [virtual-networks-create-nsg-intro-include](../../includes/virtual-networks-create-nsg-intro-include.md)]

>[AZURE.IMPORTANT]在使用 Azure 资源之前，请务必了解 Azure 当前使用两种部署模型：资源管理器部署模型和经典部署模型。在使用任何 Azure 资源之前，请确保你了解[部署模型和工具](/documentation/articles/azure-classic-rm/)。可以通过单击本文顶部的选项卡来查看不同工具的文档。本文介绍资源管理器部署模型。你还可以[在经典部署模型中创建 NSG](/documentation/articles/virtual-networks-create-nsg-classic-ps/)。

[AZURE.INCLUDE [virtual-networks-create-nsg-scenario-include](../../includes/virtual-networks-create-nsg-scenario-include.md)]

## 通过使用单击部署来部署 ARM 模板

此时不能从预览创建 NSG。但是，可以管理现有 NSG。使用公共存储库中可用的示例模板创建以上方案中所述的资源后，才能管理 NSG。部署[此模板](http://github.com/telmosampaio/azure-templates/tree/master/201-IaaS-WebFrontEnd-SQLBackEnd-NSG)，单击**部署至 Azure**，根据需要替换默认参数值，然后按照门户中的说明进行操作。

## 在现有 NSG 中创建规则

若要从门户中在现有 NSG 中创建规则，请按照下面的步骤操作。

1. 从浏览器导航到 https://portal.azure.cn，如有必要，请使用 Azure 帐户登录。
2. 单击“浏览>”>“网络安全组”。

![门户 - NSG](./media/virtual-networks-create-nsg-arm-pportal/figure1.png)

3. 在 NSG 列表中，单击“NSG-FrontEnd”>“入站安全规则”

![门户 - NSG 前端](./media/virtual-networks-create-nsg-arm-pportal/figure2.png)

4. 在“入站安全规则”列表中，单击“添加”。

![门户 - 添加规则](./media/virtual-networks-create-nsg-arm-pportal/figure3.png)

5. 在“添加入站安全规则”边栏选项卡中创建一个名为 *web-rule* 的规则，其优先级为 *200*，允许端口 *80* 通过 *TCP* 访问来自任何来源的任何 VM，然后单击“确定”。请注意这些设置中大多数已经是默认值。

![门户 - 规则设置](./media/virtual-networks-create-nsg-arm-pportal/figure4.png)

6. 几秒钟后可在 NSG 中看到新规则。

![门户 - 新建规则](./media/virtual-networks-create-nsg-arm-pportal/figure5.png)

<!---HONumber=Mooncake_Quality_Review_1202_2016-->