<properties linkid="manage-services-storage-net-shared-access-signature-part-2" urlDisplayName="" pageTitle="Create and use a SAS with the Blob Service | Microsoft Azure" metaKeywords="Azure blob, shared access signatures, stored access policy" description="Explore generating and using shared access signatures with the Blob service" metaCanonical="" services="storage" documentationCenter="" title="Part 2: Create and Use a SAS with the Blob Service" solutions="" authors="tamram" manager="mbaldwin" editor="cgronlun" />

# 共享访问签名，第 2 部分：创建 SAS 并将 SAS 用于 Blob 服务

本教程的[第 1 部分][]介绍了共享访问签名 (SAS) 并且说明了有关使用共享访问签名的最佳实践。第 2 部分将演示如何生成共享访问签名以及如何将共享访问签名用于 Azure Blob 服务。示例是用 C\# 编写的并使用了 Azure .NET 存储客户端库。涉及的方案包括使用共享访问签名的以下方面：

-   在容器上生成共享访问签名
-   在 Blob 上生成共享访问签名
-   创建存储访问策略来管理容器的资源上的签名
-   从客户端应用程序测试共享访问签名

# 关于本教程

在本教程中，我们将通过创建两个控制台应用程序重点说明如何为容器和 Blob 创建共享访问签名。第一个控制台应用程序将在一个容器和 Blob 上生成共享访问签名。该应用程序知道存储帐户密钥。第二个控制台应用程序（将充当客户端应用程序）使用第一个应用程序创建的共享访问签名访问容器和 Blob 资源。该应用程序仅使用共享访问签名验证其对容器和 Blob 资源的访问 – 它不知道帐户密钥。

# 第 1 部分：创建控制台应用程序以便生成共享访问签名

首先，确保你安装了 Azure .NET 存储客户端库（2.0 版）。你可以安装包含该客户端库的最新程序集的 [NuGet 程序包][]；这是确保你具有最新修补程序的建议方法。你还可以通过下载包含该客户端库的最新 [Azure SDK for .NET][] 版本来下载该客户端库。

在 Visual Studio 中，创建一个新的 Windows 控制台应用程序并将其命名为 **GenerateSharedAccessSignatures**。使用以下方法之一添加对 **Microsoft.WindowsAzure.Configuration.dll** 和 **Microsoft.WindowsAzure.Storage.dll** 的引用：

-   如果要安装 NuGet 程序包，请首先安装 [NuGet Package Manager Extension for Visual Studio][]。在 Visual Studio 中，选择**“项目”|“管理 NuGet 包”**，在线搜索**“Azure 存储空间”**，然后按照说明进行安装。
-   另外，还可以在你安装的 Azure SDK 中找到这些程序集，然后添加对它们的引用。

在 Program.cs 文件的顶部，添加以下 **using** 语句：

    using System.IO;    
    using Microsoft.WindowsAzure;
    using Microsoft.WindowsAzure.Storage;
    using Microsoft.WindowsAzure.Storage.Blob;

编辑 app.config 文件，使其所含配置设置中的连接字符串指向你的存储帐户。你的 app.config 文件应如下所示：

    <configuration>
    <startup> 
    <supportedRuntime version="v4.0" sku=".NETFramework,Version=v4.5" />
    </startup>
    <appSettings>
    <add key="StorageConnectionString" value="DefaultEndpointsProtocol=https;AccountName=myaccount;AccountKey=mykey"/>
    </appSettings> 
    </configuration>

## 为容器生成共享访问签名 URI

开始时，我们将添加一个方法以在新容器上生成共享访问签名。在本例中，该签名未与任何存储访问策略相关联，因此，它在 URI 上携带信息，指示其到期时间以及将授予的权限。

首先，向 **Main()** 方法中添加代码，以便验证对你的存储帐户的访问并且创建一个新容器：

    static void Main(string[] args)
    {
    //分析连接字符串并返回对存储帐户的引用。
    CloudStorageAccount storageAccount = CloudStorageAccount.Parse(CloudConfigurationManager.GetSetting("StorageConnectionString"));
        
    //创建 Blob 客户端对象。
    CloudBlobClient blobClient = storageAccount.CreateCloudBlobClient();
        
    //获取对将用于示例代码的容器的引用，如果该容器不存在，则创建它。
    CloudBlobContainer container = blobClient.GetContainerReference("sascontainer");
    container.CreateIfNotExists();
        
    //在此处插入对下方所创建方法的调用...
        
    //在关闭控制台窗口之前要求用户输入。
    Console.ReadLine();
    }

接下来，添加一个新方法，该方法为容器生成共享访问签名并且返回签名 URI：

    static string GetContainerSasUri(CloudBlobContainer container)
    {
    //设置容器的到期时间和权限。
    //在本例中未指定开始时间，因此共享访问签名将立即生效。
    SharedAccessBlobPolicy sasConstraints = new SharedAccessBlobPolicy();
    sasConstraints.SharedAccessExpiryTime = DateTime.UtcNow.AddHours(4);
    sasConstraints.Permissions = SharedAccessBlobPermissions.Write | SharedAccessBlobPermissions.List;
        
    //在容器上生成共享访问签名，直接在签名上设置约束。
    string sasContainerToken = container.GetSharedAccessSignature(sasConstraints);
        
    //返回容器的 URI 字符串，包括 SAS 令牌。
    return container.Uri + sasContainerToken;
    }

在 **Main()** 方法的底部，在对 **Console.ReadLine()** 进行调用之前，添加以下行以调用 **GetContainerSasUri()** 并将签名 URI 写入控制台窗口：

    //为容器生成 SAS URI，不使用存储访问策略。
    Console.WriteLine("Container SAS URI:" + GetContainerSasUri(container));
    Console.WriteLine();

编译并且运行以输出新容器的共享访问签名 URI。该 URI 将类似于以下 URI：

<https://storageaccount.blob.core.chinacloudapi.cn/sascontainer?sv=2012-02-12&se=2013-04-13T00%3A12%3A08Z&sr=c&sp=wl&sig=t%2BbzU9%2B7ry4okULN9S0wst%2F8MCUhTjrHyV9rDNLSe8g%3D>

在你运行代码后，你在容器上创建的共享访问签名将在接下来的四个小时内有效。该签名向客户端授予列出容器中的 Blob 以及将新 Blob 写入容器的权限。

## 为 Blob 生成共享访问签名 URI

接下来，我们将编写类似的代码，以在容器内创建新 Blob 并且为其生成共享访问签名。该共享访问签名不与任何存储访问策略相关联，因此，它在 URI 中包括了开始时间、到期时间和权限信息。

添加一个新方法，该方法创建一个新 Blob 并且向其写入某些文本，然后生成共享访问签名并且返回签名 URI：

    static string GetBlobSasUri(CloudBlobContainer container)
    {
    // 获取对容器内的某个 Blob 的引用。
    CloudBlockBlob blob = container.GetBlockBlobReference("sasblob.txt");
        
    //将文本上载到 Blob。如果 Blob 尚不存在，则会创建一个。 
    //如果 Blob 已存在，则将覆盖其现有内容。
    string blobContent = "This blob will be accessible to clients via a Shared Access Signature.";
    MemoryStream ms = new MemoryStream(Encoding.UTF8.GetBytes(blobContent));
    ms.Position = 0;
    using (ms)
        {
    blob.UploadFromStream(ms);
        }
        
    //设置 Blob 的到期时间和权限。
    //在本例中，开始时间设置为几分钟前，以减轻时钟偏移。
    //共享访问签名将立即生效。
    SharedAccessBlobPolicy sasConstraints = new SharedAccessBlobPolicy();
    sasConstraints.SharedAccessStartTime = DateTime.UtcNow.AddMinutes(-5);
    sasConstraints.SharedAccessExpiryTime = DateTime.UtcNow.AddHours(4);
    sasConstraints.Permissions = SharedAccessBlobPermissions.Read | SharedAccessBlobPermissions.Write;
        
    //在 Blob 上生成共享访问签名，直接在签名上设置约束。
    string sasBlobToken = blob.GetSharedAccessSignature(sasConstraints);
        
    //返回容器的 URI 字符串，包括 SAS 令牌。
    return blob.Uri + sasBlobToken;
    }

在 **Main()** 方法的底部，在对 **Console.ReadLine()** 进行调用之前，添加以下行以调用 **GetBlobSasUri()**，并将共享访问签名 URI 写入控制台窗口：

    //为容器内的某个 Blob 生成 SAS URI，不使用存储访问策略。
    Console.WriteLine("Blob SAS URI:" + GetBlobSasUri(container));
    Console.WriteLine();

编译并且运行以输出新 Blob 的共享访问签名 URI。该 URI 将类似于以下 URI：

<https://storageaccount.blob.core.chinacloudapi.cn/sascontainer/sasblob.txt?sv=2012-02-12&st=2013-04-12T23%3A37%3A08Z&se=2013-04-13T00%3A12%3A08Z&sr=b&sp=rw&sig=dF2064yHtc8RusQLvkQFPItYdeOz3zR8zHsDMBi4S30%3D>

## 在容器上创建存储访问策略

现在，让我们在容器上创建一个存储访问策略，该策略将定义与其关联的任何共享访问签名的约束。

在前面的示例中，我们在共享访问签名 URI 自身上指定了开始时间（隐式或显式）、到期时间以及权限。在接下来的示例中，我们将在存储访问策略上指定它们，而不是在共享访问签名上指定它们。这样做将使我们不必重新发布共享访问签名即可更改这些约束。

请注意，可以使一个或多个约束作用于共享访问签名上，使其余的约束作用于存储访问策略上。但是，你只能在这两者之一中指定开始时间、到期时间和权限；例如，不能既在共享访问签名上指定权限，又在存储访问策略上指定权限。

添加一个新方法，该方法创建一个新存储访问策略并且返回该策略的名称：

    static void CreateSharedAccessPolicy(CloudBlobClient blobClient, CloudBlobContainer container, string policyName)
    {
    //创建一个新的存储访问策略并定义其约束。
    SharedAccessBlobPolicy sharedPolicy = new SharedAccessBlobPolicy()
        {
    SharedAccessExpiryTime = DateTime.UtcNow.AddHours(10),
    Permissions = SharedAccessBlobPermissions.Read | SharedAccessBlobPermissions.Write | SharedAccessBlobPermissions.List
        };
        
    //获取容器的现有权限。
    BlobContainerPermissions permissions = new BlobContainerPermissions();
        
    //将新策略添加到容器的权限。
    permissions.SharedAccessPolicies.Clear();
    permissions.SharedAccessPolicies.Add(policyName, sharedPolicy);
    container.SetPermissions(permissions);
    }

在 **Main()** 方法的底部，在对 **Console.ReadLine()** 进行调用之前，添加以下行以调用 **CreateSharedAccessPolicy()** 方法：

    //在容器上创建一个访问策略，可以选择使用该策略为 
    //容器和 Blob 上的共享访问签名提供约束。
    string sharedAccessPolicyName = "tutorialpolicy";
    CreateSharedAccessPolicy(blobClient, container, sharedAccessPolicyName);

## 在使用访问策略的容器上生成共享访问签名 URI

接下来，我们将在之前创建的容器上创建另一个共享访问签名，但这次，我们要将该签名与我们在之前示例中创建的访问策略相关联。

添加一个新方法以便在容器上生成另一个共享访问签名：

    static string GetContainerSasUriWithPolicy(CloudBlobContainer container, string policyName)
    {
    //在容器上生成共享访问签名。在本例中，共享访问签名的所有约束 
    //都是在存储访问策略上指定的。
    string sasContainerToken = container.GetSharedAccessSignature(null, policyName);
        
    //返回容器的 URI 字符串，包括 SAS 令牌。
    return container.Uri + sasContainerToken;
    }

在 **Main()** 方法的底部，在对 **Console.ReadLine()** 进行调用之前，添加以下行以调用 **GetContainerSasUriWithPolicy** 方法：

    //为容器生成 SAS URI，使用存储访问策略在 SAS 上设置约束。
    Console.WriteLine("Container SAS URI using stored access policy:" + GetContainerSasUriWithPolicy(container, sharedAccessPolicyName));
    Console.WriteLine();

## 在使用访问策略的 Blob 上生成共享访问签名 URI

最后，我们将添加一个类似的方法，以便创建另一个 Blob 并且生成与某一访问策略相关联的共享访问签名。

添加一个新方法以便创建 Blob 并且生成共享访问签名：

    static string GetBlobSasUriWithPolicy(CloudBlobContainer container, string policyName)
    {
    // 获取对容器内的某个 Blob 的引用。
    CloudBlockBlob blob = container.GetBlockBlobReference("sasblobpolicy.txt");
        
    //将文本上载到 Blob。如果 Blob 尚不存在，则会创建一个。 
    //如果 Blob 已存在，则将覆盖其现有内容。
    string blobContent = "This blob will be accessible to clients via a shared access signature. " + 
    "A stored access policy defines the constraints for the signature.";
    MemoryStream ms = new MemoryStream(Encoding.UTF8.GetBytes(blobContent));
    ms.Position = 0;
    using (ms)
        {
    blob.UploadFromStream(ms);
        }
        
    //在 Blob 上生成共享访问签名。
    string sasBlobToken = blob.GetSharedAccessSignature(null, policyName);
        
    //返回容器的 URI 字符串，包括 SAS 令牌。
    return blob.Uri + sasBlobToken;
    }

在 **Main()** 方法的底部，在对 **Console.ReadLine()** 进行调用之前，添加以下行以调用 **GetBlobSasUriWithPolicy** 方法：

    //为容器内的某个 Blob 生成 SAS URI，使用存储访问策略在 SAS 上设置约束。
    Console.WriteLine("Blob SAS URI using stored access policy:" + GetBlobSasUriWithPolicy(container, sharedAccessPolicyName));
    Console.WriteLine();

现在，完整的 **Main()** 方法看起来应如下所示。运行它以将共享访问签名 URI 写入控制台窗口，然后将它们复制并粘贴到一个文本文件中以在本教程的第二部分中使用。

    static void Main(string[] args)
    {
    //分析连接字符串并返回对存储帐户的引用。
    CloudStorageAccount storageAccount = CloudStorageAccount.Parse(CloudConfigurationManager.GetSetting("StorageConnectionString"));
        
    //创建 Blob 客户端对象。
    CloudBlobClient blobClient = storageAccount.CreateCloudBlobClient();
        
    //获取对将用于示例代码的容器的引用，如果该容器不存在，则创建它。
    CloudBlobContainer container = blobClient.GetContainerReference("sascontainer");
    container.CreateIfNotExists();
        
    //为容器生成 SAS URI，不使用存储访问策略。
    Console.WriteLine("Container SAS URI:" + GetContainerSasUri(container));
    Console.WriteLine();
        
    //为容器内的某个 Blob 生成 SAS URI，不使用存储访问策略。
    Console.WriteLine("Blob SAS URI:" + GetBlobSasUri(container));
    Console.WriteLine();
        
    //在容器上创建一个访问策略，可以选择使用该策略为 
    //容器和 Blob 上的共享访问签名提供约束。
    string sharedAccessPolicyName = "tutorialpolicy";
    CreateSharedAccessPolicy(blobClient, container, sharedAccessPolicyName);
        
    //为容器生成 SAS URI，使用存储访问策略在 SAS 上设置约束。
    Console.WriteLine("Container SAS URI using stored access policy:" + GetContainerSasUriWithPolicy(container, sharedAccessPolicyName));
    Console.WriteLine();
        
    //为容器内的某个 Blob 生成 SAS URI，使用存储访问策略在 SAS 上设置约束。
    Console.WriteLine("Blob SAS URI using stored access policy:" + GetBlobSasUriWithPolicy(container, sharedAccessPolicyName));
    Console.WriteLine();
        
    Console.ReadLine();
    }

运行 GenerateSharedAccessSignatures 控制台应用程序时，你将会在控制台窗口中看到如下输出。它们是你在本教程的第 2 部分中将使用的共享访问签名。

![sas-console-output-1][]

# 第 2 部分：创建控制台应用程序来测试共享访问签名

为了测试在之前的示例中创建的共享访问签名，我们将创建另一个控制台应用程序，该应用程序将使用这些签名在容器和 Blob 上执行操作。

请注意，如果自你完成本教程的第一部分后已过去了 4 个多小时，则你生成的到期时间设置为 4 小时的签名将不再有效。同样，与存储访问策略相关联的签名将在 10 小时后到期。如果已过去了这两个时间间隔中的一个或两个，你应该在第一个控制台应用程序中运行代码，生成全新的可用于本教程第二部分的共享访问签名。

在 Visual Studio 中，创建一个新的 Windows 控制台应用程序并将其命名为 **ConsumeSharedAccessSignatures**。像之前所做的一样添加对 **Microsoft.WindowsAzure.Configuration.dll** 和 **Microsoft.WindowsAzure.Storage.dll** 的引用：

在 Program.cs 文件的顶部，添加以下 **using** 语句：

    using System.IO;
    using Microsoft.WindowsAzure.Storage;
    using Microsoft.WindowsAzure.Storage.Blob;

在 **Main()** 方法的正文中，添加以下约束，并且将其值更新为你在本教程的第 1 部分中生成的共享访问签名。

    static void Main(string[] args)
    {
    string containerSAS = "<your container SAS>";
    string blobSAS = "<your blob SAS>";
    string containerSASWithAccessPolicy = "<your container SAS with access policy>";
    string blobSASWithAccessPolicy = "<your blob SAS with access policy>";
    }

## 添加方法以便尝试使用共享访问签名执行容器操作

接下来，我们将添加一个方法，该方法将在容器上使用共享访问签名测试一些有代表性的容器操作。请注意，将使用共享访问签名返回对容器的引用，并且将单独基于该签名验证对容器的访问。

将以下方法添加到 Program.cs：

    static void UseContainerSAS(string sas)
    {
    //尝试使用所提供的 SAS 执行容器操作。

    //返回对使用 SAS URI 的容器的引用。
    CloudBlobContainer container = new CloudBlobContainer(new Uri(sas));

    //创建一个列表来存储对容器执行的列表操作返回的各个 Blob URI。
    List<Uri> blobUris = new List<Uri>();

    try
        {
    //写入操作：将一个新的 Blob 写入到容器中。 
    CloudBlockBlob blob = container.GetBlockBlobReference("blobCreatedViaSAS.txt");
    string blobContent = "This blob was created with a shared access signature granting write permissions to the container. ";
    MemoryStream msWrite = new MemoryStream(Encoding.UTF8.GetBytes(blobContent));
    msWrite.Position = 0;
    using (msWrite)
            {
    blob.UploadFromStream(msWrite);
            }
    Console.WriteLine("Write operation succeeded for SAS " + sas);
    Console.WriteLine();
        }
    catch (StorageException e)
        {
    Console.WriteLine("Write operation failed for SAS " + sas);
    Console.WriteLine("Additional error information:" + e.Message);
    Console.WriteLine();
        }

    try
        {
    //列表操作：列出容器中的 Blob，包括刚才添加的那个。
    foreach (ICloudBlob blobListing in container.ListBlobs())
            {
    blobUris.Add(blobListing.Uri);
            }
    Console.WriteLine("List operation succeeded for SAS " + sas);
    Console.WriteLine();
        }
    catch (StorageException e)
        {
    Console.WriteLine("List operation failed for SAS " + sas);
    Console.WriteLine("Additional error information:" + e.Message);
    Console.WriteLine();
        }

    try
        {
    //读取操作：获取对容器中的某个 Blob 的引用并对其进行读取。 
    CloudBlockBlob blob = container.GetBlockBlobReference(blobUris[0].ToString());
    MemoryStream msRead = new MemoryStream();
    msRead.Position = 0;
    using (msRead)
            {
    blob.DownloadToStream(msRead);
    Console.WriteLine(msRead.Length);
            }
    Console.WriteLine("Read operation succeeded for SAS " + sas);
    Console.WriteLine();
        }
    catch (StorageException e)
        {
    Console.WriteLine("Read operation failed for SAS " + sas);
    Console.WriteLine("Additional error information:" + e.Message);
    Console.WriteLine();
        }
    Console.WriteLine();

    try
        {
    //删除操作：删除容器中的 Blob。
    CloudBlockBlob blob = container.GetBlockBlobReference(blobUris[0].ToString());
    blob.Delete();
    Console.WriteLine("Delete operation succeeded for SAS " + sas);
    Console.WriteLine();
        }
    catch (StorageException e)
        {
    Console.WriteLine("Delete operation failed for SAS " + sas);
    Console.WriteLine("Additional error information:" + e.Message);
    Console.WriteLine();
        }        
    }

更新 **Main()** 方法以使用你在容器上创建的两个共享访问签名调用 **UseContainerSAS()**：

    static void Main(string[] args)
    {
    string containerSAS = "<your container SAS>";
    string blobSAS = "<your blob SAS>";
    string containerSASWithAccessPolicy = "<your container SAS with access policy>";
    string blobSASWithAccessPolicy = "<your blob SAS with access policy>";

    //使用在容器上创建的共享访问签名调用测试方法，可以使用也可以不使用访问策略。
    UseContainerSAS(containerSAS);
    UseContainerSAS(containerSASWithAccessPolicy); 
        
    //使用在 Blob 上创建的共享访问签名调用测试方法，可以使用也可以不使用访问策略。
    UseBlobSAS(blobSAS);
    UseBlobSAS(blobSASWithAccessPolicy);

    Console.ReadLine();
    }

## 添加方法以便尝试使用共享访问签名执行 Blob 操作

最后，我们将添加一个方法，该方法将在 Blob 上使用共享访问签名测试一些有代表性的 Blob 操作。在这个例子中，我们使用在共享访问签名中传入的构造函数 **CloudBlockBlob(String)** 返回对该 Blob 的引用。无需其他身份验证；它仅基于签名。

将以下方法添加到 Program.cs：

    static void UseBlobSAS(string sas)
    {
    //尝试使用所提供的 SAS 执行 Blob 操作。

    //返回对使用 SAS URI 的 Blob 的引用。
    CloudBlockBlob blob = new CloudBlockBlob(new Uri(sas));

    try
        {
    //写入操作：将一个新的 Blob 写入到容器中。 
    string blobContent = "This blob was created with a shared access signature granting write permissions to the blob. ";
    MemoryStream msWrite = new MemoryStream(Encoding.UTF8.GetBytes(blobContent));
    msWrite.Position = 0;
    using (msWrite)
            {
    blob.UploadFromStream(msWrite);
            }
    Console.WriteLine("Write operation succeeded for SAS " + sas);
    Console.WriteLine();
        }
    catch (StorageException e)
        {
    Console.WriteLine("Write operation failed for SAS " + sas);
    Console.WriteLine("Additional error information:" + e.Message);
    Console.WriteLine();
        }

    try
        {
    //读取操作：读取 Blob 的内容。
    MemoryStream msRead = new MemoryStream();
    using (msRead)
            {
    blob.DownloadToStream(msRead);
    msRead.Position = 0;
    using (StreamReader reader = new StreamReader(msRead, true))
                {
    string line;
    while ((line = reader.ReadLine()) != null)
                    {
    Console.WriteLine(line);
                    }
                }
            }
    Console.WriteLine("Read operation succeeded for SAS " + sas);
    Console.WriteLine();
        }
    catch (StorageException e)
        {
    Console.WriteLine("Read operation failed for SAS " + sas);
    Console.WriteLine("Additional error information:" + e.Message);
    Console.WriteLine();
        }

    try
        {
    //删除操作：删除 Blob。
    blob.Delete();
    Console.WriteLine("Delete operation succeeded for SAS " + sas);
    Console.WriteLine();
        }
    catch (StorageException e)
        {
    Console.WriteLine("Delete operation failed for SAS " + sas);
    Console.WriteLine("Additional error information:" + e.Message);
    Console.WriteLine();
        }        
    }

更新 **Main()** 方法以使用你在 Blob 上创建的两个共享访问签名调用 **UseBlobSAS()**：

    static void Main(string[] args)
    {
    string containerSAS = "<your container SAS>";
    string blobSAS = "<your blob SAS>";
    string containerSASWithAccessPolicy = "<your container SAS with access policy>";
    string blobSASWithAccessPolicy = "<your blob SAS with access policy>";

    //使用在容器上创建的共享访问签名调用测试方法，可以使用也可以不使用访问策略。
    UseContainerSAS(containerSAS);
    UseContainerSAS(containerSASWithAccessPolicy); 
        
    //使用在 Blob 上创建的共享访问签名调用测试方法，可以使用也可以不使用访问策略。
    UseBlobSAS(blobSAS);
    UseBlobSAS(blobSASWithAccessPolicy);

    Console.ReadLine();
    }

运行该控制台应用程序并观察输出，查看对各个签名允许的操作。控制台窗口中的输出将与下面的输出类似：

![sas-console-output-2][]

# 后续步骤

[共享访问签名，第 1 部分：了解 SAS 模型][第 1 部分]

[管理对 Azure 存储资源的访问][]

[使用共享访问签名委托访问 (REST API)][]

[介绍表和队列 SAS][]

  [第 1 部分]: ../storage-dotnet-shared-access-signature-part-1/
  [NuGet 程序包]: http://nuget.org/packages/WindowsAzure.Storage/ "NuGet 程序包"
  [Azure SDK for .NET]: http://www.windowsazure.cn/zh-cn/downloads/?sdk=net
  [NuGet Package Manager Extension for Visual Studio]: http://visualstudiogallery.msdn.microsoft.com/27077b70-9dad-4c64-adcf-c7cf6bc9970c
  [sas-console-output-1]: ./media/storage-dotnet-shared-access-signature-part-2/sas-console-output-1.PNG
  [sas-console-output-2]: ./media/storage-dotnet-shared-access-signature-part-2/sas-console-output-2.PNG
  [管理对 Azure 存储资源的访问]: http://msdn.microsoft.com/zh-cn/library/azure/ee393343.aspx
  [使用共享访问签名委托访问 (REST API)]: http://msdn.microsoft.com/zh-cn/library/azure/ee395415.aspx
  [介绍表和队列 SAS]: http://blogs.msdn.com/b/windowsazurestorage/archive/2012/06/12/introducing-table-sas-shared-access-signature-queue-sas-and-update-to-blob-sas.aspx
