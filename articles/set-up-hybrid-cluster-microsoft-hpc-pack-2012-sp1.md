<properties linkid="" urlDisplayName="" pageTitle="" metaKeywords="" description="Learn how to use Microsoft HPC Pack and Azure to set up a small, hybrid high performance computing (HPC) cluster" metaCanonical="" services="" documentationCenter="" title="Set up a Hybrid Cluster with Microsoft HPC Pack" authors="danlep" solutions="" manager="dongill" editor="mattshel" />

# 使用 Microsoft HPC Pack 设置混合群集

本教程说明如何使用 Microsoft HPC Pack 2012 R2 和 Azure 设置小型的混合高性能计算 (HPC) 群集。该群集在 Azure 云服务中将包括一个本地头节点（一台运行 Windows Server 操作系统和 HPC Pack 的计算机）和一些你按需部署为辅助角色实例的计算节点。你可以在混合群集上运行计算作业。

![混合 HPC 群集][混合 HPC 群集]

本教程展示一个方法（有时称作群集“移至云”）来使用 Azure 中的可伸缩的按需计算资源运行进行大量计算的应用程序。

本教程假定你之前未使用过计算群集或 HPC Pack。它只是为了出于演示目的帮助你快速部署混合群集。有关在生产环境中以更高层级部署混合 HPC Pack 群集的注意事项和步骤，请参阅[详细指南][详细指南]。

> [WACOM.NOTE] Azure 为你的计算资源提供了适合不同工作负载的[大小范围][大小范围]。例如，A8 和 A9 实例结合了某些 HPC 应用程序所需要的高性能和对低延迟、高吞吐量应用程序网络的访问。有关信息，请参阅 [A8 和 A9 计算密集型实例：使用 HPC Pack 快速启动][A8 和 A9 计算密集型实例：使用 HPC Pack 快速启动]。

本教程将指导你完成以下基本步骤：

-   [先决条件][先决条件]
-   [在头节点上安装 HPC Pack][在头节点上安装 HPC Pack]
-   [准备 Azure 订阅][准备 Azure 订阅]
-   [配置头节点][配置头节点]
-   [向群集添加 Azure 节点][向群集添加 Azure 节点]
-   [启动 Azure 节点][启动 Azure 节点]
-   [跨群集运行命令][跨群集运行命令]
-   [运行测试作业][运行测试作业]
-   [停止 Azure 节点][停止 Azure 节点]

## 先决条件

> [WACOM.NOTE] 若要完成本教程，你需要一个 Azure 帐户。如果你没有帐户，只需花费几分钟就能创建一个免费试用帐户。有关详细信息，请参阅 [Azure 免费试用][Azure 免费试用]。

此外，对于本教程，你还需要以下方面。

-   运行某个版本的 Windows Server 2012 R2 或 Windows Server 2012 的本地计算机。该计算机将是 HPC 群集的头节点。如果你尚未运行 Windows Server，可下载并安装[评估版][评估版]。

    -   该计算机必须加入 Active Directory 域中。

    -   确保未安装任何其他服务器角色或角色服务。

    -   要支持 HPC Pack，操作系统必须安装以下语言之一的版本：英语、日语，或简体中文

    -   确认安装了重要更新和关键更新。

-   HPC Pack 2012 R2 的安装文件，它是免费提供的。[下载][下载]完整安装包并将文件复制到头节点计算机或某个网络位置。选择语言与你所安装的 Windows Server 的语言相同的安装文件。

-   在头节点上具有本地管理员权限的域帐户。

-   端口 443 上从头节点到 Azure 的 TCP 连接。

## 在头节点上安装 HPC Pack

首先将 Microsoft HPC Pack 安装到将作为群集头节点的本地计算机上。

1.  通过使用具有本地管理员权限的域帐户登录到头节点。

2.  通过从 HPC Pack 安装文件运行 Setup.exe 启动 HPC Pack 安装向导。

3.  在“HPC Pack 2012 R2 安装”屏幕上，单击“全新安装或向现有安装添加新功能”。

    ![HPC Pack 2012 安装][HPC Pack 2012 安装]

4.  在“Microsoft 软件用户协议”页上，单击“下一步”。

5.  在“选择安装类型”页上，单击“通过创建头节点创建新 HPC 群集”，然后单击“下一步”。

    ![选择安装类型][选择安装类型]

6.  该向导会运行若干个安装前测试。如果通过了全部测试，则在“安装规则”页上单击“下一步”。否则，查看提供的信息并且在你的环境中进行必需的更改。然后再次运行测试或者根据需要再次启动安装向导。

    ![安装规则][安装规则]

7.  在“HPC 数据库配置”页上，确保对于所有 HPC 数据库都选择“头节点”，然后单击“下一步”。

    ![数据库配置][数据库配置]

8.  在其余向导页上接受默认选择。在“安装所需组件”页上，单击“安装”。

    ![安装][安装]

9.  在安装完毕后，取消选中“启动 HPC 群集管理器”，然后单击“完成”。（你将在后面的步骤中启动 HPC 群集管理器，以便完成头节点的配置。）

    ![完成][完成]

## 准备 Azure 订阅

使用[管理门户][管理门户]来执行以下 Azure 订阅步骤。之所以需要些步骤，是因为之后你可以从本地头节点部署 Azure 节点。

-   上载管理证书（头节点和 Azure 之间的安全连接需要）

-   创建 Azure 节点（辅助角色实例）将在其中运行的 Azure 云服务

-   创建 Azure 存储帐户

    > [WACOM.NOTE] 另外请记下你的 Azure 订阅 ID，后面将会需要用到它。在你的 Azure [帐户信息][帐户信息]中可以找到此 ID。

### 上载默认管理证书

HPC Pack 将在头节点上安装称作默认 Microsoft HPC Azure 管理证书的自签名证书，你可以将该证书作为 Azure 管理证书上载。此证书是为进行测试和概念证明部署提供的。

1.  从头节点计算机，登录到[管理门户][管理门户]。

2.  单击**“设置”**，然后单击**“管理证书”**。

3.  在命令栏中，单击“上载”。

    ![证书设置][证书设置]

4.  在头节点上浏览文件 C:\\Program Files\\Microsoft HPC Pack 2012\\Bin\\hpccert.cer。然后，单击“检查”按钮。

    ![上载证书][上载证书]

你将在管理证书列表中看到“默认 HPC Azure 管理”。

### 创建 Azure 云服务

> [WACOM.NOTE] 为了获得最佳性能，在相同的地理区域或地缘组（如果配置）中创建存储帐户和云服务。

1.  在管理门户中的命令栏上，单击“新建”。

2.  依次单击“计算”、“云服务”、“快速创建”。

3.  键入云服务的 URL，然后单击“创建云服务”。

    ![创建服务][创建服务]

### 创建 Azure 存储帐户

1.  在管理门户中的命令栏上，单击“新建”。

2.  依次单击**“数据服务”**、**“存储”**、**“快速创建”**。

3.  键入帐户的 URL，然后单击“创建存储帐户”。

    ![创建存储][创建存储]

## 配置头节点

若要使用 HPC 群集管理器部署 Azure 节点和提交作业，必须首先执行一些必需的群集配置步骤。

1.  在头节点上，启动 HPC 群集管理器。如果出现“选择头节点”对话框，则单击“本地计算机”。这时会出现“部署任务列表”。

2.  在“必需的部署任务”下，单击“配置网络”。

    ![配置网络][配置网络]

3.  在网络配置向导中，选择“仅限企业网络上的所有节点”（拓扑 5）。

    ![拓扑 5][拓扑 5]

    > [WACOM.NOTE] 这是出于演示目的的最简单配置，因为头节点只需要单个网络适配器以便连接到 Active Directory 和 Internet。本教程未涉及要求附加网络的群集方案。

4.  单击“下一步”以便接受其余向导页上的默认值。然后，在“检查”选项卡上，单击“配置”，以完成网络配置。

5.  在“部署任务列表”中，单击“提供安装凭据”。

6.  在“安装凭据”对话框中，键入你要用于安装 HPC Pack 的域帐户的凭据。然后，单击“确定”。

    ![安装凭据][安装凭据]

    > [WACOM.NOTE] HPC Pack 服务仅将安装凭据用于部署本地计算节点。

7.  在“部署任务列表”中，单击“配置新节点的命名”。

8.  在“指定节点名称系列”对话框中，接受默认名称系列，然后单击“确定”。

    ![节点名称][节点名称]

    > [WACOM.NOTE] 命名服务仅为已加入域的计算节点生成名称。自动对 Azure 节点进行命名。

9.  在“部署任务列表”中，单击“创建节点模板”。你现在将使用节点模板向群集添加 Azure 节点。

10. 在创建节点模板向导中，执行以下操作：

    a. 在“选择节点模板类型”页上，单击“Azure 节点模板”，然后单击“下一步”。

    ![节点模板][节点模板]

    b. 单击“下一步”，以接受默认模板名称。

    c. 在“提供订阅信息”页面上，输入 Azure 订阅 ID（在 Azure [帐户信息][帐户信息]中）。然后，在“管理证书”中，单击“浏览”并选择“默认 HPC Azure 管理”。然后单击**“下一步”**。

    ![节点模板][1]

    d. 在“提供服务信息”页上，选择你在前一步中创建的云服务和存储帐户。然后单击**“下一步”**。

    ![节点模板][2]

    e. 单击“下一步”以便接受其余向导页上的默认值。然后，在“检查”选项卡上，单击“创建”，以创建节点模板。

    > [WACOM.NOTE] 默认情况下，Azure 节点模板包括可用于手动启动（设置）和停止节点的设置。还可以配置计划以便自动启动和停止 Azure 节点。

## 向群集添加 Azure 节点

你现在将使用节点模板向群集添加 Azure 节点。将节点添加到群集可存储其配置信息，使你能够随时将这些节点作为云服务中的角色实例启动（设置）。在角色实例在云服务中运行后，仅针对 Azure 节点向你的订阅收费。

对于本教程，你将添加两个小型节点。

1.  在 HPC 群集管理器中的“节点管理”中的“动作”窗格中，单击“添加节点”。

    ![添加节点][添加节点]

2.  在添加节点向导的“选择部署方法”页上，单击“添加 Azure 节点”，然后单击“下一步”。

    ![添加 Azure 节点][添加 Azure 节点]

3.  在“指定新节点”页上，选择你之前创建的 Azure 节点模板（默认称作“默认 AzureNode 模板”）。然后指定大小为“小型”的 **2** 个节点，然后单击“下一步”。

    ![指定节点][指定节点]

    有关可用虚拟机大小的详细信息，请参阅 [Azure 的虚拟机和云服务大小][Azure 的虚拟机和云服务大小]。

4.  在“完成添加节点向导”页上，单击“完成”。

    HPC 群集管理器中现在会出现名为 **AzureCN-0001** 和 **AzureCN-0002** 的两个 Azure 节点。它们的状态都是“未部署”。

    ![已添加节点][已添加节点]

## 启动 Azure 节点

当你想要在 Azure 中使用群集资源时，使用 HPC 群集管理器可启动（设置）Azure 节点并且使它们处于联机状态。

1.  在 HPC 群集管理器中的“节点管理”中，单击这两个或其中一个节点，然后在“动作”窗格中，单击“启动”。

    ![启动节点][启动节点]

2.  在“启动 Azure 节点”对话框中，单击“启动”。

    ![启动节点][3]

    节点将转换到“正在设置”状态。你可以查看设置日志以便跟踪设置进度。

    ![设置节点][设置节点]

3.  几分钟后，Azure 节点将完成设置并且处于“脱机”状态。在此状态下，角色实例尽管在运行，但将不会接受群集作业。

4.  若要确认角色实例正在运行，请在[管理门户][管理门户]中单击“云服务”，单击你的云服务的名称，然后单击“实例”。

    ![运行实例][运行实例]

    你将看到两个辅助角色实例正在服务中运行。HPC Pack 还将自动部署两个 **HpcProxy** 实例（中等大小）以便处理头节点和 Azure 之间的通信。

5.  若要使 Azure 节点处于联机状态以便运行群集作业，请选择这些节点，右键单击，然后单击“使联机”。

    ![脱机节点][脱机节点]

    HPC 群集管理器指示节点处于“联机”状态。

## 跨群集运行命令

你可以使用 HPC Pack 的 **clusrun** 命令对一个或多个群集节点运行命令或应用程序。下面举一个简单的例子，介绍如何使用 **clusrun** 获取 Azure 节点的 IP 配置。

1.  在头节点上，打开命令提示符。

2.  输入以下命令：

    `clusrun /nodes:azurecn* ipconfig`

3.  你将看到与下面类似的输出。

    ![Clusrun][Clusrun]

## 运行测试作业

你可以提交在混合群集上运行的测试作业。这个示例是一个简单的“参数分析”作业（一种固有的并行计算），它运行多个子任务，这些子任务通过使用 **set /a** 命令向其本身添加整数。群集中的所有节点都参与完成针对 1 到 100 的整数的子任务。

1.  在 HPC 群集管理器中的“节点管理”中的“动作”窗格中，单击“新建参数分析作业”。

    ![新作业][新作业]

2.  在“新建参数分析作业”对话框的“命令行”中，键入`set /a *+*` （覆盖出现的默认命令行）。对于其余设置保留使用默认值，然后单击“提交”来提交作业。

    ![参数分析][参数分析]

3.  在该作业完成后，双击“我的分析任务”作业。

4.  单击“查看任务”，然后单击某一子任务以便查看该子任务的计算输出。

    ![任务结果][任务结果]

5.  若要查看哪一节点执行了针对该子任务的计算，请单击“分配的节点”。（你的群集可能显示不同节点名称。）

    ![任务结果][4]

## 停止 Azure 节点

在你试用群集后，可以使用 HPC 群集管理器停止 Azure 节点，以免向你的帐户收取不必要的费用。这将停止云服务并且删除 Azure 角色实例。

1.  在 HPC 群集管理器的“节点管理”中，将两个 Azure 节点都选中。然后，在“操作”窗格中，单击“停止”。

    ![停止节点][停止节点]

2.  在“停止 Azure 节点”对话框中，单击“停止”。

    ![停止节点][5]

3.  节点将转换到“正在停止”状态。几分钟后，HPC 群集管理器将显示节点处于“未部署”状态。

    ![未部署节点][未部署节点]

4.  若要确认角色实例不再在 Azure 继续运行，请在[管理门户][管理门户]中单击“云服务”，单击你的云服务的名称，然后单击“实例”。将不会有任何实例部署在生产环境中。

    ![无实例][无实例]

    本教程至此完毕。

## 相关资源

-   [HPC Pack 2012 R2 和 HPC Pack 2012][HPC Pack 2012 R2 和 HPC Pack 2012]
-   [使用 Microsoft HPC Pack 迸发到 Azure][详细指南]
-   [将应用程序部署到 Azure 节点][将应用程序部署到 Azure 节点]
-   [Azure 虚拟机中的 Microsoft HPC Pack][Azure 虚拟机中的 Microsoft HPC Pack]

  [混合 HPC 群集]: ./media/set-up-hybrid-cluster-microsoft-hpc-pack-2012-sp1/hybrid_cluster_overview.png
  [详细指南]: http://go.microsoft.com/fwlink/p/?LinkID=200493
  [大小范围]: http://go.microsoft.com/fwlink/p/?LinkId=389844
  [A8 和 A9 计算密集型实例：使用 HPC Pack 快速启动]: http://go.microsoft.com/fwlink/p/?Linkid=328042
  [先决条件]: #BKMK_Prereq
  [在头节点上安装 HPC Pack]: #BKMK_DeployHN
  [准备 Azure 订阅]: #BKMK_Prpare
  [配置头节点]: #BKMK_ConfigHN
  [向群集添加 Azure 节点]: #BKMK_worker
  [启动 Azure 节点]: #BKMK_start
  [跨群集运行命令]: #BKMK_RunCommand
  [运行测试作业]: #BKMK_RunJob
  [停止 Azure 节点]: #BKMK_stop
  [Azure 免费试用]: http://www.windowsazure.com/en-us/pricing/free-trial/
  [评估版]: http://technet.microsoft.com/evalcenter/dn205286.aspx
  [下载]: http://go.microsoft.com/fwlink/p/?linkid=389557
  [HPC Pack 2012 安装]: ./media/set-up-hybrid-cluster-microsoft-hpc-pack-2012-sp1/install_hpc1.png
  [选择安装类型]: ./media/set-up-hybrid-cluster-microsoft-hpc-pack-2012-sp1/install_hpc2.png
  [安装规则]: ./media/set-up-hybrid-cluster-microsoft-hpc-pack-2012-sp1/install_hpc3.png
  [数据库配置]: ./media/set-up-hybrid-cluster-microsoft-hpc-pack-2012-sp1/install_hpc4.png
  [安装]: ./media/set-up-hybrid-cluster-microsoft-hpc-pack-2012-sp1/install_hpc6.png
  [完成]: ./media/set-up-hybrid-cluster-microsoft-hpc-pack-2012-sp1/install_hpc7.png
  [管理门户]: https://manage.windowsazure.cn
  [帐户信息]: [https://account.windowsazure.com/Subscriptions
  [证书设置]: ./media/set-up-hybrid-cluster-microsoft-hpc-pack-2012-sp1/upload_cert1.png
  [上载证书]: ./media/set-up-hybrid-cluster-microsoft-hpc-pack-2012-sp1/install_hpc10.png
  [创建服务]: ./media/set-up-hybrid-cluster-microsoft-hpc-pack-2012-sp1/createservice1.png
  [创建存储]: ./media/set-up-hybrid-cluster-microsoft-hpc-pack-2012-sp1/createstorage1.png
  [配置网络]: ./media/set-up-hybrid-cluster-microsoft-hpc-pack-2012-sp1/config_hpc2.png
  [拓扑 5]: ./media/set-up-hybrid-cluster-microsoft-hpc-pack-2012-sp1/config_hpc3.png
  [安装凭据]: ./media/set-up-hybrid-cluster-microsoft-hpc-pack-2012-sp1/config_hpc6.png
  [节点名称]: ./media/set-up-hybrid-cluster-microsoft-hpc-pack-2012-sp1/config_hpc8.png
  [节点模板]: ./media/set-up-hybrid-cluster-microsoft-hpc-pack-2012-sp1/config_hpc10.png
  [1]: ./media/set-up-hybrid-cluster-microsoft-hpc-pack-2012-sp1/config_hpc12.png
  [2]: ./media/set-up-hybrid-cluster-microsoft-hpc-pack-2012-sp1/config_hpc13.png
  [添加节点]: ./media/set-up-hybrid-cluster-microsoft-hpc-pack-2012-sp1/add_node1.png
  [添加 Azure 节点]: ./media/set-up-hybrid-cluster-microsoft-hpc-pack-2012-sp1/add_node1_1.png
  [指定节点]: ./media/set-up-hybrid-cluster-microsoft-hpc-pack-2012-sp1/add_node2.png
  [Azure 的虚拟机和云服务大小]: http://msdn.microsoft.com/library/windowsazure/dn197896.aspx
  [已添加节点]: ./media/set-up-hybrid-cluster-microsoft-hpc-pack-2012-sp1/add_node3.png
  [启动节点]: ./media/set-up-hybrid-cluster-microsoft-hpc-pack-2012-sp1/add_node4.png
  [3]: ./media/set-up-hybrid-cluster-microsoft-hpc-pack-2012-sp1/add_node5.png
  [设置节点]: ./media/set-up-hybrid-cluster-microsoft-hpc-pack-2012-sp1/add_node6.png
  [运行实例]: ./media/set-up-hybrid-cluster-microsoft-hpc-pack-2012-sp1/view_instances1.png
  [脱机节点]: ./media/set-up-hybrid-cluster-microsoft-hpc-pack-2012-sp1/add_node7.png
  [Clusrun]: ./media/set-up-hybrid-cluster-microsoft-hpc-pack-2012-sp1/clusrun1.png
  [新作业]: ./media/set-up-hybrid-cluster-microsoft-hpc-pack-2012-sp1/test_job1.png
  [参数分析]: ./media/set-up-hybrid-cluster-microsoft-hpc-pack-2012-sp1/param_sweep1.png
  [任务结果]: ./media/set-up-hybrid-cluster-microsoft-hpc-pack-2012-sp1/view_job361.png
  [4]: ./media/set-up-hybrid-cluster-microsoft-hpc-pack-2012-sp1/view_job362.png
  [停止节点]: ./media/set-up-hybrid-cluster-microsoft-hpc-pack-2012-sp1/stop_node1.png
  [5]: ./media/set-up-hybrid-cluster-microsoft-hpc-pack-2012-sp1/stop_node2.png
  [未部署节点]: ./media/set-up-hybrid-cluster-microsoft-hpc-pack-2012-sp1/stop_node4.png
  [无实例]: ./media/set-up-hybrid-cluster-microsoft-hpc-pack-2012-sp1/view_instances2.png
  [HPC Pack 2012 R2 和 HPC Pack 2012]: http://go.microsoft.com/fwlink/p/?LinkID=263697
  [将应用程序部署到 Azure 节点]: http://go.microsoft.com/fwlink/p/?LinkId=325317
  [Azure 虚拟机中的 Microsoft HPC Pack]: http://go.microsoft.com/fwlink/p/?linkid=330375
