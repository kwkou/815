<properties
	pageTitle="教程 - Azure Batch Python 客户端入门 | Azure"
	description="了解 Azure Batch 的基本概念，以及如何使用一个简单方案开发 Batch 服务"
	services="batch"
	documentationCenter="python"
	authors="mmacy"
	manager="timlt"
	editor=""/>  


<tags
	ms.service="batch"
	ms.devlang="python"
	ms.topic="hero-article"
	ms.tgt_pltfrm="na"
	ms.workload="big-compute"
	ms.date="09/08/2016"
	ms.author="marsma"
   	wacn.date="11/16/2016"/>  


# Azure Batch Python 客户端入门

> [AZURE.SELECTOR]
- [.NET](/documentation/articles/batch-dotnet-get-started/)
- [Python](/documentation/articles/batch-python-tutorial/)

在介绍以 Python 编写的小型 Batch 应用程序时，我们了解了 [Azure Batch][azure_batch] 和 [Batch Python][py_azure_sdk] 客户端的基础知识。我们将探讨两个示例脚本如何使用 Batch 服务来处理云中 Linux 虚拟机上的并行工作负荷，以及这些脚本如何与 [Azure 存储](/documentation/articles/storage-introduction/)交互来暂存和检索文件。你将了解常见的 Batch 应用程序工作流，并基本了解 Batch 的主要组件，例如作业、任务、池和计算节点。

![Batch 解决方案工作流（基础）][11]

## 先决条件

本文假设你有 Python 的实践知识，并熟悉 Linux。本文还假定，你能够满足下面为 Azure 和 Batch 及存储服务指定的帐户创建要求。

### 帐户

- **Azure 帐户**：如果没有 Azure 订阅，可以[创建一个 Azure 帐户][azure_free_account]。
- **Batch 帐户**：获取 Azure 订阅后，请[创建 Azure Batch 帐户](/documentation/articles/batch-account-create-portal/)。
- **存储帐户**：请参阅[关于 Azure 存储帐户](/documentation/articles/storage-create-storage-account/)中的[创建存储帐户](/documentation/articles/storage-create-storage-account/#create-a-storage-account/)。

### 代码示例

Python 教程[代码示例][github_article_samples]是 GitHub 上的 [azure-batch-samples][github_samples] 存储库中提供的众多 Batch 代码示例之一。单击存储库主页上的“克隆或下载”>“下载 ZIP”，或单击“azure-batch-samples-master.zip”直接下载链接，即可下载所有示例。[][github_samples_zip]解压缩 ZIP 文件的内容后，在 `article_samples` 目录中可找到本教程的两个脚本：

`/azure-batch-samples/Python/Batch/article_samples/python_tutorial_client.py`<br/> `/azure-batch-samples/Python/Batch/article_samples/python_tutorial_task.py`

### Python 环境

若要在本地工作站上运行 *python\_tutorial\_client.py* 示例脚本，需要与版本 **2.7** 或 **3.3-3.5** 兼容的 **Python 解释程序**。此脚本已在 Linux 和 Windows 上测试。

还需要安装 **Azure Batch** 和 **Azure 存储** Python 包。为此，可以使用以下网页上提供的 **pip** 和 *requirements.txt*：

`/azure-batch-samples/Python/Batch/requirements.txt`  


发出以下 **pip** 命令以安装 Batch 和存储包：

`pip install -r requirements.txt`  


或者，可以手动方式安装 [azure-batch][pypi_batch] 和 [azure-storage][pypi_storage] Python 包。

`pip install azure-batch==0.30.0rc4`<br/> `pip install azure-storage==0.30.0`

> [AZURE.TIP] 如果使用无特权帐户，可能需要在命令前面加上 `sudo`。例如，`sudo pip install -r requirements.txt`。有关如何安装 Python 包的详细信息，请参阅 readthedocs.io 中的 [Installing Packages][pypi_install]（安装包）。

## Batch Python 教程代码示例

Batch Python 教程代码示例由两个 Python 脚本和若干数据文件组成。

- **python\_tutorial\_client.py**：与 Batch 和存储空间服务交互，在计算节点（虚拟机）上执行并行工作负荷。*python\_tutorial\_client.py* 脚本在本地工作站上运行。

- **python\_tutorial\_task.py**：在 Azure 中的计算节点上运行从而执行实际工作的脚本。在示例中，*python\_tutorial\_task.py* 将分析从 Azure 存储下载的文件（输入文件）中的文本。然后，它会生成一个文本文件（输出文件），其中包含出现在输入文件中的头三个单词的列表。创建输出文件后，*python\_tutorial\_task.py* 会将该文件上载到 Azure 存储。这样，便可以将文件下载到工作站上运行的客户端脚本。*python\_tutorial\_task.py* 脚本在 Batch 服务中的多个计算节点上并行运行。

- **./data/taskdata*.txt**：这三个文本文件为计算节点上运行的任务提供输入。

下图演示了客户端和任务脚本执行的主要操作。此基本工作流是通过 Batch 创建的许多计算解决方案中常见的工作流。尽管它并未演示 Batch 服务提供的每项功能，但几乎每个 Batch 方案都包含此工作流的某些部分。

![Batch 示例工作流][8]

[**步骤 1.**](#step-1-create-storage-containers) 在 Azure Blob 存储中创建**容器**。<br/>
 
[**步骤 2.**](#step-2-upload-task-script-and-data-files) 将任务脚本和输入文件上载到容器。<br/> 

[**步骤 3.**](#step-3-create-batch-pool) 创建 Batch **池**。<br/>

  &nbsp;&nbsp;&nbsp;&nbsp;**3a.** 池 **StartTask** 在节点加入池时将任务脚本 (python\_tutorial\_task.py) 下载到节点。<br/> 

[**步骤 4.**](#step-4-create-batch-job) 创建 Batch **作业**。<br/> 

[**步骤 5.**](#step-5-add-tasks-to-job) 将**任务**添加到作业。<br/>

  &nbsp;&nbsp;&nbsp;&nbsp;**5a.** 任务计划在节点上执行。<br/>

  &nbsp;&nbsp;&nbsp;&nbsp;**5b.** 每项任务从 Azure 存储空间下载其输入数据，然后开始执行。<br/> 

[**步骤 6.**](#step-6-monitor-tasks) 监视任务。<br/>

  &nbsp;&nbsp;&nbsp;&nbsp;**6a.** 当任务完成时，会将其输出数据上载到 Azure 存储空间。<br/> 

[**步骤 7.**](#step-7-download-task-output) 从存储空间下载任务输出。

如前所述，并非每个 Batch 解决方案都会执行这些具体步骤，此类方案可能包含更多步骤，但本示例将演示 Batch 方案中的常见过程。

## 准备客户端脚本

在运行示例之前，请将 Batch 和存储帐户凭据添加到 *python\_tutorial\_client.py*。如果尚未这样做，请在偏好的编辑器中打开此文件，并使用凭据更新以下代码行。

python

	# Update the Batch and Storage account credential strings below with the values 
	# unique to your accounts.These are used when constructing connection strings 
	# for the Batch and Storage client objects.
	
	# Batch account credentials
	batch_account_name = "";
	batch_account_key  = "";
	batch_account_url  = "";
	
	# Storage account credentials
	storage_account_name = "";
	storage_account_key  = "";

可以在 [Azure 门户预览][azure_portal]中每项服务的帐户边栏选项卡中查找 Batch 和存储帐户凭据：

![门户中的 Batch 凭据][9] ![门户中的存储空间凭据][10]<br/>

在以下部分中，我们将分析脚本处理 Batch 服务中工作负荷的步骤。建议你在执行本文余下部分所述的步骤时，经常在编辑器中查看脚本。

导航到 **python\_tutorial\_client.py** 中的以下行以开始执行步骤 1：

python

         if __name__ == '__main__':

## 步骤 1：创建存储容器

![在 Azure 存储空间中创建容器][1] <br/>

Batch 包含的内置支持支持与 Azure 存储空间交互。存储帐户中的容器将为 Batch 帐户中运行的任务提供所需的文件。这些容器还提供存储任务生成的输出数据所需的位置。*python\_tutorial\_client.py* 脚本执行的第一个操作是在 [Azure Blob 存储](/documentation/articles/storage-introduction/#blob-storage/)中创建三个容器：

- **应用程序**：此容器存储任务运行的 Python 脚本 *python\_tutorial\_task.py*。
- **输入**：任务将从*输入*容器下载所要处理的数据文件。
- **输出**：当任务完成输入文件的处理时，会将其结果上载到*输出*容器。

为了与存储帐户交互并创建容器，我们将使用 [azure-storage][pypi_storage] 包来创建 [BlockBlobService][py_blockblobservice] 对象 -“Blob 客户端”。 然后，使用 Blob 客户端在存储帐户中创建三个容器。

python

	 # Create the blob client, for use in obtaining references to
	 # blob storage containers and uploading files to containers.
	 blob_client = azureblob.BlockBlobService(
	     account_name=_STORAGE_ACCOUNT_NAME,
	     account_key=_STORAGE_ACCOUNT_KEY)
	
	 # Use the blob client to create the containers in Azure Storage if they
	 # don't yet exist.
	 app_container_name = 'application'
	 input_container_name = 'input'
	 output_container_name = 'output'
	 blob_client.create_container(app_container_name, fail_on_exist=False)
	 blob_client.create_container(input_container_name, fail_on_exist=False)
	 blob_client.create_container(output_container_name, fail_on_exist=False)

创建容器之后，应用程序现在即可上载任务使用的文件。

> [AZURE.TIP] [How to use Azure Blob storage from Python](/documentation/articles/storage-python-how-to-use-blob-storage/) 对如何使用 Azure 存储容器和 Blob 做了全面的概述。当你开始使用 Batch 时，它应该位于阅读列表顶部附近。

## 步骤 2：上载任务脚本和数据文件

![将任务应用程序和输入（数据）文件上载到容器][2] <br/>

在文件上载操作中，*python\_tutorial\_client.py* 先定义**应用程序**和**输入**文件在本地计算机上的路径的集合，然后将这些文件上载到上一步骤创建的容器。

python

	 # Paths to the task script. This script will be executed by the tasks that
	 # run on the compute nodes.
	 application_file_paths = [os.path.realpath('python_tutorial_task.py')]
	
	 # The collection of data files that are to be processed by the tasks.
	 input_file_paths = [os.path.realpath('./data/taskdata1.txt'),
	                     os.path.realpath('./data/taskdata2.txt'),
	                     os.path.realpath('./data/taskdata3.txt')]
	
	 # Upload the application script to Azure Storage. This is the script that
	 # will process the data files, and is executed by each of the tasks on the
	 # compute nodes.
	 application_files = [
	     upload_file_to_container(blob_client, app_container_name, file_path)
	     for file_path in application_file_paths]
	
	 # Upload the data files. This is the data that will be processed by each of
	 # the tasks executed on the compute nodes in the pool.
	 input_files = [
	     upload_file_to_container(blob_client, input_container_name, file_path)
	     for file_path in input_file_paths]

使用列表推导，针对集合中的每个文件调用 `upload_file_to_container` 函数并填充两个 [ResourceFile][py_resource_file] 集合。`upload_file_to_container` 函数如下所示：

	def upload_file_to_container(block_blob_client, container_name, file_path):
	    """
	    Uploads a local file to an Azure Blob storage container.
	
	    :param block_blob_client: A blob service client.
	    :type block_blob_client: `azure.storage.blob.BlockBlobService`
	    :param str container_name: The name of the Azure Blob storage container.
	    :param str file_path: The local path to the file.
	    :rtype: `azure.batch.models.ResourceFile`
	    :return: A ResourceFile initialized with a SAS URL appropriate for Batch
	    tasks.
	    """
	    blob_name = os.path.basename(file_path)
	
	    print('Uploading file {} to container [{}]...'.format(file_path,
	                                                          container_name))
	
	    block_blob_client.create_blob_from_path(container_name,
	                                            blob_name,
	                                            file_path)
	
	    sas_token = block_blob_client.generate_blob_shared_access_signature(
	        container_name,
	        blob_name,
	        permission=azureblob.BlobPermissions.READ,
	        expiry=datetime.datetime.utcnow() + datetime.timedelta(hours=2))
	
	    sas_url = block_blob_client.make_blob_url(container_name,
	                                              blob_name,
	                                              sas_token=sas_token)
	
	    return batchmodels.ResourceFile(file_path=blob_name,
	                                    blob_source=sas_url)

### ResourceFiles

[ResourceFile][py_resource_file] 提供 Batch 中的任务，以及 Azure 存储空间中将在任务运行之前下载到计算节点的文件的 URL。[ResourceFile][py_resource_file].**blob\_source** 属性指定存在于 Azure 存储的文件的完整 URL。该 URL 还可以包含用于对文件进行安全访问的共享访问签名 (SAS)。Batch 中的大多数任务类型都包含 *ResourceFiles* 属性，这些类型包括：

- [CloudTask][py_task]
- [StartTask][py_starttask]
- [JobPreparationTask][py_jobpreptask]
- [JobReleaseTask][py_jobreltask]

本示例未使用 JobPreparationTask 或 JobReleaseTask 任务类型，但读者可以通过 [Run job preparation and completion tasks on Azure Batch compute nodes](/documentation/articles/batch-job-prep-release/)（在 Azure Batch 计算节点上运行作业准备和完成任务）详细了解这些任务类型。

### 共享访问签名 (SAS)

共享访问签名是一些字符串，可以提供对 Azure 存储空间中容器和 Blob 的安全访问。*python\_tutorial\_client.py* 脚本使用 Blob 和容器共享访问签名，并演示如何从存储空间服务获取这些共享访问签名字符串。

- **Blob 共享访问签名**：池的 StartTask 在从存储空间下载任务脚本和输入数据文件时使用 Blob 共享访问签名（请参阅下面的[步骤 3](#step-3-create-batch-pool)）。*python\_tutorial\_client.py* 中的 `upload_file_to_container` 函数包含可用于获取每个 Blob 的共享访问签名的代码。它通过调用存储模块中的 [BlockBlobService.make\_blob\_url][py_make_blob_url] 实现此目的。

- **容器共享访问签名**：每个任务在计算节点上完成其工作后，会将其输出文件上载到 Azure 存储中的*输出*容器。为此，*python\_tutorial\_task.py* 将使用提供容器写入访问权限的容器共享访问签名。*python\_tutorial\_client.py* 中的 `get_container_sas_token` 函数获取容器的共享访问签名，然后该签名将以命令行参数的形式传递给任务。步骤 5 [将任务添加到作业](#step-5-add-tasks-to-job)介绍了容器 SAS 的用法。

> [AZURE.TIP] 请查看有关共享访问签名的两篇系列教程的[第 1 部分：了解 SAS 模型](/documentation/articles/storage-dotnet-shared-access-signature-part-1/)和[第 2 部分：创建 SAS 并将其用于 Blob 服务](/documentation/articles/storage-dotnet-shared-access-signature-part-2/)，以详细了解如何提供对存储帐户中数据的安全访问。

## 步骤 3：创建 Batch 池

![创建 Batch 池][3] <br/>

Batch **池**是 Batch 执行作业任务时所在的计算节点（虚拟机）集合。

将任务脚本和数据文件上载到存储帐户之后，*python\_tutorial\_client.py* 将使用 Batch Python 模块开始与 Batch 服务交互。为此，将创建 [BatchServiceClient][py_batchserviceclient]：

python

	 # Create a Batch service client. We'll now be interacting with the Batch
	 # service in addition to Storage.
	 credentials = batchauth.SharedKeyCredentials(_BATCH_ACCOUNT_NAME,
	                                              _BATCH_ACCOUNT_KEY)
	
	 batch_client = batch.BatchServiceClient(
	     credentials,
	     base_url=_BATCH_ACCOUNT_URL)

接下来，调用 `create_pool`，在 Batch 帐户中创建计算节点池。

python

	def create_pool(batch_service_client, pool_id,
	                resource_files, distro, version):
	    """
	    Creates a pool of compute nodes with the specified OS settings.

	
	    :param batch_service_client: A Batch service client.
	    :type batch_service_client: `azure.batch.BatchServiceClient`
	    :param str pool_id: An ID for the new pool.
	    :param list resource_files: A collection of resource files for the pool's
	    start task.
	    :param str distro: The Linux distribution that should be installed on the
	    compute nodes, e.g. 'Ubuntu' or 'CentOS'.
	    :param str version: The version of the operating system for the compute
	    nodes, e.g. '15' or '14.04'.
	    """
	    print('Creating pool [{}]...'.format(pool_id))
	
	    # Create a new pool of Linux compute nodes using an Azure Virtual Machines
	    # Marketplace image. For more information about creating pools of Linux
	    # nodes, see:
	    # https://www.azure.cn/documentation/articles/batch-linux-nodes/
	
	    # Specify the commands for the pool's start task. The start task is run
	    # on each node as it joins the pool, and when it's rebooted or re-imaged.
	    # We use the start task to prep the node for running our task script.
	    task_commands = [
	        # Copy the python_tutorial_task.py script to the "shared" directory
	        # that all tasks that run on the node have access to.
	        'cp -r $AZ_BATCH_TASK_WORKING_DIR/* $AZ_BATCH_NODE_SHARED_DIR',
	        # Install pip and then the azure-storage module so that the task
	        # script can access Azure Blob storage
	        'apt-get update',
	        'apt-get -y install python-pip',
	        'pip install azure-storage']
	
	    # Get the virtual machine configuration for the desired distro and version.
	    # For more information about the virtual machine configuration, see:
	    # https://www.azure.cn/documentation/articles/batch-linux-nodes/
	    vm_config = get_vm_config_for_distro(batch_service_client, distro, version)
	
	    new_pool = batch.models.PoolAddParameter(
	        id=pool_id,
	        virtual_machine_configuration=vm_config,
	        vm_size=_POOL_VM_SIZE,
	        target_dedicated=_POOL_NODE_COUNT,
	        start_task=batch.models.StartTask(
	            command_line=wrap_commands_in_shell('linux', task_commands),
	            run_elevated=True,
	            wait_for_success=True,
	            resource_files=resource_files),
	        )
	
	    try:
	        batch_service_client.pool.add(new_pool)
	    except batchmodels.batch_error.BatchErrorException as err:
	        print_batch_exception(err)
	        raise
	}

创建池时，应定义 [PoolAddParameter][py_pooladdparam] 用于指定池的几个属性：

- 池的 **ID**（*id* - 必需）<p/>与 Batch 中的大多数实体一样，新池在 Batch 帐户中必须具有唯一 ID。代码将使用池 ID 引用此池，这也是在 Azure [门户][azure_portal]中识别池的方式。

- **计算节点数**（*target\_dedicated* - 必需）<p/>此属性指定应在池中部署多少个 VM。必须注意，所有 Batch 帐户都有默认**配额**，用于限制 Batch 帐户中的**核心**（因此也包括计算节点）数目。可以在 [Quotas and limits for the Azure Batch service](/documentation/articles/batch-quota-limit/)（Azure Batch 服务的配额和限制）中找到默认配额以及如何[提高配额](/documentation/articles/batch-quota-limit/#increase-a-quota/)（例如 Batch 帐户中的核心数目上限）的说明。如果你有类似于“为什么我的池不能包含 X 个以上的节点？”的疑惑，则原因可能在于此核心配额。

- 节点的**操作系统**（*virtual\_machine\_configuration* **或** *cloud\_service\_configuration* - 必需）<p/>在 *python\_tutorial\_client.py* 中，我们使用通过 `get_vm_config_for_distro` 帮助器函数获取的 [VirtualMachineConfiguration][py_vm_config] 来创建 Linux 节点池。此帮助器函数使用 [list\_node\_agent\_skus][py_list_skus] 来获取兼容的 [Azure 虚拟机应用商店][vm_marketplace]映像列表并从中选择映像。可以改为指定 [CloudServiceConfiguration][py_cs_config] 并从云服务创建 Windows 节点池。有关这两种配置的详细信息，请参阅 [Provision Linux compute nodes in Azure Batch pools](/documentation/articles/batch-linux-nodes/)（在 Azure Batch 池中预配 Linux 计算节点）。

- **计算节点的大小**（*vm\_size* - 必需）<p/>由于我们要为 [VirtualMachineConfiguration][py_vm_config] 指定 Linux 节点，因此应根据 [Sizes for virtual machines in Azure](/documentation/articles/virtual-machines-linux-sizes/)（Azure 中虚拟机的大小）指定 VM 大小（在本示例中为 `STANDARD_A1`）。同样，请参阅 [Provision Linux compute nodes in Azure Batch pools](/documentation/articles/batch-linux-nodes/)（在 Azure Batch 池中预配 Linux 计算节点）以获取详细信息。

- **启动任务**（*start\_task* - 可选）<p/>还可以连同上述物理节点属性一起指定池的 [StartTask][py_starttask]（不是必需的）。StartTask 在每个节点加入池以及每次重新启动节点时在该节点上运行。StartTask 特别适合用于准备计算节点，以便执行任务，例如安装任务将要运行的应用程序。<p/>在本示例应用程序中，StartTask 将它从存储空间下载的文件（使用 StartTask 的 **resource\_files** 属性指定），从 StartTask *工作目录*复制到在节点上运行的所有任务可以访问的*共享目录*。本质上，这会在节点加入池时，将 `python_tutorial_task.py` 复制到每个节点上的共享目录，因此该节点上运行的任何任务都可以访问它。

可能注意到了对 `wrap_commands_in_shell` 帮助器函数的调用。此函数采用不同命令的集合，并针对任务的命令行属性创建适当的单个命令行。

此外，在上述代码片段中，值得注意的问题是，StartTask 的 **command\_line** 属性中使用了两个环境变量：`AZ_BATCH_TASK_WORKING_DIR` 和 `AZ_BATCH_NODE_SHARED_DIR`。将自动为 Batch 池中的每个计算节点配置多个特定于 Batch 的环境变量。由任务执行的任何进程都可以访问这些环境变量。

> [AZURE.TIP] 若要深入了解 Batch 池中计算节点上可用的环境变量，以及有关任务工作目录的信息，请参阅 [overview of Azure Batch features](/documentation/articles/batch-api-basics/)（Azure Batch 功能概述）中的 **Environment settings for tasks**（任务的环境设置）及 **Files and directories**（文件和目录）。

## 步骤 4：创建 Batch 作业

![创建 Batch 作业][4]<br/>

Batch **作业**是任务的集合，它与计算节点池相关联。作业中的任务在关联池的计算节点上执行。

不仅可以使用作业来组织和跟踪相关工作负荷中的任务，还可以使用它来实施特定的约束，例如作业（并扩展到其任务）的最大运行时，以及 Batch 帐户中其他作业的相关作业优先级。不过在本示例中，该作业仅与步骤 3 中创建的池关联。未配置任何其他属性。

所有 Batch 作业都与特定的池关联。此关联指示将要在其上执行作业任务的节点。可以通过 [PoolInformation][py_poolinfo] 属性指定池，如以下代码片段所示。

python

	def create_job(batch_service_client, job_id, pool_id):
	    """
	    Creates a job with the specified ID, associated with the specified pool.
	
	    :param batch_service_client: A Batch service client.
	    :type batch_service_client: `azure.batch.BatchServiceClient`
	    :param str job_id: The ID for the job.
	    :param str pool_id: The ID for the pool.
	    """
	    print('Creating job [{}]...'.format(job_id))
	
	    job = batch.models.JobAddParameter(
	        job_id,
	        batch.models.PoolInformation(pool_id=pool_id))
	
	    try:
	        batch_service_client.job.add(job)
	    except batchmodels.batch_error.BatchErrorException as err:
	        print_batch_exception(err)
	        raise

创建作业后，可以添加任务来执行工作。

## 步骤 5：将任务添加到作业

![将任务添加到作业][5]<br/> 
*(1) 将任务添加到作业；(2) 将任务计划为在节点上运行；(3) 任务下载要处理的数据文件*

Batch **任务**是在计算节点上执行的各个工作单位。任务有一个命令行，可运行在该命令行中指定的脚本或可执行文件。

若要实际执行工作，必须将任务添加到作业。每个 [CloudTask][py_task] 都是使用命令行属性以及任务在其命令行自动执行之前下载到节点的 [ResourceFiles][py_resource_file]（如同池的 StartTask）进行配置的。在本示例中，每个任务只处理一个文件。因此，其 ResourceFiles 集合包含单个元素。

python

	def add_tasks(batch_service_client, job_id, input_files,
	              output_container_name, output_container_sas_token):
	    """
	    Adds a task for each input file in the collection to the specified job.
	
	    :param batch_service_client: A Batch service client.
	    :type batch_service_client: `azure.batch.BatchServiceClient`
	    :param str job_id: The ID of the job to which to add the tasks.
	    :param list input_files: A collection of input files. One task will be
	     created for each input file.
	    :param output_container_name: The ID of an Azure Blob storage container to
	    which the tasks will upload their results.
	    :param output_container_sas_token: A SAS token granting write access to
	    the specified Azure Blob storage container.
	    """
	
	    print('Adding {} tasks to job [{}]...'.format(len(input_files), job_id))
	
	    tasks = list()
	
	    for input_file in input_files:
	
	        command = ['python $AZ_BATCH_NODE_SHARED_DIR/python_tutorial_task.py '
	                   '--filepath {} --numwords {} --storageaccount {} '
	                   '--storagecontainer {} --sastoken "{}"'.format(
	                    input_file.file_path,
	                    '3',
	                    _STORAGE_ACCOUNT_NAME,
	                    output_container_name,
	                    output_container_sas_token)]
	
	        tasks.append(batch.models.TaskAddParameter(
	                'topNtask{}'.format(input_files.index(input_file)),
	                wrap_commands_in_shell('linux', command),
	                resource_files=[input_file]
	                )
	        )
	
	    batch_service_client.task.add_collection(job_id, tasks)

> [AZURE.IMPORTANT] 在访问环境变量（例如 `$AZ_BATCH_NODE_SHARED_DIR`）或执行节点的 `PATH` 中找不到的应用程序时，任务命令行必须显式调用 shell，例如，包含 `/bin/sh -c MyTaskApplication $MY_ENV_VAR`。如果任务在节点的 `PATH` 中执行应用程序，而且不引用任何环境变量，则就不必要满足此要求。

在上述代码片段中的 `for` 循环内，可以看到已构造任务的命令行，其中有五个命令行参数已传递到 *python\_tutorial\_task.py*：

1. **filepath**：这是节点上现有文件的本地路径。在上面步骤 2 所述的 `upload_file_to_container` 中创建 ResourceFile 对象时，已将文件名用于此属性（ResourceFile 构造函数中的 `file_path` 参数）。这意味着可以在节点上 *python\_tutorial\_task.py* 所在的同一目录中找到该文件。

2. **numwords**：前 *N* 个单词应写入输出文件。

3. **storageaccount**：存储帐户的名称，其拥有任务输出应上载到的容器。

4. **storagecontainer**：输出文件应上载到的存储容器的名称。

5. **sastoken**：共享访问签名 (SAS)，提供对 Azure 存储中**输出**容器的写访问权限。*Python\_tutorial\_task.py* 脚本在创建其 BlockBlobService 引用时使用此共享访问签名。此参数提供对容器的写访问权限，且不需要存储帐户的访问密钥。

python

	# NOTE: Taken from python\_tutorial\_task.py
	
	# Create the blob client using the container's SAS token.
	# This allows us to create a client that provides write
	# access only to the container.
	blob_client = azureblob.BlockBlobService(account_name=args.storageaccount,
	                                         sas_token=args.sastoken)

## 步骤 6：监视任务

![监视任务][6]<br/>
*脚本将会：(1) 监视任务的完成状态，(2) 监视将结果数据上载到 Azure 存储的任务*

任务在添加到作业后，将自动排入队列并计划在与作业关联的池中的计算节点上执行。根据你指定的设置，Batch 将为你处理所有任务排队、计划、重试和其他任务管理工作。

监视任务的执行有许多方法。*python\_tutorial\_client.py* 中的 `wait_for_tasks_to_complete` 函数提供监视任务特定状态的简单示例，在本例中为 [completed][py_taskstate] 状态。

python

	def wait_for_tasks_to_complete(batch_service_client, job_id, timeout):
	    """
	    Returns when all tasks in the specified job reach the Completed state.
	
	    :param batch_service_client: A Batch service client.
	    :type batch_service_client: `azure.batch.BatchServiceClient`
	    :param str job_id: The id of the job whose tasks should be to monitored.
	    :param timedelta timeout: The duration to wait for task completion. If all
	    tasks in the specified job do not reach Completed state within this time
	    period, an exception will be raised.
	    """
	    timeout_expiration = datetime.datetime.now() + timeout
	
	    print("Monitoring all tasks for 'Completed' state, timeout in {}..."
	          .format(timeout), end='')
	
	    while datetime.datetime.now() < timeout_expiration:
	        print('.', end='')
	        sys.stdout.flush()
	        tasks = batch_service_client.task.list(job_id)
	
	        incomplete_tasks = [task for task in tasks if
	                            task.state != batchmodels.TaskState.completed]
	        if not incomplete_tasks:
	            print()
	            return True
	        else:
	            time.sleep(1)
	
	    print()
	    raise RuntimeError("ERROR: Tasks did not reach 'Completed' state within "
	                       "timeout period of " + str(timeout))

## 步骤 7：下载任务输出

![从存储空间下载任务输出][7]

完成作业后，可以从 Azure 存储空间下载任务的输出。可通过在 *python\_tutorial\_client.py* 中调用 `download_blobs_from_container` 来实现此目的：

python

	def download_blobs_from_container(block_blob_client,
	                                  container_name, directory_path):
	    """
	    Downloads all blobs from the specified Azure Blob storage container.
	
	    :param block_blob_client: A blob service client.
	    :type block_blob_client: `azure.storage.blob.BlockBlobService`
	    :param container_name: The Azure Blob storage container from which to
	     download files.
	    :param directory_path: The local directory to which to download the files.
	    """
	    print('Downloading all files from container [{}]...'.format(
	        container_name))
	
	    container_blobs = block_blob_client.list_blobs(container_name)
	
	    for blob in container_blobs.items:
	        destination_file_path = os.path.join(directory_path, blob.name)
	
	        block_blob_client.get_blob_to_path(container_name,
	                                           blob.name,
	                                           destination_file_path)
	
	        print('  Downloaded blob [{}] from container [{}] to {}'.format(
	            blob.name,
	            container_name,
	            destination_file_path))
	
	    print('  Download complete!')

> [AZURE.NOTE] 在 *python\_tutorial\_client.py* 中调用 `download_blobs_from_container` 可指定应将文件下载到主目录。可以随意修改此输出位置。

## 步骤 8：删除容器

由于你需要对位于 Azure 存储空间中的数据付费，因此我们建议删除 Batch 作业不再需要的所有 Blob。在 *python\_tutorial\_client.py* 中，可通过调用 [BlockBlobService.delete\_container][py_delete_container] 三次来实现此目的：

	# Clean up storage resources
	print('Deleting containers...')
	blob_client.delete_container(app_container_name)
	blob_client.delete_container(input_container_name)
	blob_client.delete_container(output_container_name)

## 步骤 9：删除作业和池

在最后一个步骤，系统将提示删除 *python\_tutorial\_client.py* 脚本创建的作业和池。虽然作业和任务本身不收费，但计算节点收费。因此，建议你只在需要的时候分配节点。在维护过程中，可能需要删除未使用的池。

BatchServiceClient 的 [JobOperations][py_job] 和 [PoolOperations][py_pool] 都有对应的删除方法（在确认删除时调用）：

python
	
	# Clean up Batch resources (if the user so chooses).
	if query_yes_no('Delete job?') == 'yes':
	    batch_client.job.delete(_JOB_ID)
	
	if query_yes_no('Delete pool?') == 'yes':
	    batch_client.pool.delete(_POOL_ID)

> [AZURE.IMPORTANT] 请记住，你需要支付计算资源的费用，而删除未使用的池可将费用降到最低。另请注意，删除池也会删除该池内的所有计算节点，并且删除池后，将无法恢复节点上的任何数据。

## 运行示例脚本

从教程[代码示例][github_article_samples]运行 *python\_tutorial\_client.py* 脚本时，控制台输出如下所示。出现 `Monitoring all tasks for 'Completed' state, timeout in 0:20:00...` 后将会暂停，此时会创建、启动池的计算节点，然后执行池启动任务中的命令。在执行期间和之后，可以使用 [Azure 门户预览][azure_portal]监视池、计算节点、作业和任务。使用 [Azure 门户预览][azure_portal]或 [Azure 存储资源管理器][storage_explorer]可以查看应用程序创建的存储资源（容器和 Blob）。

以默认配置运行应用程序时，典型的执行时间**大约为 5-7 分钟**。

	Sample start: 2016-05-20 22:47:10

	Uploading file /home/user/py_tutorial/python_tutorial_task.py to container [application]...
	Uploading file /home/user/py_tutorial/data/taskdata1.txt to container [input]...
	Uploading file /home/user/py_tutorial/data/taskdata2.txt to container [input]...
	Uploading file /home/user/py_tutorial/data/taskdata3.txt to container [input]...
	Creating pool [PythonTutorialPool]...
	Creating job [PythonTutorialJob]...
	Adding 3 tasks to job [PythonTutorialJob]...
	Monitoring all tasks for 'Completed' state, timeout in 0:20:00..........................................................................
	  Success! All tasks reached the 'Completed' state within the specified timeout period.
	Downloading all files from container [output]...
	  Downloaded blob [taskdata1_OUTPUT.txt] from container [output] to /home/user/taskdata1_OUTPUT.txt
	  Downloaded blob [taskdata2_OUTPUT.txt] from container [output] to /home/user/taskdata2_OUTPUT.txt
	  Downloaded blob [taskdata3_OUTPUT.txt] from container [output] to /home/user/taskdata3_OUTPUT.txt
	  Download complete!
	Deleting containers...
	
	Sample end: 2016-05-20 22:53:12
	Elapsed time: 0:06:02
	
	Delete job? [Y/n]
	Delete pool? [Y/n]

	Press ENTER to exit...

## 后续步骤

随意更改 *python\_tutorial\_client.py* 和 *python\_tutorial\_task.py*，体验不同的计算方案。例如，尝试将执行延迟添加到 *python\_tutorial\_task.py*，模拟长时间运行的任务并在门户中监视这些任务。尝试添加更多任务，或调整计算节点的数目。添加逻辑来检查并允许使用现有池，以加速执行时间。

熟悉 Batch 解决方案的基本工作流后，接下来可以深入了解 Batch 服务的其他功能。

- 查看 [Overview of Azure Batch features](/documentation/articles/batch-api-basics/)（Azure Batch 功能概述）一文。如果对该服务不熟悉，建议阅读此文。
- 从 [Batch 学习路径][batch_learning_path]中**有关开发的深度知识**下面列出的其他 Batch 开发文章着手。
- 通过 [TopNWords][github_topnwords] 示例了解有关使用 Batch 处理“前 N 个单词”工作负荷的不同实现方式。

[azure_batch]: /home/features/batch/
[azure_free_account]: /pricing/1rmb-trial/
[azure_portal]: https://portal.azure.cn
[batch_learning_path]: https://azure.microsoft.com/documentation/learning-paths/batch/
[blog_linux]: http://blogs.technet.com/b/windowshpc/archive/2016/03/30/introducing-linux-support-on-azure-batch.aspx
[github_samples]: https://github.com/Azure/azure-batch-samples
[github_samples_zip]: https://github.com/Azure/azure-batch-samples/archive/master.zip
[github_topnwords]: https://github.com/Azure/azure-batch-samples/tree/master/CSharp/TopNWords
[github_article_samples]: https://github.com/Azure/azure-batch-samples/tree/master/Python/Batch/article_samples

[nuget_packagemgr]: https://visualstudiogallery.msdn.microsoft.com/27077b70-9dad-4c64-adcf-c7cf6bc9970c
[nuget_restore]: https://docs.nuget.org/consume/package-restore/msbuild-integrated#enabling-package-restore-during-build

[py_account_ops]: http://azure-sdk-for-python.readthedocs.org/en/latest/ref/azure.batch.operations.html#azure.batch.operations.AccountOperations
[py_azure_sdk]: https://pypi.python.org/pypi/azure
[py_batch_docs]: http://azure-sdk-for-python.readthedocs.org/en/latest/ref/azure.batch.html
[py_batch_package]: https://pypi.python.org/pypi/azure-batch
[py_batchserviceclient]: http://azure-sdk-for-python.readthedocs.io/en/latest/ref/azure.batch.html#azure.batch.BatchServiceClient
[py_blockblobservice]: http://azure.github.io/azure-storage-python/ref/azure.storage.blob.blockblobservice.html#azure.storage.blob.blockblobservice.BlockBlobService
[py_cloudtask]: http://azure-sdk-for-python.readthedocs.io/en/latest/ref/azure.batch.models.html#azure.batch.models.CloudTask
[py_computenodeuser]: http://azure-sdk-for-python.readthedocs.org/en/latest/ref/azure.batch.models.html#azure.batch.models.ComputeNodeUser
[py_cs_config]: http://azure-sdk-for-python.readthedocs.io/en/latest/ref/azure.batch.models.html#azure.batch.models.CloudServiceConfiguration
[py_delete_container]: http://azure.github.io/azure-storage-python/ref/azure.storage.blob.baseblobservice.html#azure.storage.blob.baseblobservice.BaseBlobService.delete_container
[py_gen_blob_sas]: http://azure.github.io/azure-storage-python/ref/azure.storage.blob.baseblobservice.html#azure.storage.blob.baseblobservice.BaseBlobService.generate_blob_shared_access_signature
[py_gen_container_sas]: http://azure.github.io/azure-storage-python/ref/azure.storage.blob.baseblobservice.html#azure.storage.blob.baseblobservice.BaseBlobService.generate_container_shared_access_signature
[py_image_ref]: http://azure-sdk-for-python.readthedocs.io/en/latest/ref/azure.batch.models.html#azure.batch.models.ImageReference
[py_imagereference]: http://azure-sdk-for-python.readthedocs.org/en/latest/ref/azure.batch.models.html#azure.batch.models.ImageReference
[py_job]: http://azure-sdk-for-python.readthedocs.io/en/latest/ref/azure.batch.operations.html#azure.batch.operations.JobOperations
[py_jobpreptask]: http://azure-sdk-for-python.readthedocs.io/en/latest/ref/azure.batch.models.html#azure.batch.models.JobPreparationTask
[py_jobreltask]: http://azure-sdk-for-python.readthedocs.io/en/latest/ref/azure.batch.models.html#azure.batch.models.JobReleaseTask
[py_list_skus]: http://azure-sdk-for-python.readthedocs.io/en/latest/ref/azure.batch.operations.html#azure.batch.operations.AccountOperations.list_node_agent_skus
[py_make_blob_url]: http://azure.github.io/azure-storage-python/ref/azure.storage.blob.baseblobservice.html#azure.storage.blob.baseblobservice.BaseBlobService.make_blob_url
[py_pool]: http://azure-sdk-for-python.readthedocs.io/en/latest/ref/azure.batch.operations.html#azure.batch.operations.PoolOperations
[py_pooladdparam]: http://azure-sdk-for-python.readthedocs.io/en/latest/ref/azure.batch.models.html#azure.batch.models.PoolAddParameter
[py_poolinfo]: http://azure-sdk-for-python.readthedocs.io/en/latest/ref/azure.batch.models.html#azure.batch.models.PoolInformation
[py_resource_file]: http://azure-sdk-for-python.readthedocs.io/en/latest/ref/azure.batch.models.html#azure.batch.models.ResourceFile
[py_samples_github]: https://github.com/Azure/azure-batch-samples/tree/master/Python/Batch/
[py_starttask]: http://azure-sdk-for-python.readthedocs.io/en/latest/ref/azure.batch.models.html#azure.batch.models.StartTask
[py_starttask]: http://azure-sdk-for-python.readthedocs.io/en/latest/ref/azure.batch.models.html#azure.batch.models.StartTask
[py_task]: http://azure-sdk-for-python.readthedocs.io/en/latest/ref/azure.batch.models.html#azure.batch.models.CloudTask
[py_taskstate]: http://azure-sdk-for-python.readthedocs.io/en/latest/ref/azure.batch.models.html#azure.batch.models.TaskState
[py_vm_config]: http://azure-sdk-for-python.readthedocs.io/en/latest/ref/azure.batch.models.html#azure.batch.models.VirtualMachineConfiguration
[pypi_batch]: https://pypi.python.org/pypi/azure-batch
[pypi_storage]: https://pypi.python.org/pypi/azure-storage

[pypi_install]: http://python-packaging-user-guide.readthedocs.io/en/latest/installing/
[storage_explorer]: http://storageexplorer.com/
[visual_studio]: https://www.visualstudio.com/products/vs-2015-product-editions
[vm_marketplace]: https://azure.microsoft.com/marketplace/virtual-machines/

[1]: ./media/batch-python-tutorial/batch_workflow_01_sm.png "在 Azure 存储空间中创建容器"
[2]: ./media/batch-python-tutorial/batch_workflow_02_sm.png "将任务应用程序和输入（数据）文件上载到容器"
[3]: ./media/batch-python-tutorial/batch_workflow_03_sm.png "创建 Batch 池"
[4]: ./media/batch-python-tutorial/batch_workflow_04_sm.png "创建 Batch 作业"
[5]: ./media/batch-python-tutorial/batch_workflow_05_sm.png "将任务添加到作业"
[6]: ./media/batch-python-tutorial/batch_workflow_06_sm.png "监视任务"
[7]: ./media/batch-python-tutorial/batch_workflow_07_sm.png "从存储空间下载任务输出"
[8]: ./media/batch-python-tutorial/batch_workflow_sm.png "Batch 解决方案工作流（完整流程图）"
[9]: ./media/batch-python-tutorial/credentials_batch_sm.png "门户中的 Batch 凭据"
[10]: ./media/batch-python-tutorial/credentials_storage_sm.png "门户中的存储空间凭据"
[11]: ./media/batch-python-tutorial/batch_workflow_minimal_sm.png "Batch 解决方案工作流（精简流程图）"

<!---HONumber=Mooncake_1107_2016-->
