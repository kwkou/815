<properties
    pageTitle="对群集的客户端访问进行身份验证 | Azure"
    description="介绍如何对 Service Fabric 群集的客户端访问进行身份验证以及如何确保客户端与群集之间的通信安全。"
    services="service-fabric"
    documentationcenter=".net"
    author="rwike77"
    manager="timlt"
    editor="" />
<tags
    ms.assetid="759a539e-e5e6-4055-bff5-d38804656e10"
    ms.service="service-fabric"
    ms.devlang="dotnet"
    ms.topic="article"
    ms.tgt_pltfrm="na"
    ms.workload="na"
    ms.date="11/11/2016"
    wacn.date="01/25/2017"
    ms.author="ryanwi" />

# 连接到安全群集
当客户端连接到 Service Fabric 群集节点时，可以使用证书安全性或 Azure Active Directory (AAD) 来与客户端建立经过身份验证的安全通信。此身份验证可确保只有经过授权的用户才能访问该群集和部署的应用程序，以及执行管理任务。创建群集时，必须事先在该群集上启用证书或 AAD 安全性。有关群集安全方案的详细信息，请参阅[群集安全性](/documentation/articles/service-fabric-cluster-security/)。如果要连接到使用证书保护的群集，在要连接到群集的计算机上[设置客户端证书](/documentation/articles/service-fabric-connect-to-secure-cluster/#connectsecureclustersetupclientcert)。

<a id="connectsecureclustercli"></a>

## 使用 Azure CLI 连接到安全群集
以下 Azure CLI 命令说明如何连接到安全群集。

### 使用客户端证书连接到安全群集
证书详细信息必须与群集节点上的证书匹配。

如果证书具有证书颁发机构 (CA)，需要按以下示例所示添加参数 `--ca-cert-path`：


 	azure servicefabric cluster connect --connection-endpoint https://ip:19080 --client-key-path /tmp/key --client-cert-path /tmp/cert --ca-cert-path /tmp/ca1,/tmp/ca2 

如果有多个 CA，请使用逗号作为分隔符。

如果证书中的“公用名”与连接终结点不匹配，可以使用 `--strict-ssl-false` 参数绕过验证。


	azure servicefabric cluster connect --connection-endpoint https://ip:19080 --client-key-path /tmp/key --client-cert-path /tmp/cert --ca-cert-path /tmp/ca1,/tmp/ca2 --strict-ssl-false 


如果想要跳过 CA 验证，可以添加 ``--reject-unauthorized-false`` 参数，如以下命令中所示：


	azure servicefabric cluster connect --connection-endpoint https://ip:19080 --client-key-path /tmp/key --client-cert-path /tmp/cert --reject-unauthorized-false 


若要连接到使用自签名证书保护的群集，请使用以下命令删除 CA 验证和公用名验证：


	azure servicefabric cluster connect --connection-endpoint https://ip:19080 --client-key-path /tmp/key --client-cert-path /tmp/cert --strict-ssl-false --reject-unauthorized-false


连接后，应该能够运行其他 CLI 命令来与群集交互。

<a id="connectsecurecluster"></a>

## 使用 PowerShell 连接到安全群集
在通过 PowerShell 对群集执行操作之前，请首先建立与群集的连接。群集连接用于给定 PowerShell 会话中的所有后续命令。

### 连接到不安全的群集

若要连接到不安全群集，请向 **Connect-ServiceFabricCluster** 命令提供群集终结点地址：


	Connect-ServiceFabricCluster -ConnectionEndpoint <Cluster FQDN>:19000 


### 使用 Azure Active Directory 连接到安全群集

若要连接到使用 Azure Active Directory 授权群集管理员访问权限的安全集群，请提供群集证书指纹，并使用 *AzureActiveDirectory* 标志。


	Connect-ServiceFabricCluster -ConnectionEndpoint <Cluster FQDN>:19000 `
	-ServerCertThumbprint <Server Certificate Thumbprint> `
	-AzureActiveDirectory


### 使用客户端证书连接到安全群集
运行以下 PowerShell 命令以连接到使用客户端证书授权管理员访问权限的安全群集。提供群集证书指纹以及已授予群集管理权限的客户端证书的指纹。证书详细信息必须与群集节点上的证书匹配。


	Connect-ServiceFabricCluster -ConnectionEndpoint <Cluster FQDN>:19000 `
	          -KeepAliveIntervalInSec 10 `
	          -X509Credential -ServerCertThumbprint <Certificate Thumbprint> `
	          -FindType FindByThumbprint -FindValue <Certificate Thumbprint> `
	          -StoreLocation CurrentUser -StoreName My


*ServerCertThumbprint* 是群集节点上安装的服务器证书的指纹。*FindValue* 是管理客户端证书的指纹。填充参数时，命令如以下示例所示：


	Connect-ServiceFabricCluster -ConnectionEndpoint clustername.chinaeast.cloudapp.chinacloudapi.cn:19000 `
	          -KeepAliveIntervalInSec 10 `
	          -X509Credential -ServerCertThumbprint A8136758F4AB8962AF2BF3F27921BE1DF67F4326 `
	          -FindType FindByThumbprint -FindValue 71DE04467C9ED0544D021098BCD44C71E183414E `
	          -StoreLocation CurrentUser -StoreName My



<a id="connectsecureclusterfabricclient"></a>

## 使用 FabricClient API 连接到安全群集
Service Fabric SDK 为群集管理提供 [FabricClient](https://msdn.microsoft.com/zh-cn/library/system.fabric.fabricclient.aspx) 类。

### 连接到不安全的群集

若要连接到远程不安全群集，只需创建一个 FabricClient 实例并提供群集地址：


	FabricClient fabricClient = new FabricClient("clustername.chinaeast.cloudapp.chinacloudapi.cn:19000");


对于在群集内运行的代码（例如，在可靠服务中），请创建 FabricClient，无需指定群集地址。FabricClient 连接到代码当前正在运行的节点上的本地管理网关，从而避免额外的网络跃点。


	FabricClient fabricClient = new FabricClient();


### 使用客户端证书连接到安全群集

群集中的节点必须具有有效的证书，在 SAN 中，这些证书的公用名或 DNS 名出现在 [FabricClient](https://msdn.microsoft.com/zh-cn/library/system.fabric.fabricclient.aspx) 上设置的 [RemoteCommonNames 属性](https://msdn.microsoft.com/zh-cn/library/azure/system.fabric.x509credentials.remotecommonnames.aspx)中。按照此流程操作可在客户端与群集节点之间进行相互身份验证。


	string clientCertThumb = "71DE04467C9ED0544D021098BCD44C71E183414E";
	string serverCertThumb = "A8136758F4AB8962AF2BF3F27921BE1DF67F4326";
	string CommonName = "www.clustername.chinaeast.cloudapp.chinacloudapi.cn";
	string connection = "clustername.chinaeast.cloudapp.chinacloudapi.cn:19000";
	
	X509Credentials xc = GetCredentials(clientCertThumb, serverCertThumb, CommonName);
	FabricClient fc = new FabricClient(xc, connection);
	Task<bool> t = fc.PropertyManager.NameExistsAsync(new Uri("fabric:/any"));
	try
	{
	    bool result = t.Result;
	    Console.WriteLine("Cluster is connected");
	}
	catch (AggregateException ae)
	{
	    Console.WriteLine("Connect failed: {0}", ae.InnerException.Message);
	}
	catch (Exception e)
	{
	    Console.WriteLine("Connect failed: {0}", e.Message);
	}
	
	...
	
	static X509Credentials GetCredentials(string clientCertThumb, string serverCertThumb, string name)
	{
	    X509Credentials xc = new X509Credentials();

    	    // Client certificate
	    xc.StoreLocation = StoreLocation.CurrentUser;
	    xc.StoreName = "MY";
	    xc.FindType = X509FindType.FindByThumbprint;
	    xc.FindValue = thumb;
    	    // Server certificate
	    xc.RemoteCertThumbprints.Add(thumb);
	    xc.RemoteCommonNames.Add(name);
	    xc.ProtectionLevel = ProtectionLevel.EncryptAndSign;
	    return xc;
	}


### 使用 Azure Active Directory 连接到安全群集

按照此流程操作可为客户端标识启用 Azure Active Directory，为服务器标识启用服务器证书。

使用交互模式，此模式会弹出 AAD 交互式登录对话框：


	string serverCertThumb = "A8136758F4AB8962AF2BF3F27921BE1DF67F4326";
	string connection = "clustername.chinaeast.cloudapp.chinacloudapi.cn:19000";

	ClaimsCredentials claimsCredentials = new ClaimsCredentials();
	claimsCredentials.ServerThumbprints.Add(serverCertThumb);

	FabricClient fc = new FabricClient(
	    claimsCredentials,
	    connection);

	try
	{
	    var ret = fc.ClusterManager.GetClusterManifestAsync().Result;
	    Console.WriteLine(ret.ToString());
	}
	catch (AggregateException ae)
	{
	    Console.WriteLine("Connect failed: {0}", ae.InnerException.Message);
	}
	catch (Exception e)
	{
	    Console.WriteLine("Connect failed: {0}", e.Message);
	}


使用没有任何人工交互的静默模式：

（此示例依赖于 Microsoft.IdentityModel.Clients.ActiveDirectory，版本：2.19.208020213

有关如何获取令牌以及其他信息，请参阅 [Microsoft.IdentityModel.Clients.ActiveDirectory Namespace](https://msdn.microsoft.com/zh-cn/library/microsoft.identitymodel.clients.activedirectory.aspx)（Microsoft.IdentityModel.Clients.ActiveDirectory 命名空间））


	string tenantId = "c15cfcea-02c1-40dc-8466-fbd0ee0b05d2";
	string clientApplicationId = "118473c2-7619-46e3-a8e4-6da8d5f56e12";
	string webApplicationId = "53E6948C-0897-4DA6-B26A-EE2A38A690B4";

	string token = GetAccessToken(
	    tenantId,
	    webApplicationId,
	    clientApplicationId,
	    "urn:ietf:wg:oauth:2.0:oob"
	    );

	string serverCertThumb = "A8136758F4AB8962AF2BF3F27921BE1DF67F4326";
	string connection = "clustername.chinaeast.cloudapp.chinacloudapi.cn:19000";
	ClaimsCredentials claimsCredentials = new ClaimsCredentials();
	claimsCredentials.ServerThumbprints.Add(serverCertThumb);
	claimsCredentials.LocalClaims = token;

	FabricClient fc = new FabricClient(
	   claimsCredentials,
	   connection);

	try
	{
	    var ret = fc.ClusterManager.GetClusterManifestAsync().Result;
	    Console.WriteLine(ret.ToString());
	}
	catch (AggregateException ae)
	{
	    Console.WriteLine("Connect failed: {0}", ae.InnerException.Message);
	}
	catch (Exception e)
	{
	    Console.WriteLine("Connect failed: {0}", e.Message);
	}

	...

	static string GetAccessToken(
	    string tenantId,
	    string resource,
	    string clientId,
	    string redirectUri)
	{
	    string authorityFormat = @"https://login.partner.microsoftonline.cn/{0}";
	    string authority = string.Format(CultureInfo.InvariantCulture, authorityFormat, tenantId);
	    AuthenticationContext authContext = new AuthenticationContext(authority);

	    string token = "";
	    try
	    {
	        var authResult = authContext.AcquireToken(
	            resource,
	            clientId,
	            new UserCredential("TestAdmin@clustenametenant.partner.onmsmicrosoft.cn", "TestPassword"));
	        token = authResult.AccessToken;
	    }
	    catch (AdalException ex)
	    {
	        Console.WriteLine("Get AccessToken failed: {0}", ex.Message);
	    }

	    return token;
	}



<a id="connectsecureclustersfx"></a>

## 使用 Service Fabric Explorer 连接到安全群集
若要连接到给定群集的 [Service Fabric Explorer](/documentation/articles/service-fabric-visualizing-your-cluster/)，请将浏览器指向：

`http://<your-cluster-endpoint>:19080/Explorer`  


Azure 门户的群集基本信息窗格中也提供了完整 URL。

### 使用 Azure Active Directory 连接到安全群集

若要连接到用 AAD 保护的群集，请将浏览器指向：

`https://<your-cluster-endpoint>:19080/Explorer`  


系统将自动提示使用 AAD 登录。

### 使用客户端证书连接到安全群集

若要连接到用证书保护的群集，请将浏览器指向：

`https://<your-cluster-endpoint>:19080/Explorer`  


系统将自动提示选择客户端证书。

<a id="connectsecureclustersetupclientcert"></a>
## 在远程计算机上设置客户端证书
至少应有两个证书用于保护群集，一个用于保护群集和服务器证书，另一个用于保护客户端访问。建议您还使用其他辅助证书和客户端访问证书。若要使用证书安全性来保护客户端与群集节点之间的通信，首先需要获取和安装客户端证书。可将证书安装到本地计算机或当前用户的个人（我的）存储中。你还需要服务器证书的指纹，以便客户端可以对群集进行身份验证。

运行以下 PowerShell cmdlet，在访问群集的计算机上设置客户端证书。


	Import-PfxCertificate -Exportable -CertStoreLocation Cert:\CurrentUser\My `
	        -FilePath C:\docDemo\certs\DocDemoClusterCert.pfx `
	        -Password (ConvertTo-SecureString -String test -AsPlainText -Force)


如果它是自签名证书，则需要将其导入计算机的“受信任人”存储中才能使用此证书连接到安全群集。


	Import-PfxCertificate -Exportable -CertStoreLocation Cert:\CurrentUser\TrustedPeople `
	-FilePath C:\docDemo\certs\DocDemoClusterCert.pfx `
	-Password (ConvertTo-SecureString -String test -AsPlainText -Force)


## 后续步骤

- [Service Fabric 群集升级过程与期望](/documentation/articles/service-fabric-cluster-upgrade/)
- [在 Visual Studio 中管理 Service Fabric 应用程序](/documentation/articles/service-fabric-manage-application-in-visual-studio/)。
- [Service Fabric 运行状况模型简介](/documentation/articles/service-fabric-health-introduction/)
- [应用程序安全性和 RunAs](/documentation/articles/service-fabric-application-runas-security/)

<!---HONumber=Mooncake_Quality_Review_0125_2017-->