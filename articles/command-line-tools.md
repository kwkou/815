<properties linkid="manage-linux-other-resources-command-line-tools" urlDisplayName="Command-Line Tools" pageTitle="适用于 Mac 和 Linux 的 Azure 命令行工具" metaKeywords="Azure command-line, Azure tools Mac, Azure tools Linux" description="了解如何在 Azure 中使用针对 Mac 和 Linux 的命令行工具。" metaCanonical="" services="web-sites,virtual-machines,mobile-services,cloud-services" documentationCenter="" title="" authors="larryfr" solutions="" manager="" editor="" />

# 针对 Mac 和 Linux 的 Azure 命令行工具

此工具提供了用于从 Mac 和 Linux 桌面创建、部署和管理虚拟机和网站的功能。此功能类似于随面向 .NET、Node.JS 和 PHP 的 Azure SDK 一起安装的 Windows PowerShell cmdlet 所提供的功能。

若要在 Mac 上安装该工具，请下载并运行 [Azure SDK 安装程序][Azure SDK 安装程序]。

若要在 Linux 上安装该工具，请安装最新版本的 Node.JS，然后使用 NPM 进行安装：

    npm install azure-cli -g

可选参数显示在方括号中（例如，[参数]）。其他所有参数都是必需的。

除了此处记录的特定于命令的可选参数外，还有三个可用于显示详细输出（例如请求选项和状态代码）的可选参数。-v 参数提供详细输出，而 -vv 参数提供更详细的输出。--json 选项将以原始的 json 格式输出结果。

**目录：**

-   [管理帐户信息和发布设置][管理帐户信息和发布设置]
-   [管理 Azure 虚拟机的命令][管理 Azure 虚拟机的命令]
-   [管理 Azure 虚拟机终结点的命令][管理 Azure 虚拟机终结点的命令]
-   [管理 Azure 虚拟机映像的命令][管理 Azure 虚拟机映像的命令]
-   [管理 Azure 虚拟机数据磁盘的命令][管理 Azure 虚拟机数据磁盘的命令]
-   [管理 Azure 云服务的命令][管理 Azure 云服务的命令]
-   [管理 Azure 证书的命令][管理 Azure 证书的命令]
-   [管理网站的命令][管理网站的命令]
-   [管理工具本地设置][管理工具本地设置]
-   [管理 Service Bus 的命令][管理 Service Bus 的命令]
-   [管理 SQL Database 的命令][管理 SQL Database 的命令]
-   [管理虚拟网络的命令][管理虚拟网络的命令]

## <a name="Manage_your_account_information_and_publish_settings"></a>管理帐户信息和发布设置

该工具使用你的 Azure 订阅信息连接到你的帐户。可从 Azure 门户的发布设置文件中获取此信息，如下所述。稍后可以导入发布设置文件作为永久性本地配置设置，该工具会将此设置用于后续操作。你只需导入你的发布设置一次。

**account download [options]**

此命令启动浏览器以从 Azure 门户下载你的 .publishsettings 文件。

    ~$ azure account download
    info:   Executing command account download
    info:   Launching browser to https://manage.windowsazure.cn/PublishSettings
    help:   Save the downloaded file, then execute the command
    help:   account import <file>
    info:   account download command OK

**account import [options] \<file\>**

此命令导入 publishsettings 文件或证书以便日后可以供该工具使用。

    ~$ azure account import publishsettings.publishsettings
    info:   Importing publish settings file publishsettings.publishsettings
    info:   Found subscription: 3-Month Free Trial
    info:   Found subscription: Pay-As-You-Go
    info:   Setting default subscription to: 3-Month Free Trial
    warn:   The 'publishsettings.publishsettings' file contains sensitive information.
    warn:   Remember to delete it now that it has been imported.
    info:   Account publish settings imported successfully

<div class="dev-callout">

**说明**
publishsettings 文件可以包含有关多个订阅的详细信息（即，订阅名称和 ID）。当你导入 publishsettings 文件时，第一个订阅将用作默认订阅。若要使用其他订阅，请运行以下命令。

`~$ azure config set subscription <other-subscription-id>`

</div>

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

**account set [options] \<subscription\>**

设置当前订阅

### 管理地缘组的命令

**account affinity-group list [options]**

此命令列出你的 Azure 地缘组。

可以在一组虚拟机跨越多台物理计算机时设置地缘组。地缘组指定物理计算机应尽可能彼此接近，从而减少网络延迟。

    ~$ azure account affinity-group list
    + Fetching affinity groups
    data:   Name                                  Label   Location
    data:   ------------------------------------  ------  --------
    data:   535EBAED-BF8B-4B18-A2E9-8755FB9D733F  opentec China East
    info:   account affinity-group list command OK

**account affinity-group create [options] \<name\>**

此命令创建新的地缘组

    ~$ azure account affinity-group create opentec -l "China East"
    info:    Executing command account affinity-group create
    + Creating affinity group
    info:    account affinity-group create command OK

**account affinity-group show [options] \<name\>**

此命令显示地缘组的详细信息

    ~$ azure account affinity-group show opentec
    info:    Executing command account affinity-group show
    + Getting affinity groups
    data:    $ xmlns "http://schemas.microsoft.com/windowsazure"
    data:    $ xmlns:i "http://www.w3.org/2001/XMLSchema-instance"
    data:    Name "opentec"
    data:    Label "b3BlbnRlYw=="
    data:    Description $ i:nil "true"
    data:    Location "China East"
    data:    HostedServices ""
    data:    StorageServices ""
    data:    Capabilities Capability 0 "PersistentVMRole"
    data:    Capabilities Capability 1 "HighMemory"
    info:    account affinity-group show command OK

**account affinity-group delete [options] \<name\>**

此命令将删除指定的地缘组

    ~$ azure account affinity-group delete opentec
    info:    Executing command account affinity-group delete
    Delete affinity group opentec? [y/n] y
    + Deleting affinity group
    info:    account affinity-group delete command OK

### 管理帐户环境的命令

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

## <a name="Commands_to_manage_your_Azure_virtual_machines"></a>管理 Azure 虚拟机的命令

下图显示了如何在 Azure 云服务的生产部署环境中托管 Azure 虚拟机。

![Azure 技术图表][Azure 技术图表]

**create-new** 在 Blob 存储中创建驱动器（即图中的 e:\\）；**attach** 会将已创建但未附加的磁盘附加到虚拟机。

**vm create [options] \<dns-name\> \<image\> \<userName\> [password]**

此命令创建新的 Azure 虚拟机。默认情况下，所创建的每台虚拟机位于其自己的云服务中；但是，你可以使用此处提及的 -c 选项指定将虚拟机添加到现有云服务。

请注意，vm create 命令（如 Azure 门户）只会在生产部署环境中创建虚拟机。目前没有用于在云服务的过渡部署环境中创建虚拟机的选项。请注意，如果你的订阅中尚不包含 Azure 存储帐户，此命令将创建一个。

你可以通过 --location 参数指定位置，也可以通过 --affinity-group 参数指定地缘组。如果没有提供这两个参数，系统将提示你从有效位置列表中提供一个。

所提供密码的长度必须为 8-123 个字符，并且满足你要用于此虚拟机的操作系统的密码复杂度要求。

如果你预计需要使用 SSH 来管理部署的 Linux 虚拟机（通常都是如此），则必须在创建该虚拟机时通过 -e 选项启用 SSH。在创建该虚拟机后无法启用 SSH。

Windows 虚拟机稍后可以通过添加端口 3389 作为终结点来启用 RDP。

此命令支持以下可选参数：

**-c, --connect** 可在托管服务中已创建的部署中创建虚拟机。如果此选项中未使用 -vmname，将自动生成新虚拟机的名称。
**-n, --vm-name** 指定虚拟机的名称。默认情况下，此参数采用托管服务名称。如果未指定 -vmname，将生成 \<service-name\>\<id\> 形式的新虚拟机名称，其中 \<id\> 是服务中现有虚拟机的数量加上 1。例如，如果你使用此命令向拥有一台现有虚拟机的托管服务 MyService 中添加一台新虚拟机，则会将新虚拟机命名为 MyService2。
**-u --blob-url** 指定从中创建虚拟机系统磁盘的 Blob 存储 URL。
**-z, --vm-size** 指定虚拟机的大小。有效值为“特小型”、“小型”、“中型”、“大型”和“特大型”。默认值为“小型”。
**-r** 添加与 Windows 虚拟机的 RDP 连接。
**-e, --ssh** 添加与 Windows 虚拟机的 SSH 连接。
**-t, --ssh-cert** 指定 SSh 证书。
**-s** 订阅
**-o, --community** 指定的映像为社区映像
**-w** 虚拟网络名称
**-l, --location** 指定位置（例如“美国中北部”）。
**-a, --affinity-group** 指定地缘组。
**-w, --virtual-network-name** 指定要在其中添加新虚拟机的虚拟网络。可从 Azure 门户设置和管理虚拟网络。
**-b, --subnet-names** 指定要分配给虚拟机的子网名称。

在此示例中，MSFT\_\_Win2K8R2SP1-120514-1520-141205-01-zh-cn-30GB 是该平台提供的映像。有关操作系统映像的更多信息，请参阅 VM 映像列表。

    ~$ azure vm create my-vm-name MSFT__Windows-Server-2008-R2-SP1.11-29-2011 username --location "East China" -r
    info:   Executing command vm create
    Enter VM 'my-vm-name' password: ************                                     
    info:   vm create command OK

**vm create-from \<dns-name\> \<role-file\>**

此命令从 JSON 角色文件创建新的 Azure 虚拟机。

    ~$ azure vm create-from my-vm example.json
    info:   OK

**vm list [options]**

此命令列出 Azure 虚拟机。-json 选项指定以原始 JSON 格式返回结果。

    ~$ azure vm list
    info:   Executing command vm list
    data:   DNS Name                          VM Name      Status                  
    data:   --------------------------------  -----------  ---------
    data:   my-vm-name.chinacloudapp.cn        my-vm        ReadyRole
    info:   vm list command OK

**vm location list [options]**

此命令列出所有可用的 Azure 帐户位置。

    ~$ azure vm location list
    info:   Executing command vm location list
    data:   Name                   Display Name                                    
    data:   ---------------------  ------------
    data:   Azure         China East     
    info:   account location list command OK

**vm show [options] \<name\>**

此命令显示有关 Azure 虚拟机的详细信息。-json 选项指定以原始 JSON 格式返回结果。

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

**vm delete [options] \<name\>**

此命令删除 Azure 虚拟机。默认情况下，此命令不删除从中创建操作系统磁盘和数据磁盘的 Azure Blob。若要删除 Blob 以及作为其基础的虚拟机，请指定 -b 选项。

    ~$ azure vm delete my-vm 
    info:   Executing command vm delete
    info:   vm delete command OK

**vm start [options] \<name\>**

此命令启动 Azure 虚拟机。

    ~$ azure vm start my-vm
    info:   Executing command vm start
    info:   vm start command OK

**vm restart [options] \<name\>**

此命令重新启动 Azure 虚拟机。

    ~$ azure vm restart my-vm
    info:   Executing command vm restart
    info:   vm restart command OK

**vm shutdown [options] \<name\>**

此命令关闭 Azure 虚拟机。可以使用 -p 选项，以指定在关闭时不释放计算资源。

    ~$ azure vm shutdown my-vm
    info:   Executing command vm shutdown
    info:   vm shutdown command OK  

**vm capture \<vm-name\> \<target-image-name\>**

此命令捕获 Azure 虚拟机映像。

除非虚拟机状态为“已停止”，否则无法捕获虚拟机映像。

    ~$ azure.cmd vm capture my-vm mycaptureimagename --delete
    info:   Executing command vm capture
    + Fetching VMs
    + Capturing VM
    info:   vm capture command OK

**vm export [options] \<vm-name\> \<file-path\>**

此命令将一个 Azure 虚拟机映像导出到文件

    ~$ azure vm export "myvm" "C:\"
    info:    Executing command vm export
    + Getting virtual machines
    + Exporting the VM
    info:   vm export command OK

## <a name="Commands_to_manage_your_Azure_virtual_machine_endpoints"></a>管理 Azure 虚拟机终结点的命令

下图显示了多个虚拟机实例的典型部署的体系结构。请注意，在本示例中，端口 3389 在每台虚拟机上均为打开状态（用于进行 RDP 访问），并且负载平衡器用于将流量路由到虚拟机的每台虚拟机上还有一个内部 IP 地址（例如，168.55.11.1）。此内部 IP 地址也可用于虚拟机之间的通信。

![azurenetworkdiagram][azurenetworkdiagram]

虚拟机的外部请求将通过负载平衡器。因此，不能针对包含多台虚拟机的部署中的特定虚拟机指定请求。对于包含多台虚拟机的部署，必须在虚拟机 (vm-port) 与负载平衡器 (lb-port) 之间配置端口映射。

**vm endpoint create \<vm-name\> \<lb-port\> [vm-port]**

此命令创建虚拟机终结点。你可以使用 -u 或 --enable-direct-server-return 来指定是否在此终结点上启用直接服务器返回，默认情况下禁用。

    ~$ azure vm endpoint create my-vm 8888 8888
    azure vm endpoint create my-vm 8888 8888
    info:   Executing command vm endpoint create
    + Fetching VM
    + Reading network configuration
    + Updating network configuration
    info:   vm endpoint create command OK

\*\*vm endpoint create-multiple [options] \<vm-name\> \<lb-port\>[:\<vm-port\>[:\<protocol\>[:\<lb-set-name\>[:\<prob-protocol\>:\<lb-prob-port\>[:\<prob-path\>]]]]] ]{1-\*}\*\*

创建多个 VM 终结点。你可以使用 -u 或 --enable-direct-server-return 来指定是否在此终结点上启用直接服务器返回，默认情况下禁用。

**vm endpoint delete \<vm-name\> \<lb-port\>**

此命令删除虚拟机终结点。

    ~$ azure vm endpoint delete my-vm 8888
    azure vm endpoint delete my-vm 8888
    info:   Executing command vm endpoint delete
    + Fetching VM
    + Reading network configuration
    + Updating network configuration
    info:   vm endpoint delete command OK

**vm endpoint list \<vm-name\>**

此命令列出所有虚拟机终结点。-json 选项指定以原始 JSON 格式返回结果。

    ~$ azure vm endpoint list my-linux-vm
    data:   Name  External Port  Local Port                                        
    data:   ----  -------------  ----------
    data:   ssh   22             22

**vm endpoint update [options] \<vm-name\> \<endpoint-name\>**

此命令使用这些选项将 VM 终结点更新为新值。

    -n, --endpoint-name <name>          the new endpoint name
    -t, --lb-port <port>                the new load balancer port
    -t, --vm-port <port>                the new local port port
    -o, --endpoint-protocol <protocol>  the new transport layer protocol for port (tcp or udp) 

**vm endpoint show [options] \<vm-name\>**

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

## <a name="Commands_to_manage_your_Azure_virtual_machine_images"></a>管理 Azure 虚拟机映像的命令

虚拟机映像是所捕获的、可根据需要进行复制的已配置虚拟机。

**vm image list [options]**

此命令获取虚拟机映像的列表。有三种类型的映像：Microsoft 创建的映像（以“MSFT”作为前缀）、第三方创建的映像（通常以供应商的名称作为前缀）以及你创建的映像。若要创建映像，你可以捕获现有虚拟机或从上载到 Blob 存储的自定义 .vhd 创建映像。有关使用自定义 .vhd 的详细信息，请参阅 VM 映像创建。
-json 选项指定以原始 JSON 格式返回结果。

    ~$ azure vm image list
    data:   Name                                                                   Category   OS
    data:   ---------------------------------------------------------------------  ---------  -------
    data:   CANONICAL__Canonical-Ubuntu-12-04-20120519-2012-05-19-zh-cn-30GB.vhd   Canonical  Linux
    data:   MSFT__Windows-Server-2008-R2-SP1.11-29-2011                            Microsoft  Windows
    data:   MSFT__Windows-Server-2008-R2-SP1-with-SQL-Server-2012-Eval.11-29-2011  Microsoft  Windows
    data:   MSFT__Windows-Server-8-Beta.zh-cn.30GB.2012-03-22                      Microsoft  Windows
    data:   MSFT__Windows-Server-8-Beta.2-17-2012                                  Microsoft  Windows
    data:   MSFT__Windows-Server-2008-R2-SP1.zh-cn.30GB.2012-3-22                  Microsoft  Windows
    data:   OpenLogic__OpenLogic-CentOS-62-20120509-zh-cn-30GB.vhd                 OpenLogic  Linux
    data:   SUSE__SUSE-Linux-Enterprise-Server-11SP2-20120521-zh-cn-30GB.vhd       SUSE       Linux
    data:   SUSE__OpenSUSE64121-03192012-zh-cn-15GB.vhd                            SUSE       Linux
    data:   WIN2K8-R2-WINRM                                                        User       Windows
    info:   vm image list command OK   

**vm image show [options] \<name\>**

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

**vm image delete [options] \<name\>**

此命令删除虚拟机映像。

    ~$ azure vm image delete my-vm-image
    info:   Executing command vm image delete
    info:   VM image deleted: my-vm-image                                         
    info:   vm image delete command OK

**vm image create \<name\> [source-path]**

此命令创建虚拟机映像。你的自定义 .vhd 文件将上载到 Blob 存储，然后从该位置创建虚拟机映像。然后可使用此虚拟机映像创建虚拟机。Location 和 OS 参数是必需的。

某些系统会施加每进程文件描述符限制。如果超出此限制，工具将显示文件描述符限制错误。你可以使用 -p \<number\> 参数再次运行该命令，以减少最大并行上载数。默认的最大并行上载数为 96。

    ~$ azure vm image create mytestimage ./Sample.vhd -o windows -l "West US"
    info:   Executing command vm image create
    + Retrieving storage accounts
    info:   VHD size : 13 MB
    info:   Uploading 13312.5 KB
    Requested:100.0% Completed:100.0% Running: 105 Time:    8s Speed:  1721 KB/s
    info:   http://myaccount.blob.core.azure.com/vm-images/Sample.vhd is uploaded successfully
    info:   vm image create command OK

## <a name="Commands_to_manage_your_Azure_virtual_machine_data_disks"></a>管理 Azure 虚拟机数据磁盘的命令

数据磁盘是 Blob 存储中可供虚拟机使用的 .vhd 文件。有关如何将数据磁盘部署到 Blob 存储的更多信息，请参阅前面所示的 Azure 技术图表。

用于附加数据磁盘的命令（azure vm disk attach 和 azure vm disk attach-new）会根据 SCSI 协议的要求向附加的数据磁盘分配逻辑单元号 (LUN)。将向附加到虚拟机的第一个数据磁盘分配 LUN 0，向下一个分配 LUN 1，依此类推。

当你使用 azure vm disk detach 命令分离数据磁盘时，请使用 \<lun\> 参数指明要分离的磁盘。

<div class="dev-callout">

**说明**
请注意，应始终按相反的顺序分离数据磁盘，即，从已分配的编号最高的 LUN 开始。Linux SCSI 层不支持在仍附加有编号较高的 LUN 时分离编号较低的 LUN。例如，你不应在仍附加有 LUN 1 的情况下分离 LUN 0。

</div>

**vm disk show [options] \<name\>**

此命令显示有关 Azure 磁盘的详细信息。

    ~$ azure vm disk show anucentos-anucentos-0-20120524070008
    info:   Executing command vm disk show
    data:   AttachedTo DeploymentName "mycentos"
    data:   AttachedTo HostedServiceName "myanucentos"
    data:   AttachedTo RoleName "myanucentos"
    data:   OS "Linux"
    data:   Location "Azure Preview"
    data:   LogicalDiskSizeInGB "30"
    data:   MediaLink "http://myaccout.blob.core.chinacloudapi.cn/vhds/ericdc-ericdc-2014-05-31.vhd"
    data:   Name "ericdc-ericdc-2014-05-31.vhd"
    data:   SourceImageName "OpenLogic__OpenLogic-CentOS-62-20120509-zh-cn-30GB.vhd"
    info:   vm disk show command OK

**vm disk list [options] [vm-name]**

此命令列出 Azure 磁盘，或者附加到指定虚拟机的磁盘。如果此命令运行时带虚拟机名称参数，将返回附加到该虚拟机的所有磁盘。Lun 1 是使用虚拟机创建的，而列出的其他任何磁盘都将单独附加。

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

**vm disk delete [options] \<name\>**

此命令从个人存储库中删除 Azure 磁盘。在删除磁盘之前必须从虚拟机中分离该磁盘。

    ~$ azure vm disk delete mycentos-mycentos-2-20120525055052
    info:   Executing command vm disk delete
    info:   Disk deleted: mycentos-mycentos-2-20120525055052                  
    info:   vm disk delete command OK

**vm disk create \<name\> [source-path]**

此命令上载和注册 Azure 磁盘。必须指定 --blob-url、--location 或 --affinity-group。如果将此命令与 [source-path] 结合使用，将上载指定的 .vhd 文件并创建新映像。然后你可以使用 vm disk attach 将此映像附加到虚拟机。

某些系统会施加每进程文件描述符限制。如果超出此限制，工具将显示文件描述符限制错误。你可以使用 -p \<number\> 参数再次运行该命令，以减少最大并行上载数。默认的最大并行上载数为 96。

    ~$ azure vm disk create my-data-disk ~/test.vhd --location "Western US"
    info:   Executing command vm disk create
    info:   VHD size : 10 MB                                                       
    info:   Uploading 10240.5 KB
    Requested:100.0% Completed:100.0% Running:  81 Time:   11s Speed:   952 KB/s 
    info:   http://account.blob.core.chinacloudapi.cn/disks/test.vhd is uploaded successfully
    info:   vm disk create command OK

**vm disk upload [options] \<source-path\> \<blob-url\> \<storage-account-key\>**

此命令允许你上载 VM 磁盘

    ~$ azure vm disk upload "http://sourcestorage.blob.core.chinacloudapi.cn/vhds/sample.vhd" "http://destinationstorage.blob.core.chinacloudapi.cn/vhds/sample.vhd" "DESTINATIONSTORAGEACCOUNTKEY"
    info:   Executing command vm disk upload                                                      
    info:   Uploading 12351.5 KB
    info:   vm disk upload command OK

**vm disk attach \<vm-name\> \<disk-image-name\>**

此命令将 Blob 存储中的现有磁盘附加到云服务中部署的现有虚拟机。

    ~$ azure vm disk attach my-vm my-vm-my-vm-2-201242418259
    info:   Executing command vm disk attach
    info:   vm disk attach command OK

**vm disk attach-new \<vm-name\> \<size-in-gb\> [blob-url]**

此命令将数据磁盘附加到 Azure 虚拟机。在本示例中，20 GB 是要附加的新磁盘的大小。你可以选择使用 Blob URL 作为显式指定要创建的目标 Blob 的最后一个参数。如果你不指定 Blob URL，将自动生成一个 Blob 对象。

    ~$ azure vm disk attach-new nick-test36 20 http://nghinazz.blob.core.chinacloudapi.cn/vhds/vmdisk1.vhd
    info:   Executing command vm disk attach-new
    info:   vm disk attach-new command OK  

**vm disk detach \<vm-name\> \<lun\>**

此命令将数据磁盘与 Azure 虚拟机分离。\<lun\> 用于标识要分离的磁盘。若要获取在分离某个磁盘之前与该磁盘关联的磁盘的列表，请使用 vm disk-list \<vm-name\>。

    ~$ azure vm disk detach my-vm 2
    info:   Executing command vm disk detach
    info:   vm disk detach command OK

## <a name="Commands_to_manage_your_Azure_cloud_services"></a>管理 Azure 云服务的命令

Azure 云服务是托管在 Web 角色和辅助角色上的应用程序和服务。以下命令可用于管理 Azure 云服务。

**service create [options] \<serviceName\>**

此命令创建新的云服务

    ~$ azure service create newservicemsopentech
    info:    Executing command service create
    + Getting locations
    help:    Location:
      1) China East
      2) China North
      : 1
    + Creating cloud service
    data:    Cloud service name newservicemsopentech
    info:    service create command OK

**service show [options] \<serviceName\>**

此命令显示 Azure 云服务的详细信息

    ~$ azure service show newservicemsopentech
    info:    Executing command service show
    + Getting cloud service
    data:    Name newservicemsopentech
    data:    Url https://management.core.windows.net/9e672699-1055-41ae-9c36-e85152f2e352/services/hostedservices/newservicemsopentech
    data:    Properties location China East
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

**service delete [options] \<name\>**

此命令删除 Azure 云服务。

    ~$ azure cloud-service delete myservice
    info:   Executing command cloud-service delete myservice 
    info:   cloud-service delete command OK

## <a name="Commands_to_manage_your_Azure_certificates"></a>管理 Azure 证书的命令

Azure 证书是已连接到你的 Azure 帐户的证书（即 SSL 证书）。

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

**service cert create \<dns-prefix\> \<file\> [password]**

此命令上载证书。将没有密码保护的证书的密码提示保留为空。

    ~$ azure service cert create nghinazz ~/publishSet.pfx
    info:   Executing command service cert create
    Cert password: 
    + Creating certificate                                                         
    info:   service cert create command OK

**service cert delete [options] \<thumbprint\>**

此命令删除证书。

    ~$ azure service cert delete 262DBF95B5E61375FA27F1E74AC7D9EAE842916C
    info:   Executing command service cert delete
    + Deleting certificate                                                         
    info:   nghinazz : cert deleted
    info:   service cert delete command OK

## <a name="Commands_to_manage_your_web_sites"></a>管理网站的命令

Azure 网站是可通过 URI 访问的 Web 配置。网站托管在虚拟机中，但你无需自己考虑创建和部署虚拟机的详细步骤。这些详细步骤将由 Azure 为你完成。

**site list [options]**

此命令列出你的网站。

    ~$ azure site list
    info:   Executing command site list
    data:   Name            State    Host names                                        
    data:   --------------  -------  --------------------------------------------------
    data:   mongosite       Running  mongosite.antdf0.antares.windows.net     
    data:   myphpsite       Running  myphpsite.antdf0.antares.windows.net     
    data:   mydrupalsite36  Running  mydrupalsite36.antdf0.antares.windows.net
    info:   site list command OK

**site set [options] [name]**

此命令将设置你的网站 [名称] 的配置选项

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

此命令创建新的网站和本地目录。

    ~$ azure site create mysite
    info:   Executing command site create
    info:   Using location northeuropewebspace
    info:   Creating a new web site
    info:   Created web site at  mysite.antdf0.antares.windows.net
    info:   Initializing repository
    info:   Repository initialized
    info:   site create command OK

<div class="dev-callout">

**说明**
站点名称必须是唯一的。你无法创建与现有站点具有相同 DNS 名称的站点。

</div>

**site portal [options] [name]**

此命令在浏览器中打开门户以便你能管理你的网站。

    ~$ azure site portal mysite
    info:   Executing command site portal
    info:   Launching browser to https://windows.azure.net/#Workspaces/WebsiteExtension/Website/mysite/dashboard
    info:   site portal command OK

**site browse [options] [name]**

此命令在浏览器中打开你的网站。

    ~$ azure site browse mysite
    info:   Executing command site browse
    info:   Launching browser to http://mysite.antdf0.antares-test.windows-int.net
    info:   site browse command OK

**site show [options] [name]**

此命令显示网站的详细信息。

    ~$ azure site show mysite
    info:   Executing command site show
    info:   Showing details for site
    data:   Site AdminEnabled true
    data:   Site HostNames mysite.antdf0.antares-test.windows-int.net
    data:   Site Name mysite
    data:   Site Owner 00060000814EDDEE
    data:   Site RepositorySiteName mysite
    data:   Site SelfLink https://s1.api.antdf0.antares.windows.net:454/subscriptions/444e62ff-4c5f-4116-a695-5c803ed584a5/webspaces/northeuropewebspace/sites/mysite
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

此命令删除网站。

    ~$ azure site delete mysite
    info:   Executing command site delete
    info:   Deleting site mysite
    info:   Site mysite has been deleted
    info:   site delete command OK

**site start [options] [name]**

此命令启动网站。

    ~$ azure site start mysite
    info:   Executing command site start
    info:   Starting site mysite
    info:   Site mysite has been started
    info:   site start command OK

**site stop [options] [name]**

此命令停止网站。

    ~$ azure site stop mysite
    info:   Executing command site stop
    info:   Stopping site mysite
    info:   Site mysite has been stopped
    info:   site stop command OK

**site location list [options]**

此命令将列出你的网站位置

    ~$ azure site location list
    info:    Executing command site location list
    + Getting locations
    data:    Name
    data:    ----------------
    data:    China North
    data:    China East
    info:    site location list command OK

### 管理网站应用程序设置的命令

**site appsetting list [options] [name]**

此命令将列出添加到网站的应用程序设置

    ~$ azure site appsetting list
    info:    Executing command site appsetting list
    Web site name: mydemosite
    + Getting sites
    + Getting site config information
    data:    Name  Value
    data:    ----  -----
    data:    test  value
    info:    site appsetting list command OK

**site appsetting add [options] \<keyvaluepair\> [name]**

此命令将应用程序设置作为键值对添加到你的网站

    ~$ azure site appsetting add test=value
    info:    Executing command site appsetting add
    Web site name: mydemosite
    + Getting sites
    + Getting site config information
    + Updating site config information
    info:    site appsetting add command OK

**site appsetting delete [options] \<key\> [name]**

此命令从网站中删除指定应用程序设置

    ~$ azure site appsetting delete test
    info:    Executing command site appsetting delete
    Web site name: mydemosite
    + Getting sites
    + Getting site config information
    Delete application setting test? [y/n] y
    + Updating site config information
    info:    site appsetting delete command OK

**site appsetting show [options] \<key\> [name]**

此命令显示指定应用程序设置的详细信息

    ~$ azure site appsetting show test
    info:    Executing command site appsetting show
    Web site name: mydemosite
    + Getting sites
    + Getting site config information
    data:    Value:  value
    info:    site appsetting show command OK

### 管理网站证书的命令

**site cert list [options] [name]**

此命令显示网站证书的列表

    ~$ azure site cert list
    info:    Executing command site cert list
    Web site name: mydemosite
    + Getting sites
    + Getting site information
    data:    Subject                       Expiration Date                    Thumbprint
    data:    ----------------------------  -----------------------------------------
    ----------------  ----------------------------------------
    data:    *.msopentech.com              Fri Nov 28 2014 09:49:57 GMT-0800 (Pacific Standard Time)  A40E82D3DC0286D1F58650E570ECF8224F69A148
    data:    msopentech.azurewebsites.net  Fri Jun 19 2015 11:57:32 GMT-0700 (Pacific Daylight Time)  CE1CD6538852BF7A5DC32001C2E26A29B541F0E8
    info:    site cert list command OK

**site cert add [options] \<certificate-path\> [name]**

**site cert delete [options] \<thumbprint\> [name]**

**site cert show [options] \<thumbprint\> [name]**

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

### 管理网站连接字符串的命令

**site connectionstring list [options] [name]**

**site connectionstring add [options] \<connectionname\> \<value\> \<type\> [name]**

**site connectionstring delete [options] \<connectionname\> [name]**

**site connectionstring show [options] \<connectionname\> [name]**

### 管理网站默认文档的命令

**site defaultdocument list [options] [name]**

**site defaultdocument add [options] \<document\> [name]**

**site defaultdocument delete [options] \<document\> [name]**

### 管理网站部署的命令

**site deployment list [options] [name]**

**site deployment show [options] \<commitId\> [name]**

**site deployment redeploy [options] \<commitId\> [name]**

**site deployment github [options] [name]**

**site deployment user set [options] [username] [pass]**

### 管理网站域的命令

**site domain list [options] [name]**

**site domain add [options] \<dn\> [name]**

**site domain delete [options] \<dn\> [name]**

### 管理网站处理程序映射的命令

**site handler list [options] [name]**

**site handler add [options] \<extension\> \<processor\> [name]**

**site handler delete [options] \<extension\> [name]**

### 管理网站诊断的命令

**site log download [options] [name]**

下载网站诊断的 .zip 文件

    ~$ azure site log download
    info:    Executing command site log download
    Web site name: mydemosite
    + Getting sites
    + Getting site information
    + Downloading diagnostic log to diagnostics.zip
    info:    site log download command OK

**site log tail [options] [name]**

此命令将用户终端连接到日志流式处理服务

    ~$ azure site log tail
    info:    Executing command site log tail
    Web site name: mydemosite
    + Getting sites
    + Getting site information
    2013-11-19T17:24:17  Welcome, you are now connected to log-streaming service.

**site log set [options] [name]**

此命令配置诊断选项

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

### 管理网站存储库的命令

**site repository branch [options] \<branch\> [name]**

**site repository delete [options] [name]**

**site repository sync [options] [name]**

### 管理网站缩放的命令

**site scale mode [options] \<mode\> [name]**

**site scale instances [options] \<instances\> [name]**

<!-- ##<a name="Commands_to_manage_mobile_services"></a>Commands to manage Azure Mobile Services  Azure Mobile Services brings together a set of Azure services that enable backend capabilities for your apps. Mobile Services commands are divided into the following categories:  + [Commands to manage mobile service instances](#Mobile_Services) + [Commands to manage mobile service configuration](#Mobile_Configuration) + [Commands to manage mobile service tables](#Mobile_Tables) + [Commands to manage mobile service scripts](#Mobile_Scripts) + [Commands to manage scheduled jobs](#Mobile_Jobs) + [Commands to scale a mobile service](#Mobile_Scale)  The following options apply to most Mobile Services commands:  + **-h** or **--help**: Display output usage information. + **-s `<id>`** or **--subscription `<id>`**: Use a specific subscription, specified as `<id>`. + **-v** or **--verbose**: Write verbose output. + **--json**: Write JSON output.  ###<a name="Mobile_Services"></a>Commands to manage mobile service instances  **mobile locations [options]**  This command lists geographic locations supported by Mobile Services.      ~$ azure mobile locations     info:    Executing command mobile locations     info:    East US (default)     info:    West US             info:    North Europe  **mobile create [options] [servicename] [sqlAdminUsername] [sqlAdminPassword]**  This command creates a mobile service along with a SQL Database and server.      ~$ azure mobile create todolist your_login_name Secure$Password     info:    Executing command mobile create     + Creating mobile service     info:    Overall application state: Healthy     info:    Mobile service (todolist) state: ProvisionConfigured     info:    SQL database (todolist_db) state: Provisioned     info:    SQL server (e96ean1c6v) state: ProvisionConfigured     info:    mobile create command OK  This command supports the following additional options:  + **-r `<sqlServer>`**  or **--sqlServer `<sqlServer>`**:  Use an existing SQL Database server, specified as `<sqlServer>`. + **-d `<sqlDb>`** or **--sqlDb `<sqlDb>`**: Use existing SQL database, specified as `<sqlDb>`. + **-l `<location>`** or **--location `<location>`**: Create the service in a specific location, specified as `<location>`. Run azure mobile locations to get available locations.   + **--sqlLocation `<location>`**: Create the SQL server in a specific `<location>`; defaults to the location of the mobile service.  **mobile delete [options] [servicename]**  This command deletes a mobile service along with its SQL Database and server.      ~$ azure mobile delete todolist -a -q     info:    Executing command mobile delete     data:    Mobile service todolist     data:    SQL database todolistAwrhcL60azo1C401     data:    SQL server fh1kvbc7la     + Deleting mobile service     info:    Deleted mobile service     + Deleting SQL server     info:    Deleted SQL server     + Deleting mobile application     info:    Deleted mobile application     info:    mobile delete command OK  This command supports the following additional options:  + **-d** or **--deleteData**: Delete all data from this mobile service from the database. + **-a** or **--deleteAll**: Delete the SQL Database and server. + **-q or **--quiet**: Do not prompt for confirmation. Use this option in automated scripts.  **mobile list [options]**  This command lists your mobile services.      ~$ azure mobile list     info:    Executing command mobile list     data:    Name          State  URL     data:    ------------  -----  --------------------------------------     data:    todolist      Ready  https://todolist.azure-mobile.net/     data:    mymobileapp   Ready  https://mymobileapp.azure-mobile.net/     info:    mobile list command OK  **mobile show [options] [servicename]**  This command displays details about a mobile service.      ~$ azure mobile show todolist     info:    Executing command mobile show     + Getting information     info:    Mobile application     data:    status Healthy     data:    Mobile service name todolist     data:    Mobile service status ProvisionConfigured     data:    SQL database name todolistAwrhcL60azo1C401     data:    SQL database status Linked     data:    SQL server name fh1kvbc7la     data:    SQL server status Linked     info:    Mobile service     data:    name todolist     data:    state Ready     data:    applicationUrl https://todolist.azure-mobile.net/     data:    applicationKey XXXXXXXXXXXXXXXXXXXXXXXXXXXXXX     data:    masterKey XXXXXXXXXXXXXXXXXXXXXXXXXXXXXX     data:    webspace WESTUSWEBSPACE     data:    region West US     data:    tables TodoItem     info:    mobile show command OK   **mobile restart [options] [servicename]**  This command restarts a mobile service instance.      ~$ azure mobile restart todolist     info:    Executing command mobile restart     + Restarting mobile service     info:    Service was restarted.     info:    mobile restart command OK  **mobile log [options] [servicename]**  This command returns mobile service logs, filtering out all log types but `error`.      ~$ azure mobile log todolist -t error     info:    Executing command mobile log     data:     data:    timeCreated 2013-01-07T16:04:43.351Z     data:    type error     data:    source /scheduler/TestingLogs.js     data:    message This is an error.     data:     info:    mobile log command OK  This command supports the following additional options:  + **-r `<query>`** or **--query `<query>`**: Executes the specified log query. + **-t `<type>`** or **--type `<type>`**:  Filter the returned logs by entry `<type>`, which can be `information`, `warning`, or `error`. + **-k `<skip>`** or **--skip `<skip>`**: Skips the number of rows specified by `<skip>`. + **-p `<top>`** or **--top `<top>`**: Returns a specific number of rows, specified by `<top>`.  <div class="dev-callout"><b>Note</b>    <p>The **--query** parameter takes precedence over **--type**, **--skip**, and **--top**.</p> </div>  **mobile key regenerate [options] [servicename] [type]**  This command regenerates the mobile service application key.      ~$ azure mobile key regenerate todolist application     info:    Executing command mobile key regenerate     info:    New application key is SmLorAWVfslMcOKWSsuJvuzdJkfUpt40     info:    mobile key regenerate command OK  Key types are `master` and `application`.  <div class="dev-callout"><b>Note</b>    <p>When you regenerate keys, clients that use the old key may be unable to access your mobile service. When you regenerate the application key, you should update your app with the new key value. </p> </div>   ###<a name="Mobile_Configuration"></a>Commands to manage mobile service configuration  **mobile config list [options] [servicename]**  This command lists configuration options for a mobile service.      ~$ azure mobile config list todolist     info:    Executing command mobile config list     + Getting mobile service configuration     data:    dynamicSchemaEnabled true     data:    microsoftAccountClientSecret Not configured     data:    microsoftAccountClientId Not configured     data:    microsoftAccountPackageSID Not configured     data:    facebookClientId Not configured     data:    facebookClientSecret Not configured     data:    twitterClientId Not configured     data:    twitterClientSecret Not configured     data:    googleClientId Not configured     data:    googleClientSecret Not configured     data:    apnsMode none     data:    apnsPassword Not configured     data:    apnsCertifcate Not configured     info:    mobile config list command OK  **mobile config get [options] [servicename] [key]**  This command gets a specific configuration option for a mobile service, in this case dynamic schema.      ~$ azure mobile config get todolist dynamicSchemaEnabled     info:    Executing command mobile config get     data:    dynamicSchemaEnabled true     info:    mobile config get command OK  **mobile config set [options] [servicename] [key] [value]**  This command sets a specific configuration option for a mobile service, in this case dynamic schema.      ~$ azure mobile config set todolist dynamicSchemaEnabled false     info:    Executing command mobile config set     info:    mobile config set command OK   ###<a name="Mobile_Tables"></a>Commands to manage mobile service tables  **mobile table list [options] [servicename]**  This command lists all tables in your mobile service.      ~$azure mobile table list todolist     info:    Executing command mobile table list     data:    Name      Indexes  Rows     data:    --------  -------  ----     data:    Channel   1        0     data:    TodoItem  1        0     info:    mobile table list command OK  **mobile table show [options] [servicename] [tablename]**  This command shows returns details about a specific table.      ~$azure mobile table show todolist     info:    Executing command mobile table show     + Getting table information     info:    Table statistics:     data:    Number of records 5     info:    Table operations:     data:    Operation  Script       Permissions     data:    ---------  -----------  -----------     data:    insert     1900 bytes   user     data:    read       Not defined  user     data:    update     Not defined  user     data:    delete     Not defined  user     info:    Table columns:     data:    Name  Type           Indexed     data:    ----  -------------  -------     data:    id    bigint(MSSQL)  Yes     data:    text      string     data:    complete  boolean     info:    mobile table show command OK  **mobile table create [options] [servicename] [tablename]**  This command creates a table.      ~$azure mobile table create todolist Channels     info:    Executing command mobile table create     + Creating table     info:    mobile table create command OK  This command supports the following additional option:  + **-p `<permissions>`** or **--permissions `<permissions>`**: Comma-delimited list of `<operation>`=`<permission>` pairs, where `<operation>` is `insert`, `read`, `update`, or `delete` and `<permissions>` is `public`, `application` (default), `user`, or `admin`. + **--integerId**: Create a table with an integer id column.  **mobile data read [options] [servicename] [tablename] [query]**  This command reads data from a table.      ~$azure mobile data read todolist TodoItem     info:    Executing command mobile data read     data:    id  text     complete     data:    --  -------  --------     data:    1   item #1  false     data:    2   item #2  true     data:    3   item #3  false     data:    4   item #4  true     info:    mobile data read command OK  This command supports the following additional options:  + **-k `<skip>`** or **--skip `<skip>`**: Skips the number of rows specified by `<skip>`. + **-t `<top>`** or **--top `<top>`**: Returns a specific number of rows, specified by `<top>`. + **-l** or **--list**: Returns data in a list format.  **mobile table update [options] [servicename] [tablename]**  This command changes delete permissions on a table to administrators only.      ~$azure mobile table update todolist Channels -p delete=admin     info:    Executing command mobile table update     + Updating permissions     info:    Updated permissions     info:    mobile table update command OK  This command supports the following additional options:  + **-p `<permissions>`** or **--permissions `<permissions>`**: Comma-delimited list of `<operation>`=`<permission>` pairs, where `<operation>` is `insert`, `read`, `update`, or `delete` and `<permissions>` is `public`, `application` (default), `user`, or `admin`. + **--deleteColumn `<columns>`**: Comma-delimited list of columns to delete, as `<columns>`. + **-q** or **--quiet**: Deletes columns without prompting for confirmation. + **--addIndex `<columns>`**: Comma-delimited list of columns to include in the index. + **--deleteIndex `<columns>`**: Comma-delimited list of columns to exclude from the index.  **mobile table delete [options] [servicename] [tablename]**  This command deletes a table.      ~$azure mobile table delete todolist Channels     info:    Executing command mobile table delete     Do you really want to delete the table (yes/no): yes     + Deleting table     info:    mobile table delete command OK  Specify the -q parameter to delete the table without confirmation. Do this to prevent blocking of automation scripts.  **mobile data truncate [options] [servicename] [tablename]**  This commands removes all rows of data from the table.      ~$azure mobile data truncate todolist TodoItem     info:    Executing command mobile data truncate     info:    There are 7 data rows in the table.     Do you really want to delete all data from the table? (y/n): y     info:    Deleted 7 rows.     info:    mobile data truncate command OK   ###<a name="Mobile_Scripts"></a>Commands to manage scripts  Commands in this section are used to manage the server scripts that belong to a mobile service. For more information, see [Work with server scripts in Mobile Services](http://www.windowsazure.com/zh-cn/develop/mobile/how-to-guides/work-with-server-scripts/).  **mobile script list [options] [servicename]**  This command lists registered scripts, including both table and scheduler scripts.      ~$azure mobile script list todolist     info:    Executing command mobile script list     + Getting script information     info:    Table scripts     data:    Name                   Size     data:    ---------------------  ----     data:    table/TodoItem.delete  256     data:    table/Devices.insert   1660     error:   Unable to get shared scripts     info:    Scheduler scripts     data:    Name                 Status     Interval   Last run   Next run     data:    -------------------  ---------  ---------  ---------  ---------     data:    scheduler/undefined  undefined  undefined  undefined  undefined     data:    scheduler/undefined  undefined  undefined  undefined  undefined     info:    mobile script list command OK  **mobile script upload [options] [servicename] [scriptname]**  This command uploads a new script named `todoitem.insert.js` from the `table` subfolder.      ~$azure mobile script upload todolist table/todoitem.insert.js     info:    Executing command mobile script upload     info:    mobile script upload command OK  The name of the file must be composed from the table and operation names, and it must be located in the table subfolder relative to the location where the command is executed. You can also use the **-f `<file>`** or **--file `<file>`** parameter to specify a different filename and path to the file that contains the script to register.  **mobile script download [options] [servicename] [scriptname]**  This command downloads the insert script from the TodoItem table to a file named `todoitem.insert.js` in the `table` subfolder.      ~$azure mobile script download todolist table/todoitem.insert.js     info:    Executing command mobile script download     info:    Saved script to ./table/todoitem.insert.js     info:    mobile script download command OK  This command supports the following additional options:  + **-p `<path>`** or **--path `<path>`**: The location in the file in which to save the script, where the current working directory is the default. + **-f `<file>`** or **--file `<file>`**: The name of the file in which to save the script. + **-o** or **--override**: Overwrite an existing file. + **-c** or **--console**: Write the script to the console instead of to a file.  **mobile script delete [options] [servicename] [scriptname]**  This command removes the existing insert script from the TodoItem table.      ~$azure mobile script delete todolist table/todoitem.insert.js     info:    Executing command mobile script delete     info:    mobile script delete command OK  ###<a name="Mobile_Jobs"></a>Commands to manage scheduled jobs  Commands in this section are used to manage scheduled jobs that belong to a mobile service. For more information, see [Schedule jobs](http://msdn.microsoft.com/zh-cn/library/windowsazure/jj860528.aspx).  **mobile job list [options] [servicename]**  This command lists scheduled jobs.      ~$azure mobile job list todolist     info:    Executing command mobile job list     info:    Scheduled jobs     data:    Job name    Script name           Status    Interval     Last run              Next run     data:    ----------  --------------------  --------  -----------  --------------------  --------------------     data:    getUpdates  scheduler/getUpdates  enabled   15 [minute]  2013-01-14T16:15:00Z  2013-01-14T16:30:00Z     info:    You can manipulate scheduled job scripts using the 'azure mobile script' command.     info:    mobile job list command OK  **mobile job create [options] [servicename] [jobname]**  This command creates a new job named `getUpdates` that is scheduled to run hourly.      ~$azure mobile job create -i 1 -u hour todolist getUpdates      info:    Executing command mobile job create     info:    Job was created in disabled state. You can enable the job using the 'azure mobile job update' command.     info:    You can manipulate the scheduled job script using the 'azure mobile script' command.     info:    mobile job create command OK  This command supports the following additional options:  + **-i `<number>`** or **--interval `<number>`**: The job interval, as an integer; the default value is `15`. + **-u `<unit>`** or **--intervalUnit `<unit>`**: The unit for the _interval_, which can be one of the following values:      + **minute** (default)     + **hour**     + **day**     + **month**      + **none** (on-demand jobs) + **-t `<time>`** **--startTime `<time>`** The start time of the first run for the script, in ISO format; the default value is `now`.  <div class="dev-callout"><b>Note</b>    <p>New jobs are created in a disabled state because a script must still be uploaded. Use the <strong>mobile script upload</strong> command to upload a script and the <strong>mobile job update</strong> command to enable the job.</p> </div>   **mobile job update [options] [servicename] [jobname]**  The following command enables the disabled `getUpdates` job.      ~$azure mobile job update -a enabled todolist getUpdates      info:    Executing command mobile job update     info:    mobile job update command OK  This command supports the following additional options:  + **-i `<number>`** or **--interval `<number>`**: The job interval, as an integer; the default value is `15`. + **-u `<unit>`** or **--intervalUnit `<unit>`**: The unit for the _interval_, which can be one of the following values:      + **minute** (default)     + **hour**     + **day**     + **month**      + **none** (on-demand jobs) + **-t `<time>`** **--startTime `<time>`** The start time of the first run for the script, in ISO format; the default value is `now`. + **-a `<status>`** or **--status `<status>`**: The job status, which can be either `enabled` or `disabled`.  **mobile job delete [options] [servicename] [jobname]**  This command removes the getUpdates scheduled job from the TodoList server.      ~$azure mobile job delete todolist getUpdates     info:    Executing command mobile job delete     info:    mobile job delete command OK  <div class="dev-callout"><b>Note</b>    <p>Deleting a job also deletes the uploaded script.</p> </div>   ###<a name="Mobile_Scale"></a>Commands to scale a mobile service  Commands in this section are used to scale a mobile service. For more information, see [Scaling a mobile service](http://msdn.microsoft.com/zh-cn/library/windowsazure/jj193178.aspx).  **mobile scale show [options] [servicename]**  This command displays scale information, including current compute mode and number of instances.      ~$azure mobile scale show todolist     info:    Executing command mobile scale show     data:    webspace WESTUSWEBSPACE     data:    computeMode Free     data:    numberOfInstances 1     info:    mobile scale show command OK  **mobile scale change [options] [servicename]**  This command changes the scale of the mobile service from free to premium mode.      ~$azure mobile scale change -c Reserved -i 1 todolist     info:    Executing command mobile scale change     + Rescaling the mobile service     info:    mobile scale change command OK  This command supports the following additional options:  + **-c `<mode>`** or **--computeMode `<mode>`**: The compute mode must be either `Free` or `Reserved`. + **-i `<count>` or **--numberOfInstances `<count>`**: The number of instances used when running in reserved mode.  <div class="dev-callout"><b>Note</b>    <p>When you set compute mode to `Reserved`, all of your mobile services in the same region run in premium mode.</p> </div>   -->

## <a name="Manage_tool_local_settings"></a>管理工具本地设置

本地设置是你的订阅 ID 和默认存储帐户名称。

**config list [options]**

此命令显示配置设置。

    ~$ azure config list
    info:   Displaying config settings
    data:   Setting                Value                               
    data:   ---------------------  ------------------------------------
    data:   subscription           32-digit-subscription-key
    data:   defaultStorageAccount  name

**config set [options] \<name\>,\<value\>**

此命令更改配置设置。

    ~$ azure config set defaultStorageAccount myname
    info:   Setting 'defaultStorageAccount' to value 'myname'
    info:   Changes saved.

## <a name="Commands_to_manage_service_bus"></a>管理 Service Bus 的命令

使用这些命令来管理你的 Service Bus 帐户

**sb namespace create \<name\> \<location\>**

创建一个新的 Service Bus 命名空间

    ~$ azure sb namespace create mysbnamespacea-test "China East"
    info:    Executing command sb namespace create
    + Creating namespace mysbnamespacea-test in region China East
    data:    Name: mysbnamespacea-test
    data:    Region: China East
    data:    DefaultKey: fBu8nQ9svPIesFfMFVhCFD+/sY0rRbifWMoRpYy0Ynk=
    data:    Status: Activating
    data:    CreatedAt: 2013-11-14T16:23:29.32Z
    data:    AcsManagementEndpoint: https://mysbnamespacea-test-sb.accesscontrol.chinacloudapi.cn/
    data:    ServiceBusEndpoint: https://mysbnamespacea-test.servicebus.chinacloudapi.cn/
    data:    SubscriptionId: 8679c8be3b0549d9b8fb4bd232a48931
    data:    Enabled: true
    info:    sb namespace create command OK

**sb namespace show \<name\>**

显示有关特定命名空间的详细信息

    ~$ azure sb namespace show mysbnamespacea-test
    info:    Executing command sb namespace show
    + Getting namespace
    data:    Name: mysbnamespacea-test
    data:    Region: China East
    data:    DefaultKey: fBu8nQ9svPIesFfMFVhCFD+/sY0rRbifWMoRpYy0Ynk=
    data:    Status: Active
    data:    CreatedAt: 2013-11-14T16:23:29.32Z
    data:    AcsManagementEndpoint: https://mysbnamespacea-test-sb.accesscontrol.chinacloudapi.cn/
    data:    ServiceBusEndpoint: https://mysbnamespacea-test.servicebus.chinacloudapi.cn/

    data:    SubscriptionId: 8679c8be3b0549d9b8fb4bd232a48931
    data:    Enabled: true
    data:    UpdatedAt: 2013-11-14T16:25:37.85Z
    info:    sb namespace show command OK

**sb namespace list**

列出为你的帐户创建的所有命名空间

    ~$ azure sb namespace list
    info:    Executing command sb namespace list
    + Getting namespaces
    data:    Name                 Region   Status
    data:    -------------------  -------  ------
    data:    mysbnamespacea-test  China East  Active
    info:    sb namespace list command OK

**sb namespace delete \<name\>**

删除命名空间

    ~$ azure sb namespace delete mysbnamespacea-test
    info:    Executing command sb namespace delete
    Delete namespace mysbnamespacea-test? [y/n] y
    + Deleting namespace mysbnamespacea-test
    info:    sb namespace delete command OK

**sb namespace location list**

显示所有可用命名空间位置的列表

    ~$ azure sb namespace location list
    info:    Executing command sb namespace location list
    + Getting locations
    data:    Name              Code
    data:    ----------------  ----------------
    data:    China East        China East 
    data:    China North       China North
    info:    sb namespace location list command OK

**sb namespace verify \<name\>**

检查命名空间是否可用

## <a name="Commands_to_manage_sql"></a>管理 SQL Database 的命令

使用这些命令来管理你的 Azure SQL Database

### 管理 SQL Server 的命令

使用这些命令来管理你的 SQL Server

**sql server create \<administratorLogin\> \<administratorPassword\> \<location\>**

创建新的数据库服务器

    ~$ azure sql server create test T3stte$t "China East"
    info:    Executing command sql server create
    + Creating SQL Server
    data:    Server Name i1qwc540ts
    info:    sql server create command OK

**sql server show \<name\>**

显示服务器详细信息

    ~$ azure sql server show xclfgcndfg
    info:    Executing command sql server show
    + Getting SQL server
    data:    SQL Server Name xclfgcndfg
    data:    SQL Server AdministratorLogin msopentechforums
    data:    SQL Server Location China East
    data:    SQL Server FullyQualifiedDomainName xclfgcndfg.database.chinacloudapi.cn
    info:    sql server show command OK

**sql server list**

获取服务器的列表

    ~$ azure sql server list
    info:    Executing command sql server list
    + Getting SQL server
    data:    Name        Location
    data:    ----------  --------
    data:    xclfgcndfg  China East
    info:    sql server list command OK

**sql server delete \<name\>**

删除服务器

    ~$ azure sql server delete i1qwc540ts
    info:    Executing command sql server delete
    Delete server i1qwc540ts? [y/n] y
    + Removing SQL Server
    info:    sql server delete command OK

### 管理 SQL Database 的命令

使用这些命令来管理你的 SQL Database

**sql db create [options] \<serverName\> \<databaseName\> \<administratorPassword\>**

创建一个新的数据库实例

    ~$ azure sql db create fr8aelne00 newdb test
    info:    Executing command sql db create
    Administrator password: ********
    + Creating SQL Server Database
    info:    sql db create command OK

**sql db show [options] \<serverName\> \<databaseName\> \<administratorPassword\>**

显示数据库详细信息

    C:\windows\system32>azure sql db show fr8aelne00 newdb test
    info:    Executing command sql db show
    Administrator password: ********
    + Getting SQL server databases
    data:    Database _ ContentRootElement=m:properties, id=https://fr8aelne00.datab
    ase.windows.net/v1/ManagementService.svc/Server2('fr8aelne00')/Databases(4), ter
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

**sql db list [options] \<serverName\> \<administratorPassword\>**

列出数据库

    ~$ azure sql db list fr8aelne00 test
    info:    Executing command sql db list
    Administrator password: ********
    + Getting SQL server databases
    data:    Name    Edition  Collation                     MaxSizeInGB
    data:    ------  -------  ----------------------------  -----------
    data:    master  Web      SQL_Latin1_General_CP1_CI_AS  5
    info:    sql db list command OK

**sql db delete [options] \<serverName\> \<databaseName\> \<administratorPassword\>**

删除数据库

    ~$ azure sql db delete fr8aelne00 newdb test
    info:    Executing command sql db delete
    Administrator password: ********
    Delete database newdb? [y/n] y
    + Getting SQL server databases
    + Removing database
    info:    sql db delete command OK

### 管理 SQL Server 防火墙规则的命令

使用这些命令来管理 SQL Server 防火墙规则

**sql firewallrule create [options] \<serverName\> \<ruleName\> \<startIPAddress\> \<endIPAddress\>**

为 SQL Server 创建新的防火墙规则

    ~$ azure sql firewallrule create fr8aelne00 allowed 131.107.0.0 131.107.255.255
    info:    Executing command sql firewallrule create
    + Creating Firewall Rule
    info:    sql firewallrule create command OK

**sql firewallrule show [options] \<serverName\> \<ruleName\>**

显示防火墙规则详细信息

    ~$ azure sql firewallrule show fr8aelne00 allowed
    info:    Executing command sql firewallrule show
    + Getting firewall rule
    data:    Firewall rule Name allowed
    data:    Firewall rule Type Microsoft.SqlAzure.FirewallRule
    data:    Firewall rule State Normal
    data:    Firewall rule SelfLink https://management.core.windows.net/9e672699-105
    5-41ae-9c36-e85152f2e352/services/sqlservers/servers/fr8aelne00/firewallrules/allowed
    data:    Firewall rule ParentLink https://management.core.windows.net/9e672699-1
    055-41ae-9c36-e85152f2e352/services/sqlservers/servers/fr8aelne00
    data:    Firewall rule StartIPAddress 131.107.0.0
    data:    Firewall rule EndIPAddress 131.107.255.255
    info:    sql firewallrule show command OK

**sql firewallrule list [options] \<serverName\>**

列出防火墙规则

    ~$ azure sql firewallrule list fr8aelne00
    info:    Executing command sql firewallrule list
    \data:    Name     Start IP address  End IP address
    data:    -------  ----------------  ---------------
    data:    allowed  131.107.0.0       131.107.255.255
    +
    info:    sql firewallrule list command OK

**sql firewallrule delete [options] \<serverName\> \<ruleName\>**

此命令将删除防火墙规则

    ~$ azure sql firewallrule delete fr8aelne00 allowed
    info:    Executing command sql firewallrule delete
    Delete rule allowed? [y/n] y
    + Removing firewall rule
    info:    sql firewallrule delete command OK

## <a name="Commands_to_manage_vnet"></a>管理虚拟网络的命令

使用这些命令来管理你的虚拟网络

**network vnet create [options] \<location\>**

创建新的虚拟网络

    ~$ azure network vnet create vnet1 --location "China East" -v
    info:    Executing command network vnet create
    info:    Using default address space start IP: 10.0.0.0
    info:    Using default address space cidr: 8
    info:    Using default subnet start IP: 10.0.0.0
    info:    Using default subnet cidr: 11
    verbose: Address Space [Starting IP/CIDR (Max VM Count)]: 10.0.0.0/8 (16777216)
    verbose: Subnet [Starting IP/CIDR (Max VM Count)]: 10.0.0.0/11 (2097152)
    verbose: Getting Network Configuration
    verbose: Getting or creating affinity group
    verbose: Getting Affinity Groups
    verbose: Getting Locations
    verbose: Creating new affinity group AG1-CLI-22814b5a4049ca4d
    info:    Using affinity group AG1-CLI-22814b5a4049ca4d
    verbose: Updating Network Configuration
    info:    network vnet create command OK

**network vnet show \<name\>**

显示虚拟网络的详细信息

    ~$ azure network vnet show vnet1
    info:    Executing command network vnet show
    + Getting Virtual Networks
    data:    Name "vnet1"
    data:    Id "25786fbe-08e8-4e7e-b1de-b98b7e586c7a"
    data:    AffinityGroup "AG1-CLI-22814b5a4049ca4d"
    data:    State "Created"
    data:    AddressSpace AddressPrefixes 0 "10.0.0.0/8"
    data:    Subnets 0 Name "subnet-1"
    data:    Subnets 0 AddressPrefix "10.0.0.0/11"
    info:    network vnet show command OK

**vnet list**

列出所有现有的虚拟网络

    ~$ azure network vnet list
    info:    Executing command network vnet list
    + Getting Virtual Networks
    data:    Name        Status   AffinityGroup
    data:    ----------  -------  -------------
    data:    vnet1      Created  AG1-CLI-22814b5a4049ca4d
    data:    vnet2      Created  AG1-CLI-22814b5a4049ca4d
    data:    vnet3      Created  AG1-CLI-22814b5a4049ca4d
    data:    vnet4      Created  AG1-CLI-22814b5a4049ca4d
    info:    network vnet list command OK

**network vnet show \<name\>**

显示指定虚拟网络的详细信息

    ~$ azure network vnet show opentechvn1
    info:    Executing command network vnet show
    + Getting Virtual Networks
    data:    Name "opentechvn1"
    data:    Id "cab41cb0-396a-413b-83a1-302f0f1c867d"
    data:    AffinityGroup "AG-CLI-456f89eaa7fae2b3"
    data:    State "Created"
    data:    AddressSpace AddressPrefixes 0 "10.100.23.255/27"
    data:    Subnets 0 Name "frontend"
    data:    Subnets 0 AddressPrefix "10.100.23.224/29"
    info:    network vnet show command OK

**network vnet delete \<name\>**

删除指定的虚拟网络

    ~$ azure network vnet delete opentechvn1
    info:    Executing command network vnet delete
    + Getting Network Configuration
    Delete the virtual network opentechvn1 ?  (y/n) y
    + Deleting the virtual network opentechvn1
    info:    network vnet delete command OK

**network export [file-path]**

对于高级网络配置，你可以通过本地方式导出网络配置。请注意，导出的网络配置包括 DNS 服务器设置、虚拟网络设置、本地网络站点设置和其他设置。

**network import [file-path]**

导入本地网络配置。

**network dnsserver register [options] \<dnsIP\>**

注册你计划在网络配置中用来进行名称解析的 DNS 服务器

    ~$ azure network dnsserver register 98.138.253.109 --dns-id FrontEndDnsServer
    info:    Executing command network dnsserver register
    + Getting Network Configuration
    + Updating Network Configuration
    info:    network dnsserver register command OK

**network dnsserver list**

列出在你的网络配置中注册的所有 DNS 服务器

    ~$ azure network dnsserver list
    info:    Executing command network dnsserver list
    + Getting Network Configuration
    data:    DNS Server ID         DNS Server IP
    data:    --------------------  --------------
    data:    DNS-bb39b4ac34d66a86  44.55.22.11
    data:    FrontEndDnsServer     98.138.253.109
    info:    network dnsserver list command OK

**network dnsserver unregister [options] \<dnsIP\>**

从网络配置中删除 DNS 服务器条目

    ~$ azure network dnsserver unregister 77.88.99.11
    info:    Executing command network dnsserver unregister
    + Getting Network Configuration
    Delete the DNS server entry dns-4 ( 77.88.99.11 ) %s ? (y/n) y
    + Deleting the DNS server entry dns-4 ( 77.88.99.11 )
    info:    network dnsserver unregister command OK

  [Azure SDK 安装程序]: http://go.microsoft.com/fwlink/?LinkId=252249
  [管理帐户信息和发布设置]: #Manage_your_account_information_and_publish_settings
  [管理 Azure 虚拟机的命令]: #Commands_to_manage_your_Azure_virtual_machines
  [管理 Azure 虚拟机终结点的命令]: #Commands_to_manage_your_Azure_virtual_machine_endpoints
  [管理 Azure 虚拟机映像的命令]: #Commands_to_manage_your_Azure_virtual_machine_images
  [管理 Azure 虚拟机数据磁盘的命令]: #Commands_to_manage_your_Azure_virtual_machine_data_disks
  [管理 Azure 云服务的命令]: #Commands_to_manage_your_Azure_cloud_services
  [管理 Azure 证书的命令]: #Commands_to_manage_your_Azure_certificates
  [管理网站的命令]: #Commands_to_manage_your_web_sites
  [管理工具本地设置]: #Manage_tool_local_settings
  [管理 Service Bus 的命令]: #Commands_to_manage_service_bus
  [管理 SQL Database 的命令]: #Commands_to_manage_sql
  [管理虚拟网络的命令]: #Commands_to_manage_vnet
  [Azure 技术图表]: ./media/command-line-tools/architecturediagram.jpg
  [azurenetworkdiagram]: ./media/command-line-tools/networkdiagram.jpg
