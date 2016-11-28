<properties
	pageTitle="Azure Batch 池中的 Linux 节点 | Azure"
	description="了解如何处理 Azure Batch 中 Linux 虚拟机池上的并行计算工作负荷。"
	services="batch"
	documentationCenter="python"
	authors="mmacy"
	manager="timlt"
	editor="" />  


<tags
	ms.service="batch"
	ms.devlang="multiple"
	ms.topic="article"
	ms.tgt_pltfrm="vm-linux"
	ms.workload="na"
	ms.date="09/08/2016"
	wacn.date="11/28/2016"
	ms.author="marsma" />  


# 在 Azure Batch 池中预配 Linux 计算节点

可以使用 Azure Batch 在 Linux 和 Windows 虚拟机上运行并行计算工作负荷。本文详细说明如何使用 [Batch Python][py_batch_package] 和 [Batch .NET][api_net] 客户端库在 Batch 服务中创建 Linux 计算节点池。

> [AZURE.NOTE] [Application packages]Linux 计算节点目前不支持 (/documentation/articles/batch-application-packages/)。

<a name="virtual-machine-configuration"></a>
## 虚拟机配置

在 Batch 中创建计算节点池时，可以使用两个选项来选择节点大小和操作系统：“云服务配置”和“虚拟机配置”。

“云服务配置” *只* 提供 Windows 计算节点。[Sizes for Cloud Services](/documentation/articles/cloud-services-sizes-specs/)（云服务的大小）中列出了可用的计算节点大小，[Azure Guest OS releases and SDK compatibility matrix](/documentation/articles/cloud-services-guestos-update-matrix/)（Azure 来宾 OS 版本和 SDK 兼容性对照表）中列出了可用的操作系统。创建包含 Azure 云服务节点的池时，只需指定可在上述文章中所述的节点大小及其“OS 系列”。对于 Windows 计算节点池，最常使用的是云服务。

“虚拟机配置”为计算节点提供 Linux 和 Windows 映像。[Sizes for virtual machines in Azure](/documentation/articles/virtual-machines-linux-sizes/)（Azure 中虚拟机的大小）(Linux) 和 [Sizes for virtual machines in Azure](/documentation/articles/virtual-machines-windows-sizes/)（Azure 中虚拟机的大小）(Windows) 中列出了可用的计算节点大小。创建包含虚拟机配置节点的池时，必须指定节点的大小、虚拟机映像引用，以及要在节点上安装的 Batch 节点代理 SKU。

### 虚拟机映像引用

Batch 服务使用[虚拟机规模集](/documentation/articles/virtual-machine-scale-sets-overview/)提供 Linux 计算节点。这些虚拟机的操作系统映像由 [Azure 应用商店][vm_marketplace]提供。配置虚拟机映像引用时，需指定应用商店虚拟机映像的属性。创建虚拟机映像引用时，需提供以下属性：

| **映像引用属性** | **示例** |
| ----------------- | ------------------------ |
| 发布者 | Canonical |
| 产品 | UbuntuServer |
| SKU | 14\.04.4-LTS |
| 版本 | 最新 |

> [AZURE.TIP] 可以在 [Navigate and select Linux virtual machine images in Azure with CLI or PowerShell](/documentation/articles/virtual-machines-linux-cli-ps-findimage/)（使用 CLI 或 PowerShell 在 Azure 中导航和选择 Linux 虚拟机映像）中详细了解这些属性，以及如何列出应用商店映像。请注意，目前并非所有应用商店映像都与 Batch 兼容。有关详细信息，请参阅[节点代理 SKU](#node-agent-sku)。

<a name="node-agent-sku"></a>
### 节点代理 SKU

Batch 节点代理是一个程序，它在池中的每个节点上运行，并在节点与 Batch 服务之间提供命令和控制接口。节点代理对于不同操作系统有不同的实现（称为 SKU）。从根本上讲，在创建虚拟机配置时，需要先指定虚拟机映像引用，然后指定要在其上安装映像的代理节点。通常，每个节点代理 SKU 与多个虚拟机映像兼容。下面是节点代理 SKU 的几个示例：

* batch.node.ubuntu 14.04
* batch.node.centos 7
* batch.node.windows amd64

> [AZURE.IMPORTANT] 并非应用商店中的所有可用虚拟机映像都与当前可用的 Batch 节点代理兼容。必须使用 Batch SDK 来列出可用的节点代理 SKU 及其兼容的虚拟机映像。有关详细信息，请参阅本文稍后的[虚拟机映像列表](#list-of-virtual-machine-images)。

## 创建 Linux 池：Batch Python

以下代码片段示范如何使用[用于 Python 的 Azure Batch 客户端库][py_batch_package]创建 Ubuntu Server 计算节点池。有关 Batch Python 模块的参考文档可在此处找到： [azure.batch package ][py_batch_docs] 阅读文档。

此代码片段显式创建 [ImageReference][py_imagereference]，并指定它的每个属性（publisher、offer、SKU、version）。但是，我们建议在生产代码中使用 [list\_node\_agent\_skus][py_list_skus] 方法在运行时从可用映像和节点代理 SKU 组合中做出决定和选择。

python

	# Import the required modules from the
	# Azure Batch Client Library for Python
	import azure.batch.batch_service_client as batch
	import azure.batch.batch_auth as batchauth
	import azure.batch.models as batchmodels
	
	# Specify Batch account credentials
	account = "<batch-account-name>"
	key = "<batch-account-key>"
	batch_url = "<batch-account-url>"
	
	# Pool settings
	pool_id = "LinuxNodesSamplePoolPython"
	vm_size = "STANDARD_A1"
	node_count = 1
	
	# Initialize the Batch client
	creds = batchauth.SharedKeyCredentials(account, key)
	config = batch.BatchServiceClientConfiguration(creds, base_url = batch_url)
	client = batch.BatchServiceClient(config)
	
	# Create the unbound pool
	new_pool = batchmodels.PoolAddParameter(id = pool_id, vm_size = vm_size)
	new_pool.target_dedicated = node_count
	
	# Configure the start task for the pool
	start_task = batchmodels.StartTask()
	start_task.run_elevated = True
	start_task.command_line = "printenv AZ_BATCH_NODE_STARTUP_DIR"
	new_pool.start_task = start_task
	
	# Create an ImageReference which specifies the Marketplace
	# virtual machine image to install on the nodes.
	ir = batchmodels.ImageReference(
	    publisher = "Canonical",
	    offer = "UbuntuServer",
	    sku = "14.04.2-LTS",
	    version = "latest")
	
	# Create the VirtualMachineConfiguration, specifying
	# the VM image reference and the Batch node agent to
	# be installed on the node.
	vmc = batchmodels.VirtualMachineConfiguration(
	    image_reference = ir,
	    node_agent_sku_id = "batch.node.ubuntu 14.04")
	
	# Assign the virtual machine configuration to the pool
	new_pool.virtual_machine_configuration = vmc
	
	# Create pool in the Batch service
	client.pool.add(new_pool)

如上所述，我们建议不要显式创建 [ImageReference][py_imagereference]，而是使用 [list\_node\_agent\_skus][py_list_skus] 方法从当前支持的节点代理/应用商店映像组合中动态选择。以下 Python 代码片段演示此方法的用法。

python

	# Get the list of node agents from the Batch service
	nodeagents = client.account.list\_node\_agent\_skus()
	
	# Obtain the desired node agent
	ubuntu1404agent = next(agent for agent in nodeagents if "ubuntu 14.04" in agent.id)
	
	# Pick the first image reference from the list of verified references
	ir = ubuntu1404agent.verified_image_references[0]
	
	# Create the VirtualMachineConfiguration, specifying the VM image
	# reference and the Batch node agent to be installed on the node.
	vmc = batchmodels.VirtualMachineConfiguration(
	    image_reference = ir,
	    node_agent_sku_id = ubuntu1404agent.id)

## 创建 Linux 池：Batch .NET

以下代码片段示范如何使用 [Batch .NET][nuget_batch_net] 客户端库创建 Ubuntu Server 计算节点池。可以在 MSDN 上找到 [Batch .NET 参考文档][api_net]。

以下代码片段使用 [PoolOperations][net_pool_ops].[ListNodeAgentSkus][net_list_skus] 方法从当前支持的应用商店映像和节点代理 SKU 组合列表中做出选择。这种做法非常有效，因为支持的组合列表可能随着时间改变。最常见的情况是添加了支持的组合。

csharp

	// Pool settings
	const string poolId = "LinuxNodesSamplePoolDotNet";
	const string vmSize = "STANDARD\_A1";
	const int nodeCount = 1;
	
	// Obtain a collection of all available node agent SKUs.
	// This allows us to select from a list of supported
	// VM image/node agent combinations.
	List<NodeAgentSku> nodeAgentSkus =
	    batchClient.PoolOperations.ListNodeAgentSkus().ToList();
	
	// Define a delegate specifying properties of the VM image
	// that we wish to use.
	Func<ImageReference, bool> isUbuntu1404 = imageRef =>
	    imageRef.Publisher == "Canonical" &&
	    imageRef.Offer == "UbuntuServer" &&
	    imageRef.SkuId.Contains("14.04");
	
	// Obtain the first node agent SKU in the collection that matches
	// Ubuntu Server 14.04. Note that there are one or more image
	// references associated with this node agent SKU.
	NodeAgentSku ubuntuAgentSku = nodeAgentSkus.First(sku =>
	    sku.VerifiedImageReferences.Any(isUbuntu1404));
	
	// Select an ImageReference from those available for node agent.
	ImageReference imageReference =
	    ubuntuAgentSku.VerifiedImageReferences.First(isUbuntu1404);
	
	// Create the VirtualMachineConfiguration for use when actually
	// creating the pool
	VirtualMachineConfiguration virtualMachineConfiguration =
	    new VirtualMachineConfiguration(
	        imageReference: imageReference,
	        nodeAgentSkuId: ubuntuAgentSku.Id);
	
	// Create the unbound pool object using the VirtualMachineConfiguration
	// created above
	CloudPool pool = batchClient.PoolOperations.CreatePool(
	    poolId: poolId,
	    virtualMachineSize: vmSize,
	    virtualMachineConfiguration: virtualMachineConfiguration,
	    targetDedicated: nodeCount);
	
	// Commit the pool to the Batch service
	pool.Commit();

尽管上述代码片段使用了 [PoolOperations][net_pool_ops].[ListNodeAgentSkus][net_list_skus] 方法动态列出并从支持的映像和节点代理 SKU 组合中做出选择（建议的做法），但也可以显式配置 [ImageReference][net_imagereference]：

csharp

	ImageReference imageReference = new ImageReference(
	    publisher: "Canonical",
	    offer: "UbuntuServer",
	    skuId: "14.04.2-LTS",
	    version: "latest");

<a name="list-of-virtual-machine-images"></a>
## 虚拟机映像列表

下表列出了本文上次更新时，与可用 Batch 节点代理兼容的应用商店虚拟机映像。请务必注意，此列表并非永久不变，因为可能随时会添加或删除映像和节点代理。建议 Batch 应用程序和服务始终使用 [list\_node\_agent\_skus][py_list_skus] (Python) 和 [ListNodeAgentSkus][net_list_skus] (Batch .NET)，从当前可用的 SKU 中做出决定和选择。

> [AZURE.WARNING] 以下列表可随时更改。请始终使用 Batch API 中提供的**列出节点代理 SKU** 方法来列出，然后在运行 Batch 作业时从兼容的虚拟机和节点代理 SKU 中做出选择。

| **发布者** | **产品** | **映像 SKU** | **版本** | **节点代理 SKU ID** |
| ------- | ------- | ------- | ------- | ------- |
| Canonical | UbuntuServer | 14\.04.0-LTS | 最新 | batch.node.ubuntu 14.04 |
| Canonical | UbuntuServer | 14\.04.1-LTS | 最新 | batch.node.ubuntu 14.04 |
| Canonical | UbuntuServer | 14\.04.2-LTS | 最新 | batch.node.ubuntu 14.04 |
| Canonical | UbuntuServer | 14\.04.3-LTS | 最新 | batch.node.ubuntu 14.04 |
| Canonical | UbuntuServer | 14\.04.4-LTS | 最新 | batch.node.ubuntu 14.04 |
| Canonical | UbuntuServer | 14\.04.5-LTS | 最新 | batch.node.ubuntu 14.04 |
| Canonical | UbuntuServer | 16\.04.0-LTS | 最新 | batch.node.ubuntu 16.04 |
| Credativ | Debian | 8 | 最新 | batch.node.debian 8 |
| OpenLogic | CentOS | 7\.0 | 最新 | batch.node.centos 7 |
| OpenLogic | CentOS | 7\.1 | 最新 | batch.node.centos 7 |
| OpenLogic | CentOS-HPC | 7\.1 | 最新 | batch.node.centos 7 |
| OpenLogic | CentOS | 7\.2 | 最新 | batch.node.centos 7 |
| Oracle | Oracle-Linux | 7\.0 | 最新 | batch.node.centos 7 |
| SUSE | openSUSE | 13\.2 | 最新 | batch.node.opensuse 13.2 |
| SUSE | openSUSE-Leap | 42\.1 | 最新 | batch.node.opensuse 42.1 |
| SUSE | SLES-HPC | 12 | 最新 | batch.node.opensuse 42.1 |
| SUSE | SLES | 12-SP1 | 最新 | batch.node.opensuse 42.1 |
| microsoft-ads | standard-data-science-vm | standard-data-science-vm | 最新 | batch.node.windows amd64 |
| microsoft-ads | linux-data-science-vm | linuxdsvm | 最新 | batch.node.centos 7 |
| MicrosoftWindowsServer | WindowsServer | 2008-R2-SP1 | 最新 | batch.node.windows amd64 |
| MicrosoftWindowsServer | WindowsServer | 2012-Datacenter | 最新 | batch.node.windows amd64 |
| MicrosoftWindowsServer | WindowsServer | 2012-R2-Datacenter | 最新 | batch.node.windows amd64 |
| MicrosoftWindowsServer | WindowsServer | Windows-Server-Technical-Preview | 最新 | batch.node.windows amd64 |

<a name="connect-to-linux-nodes"></a>
## 连接到 Linux 节点

在开发期间或进行故障排除时，可能会发现需要登录到池中的节点。不同于 Windows 计算节点，你无法使用远程桌面协议 (RDP) 连接到 Linux 节点。相反，Batch 服务在每个节点上启用 SSH 访问以建立远程连接。

以下 Python 代码片段将在池中的每个节点上创建一个用户（远程连接时需要）。然后列显每个节点的安全外壳 (SSH) 连接信息。

python

	import datetime
	import getpass
	import azure.batch.batch_service_client as batch
	import azure.batch.batch_auth as batchauth
	import azure.batch.models as batchmodels
	
	# Specify your own account credentials
	batch_account_name = ''
	batch_account_key = ''
	batch_account_url = ''
	
	# Specify the ID of an existing pool containing Linux nodes
	# currently in the 'idle' state
	pool_id = ''
	
	# Specify the username and prompt for a password
	username = 'linuxuser'
	password = getpass.getpass()

	# Create a BatchClient
	credentials = batchauth.SharedKeyCredentials(
	    batch_account_name,
	    batch_account_key
	)
	batch_client = batch.BatchServiceClient(
	        credentials,
	        base_url=batch_account_url
	)

	# Create the user that will be added to each node in the pool
	user = batchmodels.ComputeNodeUser(username)
	user.password = password
	user.is_admin = True
	user.expiry_time = \
	    (datetime.datetime.today() + datetime.timedelta(days=30)).isoformat()

	# Get the list of nodes in the pool
	nodes = batch_client.compute_node.list(pool_id)

	# Add the user to each node in the pool and print
	# the connection information for the node
	for node in nodes:
	    # Add the user to the node
	    batch_client.compute_node.add_user(pool_id, node.id, user)

	    # Obtain SSH login information for the node
	    login = batch_client.compute_node.get_remote_login_settings(pool_id,
	                                                                node.id)

	    # Print the connection info for the node
	    print("{0} | {1} | {2} | {3}".format(node.id,
	                                         node.state,
	                                         login.remote_login_ip_address,
	                                         login.remote_login_port))

下面是针对包含四个 Linux 节点的池运行上述代码后的示例输出：

	Password:
	tvm-1219235766_1-20160414t192511z | ComputeNodeState.idle | 13.91.7.57 | 50000
	tvm-1219235766_2-20160414t192511z | ComputeNodeState.idle | 13.91.7.57 | 50003
	tvm-1219235766_3-20160414t192511z | ComputeNodeState.idle | 13.91.7.57 | 50002
	tvm-1219235766_4-20160414t192511z | ComputeNodeState.idle | 13.91.7.57 | 50001

请注意，在节点上创建用户时不需要指定密码，而可以指定 SSH 公钥。在 Python SDK 中，此操作可通过在 [ComputeNodeUser][py_computenodeuser] 上使用 **ssh\_public\_key** 参数来完成。在 .NET 中，此操作可通过使用 [ComputeNodeUser][net_computenodeuser].[SshPublicKey][net_ssh_key] 属性来完成。

## 定价

Azure Batch 构建在 Azure 云服务和 Azure 虚拟机技术基础之上。Batch 服务本身是免费提供的，这意味着，只需支付 Batch 解决方案使用的计算资源费用。如果选择“云服务配置”，我们会根据[云服务定价][cloud_services_pricing]结构收费。如果选择“虚拟机配置”，我们会根据[虚拟机定价][vm_pricing]结构收费。

## 后续步骤

### Batch Python 教程

有关如何配合 Python 使用 Batch 的更深入教程，请参阅 [Get started with the Azure Batch Python client](/documentation/articles/batch-python-tutorial/)（Azure Batch Python 客户端入门）。该教程随附的[代码示例][github_samples_pyclient]包含一个帮助器函数 `get_vm_config_for_distro`，用于演示获取虚拟机配置的另一种方法。

### Batch Python 代码示例

查看 GitHub 上 azure-batch-samples 存储库中的其他 [Python 代码示例][github_samples_py]，获取演示如何执行常见 Batch 操作（例如创建池、作业和任务）的多个脚本。Python 示例随附的 [README][github_py_readme] 文件包含有关如何安装所需包的详细信息。

### Batch 论坛

MSDN 上的 [Azure Batch 论坛][forum]是探讨 Batch 服务以及咨询其相关问题的不错场所。欢迎前往阅读这些帮忙解决“棘手问题”的贴子，并发布构建 Batch 解决方案时遇到的问题。

[api_net]: http://msdn.microsoft.com/zh-cn/library/azure/mt348682.aspx
[api_net_mgmt]: https://msdn.microsoft.com/zh-cn/library/azure/mt463120.aspx
[api_rest]: http://msdn.microsoft.com/zh-cn/library/azure/dn820158.aspx
[cloud_services_pricing]: /pricing/details/cloud-services/
[forum]: https://social.msdn.microsoft.com/forums/azure/en-US/home?forum=azurebatch
[github_samples]: https://github.com/Azure/azure-batch-samples
[github_samples_py]: https://github.com/Azure/azure-batch-samples/tree/master/Python/Batch
[github_samples_pyclient]: https://github.com/Azure/azure-batch-samples/blob/master/Python/Batch/article_samples/python_tutorial_client.py
[portal]: https://portal.azure.cn
[net_cloudpool]: https://msdn.microsoft.com/zh-cn/library/azure/microsoft.azure.batch.cloudpool.aspx
[net_computenodeuser]: https://msdn.microsoft.com/zh-cn/library/azure/microsoft.azure.batch.computenodeuser.aspx
[net_imagereference]: https://msdn.microsoft.com/zh-cn/library/azure/microsoft.azure.batch.imagereference.aspx
[net_list_skus]: https://msdn.microsoft.com/zh-cn/library/azure/microsoft.azure.batch.pooloperations.listnodeagentskus.aspx
[net_pool_ops]: https://msdn.microsoft.com/zh-cn/library/azure/microsoft.azure.batch.pooloperations.aspx
[net_ssh_key]: https://msdn.microsoft.com/zh-cn/library/azure/microsoft.azure.batch.computenodeuser.sshpublickey.aspx
[nuget_batch_net]: https://www.nuget.org/packages/Azure.Batch/
[rest_add_pool]: https://msdn.microsoft.com/zh-cn/library/azure/dn820174.aspx
[py_account_ops]: http://azure-sdk-for-python.readthedocs.org/en/dev/ref/azure.batch.operations.html#azure.batch.operations.AccountOperations
[py_azure_sdk]: https://pypi.python.org/pypi/azure
[py_batch_docs]: http://azure-sdk-for-python.readthedocs.org/en/dev/ref/azure.batch.html
[py_batch_package]: https://pypi.python.org/pypi/azure-batch
[py_computenodeuser]: http://azure-sdk-for-python.readthedocs.org/en/dev/ref/azure.batch.models.html#azure.batch.models.ComputeNodeUser
[py_imagereference]: http://azure-sdk-for-python.readthedocs.org/en/dev/ref/azure.batch.models.html#azure.batch.models.ImageReference
[py_list_skus]: http://azure-sdk-for-python.readthedocs.org/en/dev/ref/azure.batch.operations.html#azure.batch.operations.AccountOperations.list_node_agent_skus
[vm_marketplace]: https://azure.microsoft.com/marketplace/virtual-machines/
[vm_pricing]: /pricing/details/virtual-machines/

<!---HONumber=Mooncake_1017_2016-->
