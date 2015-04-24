<properties
	pageTitle="������ Mac �� Linux �� Azure �����й���"
	description="�˽������ Azure ��ʹ�������� Mac �� Linux �������й��ߡ�"
	services="web-sites, virtual-machines, mobile-services, cloud-services"
	documentationCenter=""
	authors="squillace"
	manager="timlt"
	editor="tysonn"/>
<tags ms.service="web-sites, virtual-machines, mobile-services, cloud-services"
    ms.date="03/10/2015"
    wacn.date="04/15/2015"
    />

# ������ Mac �� Linux �� Azure �����й���

�˹����ṩ�����ڴ� Mac �� Linux ���洴��������͹���������Ĺ��ܡ��˹��������������� .NET��Node.JS �� PHP �� Azure SDK һ��װ�� Windows PowerShell cmdlet ���ṩ�Ĺ��ܡ�

��Ҫ�� Mac �ϰ�װ�ù��ߣ������ز����� [Azure SDK ��װ����](http://az412849.vo.msecnd.net/downloads04/azure-cli.0.8.17.dmg)��

��Ҫ�� Linux �ϰ�װ�ù��ߣ��밲װ���°汾�� Node.JS��Ȼ��ʹ�� NPM ���а�װ��

    npm install azure-cli -g

��ѡ������ʾ�ڷ������У����磬[����]�����������в������Ǳ���ġ�

���˴˴���¼���ض�������Ŀ�ѡ�����⣬����������������ʾ��ϸ�������������ѡ���״̬���룩�Ŀ�ѡ������-v �����ṩ��ϸ������� -vv �����ṩ����ϸ�������--json ѡ���ԭʼ�� json ��ʽ��������

**Ŀ¼��**

* [�����ʻ���Ϣ�ͷ�������](#Manage_your_account_information_and_publish_settings)
* [���ڹ��� Azure �����������](#Commands_to_manage_your_Azure_virtual_machines)
* [���ڹ��� Azure ������ս�������](#Commands_to_manage_your_Azure_virtual_machine_endpoints)
* [���ڹ��� Azure �����ӳ�������](#Commands_to_manage_your_Azure_virtual_machine_images)
* [���ڹ��� Azure ��������ݴ��̵�����](#Commands_to_manage_your_Azure_virtual_machine_data_disks)
* [���ڹ��� Azure �Ʒ��������](#Commands_to_manage_your_Azure_cloud_services)
* [���ڹ��� Azure ֤�������](#Commands_to_manage_your_Azure_certificates)
* [���ڹ�����վ������](#Commands_to_manage_your_web_sites)
* [���ڹ��� Azure �ƶ����������](#Commands_to_manage_mobile_services)
* [������߱�������](#Manage_tool_local_settings)
* [���ڹ��� Service Bus ������](#Commands_to_manage_service_bus)
* [���ڹ���洢���������](#Commands_to_manage_your_Storage_objects)
* [���ڹ��� SQL ���ݿ������](#Commands_to_manage_sql)
* [���ڹ����������������](#Commands_to_manage_vnet)

## <a name="Manage_your_account_information_and_publish_settings"></a>�����ʻ���Ϣ�ͷ�������
�ù���ʹ����� Azure ������Ϣ���ӵ�����ʻ������Դ� Azure �Ż��еķ��������ļ��л�ȡ����Ϣ���������������Ե��뷢�������ļ���Ϊ�����Ա����������ã��ù��߻Ὣ���������ں�����������ֻ�赼����ķ�������һ�Ρ�

**account download [options]** 

������������ <a href="https://manage.windowsazure.cn/publishsettings/index?client=xplat">https://manage.windowsazure.cn/publishsettings</a>
���������ص��ļ���

**account import [options] &lt;file>**


������� publishsettings �ļ���֤���Ա��պ���Թ��ù���ʹ�á�

	~$ azure account import publishsettings.publishsettings
	info:   Importing publish settings file publishsettings.publishsettings
	info:   Found subscription: Free Trial
	info:   Found subscription: Pay-As-You-Go
	info:   Setting default subscription to: Free Trial
	warn:   The 'publishsettings.publishsettings' file contains sensitive information.
	warn:   Remember to delete it now that it has been imported.
	info:   Account publish settings imported successfully

> [AZURE.NOTE] publishsettings �ļ����԰����йض�����ĵ���ϸ��Ϣ�������������ƺ� ID�������㵼�� publishsettings �ļ�ʱ����һ�����Ľ�����Ĭ�϶��ġ���Ҫʹ���������ģ��������������
<code>~$ azure config set subscription &lt;other-subscription-id&gt;</code>

**account clear [options]**

������ɾ��������Ѵ洢�������á�������ڴ˼������ʹ����ù��ߣ�����ϣ��ȷ���պ���ͨ������ʻ�ʹ�øù��ߣ���ʹ�ô����

	~$ azure account clear
	Clearing account info.
	info:   OK

**account list [options]**

�г�����Ķ���

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

���õ�ǰ����

### ���ڹ����Ե�������

**account affinity-group list [options]**

�������г���� Azure ��Ե�顣

������һ���������Խ��̨��������ʱ���õ�Ե�顣��Ե��ָ����������Ӧ�����ܱ˴˽ӽ����Ӷ����������ӳ١�

	~$ azure account affinity-group list
	+ Fetching affinity groups
	data:   Name                                  Label   Location
	data:   ------------------------------------  ------  --------
	data:   535EBAED-BF8B-4B18-A2E9-8755FB9D733F  opentec  China North
	info:   account affinity-group list command OK

**account affinity-group create [options] &lt;name&gt;**

��������µĵ�Ե��

	~$ azure account affinity-group create opentec -l "China North"
	info:    Executing command account affinity-group create
	+ Creating affinity group
	info:    account affinity-group create command OK

**account affinity-group show [options] &lt;name&gt;**

��������ʾ��Ե�����ϸ��Ϣ

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

������ɾ��ָ���ĵ�Ե��

	~$ azure account affinity-group delete opentec
	info:    Executing command account affinity-group delete
	Delete affinity group opentec? [y/n] y
	+ Deleting affinity group
	info:    account affinity-group delete command OK

### ���ڹ����ʻ�����������

**account env list [options]**

�ʻ������б�

	C:\windows\system32>azure account env list
	info:    Executing command account env list
	data:    Name
	data:    ---------------
	data:    AzureCloud
	data:    AzureChinaCloud
	info:    account env list command OK

**account env show [options] [environment]**

��ʾ�ʻ�������ϸ��Ϣ

	~$ azure account env show AzureChinaCloud
	info:    Executing command account env show
	data:    Name:                       AzureChinaCloud
	data:    activeDirectoryEndpointUrl: https://login.chinacloudapi.cn
	data:    activeDirectoryResourceId:  https://management.core.chinacloudapi.cn/
	data:    commonTenantName:           common
	data:    galleryEndpointUrl:
	data:    isPublicEnvironment:        true
	data:    managementEndpointUrl:      https://management.core.chinacloudapi.cn
	data:    portalUrl:                  https://manage.windowsazure.cn/
	data:    publishingProfileUrl:       https://manage.windowsazure.cn/publishsettings/index?client=xplat
	data:    resourceManagerEndpointUrl:
	data:    sqlManagementEndpointUrl:
	data:    sqlServerHostnameSuffix:    .database.chinacloudapi.cn
	info:    account env show command OK

**account env add [options] [environment]**

�����һ��������ӵ��ʻ�

**account env set [options] [environment]**

�����������ʻ�����

**account env delete [options] [environment]**

��������ʻ���ɾ��ָ���Ļ���

## <a name="Commands_to_manage_your_Azure_virtual_machines"></a>���ڹ��� Azure �����������
��ͼ��ʾ������� Azure �Ʒ�����������𻷾����й� Azure �������

![Azure Technical Diagram](./media/virtual-machines-command-line-tools/architecturediagram.jpg)

**create-new** �� Blob �洢�д���������������ͼ�е� e:\����**attach** �Ὣ�Ѵ�����δ���ӵĴ��̸��ӵ��������

**vm create [options] &lt;dns-name> &lt;image> &lt;userName> [password]**

��������µ� Azure �������Ĭ������£���������ÿ̨����� (vm) ��λ�����Լ����Ʒ����У����ǣ������ʹ�ô˴��ἰ�� -c ѡ��ָ�����������ӵ������Ʒ����С�

vm create ������ Azure �Ż�һ����ֻ�����������𻷾��д����������Ŀǰû���������Ʒ���Ĺ��ɲ��𻷾��д����������ѡ������Ķ���û�����е� Azure �洢�ʻ������������һ����

�����ͨ�� --location ����ָ��λ�ã�Ҳ����ͨ�� --affinity-group ����ָ����Ե�顣��������������κ�һ����û���ṩ��ϵͳ����ʾ�����Чλ���б����ṩһ����

���ṩ����ĳ��ȱ���Ϊ 8-123 ���ַ�������������Ҫ���ڴ�������Ĳ���ϵͳ�����븴�Ӷ�Ҫ��

�����Ԥ����Ҫʹ�� SSH ���������� Linux �������ͨ��������ˣ���������ڴ��������ʱͨ�� -e ѡ������ SSH���ڴ�������������޷����� SSH��

Windows ������Ժ����ͨ����Ӷ˿� 3389 ��Ϊ�ս�������� RDP��

������֧�����¿�ѡ������

**-c, --connect** ���йܷ������Ѵ����Ĳ����д�������������δ�� -vmname ���ѡ��һ��ʹ�ã����Զ�����������������ơ�<br />
**-n, --vm-name** ָ������������ơ�Ĭ������£��˲��������йܷ������ơ����δָ�� -vmname�������� &lt;service-name>&lt;id> ��ʽ������������ƣ����� &lt;id> �Ƿ������������������������ 1�����磬�����ʹ�ô�������ӵ��һ̨������������йܷ��� MyService �����һ̨�����������Ὣ�����������Ϊ MyService2��<br />
**-u, --blob-url** ָ�����д��������ϵͳ���̵�Ŀ�� Blob �洢 URL��
<br>**-z, --vm-size** ָ��������Ĵ�С����ЧΪ"extrasmall"��"small"��"medium"��"large"��"extralarge"��Ĭ��ֵΪ "small"��
**-r** ����� Windows ������� RDP ���ӡ�
<br>**-e, --ssh** ����� Windows ������� SSH ���ӡ�
<br>**-t, --ssh-cert** ָ�� SSH ֤�顣
<br>**-s** ����
<br>**-o, --community** ָ����ӳ����һ������ӳ��
**-w** ������������ <br/>
**-l, --location** ָ��λ�ã����� "North Central US"����
<br>**-a, --affinity-group** ָ����Ե�顣<br />
**-w, --virtual-network-name** ָ��Ҫ�������������������������硣�ɴ� Azure �Ż����ú͹����������硣<br />
**-b, --subnet-names** ָ��Ҫ������������������ơ�

�ڴ�ʾ���У�55bc2b193643443bb879a78bda516fc8__Windows-Server-2012-R2-201502.01-zh.cn-127GB.vhd ����ƽ̨�ṩ��һ��ӳ���йز���ϵͳӳ�����ϸ��Ϣ������� VM ӳ���б��

	~$ azure vm create my-vm-name 55bc2b193643443bb879a78bda516fc8__Windows-Server-2012-R2-201502.01-zh.cn-127GB.vhd username --location "China North" -r
	info:   Executing command vm create
	Enter VM 'my-vm-name' password: ************
	info:   vm create command OK

**vm create-from &lt;dns-name> &lt;role-file>**

������� JSON ��ɫ�ļ������µ� Azure �������

	~$ azure vm create-from my-vm example.json
	info:   OK

**vm list [options]**

�������г� Azure �������-json ѡ��ָ����ԭʼ JSON ��ʽ���ؽ����

	~$ azure vm list
	info:   Executing command vm list
	data:    Name     	Status              Location  	DNS Name              	IP Address
	data:    -------  	------------------  --------  	--------------------  	----------
	data:    my-vm-name	ReadyRole			China North	my-vm.chinacloudapp.cn
	info:    vm list command OK
	info:   vm list command OK

**vm location list [options]**

�������г����п��õ� Azure �ʻ�λ�á�

	~$ azure vm location list
	info:    Executing command vm location list
	data:    Name
	data:    -----------
	data:    China East
	data:    China North
	info:    vm location list command OK

**vm show [options] &lt;name>**

��������ʾ�й� Azure ���������ϸ��Ϣ��--json ѡ��ָ����ԭʼ JSON ��ʽ���ؽ����

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
	data:       Image: '55bc2b193643443bb879a78bda516fc8__Windows-Server-2012-R2-201502.01-zh.cn-127GB.vhd',
	data:       OSVersion: 'WA-GUEST-OS-1.18_201203-01'
	data:   }
	info:   vm show command OK

**vm delete [options] &lt;name>**

������ɾ�� Azure �������Ĭ������£������ɾ�����д�������ϵͳ���̺����ݴ��̵� Azure Blob����Ҫɾ���� Blob �Լ���Ϊ����������������ָ�� -b ѡ�

	~$ azure vm delete my-vm
	info:   Executing command vm delete
	info:   vm delete command OK

**vm start [options] &lt;name>**

��������� Azure �������

	~$ azure vm start my-vm
	info:   Executing command vm start
	info:   vm start command OK

**vm restart [options] &lt;name>**

������������� Azure �������

	~$ azure vm restart my-vm
	info:   Executing command vm restart
	info:   vm restart command OK

**vm shutdown [options] &lt;name>**

������ر� Azure �����������ʹ�� -p ѡ��ָ���ڹر�ʱ���ͷż�����Դ��

```
~$ azure vm shutdown my-vm
info:   Executing command vm shutdown
info:   vm shutdown command OK  
```

**vm capture &lt;vm-name> &lt;target-image-name>**

������� Azure �����ӳ��

ֻ�е������״̬Ϊ**��ֹͣ**ʱ�����ܲ��������ӳ����ر��������Ȼ���ټ���������

	~$ azure.cmd vm capture my-vm mycaptureimagename --delete
	info:   Executing command vm capture
	+ Fetching VMs
	+ Capturing VM
	info:   vm capture command OK

**vm export [options] &lt;vm-name> &lt;file-path>**

�����һ�� Azure �����ӳ�񵼳����ļ�

	~$ azure vm export "myvm" "C:\"
	info:    Executing command vm export
	+ Getting virtual machines
	+ Exporting the VM
	info:   vm export command OK

## <a name="Commands_to_manage_your_Azure_virtual_machine_endpoints"></a>���ڹ��� Azure ������ս�������
��ͼ��ʾ�˰�����������ʵ���ĵ��Ͳ������ϵ�ṹ����ע�⣬�ڴ�ʾ���У��˿� 3389 ��ÿ̨������Ͼ�Ϊ��״̬�����ڽ��� RDP ���ʣ������Ҹ���ƽ�������ڽ�����·�ɵ��������ÿ̨������ϻ���һ���ڲ� IP ��ַ������ 168.55.11.1�������ڲ� IP ��ַҲ�����������֮���ͨ�š�

![azurenetworkdiagram](./media/virtual-machines-command-line-tools/networkdiagram.jpg)

����������ⲿ����ͨ������ƽ��������ˣ�������԰�����̨������Ĳ����е��ض������ָ�����󡣶��ڰ�����̨������Ĳ��𣬱���������� (vm-port) �븺��ƽ���� (lb-port) ֮�����ö˿�ӳ�䡣

**vm endpoint create &lt;vm-name> &lt;lb-port> [vm-port]**

�������������ս�㡣�㻹����ʹ�� -u �� --enable-direct-server-return ��ָ���Ƿ��ڴ��ս��������ֱ�ӷ��������أ�Ĭ�������Ϊ���á�

	~$ azure vm endpoint create my-vm 8888 8888
	azure vm endpoint create my-vm 8888 8888
	info:   Executing command vm endpoint create
	+ Fetching VM
	+ Reading network configuration
	+ Updating network configuration
	info:   vm endpoint create command OK

**vm endpoint create-multiple [options] &lt;vm-name> &lt;lb-port>[:&lt;vm-port>[:&lt;protocol>[:&lt;lb-set-name>[:&lt;prob-protocol>:&lt;lb-prob-port>[:&lt;prob-path>]]]]] ]{1-*}**

������� VM �ս�㡣�㻹����ʹ�� -u �� --enable-direct-server-return ��ָ���Ƿ��ڴ��ս��������ֱ�ӷ��������أ�Ĭ�������Ϊ���á�

**vm endpoint delete &lt;vm-name> &lt;lb-port>**

������ɾ��������ս�㡣

	~$ azure vm endpoint delete my-vm 8888
	azure vm endpoint delete my-vm 8888
	info:   Executing command vm endpoint delete
	+ Fetching VM
	+ Reading network configuration
	+ Updating network configuration
	info:   vm endpoint delete command OK

**vm endpoint list &lt;vm-name>**

�������г�����������ս�㡣-json ѡ��ָ����ԭʼ JSON ��ʽ���ؽ����

	~$ azure vm endpoint list my-linux-vm
	data:   Name  External Port  Local Port
	data:   ----  -------------  ----------
	data:   ssh   22             22

**vm endpoint update [options] &lt;vm-name> &lt;endpoint-name>**

������ʹ����Щѡ� VM �ս�����Ϊ��ֵ��

    -n, --endpoint-name <name>          the new endpoint name
    -lo, --lb-port <port>                the new load balancer port
    -t, --vm-port <port>                the new local port
    -o, --endpoint-protocol <protocol>  the new transport layer protocol for port (tcp or udp)

**vm endpoint show [options] &lt;vm-name>**

��������ʾ VM ���ս�����ϸ��Ϣ

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

## <a name="Commands_to_manage_your_Azure_virtual_machine_images"></a>���ڹ��� Azure �����ӳ�������

�����ӳ����������ġ��ɸ�����Ҫ���и��Ƶ��������������

**vm image list [options]**

�������ȡ�����ӳ����б�����������͵�ӳ��Microsoft ������ӳ����"MSFT"��Ϊǰ׺����������������ӳ��ͨ���Թ�Ӧ�̵�������Ϊǰ׺���Լ��㴴����ӳ����Ҫ����ӳ������Բ������������������ص� Blob �洢���Զ��� .vhd ����ӳ���й�ʹ���Զ��� .vhd ����ϸ��Ϣ������� vm image create��
-json ѡ��ָ����ԭʼ JSON ��ʽ���ؽ����

	~$ azure vm image list
	data:   Name                                                                   Category   OS
	data:   ---------------------------------------------------------------------  ---------  -------
	data:   CANONICAL__Canonical-Ubuntu-12-04-20120519-2012-05-19-en-us-30GB.vhd   Canonical  Linux
	data:   MSFT__Windows-Server-2008-R2-SP1.11-29-2011                            Microsoft  Windows
	data:   MSFT__Windows-Server-2008-R2-SP1-with-SQL-Server-2012-Eval.11-29-2011  Microsoft  Windows
	data:   MSFT__Windows-Server-8-Beta.en-us.30GB.2012-03-22                      Microsoft  Windows
	data:   MSFT__Windows-Server-8-Beta.2-17-2012                                  Microsoft  Windows
	data:   MSFT__Windows-Server-2008-R2-SP1.en-us.30GB.2012-3-22                  Microsoft  Windows
	data:   OpenLogic__OpenLogic-CentOS-62-20120509-en-us-30GB.vhd                 OpenLogic  Linux
	data:   SUSE__SUSE-Linux-Enterprise-Server-11SP2-20120521-en-us-30GB.vhd       SUSE       Linux
	data:   SUSE__OpenSUSE64121-03192012-en-us-15GB.vhd                            SUSE       Linux
	data:   WIN2K8-R2-WINRM                                                        User       Windows
	info:   vm image list command OK

**vm image show [options] &lt;name>**

��������ʾ�����ӳ�����ϸ��Ϣ��

	~$ azure vm image show 55bc2b193643443bb879a78bda516fc8__Windows-Server-2012-R2-201502.01-zh.cn-127GB.vhd 
	+ Fetching VM image
	data:    category "Public"
	data:    label "Windows Server 2012 R2 Datacenter, February 2015 (zh-cn)"
	data:    location "China East;China North"
	data:    logicalSizeInGB 128
	data:    name "55bc2b193643443bb879a78bda516fc8__Windows-Server-2012-R2-201502.01-zh.cn-127GB.vhd"
	data:    operatingSystemType "Windows"
	data:    description "At the heart of the Microsoft Cloud OS vision, Windows Server 2012 R2 brings Microsoft's experience delivering global-scale cloud services into your infrastructure. It offers enterprise-class performance, flexibility for your applications and excellent economics for your datacenter and hybrid cloud environment. This image includes Windows Server 2012 R2 Update."
	data:    imageFamily "Windows Server 2012 R2 Datacenter (zh-cn)"
	data:    publishedDate 2015-02-11T08:00:00.000Z
	data:    isPremium false
	data:    iconUri "WindowsServer2012R2_100.png"
	data:    publisherName "Microsoft"
	data:    smallIconUri "WindowsServer2012R2_45.png"
	info:    vm image show command OK

**vm image delete [options] &lt;name>**

������ɾ�������ӳ��

	~$ azure vm image delete my-vm-image
	info:   Executing command vm image delete
	info:   VM image deleted: my-vm-image
	info:   vm image delete command OK

**vm image create &lt;name> [source-path]**

������������ӳ������Զ��� .vhd �ļ������ص� Blob �洢��Ȼ��Ӹ�λ�ô��������ӳ��Ȼ���ʹ�ô������ӳ�񴴽��������Location �� OS �����Ǳ���ġ�

ĳЩϵͳ��ʩ��ÿ�����ļ����������ơ�������������ƣ����߽���ʾ�ļ����������ƴ��������ʹ�� -p &lt;number> �����ٴ����д�����Լ�С�������������Ĭ�ϵ������������Ϊ 96��

	~$ azure vm image create mytestimage ./Sample.vhd -o windows -l "China North"
	info:   Executing command vm image create
	+ Retrieving storage accounts
	info:   VHD size : 13 MB
	info:   Uploading 13312.5 KB
	Requested:100.0% Completed:100.0% Running: 105 Time:    8s Speed:  1721 KB/s
	info:   http://myaccount.blob.core.chinacloudapi.cn/vm-images/Sample.vhd is uploaded successfully
	info:   vm image create command OK

## <a name="Commands_to_manage_your_Azure_virtual_machine_data_disks"></a>���ڹ��� Azure ��������ݴ��̵�����

���ݴ����� Blob �洢�пɹ������ʹ�õ� .vhd �ļ����й���ν����ݴ��̲��� Blob �洢����ϸ��Ϣ�������ǰ����ʾ�� Azure ����ͼ���

���ڸ������ݴ��̵����azure vm disk attach �� azure vm disk attach-new������� SCSI Э���Ҫ���򸽼ӵ����ݴ��̷����߼���Ԫ�� (LUN)�����򸽼ӵ�������ĵ�һ�����ݴ��̷��� LUN 0������һ������ LUN 1���������ơ�

��ʹ�� azure vm disk detach ����������ݴ���ʱ����ʹ�� &lt;lun&gt; ����ָ��Ҫ����Ĵ��̡�

> [AZURE>NOTE] ��ע�⣬Ӧʼ�հ��෴��˳��������ݴ��̣��������ѷ���ı����ߵ� LUN ��ʼ��Linux SCSI �㲻֧�����Ը����б�Žϸߵ� LUN ʱ�����Žϵ͵� LUN�����磬��Ӧ���Ը����� LUN 1 ������·��� LUN 0��

**vm disk show [options] &lt;name>**

��������ʾ�й� Azure ���̵���ϸ��Ϣ��

	~$ azure vm disk show anucentos-anucentos-0-20120524070008
	info:   Executing command vm disk show
	data:   AttachedTo DeploymentName "mycentos"
	data:   AttachedTo HostedServiceName "myanucentos"
	data:   AttachedTo RoleName "myanucentos"
	data:   OS "Linux"
	data:   Location "China North"
	data:   LogicalDiskSizeInGB "30"
	data:   MediaLink "http://mystorageaccount.blob.core.chinacloudapi.cn/vhd-store/mycentos-cb39b8223b01f95c.vhd"
	data:   Name "mycentos-mycentos-0-20120524070008"
	data:   SourceImageName "OpenLogic__OpenLogic-CentOS-62-20120509-en-us-30GB.vhd"
	info:   vm disk show command OK

**vm disk list [options] [vm-name]**

�������г� Azure ���̣����߸��ӵ�ָ��������Ĵ��̡�������д�����ʱʹ������������Ʋ����������ظ��ӵ�������������д��̡�Lun 1 ��������������ģ����г����κ��������̶��ǵ������ӵġ�

	~$ azure vm disk list mycentos
	info:   Executing command vm disk list
	data:   Lun  Size(GB)  Blob-Name
	data:   ---  --------  --------------------------------
	data:   1    30        mycentos-cb39b8223b01f95c.vhd
	data:   2    10        mycentos-e3f0d717950bb78d.vhd
	info:   vm disk list command OK

�ڲ�ʹ����������Ʋ����������ִ�д�����������д��̡�

	~$ azure vm disk list
	data:   Name                                        OS
	data:   ------------------------------------------  -------
	data:   mycentos-mycentos-0-20120524070008          Linux
	data:   mycentos-mycentos-2-20120525055052
	data:   mywindows-winvm-20120522223119              Windows
	info:   vm disk list command OK

**vm disk delete [options] &lt;name>**

������Ӹ��˴洢����ɾ�� Azure ���̡���ɾ������֮ǰ�����������з���ô��̡�

	~$ azure vm disk delete mycentos-mycentos-2-20120525055052
	info:   Executing command vm disk delete
	info:   Disk deleted: mycentos-mycentos-2-20120525055052
	info:   vm disk delete command OK

**vm disk create &lt;name> [source-path]**

���������غ�ע�� Azure ���̡�����ָ�� --blob-url��--location �� --affinity-group��������������� [source-path] ���ʹ�ã�������ָ���� .vhd �ļ���������ӳ��Ȼ�������ʹ�� vm disk attach ����ӳ�񸽼ӵ��������

ĳЩϵͳ��ʩ��ÿ�����ļ����������ơ�������������ƣ����߽���ʾ�ļ����������ƴ��������ʹ�� -p &lt;number> �����ٴ����д�����Լ�С�������������Ĭ�ϵ������������Ϊ 96��

	~$ azure vm disk create my-data-disk ~/test.vhd --location "China North"
	info:   Executing command vm disk create
	info:   VHD size : 10 MB
	info:   Uploading 10240.5 KB
	Requested:100.0% Completed:100.0% Running:  81 Time:   11s Speed:   952 KB/s
	info:   http://account.blob.core.chinacloudapi.cn/disks/test.vhd is uploaded successfully
	info:   vm disk create command OK

**vm disk upload [options] &lt;source-path> &lt;blob-url> &lt;storage-account-key>**

�������������� vm ����

	~$ azure vm disk upload "http://sourcestorage.blob.core.chinacloudapi.cn/vhds/sample.vhd" "http://destinationstorage.blob.core.chinacloudapi.cn/vhds/sample.vhd" "DESTINATIONSTORAGEACCOUNTKEY"
	info:   Executing command vm disk upload
	info:   Uploading 12351.5 KB
	info:   vm disk upload command OK

**vm disk attach &lt;vm-name> &lt;disk-image-name>**

����� Blob �洢�е����д��̸��ӵ��Ʒ����в���������������

	~$ azure vm disk attach my-vm my-vm-my-vm-2-201242418259
	info:   Executing command vm disk attach
	info:   vm disk attach command OK

**vm disk attach-new &lt;vm-name> &lt;size-in-gb> [blob-url]**

��������ݴ��̸��ӵ� Azure ��������ڴ�ʾ���У�20 ��Ҫ���ӵ��´��̵Ĵ�С���� GB Ϊ��λ���������ѡ��ʹ�� Blob URL ��Ϊ���һ����������ʽָ��Ҫ������Ŀ�� Blob�������ָ�� Blob URL�����Զ�����һ�� Blob ����

	~$ azure vm disk attach-new nick-test36 20 http://nghinazz.blob.core.chinacloudapi.cn/vhds/vmdisk1.vhd
	info:   Executing command vm disk attach-new
	info:   vm disk attach-new command OK  

**vm disk detach &lt;vm-name> &lt;lun>**

��������븽�ӵ� Azure ����������ݴ��̡�&lt;lun> ���ڱ�ʶҪ����Ĵ��̡�Ҫ�ڷ���ĳ������֮ǰ��ȡ��ô��̹����Ĵ��̵��б����ʹ�� vm disk-list &lt;vm-name>��

	~$ azure vm disk detach my-vm 2
	info:   Executing command vm disk detach
	info:   vm disk detach command OK

## <a name="Commands_to_manage_your_Azure_cloud_services"></a>���ڹ��� Azure �Ʒ��������

Azure �Ʒ������й��� Web ��ɫ�͸�����ɫ�ϵ�Ӧ�ó���ͷ���������������ڹ��� Azure �Ʒ���

**service create [options] &lt;serviceName>**

��������µ��Ʒ���

	~$ azure service create newservicemsopentech
	info:    Executing command service create
	+ Getting locations
	help:    Location:
	  1) China East
	  2) China North
	  : 2
	+ Creating cloud service
	data:    Cloud service name newservicemsopentech
	info:    service create command OK

**service show [options] &lt;serviceName>**

��������ʾ Azure �Ʒ������ϸ��Ϣ

	~$ azure service show newservicemsopentech
	info:    Executing command service show
	+ Getting cloud service
	data:    Name newservicemsopentech
	data:    Url https://management.core.chinacloudapi.cn/9e672699-1055-41ae-9c36-e85152f2e352/services/hostedservices/newservicemsopentech
	data:    Properties location China North
	data:    Properties label newservicemsopentech
	data:    Properties status Created
	data:    Properties dateCreated
	data:    Properties dateLastModified
	info:    service show command OK

**service list [options]**

�������г� Azure �Ʒ���

	~$ azure service list
	info:   Executing command service list
	data:   Name         Status
	data:   -----------  -------
	data:   service1     Created
	data:   service2     Created
	info:   service list command OK

**service delete [options] &lt;name>**

������ɾ�� Azure �Ʒ���

	~$ azure service delete myservice
	info:   Executing command service delete myservice
	info:   cloud-service delete command OK

��Ҫǿ��ɾ������ʹ��"-q"������


## <a name="Commands_to_manage_your_Azure_certificates"></a>���ڹ��� Azure ֤�������

Azure ����֤�������ӵ���� Azure �ʻ��� SSL ֤�顣�й� Azure ֤�����ϸ��Ϣ�������[����֤��](http://msdn.microsoft.com/zh-cn/library/azure/gg981929.aspx)��

**service cert list [options]**

�������г� Azure ֤�顣

	~$ azure service cert list
	info:   Executing command service cert list
	+ Fetching cloud services
	+ Fetching certificates
	data:   Service   Thumbprint                                Algorithm
	data:   --------  ----------------------------------------  ---------
	data:   myservice  262DBF95B5E61375FA27F1E74AC7D9EAE842916C  sha1
	info:   service cert list command OK

**service cert create &lt;dns-prefix> &lt;file> [password]**

����������֤�顣����û�����뱣����֤�飬�뽫������ʾ����Ϊ�ա�

	~$ azure service cert create nghinazz ~/publishSet.pfx
	info:   Executing command service cert create
	Cert password:
	+ Creating certificate
	info:   service cert create command OK

**service cert delete [options] &lt;thumbprint>**

������ɾ��֤�顣

	~$ azure service cert delete 262DBF95B5E61375FA27F1E74AC7D9EAE842916C
	info:   Executing command service cert delete
	+ Deleting certificate
	info:   nghinazz : cert deleted
	info:   service cert delete command OK


## <a name="Commands_to_manage_your_web_sites"></a>���ڹ�����վ������

Azure ��վ�ǿ�ͨ�� URI ���ʵ� Web ���á���վ�й���������У����������Լ����Ǵ����Ͳ������������ϸ���衣��Щ��ϸ���轫�� Azure Ϊ����ɡ�

**site list [options]**

�������г������վ��

	~$ azure site list
	info:   Executing command site list
	data:   Name            State    Host names
	data:   --------------  -------  --------------------------------------------------
	data:   mongosite       Running  mongosite.antdf0.antares.chinacloudapi.cn
	data:   myphpsite       Running  myphpsite.antdf0.antares.chinacloudapi.cn
	data:   mydrupalsite36  Running  mydrupalsite36.antdf0.antares.chinacloudapi.cn
	info:   site list command OK

**site set [options] [name]**

��������������վ [name] ������ѡ��

	~$ azure site set
	info:    Executing command site set
	Web site name: mydemosite
	+ Getting sites
	+ Updating site config information
	info:    site set command OK

**site deploymentscript [options]**

���������һ���Զ��岿��ű�

	~$ azure site deploymentscript --node
	info:    Executing command site deploymentscript
	info:    Generating deployment script for node.js Web Site
	info:    Generated deployment script files
	info:    site deploymentscript command OK

**site create [options] [name]**

��������µ���վ�ͱ���Ŀ¼��

	~$ azure site create mysite
	info:   Executing command site create
	info:   Using location northeuropewebspace
	info:   Creating a new web site
	info:   Created web site at  mysite.antdf0.antares.chinacloudapi.cn
	info:   Initializing repository
	info:   Repository initialized
	info:   site create command OK

> [AZURE.NOTE] վ�����Ʊ�����Ψһ�ġ����޷�����������վ�������ͬ DNS ���Ƶ�վ�㡣

**site browse [options] [name]**

��������������д������վ��

	~$ azure site browse mysite
	info:   Executing command site browse
	info:   Launching browser to http://mysite.antdf0.antares-test.windows-int.net
	info:   site browse command OK

**site show [options] [name]**

��������ʾ��վ����ϸ��Ϣ��

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

������ɾ��ĳ����վ��

	~$ azure site delete mysite
	info:   Executing command site delete
	info:   Deleting site mysite
	info:   Site mysite has been deleted
	info:   site delete command OK

 **site swap [options] [name]**

�������������վ��ۡ�

������֧�����¸���ѡ�

**-q �� **--quiet**������ʾȷ�ϡ����Զ����ű���ʹ�ô�ѡ�


**site start [options] [name]**

���������ĳ����վ��

	~$ azure site start mysite
	info:   Executing command site start
	info:   Starting site mysite
	info:   Site mysite has been started
	info:   site start command OK

**site stop [options] [name]**

������ֹͣĳ����վ��

	~$ azure site stop mysite
	info:   Executing command site stop
	info:   Stopping site mysite
	info:   Site mysite has been stopped
	info:   site stop command OK

**site restart [options] [name]

������ֹͣȻ�����ָ������վ��

������֧�����¸���ѡ�

**--slot** &lt;slot>��Ҫ��������Ĳ�۵����ơ�


**site location list [options]**

�������г������վλ�á�

	~$ azure site location list
	info:    Executing command site location list
	+ Getting locations
	data:    Name
	data:    ----------------
	data:    China North
	data:    China East
	info:    site location list command OK

### ���ڹ��������վӦ�ó������õ�����

**site appsetting list [options] [name]**

�������г���ӵ���վ��Ӧ�ó������á�

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

�����Ӧ�ó���������Ϊ��ֵ����ӵ������վ��

	~$ azure site appsetting add test=value
	info:    Executing command site appsetting add
	Web site name: mydemosite
	+ Getting sites
	+ Getting site config information
	+ Updating site config information
	info:    site appsetting add command OK

**site appsetting delete [options] &lt;key> [name]**

���������վ��ɾ��ָ����Ӧ�ó������á�

	~$ azure site appsetting delete test
	info:    Executing command site appsetting delete
	Web site name: mydemosite
	+ Getting sites
	+ Getting site config information
	Delete application setting test? [y/n] y
	+ Updating site config information
	info:    site appsetting delete command OK

**site appsetting show [options] &lt;key> [name]**

��������ʾָ��Ӧ�ó������õ���ϸ��Ϣ

	~$ azure site appsetting show test
	info:    Executing command site appsetting show
	Web site name: mydemosite
	+ Getting sites
	+ Getting site config information
	data:    Value:  value
	info:    site appsetting show command OK

### ���ڹ�����վ֤�������

**site cert list [options] [name]**

��������ʾ��վ֤����б��

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

��������ʾ֤����ϸ��Ϣ

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

### ���ڹ�����վ�����ַ���������

**site connectionstring list [options] [name]**

**site connectionstring add [options] &lt;connectionname> &lt;value> &lt;type> [name]**

**site connectionstring delete [options] &lt;connectionname> [name]**

**site connectionstring show [options] &lt;connectionname> [name]**

### ���ڹ�����վĬ���ĵ�������

**site defaultdocument list [options] [name]**

**site defaultdocument add [options] &lt;document> [name]**

**site defaultdocument delete [options] &lt;document> [name]**

### ���ڹ�����վ���������

**site deployment list [options] [name]**

**site deployment show [options] &lt;commitId> [name]**

**site deployment redeploy [options] &lt;commitId> [name]**

**site deployment github [options] [name]**

**site deployment user set [options] [username] [pass]**

### ���ڹ�����վ�������

**site domain list [options] [name]**

**site domain add [options] &lt;dn> [name]**

**site domain delete [options] &lt;dn> [name]**

### ���ڹ�����վ�������ӳ�������

**site handler list [options] [name]**

**site handler add [options] &lt;extension> &lt;processor> [name]**

**site handler delete [options] &lt;extension> [name]**

### ���ڹ�����վ Web ��ҵ������

**site job list [options] [name]**

�������г�ĳ����վ�µ����� web ��ҵ��

������֧�����¸���ѡ�

+ **--job-type** &lt;job-type>����ѡ��web ��ҵ�����͡���ЧֵΪ"triggered"��"continuous"��Ĭ������·���
�������͵� web ��ҵ��
+ **--slot** &lt;slot>:Ҫ��������Ĳ�۵����ơ�

**site job show [options] &lt;jobName> &lt;jobType> [name]**

��������ʾ�ض� web ��ҵ����ϸ��Ϣ��

������֧�����¸���ѡ�

+ **--job-name** &lt;job-name>�����衣web ��ҵ�����ơ�
+ **--job-type** &lt;job-type>�����衣web ��ҵ�����͡���ЧֵΪ"triggered"��"continuous"��
+ **--slot** &lt;slot>��Ҫ��������Ĳ�۵����ơ�

**site job delete [options] &lt;jobName> &lt;jobType> [name]**

������ɾ��ָ���� web ��ҵ��

������֧�����¸���ѡ�

+ **--job-name** &lt;job-name>    ���衣web ��ҵ�����ơ�
+ **--job-type** &lt;job-type>    ���衣web ��ҵ�����͡���ЧֵΪ"triggered"��"continuous"��
+ **-q** �� **--quiet**������ʾȷ�ϡ����Զ����ű���ʹ�ô�ѡ�
+ **--slot** &lt;slot>��Ҫ��������Ĳ�۵����ơ�

**site job upload [options] &lt;jobName> &lt;jobType> <jobFile> [name]**

������ɾ��ָ���� web ��ҵ��

������֧�����¸���ѡ�

+ **--job-name** &lt;job-name>�����衣web ��ҵ�����ơ�
+ **--job-type** &lt;job-type>�����衣web ��ҵ�����͡���ЧֵΪ"triggered"��"continuous"��
+ **--job-file** &lt;job-file>�����衣��ҵ�ļ���
+ **--slot** &lt;slot>��Ҫ��������Ĳ�۵����ơ�

**site job start [options] &lt;jobName> &lt;jobType> [name]**

���������ָ���� web ��ҵ��

������֧�����¸���ѡ�

+ **--job-name** &lt;job-name>�����衣web ��ҵ�����ơ�
+ **--job-type** &lt;job-type>�����衣web ��ҵ�����͡���ЧֵΪ"triggered"��"continuous"��
+ **--slot** &lt;slot>��Ҫ��������Ĳ�۵����ơ�

**site job stop [options] &lt;jobName> &lt;jobType> [name]**

������ָֹͣ���� web ��ҵ��ֻ��������ҵ����ֹͣ��

������֧�����¸���ѡ�

+ **--job-name** &lt;job-name>�����衣web ��ҵ�����ơ�
+ **--slot** &lt;slot>��Ҫ��������Ĳ�۵����ơ�

### ���ڹ�����վ Web ��ҵ��ʷ��¼������

**site job history list [options] [jobName] [name]**

��������ʾָ���� web ��ҵ��������ʷ��¼��

������֧�����¸���ѡ�

+ **--job-name** &lt;job-name>�����衣web ��ҵ�����ơ�
+ **--slot** &lt;slot>��Ҫ��������Ĳ�۵����ơ�

**site job history show [options] [jobName] [runId] [name]**

�������ȡΪָ���� web ��ҵ���е���ҵ����ϸ��Ϣ��

������֧�����¸���ѡ�

+ **--job-name** &lt;job-name>�����衣web ��ҵ�����ơ�
+ **--run-id** &lt;run-id>����ѡ��������ʷ��¼�� id�����δָ��������ʾ�������С�
+ **--slot** &lt;slot>��Ҫ��������Ĳ�۵����ơ�

### ���ڹ�����վ��ϵ�����

**site log download [options] [name]**

���ذ��������վ��ϵ� .zip �ļ���

	~$ azure site log download
	info:    Executing command site log download
	Web site name: mydemosite
	+ Getting sites
	+ Getting site information
	+ Downloading diagnostic log to diagnostics.zip
	info:    site log download command OK

**site log tail [options] [name]**

���������ն����ӵ���־��ʽ�������

	~$ azure site log tail
	info:    Executing command site log tail
	Web site name: mydemosite
	+ Getting sites
	+ Getting site information
	2013-11-19T17:24:17  Welcome, you are now connected to log-streaming service.

**site log set [options] [name]**

���������������վ�����ѡ�

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

### ���ڹ�����վ�洢�������

**site repository branch [options] &lt;branch> [name]**

**site repository delete [options] [name]**

**site repository sync [options] [name]**

### ���ڹ�����վ���ŵ�����

**site scale mode [options] &lt;mode> [name]**

**site scale instances [options] &lt;instances> [name]**


## <a name="Commands_to_manage_mobile_services"></a>���ڹ��� Azure �ƶ����������

Azure �ƶ���������һϵ��֧�����Ӧ�ó���ĺ�˹��ܵ� Azure �����ƶ����������Ϊ���¼��ࣺ

+ [���ڹ����ƶ�����ʵ��������](#Mobile_Services)
+ [���ڹ����ƶ��������õ�����](#Mobile_Configuration)
+ [���ڹ����ƶ�����������](#Mobile_Tables)
+ [���ڹ����ƶ�����ű�������](#Mobile_Scripts)
+ [���ڹ���ƻ�����ҵ������](#Mobile_Jobs)
+ [���������ƶ����������](#Mobile_Scale)

����ѡ�������ڶ����ƶ��������

+ **-h** �� **--help**����ʾ����÷���Ϣ��
+ **-s `<id>`** �� **--subscription `<id>`**��ʹ��ָ��Ϊ `<id>` ���ض����ġ�
+ **-v** �� **--verbose**��д����ϸ�����
+ **--json**��д�� JSON �����

### <a name="Mobile_Services"></a>���ڹ����ƶ�����ʵ��������

**mobile locations [options]**

�������г��ƶ�����֧�ֵĵ���λ�á�

	~$ azure mobile locations
	+ Getting mobile service locations
	info:    Executing command mobile locations
	info:    China North (default)

**mobile create [options] [servicename] [sqlAdminUsername] [sqlAdminPassword]**

��������ƶ������Լ� SQL ���ݿ�ͷ�������

	~$ azure mobile create todolist your_login_name Secure$Password
	info:    Executing command mobile create
	+ Creating mobile service
	info:    Overall application state: Healthy
	info:    Mobile service (todolist) state: ProvisionConfigured
	info:    SQL database (todolist_db) state: Provisioned
	info:    SQL server (e96ean1c6v) state: ProvisionConfigured
	info:    mobile create command OK

������֧�����¸���ѡ�

+ **-r `<sqlServer>`** �� **--sqlServer `<sqlServer>`**��ʹ��ָ��Ϊ `<sqlServer>` ������ SQL ���ݿ��������
+ **-d `<sqlDb>`** �� **--sqlDb `<sqlDb>`**��ʹ��ָ��Ϊ `<sqlDb>` ������ SQL ���ݿ⡣
+ **-l `<location>`** �� **--location `<location>`**����ָ��Ϊ `<location>` ���ض�λ���д��������������� azure mobile locations �ɻ�ȡ����λ�á�
+ **--sqlLocation `<location>`**�����ض��� `<location>` �д��� SQL ��������Ĭ��Ϊ�ƶ������λ�á�

**mobile delete [options] [servicename]**

������ɾ���ƶ������� SQL ���ݿ�ͷ�������

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

������֧�����¸���ѡ�

+ **-d** �� **--deleteData**���Ӵ��ƶ���������ݿ���ɾ���������ݡ�
+ **-a** �� **--deleteAll**��ɾ�� SQL ���ݿ�ͷ�������
+ **-q** �� **--quiet**������ʾȷ�ϡ����Զ����ű���ʹ�ô�ѡ�

**mobile list [options]**

�������г�����ƶ�����

	~$ azure mobile list
	info:    Executing command mobile list
	data:    Name          State  URL
	data:    ------------  -----  --------------------------------------
	data:    todolist      Ready  https://todolist.azure-mobile.cn/
	data:    mymobileapp   Ready  https://mymobileapp.azure-mobile.cn/
	info:    mobile list command OK

**mobile show [options] [servicename]**

��������ʾ�й��ƶ��������ϸ��Ϣ��

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
	data:    applicationUrl https://todolist.azure-mobile.cn/
	data:    applicationKey XXXXXXXXXXXXXXXXXXXXXXXXXXXXXX
	data:    masterKey XXXXXXXXXXXXXXXXXXXXXXXXXXXXXX
	data:    webspace WESTUSWEBSPACE
	data:    region China North
	data:    tables TodoItem
	info:    mobile show command OK

**mobile restart [options] [servicename]**

��������������ƶ�����ʵ����

	~$ azure mobile restart todolist
	info:    Executing command mobile restart
	+ Restarting mobile service
	info:    Service was restarted.
	info:    mobile restart command OK

**mobile log [options] [servicename]**

��������ƶ�������־��ɸѡ���  `error` ֮���������־���͡�

	~$ azure mobile log todolist -t error
	info:    Executing command mobile log
	data:
	data:    timeCreated 2013-01-07T16:04:43.351Z
	data:    type error
	data:    source /scheduler/TestingLogs.js
	data:    message This is an error.
	data:
	info:    mobile log command OK

������֧�����¸���ѡ�

+ **-r `<query>`** �� **--query `<query>`**��ִ��ָ������־��ѯ��
+ **-t `<type>`** �� **--type `<type>`**������Ŀ `<type>`��������  `information`�� `warning` ��  `error`��ɸѡ���ص���־��
+ **-k `<skip>`** �� **--skip `<skip>`**�������� `<skip>` ָ����������
+ **-p `<top>`** �� **--top `<top>`**�������� `<top>` ָ�����ض�������

> [AZURE.NOTE] **--query** ���������� **--type**��**--skip** �� **--top**��

**mobile recover [options] [unhealthyservicename] [healthyservicename]**

������ͨ�������������ƶ������ƶ�����ͬ�����е������ƶ�����������ָ���

������֧�����¸���ѡ�

**-q** �� **--quiet**������ʾȷ�ϻָ���

**mobile key regenerate [options] [servicename] [type]**

���������������ƶ�����Ӧ�ó�����Կ��

	~$ azure mobile key regenerate todolist application
	info:    Executing command mobile key regenerate
	info:    New application key is SmLorAWVfslMcOKWSsuJvuzdJkfUpt40
	info:    mobile key regenerate command OK

��Կ����Ϊ  `master` ��  `application`��

> [AZURE.NOTE] ������������Կʱ��ʹ�þ���Կ�Ŀͻ��˿����޷���������ƶ����񡣵���������Ӧ�ó�����Կʱ��Ӧʹ������Կֵ�������Ӧ�ó���

**mobile key set [options] [servicename] [type] [value]**

������ƶ�������Կ����Ϊһ���ض�ֵ��


### <a name="Mobile_Configuration"></a>���ڹ����ƶ��������õ�����

**mobile config list [options] [servicename]**

�������г��ƶ����������ѡ�

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

�������ȡ�ƶ�������ض�����ѡ��ڴ�ʾ����Ϊ��̬�ܹ���

	~$ azure mobile config get todolist dynamicSchemaEnabled
	info:    Executing command mobile config get
	data:    dynamicSchemaEnabled true
	info:    mobile config get command OK

**mobile config set [options] [servicename] [key] [value]**

�����������ƶ�������ض�����ѡ��ڴ�ʾ����Ϊ��̬�ܹ���

	~$ azure mobile config set todolist dynamicSchemaEnabled false
	info:    Executing command mobile config set
	info:    mobile config set command OK


### <a name="Mobile_Tables"></a>���ڹ����ƶ�����������

**mobile table list [options] [servicename]**

�������г�����ƶ������е����б��

	~$azure mobile table list todolist
	info:    Executing command mobile table list
	data:    Name      Indexes  Rows
	data:    --------  -------  ----
	data:    Channel   1        0
	data:    TodoItem  1        0
	info:    mobile table list command OK

**mobile table show [options] [servicename] [tablename]**

��������ʾ�й��ض���ķ������ݵ���ϸ��Ϣ��

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

����������

	~$azure mobile table create todolist Channels
	info:    Executing command mobile table create
	+ Creating table
	info:    mobile table create command OK

������֧�����¸���ѡ�

+ **-p `&lt;permissions>`** �� **--permissions `&lt;permissions>`**���Զ��ŷָ�� `<operation>`=`<permission>` �Ե��б�����У�`<operation>`  ��  `insert`�� `read`�� `update` ��  `delete`��`&lt;permissions>` ��  `public`�� `application`��Ĭ��ֵ�� `user` ��  `admin`��

**mobile data read [options] [servicename] [tablename] [query]**

�������ȡ���е����ݡ�

	~$azure mobile data read todolist TodoItem
	info:    Executing command mobile data read
	data:    id  text     complete
	data:    --  -------  --------
	data:    1   item #1  false
	data:    2   item #2  true
	data:    3   item #3  false
	data:    4   item #4  true
	info:    mobile data read command OK

������֧�����¸���ѡ�

+ **-k `<skip>`** �� **--skip `<skip>`**�������� `<skip>` ָ����������
+ **-t `<top>`** �� **--top `<top>`**�������� `<top>` ָ�����ض�������
+ **-l** �� **--list**�����б��ʽ�������ݡ�

**mobile table update [options] [servicename] [tablename]**

������ֻ���Ĺ���Ա�Ա��ɾ��Ȩ�ޡ�

	~$azure mobile table update todolist Channels -p delete=admin
	info:    Executing command mobile table update
	+ Updating permissions
	info:    Updated permissions
	info:    mobile table update command OK

������֧�����¸���ѡ�

+ **-p `&lt;permissions>`** �� **--permissions `&lt;permissions>`**���Զ��ŷָ�� `<operation>`=`<permission>` �Ե��б�����У�`<operation>` ��  `insert`�� `read`�� `update` ��  `delete`��`&lt;permissions>` �� `public`�� `application`��Ĭ��ֵ���� `user` ��  `admin`��
+ **--deleteColumn `<columns>`**���Զ��ŷָ��Ҫɾ�����У�ָ��Ϊ `<columns>`�����б��
+ **-q** �� **--quiet**��ɾ���ж�����ʾȷ��
+ **--addIndex `<columns>`**��Ҫ�����������е��еĶ��ŷָ��б��
+ **--deleteIndex `<columns>`**��Ҫ���������ų����еĶ��ŷָ��б��

**mobile table delete [options] [servicename] [tablename]**

������ɾ��ĳ�����

	~$azure mobile table delete todolist Channels
	info:    Executing command mobile table delete
	Do you really want to delete the table (yes/no): yes
	+ Deleting table
	info:    mobile table delete command OK

ָ�� -q �������ڲ���ʾȷ�ϵ������ɾ�����ִ�д˲������Է�ֹ��ֹ�Զ����ű���

**mobile data truncate [options] [servicename] [tablename]**

������ӱ���ɾ�����������С�

	~$azure mobile data truncate todolist TodoItem
	info:    Executing command mobile data truncate
	info:    There are 7 data rows in the table.
	Do you really want to delete all data from the table? (y/n): y
	info:    Deleted 7 rows.
	info:    mobile data truncate command OK


### <a name="Mobile_Scripts"></a>���ڹ���ű�������

�������е��������ڹ��������ƶ�����ķ������ű����й���ϸ��Ϣ�������[ʹ���ƶ������еķ������ű�](/documentation/articles/mobile-services-how-to-use-server-scripts/)��

**mobile script list [options] [servicename]**

�������г���ע��Ľű���������ͼƻ�����ű���

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

���������ű��� TodoItem �����ص�  `table` ���ļ�������Ϊ  `todoitem.insert.js` ���ļ��С�

	~$azure mobile script download todolist table/todoitem.insert.js
	info:    Executing command mobile script download
	info:    Saved script to ./table/todoitem.insert.js
	info:    mobile script download command OK

������֧�����¸���ѡ�

+ **-p `<path>`** �� **--path `<path>`**���ļ������ڱ���ű���λ�ã����е�ǰ����Ŀ¼��Ĭ��ֵ��
+ **-f `<file>`** �� **--file `<file>`**��Ҫ���ű����������е��ļ������ơ�
+ **-o** �� **--override**�����������ļ���
+ **-c** �� **--console**�����ű�д�뵽����̨�������ļ���

**mobile script upload [options] [servicename] [scriptname]**

�������  `table` ���ļ���������Ϊ  `todoitem.insert.js` ���½ű���

	~$azure mobile script upload todolist table/todoitem.insert.js
	info:    Executing command mobile script upload
	info:    mobile script upload command OK

���ļ������Ʊ����ɱ�Ͳ���������ɣ����Ҹ��ļ�����λ�����������ִ��λ�õ� table ���ļ����С��㻹����ʹ�� **-f `<file>`** �� **--file `<file>`** ����ָ����������Ҫע��Ľű����ļ����ļ�����·����


**mobile script delete [options] [servicename] [scriptname]**

������� TodoItem ����ɾ�����в���ű���

	~$azure mobile script delete todolist table/todoitem.insert.js
	info:    Executing command mobile script delete
	info:    mobile script delete command OK

### <a name="Mobile_Jobs"></a>���ڹ����Ѽƻ���ҵ������

�������е��������ڹ��������ƶ�������Ѽƻ���ҵ���й���ϸ��Ϣ�������[�ƻ���ҵ](https://msdn.microsoft.com/zh-cn/library/azure/jj860528.aspx)��

**mobile job list [options] [servicename]**

�������г��Ѽƻ���ҵ��

	~$azure mobile job list todolist
	info:    Executing command mobile job list
	info:    Scheduled jobs
	data:    Job name    Script name           Status    Interval     Last run              Next run
	data:    ----------  --------------------  --------  -----------  --------------------  --------------------
	data:    getUpdates  scheduler/getUpdates  enabled   15 [minute]  2013-01-14T16:15:00Z  2013-01-14T16:30:00Z
	info:    You can manipulate scheduled job scripts using the 'azure mobile script' command.
	info:    mobile job list command OK

**mobile job create [options] [servicename] [jobname]**

��������ƻ�ΪÿСʱ���е���Ϊ  `getUpdates` ������ҵ��

	~$azure mobile job create -i 1 -u hour todolist getUpdates
	info:    Executing command mobile job create
	info:    Job was created in disabled state. You can enable the job using the 'azure mobile job update' command.
	info:    You can manipulate the scheduled job script using the 'azure mobile script' command.
	info:    mobile job create command OK

������֧�����¸���ѡ�

+ **-i `<number>`** �� **--interval `<number>`**����ҵʱ�������ֵ����Ϊ������Ĭ��ֵΪ `15`��
+ **-u `<unit>`** �� **--intervalUnit `<unit>`**��_ʱ����_�ĵ�λ������������ֵ֮һ��
	+ **minute**��Ĭ��ֵ��
	+ **hour**
	+ **day**
	+ **month**
	+ **none**��������ҵ��
+ **-t `<time>`** **--startTime `<time>`**�ű����״����п�ʼʱ�䣬���� ISO ��ʽ��Ĭ��ֵΪ  `now`��

> [AZURE.NOTE] ����������ҵ���ڽ���״̬����Ϊ���������ؽű�����ʹ�� **mobile script upload** �������ؽű���ʹ�� **mobile job update** ����������ҵ��

**mobile job update [options] [servicename] [jobname]**

�������������ѽ��õ�  `getUpdates` ��ҵ��

	~$azure mobile job update -a enabled todolist getUpdates
	info:    Executing command mobile job update
	info:    mobile job update command OK

������֧�����¸���ѡ�

+ **-i `<number>`** �� **--interval `<number>`**����ҵʱ�������ֵ����Ϊ������Ĭ��ֵΪ"15"��
+ **-u `<unit>`** �� **--intervalUnit `<unit>`**��_ʱ����_�ĵ�λ������������ֵ֮һ��
	+ **minute**��Ĭ��ֵ��
	+ **hour**
	+ **day**
	+ **month**
	+ **none**��������ҵ��
+ **-t `<time>`** **--startTime `<time>`**�ű����״����п�ʼʱ�䣬���� ISO ��ʽ��Ĭ��ֵΪ  `now`��
+ **-a `<status>`** �� **--status `<status>`**����ҵ״̬��������  `enabled` ��  `disabled`��

**mobile job delete [options] [servicename] [jobname]**

������� TodoList ��������ɾ�� getUpdates �Ѽƻ���ҵ��

	~$azure mobile job delete todolist getUpdates
	info:    Executing command mobile job delete
	info:    mobile job delete command OK

> [AZURE.NOTE] ɾ����ҵҲ��ɾ�������صĽű���

### <a name="Mobile_Scale"></a>���������ƶ����������

�������е��������������ƶ������й���ϸ��Ϣ�������[�����ƶ�����](https://msdn.microsoft.com/zh-cn/library/azure/jj193178.aspx)��

**mobile scale show [options] [servicename]**

��������ʾ������Ϣ��������ǰ����ģʽ��ʵ������

	~$azure mobile scale show todolist
	info:    Executing command mobile scale show
	data:    webspace WESTUSWEBSPACE
	data:    computeMode Free
	data:    numberOfInstances 1
	info:    mobile scale show command OK

**mobile scale change [options] [servicename]**

������ƶ�����Ĺ�ģ�����ģʽ����Ϊ�߼�ģʽ��

	~$azure mobile scale change -c Reserved -i 1 todolist
	info:    Executing command mobile scale change
	+ Rescaling the mobile service
	info:    mobile scale change command OK

������֧�����¸���ѡ�

+ **-c `<mode>`** �� **--computeMode `<mode>`**������ģʽ����Ϊ  `Free` ��  `Reserved`��
+ **-i `<count>`** �� **--numberOfInstances `<count>`**���ڱ���ģʽ������ʱʹ�õ�ʵ������

> [AZURE.NOTE] ������ģʽ����Ϊ  `Reserved` ʱ��ͬһ�����е������ƶ����񶼽��ڸ߼�ģʽ�����С�


### ����Ϊ�ƶ���������Ԥ���湦�ܵ�����

**mobile preview list [options] [servicename]**

��������ʾ��ָ���ķ����Ͽ��õ�Ԥ���湦�ܣ��Լ������Ƿ������á�

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

������Ϊ�ƶ���������ָ����Ԥ���湦�ܡ���ע�⣬һ�����ã����޷�Ϊ�ƶ��������Ԥ���湦�ܡ�

### ���ڹ����ƶ����� API ������

**mobile api list [options] [servicename]**

�������г���Ϊ����ƶ����񴴽����ƶ������Զ��� API��

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

�����ƶ������Զ��� API

	~$ azure mobile api create mysite myCustomRetrieveAPI
	info:    Executing command mobile api create
	+ Creating custom API: 'myCustomRetrieveAPI'
	info:    API was created successfully. You can modify the API using the 'azure mobile script' command.
	info:    mobile api create command OK

������֧�����¸���ѡ�

**-p** �� **--permissions** &lt;permissions>��  �Զ��ŷָ�� &lt;method>=&lt;permission> �Ե��б��

**mobile api update [options] [servicename] [apiname]**

���������ָ�����ƶ������Զ��� API��

������֧�����¸���ѡ�

������֧�����¸���ѡ�

+ **-p** �� **--permissions** &lt;permissions>���Զ��ŷָ�� &lt;method>=&lt;permission>  �Ե��б��
+ **-f** �� **--force**�����Ƕ�Ȩ��Ԫ�����ļ����κ��Զ�����ġ�

**mobile api delete [options] [servicename] [apiname]**

	~$ azure mobile api delete mysite myCustomRetrieveAPI
	info:    Executing command mobile api delete
	+ Deleting API: 'myCustomRetrieveAPI'
	info:    mobile api delete command OK

������ɾ��ָ�����ƶ������Զ��� API��

### ���ڹ����ƶ�Ӧ�ó����Ӧ�ó������õ�����

**mobile appsetting list [options] [servicename]**

��������ʾָ���ķ�����ƶ�Ӧ�ó����Ӧ�ó������á�

	~$ azure mobile appsetting list mysite
	info:    Executing command mobile appsetting list
	+ Retrieving app settings
	data:    Name               Value
	data:    -----------------  -----
	data:    enablebetacontent  true
	info:    mobile appsetting list command OK

**mobile appsetting add [options] [servicename] [name] [value]**

������Ϊ�ƶ���������Զ���Ӧ�ó������á�

	~$ azure mobile appsetting add mysite enablebetacontent true
	info:    Executing command mobile appsetting add
	+ Retrieving app settings
	+ Adding app setting
	info:    mobile appsetting add command OK

**mobile appsetting delete [options] [servicename] [name]**

������Ϊ�ƶ�����ɾ��ָ����Ӧ�ó������á�

	~$ azure mobile appsetting delete mysite enablebetacontent
	info:    Executing command mobile appsetting delete
	+ Retrieving app settings
	+ Removing app setting 'enablebetacontent'
	info:    mobile appsetting delete command OK

**mobile appsetting show [options] [servicename] [name]**

������Ϊ�ƶ�����ɾ��ָ����Ӧ�ó������á�

	~$ azure mobile appsetting show mysite enablebetacontent
	info:    Executing command mobile appsetting show
	+ Retrieving app settings
	info:    enablebetacontent: true
	info:    mobile appsetting show command OK

## <a name="Manage_tool_local_settings"></a>������߱�������

����������ָ��Ķ��� ID ��Ĭ�ϴ洢�ʻ����ơ�

**config list [options]**

��������ʾ�������á�

	~$ azure config list
	info:   Displaying config settings
	data:   Setting                Value
	data:   ---------------------  ------------------------------------
	data:   subscription           32-digit-subscription-key
	data:   defaultStorageAccount  name

**config set [options] &lt;name&gt;,&lt;value&gt;**

����������������á�

	~$ azure config set defaultStorageAccount myname
	info:   Setting 'defaultStorageAccount' to value 'myname'
	info:   Changes saved.

## <a name ="Commands_to_manage_service_bus"></a>���ڹ��� Service Bus ������

ʹ����Щ������������� Service Bus �ʻ�

**sb namespace check [options] &lt;name>**

���ĳ�� service bus �����ռ��Ƿ�Ϸ������á�

**sb namespace create &lt;name> &lt;location>**

�����µ� Service Bus �����ռ䡣

	~$ azure sb namespace create mysbnamespacea-test "China North"
	info:    Executing command sb namespace create
	+ Creating namespace mysbnamespacea-test in region China North
	data:    name: mysbnamespacea-test
	data:    region: China North
	data:    status: Activating
	data:    createdAt: Fri Mar 20 2015 11:07:22 GMT+0800 (�й���׼ʱ��)
	data:    acsManagementEndpoint: https://mysbnamespacea-test-sb.accesscontrol.chinacloudapi.cn/
	data:    serviceBusEndpoint: https://mysbnamespacea-test.servicebus.chinacloudapi.cn/
	data:    subscriptionId: c333413ef84b4cc2944efe29b05c237f
	data:    enabled: true
	info:    sb namespace create command OK


**sb namespace delete &lt;name>**

ɾ��ĳ�������ռ䡣

	~$ azure sb namespace delete mysbnamespacea-test
	info:    Executing command sb namespace delete
	Delete namespace mysbnamespacea-test? [y/n] y
	+ Deleting namespace mysbnamespacea-test
	info:    sb namespace delete command OK

**sb namespace list**

�г�Ϊ����ʻ����������������ռ䡣

	~$ azure sb namespace list
	info:    Executing command sb namespace list
	+ Getting namespaces
	data:    Name                 Region       Status
	data:    -------------------  -----------  ------
	data:    mysbnamespacea-test  China North  Active
	info:    sb namespace list command OK


**sb namespace location list**

��ʾ���п��õ������ռ�λ�õ��б��

	~$ azure sb namespace location list
	info:    Executing command sb namespace location list
	+ Getting locations
	data:    Name              Code
	data:    ----------------  ----------------
	data:    China East   China East
	data:    China North  China North
	info:    sb namespace location list command OK

**sb namespace show &lt;name>**

��ʾ�й��ض������ռ����ϸ��Ϣ��

	~$ azure sb namespace show mysbnamespacea-test
	info:    Executing command sb namespace show
	+ Getting namespace
	data:    Name: mysbnamespacea-test
	data:    region: China North
	data:    status: Activating
	data:    createdAt: Fri Mar 20 2015 11:10:07 GMT+0800 (�й���׼ʱ��)
	data:    acsManagementEndpoint: https://mysbnamespacea-test-sb.accesscontrol.chinacloudapi.cn/
	data:    serviceBusEndpoint: https://mysbnamespacea-test.servicebus.chinacloudapi.cn/
	data:    subscriptionId: c333413ef84b4cc2944efe29b05c237f
	data:    enabled: true
	info:    sb namespace show command OK

**sb namespace verify &lt;name>**

��������ռ��Ƿ���á�

## <a name="Commands_to_manage_your_Storage_objects"></a>���ڹ���洢���������

### ���ڹ���洢�ʻ�������

**storage account list [options]**

��������ʾ��Ķ����ϵĴ洢�ʻ���

	~$ azure storage account list
	info:    Executing command storage account list
	+ Getting storage accounts
	data:    Name             Label  Location
	data:    ---------------  -----  --------
	data:    mybasestorage           China North
	info:    storage account list command OK

**storage account show [options] <name>**

��������ʾ�й�ָ���Ĵ洢�ʻ�����Ϣ������ URI ���ʻ����ԡ�

**storage account create [options] <name>**

����������ṩ��ѡ����洢�ʻ���

	~$ azure storage account create mybasestorage --label PrimaryStorage --location "China North"
	info:    Executing command storage account create
	+ Creating storage account
	info:    storage account create command OK

������֧�����¸���ѡ�

+ **-e** �� **--label** &lt;label>���洢�ʻ��ı�ǩ��
+ **-d** �� **--description** &lt;description>���洢�ʻ���˵����
+ **-l** �� **--location** &lt;name>��Ҫ�����д����洢�ʻ��ĵ�������
+ **-a** �� **--affinity-group** &lt;name>��Ҫ��洢�ʻ������ĵ�Ե�顣
+ **--geoReplication**��ָʾ�Ƿ����õ����ơ�
+ **--disable-geoReplication**��ָʾ�Ƿ���õ����ơ�

**storage account set [options] <name>**

���������ָ���Ĵ洢�ʻ���

	~$ azure storage account set mybasestorage --geoReplication
	info:    Executing command storage account set
	+ Updating storage account
	info:    storage account set command OK

������֧�����¸���ѡ�

+ **-e** or **--label** &lt;label>���洢�ʻ��ı�ǩ��
+ **-d** �� **--description** &lt;description>���洢�ʻ���˵����
+ **-l** �� **--location** &lt;name>��Ҫ�����д����洢�ʻ��ĵ�������
+ **--geoReplication**��ָʾ�Ƿ����õ����ơ�
+ **--disable-geoReplication**��ָʾ�Ƿ���õ����ơ�

**storage account delete [options] <name>**

������ɾ��ָ���Ĵ洢�ʻ���

������֧�����¸���ѡ�

**-q** �� **--quiet**������ʾȷ�ϡ����Զ����ű���ʹ�ô�ѡ�

### ���ڹ���洢�ʻ���Կ������

**storage account keys list [options] <name>**

�������г�ָ���Ĵ洢�ʻ�����Ҫ�͸�����Կ��

**storage account keys renew [options] <name>**

### ���ڹ���洢����������

**storage container list [options] [prefix]**

��������ʾָ���Ĵ洢�ʻ��Ĵ洢�����б���洢�ʻ���ͨ�������ַ������ߴ洢�ʻ����ƺ��ʻ���Կָ���ġ�

������֧�����¸���ѡ�

+ **-p** �� **-prefix** &lt;prefix>���洢��������ǰ׺��
+ **-a** �� **--account-name** &lt;accountName>���洢�ʻ����ơ�
+ **-k** �� **--account-key** &lt;accountKey>���洢�ʻ���Կ��
+ **-c** �� **--connection-string** &lt;connectionString>���洢�����ַ�����
+ **--debug**���ڵ���ģʽ������ storage ���

**storage container show [options] [container]**
**storage container create [options] [container]**

������Ϊָ���Ĵ洢�ʻ������洢�������洢�ʻ���ͨ�������ַ������ߴ洢�ʻ����ƺ��ʻ���Կָ���ġ�

������֧�����¸���ѡ�

+ **--container** &lt;container>��Ҫ�����Ĵ洢���������ơ�
+ **-p** �� **-prefix** &lt;prefix>���洢��������ǰ׺��
+ **-a** �� **--account-name** &lt;accountName>���洢�ʻ�����
+ **-k** �� **--account-key** &lt;accountKey>���洢�ʻ���Կ
+ **-c** �� **--connection-string** &lt;connectionString>���洢�����ַ���
+ **--debug**���ڵ���ģʽ������ storage ���

**storage container delete [options] [container]**

������ɾ��ָ���Ĵ洢�������洢�ʻ���ͨ�������ַ������ߴ洢�ʻ����ƺ��ʻ���Կָ���ġ�

������֧�����¸���ѡ�

+ **--container** &lt;container>��Ҫ�����Ĵ洢���������ơ�
+ **-p** �� **-prefix** &lt;prefix>���洢��������ǰ׺��
+ **-a** �� **--account-name** &lt;accountName>���洢�ʻ����ơ�
+ **-k** �� **--account-key** &lt;accountKey>���洢�ʻ���Կ��
+ **-c** �� **--connection-string** &lt;connectionString>���洢�����ַ�����
+ **--debug**���ڵ���ģʽ������ storage ���

**storage container set [options] [container]**

���������ô洢�����ķ��ʿ����б���洢�ʻ���ͨ�������ַ������ߴ洢�ʻ����ƺ��ʻ���Կָ���ġ�

������֧�����¸���ѡ�

+ **--container** &lt;container>��Ҫ�����Ĵ洢���������ơ�
+ **-p** �� **-prefix** &lt;prefix>���洢��������ǰ׺��
+ **-a** �� **--account-name** &lt;accountName>���洢�ʻ����ơ�
+ **-k** �� **--account-key** &lt;accountKey>���洢�ʻ���Կ��
+ **-c** �� **--connection-string** &lt;connectionString>���洢�����ַ�����
+ **--debug**���ڵ���ģʽ������ storage ���

### ���ڹ���洢 blob ������

**storage blob list [options] [container] [prefix]**

�������ָ���Ĵ洢�����еĴ洢 blob ���б��

������֧�����¸���ѡ�

+ **--container** &lt;container>��Ҫ�����Ĵ洢���������ơ�
+ **-p** �� **-prefix** &lt;prefix>���洢��������ǰ׺��
+ **-a** �� **--account-name** &lt;accountName>���洢�ʻ����ơ�
+ **-k** �� **--account-key** &lt;accountKey>���洢�ʻ���Կ��
+ **-c** �� **--connection-string** &lt;connectionString>���洢�����ַ�����
+ **--debug**���ڵ���ģʽ������ storage ���

**storage blob show [options] [container] [blob]**

��������ʾָ���Ĵ洢 blob ����ϸ��Ϣ��

������֧�����¸���ѡ�

+ **--container** &lt;container>��Ҫ�����Ĵ洢���������ơ�
+ **-p** �� **-prefix** &lt;prefix>���洢��������ǰ׺��
+ **-a** �� **--account-name** &lt;accountName>���洢�ʻ����ơ�
+ **-k** �� **--account-key** &lt;accountKey>���洢�ʻ���Կ��
+ **-c** �� **--connection-string** &lt;connectionString>���洢�����ַ�����
+ **--debug**���ڵ���ģʽ������ storage ���

**storage blob delete [options] [container] [blob]**

������֧�����¸���ѡ�

+ **--container** &lt;container>��Ҫ�����Ĵ洢���������ơ�
+ **-b** �� **--blob** &lt;blobName>��Ҫɾ���Ĵ洢 blob �����ơ�
+ **-q** �� **--quiet**��ɾ��ָ���Ĵ洢 blob �Ҳ�ȷ�ϡ�
+ **-a** �� **--account-name** &lt;accountName>���洢�ʻ����ơ�
+ **-k** �� **--account-key** &lt;accountKey>���洢�ʻ���Կ��
+ **-c** �� **--connection-string** &lt;connectionString>���洢�����ַ�����
+ **--debug**���ڵ���ģʽ������ storage ���

**storage blob upload [options] [file] [container] [blob]**

�����ָ�����ļ����ص�ָ���Ĵ洢 blob��

������֧�����¸���ѡ�

+ **--container** &lt;container>��Ҫ�����Ĵ洢���������ơ�
+ **-b** �� **--blob** &lt;blobName>��Ҫ���صĴ洢 blob �����ơ�
+ **-t** �� **--blobtype** &lt;blobtype>���洢 blob ���ͣ�Page �� Block��
+ **-p** �� **--properties** &lt;properties>�����ص��ļ��Ĵ洢 blob ���ԡ��������Էֺ� (;) �ָ��"��=ֵ"�ԡ����õ������� contentType��contentEncoding��contentLanguage �� cacheControl��
+ **-m** �� **--metadata** &lt;metadata>�����ص��ļ��Ĵ洢 blob Ԫ���ݡ�Ԫ�������Էֺ� (;) �ָ��"��=ֵ"�ԡ�
+ **--concurrenttaskcount** &lt;concurrenttaskcount>��������������������Ŀ��
+ **-q** �� **--quiet**������ָ���Ĵ洢 blob �Ҳ�ȷ�ϡ�
+ **-a** �� **--account-name** &lt;accountName>���洢�ʻ����ơ�
+ **-k** �� **--account-key** &lt;accountKey>���洢�ʻ���Կ��
+ **-c** �� **--connection-string** &lt;connectionString>���洢�����ַ�����
+ **--debug**���ڵ���ģʽ������ storage ���

**storage blob download [options] [container] [blob] [destination]**

����������ָ���Ĵ洢 blob��

������֧�����¸���ѡ�

+ **--container** &lt;container>��Ҫ�����Ĵ洢���������ơ�
+ **-b** �� **--blob** &lt;blobName>���洢 blob ���ơ�
+ **-d** �� **--destination** [destination]������Ŀ����ļ���Ŀ¼·����
+ **-m** �� **--checkmd5**�����ص��ļ���У�� md5sum��
+ **--concurrenttaskcount** &lt;concurrenttaskcount>  ������������������Ŀ
+ **-q** �� **--quiet**������Ŀ���ļ��Ҳ�ȷ�ϡ�
+ **-a** �� **--account-name** &lt;accountName>���洢�ʻ����ơ�
+ **-k** �� **--account-key** &lt;accountKey>���洢�ʻ���Կ��
+ **-c** �� **--connection-string** &lt;connectionString>���洢�����ַ�����
+ **--debug**���ڵ���ģʽ������ storage ���

## <a name ="Commands_to_manage_sql"></a>���ڹ��� SQL ���ݿ������

ʹ����Щ������������� Azure SQL ���ݿ�

### ���ڹ��� SQL Server �����ݿ�

ʹ����Щ������������� SQL Server

**sql server create &lt;administratorLogin> &lt;administratorPassword> &lt;location>**

�����µ����ݿ������

	~$ azure sql server create test T3stte$t "China North"
	info:    Executing command sql server create
	+ Creating SQL Server
	data:    Server Name i1qwc540ts
	info:    sql server create command OK

**sql server show &lt;name>**

��ʾ��������ϸ��Ϣ��

	~$ azure sql server show xclfgcndfg
	info:    Executing command sql server show
	+ Getting SQL server
	data:    SQL Server Name xclfgcndfg
	data:    SQL Server AdministratorLogin msopentechforums
	data:    SQL Server Location China North
	data:    SQL Server FullyQualifiedDomainName xclfgcndfg.database.chinacloudapi.cn
	info:    sql server show command OK

**sql server list**

��ȡ���������б��

	~$ azure sql server list
	info:    Executing command sql server list
	+ Getting SQL server
	data:    Name        Location
	data:    ----------  --------
	data:    xclfgcndfg  China North
	info:    sql server list command OK

**sql server delete &lt;name>**

ɾ��������

	~$ azure sql server delete i1qwc540ts
	info:    Executing command sql server delete
	Delete server i1qwc540ts? [y/n] y
	+ Removing SQL Server
	info:    sql server delete command OK

### ���ڹ��� SQL ���ݿ������

ʹ����Щ������������� SQL ���ݿ⡣

**sql db create [options] &lt;serverName> &lt;databaseName> &lt;administratorPassword>**

�����µ����ݿ�ʵ��

	~$ azure sql db create fr8aelne00 newdb test
	info:    Executing command sql db create
	Administrator password: ********
	+ Creating SQL Server Database
	info:    sql db create command OK

**sql db show [options] &lt;serverName> &lt;databaseName> &lt;administratorPassword>**

��ʾ���ݿ���ϸ��Ϣ��

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

�г����ݿ⡣

	~$ azure sql db list fr8aelne00 test
	info:    Executing command sql db list
	Administrator password: ********
	+ Getting SQL server databases
	data:    Name    Edition  Collation                     MaxSizeInGB
	data:    ------  -------  ----------------------------  -----------
	data:    master  Web      SQL_Latin1_General_CP1_CI_AS  5
	info:    sql db list command OK

**sql db delete [options] &lt;serverName> &lt;databaseName> &lt;administratorPassword>**

ɾ�����ݿ⡣

	~$ azure sql db delete fr8aelne00 newdb test
	info:    Executing command sql db delete
	Administrator password: ********
	Delete database newdb? [y/n] y
	+ Getting SQL server databases
	+ Removing database
	info:    sql db delete command OK

### ���ڹ��� SQL Server ����ǽ���������

ʹ����Щ���������� SQL Server ����ǽ����

**sql firewallrule create [options] &lt;serverName> &lt;ruleName> &lt;startIPAddress> &lt;endIPAddress>**

Ϊ SQL Server �����µķ���ǽ����

	~$ azure sql firewallrule create fr8aelne00 allowed 131.107.0.0 131.107.255.255
	info:    Executing command sql firewallrule create
	+ Creating Firewall Rule
	info:    sql firewallrule create command OK

**sql firewallrule show [options] &lt;serverName> &lt;ruleName>**

��ʾ����ǽ������ϸ��Ϣ��

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

�г�����ǽ����

	~$ azure sql firewallrule list fr8aelne00
	info:    Executing command sql firewallrule list
	\data:    Name     Start IP address  End IP address
	data:    -------  ----------------  ---------------
	data:    allowed  131.107.0.0       131.107.255.255
	+
	info:    sql firewallrule list command OK

**sql firewallrule delete [options] &lt;serverName> &lt;ruleName>**

�����ɾ������ǽ����

	~$ azure sql firewallrule delete fr8aelne00 allowed
	info:    Executing command sql firewallrule delete
	Delete rule allowed? [y/n] y
	+ Removing firewall rule
	info:    sql firewallrule delete command OK

## <a name ="Commands_to_manage_vnet"></a>���ڹ����������������

ʹ����Щ���������������������

**network vnet create [options] &lt;location>**

�����µ��������硣

	~$ azure network vnet create vnet1 --location "China North" -v
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

��ʾ�����������ϸ��Ϣ��

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

�г��������е��������硣

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

ɾ��ָ�����������硣

	~$ azure network vnet delete opentechvn1
	info:    Executing command network vnet delete
	+ Fetching Network Configuration
	Delete the virtual network opentechvn1 ?  (y/n) y
	+ Deleting the virtual network opentechvn1
	info:    network vnet delete command OK

**network export [file-path]**

��Ҫ���и߼��������ã����ڱ��ص�������������á�ע�⣬�������������ð��� DNS ���������á������������á���������վ�����ú��������á�

**network import [file-path]**

���뱾���������á�

**network dnsserver register [options] &lt;dnsIP>**

ע����ƻ������������������������ƽ����� DNS ��������

	~$ azure network dnsserver register 98.138.253.109 --dns-id FrontEndDnsServer
	info:    Executing command network dnsserver register
	+ Fetching Network Configuration
	+ Updating Network Configuration
	info:    network dnsserver register command OK

**network dnsserver list**

�г����������������ע������� DNS ��������

	~$ azure network dnsserver list
	info:    Executing command network dnsserver list
	+ Fetching Network Configuration
	data:    DNS Server ID         DNS Server IP
	data:    --------------------  --------------
	data:    DNS-bb39b4ac34d66a86  44.55.22.11
	data:    FrontEndDnsServer     98.138.253.109
	info:    network dnsserver list command OK

**network dnsserver unregister [options] &lt;dnsIP>**

������������ɾ�� DNS ��������Ŀ��

	~$ azure network dnsserver unregister 77.88.99.11
	info:    Executing command network dnsserver unregister
	+ Fetching Network Configuration
	Delete the DNS server entry dns-4 ( 77.88.99.11 ) %s ? (y/n) y
	+ Deleting the DNS server entry dns-4 ( 77.88.99.11 )
	info:    network dnsserver unregister command OK

<!--HONumber=50-->