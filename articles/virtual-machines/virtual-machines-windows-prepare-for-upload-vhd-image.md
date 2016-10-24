<properties
	pageTitle="准备好要上载到 Azure 的 Windows VHD | Azure"
	description="将 Windows VHD 上载到 Azure 之前为其做好准备的建议做法"
	services="virtual-machines-windows"
	documentationCenter=""
	authors="genlin"
	manager="timlt"
	editor=""
	tags="azure-resource-manager"/>  


<tags
	ms.service="virtual-machines-windows"
	ms.workload="infrastructure-services"
	ms.tgt_pltfrm="vm-windows"
	ms.devlang="na"
	ms.topic="article"
	ms.date="09/18/2016"
	wacn.date=""
	ms.author="glimoli;genli"/>  


# 准备好要上载到 Azure 的 Windows VHD
若要将 Windows VM 从本地上载到 Azure，必须正确地准备好虚拟硬盘 (VHD)。在将 VHD 上载到 Azure 之前，建议先完成几个步骤。本文介绍如何准备需上载到 Azure 的 Windows VHD，以及[何时以何种方式使用 Sysprep](#step23)。

## 准备虚拟磁盘

>[AZURE.NOTE] Azure 不支持更新的 VHDX 格式。VHD 的大小必须是固定的而不是动态的。如果需要，请参阅下面有关如何从 VHDX 或动态磁盘进行转换的详细说明。VHD 允许的最大大小为 1,023 GB。

确保 Windows VHD 在本地服务器上正常工作。在尝试转换磁盘或将其上载到 Azure 之前，请解决 VM 本身内部的所有错误。

如果要将虚拟磁盘转换为 Azure 所需的格式，请使用以下部分中所述的方法之一。在运行任何虚拟磁盘转换过程或 Sysprep 之前备份 VM。

### 使用 Hyper-V 管理器转换
- 打开 Hyper-V 管理器，在左侧选择本地计算机。在本地计算机上面的菜单中，单击“操作”>“编辑磁盘”。
	- 在“查找虚拟硬盘”屏幕上，浏览到并选择你的虚拟磁盘。
	- 在下一个屏幕上选择“转换”
		- 如果需要从 VHDX 进行转换，请选择“VHD”，然后单击“下一步”
		- 如果需要从动态磁盘进行转换，请选择“固定大小”，然后单击“下一步”

	- 浏览到并选择“新 VHD 文件的路径”。
	- 单击“完成”以关闭。

### 使用 PowerShell 转换
可以使用 [Convert-VHD PowerShell cmdlet](http://technet.microsoft.com/zh-cn/library/hh848454.aspx) 来转换虚拟磁盘。以下示例演示如何从 VHDX 转换为 VHD，然后从动态类型转换为固定类型：

	Convert-VHD -Path c:\test\MY-VM.vhdx -DestinationPath c:\test\MY-NEW-VM.vhd -VHDType Fixed

### 从 VMware VMDK 磁盘格式转换
如果你有 [VMDK 文件格式](https://en.wikipedia.org/wiki/VMDK)的 Windows VM 映像，可以使用 [Microsoft 虚拟机转换器](https://www.microsoft.com/download/details.aspx?id=42497)将其转换为 VHD。有关详细信息，请阅读博客 [How to Convert a VMware VMDK to Hyper-V VHD](http://blogs.msdn.com/b/timomta/archive/2015/06/11/how-to-convert-a-vmware-vmdk-to-hyper-v-vhd.aspx)（如何将 VMware VMDK 转换为 Hyper-V VHD）。

## 准备要上载的 Windows 配置

> [AZURE.NOTE] 以[管理权限](https://technet.microsoft.com/zh-cn/library/cc947813.aspx)运行以下所有命令。

1. 删除路由表中的所有静态持久性路由：

	- 若要查看路由表，请运行 `route print`。
	- 请查看**持久性路由**部分。如果有持久性路由，请使用 route delete 将它删除。

2. 删除 WinHTTP 代理：

		netsh winhttp reset proxy

3. 将磁盘 SAN 策略配置为 [Onlineall](https://technet.microsoft.com/zh-cn/library/gg252636.aspx)：

		diskpart san policy=onlineall

4. 为 Windows 使用协调世界时 (UTC) 时间，将 Windows 时间 (w32time) 服务的启动类型设置为“自动”：

		REG ADD HKLM\SYSTEM\CurrentControlSet\Control\TimeZoneInformation /v RealTimeIsUniversal /t REG_DWORD /d 1
		sc config w32time start= auto


## 配置 Windows 服务
5. 确保下面的每个 Windows 服务均设置为 **Windows 默认值**。这些服务是使用以下列表中所述的启动设置配置的。可以运行以下命令重置启动设置：

		sc config bfe start= auto

		sc config dcomlaunch start= auto

		sc config dhcp start= auto

		sc config dnscache start= auto

		sc config IKEEXT start= auto

		sc config iphlpsvc start= auto

		sc config PolicyAgent start= demand

		sc config LSM start= auto

		sc config netlogon start= demand

		sc config netman start= demand

		sc config NcaSvc start= demand

		sc config netprofm start= demand

		sc config NlaSvc start= auto

		sc config nsi start= auto

		sc config RpcSs start= auto

		sc config RpcEptMapper start= auto

		sc config termService start= demand

		sc config MpsSvc start= auto

		sc config WinHttpAutoProxySvc start= demand

		sc config LanmanWorkstation start= auto

		sc config RemoteRegistry start= auto

## 远程桌面配置
6. 如果有任何自签名证书绑定到远程桌面协议 (RDP) 侦听器，请删除这些证书：

		REG DELETE "HKLM\SYSTEM\CurrentControlSet\Control\Terminal Server\WinStations\RDP-Tcp\SSLCertificateSHA1Hash"

	有关配置 RDP 侦听器证书的详细信息，请参阅 [Listener Certificate Configurations in Windows Server](https://blogs.technet.microsoft.com/askperf/2014/05/28/listener-certificate-configurations-in-windows-server-2012-2012-r2/)（Windows Server 中的侦听器证书配置）

7. 配置 RDP 服务的 [KeepAlive](https://technet.microsoft.com/zh-cn/library/cc957549.aspx) 值：

		REG ADD "HKLM\SOFTWARE\Policies\Microsoft\Windows NT\Terminal Services" /v KeepAliveEnable /t REG_DWORD  /d 1 /f

		REG ADD "HKLM\SOFTWARE\Policies\Microsoft\Windows NT\Terminal Services" /v KeepAliveInterval /t REG_DWORD  /d 1 /f

		REG ADD "HKLM\SYSTEM\CurrentControlSet\Control\Terminal Server\Winstations\RDP-Tcp" /v KeepAliveTimeout /t REG_DWORD /d 1 /f

8. 配置 RDP 服务的身份验证模式：

		REG ADD "HKLM\SYSTEM\CurrentControlSet\Control\Terminal Server\WinStations\RDP-Tcp" /v UserAuthentication /t REG_DWORD  /d 1 /f

		REG ADD "HKLM\SYSTEM\CurrentControlSet\Control\Terminal Server\WinStations\RDP-Tcp" /v SecurityLayer /t REG_DWORD  /d 1 /f

		REG ADD "HKLM\SYSTEM\CurrentControlSet\Control\Terminal Server\WinStations\RDP-Tcp" /v fAllowSecProtocolNegotiation /t REG_DWORD  /d 1 /f

9. 通过在注册表中添加以下子项来启用 RDP 服务：

		REG ADD "HKLM\SYSTEM\CurrentControlSet\Control\Terminal Server" /v fDenyTSConnections /t REG_DWORD  /d 0 /f

## 配置 Windows 防火墙规则
10. 允许 WinRM 通过三个防火墙配置文件（“域”、“专用”和“公共”）并启用 PowerShell 远程服务：

		Enable-PSRemoting -force

11. 确保已部署以下来宾操作系统防火墙规则：

	- 入站

			netsh advfirewall firewall set rule dir=in name="File and Printer Sharing (Echo Request - ICMPv4-In)" new enable=yes

			netsh advfirewall firewall set rule dir=in name="Network Discovery (LLMNR-UDP-In)" new enable=yes

			netsh advfirewall firewall set rule dir=in name="Network Discovery (NB-Datagram-In)" new enable=yes

			netsh advfirewall firewall set rule dir=in name="Network Discovery (NB-Name-In)" new enable=yes

			netsh advfirewall firewall set rule dir=in name="Network Discovery (Pub-WSD-In)" new enable=yes

			netsh advfirewall firewall set rule dir=in name="Network Discovery (SSDP-In)" new enable=yes

			netsh advfirewall firewall set rule dir=in name="Network Discovery (UPnP-In)" new enable=yes

			netsh advfirewall firewall set rule dir=in name="Network Discovery (WSD EventsSecure-In)" new enable=yes

			netsh advfirewall firewall set rule dir=in name="Windows Remote Management (HTTP-In)" new enable=yes

			netsh advfirewall firewall set rule dir=in name="Windows Remote Management (HTTP-In)" new enable=yes

	- 入站和出站

			netsh advfirewall firewall set rule group="Remote Desktop" new enable=yes

			netsh advfirewall firewall set rule group="Core Networking" new enable=yes

	- 出站

		netsh advfirewall firewall set rule dir=out name="Network Discovery (LLMNR-UDP-Out)" new enable=yes

		netsh advfirewall firewall set rule dir=out name="Network Discovery (NB-Datagram-Out)" new enable=yes

		netsh advfirewall firewall set rule dir=out name="Network Discovery (NB-Name-Out)" new enable=yes

		netsh advfirewall firewall set rule dir=out name="Network Discovery (Pub-WSD-Out)" new enable=yes

		netsh advfirewall firewall set rule dir=out name="Network Discovery (SSDP-Out)" new enable=yes

		netsh advfirewall firewall set rule dir=out name="Network Discovery (UPnPHost-Out)" new enable=yes

		netsh advfirewall firewall set rule dir=out name="Network Discovery (UPnP-Out)" new enable=yes

		netsh advfirewall firewall set rule dir=out name="Network Discovery (WSD Events-Out)" new enable=yes

		netsh advfirewall firewall set rule dir=out name="Network Discovery (WSD EventsSecure-Out)" new enable=yes

		netsh advfirewall firewall set rule dir=out name="Network Discovery (WSD-Out)" new enable=yes


## 其他 Windows 配置步骤
12. 运行 `winmgmt /verifyrepository`，确认 Windows Management Instrumentation (WMI) 存储库是否一致。如果存储库已损坏，请参阅[这篇博客文章](https://blogs.technet.microsoft.com/askperf/2014/08/08/wmi-repository-corruption-or-not)。

13. 确保“引导配置数据”(BCD) 设置与以下内容相符：

		bcdedit /set {bootmgr} integrityservices enable

		bcdedit /set {default} device partition=C:

		bcdedit /set {default} integrityservices enable

		bcdedit /set {default} recoveryenabled Off

		bcdedit /set {default} osdevice partition=C:

		bcdedit /set {default} bootstatuspolicy IgnoreAllFailures

14. 删除所有其他传输驱动程序接口筛选器，例如分析 TCP 数据包的软件。
15. 为了确保磁盘正常运行且一致，请运行 `CHKDSK /f` 命令。
16.	卸载与物理组件相关的任何其他第三方软件和驱动程序，或卸载任何其他虚拟化技术。
17. 确保第三方应用程序未使用端口 3389。此端口用于 Azure 中的 RDP 服务。可以使用 `netstat -anob` 命令检查应用程序所使用的端口。
18.	如果要上载的 Windows VHD 是域控制器，请遵循[这些附加步骤](https://support.microsoft.com/kb/2904015)来准备磁盘。
19.	重新启动 VM，确保 Windows 仍可正常运行，并可以使用 RDP 连接来访问它。
20.	重置当前本地管理员密码，确保可以使用此帐户通过 RDP 连接登录 Windows。此访问权限由“允许通过远程桌面服务登录”策略对象控制。此对象位于“计算机配置”\\“Windows 设置”\\“安全设置”\\“本地策略”\\“用户权限分配”下面。


## 安装 Windows 更新
22. 安装最新的 Windows 更新。如果无法安装最新更新，请确保安装以下更新：

	- [KB3137061](https://support.microsoft.com/kb/3137061)：发生网络中断和数据损坏后，Azure VM 无法恢复

	- [KB3115224](https://support.microsoft.com/kb/3115224)：对运行 Windows Server 2012 R2 或 Windows Server 2012 主机的 VM 所做的可靠性改进

	- [KB3140410](https://support.microsoft.com/kb/3140410) MS16-031：Microsoft Windows 安全更新，解决权限提升过程中的问题：2016 年 3 月 8 日

	- [KB3063075](https://support.microsoft.com/kb/3063075)：在 Azure 中运行 Windows Server 2012 R2 虚拟机时记录了许多的 ID 129 事件

	- [KB3137061](https://support.microsoft.com/kb/3137061)：发生网络中断和数据损坏后，Azure VM 无法恢复

	- [KB3114025](https://support.microsoft.com/kb/3114025)：从 Windows 8.1 或 Server 2012 R2 访问 Azure 文件存储时性能不佳

	- [KB3033930](https://support.microsoft.com/kb/3033930)：该修补程序提升了 Windows 中 Azure 服务每个进程的 RIO 缓冲区的 64K 限制

	- [KB3004545](https://support.microsoft.com/kb/3004545)：无法在 Windows 中通过 VPN 连接访问托管服务的 Azure 中的虚拟机

	- [KB3082343](https://support.microsoft.com/kb/3082343)：当 Azure 站点到站点 VPN 隧道使用 Windows Server 2012 R2 RRAS 时跨界 VPN 连接断开

	- [KB3140410](https://support.microsoft.com/kb/3140410) MS16-031：Microsoft Windows 安全更新，解决权限提升过程中的问题：2016 年 3 月 8 日

	- [KB3146723](https://support.microsoft.com/kb/3146723) MS16-048：描述 CSRSS 的安全更新：2016 年 4 月 12 日
	- [KB2904100](https://support.microsoft.com/kb/2904100)：Windows 中发生磁盘 I/O 期间系统会冻结 <a id="step23"></a>
23. 如果你想要创建一个映像并从中部署多个计算机，则在将 VHD 上载到 Azure 之前，需要运行 `sysprep` 来通用化该映像。不需运行 `sysprep` 即可使用专用的 VHD。有关如何创建通用化映像的详细信息，请参阅以下文章：

	- [Create a VM image from an existing Azure VM using the Resource Manager deployment model（使用 Resource Manager 部署模型从现有 Azure VM 创建 VM 映像）](/documentation/articles/virtual-machines-windows-capture-image/)
	- [Create a VM image from an existing Azure VM using the Classic deployment modem（使用经典部署模型从现有 Azure VM 创建 VM 映像）](/documentation/articles/virtual-machines-windows-classic-capture-image/)
	- [针对服务器角色的 Sysprep 支持](https://msdn.microsoft.com/windows/hardware/commercialize/manufacture/desktop/sysprep-support-for-server-roles)



## 其他建议的配置

以下设置不影响 VHD 上载。但是，我们强烈建议你配置这些设置。

- 安装 [Azure 虚拟机代理](http://go.microsoft.com/fwlink/?LinkID=394789&clcid=0x409)。安装该代理后，可以启用 VM 扩展。VM 扩展实现了要用于 VM 的大多数关键功能，例如重置密码、配置 RDP 和其他许多功能。

- 转储日志可帮助排查 Windows 崩溃问题。启用转储日志收集：

		REG ADD "HKLM\SYSTEM\CurrentControlSet\Control\CrashControl" /v CrashDumpEnabled /t REG_DWORD /d 2 /f`

		REG ADD "HKLM\SOFTWARE\Microsoft\Windows\Windows Error Reporting\LocalDumps" /v DumpFolder /t REG_EXPAND_SZ /d "c:\CrashDumps" /f

		REG ADD "HKLM\SOFTWARE\Microsoft\Windows\Windows Error Reporting\LocalDumps" /v DumpCount /t REG_DWORD /d 10 /f

		REG ADD "HKLM\SOFTWARE\Microsoft\Windows\Windows Error Reporting\LocalDumps" /v DumpType /t REG_DWORD /d 2 /f

		sc config wer start= auto

- 在 Azure 中创建 VM 后，在 D: 驱动器上配置系统定义大小的页面文件

		REG ADD "HKLM\SYSTEM\CurrentControlSet\Control\Session Manager\Memory Management" /t REG_MULTI_SZ /v PagingFiles /d "D:\pagefile.sys 0 0" /f

## 后续步骤

- [将 Windows VM 映像上载到 Azure 以进行资源管理器部署](/documentation/articles/virtual-machines-windows-upload-image/)

<!---HONumber=Mooncake_1017_2016-->