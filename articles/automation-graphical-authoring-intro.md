<properties 
   pageTitle="Azure 自动化中的图形创作"
   description="图形创作可以让你在不使用代码的情况下，为 Azure 自动化创建 Runbook。本文介绍了图形创作以及开始创建图形 Runbook 所需的所有详细信息。"
   services="automation"   
   documentationCenter=""
   authors="bwren"
   manager="stevenka"
   editor="tysonn"/>
   
<tags ms.service="automation" ms.date="05/04/2015" wacn.date="06/16/2015"/> 

# Azure 自动化中的图形创作

## 介绍

图形创作可以用来为 Azure 自动化创建 Runbook，不需要编写复杂的基础性的 Windows PowerShell 工作流代码。你可以从 cmdlet 库中添加活动，并可将其他活动添加到画布上，然后将它们链接到一起形成工作流。

本文提供了图形创作简介，以及着手创建图形 Runbook 所需的概念。

## 图形 Runbook

Azure 自动化中的所有 Runbook 都是 Windows PowerShell 工作流。图形 Runbook 会生成由自动化辅助角色运行的 PowerShell 代码，但你无法看到它，也不能直接修改它。无法将图形 Runbook 转换为文本形式的 Runbook，也无法将现有的文本形式的 Runbook 导入图形编辑器中。


## 图形编辑器概述

通过创建或编辑图形 Runbook，你可以在 Azure 预览门户中打开图形编辑器。

![图形工作区](media/automation-graphical-authoring-intro/graphical-editor.png)


以下各节介绍图形编辑器中的控件。


### 画布  

画布是设计 Runbook 的地方。你可以将库控件中节点的活动添加到 Runbook，然后将其通过链接进行连接，以便定义 Runbook 的逻辑。

### 库控件

库控件是你选择要添加到 Runbook 的[活动](#activities)的地方。你可以将活动添加到画布，然后再将它们连接到其他活动。它包括下表中描述的四个部分。

| 部分 | 说明 |
|:---|:---|
| Cmdlet | 包括可以在你的 Runbook 中使用的所有 cmdlet。Cmdlet 按模块组织。所有安装在你的自动化帐户中的模块都可用。 |
| Runbook | 包括你自动化帐户中按标记组织的 Runbook。由于一个 Runbook 可以有多个标记，因此该 Runbook 可能会列在多个标记下。可以将这些 Runbook 添加到画布中，用作子 Runbook。会显示当前正在编辑的 Runbook，但不能将其添加到画布中，因为 Runbook 不能调用其自身。
| 资产 | 包括自动化帐户中能够在 Runbook 中使用的[自动化资产](http://msdn.microsoft.com/zh-cn/library/dn939988.aspx)。将资产添加到 Runbook 中时，它会添加一个可以获取所选资产的工作流活动。在使用变量资产的情况下，你可以选择是否添加用于获取变量或设置变量的活动。
| Runbook 控件 | 包括可以在当前 Runbook 中使用的 Runbook 控件活动。*交接点*采用多个输入，并会等到所有输入完成后才会继续执行工作流。*工作流脚本*可运行一行或多行 PowerShell 工作流代码。你可以将此活动用于自定义代码或通过其他活动不能实现的功能。|

### 配置控件

配置控件是你为在画布上选择的对象提供详细信息的地方。此控件中提供的属性取决于所选对象的类型。当你在配置控件中选择一个选项时，它会打开用于提供额外信息的其他边栏选项卡。

### 测试控件

首次启动图形编辑器时，不显示测试控件。它会在你以交互方式[测试图形 Runbook](#graphical-runbook-procedures) 时打开。

## 图形 Runbook 过程 

### 测试图形 Runbook

你可以在 Azure 预览门户中测试 Runbook 的草稿版，对 Runbook 的已发布版不做任何更改，也可以在新的 Runbook 发布前对其进行测试。这样你就可以确保新的 Runbook 在替代已发布版之前各项功能运行正常。测试 Runbook 时，将执行草稿版 Runbook，并会完成其所执行的任何操作。不会创建作业历史记录，但会在“测试输出窗格”中显示输出。

若要打开 Runbook 的测试控件，可先打开要编辑的该 Runbook，然后单击“测试窗格”按钮。

![测试窗格按钮](media/automation-graphical-authoring-intro/runbook-edit-test-pane.png)

测试控件会提示你输入参数，你可以通过单击“启动”按钮来启动 Runbook。

![测试控件按钮](media/automation-graphical-authoring-intro/runbook-test-start.png)

### 发布图形 Runbook

Azure 自动化中的每个 Runbook 都有草稿版和已发布版。只有已发布版才能用来运行，只有草稿版才能用来编辑。已发布版不受对草稿版所做的任何更改的影响。当草稿版可以使用时，你可以发布草稿版，这样草稿版就会覆盖已发布版。

发布图形 Runbook 时，你可以先打开要编辑的 Runbook，然后单击“发布”按钮即可。

![发布按钮](media/automation-graphical-authoring-intro/runbook-edit-publish.png)

当 Runbook 尚未发布时，其状态为“新建”。当 Runbook 发布后，其状态为“已发布”。如果你对已发布的 Runbook 进行编辑，而草稿版不同于已发布版，则 Runbook 的状态为“正在编辑”。

![Runbook 状态](media/automation-graphical-authoring-intro/runbook-statuses.png)

你还可以选择还原到 Runbook 的已发布版。这会抛弃自上次发布 Runbook 以来所做的任何更改，并会将 Runbook 的草稿版替换为已发布版。

![“恢复到已发布版本”按钮](media/automation-graphical-authoring-intro/runbook-edit-revert-published.png)


## 活动

活动是 Runbook 的构建基块。活动可以是一个 PowerShell cmdlet，一个子 Runbook，也可以是一个工作流活动。将活动添加到 Runbook 时，可以右键单击库控件中的该活动，然后选择“添加到画布”。然后，你可以通过单击拖放的方式将活动置于画布上你喜欢的任何位置。活动在画布上的位置不会以任何方式影响 Runbook 的运行。你可以对 Runbook 进行任何形式的布局，只要你觉得该布局最适合实现 Runbook 操作的可视化即可。

![添加到画布](media/automation-graphical-authoring-intro/add-to-canvas.png)

在画布上选择需要在“配置”边栏选项卡中配置其属性和参数的活动。你可以将活动的“标签”更改为你觉得更具描述性的内容。原始 cmdlet 仍在运行，你只是更改其要在图形编辑器中使用的显示名称。该标签在 Runbook 中必须是唯一的。

### 参数集

参数集用于定义会接受特定 cmdlet 的值的必需参数和可选参数。所有 cmdlet 都有至少一个参数集，一些 cmdlet 会有多个参数集。如果某个 cmdlet 有多个参数集，则你必须选择要使用的那个参数集，然后才能配置参数。你能够配置的参数将取决于你所选择的参数集。你可以更改某个活动使用的参数集，只需选中“参数集”，然后选择另一参数集即可。在这种情况下，你配置的任何参数值都将丢失。

在下面的示例中，Get-AzureVM cmdlet 有两个参数集。在你选择其中一个参数集之前，你将无法配置参数值。ListAllVMs 参数集可用于返回所有虚拟机，并具有一个可选参数。GetVMByServiceandVMName 可用于指定你希望返回的虚拟机，具有一个必需参数和两个可选参数。

![参数集](media/automation-graphical-authoring-intro/parameter-set.png)

#### 参数值

当你指定某个参数的值时，你可以选择一个数据源，以便确定如何指定该值。可用于特定参数的数据源将取决于该参数的有效值。例如，对于不允许 Null 值的参数，Null 不会是可用选项。

| 数据源 | 说明 |
|:---|:---|
|活动输出|工作流中某个位于当前活动前面的活动的输出。将列出所有有效的活动。只选择要将其输出用于参数值的活动。如果该活动输出的对象具有多个属性，你可以在选择活动之后键入属性的名称。|
|常量值|键入参数的值。这仅适用于以下数据类型：Int32、Int64、字符串、布尔值、DateTime、开关。 |
|空字符串|空字符串值。|
|Null|一个 Null 值。|
|PowerShell 表达式|指定简单的 PowerShell 表达式。对表达式的计算将先于用于参数值的活动和结果之前。你可以使用变量来引用活动或 Runbook 输入参数的输出。|
|取消选择|清除以前配置的任何值。|


#### 可选的其他参数

所有 cmdlet 都会有提供其他参数的选项。这些是 PowerShell 通用参数或其他自定义参数。系统会显示一个文本框，你可以在其中使用 PowerShell 语法提供参数。例如，若要使用 **Verbose** 通用参数，你可以指定 **"-Verbose:$True"**。

### 工作流脚本控件

工作流脚本控件是可以接受 PowerShell 工作流代码的特殊活动，用于提供在其他情况下也许不可用的功能。这不是完整的工作流，但必须包含有效的 PowerShell 工作流代码行。它不能接受参数，但可以使用针对活动输出和 Runbook 输入参数的变量。活动的任何输出都将添加到数据总线中，除非它没有传出链接，这种情况下，会将它添加到 Runbook 的输出中。

例如，下面的代码使用称为 $NumberOfDays 的 Runbook 输入变量执行日期计算。然后，它会将计算所得的日期时间作为输出发送，供 Runbook 中的后续活动使用。

    $DateTimeNow = InlineScript{(Get-Date).ToUniversalTime()}
    $DateTimeStart = InlineScript{($using:DateTimeNow).AddDays(-$using:NumberOfDays)}
	$DateTimeStart


## 链接和工作流

图形 Runbook 中的**链接**用于连接两个活动。它作为箭头显示在画布上，从源活动指向目标活动。活动按箭头的方向运行，源活动完成后才会开始目标活动。

### 创建链接

在两个活动之间创建链接，方法是：先选择源活动，然后单击图形底部的圆圈。将箭头拖到目标活动，然后放开。

![创建链接](media/automation-graphical-authoring-intro/create-link.png)

选择可在“配置”边栏选项卡中配置其属性的链接。这种情况下，将会包括下表中描述的链接类型。

| 链接类型 | 说明 |
|:---|:---|
| 管道 | 对于源活动中的每个对象输出，目标活动都将运行一次。如果源活动没有生成任何输出，目标活动将不会运行。源活动的输出可用作对象。 |
| 序列 | 目标活动只运行一次。它会接收来自源活动的对象数组。源活动的输出可用作对象数组。 |

### 启动活动

图形 Runbook 会通过任何没有传入链接的活动启动。这通常只是一个可充当 Runbook 启动活动的活动。如果多个活动没有传入链接，则 Runbook 在启动时，将并行运行这些活动。然后，它会在每个活动完成时，按链接来运行其他活动。

### 条件

当你指定链接的条件时，目标活动仅在该条件解析为 true 的情况下运行。通常会在条件中使用 $ActivityOutput 变量来检索源活动的输出。

对于管道链接，你可以为一个对象指定一个条件，然后针对源活动的每个对象输出对条件进行评估。然后会针对符合条件的每个对象运行目标活动。例如，通过 Get AzureVM 源活动，可以将以下语法用于条件管道链接，以便只检索当前正在运行的虚拟机。

	$ActivityOutput['Get-AzureVM'].PowerState -eq 'Started'

对于序列链接，只会对条件评估一次，因为会返回单个包含源活动中所有对象输出的数组。因此，不能使用序列链接来进行管道链接这样的筛选，但可以通过该链接来直接确定是否会运行下一活动。以下代码显示了同样的通过评估 Get-AzureVM 的输出来确定正在运行的虚拟机的示例。在这种情况下，该代码会遍历数组中的每个对象，并在至少有一个虚拟机运行时解析为 true。目标活动将负责分析该数据。

	$test = $false
	$VMs = $ActivityOutput['Get-AzureVm']
	Foreach ($VM in VMs)
	{
		If ($VM.PowerState –eq 'Started')
			{
				$test = $true
			}
	}
	$test

使用条件链接时，将通过条件来筛选该分支中从源活动到其他活动所提供的数据。如果某个活动是多个链接的源，则每个分支中可供活动使用的数据将取决于连接到该分支的链接中的条件。

例如，下面的 Runbook 中的源活动可获取所有虚拟机。它具有两个条件链接，以及一个不带条件的链接。第一个条件链接使用表达式 *$ActivityOutput['Get-AzureVM'].PowerState -eq 'Started'* 来筛选当前运行的虚拟机。第二个条件链接使用表达式 *$ActivityOutput['Get-AzureVM'].PowerState -eq 'Stopped'* 来筛选当前已停止的虚拟机。

![条件链接示例](media/automation-graphical-authoring-intro/conditional-links.png)

任何按第一个链接进行并使用 Get-AzureVM 提供的活动输出的活动都只会获取在 Get-AzureVM 运行时已启动的虚拟机。任何按第二个链接进行的活动都只会获取在 Get-AzureVM 运行时已停止的虚拟机。任何按第三个链接进行的活动都会获取所有虚拟机，不管这些虚拟机的运行状态如何。

### 交接点

一个交接点是一个特殊活动，它会一直等到所有传入分支完成为止。这可以让你并行运行多个活动，并确保在所有活动完成后再继续。

尽管一个接合点可以有无限数量的传入链接，但这些链接中可以成为管道的链接不能超过一个。传入序列链接的数目不受限制。系统允许你创建具有多个传入管道链接的结合点并保存 Runbook，但在运行时会失败。

下面的示例是某个 Runbook 的一部分，该 Runbook 可以启动一组虚拟机，同时还会下载要应用到这些虚拟机的修补程序。使用接合点是为了确保在 Runbook 继续之前两个进程都已完成。

![交接点](media/automation-graphical-authoring-intro/junction.png)

### 周期

一个周期是指目标活动需要多长时间才能通过链接回到其源活动，或者回到最终会通过链接回到其源活动的其他活动。目前不允许使用周期来进行图形创作。如果你的 Runbook 有一个周期，你可以正常保存它，但在运行它时会收到一个错误。

![周期](media/automation-graphical-authoring-intro/cycle.png)

### 循环

循环是指将某个活动重复指定次数，或者在满足特定条件之前一直重复该活动。图形 Runbook 目前不支持循环。

### 在活动之间共享数据

由活动通过传出链接输出的任何数据都会写入 Runbook 的*数据总线*。Runbook 中的任何活动都可以使用数据总线上的数据来填充参数值或添加脚本代码。一个活动可以访问工作流中任何以前的活动的输出。

数据写入数据总线的方式取决于活动的链接的类型。就**管道**来说，数据是作为多个对象输出的。就**序列**链接来说，数据是以数组形式输出的。如果只有一个值，数据将作为单个元素数组输出。

你可以使用两种方法之一访问数据总线上的数据。第一种方法是使用**活动输出**数据源来填充另一活动的参数。如果输出是一个对象，可以指定单个属性。

![活动输出](media/automation-graphical-authoring-intro/activity-output-datasource.png)

你还可以在 **PowerShell 表达式**数据源中检索活动的输出，或者使用 ActivityOutput 变量通过**工作流脚本**活动这样做。如果输出是一个对象，可以指定单个属性。ActivityOutput 变量使用以下语法。

	$ActivityOutput['Activity Label']
	$ActivityOutput['Activity Label'].PropertyName 

### 检查点

你的 Runbook 中用于设置[检查点](automation-runbook-concepts/#checkpoints)的指南同样也适用于图形 Runbook。你可以为需要在其中设置检查点的 Checkpoint-Workflow cmdlet 添加活动。然后，如果 Runbook 从其他辅助角色上的这个检查点启动，则应使用 Add-AzureAccount 来跟踪此活动。

## 通过 Azure 资源进行身份验证

Azure 自动化中的大多数 Runbook 将需要通过 Azure 资源进行身份验证。此身份验证所使用的典型方法是通过 Add-AzureAccount cmdlet 以及代表有权访问 Azure 帐户的 Active Directory 用户的[凭据资产](http://msdn.microsoft.com/zh-cn/library/dn940015.aspx)进行的。这在[配置 Azure 自动化](automation-configuring)中进行了讨论。

你可以将此功能添加到图形 Runbook，只需将凭据资产添加到画布，然后完成 Add-AzureAccount 活动即可。Add-AzureAccount 使用凭据活动作为其输入。下面的示例对此进行了演示。

![身份验证活动](media/automation-graphical-authoring-intro/authentication-activities.png)

你必须在 Runbook 启动时以及每个检查点之后进行身份验证。这意味着在每个 Checkpoint-Workflow 活动之后添加一个表示添加凭据的 Add-AzureAccount 活动。你不需要添加凭据活动，因为你可以使用相同的

![活动输出](media/automation-graphical-authoring-intro/authentication-activity-output.png)

## Runbook 输入和输出

### Runbook 输入

Runbook 可能会要求用户提供输入（如果该用户是通过 Azure 预览门户启动的该 Runbook），或者会要求另一 Runbook 提供输入（如果当前的 Runbook 是小孩在用）。例如，如果你使用 Runbook 创建虚拟机，则每次启动 Runbook 时，你可能需要提供虚拟机名称、其他属性等信息。

你可以通过定义一个或多个输入参数来接受 Runbook 的输入。每次启动 Runbook 时，你都需要为这些参数提供值。通过 Azure 预览门户启动 Runbook 时，该门户会提示你为每个 Runbook 的输入参数提供值。

你可以通过单击 Runbook 工具栏上的“输入和输出”按钮来访问 Runbook 的输入参数。

![Runbook 输入和输出](media/automation-graphical-authoring-intro/runbook-edit-input-output.png)

这种情况下将打开**输入和输出**控件，你可以通过单击“添加输入”在该控件中编辑现有输入参数或创建新的参数。

![添加输入](media/automation-graphical-authoring-intro/runbook-edit-add-input.png)

按下表中的属性定义每个输入参数。

|属性|说明|
|:---|:---|
| Name | 参数的唯一名称。此项只能包含字母数字字符，不能包含空格。 |
| 说明 | 针对输入参数的可选说明。 |
| 类型 | 参数值应有的数据类型。提示输入时，Azure 预览门户将针对每个参数的数据类型提供相应的控件。 |
| 必需 | 指定是否必须为该参数提供值。如果没有为每个没有定义默认值的必需参数提供值，将无法启动 Runbook。 |
| 默认值 | 指定在未提供值的情况下，对参数使用什么值。此项可以为 Null 或特定值。 |


### Runbook 输出

由任何没有传出链接的活动创建的数据将添加到 [Runbook 的输出](http://msdn.microsoft.com/zh-cn/library/azure/dn879148.aspx)。输出将与 Runbook 作业一起保存，在该 Runbook 作为子 Runbook 使用的情况下，还可供父 Runbook 使用。


## 相关文章

- [Azure 自动化 Runbook 概念](automation-runbook-concepts)
- [自动化资产](http://msdn.microsoft.com/zh-cn/library/azure/dn939988.aspx)

<!---HONumber=60-->