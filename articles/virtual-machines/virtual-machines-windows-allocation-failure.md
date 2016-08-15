<properties
	pageTitle="排查 Windows VM 分配失败 | Azure"
	description="在 Azure 中创建、重新启动 Windows VM 或调整其大小时排查分配失败"
	services="virtual-machines-windows, azure-resource-manager"
	documentationCenter=""
	authors="jiangchen79"
	manager="felixwu"
	editor=""
	tags="top-support-issue,azure-resource-manager,azure-service-management"/>

<tags
	ms.service="virtual-machines-windows"
	ms.date="02/02/2016"
	wacn.date="05/24/2016"/>



# 在 Azure 中创建、重新启动 Windows VM 或调整其大小时排查分配失败

创建 VM、重新启动已停止（解除分配）的 VM 和重设 VM 大小时，Azure 会为你的订阅分配计算资源。执行这些操作时，即使尚未达到 Azure 订阅限制，也可能偶尔发生错误。本文说明一些常见分配故障的原因，并建议可能的补救方法。规划服务的部署时，本信息可能也有用。你也可以[在 Azure 中创建、重新启动 Linux VM 或调整其大小时排查分配失败](/documentation/articles/virtual-machines-linux-allocation-failure/)。

[AZURE.INCLUDE [virtual-machines-common-allocation-failure](../includes/virtual-machines-common-allocation-failure.md)]

<!---HONumber=Mooncake_0314_2016-->