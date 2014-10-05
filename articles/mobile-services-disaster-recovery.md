<properties linkid="mobile-services-recovery-disaster" urlDisplayName="Recover your mobile service in the event of a disaster" pageTitle="Recover your mobile service in the event of a disaster - Azure Mobile Services" metaKeywords="" description="Learn how to recover your mobile service in the event of a disaster." metaCanonical="" services="" documentationCenter="Mobile" title="Recover your mobile service in the event of a disaster" authors="yavorg" solutions="" manager="" editor="" />

# 在发生灾难时恢复移动服务

当你使用 Azure 移动服务部署应用程序时，可以使用它的内置功能来确保发生可用性问题（例如服务器故障、网络中断、数据丢失和大范围的设施损毁）时保持业务连续性。使用 Azure 移动服务部署应用程序时，你可以像部署传统的本地解决方案一样，利用你必须设计、实施和管理的许多容错与基础结构功能。Azure 可让你花费少量的成本消除大量的潜在故障。

<a name="prepare"></a>
## 准备针对可能的灾难做好准备

为了在出现可用性问题时方便进行恢复，你可以提前做好准备：

-   "在 Azure 移动服务 SQL Database 中备份数据"
    移动服务应用程序数据存储在 Azure SQL Database 中。我们建议你根据 [SQL Database 业务连续性指南][]中的说明备份这些数据。
-   "备份移动服务脚本"
    我们建议你将移动服务脚本存储在源代码管理系统（例如 [Team Foundation Service][] 或 [GitHub]）中，而不只是依赖于移动服务自身中的副本。你可以使用移动服务[源代码管理功能][]或[使用 Azure 命令行工具][]通过 Azure 门户下载这些脚本。请特别注意门户中标记为“预览”的功能，因为不保证你能够恢复这些脚本，你可能需要从自己的源代码管理原件进行恢复。
-   "保留辅助移动服务"
    当移动服务发生可用性问题时，你可能需要将它重新部署到备用的 Azure 区域。为确保有可用容量（例如，在极少见的情况下，整个区域都会丢失），建议你在备用区域中创建一个辅助移动服务，并将其模式设置为等同于或者高于主服务的模式。（如果主服务处于共享模式，则你可以将辅助服务设置为共享模式或保留模式。但如果主服务处于保留模式，则辅助服务也必须处于保留模式。）

<a name="watch"></a>
## 观察观察问题的迹象

以下情况表示出现了可能需要你执行恢复操作的问题：

-   连接到移动服务的应用程序长时间无法与移动服务通信。
-   移动服务状态在 [Azure 门户][]中显示为“不正常” 。
-   在 Azure 门户中，移动服务的每个选项卡顶部显示“不正常”标题，并且管理操作生成错误消息 。
-   [Azure 服务仪表板][]指示出现了可用性问题。

<a name="recover"></a>
## 恢复发生灾难后进行恢复

出现问题时，请使用服务仪表板来获取指导和更新。

当服务仪表板发出提示时，请执行以下步骤，以便在备用 Azure 区域中将移动服务还原到某种运行状态。如果你提前创建了辅助服务，则会使用它的容量来还原主服务。由于主服务的 URL 和应用程序密钥在恢复后不会更改，因此你不需要更新引用该 URL 和密钥的应用程序。

若要在发生中断后恢复移动服务，请执行以下操作：

1.  在 Azure 门户中，确保服务的状态报告为“不正常” 。

2.  如果你已保留了辅助移动服务，则可以跳过此步骤。

	如果你尚未保留辅助移动服务，现在请在另一个 Azure 区域中创建一个辅助移动服务。将它的模式设置为等同于或高于主服务的模式。（如果主服务处于共享模式，则你可以将辅助服务设置为共享模式或保留模式。但如果主服务处于保留模式，则辅助服务也必须处于保留模式。）

1.  根据[使用命令行工具自动操作移动服务][使用 Azure 命令行工具]中所述，将 Azure 命令行工具配置为可处理你的订阅。

2.  现在，你可以使用辅助服务来恢复主服务。

    <div class="dev-callout"><b>重要说明</b>

    <p>当你执行此步骤中的命令时，将会删除辅助服务，以便能够使用它的容量来恢复主服务。我们建议你在运行该命令之前备份脚本和设置（如果你想要保留的话）。</p>
	</div>

	准备就绪后，执行以下命令：

	        azure mobile recover PrimaryService SecondaryService
	    info:Executing command mobile recover
	    警告：this action will use the capacity from the mobile service 'SecondaryService' and delete it.Do you want to recover the mobile service 'PrimaryService'?(y/n):y
	    + Performing recovery
	    + Cleaning up
	    info:Recovery complete
	    info:mobile recover command OK

	<div class="dev-callout"><b>Note</b>
	<p>It may take a few minutes after the command completes until you can see the changes in the portal.</p>
	</div>

1.  验证是否已正确恢复所有脚本，方法是将其与源代码管理中的原件进行比较。大多数情况下，脚本会自动恢复且不会丢失数据，但如果你发现存在差异，可以手动恢复该脚本。

2.  确保恢复后的服务能够与 Azure SQL Database 通信。recover 命令将恢复移动服务，但会保留与原始数据库的连接。如果 Azure 主区域中的问题也影响到了数据库，则恢复后的服务可能仍然不会正常运行。你可以使用 Azure 服务仪表板来检查给定区域的数据库状态。如果原始数据库不工作，则你可以恢复它：

    -   根据 [SQL Database 业务连续性指南][]中所述，将 Azure SQL Database 恢复到你刚刚在其中恢复了移动服务的 Azure 区域。
    -   在 Azure 门户中，在移动服务的“配置”选项卡上选择“更改数据库”，然后选择刚刚恢复的数据库 。

现在，你所处的状态应该就是移动服务在新 Azure 区域中恢复到的状态，并且该移动服务正在使用其原始 URL 接收来自应用商店应用程序的流量。

  [SQL Database 业务连续性指南]: http://msdn.microsoft.com/zh-cn/library/windowsazure/hh852669.aspx
  [Team Foundation Service]: http://tfs.visualstudio.com/
  [源代码管理功能]: http://www.windowsazure.com/zh-cn/develop/mobile/tutorials/store-scripts-in-source-control/
  [使用 Azure 命令行工具]: http://www.windowsazure.com/zh-cn/develop/mobile/tutorials/command-line-administration/
  [Azure 门户]: http://manage.windowsazure.cn/
  [Azure 服务仪表板]: http://www.windowsazure.com/zh-cn/support/service-dashboard/
