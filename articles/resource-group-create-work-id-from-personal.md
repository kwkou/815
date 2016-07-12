<properties
   pageTitle="在 AAD 中创建工作或学校标识 | Azure"
   description="了解如何在 Azure Active Directory 中创建工作或学校标识以配合 Linux 虚拟机使用。"
   services="virtual-machines-linux"
   documentationCenter=""
   authors="squillace"
   manager="timlt"
   editor=""
   tags="azure-service-management,azure-resource-manager"/>

<tags
   ms.service="virtual-machines-linux"
   ms.date="12/08/2015"
   wacn.date="05/24/2016"/>

# 在 Azure Active Directory 中创建工作或学校标识以配合 Linux 虚拟机使用

如果你创建了个人 Azure 帐户 -- 你使用 *Azure.cn 帐户* 标识来创建。Azure 的许多强大功能 - [资源组模板](/documentation/articles/resource-group-overview/)就是一个例子 - 需要有工作或学校帐户（由 Azure Active Directory 管理的标识）才能运行。你可以遵循以下说明创建新的工作帐户或学校帐户，因为在个人 Azure 帐户方面，最有利的特点之一是，这种帐户附带了一个默认 Azure Active Directory 域，使用该域可以创建新的工作或学校帐户，以用于需要这种帐户的 Azure 功能。

此外，利用最近所做的更改，你可以使用[此处](/documentation/articles/xplat-cli-connect/)所述的 `azure login -e AzureChinaCloud` 交互式登录方法，通过任何类型的 Azure 帐户管理你的订阅。你可以使用该机制，也可以遵循后面的说明。你也可[在 Azure 活动目录创建一个工作或学校账户来使用 Windows VM](/documentation/articles/virtual-machines-windows-create-aad-work-id/)。

[AZURE.INCLUDE [了解部署模型](../includes/learn-about-deployment-models-both-include.md)]

[AZURE.INCLUDE [virtual-machines-common-create-aad-work-id](../includes/virtual-machines-common-create-aad-work-id.md)]

<!---HONumber=Mooncake_0118_2016-->