<!-- need to be verified -->

<properties
    pageTitle="将 Linux 故障排除 VM 与 CLI 配合使用 | Azure"
    description="了解如何通过使用 Azure CLI 将 OS 磁盘连接到恢复 VM 来排查 Linux VM 问题"
    services="virtual-machines-linux"
    documentationCenter=""
    authors="iainfoulds"
    manager="timlt"
    editor="" />
<tags 
    ms.service="virtual-machines-linux"
    ms.devlang="na"
    ms.topic="article"
    ms.tgt_pltfrm="vm-linux"
    ms.workload="infrastructure"
    ms.date="11/14/2016"
    wacn.date="12/20/2016"
    ms.author="iainfou" />

# 通过使用 Azure CLI 将 OS 磁盘附加到恢复 VM 来对 Linux VM 进行故障排除
如果 Linux 虚拟机 (VM) 遇到启动或磁盘错误，则可能需要对虚拟硬盘本身执行故障排除步骤。一个常见示例是 `/etc/fstab` 中存在无效条目，使 VM 无法成功启动。本文详细介绍如何使用 Azure CLI 将虚拟硬盘连接到另一个 Linux VM，以修复任何错误，然后重新创建原始 VM。


## 恢复过程概述
故障排除过程如下所示：

1. 删除遇到问题的 VM，保留虚拟硬盘。
2. 将虚拟硬盘附加并装入到另一个 Linux VM，以便进行故障排除。
3. 连接到故障排除 VM。编辑文件或运行任何工具以修复原始虚拟硬盘上的问题。
4. 从故障排除 VM 卸载并分离虚拟硬盘。
5. 使用原始虚拟硬盘创建 VM。

确保已登录 [Azure CLI](/documentation/articles/xplat-cli-install/) 并使用 Resource Manager 模式：

    azure config mode arm

在以下示例中，请将参数名称替换为你自己的值。示例参数名称包括 `myResourceGroup`、`mystorageaccount` 和 `myVM`。


## 确定启动问题
检查串行输出以确定 VM 不能正常启动的原因。一个常见示例是 `/etc/fstab` 中存在无效条目，或底层虚拟硬盘已删除或移动。

以下示例从名为 `myResourceGroup` 的资源组中名为 `myVM` 的 VM 获取串行输出：

    azure vm get-serial-output --resource-group myResourceGroup --name myVM

检查串行输出，以确定 VM 无法启动的原因。如果串行输出未提供任何指示，则在将虚拟硬盘连接到故障排除 VM 后，可能需要查看 `/var/log` 中的日志文件。


## 查看现有虚拟硬盘的详细信息
在将虚拟硬盘附加到另一个 VM 之前，需要标识虚拟硬盘 (VHD) 的名称。

以下示例从名为 `myResourceGroup` 的资源组中获取名为 `myVM` 的 VM 的信息：

    azure vm show --resource-group myResourceGroup --name myVM

在上述命令的输出中查找 `Vhd URI`。以下截取的示例输出在最后一行显示 `Vhd URI`：

    info:    Executing command vm show
    + Looking up the VM "myVM"
    + Looking up the NIC "myNic"
    + Looking up the public ip "myPublicIP"
    ...
    data:
    data:      OS Disk:
    data:        OSType                      :Linux
    data:        Name                        :myVM
    data:        Caching                     :ReadWrite
    data:        CreateOption                :FromImage
    data:        Vhd:
    data:          Uri                       :https://mystorageaccount.blob.core.chinacloudapi.cn/vhds/myVM201610292712.vhd

## 删除现有 VM
虚拟硬盘和 VM 是 Azure 中两个不同的资源。虚拟硬盘是操作系统本身，存储应用程序和配置。VM 本身只是定义大小或位置的元数据，并引用虚拟硬盘或虚拟网络接口卡 (NIC) 等资源。每个虚拟硬盘在附加到 VM 时分配有一个租约。虽然即使在 VM 正在运行时，也可以附加和分离数据磁盘，但是除非删除 VM 资源，否则无法分离 OS 磁盘。即使 VM 处于停止和解除分配状态，租约也继续将 OS 磁盘与 VM 相关联。

恢复 VM 的第一步是删除 VM 资源本身。删除 VM 时会将虚拟硬盘留在存储帐户中。删除 VM 后，可将虚拟硬盘附加到另一个 VM，以进行故障排除和解决这些错误。

以下示例从名为 `myResourceGroup` 的资源组中删除名为 `myVM` 的 VM：

    azure vm delete --resource-group myResourceGroup --name myVM 

等到 VM 已完成删除，然后再将虚拟硬盘附加到另一个 VM。虚拟硬盘上将其与 VM 关联的租约需要释放，然后才能将虚拟硬盘附加到另一个 VM。


## 将现有虚拟硬盘附加到另一个 VM
在后续几个步骤中，将使用另一个 VM 进行故障排除。将现有虚拟硬盘附加到此故障排除 VM，以浏览和编辑磁盘的内容。例如，此过程允许用户更正任何配置错误或者查看其他应用程序或系统日志文件。选择或创建另一个 VM 以用于故障排除。

附加现有的虚拟硬盘时，指定在前面的 `azure vm show` 命令中获取的磁盘 URL。以下示例将现有的虚拟硬盘附加到名为 `myResourceGroup` 的资源组中名为 `myVMRecovery` 的故障排除 VM：

    azure vm disk attach --resource-group myResourceGroup --name myVMRecovery \
        --vhd-url https://mystorageaccount.blob.core.chinacloudapi.cn/vhds/myVM.vhd

## 装载附加的数据磁盘

1. 使用相应的凭据通过 SSH 连接到故障排除 VM。如果此磁盘是附加到故障排除 VM 的第一个数据磁盘，则此磁盘可能已连接到 `/dev/sdc`。使用 `dmseg` 查看附加的磁盘：

        dmesg | grep SCSI

    输出类似于以下示例：

        [    0.294784] SCSI subsystem initialized
        [    0.573458] Block layer SCSI generic (bsg) driver version 0.4 loaded (major 252)
        [    7.110271] sd 2:0:0:0: [sda] Attached SCSI disk
        [    8.079653] sd 3:0:1:0: [sdb] Attached SCSI disk
        [ 1828.162306] sd 5:0:0:0: [sdc] Attached SCSI disk

    在前面的示例中，OS 磁盘位于 `/dev/sda`，为每个 VM 提供的临时磁盘位于 `/dev/sdb`。如果有多个数据磁盘，它们应位于 `/dev/sdd`、`/dev/sde`，依次类推。

2. 创建一个目录来装载现有的虚拟硬盘。以下示例创建一个名为 `troubleshootingdisk` 的目录：

        sudo mkdir /mnt/troubleshootingdisk

3. 如果现有的虚拟硬盘上有多个分区，则装载所需的分区。以下示例在 `/dev/sdc1` 中装载第一个主分区：

        sudo mount /dev/sdc1 /mnt/troubleshootingdisk

    > [AZURE.NOTE]
    最佳做法是使用虚拟硬盘的全局唯一标识符 (UUID) 装载 Azure 中 VM 上的数据磁盘。对于此简短的故障排除方案，不必要使用 UUID 装载虚拟硬盘。但是，在正常使用时，编辑 `/etc/fstab` 以使用设备名称（而不是 UUID）装载虚拟硬盘可能会导致 VM 无法启动。


## 修复原始虚拟硬盘上的问题
装载现有虚拟硬盘后，现在可以根据需要执行任何维护和故障排除步骤。解决问题后，请继续执行以下步骤。


## 卸载并分离原始虚拟硬盘
解决错误后，可从故障排除 VM 中卸载并分离现有虚拟硬盘。在将虚拟硬盘附加到故障排除 VM 的租约释放前，不能将该虚拟硬盘用于任何其他 VM。

1. 通过 SSH 会话登录到故障排除 VM 中，卸载现有的虚拟硬盘。首先更改出装入点的父目录：

        cd /

    现在卸载现有的虚拟硬盘。以下示例卸载 `/dev/sdc1` 中的设备：

        sudo umount /dev/sdc1

2. 现在从 VM 中分离虚拟硬盘。退出故障排除 VM 的 SSH 会话。在 Azure CLI 中，首先列出附加到故障排除 VM 的数据磁盘。以下示例列出附加到名为 `myResourceGroup` 的资源组中名为 `myVMRecovery` 的 VM 的数据磁盘：

        azure vm disk list --resource-group myResourceGroup --vm-name myVMRecovery

    记下现有虚拟硬盘的 `Lun` 值。以下示例命令输出显示在 LUN 0 附加的现有虚拟磁盘：

        info:    Executing command vm disk list
        + Looking up the VM "myVMRecovery"
        data:    Name              Lun  DiskSizeGB  Caching  URI
        data:    ------            ---  ----------  -------  ------------------------------------------------------------------------
        data:    myVM              0                None     https://mystorageaccount.blob.core.chinacloudapi.cn/vhds/myVM.vhd
        info:    vm disk list command OK

    使用适当的 `Lun` 值将数据磁盘从 VM 中分离：

        azure vm disk detach --resource-group myResourceGroup --vm-name myVMRecovery \
            --lun 0

## 从原始硬盘创建 VM
若要从原始虚拟硬盘创建 VM，请使用[此 Azure Resource Manager 模板](https://github.com/Azure/azure-quickstart-templates/tree/master/201-vm-specialized-vhd-existing-vnet)。实际的 JSON 模板位于以下链接中：

- https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/201-vm-specialized-vhd-existing-vnet/azuredeploy.json  


该模板使用先前命令中的 VHD URL 将 VM 部署到现有虚拟网络中。以下示例将模板部署到名为 `myResourceGroup` 的资源组：

    azure group deployment create --resource-group myResourceGroup --name myDeployment \
        --template-uri https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/201-vm-specialized-vhd-existing-vnet/azuredeploy.json

响应有关模板的提示，例如 VM 名称（以下示例中为 `myDeployedVM`）、OS 类型 (`Linux`) 和 VM 大小 (`Standard_DS1_v2`)。`osDiskVhdUri` 与前面将现有虚拟硬盘附加到故障排除 VM 时使用的相同。命令输出和提示的示例如下所示：

    info:    Executing command group deployment create
    info:    Supply values for the following parameters
    vmName:  myDeployedVM
    osType:  Linux
    osDiskVhdUri:  https://mystorageaccount.blob.core.chinacloudapi.cn/vhds/myVM201610292712.vhd
    vmSize:  Standard_DS1_v2
    existingVirtualNetworkName:  myVnet
    existingVirtualNetworkResourceGroup:  myResourceGroup
    subnetName:  mySubnet
    dnsNameForPublicIP:  mypublicipdeployed
    + Initializing template configurations and parameters
    + Creating a deployment
    info:    Created template deployment "mydeployment"
    + Waiting for deployment to complete
    +

## 重新启用启动诊断

从现有虚拟硬盘创建 VM 时，启动诊断可能不会自动启用。以下示例在名为 `myResourceGroup` 的资源组中名为 `myDeployedVM` 的 VM 上启用诊断扩展：

    azure vm enable-diag --resource-group myResourceGroup --name myDeployedVM

## 后续步骤
如果在连接到 VM 时遇到问题，请参阅 [Troubleshoot SSH connections to an Azure VM](/documentation/articles/virtual-machines-linux-troubleshoot-ssh-connection/)（排查 Azure VM 的 SSH 连接问题）。有关访问 VM 上运行的应用时遇到的问题，请参阅 [Troubleshoot application connectivity issues on a Linux VM](/documentation/articles/virtual-machines-linux-troubleshoot-app-connection/)（排查 Linux VM 上的应用程序连接问题）。

<!---HONumber=Mooncake_1212_2016-->