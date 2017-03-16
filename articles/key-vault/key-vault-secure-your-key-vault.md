<properties
    pageTitle="保护密钥保管库 | Azure"
    description="管理密钥保管库的访问权限，以便对保管库、密钥和机密进行管理。密钥保管库的身份验证和授权模型，以及如何保护密钥保管库"
    services="key-vault"
    documentationcenter=""
    author="amitbapat"
    manager="mbaldwin"
    tags="azure-resource-manager" />
<tags
    ms.assetid="e5b4e083-4a39-4410-8e3a-2832ad6db405"
    ms.service="key-vault"
    ms.workload="identity"
    ms.tgt_pltfrm="na"
    ms.devlang="na"
    ms.topic="hero-article"
    ms.date="01/07/2017"
    wacn.date="03/16/2017"
    ms.author="ambapat" />  


# 保护密钥保管库

Azure 密钥保管库是保护云应用程序加密密钥和机密（例如证书、连接字符串与密码）的云服务。由于这些数据极其机密且对企业至关重要，因此用户希望保护对密钥保管库的访问，以便只有经过授权的应用程序和用户才能访问密钥保管库。本文概述密钥保管库访问模型，介绍身份验证和授权，并举例说明如何保护对云应用程序密钥保管库的访问。

## 概述

可通过以下两个独立接口来控制对密钥保管库的访问：管理平面和数据平面。在调用方（用户或应用程序）获取密钥保管库的访问权限之前，需要在这两个平面上经过适当的身份验证和授权。身份验证建立调用方的标识，授权确定允许调用方执行哪些操作。

管理平面和数据平面都使用 Azure Active Directory 进行身份验证。管理平面使用基于角色的访问控制 \(RBAC\) 进行授权，数据平面使用密钥保管库访问策略进行授权。

下面是相关主题的简要概述：

[使用 Azure Active Directory 进行身份验证](#authentication-using-azure-active-direcrory) - 此部分介绍调用方如何通过管理平面和数据平面使用 Azure Active Directory 进行身份验证，以便访问密钥保管库。

[管理平面和数据平面](#management-plane-and-data-plane) - 管理平面和数据平面是用于访问密钥保管库的两个访问平面。每个访问平面支持特定的操作。此部分介绍访问终结点、支持的操作以及每个平面使用的访问控制方法。

[管理平面访问控制](#management-plane-access-control) - 此部分介绍如何使用基于角色的访问控制来访问管理平面操作。

[数据平面访问控制](#data-plane-access-control) - 此部分介绍如何使用密钥保管库访问策略来控制数据平面访问。

[示例](#example) - 此示例介绍如何为密钥保管库设置访问控制，使三个不同的团队（安全团队、开发人员/操作人员和审核人员）能够执行特定的任务，在 Azure 中开发、管理和监视应用程序。

## 使用 Azure Active Directory 进行身份验证 <a name="authentication-using-azure-active-direcrory"></a>
在 Azure 订阅中创建密钥保管库时，该密钥保管库会自动关联到该订阅的 Azure Active Directory 租户。所有调用方（用户和应用程序）必须在此租户中注册才能访问此密钥保管库。应用程序或用户必须使用 Azure Active Directory 进行身份验证才能访问密钥保管库。这一点适用于管理平面访问和数据平面访问。在这两种情况下，应用程序可通过两种方式访问密钥保管库：

- **用户+ 应用访问** - 通常适用于代表登录用户访问密钥保管库的应用程序。Azure PowerShell 和 Azure 门户预览就是这种访问类型的例子。可使用两种方法向用户授予访问权限：一种方法是授予用户从任何应用程序访问密钥保管库的权限，另一种方法是仅当用户使用特定应用程序时才向其授予密钥保管库访问权限（称为复合标识）。
- **仅限应用的访问** - 适用于运行后台程序服务、后台作业等的应用程序。将向应用程序的标识授予密钥保管库访问权限。

这两种类型的应用程序使用任何[受支持的身份验证方法](/documentation/articles/active-directory-authentication-scenarios/)在 Azure Active Directory 中进行身份验证，并获取令牌。使用的身份验证方法取决于应用程序类型。然后，应用程序使用此令牌并将 REST API 请求发送到密钥保管库。在管理平面访问模式中，请求将通过 Azure资源管理器终结点路由。访问数据平面时，应用程序直接与密钥保管库终结点对话。查看有关[整个身份验证流](/documentation/articles/active-directory-protocols-oauth-code/)的详细信息。

应用程序请求其令牌的资源名称会有所不同，具体取决于应用程序访问的是管理平面还是数据平面。因此，根据具体的 Azure 环境，资源名称是本部分稍后提供的表格中所述的管理平面或数据平面终结点。

使用单个机制在管理平面和数据平面上进行身份验证机制具有自身的优势：

- 组织可以集中控制对内部所有密钥保管库的访问
- 离职的用户会立即失去对组织中所有密钥保管库的访问权限
- 组织可以通过 Azure Active Directory 中的选项自定义身份验证（例如，启用多重身份验证来提高安全性）

## 管理平面和数据平面 <a name="management-plane-and-data-plane"></a>
Azure 密钥保管库是可以在 Azure资源管理器部署模型中使用的 Azure 服务。创建密钥保管库时，将会获得一个虚拟容器，可在其中创建密钥、机密和证书等其他对象。然后，可以使用管理平面和数据平面访问密钥保管库来执行特定的操作。管理平面接口用于管理密钥保管库本身，例如，创建、删除、更新密钥保管库属性，以及设置数据平面的访问策略。数据平面接口用于在密钥保管库中添加，以及删除、修改和使用其中存储的密钥、机密和证书。

可通过不同的终结点访问管理平面和数据平面接口（参阅下表）。表格中的第二列描述这些终结点在不同 Azure 环境中的 DNS 名称。第三列描述可在每个访问平面中执行的操作。此外，每个访问平面具有自身的访问控制机制：对于管理平面，访问控制是使用 Azure资源管理器基于角色的访问控制 \(RBAC\) 设置的；对于数据平面，访问控制是使用密钥保管库访问策略设置的。

| 访问平面 | 访问终结点 | 操作 | 访问控制机制 |
| --- | --- | --- | --- |
| 管理平面 |**全球：**<br>management.azure.com:443<br><br> **Azure China：**<br>management.chinacloudapi.cn:443<br><br> **Azure US Government：**<br>management.usgovcloudapi.net:443<br><br> **Azure Germany：**<br>management.microsoftazure.de:443 |创建/读取/更新/删除密钥保管库<br>设置密钥保管库的访问策略<br>设置密钥保管库的标记 |Azure资源管理器基于角色的访问控制 \(RBAC\) |
| 数据平面 |**全球：**<br> &lt;vault-name&gt;.vault.chinacloudapi.cn:443<br><br> **Azure China：**<br> &lt;vault-name&gt;.vault.azure.cn:443<br><br> **Azure US Government：**<br> &lt;vault-name&gt;.vault.usgovcloudapi.net:443<br><br> **Azure Germany：**<br> &lt;vault-name&gt;.vault.microsoftazure.de:443 |密钥：解密、加密、解包密钥、包装密钥、验证、签名、获取、列出、更新、创建、导入、删除、备份、还原<br><br>机密：获取、列出、设置、删除 |密钥保管库访问策略 |

管理平面访问控制与数据平面访问控制相互独立。例如，如果想要授权应用程序使用密钥保管库中的密钥，只需使用密钥保管库访问策略授予数据平面访问权限，而不需要向此应用程序授予管理平面访问权限。相反，如果希望用户有权读取保管库属性和标记但无权访问密钥、机密或证书，则可以使用 RBAC 向此用户授予“读取”访问权限，而不需要授予数据平面访问权限。

## 管理平面访问控制 <a name="management-plane-access-control"></a>
管理平面由影响密钥保管库本身的操作构成。例如，你可以创建或删除密钥保管库。可以获取订阅中保管库的列表。可以检索密钥保管库属性（例如 SKU 和标记），设置密钥保管库访问策略来控制有权访问密钥保管库中密钥和机密的用户与应用程序。管理平面访问控制使用 RBAC。请在前一部分的表格中查看可通过管理平面执行的密钥保管库操作的完整列表。

### 基于角色的访问控制 \(RBAC\)
每个 Azure 订阅都有一个 Azure Active Directory。可为此目录中的用户、组和应用程序授予访问权限，以便在使用 Azure资源管理器部署模型的 Azure 订阅中管理资源。这种类型的访问控制称为基于角色的访问控制 \(RBAC\)。若要管理此访问权限，可以使用 [Azure 门户预览](https://portal.azure.cn/)、[Azure CLI 工具](/documentation/articles/xplat-cli-install/)、[PowerShell](/documentation/articles/powershell-install-configure/) 或 [Azure资源管理器REST API](https://msdn.microsoft.com/zh-cn/library/azure/dn906885.aspx)。

使用 Azure资源管理器模型时，可以在资源组中创建密钥保管库，使用 Azure Active Directory 来控制对此密钥保管库的管理平面的访问。例如，可以授予用户或组管理特定资源组中密钥保管库的权限。

可以通过分配相应的 RBAC 角色，在特定范围内向用户、组和应用程序授予访问权限。例如，若要向某个用户授予管理密钥保管库的权限，可在特定的范围内向此用户分配预定义的角色“密钥保管库参与者”。在本例中，范围是订阅、资源组或特定的密钥保管库。在订阅级别分配的角色适用于该订阅中的所有资源组和资源。在资源组级别分配的角色适用于该资源组中的所有资源。为特定资源分配的角色仅适用于该资源。有几个预定义的角色（请参阅 [RBAC：内置角色](/documentation/articles/role-based-access-built-in-roles/)）。如果预定义的角色不符合你的需要，你也可以定义自己的角色。

> [AZURE.IMPORTANT]
请注意，如果某个用户对密钥保管库管理平面拥有“参与者”权限 \(RBAC\)，该用户可以通过设置密钥保管库访问策略（用于控制对数据平面的访问），向自己授予数据平面访问权限。因此，建议严格控制谁对密钥保管库拥有“参与者”访问权限，确保只有授权的人员可以访问和管理你的密钥保管库、密钥、机密和证书。
> 
> 

## 数据平面访问控制 <a name="data-plane-access-control"></a>
密钥保管库数据平面由影响密钥保管库中的对象（例如密钥、机密和证书）的操作组成。这包括创建、导入、更新、列出、备份和还原密钥等密钥操作，签名、验证、加密、解密、包装和解包等加密操作，以及设置密钥的标记和其他属性。同样，还包括获取、设置、列出和删除等有关机密的操作。

可通过设置密钥保管库的访问策略来授予数据平面访问权限。用户、组或应用程序必须对密钥保管库的管理平面拥有“参与者”权限 \(RBAC\)，才能设置该密钥保管库的访问策略。可以向用户、组或应用程序授予针对密钥保管库中的密钥或机密执行特定操作的访问权限。每个密钥保管库最多支持 16 个访问策略项。创建一个 Azure Active Directory 安全组并将用户添加到该组，可向多个用户授予对密钥保管库的数据平面访问权限。

### 密钥保管库访问策略
密钥保管库访问策略单独授予对密钥、机密和证书的权限。例如，可以向用户授予仅限密钥的访问权限，而不授予对机密的权限。但是，对访问密钥、机密或证书的权限是在保管库级别分配的。换而言之，密钥保管库访问策略不支持对象级权限。可以使用 [Azure 门户预览](https://portal.azure.cn/)、[Azure CLI 工具](/documentation/articles/xplat-cli-install/)、[PowerShell](/documentation/articles/powershell-install-configure/) 或[密钥保管库管理 REST API](https://msdn.microsoft.com/zh-cn/library/azure/mt620024.aspx) 设置密钥保管库的访问策略。

> [AZURE.IMPORTANT]
请注意，密钥保管库访问策略在保管库级别应用。例如，授予某个用户创建和删除密钥的权限时，该用户可以针对该密钥保管库中的所有密钥执行这些操作。
> 
> 

## 示例 <a name="example"></a>
假设开发中的某个应用程序使用证书进行 SSL 访问，使用 Azure 存储来存储数据，使用 RSA 2048 位密钥执行签名操作。另外，此应用程序在 VM（或 VM 规模集）中运行。在这种情况下，可以使用密钥保管库来存储所有应用程序机密，使用密钥保管库来存储应用程序在 Azure Active Directory 中进行身份验证所用的启动证书。

下面是密钥保管库中存储的所有密钥和机密的摘要。

- **SSL 证书** - 用于 SSL
- **存储密钥** - 用于获取对存储帐户的访问权限
- **RSA 2048 位密钥** - 用于签名操作
- **启动证书** - 用于在 Azure Active Directory 中进行身份验证，获取对密钥保管库的访问权限，以便提取存储密钥和使用 RSA 密钥进行签名。

现在，让我们与管理、部署和审核此应用程序的人员会面。本示例使用了三个角色。

- **安全团队** - 通常是来自“CSO（首席安全官）办公室”或同等级别的 IT 人员，他们负责妥善保管机密，例如 SSL 证书、用于签名的 RSA 密钥、数据库的连接字符串和存储帐户密钥。
- **开发人员/操作人员** - 开发此应用程序，然后将其部署到 Azure 的人员。通常，他们不是安全团队的成员，因此不应有权访问 SSL 证书、RSA 密钥等任何敏感数据，但他们部署的应用程序应有权访问这些数据。
- **审核人员** - 通常是独立于开发人员和普通 IT 人员的一组不同的人员。他们的职责是评审证书、密钥等是否得到适当的使用和维护，确保遵从数据安全标准。

另外，有一个角色超出了此应用程序的范围，但此处有必要提到它，那就是订阅（或资源组）的管理员。订阅管理员设置安全团队的初始访问权限。在本示例中，我们假设订阅管理员已向安全团队授予此应用程序所需的全部资源所在的资源组的访问权限。

现在，我们看看在此应用程序的上下文中，每个角色可执行哪些操作。

- **安全团队**
  - 创建密钥保管库
  - 启用密钥保管库日志记录
  - 添加密钥/机密
  - 为灾难恢复创建密钥备份
  - 设置密钥保管库访问策略，向用户和应用程序授予执行特定操作的权限
  - 定期滚动更新密钥/机密
- **开发人员/操作人员**
  - 从安全团队获取对启动证书和 SSL 证书（指纹）、存储密钥（机密 URI）和签名密钥（密钥 URI）的引用
  - 开发和部署以编程方式访问密钥与机密的应用程序
- **审核人员**
  - 审查使用情况日志，确认密钥/机密是否得到适当的使用，以及是否遵从数据安全标准

现在，我们看看每个角色（和应用程序）在执行其分配的任务时需要对密钥保管库拥有哪些访问权限。

| 用户角色 | 管理平面权限 | 数据平面权限 |
| --- | --- | --- |
| 安全团队 |密钥保管库参与者 |密钥：备份、创建、删除、获取、导入、列出、还原<br>机密：所有 |
| 开发人员/操作人员 |需要密钥保管库部署权限，使部署的 VM 可以从密钥保管库中提取机密 |无 |
| 审核人员 |无 |密钥：列出<br>机密：列出 |
| 应用程序 |无 |密钥：签名<br>机密：获取 |

> [AZURE.NOTE]
审核员需要对密钥和机密拥有列出权限，以便能够检查日志中未发出的密钥和机密的属性，例如标记、激活日期和过期日期。
> 
> 

除了对密钥保管库的权限以外，所有三个角色还需要有权访问其他资源。例如，能够部署 VM（或 Web 应用等）。 开发人员/操作人员还需要对这些资源类型拥有“参与者”访问权限。审核人员需要对密钥保管库日志所存储到的存储帐户拥有读取访问权限。

本文的重点是如何保护对密钥保管库的访问，因此，只阐述了与此主题相关的内容，略过了有关部署证书和以编程方式访问密钥和机密的详细信息。其他文章已介绍这些详细信息。[此博客文章](https://blogs.technet.microsoft.com/kv/2016/09/14/updated-deploy-certificates-to-vms-from-customer-managed-key-vault/)介绍了如何将密钥保管库中存储的证书部署到 VM，[此示例代码](https://www.microsoft.com/en-us/download/details.aspx?id=45343)演示了如何使用启动证书在 Azure AD 中进行身份验证，以获取对密钥保管库的访问权限。

可以使用 Azure 门户预览授予大部分访问权限，但若要授予细粒度的权限，可能要使用 Azure PowerShell（或 Azure CLI）才能实现所需的结果。

以下 PowerShell 代码片段假设：

- Azure Active Directory 管理员创建了代表三个角色（即 Contoso 安全团队，Contoso 应用开发人员、Contoso 应用审核人员）的安全组。管理员还将用户添加到了他们所属的组。
- **ContosoAppRG** 是所有资源所在的资源组。**contosologstorage** 是日志的存储位置。
- 密钥保管库 **ContosoKeyVault** 以及密钥保管库日志所用的存储帐户 **contosologstorage** 必须位于同一个 Azure 位置

首先，订阅管理员向安全小组分配“密钥保管库参与者”和“用户访问管理员”角色。这样，安全团队便可以管理对其他资源的访问权限，以及管理资源组 ContosoAppRG 中的密钥保管库。


	New-AzureRmRoleAssignment -ObjectId (Get-AzureRmADGroup -SearchString 'Contoso Security Team')[0].Id -RoleDefinitionName "key vault Contributor" -ResourceGroupName ContosoAppRG
	New-AzureRmRoleAssignment -ObjectId (Get-AzureRmADGroup -SearchString 'Contoso Security Team')[0].Id -RoleDefinitionName "User Access Administrator" -ResourceGroupName ContosoAppRG


以下脚本演示安全团队如何创建密钥保管库、设置日志记录，以及设置对其他角色和应用程序的访问权限。


	# Create key vault and enable logging
	$sa = Get-AzureRmStorageAccount -ResourceGroup ContosoAppRG -Name contosologstorage
	$kv = New-AzureRmKeyVault -VaultName ContosoKeyVault -ResourceGroup ContosoAppRG -SKU premium -Location 'ChinaNorth' -EnabledForDeployment
	Set-AzureRmDiagnosticSetting -ResourceId $kv.ResourceId -StorageAccountId $sa.Id -Enabled $true -Categories AuditEvent

	# Data plane permissions for Security team
	Set-AzureRmKeyVaultAccessPolicy -VaultName ContosoKeyVault -ObjectId (Get-AzureRmADGroup -SearchString 'Contoso Security Team')[0].Id -PermissionToKeys backup,create,delete,get,import,list,restore -PermissionToSecrets all

	# Management plane permissions for Dev/ops
	# Create a new role from an existing role
	$devopsrole = Get-AzureRmRoleDefinition -Name "Virtual Machine Contributor"
	$devopsrole.Id = $null
	$devopsrole.Name = "Contoso App Devops"
	$devopsrole.Description = "Can deploy VMs that need secrets from key vault"
	$devlopsrole.AssignableScopes = @("/subscriptions/<SUBSCRIPTION-GUID>")

	# Add permission for dev/ops so they can deploy VMs that have secrets deployed from key vaults
	$devopsrole.Actions.Add("Microsoft.KeyVault/vaults/deploy/action")
	New-AzureRmRoleDefinition -Role $role

	# Assign this newly defined role to Dev ops security group
	New-AzureRmRoleAssignment -ObjectId (Get-AzureRmADGroup -SearchString 'Contoso App Devops')[0].Id -RoleDefinitionName "Contoso App Devops" -ResourceGroupName ContosoAppRG

	# Data plane permissions for Auditors
	Set-AzureRmKeyVaultAccessPolicy -VaultName ContosoKeyVault -ObjectId (Get-AzureRmADGroup -SearchString 'Contoso App Auditors')[0].Id -PermissionToKeys list -PermissionToSecrets list


自定义的角色只能分配给创建 ContosoAppRG 资源组所在的订阅。如果要为其他订阅中的其他项目使用相同的自定义角色，应该在角色范围中添加更多的订阅。

为拥有“部署/操作”权限的开发人员/操作人员分配的自定义角色将划归到资源组。这样，只有在资源组“ContosoAppRG”中创建的 VM 才能获取机密（SSL 证书和启动证书）。开发/操作团队成员在其他资源组中创建的任何 VM 都无法获取这些机密，即使它们知道机密 URI。

本示例描绘了一个简单方案。现实的方案可能更复杂，需要你根据需求调整对密钥保管库的权限。例如，在本示例中，假设安全团队会向开发人员/操作人员团队提供他们需要在应用程序中使用的密钥和机密引用（URI 和指纹）。因此，安全团队不需要向开发人员/操作人员授予任何数据平面访问权限。另外，请注意，本示例重点演示如何保护密钥保管库。保护 [VM](/home/features/virtual-machines/)、[存储帐户](/documentation/articles/storage-security-guide/)和其他 Azure 资源时，也应该考虑类似的因素。

> [AZURE.NOTE]
注意：本示例演示如何在生产环境中锁定密钥保管库访问权限。开发人员应该获取自己的、拥有完全权限的订阅或资源组，以便能够管理用于开发应用程序的保管库、VM 和存储帐户。
> 
> 

## 资源
  
  此文解释了 Azure Active Directory 基于角色的访问控制及其工作原理。
- [RBAC: Built in Roles（RBAC：内置角色）](/documentation/articles/role-based-access-built-in-roles/)
  
  此文详细说明了 RBAC 中所有可用内置角色。
- [了解资源管理器部署和经典部署](/documentation/articles/resource-manager-deployment-model/)
  
  此文介绍了资源管理器部署和经典部署模型，并说明使用资源管理器和资源组的优点。
- [使用 Azure PowerShell 管理基于角色的访问控制](/documentation/articles/role-based-access-control-manage-access-powershell/)
  
  此文介绍如何使用 Azure PowerShell 管理基于角色的访问控制。
- [Managing Role-Based Access Control with the REST API（使用 REST API 管理基于角色的访问控制）](/documentation/articles/role-based-access-control-manage-access-rest/)
  
  此文说明如何使用 REST API 来管理 RBAC。
- [使用 OAuth 2.0 和 Azure Active Directory 来授权访问 Web 应用程序](/documentation/articles/active-directory-protocols-oauth-code/)
  
  此文介绍使用 Azure Active Directory 进行身份验证时遵循的整个 OAuth 2.0 流程。
- [key vault Management REST APIs（密钥保管库管理 REST API）](https://msdn.microsoft.com/zh-cn/library/azure/mt620024.aspx)
  
  此参考文档介绍用于以编程方式管理密钥保管库的 REST API，内容包括如何设置密钥保管库访问策略。
- [key vault REST APIs（密钥保管库 REST API）](https://msdn.microsoft.com/zh-cn/library/azure/dn903609.aspx)
  
  密钥保管库 REST API 参考文档的链接。
- [Key access control（密钥访问控制）](https://msdn.microsoft.com/zh-cn/library/azure/dn903623.aspx#BKMK_KeyAccessControl)
  
  机密访问控制参考文档的链接。
- [Secret access control（机密访问控制）](https://msdn.microsoft.com/zh-cn/library/azure/dn903623.aspx#BKMK_SecretAccessControl)
  
  密钥访问控制参考文档的链接。
- 使用 PowerShell [设置](https://msdn.microsoft.com/zh-cn/library/mt603625.aspx)和[删除](https://msdn.microsoft.com/zh-cn/library/mt619427.aspx)密钥保管库访问策略
  
  有关用于管理密钥保管库访问策略的 PowerShell cmdlet 的参考文档链接。

## 后续步骤

有关面向管理员的入门教程，请参阅 [Azure 密钥保管库入门](/documentation/articles/key-vault-get-started/)。

有关将密钥和机密与 Azure 密钥保管库配合使用的详细信息，请参阅[关于密钥和机密](https://msdn.microsoft.com/zh-cn/library/azure/dn903623.aspx)。

如果在密钥保管库方面有任何问题，请访问 [Azure 密钥保管库论坛](https://social.msdn.microsoft.com/Forums/zh-cn/home?forum=AzureKeyVault)

<!---HONumber=Mooncake_0220_2017-->
<!--Update_Description: wording update-->