<properties
    pageTitle="Azure 文件存储问题疑难解答 | Azure"
    description="Azure 文件存储问题疑难解答"
    services="storage"
    documentationcenter=""
    author="genlin"
    manager="felixwu"
    editor="na"
    tags="storage" />  

<tags
    ms.assetid="fbc5f600-131e-4b99-828a-42d0de85fff7"
    ms.service="storage"
    ms.workload="na"
    ms.tgt_pltfrm="na"
    ms.devlang="na"
    ms.topic="article"
    ms.date="11/13/2016"
    wacn.date="12/05/2016"
    ms.author="genli" />  


# Azure 文件存储问题疑难解答
本文列出了从 Windows 和 Linux 客户端连接时与 Azure 文件存储相关的常见问题。它还提供了这些问题的可能原因和解决方法。

**常规问题（在 Windows 和 Linux 客户端中均存在）**

* [尝试打开文件时配额出错](#quotaerror)
* [从 Windows 或 Linux 访问 Azure 文件存储时性能不佳](#slowboth)

**Windows 客户端问题**

* [从 Windows 8.1 或 Windows Server 2012 R2 访问 Azure 文件存储时性能不佳](#windowsslow)
* [尝试装载 Azure 文件共享时出现错误 53](#error53)
* [Net use 成功，但未显示装载在 Windows 资源管理器上的 Azure 文件共享](#netuse)
* [我的存储帐户包含“/”且 net use 命令失败](#slashfails)
* [我的应用程序/服务无法访问装载的 Azure 文件驱动器。](#accessfiledrive)
* [其他性能优化建议](#additional)

**Linux 客户端问题**

* [将文件上传/复制到 Azure 文件时出现错误“正在将文件复制到不支持加密的目标”](#encryption)
* [现有文件共享上出现“主机已关闭”错误，或者在装入点上执行列表命令时 shell 挂起](#errorhold)
* [尝试在 Linux VM 上装载 Azure 文件时出现装入错误 115 ](#error15)
* [Linux VM 在类似“ls”的命令中遇到随机延迟](#delayproblem)

<a id="quotaerror"></a>

## 尝试打开文件时配额出错
在 Windows 中，将收到类似下文的错误消息：

**1816 ERROR\_NOT\_ENOUGH\_QUOTA <--> 0xc0000044**

**STATUS\_QUOTA\_EXCEEDED**

**可用配额不足，无法处理此命令**

**无效句柄值 GetLastError: 53**

在 Linux 中，将收到类似下文的错误消息：

**<文件名> [权限被拒绝]**

**已超出磁盘配额**

### 原因
问题原因是已达到文件所允许的并发打开句柄数上限。

### 解决方案
关闭某些句柄以减少并发打开句柄数，然后重试。有关详细信息，请参阅 [Azure 存储性能和可伸缩性核对清单](/documentation/articles/storage-performance-checklist/)。

<a id="slowboth"></a>

## 从 Windows 或 Linux 访问文件存储时性能不佳
* 如果没有特定的 I/O 大小下限要求，建议使用 1 MB 的 I/O 大小获得最佳性能。
* 如果知道使用写入扩展的文件的最终大小，并且当尚未在文件上写入的尾部包含零时软件没有兼容性问题，请提前设置文件大小，而不是每次写入都是扩展写入。

<a id="windowsslow"></a>

## 从 Windows 8.1 或 Windows Server 2012 R2 访问文件存储时的性能不佳
对于运行 Windows 8.1 或 Windows Server 2012 R2 的客户端，请确保安装有修补程序 [KB3114025](https://support.microsoft.com/zh-cn/kb/3114025)。该程序可提升创建和关闭句柄时的性能。

可运行以下脚本，检查是否安装了该修补程序：

`reg query HKLM\SYSTEM\CurrentControlSet\Services\LanmanWorkstation\Parameters\Policies`  


如果已安装，将显示以下输出：

**HKEY\_LOCAL\_MACHINE\\SYSTEM\\CurrentControlSet\\Services\\LanmanWorkstation\\Parameters\\Policies**

**{96c345ef-3cac-477b-8fcd-bea1a564241c} REG\_DWORD 0x1**

> [AZURE.NOTE]自 2015 年 12 月起，Azure 应用商店中的 Windows Server 2012 R2 映像默认安装有修补程序 KB3114025。


<a id="additional"></a>

## 其他性能优化建议
请勿为请求写入权限但不允许读取权限的缓存 I/O 创建或打开文件。这就是说，当调用 **CreateFile()** 时，永远不要只指定 **GENERIC\_WRITE**，而是要始终指定 **GENERIC\_READ |GENERIC\_WRITE**。只写句柄无法在本地缓存小型写入，即使它是文件唯一打开的句柄。这会严重影响小型写入操作的性能。请注意，指向 CRT **fopen()** 的“a”模式会打开只写句柄。

<a id="error53"></a>

## 尝试装载或卸载 Azure 文件共享时出现“错误 53”
此问题的原因可能是：

### 原因 1
“发生系统错误 53。访问被拒绝。” 出于安全原因，如果信道未加密，且未从 Azure 文件共享所驻留的数据中心尝试连接，则到 Azure 文件共享的连接将受阻。如果用户的客户端 OS 不支持 SMB 加密，则不会加密信道。当用户尝试从本地或其他数据中心装载文件共享时，这将表示为“发生系统错误 53。访问被拒绝”错误消息。Windows 8、Windows Server 2012 及更高版本的每次协商均要求其包含支持加密的 SMB 3.0。

### 原因 1 的解决方案
通过满足 Windows 8、Windows Server 2012 或更高版本要求的客户端进行连接，或用于连接的虚拟机要与 Azure 文件共享所用的 Azure 存储帐户位于同一数据中心。

### 原因 2
如果端口 445 到 Azure 文件数据中心的出站通信受阻，则在安装 Azure 文件共享时可能会出现“系统错误53”。请单击[此处](http://social.technet.microsoft.com/wiki/contents/articles/32346.azure-summary-of-isps-that-allow-disallow-access-from-port-445.aspx)，概要查看允许或禁止从端口 445 进行访问的 ISP。

Comcast 和某些 IT 组织阻止此端口。若要了解是否由此造成“系统错误 53”，可使用 Portqry 查询 TCP:445 终结点。如果 TCP:445 终结点显示为“已筛选”，则表示 TCP端口受阻。示例查询如下：

`g:\DataDump\Tools\Portqry>PortQry.exe -n [storage account name].file.core.chinacloudapi.cn -p TCP -e 445`  


如果 TCP 445 受到网络路径中的规则阻止，将显示以下输出：

**TCP 端口 (445microsoft-ds 服务): 筛选**

若要深入了解如何使用 Portqry，请参阅 [Portqry.exe 命令行实用工具说明](https://support.microsoft.com/zh-cn/kb/310099)。

### 原因 2 的解决方案
与 IT 组织配合，向 [Azure IP 范围](https://www.microsoft.com/download/details.aspx?id=41653)开放端口 445 出站。

### 原因 3
如果在客户端上启用 NTLMv1 通信，也会收到“系统错误 53”。启用 NTLMv1 会降低客户端的安全性。因此，将阻止 Azure 文件的通信。若要验证这是否是错误原因，请验证以下注册表子项是否设为值 3：

HKLM\\SYSTEM\\CurrentControlSet\\Control\\Lsa > LmCompatibilityLevel。

有关详细信息，请参阅 TechNet 上的 [LmCompatibilityLevel](https://technet.microsoft.com/zh-cn/library/cc960646.aspx) 主题。

### 原因 3 的解决方案
若要解决此问题，请将 HKLM\\SYSTEM\\CurrentControlSet\\Control\\Lsa 注册表项中的 LmCompatibilityLevel 值恢复为默认值 3。

Azure 文件仅支持 NTLMv2 身份验证。请确保客户端上应用了组策略。这将防止发生此错误。这也被视为最佳安全方案。有关详细信息，请参阅[如何通过组策略配置客户端以使用 NTLMv2](https://technet.microsoft.com/zh-cn/library/jj852207(v=ws.11).aspx)

建议策略设置为**仅发送 NTLMv2 响应**。这对应于注册表值为 3。客户端仅使用 NTLMv2 身份验证；如果服务器支持，则使用 NTLMv2 会话安全。域控制器接受 LM、NTLM 和 NTLMv2 身份验证。

<a id="netuse"></a>

## Net use 成功，但未显示装载在 Windows 资源管理器上的 Azure 文件共享
### 原因
默认情况下，Windows 资源管理器不以管理员身份运行。如果通过 Administrator 命令提示符运行 **net use**，会将网络驱动器映射为“管理员身份”。 由于映射的驱动器以用户为中心，如果在其他用户帐户上安装了这些驱动器，则登录的用户帐户不会显示它们。

### 解决方案
通过非管理员命令行中装载共享。或者，可按照[此 TechNet 主题](https://technet.microsoft.com/zh-cn/library/ee844140.aspx)配置 **EnableLinkedConnections** 注册表值。

<a id="slashfails"></a>

## 我的存储帐户包含“/”且 net use 命令失败
### 原因
当 **net use** 命令在命令提示符 (cmd.exe) 下运行时，添加“/”作为命令行选项对其进行解析。这会导致驱动器映射失败。

### 解决方案
可使用下述某个步骤解决此问题：

• 使用以下 PowerShell 命令：

`New-SmbMapping -LocalPath y: -RemotePath \\server\share  -UserName acountName -Password "password can contain / and \ etc"`  


可在批处理文件中执行以下命令

`Echo new-smbMapping ... | powershell -command –`  


• 将密钥用双引号括起以解决此问题（除非第一个字符是“/”）。若是此例外，则使用交互模式，然后单独输入密码或重新生成密钥，获取不以正斜杠 (/) 字符开头的密钥。

<a id="accessfiledrive"></a>

## 我的应用程序/服务无法访问装载的 Azure 文件驱动器
### 原因
根据用户装载驱动器。如果应用程序或服务在其他用户帐户下运行，则驱动器不可见。

### 解决方案
从应用程序所在的同一用户帐户下装载驱动器。可使用 psexec 等工具完成此操作。

或者，可创建具有网络服务或系统帐户相同特权的新用户，然后运行该帐户下的 **cmdkey** 和 **net use**。用户名称应为存储帐户名称，密码应为存储帐户密钥。还有一种针对 **net use** 的方法，就是传入 **net use** 命令的用户名和密码参数中的存储帐户名及密钥。

按照说明操作后，在为系统/网络服务帐户运行 **net use** 时，可能会收到以下错误消息：“发生系统错误 1312。指定的登录会话不存在。其可能已终止”。若发生此情况，请确保传递到 **net use** 的用户名包含域信息（例如“[存储帐户名].file.core.chinacloudapi.cn”）。

<a id="encryption"></a>

## “正在将文件复制到不支持加密的目标”错误
### 原因
可将 Bitlocker 加密的文件复制到 Azure 文件。但文件存储不支持 NTFS EFS。因此，此情况下可能要使用 EFS。如果通过 EFS 加密文件，则可能无法复制到文件存储，除非复制命令将解密复制后的文件。

### 解决方法
必须先解密，才能将文件复制到文件存储。为此，可执行下述某种方法：

• 使用 **copy /d**。

• 设置以下注册表项：

* 路径=HKLM\\Software\\Policies\\Microsoft\\Windows\\System
* 值类型=DWORD
* 名称= CopyFileAllowDecryptedRemoteDestination
* 值= 1

但请注意，设置注册表项会影响所有到网络共享的复制操作。

<a id="errorhold"></a>

## 现有文件共享上出现“主机已关闭”错误，或者在装入点上运行列表命令时 shell 挂起
### 原因
客户端长时间处于空闲状态时，Linux 客户端上将出现此错误。此错误发生时，客户端断开连接且客户端连接超时。

### 解决方案
Linux 内核的[更改集](https://git.kernel.org/cgit/linux/kernel/git/torvalds/linux.git/commit/fs/cifs?id=4fcd1813e6404dd4420c7d12fb483f9320f0bf93)中修复了此问题，该更改集正在等待移植到 Linux 分发中。

为了解决此问题，请在 Azure 文件共享中保留一个定期写入的文件，保持连接并避免进入空闲状态。此操作必须为写入操作，例如在文件上重新写入创建/修改日期。否则可能会收到缓存结果，且操作可能不会触发连接。

<a id="error15"></a>

## 尝试在 Linux VM 上装载 Azure 文件时出现“装入错误 115”
### 原因
Linux 分发尚不支持 SMB 3.0 中的加密功能。在某些分发中，用户尝试通过 SMB 3.0 装载 Azure 文件时可能因功能缺失而收到“115”错误消息。

### 解决方案
如果使用的 Linux SMB 客户端不支持加密，装载 Azure 文件时使用的 SMB 2.1 需来自文件存储帐户所在的数据中心内的 Linux VM。

<a id="delayproblem"></a>

## Linux VM 在类似“ls”的命令中遇到随机延迟
### 原因
装载命令中没有 **serverino** 选项时，会发生此情况。若没有 **serverino**，ls 命令会在每个文件上运行 **stat**。

### 解决方案
请检查“/etc/fstab”条目中的 **serverino**：

//azureuser.file.core.chinacloudapi.cn/wms/comer on /home/sampledir type cifs (rw,nodev,relatime,vers=2.1,sec=ntlmssp,cache=strict,username=xxx,domain=X,
file\_mode=0755,dir\_mode=0755,serverino,rsize=65536,wsize=65536,actimeo=1)

如果 **serverino** 选项不存在，请选中 **serverino** 选项，卸载并再次装载 Azure 文件。

## 了解详细信息
* [在 Windows 上开始使用 Azure 文件存储](/documentation/articles/storage-dotnet-how-to-use-files/)
* [开始在 Linux 上使用 Azure 文件存储](/documentation/articles/storage-how-to-use-files-linux/)

<!---HONumber=Mooncake_1128_2016-->