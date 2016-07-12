<properties
   pageTitle="对访问群集的客户端进行身份验证 | Azure"
   description="介绍如何使用证书对访问 Service Fabric 群集的客户端进行身份验证，以及如何保护客户端与群集之间的通信。"
   services="service-fabric"
   documentationCenter=".net"
   authors="rwike77"
   manager="timlt"
   editor=""/>

<tags
   ms.service="service-fabric"
   ms.date="06/01/2016"
   wacn.date="07/04/2016"/>

# 连接到安全群集
当客户端连接到 Service Fabric 群集节点时，可以使用证书安全性来与客户端建立经过身份验证的安全通信。这可确保只有经过授权的用户才能访问该群集和部署的应用程序，以及执行管理任务。创建群集时，必须事先在该群集上启用证书安全性。有关群集安全方案的详细信息，请参阅 [Cluster security](/documentation/articles/service-fabric-cluster-security/)（群集安全性）。

若要使用证书安全性来保护客户端与与群集节点之间的通信，必须先获取客户端证书，并将其安装到本地计算机上的个人（我的）存储或当前用户的个人存储。

运行以下 PowerShell cmdlet，在要用于访问群集的计算机上设置证书。

```powershell
Import-PfxCertificate -Exportable -CertStoreLocation Cert:\CurrentUser\My `
        -FilePath C:\docDemo\certs\DocDemoClusterCert.pfx `
        -Password (ConvertTo-SecureString -String test -AsPlainText -Force)
```

如果它是自签名证书，则需要将其导入计算机的“受信任人”存储中才能使用此证书连接到安全群集。

```powershell
Import-PfxCertificate -Exportable -CertStoreLocation Cert:\CurrentUser\TrustedPeople `
-FilePath C:\docDemo\certs\DocDemoClusterCert.pfx `
-Password (ConvertTo-SecureString -String test -AsPlainText -Force)
```

<a id="connectsecurecluster"></a>
## 使用 PowerShell 连接到安全群集

运行以下 PowerShell 命令连接到安全群集。证书详细信息必须与群集节点上的证书匹配。

```powershell
Connect-ServiceFabricCluster -ConnectionEndpoint <Cluster FQDN>:19000 `
          -KeepAliveIntervalInSec 10 `
          -X509Credential -ServerCertThumbprint <Certificate Thumbprint> `
          -FindType FindByThumbprint -FindValue <Certificate Thumbprint> `
          -StoreLocation CurrentUser -StoreName My
```

例如，上述 PowerShell 命令应该类似于以下内容：

```powershell
Connect-ServiceFabricCluster -ConnectionEndpoint clustername.chinaeast.chinacloudapp.cn:19000 `
          -KeepAliveIntervalInSec 10 `
          -X509Credential -ServerCertThumbprint C179E609BBF0B227844342535142306F3913D6ED `
          -FindType FindByThumbprint -FindValue C179E609BBF0B227844342535142306F3913D6ED `
          -StoreLocation CurrentUser -StoreName My
```

## 使用 FabricClient API 连接到安全群集
请参阅下面的 [FabricClient](https://msdn.microsoft.com/zh-cn/library/system.fabric.fabricclient.aspx)。群集中的节点必须具有有效的证书，在 SAN 中，这些证书的公用名或 DNS 名出现在 [FabricClient](https://msdn.microsoft.com/zh-cn/library/system.fabric.fabricclient.aspx) 上设置的 [RemoteCommonNames 属性](https://msdn.microsoft.com/zh-cn/library/azure/system.fabric.x509credentials.remotecommonnames.aspx)中。这样就可以在客户端与群集节点之间进行相互身份验证。

```csharp
string thumb = "C179E609BBF0B227844342535142306F3913D6ED";
string CommonName = "www.clustername.westus.azure.com";
string connection = "clustername.chinaeast.chinacloudapp.cn:19000";

X509Credentials xc = GetCredentials(thumb, CommonName);
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

static X509Credentials GetCredentials(string thumb, string name)
{
    X509Credentials xc = new X509Credentials();
    xc.StoreLocation = StoreLocation.CurrentUser;
    xc.StoreName = "MY";
    xc.FindType = X509FindType.FindByThumbprint;
    xc.FindValue = thumb;
    xc.RemoteCertThumbprints.Add(thumb);
    xc.RemoteCommonNames.Add(name);
    xc.ProtectionLevel = ProtectionLevel.EncryptAndSign;
    return xc;
}
```


## 后续步骤

- [Service Fabric 群集升级过程与期望](/documentation/articles/service-fabric-cluster-upgrade/)
- [在 Visual Studio 中管理 Service Fabric 应用程序](/documentation/articles/service-fabric-manage-application-in-visual-studio/)。
- [Service Fabric 运行状况模型简介](/documentation/articles/service-fabric-health-introduction/)
- [应用程序安全性和 RunAs](/documentation/articles/service-fabric-application-runas-security/)

<!---HONumber=Mooncake_0627_2016-->