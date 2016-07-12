<properties 
	pageTitle="从 Web 应用程序使用 Azure 密钥保管库 | Azure" 
	description="本教程帮助你了解如何从 Web 应用程序使用 Azure 密钥保管库。" 
	services="key-vault" 
	documentationCenter="" 
	authors="adhurwit"
	manager=""
	tags="azure-resource-manager"/>

<tags 
	ms.service="key-vault" 
	ms.date="04/13/2016" 
	wacn.date="05/13/2016"/>

# 从 Web 应用程序使用 Azure 密钥保管库 #

## 介绍  
本教程帮助你了解如何从 Azure 中的 Web 应用程序使用 Azure 密钥保管库。其中将会引导你访问 Azure 密钥保管库中的机密，以便可以在 Web 应用程序中使用该机密。

**估计完成时间：**15 分钟


有关 Azure 密钥保管库的概述信息，请参阅[什么是 Azure 密钥保管库？](/documentation/articles/key-vault-whatis/)

## 先决条件 

若要完成本教程，你必须准备好以下各项：

- Azure 密钥保管库中的机密的 URI
- 已在 Azure Active Directory 中注册的、有权访问你的密钥保管库的 Web 应用程序的客户端 ID 和客户端密码
- Web 应用程序。我们将演示针对 Azure 中作为 Web 应用程序部署的 ASP.NET MVC 应用程序的步骤。 

> [AZURE.NOTE] 你必须已完成 [Azure 密钥保管库入门](/documentation/articles/key-vault-get-started/)中列出的适用于本教程的步骤，以便获取 Web 应用程序的机密、客户端 ID 和客户端密钥的 URI。

要访问密钥保管库的 Web 应用程序已在 Azure Active Directory 中注册，因此有权访问你的密钥保管库。如果这不是这样，请返回入门教程中的“注册应用程序”，并重复列出的步骤。

本教程面向 Web 开发人员，他们已经了解有关在 Azure 上创建 Web 应用程序的基本知识。有关 Azure Web Apps 的详细信息，请参阅 [Web Apps 概述](/home/features/web-site)。



## <a id="packages"></a>添加 NuGet 包 ##
需要在 Web 应用程序上安装两个包。

- Active Directory 身份验证库 - 包含用来与 Azure Active Directory 交互以及管理用户标识的方法
- Azure 密钥保管库库 - 包含用来与 Azure 密钥保管库交互的方法


可以在 Package Manager Console 中使用 Install-Package 命令安装这两个包。

	// this is currently the latest stable version of ADAL
	Install-Package Microsoft.IdentityModel.Clients.ActiveDirectory -Version 2.16.204221202

	Install-Package Microsoft.Azure.KeyVault 


## <a id="webconfig"></a>修改 Web.Config ##
需要按如下所示将三个应用程序设置添加到 web.config 文件。

	<!-- ClientId and ClientSecret refer to the web application registration with Azure Active Directory -->
    <add key="ClientId" value="clientid" />
    <add key="ClientSecret" value="clientsecret" />

	<!-- SecretUri is the URI for the secret in Azure Key Vault -->
    <add key="SecretUri" value="secreturi" />


如果你不打算将应用程序作作为 Azure Web 应用程序托管，则应在 web.config 中添加实际的客户端 Id、客户端密钥和机密 URI 值。否则，请将这些虚构值，因为我们将在 Azure 门户中添加实际值以提高安全级别。


## <a id="gettoken"></a>添加方法以获取访问令牌 ##
若要使用密钥保管库 API，你需要一个访问令牌。密钥保管库客户端将处理对密钥保管库 API 的调用，但你需要为该 API 提供一个用于获取访问令牌的函数。

以下代码可从 Azure Active Directory 获取访问令牌。可将此代码添加在应用程序中的任意位置。我想要添加一个 Utils 或 EncryptionHelper 类。

	//add these using statements
    using Microsoft.IdentityModel.Clients.ActiveDirectory;
	using System.Web.Configuration;
	
	//this is an optional property to hold the secret after it is retrieved
	public static string EncryptSecret { get; set; }

	//the method that will be provided to the KeyVaultClient
	public async static Task<string> GetToken(string authority, string resource, string scope)
    {
	    var authContext = new AuthenticationContext(authority);
	    ClientCredential clientCred = new ClientCredential(WebConfigurationManager.AppSettings["ClientId"],
                    WebConfigurationManager.AppSettings["ClientSecret"]);
	    AuthenticationResult result = await authContext.AcquireTokenAsync(resource, clientCred);
	    
	    if (result == null)
	    	throw new InvalidOperationException("Failed to obtain the JWT token");
	    
	    return result.AccessToken;
    }

> [AZURE.NOTE] 使用客户端 ID 和客户端密码是对 Azure AD 应用程序进行身份验证的最简单方法。并且在 web 应用程序中使用它可以实现职责分离，并更好地控制密钥管理。但它不依赖于将客户端密码放入配置设置中，对于某些人来说这可能就像将要保护的机密放入配置设置中一样具有风险。有关如何使用客户端 ID 和证书（而不是客户端 ID 和客户端密码）对 Azure AD 应用程序进行身份验证的讨论，请参阅下文。



## <a id="appstart"></a>在 Application Start 中检索机密 ##
现在，我们需要添加代码来调用密钥保管库 API 并检索机密。以下代码可添加到任何位置，前提是在使用之前调用它。我已将此代码放在 Global.asax 中的 Application Start 事件内，这样，在启动应用程序时，该代码将运行一次，并使机密可用于应用程序。

	//add these using statements
	using Microsoft.Azure.KeyVault;
	using System.Web.Configuration;

	// I put my GetToken method in a Utils class. Change for wherever you placed your method. 
    var kv = new KeyVaultClient(new KeyVaultClient.AuthenticationCallback(Utils.GetToken));

	var sec = kv.GetSecretAsync(WebConfigurationManager.AppSettings["SecretUri"]).Result.Value;
	
	//I put a variable in a Utils class to hold the secret for general  application use. 
    Utils.EncryptSecret = sec;


## 使用证书（而不是客户端密码）进行身份验证 
另一种对 Azure AD 应用程序进行身份验证的方法是使用客户端 ID 和证书（而不是客户端 ID 和客户端密码）。下面是在 Azure Web 应用中使用证书的步骤：

1. 获取或创建证书
2. 将证书与 Azure AD 应用程序相关联
3. 在 Web 应用中添加代码以使用证书
4. 将证书添加到 Web 应用


**获取或创建证书** 出于我们的目的，我们将生成测试证书。下面是几个可在开发人员命令提示符下使用以创建证书的命令。将目录更改为要在其中创建证书文件的位置。

	makecert -sv mykey.pvk -n "cn=KVWebApp" KVWebApp.cer -b 07/31/2015 -e 07/31/2016 -r
	pvk2pfx -pvk mykey.pvk -spc KVWebApp.cer -pfx KVWebApp.pfx -po test123

记下 .pfx 的结束日期和密码（在此示例中为：07/31/2016 和 test123）。稍后你将需要它们。

有关创建测试证书的详细信息，请参阅[如何：创建自己的测试证书](https://msdn.microsoft.com/zh-cn/library/ff699202.aspx)


**将证书与 Azure AD 应用程序相关联** 既然你拥有一个证书，你需要将其与 Azure AD 应用程序相关联。但当前 Azure 管理门户不支持此操作。你需要改用 Powershell。以下是你需要运行的命令：

	$x509 = New-Object System.Security.Cryptography.X509Certificates.X509Certificate2
	
	PS C:\> $x509.Import("C:\data\KVWebApp.cer")
	
	PS C:\> $credValue = [System.Convert]::ToBase64String($x509.GetRawCertData())
	
	PS C:\> $now = [System.DateTime]::Now
	
	# this is where the end date from the cert above is used
	PS C:\> $yearfromnow = [System.DateTime]::Parse("2016-07-31") 
	
	PS C:\> $adapp = New-AzureRmADApplication -DisplayName "KVWebApp" -HomePage "http://kvwebapp" -IdentifierUris "http://kvwebapp" -KeyValue $credValue -KeyType "AsymmetricX509Cert" -KeyUsage "Verify" -StartDate $now -EndDate $yearfromnow
	
	PS C:\> $sp = New-AzureRmADServicePrincipal -ApplicationId $adapp.ApplicationId
	
	PS C:\> Set-AzureRmKeyVaultAccessPolicy -VaultName 'contosokv' -ServicePrincipalName $sp.ServicePrincipalName -PermissionsToKeys all -ResourceGroupName 'contosorg'
	
	# get the thumbprint to use in your app settings
	PS C:\>$x509.Thumbprint

运行这些命令后，你可以在 Azure AD 中看到该应用程序。如果你最初未看到该应用程序，可搜索“我的公司拥有的应用程序”，而不是“我的公司使用的应用程序”。

若要了解有关 Azure AD 应用程序对象和 ServicePrincipal 对象的详细信息，请参阅[应用程序对象和服务主体对象](/documentation/articles/active-directory-application-objects/)



**在 Web 应用中添加代码以使用证书** 现在，我们将在 Web 应用中添加代码以访问证书并使用它进行身份验证。

首先是用于访问证书的代码。

    public static class CertificateHelper
    {
        public static X509Certificate2 FindCertificateByThumbprint(string findValue)
        {
            X509Store store = new X509Store(StoreName.My, StoreLocation.CurrentUser);
            try
            {
                store.Open(OpenFlags.ReadOnly);
                X509Certificate2Collection col = store.Certificates.Find(X509FindType.FindByThumbprint, 
                    findValue, false); // Don't validate certs, since the test root isn't installed.
                if (col == null || col.Count == 0)
                    return null;
                return col[0];
            }
            finally
            {
                store.Close();
            }
        }
    }


请注意，StoreLocation 是 CurrentUser，而不是 LocalMachine。并且，我们为 Find 方法提供“false”，因为我们使用的是测试证书。


其次是使用 CertificateHelper 并创建 ClientAssertionCertificate 的代码，这是身份验证所需的。

    public static ClientAssertionCertificate AssertionCert { get; set; }

    public static void GetCert()
    {
        var clientAssertionCertPfx = CertificateHelper.FindCertificateByThumbprint(WebConfigurationManager.AppSettings["thumbprint"]);
        AssertionCert = new ClientAssertionCertificate(WebConfigurationManager.AppSettings["clientid"], clientAssertionCertPfx);
    }


以下是新代码，用于获取访问令牌。这将替换上面的 GetToken 方法。为方便起见，我为它起了不同名称。

    public static async Task<string> GetAccessToken(string authority, string resource, string scope)
    {
        var context = new AuthenticationContext(authority, TokenCache.DefaultShared);
        var result = await context.AcquireTokenAsync(resource, AssertionCert);
        return result.AccessToken;
    }

为了便于使用，我已将所有这些代码放到我的 Web 应用项目的 Utils 类中。

最后的代码更改是在 Application\_Start 方法中。首先，我们需要调用 GetCert() 方法以加载 ClientAssertionCertificate。然后，我们将更改在创建新的 KeyVaultClient 时提供的回调方法。请注意，这将替换上面使用的代码。

    Utils.GetCert();
    var kv = new KeyVaultClient(new KeyVaultClient.AuthenticationCallback(Utils.GetAccessToken));


**将证书添加到 Web 应用** 将证书添加到 Web 应用是一个简单的分为两步的过程。首先，转到 Azure 门户并导航到你的 Web 应用。在你的 Web 应用的“设置”边栏选项卡中，单击“自定义域和 SSL”所对应的条目。在打开的边栏选项卡中，你将能够上载上面创建的证书 KVWebApp.pfx，请确保记住 pfx 的密码。


你需要执行的最后一项操作是将应用程序设置添加到 Web 应用中，该设置名为 WEBSITE\_LOAD\_CERTIFICATES，值为 *。这将确保加载所有证书。如果你只想加载已上载的证书，则可以输入这些证书的指纹的逗号分隔列表。

若要了解有关将证书添加到 Web 应用的详细信息，请参阅[在 Azure 网站应用程序中使用证书](https://azure.microsoft.com/blog/2014/10/27/using-certificates-in-azure-websites-applications)



## <a id="next"></a>后续步骤 ##


有关编程参考，请参阅 [Azure 密钥保管库 C# 客户端 API 参考](https://msdn.microsoft.com/zh-cn/library/azure/dn903628.aspx)。


<!--Image references-->
[1]: ./media/key-vault-use-from-web-application/PortalAppSettings.png
[2]: ./media/key-vault-use-from-web-application/PortalAddCertificate.png
 

<!---HONumber=Mooncake_0307_2016-->