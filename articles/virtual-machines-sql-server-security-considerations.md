<properties 
	pageTitle="Azure 虚拟机中的 SQL Server 的安全注意事项"
	description="提供用于保护在 Azure 虚拟机中运行的 SQL Server 的一般指南。"
	services="virtual-machines"
	documentationCenter="na"
	authors="rothja"
	manager="jeffreyg"
	editor="monicar" />
<tags 
	ms.service="virtual-machines"
	ms.date="08/19/2015"
	wacn.date="09/15/2015" />

# Azure 虚拟机中的 SQL Server 的安全注意事项

本主题包括可帮助建立对 Azure VM 中的 SQL Server 实例的安全访问的总体安全准则。但是，为了确保更好地保护你在 Azure 中的 SQL Server 数据库实例，我们建议你除了实施传统的本地安全做法外，还实施 Azure 的安全最佳做法。

有关 SQL Server 安全做法的详细信息，请参阅 [SQL Server 2008 R2 安全最佳做法 - 操作和管理任务](http://download.microsoft.com/download/1/2/A/12ABE102-4427-4335-B989-5DA579A4D29D/SQL_Server_2008_R2_Security_Best_Practice_Whitepaper.docx)

Azure 遵守多个行业法规和标准，使你能够使用在虚拟机中运行的 SQL Server 构建合规的解决方案。有关 Azure 合规性的信息，请参阅 [Azure 信任中心](/support/trust-center/)。

下面是在配置和连接到 Azure VM 中的 SQL Server 实例时应考虑的安全建议的列表。

## 有关管理帐户的注意事项：

- 创建一个未命名为 **Administrator** 的唯一本地管理员帐户。

- 对所有帐户使用复杂的强密码。有关如何创建强密码的详细信息，请参阅安全中心中的[创建强密码](http://go.microsoft.com/fwlink/?LinkId=293596)一文。

- 默认情况下，Azure 在 SQL Server 虚拟机安装期间选择 Windows 身份验证。因此，禁用 **SA** 登录名，并由安装程序分配密码。我们建议不应使用或启用 **SA** 登录名。以下是在需要 SQL 登录时的备用策略：
	- 创建一个具有 **CONTROL SERVER** 权限的 SQL 帐户。
	- 如果必须使用 **SA** 登录名，请启用该登录名，将其重命名并分配一个新密码。
	- 前面提到的这两个选项都需要将身份验证模式更改为 SQL Server 和 Windows 身份验证模式。有关详细信息，请参阅“更改服务器身份验证模式”。

- 创建一个具有 CONTROL SERVER 权限的 SQL 帐户。

- 如果必须使用 SA 登录名，请启用该登录名，将其重命名并分配一个新密码。

- 前面提到的这两个选项都需要将身份验证模式更改为 **SQL Server 和 Windows 身份验证模式**。有关详细信息，请参阅[更改服务器身份验证模式](https://msdn.microsoft.com/zh-cn/library/ms188670.aspx)。

## 有关保护与 Azure 虚拟机的连接的注意事项：

- 请考虑使用 [Azure 虚拟网络](/documentation/articles/virtual-networks-overview)（而不是公共 RDP 端口）来管理虚拟机。

- 删除虚拟机上的任何终结点（如果不使用它们）。

- 为 Azure 虚拟机中的 SQL Server 数据库引擎实例启用加密连接选项。使用签名证书配置 SQL Server 实例。有关详细信息，请参阅[为数据库引擎启用加密连接](https://msdn.microsoft.com/zh-cn/library/ms191192.aspx)和[连接字符串语法](https://msdn.microsoft.com/zh-cn/library/ms254500.aspx)。

- 如果只应从特定网络访问虚拟机，请使用 Windows 防火墙将访问限制为特定 IP 地址或网络子网。还可以考虑在终结点上添加 ACL 以将流量限制为你允许的客户端。有关在终结点上使用 ACL 的说明，请参阅[管理终结点上的 ACL](/documentation/articles/virtual-machines-set-up-endpoints#manage-the-acl-on-an-endpoint)

## 后续步骤

如果你还对有关性能的最佳实践感兴趣，请参阅 [Azure 虚拟机中 SQL Server 的性能最佳实践](https://msdn.microsoft.com/zh-cn/library/azure/dn133149.aspx)。

有关其他与在 Azure VM 中运行 SQL Server 相关的主题，请参阅 [Azure 虚拟机上的 SQL Server](/documentation/articles/virtual-machines-sql-server-infrastructure-services)。

<!---HONumber=69-->