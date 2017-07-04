## 准备对 Resource Manager 请求进行身份验证

你必须使用 [Azure Resource Manager][lnk-authenticate-arm] 配合 Azure Active Directory (AD) 来验证所有针对资源执行的操作。最简单的配置方式是使用 PowerShell 或 Azure CLI。

继续之前，你应安装 [Azure PowerShell 1.0][lnk-powershell-install] 或更高版本。

以下步骤说明如何使用 PowerShell 设置 AD 应用程序的密码身份验证。可以在标准 PowerShell 会话中运行这些命令。

1. 使用以下命令登录到你的 Azure 订阅：

    ```
    Login-AzureRmAccount -Environment $(Get-AzureRmEnvironment -Name AzureChinaCloud)
    ```

2. 记下你的 TenantId 和 SubscriptionId。稍后你将需要它们。

3. 使用以下命令并替换占位符，以创建新的 Azure Active Directory 应用程序：

    - **{Display name}**：应用程序的显示名称，如 MySampleApp
    - **{Home page URL}**：应用程序主页的 URL，如 **http://mysampleapp/home**。此 URL 不需要指向实际的应用程序。
    - **{Application identifier}**：唯一标识符，如 **http://mysampleapp**。此 URL 不需要指向实际的应用程序。
    - **{Password}**：用来对应用程序进行身份验证的密码。

    ```
    New-AzureRmADApplication -DisplayName {Display name} -HomePage {Home page URL} -IdentifierUris {Application identifier} -Password {Password}
    ```
    
4. 请记下创建的应用程序的 **ApplicationId**。稍后你将需要此项。

5. 使用以下命令（将 **{MyApplicationId}** 替换为上一步骤中的 **ApplicationId**）创建新的服务主体：

    ```
    New-AzureRmADServicePrincipal -ApplicationId {MyApplicationId}
    ```
    
6. 使用以下命令（将 **{MyApplicationId}** 替换为 **ApplicationId**）设置角色分配。

    ```
    New-AzureRmRoleAssignment -RoleDefinitionName Owner -ServicePrincipalName {MyApplicationId}
    ```
    
现在，你已创建可从自定义 C# 应用程序进行身份验证的 Azure AD 应用程序。在本教程的后续内容中，你需要用到以下值：

- TenantId
- SubscriptionId
- ApplicationId
- 密码

[lnk-authenticate-arm]: https://msdn.microsoft.com/zh-cn/library/azure/dn790557.aspx
[lnk-powershell-install]: /documentation/articles/powershell-install-configure/

<!---HONumber=Mooncake_0321_2016-->