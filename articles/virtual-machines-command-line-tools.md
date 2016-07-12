<properties
	pageTitle="配合使用 Azure CLI 和服务管理 | Azure"
	description="了解如何使用适用于 Mac、Linux 和 Windows 的命令行工具，在经典（Azure 服务管理）模式下使用 Azure CLI 管理 Azure。"
	services="virtual-machines, mobile-services, cloud-services"
	documentationCenter=""
	authors="dlepow"
	manager="timlt"
	editor="tysonn"
	tags="azure-service-management"/>

<tags
	ms.service="multiple"
	ms.date="03/08/2016"
	wacn.date="05/23/2016"/>

# 将适用于 Mac、Linux 和 Windows 的 Azure CLI 与 Azure 服务管理配合使用

> [AZURE.IMPORTANT]Azure 具有用于创建和处理资源的两个不同的部署模型：[资源管理器和经典](/documentation/articles/resource-manager-deployment-model/)。本文介绍使用经典部署模型。Azure 建议大多数新部署使用[资源管理器模型](/documentation/articles/azure-cli-arm-commands/)。

本文介绍如何在服务管理模式（asm 模式）下使用 Azure CLI 在 Mac、Linux 和 Windows 计算机的命令行中创建、管理和删除服务。你可以使用 Azure SDK 的各种库、Azure PowerShell 和 Azure 门户执行许多相同的任务。在服务管理模式下使用 Azure 服务从概念上讲类似于创建和管理各个 Azure 概念和服务（如 Web 应用、虚拟机、虚拟网络、存储器等）。

> [AZURE.NOTE]
若要开始使用，首先[安装 Azure CLI](/documentation/articles/xplat-cli-install/)，并[登录以使用与你的帐户关联的 Azure 资源](/documentation/articles/xplat-cli-connect/)。

## 本文的讨论范围

本文提供了用于经典（服务管理）部署模型的常用 Azure CLI 命令的语法和选项。它并不是完整的参考，并且你的 CLI 版本可能会显示某些不同的命令或参数。要在服务管理模式下在命令行中查看当前的命令语法和选项，请键入 `azure help`；要显示某个命令的帮助，请键入 `azure help [command]`。你还可以在创建和管理具体 Azure 服务的说明文档中找到 CLI 示例。

可选参数显示在方括号中（例如，[参数]）。其他所有参数都是必需的。

除了此处记录的特定于命令的可选参数外，还有三个可用于显示详细输出（例如请求选项和状态代码）的可选参数。-v 参数提供详细输出，而 -vv 参数提供更详细的输出。--json 选项将以原始的 json 格式输出结果。

## 设置服务管理模式

当前，首次安装 CLI 时，在默认情况下启用服务管理模式。如果需要，请使用以下命令启用 Azure CLI 服务管理命令。

	azure config mode asm

>[AZURE.NOTE]Azure 资源管理器模式与 Azure 服务管理模式互斥。即在一种模式下创建的资源不能从另一种模式进行管理。

## 管理帐户信息和发布设置
该工具使用你的 Azure 订阅信息连接到你的帐户。可以从 Azure 门户中的发布设置文件中获取此信息，如下所述。可以导入发布设置文件作为永久性本地配置设置，该工具会将此设置用于后续操作。你只需导入你的发布设置一次。

**account download [options]**

此命令启动浏览器以从 Azure 门户下载你的 .publishsettings 文件。

	~$ azure account download
	info:   Executing command account download
	info:   Launching browser to https://windows.azure.com/download/publishprofile.aspx
	help:   Save the downloaded file, then execute the command
	help:   account import <file>
	info:   account download command OK

**account import [options] &lt;file>**


此命令导入 publishsettings 文件或证书以便日后可以供该工具使用。

	~$ azure account import publishsettings.publishsettings
	info:   Importing publish settings file publishsettings.publishsettings
	info:   Found subscription: 3-Month 1-RMB Trial
	info:   Found subscription: Pay-As-You-Go
	info:   Setting default subscription to: 3-Month 1-RMB Trial
	warn:   The 'publishsettings.publishsettings' file contains sensitive information.
	warn:   Remember to delete it now that it has been imported.
	info:   Account publish settings imported successfully

> [AZURE.NOTE]publishsettings 文件可以包含有关多个订阅的详细信息（即，订阅名称和 ID）。当你导入 publishsettings 文件时，第一个订阅将用作默认订阅。若要使用不同订阅，请运行以下命令。<code>~$ azure config set subscription &lt;other-subscription-id&gt;</code>

**account clear [options]**

此命令删除已导入的存储的发布设置。如果你在此计算机上使用完该工具，并且希望确保日后不能通过你的帐户使用该工具，则使用此命令。

	~$ azure account clear
	Clearing account info.
	info:   OK

**account list [options]**

列出导入的订阅

	~$ azure account list
	info:    Executing command account list
	data:    Name                                    Id
	       Current
	data:    --------------------------------------  -------------------------------
	-----  -------
	data:    Forums Subscription                     8679c8be-3b05-49d9-b8fb  true
	data:    Evangelism Team Subscription            9e672699-1055-41ae-9c36  false
	data:    MSOpenTech-Prod                         c13e6a92-706e-4cf5-94b6  false

**account set [options] &lt;subscription&gt;**

设置当前订阅

###管理地缘组的命令

**account affinity-group list [options]**

此命令列出你的 Azure 地缘组。

可以在一组虚拟机跨越多台物理计算机时设置地缘组。地缘组指定物理计算机应尽可能彼此接近，从而减少网络延迟。

	~$ azure account affinity-group list
	+ Fetching affinity groups
	data:   Name                                  Label   Location
	data:   ------------------------------------  ------  --------
	data:   535EBAED-BF8B-4B18-A2E9-8755FB9D733F  opentec  West US
	info:   account affinity-group list command OK

**account affinity-group create [options] &lt;name&gt;**

此命令创建新的地缘组

	~$ azure account affinity-group create opentec -l "China North"
	info:    Executing command account affinity-group create
	+ Creating affinity group
	info:    account affinity-group create command OK

**account affinity-group show [options] &lt;name&gt;**

此命令显示地缘组的详细信息

	~$ azure account affinity-group show opentec
	info:    Executing command account affinity-group show
	+ Getting affinity groups
	data:    $ xmlns "http://schemas.microsoft.com/windowsazure"
	data:    $ xmlns:i "http://www.w3.org/2001/XMLSchema-instance"
	data:    Name "opentec"
	data:    Label "b3BlbnRlYw=="
	data:    Description $ i:nil "true"
	data:    Location "China North"
	data:    HostedServices ""
	data:    StorageServices ""
	data:    Capabilities Capability 0 "PersistentVMRole"
	data:    Capabilities Capability 1 "HighMemory"
	info:    account affinity-group show command OK

**account affinity-group delete [options] &lt;name&gt;**

此命令将删除指定的地缘组

	~$ azure account affinity-group delete opentec
	info:    Executing command account affinity-group delete
	Delete affinity group opentec? [y/n] y
	+ Deleting affinity group
	info:    account affinity-group delete command OK

###管理帐户环境的命令

**account env list [options]**

帐户环境列表

	C:\windows\system32>azure account env list
	info:    Executing command account env list
	data:    Name
	data:    ---------------
	data:    AzureCloud
	data:    AzureChinaCloud
	info:    account env list command OK

**account env show [options] [environment]**

显示帐户环境详细信息

	~$ azure account env show
	info:    Executing command account env show
	Environment name: AzureCloud
	data:    Environment publishingProfile  http://go.microsoft.com/fwlink/?LinkId=2544
	data:    Environment portal  http://go.microsoft.com/fwlink/?LinkId=2544
	info:    account env show command OK

**account env add [options] [environment]**

此命令将一个环境添加到帐户

**account env set [options] [environment]**

此命令设置帐户环境

**account env delete [options] [environment]**

此命令从帐户中删除指定的环境

## 用于管理 Azure 虚拟机的命令
下图显示了如何在 Azure 云服务的生产部署环境中托管 Azure 虚拟机。

![Azure 技术图表](./media/virtual-machines-command-line-tools/architecturediagram.jpg)

**create-new** 在 Blob 存储中创建驱动器（即，图中的 e:\\）；**attach** 会将已创建但未附加的磁盘附加到虚拟机。

**vm create [options] &lt;dns-name> &lt;image> &lt;userName> [password]**

此命令创建新的 Azure 虚拟机。默认情况下，所创建的每台虚拟机 (vm) 都位于其自己的云服务中；但是，你可以使用此处提及的 -c 选项指定将虚拟机添加到现有云服务中。

vm create 命令与 Azure 门户一样，只会在生产部署环境中创建虚拟机。目前没有用于在云服务的过渡部署环境中创建虚拟机的选项。如果你的订阅没有现有的 Azure 存储帐户，此命令将创建一个。

你可以通过 --location 参数指定位置，也可以通过 --affinity-group 参数指定地缘组。如果这两个参数任何一个都没有提供，系统将提示你从有效位置列表中提供一个。

所提供密码的长度必须为 8-123 个字符，并且满足你要用于此虚拟机的操作系统的密码复杂度要求。

如果你预计需要使用 SSH 来管理部署的 Linux 虚拟机（通常都是如此），则必须在创建虚拟机时通过 -e 选项启用 SSH。在创建该虚拟机后无法启用 SSH。

Windows 虚拟机稍后可以通过添加端口 3389 作为终结点来启用 RDP。

此命令支持以下可选参数：

**-c, --connect** 在托管服务中已创建的部署中创建虚拟机。如果 -vmname 未与此选项一起使用，将自动生成新虚拟机的名称。<br /> 
**-n, --vm-name** 指定虚拟机的名称。默认情况下，此参数采用托管服务名称。如果未指定 -vmname，将生成 &lt;service-name>&lt;id> 形式的新虚拟机名称，其中 &lt;id> 是服务中现有虚拟机的数量加上 1。例如，如果你使用此命令向拥有一个现有虚拟机的托管服务 MyService 中添加新虚拟机，则会将新虚拟机命名为 MyService2。<br /> 
**-u, --blob-url** 指定从中创建虚拟机系统磁盘的目标 Blob 存储 URL。<br /> 
**-z, --vm-size** 指定虚拟机的大小。有效值为：
“ExtraSmall”、“Small”、“Medium”、“Large”、“ExtraLarge”、“A5”、“A6”、“A7”、“A8”、“A9”、“A10”、“A11”、“Basic\_A0”、“Basic\_A1”、“Basic\_A2”、  
“Basic\_A3”、“Basic\_A4”、“Standard\_D1”、“Standard\_D2”、“Standard\_D3”、“Standard\_D4”、“Standard\_D11”、“Standard\_D12”、“Standard\_D13”、  
“Standard\_D14”、“Standard\_DS1”、“Standard\_DS2”、“Standard\_DS3”、“Standard\_DS4”、“Standard\_DS11”、“Standard\_DS12”、“Standard\_DS13”、  
“Standard\_DS14”、“Standard\_G1”、“Standard\_G2”、“Standard\_G3”、“Standard\_G4”、“Standard_G55”。默认值为“Small”。<br /> 
**-r** 添加到 Windows 虚拟机的 RDP 连接。<br />
**-e, --ssh** 添加到 Windows 虚拟机的 SSH 连接。<br /> 
**-t, --ssh-cert** 指定 SSH 证书。<br /> 
**-s** 订阅。<br /> 
**-o, --community** 指定的映像是社区映像。<br /> 
**-w** 虚拟网络名称。<br/> 
**-l, --location** 指定位置（例如，“China North”）。<br /> 
**-a, --affinity-group** 指定地缘组。<br /> 
**-w, --virtual-network-name** 指定要在其中添加新虚拟机的虚拟网络。可从 Azure 门户设置和管理虚拟网络。<br /> 
**-b, --subnet-names** 指定要分配虚拟机的子网名称。

在此示例中，MSFT\__Win2K8R2SP1-120514-1520-141205-01-zh-CN-30GB 是该平台提供的映像。有关操作系统映像的详细信息，请参阅 VM 映像列表。

	~$ azure vm create my-vm-name MSFT__Windows-Server-2008-R2-SP1.11-29-2011 username --location "West US" -r
	info:   Executing command vm create
	Enter VM 'my-vm-name' password: ************
	info:   vm create command OK

**vm create-from &lt;dns-name> &lt;role-file>**

此命令从 JSON 角色文件创建新的 Azure 虚拟机。

	~$ azure vm create-from my-vm example.json
	info:   OK

**vm list [options]**

此命令列出 Azure 虚拟机。--json 选项指定以原始 JSON 格式返回结果。

	~$ azure vm list
	info:   Executing command vm list
	data:   DNS Name                          VM Name      Status
	data:   --------------------------------  -----------  ---------
	data:   my-vm-name.cloudapp-preview.net        my-vm        ReadyRole
	info:   vm list command OK

**vm location list [options]**

此命令列出所有可用的 Azure 帐户位置。

	~$ azure vm location list
	info:   Executing command vm location list
	data:   Name                   Display Name
	data:   ---------------------  ------------
	data:   Azure Preview  China North
	info:   account location list command OK

**vm show [options] &lt;name>**

此命令显示有关 Azure 虚拟机的详细信息。--json 选项指定以原始 JSON 格式返回结果。

	~$ azure vm show my-vm
	info:   Executing command vm show
	data:   {
	data:       InstanceSize: 'Small',
	data:       InstanceStatus: 'ReadyRole',
	data:       DataDisks: [],
	data:       IPAddress: '10.26.192.206',
	data:       DNSName: 'my-vm.chinacloudapp.cn',
	data:       InstanceStateDetails: {},
	data:       VMName: 'my-vm',
	data:       Network: {
	data:           Endpoints: [
	data:               {
	data:                   Protocol: 'tcp',
	data:                   Vip: '65.52.250.250',
	data:                   Port: '63238' ,
	data:                   LocalPort: '3389',
	data:                   Name: 'RemoteDesktop'
	data:               }
	data:           ]
	data:       },
	data:       Image: 'MSFT__Windows-Server-2008-R2-SP1.11-29-2011',
	data:       OSVersion: 'WA-GUEST-OS-1.18_201203-01'
	data:   }
	info:   vm show command OK

**vm delete [options] &lt;name>**

此命令删除 Azure 虚拟机。默认情况下，此命令不删除从中创建操作系统磁盘和数据磁盘的 Azure Blob。若要删除该 Blob 以及作为其基础的虚拟机，请指定 -b 选项。

	~$ azure vm delete my-vm
	info:   Executing command vm delete
	info:   vm delete command OK

**vm start [options] &lt;name>**

此命令启动 Azure 虚拟机。

	~$ azure vm start my-vm
	info:   Executing command vm start
	info:   vm start command OK

**vm restart [options] &lt;name>**

此命令重新启动 Azure 虚拟机。

	~$ azure vm restart my-vm
	info:   Executing command vm restart
	info:   vm restart command OK

**vm shutdown [options] &lt;name>**

此命令关闭 Azure 虚拟机。可以使用 -p 选项指定在关闭时不释放计算资源。

	
	~$ azure vm shutdown my-vm
	info:   Executing command vm shutdown
	info:   vm shutdown command OK  
	

**vm capture &lt;vm-name> &lt;target-image-name>**

此命令捕获 Azure 虚拟机映像。

只有当虚拟机状态为**已停止**时，才能捕获虚拟机映像。请关闭虚拟机，然后再继续操作。

	~$ azure.cmd vm capture my-vm mycaptureimagename --delete
	info:   Executing command vm capture
	+ Fetching VMs
	+ Capturing VM
	info:   vm capture command OK

**vm export [options] &lt;vm-name> &lt;file-path>**

此命令将一个 Azure 虚拟机映像导出到文件

	~$ azure vm export "myvm" "C:"
	info:    Executing command vm export
	+ Getting virtual machines
	+ Exporting the VM
	info:   vm export command OK

##  用于管理 Azure 虚拟机终结点的命令
下图显示了多个虚拟机实例的典型部署的体系结构。请注意，在本示例中，端口 3389 在每台虚拟机上均为打开状态（用于进行 RDP 访问），并且负载平衡器用于将流量路由到虚拟机的每台虚拟机上还有一个内部 IP 地址（例如，168.55.11.1）。此内部 IP 地址也可用于虚拟机之间的通信。

![azurenetworkdiagram](./media/virtual-machines-command-line-tools/networkdiagram.jpg)

虚拟机的外部请求将通过负载平衡器。因此，不能针对包含多台虚拟机的部署中的特定虚拟机指定请求。对于包含多台虚拟机的部署，必须在虚拟机 (vm-port) 与负载平衡器 (lb-port) 之间配置端口映射。

**vm endpoint create &lt;vm-name> &lt;lb-port> [vm-port]**

此命令创建虚拟机终结点。你还可以使用 -u 或 --enable-direct-server-return 来指定是否在此终结点上启用直接服务器返回，默认情况下为禁用。

	~$ azure vm endpoint create my-vm 8888 8888
	azure vm endpoint create my-vm 8888 8888
	info:   Executing command vm endpoint create
	+ Fetching VM
	+ Reading network configuration
	+ Updating network configuration
	info:   vm endpoint create command OK

**vm endpoint create-multiple [options] &lt;vm-name> &lt;lb-port>[:&lt;vm-port>[:&lt;protocol>[:&lt;enable-direct-server-return>[:&lt;lb-set-name>[:&lt;probe-protocol>[:&lt;probe-port>[:&lt;probe-path>[:&lt;internal-lb-name>]]]]]]]] {1-*}**

创建多个 VM 终结点。

**vm endpoint delete [options] &lt;vm-name> &lt;endpoint-name>**

此命令删除虚拟机终结点。

	~$ azure vm endpoint delete my-vm http
	azure vm endpoint delete my-vm http
	info:   Executing command vm endpoint delete
	+ Fetching VM
	+ Reading network configuration
	+ Updating network configuration
	info:   vm endpoint delete command OK

**vm endpoint list &lt;vm-name>**

此命令列出所有虚拟机终结点。--json 选项指定以原始 JSON 格式返回结果。

	~$ azure vm endpoint list my-linux-vm
	data:   Name  External Port  Local Port
	data:   ----  -------------  ----------
	data:   ssh   22             22

**vm endpoint update [options] &lt;vm-name> &lt;endpoint-name>**

此命令使用这些选项将 VM 终结点更新为新值。

    -n, --endpoint-name <name>          the new endpoint name
    -lo, --lb-port <port>                the new load balancer port
    -t, --vm-port <port>                the new local port
    -o, --endpoint-protocol <protocol>  the new transport layer protocol for port (tcp or udp)

**vm endpoint show [options] &lt;vm-name>**

此命令显示 VM 上终结点的详细信息

	~$ azure vm endpoint show "mycouchvm"
	info:    Executing command vm endpoint show
	+ Getting virtual machines
	data:    Network Endpoints 0 LoadBalancedEndpointSetName "CouchDB_EP-5984"
	data:    Network Endpoints 0 LocalPort "5984"
	data:    Network Endpoints 0 Name "CouchDB_EP"
	data:    Network Endpoints 0 Port "5984"
	data:    Network Endpoints 0 Protocol "tcp"
	data:    Network Endpoints 0 Vip "168.61.9.97"
	data:    Network Endpoints 1 LoadBalancedEndpointSetName "CouchEP_2-2020"
	data:    Network Endpoints 1 LocalPort "2020"
	data:    Network Endpoints 1 Name "CouchEP_2"
	data:    Network Endpoints 1 Port "2020"
	data:    Network Endpoints 1 Protocol "tcp"
	data:    Network Endpoints 1 Vip "168.61.9.97"
	data:    Network Endpoints 2 LocalPort "3389"
	data:    Network Endpoints 2 Name "RemoteDesktop"
	data:    Network Endpoints 2 Port "3389"
	data:    Network Endpoints 2 Protocol "tcp"
	data:    Network Endpoints 2 Vip "168.61.9.97"
	info:    vm endpoint show command OK

## 用于管理 Azure 虚拟机映像的命令

虚拟机映像是所捕获的、可根据需要进行复制的已配置虚拟机。

**vm image list [options]**

此命令获取虚拟机映像的列表。有三种类型的映像：Microsoft 创建的映像（以“MSFT”作为前缀）、第三方创建的映像（通常以供应商的名称作为前缀）以及你创建的映像。若要创建映像，你可以捕获现有虚拟机或从上载到 Blob 存储的自定义 .vhd 创建映像。有关使用自定义 .vhd 的更多信息，请参见 VM 映像创建。--json 选项指定以原始 JSON 格式返回结果。

	~$ azure vm image list
	data:   Name                                                                   Category   OS
	data:   ---------------------------------------------------------------------  ---------  -------
	data:   CANONICAL__Canonical-Ubuntu-12-04-20120519-2012-05-19-zh-CN-30GB.vhd   Canonical  Linux
	data:   MSFT__Windows-Server-2008-R2-SP1.11-29-2011                            Microsoft  Windows
	data:   MSFT__Windows-Server-2008-R2-SP1-with-SQL-Server-2012-Eval.11-29-2011  Microsoft  Windows
	data:   MSFT__Windows-Server-8-Beta.zh-CN.30GB.2012-03-22                      Microsoft  Windows
	data:   MSFT__Windows-Server-8-Beta.2-17-2012                                  Microsoft  Windows
	data:   MSFT__Windows-Server-2008-R2-SP1.zh-CN.30GB.2012-3-22                  Microsoft  Windows
	data:   OpenLogic__OpenLogic-CentOS-62-20120509-zh-CN-30GB.vhd                 OpenLogic  Linux
	data:   SUSE__SUSE-Linux-Enterprise-Server-11SP2-20120521-zh-CN-30GB.vhd       SUSE       Linux
	data:   SUSE__OpenSUSE64121-03192012-zh-CN-15GB.vhd                            SUSE       Linux
	data:   WIN2K8-R2-WINRM                                                        User       Windows
	info:   vm image list command OK

**vm image show [options] &lt;name>**

此命令显示虚拟机映像的详细信息。

	~$ azure vm image show MSFT__Windows-Server-2008-R2-SP1.11-29-2011
	+ Fetching VM image
	info:   Executing command vm image show
	data:   {
	data:       Label: 'Windows Server 2008 R2 SP1, Nov 2011',
	data:       Name: 'MSFT__Windows-Server-2008-R2-SP1.11-29-2011',
	data:       Description: 'Microsoft Windows Server 2008 R2 SP1',
	data:       @: { xmlns: 'http://schemas.microsoft.com/windowsazure', xmlns:i: 'http://www.w3.org/2001/XMLSchema-instance' },
	data:       Category: 'Microsoft',
	data:       OS: 'Windows',
	data:       Eula: 'http://www.microsoft.com',
	data:       LogicalSizeInGB: '30'
	data:   }
	info:   vm image show command OK

**vm image delete [options] &lt;name>**

此命令删除虚拟机映像。

	~$ azure vm image delete my-vm-image
	info:   Executing command vm image delete
	info:   VM image deleted: my-vm-image
	info:   vm image delete command OK

**vm image create &lt;name> [source-path]**

此命令创建虚拟机映像。你的自定义 .vhd 文件将上载到 Blob 存储，然后从该位置创建虚拟机映像。然后可使用此虚拟机映像创建虚拟机。Location 和 OS 参数是必需的。

某些系统会施加每进程文件描述符限制。如果超出此限制，工具将显示文件描述符限制错误。你可以使用 -p &lt;number> 参数再次运行此命令，以减小最大并行上载数。默认的最大并行上载数为 96。

	~$ azure vm image create mytestimage ./Sample.vhd -o windows -l "China North"
	info:   Executing command vm image create
	+ Retrieving storage accounts
	info:   VHD size : 13 MB
	info:   Uploading 13312.5 KB
	Requested:100.0% Completed:100.0% Running: 105 Time:    8s Speed:  1721 KB/s
	info:   http://myaccount.blob.core.azure.com/vm-images/Sample.vhd is uploaded successfully
	info:   vm image create command OK

##<a name="commands-to-manage-your-azure-virtual-machine-data-disks"></a> 用于管理 Azure 虚拟机数据磁盘的命令

数据磁盘是 Blob 存储中可供虚拟机使用的 .vhd 文件。有关如何将数据磁盘部署到 Blob 存储的详细信息，请参阅前面所示的 Azure 技术图表。

用于附加数据磁盘的命令（azure vm disk attach 和 azure vm disk attach-new）会根据 SCSI 协议的要求向附加的数据磁盘分配逻辑单元号 (LUN)。将向附加到虚拟机的第一个数据磁盘分配 LUN 0，向下一个分配 LUN 1，依此类推。

当使用 azure vm disk detach 命令分离数据磁盘时，请使用 &lt;lun&gt; 参数指明要分离的磁盘。

> [AZURE>NOTE] 请注意，应始终按相反的顺序分离数据磁盘，即，从已分配的编号最高的 LUN 开始。Linux SCSI 层不支持在仍附加有编号较高的 LUN 时分离编号较低的 LUN。例如，不应在仍附加有 LUN 1 的情况下分离 LUN 0。

**vm disk show [options] &lt;name>**

此命令显示有关 Azure 磁盘的详细信息。

	~$ azure vm disk show anucentos-anucentos-0-20120524070008
	info:   Executing command vm disk show
	data:   AttachedTo DeploymentName "mycentos"
	data:   AttachedTo HostedServiceName "myanucentos"
	data:   AttachedTo RoleName "myanucentos"
	data:   OS "Linux"
	data:   Location "Azure Preview"
	data:   LogicalDiskSizeInGB "30"
	data:   MediaLink "http://mystorageaccount.blob.core.azure-preview.com/vhd-store/mycentos-cb39b8223b01f95c.vhd"
	data:   Name "mycentos-mycentos-0-20120524070008"
	data:   SourceImageName "OpenLogic__OpenLogic-CentOS-62-20120509-zh-CN-30GB.vhd"
	info:   vm disk show command OK

**vm disk list [options] [vm-name]**

此命令列出 Azure 磁盘，或者附加到指定虚拟机的磁盘。如果运行此命令时使用了虚拟机名称参数，将返回附加到该虚拟机的所有磁盘。Lun 1 是随虚拟机创建的，而列出的任何其他磁盘都是单独附加的。

	~$ azure vm disk list mycentos
	info:   Executing command vm disk list
	data:   Lun  Size(GB)  Blob-Name
	data:   ---  --------  --------------------------------
	data:   1    30        mycentos-cb39b8223b01f95c.vhd
	data:   2    10        mycentos-e3f0d717950bb78d.vhd
	info:   vm disk list command OK

在不使用虚拟机名称参数的情况下执行此命令将返回所有磁盘。

	~$ azure vm disk list
	data:   Name                                        OS
	data:   ------------------------------------------  -------
	data:   mycentos-mycentos-0-20120524070008          Linux
	data:   mycentos-mycentos-2-20120525055052
	data:   mywindows-winvm-20120522223119              Windows
	info:   vm disk list command OK

**vm disk delete [options] &lt;name>**

此命令从个人存储库中删除 Azure 磁盘。在删除磁盘之前必须从虚拟机中分离该磁盘。

	~$ azure vm disk delete mycentos-mycentos-2-20120525055052
	info:   Executing command vm disk delete
	info:   Disk deleted: mycentos-mycentos-2-20120525055052
	info:   vm disk delete command OK

**vm disk create &lt;name> [source-path]**

此命令将上载并注册 Azure 磁盘。必须指定 --blob-url、--location 或 --affinity-group。如果将此命令与 [source-path] 结合使用，将上载指定的 .vhd 文件并创建新映像。然后你可以使用 vm disk attach 将此映像附加到虚拟机。

某些系统会施加每进程文件描述符限制。如果超出此限制，工具将显示文件描述符限制错误。你可以使用 -p &lt;number> 参数再次运行此命令，以减小最大并行上载数。默认的最大并行上载数为 96。

	~$ azure vm disk create my-data-disk ~/test.vhd --location "China North"
	info:   Executing command vm disk create
	info:   VHD size : 10 MB
	info:   Uploading 10240.5 KB
	Requested:100.0% Completed:100.0% Running:  81 Time:   11s Speed:   952 KB/s
	info:   http://account.blob.core.azure.com/disks/test.vhd is uploaded successfully
	info:   vm disk create command OK

**vm disk upload [options] &lt;source-path> &lt;blob-url> &lt;storage-account-key>**

此命令用来上载 vm 磁盘

	~$ azure vm disk upload "http://sourcestorage.blob.core.chinacloudapi.cn/vhds/sample.vhd" "http://destinationstorage.blob.core.chinacloudapi.cn/vhds/sample.vhd" "DESTINATIONSTORAGEACCOUNTKEY"
	info:   Executing command vm disk upload
	info:   Uploading 12351.5 KB
	info:   vm disk upload command OK

**vm disk attach &lt;vm-name> &lt;disk-image-name>**

此命令将 Blob 存储中的现有磁盘附加到云服务中部署的现有虚拟机。

	~$ azure vm disk attach my-vm my-vm-my-vm-2-201242418259
	info:   Executing command vm disk attach
	info:   vm disk attach command OK

**vm disk attach-new &lt;vm-name> &lt;size-in-gb> [blob-url]**

此命令将数据磁盘附加到 Azure 虚拟机。在此示例中，20 是要附加的新磁盘的大小（以 GB 为单位）。你可以选择使用 Blob URL 作为显式指定要创建的目标 Blob 的最后一个参数。如果你不指定 Blob URL，将自动生成一个 Blob 对象。

	~$ azure vm disk attach-new nick-test36 20 http://nghinazz.blob.core.azure-preview.com/vhds/vmdisk1.vhd
	info:   Executing command vm disk attach-new
	info:   vm disk attach-new command OK  

**vm disk detach &lt;vm-name> &lt;lun>**

此命令将数据磁盘与 Azure 虚拟机分离。&lt;lun> 标识要分离的磁盘。要在分离某个磁盘之前获取与该磁盘关联的磁盘的列表，请使用 vm disk-list &lt;vm-name>。

	~$ azure vm disk detach my-vm 2
	info:   Executing command vm disk detach
	info:   vm disk detach command OK

## 用于管理 Azure 云服务的命令

Azure 云服务是托管在 Web 角色和辅助角色上的应用程序和服务。以下命令可用于管理 Azure 云服务。

**service create [options] &lt;serviceName>**

此命令创建新的云服务

	~$ azure service create newservicemsopentech
	info:    Executing command service create
	+ Getting locations
	help:    Location:
	  1) East Asia
	  2) Southeast Asia
	  3) North Europe
	  4) West Europe
	  5) East US
	  6) West US
	  : 6
	+ Creating cloud service
	data:    Cloud service name newservicemsopentech
	info:    service create command OK

**service show [options] &lt;serviceName>**

此命令显示 Azure 云服务的详细信息

	~$ azure service show newservicemsopentech
	info:    Executing command service show
	+ Getting cloud service
	data:    Name newservicemsopentech
	data:    Url https://management.core.chinacloudapi.cn/9e672699-1055-41ae-9c36-e85152f2e352/services/hostedservices/newservicemsopentech
	data:    Properties location West US
	data:    Properties label newservicemsopentech
	data:    Properties status Created
	data:    Properties dateCreated
	data:    Properties dateLastModified
	info:    service show command OK

**service list [options]**

此命令列出 Azure 云服务。

	~$ azure service list
	info:   Executing command service list
	data:   Name         Status
	data:   -----------  -------
	data:   service1     Created
	data:   service2     Created
	info:   service list command OK

**service delete [options] &lt;name>**

此命令删除 Azure 云服务。

	~$ azure service delete myservice
	info:   Executing command service delete myservice
	info:   cloud-service delete command OK

若要强制删除，请使用 `-q` 参数。


## 用于管理 Azure 证书的命令

Azure 服务证书是连接到你的 Azure 帐户的 SSL 证书。有关 Azure 证书的详细信息，请参阅[管理证书](http://msdn.microsoft.com/zh-cn/library/azure/gg981929.aspx)。

**service cert list [options]**

此命令列出 Azure 证书。

	~$ azure service cert list
	info:   Executing command service cert list
	+ Fetching cloud services
	+ Fetching certificates
	data:   Service   Thumbprint                                Algorithm
	data:   --------  ----------------------------------------  ---------
	data:   myservice  262DBF95B5E61375FA27F1E74AC7D9EAE842916C  sha1
	info:   service cert list command OK

**service cert create &lt;dns-prefix> &lt;file> [password]**

此命令上载证书。将没有密码保护的证书的密码提示保留为空。

	~$ azure service cert create nghinazz ~/publishSet.pfx
	info:   Executing command service cert create
	Cert password:
	+ Creating certificate
	info:   service cert create command OK

**service cert delete [options] &lt;thumbprint>**

此命令删除证书。

	~$ azure service cert delete 262DBF95B5E61375FA27F1E74AC7D9EAE842916C
	info:   Executing command service cert delete
	+ Deleting certificate
	info:   nghinazz : cert deleted
	info:   service cert delete command OK

## 用于管理 Web 应用的命令

Azure Web 应用是可通过 URI 访问的 Web 配置。 Web 应用在虚拟机中托管，但你无需自己考虑创建和部署虚拟机的详细步骤。这些详细步骤将由 Azure 为你完成。

**site list [options]**

此命令列出你的 Web 应用。

	~$ azure site list
	info:   Executing command site list
	data:   Name            State    Host names
	data:   --------------  -------  --------------------------------------------------
	data:   mongosite       Running  mongosite.antdf0.antares.chinacloudapi.cn
	data:   myphpsite       Running  myphpsite.antdf0.antares.chinacloudapi.cn
	data:   mydrupalsite36  Running  mydrupalsite36.antdf0.antares.chinacloudapi.cn
	info:   site list command OK

**site set [options] [name]**

此命令将设置你的 Web 应用[名称] 的配置选项

	~$ azure site set
	info:    Executing command site set
	Web site name: mydemosite
	+ Getting sites
	+ Updating site config information
	info:    site set command OK

**site deploymentscript [options]**

此命令将生成一个自定义部署脚本

	~$ azure site deploymentscript --node
	info:    Executing command site deploymentscript
	info:    Generating deployment script for node.js Web Site
	info:    Generated deployment script files
	info:    site deploymentscript command OK

**site create [options] [name]**

此命令创建新的 Web 应用和本地目录。

	~$ azure site create mysite
	info:   Executing command site create
	info:   Using location northeuropewebspace
	info:   Creating a new web site
	info:   Created web site at  mysite.antdf0.antares.chinacloudapi.cn
	info:   Initializing repository
	info:   Repository initialized
	info:   site create command OK

> [AZURE.NOTE]站点名称必须是唯一的。你无法创建与现有站点具有相同 DNS 名称的站点。

**site browse [options] [name]**

此命令在浏览器中打开你的 Web 应用。

	~$ azure site browse mysite
	info:   Executing command site browse
	info:   Launching browser to http://mysite.antdf0.antares-test.windows-int.net
	info:   site browse command OK

**site show [options] [name]**

此命令显示 Web 应用的详细信息。

	~$ azure site show mysite
	info:   Executing command site show
	info:   Showing details for site
	data:   Site AdminEnabled true
	data:   Site HostNames mysite.antdf0.antares-test.windows-int.net
	data:   Site Name mysite
	data:   Site Owner 00060000814EDDEE
	data:   Site RepositorySiteName mysite
	data:   Site SelfLink https://s1.api.antdf0.antares.chinacloudapi.cn:454/subscriptions/444e62ff-4c5f-4116-a695-5c803ed584a5/webspaces/northeuropewebspace/sites/mysite
	data:   Site State Running
	data:   Site UsageState Normal
	data:   Site WebSpace northeuropewebspace
	data:   Config AppSettings
	data:   Config ConnectionStrings
	data:   Config DefaultDocuments 0=Default.htm, 1=Default.asp, 2=index.htm, 3=index.html, 4=iisstart.htm, 5=default.aspx, 6=index.php, 7=hostingstart.aspx
	data:   Config DetailedErrorLoggingEnabled false
	data:   Config HttpLoggingEnabled false
	data:   Config Metadata
	data:   Config NetFrameworkVersion v4.0
	data:   Config NumberOfWorkers 1
	data:   Config PhpVersion 5.3
	data:   Config PublishingPassword rJ}[Er2v[Y]q16B6vTD]n$[C2z}Z.pvgLfRcLnAp%ax]xstiLny};o@vmMAote@d
	data:   Config RequestTracingEnabled false
	data:   Repository https://mysite.scm.antdf0.antares-test.windows-int.net/
	info:   site show command OK

**site delete [options] [name]**

此命令删除 Web 应用。

	~$ azure site delete mysite
	info:   Executing command site delete
	info:   Deleting site mysite
	info:   Site mysite has been deleted
	info:   site delete command OK

 **site swap [options] [name]**

此命令交换两个 Web 应用插槽。

此命令支持以下附加选项：

****-q 或 **--quiet**：不提示确认。在自动化脚本中使用此选项。


**site start [options] [name]**

此命令启动 Web 应用。

	~$ azure site start mysite
	info:   Executing command site start
	info:   Starting site mysite
	info:   Site mysite has been started
	info:   site start command OK

**site stop [options] [name]**

此命令停止 Web 应用。

	~$ azure site stop mysite
	info:   Executing command site stop
	info:   Stopping site mysite
	info:   Site mysite has been stopped
	info:   site stop command OK

****site restart [options] [name]

此命令停止然后启动指定的 Web 应用。

此命令支持以下附加选项：

**--slot** &lt;slot>：要重新启动的插槽的名称。


**site location list [options]**

此命令列出你的 Web 应用位置。

	~$ azure site location list
	info:    Executing command site location list
	+ Getting locations
	data:    Name
	data:    ----------------
	data:    West Europe
	data:    China North
	data:    North Central China
	data:    North Europe
	data:    East Asia
	data:    ChinaEast
	info:    site location list command OK

###用于管理 Web 应用应用程序设置的命令

**site appsetting list [options] [name]**

此命令列出添加到 Web 应用的应用设置。

	~$ azure site appsetting list
	info:    Executing command site appsetting list
	Web site name: mydemosite
	+ Getting sites
	+ Getting site config information
	data:    Name  Value
	data:    ----  -----
	data:    test  value
	info:    site appsetting list command OK

**site appsetting add [options] &lt;keyvaluepair> [name]**

此命令将应用设置作为键值对添加到你的 Web 应用。

	~$ azure site appsetting add test=value
	info:    Executing command site appsetting add
	Web site name: mydemosite
	+ Getting sites
	+ Getting site config information
	+ Updating site config information
	info:    site appsetting add command OK

**site appsetting delete [options] &lt;key> [name]**

此命令从 Web 应用中删除指定的应用设置。

	~$ azure site appsetting delete test
	info:    Executing command site appsetting delete
	Web site name: mydemosite
	+ Getting sites
	+ Getting site config information
	Delete application setting test? [y/n] y
	+ Updating site config information
	info:    site appsetting delete command OK

**site appsetting show [options] &lt;key> [name]**

此命令显示指定应用程序设置的详细信息

	~$ azure site appsetting show test
	info:    Executing command site appsetting show
	Web site name: mydemosite
	+ Getting sites
	+ Getting site config information
	data:    Value:  value
	info:    site appsetting show command OK

###用于管理 Web 应用证书的命令

**site cert list [options] [name]**

此命令显示 Web 应用证书的列表。

	~$ azure site cert list
	info:    Executing command site cert list
	Web site name: mydemosite
	+ Getting sites
	+ Getting site information
	data:    Subject                       Expiration Date	                  Thumbprint
	data:    ----------------------------  -----------------------------------------
	----------------  ----------------------------------------
	data:    *.msopentech.com              Fri Nov 28 2014 09:49:57 GMT-0800 (Pacific Standard Time)  A40E82D3DC0286D1F58650E570ECF8224F69A148
	data:    msopentech.chinacloudsites.cn  Fri Jun 19 2015 11:57:32 GMT-0700 (Pacific Daylight Time)  CE1CD6538852BF7A5DC32001C2E26A29B541F0E8
	info:    site cert list command OK

**site cert add [options] &lt;certificate-path> [name]**

**site cert delete [options] &lt;thumbprint> [name]**

**site cert show [options] &lt;thumbprint> [name]**

此命令显示证书详细信息

	~$ azure site cert show CE1CD65852B38DC32001C2E0E8F7A526A29B541F
	info:    Executing command site cert show
	Web site name: mydemosite
	+ Getting sites
	+ Getting site information
	data:    Certificate hostNames 0=msopentech.chinacloudsites.cn
	data:    Certificate expirationDate
	data:    Certificate friendlyName msopentech.chinacloudsites.cn
	data:    Certificate issueDate
	data:    Certificate issuer CN=MSIT Machine Auth CA 2, DC=redmond, DC=corp, DC=microsoft, DC=com
	data:    Certificate subjectName msopentech.chinacloudsites.cn
	data:    Certificate thumbprint CE1CD65852B38DC32001C2E0E8F7A526A29B541F
	info:    site cert show command OK

###用于管理 Web 应用连接字符串的命令

**site connectionstring list [options] [name]**

**site connectionstring add [options] &lt;connectionname> &lt;value> &lt;type> [name]**

**site connectionstring delete [options] &lt;connectionname> [name]**

**site connectionstring show [options] &lt;connectionname> [name]**

###用于管理 Web 应用默认文档的命令

**site defaultdocument list [options] [name]**

**site defaultdocument add [options] &lt;document> [name]**

**site defaultdocument delete [options] &lt;document> [name]**

###用于管理 Web 应用部署的命令

**site deployment list [options] [name]**

**site deployment show [options] &lt;commitId> [name]**

**site deployment redeploy [options] &lt;commitId> [name]**

**site deployment github [options] [name]**

**site deployment user set [options] [username] [pass]**

###用于管理 Web 应用域的命令

**site domain list [options] [name]**

**site domain add [options] &lt;dn> [name]**

**site domain delete [options] &lt;dn> [name]**

###用于管理 Web 应用处理程序映射的命令

**site handler list [options] [name]**

**site handler add [options] &lt;extension> &lt;processor> [name]**

**site handler delete [options] &lt;extension> [name]**

###用于管理 Web 作业的命令

**site job list [options] [name]**

此命令列出某个 Web 应用下的所有 Web 作业。

此命令支持以下附加选项：

+ **--job-type** &lt;job-type>：可选。web 作业的类型。有效值为“triggered”或“continuous”。默认情况下返回所有类型的 web 作业。
+ **--slot** &lt;slot>：要重新启动的插槽的名称。

**site job show [options] &lt;jobName> &lt;jobType> [name]**

此命令显示特定 web 作业的详细信息。

此命令支持以下附加选项：

+ **--job-name** &lt;job-name>：必需。web 作业的名称。
+ **--job-type** &lt;job-type>：必需。web 作业的类型。有效值为“triggered”或“continuous”。
+ **--slot** &lt;slot>：要重新启动的插槽的名称。

**site job delete [options] &lt;jobName> &lt;jobType> [name]**

此命令删除指定的 web 作业。

此命令支持以下附加选项：

+ **--job-name** &lt;job-name> 必需。web 作业的名称。
+ **--job-type** &lt;job-type> 必需。web 作业的类型。有效值为“triggered”或“continuous”。
+ **-q** 或 **--quiet**：不提示确认。在自动化脚本中使用此选项。
+ **--slot** &lt;slot>：要重新启动的插槽的名称。

**site job upload [options] &lt;jobName> &lt;jobType> <jobFile> [name]**

此命令删除指定的 web 作业。

此命令支持以下附加选项：

+ **--job-name** &lt;job-name>：必需。web 作业的名称。
+ **--job-type** &lt;job-type>：必需。web 作业的类型。有效值为“triggered”或“continuous”。
+ **--job-file** &lt;job-file>：必需。作业文件。
+ **--slot** &lt;slot>：要重新启动的插槽的名称。

**site job start [options] &lt;jobName> &lt;jobType> [name]**

此命令启动指定的 web 作业。

此命令支持以下附加选项：

+ **--job-name** &lt;job-name>：必需。web 作业的名称。
+ **--job-type** &lt;job-type>：必需。web 作业的类型。有效值为“triggered”或“continuous”。
+ **--slot** &lt;slot>：要重新启动的插槽的名称。

**site job stop [options] &lt;jobName> &lt;jobType> [name]**

此命令停止指定的 web 作业。只有连续作业可以停止。

此命令支持以下附加选项：

+ **--job-name** &lt;job-name>：必需。web 作业的名称。
+ **--slot** &lt;slot>：要重新启动的插槽的名称。

###用于管理 Web 作业历史记录的命令

**site job history list [options] [jobName] [name]**

此命令显示指定的 web 作业的运行历史记录。

此命令支持以下附加选项：

+ **--job-name** &lt;job-name>：必需。web 作业的名称。
+ **--slot** &lt;slot>：要重新启动的插槽的名称。

**site job history show [options] [jobName] [runId] [name]**

此命令获取为指定的 web 作业运行的作业的详细信息。

此命令支持以下附加选项：

+ **--job-name** &lt;job-name>：必需。web 作业的名称。
+ **--run-id** &lt;run-id>：可选。运行历史记录的 id。如果未指定，则显示最新运行。
+ **--slot** &lt;slot>：要重新启动的插槽的名称。

###用于管理 Web 应用诊断的命令

**site log download [options] [name]**

下载包含你的 Web 应用诊断的 .zip 文件。

	~$ azure site log download
	info:    Executing command site log download
	Web site name: mydemosite
	+ Getting sites
	+ Getting site information
	+ Downloading diagnostic log to diagnostics.zip
	info:    site log download command OK

**site log tail [options] [name]**

此命令将你的终端连接到日志流式处理服务。

	~$ azure site log tail
	info:    Executing command site log tail
	Web site name: mydemosite
	+ Getting sites
	+ Getting site information
	2013-11-19T17:24:17  Welcome, you are now connected to log-streaming service.

**site log set [options] [name]**

此命令配置你的 Web 应用的诊断选项。

	~$ azure site log set -a
	info:    Executing command site log set
	+ Getting output options
	help:    Output:
	  1) file
	  2) storage
	  : 1
	Web site name: mydemosite
	+ Getting locations
	+ Getting sites
	+ Getting site information
	+ Getting diagnostic settings
	+ Updating diagnostic settings
	info:    site log set command OK

###用于管理 Web 应用存储库的命令

**site repository branch [options] &lt;branch> [name]**

**site repository delete [options] [name]**

**site repository sync [options] [name]**

###用于管理 Web 应用缩放的命令

**site scale mode [options] &lt;mode> [name]**

**site scale instances [options] &lt;instances> [name]**


## 用于管理 Azure 移动服务的命令

Azure 移动服务汇聚了一系列支持你的应用程序的后端功能的 Azure 服务。移动服务命令分为以下几类：

+ [用于管理移动服务实例的命令](#Mobile_Services)
+ [用于管理移动服务配置的命令](#Mobile_Configuration)
+ [用于管理移动服务表的命令](#Mobile_Tables)
+ [用于管理移动服务脚本的命令](#Mobile_Scripts)
+ [用于管理已计划作业的命令](#Mobile_Jobs)
+ [用于缩放移动服务的命令](#Mobile_Scale)

以下选项适用于多数移动服务命令：

+ **-h** 或 **--help**：显示输出用法信息。
+ **-s `<id>`** 或 **--subscription`<id>`**：使用指定为 `<id>` 的特定订阅。
+ **-v** 或 **--verbose**：写入详细输出。
+ **--json**：写入 JSON 输出。

### <a name="Mobile_Services"></a>用于管理移动服务实例的命令

**mobile locations [options]**

此命令列出移动服务支持的地理位置。

	~$ azure mobile locations
	info:    Executing command mobile locations
	info:    East US (default)
	info:    West US
	info:    North Europe

**mobile create [options] [servicename] [sqlAdminUsername] [sqlAdminPassword]**

此命令创建移动服务以及 SQL 数据库和服务器。

	~$ azure mobile create todolist your_login_name Secure$Password
	info:    Executing command mobile create
	+ Creating mobile service
	info:    Overall application state: Healthy
	info:    Mobile service (todolist) state: ProvisionConfigured
	info:    SQL database (todolist_db) state: Provisioned
	info:    SQL server (e96ean1c6v) state: ProvisionConfigured
	info:    mobile create command OK

此命令支持以下附加选项：

+ **-r `<sqlServer>`** 或 **--sqlServer`<sqlServer>`**：使用指定为 `<sqlServer>` 的现有 SQL 数据库服务器。
+ **-d `<sqlDb>`** 或 **--sqlDb `<sqlDb>`**：使用指定为 `<sqlDb>` 的现有 SQL 数据库。
+ **-l `<location>`** 或 **--location `<location>`**：在指定为 `<location>` 的特定位置创建服务。运行 azure mobile locations 可获取可用位置。
+ **--sqlLocation `<location>`**：在特定的 `<location>` 中创建 SQL 服务器；默认为移动服务的位置。

**mobile delete [options] [servicename]**

此命令删除移动服务及其 SQL 数据库和服务器。

	~$ azure mobile delete todolist -a -q
	info:    Executing command mobile delete
	data:    Mobile service todolist
	data:    SQL database todolistAwrhcL60azo1C401
	data:    SQL server fh1kvbc7la
	+ Deleting mobile service
	info:    Deleted mobile service
	+ Deleting SQL server
	info:    Deleted SQL server
	+ Deleting mobile application
	info:    Deleted mobile application
	info:    mobile delete command OK

此命令支持以下附加选项：

+ **-d** 或 **--deleteData**：从此移动服务的数据库中删除所有数据。
+ **-a** 或 **--deleteAll**：删除 SQL 数据库和服务器。
+ **-q** 或 **--quiet**：不提示确认。在自动化脚本中使用此选项。

**mobile list [options]**

此命令列出您的移动服务。

	~$ azure mobile list
	info:    Executing command mobile list
	data:    Name          State  URL
	data:    ------------  -----  --------------------------------------
	data:    todolist      Ready  https://todolist.azure-mobile.net/
	data:    mymobileapp   Ready  https://mymobileapp.azure-mobile.net/
	info:    mobile list command OK

**mobile show [options] [servicename]**

此命令显示有关移动服务的详细信息。

	~$ azure mobile show todolist
	info:    Executing command mobile show
	+ Getting information
	info:    Mobile application
	data:    status Healthy
	data:    Mobile service name todolist
	data:    Mobile service status ProvisionConfigured
	data:    SQL database name todolistAwrhcL60azo1C401
	data:    SQL database status Linked
	data:    SQL server name fh1kvbc7la
	data:    SQL server status Linked
	info:    Mobile service
	data:    name todolist
	data:    state Ready
	data:    applicationUrl https://todolist.azure-mobile.net/
	data:    applicationKey XXXXXXXXXXXXXXXXXXXXXXXXXXXXXX
	data:    masterKey XXXXXXXXXXXXXXXXXXXXXXXXXXXXXX
	data:    webspace WESTUSWEBSPACE
	data:    region West US
	data:    tables TodoItem
	info:    mobile show command OK

**mobile restart [options] [servicename]**

此命令重新启动移动服务实例。

	~$ azure mobile restart todolist
	info:    Executing command mobile restart
	+ Restarting mobile service
	info:    Service was restarted.
	info:    mobile restart command OK

**mobile log [options] [servicename]**

此命令返回移动服务日志，筛选掉除`error`之外的所有日志类型。

	~$ azure mobile log todolist -t error
	info:    Executing command mobile log
	data:
	data:    timeCreated 2013-01-07T16:04:43.351Z
	data:    type error
	data:    source /scheduler/TestingLogs.js
	data:    message This is an error.
	data:
	info:    mobile log command OK

此命令支持以下附加选项：

+ **-r`<query>`** 或 **--query `<query>`**：执行指定的日志查询。
+ **-t`<type>`** 或 **--type`<type>`**：按条目 `<type>`（可能是 `information`、`warning` 或 `error`）筛选返回的日志。
+ **-k`<skip>`** 或 **--skip`<skip>`**：跳过 `<skip>` 指定的行数。
+ **-p`<top>`** 或 **--top `<top>`**：返回由 `<top>` 指定的特定行数。

> [AZURE.NOTE]**--query** 参数优先于 **--type**、**--skip** 和 **--top**。

**mobile recover [options] [unhealthyservicename] [healthyservicename]**

此命令通过将不正常的移动服务移动到不同区域中的正常移动服务来将其恢复。

此命令支持以下附加选项：

**-q** 或 **--quiet**：不提示确认恢复。

**mobile key regenerate [options] [servicename] [type]**

此命令重新生成移动服务应用程序密钥。

	~$ azure mobile key regenerate todolist application
	info:    Executing command mobile key regenerate
	info:    New application key is SmLorAWVfslMcOKWSsuJvuzdJkfUpt40
	info:    mobile key regenerate command OK

密钥类型为 `master` 和 `application`。

> [AZURE.NOTE]当重新生成密钥时，使用旧密钥的客户端可能无法访问你的移动服务。当重新生成应用程序密钥时，应使用新密钥值更新你的应用程序。

**mobile key set [options] [servicename] [type] [value]**

此命令将移动服务密钥设置为一个特定值。


### <a name="Mobile_Configuration"></a>用于管理移动服务配置的命令

**mobile config list [options] [servicename]**

此命令列出移动服务的配置选项。

	~$ azure mobile config list todolist
	info:    Executing command mobile config list
	+ Getting mobile service configuration
	data:    dynamicSchemaEnabled true
	data:    microsoftAccountClientSecret Not configured
	data:    microsoftAccountClientId Not configured
	data:    microsoftAccountPackageSID Not configured
	data:    facebookClientId Not configured
	data:    facebookClientSecret Not configured
	data:    twitterClientId Not configured
	data:    twitterClientSecret Not configured
	data:    googleClientId Not configured
	data:    googleClientSecret Not configured
	data:    apnsMode none
	data:    apnsPassword Not configured
	data:    apnsCertifcate Not configured
	info:    mobile config list command OK

**mobile config get [options] [servicename] [key]**

此命令获取移动服务的特定配置选项，在此示例中为动态架构。

	~$ azure mobile config get todolist dynamicSchemaEnabled
	info:    Executing command mobile config get
	data:    dynamicSchemaEnabled true
	info:    mobile config get command OK

**mobile config set [options] [servicename] [key] [value]**

此命令设置移动服务的特定配置选项，在此示例中为动态架构。

	~$ azure mobile config set todolist dynamicSchemaEnabled false
	info:    Executing command mobile config set
	info:    mobile config set command OK


### <a name="Mobile_Tables"></a>用于管理移动服务表的命令

**mobile table list [options] [servicename]**

此命令列出您的移动服务中的所有表。

	~$azure mobile table list todolist
	info:    Executing command mobile table list
	data:    Name      Indexes  Rows
	data:    --------  -------  ----
	data:    Channel   1        0
	data:    TodoItem  1        0
	info:    mobile table list command OK

**mobile table show [options] [servicename] [tablename]**

此命令显示有关特定表的返回内容的详情。

	~$azure mobile table show todolist
	info:    Executing command mobile table show
	+ Getting table information
	info:    Table statistics:
	data:    Number of records 5
	info:    Table operations:
	data:    Operation  Script       Permissions
	data:    ---------  -----------  -----------
	data:    insert     1900 bytes   user
	data:    read       Not defined  user
	data:    update     Not defined  user
	data:    delete     Not defined  user
	info:    Table columns:
	data:    Name  Type           Indexed
	data:    ----  -------------  -------
	data:    id    bigint(MSSQL)  Yes
	data:    text      string
	data:    complete  boolean
	info:    mobile table show command OK

**mobile table create [options] [servicename] [tablename]**

此命令创建表。

	~$azure mobile table create todolist Channels
	info:    Executing command mobile table create
	+ Creating table
	info:    mobile table create command OK

此命令支持以下附加选项：

+ **-p`&lt;permissions>`** 或 **--permissions`&lt;permissions>`**：以逗号分隔的 `<operation>`=`<permission>` 对列表，其中 `<operation>` 为 `insert`、`read`、`update` 或 `delete`；`&lt;permissions>` 为 `public`、`application`（默认值）、`user` 或 `admin`。

**mobile data read [options] [servicename] [tablename] [query]**

此命令读取表中的数据。

	~$azure mobile data read todolist TodoItem
	info:    Executing command mobile data read
	data:    id  text     complete
	data:    --  -------  --------
	data:    1   item #1  false
	data:    2   item #2  true
	data:    3   item #3  false
	data:    4   item #4  true
	info:    mobile data read command OK

此命令支持以下附加选项：

+ **-k`<skip>`** 或 **--skip`<skip>`**：跳过 `<skip>` 指定的行数。
+ **-t`<top>`** 或 **--top `<top>`**：返回由 `<top>` 指定的特定行数。
+ **-l** 或 **--list**：以列表格式返回数据。

**mobile table update [options] [servicename] [tablename]**

此命令只更改管理员对表的删除权限。

	~$azure mobile table update todolist Channels -p delete=admin
	info:    Executing command mobile table update
	+ Updating permissions
	info:    Updated permissions
	info:    mobile table update command OK

此命令支持以下附加选项：

+ **-p`&lt;permissions>`** 或 **--permissions`&lt;permissions>`**：以逗号分隔的 `<operation>`=`<permission>` 对列表，其中 `<operation>` 为 `insert`、`read`、`update` 或 `delete`；`&lt;permissions>` 为 `public`、`application`（默认值）、`user` 或 `admin`。
+ **--deleteColumn `<columns>`**：要删除的列的逗号分隔列表，如 `<columns>`。
+ **-q** 或 **--quiet**：删除列而不提示确认。
+ **--addIndex`<columns>`**：要包含在索引中的列的逗号分隔列表。
+ **--deleteIndex`<columns>`**：要从索引中排除的列的逗号分隔列表。

**mobile table delete [options] [servicename] [tablename]**

此命令删除表。

	~$azure mobile table delete todolist Channels
	info:    Executing command mobile table delete
	Do you really want to delete the table (yes/no): yes
	+ Deleting table
	info:    mobile table delete command OK

指定 -q 参数可在不提示确认的情况下删除表。执行此操作可以防止阻止自动化脚本。

**mobile data truncate [options] [servicename] [tablename]**

此命令从表中删除所有数据行。

	~$azure mobile data truncate todolist TodoItem
	info:    Executing command mobile data truncate
	info:    There are 7 data rows in the table.
	Do you really want to delete all data from the table? (y/n): y
	info:    Deleted 7 rows.
	info:    mobile data truncate command OK


### <a name="Mobile_Scripts"></a>用于管理脚本的命令

本部分中的命令用于管理属于移动服务的服务器脚本。有关详细信息，请参阅[使用移动服务中的服务器脚本](/documentation/articles/mobile-services-how-to-use-server-scripts/)。

**mobile script list [options] [servicename]**

此命令列出注册的脚本，包括表和计划程序脚本。

	~$azure mobile script list todolist
	info:    Executing command mobile script list
	+ Getting script information
	info:    Table scripts
	data:    Name                   Size
	data:    ---------------------  ----
	data:    table/TodoItem.delete  256
	data:    table/Devices.insert   1660
	error:   Unable to get shared scripts
	info:    Scheduler scripts
	data:    Name                 Status     Interval   Last run   Next run
	data:    -------------------  ---------  ---------  ---------  ---------
	data:    scheduler/undefined  undefined  undefined  undefined  undefined
	data:    scheduler/undefined  undefined  undefined  undefined  undefined
	info:    mobile script list command OK

**mobile script download [options] [servicename] [scriptname]**

此命令将插入脚本从 TodoItem 表下载到 `table` 子文件夹中名为 `todoitem.insert.js` 的文件中。

	~$azure mobile script download todolist table/todoitem.insert.js
	info:    Executing command mobile script download
	info:    Saved script to ./table/todoitem.insert.js
	info:    mobile script download command OK

此命令支持以下附加选项：

+ **-p`<path>`** 或 **--path `<path>`**：文件中用于保存脚本的位置，其中当前工作目录是默认值。
+ **-f`<file>`** 或 **--file`<file>`**：要将脚本保存在其中的文件的名称。
+ **-o** 或 **--override**：覆盖现有文件。
+ **-c** 或 **--console**：将脚本写入到控制台而不是文件。

**mobile script upload [options] [servicename] [scriptname]**

此命令从 `table` 子文件夹上载名为 `todoitem.insert.js` 的新脚本。

	~$azure mobile script upload todolist table/todoitem.insert.js
	info:    Executing command mobile script upload
	info:    mobile script upload command OK

该文件的名称必须由表和操作名称组成，并且该文件必须位于相对于命令执行位置的 table 子文件夹中。你还可以使用 **-f `<file>`** 或 **--file `<file>`** 参数指定其他包含要注册的脚本的文件的文件名和路径。


**mobile script delete [options] [servicename] [scriptname]**

此命令从 TodoItem 表中删除现有插入脚本。

	~$azure mobile script delete todolist table/todoitem.insert.js
	info:    Executing command mobile script delete
	info:    mobile script delete command OK

### <a name="Mobile_Jobs"></a>用于管理已计划作业的命令

本部分中的命令用于管理属于移动服务的已计划作业。有关详细信息，请参阅[计划作业](http://msdn.microsoft.com/zh-cn/library/windowsazure/jj860528.aspx)。

**mobile job list [options] [servicename]**

此命令列出计划作业。

	~$azure mobile job list todolist
	info:    Executing command mobile job list
	info:    Scheduled jobs
	data:    Job name    Script name           Status    Interval     Last run              Next run
	data:    ----------  --------------------  --------  -----------  --------------------  --------------------
	data:    getUpdates  scheduler/getUpdates  enabled   15 [minute]  2013-01-14T16:15:00Z  2013-01-14T16:30:00Z
	info:    You can manipulate scheduled job scripts using the 'azure mobile script' command.
	info:    mobile job list command OK

**mobile job create [options] [servicename] [jobname]**

此命令创建计划为每小时运行的名为 `getUpdates` 的新作业。

	~$azure mobile job create -i 1 -u hour todolist getUpdates
	info:    Executing command mobile job create
	info:    Job was created in disabled state. You can enable the job using the 'azure mobile job update' command.
	info:    You can manipulate the scheduled job script using the 'azure mobile script' command.
	info:    mobile job create command OK

此命令支持以下附加选项：

+ **-i`<number>`** 或 **--interval`<number>`**：作业时间间隔，数值类型为整数；默认值为 `15`。
+ **-u`<unit>`** 或 **--intervalUnit `<unit>`**：_时间间隔_的单位，可以是以下值之一：
	+ **minute**（默认值）
	+ **hour**
	+ **day**
	+ **month**
	+ **none**（按需作业）
+ **-t`<time>`** **--startTime `<time>`** 脚本的首次运行开始时间，采用 ISO 格式；默认值为 `now`。

> [AZURE.NOTE]创建的新作业处于禁用状态，因为还必须上载脚本。请使用 **mobile script upload** 命令上载脚本并使用 **mobile job update** 命令启用作业。

**mobile job update [options] [servicename] [jobname]**

以下命令启用已禁用的`getUpdates`作业。

	~$azure mobile job update -a enabled todolist getUpdates
	info:    Executing command mobile job update
	info:    mobile job update command OK

此命令支持以下附加选项：

+ **-i`<number>`** 或 **--interval`<number>`**：作业时间间隔，数值类型为整数；默认值为 `15`。
+ **-u`<unit>`** 或 **--intervalUnit `<unit>`**：_时间间隔_的单位，可以是以下值之一：
	+ **minute**（默认值）
	+ **hour**
	+ **day**
	+ **month**
	+ **none**（按需作业）
+ **-t`<time>`** **--startTime `<time>`** 脚本的首次运行开始时间，采用 ISO 格式；默认值为 `now`。
+ **-a`<status>`** 或 **--status `<status>`**：作业状态，可以是 `enabled` 或 `disabled`。

**mobile job delete [options] [servicename] [jobname]**

此命令从 TodoList 服务器中删除 getUpdates 已计划作业。

	~$azure mobile job delete todolist getUpdates
	info:    Executing command mobile job delete
	info:    mobile job delete command OK

> [AZURE.NOTE]删除作业也将删除已上载的脚本。

### <a name="Mobile_Scale"></a>用于缩放移动服务的命令

本部分中的命令用于缩放移动服务。有关详细信息，请参阅[缩放移动服务](http://msdn.microsoft.com/zh-cn/library/windowsazure/jj193178.aspx)。

**mobile scale show [options] [servicename]**

此命令显示缩放信息，包括当前计算模式和实例数。

	~$azure mobile scale show todolist
	info:    Executing command mobile scale show
	data:    webspace WESTUSWEBSPACE
	data:    computeMode Free
	data:    numberOfInstances 1
	info:    mobile scale show command OK

**mobile scale change [options] [servicename]**

此命令将移动服务的规模从免费模式更改为高级模式。

	~$azure mobile scale change -c Reserved -i 1 todolist
	info:    Executing command mobile scale change
	+ Rescaling the mobile service
	info:    mobile scale change command OK

此命令支持以下附加选项：

+ **-c`<mode>`** 或 **--computeMode `<mode>`**：计算模式必须为 `Free` 或 `Reserved`。
+ **-i`<count>`** 或 **--numberOfInstances`<count>`**：在保留模式下运行时使用的实例数。

> [AZURE.NOTE]将计算模式设置为`Reserved`时，同一区域中的所有移动服务都将在高级模式下运行。


###用于为移动服务启用预览版功能的命令

**mobile preview list [options] [servicename]**

此命令显示在指定的服务上可用的预览版功能，以及它们是否已启用。

	~$ azure mobile preview list mysite
	info:    Executing command mobile preview list
	+ Getting preview features
	data:    Preview feature  Enabled
	data:    ---------------  -------
	data:    SourceControl    No
	data:    Users            No
	info:    You can enable preview features using the 'azure mobile preview enable' command.
	info:    mobile preview list command OK

**mobile preview enable [options] [servicename] [featurename]**

此命令为移动服务启用指定的预览版功能。请注意，一旦启用，将无法为移动服务禁用预览版功能。

###用于管理移动服务 API 的命令

**mobile api list [options] [servicename]**

此命令列出你为你的移动服务创建的移动服务自定义 API。

	~$ azure mobile api list mysite
	info:    Executing command mobile api list
	+ Retrieving list of APIs
	info:    APIs
	data:    Name                  Get          Put          Post         Patch        Delete
	data:    --------------------  -----------  -----------  -----------  -----------  -----------
	data:    myCustomRetrieveAPI   application  application  application  application  application
	info:    You can manipulate API scripts using the 'azure mobile script' command.
	info:    mobile api list command OK

**mobile api create [options] [servicename] [apiname]**

创建移动服务自定义 API

	~$ azure mobile api create mysite myCustomRetrieveAPI
	info:    Executing command mobile api create
	+ Creating custom API: 'myCustomRetrieveAPI'
	info:    API was created successfully. You can modify the API using the 'azure mobile script' command.
	info:    mobile api create command OK

此命令支持以下附加选项：

**-p** 或 **--permissions** &lt;permissions>：以逗号分隔的 &lt;方法>=&lt;权限> 对列表。

**mobile api update [options] [servicename] [apiname]**

此命令更新指定的移动服务自定义 API。

此命令支持以下附加选项：

此命令支持以下附加选项：

+ **-p** 或 **--permissions** &lt;permissions>：以逗号分隔的 &lt;方法>=&lt;权限> 对列表。
+ **-f** 或 **--force**：覆盖对权限元数据文件的任何自定义更改。

**mobile api delete [options] [servicename] [apiname]**

	~$ azure mobile api delete mysite myCustomRetrieveAPI
	info:    Executing command mobile api delete
	+ Deleting API: 'myCustomRetrieveAPI'
	info:    mobile api delete command OK

此命令删除指定的移动服务自定义 API。

###用于管理移动应用程序的应用程序设置的命令

**mobile appsetting list [options] [servicename]**

此命令显示指定的服务的移动应用程序的应用程序设置。

	~$ azure mobile appsetting list mysite
	info:    Executing command mobile appsetting list
	+ Retrieving app settings
	data:    Name               Value
	data:    -----------------  -----
	data:    enablebetacontent  true
	info:    mobile appsetting list command OK

**mobile appsetting add [options] [servicename] [name] [value]**

此命令为移动服务添加自定义应用程序设置。

	~$ azure mobile appsetting add mysite enablebetacontent true
	info:    Executing command mobile appsetting add
	+ Retrieving app settings
	+ Adding app setting
	info:    mobile appsetting add command OK

**mobile appsetting delete [options] [servicename] [name]**

此命令为移动服务删除指定的应用程序设置。

	~$ azure mobile appsetting delete mysite enablebetacontent
	info:    Executing command mobile appsetting delete
	+ Retrieving app settings
	+ Removing app setting 'enablebetacontent'
	info:    mobile appsetting delete command OK

**mobile appsetting show [options] [servicename] [name]**

此命令为移动服务删除指定的应用程序设置。

	~$ azure mobile appsetting show mysite enablebetacontent
	info:    Executing command mobile appsetting show
	+ Retrieving app settings
	info:    enablebetacontent: true
	info:    mobile appsetting show command OK

## 管理工具本地设置

本地设置是指你的订阅 ID 和默认存储帐户名称。

**config list [options]**

此命令显示配置设置。

	~$ azure config list
	info:   Displaying config settings
	data:   Setting                Value
	data:   ---------------------  ------------------------------------
	data:   subscription           32-digit-subscription-key
	data:   defaultStorageAccount  name

**config set [options] &lt;name&gt;,&lt;value&gt;**

此命令更改配置设置。

	~$ azure config set defaultStorageAccount myname
	info:   Setting 'defaultStorageAccount' to value 'myname'
	info:   Changes saved.

## 用于管理 Service Bus 的命令

使用这些命令来管理你的 Service Bus 帐户

**sb namespace check [options] &lt;name>**

检查某个 service bus 命名空间是否合法并可用。

**sb namespace create &lt;name> &lt;location>**

创建新的 Service Bus 命名空间。

	~$ azure sb namespace create mysbnamespacea-test "West US"
	info:    Executing command sb namespace create
	+ Creating namespace mysbnamespacea-test in region West US
	data:    Name: mysbnamespacea-test
	data:    Region: West US
	data:    DefaultKey: fBu8nQ9svPIesFfMFVhCFD+/sY0rRbifWMoRpYy0Ynk=
	data:    Status: Activating
	data:    CreatedAt: 2013-11-14T16:23:29.32Z
	data:    AcsManagementEndpoint: https://mysbnamespacea-test-sb.accesscontrol.chinacloudapi.cn/
	data:    ServiceBusEndpoint: https://mysbnamespacea-test.servicebus.chinacloudapi.cn/

	data:    ConnectionString: Endpoint=sb://mysbnamespacea-test.servicebus.windows.
	net/;SharedSecretIssuer=owner;SharedSecretValue=fBu8nQ9svPIesFfMFVhCFD+/sY0rRbif
	WMoRpYy0Ynk=
	data:    SubscriptionId: 8679c8be3b0549d9b8fb4bd232a48931
	data:    Enabled: true
	data:    _: [object Object]
	info:    sb namespace create command OK


**sb namespace delete &lt;name>**

删除某个命名空间。

	~$ azure sb namespace delete mysbnamespacea-test
	info:    Executing command sb namespace delete
	Delete namespace mysbnamespacea-test? [y/n] y
	+ Deleting namespace mysbnamespacea-test
	info:    sb namespace delete command OK

**sb namespace list**

列出为你的帐户创建的所有命名空间。

	~$ azure sb namespace list
	info:    Executing command sb namespace list
	+ Getting namespaces
	data:    Name                 Region   Status
	data:    -------------------  -------  ------
	data:    mysbnamespacea-test  West US  Active
	info:    sb namespace list command OK


**sb namespace location list**

显示所有可用的命名空间位置的列表。

	~$ azure sb namespace location list
	info:    Executing command sb namespace location list
	+ Getting locations
	data:    Name              Code
	data:    ----------------  ----------------
	data:    East Asia         East Asia
	data:    West Europe       West Europe
	data:    North Europe      North Europe
	data:    East US           East US
	data:    Southeast Asia    Southeast Asia
	data:    North Central US  North Central US
	data:    West US           West US
	data:    South Central US  South Central US
	info:    sb namespace location list command OK

**sb namespace show &lt;name>**

显示有关特定命名空间的详细信息。

	~$ azure sb namespace show mysbnamespacea-test
	info:    Executing command sb namespace show
	+ Getting namespace
	data:    Name: mysbnamespacea-test
	data:    Region: West US
	data:    DefaultKey: fBu8nQ9svPIesFfMFVhCFD+/sY0rRbifWMoRpYy0Ynk=
	data:    Status: Active
	data:    CreatedAt: 2013-11-14T16:23:29.32Z
	data:    AcsManagementEndpoint: https://mysbnamespacea-test-sb.accesscontrol.chinacloudapi.cn/
	data:    ServiceBusEndpoint: https://mysbnamespacea-test.servicebus.chinacloudapi.cn/

	data:    ConnectionString: Endpoint=sb://mysbnamespacea-test.servicebus.windows.
	net/;SharedSecretIssuer=owner;SharedSecretValue=fBu8nQ9svPIesFfMFVhCFD+/sY0rRbif
	WMoRpYy0Ynk=
	data:    SubscriptionId: 8679c8be3b0549d9b8fb4bd232a48931
	data:    Enabled: true
	data:    UpdatedAt: 2013-11-14T16:25:37.85Z
	info:    sb namespace show command OK

**sb namespace verify &lt;name>**

检查命名空间是否可用。

## 用于管理存储对象的命令

###用于管理存储帐户的命令

**storage account list [options]**

此命令显示你的订阅上的存储帐户。

	~$ azure storage account list
	info:    Executing command storage account list
	+ Getting storage accounts
	data:    Name             Label  Location
	data:    ---------------  -----  --------
	data:    mybasestorage           West US
	info:    storage account list command OK

**storage account show [options] <name>**

此命令显示有关指定的存储帐户的信息，包括 URI 和帐户属性。

**storage account create [options] <name>**

此命令根据提供的选项创建存储帐户。

	~$ azure storage account create mybasestorage --label PrimaryStorage --location "West US"
	info:    Executing command storage account create
	+ Creating storage account
	info:    storage account create command OK

此命令支持以下附加选项：

+ **-e** 或 **--label** &lt;label>：存储帐户的标签。
+ **-d** 或 **--description** &lt;description>：存储帐户的说明。
+ **-l** 或 **--location** &lt;name>：要在其中创建存储帐户的地理区域。
+ **-a** 或 **--affinity-group** &lt;name>：要与存储帐户关联的地缘组。
+ **--type**：指示要创建的帐户的类型：带冗余选项的标准存储 (LRS/GRS/RAGRS) 或高级存储 (PLRS)。

**storage account set [options] <name>**

此命令更新指定的存储帐户。

	~$ azure storage account set mybasestorage --type GRS
	info:    Executing command storage account set
	+ Updating storage account
	info:    storage account set command OK

此命令支持以下附加选项：

+ **-e** 或 **--label** &lt;label>：存储帐户的标签。
+ **-d** 或 **--description** &lt;description>：存储帐户的说明。
+ **-l** 或 **--location** &lt;name>：要在其中创建存储帐户的地理区域。
+ **--type**：指示帐户的新类型：带冗余选项的标准存储 (LRS/GRS/RAGRS) 或高级存储 (PLRS)。

**storage account delete [options] <name>**

此命令删除指定的存储帐户。

此命令支持以下附加选项：

**-q** 或 **--quiet**：不提示确认。在自动化脚本中使用此选项。

###用于管理存储帐户密钥的命令

**storage account keys list [options] <name>**

此命令列出指定的存储帐户的主要和辅助密钥。

**storage account keys renew [options] <name>**

###用于管理存储容器的命令

**storage container list [options] [prefix]**

此命令显示指定的存储帐户的存储容器列表。存储帐户是通过连接字符串或者存储帐户名称和帐户密钥指定的。

此命令支持以下附加选项：

+ **-p** 或 **-prefix** &lt;prefix>：存储容器名称前缀。
+ **-a** 或 **--account-name** &lt;accountName>：存储帐户名称。
+ **-k** 或 **--account-key** &lt;accountKey>：存储帐户密钥。
+ **-c** 或 **--connection-string** &lt;connectionString>：存储连接字符串。
+ **--debug**：在调试模式下运行 storage 命令。

**storage container show [options] [container]** **storage container create [options] [container]**

此命令为指定的存储帐户创建存储容器。存储帐户是通过连接字符串或者存储帐户名称和帐户密钥指定的。

此命令支持以下附加选项：

+ **--container** &lt;container>：要创建的存储容器的名称。
+ **-p** 或 **-prefix** &lt;prefix>：存储容器名称前缀。
+ **-a** 或 **--account-name** &lt;accountName>：存储帐户名称
+ **-k** 或 **--account-key** &lt;accountKey>：存储帐户密钥
+ **-c** 或 **--connection-string** &lt;connectionString>：存储连接字符串
+ **--debug**：在调试模式下运行 storage 命令。

**storage container delete [options] [container]**

此命令删除指定的存储容器。存储帐户是通过连接字符串或者存储帐户名称和帐户密钥指定的。

此命令支持以下附加选项：

+ **--container** &lt;container>：要创建的存储容器的名称。
+ **-p** 或 **-prefix** &lt;prefix>：存储容器名称前缀。
+ **-a** 或 **--account-name** &lt;accountName>：存储帐户名称。
+ **-k** 或 **--account-key** &lt;accountKey>：存储帐户密钥。
+ **-c** 或 **--connection-string** &lt;connectionString>：存储连接字符串。
+ **--debug**：在调试模式下运行 storage 命令。

**storage container set [options] [container]**

此命令设置存储容器的访问控制列表。存储帐户是通过连接字符串或者存储帐户名称和帐户密钥指定的。

此命令支持以下附加选项：

+ **--container** &lt;container>：要创建的存储容器的名称。
+ **-p** 或 **-prefix** &lt;prefix>：存储容器名称前缀。
+ **-a** 或 **--account-name** &lt;accountName>：存储帐户名称。
+ **-k** 或 **--account-key** &lt;accountKey>：存储帐户密钥。
+ **-c** 或 **--connection-string** &lt;connectionString>：存储连接字符串。
+ **--debug**：在调试模式下运行 storage 命令。

###用于管理存储 blob 的命令

**storage blob list [options] [container] [prefix]**

此命令返回指定的存储容器中的存储 blob 的列表。

此命令支持以下附加选项：

+ **--container** &lt;container>：要创建的存储容器的名称。
+ **-p** 或 **-prefix** &lt;prefix>：存储容器名称前缀。
+ **-a** 或 **--account-name** &lt;accountName>：存储帐户名称。
+ **-k** 或 **--account-key** &lt;accountKey>：存储帐户密钥。
+ **-c** 或 **--connection-string** &lt;connectionString>：存储连接字符串。
+ **--debug**：在调试模式下运行 storage 命令。

**storage blob show [options] [container] [blob]**

此命令显示指定的存储 blob 的详细信息。

此命令支持以下附加选项：

+ **--container** &lt;container>：要创建的存储容器的名称。
+ **-p** 或 **-prefix** &lt;prefix>：存储容器名称前缀。
+ **-a** 或 **--account-name** &lt;accountName>：存储帐户名称。
+ **-k** 或 **--account-key** &lt;accountKey>：存储帐户密钥。
+ **-c** 或 **--connection-string** &lt;connectionString>：存储连接字符串。
+ **--debug**：在调试模式下运行 storage 命令。

**storage blob delete [options] [container] [blob]**

此命令支持以下附加选项：

+ **--container** &lt;container>：要创建的存储容器的名称。
+ **-b** 或 **--blob** &lt;blobName>：要删除的存储 blob 的名称。
+ **-q** 或 **--quiet**：删除指定的存储 blob 且不确认。
+ **-a** 或 **--account-name** &lt;accountName>：存储帐户名称。
+ **-k** 或 **--account-key** &lt;accountKey>：存储帐户密钥。
+ **-c** 或 **--connection-string** &lt;connectionString>：存储连接字符串。
+ **--debug**：在调试模式下运行 storage 命令。

**storage blob upload [options] [file] [container] [blob]**

此命令将指定的文件上载到指定的存储 blob。

此命令支持以下附加选项：

+ **--container** &lt;container>：要创建的存储容器的名称。
+ **-b** 或 **--blob** &lt;blobName>：要上载的存储 blob 的名称。
+ **-t** 或 **--blobtype** &lt;blobtype>：存储 blob 类型：Page 或 Block。
+ **-p** 或 **--properties** &lt;properties>：上载的文件的存储 blob 属性。属性是以分号 (;) 分隔的“键=值”对。可用的属性有 contentType、contentEncoding、contentLanguage 和 cacheControl。
+ **-m** 或 **--metadata** &lt;metadata>：上载的文件的存储 blob 元数据。元数据是以分号 (;) 分隔的“键=值”对。
+ **--concurrenttaskcount** &lt;concurrenttaskcount>：并发上载请求的最大数目。
+ **-q** 或 **--quiet**：覆盖指定的存储 blob 且不确认。
+ **-a** 或 **--account-name** &lt;accountName>：存储帐户名称。
+ **-k** 或 **--account-key** &lt;accountKey>：存储帐户密钥。
+ **-c** 或 **--connection-string** &lt;connectionString>：存储连接字符串。
+ **--debug**：在调试模式下运行 storage 命令。

**storage blob download [options] [container] [blob] [destination]**

此命令下载指定的存储 blob。

此命令支持以下附加选项：

+ **--container** &lt;container>：要创建的存储容器的名称。
+ **-b** 或 **--blob** &lt;blobName>：存储 blob 名称。
+ **-d** 或 **--destination** [destination]：下载目标文件或目录路径。
+ **-m** 或 **--checkmd5**：下载的文件的校验 md5sum。
+ **--concurrenttaskcount** &lt;concurrenttaskcount> 并发上载请求的最大数目
+ **-q** 或 **--quiet**：覆盖目标文件且不确认。
+ **-a** 或 **--account-name** &lt;accountName>：存储帐户名称。
+ **-k** 或 **--account-key** &lt;accountKey>：存储帐户密钥。
+ **-c** 或 **--connection-string** &lt;connectionString>：存储连接字符串。
+ **--debug**：在调试模式下运行 storage 命令。

## 用于管理 SQL 数据库的命令

使用这些命令来管理你的 Azure SQL 数据库

###用于管理 SQL Server 的数据库

使用这些命令来管理你的 SQL Server

**sql server create &lt;administratorLogin> &lt;administratorPassword> &lt;location>**

创建新的数据库服务器

	~$ azure sql server create test T3stte$t "West US"
	info:    Executing command sql server create
	+ Creating SQL Server
	data:    Server Name i1qwc540ts
	info:    sql server create command OK

**sql server show &lt;name>**

显示服务器详细信息。

	~$ azure sql server show xclfgcndfg
	info:    Executing command sql server show
	+ Getting SQL server
	data:    SQL Server Name xclfgcndfg
	data:    SQL Server AdministratorLogin msopentechforums
	data:    SQL Server Location West US
	data:    SQL Server FullyQualifiedDomainName xclfgcndfg.database.chinacloudapi.cn
	info:    sql server show command OK

**sql server list**

获取服务器的列表。

	~$ azure sql server list
	info:    Executing command sql server list
	+ Getting SQL server
	data:    Name        Location
	data:    ----------  --------
	data:    xclfgcndfg  West US
	info:    sql server list command OK

**sql server delete &lt;name>**

删除服务器

	~$ azure sql server delete i1qwc540ts
	info:    Executing command sql server delete
	Delete server i1qwc540ts? [y/n] y
	+ Removing SQL Server
	info:    sql server delete command OK

###用于管理 SQL 数据库的命令

使用这些命令来管理你的 SQL 数据库。

**sql db create [options] &lt;serverName> &lt;databaseName> &lt;administratorPassword>**

创建一个新的数据库实例

	~$ azure sql db create fr8aelne00 newdb test
	info:    Executing command sql db create
	Administrator password: ********
	+ Creating SQL Server Database
	info:    sql db create command OK

**sql db show [options] &lt;serverName> &lt;databaseName> &lt;administratorPassword>**

显示数据库详细信息。

	C:\windows\system32>azure sql db show fr8aelne00 newdb test
	info:    Executing command sql db show
	Administrator password: ********
	+ Getting SQL server databases
	data:    Database _ ContentRootElement=m:properties, id=https://fr8aelne00.datab
	ase.chinacloudapi.cn/v1/ManagementService.svc/Server2('fr8aelne00')/Databases(4), ter
	m=Microsoft.SqlServer.Management.Server.Domain.Database, scheme=http://schemas.m
	icrosoft.com/ado/2007/08/dataservices/scheme, link=[rel=edit, title=Database, hr
	ef=Databases(4), rel=http://schemas.microsoft.com/ado/2007/08/dataservices/relat
	ed/Server, type=application/atom+xml;type=entry, title=Server, href=Databases(4)
	/Server, rel=http://schemas.microsoft.com/ado/2007/08/dataservices/related/Servi
	ceObjective, type=application/atom+xml;type=entry, title=ServiceObjective, href=
	Databases(4)/ServiceObjective, rel=http://schemas.microsoft.com/ado/2007/08/data
	services/related/DatabaseMetrics, type=application/atom+xml;type=entry, title=Da
	tabaseMetrics, href=Databases(4)/DatabaseMetrics, rel=http://schemas.microsoft.c
	om/ado/2007/08/dataservices/related/DatabaseCopies, type=application/atom+xml;ty
	pe=feed, title=DatabaseCopies, href=Databases(4)/DatabaseCopies], title=, update
	d=2013-11-18T19:48:27Z, name=
	data:    Database Id 4
	data:    Database Name newdb
	data:    Database ServiceObjectiveId 910b4fcb-8a29-4c3e-958f-f7ba794388b2
	data:    Database AssignedServiceObjectiveId 910b4fcb-8a29-4c3e-958f-f7ba794388b2
	data:    Database ServiceObjectiveAssignmentState 1
	data:    Database ServiceObjectiveAssignmentStateDescription Complete
	data:    Database ServiceObjectiveAssignmentErrorCode
	data:    Database ServiceObjectiveAssignmentErrorDescription
	data:    Database ServiceObjectiveAssignmentSuccessDate
	data:    Database Edition Web
	data:    Database MaxSizeGB 1
	data:    Database MaxSizeBytes 1073741824
	data:    Database CollationName SQL_Latin1_General_CP1_CI_AS
	data:    Database CreationDate
	data:    Database RecoveryPeriodStartDate
	data:    Database IsSystemObject
	data:    Database Status 1
	data:    Database IsFederationRoot
	data:    Database SizeMB -1
	data:    Database IsRecursiveTriggersOn
	data:    Database IsReadOnly
	data:    Database IsFederationMember
	data:    Database IsQueryStoreOn
	data:    Database IsQueryStoreReadOnly
	data:    Database QueryStoreMaxSizeMB
	data:    Database QueryStoreFlushPeriodSeconds
	data:    Database QueryStoreIntervalLengthMinutes
	data:    Database QueryStoreClearAll
	data:    Database QueryStoreStaleQueryThresholdDays
	info:    sql db show command OK

**sql db list [options] &lt;serverName> &lt;administratorPassword>**

列出数据库。

	~$ azure sql db list fr8aelne00 test
	info:    Executing command sql db list
	Administrator password: ********
	+ Getting SQL server databases
	data:    Name    Edition  Collation                     MaxSizeInGB
	data:    ------  -------  ----------------------------  -----------
	data:    master  Web      SQL_Latin1_General_CP1_CI_AS  5
	info:    sql db list command OK

**sql db delete [options] &lt;serverName> &lt;databaseName> &lt;administratorPassword>**

删除数据库。

	~$ azure sql db delete fr8aelne00 newdb test
	info:    Executing command sql db delete
	Administrator password: ********
	Delete database newdb? [y/n] y
	+ Getting SQL server databases
	+ Removing database
	info:    sql db delete command OK

###管理 SQL Server 防火墙规则的命令

使用这些命令来管理 SQL Server 防火墙规则

**sql firewallrule create [options] &lt;serverName> &lt;ruleName> &lt;startIPAddress> &lt;endIPAddress>**

为 SQL Server 创建新的防火墙规则。

	~$ azure sql firewallrule create fr8aelne00 allowed 131.107.0.0 131.107.255.255
	info:    Executing command sql firewallrule create
	+ Creating Firewall Rule
	info:    sql firewallrule create command OK

**sql firewallrule show [options] &lt;serverName> &lt;ruleName>**

显示防火墙规则详细信息。

	~$ azure sql firewallrule show fr8aelne00 allowed
	info:    Executing command sql firewallrule show
	+ Getting firewall rule
	data:    Firewall rule Name allowed
	data:    Firewall rule Type Microsoft.SqlAzure.FirewallRule
	data:    Firewall rule State Normal
	data:    Firewall rule SelfLink https://management.core.chinacloudapi.cn/9e672699-105
	5-41ae-9c36-e85152f2e352/services/sqlservers/servers/fr8aelne00/firewallrules/allowed
	data:    Firewall rule ParentLink https://management.core.chinacloudapi.cn/9e672699-1
	055-41ae-9c36-e85152f2e352/services/sqlservers/servers/fr8aelne00
	data:    Firewall rule StartIPAddress 131.107.0.0
	data:    Firewall rule EndIPAddress 131.107.255.255
	info:    sql firewallrule show command OK

**sql firewallrule list [options] &lt;serverName>**

列出防火墙规则。

	~$ azure sql firewallrule list fr8aelne00
	info:    Executing command sql firewallrule list
	\data:    Name     Start IP address  End IP address
	data:    -------  ----------------  ---------------
	data:    allowed  131.107.0.0       131.107.255.255
	+
	info:    sql firewallrule list command OK

**sql firewallrule delete [options] &lt;serverName> &lt;ruleName>**

此命令将删除防火墙规则。

	~$ azure sql firewallrule delete fr8aelne00 allowed
	info:    Executing command sql firewallrule delete
	Delete rule allowed? [y/n] y
	+ Removing firewall rule
	info:    sql firewallrule delete command OK

## 用于管理虚拟网络的命令

使用这些命令来管理你的虚拟网络

**network vnet create [options] &lt;location>**

创建新的虚拟网络。

	~$ azure network vnet create vnet1 --location "West US" -v
	info:    Executing command network vnet create
	info:    Using default address space start IP: 10.0.0.0
	info:    Using default address space cidr: 8
	info:    Using default subnet start IP: 10.0.0.0
	info:    Using default subnet cidr: 11
	verbose: Address Space [Starting IP/CIDR (Max VM Count)]: 10.0.0.0/8 (16777216)
	verbose: Subnet [Starting IP/CIDR (Max VM Count)]: 10.0.0.0/11 (2097152)
	verbose: Fetching Network Configuration
	verbose: Fetching or creating affinity group
	verbose: Fetching Affinity Groups
	verbose: Fetching Locations
	verbose: Creating new affinity group AG1
	info:    Using affinity group AG1
	verbose: Updating Network Configuration
	info:    network vnet create command OK

**network vnet show &lt;name>**

显示虚拟网络的详细信息。

	~$ azure network vnet show vnet1
	info:    Executing command network vnet show
	+ Fetching Virtual Networks
	data:    Name "vnet1"
	data:    Id "25786fbe-08e8-4e7e-b1de-b98b7e586c7a"
	data:    AffinityGroup "AG1"
	data:    State "Created"
	data:    AddressSpace AddressPrefixes 0 "10.0.0.0/8"
	data:    Subnets 0 Name "subnet-1"
	data:    Subnets 0 AddressPrefix "10.0.0.0/11"
	info:    network vnet show command OK

**vnet list**

列出所有现有的虚拟网络。

	~$ azure network vnet list
	info:    Executing command network vnet list
	+ Fetching Virtual Networks
	data:    Name        Status   AffinityGroup
	data:    ----------  -------  -------------
	data:    vnet1      Created  AG1
	data:    vnet2      Created  AG1
	data:    vnet3      Created  AG1
	data:    vnet4      Created  AG1
	info:    network vnet list command OK


**network vnet delete &lt;name>**

删除指定的虚拟网络。

	~$ azure network vnet delete opentechvn1
	info:    Executing command network vnet delete
	+ Fetching Network Configuration
	Delete the virtual network opentechvn1 ?  (y/n) y
	+ Deleting the virtual network opentechvn1
	info:    network vnet delete command OK

**network export [file-path]**

若要进行高级网络配置，请在本地导出你的网络配置。注意，导出的网络配置包括 DNS 服务器设置、虚拟网络设置、本地网络站点设置和其他设置。

**network import [file-path]**

导入本地网络配置。

**network dnsserver register [options] &lt;dnsIP>**

注册你计划在网络配置中用来进行名称解析的 DNS 服务器。

	~$ azure network dnsserver register 98.138.253.109 --dns-id FrontEndDnsServer
	info:    Executing command network dnsserver register
	+ Fetching Network Configuration
	+ Updating Network Configuration
	info:    network dnsserver register command OK

**network dnsserver list**

列出在你的网络配置中注册的所有 DNS 服务器。

	~$ azure network dnsserver list
	info:    Executing command network dnsserver list
	+ Fetching Network Configuration
	data:    DNS Server ID         DNS Server IP
	data:    --------------------  --------------
	data:    DNS-bb39b4ac34d66a86  44.55.22.11
	data:    FrontEndDnsServer     98.138.253.109
	info:    network dnsserver list command OK

**network dnsserver unregister [options] &lt;dnsIP>**

从网络配置中删除 DNS 服务器条目。

	~$ azure network dnsserver unregister 77.88.99.11
	info:    Executing command network dnsserver unregister
	+ Fetching Network Configuration
	Delete the DNS server entry dns-4 ( 77.88.99.11 ) %s ? (y/n) y
	+ Deleting the DNS server entry dns-4 ( 77.88.99.11 )
	info:    network dnsserver unregister command OK

<!---HONumber=79-->