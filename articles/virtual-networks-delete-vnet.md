<properties 
   pageTitle="如何删除虚拟网络 (VNet)"
   description="了解如何删除现有的 VNet"
   services="virtual-network"
   documentationCenter="na"
   authors="telmosampaio"
   manager="carolz"
   editor="tysonn"/>
<tags 
   ms.service="virtual-network"
   ms.date="05/29/2015"
   wacn.date="08/01/2015"/>

# 如何删除虚拟网络 (VNet)

如果你要删除 VNet，不能只是单击“删除”。你需要先完成几项操作：

1. **保存设置（可选）**– 如果要将虚拟网络的设置保存到一个本地文件，请在删除虚拟网络之前导出配置文件。<!--有关详细信息，请参阅[将虚拟网络设置导出到网络配置文件](https://msdn.microsoft.com/zh-cn/library/azure/dn133804.aspx)。-->保存设置可在将来根据需要重新创建 VNet。

2. **删除虚拟网络网关** – 如果为 VNet 配置了网关，需要在删除 VNet 之前删除该网关。若要删除虚拟网络网关，请转到虚拟网络的“仪表板”页。在页面底部，单击“删除网关”。
						
	可能必须等待 5-10 分钟以删除网关，然后再继续执行后续步骤。

3. **删除云服务、网站和 VM** – 如果你已将任何内容部署到 VNet，则需要删除这些内容，然后才能删除 VNet。

4. **删除虚拟网络** – 单击“名称”行的右侧以突出显示要删除的 VNet，然后单击页底部的“删除”。按照屏幕上的说明操作。

5. **此外** – 还可在删除虚拟网络后删除任何本地网络设置、DNS 服务器和地缘组。
 

<!---HONumber=64-->