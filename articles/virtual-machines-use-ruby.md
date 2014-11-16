<properties linkid="manage-services-identity-multi-factor-authentication" urlDisplayName="What is Azure Multi-Factor Authentication?" pageTitle="What is Azure Multi-Factor Authentication?" metaKeywords="" description="Learn more about Azure Multi-Factor Authentication, a method of authentication that requires the use of more than one verification method and adds a critical second layer of security to user sign-ins and transactions." metaCanonical="" services="active-directory,multi-factor-authentication" documentationCenter="" title="How to Manage Azure Virtual Machines using Ruby" authors="larryfr" solutions="" manager="" editor="" />

# 如何使用 Ruby 管理 Azure 虚拟机

本指南将演示如何以编程方式执行针对 Azure 虚拟机的常见管理任务，例如创建和配置虚拟机以及添加数据磁盘。Azure SDK for Ruby 提供对多种 Azure 服务（包括 Azure 虚拟机）的服务管理功能的访问。

## 目录

-   [什么是服务管理？][什么是服务管理？]
-   [概念][概念]
-   [创建管理证书][创建管理证书]
-   [创建 Ruby 应用程序][创建 Ruby 应用程序]
-   [配置你的应用程序以便使用 SDK][配置你的应用程序以便使用 SDK]
-   [设置 Azure 管理连接][设置 Azure 管理连接]
-   [如何：使用虚拟机][如何：使用虚拟机]
-   [如何：使用映像和磁盘][如何：使用映像和磁盘]
-   [如何：使用云服务][如何：使用云服务]
-   [如何：使用存储服务][如何：使用存储服务]
-   [后续步骤][后续步骤]

## <a name="what-is"> </a>什么是服务管理？

[Azure 提供针对服务管理操作的 REST API][Azure 提供针对服务管理操作的 REST API]，包括对 Azure 虚拟机的管理。Azure SDK for ruby 通过 **Azure::VirtualMachineSerivce** 类公开针对虚拟机的管理操作。使用该类可以访问通过 [Azure 管理门户][Azure 管理门户]提供的许多虚拟机管理功能。

尽管可以使用服务管理 API 管理在 Azure 上托管的多种服务，但本文只详细介绍了针对 Azure 虚拟机的管理。

## <a name="concepts"> </a>概念

Azure 虚拟机是作为云服务内的“角色”实现的。每个云服务都可以包含一个或多个角色，这些角色在逻辑上组合到部署中。该角色定义虚拟机的总体物理特性，例如多少内存可用或者有多少 CPU 内核等。

每个虚拟机还都具有一个操作系统磁盘，该磁盘包含可引导操作系统。一个虚拟机可具有一个或多个数据磁盘，这些磁盘是应该用于存储应用程序数据的附加磁盘。磁盘是作为在 Azure Blob 存储中存储的虚拟硬盘 (VHD) 实现的。VHD 还可作为“映像”公开，这是在虚拟机创建过程中由该虚拟机用来创建磁盘的模板。例如，使用 Ubuntu 映像创建一个新虚拟机，就会使用 Ubuntu 映像创建一个新的操作系统磁盘。

大多数映像都是由 Microsoft 或合作伙伴提供的，但你可以创建自己的映像或从 Azure 中托管的虚拟机创建映像。

## <a name="setup-certificate"> </a>创建 Azure 管理证书

在执行服务管理操作（例如通过 **Azure::VirtualMachineService** 类公开的操作）时，必须提供你的 Azure 订阅 ID 以及包含你的订阅的管理证书的文件。在向 Azure REST API 进行身份验证时，SDK 将使用它们。

你可以通过使用 Azure 跨平台命令行界面 (xplat-cli) 获取订阅 ID 和管理证书。有关安装和配置 xplat-cli 的信息，请参阅[安装和配置 Azure 跨平台命令行界面][安装和配置 Azure 跨平台命令行界面]。

在配置了 xplat-cli 后，你可以执行以下步骤以便检索 Azure 订阅 ID 和导出管理证书：

1.  若要检索订阅 ID，请使用：

        azure account list

2.  若要导出管理证书，请使用以下命令：

        azure account cert export

    在命令执行完毕后，该证书将导出到名为 \<azure-subscription-name\> 的文件中。例如，如果你的订阅名为 **mygreatsubscription**，则创建的文件将名为 **mygreatsubscription.pem**。

请记下该订阅 ID 以及包含导出的证书的 PEM 文件的位置，因为在本文档的后面将使用它们。

## <a name="create-app"></a>创建 Ruby 应用程序

创建新的 Ruby 应用程序。可在单个 **.rb** 文件中实现在本文档中使用的示例。

## <a name="configure-access"></a>配置你的应用程序

若要管理 Azure 服务，你需要下载并使用包含 Azure SDK for Ruby 的 Azure gem。

### 使用 gem 命令安装程序包

1.  使用 **PowerShell** (Windows)、**Terminal** (Mac) 或 **Bash** (UNIX) 等命令行界面导航到你在其中创建了示例应用程序的文件夹。

2.  使用以下命令安装 azure gem：

        gem install azure

    你将看到与下面类似的输出：

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
<p>如果你收到与权限相关的错误，应改用<code>sudo gem install azure</code> 。</p>
</div>

### 要求 gem

使用文本编辑器将以下内容添加到你的 Ruby 应用程序文件的顶部。这将加载 azure gem 并且使 Azure SDK for Ruby 可供你的应用程序使用：

    require 'azure'

## <a name="setup-connection"> </a>如何：连接到服务管理

若要使用 Azure 成功执行服务管理操作，你必须指定在[创建 Azure 管理证书][创建管理证书]一节中获取的订阅 ID 和证书。执行此任务的最简单方式是使用以下环境变量指定该 ID 以及指向证书文件的路径：

-   AZURE\_MANAGEMENT\_CERTIFICATE - 指向包含管理证书的 .PEM 文件的路径。

-   AZURE\_SUBSCRIPTION\_ID - 你的 Azure 订阅的订阅 ID。

你还可以使用以下命令在你的应用程序中以编程方式设置这些值：

    Azure.configure do |config|
      config.management_certificate = 'path/to/certificate'
      config.subscription_id = 'subscription ID'
    end

## <a name="virtual-machine"> </a>如何：使用虚拟机

使用 **Azure::VirtualMachineService** 类执行针对 Azure 虚拟机的管理操作。

### 如何：新建虚拟机

若要创建新虚拟机，请使用 **create\_virtual\_machine** 方法。该方法将接受包含以下参数的哈希，并且返回描述已创建的虚拟机的 **Azure::VirtualMachineManagement::VirtualMachine** 实例：

**参数**

-   **:vm\_name** - 虚拟机的名称

-   **:vm\_user** - 根或管理用户名

-   **:password** - 用于根或管理用户的密码

-   **:image** - 将用于为此虚拟机创建操作系统磁盘的操作系统映像。该操作系统磁盘将存储于在 blob 存储中创建的 VHD。

-   **:location** - 将在其中创建虚拟机的区域。该区域应与包含此虚拟机使用的磁盘的存储帐户所在的区域相同。

以下是使用这些参数创建新虚拟机的示例：

    vm_params = {
      :vm_name => 'mygreatvm',
      :vm_user => 'myuser',
      :password => 'mypassword',
      :image => 'b39f27a8b8c64d52b05eac6a62ebad85__Ubuntu-13_04-amd64-server-20130824-en-us-30GB',
      :location = 'East China'
    }

    vm_mgr = Azure::VirtualMachineService.new
    vm = vm_mgr.create_virtual_machine(vm_params)

    puts "A VM named #{vm.vm_name} was created in a cloud service named #{vm.cloud_service_name}."
    puts "It uses a disk named #{vm.disk_name}, which was created from a #{vm.os_type}-based image."
    puts "The virtual IP address of the machine is #{vm.ipaddress}."
    puts "The fully qualified domain name is #{vm.cloud_service_name}.chinacloudapp.cn."

**选项**

你可以提供可选参数的哈希，从而允许你覆盖虚拟机创建的默认行为，例如指定现有存储帐户，而不是创建新帐户。

下面是在使用 **create\_virtual\_machine** 方法时可用的选项：

-   **:storage\_account\_name** - 要用于存储磁盘映像的存储帐户的名称。如果该存储帐户已不存在，将创建一个新帐户。如果省略，将基于 :vm\_name 参数使用某一名称创建一个新的存储帐户。

-   **:cloud\_service\_name** - 要用于托管虚拟机的云服务的名称。如果该云服务已不存在，将创建一个新的云服务。如果省略，将基于 :vm\_name 参数使用某一名称创建一个新的云服务帐户。

-   **:deployment\_name** - 在部署虚拟机配置时要使用的部署的名称

-   **:tcp\_endpoints** - 要为此虚拟机公开的 TCP 端口。无需指定 SSH 终结点（对于基于 Linux 的虚拟机）和 WinRM 终结点（对于基于 Windows 的虚拟机），将自动指定它们。可以指定多个端口，用逗号分隔这些端口。若要使用不同端口号将内部端口与公共端口相关联，请使用格式“公共端口:内部端口”。例如，80:8080 将此内部端口 8080 作为公共端口 80 公开。

-   **:service\_location** - 虚拟机上的目标证书存储位置。仅适用于基于 Windows 的虚拟机。

-   **:ssh\_private\_key\_file** - 包含私钥的文件，该私钥将用于保护对基于 Linux 的虚拟机的 SSH 访问。它还用来指定用于保护 WinRM 的证书（如果选择了 HTTPS 传输）。如果省略 **:ssh\_private\_key\_file** 和 **:ssh\_certificate\_file**，SSH 将只使用密码身份验证

-   **:ssh\_certificate\_file** - 包含证书文件的文件，该证书文件将用于保护对基于 Linux 的虚拟机的 SSH 访问。它还用来指定用于保护 WinRM 的证书（如果选择了 HTTPS 传输）。如果省略 **:ssh\_private\_key\_file** 和 **:ssh\_certificate\_file**，SSH 将只使用密码身份验证

-   **:ssh\_port** - 将用于 SSH 通信的公共端口。如果省略，SSH 端口将默认为 22。

-   **:vm\_size** - 虚拟机的大小。这将确定内存大小、内核数目、带宽以及虚拟机的其他物理特性。有关可用大小和物理特性的信息，请参阅 [Azure 的虚拟机和云服务大小][Azure 的虚拟机和云服务大小]。

-   **:winrm\_transport** - 可用于 WinRM 的传输的数组。有效传输为“http”和“https”。如果将“https”指定为某一传输，你还必须使用 **:ssh\_private\_key\_file** 和 **:ssh\_certificate\_file** 指定要用于保护 HTTPS 通信安全的证书。

下面是一个创建新虚拟机的示例，该虚拟机使用小计算实例、公开针对 HTTP（本地端口 8080、公共端口 80）和 HTTPS(443) 流量的端口并且使用指定的证书文件启用针对 SSH 会话的证书身份验证：

    vm_params = {
      :vm_name => 'myvm',
      :vm_user => 'myuser',
      :password => 'mypassword',
      :image => 'b39f27a8b8c64d52b05eac6a62ebad85__Ubuntu-13_04-amd64-server-20130824-en-us-30GB',
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

### 如何：列出虚拟机

若要列出针对你的 Azure 订阅的现有虚拟机，请使用 **list\_virtual\_machines** 方法。此方法返回 **Azure::VirtualMachineManagement::VirtualMachine** 对象的数组：

    vm_mgr = Azure::VirtualMachineService.new
    virtual_machines = vm_mgr.list_virtual_machines

### 如何：获取有关虚拟机的信息

若要获取特定虚拟机的 **Azure::VirtualMachineManagement::VirtualMachine** 的实例，请使用 **get\_virtual\_machine** 方法并且提供虚拟机和云服务名称：

    vm_mgr = Azure::VirtualMachineService.new
    vm = vm_mgr.get_virtual_machine('myvm', 'mycloudservice')

### 如何：删除虚拟机

若要删除某一虚拟机，请使用 **delete\_virtual\_machine** 方法并且提供虚拟机和云服务名称：

    vm_mgr = Azure::VirtualMachineService.new
    vm = vm_mgr.delete_virtual_machine('myvm', 'mycloudservice')

<div class="dev-callout">
<b>警告</b>
<p><b>delete_virtual_machine</b> 方法将删除与虚拟机相关联的云服务和所有磁盘。</p>
</div>

### 如何：关闭虚拟机

若要关闭某一虚拟机，请使用 **shutdown\_virtual\_machine** 方法并且提供虚拟机和云服务名称：

    vm_mgr = Azure::VirtualMachineService.new
    vm = vm_mgr.shutdown_virtual_machine('myvm', 'mycloudservice')

### 如何：启动虚拟机

若要启动某一虚拟机，请使用 **start\_virtual\_machine** 方法并且提供虚拟机和云服务名称：

    vm_mgr = Azure::VirtualMachineService.new
    vm = vm_mgr.start_virtual_machine('myvm', 'mycloudservice')

## <a name="vm-images"> </a>如何：使用虚拟机映像和磁盘

使用 **Azure::VirtualMachineImageService** 类执行针对虚拟机映像的操作。使用 **Azure::VirtualMachineImageManagement::VirtualMachineDiskManagementService** 类执行针对磁盘的操作。

### 如何：列出虚拟机映像

若要列出可用虚拟机映像，请使用 **list\_virtual\_machine\_images** 方法。这将返回 **Azure::VirtualMachineImageService** 对象的数组。

    image_mgr = Azure::VirtualMachineImageService.new
    images = image_mgr.list_virtual_machine_images

### 如何：列出磁盘

若要列出你的 Azure 订阅的磁盘，请使用 **list\_virtual\_machine\_disks** 方法。这将返回 **Azure::VirtualMachineImageManagement::VirtualMachineDisk** 对象的数组。

    disk_mgr = Azure::VirtualMachineImageManagement::VirtualMachineDiskManagementService.new
    disks = disk_mgr.list_virtual_machine_disks

### 如何：删除磁盘

若要删除某一磁盘，请使用 **delete\_virtual\_machine\_disk** 方法并且指定要删除的磁盘的名称：

    disk_mgr = Azure::VirtualMachineImageManagement::VirtualMachineDiskManagementService.new
    disk_mgr.delete_virtual_machine_disk

## <a name="cloud-services"> </a>如何：使用云服务

使用 **Azure::CloudService** 类执行针对 Azure 云服务的管理操作。

### 如何：创建云服务

若要创建新的云服务，请使用 **create\_cloud\_service** 方法并且提供名称和选项哈希。有效选项为：

-   **:location** - *必填*。将在其中创建云服务的区域。

-   **:description** - 云服务的说明。

以下命令将在 East US 区域中创建一个新的云服务：

    cs_mgr = Azure::CloudService.new
    cs_mgr.create_cloud_service('mycloudservice', { :location => 'East China' })

### 如何：列出云服务

若要列出你的 Azure 订阅的云服务，请使用 **list\_cloud\_services** 方法。此方法返回 **Azure::CloudServiceManagement::CloudService** 对象的数组：

    cs_mgr = Azure::CloudService.new
    cloud_services = cs_mgr.list_cloud_services

### 如何：查看某一云服务是否已存在

若要查看某一特定的云服务是否已存在，请使用 **get\_cloud\_service** 方法并且提供该云服务的名称。如果指定名称的云服务存在，则返回 **true**；否则返回 **false**。

    cs_mgr = Azure::CloudService.new
    cs_exists = cs_mgr.get_cloud_service('mycloudservice')

### 如何：删除云服务

若要删除某一云服务，请使用 **delete\_cloud\_service** 方法并且提供该云服务的名称：

    cs_mgr = Azure::CloudService.new
    cs_mgr.delete_cloud_service('mycloudservice')

### 如何：删除部署

若要删除某一云服务的部署，请使用 **delete\_cloud\_service\_deployment** 方法并且提供该云服务的名称：

    cs_mgr = Azure::CloudService.new
    cs_mgr.delete_cloud_service_deployment('mycloudservice')

## <a name="storage-services"> </a>如何：使用存储服务

使用 **Azure::StorageService** 类执行针对 Azure 云服务的管理操作。

### 如何：创建存储帐户

若要创建新的存储帐户，请使用 **create\_storage\_account** 方法并且提供名称和选项哈希。有效选项为：

-   **:location** - *必填*。将在其中创建存储帐户的区域。

-   **:description** - 存储帐户的说明。

以下命令将在“East China”区域中创建一个新的存储帐户：

    storage_mgr = Azure::StorageService.new
    storage_mgr.create_storage_account('mystorage', { :location => 'East China' })

### 如何：列出存储帐户

若要获取针对你的 Azure 订阅的存储帐户的列表，请使用 **list\_storage\_accounts** 方法。此方法返回 **Azure::StorageManagement::StorageAccount** 对象的数组。

    storage_mgr = Azure::StorageService.new
    accounts = storage_mgr.list_storage_accounts

### 如何：查看某一存储帐户是否存在

若要查看某一特定存储帐户是否存在，请使用 **get\_storage\_account** 方法并且指定该存储帐户的名称。如果该存储帐户存在，则返回 **true**；否则返回 **false**。

    storage_mgr = Azure::StorageService.new
    store_exists = storage_mgr.get_storage_account('mystorage')

### 如何：删除存储帐户

若要删除某一存储帐户，请使用 **delete\_storage\_account** 方法并且指定该存储帐户的名称：

    storage_mgr = Azure::StorageService.new
    storage_mgr.delete_storage_account('mystorage')

## <a name="next-steps"> </a> 后续步骤

现在，你已了解以编程方式创建 Azure 虚拟机的基础知识，单击下面的链接可了解有关如何使用虚拟机的更多信息。

-   访问[虚拟机][虚拟机]功能页
-   查看 MSDN 参考：[虚拟机][1]
-   了解如何[在虚拟机上托管 Ruby on Rails 应用程序][在虚拟机上托管 Ruby on Rails 应用程序]

  [什么是服务管理？]: #what-is
  [概念]: #concepts
  [创建管理证书]: #setup-certificate
  [创建 Ruby 应用程序]: #create-app
  [配置你的应用程序以便使用 SDK]: #configure-access
  [设置 Azure 管理连接]: #setup-connection
  [如何：使用虚拟机]: #virtual-machine
  [如何：使用映像和磁盘]: #vm-images
  [如何：使用云服务]: #cloud-services
  [如何：使用存储服务]: #storage-services
  [后续步骤]: #next-steps
  [Azure 提供针对服务管理操作的 REST API]: http://msdn.microsoft.com/zh-cn/library/azure/ee460799.aspx
  [Azure 管理门户]: https://manage.windowsazure.cn
  [安装和配置 Azure 跨平台命令行界面]: http://windowsazure.cn/zh-cn/documentation/articles/xplat-cli/
  [Azure 的虚拟机和云服务大小]: http://msdn.microsoft.com/zh-cn/library/azure/dn197896.aspx
  [虚拟机]: http://windowsazure.cn/zh-cn/documentation/services/virtual-machines/
  [1]: http://msdn.microsoft.com/zh-cn/library/azure/jj156003.aspx
  [在虚拟机上托管 Ruby on Rails 应用程序]: http://windowsazure.cn/zh-cn/documentation/articles/virtual-machines-ruby-rails-web-app-linux/
