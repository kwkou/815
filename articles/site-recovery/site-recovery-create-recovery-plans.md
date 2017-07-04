<properties
    pageTitle="在 Azure Site Recovery 中创建用于故障转移和恢复的恢复计划 | Azure"
    description="介绍如何在 Azure Site Recovery 中创建和自定义用于故障转移以及恢复 VM 与物理服务器的恢复计划"
    services="site-recovery"
    documentationcenter=""
    author="rayne-wiselman"
    manager="jwhit"
    editor="" />
<tags
    ms.assetid="72408c62-fcb6-4ee2-8ff5-cab1218773f2"
    ms.service="site-recovery"
    ms.devlang="na"
    ms.topic="article"
    ms.tgt_pltfrm="na"
    ms.workload="storage-backup-recovery"
    ms.date="02/14/2017"
    wacn.date="03/10/2017"
    ms.author="raynew" />  


# 创建恢复计划


本文提供有关在 [Azure Site Recovery](/documentation/articles/site-recovery-overview/) 中创建和自定义恢复计划的信息。

 恢复计划执行以下操作：

* 定义要一起故障转移再同时启动的计算机组。
* 将计算机一起分组到恢复计划组中，为计算机之间的依赖关系建模。例如，若要故障转移并启动特定的应用程序，可将该应用程序的所有 VM 分组到同一个恢复计划组中。
* 故障转移。可以对恢复计划运行测试、计划或非计划的故障转移。


## 创建恢复计划

1. 单击“恢复计划”>“创建恢复计划”。为恢复计划指定一个名称，并指定源和目标。源位置必须具有已针对故障转移和恢复启用的虚拟机。

    - 对于 VMM 到 VMM 的复制，请选择“源类型”>“VMM”，然后选择源和目标 VMM 服务器。单击“Hyper-V”查看受保护的云。
    - 对于 VMM 到 Azure 的复制，请选择“源类型”>“VMM”。选择源 VMM 服务器，选择“Azure”作为目标。
    - 对于 Hyper-V 到 Azure 的复制（无 VMM），请选择“源类型”>“Hyper-V 站点”。选择该站点作为源，选择“Azure”作为目标。
2. 在“选择虚拟机”中，选择要添加到恢复计划中的默认组（组 1）的虚拟机（或复制组）。

## 自定义和扩展恢复计划

可以自定义和扩展恢复计划：

- **添加新组** - 向默认组添加其他恢复计划组（最多 7 个），然后向这些组添加更多计算机或复制组。组将按添加顺序进行编号。一个虚拟机或复制组只能包含在一个恢复计划组中。
- **添加手动操作** - 可以添加要在恢复计划组之前或之后运行的手动操作。恢复计划运行时，将在手动操作的插入点停止。此时会显示一个对话框，提示你指定该手动操作已完成。
- **添加脚本** - 可添加在恢复计划组前或后运行的脚本。添加脚本时，将为该组添加一组新的操作。例如，将使用以下名称创建组 1 的一组预先步骤：“组 1: 预先步骤”。该集中将列出所有预先步骤。仅当已部署 VMM 服务器时，才能在主站点上添加脚本。
- **添加 Azure Runbook** - 可通过 Azure Runbook 扩展恢复计划。例如，自动执行任务或创建单步恢复。[了解详细信息](/documentation/articles/site-recovery-runbook-automation/)。

## 添加脚本

可在恢复计划中使用 PowerShell 脚本。

 - 确保脚本使用的是 try-catch 块，以便恰当地处理异常。
    - 如果脚本中出现异常，脚本将停止运行且任务显示为失败。
    - 如果发生错误，脚本的剩余部分不会运行。
    - 如果在运行计划外故障转移时发生错误，将继续运行恢复计划。
    - 如果在运行计划内故障转移时发生错误，恢复计划将会停止。需要修复脚本，检查它是否按预期运行，然后重新运行恢复计划。
- Write-Host 命令不适用于恢复计划脚本，脚本将失败。若要创建输出，请创建转而运行主脚本的代理脚本。确保所有输出均通过 >> 命令传输。
  * 如果脚本在 600 秒内未返回，将发生超时。
  * 如果有任何内容写出到 STDERR，脚本将归类为失败。此信息将显示在脚本执行详细信息中。

如果在部署中使用 VMM：

* 恢复计划中的脚本在 VMM 服务帐户的上下文中运行。确保此帐户对脚本所在的远程共享拥有“读取”权限。测试脚本是否可在 VMM 服务帐户特权级别运行。
* Windows PowerShell 模块中随附提供了 VMM cmdlet。安装 VMM 控制台时已安装该模块。可在脚本中通过以下命令将其加载到脚本中：
   - Import-Module -Name virtualmachinemanager。[了解详细信息](https://technet.microsoft.com/zh-cn/library/hh875013.aspx)。
* 确保 VMM 部署中至少有一个库服务器。默认情况下，VMM 服务器的库共享路径位于 VMM 服务器本地，其文件夹名称为 MSCVMMLibrary。
    * 如果库共享路径在远程位置（或在本地，但不与 MSCVMMLibrary 共享），请按如下所示配置共享（例如，使用 \\libserver2.contoso.com\\share\\）：
      * 打开注册表编辑器并导航到 **HKEY\_LOCAL\_MACHINE\\SOFTWARE\\MICROSOFT\\Azure Site Recovery\\Registration**。
      * 编辑 **ScriptLibraryPath** 值，将其替换为 \\libserver2.contoso.com\\share。指定完整的 FQDN。提供对共享位置的权限。
      * 确保测试脚本时所用的用户帐户具有与 VMM 服务帐户相同的权限。这将检查独立测试的脚本是否按其在恢复计划中的相同方式进行运行。在 VMM 服务器上，将执行策略设置为绕过，如下所示：
        * 使用提升的权限打开 64 位 Windows PowerShell 控制台。
        * 键入：**Set-executionpolicy bypass**。[了解详细信息](https://technet.microsoft.com/zh-cn/library/ee176961.aspx)。

## 向计划添加脚本或手动操作

向默认恢复计划组添加 VM 或复制组并创建该计划后，可向该计划组添加脚本。

1. 打开恢复计划。
2. 单击“步骤”列表中的任意项，然后单击“脚本”或“手动操作”。
3. 指定是要在选定项的前面还是后面添加该脚本或操作。使用“上移”和“下移”按钮可以上下移动脚本的位置。
4. 如果添加 VMM 脚本，请选择“故障转移到 VMM 脚本”。在“脚本路径”中，键入共享的相对路径。在以下 VMM 示例中指定路径：**\\RPScripts\\RPScript.PS1**。
5. 如果添加 Azure 自动化 Runbook，请指定该 Runbook 所在的 Azure 自动化帐户，然后选择相应的 Azure Runbook 脚本。
6. 执行恢复计划故障转移，确保脚本按预期运行。


### VMM 脚本

如果有一个 VMM 源站点，可在 VMM 服务器上创建一个脚本，并将其包含在恢复计划中。

1. 在库共享中新建文件夹。例如，<VMMServerName>\\MSSCVMMLibrary\\RPScripts。将其放到源和目标 VMM 服务器上。
2. 创建脚本（例如，RPScript），并验证其按预期运行。
3. 将该脚本放在源和目标 VMM 服务器上的 <VMMServerName> \\MSSCVMMLibrary 位置。


## 后续步骤

[详细了解](/documentation/articles/site-recovery-failover/)如何运行故障转移。

<!---HONumber=Mooncake_0306_2017-->
<!--Update_Description: whole content update-->