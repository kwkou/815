<properties
    pageTitle="我在 Azure 自动化中的第一个 PowerShell Runbook | Azure"
    description="本教程将指导你创建、测试和发布一个简单的 PowerShell Runbook。"
    services="automation"
    documentationCenter=""
    authors="mgoedtel"
    manager="stevenka"
    editor="tysonn"/>

<tags
    ms.service="automation"
    ms.date="02/23/2016"
    wacn.date="04/20/2016"/>

# 我的第一个 PowerShell Runbook

> [AZURE.SELECTOR]
- [PowerShell 工作流](/documentation/articles/automation-first-runbook-textual)
- [PowerShell](/documentation/articles/automation-first-runbook-textual-PowerShell)

本教程将指导你在 Azure 自动化中创建 PowerShell Runbook。我们将从一个简单的 Runbook 开始，我们将测试和发布该 Runbook，同时介绍如何跟踪 Runbook 作业的状态。然后我们将通过修改 Runbook 来实际管理 Azure 资源，这种情况下将启动 Azure 虚拟机。然后我们将通过添加 Runbook 参数来使该 Runbook 更稳健。

## 先决条件

为了完成本教程，你需要满足以下条件。

-	Azure 订阅创建新存储帐户。如果你还没有帐户，则可以[注册获取试用版](/pricing/1rmb-trial)。
-	用于保存 Runbook 的[自动化帐户](/documentation/articles/automation-configuring)。
-	Azure 虚拟机。我们将停止并启动该计算机，因此其不应为生产用计算机。
-	用于对 Azure 资源进行身份验证的 [Azure Active Directory 和自动化资产](/documentation/articles/automation-configuring)。此用户必须有权启动和停止虚拟机。

## 步骤 1 - 创建新的 Runbook

我们首先创建一个输出文本 *Hello World* 的简单 Runbook 。

1.	在 Azure 管理门户中，打开你的自动化帐户。通过自动化帐户页面可快速查看此帐户中的资源。你应该已拥有某些资产。大多数资产都是自动包括在新的自动化帐户中的模块。你还应具有在[先决条件](#prerequisites)中提到的凭证资产。
2.	单击“Runbook”磁贴打开 Runbook 列表。 
3.	通过单击“添加 Runbook”按钮创建一个新 Runbook，然后“创建新 Runbook”。
4.	将该 Runbook 命名为 *MyFirstRunbook-PowerShell*。
5.	在本例中，我们将要创建一个 PowerShell Runbook，因此请选择“Powershell”作为“Runbook 类型”。  
6.	单击“创建”以创建 Runbook 并打开文本编辑器。

## 步骤 2 - 将代码添加到 Runbook

你可以直接将代码键入 Runbook 中，或者通过“库”控件选择 cmdlet、Runbook 和资产，并使用任何相关的参数将它们添加到 Runbook。在本演练中，我们将直接键入 Runbook。

1.	我们的 Runbook 目前是空的，请键入 *Write-Output "Hello World."*。
	
	![Hello World](./media/automation-first-runbook-textual-powershell/automation-helloworld.png)  
2.	单击“保存”以保存 Runbook。  

## 步骤 3 - 测试 Runbook

在我们发布 Runbook 使其可在生产中使用之前，我们要对其进行测试以确保其能正常工作。测试 Runbook 时，你可以运行其“草稿”版本并以交互方式查看其输出。

1.	单击“测试窗格”打开测试窗格。  
2.	单击“启动”以启动测试。这应该是唯一的已启用选项。
3.	将创建一个 [Runbook 作业](/documentation/articles/automation-runbook-execution)并显示其状态。作业状态初始为“排队”，该值指示它正在等待云中的 Runbook 辅助角色可用。在某个辅助角色认领该作业后，该作业状态将变为“正在启动”，然后当 Runbook 实际开始运行时，该作业状态将变为“正在运行”。  
4.	Runbook 作业完成后，会显示其输出。在本例中，我们应该看到 *Hello World*  
5.	关闭测试窗格以返回到画布。

## 步骤 4 - 发布和启动 Runbook

我们刚刚创建的 Runbook 仍处于草稿模式。我们需要发布该 Runbook，然后才可将其用于生产。当发布 Runbook 时，你可以用草稿版本覆盖现有的已发布版本。在本例中，我们还没有已发布版本，因为我们刚刚创建 Runbook。

1.	单击“发布”以发布该 Runbook，然后在出现提示时单击“是”。  
2.	如果你现在向左滚动以在 **Runbooks** 窗格中查看该 Runbook，它将显示“已发布”的“创作状态”。
3.	向右滚动查看 **MyFirstRunbook-PowerShell** 的窗格。顶部的选项允许我们启动 Runbook、查看 Runbook，以及计划其在将来的某个时刻启动，以使其可以通过 HTTP 调用来启动。
4.	我们只需启动 Runbook，因此请单击“启动”，然后在“启动 Runbook”边栏选项卡打开时单击“确定”即可。  
5.	会为我们刚刚创建的 Runbook 作业打开作业窗格。我们可以关闭此窗格，但在这种情况下我们将其保持打开，以便可以查看该作业的进度。
6.	作业状态显示在“作业摘要”中并且与我们在测试该 Runbook 时看到的状态相匹配。  
7.	一旦此 Runbook 状态显示“已完成”，单击“输出”。“输出”窗格将打开，并且我们可以看到 *Hello World*。  
8.	关闭“输出”窗格。
9.	单击“所有日志”打开 Runbook 作业的“流”窗格。应该只会在输出流中看到 *Hello World*，但此窗格也可以显示 Runbook 作业的其他流，例如，“详细”和“错误”（如果 Runbook 向其写入）。  
10.	关闭“流”窗格和“作业”窗格以返回到 MyFirstRunbook-PowerShell 窗格。
11.	单击“作业”打开此 Runbook 的“作业”窗格。这将列出此 Runbook 创建的所有作业。由于我们只运行该作业一次，应该只会看到一个列出的作业。
12.	你可以在此作业上单击以打开我们启动 Runbook 时查看的相同的作业窗格。这样你就可以回溯并查看为特定 Runbook 创建的任何作业的详细信息。

## 步骤 5 - 添加身份验证来管理 Azure 资源

我们已经测试并发布 Runbook，但到目前为止它不执行任何有用的操作。我们想要让其管理 Azure 资源。然而，除非使用[先决条件](#prerequisites)中引用的凭据对其进行身份验证，否则它将无法进行管理。我们使用 **Add-AzureAccount** cmdlet 实现此目的。

1.	通过单击 MyFirstRunbook-PowerShell 窗格上的“编辑”打开文本编辑器。  
2.	我们不再需要 **Write-Output** 行，因此请直接删除它。
3.	在“库”控件中，展开“资产”，然后展开“凭据”。
4.	右键单击你的凭据，然后单击“添加到画布”。这将会添加凭据的 **Get-AutomationPSCredential** 活动。
5.	在 **Get-AutomationPSCredential** 前面，输入 *$Credential =* 以将凭据分配给变量。
6.	在下一行中键入 *Add-AzureAccount -Environment AzureChinaCloud -Credential $Credential*。  
	
	![凭据](./media/automation-first-runbook-textual-powershell/automation-get-credential.png)
7.	单击“测试”窗格，以便我们可以测试 Runbook。 
8.	单击“启动”以启动测试。完成后，你会获得包含帐户的订阅 ID、类型、租户的输出。这是对凭据有效的确认。

## 步骤 6 – 添加用于启动虚拟机的代码

在 Runbook 对 Azure 订阅进行身份验证后，我们可以管理资源。我们将添加一个命令用于启动虚拟机。你可以在你的 Azure 订阅中选取任何虚拟机，现在我们会将该名称硬编码到 cmdlet 中。

1.	在 *Add-AzureAccount -Environment AzureChinaCloud* 后面，键入 *Start-AzureVM -Name 'VMName' -ServiceName 'VMServiceName'* 并提供要启动的虚拟机的名称和服务名称。  
	
	![StartVM](./media/automation-first-runbook-textual-powershell/automation-startvm.png)  
2.	保存 Runbook，然后单击“测试”窗格，以便我们可以测试 Runbook。
3.	单击“启动”以启动测试。一旦测试完成后，检查已启动的虚拟机。

## 步骤 7 - 将输入参数添加到 Runbook

我们的 Runbook 目前会启动我们在 Runbook 中硬编码的虚拟机，但如果可以在启动 Runbook 时指定虚拟机，它会更有用。我们现在将输入参数添加到 Runbook，以提供该功能。

1.	将 *VMName* 和 *VMServiceName* 的参数添加到 Runbook，并将这些变量与 **Start-AzureVM** cmdlet 配合使用，如下图所示。

	![添加参数](./media/automation-first-runbook-textual-powershell/automation-add-parameter.png)  
2.	保存 Runbook 并打开“测试”窗格。请注意，现在可以为将在测试中使用的两个输入变量提供值。
3.	关闭“测试”窗格。
4.	单击“发布”以发布 Runbook 的新版本。
5.	停止在上一步中启动的虚拟机。
6.	单击“启动”以启动 Runbook。键入要启动的虚拟机的 **VMName** 和 **VMServiceName**。  
7.	一旦 Runbook 完成后，检查已启动的虚拟机。

## 与 PowerShell 工作流的差异

PowerShell Runbook 与 PowerShell 工作流 Runbook 具有相同的生命周期、功能和管理，但存在一些差异和限制：

1.	PowerShell Runbook 比 PowerShell 工作流 Runbook 的运行速度更快，因为没有编译步骤。
2.	PowerShell 工作流 Runbook 支持检查点。使用检查点，PowerShell 工作流 Runbook 可以从 Runbook 中的任意点恢复，而 PowerShell Runbook 则只能从开始恢复。
3.	PowerShell 工作流 Runbook 支持并行和串行执行，而 PowerShell Runbook 则只能通过串行方式执行命令。
4.	在 PowerShell 工作流 Runbook 中，活动、命令或脚本块可以有自己的运行空间，而在 PowerShell Runbook 中，脚本中的所有内容都在单个运行空间中运行。本机 PowerShell Runbook 和 PowerShell 工作流 Runbook 之间还存在一些[语法差异](https://technet.microsoft.com/magazine/dn151046.aspx)。

## 后续步骤

-	若要开始使用 PowerShell 工作流 Runbook，请参阅 [My first PowerShell workflow runbook（我的第一个 PowerShell 工作流 Runbook）](/documentation/articles/automation-first-runbook-textual)
-	有关 PowerShell 脚本支持功能的详细信息，请参阅 [Native PowerShell script support in Azure Automation（Azure 自动化中的本机 PowerShell 脚本支持）](https://azure.microsoft.com/blog/announcing-powershell-script-support-azure-automation-2)

<!---HONumber=Mooncake_0411_2016-->
