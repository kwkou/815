## 获取资源管理器令牌

Azure Active Directory 必须使用 Azure Resource Manager 来验证所有针对资源执行的任务。此处显示的示例使用密码身份验证，有关其他方法，请参阅[对 Azure Resource Manager 请求进行身份验证][lnk-authenticate-arm]。

1. 将以下代码添加到 Program.cs 中的 **Main** 方法，以使用应用程序 ID 和密码从 Azure AD 中检索令牌。

    ```
    var authContext = new AuthenticationContext(string.Format  
      ("https://login.chinacloudapi.cn/{0}", tenantId));
    var credential = new ClientCredential(applicationId, password);
    AuthenticationResult token = authContext.AcquireTokenAsync
      ("https://management.core.chinacloudapi.cn/", credential).Result;
    
    if (token == null)
    {
      Console.WriteLine("Failed to obtain the token");
      return;
    }
    ```

2. 创建一个 **ResourceManagementClient** 对象，该对象通过在 **Main** 方法的末尾添加以下代码来使用令牌：

    ```
    var creds = new TokenCredentials(token.AccessToken);
    var client = new ResourceManagementClient(new Uri("Https://management.chinacloudapi.cn"), creds);
    client.SubscriptionId = subscriptionId;
    ```

3. 创建或获取对你使用的资源组的引用：

    ```
    var rgResponse = client.ResourceGroups.CreateOrUpdate(rgName,
        new ResourceGroup("China East"));
    if (rgResponse.Properties.ProvisioningState != "Succeeded")
    {
      Console.WriteLine("Problem creating resource group");
      return;
    }
    ```

[lnk-authenticate-arm]: https://msdn.microsoft.com/zh-cn/library/azure/dn790557.aspx

<!---HONumber=Mooncake_0321_2016-->