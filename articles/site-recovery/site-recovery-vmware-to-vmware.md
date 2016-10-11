<properties 
	pageTitle="将本地 VMware 虚拟机或物理服务器复制到辅助站点 | Azure"
	description="按照本文可使用 Azure Site Recovery 将 VMware VM 或 Windows/Linux 物理服务器复制到辅助站点。"
	services="site-recovery"
	documentationCenter=""
	authors="nsoneji"
	manager="jwhit"
	editor=""/>

<tags
	ms.service="site-recovery"
	ms.workload="backup-recovery"
	ms.tgt_pltfrm="na"
	ms.devlang="na"
	ms.topic="article"
	ms.date="09/11/2016"
	wacn.date="10/10/2016"
	ms.author="nisoneji"/>


# 将本地 VMware 虚拟机或物理服务器复制到辅助站点


## 概述

Azure Site Recovery 中的 InMage Scout 在本地 VMware 站点之间提供实时复制。InMage Scout 随附在 Azure Site Recovery 服务订阅中。


## 先决条件

- **Azure 帐户** — 你需要一个 [Azure](https://azure.cn/) 帐户。可以从[试用版](/pricing/1rmb-trial)开始。[详细了解](/home/features/site-recovery/) Site Recovery 定价。


## 步骤 1：创建保管库

1. 登录到[经典管理门户](https://manage.windowsazure.cn)。
2. 单击“数据服务”>“恢复服务”>“Site Recovery 保管库”。
3. 单击“新建”>“快速创建”。
4. 在“名称”中，输入一个友好名称以标识此保管库。
5. 在“区域”中，为保管库选择地理区域。若要查看受支持的区域，请参阅 Azure Site Recovery 价格详细信息中的“地域可用性”[](/pricing/details/site-recovery/)。

检查状态栏以确认保管库已成功创建。保管库将以“活动”状态列在主要的“恢复服务”页上。

## 步骤 2：配置保管库并下载 InMage Scout 组件

1. 单击“创建保管库”。
2. 在“恢复服务”页上，单击保管库以打开“快速启动”页。
3. 在下拉列表中，选择“两个本地 VMWare 站点之间”。
4. 下载 InMage Scout。所有必需组件的安装程序文件都包含在下载的 .zip 文件中。


## 步骤 3：安装组件更新

阅读有关最新[更新](#updates)的信息。按以下顺序在服务器上安装更新文件：

1. RX 服务器（如果有）
2. 配置服务器
3. 进程服务器
3. 主目标服务器
4. vContinuum 服务器
5. 源服务器（仅适用于 Windows 服务器）

按以下步骤安装更新：

1. 下载[更新](http://download.microsoft.com/download/8/2/3/82362183-196B-48E6-BA21-A7F4A5CB594B/ASR%20Scout%208.0.1%20Update3.zip) zip 文件。此 zip 文件包含以下文件：

	-  RX\_8.0.3.0\_GA\_Update\_3\_6684045\_17Mar16.tar.gz
	-  CX\_Windows\_8.0.3.0\_GA\_Update\_3\_5048668\_16Mar16.exe
	-  UA\_Windows\_8.0.3.0\_GA\_Update\_3\_7101745\_04Apr16.exe
	-  UA\_RHEL6-64\_8.0.3.0\_GA\_Update\_3\_7101745\_04Apr16.zip
	-  vCon\_Windows\_8.0.3.0\_GA\_Update\_3\_6873369\_16Mar16.exe

2. 解压缩 zip 文件。
3. **RX 服务器**：将 **RX\_8.0.3.0\_GA\_Update\_3\_6684045\_17Mar16.tar.gz** 复制到 RX 服务器并将其解压缩。在解压缩的文件夹中运行 **/Install**。
4. **配置服务器/进程服务器**：将 **CX\_Windows\_8.0.3.0\_GA\_Update\_3\_5048668\_16Mar16.exe** 复制到配置服务器和进程服务器。双击以运行该文件。
5. **Windows 主目标服务器**：若要更新统一代理，请将 **UA\_Windows\_8.0.3.0\_GA\_Update\_3\_7101745\_04Apr16.exe** 复制到主目标服务器。双击以运行该文件。请注意，统一代理也适用于源服务器。它应该安装在源服务器以及下面所述的服务器上。
6. **Linux 主目标服务器**：若要更新统一代理，请将 **UA\_RHEL6-64\_8.0.3.0\_GA\_Update\_3\_7101745\_04Apr16.zip** 复制到主目标服务器并将它解压缩。在解压缩的文件夹中运行 **/Install**。
7. **vContinuum 服务器**：将 **vCon\_Windows\_8.0.3.0\_GA\_Update\_3\_6873369\_16Mar16.exe** 复制到 vContinuum 服务器。确保已关闭 vContinuum 向导。双击以运行该文件。
8. **Windows 源服务器**：若要更新统一代理，请将 **UA\_Windows\_8.0.3.0\_GA\_Update\_3\_7101745\_04Apr16.exe** 复制到源服务器。双击以运行该文件。 

## 步骤 4：设置复制
1. 设置源与目标 VMware 站点之间的复制。
2. 有关指导，请使用随产品下载的 InMage Scout 文档。你也可以访问以下文档：

	- [发行说明](http://download.microsoft.com/download/4/5/0/45008861-4994-4708-BFCD-867736D5621A/InMage_Scout_Standard_Release_Notes.pdf)
	- [兼容性对照表](http://download.microsoft.com/download/C/D/A/CDA1221B-74E4-4CCF-8F77-F785E71423C0/InMage_Scout_Standard_Compatibility_Matrix.pdf)
	- [用户指南](http://download.microsoft.com/download/E/0/8/E08B3BCE-3631-4CED-8E65-E3E7D252D06D/InMage_Scout_Standard_User_Guide_8.0.1.pdf)
	- [RX 用户指南](http://download.microsoft.com/download/A/7/7/A77504C5-D49F-4799-BBC4-4E92158AFBA4/InMage_ScoutCloud_RX_User_Guide_8.0.1.pdf)
	- [快速安装指南](http://download.microsoft.com/download/6/8/5/685E761C-8493-42EB-854F-FE24B5A6D74B/InMage_Scout_Standard_Quick_Install_Guide.pdf)


##<a id="updates"></a>## 更新

### ASR Scout 8.0.1 Update 3
Update 3 包含以下 bug 修复和增强功能：

- 配置服务器和 RX 位于代理后面时无法向 Site Recovery 保管库注册。
- 恢复点目标 (RPO) 未满足的小时数不会在运行状况报告中更新。
- 当 ESX 硬件详细信息或网络详细信息包含任何 UTF-8 字符时，配置服务器不会与 RX 同步。
- Windows Server 2008 R2 域控制器无法在恢复之后启动。
- 脱机同步不按预期进行。
- 虚拟机 (VM) 故障转移后，CX UI 中的复制对删除会长时间停滞，用户将不能完成故障回复或恢复操作。
- 由一致性作业执行的整体快照操作已经过优化，可帮助减少应用程序（例如 SQL 客户端）断开连接的问题。
- 通过减少在 Windows 上创建快照所需的内存使用量，改善了一致性工具 (VACP.exe) 的性能。
- 密码大于 16 个字符时，推送安装服务会崩溃。
- vContinuum 不会在凭据更改时检查和提示输入新的 vCenter 凭据。
- 在 Linux 上，主目标缓存管理器 (cachemgr) 不会从进程服务器下载文件，从而发生复制对限制。
- 当所有节点上的物理故障转移群集 (MSCS) 磁盘顺序不一致时，将不会设置某些群集卷的复制。
<br/>请注意，群集需要重新保护才能利用这一修复。
- 在 RX 从 Scout 7.1 升级到 Scout 8.0.1 之后，SMTP 功能不按预期工作。
- 已在日志中添加更多统计信息，以便回滚操作跟踪完成此操作所需的时间。
- 已添加对源服务器上 Linux 操作系统的支持：
	- Red Hat Enterprise Linux (RHEL) 6 update 7
	- CentOS 6 Update 7
- CX 和 RX UI 现在可以针对进入位图模式的对显示通知。
- 已将以下安全修复程序添加到 RX 中：

**问题说明**|**实现过程**
---|---
通过参数篡改绕过授权|已将访问权限限制为不适用的用户。
跨站点请求伪造|实施针对每页随机生成的页面令牌概念。<br/>对于此操作，你将看到：<li>相同用户只有一个单一登录实例。</li><li>页面刷新无效--将重定向到仪表板。</li>
恶意文件上载|将文件限制为特定的扩展名。允许的扩展名：7z、aiff、asf、avi、bmp、csv、doc、docx、fla、flv、gif、gz、gzip、jpeg、jpg、log、mid、mov、mp3、mp4、mpc、mpeg、mpg、ods、odt、pdf、png、ppt、pptx、pxd、qt、ram、rar、rm、rmi、rmvb、rtf、sdc、sitd、swf、sxc、sxw、tar、tgz、tif、tiff、txt、vsd、wav、wma、wmv、xls、xlsx、xml 和 zip。
持久性跨站点脚本 | 已添加输入验证。


>[AZURE.NOTE]
>
>-	所有站点恢复更新都是累积更新。Update3 包含 Update1 和 Update2 的所有修复程序。Update 3 可以直接应用于 8.0.1 GA。
>-	在系统上应用配置服务器和 RX 更新后，便不能将其回滚。

### Azure Site Recovery Scout 8.0.1 Update 2 (Update 03Dec15)

Update 2 中的修复包括：

- **配置服务器**：修复了在 Site Recovery 中注册配置服务器时妨碍 31 天免费计量功能正常使用的问题。
- **统一代理** — 修复了 Update 1 中的问题，该问题导致从版本 8.0 升级到 8.0.1 时，更新无法安装在主目标服务器上。


### Azure Site Recovery Scout 8.0.1 Update 1

Update 1 包含以下 bug 修复和新功能：

- 每个服务器实例享有 31 天的免费保护。这样，你便可以测试功能或建立概念认证。
	- 从使用 ASR Scout 首次保护服务器的时间开始计算的前 31 天，服务器上的所有操作（包括故障转移和故障回复）都是免费的。
	- 从第 32 天起，将会根据标准实例费率，针对每个受保护的服务器，向客户拥有的站点收取 Azure Site Recovery 保护费用。
	- 在 Azure Site Recovery 保管库的“仪表板”页上随时会显示当前计费的受保护服务器数目。
- 添加了对 vSphere命令行接口 (vCLI) 5.5 Update 2 的支持。
- 已添加对源服务器上 Linux 操作系统的支持：
	- RHEL 6 Update 6
	- RHEL 5 Update 11
	- CentOS 6 Update 6
	- CentOS 5 Update 11
- 用于解决以下问题的 Bug 修复：
	- 配置服务器或 RX 服务器的保管库注册失败。
	- 当群集虚拟机在恢复期间重新受保护时，群集卷不会按预期显示。
	- 当主目标服务器托管在与本地生产虚拟机不同的 ESXi 服务器中时，故障回复失败。
	- 升级到 8.0.1 时，配置文件权限发生更改，从而影响保护和操作。
	- 不按预期强制实施重新同步阈值，从而导致不一致的复制行为。
	- RPO 设置未正常显示在配置服务器界面中。未压缩的数据值错误地显示压缩值。
	-  在 vContinuum 向导中使用“删除”操作不会按预期执行删除，因而无法从配置服务器界面删除复制内容。
	-  在 vContinuum 向导中保护 MSCS 虚拟机期间，单击磁盘视图中的“详细信息”会自动取消选择磁盘。
	- 在执行物理到虚拟 (P2V) 方案期间，所需的 HP 服务（例如 CIMnotify、CqMgHost）不会在虚拟机恢复中变为“手动”。这会导致启动时间延长。
	- 当主目标服务器上的磁盘数超过 26 个时，Linux 虚拟机保护会失败。

## 后续步骤

请在 [Azure 恢复服务论坛](https://social.msdn.microsoft.com/forums/zh-cn/home?forum=hypervrecovmgr)上发布任何问题。

<!---HONumber=Mooncake_0926_2016-->