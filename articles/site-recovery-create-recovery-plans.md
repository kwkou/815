<properties
	pageTitle="创建恢复计划 |Azure" 
	description="使用 Azure Site Recovery 创建恢复计划，以故障转移并恢复虚拟机和物理服务器组。" 
	services="site-recovery"
	documentationCenter=""
	authors="rayne-wiselman"
	manager="jwhit"
	editor=""/>

<tags
	ms.service="site-recovery"
	ms.date="02/01/2016" 
	wacn.date="03/17/2016"/>

# 创建恢复计划

Azure Site Recovery 服务有助于业务连续性和灾难恢复 (BCDR) 策略，因为它可以协调虚拟机和物理服务器的复制、故障转移和恢复。可将虚拟机复制到 Azure 或辅助本地数据中心。如需快速概览，请阅读[什么是 Azure Site Recovery？](/documentation/articles/site-recovery-overview/)。


## 概述

本文提供有关创建和自定义恢复计划的信息。

恢复计划由一个或多个包含要一起进行故障转移的虚拟机或物理服务器的有序组构成。恢复计划执行以下操作：

- 定义一起故障转移和启动的虚拟机组。
- 通过将计算机一起分组到恢复计划组中，为计算机之间的依赖关系建模。例如，如果你要故障转移并启动特定的应用程序，可以将该应用程序的虚拟机分组到同一个恢复计划组中。
- 自动化和扩展故障转移。你可以对恢复计划运行测试、计划或非计划的故障转移。你可以使用脚本、Azure 自动化和手动操作自定义恢复计划。

恢复计划显示 Site Recovery 门户中的“恢复计划”上。


请将任何评论或问题发布到本文底部，或者发布到 [Azure 恢复服务论坛](https://social.msdn.microsoft.com/Forums/zh-cn/home?forum=hypervrecovmgr)。

## 开始之前

注意以下事项：

- 恢复计划不应混合具有单个网络适配器的虚拟机和具有多个网络适配器的虚拟机。这是因为 Azure 云服务中不允许混合这些虚拟机。
- 你可以通过脚本和手动操作扩展恢复计划。注意以下事项：
	- 使用 Windows PowerShell 编写脚本。
	- 确保脚本使用的是 try-catch 块，以便恰当地处理异常。如果脚本中出现异常，则脚本将停止运行，并将任务显示为失败。如果确实发生错误，则不会运行脚本的任何剩余部分。如果在运行未计划的故障转移时发生这种情况，恢复计划将继续。如果在运行计划的故障转移时发生这种情况，恢复计划将停止。如果发生这种情况，请修复脚本，确保它按预期运行，然后重新运行恢复计划。
	- Write-Host 命令不适用于恢复计划脚本，脚本将失败。如果你要创建输出，请创建一个要进而运行主脚本的代理脚本，并确保使用 >> 命令传送所有输出。
	- 如果脚本在 600 秒内未返回，则发生超时。
	- 如果有任何内容写出到 STDERR，则脚本将归类为失败。此信息将显示在脚本执行详细信息中。
	- 如果你在部署中使用 VMM，请注意：

		- 恢复计划中的脚本在 VMM 服务帐户的上下文中运行。确保此帐户对脚本所在的远程共享具有“读取”权限，并以 VMM 服务帐户权限级别测试要运行的脚本。
		- Windows PowerShell 模块中随附了 VMM cmdlet。在安装 VMM 控制台时，会安装 VMM Windows PowerShell 模块。可以在脚本中使用以下命令将 VMM 模块加载到脚本中：Import-Module -Name virtualmachinemanager。[了解详细信息](hhttps://technet.microsoft.com/zh-cn/library/hh875013.aspx)。
		- 确保 VMM 部署中至少有一个库服务器。默认情况下，VMM 服务器的库共享路径位于 VMM 服务器本地，其文件夹名称为 MSCVMMLibrary。
		- 如果库共享路径在远程位置（或在本地，但不与 MSCVMMLibrary 共享），请按如下所示配置共享（例如，使用 \\libserver2.contoso.com\\share\\）：
			- 打开注册表编辑器并导航到 HKEY\_LOCAL\_MACHINE\\SOFTWARE\\MICROSOFT\\Azure Site Recovery\\Registration。
			-  编辑 ScriptLibraryPath 的值，将此值设置为 \\libserver2.contoso.com\\share。指定完整的 FQDN。提供对共享位置的权限。
			-  确保使用与 VMM 服务帐户具有相同权限的用户帐户来测试脚本，以确保独立测试的脚本以在恢复计划中的相同运行方式运行。在 VMM 服务器上，将执行策略设置为绕过，如下所示：
				-  使用提升的权限打开 64 位 Windows PowerShell 控制台。
				-  键入：**Set-executionpolicy bypass**。[了解详细信息](https://technet.microsoft.com/zh-cn/library/ee176961.aspx)。

## 创建恢复计划

创建恢复计划的方法取决于 Site Recovery 部署。

- **复制 VMware VM 和物理服务器**—你可以创建一个计划，并将包含虚拟机和物理服务器的复制组添加到恢复计划。
- **复制 Hyper-V VM（在 VMM 云中）**—你可以创建一个计划，并将 VMM 云中受保护的 Hyper-V 虚拟机添加到恢复计划。
- **复制 Hyper-V VM（不在 VMM 云中）**—你可以创建一个计划，并将保护组中的受保护的 Hyper-V 虚拟机添加到恢复计划。
- **SAN 复制**—你可以创建一个计划，并将包含虚拟机的复制组添加到恢复计划。之所以选择复制组而不是特定的虚拟机，是因为复制组的所有虚拟机必须一起故障转移（先在存储层发生故障转移）。


按如下所述创建恢复计划：

1. 单击“恢复计划”选项卡 >“创建恢复计划”。为恢复计划指定一个名称，并指定源和目标。源服务器必须具有启用了故障转移和恢复的虚拟机。

	- 如果要从 VMM 复制到 VMM，请选择“源类型”>“VMM”，然后选择源和目标 VMM 服务器。单击“Hyper-V”查看配置为使用 Hyper-V 副本的云。 
	- 如果要使用 SAN 从 VMM 复制到 VMM，请选择“源类型”>“VMM”，然后选择源和目标 VMM 服务器。单击“SAN”以查看针对 SAN 复制进行了配置的云。
	- 如果要从 VMM 复制到 Azure，请选择“源类型”>“VMM”。选择源 VMM 服务器，并选择“Azure”作为目标。
	- 如果要从 Hyper-V 站点复制，请选择“源类型”>“Hyper-V 站点”。选择该站点作为源，并选择“Azure”作为目标。
- 如果要从 VMware 或物理本地服务器复制到 Azure，请选择某个配置服务器作为源，并选择“Azure”作为目标

2. 在“选择虚拟机”中，选择要添加到恢复计划中的默认组 (Group 1) 的虚拟机（或复制组）。

## 自定义恢复计划

将受保护的虚拟机或复制组添加到默认恢复计划组并创建该计划后，你可以自定义该计划：

- **添加新组** - 可以添加更多的恢复计划组。添加的组将按其添加顺序编号。最多可以添加七个组。可以在这些新组中添加更多的计算机或复制组。请注意，虚拟机或复制组只能包含在一个恢复计划组中。
- **添加脚本** - 可以添加恢复计划组前面或后面的脚本。添加脚本时，将为该组添加一组新的操作。例如，将使用以下名称创建组 1 的一组预先步骤：“组 1：预先步骤”。该集中将列出所有预先步骤。请注意，仅当已部署 VMM 服务器时，才能在主站点上添加脚本。
- **添加手动操作** - 可以添加要在恢复计划组之前或之后运行的手动操作。在恢复计划运行时，将会在手动操作的插入位置停止，然后显示一个对话框，提示你指定手动操作已完成。

## 使用脚本扩展恢复计划

可以在恢复计划中添加脚本：

- 如果你有一个 VMM 源站点，可以在 VMM 服务器上创建一个脚本，并将其包含在恢复计划中。
- 如果要复制到 Azure，可以将 Azure 自动化 Runbook 集成到恢复计划中

### 创建 VMM 脚本


按如下所示创建脚本：

1. 在库共享中创建新的文件夹，例如 \\<VMMServerName>\\MSSCVMMLibrary\\RPScripts。将其放到源和目标 VMM 服务器上。
2. 创建脚本（例如，RPScript），并验证其按预期运行。
3. 将该脚本放在源和目标 VMM 服务器上的 <VMMServerName> \\MSSCVMMLibrary 位置。

### 创建 Azure 自动化 Runbook

你可以通过计划中包含的 Azure 自动化 Runbook 来扩展恢复计划。[了解详细信息](/documentation/articles/site-recovery-runbook-automation/)。


### 将自定义设置添加到恢复计划

1. 打开要自定义的恢复脚本。
2. 单击相应的控件添加虚拟机或新组。
3. 若要添加脚本或手动操作，请单击“步骤”列表中的任意项，然后单击“脚本”或“手动操作”。指定是要在选定项的前面还是后面添加该脚本或操作。使用“上移”和“下移”命令按钮可以上下移动脚本的位置。
4. 如果你要添加 VMM 脚本，请选择“故障转移到 VMM 脚本”，然后在“脚本路径”中键入共享的相对路径。对于本示例，共享位于 \<VMMServerName>\\MSSCVMMLibrary\\RPScripts 中，因此请指定路径：\\RPScripts\\RPScript.PS1。
5. 如果你要添加 Azure 自动化 Runbook，请指定该 Runbook 所在的 **Azure 自动化帐户**，然后选择相应的 **Azure Runbook 脚本**。
5. 执行恢复计划故障转移，以确保脚本按预期运行。


## 后续步骤

你可以在恢复计划中运行不同类型的故障转移，包括用于检查环境的测试故障转移，以及计划的故障转移或非计划故障转移。[了解详细信息](/documentation/articles/site-recovery-failover/)。


 

<!---HONumber=Mooncake_0307_2016-->