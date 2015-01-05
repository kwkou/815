<properties linkid="manage-linux-howto-custom-create-vm" urlDisplayName="Custom create a VM" pageTitle="Custom create a virtual machine running Linux in Azure" metaKeywords="Azure custom vm, creating custom vm" description="Learn how to create a custom virtual machine in Azure." metaCanonical="/zh-cn/manage/windows/how-to-guides/custom-create-a-vm/" services="virtual-machines" documentationCenter="" title="" authors="kathydav" solutions="" manager="dongill" editor="tysonn" />


#How to Create a Custom Virtual Machine

A *custom* virtual machine simply means a virtual machine you create using the **From Gallery** option because it gives you more configuration choices than the **Quick Create** option. These choices include:

- Connecting the VM to a virtual network
- Installing the VM Agent and extensions, such as for antimalware 
- Adding the VM to an existing cloud service 
- Adding the VM to an availability set

**Important**: If you want your virtual machine to use a virtual network so you can connect to it directly by hostname or set up cross-premises connections, make sure you specify the virtual network when you create the virtual machine. A virtual machine can be configured to join a virtual network only when you create the virtual machine. For details on virtual networks, see [Azure Virtual Network Overview](http://msdn.microsoft.com/library/azure/jj156007.aspx).

[WACOM.INCLUDE [virtual-machines-create-WindowsVM](../includes/virtual-machines-create-WindowsVM.md)]


