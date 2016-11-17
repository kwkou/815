<properties
	pageTitle="移动订阅后更改密钥保管库租户 ID | Azure"
	description="了解将订阅移至不同的租户后如何切换密钥保管库的租户 ID"
	services="key-vault"
	documentationCenter=""
	authors="amitbapat"
	manager="mbaldwin"
	tags="azure-resource-manager"/>  


<tags
	ms.service="key-vault"
	ms.workload="identity"
	ms.tgt_pltfrm="na"
	ms.devlang="na"
	ms.topic="hero-article"
	ms.date="09/13/2016"
	ms.author="ambapat"
   	wacn.date="10/19/2016"/>  


# 订阅移动后更改密钥保管库租户 ID
### 问：我的订阅已从租户 A 移动到租户 B。如何更改我的现有密钥保管库的租户 ID，并为租户 B 中的主体设置正确的 ACL？

在订阅中创建新的密钥保管库时，该密钥保管库自动绑定到该订阅的默认 Azure Active Directory 租户 ID。所有访问策略条目也都绑定到此租户 ID。将 Azure 订阅从租户 A 移到租户 B 时，租户 B.中的主体（用户和应用程序）无法访问现有的密钥保管库。若要解决此问题，需要执行以下操作：

- 将此订阅中与所有现有密钥保管库关联的租户 ID 更改为租户 B 的 ID
- 删除所有现有的访问策略条目
- 添加与租户 B 相关联的新访问策略条目

例如，如果订阅中的密钥保管库“myvault”从租户 A 移到租户 B，以下介绍了如何更改此密钥保管库的租户 ID，以及删除旧的访问策略。

<pre>
$vaultResourceId = (Get-AzureRmKeyVault -VaultName myvault).ResourceId
$vault = Get-AzureRmResource -ResourceId $vaultResourceId -ExpandProperties
$vault.Properties.TenantId = (Get-AzureRmContext).Tenant.TenantId
$vault.Properties.AccessPolicies = @()
Set-AzureRmResource -ResourceId $vaultResourceId -Properties $vault.Properties
</pre>

因为移动前此保管库位于租户 A，所以 **$vault.Properties.TenantId** 的原始值为租户 A，而 **(Get-AzureRmContext).Tenant.TenantId** 的值为租户 B。

现在，密钥保管库已经和正确的租户 ID 相关联，并且已删除旧的访问策略条目，那么请使用 [Set-AzureRmKeyVaultAccessPolicy](https://msdn.microsoft.com/zh-cn/library/mt603625.aspx) 设置新的访问策略条目。

## 后续步骤

- 如果有关于密钥保管库的问题，请访问 [Azure 密钥保管库论坛](https://social.msdn.microsoft.com/Forums/zh-cn/home?forum=AzureKeyVault)

<!---HONumber=Mooncake_1010_2016-->
