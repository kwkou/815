<properties
    pageTitle="使用 Azure 基于角色的访问控制管理备份 | Microsoft 文档"
    description="使用基于角色的访问控制来管理对恢复服务保管库中的备份管理操作的访问。"
    services="backup"
    documentationcenter=""
    author="trinadhk"
    manager="shreeshd"
    editor=""
    translationtype="Human Translation" />
<tags
    ms.assetid="3bd46b97-4b29-47a5-b5ac-ac174dd36760"
    ms.service="backup"
    ms.workload="storage-backup-recovery"
    ms.tgt_pltfrm="na"
    ms.devlang="na"
    ms.topic="article"
    ms.date="2/10/2017"
    wacn.date="04/24/2017"
    ms.author="trinadhk;markgal"
    ms.sourcegitcommit="a114d832e9c5320e9a109c9020fcaa2f2fdd43a9"
    ms.openlocfilehash="c895ca9197f4027ee3a5dc0c114b71579ac4ef41"
    ms.lasthandoff="04/14/2017" />

# <a name="use-role-based-access-control-to-manage-azure-backup-recovery-points"></a>使用基于角色的访问控制管理 Azure 备份恢复点
Azure 基于角色的访问控制 (RBAC) 可用于对 Azure 进行细致的访问管理。 使用 RBAC，你可以在团队中对职责进行分配，仅向用户授予执行作业所需的访问权限。

> [AZURE.IMPORTANT]
> Azure 备份提供的角色仅限于可在 Azure 门户或恢复服务保管库 PowerShell cmdlet 中执行的操作。 这些角色对在 Azure 备份代理客户端 UI、System Center Data Protection Manager UI 或 Azure 备份服务器 UI 中执行的操作不具有控制权。

Azure 备份提供 3 个用于控制备份管理操作的内置角色。 详细了解 [Azure RBAC 内置角色](/documentation/articles/role-based-access-built-in-roles/)

- [备份参与者](/documentation/articles/role-based-access-built-in-roles/#backup-contributor/) - 此角色具有创建和管理备份方面的所有权限，除了创建恢复服务保管库和授予他人访问权限。 可以把该角色想象成可执行每个备份管理操作的备份管理的管理员。
- [备份操作员](/documentation/articles/role-based-access-built-in-roles/#backup-operator/) - 此角色具有除删除备份和管理备份策略之外的针对参与者操作的所有权限。 此角色等效于参与者，但它不能执行破坏性操作，例如通过删除数据或删除本地资源的注册来停止备份。
- [备份读取器](/documentation/articles/role-based-access-built-in-roles/#backup-reader/) - 此角色具有查看所有备份管理操作的权限。 可以把该角色想象成一位监视者。

若要定义自己的角色以便进一步控制，请参阅如何在 Azure RBAC 中 [生成自定义角色](/documentation/articles/role-based-access-control-custom-roles/)。



## <a name="mapping-backup-built-in-roles-to-backup-management-actions"></a>将备份内置角色映射到备份管理操作
下表包含备份管理操作和执行这些操作所需的最低 RBAC 角色。

| 管理操作 | 所需的最低 RBAC 角色 |
| --- | --- |
| 创建恢复服务保管库 | 保管库资源组上的参与者 |
| 启用 Azure VM 备份 | 保管库上的备份操作员，VM 上的虚拟机参与者 |
| 按需备份 VM | 备份操作员 |
| 还原 VM | 备份操作员，将在其中部署 VM 和 Vnet 的|资源组参与者 |
| 从 VM 备份还原磁盘及单个文件 | 备份操作员 |
| 创建 Azure VM 备份的备份策略 | 备份参与者 |
| 修改 Azure VM 备份的备份策略 | 备份参与者 |
| 删除 Azure VM 备份的备份策略 | 备份参与者 |
| 停止在 VM 备份上备份（通过保留数据或删除数据） | 备份参与者 |
| 注册本地 Windows Server/客户端/SCDPM 或 Azure 备份服务器 | 备份操作员 |
| 删除已注册的本地 Windows Server/客户端/SCDPM 或 Azure 备份服务器 | 备份参与者 |

## <a name="next-steps"></a>后续步骤
- [基于角色的访问控制](/documentation/articles/role-based-access-control-configure/)：Azure 门户中的 RBAC 入门。
- 了解如何通过以下方式管理访问权限：
  - [PowerShell](/documentation/articles/role-based-access-control-manage-access-powershell/)
  - [Azure CLI](/documentation/articles/role-based-access-control-manage-access-azure-cli/)
  - [REST API](/documentation/articles/role-based-access-control-manage-access-rest/)
- [基于角色的访问控制故障排除](/documentation/articles/role-based-access-control-troubleshooting/)：获取解决常见问题的建议。

<!---Update_Description: wording update -->