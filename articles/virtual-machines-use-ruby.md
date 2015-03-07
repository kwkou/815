<properties linkid="manage-services-identity-multi-factor-authentication" urlDisplayName="What is Azure Multi-Factor Authentication?" pageTitle="什么是 Azure 多因素身份验证？" metaKeywords="" description="更多地了解 Azure 多因素身份验证，需要使用多个验证方法，并且向用户登录和事务添加关键的第二个安全层的身份验证的方法。" metaCanonical="" services="active-directory,multi-factor-authentication" documentationCenter="" title="How to Manage Azure Virtual Machines using Ruby" authors="larryfr" solutions="" manager="" editor="" />



#如何使用 Ruby 管理 Azure 虚拟机

本指南将演示如何以编程方式执行常见管理任务对 Azure 虚拟机，如创建和配置虚拟机和添加数据磁盘。Azure SDK for Ruby 提供对各种 Azure 服务，包括 Azure 虚拟机的服务管理功能的访问。

##目录

* [什么是服务管理？](#what-is)
* [概念](#concepts)
* [创建管理证书](#setup-certificate)
* [创建 Ruby 应用程序](#create-app)
* [应用程序配置为使用 SDK](#configure-access)
* [设置 Azure 管理连接](#setup-connection)
* [How To 文章：使用虚拟机](#virtual-machine)
* [How To 文章：使用映像和磁盘](#vm-images)
* [How To 文章：使用云服务](#cloud-services)
* [How To 文章：使用存储服务](#storage-services)
* [接下来的步骤](#next-steps)

## <a name="what-is"> </a>什么是服务管理？

Azure 提供了[服务管理操作的 REST Api](http://msdn.microsoft.com/zh-cn/library/azure/ee460799.aspx)，包括管理的 Azure 虚拟机。Azure SDK for ruby 公开为通过虚拟机的管理操作**azure:: virtualmachineserivce**类。大部分虚拟机管理功能可通过[Azure 管理门户](https://manage.windowsazure.cn)可使用此类访问。

尽管可以使用服务管理 API 来管理各种服务在 Azure 上托管，本文档仅对 Azure 虚拟机的管理提供详细信息。

## <a name="concepts"> </a>概念

Azure 虚拟机作为云服务中的角色实现。每个云服务可以包含一个或多个角色，在逻辑上组合到部署。该角色定义虚拟机，例如多少内存不可用的总体物理特性多少 CPU 内核数，等等。

每个虚拟机还具有操作系统磁盘，它包含可引导的操作系统。虚拟机可具有一个或多个数据磁盘，是应该用于存储应用程序数据的其他磁盘。磁盘是作为在 Azure Blob 存储中存储的虚拟硬盘 (VHD) 实现的。也可以作为映像，是用于创建虚拟机创建过程中，虚拟机所用的磁盘的模板提供 Vhd。例如，创建新的虚拟机使用 Ubuntu 映像将导致从 Ubuntu 映像中创建新的操作系统磁盘。

大多数映像都是提供由 Microsoft 或合作伙伴，但是您可以创建您自己的映像或从在 Azure 中托管的虚拟机创建映像。

## <a name="setup-certificate"> </a>创建 Azure 管理证书

在执行服务管理操作，如那些通过公开**azure:: virtualmachineservice**类，则必须提供您的 Azure 订阅 ID 和包含您的订阅的管理证书的文件。同时使用 SDK 时对 Azure REST API 进行身份验证。

你可以通过使用 Azure 跨平台命令行界面 (xplat-cli) 获取订阅 ID 和管理证书。请参阅[安装和配置 Azure 跨平台命令行界面](/zh-cn/documentation/articles/xplat-cli/) 有关安装和配置 xplat-cli 的信息。

一旦配置了 xplat-cli 后，您可以执行以下步骤以检索您的 Azure 订阅 ID 和导出管理证书：

1. 若要检索的订阅 ID，请使用：

		azure account list

2. 若要导出的管理证书，请使用以下命令：

		azure account cert export

	在命令执行完毕后，该证书将导出到名为文件 & l t; azure 订阅名称 & g t;。pem。例如，如果您的订阅名为**mygreatsubscription**，创建的文件将被命名为**mygreatsubscription.pem**。

请注意的订阅 ID 以及包含导出的证书的 PEM 文件的位置，因为这些将在本文档后面使用。

## <a name="create-app"></a>创建 Ruby 应用程序

创建新的 Ruby 应用程序。使用本文档中的示例可以实现在单个**.rb**文件。

## <a name="configure-access"></a>配置您的应用程序

若要管理 Azure 服务，您需要下载和使用包含 Azure SDK for Ruby 的 Azure gem。

### 使用 gem 命令安装程序包

1.  使用命令行界面 （如**PowerShell** (Windows)、**终端**(Mac) 或**Bash** (UNIX)，导航到你在其中创建了示例应用程序的文件夹。

2. 使用以下命令安装 azure gem：

		gem install azure

	您应该会看到与下面类似的输出：

		Fetching: mini_portile-0.5.1.gem (100%)
		Fetching: nokogiri-1.6.0-x86-mingw32.gem (100%)
		Fetching: mime-types-1.25.gem (100%)
		Fetching: systemu-2.5.2.gem (100%)
		Fetching: macaddr-1.6.1.gem (100%)
		Fetching: uuid-2.3.7.gem (100%)
		Fetching: azure-0.5.0.gem (100%)
		Successfully installed mini_portile-0.5.1
		Successfully installed nokogiri-1.6.0-x86-mingw32
		Successfully installed mime-types-1.25
		Successfully installed systemu-2.5.2
		Successfully installed macaddr-1.6.1
		Successfully installed uuid-2.3.7
		Successfully installed azure-0.5.0
		7 gems installed

	<div class="dev-callout">
	<b>说明</b>
	<p>如果您收到权限相关的错误，则使用 <code>sudo gem 安装 azure</code> 。</p>
	</div>

### 要求 gem

使用文本编辑器将以下内容添加到你的 Ruby 应用程序文件的顶部。这将加载 azure gem 并且使 Azure SDK for Ruby 可供你的应用程序使用：

	require 'azure'

## <a name="setup-connection"> </a>如何：连接到服务管理

若要成功执行与 Azure 服务管理操作，必须指定订阅 ID 和证书中获取[创建 Azure 管理证书](#setup-certificate) 节中添加以下代码。若要这样做的最简单方法是指定的 ID 和证书文件使用以下环境变量路径：

* AZURE\_MANAGEMENT\_CERTIFICATE-的路径。包含管理证书的 PEM 文件。

* AZURE\_SUBSCRIPTION\_ID-你的 Azure 订阅的订阅 ID。

你还可以使用以下命令在你的应用程序中以编程方式设置这些值：

	Azure.configure do |config|
	  config.management_certificate = 'path/to/certificate'
	  config.subscription_id = 'subscription ID'
	end

##<a name="virtual-machine"> </a>如何：使用虚拟机

使用执行管理操作对 Azure 虚拟机**azure:: virtualmachineservice**类。

###如何：新建虚拟机

若要创建新的虚拟机，请使用**create\_virtual\_machine**方法。该方法将接受一个包含以下参数和返回哈希**Azure::VirtualMachineManagement::VirtualMachine**实例，它描述已创建的虚拟机：

**参数**

* **： vm\_name** -虚拟机的名称

* **： vm\_user** -根或管理用户名

* **： 密码**-用于根或管理用户的密码

* **： 图像**-将用于创建操作系统磁盘，为此虚拟机的操作系统映像。该操作系统磁盘将存储于在 blob 存储中创建的 VHD。

* **： 位置**-将在其中创建虚拟机的区域。该区域应与包含此虚拟机使用的磁盘的存储帐户所在的区域相同。

下面是创建新的虚拟机使用这些参数的示例：

	vm_params = {
	  :vm_name => 'mygreatvm',
	  :vm_user => 'myuser',
	  :password => 'mypassword',
	  :image => 'b39f27a8b8c64d52b05eac6a62ebad85__Ubuntu-13_04-amd64-server-20130824-zh-cn-30GB',
	  :location = 'East China'
	}

	vm_mgr = Azure::VirtualMachineService.new
	vm = vm_mgr.create_virtual_machine(vm_params)

	puts "A VM named #{vm.vm_name} was created in a cloud service named #{vm.cloud_service_name}."
	puts "It uses a disk named #{vm.disk_name}, which was created from a #{vm.os_type}-based image."
	puts "The virtual IP address of the machine is #{vm.ipaddress}."
    puts "The fully qualified domain name is #{vm.cloud_service_name}.chinacloudapp.cn."

**选项**

您可以提供允许您重写默认行为的虚拟机创建，例如，指定现有存储帐户而不是创建一个新的可选参数的哈希值。

以下是在使用时的选项**create\_virtual\_machine**方法：

* **： storage\_account\_name** -要用于存储磁盘映像的存储帐户的名称。如果不存在的存储帐户，创建一个新的键。如果省略，新的存储帐户使用创建基于的名称： vm\_name param。

* **： cloud\_service\_name** -要用于承载虚拟机的云服务的名称。如果云服务尚不存在，创建一个新的键。如果省略，新的云服务帐户使用创建基于的名称： vm\_name param。

* **： deployment\_name** -部署来部署虚拟机配置时使用的名称

* **： tcp\_endpoints** -的 TCP 端口，以公开为此虚拟机。SSH 终结点 （对于基于 Linux 的虚拟机） 和 WinRM 终结点 （对于基于 Windows 的虚拟机） 不需要指定数据类型，并将自动创建。可以指定多个端口，用逗号分隔。若要将内部端口与使用不同的端口号的公共端口相关联，请使用格式**公共端口: 内部端口**。例如，80: 8080 作为公共端口 80 公开内部端口 8080。

* **： service\_location** -在 VM 上的目标证书存储区位置。仅适用于基于 Windows 的虚拟机。

* **： ssh\_private\_key\_file** -包含私钥，将用于保护对基于 Linux 的虚拟机的 SSH 访问的文件。此外可用于指定用于保护 WinRM，如果选择了 HTTPS 传输的证书。如果**： ssh\_private\_key\_file**和**： ssh\_certificate\_file**被省略，SSH 将只使用密码身份验证

* **： ssh\_certificate\_file** -包含证书文件，将用于保护对基于 Linux 的虚拟机的 SSH 访问的文件。此外可用于指定用于保护 WinRM，如果选择了 HTTPS 传输的证书。如果**： ssh\_private\_key\_file**和**： ssh\_certificate\_file**被省略，SSH 将只使用密码身份验证

* **： ssh\_port** -将用于 SSH 通信的公共端口。如果省略，SSH 端口默认为 22。

* **： vm\_size** -的 VM 的大小。这将确定内存大小、内核数目、带宽以及虚拟机的其他物理特性。请参阅[用于 Azure 的虚拟机和云服务大小](http://msdn.microsoft.com/zh-cn/library/azure/dn197896.aspx)有关可用大小和物理特征。

* **: winrm_transport** -一个可用于 WinRM 的传输的数组。有效传输为 http 和 https。如果作为传输协议指定 https，则还必须使用**： ssh\_private\_key\_file**和**： ssh\_certificate\_file**来指定用于保护 HTTPS 通信的证书。

下面是创建新的虚拟机，它使用小计算实例、 公开公开为 HTTP （本地端口 8080、 公共端口 80） 和 https (443) 流量的端口并启用针对 SSH 会话使用指定的证书文件的证书身份验证的示例：

	vm_params = {
	  :vm_name => 'myvm',
	  :vm_user => 'myuser',
	  :password => 'mypassword',
	  :image => 'b39f27a8b8c64d52b05eac6a62ebad85__Ubuntu-13_04-amd64-server-20130824-zh-cn-30GB',
	  :location = 'East China'
	}

	vm_opts = {
      :tcp_endpoints => '80:8080,443',
      :vm_size => 'Small',
      :ssh_private_key_file => '../sshkey/mykey.key',
      :ssh_certificate_file => '../sshkey/mykey.pem'
	}

	vm_mgr = Azure::VirtualMachineService.new
	vm = vm_mgr.create_virtual_machine(vm_params, vm_opts)

###如何：列出虚拟机

若要列出的您的 Azure 订阅的现有虚拟机，请使用**list\_virtual\_machines**方法。此方法返回的数组**Azure::VirtualMachineManagement::VirtualMachine**对象：

	vm_mgr = Azure::VirtualMachineService.new
	virtual_machines = vm_mgr.list_virtual_machines

###如何：获取有关虚拟机的信息

若要获取其实例**Azure::VirtualMachineManagement::VirtualMachine**对于特定的虚拟机，使用**get\_virtual\_machine**方法并且提供虚拟机和云服务名称：

	vm_mgr = Azure::VirtualMachineService.new
	vm = vm_mgr.get_virtual_machine('myvm', 'mycloudservice')

###如何：删除虚拟机

若要删除虚拟机时，使用**delete\_virtual\_machine**方法并且提供虚拟机和云服务名称：

	vm_mgr = Azure::VirtualMachineService.new
	vm = vm_mgr.delete_virtual_machine('myvm', 'mycloudservice')

<div class="dev-callout">
<b>警告</b>
<p>将显示 <b>delete_virtual_machine</b> 方法删除云服务和与虚拟机关联的任何磁盘。</p>
</div>

###如何：关闭虚拟机

若要关闭虚拟机，使用**shutdown\_virtual\_machine**方法并且提供虚拟机和云服务名称：

	vm_mgr = Azure::VirtualMachineService.new
	vm = vm_mgr.shutdown_virtual_machine('myvm', 'mycloudservice')

###如何：启动虚拟机

若要启动虚拟机，请使用**start\_virtual\_machine**方法并且提供虚拟机和云服务名称：

	vm_mgr = Azure::VirtualMachineService.new
	vm = vm_mgr.start_virtual_machine('myvm', 'mycloudservice')

##<a name="vm-images"> </a>如何：使用虚拟机映像和磁盘

使用执行对虚拟机映像的操作**azure:: virtualmachineimageservice**类。使用执行在磁盘上的操作**:: virtualmachinediskmanagementservice**类。

###如何：列出虚拟机映像

若要列出可用的虚拟机映像，请使用**list\_virtual\_machine\_images**方法。这将返回的数组**azure:: virtualmachineimageservice**对象。

	image_mgr = Azure::VirtualMachineImageService.new
	images = image_mgr.list_virtual_machine_images

###如何：列出磁盘

你的 Azure 订阅，使用的磁盘**list\_virtual\_machine\_disks**方法。这将返回的数组**Azure::VirtualMachineImageManagement::VirtualMachineDisk**对象。

	disk_mgr = Azure::VirtualMachineImageManagement::VirtualMachineDiskManagementService.new
	disks = disk_mgr.list_virtual_machine_disks

###如何：删除磁盘

若要删除磁盘，请使用**delete\_virtual\_machine\_disk**方法，并指定要删除的磁盘的名称：

	disk_mgr = Azure::VirtualMachineImageManagement::VirtualMachineDiskManagementService.new
	disk_mgr.delete_virtual_machine_disk

##<a name="cloud-services"> </a>如何：使用云服务

使用执行针对 Azure 云服务的管理操作**azure:: cloudservice**类。

###如何：创建云服务

若要创建新的云服务，请使用**create\_cloud\_service**方法，并提供一个名称和选项哈希。有效选项为：

* **： 位置**- *Required*。将在其中创建云服务的区域。

* **： 描述**-云服务的说明。

以下语句创建新的云服务在美国东部区域：

	cs_mgr = Azure::CloudService.new
	cs_mgr.create_cloud_service('mycloudservice', { :location => 'East China' })

###如何：列出云服务

若要列出你的 Azure 订阅的云服务，请使用**list\_cloud\_services**方法。此方法返回的数组**Azure::CloudServiceManagement::CloudService**对象：

	cs_mgr = Azure::CloudService.new
	cloud_services = cs_mgr.list_cloud_services

###如何：查看某一云服务是否已存在

若要检查是否已存在特定的云服务，请使用**get\_cloud\_service**方法，并提供云服务的名称。返回**true**如果云服务指定名称的存在） ；否则为**false**。

	cs_mgr = Azure::CloudService.new
	cs_exists = cs_mgr.get_cloud_service('mycloudservice')

###如何：删除云服务

若要删除云服务，请使用**delete\_cloud\_service**方法，并提供云服务的名称：

	cs_mgr = Azure::CloudService.new
	cs_mgr.delete_cloud_service('mycloudservice')

###如何：删除部署

若要删除某一云服务部署，使用**delete\_cloud\_service\_deployment**方法，并提供云服务名称：

	cs_mgr = Azure::CloudService.new
	cs_mgr.delete_cloud_service_deployment('mycloudservice')

##<a name="storage-services"> </a>如何：使用存储服务

使用执行针对 Azure 云服务的管理操作**azure:: storageservice**类。

###如何：创建存储帐户

若要创建新的存储帐户，请使用**create\_storage\_account**方法，并提供一个名称和选项哈希。有效选项为：

* **： 位置**- *Required*。将在其中创建存储帐户的区域。

* **： 描述**-存储帐户的说明。

以下语句创建新的存储帐户中国东部区域中：

	storage_mgr = Azure::StorageService.new
	storage_mgr.create_storage_account('mystorage', { :location => 'East China' })

###如何：列出存储帐户

若要获取你的 Azure 订阅的存储帐户的列表，请使用**list\_storage\_accounts**方法。此方法返回的数组**Azure::StorageManagement::StorageAccount**对象。

	storage_mgr = Azure::StorageService.new
	accounts = storage_mgr.list_storage_accounts

###如何：查看某一存储帐户是否存在

若要查看特定的存储帐户是否存在，请使用**get\_storage\_account**方法并指定存储帐户的名称。返回**true**如果存在） 的存储帐户 ；否则为**false**。

	storage_mgr = Azure::StorageService.new
	store_exists = storage_mgr.get_storage_account('mystorage')

###如何：删除存储帐户

若要删除存储帐户，请使用**delete\_storage\_account**方法并指定存储帐户的名称：

	storage_mgr = Azure::StorageService.new
	storage_mgr.delete_storage_account('mystorage')

##<a name="next-steps"> </a>后续步骤

既然您已了解以编程方式创建 Azure 虚拟机的基础知识，请按照下面的链接可了解如何执行更多有关使用虚拟机。

* 请访问[虚拟机](/zh-cn/documentation/services/virtual-machines/) 功能页
*  请参阅 MSDN 参考：[虚拟机](http://msdn.microsoft.com/zh-cn/library/azure/jj156003.aspx)
* 了解如何托管[Ruby on Rails 应用程序在虚拟机上](/zh-cn/documentation/articles/virtual-machines-ruby-rails-web-app-linux/)
