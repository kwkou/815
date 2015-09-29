<properties 
   pageTitle="Azure 自动化混合 Runbook 辅助角色"
   description="本文介绍了如何安装和使用混合 Runbook 辅助角色，该角色是 Azure 自动化的一项功能，可以用于在你本地数据中心的计算机上运行 Runbook。"
   services="automation"
   documentationCenter=""
   authors="bwren"
   manager="stevenka"
   editor="tysonn" />
<tags
   ms.service="automation"
   ms.date="07/10/2015"
   wacn.date="09/15/2015" />

# Azure 自动化混合 Runbook 辅助角色

Azure 自动化中的 Runbook 无法访问你本地数据中心的资源，因为它们运行在 Azure 云中。利用 Azure 自动化的混合 Runbook 辅助角色功能，你可以在位于数据中心的计算机上运行 Runbook，以便管理本地资源。Runbook 在 Azure 自动化中进行存储和管理，然后会发送到一个或多个本地计算机中运行。

下图说明了此功能。

![混合 Runbook 辅助角色概述](./media/automation-hybrid-runbook-worker/automation-hybrid-runbook-worker-overview.png)

你可以指定数据中心的一台或多台计算机充当混合 Runbook 辅助角色，然后通过 Azure 自动化运行 Runbook。每个辅助角色都需要可以连接到 Azure Operational Insights 和 Azure 自动化 Runbook 环境的 Microsoft 管理代理。Operational Insights 仅用于安装和维护管理代理并监视辅助角色的功能。Runbook 及其运行指令的传送由 Azure 自动化来执行。

![混合 Runbook 辅助角色组件](./media/automation-hybrid-runbook-worker/automation-hybrid-runbook-worker-components.png)

为混合 Runbook 辅助角色提供支持时，没有入站防火墙要求。本地计算机上的代理可启动与云中 Azure 自动化的所有通信。启动 Runbook 时，Azure 自动化会创建一种通过代理进行检索的指令。代理然后会拉取 Runbook 和任何参数，然后再运行 Runbook。它还会检索由 Azure 自动化中的 Runbook 使用的任何[资产](http://msdn.microsoft.com/zh-cn/library/dn939988.aspx)。

## 混合 Runbook 辅助角色组

每个混合 Runbook 辅助角色都是你在安装代理时指定的混合 Runbook 辅助角色组的成员。一个组可以包含一个代理，但是你可以在一个组中安装多个代理，以实现高可用性。

当你在混合 Runbook 辅助角色中启动 Runbook 时，你可以指定该辅助角色会在其中运行的组。组的成员会决定由哪个辅助角色来处理请求。你不能指定特定的辅助角色。

## 安装混合 Runbook 辅助角色

### 准备 Azure 自动化环境

完成以下步骤，以便针对混合 Runbook 辅助角色准备你的 Azure 自动化环境。

#### 1.创建 Azure Operational Insights 工作区  
如果你的 Azure 帐户中还没有 Operational Insights 工作区，则可按[设置 Operational Insights 工作区](https://technet.microsoft.com/zh-cn/library/mt484119.aspx)中的说明创建一个。如果你已经有一个工作区，则可以使用现有的。

#### 2.部署自动化解决方案  
Operational Insights 中的自动化解决方案可以向下推送所需组件，以便配置 Runbook 环境并提供相关支持。按照 [Operational Insights 解决方案](https://technet.microsoft.com/zh-cn/library/mt484119.aspx#1-add-solutions)中的说明安装 **Azure 自动化**包。

### 配置本地计算机
针对每台将充当混合 Runbook 辅助角色的本地计算机完成以下步骤。


#### 1.安装 Microsoft 管理代理  
Microsoft 管理代理可将计算机连接到 Operational Insights，并允许其运行解决方案中的逻辑。按照[将计算机直接连接到 Operational Insights](https://technet.microsoft.com/zh-cn/library/mt484108.aspx) 中的说明在本地计算机上安装代理，并将其连接到 Operational Insights。

#### 2.安装 Runbook 环境并连接到 Azure 自动化  
将计算机添加到 Operational Insights 时，自动化解决方案会向下推送 **HybridRegistration** PowerShell 模块，其中包含 **Add-HybridRunbookWorker** cmdlet。使用此 cmdlet 将 Runbook 环境安装到计算机上，并将其注册到 Azure 自动化。

在管理员模式下打开 PowerShell 会话，并运行以下命令以导入模块。

	Import-Module HybridRegistration

如果你收到一条错误，指出找不到模块文件，则你可能需要使用以下命令，该命令会使用模块文件的完整路径。

	Import-Module "C:\Program Files\Microsoft Monitoring Agent\Agent\AzureAutomationFiles\HybridRegistration\HybridRegistration.psd1"

然后，请使用以下语法运行 **Add-HybridRunbookWorker** cmdlet：

	Add-HybridRunbookWorker –Name <String> -EndPoint <Url> -Token <String>


- **名称**是指混合 Runbook 辅助角色组的名称。如果该组已经存在于自动化帐户中，则会将当前计算机添加到其中。如果该组不存在，则会创建它。
- **终结点**是指代理服务的 URL。你可以从 Azure 预览门户的“管理密钥”边栏选项卡上获得此项。  
- **令牌**是指“管理密钥”边栏选项卡中的**主访问密钥**。你可以通过单击自动化帐户的“元素”面板上的密钥图标来打开“管理密钥”边栏选项卡。  

	![混合 Runbook 辅助角色概述](./media/automation-hybrid-runbook-worker/elements-panel-keys.png)

#### 3.安装 PowerShell 模块  
Runbook 可以使用在 Azure 自动化环境中安装的模块中定义的任何活动和 cmdlet。不过，这些模块不会自动部署到本地计算机，因此必须手动安装。例外情况是 Azure 模块，该模块是默认安装的，可以用于访问所有 Azure 服务的 cmdlet 以及 Azure 自动化的活动。

由于混合 Runbook 辅助角色功能的主要用途是管理本地资源，你很可能需要安装支持这些资源的模块。你可以参考[安装模块](http://msdn.microsoft.com/zh-cn/library/dd878350.aspx)以获取有关安装 Windows PowerShell 模块的信息。

## 在混合 Runbook 辅助角色中启动 Runbook

[在 Azure 自动化中启动 Runbook](/documentation/articles/automation-starting-a-runbook) 介绍了用于启动 Runbook 的不同方法。混合 Runbook 辅助角色增加了一个 **RunOn** 选项，你可以在其中指定混合 Runbook 辅助角色组的名称。如果指定了组，则会由该组中的辅助角色检索和运行 Runbook。如果未指定此选项，则会在 Azure 自动化中正常运行 Runbook。

在 Azure 预览门户中启动 Runbook 时，你会看到一个 **RunOn** 选项，你可以在其中选择“Azure”或“混合辅助角色”。如果你选择“混合辅助角色”，则可以从下拉列表中选择该组。

使用 **RunOn** 参数，你可以使用以下命令通过 Windows PowerShell 在名为 MyHybridGroup 的混合 Runbook 辅助角色组中启动一个名为 Test-Runbook 的 Runbook。

	Start-AzureAutomationRunbook –AutomationAccountName "MyAutomationAccount" –Name "Test-Runbook" -RunOn "MyHybridGroup"

>[AZURE.NOTE]在 0.9.1 版的 Windows Azure PowerShell中，**RunOn** 参数已添加到 **Start-AzureAutomationRunbook** cmdlet。如果你安装的是旧版，则应[下载最新版本](/downloads)。你只需在要在其中通过 Windows PowerShell 启动 Runbook 的工作站上安装此版本。你不需要在辅助角色计算机上安装它，除非你要从该计算机启动 Runbook。你目前还不能通过其他 Runbook 在混合 Runbook 辅助角色上启动 Runbook，因为这需要在你的自动化帐户中安装最新版本的 Azure Powershell。最新版本将在 Azure 自动化中自动更新，并会快速地自动向下推送到辅助角色。


## 为混合 Runbook 辅助角色创建 Runbook

运行在 Azure 自动化中的 Runbook 和运行在混合 Runbook 辅助角色上的 Runbook 没有结构上的区别。上述两种 Runbook 使用起来可能会有很大差异，因为用于混合 Runbook 辅助角色的 Runbook 通常会管理你数据中心的本地资源，而 Azure 自动化中的 Runbook 通常会管理 Azure 云中的资源。

### Runbook 权限

在混合 Runbook 辅助角色中，Runbook 将在本地系统帐户的上下文中运行，因此必须针对要访问的资源进行身份验证。这些 Runbook 不能使用[通常用于针对 Azure 资源进行 Runbook 身份验证的方法](/documentation/articles/automation-configuring#configuring-authentication-to-azure-resources)，因为它们将要访问的资源位于 Azure 之外。

你可以在包含 cmdlet 的 Runbook 中使用[凭据](http://msdn.microsoft.com/zh-cn/library/dn940015.aspx)和[证书](http://msdn.microsoft.com/zh-cn/library/dn940013.aspx)资产，这些 cmdlet 可以让你指定凭据，方便你向不同资源进行身份验证。下面的示例显示了用于重新启动计算机的 Runbook 的一部分。它从凭据资产检索凭据，从变量资产检索计算机的名称，然后将这些值用于 Restart-Computer cmdlet。

	$Cred = Get-AutomationCredential "MyCredential"
	$Computer = Get-AutomationVariable "ComputerName"

	Restart-Computer -ComputerName $Computer  -Credential $Cred

你还可以利用 [InlineScript](/documentation/articles/automation-powershell-workflow#inline-script)，以便在其他由 [PSCredential 通用参数](http://technet.microsoft.com/zh-cn/library/jj129719.aspx)指定凭据的计算机上运行代码块。


### 创作和测试 Runbook

你可以在 Azure 自动化中编辑混合 Runbook 辅助角色的 Runbook，但如果你尝试在编辑器中测试 Runbook，则可能会遇到困难。用于访问本地资源的 PowerShell 模块可能没有安装在你的 Azure 自动化环境中，这种情况下，测试会失败。如果你安装了所需的模块，则 Runbook 会运行，但不能访问进行完整测试所需的本地资源。

## 与 Service Management Automation 的关系

[Service Management Automation (SMA)](https://technet.microsoft.com/zh-cn/library/dn469260.aspx) 是 Windows Azure Pack (WAP) 的组件，其用来运行的 Runbook 与你本地数据中心的 Azure 自动化所支持的 Runbook 相同。与 Azure 自动化不同，SMA 需要进行本地安装，安装内容包括 Windows Azure Pack 管理门户，以及用于保留 Runbook 和 SMA 配置的数据库。Azure 自动化在云中提供这些服务，只要求你在本地环境中维护混合 Runbook 辅助角色。

如果你已经是 SMA 用户，则可以将 Runbook 移到 Azure 自动化处与混合 Runbook 辅助角色一起使用，不需要进行任何更改，前提是这些 Runbook 向[为混合 Runbook 辅助角色创建Runbook](#creating-runbooks-for-hybrid-runbook-worker) 中所述的资源进行身份验证。SMA 中的 Runbook 在辅助角色服务器的服务帐户的上下文中运行，此服务器可以为 Runbook 提供该身份验证。

可以使用以下条件来确定是带有混合 Runbook 辅助角色的 Azure 自动化还是 Service Management Automation 更适合你的要求。

- SMA 需要在本地安装比 Azure 自动化需要更多本地资源和更高维护成本的 Windows Azure Pack，而 Azure 自动化只需在本地 Runbook 辅助角色中安装代理。代理由 Operational Insights 管理，这进一步降低了其维护成本。
- Azure 自动化在云中存储其 Runbook，然后将这些 Runbook 传送给本地混合 Runbook 辅助角色。如果你的安全策略不允许此行为，则应使用 SMA。
- Windows Azure Pack 可免费下载，而 Azure 自动化可能需要订阅费用。Azure。必须为 SMA 维护多个数据库。
- 使用带混合 Runbook 辅助角色的 Azure 自动化，你可以在一个位置管理云资源和本地资源的 Runbook，而不必分别管理 Azure 自动化和 SMA。
- Azure 自动化具有在 SMA 中不可用的图形创作等高级功能。 


## 相关文章

- [在 Azure 自动化中启动 Runbook](/documentation/articles/automation-starting-a-runbook)
- [在 Azure 自动化中编辑 Runbook](https://msdn.microsoft.com/zh-cn/library/dn879137.aspx)

<!---HONumber=69-->