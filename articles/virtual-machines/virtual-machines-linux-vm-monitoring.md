<!-- Ibiza Portal: tested -->

<properties
   pageTitle="启用或禁用 Azure VM 监视"
   description="介绍如何启用或禁用 Azure VM 监视"
   services="virtual-machines-linux"
   documentationCenter="virtual-machines"
   authors="kmouss"
   manager="drewm"
   editor=""/>

<tags
	ms.service="virtual-machines-linux"
	ms.date="02/08/2016"
	wacn.date="06/07/2016"/>
   
# 启用或禁用 Azure VM 监视

本部分介绍如何在 Azure 中运行的虚拟机上启用或禁用监视。如果 Azure 虚拟机是从 [Azure 门户预览](https://portal.azure.cn)部署的，并且默认以 1 分钟的时段提供监视图形，则会按默认在该虚拟机上启用监视。你可以使用门户预览或适用于 Mac、Linux 和 Windows 的 Azure 命令行界面 (Azure CLI) 来启用或禁用监视。

## 通过 Azure 门户预览启用/禁用监视
 
你可以启用 Azure VM 监视，以 1 分钟的时段提供实例相关数据（包括存储更改）。然后，可以通过门户预览图形或 API 获取 VM 的详细诊断数据。默认情况下，Azure 门户预览将启用监视，但你可以按如下所述将它关闭。可以在 VM 正在运行或处于停止状态时启用监视。

- 通过 **[https://portal.azure.cn](https://portal.azure.cn)** 打开 Azure 门户预览

- 在左侧导航栏中，单击“虚拟机”。

- 在“虚拟机”列表中，选择正在运行或已停止的实例。虚拟机边栏选项卡将会打开。

- 单击“所有设置”。

- 单击“诊断”。

- 将状态更改为“打开”或“关闭”。你还可以在此边栏选项卡中选择要为虚拟机启用的监视级别详细信息。

>[Azure.Note] 当你创建新的虚拟机时，默认情况下已启用“诊断打开”开关

![通过 Azure 门户预览启用/禁用监视。][1]


## 使用 Azure CLI 启用/禁用监视
 
启用 Azure VM 监视。

- 创建名为 PrivateConfig.json 且包含以下内容的文件。{ "storageAccountName":"the storage account to receive data", "storageAccountKey":"the key of the account" }
- 运行以下 Azure CLI 命令。

        azure vm extension set myvm LinuxDiagnostic Microsoft.OSTCExtensions 2.0 --private-config-path PrivateConfig.json

[Azure.Note] 可以从 2.0 版本更改为更高的版本（如果有）。

有关配置监视指标和示例的详细信息，请访问文档- [Using Linux Diagnostic Extension to Monitor Linux VM's performance and diagnostic data（使用 Linux 诊断扩展监视 Linux VM 的性能和诊断数据）](/documentation/articles/virtual-machines-linux-classic-diagnostic-extension/)。

<!--Image references-->
[1]: ./media/virtual-machines-linux-vm-monitoring/portal-enable-disable.png
 


<!---HONumber=Mooncake_0425_2016-->