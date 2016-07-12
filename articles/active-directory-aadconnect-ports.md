<properties
	pageTitle="Azure AD Connect：端口 | Azure"
	description="此技术参考页面描述了需要为 Azure AD Connect 打开的端口"
	services="active-directory"
	documentationCenter=""
	authors="billmath"
	manager="stevenpo"
	editor="curtand"/>

<tags
	ms.service="active-directory"
	ms.date="05/12/2016"
	wacn.date="06/14/2016"/>

# 混合标识所需的端口和协议

以下技术参考文档提供实现混合标识解决方案所需的端口和协议的信息。请使用下图并参考相应的表格。

![什么是 Azure AD Connect](./media/active-directory-aadconnect-ports/required1.png)


## 表 1 - Azure AD Connect 和本地 AD
此表描述了 Azure AD Connect 服务器与本地 AD 之间通信所需的端口和协议。

| 协议 |端口 |说明
| --------- | --------- |--------- |
| DNS|53 (TCP/UDP)| 在目标林中进行 DNS 查找。
|Kerberos|88 (TCP/UDP)| 对 AD 林进行 Kerberos 身份验证。
|MS-RPC |135 (TCP/UDP)| 该端口绑定到 AD 林后，将在初始配置 Azure AD Connect 向导期间使用。
|LDAP|389 (TCP/UDP)|用于从 AD 导入数据。数据将使用 Kerberos 签名和签章加密。
|LDAP/SSL|636 (TCP/UDP)|用于从 AD 导入数据。数据传输经过签名和加密。仅当你使用 SSL 时才使用该端口。
|RPC |1024-65353（随机高 RPC 端口）(TCP/UDP)|该端口绑定到 AD 林后，将在初始配置 Azure AD Connect 期间使用。

## 表 2 - Azure AD Connect 和 Azure AD
此表描述了 Azure AD Connect 服务器与 Azure AD 之间通信所需的端口和协议。

| 协议 |端口 |说明
| --------- | --------- |--------- |
| HTTP|80 (TCP/UDP)|用于下载 CRL（证书吊销列表）以验证 SSL 证书。
|HTTPS|443 (TCP/UDP)|用来与 Azure AD 同步。

有关 Office 365 端口和 IP 地址的列表，请参阅 [Office 365 URL 和 IP 地址范围](https://support.office.com/article/Office-365-URLs-and-IP-address-ranges-8548a211-3fe7-47cb-abb1-355ea5aa88a2)。

## 表 3 - Azure AD Connect 和联合服务器/WAP
此表描述了 Azure AD Connect 服务器与 联合服务器/WAP 服务器之间通信所需的端口和协议。

| 协议 |端口 |说明
| --------- | --------- |--------- |
| HTTP|80 (TCP/UDP)|用于下载 CRL（证书吊销列表）以验证 SSL 证书。
|HTTPS|443 (TCP/UDP)|用来与 Azure AD 同步。
|WinRM|5985| WinRM 侦听器

## 表 4 - WAP 和联合服务器
此表描述了联合服务器与 WAP 服务器之间通信所需的端口和协议。

| 协议 |端口 |说明
| --------- | --------- |--------- |
|HTTPS|443 (TCP/UDP)|用于身份验证。

## 表 5 - WAP 和用户
此表描述了用户与 WAP 服务器之间通信所需的端口和协议。

| 协议 |端口 |说明
| --------- | --------- |--------- |
|HTTPS|443 (TCP/UDP)|用于设备身份验证。
|TCP|49443 (TCP)|用于证书身份验证。


## 表 6a 和 6b - 适用于 (AD FS/Sync) 和 Azure AD 的 Azure AD Connect Health 代理
下表描述了在 Azure AD Connect Health 代理与 Azure AD 之间通信所需的终结点、端口和协议

### 表 6a - 适用于 (AD FS/Sync) 和 Azure AD 的 Azure AD Connect Health 代理的端口和协议
此表描述了 Azure AD Connect Health 代理与 Azure AD 之间通信所需的以下出站端口和协议。

| 协议 |端口 |说明
| --------- | --------- |--------- |
|HTTPS|443 (TCP/UDP)| 出站
|Azure 服务总线|5671 (TCP/UDP)| 出站

### 6b - 适用于 (AD FS/Sync) 和 Azure AD 的 Azure AD Connect Health 代理的终结点
有关终结点的列表，请参阅 [Azure AD Connect Health 代理的要求部分](/documentation/articles/active-directory-aadconnect-health/#requirements)

<!---HONumber=Mooncake_0606_2016-->