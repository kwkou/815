<properties
	pageTitle="对与基于 Linux 的 VM 的安全外壳 (SSH) 连接进行故障排除 | Windows Azure"
	description="如果无法连接基于 Linux 的 Azure 虚拟机，则可以按照这些步骤来隔离问题源。"
	services="virtual-machines"
	documentationCenter=""
	authors="dsk-2015"
	manager="timlt"
	editor=""
	tags="azure-service-management,azure-resource-manager"/>


<tags
	ms.service="virtual-machines"
	ms.date="07/07/2015"
	wacn.date="11/02/2015"/>

# 对于基于 Linux 的 Azure 虚拟机的 Secure Shell (SSH) 连接进行故障排除

如果无法连接到基于 Linux 的 Azure 虚拟机，可参见本文，其中介绍了如何隔离和更正该问题的有序方法。这同时适用于资源管理器和经典部署模型。

## 步骤 1：重置 SSH 配置、密钥或密码

按照[如何为基于 Linux 的虚拟机重置密码或 SSH](/documentation/articles/virtual-machines-linux-use-vmaccess-reset-password-or-ssh) 中的说明，在虚拟机上执行操作。根据这些说明，你可以：

- 重置密码或 SSH 密钥。
- 创建新的 sudo 用户帐户。
- 重置 SSH 配置。

如果 SSH 客户端仍然无法连接到虚拟机上的 SSH 服务，则可能由于许多原因。以下是所涉及的组件集。

![](./media/virtual-machines-troubleshoot-ssh-connections/ssh-tshoot1.png)

以下各节将逐步隔离和确定该问题的多种原因，并提供相应的解决方案和解决方法。

## 步骤 2：详细故障排除之前的预备步骤

首先，在 Azure 管理门户或 Azure 预览门户中检查虚拟机的状态。

在 Azure 管理门户中：

1. 单击“虚拟机”>“VM 名称”。
2. 单击该虚拟机所对应的“仪表板”，以检查其状态。
3. 单击“监视器”，以查看计算、存储和网络资源的最近活动。
4. 单击“终结点”以确保 SSH 流量有终结点。

在 Azure 预览门户中：

1. 单击“浏览”>“虚拟机”>“VM 名称”。如需查找在 Azure 资源管理器中创建的虚拟机，请单击“浏览”>“虚拟机 (v2)”>“VM 名称”。该虚拟机的状态窗格中应显示“正在运行”。向下滚动以显示计算、存储和网络资源的最近活动。
2. 单击“设置”以检查终结点、IP 地址和其他设置。

若要验证网络连接，请分析所配置的终结点，并确定是否可通过其他协议（例如 HTTP 或其他已知服务）连接到该虚拟机。

如果需要，可重新启动虚拟机或[调整虚拟机的大小](/documentation/articles/virtual-machines-size-specs)。

在执行这些步骤之后，重新尝试 SSH 连接。

## 步骤 3：详细的故障排除

SSH 客户端无法访问 Azure 虚拟机上的 SSH 服务可能是由于以下来源存在问题或配置错误：

- SSH 客户端计算机
- 组织边缘设备
- 云服务终结点和访问控制列表 (ACL)
- 网络安全组
- 基于 Linux 的 Azure 虚拟机

### 来源 1：SSH 客户端计算机

要使你的计算机不会成为问题或配置错误的来源，请验证你的计算机是否可以与另一台基于 Linux 的本地计算机建立 SSH 连接。

![](./media/virtual-machines-troubleshoot-ssh-connections/ssh-tshoot2.png)

如果不能，请检查你的计算机上是否有以下项：

- 本地防火墙设置阻止了入站或出站 SSH 流量 (TCP 22)
- 本地安装的客户端代理软件阻止了 SSH 连接
- 本地安装的网络监视软件阻止了 SSH 连接
- 监视流量或允许/禁止特定类型流量的其他类型的安全软件阻止了 SSH 连接

对于所有这些情况，请尝试暂时禁用可疑软件，然后尝试与本地计算机建立 SSH 连接以确定原因。然后，与网络管理员合作以更正软件的设置，从而允许 SSH 连接。

如果使用的是证书身份验证，则需验证你是否具有这些权限以访问主目录中的 .ssh 文件夹：

- Chmod 700 ~/.ssh
- Chmod 644 ~/.ssh/*.pub
- Chmod 600 ~/.ssh/id\_rsa（或其他任何可能存储有私钥的文件）
- Chmod 644 ~/.ssh/known\_hosts（包含已通过 SSH 连接到的主机）

### 来源 2：组织边缘设备

要使你的组织边缘设备不会成为问题或配置错误的来源，请验证直接连接到 Internet 的计算机是否可以与 Azure 虚拟机建立 SSH 连接。如果是通过站点到站点 VPN 或 ExpressRoute 连接来访问虚拟机，请跳转到[来源 4：网络安全组](#nsg)。

![](./media/virtual-machines-troubleshoot-ssh-connections/ssh-tshoot3.png)

如果没有直接连接到 Internet 的计算机，可以轻松地在其自己的资源组或云服务中创建新的 Azure 虚拟机，然后进行使用。有关详细信息，请参阅[在 Azure 中创建运行 Linux 的虚拟机](/documentation/articles/virtual-machines-linux-tutorial)。测试完成后，请删除资源组或虚拟机以及云服务。

如果可以创建与直接连接到 Internet 的计算机之间的 SSH 连接，则检查你的组织边缘设备中是否存在以下问题：

- 内部防火墙阻止了与 Internet 的 SSH 连接
- 代理服务器阻止了 SSH 连接
- 边界网络中的设备上运行的入侵检测或网络监视软件阻止了 SSH 连接

与网络管理员合作以更正组织边缘设备的设置，从而允许与 Internet 建立 SSH 流量连接。

### 来源 3：云服务终结点和 ACL

要使云服务终结点和 ACL 不会成为使用服务管理 API 创建的虚拟机的问题或配置错误来源，请验证同一虚拟网络中的其他 Azure 虚拟机是否可以与你的 Azure 虚拟机建立 SSH 连接。

![](./media/virtual-machines-troubleshoot-ssh-connections/ssh-tshoot4.png)

> [AZURE.NOTE]对于在 Azure 资源管理器中创建的虚拟机，请跳转到[来源 4：网络安全组](#nsg)。

如果同一虚拟网络中没有其他虚拟机，可以轻松创建一个新虚拟机。有关详细信息，请参阅[在 Azure 中创建运行 Linux 的虚拟机](/documentation/articles/virtual-machines-linux-tutorial)。测试完成后，请删除多余的虚拟机。

如果可以与同一虚拟网络中的某个虚拟机建立 SSH 连接，请检查：

- 目标虚拟机上 SSH 流量的终结点配置。终结点的专用 TCP 端口必须与虚拟机上 SSH 服务正在其上侦听的 TCP 端口（默认为 22）匹配。对于在 Azure 资源管理器中使用模板创建的虚拟机，请通过“浏览”>“虚拟机 (v2)”>“VM 名称”>“设置”>“终结点”，验证 Azure 预览门户中的 SSH TCP 端口号。
- 目标虚拟机上的 SSH 流量终结点的 ACL。ACL 允许你指定基于源 IP 地址允许或拒绝的从 Internet 传入的流量。错误配置的 ACL 可能会阻止 SSH 流量传入终结点。检查你的 ACL 以确保允许从你的代理服务器或其他边缘服务器的公共 IP 地址传入的流量。有关详细信息，请参阅[关于网络访问控制列表 (ACL)](/documentation/articles/virtual-networks-acl)。

要使终结点不会成为问题来源，请删除当前终结点，然后创建一个新的终结点并指定 **SSH** 名称（公用和专用端口号为 TCP 端口 22）。有关详细信息，请参阅[在 Azure 中的虚拟机上设置终结点](/documentation/articles/virtual-machines-set-up-endpoints)。

### <a id="nsg"></a>来源 4：网络安全组

通过使用网络安全组，可以对允许的入站和出站流量进行更精细的控制。你可以创建跨 Azure 虚拟网络中的子网和云服务的规则。检查你的网络安全组规则，以确保允许来自和去往 Internet 的 SSH 流量。有关详细信息，请参阅[关于网络安全组](/documentation/articles/virtual-networks-nsg)。

### 来源 5：基于 Linux 的 Azure 虚拟机

最后一个可能出现问题的来源是 Azure 虚拟机本身。

![](./media/virtual-machines-troubleshoot-ssh-connections/ssh-tshoot5.png)

如果尚未这样做，请按照[如何为基于 Linux 的虚拟机重置密码或 SSH](/documentation/articles/virtual-machines-linux-use-vmaccess-reset-password-or-ssh) 中的说明，在虚拟机上执行操作。

从你的计算机重试连接。如果不成功，则可能是因为以下一些问题：

- 目标虚拟机上未运行 SSH 服务。
- TCP 端口 22 上未侦听 SSH 服务。若要测试这一点，可在本地计算机上安装一个远程登录客户端，然后运行“telnet *cloudServiceName*.chinacloudapp.cn 22”。这将确定虚拟机是否允许与 SSH 终结点进行入站和出站通信。
- 目标虚拟机上的本地防火墙具有阻止入站或出站 SSH 流量的规则。
- Azure 虚拟机上运行的入侵检测或网络监视软件阻止了 SSH 连接。


## 步骤 4：将你的问题提交到 Azure 支持论坛

若要向全世界的 Azure 专家寻求帮助，你可以将问题提交到 MSDN Azure 论坛或 CSDN 论坛。有关详细信息，请参阅 [Windows Azure 论坛](http://www.windowsazure.cn/support/forums/)。

## 步骤 5：提出 Azure 支持事件

如果你已完成本文中介绍的步骤 1-4，并将问题提交到 Azure 支持论坛，但仍然无法创建 SSH 连接，则要更换另一种方法，考虑是否可以重新创建虚拟机。

如果无法重新创建虚拟机，则可能要提出 Azure 支持事件。

若要提出事件，请转到 [Azure 支持站点](/support/contact/)，然后单击“获取帮助”。

有关使用 Azure 支持的信息，请参阅[有关 Microsoft Azure 支持的常见问题解答](/support/faq/)。

## 其他资源

[如何为基于 Linux 的虚拟机重置密码或 SSH](/documentation/articles/virtual-machines-linux-use-vmaccess-reset-password-or-ssh)

[对与基于 Windows 的 Azure 虚拟机的 Windows 远程桌面连接进行故障排除](/documentation/articles/virtual-machines-troubleshoot-remote-desktop-connections)

[对在 Azure 虚拟机上运行的应用程序的访问进行故障排除](/documentation/articles/virtual-machines-troubleshoot-access-application)

<!---HONumber=76-->