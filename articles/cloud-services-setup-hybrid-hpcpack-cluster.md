<properties
	pageTitle="使用 Microsoft HPC Pack 设置混合 HPC 群集 | Azure"
	description="了解如何使用 Microsoft HPC Pack 和 Azure 设置小型的混合高性能计算 (HPC) 群集"
	services="cloud-services"
	documentationCenter=""
	authors="dlepow"
	manager="timlt"
	editor=""
	tags="azure-service-management,hpc-pack"/>

<tags
	ms.service="cloud-services"
	ms.date="04/14/2016"
	wacn.date="05/12/2016"/>


# 使用 Microsoft HPC Pack 和按需 Azure 辅助角色实例设置混合高性能计算 (HPC) 群集

使用 Microsoft HPC Pack 2012 R2 和 Azure 可以设置小型混合高性能计算 (HPC) 群集该群集将由一个本地头节点（运行 Windows Server 操作系统和 HPC Pack 的计算机）和一些计算节点（在 Azure 云服务中作为辅助角色实例按需部署）构成。你可以在混合群集上运行计算作业。

![混合 HPC 群集][Overview]

本教程展示了一种方法（有时称作群集“迸发至云”）来使用 Azure 中可伸缩的按需计算资源运行计算密集型应用程序。

本教程假定你之前未使用过计算群集或 HPC Pack。它只是为了出于演示目的帮助你快速部署混合群集。有关在生产环境中以更大规模部署混合 HPC Pack 群集的注意事项和步骤，请参阅[详细指南](http://go.microsoft.com/fwlink/p/?LinkID=200493)。有关使用 HPC Pack 的其他方案，包括在 Azure 虚拟机中自动执行群集部署，请参阅 [HPC cluster options with Microsoft HPC Pack in Azure（Azure 中 Microsoft HPC Pack 的 HPC 群集选项）](/documentation/articles/virtual-machines-windows-hpcpack-cluster-options/)。


## 先决条件

* **Azure 订阅** - 如果你没有 Azure 订阅，只需要花费几分钟就能创建一个[试用版](/pricing/1rmb-trial)。

* **运行 Windows Server 2012 R2 或 Windows Server 2012 的本地计算机** - 该计算机将是 HPC 群集的头节点。如果你尚未运行 Windows Server，可下载并安装[评估版](http://technet.microsoft.com/evalcenter/dn205286.aspx)。

	* 该计算机必须加入 Active Directory 域中。

	* 确保未安装任何其他服务器角色或角色服务。

	* 为了支持 HPC Pack，必须采用以下语言之一安装操作系统：英语、日语或简体中文。

	* 确认安装了重要更新和关键更新。

* **HPC Pack 2012 R2** - 免费[下载](http://go.microsoft.com/fwlink/p/?linkid=328024)最新版本的安装包并将文件复制到头节点计算机或某个网络位置。选择语言与你所安装的 Windows Server 的语言相同的安装文件。

* **域帐户** - 必须为此帐户配置头节点上的本地管理员权限，才能安装 HPC Pack。

* **端口 443 上的 TCP 连接**：从头节点到 Azure。

## 在头节点上安装 HPC Pack

首先将 Microsoft HPC Pack 安装到将作为群集头节点且运行着 Windows Server 的本地计算机上。

1. 通过使用具有本地管理员权限的域帐户登录到头节点。

2. 通过从 HPC Pack 安装文件运行 Setup.exe 启动 HPC Pack 安装向导。

3. 在“HPC Pack 2012 R2 安装”屏幕上，单击“全新安装或向现有安装添加新功能”。

	![HPC Pack 2012 安装][install_hpc1]

4. 在“Microsoft 软件用户协议”页上，单击“下一步”。

5. 在“选择安装类型”页上，单击“通过创建头节点创建新 HPC 群集”，然后单击“下一步”。

	![选择安装类型][install_hpc2]

6. 该向导会运行若干个安装前测试。如果通过了全部测试，则在“安装规则”页上单击“下一步”。否则，请查看提供的信息并在你的环境中进行必需的更改。然后再次运行测试或者根据需要再次启动安装向导。

	![安装规则][install_hpc3]

7. 在“HPC 数据库配置”页上，确保对于所有 HPC 数据库都选择了“头节点”，然后单击“下一步”。

	![数据库配置][install_hpc4]

8. 在其余向导页上接受默认选择。在“安装所需组件”页上，单击“安装”。

	![安装][install_hpc6]

9. 在安装完毕后，取消选中“启动 HPC 群集管理器”，然后单击“完成”。（你将在后面的步骤中启动 HPC 群集管理器。）

	![完成][install_hpc7]

## 准备 Azure 订阅
使用 [Azure 管理门户](https://manage.windowsazure.cn)来执行以下 Azure 订阅步骤。之所以需要些步骤，是为了之后你可以从本地头节点部署 Azure 节点。

- 上载管理证书（头节点和 Azure 之间的安全连接所需的）

- 创建 Azure 节点（辅助角色实例）将在其中运行的 Azure 云服务

- 创建 Azure 存储帐户

	>[AZURE.NOTE]另外请记下你的 Azure 订阅 ID，后面将需要用到它。可以在你的 Azure 帐户信息中找到它。

### 上载默认管理证书
HPC Pack 将在头节点上安装称作默认 Microsoft HPC Azure 管理证书的自签名证书，你可以将此证书作为 Azure 管理证书上载。此证书是为进行测试和概念证明部署提供的。

1. 从头节点计算机登录到 [Azure 管理门户](https://manage.windowsazure.cn)。

2. 单击“设置”>“管理证书”。

3. 在命令栏中，单击“上载”。

	![证书设置][upload_cert1]

4. 在头节点上浏览找到文件 C:\\Program Files\\Microsoft HPC Pack 2012\\Bin\\hpccert.cer。然后，单击“检查”按钮。

	![上载证书][install_hpc10]

你将在管理证书列表中看到“默认 HPC Azure 管理”。

### 创建 Azure 云服务

>[AZURE.NOTE]为了获得最佳性能，请在同一地理区域中创建云服务和存储帐户（在后面的步骤中）。

1. 在管理门户中的命令栏上，单击“新建”。

2. 单击“计算”>“云服务”>“快速创建”。

3. 键入云服务的 URL，然后单击“创建云服务”。

	![创建服务][createservice1]

### 创建 Azure 存储帐户

1. 在管理门户中的命令栏上，单击“新建”。

2. 依次单击**“数据服务”**>**“存储”**>**“快速创建”**。

3. 为帐户键入一个 URL，然后单击“创建存储帐户”。

	![创建存储][createstorage1]

## 配置头节点

若要使用 HPC 群集管理器部署 Azure 节点和提交作业，必须首先执行一些必需的群集配置步骤。

1. 在头节点上，启动 HPC 群集管理器。如果出现“选择头节点”对话框，则单击“本地计算机”。这时会出现“部署待办事项列表”。

2. 在“必需的部署任务”下，单击“配置网络”。

	![配置网络][config_hpc2]

3. 在网络配置向导中，选择“仅限企业网络上的所有节点”（拓扑 5）。

	![拓扑 5][config_hpc3]

	>[AZURE.NOTE]这是用于演示目的的最简单配置，因为头节点只需要单个网络适配器来连接到 Active Directory 和 Internet。本教程未涉及要求附加网络的群集方案。

4. 单击“下一步”以接受其余向导页上的默认值。然后，在“检查”选项卡上，单击“配置”，以完成网络配置。

5. 在“部署待办事项列表”中，单击“提供安装凭据”。

6. 在“安装凭据”对话框中，键入你要用于安装 HPC Pack 的域帐户的凭据。然后，单击“确定”。

	![安装凭据][config_hpc6]

	>[AZURE.NOTE]HPC Pack 服务仅将安装凭据用于部署已加入域的计算节点。你在本教程中添加的 Azure 节点未加入域。

7. 在“部署待办事项列表”中，单击“配置新节点的命名”。

8. 在“指定节点名称系列”对话框中，接受默认名称系列，然后单击“确定”。

	![节点名称][config_hpc8]

	>[AZURE.NOTE]命名服务仅为已加入域的计算节点生成名称。自动对 Azure 工作节点进行命名。

9. 在“部署待办事项列表”中，单击“创建节点模板”。你现在将使用节点模板向群集添加 Azure 节点。

10. 在创建节点模板向导中，执行以下操作：

	a.在“选择节点模板类型”页上，单击“Azure 节点模板”，然后单击“下一步”。

	![节点模板][config_hpc10]

	b.单击“下一步”以接受默认模板名称。

	c.在“提供订阅信息”页上，输入 Azure 订阅 ID（在你的 Azure 帐户信息中提供）。然后，在“管理证书”中，单击“浏览”并选择“默认 HPC Azure 管理”。 然后单击“下一步”。

	![节点模板][config_hpc12]

	d.在“提供服务信息”页上，选择你在前一步中创建的云服务和存储帐户。然后，单击“下一步”。

	![节点模板][config_hpc13]

	e.单击“下一步”以接受其余向导页上的默认值。然后，在“检查”选项卡上，单击“创建”，以创建节点模板。

	>[AZURE.NOTE]默认情况下，Azure 节点模板包括用于通过 HPC 群集管理器手动启动（设置）和停止节点的设置。可以选择配置计划以便自动启动和停止 Azure 节点。

## 向群集添加 Azure 节点

你现在将使用节点模板向群集添加 Azure 节点。将节点添加到群集可存储其配置信息，使你能够随时将这些节点作为云服务中的角色实例启动（设置）。在角色实例在云服务中运行后，仅针对 Azure 节点向你的订阅收费。

对于本教程，你将添加两个小型节点。

1. 在 HPC 群集管理器中的“节点管理”（在某些版本的 HPC Pack 中称为“资源管理”）中，然后在“操作”窗格中单击“添加节点”。

	![添加节点][add_node1]

2. 在添加节点向导的“选择部署方法”页上，单击“添加 Azure 节点”，然后单击“下一步”。

	![添加 Azure 节点][add_node1_1]

3. 在“指定新节点”页上，选择你之前创建的 Azure 节点模板（默认称作“默认 AzureNode 模板”）。然后指定大小为“小型”的 **2** 个节点，然后单击“下一步”。

	![指定节点][add_node2]

	有关可用大小的详细信息，请参阅 [Sizes for Cloud Services](/documentation/articles/cloud-services-sizes-specs/)（云服务的大小）。

4. 在“完成添加节点向导”页上，单击“完成”。

	 HPC 群集管理器中现在会出现名为“AzureCN-0001”和“AzureCN-0002”的两个 Azure 节点。它们的状态都是“未部署”。

	![已添加节点][add_node3]

## 启动 Azure 节点
当你希望在 Azure 中使用群集资源时，请使用 HPC 群集管理器启动（设置）Azure 节点并使它们联机。

1.	在 HPC 群集管理器中的“节点管理”（在某些版本的 HPC Pack 中称为“资源管理”）中，单击这两个或其中一个节点，然后在“操作”窗格中，单击“启动”。

	![启动节点][add_node4]

2. 在“启动 Azure 节点”对话框中，单击“启动”。

	![启动节点][add_node5]

	节点将转换到“正在预配”状态。查看预配日志可跟踪预配进度。

	![设置节点][add_node6]

3. 几分钟后，Azure 节点将完成预配并且处于“脱机”状态。在此状态下，角色实例尽管在运行，但不会接受群集作业。

4. 若要确认角色实例正在运行，请在[管理门户](https://manage.windowsazure.cn)中单击“云服务”> 你的云服务名称 >“实例”。

	![运行实例][view_instances1]

	你将看到两个辅助角色实例正在服务中运行。HPC Pack 还将自动部署两个 **HpcProxy** 实例（中等大小）以便处理头节点和 Azure 之间的通信。

5. 若要使 Azure 节点联机以便运行群集作业，请选择这些节点，右键单击，然后单击“联机”。

	![脱机节点][add_node7]

	HPC 群集管理器指示节点处于“联机”状态。

## 跨群集运行命令

若要检查安装，可使用 HPC Pack 的 **clusrun** 命令在一个或多个群集节点上运行命令或应用程序。下面举一个简单的例子，介绍如何使用 **clusrun** 获取 Azure 节点的 IP 配置。

1. 在头节点上，打开命令提示符。

2. 输入以下命令：

	`clusrun /nodes:azurecn* ipconfig`

3. 你将看到与下面类似的输出。

	![Clusrun][clusrun1]

## 运行测试作业

现在，提交在混合群集上运行的测试作业。此示例是一个简单的参数扫描作业（一种固有的并行计算）。此示例将运行多个子任务，这些子任务通过使用 **set /a** 命令向其本身添加整数。群集中的所有节点都参与完成针对 1 到 100 的整数的子任务。

1. 在 HPC 群集管理器的“作业管理”中，在“操作”窗格上单击“新建参数分析作业”。

	![新作业][test_job1]

2. 在“新建参数分析作业”对话框的“命令行”中，键入 `set /a *+*`（覆盖出现的默认命令行）。对于其余设置保留使用默认值，然后单击“提交”来提交作业。

	![参数分析][param_sweep1]

3. 在该作业完成后，双击“我的分析任务”作业。

4. 单击“查看任务”，然后单击某一子任务以便查看该子任务的计算输出。

	![任务结果][view_job361]

5. 若要查看哪一节点执行了针对该子任务的计算，请单击“分配的节点”。（你的群集可能会显示一个不同的节点名称。）

	![任务结果][view_job362]

## 停止 Azure 节点

在你试用群集后，可停止 Azure 节点，以免向你的帐户收取不必要的费用。这将停止云服务并且删除 Azure 角色实例。

1. 在 HPC 群集管理器中的“节点管理”（在某些版本的 HPC Pack 中称为“资源管理”）中，选择这两个 Azure 节点。然后，在“操作”窗格中，单击“停止”。

	![停止节点][stop_node1]

2. 在“停止 Azure 节点”对话框中，单击“停止”。

	![停止节点][stop_node2]

3. 节点将转换到“正在停止”状态。几分钟后，HPC 群集管理器将显示节点处于“未部署”状态。

	![未部署节点][stop_node4]

4. 若要确认角色实例是否不再在 Azure 中运行，请在[管理门户](https://manage.windowsazure.cn)中单击“云服务”> 你的云服务名称 >“实例”。将不会有任何实例部署在生产环境中。

	![无实例][view_instances2]

	本教程至此完毕。

## 后续步骤

* 参阅 [HPC Pack 2012 R2 和 HPC Pack 2012](http://go.microsoft.com/fwlink/p/?LinkID=263697) 的文档。

* 若要以更大的规模设置混合 HPC Pack 群集部署，请参阅 [Burst to Azure Worker Role Instances with Microsoft HPC Pack（使用 Microsoft HPC Pack 迸发到 Azure 辅助角色实例）](http://go.microsoft.com/fwlink/p/?LinkID=200493)。

* 有关在 Azure 中创建 HPC Pack 群集的其他方法，包括使用 Azure 资源管理器模板，请参阅 [HPC cluster options with Microsoft HPC Pack in Azure（在 Azure 中使用 Microsoft HPC Pack 时的 HPC 群集选项）](/documentation/articles/virtual-machines-windows-hpcpack-cluster-options/)。
* 有关 Azure 中的大型计算和 HPC 云解决方案范围的详细信息，请参阅 [Big Compute in Azure: Technical Resources for Batch and High Performance Computing (HPC)（Azure 中的大型计算：用于批处理和高性能计算 (HPC) 的技术资源）](/documentation/articles/big-compute-resources/)。


[Overview]: ./media/cloud-services-setup-hybrid-hpcpack-cluster/hybrid_cluster_overview.png
[install_hpc1]: ./media/cloud-services-setup-hybrid-hpcpack-cluster/install_hpc1.png
[install_hpc2]: ./media/cloud-services-setup-hybrid-hpcpack-cluster/install_hpc2.png
[install_hpc3]: ./media/cloud-services-setup-hybrid-hpcpack-cluster/install_hpc3.png
[install_hpc4]: ./media/cloud-services-setup-hybrid-hpcpack-cluster/install_hpc4.png
[install_hpc6]: ./media/cloud-services-setup-hybrid-hpcpack-cluster/install_hpc6.png
[install_hpc7]: ./media/cloud-services-setup-hybrid-hpcpack-cluster/install_hpc7.png
[install_hpc10]: ./media/cloud-services-setup-hybrid-hpcpack-cluster/install_hpc10.png
[upload_cert1]: ./media/cloud-services-setup-hybrid-hpcpack-cluster/upload_cert1.png
[createstorage1]: ./media/cloud-services-setup-hybrid-hpcpack-cluster/createstorage1.png
[createservice1]: ./media/cloud-services-setup-hybrid-hpcpack-cluster/createservice1.png
[config_hpc2]: ./media/cloud-services-setup-hybrid-hpcpack-cluster/config_hpc2.png
[config_hpc3]: ./media/cloud-services-setup-hybrid-hpcpack-cluster/config_hpc3.png
[config_hpc6]: ./media/cloud-services-setup-hybrid-hpcpack-cluster/config_hpc6.png
[config_hpc8]: ./media/cloud-services-setup-hybrid-hpcpack-cluster/config_hpc8.png
[config_hpc10]: ./media/cloud-services-setup-hybrid-hpcpack-cluster/config_hpc10.png
[config_hpc12]: ./media/cloud-services-setup-hybrid-hpcpack-cluster/config_hpc12.png
[config_hpc13]: ./media/cloud-services-setup-hybrid-hpcpack-cluster/config_hpc13.png
[add_node1]: ./media/cloud-services-setup-hybrid-hpcpack-cluster/add_node1.png
[add_node1_1]: ./media/cloud-services-setup-hybrid-hpcpack-cluster/add_node1_1.png
[add_node2]: ./media/cloud-services-setup-hybrid-hpcpack-cluster/add_node2.png
[add_node3]: ./media/cloud-services-setup-hybrid-hpcpack-cluster/add_node3.png
[add_node4]: ./media/cloud-services-setup-hybrid-hpcpack-cluster/add_node4.png
[add_node5]: ./media/cloud-services-setup-hybrid-hpcpack-cluster/add_node5.png
[add_node6]: ./media/cloud-services-setup-hybrid-hpcpack-cluster/add_node6.png
[add_node7]: ./media/cloud-services-setup-hybrid-hpcpack-cluster/add_node7.png
[view_instances1]: ./media/cloud-services-setup-hybrid-hpcpack-cluster/view_instances1.png
[clusrun1]: ./media/cloud-services-setup-hybrid-hpcpack-cluster/clusrun1.png
[test_job1]: ./media/cloud-services-setup-hybrid-hpcpack-cluster/test_job1.png
[param_sweep1]: ./media/cloud-services-setup-hybrid-hpcpack-cluster/param_sweep1.png
[view_job361]: ./media/cloud-services-setup-hybrid-hpcpack-cluster/view_job361.png
[view_job362]: ./media/cloud-services-setup-hybrid-hpcpack-cluster/view_job362.png
[stop_node1]: ./media/cloud-services-setup-hybrid-hpcpack-cluster/stop_node1.png
[stop_node2]: ./media/cloud-services-setup-hybrid-hpcpack-cluster/stop_node2.png
[stop_node4]: ./media/cloud-services-setup-hybrid-hpcpack-cluster/stop_node4.png
[view_instances2]: ./media/cloud-services-setup-hybrid-hpcpack-cluster/view_instances2.png

<!---HONumber=Mooncake_0503_2016-->
