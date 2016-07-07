<properties 
	pageTitle="调整虚拟机的尺寸" 
	description="本文介绍如何调整虚拟机的尺寸" 
	services="virtual machine" 
	documentationCenter="" 
	authors=""
	manager="" 
	editor=""/>
<tags ms.service="virtual-machine-aog" ms.date="" wacn.date="12/18/2015"/>

#调整虚拟机的尺寸

###本文包含以下内容

- [常见问题](#faq)
- [详细步骤](#detail)
 
## <a id="faq"></a>常见问题

- 调整虚拟机尺寸计费怎么算的？

	Azure 是按分钟对虚拟机收费，例如你的虚拟机运行了 1 个小时，但前 45 分钟是 A1 尺寸，后面 15 分钟是 A2 尺寸，则前 45 分钟是按 A1 收费，后 15 分钟是按 A2 收费。

- 修改尺寸后我的虚拟机的 IP 是否会变？

	在修改虚拟机尺寸后虚拟机的 IP 可能会改变，如果业务逻辑中不希望 IP 变化的话，请尝试“保留 IP”，关于更多有关保留 IP 的内容，请阅读：https://www.azure.cn/documentation/articles/virtual-networks-reserved-public-ip

## <a id="detail"></a>详细步骤

1. 虚拟机-(本例)VNET-BJMOSES-01
 
	![](./media/aog-virtual-machine-how-to-reset-vm-size/choose-vm.png)
 
2. 配置-虚拟机大小

	![](./media/aog-virtual-machine-how-to-reset-vm-size/reset-vm-size.png)

**请注意**：在调整虚拟机的尺寸后，虚拟机需要重新启动！
 
