<properties
    pageTitle="排查 Windows VM 部署问题 - RM | Azure"
    description="排查在 Azure 中新建 Windows 虚拟机时遇到的 Resource Manager 部署问题"
    services="virtual-machines-windows, azure-resource-manager"
    documentationcenter=""
    author="JiangChen79"
    manager="felixwu"
    editor=""
    tags="top-support-issue, azure-resource-manager" />
<tags
    ms.assetid="afc6c1a4-2769-41f6-bbf9-76f9f23bcdf4"
    ms.service="virtual-machines-windows"
    ms.workload="na"
    ms.tgt_pltfrm="vm-windows"
    ms.devlang="na"
    ms.topic="article"
    ms.date="12/16/2016"
    wacn.date="02/24/2017"
    ms.author="cjiang" />

# 排查在 Azure 中新建 Windows 虚拟机时遇到的 Resource Manager 部署问题
[AZURE.INCLUDE [virtual-machines-troubleshoot-deployment-new-vm-opening](../../includes/virtual-machines-troubleshoot-deployment-new-vm-opening-include.md)]

[AZURE.INCLUDE [support-disclaimer](../../includes/support-disclaimer.md)]

## 收集活动日志
若要开始故障排除，请收集活动日志，以识别与问题相关的错误。以下链接包含有关要遵循的过程的详细信息。

[查看部署操作](/documentation/articles/resource-manager-deployment-operations/)

[通过查看活动日志管理 Azure 资源](/documentation/articles/resource-group-audit/)

[AZURE.INCLUDE [virtual-machines-troubleshoot-deployment-new-vm-issue1](../../includes/virtual-machines-troubleshoot-deployment-new-vm-issue1-include.md)]

[AZURE.INCLUDE [virtual-machines-windows-troubleshoot-deployment-new-vm-table](../../includes/virtual-machines-windows-troubleshoot-deployment-new-vm-table.md)]

**Y：**如果 OS 是通用的 Windows，并且是使用通用设置上载和/或捕获的，则不会有任何错误。同理，如果 OS 是专用的 Windows，并且是使用专用设置上载和/或捕获的，也不会有任何错误。

**上载错误：**

**N<sup>1</sup>：**如果 OS 是通用的 Windows，但却以专用设置上载，则会发生预配超时错误，并且 VM 会卡在 OOBE 屏幕上。

**N<sup>2</sup>：**如果 OS 是专用的 Windows，但却以通用设置上载，则会发生预配失败错误，并且 VM 会卡在 OOBE 屏幕上，因为新 VM 是以原始计算机名称、用户名和密码运行的。

**解决方法**

若要解决这两个错误，请通过与 OS 相同的设置（通用/专用）使用 [Add-AzureRmVhd 上传原始 VHD](https://msdn.microsoft.com/zh-cn/library/mt603554.aspx)（本地可用）。若要以通用设置上载，请记得先运行 sysprep。

**捕获错误：**

**N<sup>3</sup>：**如果 OS 是通用的 Windows，但却捕获为专用映像，则会发生预配超时错误，因为标记为通用的原始 VM 不可用。

**N<sup>4</sup>：**如果 OS 专用于 Windows，但却捕获为通用映像，则会发生预配失败错误，因为新 VM 是以原始计算机名称、用户名和密码运行的。此外，标记为专用的原始 VM 不可用。

**解决方法**

若要解决这两个错误，请从门户中删除当前映像，并[从当前 VHD 重新捕获映像](/documentation/articles/virtual-machines-windows-vhd-copy/)，该映像具有与该 OS（通用/专用）相同的设置。

## 问题：自定义/库/应用商店映像；分配失败
当新的 VM 请求被固定到不支持所请求的 VM 大小、或没有可用空间可处理请求的群集时，便会发生此错误。

**原因 1：**群集不支持请求的 VM 大小。

**解决方法 1：**

* 使用更小的 VM 大小来重试请求。
* 如果无法更改请求的 VM 大小：
    * 停止可用性集中的所有 VM。单击“资源组”\> *你的资源组* \>“资源”\> *你的可用性集* \>“虚拟机”\> *你的虚拟机* \>“停止”。
    * 所有 VM 都停止后，创建所需大小的新 VM。
    * 先启动新 VM，选择每个已停止的 VM，然后单击“启动”。

**原因 2：**群集没有可用的资源。

**解决方法 2：**

* 稍后重试请求。
* 如果新 VM 属于不同的可用性集
    * 在不同的可用性集（位于同一区域）中创建新 VM。
    * 将新 VM 添加到同一虚拟网络。

## 后续步骤
如果你在 Azure 中启动已停止的 Windows VM 或调整现有 Windows VM 的大小时遇到问题，请参阅[排查在 Azure 中重新启动现有 Windows 虚拟机或调整其大小时遇到的 Resource Manager 部署问题](/documentation/articles/virtual-machines-windows-restart-resize-error-troubleshooting/)。

<!---HONumber=Mooncake_0220_2017-->
<!--Update_Description: wording update-->