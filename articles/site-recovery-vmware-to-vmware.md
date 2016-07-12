<properties 
	pageTitle="将本地 VMware 虚拟机或物理服务器复制到辅助站点 | Azure"
	description="按照本文可使用 Azure Site Recovery 将 VMware VM 或 Windows/Linux 物理服务器复制到辅助站点。"
	services="site-recovery" 
	documentationCenter="" 
	authors="rayne-wiselman" 
	manager="jwhit" 
	editor=""/>

<tags 
	ms.service="site-recovery" 
	ms.date="02/16/2016"
	wacn.date="04/05/2016"/>


# 将本地 VMware 虚拟机或物理服务器复制到辅助站点


##概述

Azure Site Recovery 中的 InMage Scout 在本地 VMware 站点之间提供实时复制。InMage Scout 已随附在 Azure Site Recovery 服务订阅中。


## 先决条件

- **Azure 帐户** — 你需要一个 [Azure](https://azure.cn/) 帐户。可以从[试用版](/pricing/1rmb-trial)开始。[详细了解](/home/features/site-recovery/) Site Recovery 定价。


## 步骤 1：创建保管库

1. 登录到[管理门户](https://manage.windowsazure.cn)。
2. 单击“数据服务”>“恢复服务”>“Site Recovery 保管库”。
3. 单击“新建”>“快速创建”。
4. 在“名称”中，输入一个友好名称以标识此保管库。
5. 在“区域”中，为保管库选择地理区域。若要查看受支持的区域，请参阅 Azure Site Recovery 价格详细信息中的“地域可用性”[](/home/features/site-recovery/pricing/)。

检查状态栏以确认保管库已成功创建。保管库将以“活动”形式列在主要的“恢复服务”页上。

## 步骤 2：配置保管库并下载 InMage Scout 组件

1. 单击“创建保管库”。
2. 在“恢复服务”页中，单击保管库以打开“快速启动”页。
3. 在下拉列表中，选择“两个本地 VMWare 站点之间”。
4. 下载 InMage Scout。所有必需组件的安装程序文件都包含在下载的 zip 文件中。


## 步骤 3：安装组件更新

阅读有关最新[更新](#updates)的信息。按以下顺序安装更新文件：

1. RX 服务器（如果有）
2. 配置服务器
3. 进程服务器
3. 主目标服务器。
4. vContinuum 服务器。

按如下所述安装：

1. 下载 [更新](http://download.microsoft.com/download/9/F/D/9FDC6001-1DD0-4C10-BDDD-8A9EBFC57FDF/ASR Scout 8.0.1 Update1.zip) zip 文件。此 zip 文件包含以下文件：

	-  RX_8.0.1.0_GA_Update_1_3279231_23Jun15.tar.gz
	-  CX_Windows_8.0.2.0_GA_Update_2_4306954_21Aug15.exe
	-  UA_Windows_8.0.1.0_GA_Update_1_3259401_23Jun15.exe
	-  UA_RHEL6-64_8.0.1.0_GA_Update_1_3259401_23Jun15.tar.gz
	-  vCon_Windows_8.0.1.0_GA_Update_1_3259523_23Jun15.exe
2. 解压缩 zip 文件。
2. **RX 服务器**：将 **RX_8.0.1.0_GA_Update_1_3279231_23Jun15.tar.gz** 复制到 RX 服务器并将其解压缩。在解压缩的文件夹中运行 **/Install**。
2. **配置服务器/进程服务器**：将 **CX_Windows_8.0.2.0_GA_Update_2_4306954_21Aug15.exe** 复制到配置服务器和进程服务器。双击以运行该文件。
3. **Windows 主目标服务器**：若要更新统一代理，请将 **UA_Windows_8.0.1.0_GA_Update_1_3259401_23Jun15.exe** 复制到主目标服务器。双击以运行该文件。请注意，适用于 Windows 的统一代理并不适用于源服务器。只能将它安装在 Windows 主目标服务器上。
4. **Linux 主目标服务器**：若要更新统一代理，请将 **UA_RHEL6-64_8.0.1.0_GA_Update_1_3259401_23Jun15.tar.gz** 复制到主目标服务器并将它解压缩。在解压缩的文件夹中运行 **/Install**。
5. **vContinuum 服务器**：将 **vCon_Windows_8.0.1.0_GA_Update_1_3259523_23Jun15.exe** 复制到 vContinuum 服务器。确保已关闭 vContinuum 向导。双击以运行该文件。

## 步骤 4：设置复制
5. 设置源与目标 VMware 站点之间的复制。
6. 有关指导，请使用随产品下载的 InMage Scout 文档。你也可以访问以下文档：

	- [发行说明](http://download.microsoft.com/download/4/5/0/45008861-4994-4708-BFCD-867736D5621A/InMage_Scout_Standard_Release_Notes.pdf)
	- [兼容性对照表](http://download.microsoft.com/download/C/D/A/CDA1221B-74E4-4CCF-8F77-F785E71423C0/InMage_Scout_Standard_Compatibility_Matrix.pdf)
	- [用户指南](http://download.microsoft.com/download/E/0/8/E08B3BCE-3631-4CED-8E65-E3E7D252D06D/InMage_Scout_Standard_User_Guide_8.0.1.pdf)
	- [RX 用户指南](http://download.microsoft.com/download/A/7/7/A77504C5-D49F-4799-BBC4-4E92158AFBA4/InMage_ScoutCloud_RX_User_Guide_8.0.1.pdf)
	- [快速安装指南](http://download.microsoft.com/download/6/8/5/685E761C-8493-42EB-854F-FE24B5A6D74B/InMage_Scout_Standard_Quick_Install_Guide.pdf)


##<a id="updates"></a> 更新
### ASR Scout 8.0.1 Update 03Dec15

更新 03-Dec-15 中的修复包括：

- **配置服务器** — 修复了在 Site Recovery 中注册配置服务器时妨碍 31 天免费计量功能正常使用的问题。
- **统一代理** — 修复了主目标的 Update 1 中的问题，该问题导致从版本 8.0 升级到 8.0.1 时，更新无法安装在主目标服务器上。

>[AZURE.NOTE]
>
>-	所有 ASR 更新都是累积更新。
>-	CS 和 RX 更新在应用到系统以后都无法回退。


### ASR Scout 8.0.1 Update 1

此最新更新包含 Bug 修复和新功能：

- 每个服务器实例享有 31 天的免费保护。这样，你便可以测试功能或建立概念认证。
	- 从使用 ASR Scout 首次保护服务器的时间开始计算的前 31 天，服务器上的所有操作（包括故障转移和故障回复）都是免费的。
	- 从第 32 天起，将会根据标准实例费率，针对每个受保护的服务器，向客户拥有的站点收取 Azure Site Recovery 保护费用。
	- 在 Azure Site Recovery 保管库的“仪表板”页上随时会显示当前计费的受保护服务器数目。
- 已添加 vCLI 5.5 Update 2 的支持。
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
	- 在执行 P2V 方案期间，所需的 HP 服务（例如 CIMnotify、CqMgHost）不会在恢复虚拟机中变为“手动”，因而导致引导时间延长。
	- 当主目标服务器上的磁盘数超过 26 个时，受保护的 Linux 虚拟机发生故障。
	
## 后续步骤

请在 [Azure 恢复服务论坛](https://social.msdn.microsoft.com/forums/zh-cn/home?forum=hypervrecovmgr)上发布你的任何问题。

<!---HONumber=Mooncake_0328_2016-->