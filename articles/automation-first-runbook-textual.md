<properties
    pageTitle="我在 Azure 自动化中的第一个 PowerShell 工作流 Runbook | Azure"
    description="本教程指导你使用 PowerShell 工作流创建、测试和发布一个简单的文本 Runbook。"
    services="automation"
    documentationCenter=""
    authors="mgoedtel"
    manager="stevenka"
    editor=""/>

<tags
    ms.service="automation"
    ms.date="05/16/2016"
    wacn.date="06/30/2016"/>


# 我的第一个 PowerShell 工作流 Runbook

本教程将指导你在 Azure 自动化中创建 PowerShell 工作流 Runbook。我们将从一个简单的 Runbook 开始，我们将测试和发布该 Runbook，同时介绍如何跟踪 Runbook 作业的状态。然后我们将通过修改 Runbook 来实际管理 Azure 资源，这种情况下将启动 Azure 虚拟机。然后我们将通过添加 Runbook 参数来使该 Runbook 更稳健。

## 先决条件

为了完成本教程，你需要满足以下条件。

- Azure 订阅创建新存储帐户。如果你没有订阅，可以[注册试用版](/pricing/1rmb-trial)。
- 用于保存 Runbook 的自动化帐户。
- Azure 虚拟机。我们将停止并启动该计算机，因此其不应为生产用计算机。
- 用于对 Azure 资源进行身份验证的 Azure Active Directory 和自动化资产。此用户必须有权启动和停止虚拟机。

## 步骤 1 - 创建新的 Runbook

我们首先创建一个输出文本 *Hello World* 的简单 Runbook 。

1. 在 Azure 经典管理门户中，单击“新建”>“应用程序服务”>“自动化”>“RUNBOOK”>“快速创建”。
2. 输入 **RUNBOOK 名称**和**说明**，然后选择一个**自动化帐户**。如果你没有自动化帐户，可以通过选择“创建新的自动化帐户”并输入“帐户名”来创建一个
3. 创建 Runbook 后，可以在自动化帐户的“RUNBOOK”磁贴中找到它。
4. 单击 Runbook 的“创作”磁贴以启用文本编辑器。

## 步骤 2 - 将代码添加到 Runbook

你可以直接将代码键入 Runbook 中，或者通过“库”控件选择 cmdlet、Runbook 和资产，并使用任何相关的参数将它们添加到 Runbook。在本演练中，我们将直接键入 Runbook。

1.	我们的 Runbook 目前是空的，只有必需的工作流关键字、Runbook 名称以及括住整个工作流的大括号。

	    Workflow MyFirstRunbook-Workflow
	    {
	    }

2.	在括号之间键入 *Write-Output "Hello World"*。

	    Workflow MyFirstRunbook-Workflow
	    {
	      Write-Output "Hello World"
	    }

3.	单击“保存”以保存 Runbook。<br>

## 步骤 3 - 测试 Runbook

在我们发布 Runbook 使其可在生产中使用之前，我们要对其进行测试以确保其能正常工作。测试 Runbook 时，你可以运行其“草稿”版本并以交互方式查看其输出。

2. 单击“测试”打开“测试”窗格。然后单击“是”确认。
3. 几秒钟后，你将看到输出窗格中显示消息“Hello World”。

## 步骤 4 - 发布和启动 Runbook

我们刚刚创建的 Runbook 仍处于草稿模式。我们需要发布该 Runbook，然后才可将其用于生产。当发布 Runbook 时，你可以用草稿版本覆盖现有的已发布版本。在本例中，我们还没有已发布版本，因为我们刚刚创建 Runbook。

1. 单击“发布”以发布该 Runbook，然后在出现提示时单击“是”。 
2. 如果你在自动化帐户的“Runbook”磁贴中查看该 Runbook，它的“创作状态”会显示为“已发布”。
4. 单击“启动”以启动 Runbook，然后在出现提示时单击“是”。
5. 单击“查看作业”以查看刚刚启动的作业的摘要。
6. 作业状态显示在“作业摘要”中，几秒钟后，“输出”下面会显示前面执行测试后的相同输出。
9. 返回到你的 Runbook。在“作业”磁贴下，你可以看到创建的作业列表将筛选上述 Runbook。
10. 单击一个作业，你可以看到该作业的“摘要”和“历史记录”。 

## 步骤 5 - 添加身份验证来管理 Azure 资源

我们已经测试并发布 Runbook，但到目前为止它不执行任何有用的操作。我们想要让其管理 Azure 资源。然而，除非使用[先决条件](#prerequisites)中引用的凭据对其进行身份验证，否则它将无法进行管理。我们使用 **Add-AzureAccount** cmdlet 实现此目的。

1.  通过单击“创作”磁贴 >“草稿”标记 >“编辑 Runbook”打开文本编辑器。
2.  我们不再需要 **Write-Output** 行，因此请直接删除它。
3.  将光标放在大括号之间的空白行上。
3.  单击“插入”>“设置”>“获取 Windows PowerShell 凭据”，然后选择所需的凭据。
4.  如果你没有凭据，可以通过单击“管理”>“添加凭据”来创建一个凭据。
5.  在 **Get-AutomationPSCredential** 前面，输入 *$Credential =* 以将凭据分配给变量。 
3.  在下一行中键入 *Add-AzureAccount -Credential $Credential –Environment AzureChinaCloud*。

		workflow test
		{
    		$Credential = Get-AutomationPSCredential -Name "<your credential>"
    		Add-AzureAccount –Credential $Credential –Environment AzureChinaCloud
		}

3. 单击“测试”，然后在出现提示时单击“是”。
10.  完成后，你应会收到类似于下面的输出，这是向凭据中的用户返回的信息。其中确认凭据有效。

		PSComputeerName			: localhost
		PSSourceJobInstanceID	: xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx
		Id						: azureuser@contoso.com
		Type					: User
		Subscriptions			: xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx
		Tenants					: xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx

## 步骤 6 – 添加用于启动虚拟机的代码

在 Runbook 对 Azure 订阅进行身份验证后，我们可以管理资源。我们将添加一个命令用于启动虚拟机。你可以在你的 Azure 订阅中选取任何虚拟机，现在我们会将该名称硬编码到 cmdlet 中。

1. 在 *Add-AzureAccount* 后面，键入 *Start-AzureVM -Name 'VMName' -ServiceName 'VMServiceName'* 并提供要启动的虚拟机的名称和服务名称。 

		workflow test
		{
    		$Credential = Get-AutomationPSCredential -Name "<your credential>"
    		Add-AzureAccount –Credential $Credential –Environment AzureChinaCloud
    		Start-AzureVM -Name "<your vm>" -ServiceName "<your vm service>"
		}

9. 保存 Runbook，然后单击“测试”，以便我们可以测试 Runbook。

## 步骤 7 - 将输入参数添加到 Runbook

我们的 Runbook 目前会启动我们在 Runbook 中硬编码的虚拟机，但如果可以在启动 Runbook 时指定虚拟机，它会更有用。我们现在将输入参数添加到 Runbook，以提供该功能。

1. 将 *VMName* 和 *VMServiceName* 的参数添加到 Runbook，并将这些变量与 **Start-AzureVM** cmdlet 配合使用，如下图所示。<br>

		workflow test
		{
    		Param (
        		[string]$VMName,
        		[String]$VMServiceName
    		)
    
    		$Credential = Get-AutomationPSCredential -Name "<your credential>"
    		Add-AzureAccount –Credential $Credential –Environment AzureChinaCloud
    		Start-AzureVM -Name $VMName -ServiceName $VMServiceName
		}

9. 保存 Runbook 并打开“测试”窗格。请注意，现在可以为将在测试中使用的两个输入变量提供值。
11.  关闭“测试”窗格。
12.  单击“发布”以发布 Runbook 的新版本。
13.  停止在上一步中启动的虚拟机。
13.  单击“启动”以启动 Runbook。键入要启动的虚拟机的 **VMName** 和 **VMServiceName**。
14.  一旦 Runbook 完成后，检查已启动的虚拟机。

<!---HONumber=Mooncake_0411_2016-->