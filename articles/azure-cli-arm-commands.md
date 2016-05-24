<properties
	pageTitle="配合使用 Azure CLI 和资源管理器 | Azure"
	description="了解如何使用适用于 Mac、Linux 和 Windows 的 Azure CLI，在 Azure 资源管理器模式下管理 Azure 资源。"
	services="virtual-machines,virtual-network,mobile-services,cloud-services"
	documentationCenter=""
	authors="dlepow"
	manager="timlt"
	editor=""
	tags="azure-resource-manager"/>

<tags
	ms.service="multiple"
	ms.date="03/07/2016"
	wacn.date="05/24/2016"/>

# 将适用于 Mac、Linux 和 Windows 的 Azure CLI 与 Azure 资源管理器配合使用

[AZURE.INCLUDE [了解部署模型](../includes/learn-about-deployment-models-rm-include.md)]

本文介绍如何在 Azure 资源管理器模式下使用 Azure 命令行接口 (Azure CLI)，在 Mac、Linux 和 Windows 计算机的命令行中创建、管理和删除服务。你可以使用 Azure SDK 的各种库、Azure PowerShell 和 Azure 门户执行许多相同的任务。

Azure 资源管理器可让你创建一组资源 - 网站、数据库等 - 作为单个可部署单元。然后，可以通过一个协调的操作为应用程序部署、更新或删除所有资源。在部署的 JSON 模板中描述组资源，然后，可以针对不同的环境（如测试、过渡和生产）使用该模板。

## 本文的讨论范围

本文提供了用于资源管理器部署模型的常用 Azure CLI 命令的语法和选项。它并不是完整的参考，并且你的 CLI 版本可能会显示某些不同的命令或参数。要在资源管理器模式下在命令行中查看当前的命令语法和选项，请键入 `azure help`；要显示某个命令的帮助，请键入 `azure help [command]`。你还可以在创建和管理具体 Azure 服务的说明文档中找到 CLI 示例。

可选参数显示在方括号中（例如，[参数]）。其他所有参数都是必需的。

除了此处记录的特定于命令的可选参数外，还有三个可用于显示详细输出（例如请求选项和状态代码）的可选参数。-v 参数提供详细输出，而 -vv 参数提供更详细的输出。--json 选项将以原始的 json 格式输出结果。使用 --json 开关的情况很常见，在获取和了解返回资源信息、状态和日志的 Azure CLI 操作的结果以及使用模板时，该开关非常重要。你可能想要安装 JSON 分析器工具（如 **jq** 或 **jsawk**）或使用你偏爱的语言库。

## 命令性和声明性方法

与 [Azure 服务管理模式](/documentation/articles/virtual-machines-command-line-tools)一样，Azure CLI 的资源管理器模式可提供命令让你在命令行上强制创建资源。例如，如果键入 `azure group create <groupname> <location>`，则会要求 Azure 创建资源组；如果键入 `azure group deployment create <resourcegroup> <deploymentname>`，则会指示 Azure 创建包含任意项数的部署，并将其放在组中。由于每种类型的资源都有强制命令，你可以将这些命令链接在一起，以创建相当复杂的部署。

但是，使用用于描述资源组的资源组_模板_是一种强大得多的声明性方法，它允许你针对（几乎）任何目的自动完成包含（几乎）任意数量的资源的复杂部署。使用模板时，唯一的强制性命令是单一部署。有关模板、资源和资源组的一般概述，请参阅 [Azure 资源组概述](/documentation/articles/resource-group-overview)。

##用法要求

对配合使用资源管理器模式和 Azure CLI 的设置要求如下：

- 一个 Azure 帐户（[在此处获取试用版](/pricing/1rmb-trial/)）
- [安装 Azure CLI](/documentation/articles/xplat-cli-install)


获取帐户并安装 Azure CLI 后，你必须

- [配置 Azure CLI](/documentation/articles/xplat-cli-connect) 以使用工作或学校帐户或 Microsoft 帐户标识
- 通过键入 `azure config mode arm` 切换到资源管理器模式。


## azure account：管理你的帐户信息
该工具使用你的 Azure 订阅信息连接到你的帐户。

**列出导入的订阅**

	account list [options]

**显示有关订阅的详细信息**

	account show [options] [subscriptionNameOrId]

**设置当前订阅**

	account set [options] <subscriptionNameOrId>

**删除订阅或环境，或清除所有存储的帐户和环境信息**

	account clear [options]

**用于管理帐户环境的命令**

	account env list [options]
	account env show [options] [environment]
	account env add [options] [environment]
	account env set [options] [environment]
	account env delete [options] [environment]

## azure ad：用于显示 Active Directory 对象的命令

**用于显示 Active Directory 应用程序的命令**

	ad app create [options]
	ad app delete [options] <object-id>

**用于显示 Active Directory 组的命令**

	ad group list [options]
	ad group show [options]

**用于提供 Active Directory 子组或成员信息的命令**

	ad group member list [options] [objectId]

**用于显示 Active Directory 服务主体的命令**

	ad sp list [options]
	ad sp show [options]
	ad sp create [options] <application-id>
	ad sp delete [options] <object-id>

**用于显示 Active Directory 用户的命令**

	ad user list [options]
	ad user show [options]

## azure availset：用于管理可用性集的命令

**在资源组中创建可用性集**

	availset create [options] <resource-group> <name> <location> [tags]

**列出资源组中的可用性集**

	availset list [options] <resource-group>

**获取资源组中的一个可用性集**

	availset show [options] <resource-group> <name>

**删除资源组中的一个可用性集**

	availset delete [options] <resource-group> <name>

## azure config：用于管理本地设置的命令

**列出 Azure CLI 配置设置**

	conconfig list [options]

**删除配置设置**

	conconfig delete [options] <name>

**更新配置设置**

	conconfig set <name> <value>

**将 Azure CLI 工作模式设置为 arm 或 asm**

	conconfig mode [options] <modename>


## azure feature：用于管理帐户功能的命令

**列出订阅可用的所有功能**

	feature list [options]

**显示一个功能**

	feature show [options] <providerName> <featureName>

**注册资源提供程序的预览功能**

	feature register [options] <providerName> <featureName>

## azure group：用于管理资源组的命令

**创建新的资源组**

	group create [options] <name> <location>

**向资源组设置标记**

	group set [options] <name> <tags>

**删除资源组**

	group delete [options] <name>

**列出订阅的资源组**

	group list [options]

**显示订阅的资源组**

	group show [options] <name>

**用于管理资源组日志的命令**

	group log show [options] [name]

**用于管理资源组中的部署的命令**

	group deployment create [options] [resource-group] [name]
	group deployment list [options] <resource-group> [state]
	group deployment show [options] <resource-group> [deployment-name]
	group deployment stop [options] <resource-group> [deployment-name]

**用于管理本地或库资源组模板的命令**

	group template list [options]
	group template show [options] <name>
	group template download [options] [name] [file]
	group template validate [options] <resource-group>

## azure insights：与监视 Insights（事件、警报规则、自动缩放设置、度量值）相关的命令

**检索订阅、correlationId、资源组、资源或资源提供程序的操作日志**

	insights logs list [options]

## azure location：用于获取所有资源类型的可用位置的命令

**列出可用位置**

	location list [options]

## azure network：用于管理网络资源的命令

在 Azure 中国，VNet 还不能用 ARM 进行管理。有关 VNet 的 Azure CLI ASM 命令，请阅读以下的文章：

- [使用 Azure CLI 创建虚拟网络（经典）](/documentation/articles/virtual-networks-create-vnet-classic-cli)
- [如何在 Azure CLI 中创建 NSG（经典）](/documentation/articles/virtual-networks-create-nsg-classic-cli)
- [使用 Azure CLI 控制路由和使用虚拟设备（经典）](/documentation/articles/virtual-network-create-udr-classic-cli)
- [使用 Azure CLI 部署多 NIC VM（经典）](/documentation/articles/virtual-network-deploy-multinic-classic-cli)
- [如何在 Azure CLI 中设置静态专用 IP 地址（经典）](/documentation/articles/virtual-networks-static-private-ip-classic-cli)

## azure provider：用于管理资源提供程序注册的命令

**列出 ARM 中当前已注册的提供程序**

	provider list [options]

**显示有关请求的提供程序命名空间的详细信息**

	provider show [options] <namespace>

**将提供程序注册到订阅**

	provider register [options] <namespace>

**从订阅中取消注册提供程序**

	provider unregister [options] <namespace>

## azure resource：用于管理资源的命令

**在资源组中创建资源**

	resource create [options] <resource-group> <name> <resource-type> <location> <api-version>

**更新资源组中不带任何模板或参数的资源**

	resource set [options] <resource-group> <name> <resource-type> <properties> <api-version>

**列出资源**

	resource list [options] [resource-group]

**获取资源组或订阅中的一个资源**

	resource show [options] <resource-group> <name> <resource-type> <api-version>

**删除资源组中的资源**

	resource delete [options] <resource-group> <name> <resource-type> <api-version>

## azure role：用于管理 Azure 角色的命令

**获取所有可用的角色定义**

	role list [options]

**获取一个可用的角色定义**

	role show [options] [name]

**用于管理角色分配的命令**

	role assignment create [options] [objectId] [upn] [mail] [spn] [role] [scope] [resource-group] [resource-type] [resource-name]
	role assignment list [options] [objectId] [upn] [mail] [spn] [role] [scope] [resource-group] [resource-type] [resource-name]
	role assignment delete [options] [objectId] [upn] [mail] [spn] [role] [scope] [resource-group] [resource-type] [resource-name]

## azure storage：用于管理存储对象的命令

在 Azure 中国，Azure 存储还不能用 ARM 进行管理。有关 Azure 存储的 Azure CLI ASM 命令，请阅读[使用 Azure CLI 管理 Azure 存储服务](/documentation/articles/storage-azure-cli)

## azure tag：用于管理资源管理器标记的命令

**添加标记**

	tag create [options] <name> <value>

**删除整个标记或标记值**

	tag delete [options] <name> <value>

**列出标记信息**

	tag list [options]

**获取标记**

	tag show [options] [name]

## azure vm：用于管理 Azure 虚拟机的命令

在 Azure 中国，Azure 虚拟机还不能用 ARM 进行管理。有关 Azure 虚拟机的 Azure CLI ASM 命令，请阅读[将适用于 Mac、Linux 和 Windows 的 Azure CLI 与 Azure 服务管理配合使用](/documentation/articles/virtual-machines-command-line-tools)

## azure hdinsight：用于管理 HDInsight 群集的命令

在 Azure 中国，Azure HDInsight 还不能用 ARM 进行管理。有关 Azure HDInsight 的 Azure CLI ASM 命令，请阅读[使用 Azure CLI 管理 HDInsight 中的 Hadoop 群集](/documentation/articles/hdinsight-administer-use-command-line)

<!---HONumber=Mooncake_0118_2016-->