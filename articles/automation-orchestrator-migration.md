<properties
   pageTitle="从 Orchestrator 迁移到 Azure 自动化 | Windows Azure"
   description="介绍如何将 Runbook 和集成包从 System Center Orchestrator 迁移到 Azure 自动化。"
   services="automation"
   documentationCenter=""
   authors="bwren"
   manager="stevenka"
   editor="tysonn" />
<tags
   ms.service="automation"
   ms.date="08/18/2015"
   wacn.date="09/15/2015" />


# 从 Orchestrator 迁移到 Azure 自动化

[System Center Orchestrator](http://technet.microsoft.com/zh-cn/library/hh237242.aspx) 中的 Runbook 基于专为 Orchestrator 编写的集成包中的活动，而 Azure 自动化中的 Runbook 则基于 Windows PowerShell 工作流。Azure 自动化中的图形 Runbook 具有的外观类似于其活动用于表示 PowerShell cmdlet、子 Runbook 和资产的 Orchestrator Runbook。

[System Center Orchestrator 迁移工具包](http://www.microsoft.com/download/details.aspx?id=47323&WT.mc_id=rss_alldownloads_all)包含的工具可帮助你将 Runbook 从 Orchestrator 转换为 Azure 自动化。除了转换 Runbook 本身，你还必须将包含所用活动的集成包转换为包含 Windows PowerShell cmdlet 的集成模块。

下面是将 Orchestrator Runbook 转换为 Azure 自动化的基本过程。这每个步骤都在下面的相应部分进行了详细介绍。

1.  下载包含本文讨论的工具和模块的 [System Center Orchestrator 迁移工具包](http://www.microsoft.com/download/details.aspx?id=47323&WT.mc_id=rss_alldownloads_all)。
2.  将[标准活动模块](#standard-activities-module)安装到 Azure 自动化中。这包括标准 Orchestrator 活动的转换后版本，可供已转换的 Runbook 使用。
2.  针对 Runbook 使用的集成包，在 Azure 自动化中安装 [System Center Orchestrator 集成模块](#system-center-orchestrator-integration-modules)。
3.  使用[集成包转换器](#integration-pack-converter)转换自定义的和第三方的集成包，并在 Azure 自动化中进行安装。
4.  在 Azure 自动化的 Orchestrator 中通过手动方式重新创建全局资产，因为没有自动化方法来执行此迁移。
5.  使用 [Runbook 转换器](#runbook-converter-coming-soon)（即将推出）来转换 Orchestrator Runbook，并在 Azure 自动化中进行安装。
6.  在本地数据中心配置[混合 Runbook 辅助角色](#hybrid-runbook-worker)，以便运行转换的 Runbook。

## Service Management Automation

[Service Management Automation](https://technet.microsoft.com/zh-cn/library/dn469260.aspx) (SMA) 在本地数据中心（如 Orchestrator）运行 Runbook，并使用相同的集成模块（如 Azure 自动化）。当它可用时，[Runbook 转换器](#runbookconverter)会将 Orchestrator Runbook 转换为图形 Runbook，虽然 SMA 中并不支持图形 Runbook。你仍可将[标准活动模块](#standardactivitiesmodule)以及任何已转换的[集成包](#integrationpackconverter)安装到 SMA 中，但必须通过手动方式[重新编写 Runbook](http://technet.microsoft.com/zh-cn/library/dn469262.aspx)。

## 混合 Runbook 辅助角色

Orchestrator 中的 Runbook 存储在数据库服务器上，运行在 Runbook 服务器上，这两种服务器都位于本地数据中心。Azure 自动化中的 Runbook 存储在 Azure 云中，并可使用[混合 Runbook 辅助角色](/documentation/articles/automation-hybrid-runbook-worker)运行在本地数据中心。这是通常情况下运行从 Orchestrator 转换过来的 Runbook 的方式，因为这些 Runbook 是设计在本地服务器上运行的。

## 集成包转换器

集成包转换器会将使用 Orchestrator 集成工具包 (OIT) 创建的集成包转换成基于 Windows PowerShell 并可导入 Azure 自动化或 Service Management Automation 中的集成模块。

当你运行集成包转换器时，系统将显示一个向导，你可以通过该向导选择集成包 (.oip) 文件。然后，该向导会列出该集成包中包括的活动，并允许你选择要迁移的活动。完成向导的操作后，向导会创建一个模块，其中包含原始集成包中每个活动的相应 cmdlet。


### Parameters

集成包中活动的任何属性都将转换为集成模块中相应 cmdlet 的参数。Windows PowerShell cmdlet 有一组可以用于所有 cmdlet 的[通用参数](http://technet.microsoft.com/zh-cn/library/hh847884.aspx)。例如，-Verbose 参数会导致 cmdlet 输出关于其操作的详细信息。cmdlet 的参数与通用参数不能有相同的名称。如果某个活动的属性与通用参数具有相同的名称，向导会提示你为参数提供另一个名称。

### 监视活动

Orchestrator 中的监视 Runbook 以[监视活动](http://technet.microsoft.com/zh-cn/library/hh403827.aspx)开头，并会持续运行，等待被特定事件调用。Azure 自动化不支持监视 Runbook，因此集成包中的任何监视活动都不会进行转换。与之相反，系统会在集成模块中为监视活动创建一个占位符 cmdlet。此 cmdlet 没有任何功能，但可以通过它来安装使用它的任何已转换 Runbook。此 Runbook 将不能在 Azure 自动化中运行，但可以进行安装，因此用户可以对其进行修改。

### 不能转换的集成包

不能使用此工具转换不是通过 OIT 创建的集成包（包括由 Microsoft 创建的某些集成包）。由 Microsoft 提供的集成包都已转换为集成模块，因此可以在 Azure 自动化或 Service Management Automation 中安装。


## 标准活动模块

Orchestrator 包括一组[标准活动](http://technet.microsoft.com/zh-cn/library/hh403832.aspx)，这些活动未包括在集成包中，而是由多个 Runbook 使用。标准活动模块是一个集成模块，其中包含每个此类活动的 cmdlet 等效项。在导入任何使用标准活动的已转换 Runbook 之前，必须在 Azure 自动化中安装此集成模块。

除了支持转换后的 Runbook，标准活动模块中的 cmdlet 还可由熟悉 Orchestrator 的人用来在 Azure 自动化中构建新的 Runbook。虽然可以使用 cmdlet 来执行所有标准活动的功能，但这些活动可能会以不同方式运行。转换后的标准活动模块中的 cmdlet 的工作方式与其相应活动的工作方式相同，并使用相同的参数。这可以帮助现在的 Orchestrator Runbook 作者过渡到 Azure 自动化 Runbook。

## Runbook 转换器（即将推出）

此工具会将 Orchestrator Runbook 转换成可以导入 Azure 自动化中的图形 Runbook。有关此工具的更多详细信息，将在其发布时提供在此处。

## 相关文章

- [System Center 2012 - Orchestrator](http://technet.microsoft.com/zh-cn/library/hh237242.aspx)
- [Service Management Automation](https://technet.microsoft.com/zh-cn/library/dn469260.aspx)
- [混合 Runbook 辅助角色](/documentation/articles/automation-hybrid-runbook-worker)
- [Orchestrator 标准活动](http://technet.microsoft.com/zh-cn/library/hh403832.aspx)

<!---HONumber=69-->