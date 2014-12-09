<properties linkid="automation-create-runbook-from-samples" urlDisplayName="Get Started with Azure Automation" pageTitle="Get Started with Azure Automation" metaKeywords="" description="Learn how to import and run an automation job in Azure." metaCanonical="" services="automation" documentationCenter="" title="Get Started with Azure Automation" authors="" solutions="" manager="" editor="" />

# Azure 自动化入门

通过 Microsoft Azure 自动化，开发人员可以自动完成通常要在云环境中执行的手动、长时间进行、易出错且重复性高的任务。你可以使用 Runbook 来创建、监视、管理和部署 Azure 环境中的资源。所谓的 Runbook，基本上就是指 Windows PowerShell 工作流。若要了解有关自动化的详细信息，请参阅[自动化概述指南][自动化概述指南]。

本教程将逐步引导你完成将一个示例“Hello World”Runbook 导入 Azure 自动化、执行该 Runbook 并查看其输出的步骤。

> [WACOM.NOTE] 如需有关自动化入门的更多帮助，请了解如何使用[此处][此处]提供的 PowerShell cmdlet 自动执行 Azure 操作。

## 示例和实用 Runbook

Azure 自动化团队创建了许多 Runbook 示例以帮助你开始使用自动化。这些示例运用了基本的自动化概念，旨在帮助你了解如何自行编写 Runbook。

自动化团队还创建了实用 Runbook，你可以将其用作较为繁重的自动化任务的构造块。

> [WACOM.NOTE] 最佳做法是编写小型模块化且可重复使用的 Runbook。另外，我们强烈建议你在熟悉自动化之后，针对常用的案例创建你自己的实用 Runbook。

可以在[脚本中心][脚本中心]查看和下载自动化团队的示例和实用 Runbook。

## 自动化社区和反馈

[脚本中心][1]还发布了社区和其他 Microsoft 团队提供的 Runbook。

**欢迎提供反馈！**如果你正在寻求自动化 Runbook 解决方案或集成模块，请在脚本中心发布脚本请求。如果你对自动化的新功能有任何看法，请在[用户之声][用户之声]上发表你的看法。

[WACOM.INCLUDE [create-account-note](../includes/create-account-note.md)]

## 本教程的高级步骤

1.  [注册自动化预览版][注册自动化预览版]
2.  [下载示例 Runbook][下载示例 Runbook]
3.  [导入、运行示例 Runbook 并查看其输出][导入、运行示例 Runbook 并查看其输出]

## <a name="preview"></a>注册 Azure 自动化预览版

若要开始使用自动化，你需要一个启用了 Microsoft Azure 自动化预览版功能的有效 Azure 订阅。

-   在**“预览版功能”**页上单击**“立即试用”**。

    ![启用预览版][启用预览版]

## <a name="download-sample"></a>从脚本中心下载示例 Runbook

1.  转到[脚本中心][脚本中心]，然后单击**“Azure 自动化的 Hello World”**。

2.  单击**“下载”**旁边的文件名 **Write-HelloWorld.ps1**，然后将该文件保存到你的计算机。

## <a name="import-sample"></a>在 Azure 自动化中导入、运行和查看示例 Runbook

1.  登录到 [Azure 管理门户][Azure 管理门户]。

2.  在管理门户中，单击**“创建自动化帐户”**。

    > [WACOM.NOTE] 如果你已创建自动化帐户，则可以跳转到步骤 4。

    ![创建帐户][创建帐户]

3.  在**“添加新的自动化帐户”**页上输入帐户名，然后单击复选标记。

    ![添加新帐户][添加新帐户]

4.  在**“自动化”**页上，单击你刚刚创建的新帐户。

    ![新帐户][新帐户]

5.  单击**“RUNBOOK”**。

    ![Runbook 选项卡][Runbook 选项卡]

6.  单击**“导入”**。

    ![导入][导入]

7.  浏览到下载的 **Write-HelloWorld.ps1** 脚本，然后单击复选标记。

    ![浏览][浏览]

8.  单击**“Write-HelloWorld”**。

    ![导入的 Runbook][导入的 Runbook]

9.  单击**“创作”**，然后单击**“草稿”**。对于此 Runbook，你不需要做出任何修改。

    现在，你可以看到 **Write-HelloWorld.ps1** 的内容。可以在草稿模式下修改 Runbook 的内容。

    ![创作草稿][创作草稿]

10. 单击**“发布”**以提升 Runbook，使其随时可在生产环境中使用。

    ![发布][发布]

11. 当出现保存并发布 Runbook 的提示时，请单击**“是”**。

    ![有关保存并发布 Runbook 的提示][有关保存并发布 Runbook 的提示]

12. 单击**“已发布”**，然后单击**“启动”**。

    ![已发布][已发布]

13. 在**“指定 Runbook 参数值”**页上，键入要用作 Write-HelloWorld.ps1 脚本的输入参数的**名称**，然后单击复选标记。

    ![Runbook 参数][Runbook 参数]

14. 单击**“作业”**以查看你刚刚启动的 Runbook 作业的状态，然后单击**“作业启动”**列中的时间戳以查看作业摘要。

    ![Runbook 状态][Runbook 状态]

15. 在**“摘要”**页上，你可以查看作业的摘要、输入参数和输出。

    ![Runbook 摘要][Runbook 摘要]

## 另请参阅

-   [自动化概述][自动化概述]
-   [Runbook 创作指南][Runbook 创作指南]
-   [自动化论坛][自动化论坛]
-   [使用 Azure 自动化和 PowerShell cmdlet 自动执行 Azure 操作][此处]

  [自动化概述指南]: http://go.microsoft.com/fwlink/p/?LinkId=392861
  [此处]: http://blogs.technet.com/b/keithmayer/archive/2014/04/04/step-by-step-getting-started-with-windows-azure-automation.aspx
  [脚本中心]: http://go.microsoft.com/fwlink/p/?LinkId=393029
  [1]: http://go.microsoft.com/fwlink/?LinkID=391681
  [用户之声]: http://feedback.windowsazure.com/forums/34192--general-feedback
  [注册自动化预览版]: #preview
  [下载示例 Runbook]: #download-sample
  [导入、运行示例 Runbook 并查看其输出]: #import-sample
  [启用预览版]: ./media/automation/automation_00_EnablePreview.png
  [Azure 管理门户]: http://manage.windowsazure.com
  [创建帐户]: ./media/automation/automation_01_CreateAccount.png
  [添加新帐户]: ./media/automation/automation_02_addnewautoacct.png
  [新帐户]: ./media/automation/automation_03_NewAutoAcct.png
  [Runbook 选项卡]: ./media/automation/automation_04_RunbooksTab.png
  [导入]: ./media/automation/automation_05_Import.png
  [浏览]: ./media/automation/automation_06_Browse.png
  [导入的 Runbook]: ./media/automation/automation_07_ImportedRunbook.png
  [创作草稿]: ./media/automation/automation_08_AuthorDraft.png
  [发布]: ./media/automation/automation_085_Publish.png
  [有关保存并发布 Runbook 的提示]: ./media/automation/automation_09_SavePubPrompt.png
  [已发布]: ./media/automation/automation_10_PublishStart.png
  [Runbook 参数]: ./media/automation/automation_11_RunbookParams.png
  [Runbook 状态]: ./media/automation/automation_12_RunbookStatus.png
  [Runbook 摘要]: ./media/automation/automation_13_RunbookSummary_callouts.png
  [自动化概述]: http://go.microsoft.com/fwlink/p/?LinkId=392860
  [Runbook 创作指南]: http://go.microsoft.com/fwlink/p/?LinkID=301740
  [自动化论坛]: http://go.microsoft.com/fwlink/p/?LinkId=390561
