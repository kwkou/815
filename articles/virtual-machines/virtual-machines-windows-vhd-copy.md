<properties
    pageTitle="在 Azure 中创建专用 VM 的副本 | Azure"
    description="了解如何在 Resource Manager 部署模型中，为 Azure 中运行的专用 Windows VM 创建副本。"
    services="virtual-machines-windows"
    documentationcenter=""
    author="cynthn"
    manager="timlt"
    editor=""
    tags="azure-resource-manager" />
<tags
    ms.assetid="ce7e6cd3-6a4a-4fab-bf66-52f699b1398a"
    ms.service="virtual-machines-windows"
    ms.workload="infrastructure-services"
    ms.tgt_pltfrm="vm-windows"
    ms.devlang="na"
    ms.topic="article"
    ms.date="03/22/2017"
    wacn.date="05/15/2017"
    ms.author="cynthn"
    ms.translationtype="Human Translation"
    ms.sourcegitcommit="457fc748a9a2d66d7a2906b988e127b09ee11e18"
    ms.openlocfilehash="d44c30d52ca51beb734f623ab44d55b4006e9ebb"
    ms.contentlocale="zh-cn"
    ms.lasthandoff="05/05/2017" />

# <a name="create-a-copy-of-a-specialized-windows-vm-running-in-azure"></a>为 Azure 中运行的专用 Windows VM 创建副本
本文说明如何使用 AZCopy 工具从 Azure 中运行的专用 Windows VM 创建 VHD 副本。 然后可以使用 VHD 的副本创建新的 VM。 

* 如果要复制一般化 VM，请参阅[如何从现有一般化 Azure VM 创建 VM 映像](/documentation/articles/virtual-machines-windows-capture-image/)。
* 如果想要从本地 VM（例如，使用 Hyper-V 创建的 VM）上载 VHD，请参阅 [Upload a Windows VHD from an on-premises VM to Azure](/documentation/articles/virtual-machines-windows-upload-image/)（将 Windows VHD 从本地 VM 上载到 Azure）。

## <a name="before-you-begin"></a>开始之前
请确保：

* 获取有关**源和目标存储帐户**的信息。 对于源 VM，需要具有存储帐户和容器名称。 通常，容器名称为 **vhds**。 还需要获取目标存储帐户。 如果尚未拥有存储帐户，可以使用门户（“更多服务”>“存储帐户”>“添加”）或使用 [New-AzureRmStorageAccount](https://msdn.microsoft.com/zh-cn/library/mt607148.aspx) cmdlet 创建一个存储帐户。 
* 已安装 Azure [PowerShell 1.0](https://docs.microsoft.com/zh-cn/powershell/azureps-cmdlets-docs)（或更高版本）。
* 已下载并安装 [AzCopy 工具](/documentation/articles/storage-use-azcopy/)。 

## <a name="deallocate-the-vm"></a>解除分配 VM
解除分配 VM，释放要复制的 VHD。 

* **门户**：单击“虚拟机” > “myVM”>“停止”
* **Powershell**：`Stop-AzureRmVM -ResourceGroupName myResourceGroup -Name myVM`解除分配资源组 **myResourceGroup** 中名为 **myVM** 的 VM。

Azure 门户中该 VM 的“状态”将从“已停止”更改为“已停止(已解除分配)”。

## <a name="get-the-storage-account-urls"></a>获取存储帐户 URL
需要源和目标存储帐户的 URL。 URL 类似于： `https://<storageaccount>.blob.core.chinacloudapi.cn/<containerName>/`。 如果已经知道了存储帐户和容器名称，则只需替换括号之间的信息即可创建 URL。 

可以使用 Azure 门户或 Azure PowerShell 获取 URL：

* **门户**：单击“更多服务” > “存储帐户” > “Blob”，源 VHD 文件可能在 **vhd** 容器中。 单击容器的“属性”并复制标记为 **URL** 的文本。 你将需要用到源和目标容器的 URL。 
* **Powershell**：使用 `Get-AzureRmVM -ResourceGroupName "myResourceGroup" -Name "myVM"` 可获取资源组 **myResourceGroup** 中名为 **myVM** 的 VM 的信息。 在结果中，查看 **Vhd Uri** 的 **Storage profile** 部分。 URI 的第一部分是容器的 URL，最后一部分是 VM 的 OS VHD 名称。

## <a name="get-the-storage-access-keys"></a>获取存储访问密钥
查找源和目标存储帐户的访问密钥。 有关访问密钥的详细信息，请参阅[关于 Azure 存储帐户](/documentation/articles/storage-create-storage-account/)。

* **门户**：单击“更多服务” > “存储帐户” > “所有设置” > “访问密钥”。 复制标记为 **key1** 的密钥。
* **Powershell**：使用 `Get-AzureRmStorageAccountKey -Name mystorageaccount -ResourceGroupName myResourceGroup` 可获取资源组 **myResourceGroup** 中存储帐户 **mystorageaccount** 的存储密钥。 复制标记为 **key1** 的密钥。

## <a name="copy-the-vhd"></a>复制 VHD
可以使用 AzCopy 在存储帐户之间复制文件。 对于目标容器，如果指定的容器不存在，系统会自动创建该容器。 

若要使用 AzCopy，请在本地计算机上打开命令提示符，然后导航到安装 AzCopy 的文件夹。 路径类似于 *C:\Program Files (x86)\Microsoft SDKs\Azure\AzCopy*。 

若要复制容器中的所有文件，请使用 **/S** 开关。 此开关可用于复制 OS VHD 和所有数据磁盘（如果它们在同一个容器中）。 本示例演示如何将存储帐户 **mysourcestorageaccount** 中容器 **mysourcecontainer** 内的所有文件复制到存储帐户 **mydestinationstorageaccount** 中的容器 **mydestinationcontainer**。 将存储帐户和容器的名称替换为自己的名称。 将 `<sourceStorageAccountKey1>` 和 `<destinationStorageAccountKey1>` 替换为自己的密钥。

    AzCopy /Source:https://mysourcestorageaccount.blob.core.chinacloudapi.cn/mysourcecontainer `
        /Dest:https://mydestinationatorageaccount.blob.core.chinacloudapi.cn/mydestinationcontainer `
        /SourceKey:<sourceStorageAccountKey1> /DestKey:<destinationStorageAccountKey1> /S

如果只想要复制包含多个文件的容器中的特定 VHD，则还可以使用 /Pattern 开关指定文件名。 在本示例中，将复制名为 **myFileName.vhd** 的文件。

    AzCopy /Source:https://mysourcestorageaccount.blob.core.chinacloudapi.cn/mysourcecontainer `
      /Dest:https://mydestinationatorageaccount.blob.core.chinacloudapi.cn/mydestinationcontainer `
      /SourceKey:<sourceStorageAccountKey1> /DestKey:<destinationStorageAccountKey1> `
      /Pattern:myFileName.vhd

完成后，将出现如下所示的消息：

    Finished 2 of total 2 file(s).
    [2016/10/07 17:37:41] Transfer summary:
    -----------------
    Total files transferred: 2
    Transfer successfully:   2
    Transfer skipped:        0
    Transfer failed:         0
    Elapsed time:            00.00:13:07

## <a name="troubleshooting"></a>故障排除
* 使用 AZCopy 时，如果看到错误“服务器无法对请求进行身份验证”，请确保授权标头的值构成正确（包括签名）。 如果使用的是密钥 2 或辅助存储密钥，则请尝试使用主密钥或第一个存储密钥。

## <a name="next-steps"></a>后续步骤
* 可通过[将 VHD 的副本作为 OS 磁盘附加到 VM](/documentation/articles/virtual-machines-windows-create-vm-specialized/) 创建新 VM。

<!--Update_Description: wording update-->