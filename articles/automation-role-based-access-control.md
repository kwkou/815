<properties 
   pageTitle="Azure 自动化中基于角色的访问控制 | Azure"
   description="基于角色的访问控制 (RBAC) 可用于对 Azure 资源进行访问管理。本文介绍如何设置 Azure 自动化中的 RBAC。"
   services="automation"
   documentationCenter=""
   authors="SnehaGunda"
   manager="stevenka"
   editor="tysonn" />
<tags 
   ms.service="automation"
   ms.date="04/04/2016"
   wacn.date="05/13/2016"/>

# Azure 自动化中基于角色的访问控制

## 基于角色的访问控制

基于角色的访问控制 (RBAC) 可用于对 Azure 资源进行访问管理。使用 RBAC，你可以在团队中对职责进行分配，仅授予执行作业所需的对用户、组和应用程序的适当访问权限。可以使用 Azure 门户、Azure 命令行工具或 Azure 管理 API 将基于角色的访问权限授予用户。

## 自动化帐户中的 RBAC

在 Azure 自动化中，访问权限是通过将相应的 RBAC 角色分配给自动化帐户作用域的用户、组和应用程序来授予的。以下是自动化帐户所支持的内置角色：

|**角色** | **说明** |
|:--- |:---|
| 所有者 | 所有者角色允许访问自动化帐户中的所有资源和操作，包括访问其他用户、组和应用程序以管理自动化帐户。 |
| 参与者 | 参与者角色允许你管理所有事项，修改其他用户对自动化帐户的访问权限除外。 |
| 读取器 | 读者角色允许你查看自动化帐户中的所有资源，但不能进行任何更改。 |
| 自动化操作员 | 自动化操作员角色允许你执行作业的启动、停止、暂停、恢复和计划等操作任务。如果你想要防止他人查看或修改你的自动化帐户资源（例如凭据资产和 Runbook），但仍允许所在组织的成员执行这些 Runbook，则可使用此角色。“自动化操作员操作”列出了自动化操作员角色所支持的针对自动化帐户及其资源的操作。 |
| 用户访问管理员 | 用户访问管理员角色允许你管理用户对 Azure 自动化帐户的访问。 |

在本文中，我们将指导你在 Azure 自动化中设置 RBAC。

## 使用 Azure 门户为自动化帐户配置 RBAC

1.	登录到 [Azure 门户](https://manage.windowsazure.cn)，然后从“自动化帐户”边栏选项卡中打开你的自动化帐户。  

2.	单击右上角的“访问”控件。此时会打开“用户”边栏选项卡，你可以在其中添加新的用户、组和应用程序，以便管理你的自动化帐户并查看现有的可以针对自动化帐户进行配置的角色。

>[AZURE.NOTE]  **订阅管理员**已作为默认用户存在。订阅管理员 Active Directory 组包括 Azure 订阅的服务管理员和共同管理员。服务管理员是 Azure 订阅及其资源的所有者，并且还会让自动化帐户的所有者角色继承下去。这意味着，访问权限是**继承的**（针对订阅的**服务管理员和共同管理员**而言），并且是针对所有其他用户**分配的**。单击“订阅管理员”，以便查看有关其权限的更多详细信息。

### 添加新用户并分配角色

1.	在“用户”边栏选项卡中，单击“添加”打开“添加访问权限”边栏选项卡，以便添加用户、组或应用程序并向其分配角色。  



2.	从可用角色列表中选择一个角色。我们将选择“读者”角色，但你可以选择自动化帐户所支持的任何可用的内置角色，或者你所定义的任何自定义角色。



3.	单击“添加用户”以打开“添加用户”边栏选项卡。如果你已经添加过管理订阅的用户、组或应用程序，系统会列出这些用户，你可以选择他们来添加访问权限。如果没有列出任何用户，或者没有列出你想要添加的用户，则请单击“邀请”打开“邀请来宾”边栏选项卡，以便邀请具有有效 Microsoft 帐户电子邮件地址（例如 Outlook.com、OneDrive 或 Xbox Live ID）的用户。输入用户的电子邮件地址以后，请单击“选择”以添加用户，然后单击“确定”。


 
现在，你会看到添加到“用户”边栏选项卡且被分配了“读者”角色的用户。



你也可以通过“角色”边栏选项卡向用户分配角色。单击“用户”边栏选项卡中的“角色”以打开“角色”边栏选项卡。在该边栏选项卡中，你可以查看角色的名称以及分配给该角色的用户和组的数目。


   
>[AZURE.NOTE] 基于角色的访问控制只能在自动帐户级别设置，不能在自动帐户下的任何资源上设置。

可以将多个角色分配给用户、组或应用程序。例如，如果将“自动化操作员”角色和“读者”角色一起添加给用户，用户就可以查看所有自动化资源并执行 Runbook 作业。你可以展开下拉列表，以便查看分配给用户的角色的列表。

 
 
### 删除用户

你可以删除不管理自动化帐户或不再为组织工作的用户的访问权限。下面是删除用户的步骤：

1.	在“用户”边栏选项卡中，选择要删除的已分配角色。

2.	单击“分配详细信息”边栏选项卡中的“删除”按钮。

3.	单击“是”确认删除操作。

## 被分配了角色的用户

被分配了角色的用户登录到自动化帐户以后，就可以查看“默认目录”列表中列出的所有者的帐户。为了查看被添加到的自动化帐户，该用户必须将默认目录切换成所有者的默认目录。


### 自动化操作员角色的用户体验

被分配了“自动化操作员”角色的用户在查看被分配到的自动化帐户时，只能查看在自动化帐户中创建的 Runbook、Runbook 作业和计划的列表，而不能查看其定义。该用户可以启动、停止、暂停、恢复或计划 Runbook 作业。该用户将无法访问其他自动化资源，例如配置、混合辅助角色组或 DSC 节点。

![不能访问资源](./media/automation-role-based-access-control/automation-10-no-access-to-resources.png)

当用户单击 Runbook 时，系统不会提供查看源或编辑 Runbook 所需的命令，因为自动化操作员角色不允许访问它们。



用户将具有查看和创建计划的访问权限，但没有任何其他资产类型的访问权限。



## 使用 Azure PowerShell 为自动化帐户配置 RBAC

还可以使用以下 Azure PowerShell cmdlet 为自动化帐户配置基于角色的访问权限。

• [Get-AzureRmRoleDefinition](https://msdn.microsoft.com/zh-cn/library/mt603792.aspx) 列出 Azure Active Directory RBAC 中提供的所有角色。你可以使用此命令和 **Name** 属性来列出具有特定角色的所有用户。  
    **示例：**  
    ![获取角色定义](./media/automation-role-based-access-control/automation-14-get-azurerm-role-definition.png)

• [Get-AzureRmRoleAssignment](https://msdn.microsoft.com/zh-cn/library/mt619413.aspx) 列出指定作用域的 Azure RBAC 角色分配。在没有任何参数的情况下，此命令返回在订阅下进行的所有角色分配。使用 **ExpandPrincipalGroups** 参数可列出针对指定用户和用户所在组的访问权限分配情况。  
    **示例：** 使用以下命令列出自动化帐户中的所有用户及其角色。

    Get-AzureRMRoleAssignment -scope “/subscriptions/<SubscriptionID>/resourcegroups/<Resource Group Name>/Providers/Microsoft.Automation/automationAccounts/<Automation Account Name>” 

![获取角色分配](./media/automation-role-based-access-control/automation-15-get-azurerm-role-assignment.png)

• [New-AzureRmRoleAssignment](https://msdn.microsoft.com/zh-cn/library/mt603580.aspx)：授予对特定作用域的用户、组和应用程序的访问权限。  
    **示例：** 使用以下命令为“自动化帐户”作用域的用户创建“自动化操作员”新角色。

    New-AzureRmRoleAssignment -SignInName <sign-in Id of a user you wish to grant access> -RoleDefinitionName "Automation operator" -Scope “/subscriptions/<SubscriptionID>/resourcegroups/<Resource Group Name>/Providers/Microsoft.Automation/automationAccounts/<Automation Account Name>”  

![新建角色分配](./media/automation-role-based-access-control/automation-16-new-azurerm-role-assignment.png)

• 使用 [Remove-AzureRmRoleAssignment](https://msdn.microsoft.com/zh-cn/library/mt603781.aspx) 删除对特定作用域的指定用户、组或应用程序的访问权限。
    **示例：** 使用以下命令为“自动化帐户”作用域的用户创建“自动化操作员”新角色。

    Remove-AzureRmRoleAssignment -SignInName "<sign-in Id of a user you wish to remove>" -RoleDefinitionName "Automation Operator" -Scope “/subscriptions/<SubscriptionID>/resourcegroups/<Resource Group Name>/Providers/Microsoft.Automation/automationAccounts/<Automation Account Name>”

在上述 cmdlet 中，请将登录名称、订阅 ID、资源组名称和自动化帐户名称替换为你的帐户详细信息。当系统提示你继续操作以删除角色分配时，请选择“是”。


## 后续步骤

- 有关以不同方式启动 Runbook 的详细信息，请参阅[启动 Runbook](/documentation/articles/automation-starting-a-runbook)

<!---HONumber=Mooncake_0307_2016-->
