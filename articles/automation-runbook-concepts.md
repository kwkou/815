<properties 
   pageTitle="Azure Automation Runbook 概念"
   description="介绍在 Azure Automation 中创建 Runbook 时应了解的基本概念。 "
   services="automation"
   documentationCenter=""
   authors="bwren"
   manager="stevenka"
   editor="tysonn" />
<tags 
wacn.date="05/15/2015"
   ms.service="automation"
   ms.devlang="na"
   ms.topic="article"
   ms.tgt_pltfrm="na"
   ms.workload="infrastructure-services"
   ms.date="04/13/2015"
   ms.author="bwren" />

# Azure Automation Runbook 概念

Azure Automation 中的 Runbook 作为 Windows PowerShell 工作流实现。本部分提供 Automation Runbook 通用的工作流关键功能的简要概述。[Windows PowerShell 工作流入门](https://technet.microsoft.com/zh-CN/library/jj134242.aspx)中提供了有关工作流的完整详细信息。


## Windows PowerShell 工作流

工作流是一系列编程的连接步骤，用于执行长时间运行的任务，或者要求跨多个设备或托管节点协调多个步骤。与标准脚本相比，工作流的好处包括能够同时执行针对多台设备的操作以及自动从故障中恢复的能力。Windows PowerShell 工作流是利用 Windows Workflow Foundation 的 Windows PowerShell 脚本。尽管工作流是使用 Windows PowerShell 语法编写的并通过 Windows PowerShell 启动，但它将由 Windows Workflow Foundation 进行处理。

### 基本结构

Windows PowerShell 工作流以 **Workflow** 关键字开头，后接括在大括号中的脚本正文。工作流名称接在 **Workflow** 关键字的后面，如以下语法中所示。工作流名称与 Automation Runbook 的名称匹配。

    Workflow Test-Runbook
    {
       <Commands>
    }

若要将参数添加到工作流，请使用 **Param** 关键字，如以下语法中所示。当用户启动 Runbook 时，管理门户将提示用户为这些参数提供值。此示例使用了可选的 Parameter 属性，该属性指定参数是否是必需的。

    Workflow Test-Runbook
    {
      Param
      (
       [Parameter(Mandatory=<$True | $False>]
       [Type]$<ParameterName>,

       [Parameter(Mandatory=<$True | $False>]
       [Type]$<ParameterName>
      )
      <Commands>
    }

### 命名

工作流的名称应符合 Windows PowerShell 的标准动词-名词格式。你可以参考 [Windows PowerShell 命令的已批准谓词](https://msdn.microsoft.com/zh-CN/library/windows/desktop/ms714428(v=vs.85).aspx)，以获取可用的已批准谓词列表。工作流名称与 Automation Runbook 的名称匹配。如果正在导入某个 Runbook，其文件名必须与工作流名称匹配，并且必须以 .ps1 结尾。

### 限制

有关 Windows PowerShell 工作流与 Windows PowerShell 之间的限制和语法差异的完整列表，请参阅[脚本工作流与脚本之间的语法差异](https://technet.microsoft.com/zh-CN/library/jj574140.aspx)。

## 活动

活动是工作流中的特定任务。就像脚本由一个或多个命令构成一样，工作流由一个或多个按顺序执行的活动构成。Windows PowerShell 工作流在运行工作流时，会自动将许多 Windows PowerShell cmdlet 转换为活动。当你在 Runbook 中指定其中的某个 cmdlet 时，相应的活动实际上由 Windows Workflow Foundation 运行。对于这些没有相应活动的 cmdlet，Windows PowerShell 工作流将自动在 [InlineScript](#inlinescript) 活动中运行该 cmdlet。有一组 cmdlet 已被排除，不能包含在工作流中，除非你显式将它们包含在 InlineScript 块中。有关这些概念的更多详细信息，请参阅[在脚本工作流中使用活动](https://technet.microsoft.com/zh-CN/library/jj574194.aspx)。

工作流活动共享一组公用参数来配置其操作。有关工作流公用参数的详细信息，请参阅 [about_WorkflowCommonParameters](https://technet.microsoft.com/zh-CN/library/jj129719.aspx)。

## 集成模块

 *Integration Module*是包含 Windows PowerShell 模块的程序包，可导入 Azure Automation。集成模块中已导入 Azure Automation 的 Cmdlet 将自动提供给同一 Automation 帐户中的所有 Runbook 使用。由于 Azure Automation 基于 Windows PowerShell 4.0，因此支持自动加载模块，这意味着，无需使用 [Import-Module](https://technet.microsoft.com/zh-CN/library/hh849725.aspx) 将它们导入脚本，就能使用已安装模块中的 cmdlet。

## 并行执行

Windows PowerShell 工作流的一个优点是能够与典型脚本一样并行而不是按顺序执行一组命令。这在 Runbook 中特别有用，因为它们可以执行多个需要花费很长时间才能完成的操作。例如，Runbook 可能会设置一组虚拟机。此时，可以同时执行相关操作，而不用按顺序逐个执行每个设置过程，从而提高了整体效率。仅当所有操作都已完成时，Runbook 才会继续。

你可以使用 **Parallel** 关键字来配合同时运行的多个命令创建一个脚本块。这将使用如下所示的语法。在这种情况下，Activity1 和 Activity2 将同时启动。只有在 Activity1 和 Activity2 已完成后，activity3 才会启动。

    Parallel
    {
      <Activity1>
      <Activity2>
    }
    <Activity3>

你可以使用 **ForEach -Parallel** 构造来同时处理集合中每个项的命令。尽管脚本块中的命令按顺序运行，但集合中的项是并行处理的。这将使用如下所示的语法。在这种情况下，Activity1 将在同一时间对集合中的所有项启动。对于每个项，Activity2 将在 Activity1 完成后启动。只有在对所有项完成 Activity1 和 Activity2 后，activity3 才会启动。

    ForEach -Parallel ($<item> in $<collection>)
    {
      <Activity1>
      <Activity2>
    }
    <Activity3>

**Sequence** 关键字用于按顺序运行 **Parallel** 脚本块中的命令。**Sequence** 脚本块与其他命令同时运行，但块中的命令按顺序运行。这将使用如下所示的语法。在这种情况下，Activity1、Activity2 和 Activity3 将同时启动。只有在 Activity3 完成后，activity4 才会启动。Activity5 在所有其他活动已完成后启动。

    Parallel
    {
      <Activity1>
      <Activity2>

      Sequence 
      { 
        <Activity3>  
        <Activity4>
      }

    }
    <Activity5>

## 检查点

 *checkpoint*是工作流当前状态的快照，它包括变量的当前值以及到该点为止生成的任何输出。要在 Runbook 中完成的最后一个检查点将保存到 Automation 数据库，以便在出现问题（例如，运行时计算机关闭）时可以恢复工作流。如果没有检查点，工作流将从头开始。在完成 Runbook 作业后，将删除这些检查点数据。

你可以使用 **Checkpoint-Workflow** 活动在工作流中设置一个检查点。当你在 Runbook 中包含此活动时，会立即获取检查点。如果异常暂停了 Runbook，则当作业恢复时，将从设置的最后一个检查点处恢复。

在以下示例代码中，Activity2 后的发生的异常导致 Runbook 暂停。当作业恢复时，它会通过运行 Activity2 来启动，因为此活动在设置的最后一个检查点之后。

    <Activity1>
    Checkpoint-Workflow
    <Activity2>
    <Exception>
    <Activity3>

活动可能容易出现异常，并且不应是重复如果恢复了 Runbook 后，应在 Runbook 中设置检查点。例如，你的 Runbook 可能创建虚拟机。你可以在命令之前和之后设置一个检查点以创建虚拟机。如果创建失败，则在 Runbook 恢复时会重复命令。如果创建成功但 Runbook 随后失败，则恢复 Runbook 时不会再创建虚拟机。

关于检查点的详细信息，请参阅[将检查点添加到脚本工作流](https://technet.microsoft.com/zh-CN/library/jj574114.aspx)。

## 暂停工作流

你可以使用 **Suspend-Workflow** 活动强制 Runbook 暂停自身。此活动将设置检查点，并导致工作流立即暂停。暂停工作流可用于可能需要运行另一组活动之前要执行一个手动步骤的 Runbook。

有关暂停工作流的详细信息，请参阅[使工作流暂停自身](https://technet.microsoft.com/zh-CN/library/jj574175.aspx)。

## InlineScript

**InlineScript** 活动在传统的 PowerShell 脚本而不是 PowerShell 工作流中运行命令块，并将其输出返回到工作流。尽管工作流中的命令将发送到 Windows Workflow Foundation 进行处理，但 InlineScript 块中的命令将由 Windows PowerShell 处理。该活动使用标准的工作流公用参数，包括 **PSCredential**，它可让你指定使用备用凭据运行代码块。

InlineScript 使用如下所示的语法。

    InlineScript
    {
      <Script Block>
    } <Common Parameters>

尽管 InlineScript 活动在某些 Runbook 中可能至关重要，但它们不支持工作流构造，并且只能出于以下原因才使用：

- 你无法使用 InlineScript 块中的检查点。如果块中发生失败，它必须从块的开头恢复。
- InlineScript 会影响 Runbook 的可伸缩性，因为它包含 Windows PowerShell 会话的 InlineScript 块的整个长度。
- 活动（如 Get-AutomationVariable 和 Get-AutomationPSCredential）在 InlineScript 块中不可用。  

如果你确实需要使用 InlineScript，应尽量减小其作用域。例如，如果 Runbook 要循环访问某个集合，同时将相同的操作应用于每个项，则该循环应发生在 InlineScript 之外。这将提供以下优势：

- 你可以在每次迭代后设置工作流的[检查点](#checkpoints) 。如果作业已暂停或中断，然后恢复，则循环将可以恢复。
- 你可以使用 **ForEach - Parallel** 来并发处理集合项。

如果你确实要在 Runbook 中使用 InlineScript，请记住以下建议：

- 你可以使用 **$Using** 作用域修饰符将值传递给脚本。例如，在 InlineScript 外部设置的名为 $abc 的变量将变为 InlineScript 内部的 $using:abc。

- 若要从 InlineScript 返回输出，请将输出分配给变量，并将要返回的任何数据输出到输出流。以下示例将字符串"hi"分配给名为 $output 的变量。

	<pre><code>$output = InlineScript { Write-Output "hi" }</code></pre>

- 避免在 InlineScript 作用域内部定义工作流。即使某些工作流看起来可能正常运行，但这不是经过测试的方案。因此，你可能会遇到令人困惑的错误消息或意外的行为。

有关使用 InlineScript 的更多详细信息，请参阅[在工作流中运行 Windows PowerShell 命令](https://technet.microsoft.com/zh-CN/library/jj574197.aspx)和 [about_InlineScript](https://technet.microsoft.com/zh-CN/library/jj649082.aspx)。


## 相关文章

- [创建或导入 Runbook](https://technet.microsoft.com/zh-CN/library/dn919921.aspx)

<!--HONumber=53-->