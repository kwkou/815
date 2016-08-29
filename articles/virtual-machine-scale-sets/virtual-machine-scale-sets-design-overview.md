<properties
	pageTitle="设计规模化的虚拟机规模集 | Azure"
	description="了解如何设计规模化的虚拟机规模集"
	keywords="linux 虚拟机, 虚拟机缩放集" 
	services="virtual-machine-scale-sets"
	documentationCenter=""
	authors="gatneil"
	manager="madhana"
	editor="tysonn"
	tags="azure-resource-manager" />

<tags
	ms.service="virtual-machine-scale-sets"
	ms.date="07/28/2016"
	wacn.date="08/29/2016"/>  


# 设计规模化的 VM 规模集

本主题讨论虚拟机规模集的设计注意事项。有关什么是虚拟机规模集的信息，请参阅[虚拟机规模集概述](/documentation/articles/virtual-machine-scale-sets-overview/)。


## 存储

规模集使用存储帐户来存储集中 VM 的操作系统磁盘。我们建议每个存储帐户采用 20 台 VM 或更低的比率。我们还建议存储帐户名称的开始字符采用字母表的字符。这有助于在多个不同的内部系统中分散负载。例如，在下面的模板中，我们使用 uniqueString ARM 模板函数来生成附加到存储帐户名称的前缀哈希：[https://github.com/Azure/azure-quickstart-templates/tree/master/201-vmss-linux-nat](https://github.com/Azure/azure-quickstart-templates/tree/master/201-vmss-linux-nat)。


## 预配过度

从 2016-03-30 API 版本开始，VM 规模集默认设置为“预配过度”VM。这意味着，规模集实际启动了更多的 VM，然后删除上一次启动的额外 VM。这提高了预配成功率，因为如果一个 VM 都未设置成功，Azure Resource Manager 将认为整个部署“失败”。不会对这些额外的 VM 计费，并且不将其计入配额限制。

这样可以改进预配成功率，但会导致应用程序（其设计目的不是处理突然消失的 VM）行为混乱。为了关闭预配过度，请确保您的模板中包含以下字符串："overprovision": "false"

如果关闭预配过度，每个存储帐户中可以得到更大的 VM 比率，但我们不建议此比率高于 40。


## 限制
基于自定义映像的（由您生成的）规模集必须在一个存储帐户中创建所有操作系统磁盘 VHD。因此，基于自定义映像的规模集中 VM 的最大建议数目为 20。如果关闭预配过度，最大可为 40。

基于平台映像的规模集当前被限制为 100 台 VM（我们建议为此规模采用 5 个存储帐户）。

对于高出这些限制所允许的 VM，需要部署多个规模集。[有关如何执行此操作的示例，请参阅此模板。](https://github.com/Azure/azure-quickstart-templates/tree/master/301-custom-images-at-scale)

<!---HONumber=Mooncake_0822_2016-->