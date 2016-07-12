<properties 
	pageTitle="使用 PowerShell 创建具有本机模式报表服务器的虚拟机 |Azure"
	description="本主题说明并指导你完成 SQL Server Reporting Services 本机模式报表服务器在 Azure 虚拟机中的部署和配置。"
	services="virtual-machines-windows"
	documentationCenter="na"
	authors="rothja"
	manager="jeffreyg"
	editor="monicar" 
	tags="azure-service-management"/>
<tags 
	ms.service="virtual-machines-windows"
	ms.date="05/13/2016"
	wacn.date="06/29/2016" />

# 使用 PowerShell 创建运行本机模式报表服务器的 Azure VM

> [AZURE.IMPORTANT]Azure 具有用于创建和处理资源的两个不同的部署模型：[资源管理器和经典](/documentation/articles/resource-manager-deployment-model/)。本文介绍使用经典部署模型。Azure 建议大多数新部署使用资源管理器模型。

本主题说明并指导你完成 SQL Server Reporting Services 本机模式报表服务器在 Azure 虚拟机中的部署和配置。本文档中的步骤使用一系列手动步骤来创建虚拟机以及用于在 VM 上配置 Reporting Services 的 Windows PowerShell 脚本。配置脚本包括为 HTTP 或 HTTPs 打开防火墙端口。

>[AZURE.NOTE]如果在报表服务器上不需要 **HTTPS**，请**跳过步骤 2**。
><p>在步骤 1 中创建 VM 后，转到使用脚本部分配置报表服务器和 HTTP。在你运行该脚本后，报表服务器便可以使用。

## 前提条件和假设

- “Azure 订阅”：验证你的 Azure 订阅中可用的核心数。如果你创建的建议 VM 大小为 **A3**，你需要 **4** 个可用核心。如果你使用的 VM 大小为 **A2**，你需要 **2** 个可用核心。
	
	- 若要验证你的订阅的核心限制，在 Azure 经典管理门户中，单击右侧窗格中的“设置”，然后单击顶部菜单中的“使用情况”。
	
	- 若要增加核心配额，请联系 [Azure 支持](/support/contact/)。有关 VM 大小信息，请参阅 [Azure 的虚拟机大小](/documentation/articles/virtual-machines-windows-sizes/)。

- **Windows PowerShell 脚本编制**：本主题假定你有 Windows PowerShell 的基础工作知识。有关使用 Windows PowerShell 的详细信息，请参阅以下部分：

	- [在 Windows Server 上启动 Windows PowerShell](https://technet.microsoft.com/zh-cn/library/hh847814.aspx)
	
	- [Windows PowerShell 入门](https://technet.microsoft.com/zh-cn/library/hh857337.aspx)

## 步骤 1：设置 Azure 虚拟机

1. 浏览到 [Azure 经典管理门户](https://manage.windowsazure.cn)。

1. 单击左侧窗格中的“虚拟机”。

	![Azure 虚拟机](./media/virtual-machines-windows-classic-ps-sql-report/IC660124.gif)

1. 单击“新建”。

	![新建按钮](./media/virtual-machines-windows-classic-ps-sql-report/IC692019.gif)

1. 单击“从库中”。

	![从库创建 VM](./media/virtual-machines-windows-classic-ps-sql-report/IC692020.gif)

1. 单击 **SQL Server 2014 RTM Standard – Windows Server 2012 R2**，然后单击箭头以继续。

	![下一步](./media/virtual-machines-windows-classic-ps-sql-report/IC692021.gif)

	如果你需要 Reporting Services 数据驱动订阅功能，则选择 **SQL Server 2014 RTM Enterprise – Windows Server 2012 R2**。有关 SQL Server 版本和功能支持的详细信息，请参阅 [SQL Server 2012 各版本支持的功能](https://msdn.microsoft.com/zh-cn/library/cc645993.aspx#Reporting)。

1. 在“虚拟机配置”页上，编辑以下字段：
									
	- 如果“版本发布日期”超过一个，请选择最新版本。
	
	- **虚拟机名称**：虚拟机名称在下一个配置页上还用作默认云服务 DNS 名称。Azure 服务中的 DNS 名称必须唯一。请考虑为 VM 配置一个描述虚拟机用途的计算机名称。例如 ssrsnativecloud。
	
	- **层**：标准
	
	- **大小：A3** 是 SQL Server 工作负荷的建议 VM 大小。如果 VM 仅用作报表服务器，A2 的 VM 大小就足够了，除非报表服务器遇到大量工作负荷。有关 VM 定价信息，请参阅[虚拟机定价](/home/features/virtual-machines/pricing/)。
	
	- **新用户名**：将你提供的名称创建为 VM 上的管理员。
	
	- **新密码**和**确认**。此密码用于新的管理员帐户并建议使用强密码。
	
	- 单击“下一步”。![下一步](./media/virtual-machines-windows-classic-ps-sql-report/IC692021.gif)

1. 在下一页上，编辑以下字段：

	- **云服务**：选择“创建新的云服务”。
	
	- **云服务 DNS 名称**：这是与 VM 关联的云服务的公共 DNS 名称。默认名称是为 VM 名称键入的名称。如果在该主题的后续步骤中，你创建受信任的 SSL 证书，则该 DNS 名称将用于该证书的“**Issued to**”的值。
	
	- **区域/地缘组/虚拟网络**：选择离你的最终用户最近的区域。
	
	- **存储帐户**：使用自动生成的存储帐户。
	
	- **可用性集**：无。
	
	- **终结点**：保留“远程桌面”和 **PowerShell** 终结点，然后添加一个 HTTP 或 HTTPS 终结点，具体取决于你的环境。

		- **HTTP**：默认公共和专用端口均为 **80**。请注意，如果你使用非 80 的专用端口，请修改 http 脚本中的 **$HTTPport = 80**。

		- **HTTPS**：默认公共和专用端口均为 **443**。最佳安全方案是更改私有端口并配置你的防火墙和报表服务器以使用私有端口。有关终结点的详细信息，请参阅[如何设置与虚拟机的通信](/documentation/articles/virtual-machines-windows-classic-setup-endpoints/)。请注意，如果你使用非 443 的端口，请更改 HTTPS 脚本中的参数 **$HTTPsport = 443**。
	
	- 单击“下一步”。![下一步](./media/virtual-machines-windows-classic-ps-sql-report/IC692021.gif)

1. 在向导的最后一页上，保持选中默认的“安装 VM 代理”。本主题中的步骤不使用 VM 代理，但如果你计划保留此 VM，VM 代理和扩展将允许你增强该 VM。有关 VM 代理的详细信息，请参阅 [VM 代理和扩展 – 第 1 部分](http://azure.microsoft.com/blog/2014/04/11/vm-agent-and-extensions-part-1/)。安装并运行的一个默认扩展是“BGINFO”扩展，它在 VM 桌面上显示系统信息，如内部 IP 和驱动器可用空间。

1. 单击“完成”。![确定](./media/virtual-machines-windows-classic-ps-sql-report/IC660122.gif)

1. 在预配过程中，VM 的“状态”显示为“正在启动(正在预配)”，然后在 VM 预配完成并可供使用时显示为“正在运行”。

## 步骤 2：创建服务器证书

>[AZURE.NOTE]如果在报表服务器上不需要 HTTPS，则可以**跳过步骤 2** 并转到**使用脚本来配置报表服务器和 HTTP** 部分。使用 HTTP 脚本快速配置报表服务器，报表服务器便可以使用了。

为了在 VM 上使用 HTTPS，你需要受信任的 SSL 证书。具体取决于你的方案，你可以使用以下两种方法之一：

- 由证书颁发机构 (CA) 颁发并受 Microsoft 信任的有效 SSL 证书。CA 根证书都需要通过 Microsoft 根证书计划分发。

- 自签名证书。不建议将自签名证书用于生产环境。

### 使用由受信任证书颁发机构 (CA) 创建的证书

1. **从证书颁发机构请求 Web 应用的服务器证书**。 

	你可以使用 Web 服务器证书向导来生成一个发送到证书颁发机构的证书请求文件 (Certreq.txt) 或生成一个针对联机证书颁发机构的请求。例如，Windows Server 2012 中的 Microsoft 证书服务。根据你的服务器证书提供的标识保证级别，证书颁发机构需要几天到几个月来批准你的请求并向你发送证书文件。

	有关请求服务器证书的详细信息，请参阅以下部分：
	
	- 使用 [Certreq](https://technet.microsoft.com/zh-cn/library/cc725793.aspx)。
	
	- 用于管理 Windows Server 2012 的安全工具。

	[用于管理 Windows Server 2012 的安全工具](https://technet.microsoft.com/zh-cn/library/jj730960.aspx)

	>[AZURE.NOTE]受信任的 SSL 证书的 **issued to** 字段应与你用于新 VM 的**云服务 DNS 名称**相同。

1. **在 Web 服务器上安装服务器证书**。Web 服务器在这种情况下是托管报表服务器的 VM， Web 应用在配置 Reporting Services 时的后续步骤中创建。有关通过使用证书 MMC 管理单元在 Web 服务器上安装服务器证书的详细信息，请参阅[安装服务器证书](https://technet.microsoft.com/zh-cn/library/cc740068)。
	
	如果要使用本主题中包含的脚本来配置报表服务器，则需要证书 **Thumbprint** 的值作为该脚本的参数。有关如何获取证书指纹的详细信息，请参阅以下部分。

1. 将服务器证书分配给报表服务器。将在下一部分中配置报表服务器时完成分配。

###<a name="to-use-the-virtual-machines-self-signed-certificate"></a> 使用虚拟机自签名证书

当设置 VM 时已在 VM 上创建了自签名证书。证书具有与 VM DNS 名称相同的名称。为避免出现证书错误，它必须在 VM 本身上受信任，并且受该站点的所有用户信任。

1. 若要信任本地 VM 上的证书的根 CA，请将该证书添加到**受信任的根证书颁发机构**。以下是所需步骤的摘要。有关如何信任 CA 的详细步骤，请参阅[安装服务器证书](https://technet.microsoft.com/zh-cn/library/cc740068)。

	1. 从 Azure 经典管理门户中，选择 VM 并单击“连接”。根据你的浏览器配置，可能会向你提示保存一个用于连接到 VM 的 .rdp 文件。
	
		![连接到 Azure 虚拟机](./media/virtual-machines-windows-classic-ps-sql-report/IC650112.gif) 使用你在创建 VM 时配置的用户 VM 名称、用户名和密码。
	
		例如，在下图中，VM 名称是 **ssrsnativecloud**，用户名是 **testuser**。
		
		![登录名包含 VM 名称](./media/virtual-machines-windows-classic-ps-sql-report/IC764111.png)
	
	1. 运行 mmc.exe。有关详细信息，请参阅[如何：使用 MMC 管理单元查看证书](https://msdn.microsoft.com/zh-cn/library/ms788967.aspx)。
	
	1. 在控制台应用程序“文件”菜单中，添加“证书”管理单元，在系统提示时选择“计算机帐户”，然后单击“下一步”。
	
	1. 选择要管理的“本地计算机”，然后单击“完成”。
	
	1. 单击“确定”，然后展开“证书 – 个人”节点，最后单击“证书”。证书以 VM 的 DNS 名称命名，并以 **chinacloudapp.cn** 结尾。右键单击证书名称，然后单击“复制”。
	
	1. 展开“受信任的根证书颁发机构”节点，然后右键单击“证书”，最后单击“粘贴”。
	
	1. 若要验证，双击“受信任的根证书颁发机构”下的证书名称，验证其中不存在错误并且你能看到你的证书。如果要使用本主题中包含的 HTTPS 脚本来配置报表服务器，则需要证书 **Thumbprint** 的值作为该脚本的参数。**要获取该指纹值**，请完成下列操作。[使用脚本来配置报表服务器和 HTTPS](#use-script-to-configure-the-report-server-and-HTTPS) 部分中还有一个 PowerShell 示例用于检索该指纹。
		
		1. 双击该证书名称，例如 ssrsnativecloud.chinacloudapp.cn。
		
		1. 单击“详细信息”选项卡。
		
		1. 单击“指纹”。指纹的值显示在详细信息字段中，例如，‎a6 08 3c df f9 0b f7 e3 7c 25 ed a4 ed 7e ac 91 9c 2c fb 2f。
		
		1. 复制指纹并保存该值供以后或立即编辑脚本时使用。
		
		1. (*)在运行该脚本之前，删除值对之间的空格。例如，之前提到过的指纹现在将为 a6083cdff90bf7e37c25eda4ed7eac919c2cfb2f。
		
		1. 将服务器证书分配给报表服务器。将在下一部分中配置报表服务器时完成分配。

如果使用自签名的 SSL 证书，证书上的名称已经与 VM 的主机名匹配。因此，计算机的 DNS 已全局注册并且可以从任何客户端访问。

## 步骤 3： 配置报表服务器

本部分将指导你完成将 VM 配置为 Reporting Services 本机模式报表服务器。可以使用以下方法之一来配置报表服务器：

- 使用脚本来配置报表服务器

- 使用配置管理器配置报表服务器。

有关更多详细步骤，请参阅[连接到虚拟机并启动 Reporting Services 配置管理器](/documentation/articles/virtual-machines-windows-classic-ps-sql-bi/#connect-to-the-virtual-machine-and-start-the-reporting-services-configuration-manager)。

**身份验证说明：**Windows 身份验证是建议的身份验证方法，并且它是默认的 Reporting Services 身份验证方法。只有在 VM 配置的用户可以访问 Reporting Services 并分配给 Reporting Services 角色。

### 使用脚本来配置报表服务器和 HTTP

若要使用 Windows PowerShell 脚本配置报表服务器，请完成以下步骤。该配置包括 HTTP 而不是 HTTPS：

1. 从 Azure 经典管理门户中，选择 VM 并单击“连接”。根据你的浏览器配置，可能会向你提示保存一个用于连接到 VM 的 .rdp 文件。

	![连接到 Azure 虚拟机](./media/virtual-machines-windows-classic-ps-sql-report/IC650112.gif) 使用你在创建 VM 时配置的用户 VM 名称、用户名和密码。

	例如，在下图中，VM 名称是 **ssrsnativecloud**，用户名是 **testuser**。
	
	![登录名包含 VM 名称](./media/virtual-machines-windows-classic-ps-sql-report/IC764111.png)

1. 在 VM 上，使用管理权限打开 **Windows PowerShell ISE**。默认情况下，将 PowerShell ISE 安装在 Windows server 2012 上。建议使用 ISE 而不是标准 Windows PowerShell 窗口，以便你可以将脚本粘贴到 ISE，修改脚本，然后运行该脚本。

1. 在 Windows PowerShell ISE 中，单击“视图”菜单，然后单击“显示脚本窗格”。

1. 复制以下脚本，并将该脚本粘贴到 Windows PowerShell ISE 脚本窗格。

		## This script configures a Native mode report server without HTTPS
		$ErrorActionPreference = "Stop"
		
		$server = $env:COMPUTERNAME
		$HTTPport = 80 # change the value if you used a different port for the private HTTP endpoint when the VM was created.
		
		## Set PowerShell execution policy to be able to run scripts
		Set-ExecutionPolicy RemoteSigned -Force
		
		## Utility method for verifying an operation's result
		function CheckResult
		{
		    param($wmi_result, $actionname)
		    if ($wmi_result.HRESULT -ne 0) {
		        write-error "$actionname failed. Error from WMI: $($wmi_result.Error)"
		    }
		}
		
		$starttime=Get-Date
		write-host -foregroundcolor DarkGray $starttime StartTime
		
		## ReportServer Database name - this can be changed if needed
		$dbName='ReportServer'
		
		## Register for MSReportServer_ConfigurationSetting
		## Change the version portion of the path to "v11" to use the script for SQL Server 2012
		$RSObject = Get-WmiObject -class "MSReportServer_ConfigurationSetting" -namespace "root\Microsoft\SqlServer\ReportServer\RS_MSSQLSERVER\v12\Admin"
		
		## Report Server Configuration Steps
		
		## Setting the web service URL ##
		write-host -foregroundcolor green "Setting the web service URL"
		write-host -foregroundcolor green ">>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>"
		$time=Get-Date
		write-host -foregroundcolor DarkGray $time
		
		## SetVirtualDirectory for ReportServer site
		    write-host 'Calling SetVirtualDirectory'
		    $r = $RSObject.SetVirtualDirectory('ReportServerWebService','ReportServer',1033)
		    CheckResult $r "SetVirtualDirectory for ReportServer"
		
		## ReserveURL for ReportServerWebService - port $HTTPport (for local usage)
		    write-host "Calling ReserveURL port $HTTPport"
		    $r = $RSObject.ReserveURL('ReportServerWebService',"http://+:$HTTPport",1033)
		    CheckResult $r "ReserveURL for ReportServer port $HTTPport" 
		   
		## Setting the Database ##
		write-host -foregroundcolor green "Setting the Database"
		write-host -foregroundcolor green ">>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>"
		$time=Get-Date
		write-host -foregroundcolor DarkGray $time
		
		## GenerateDatabaseScript - for creating the database
		    write-host "Calling GenerateDatabaseCreationScript for database $dbName"
		    $r = $RSObject.GenerateDatabaseCreationScript($dbName,1033,$false)
		    CheckResult $r "GenerateDatabaseCreationScript"
		    $script = $r.Script
		
		## Execute sql script to create the database
		    write-host 'Executing Database Creation Script'
		    $savedcvd = Get-Location
		    Import-Module SQLPS              ## this automatically changes to sqlserver provider
		    Invoke-SqlCmd -Query $script
		    Set-Location $savedcvd
		  
		## GenerateGrantRightsScript 
		    $DBUser = "NT Service\ReportServer"
		    write-host "Calling GenerateDatabaseRightsScript with user $DBUser"
		    $r = $RSObject.GenerateDatabaseRightsScript($DBUser,$dbName,$false,$true)
		    CheckResult $r "GenerateDatabaseRightsScript"
		    $script = $r.Script
		
		## Execute grant rights script
		    write-host 'Executing Database Rights Script'
		    $savedcvd = Get-Location
		    cd sqlserver:\
		    Invoke-SqlCmd -Query $script
		    Set-Location $savedcvd
		
		## SetDBConnection - uses Windows Service (type 2), username is ignored
		    write-host "Calling SetDatabaseConnection server $server, DB $dbName"
		    $r = $RSObject.SetDatabaseConnection($server,$dbName,2,'','')
		    CheckResult $r "SetDatabaseConnection"  
		
		## Setting the Report Manager URL ##
		
		write-host -foregroundcolor green "Setting the Report Manager URL"
		write-host -foregroundcolor green ">>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>"
		$time=Get-Date
		write-host -foregroundcolor DarkGray $time
		
		## SetVirtualDirectory for Reports (Report Manager) site
		    write-host 'Calling SetVirtualDirectory'
		    $r = $RSObject.SetVirtualDirectory('ReportManager','Reports',1033)
		    CheckResult $r "SetVirtualDirectory"
		
		## ReserveURL for ReportManager  - port $HTTPport
		    write-host "Calling ReserveURL for ReportManager, port $HTTPport"
		    $r = $RSObject.ReserveURL('ReportManager',"http://+:$HTTPport",1033)
		    CheckResult $r "ReserveURL for ReportManager port $HTTPport"
		
		write-host -foregroundcolor green "Open Firewall port for $HTTPport"
		write-host -foregroundcolor green ">>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>"
		$time=Get-Date
		write-host -foregroundcolor DarkGray $time
		
		## Open Firewall port for $HTTPport
		    New-NetFirewallRule -DisplayName “Report Server (TCP on port $HTTPport)” -Direction Inbound –Protocol TCP –LocalPort $HTTPport
		    write-host "Added rule Report Server (TCP on port $HTTPport) in Windows Firewall"
		
		write-host 'Operations completed, Report Server is ready'
		write-host -foregroundcolor DarkGray $starttime StartTime
		$time=Get-Date
		write-host -foregroundcolor DarkGray $time

1. 如果你使用非 80 的 HTTP 端口创建 VM，修改参数 $HTTPport = 80。

1. 该脚本当前是为 Reporting Services 配置的。如果你想要为 Reporting Services 运行该脚本，在 Get-WmiObject 语句上将指向命名空间的路径的版本部分修改为“v11”。

1. 运行该脚本。

**验证**：若要验证基本报表服务器功能是否正常工作，请参阅本主题中稍后的[验证配置](#verify-the-configuration)部分。

###<a name="use-script-to-configure-the-report-server-and-HTTPS"></a> 使用脚本来配置报表服务器和 HTTPS

若要使用 Windows PowerShell 来配置报表服务器，请完成以下步骤。该配置包括 HTTPS 而不是 HTTP。

1. 从 Azure 经典管理门户中，选择 VM 并单击“连接”。根据你的浏览器配置，可能会向你提示保存一个用于连接到 VM 的 .rdp 文件。

	![连接到 Azure 虚拟机](./media/virtual-machines-windows-classic-ps-sql-report/IC650112.gif) 使用你在创建 VM 时配置的用户 VM 名称、用户名和密码。

	例如，在下图中，VM 名称是 **ssrsnativecloud**，用户名是 **testuser**。

	![登录名包含 VM 名称](./media/virtual-machines-windows-classic-ps-sql-report/IC764111.png)

1. 在 VM 上，使用管理权限打开 **Windows PowerShell ISE**。默认情况下，将 PowerShell ISE 安装在 Windows server 2012 上。建议使用 ISE 而不是标准 Windows PowerShell 窗口，以便你可以将脚本粘贴到 ISE，修改脚本，然后运行该脚本。

1. 若要允许运行脚本，运行以下 Windows PowerShell 命令：

		Set-ExecutionPolicy RemoteSigned

	然后，可以执行以下操作以验证策略：

		Get-ExecutionPolicy

1. 在 Windows PowerShell ISE 中，单击“视图”菜单，然后单击“显示脚本窗格”。

1. 复制以下脚本并将其粘贴到 Windows PowerShell ISE 脚本窗格。

		## This script configures the report server, including HTTPS
		$ErrorActionPreference = "Stop"
		$httpsport=443 # modify if you used a different port number when the HTTPS endpoint was created.
		
		# You can run the following command to get (.chinacloudapp.cn certificates) so you can copy the thumbprint / certificate hash
		#dir cert:\LocalMachine -rec | Select-Object * | where {$_.issuer -like "*cloudapp*" -and $_.pspath -like "*root*"} | select dnsnamelist, thumbprint, issuer
		#
		# The certifacte hash is a REQUIRED parameter
		$certificatehash="" 
		# the certificate hash should not contain spaces
		
		if ($certificatehash.Length -lt 1) 
		{
		    write-error "certificatehash is a required parameter"
		} 
		# Certificates should be all lower case
		$certificatehash=$certificatehash.ToLower()
		$server = $env:COMPUTERNAME
		# If the certificate is not a wildcard certificate, comment out the following line, and enable the full $DNSNAme reference.
		$DNSName="+"
		#$DNSName="$server.chinacloudapp.cn"
		$DNSNameAndPort = $DNSName + ":$httpsport"
		
		## Utility method for verifying an operation's result
		function CheckResult
		{
		    param($wmi_result, $actionname)
		    if ($wmi_result.HRESULT -ne 0) {
		        write-error "$actionname failed. Error from WMI: $($wmi_result.Error)"
		    }
		}
		
		$starttime=Get-Date
		write-host -foregroundcolor DarkGray $starttime StartTime
		
		## ReportServer Database name - this can be changed if needed
		$dbName='ReportServer'
		
		write-host "The script will use $DNSNameAndPort as the DNS name and port" 
		
		## Register for MSReportServer_ConfigurationSetting
		## Change the version portion of the path to "v11" to use the script for SQL Server 2012
		$RSObject = Get-WmiObject -class "MSReportServer_ConfigurationSetting" -namespace "root\Microsoft\SqlServer\ReportServer\RS_MSSQLSERVER\v12\Admin"
		
		## Reporting Services Report Server Configuration Steps
		
		## 1. Setting the web service URL ##
		write-host -foregroundcolor green "Setting the web service URL"
		write-host -foregroundcolor green ">>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>"
		$time=Get-Date
		write-host -foregroundcolor DarkGray $time
		
		## SetVirtualDirectory for ReportServer site
		    write-host 'Calling SetVirtualDirectory'
		    $r = $RSObject.SetVirtualDirectory('ReportServerWebService','ReportServer',1033)
		    CheckResult $r "SetVirtualDirectory for ReportServer"
		
		## ReserveURL for ReportServerWebService - port 80 (for local usage)
		    write-host 'Calling ReserveURL port 80'
		    $r = $RSObject.ReserveURL('ReportServerWebService','http://+:80',1033)
		    CheckResult $r "ReserveURL for ReportServer port 80" 
		
		## ReserveURL for ReportServerWebService - port $httpsport
		    write-host "Calling ReserveURL port $httpsport, for URL: https://$DNSNameAndPort"
		    $r = $RSObject.ReserveURL('ReportServerWebService',"https://$DNSNameAndPort",1033)
		    CheckResult $r "ReserveURL for ReportServer port $httpsport" 
		
		## CreateSSLCertificateBinding for ReportServerWebService port $httpsport
		    write-host "Calling CreateSSLCertificateBinding port $httpsport, with certificate hash: $certificatehash"
		    $r = $RSObject.CreateSSLCertificateBinding('ReportServerWebService',$certificatehash,'0.0.0.0',$httpsport,1033)
		    CheckResult $r "CreateSSLCertificateBinding for ReportServer port $httpsport" 
		    
		## 2. Setting the Database ##
		write-host -foregroundcolor green "Setting the Database"
		write-host -foregroundcolor green ">>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>"
		$time=Get-Date
		write-host -foregroundcolor DarkGray $time
		
		## GenerateDatabaseScript - for creating the database
		    write-host "Calling GenerateDatabaseCreationScript for database $dbName"
		    $r = $RSObject.GenerateDatabaseCreationScript($dbName,1033,$false)
		    CheckResult $r "GenerateDatabaseCreationScript"
		    $script = $r.Script
		
		## Execute sql script to create the database
		    write-host 'Executing Database Creation Script'
		    $savedcvd = Get-Location
		    Import-Module SQLPS                    ## this automatically changes to sqlserver provider
		    Invoke-SqlCmd -Query $script
		    Set-Location $savedcvd
		  
		## GenerateGrantRightsScript 
		    $DBUser = "NT Service\ReportServer"
		    write-host "Calling GenerateDatabaseRightsScript with user $DBUser"
		    $r = $RSObject.GenerateDatabaseRightsScript($DBUser,$dbName,$false,$true)
		    CheckResult $r "GenerateDatabaseRightsScript"
		    $script = $r.Script
		
		## Execute grant rights script
		    write-host 'Executing Database Rights Script'
		    $savedcvd = Get-Location
		    cd sqlserver:\
		    Invoke-SqlCmd -Query $script
		    Set-Location $savedcvd
		
		## SetDBConnection - uses Windows Service (type 2), username is ignored
		    write-host "Calling SetDatabaseConnection server $server, DB $dbName"
		    $r = $RSObject.SetDatabaseConnection($server,$dbName,2,'','')
		    CheckResult $r "SetDatabaseConnection"  
		
		## 3. Setting the Report Manager URL ##
		
		write-host -foregroundcolor green "Setting the Report Manager URL"
		write-host -foregroundcolor green ">>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>"
		$time=Get-Date
		write-host -foregroundcolor DarkGray $time
		
		## SetVirtualDirectory for Reports (Report Manager) site
		    write-host 'Calling SetVirtualDirectory'
		    $r = $RSObject.SetVirtualDirectory('ReportManager','Reports',1033)
		    CheckResult $r "SetVirtualDirectory"
		
		## ReserveURL for ReportManager  - port 80
		    write-host 'Calling ReserveURL for ReportManager, port 80'
		    $r = $RSObject.ReserveURL('ReportManager','http://+:80',1033)
		    CheckResult $r "ReserveURL for ReportManager port 80"
		
		## ReserveURL for ReportManager - port $httpsport
		    write-host "Calling ReserveURL port $httpsport, for URL: https://$DNSNameAndPort"
		    $r = $RSObject.ReserveURL('ReportManager',"https://$DNSNameAndPort",1033)
		    CheckResult $r "ReserveURL for ReportManager port $httpsport" 
		
		## CreateSSLCertificateBinding for ReportManager port $httpsport
		    write-host "Calling CreateSSLCertificateBinding port $httpsport with certificate hash: $certificatehash"
		    $r = $RSObject.CreateSSLCertificateBinding('ReportManager',$certificatehash,'0.0.0.0',$httpsport,1033)
		    CheckResult $r "CreateSSLCertificateBinding for ReportManager port $httpsport" 
		
		write-host -foregroundcolor green "Open Firewall port for $httpsport"
		write-host -foregroundcolor green ">>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>"
		$time=Get-Date
		write-host -foregroundcolor DarkGray $time
		
		## Open Firewall port for $httpsport
		    New-NetFirewallRule -DisplayName “Report Server (TCP on port $httpsport)” -Direction Inbound –Protocol TCP –LocalPort $httpsport
		    write-host "Added rule Report Server (TCP on port $httpsport) in Windows Firewall"
		
		write-host 'Operations completed, Report Server is ready'
		write-host -foregroundcolor DarkGray $starttime StartTime
		$time=Get-Date
		write-host -foregroundcolor DarkGray $time

1. 修改脚本中的 **$certificatehash** 参数：

	- 这是**必需**的参数。如果未从前面的步骤中保存证书值，使用以下一个方法来从证书指纹复制证书哈希值：

		在 VM 上，打开 Windows PowerShell ISE 并运行以下命令：

			dir cert:\LocalMachine -rec | Select-Object * | where {$_.issuer -like "*cloudapp*" -and $_.pspath -like "*root*"} | select dnsnamelist, thumbprint, issuer

		输出与以下内容类似：例如，如果该脚本返回一个空白行，则尚未为 VM 配置证书，请参阅[使用虚拟机自签名证书](#to-use-the-virtual-machines-self-signed-certificate)部分。
	
	或者
	
	- 在 VM 上运行 mmc.exe，然后添加“证书”管理单元。
	
	- 在“受信任的根证书颁发机构”节点下，双击你的证书名称。如果你使用的是 VM 的自签名证书，则证书将以该 VM 的 DNS 名称命名，并以 **chinacloudapp.cn** 结尾。
	
	- 单击“详细信息”选项卡。
	
	- 单击“指纹”。指纹的值显示在详细信息字段中，例如，af 11 60 b6 4b 28 8 d 89 0a 82 12 ff 6b a9 c3 66 4f 31 90 48
	
	- **在运行该脚本之前**，删除值对之间的空格。例如，af1160b64b288d890a8212ff6ba9c3664f319048

1. 修改 **$httpsport** 参数：

	- 如果你将端口 443 用于 HTTPS 终结点，那么你不必在脚本中更新此参数。否则，使用你在 VM 上配置 HTTPS 私有终结点时选择的端口值。

1. 修改 **$DNSName** 参数：

	- 该脚本为通配符证书 $DNSName ="+" 而配置。如果你不想要为通配符证书绑定进行配置，请注释掉 $DNSName ="+" 并启用以下行：完整 $DNSNAme 引用，# # $DNSName="$server.chinacloudapp.cn"。

		如果不希望使用用于 Reporting Services 的虚拟机 DNS 名称，请更改 $DNSName 值。如果使用该参数，该证书必须也使用此名称，并且你须在 DNS 服务器上全局注册该名称。

1. 该脚本当前是为 Reporting Services 配置的。如果你想要为 Reporting Services 运行该脚本，在 Get-WmiObject 语句上将指向命名空间的路径的版本部分修改为“v11”。

1. 运行该脚本。

**验证**：若要验证基本报表服务器功能是否正常工作，请参阅本主题中稍后的[验证配置](#verify-the-connection)部分。若要验证证书绑定，请使用具有管理权限的身份打开命令提示符，然后运行以下命令：

	netsh http show sslcert

结果将包括以下内容：

	IP:port                      : 0.0.0.0:443

	Certificate Hash             : f98adf786994c1e4a153f53fe20f94210267d0e7

### 使用配置管理器配置报表服务器

如果你不想要运行 PowerShell 脚本来配置报表服务器，请按照本部分中的步骤使用 Reporting Services 本机模式配置管理器来配置报表服务器。

1. 从 Azure 经典管理门户中，选择 VM 并单击“连接”。使用在创建 VM 时配置的用户名和密码。

	![连接到 Azure 虚拟机](./media/virtual-machines-windows-classic-ps-sql-report/IC650112.gif)

1. 运行 Windows 更新并将更新安装到 VM。如果需要重新启动 VM，请重新启动 VM 并从 Azure 经典管理门户重新连接到该 VM。

1. 从 VM 上的“开始”菜单，键入 **Reporting Services** 并打开“Reporting Services 配置管理器”。

1. 保留“服务器名称”和“报表服务器实例”的默认值。单击“连接”。

1. 在左窗格中，单击“Web 服务 URL”。

1. 默认情况下，为 HTTP 端口 80 配置 RS 且 IP 为“全部分配”。若要添加 HTTPS：

	1. 在“SSL 证书”中：选择要使用的证书，例如，[VM 名称].chinacloudapp.cn。如果未列出证书，请参阅**第 2 步：创建服务器证书**部分，以了解如何在 VM 上安装和信任证书的信息。
	
	1. 在“SSL 端口”下：选择 443。如果使用不同的专用端口配置了 VM 中的 HTTPS 私有终结点，此处使用该值。
	
	1. 单击“应用”并等待操作完成。

1. 在左窗格中，单击“数据库”。

	1. 单击“更改数据库”。
	
	1. 单击“创建新的报表服务器数据库”，然后单击“下一步”。
	
	1. 将默认的“服务器名称”保留为 VM 名称，将默认的“身份验证类型”保留为“当前用户”-“集成安全性”。单击**“下一步”**。
	
	1. 将默认的“数据库名称”保留为 **ReportServer**，然后单击“下一步”。
	
	1. 将默认的“身份验证类型”保留为“服务凭据”，然后单击“下一步”。
	
	1. 在“摘要”页上单击“下一步”。
	
	1. 配置完成后，单击“完成”。

1. 在左窗格中，单击“报表管理器 URL”。将默认的“虚拟目录”保留为“报告”，然后单击“应用”。

1. 单击“退出”以关闭 Reporting Services 配置管理器。

## 步骤 4：打开 Windows 防火墙端口

>[AZURE.NOTE]如果使用了其中一个脚本来配置报表服务器，则可以跳过本节。该脚本包含一个用于打开防火墙端口的步骤。HTTP 默认端口是 80，HTTPS 默认端口是 443。

若要远程连接到虚拟机上的报表管理器或报表服务器，在 VM 上需要 TCP 终结点。在 VM 防火墙中需要打开同一个端口。设置 VM 时创建了该终结点。

本部分提供有关如何打开防火墙端口的基本信息。有关详细信息，请参阅[为报表服务器访问配置防火墙](https://technet.microsoft.com/zh-cn/library/bb934283.aspx)

>[AZURE.NOTE]如果使用了脚本来配置报表服务器，则可以跳过本节。该脚本包含一个用于打开防火墙端口的步骤。

如果你为非 443 的 HTTPS 配置一个私有端口，请相应地修改下面的脚本。若要打开 Windows 防火墙上的端口 **443**，请完成下列操作：

1. 使用管理权限打开 Windows PowerShell 窗口。

1. 如果你在 VM 上配置 HTTPS 终结点时使用了非 443 的端口，则更新下面命令中的端口，然后运行命令：

		New-NetFirewallRule -DisplayName “Report Server (TCP on port 443)” -Direction Inbound –Protocol TCP –LocalPort 443

1. 命令完成后，将在命令提示符中显示 **Ok**。

若要验证端口已打开，请打开 Windows PowerShell 窗口并运行以下命令：

	get-netfirewallrule | where {$_.displayname -like "*report*"} | select displayname,enabled,action

##<a name="verify-the-configuration" id="verify-the-connection"></a> 验证配置

若要验证基本报表服务器功能现在是否正常工作，请使用管理权限打开你的浏览器，然后浏览到以下报表服务器和报表管理器 URL：

- 在 VM 上，浏览到报表服务器 URL：

		http://localhost/reportserver

- 在 VM 上，浏览到报表管理器 URL：

		http://localhost/Reports

- 从本地计算机，浏览到 VM 上的**远程**报表管理器。相应地更新下面示例中的 DNS 名称。当系统提示输入密码时，使用你在设置 VM 时创建的管理员凭据。用户名采用 [域] [用户名] 格式，其中域为 VM 计算机名称，例如，ssrsnativecloud\\testuser。如果你使用的不是 HTTP**S**，请删除 URL 中的 **s**。有关在虚拟机上创建更多用户的信息，请参阅以下部分。

		https://ssrsnativecloud.chinacloudapp.cn/Reports

- 从本地计算机，浏览到远程报表管理器 URL。相应地更新下面示例中的 DNS 名称。如果你使用的不是 HTTPS，请删除 URL 中的 s。

		https://ssrsnativecloud.chinacloudapp.cn/ReportServer

## 创建用户和分配角色

在配置和验证报表服务器之后，一个常见管理任务是创建一个或多个用户并将用户分配到 Reporting Services 角色。有关详细信息，请参阅以下主题：

- [创建本地用户帐户](https://technet.microsoft.com/zh-cn/library/cc770642.aspx)

- [授予用户对报表服务器（报表管理器）的访问权限](https://msdn.microsoft.com/zh-cn/library/ms156034.aspx)

- [创建和管理角色分配](https://msdn.microsoft.com/zh-cn/library/ms155843.aspx)

## 创建报表并将其发布到 Azure 虚拟机

下表汇总一些选项，可用于将现有报表从本地计算机发布到 Azure 虚拟机上托管的报表服务器：

- **RS.exe 脚本**：使用 RS.exe 脚本将报表项从现有报表服务器复制到你的 Azure 虚拟机。有关详细信息，请参阅[用示例 Reporting Services rs.exe 脚本在报表服务器之间迁移内容](https://msdn.microsoft.com/zh-cn/library/dn531017.aspx)中的“本机模式到本机模式 – Azure 虚拟机”部分。

- **报表生成器**：虚拟机包括 Microsoft SQL Server 报表生成器的单击一次版本。若要首次在虚拟机上启动报表生成器：

	1. 使用管理权限启动你的浏览器。
	
	1. 浏览到虚拟机上的报表管理器，然后单击功能区中的“报表生成器”。

	有关详细信息，请参阅[安装、卸载和支持报表生成器](https://msdn.microsoft.com/zh-cn/library/ff519551.aspx)。

- **SQL Server Data Tools：VM**：如果创建的 VM 安装了 SQL Server 2012，则 SQL Server Data Tools 将安装在该虚拟机上并可用于在该虚拟机上创建**报表服务器项目**和报表。SQL Server Data Tools 可以将报表发布到虚拟机上的报表服务器。
	
	如果你创建的 VM 安装了 SQL Server 2014，则可以安装适用于 Visual Studio 的 SQL Server Data Tools- BI。有关详细信息，请参阅以下主题：

	- [适用于 Visual Studio 2013 的 Microsoft SQL Server Data Tools-Business Intelligence](https://www.microsoft.com/download/details.aspx?id=42313)

	- [适用于 Visual Studio 2012 的 Microsoft SQL Server Data Tools-Business Intelligence](https://www.microsoft.com/download/details.aspx?id=36843)

	- [SQL Server Data Tools 和 SQL Server Business Intelligence (SSDT-BI)](http://curah.microsoft.com/30004/sql-server-data-tools-ssdt-and-sql-server-business-intelligence)

- **SQL Server Data Tools：远程**：在本地计算机上，在 SQL Server Data Tools 中创建一个包含 Reporting Services 报表的 Reporting Services 项目。将项目配置为连接到 web 服务 URL。

	![SSRS 项目的 ssdt 项目属性](./media/virtual-machines-windows-classic-ps-sql-report/IC650114.gif)

- **使用脚本**：使用脚本来复制报表服务器内容。有关详细信息，请参阅[使用示例 Reporting Services rs.exe 脚本在报表服务器之间迁移内容](https://msdn.microsoft.com/zh-cn/library/dn531017.aspx)。

## 如果使用的不是 VM，将成本降到最低

>[AZURE.NOTE]为了在不使用 Azure 虚拟机时尽可能减少费用，可从 Azure 经典管理门户关闭 VM。如果使用 VM 内的 Windows 电源选项来关闭 VM，仍将为 VM 支付相同的金额。若要减少费用，你需要在 Azure 经典管理门户中关闭 VM。如果你不再需要该 VM，请记住删除 VM 和关联的 .vhd 文件以避免产生存储费用。有关详细信息，请参阅[虚拟机定价详细信息](/home/features/virtual-machines/pricing/)中的常见问题部分。

## 更多信息

### 资源

- 有关对 SQL Server Business Intelligence 和 SharePoint 2013 进行多服务器部署的相关类似内容，请参阅[在 Azure 虚拟机中部署 SQL Server Business Intelligence](/documentation/articles/virtual-machines-windows-sql-server-iaas-overview/)。

- 有关在 Azure 虚拟机上部署 SQL Server Business Intelligence 的常规信息，请参阅[在 Azure 虚拟机中部署 SQL Server Business Intelligence](/documentation/articles/virtual-machines-windows-classic-ps-sql-bi/)。

- 有关 Azure 计算费用成本的详细信息，请参阅 [Azure 定价计算器](/pricing/calculator/?scenario=virtual-machines)的“虚拟机”选项卡。

### 社区内容

- 有关如何在不使用脚本的情况下创建 Reporting Services 本机模式报表服务器的分步说明，请参阅[在 Azure 虚拟机上托管 SQL 报表服务](http://adititechnologiesblog.blogspot.in/2012/07/hosting-sql-reporting-service-on-azure.html)。

### 指向 Azure VM 中 SQL Server 的其他资源的链接

[Azure 虚拟机中的 SQL Server 概述](/documentation/articles/virtual-machines-windows-sql-server-iaas-overview/)

<!---HONumber=Mooncake_0104_2016-->