<properties
    pageTitle="管理经典虚拟网络所用的 DNS 服务器 - Azure 门户（经典）| Azure"
    description="了解如何使用 Azure 门户（经典）添加和删除经典虚拟网络中的 DNS 服务器。"
    services="virtual-network"
    documentationcenter="na"
    author="jimdial"
    manager="timlt"
    editor="tysonn" />
<tags
    ms.assetid="b582be7d-dc78-4cfe-a766-185bd7e5dc68"
    ms.service="virtual-network"
    ms.devlang="na"
    ms.topic="article"
    ms.tgt_pltfrm="na"
    ms.workload="infrastructure-services"
    ms.date="03/15/2016"
    wacn.date="03/31/2017"
    ms.author="jdial"
    ms.custom="H1Hack27Feb2017" />  


# 通过 Azure 门户（经典），管理经典虚拟网络使用的 DNS 服务器

可在 Azure 门户（经典）或网络配置文件中，管理虚拟网络 (VNet) 中所用的 DNS 服务器的列表。对于每个 VNet，最多可以添加 12 个 DNS 服务器。指定 DNS 服务器时，请确认按所处环境的正确顺序列出 DNS 服务器，这一点很重要。DNS 服务器列表不采用循环机制。将按指定服务器的顺序使用这些服务器。如果可访问列表上的第一个 DNS 服务器，则无论该 DNS 服务器是否工作正常，客户端都将使用它。若要更改你的虚拟网络的 DNS 服务器顺序，请从列表中删除 DNS 服务器，然后按你所需的顺序重新添加这些服务器。

> [AZURE.WARNING]
更新 DNS 列表后，必须重新启动位于虚拟网络中的虚拟机，以使其选取新的 DNS 服务器设置。虚拟机将继续使用其当前配置，直至其重新启动为止。
> 
> 

## 通过 Azure 门户（经典），编辑虚拟网络的 DNS 服务器列表
1. 登录到 [Azure 门户（经典）](https://manage.windowsazure.cn)。
2. 在导航窗格中，单击“网络”，然后在“名称”列中单击你的虚拟网络的名称。
3. 单击**“配置”**。
4. 在“DNS 服务器”中，你可以进行以下配置：
   
    * **注册（添加）新的 DNS 服务器 - **只需在框中键入名称和 IP 地址。这样可将 DNS 服务器添加到虚拟网络 DNS 服务器列表，并向 Azure 注册 DNS 服务器。
    * **添加以前注册的 DNS 服务器 - **如果已向 Azure 注册了 DNS 服务器，则可从预填充列表中选择该服务器。
    * **从虚拟网络中删除 DNS 服务器 - **单击要删除的服务器旁边的 X。请注意，这样只能将服务器从此虚拟网络列表中删除。DNS 服务器在 Azure 中仍为注册状态，可供其他虚拟网络使用。若要从订阅中删除 DNS 服务器，请转到“网络”->“DNS 服务器”页面。
    * **重新排列 DNS 服务器的顺序 - **删除列出的所有 DNS 服务器，然后按照所需顺序将其重新添加到列表中。请记住，这不是循环 DNS 列表。
    * **重命名 DNS 服务器 - **在列表中突出显示 DNS 服务器，然后键入新名称。这样将在 Azure 中注册一个新的 DNS 服务器，并将其添加到虚拟网络的 DNS 服务器列表。旧的 DNS 服务器及其 IP 地址在 Azure 中仍将保持注册状态。如果你不将它用于任何其他虚拟网络，可在“DNS 服务器”页上删除它。
5. 在页面底部单击“保存”以保存新的 DNS 服务器配置。
6. 重新启动位于虚拟网络中的虚拟机以使其可获取新的 DNS 设置。

## 使用网络配置文件编辑 DNS 服务器列表
若要使用网络配置文件编辑 DNS 服务器列表，首先需从经典管理门户中导出配置设置。然后编辑网络配置文件，再通过 Azure 门户（经典）将其导入原先位置。下面概括列出了用于完成此过程的步骤。

1. 将虚拟网络设置导出到网络配置文件。有关导出网络配置设置的详细信息和步骤，请参阅[将虚拟网络设置导出到网络配置文件](/documentation/articles/virtual-networks-using-network-configuration-file/)。
2. 指定虚拟网络的 DNS 服务器信息。有关指定 DNS 服务器的详细信息，请参阅[在虚拟网络配置文件中指定 DNS 服务器](/documentation/articles/virtual-networks-specifying-a-dns-settings-in-a-virtual-network-configuration-file/)。有关网络配置文件的其他信息，请参阅 [Azure 虚拟网络配置架构](https://msdn.microsoft.com/zh-cn/library/azure/jj157100.aspx)和[使用网络配置文件配置虚拟网络](/documentation/articles/virtual-networks-using-network-configuration-file/)。
3. 导入网络配置文件。有关导入网络配置文件的详细信息和步骤，请参阅[导入网络配置文件](/documentation/articles/virtual-networks-using-network-configuration-file/)。
4. 重新启动位于虚拟网络中的虚拟机以使其可获取新的 DNS 设置。

<!---HONumber=Mooncake_0327_2017-->
<!--Update_Description: wording update-->