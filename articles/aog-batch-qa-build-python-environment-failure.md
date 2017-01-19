<properties
	pageTitle="搭建 Windows Batch Python 环境"
	description="解决搭建 Windows Batch Python 环境出错的问题"
	service=""
	resource="batch"
	authors=""
	displayOrder=""
	selfHelpType=""
	supportTopicIds=""
	productPesIds=""
	resourceTags="Batch,Python"
	cloudEnvironments="MoonCake" />
<tags
	ms.service="batch-aog"
	ms.date=""
	wacn.date="01/12/2017" />
# 搭建 Windows Batch Python 环境

## **问题描述**

参照[ Azure Batch Python 客户端入门](/documentation/articles/batch-python-tutorial/)文档，在 China Azure 环境下运行 Batch Python 示例时发现会遇到诸多问题。

## **问题分析**

该资料由微软官方提供，目前还无法直接在由世纪互联运营的 China Azure 环境下直接使用，主要问题如下：

1. 镜像问题：

	官方示例中是基于SKU方式来创建Batch计算节点，但因网络原因，目前中国无法访问 Azure 虚拟机应用商店映像(已经向产品组提交 Bug,后续会修复该问题)：

	    new_pool = batch.models.PoolAddParameter(
	            id=pool_id,
	        virtual_machine_configuration=batchmodels.VirtualMachineConfiguration(
	            image_reference=image_ref_to_use,
	            node_agent_sku_id=sku_to_use),
	        vm_size=_POOL_VM_SIZE,
	        target_dedicated=_POOL_NODE_COUNT,
	        start_task=batch.models.StartTask(
	            command_line=common.helpers.wrap_commands_in_shell('linux',
	                                                               task_commands),
	            run_elevated=True,
	            wait_for_success=True,
	            resource_files=resource_files),
	    )

2. 环境问题：

	以下是本地运行实例、Batch 节点依赖的运行时环境：

	- python 2.7 或 3.3+ 
	- pip 9.0.1  
	- cryptography
	- azure-batch
	- azure-storage  

我们在尝试安装以上依赖环境时遇到了诸多问题，首要的问题就是如何在 Batch 节点中安装 Python 环境（示例中是基于 Linux Batch 节点，并未给出 Windows 安装 Python 的示例），因为 Windows Batch Pool 也是基于云服务建立的计算节点，因此我们参考了云服务中通过 PowerShell 安装 Python 的资料，参考：[云服务配置 Python 环境](/documentation/articles/cloud-services-python-ptvs/)。

在测试过程中，我们发现 China Azure 网络环境访问 [Python 官网](https://www.python.org/)较慢，因此在访问国外资源、国外镜像的时候会遇到各种问题，导致运行示例依赖的运行时环境无法配置成功。

## **解决方法**

我们纠正了较多的代码、脚本，从[这里](https://github.com/hello-azure/azure-batch-windows-sample/)下载完整示例。以下是调整的内容：

1.	镜像问题解决方案

	基于 CloudServiceConfiguration 从云服务创建 Windows Batch Pool 代码实现：

		new_pool = batch.models.PoolAddParameter(
		        id=pool_id,
		        cloud_service_configuration=batchmodels.CloudServiceConfiguration(
		            os_family="4",
		            target_os_version="*"),
		        vm_size=_POOL_VM_SIZE,
		        target_dedicated=_POOL_NODE_COUNT,
		        start_task=batch.models.StartTask(
		            command_line=common.helpers.wrap_commands_in_shell('windows', task_commands),
		            run_elevated=True,
		            wait_for_success=True,
		            resource_files=resource_files),
		    )

2.	环境问题解决方案

	提供国内访问资源及镜像：
	
		$nl = [Environment]::NewLine
		Write-Output "Download python to install...$nl"
		
		$url = "https://devstorage.blob.core.chinacloudapi.cn/files/python-3.5.2-amd64.exe"
		$outFile = "${env:TEMP}\python-3.5.2-amd64.exe"
		Write-Output "Downloading $url to $outFile$nl"
		Invoke-WebRequest $url -OutFile $outFile
		
		Write-Output "Installing$nl"
		Start-Process "$outFile" -ArgumentList "/quiet", "InstallAllUsers=1" -Wait
		
		Write-Output "Update pip and add dependency"
		py -m pip install -U pip -i http://mirrors.aliyun.com/pypi/simple/ --trusted-host mirrors.aliyun.com 
		py -m pip install cryptography -i http://mirrors.aliyun.com/pypi/simple/ --trusted-host mirrors.aliyun.com
		py -m pip install azure-batch -i http://mirrors.aliyun.com/pypi/simple/ --trusted-host mirrors.aliyun.com
		py -m pip install azure-storage -i http://mirrors.aliyun.com/pypi/simple/ --trusted-host mirrors.aliyun.com
		
		Write-Output "Done$nl" 

## 运行测试：

	Sample start: 2016-12-14 05:34:52
	Uploading file C:\Users\kevin\Desktop\george-python-test\azure-batch-samples-mas ter\Python\Batch\article_samples\python_tutorial_task.py to container [applicati on]... Uploading file C:\Users\kevin\Desktop\george-python-test\azure-batch-samples-mas ter\Python\Batch\article_samples\PrepPython35.ps1 to container [application]... Uploading file C:\Users\kevin\Desktop\george-python-test\azure-batch-samples-mas ter\Python\Batch\article_samples\data\taskdata1.txt to container [input]... Uploading file C:\Users\kevin\Desktop\george-python-test\azure-batch-samples-mas ter\Python\Batch\article_samples\data\taskdata2.txt to container [input]... Uploading file C:\Users\kevin\Desktop\george-python-test\azure-batch-samples-mas ter\Python\Batch\article_samples\data\taskdata3.txt to container [input]... Creating pool [PythonTutorialPool01]... Creating job [PythonTutorialJob01]... Adding 3 tasks to job [PythonTutorialJob01]... Monitoring all tasks for 'Completed' state, timeout in 0:30:00.................. ................................................................................ ................................................................................ ................................................................................ ................................................................................ .................................................................... Success! All tasks reached the 'Completed' state within the specified timeout period. C:\Users\kevin Downloading all files from container [output]... Downloaded blob [taskdata1_OUTPUT.txt] from container [output] to C:\Users\kev in\taskdata1_OUTPUT.txt Downloaded blob [taskdata2_OUTPUT.txt] from container [output] to C:\Users\kev in\taskdata2_OUTPUT.txt Downloaded blob [taskdata3_OUTPUT.txt] from container [output] to C:\Users\kev in\taskdata3_OUTPUT.txt Download complete! Deleting containers...
	Sample end: 2016-12-14 05:42:20 Elapsed time: 0:07:28
	Delete job? [Y/n]
	________________________________________
	Word Count
	to: 18 and: 17 you: 14
	Node: tvm-884213370_1-20161214t053541z Task: topNtask0 Job: PythonTutorialJob01 Pool: PythonTutorialPool01
