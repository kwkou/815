<properties
    pageTitle="使用版本 1.0.0 的 Azure ARM SDK for Java 创建虚拟机时报错"
    description="使用版本 1.0.0 的 Azure ARM SDK for Java 创建虚拟机时报错"
    service=""
    resource="virtualmachines"
    authors="Miley Chen"
    displayOrder=""
    selfHelpType=""
    supportTopicIds=""
    productPesIds=""
    resourceTags="Virtual Machines, ARM, SDK, Java"
    cloudEnvironments="MoonCake" />
<tags
    ms.service="virtual-machines-aog"
    ms.date=""
    wacn.date="05/22/2017" />

# 使用版本 1.0.0 的 Azure ARM SDK for Java 创建虚拟机时报错

## **问题描述**

我们可以通过使用 Azure ARM SDK 来管理 Azure 上的资源，因此我们也可以通过 SDK 来创建 ARM 类型的虚拟机，当我们使用 1.0.0 版本的 Azure SDK for Java 来创建 ARM 虚拟时，会遇到如下错误：

    com.microsoft.azure.CloudException: Managed Disks are not supported in this region。

## **问题分析**

使用版本 1.0.0 之前的 SDK 如 -beta3 来创建虚拟机是使用基于 storage account (OS and Data) disk，但是随后有了 managed disks，因此 1.0.0 版本的 SDK 是通过 managed disks 来创建的虚拟机。

目前中国区域的 1.0.0 版本的 SDK 还不支持 managed disks，所以出现了下面的报错。我们目前正在积极推进相关功能在中国区域的上线。

## **解决方案**

可以通过使用 `withUnmanagedDisks()` 来定义创建虚拟机，可以参考[链接](https://github.com/Azure-Samples/compute-java-manage-virtual-machine-with-unmanaged-disks/blob/master/src/main/java/com/microsoft/azure/management/compute/samples/ManageVirtualMachineWithUnmanagedDisks.java#L72)。

例如：

    VirtualMachine windowsVM = azure.virtualMachines().define(vmName)
                    .withRegion(vmRegion)
                    .withNewResourceGroup(resourceGroupName)
                    .withNewPrimaryNetwork("10.0.0.0/28")
                    .withPrimaryPrivateIPAddressDynamic()
                    .withoutPrimaryPublicIPAddress()
                    .withPopularWindowsImage(KnownWindowsVirtualMachineImage.WINDOWS_SERVER_2012_R2_DATACENTER)
                    .withAdminUsername(vmUserName)
                    .withAdminPassword(vmPassword)
                    .withUnmanagedDisks()
                    .withSize(VirtualMachineSizeTypes.STANDARD_D3_V2)
                    .create();