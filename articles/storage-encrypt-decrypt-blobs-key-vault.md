<properties
   pageTitle="在 Azure 存储空间中使用 Azure 密钥保管库加密和解密 blob | Azure"
   description="本教程将指导你如何将 Azure 存储空间的客户端加密与 Azure 密钥保管库配合使用来加密和解密 blob。"
   services="storage"
   documentationCenter=""
   authors="adhurwit"
   manager=""
   editor=""/>

<tags
   ms.service="storage"
   ms.date="03/31/2016"
   wacn.date="05/16/2016"/>

# 教程：在 Azure 存储空间中使用 Azure 密钥保管库加密和解密 Blob

## 介绍

本教程介绍如何结合使用客户端存储加密与 Azure 密钥保管库。其中将引导您完成如何在控制台应用程序中使用这些技术加密和解密 blob。

**估计完成时间：**20 分钟。

有关 Azure 密钥保管库的概述信息，请参阅[什么是 Azure 密钥保管库？](/documentation/articles/key-vault-whatis/)

有关 Azure 存储空间的客户端加密的概述信息，请参阅 [Azure 存储空间客户端加密和 Azure 密钥保管库](/documentation/articles/storage-client-side-encryption/)


## 先决条件

若要完成本教程，你必须准备好以下各项：

- Azure 存储帐户
- Visual Studio 2013 或更高版本
- Azure PowerShell


## 客户端加密概述

有关 Azure 存储空间的客户端加密的概述，请参阅 [Azure 存储空间客户端加密和 Azure 密钥保管库](/documentation/articles/storage-client-side-encryption/)

下面是客户端加密的工作原理的简要说明：

1. Azure 存储客户端 SDK 生成内容加密密钥 (CEK)，这是一次性使用对称密钥。
2. 使用此 CEK 对客户数据进行加密。
3. 然后，使用密钥加密密钥 (KEK) 对此 CEK 进行包装（加密）。KEK 由密钥标识符标识，可以是非对称密钥对或对称密钥，还可以在本地托管或存储在 Azure 密钥保管库中。存储空间客户端本身永远无法访问 KEK。它只能调用密钥保管库提供的密钥包装算法。客户可以根据需要选择使用自定义提供程序进行密钥包装/解包。
4. 然后，将已加密的数据上载到 Azure 存储服务。


## 设置你的 Azure 密钥保管库
若要继续本教程，你需要执行教程 [Azure 密钥保管库入门](/documentation/articles/key-vault-get-started/)中所述的以下步骤：

- 创建密钥保管库。
- 将密钥或密码添加到密钥保管库。
- 将应用程序注册到 Azure Active Directory。
- 授权应用程序使用密钥或密码。

记下将应用程序注册到 Azure Active Directory 时生成的 ClientID 和 ClientSecret。

在密钥保管库中创建这两个密钥。我们在本教程的其余部分将假定使用了以下名称：ContosoKeyVault 和 TestRSAKey1。


## 使用程序包和 AppSettings 创建控制台应用程序

在 Visual Studio 中创建新的控制台应用程序。

在 Package Manager Console 中添加必要的 Nuget 包。

	Install-Package WindowsAzure.Storage

	// This is the latest stable release for ADAL.
	Install-Package Microsoft.IdentityModel.Clients.ActiveDirectory -Version 2.16.204221202

	Install-Package Microsoft.Azure.KeyVault
	Install-Package Microsoft.Azure.KeyVault.Extensions


将 AppSettings 添加到 App.Config。

	<appSettings>
	    <add key="accountName" value="myaccount"/>
	    <add key="accountKey" value="theaccountkey"/>
	    <add key="clientId" value="theclientid"/>
	    <add key="clientSecret" value="theclientsecret"/>
    	<add key="container" value="stuff"/>
	</appSettings>

添加以下 `using` 语句并确保将对 System.Configuration 的引用添加到项目中。

	using Microsoft.IdentityModel.Clients.ActiveDirectory;
	using System.Configuration;
	using Microsoft.WindowsAzure.Storage.Auth;
	using Microsoft.WindowsAzure.Storage;
	using Microsoft.WindowsAzure.Storage.Blob;
	using Microsoft.Azure.KeyVault;
	using System.Threading;		
	using System.IO;


## 添加方法以便为控制台应用程序获取令牌

以下方法由密钥保管库类使用，这些类需要进行身份验证才能访问你的密钥保管库。

	private async static Task<string> GetToken(string authority, string resource, string scope)
	{
	    var authContext = new AuthenticationContext(authority);
	    ClientCredential clientCred = new ClientCredential(
	        ConfigurationManager.AppSettings["clientId"],
	        ConfigurationManager.AppSettings["clientSecret"]);
		AuthenticationResult result = await authContext.AcquireTokenAsync(resource, clientCred);

	    if (result == null)
	        throw new InvalidOperationException("Failed to obtain the JWT token");

	    return result.AccessToken;
	}

## 在你的程序中访问存储和密钥保管库

在 Main 函数中，添加以下代码。

	// This is standard code to interact with Blob storage.
	StorageCredentials creds = new StorageCredentials(
		ConfigurationManager.AppSettings["accountName"],
       	ConfigurationManager.AppSettings["accountKey"]);
	CloudStorageAccount account = new CloudStorageAccount(creds, useHttps: true);
	CloudBlobClient client = account.CreateCloudBlobClient();
	CloudBlobContainer contain = client.GetContainerReference(ConfigurationManager.AppSettings["container"]);
	contain.CreateIfNotExists();

	// The Resolver object is used to interact with Key Vault for Azure Storage.
	// This is where the GetToken method from above is used.
	KeyVaultKeyResolver cloudResolver = new KeyVaultKeyResolver(GetToken);


> [AZURE.NOTE] 密钥保管库对象模型
>
>务必了解，实际上有两个密钥保管库对象模型：一个基于 REST API（KeyVault 命名空间），另一个是客户端加密的扩展。

> 密钥保管库客户端与 REST API 进行交互，并了解密钥保管库中包含的两种模型的 JSON Web 密钥和密码。

> 密钥保管库扩展似乎是专门为 Azure 存储空间中的客户端加密所创建的类。根据密钥解析程序的概念，它们包含密钥 (IKey) 和类的接口。您需要了解两种 IKey 实现：RSAKey 和 SymmetricKey。现在它们碰巧与密钥保管库中包含的内容保持一致，但此时它们是独立的类（因此，密钥保管库客户端检索到的密钥与秘密检索未实现 IKey）。


## 加密 blob 和上载
添加以下代码以加密 blob 并将其上载到 Azure 存储帐户。使用的 **ResolveKeyAsync** 方法会返回 IKey。


	// Retrieve the key that you created previously.
	// The IKey that is returned here is an RsaKey.
	// Remember that we used the names contosokeyvault and testrsakey1.
    var rsa = cloudResolver.ResolveKeyAsync("https://contosokeyvault.vault.chinacloudapi.cn/keys/TestRSAKey1", CancellationToken.None).GetAwaiter().GetResult();


	// Now you simply use the RSA key to encrypt by setting it in the BlobEncryptionPolicy.
	BlobEncryptionPolicy policy = new BlobEncryptionPolicy(rsa, null);
	BlobRequestOptions options = new BlobRequestOptions() { EncryptionPolicy = policy };

	// Reference a block blob.
	CloudBlockBlob blob = contain.GetBlockBlobReference("MyFile.txt");

	// Upload using the UploadFromStream method.
	using (var stream = System.IO.File.OpenRead(@"C:\data\MyFile.txt"))
		blob.UploadFromStream(stream, stream.Length, null, options, null);


下面是一个 blob 的 [Azure 管理门户](manage.windowsazure.cn)的屏幕截图，该 blob 已使用客户端加密通过密钥保管库中存储的密钥进行加密。**KeyId** 属性是密钥保管库中充当 KEK 的密钥的 URI。**EncryptedKey** 属性包含 CEK 的加密版本。

![显示包含加密元数据的 Blob 元数据的屏幕截图][1]

> [AZURE.NOTE]如果您看一下 BlobEncryptionPolicy 构造函数，您将看到它可以接受密钥和/或解析程序。请注意，您现在无法将解析程序用于加密，因为它当前不支持默认密钥。



## 解密 blob 并下载
当使用解析程序类有意义时，实际上就是解密。用于加密的密钥的 ID 与其元数据中的 Blob 相关联，因此你没有理由检索该密钥，请记住密钥与 blob 之间的关联关系。你只需确保该密钥保留在密钥保管库中。

RSA 密钥的私钥则保留在密钥保管库中，因此，为了进行解密，来自包含 CEK 的 blob 元数据的加密密钥将发送到密钥保管库进行解密。

添加以下代码以解密刚刚上载的 blob。

	// In this case, we will not pass a key and only pass the resolver because
	// this policy will only be used for downloading / decrypting.
	BlobEncryptionPolicy policy = new BlobEncryptionPolicy(null, cloudResolver);
	BlobRequestOptions options = new BlobRequestOptions() { EncryptionPolicy = policy };

    using (var np = File.Open(@"C:\data\MyFileDecrypted.txt", FileMode.Create))
	    blob.DownloadToStream(np, null, options, null);


> [AZURE.NOTE] 可以通过几个其他类型的解析程序来简化密钥管理，其中包括：AggregateKeyResolver 和 CachingKeyResolver。


## 使用密钥保管库密码
将密码用于客户端加密的方式是通过 SymmetricKey 类，因为密码实际上是一种对称密钥。但是，如上所述，密钥保管库中的密码不会完全映射到 SymmetricKey。这里要注意几个问题：


- SymmetricKey 中的密钥必须是固定长度：128、192、256、384 或 512 位。
- SymmetricKey 中的密钥应采用 Base64 编码。
- 用作 SymmetricKey 的密钥保管库密钥需要在密钥保管库中具有“application/octet-stream”内容类型。

以下是使用 PowerShell 在密钥保管库中创建可用作 SymmetricKey 的密钥的示例。注意：硬编码值 $key 仅用于演示目的。在你自己的代码中需要生成此密钥。

	// Here we are making a 128-bit key so we have 16 characters.
	// 	The characters are in the ASCII range of UTF8 so they are
	//	each 1 byte. 16 x 8 = 128.
	$key = "qwertyuiopasdfgh"
	$b = [System.Text.Encoding]::UTF8.GetBytes($key)
	$enc = [System.Convert]::ToBase64String($b)
	$secretvalue = ConvertTo-SecureString $enc -AsPlainText -Force

	// Substitute the VaultName and Name in this command.
	$secret = Set-AzureKeyVaultSecret -VaultName 'ContoseKeyVault' -Name 'TestSecret2' -SecretValue $secretvalue -ContentType "application/octet-stream"

在控制台应用程序中，可以使用与之前相同的调用将此密钥作为 SymmetricKey 进行检索。

	SymmetricKey sec = (SymmetricKey) cloudResolver.ResolveKeyAsync(
    	"https://contosokeyvault.vault.chinacloudapi.cn/secrets/TestSecret2/", 
        CancellationToken.None).GetAwaiter().GetResult();

就这么简单。请尽情享受其中的乐趣！

## 后续步骤

有关将 Azure 存储空间与 C# 配合使用的详细信息，请参阅[用于 .NET 的 Azure 存储客户端库](https://msdn.microsoft.com/zh-cn/library/azure/dn261237.aspx)。

有关 Blob REST API 的详细信息，请参阅 [Blob 服务 REST API](https://msdn.microsoft.com/zh-cn/library/azure/dd135733.aspx)。

有关 Azure 存储空间的最新信息，请转到 [Azure 存储空间团队博客](http://blogs.msdn.com/b/windowsazurestorage/)。


<!--Image references-->
[1]: ./media/storage-encrypt-decrypt-blobs-key-vault/blobmetadata.png

<!---HONumber=Mooncake_0509_2016-->