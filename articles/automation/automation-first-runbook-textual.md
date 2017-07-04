<properties
    pageTitle="我在 Azure 自动化中的第一个 PowerShell 工作流 Runbook | Azure"
    description="本教程指导你使用 PowerShell 工作流创建、测试和发布一个简单的文本 Runbook。"
    services="automation"
    documentationcenter=""
    author="mgoedtel"
    manager="jwhit"
    editor=""
    keywords="powershell 工作流, powershell 工作流示例, 工作流 powershell"
    translationtype="Human Translation" />
<tags
    ms.assetid="0002d7f7-e2b5-46e3-b5eb-4596b84fd526"
    ms.service="automation"
    ms.workload="tbd"
    ms.tgt_pltfrm="na"
    ms.devlang="na"
    ms.topic="get-started-article"
    ms.date="03/26/2017"
    wacn.date="05/02/2017"
    ms.author="magoedte;bwren"
    ms.sourcegitcommit="78da854d58905bc82228bcbff1de0fcfbc12d5ac"
    ms.openlocfilehash="f5b71edf8211b2cfa4cb802eb72d15e5e27eb2d9"
    ms.lasthandoff="04/22/2017" />

# <a name="my-first-powershell-workflow-runbook"></a>我的第一个 PowerShell 工作流 Runbook

本教程将指导你在 Azure 自动化中创建 PowerShell 工作流 Runbook。 我们将从一个简单的 Runbook 开始，我们将测试和发布该 Runbook，同时介绍如何跟踪 Runbook 作业的状态。 然后我们将通过修改 Runbook 来实际管理 Azure 资源，这种情况下将启动 Azure 虚拟机。 然后我们将通过添加 Runbook 参数来使该 Runbook 更稳健。

## <a name="prerequisites"></a>先决条件
若要完成本教程，需要满足以下条件：

* Azure 订阅。 如果还没有，可以[注册一个帐户](/pricing/1rmb-trial)。
* 自动化帐户 ，用来保存 Runbook 以及向 Azure 资源进行身份验证。 此帐户必须有权启动和停止虚拟机。
* Azure 虚拟机。 我们需停止和启动该虚拟机，因此其不应为生产用 VM。

## <a name="step-1---create-new-runbook"></a>步骤 1 - 创建新的 Runbook
我们首先创建一个输出文本 *Hello World*的简单 Runbook 。

1. 在 Azure 经典管理门户中，单击“新建” > “应用服务” > “自动化” > “RUNBOOK” > “快速创建”
2. 输入 **RUNBOOK 名称**和**说明**，然后选择一个**自动化帐户**。 如果没有自动化帐户，可以通过选择“创建新的自动化帐户”并输入**帐户名**来创建一个
3. 创建 Runbook 后，可在自动化帐户的“RUNBOOK”磁贴下找到该 Runbook。
4. 单击 Runbook 的“创作”磁贴，以启用文本编辑器。

## <a name="step-2---add-code-to-the-runbook"></a>步骤 2 - 将代码添加到 Runbook
你可以直接将代码键入 Runbook 中，或者通过“库”控件选择 cmdlet、Runbook 和资产，并使用任何相关的参数将它们添加到 Runbook。 在本演练中，我们将直接键入 Runbook。

1. 我们的 Runbook 目前是空的，只有必需的 *workflow* 关键字、Runbook 名称以及括住整个工作流的大括号。

        Workflow MyFirstRunbook-Workflow
        {
        }

2. 在大括号之间键入 *Write-Output "Hello World"* 。

        Workflow MyFirstRunbook-Workflow
        {
        Write-Output "Hello World"
        }

3. 通过单击“保存” 保存 Runbook。

## <a name="step-3---test-the-runbook"></a>步骤 3 - 测试 Runbook
在我们发布 Runbook 使其可在生产中使用之前，我们要对其进行测试以确保其能正常工作。 测试 Runbook 时，你可以运行其“草稿”版本并以交互方式查看其输出  。

2. 单击“测试”打开“测试”窗格。 然后单击“是”进行确认。
3. 几秒钟后，将看到输出窗格中显示消息“Hello World”。

## <a name="step-4---publish-and-start-the-runbook"></a>步骤 4 - 发布和启动 Runbook
我们刚刚创建的 Runbook 仍处于草稿模式。 我们需要发布该 Runbook，然后才可将其用于生产。 当发布 Runbook 时，你可以用草稿版本覆盖现有的已发布版本。 在本例中，我们还没有已发布版本，因为我们刚刚创建 Runbook。

1. 单击“发布”以发布该 Runbook，然后在出现提示时单击“是”。 
2. 如果现在在自动化帐户的“Runbook”磁贴中查看该 Runbook，其“创作状态”将显示为“已发布”。
4. 单击“启动”以启动 Runbook，然后在出现提示时单击“是”。
5. 单击“查看作业”以查看刚刚启动的作业的摘要。
6. 作业状态显示在“作业摘要”中，几秒钟后，“输出”下面会显示前面执行测试后的相同输出。
9. 返回 Runbook。 在“作业”磁贴下，可见筛选出了已创建的作业。
10. 单击一个作业，可看到该作业的“摘要”和“历史记录”。 

## <a name="step-5---add-authentication-to-manage-azure-resources"></a>步骤 5 - 添加身份验证来管理 Azure 资源
我们已经测试并发布 Runbook，但到目前为止它不执行任何有用的操作。 我们想要让其管理 Azure 资源。 然而，除非使用[“先决条件”](#prerequisites)中引用的凭据对其进行身份验证，否则它将无法进行管理 。 我们使用 **Add-AzureRMAccount** cmdlet 实现此目的。

1. 通过单击“创作”磁贴 >“草稿”标记 >“编辑 Runbook”打开文本编辑器。
2. 我们不再需要 **Write-Output** 行，因此请直接删除它。
3. 将光标放在大括号之间的空白行上。
3. 单击“插入” > “设置” > “获取 Windows PowerShell 凭据”，然后选择所需的凭据。
4. 如果没有凭据，可单击“管理” > “添加凭据”以创建并添加凭据。
5. 在 **Get-AutomationPSCredential**前面，输入 *$Credential =* 以将凭据分配给变量。 
3. 在下一行中键入 *Add-AzureRmAccount -Credential $Credential -EnvironmentName AzureChinaCloud*。

        workflow test
        {
            $Credential = Get-AutomationPSCredential -Name "<your credential>"
            Add-AzureRmAccount -Credential $Credential -EnvironmentName AzureChinaCloud
        }

3. 单击“测试”，然后在出现提示时单击“是”。
10.  完成后，你应会收到类似于下面的输出，这是向凭据中的用户返回的信息。  这是对凭据有效的确认。

        PSComputeerName            : localhost    PSSourceJobInstanceID    : xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx    Id                        : azureuser@contoso.com    类型                    : 用户    订阅            : xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx    租户                    : xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx

## <a name="step-6---add-code-to-start-a-virtual-machine"></a>步骤 6 – 添加用于启动虚拟机的代码
现在我们的 Runbook 要对 Azure 订阅进行身份验证，我们可以管理资源。 我们将添加一个命令，用于启动虚拟机。 你可以在 Azure 订阅中选取任何虚拟机。而现在，我们需将该名称硬编码到 Runbook。

1. 在 *Add-AzureRmAccount* 后面，键入 *Start-AzureRmVM -Name 'VMName' -ResourceGroupName 'NameofResourceGroup'*（提供要启动的虚拟机的名称和资源组名称）。  

        workflow MyFirstRunbook-Workflow
        {
        $Conn = Get-AutomationConnection -Name AzureRunAsConnection
        Add-AzureRMAccount –EnvironmentName AzureChinaCloud -ServicePrincipal -Tenant $Conn.TenantID -ApplicationId $Conn.ApplicationID -CertificateThumbprint $Conn.CertificateThumbprint
        Start-AzureRmVM -Name 'VMName' -ResourceGroupName 'ResourceGroupName'
        }

2. 保存 Runbook，然后单击“测试”以便测试 Runbook。

## <a name="step-7---add-an-input-parameter-to-the-runbook"></a>步骤 7 - 将输入参数添加到 Runbook
我们的 Runbook 目前会启动我们在 Runbook 中硬编码的虚拟机，但如果可以在启动 Runbook 时指定虚拟机，它会更有用。 我们现在将输入参数添加到 Runbook，以提供该功能。

1. 将 *VMName* 和 *ResourceGroupName* 的参数添加到 Runbook，并将这些变量与 **Start-AzureRmVM** cmdlet 配合使用，如以下示例所示。

        workflow MyFirstRunbook-Workflow
        {
         Param(
          [string]$VMName,
          [string]$ResourceGroupName
         )  
        $Conn = Get-AutomationConnection -Name AzureRunAsConnection
        Add-AzureRMAccount –EnvironmentName AzureChinaCloud -ServicePrincipal -Tenant $Conn.TenantID -ApplicationId $Conn.ApplicationID -CertificateThumbprint $Conn.CertificateThumbprint
        Start-AzureRmVM -Name $VMName -ResourceGroupName $ResourceGroupName
        }

2. 保存 Runbook 并打开“测试”窗格。 请注意，现在可以为将在测试中使用的两个输入变量提供值。
3. 关闭“测试”窗格。
4. 单击“发布”以发布 Runbook 的新版本  。
5. 停止在上一步中启动的虚拟机。
6. 单击“启动”以启动 Runbook 。 键入要启动的虚拟机的 **VMName** 和 **ResourceGroupName**。
7. 一旦 Runbook 完成后，检查已启动的虚拟机。

<!--Update_Description: wording update-->