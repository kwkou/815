<properties
	pageTitle="Azure 自动化的 Runbook 和模块库 | Azure"
	description="你可以安装并在 Azure 自动化环境中使用 Microsoft 和社区提供的 Runbook 与模块。本文介绍如何访问这些资源，以及在库中补充你的 Runbook。"
	services="automation"
	documentationCenter=""
	authors="mgoedtel"
	manager="stevenka"
	editor="tysonn" />
<tags
	ms.service="automation"
	ms.date="03/02/2016"
	wacn.date="06/30/2016" />


# Azure 自动化的 Runbook 和模块库

你无需在 Azure 自动化中创建自己的 Runbook 和模块，即可访问 Microsoft 和社区构建的各种解决方案。你可以在不加以修改的情况下直接使用这些解决方案，或者使用它们作为起点并根据具体的要求进行编辑。

可以从 [Runbook 库](#runbooks-in-runbook-gallery)获取 Runbook，并从 [PowerShell 库](#modules-in-powerShell-gallery)获取模块。你还可以通过共享开发的解决方案来为社区做出贡献。

##<a name="runbooks-in-runbook-gallery"></a> Runbook 库中的 Runbook

[Runbook 库](http://gallery.technet.microsoft.com/scriptcenter/site/search?f[0].Type=RootCategory&f[0].Value=WindowsAzure&f[1].Type=SubCategory&f[1].Value=WindowsAzure_automation&f[1].Text=Automation)提供各种来自 Microsoft 的 Runbook，以及可导入 Azure 自动化中的社区。你可以从 [TechNet 脚本中心](http://gallery.technet.microsoft.com)托管的库下载 Runbook，或者在 Azure 经典管理门户中直接从该库导入 Runbook。

直接从 Runbook 库导入只能使用 Azure 经典管理门户来完成，而不能使用 Windows PowerShell 执行此功能。

>[AZURE.NOTE] 你应验证从 Runbook 库获取的任何 Runbook 的内容，在生产环境中安装和运行这些 Runbook 时，请谨慎操作。

### 使用 Azure 经典管理门户从 Runbook 库导入 Runbook

1. 在 Azure 经典管理门户中，单击“添加”>“应用程序服务”>“自动化”>“Runbook”>“从库中”。
2. 选择一个类别以查看相关的 Runbook，然后选择一个 Runbook 以查看其详细信息。选择所需的 Runbook 后，单击右箭头按钮。![Runbook 库](./media/automation-runbook-gallery/runbook-gallery.png)
3. 查看该 Runbook 的内容，并记下说明中所述的任何要求。完成后，单击右箭头按钮。
4. 输入 Runbook 详细信息，然后单击复选标记按钮。Runbook 名称已填入。
5. 该 Runbook 将出现在自动化帐户的“Runbook”选项卡中。


###<a name="AddRunbook"></a> 将 Runbook 添加到 Runbook 库

Azure 建议你将 Runbook 添加到你认为对其他客户有用的 Runbook 库中。你可以通过连同以下详细信息[将 Runbook 上载到脚本中心](http://gallery.technet.microsoft.com/site/upload)，来添加 Runbook。

- 你必须为向导中要显示的 Runbook 指定“Azure”作为“类别”，指定“自动化”作为“子类别”。  

- 上载内容必须是单个 .ps1 或 .graphrunbook 文件。如果 Runbook 需要任何模块、子 Runbook 或资产，则你应该在提交内容的说明和 Runbook 的注释部分列出这些内容。如果你的解决方案需要多个 Runbook，请单独上载每个 Runbook 并在各自的说明中列出相关 Runbook 的名称。请确保使用相同的标记，以便它们在同一类别中显示。用户阅读说明后才会知道，此 Runbook 正常工作需要其他 Runbook。

- 使用“插入代码段”图标将 PowerShell 工作流代码段插入说明中。

- Runbook 库结果中将显示上载摘要，因此，你应该提供详细信息，以帮助用户了解 Runbook 的功能。

- 你应该为上载内容分配一到三个以下标记。Runbook 将在向导中与标记匹配的类别下列出。该向导将忽略不在此列表中的所有标记。如果你未指定任何匹配的标记，则 Runbook 将在“其他”类别下列出。

 - 备份
 - 容量管理
 - 更改控制
 - 合规性
 - 开发/测试环境
 - 灾难恢复
 - 监视
 - 修补
 - 设置
 - 补救
 - VM 生命周期管理


- 自动化每小时更新一次该库，因此，你无法立即看见上载内容。如果一小时后你在该库中看不到你的 Runbook，请查看[将 Runbook 添加到 Runbook 库](#AddRunbook)部分中所述的要求。


## 请求 Runbook 或模块

你可以将请求发送到[用户之声](/product-feedback)。如果你需要 Runbook 编写帮助，或对 PowerShell 存有疑问，请将问题发布到我们的[论坛](http://social.msdn.microsoft.com/Forums/windowsazure/zh-cn/home?forum=azureautomation&filter=alltypes&sort=lastpostdesc)。

## 相关文章

- [在 Azure 自动化中创建或导入 Runbook](/documentation/articles/automation-creating-importing-runbook/)
- [了解 PowerShell 工作流](/documentation/articles/automation-powershell-workflow/)

<!---HONumber=Mooncake_0411_2016-->