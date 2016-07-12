<properties 
	pageTitle="创建 SAS 并将 SAS 用于 Blob 存储 | Azure" 
	description="本教程演示如何创建共享访问签名以用于 Blob 存储，以及如何从客户端应用程序使用这些签名。"
	services="storage" 
	documentationCenter="" 
	authors="tamram" 
	manager="adinah" 
	editor="cgronlun"/>

<tags 
	ms.service="storage" 
	ms.date="02/14/2016"
	wacn.date="04/11/2016"/>



# 共享访问签名，第 2 部分：创建 SAS 并将 SAS 用于 Blob 存储

## 概述

本教程的[第 1 部分](/documentation/articles/storage-dotnet-shared-access-signature-part-1/)介绍了共享访问签名 (SAS) 并且说明了有关使用共享访问签名的最佳实践。第 2 部分将演示如何生成共享访问签名以及如何将共享访问签名用于 Blob 存储。示例是用 C# 编写的并使用了 Azure .NET 存储客户端库。涉及的任务包括使用共享访问签名的以下方面：

- 在容器上生成共享访问签名
- 在 Blob 上生成共享访问签名
- 创建用于管理容器资源上的签名的存储访问策略
- 从客户端应用程序测试共享访问签名

## 关于本教程

在本教程中，我们将通过创建两个控制台应用程序重点说明如何为容器和 Blob 创建共享访问签名。第一个控制台应用程序将在一个容器和 Blob 上生成共享访问签名。该应用程序知道存储帐户密钥。第二个控制台应用程序（将充当客户端应用程序）使用第一个应用程序创建的共享访问签名访问容器和 Blob 资源。该应用程序仅使用共享访问签名验证其对容器和 Blob 资源的访问 – 它不知道帐户密钥。

## 第 1 部分：创建控制台应用程序以便生成共享访问签名

首先，确保你安装了 Azure .NET 存储客户端库。你可以安装包含该客户端库的最新程序集的 [NuGet 程序包](http://nuget.org/packages/WindowsAzure.Storage/ "NuGet 包")；这是确保你具有最新修补程序的建议方法。你还可以通过下载包含该客户端库的最新 [Azure SDK for .NET](/downloads/) 版本来下载该客户端库。

在 Visual Studio 中，创建一个新的 Windows 控制台应用程序并将其命名为 **GenerateSharedAccessSignatures**。使用以下方法之一添加对 **Microsoft.WindowsAzure.Configuration.dll** 和 **Microsoft.WindowsAzure.Storage.dll** 的引用：

- 	如果你想要安装 NuGet 程序包，请首先安装 [NuGet Package Manager Extension for Visual Studio](http://visualstudiogallery.msdn.microsoft.com/27077b70-9dad-4c64-adcf-c7cf6bc9970c)。在 Visual Studio 中，选择“项目”|“管理 NuGet 包”，在线搜索“Azure 存储空间”，然后按照说明进行安装。
- 	另外，还可以在你安装的 Azure SDK 中找到这些程序集，然后添加对它们的引用。

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
        <add key="StorageConnectionString" value="DefaultEndpointsProtocol=https;AccountName=myaccount;AccountKey=mykey;EndpointSuffix=core.chinacloudapi.cn"/>
      </appSettings>
    </configuration>

### 为容器生成共享访问签名 URI

开始时，我们将添加一个方法以便在新容器上生成共享访问签名。在这个例子中，该签名不与某一存储访问策略相关联，因此，它在 URI 上携带信息，指示其到期时间以及将授予的权限。

首先，向 **Main()** 方法中添加代码，以便验证对你的存储帐户的访问并且创建一个新容器：

    static void Main(string[] args)
    {
	    //Parse the connection string and return a reference to the storage account.
	    CloudStorageAccount storageAccount = CloudStorageAccount.Parse(CloudConfigurationManager.GetSetting("StorageConnectionString"));

	    //Create the blob client object.
	    CloudBlobClient blobClient = storageAccount.CreateCloudBlobClient();

	    //Get a reference to a container to use for the sample code, and create it if it does not exist.
	    CloudBlobContainer container = blobClient.GetContainerReference("sascontainer");
	    container.CreateIfNotExists();

	    //Insert calls to the methods created below here...

	    //Require user input before closing the console window.
	    Console.ReadLine();
    }

接下来，添加一个新方法，该方法为容器生成共享访问签名并且返回签名 URI：

    static string GetContainerSasUri(CloudBlobContainer container)
    {
	    //Set the expiry time and permissions for the container.
	    //In this case no start time is specified, so the shared access signature becomes valid immediately.
	    SharedAccessBlobPolicy sasConstraints = new SharedAccessBlobPolicy();
	    sasConstraints.SharedAccessExpiryTime = DateTime.UtcNow.AddHours(24);
	    sasConstraints.Permissions = SharedAccessBlobPermissions.Write | SharedAccessBlobPermissions.List;

	    //Generate the shared access signature on the container, setting the constraints directly on the signature.
	    string sasContainerToken = container.GetSharedAccessSignature(sasConstraints);

	    //Return the URI string for the container, including the SAS token.
	    return container.Uri + sasContainerToken;
    }

在 **Main()** 方法的底部，在调用 **Console.ReadLine()** 之前，添加以下代码行以调用 **GetContainerSasUri()** 并将签名 URI 写入控制台窗口：

    //Generate a SAS URI for the container, without a stored access policy.
    Console.WriteLine("Container SAS URI: " + GetContainerSasUri(container));
    Console.WriteLine();

编译并且运行以输出新容器的共享访问签名 URI。该 URI 将类似以下 URI：

	https://storageaccount.blob.core.chinacloudapi.cn/sascontainer?sv=2012-02-12&se=2013-04-13T00%3A12%3A08Z&sr=c&sp=wl&sig=t%2BbzU9%2B7ry4okULN9S0wst%2F8MCUhTjrHyV9rDNLSe8g%3D

在你运行代码后，你在容器上创建的共享访问签名将在接下来的 24 个小时内有效。该签名向客户端授予列出容器中的 Blob 以及将新 Blob 写入容器的权限。

### 为 Blob 生成共享访问签名 URI

接下来，我们将编写类似的代码，以便在容器内创建新 Blob 并且为其生成共享访问签名。该共享访问签名不与某一存储访问策略相关联，因此，它包含有关 URI 的开始时间、到期时间和权限。

添加一个新方法，该方法创建一个新 Blob 并且向其写入某些文本，然后生成共享访问签名并且返回签名 URI：

    static string GetBlobSasUri(CloudBlobContainer container)
    {
	    //Get a reference to a blob within the container.
	    CloudBlockBlob blob = container.GetBlockBlobReference("sasblob.txt");

	    //Upload text to the blob. If the blob does not yet exist, it will be created.
	    //If the blob does exist, its existing content will be overwritten.
	    string blobContent = "This blob will be accessible to clients via a Shared Access Signature.";
	    MemoryStream ms = new MemoryStream(Encoding.UTF8.GetBytes(blobContent));
	    ms.Position = 0;
	    using (ms)
	    {
		    blob.UploadFromStream(ms);
	    }

	    //Set the expiry time and permissions for the blob.
	    //In this case the start time is specified as a few minutes in the past, to mitigate clock skew.
	    //The shared access signature will be valid immediately.
	    SharedAccessBlobPolicy sasConstraints = new SharedAccessBlobPolicy();
	    sasConstraints.SharedAccessStartTime = DateTime.UtcNow.AddMinutes(-5);
	    sasConstraints.SharedAccessExpiryTime = DateTime.UtcNow.AddHours(24);
	    sasConstraints.Permissions = SharedAccessBlobPermissions.Read | SharedAccessBlobPermissions.Write;

	    //Generate the shared access signature on the blob, setting the constraints directly on the signature.
	    string sasBlobToken = blob.GetSharedAccessSignature(sasConstraints);

	    //Return the URI string for the container, including the SAS token.
	    return blob.Uri + sasBlobToken;
    }

在 **Main()** 方法的底部，在调用 **Console.ReadLine()** 之前，添加以下代码行以调用 **GetBlobSasUri()**，并将共享访问签名 URI 写入控制台窗口：

    //Generate a SAS URI for a blob within the container, without a stored access policy.
    Console.WriteLine("Blob SAS URI: " + GetBlobSasUri(container));
    Console.WriteLine();


编译并且运行以输出新 Blob 的共享访问签名 URI。该 URI 将类似以下 URI：

	https://storageaccount.blob.core.chinacloudapi.cn/sascontainer/sasblob.txt?sv=2012-02-12&st=2013-04-12T23%3A37%3A08Z&se=2013-04-13T00%3A12%3A08Z&sr=b&sp=rw&sig=dF2064yHtc8RusQLvkQFPItYdeOz3zR8zHsDMBi4S30%3D

### 在容器上创建存储访问策略

现在来在容器上创建一个存储访问策略，该策略定义与其相关联的任何共享访问签名的约束。

在前面的示例中，我们指定了开始时间（隐式或显式的）、到期时间以及共享访问签名 URI 本身的权限。在接下来的示例中，我们将在存储访问策略上指定它们，而不是在共享访问签名上指定它们。这样做将使我们不必重新发布共享访问签名即可更改这些约束。

可以使一个或多个约束作用于共享访问签名上，使其余的约束作用于存储访问策略上。但是，你只能在这两者之一中指定开始时间、到期时间和权限；例如，不能对共享访问签名指定权限，同时也对存储访问策略指定权限。

注意，当你向容器添加一个访问策略时，你必须获取容器的现有权限，添加新的访问策略，然后设置容器的权限。

添加一个新方法，该方法在窗口上创建一个新的存储访问策略并返回该策略的名称：

    static void CreateSharedAccessPolicy(CloudBlobClient blobClient, CloudBlobContainer container,
		string policyName)
    {
        //Create a new shared access policy and define its constraints.
        SharedAccessBlobPolicy sharedPolicy = new SharedAccessBlobPolicy()
        {
            SharedAccessExpiryTime = DateTime.UtcNow.AddHours(24),
            Permissions = SharedAccessBlobPermissions.Write | SharedAccessBlobPermissions.List | SharedAccessBlobPermissions.Read
        };

        //Get the container's existing permissions.
        BlobContainerPermissions permissions = container.GetPermissions();

        //Add the new policy to the container's permissions, and set the container's permissions.
        permissions.SharedAccessPolicies.Add(policyName, sharedPolicy);
        container.SetPermissions(permissions);
    }

在 **Main()** 方法的底部，在调用 **Console.ReadLine()** 之前，添加以下代码行以首先清除任何现有访问策略，然后调用 **CreateSharedAccessPolicy()** 方法：

    //Clear any existing access policies on container.
    BlobContainerPermissions perms = container.GetPermissions();
    perms.SharedAccessPolicies.Clear();
    container.SetPermissions(perms);

    //Create a new access policy on the container, which may be optionally used to provide constraints for
    //shared access signatures on the container and the blob.
    string sharedAccessPolicyName = "tutorialpolicy";
    CreateSharedAccessPolicy(blobClient, container, sharedAccessPolicyName);

注意，当清除容器上的访问策略时，你必须首先获取容器的现有权限，接着清除权限，然后再次设置权限。

### 在使用访问策略的容器上生成共享访问签名 URI

接下来，我们将在之前创建的容器上创建另一个共享访问签名，但这次，我们要将该签名与我们在之前示例中创建的访问策略相关联。

添加一个新方法以便在容器上生成另一个共享访问签名：

    static string GetContainerSasUriWithPolicy(CloudBlobContainer container, string policyName)
    {
	    //Generate the shared access signature on the container. In this case, all of the constraints for the
	    //shared access signature are specified on the stored access policy.
	    string sasContainerToken = container.GetSharedAccessSignature(null, policyName);

	    //Return the URI string for the container, including the SAS token.
	    return container.Uri + sasContainerToken;
    }

在 **Main()** 方法的底部，在调用 **Console.ReadLine()** 之前，添加以下代码行以调用 **GetContainerSasUriWithPolicy** 方法：

    //Generate a SAS URI for the container, using a stored access policy to set constraints on the SAS.
    Console.WriteLine("Container SAS URI using stored access policy: " + GetContainerSasUriWithPolicy(container, sharedAccessPolicyName));
    Console.WriteLine();

### 在使用访问策略的 Blob 上生成共享访问签名 URI

最后，我们将添加一个类似方法，以便创建另一个 Blob 并且生成与某一访问策略相关联的共享访问签名。

添加一个新方法以便创建 Blob 并且生成共享访问签名：

    static string GetBlobSasUriWithPolicy(CloudBlobContainer container, string policyName)
    {
	    //Get a reference to a blob within the container.
	    CloudBlockBlob blob = container.GetBlockBlobReference("sasblobpolicy.txt");

	    //Upload text to the blob. If the blob does not yet exist, it will be created.
	    //If the blob does exist, its existing content will be overwritten.
	    string blobContent = "This blob will be accessible to clients via a shared access signature. " +
	    "A stored access policy defines the constraints for the signature.";
	    MemoryStream ms = new MemoryStream(Encoding.UTF8.GetBytes(blobContent));
	    ms.Position = 0;
	    using (ms)
	    {
		    blob.UploadFromStream(ms);
	    }

	    //Generate the shared access signature on the blob.
	    string sasBlobToken = blob.GetSharedAccessSignature(null, policyName);

	    //Return the URI string for the container, including the SAS token.
	    return blob.Uri + sasBlobToken;
    }

在 **Main()** 方法的底部，在调用 **Console.ReadLine()** 之前，添加以下代码行以调用 **GetBlobSasUriWithPolicy** 方法：

    //Generate a SAS URI for a blob within the container, using a stored access policy to set constraints on the SAS.
    Console.WriteLine("Blob SAS URI using stored access policy: " + GetBlobSasUriWithPolicy(container, sharedAccessPolicyName));
    Console.WriteLine();

现在，完整的 **Main()** 方法看起来应如下所示。运行它以将共享访问签名 URI 写入控制台窗口，然后将它们复制并粘贴到一个文本文件中以在本教程的第二部分中使用。

    static void Main(string[] args)
    {
	    //Parse the connection string and return a reference to the storage account.
	    CloudStorageAccount storageAccount = CloudStorageAccount.Parse(CloudConfigurationManager.GetSetting("StorageConnectionString"));

	    //Create the blob client object.
	    CloudBlobClient blobClient = storageAccount.CreateCloudBlobClient();

	    //Get a reference to a container to use for the sample code, and create it if it does not exist.
	    CloudBlobContainer container = blobClient.GetContainerReference("sascontainer");
	    container.CreateIfNotExists();

	    //Generate a SAS URI for the container, without a stored access policy.
	    Console.WriteLine("Container SAS URI: " + GetContainerSasUri(container));
	    Console.WriteLine();

	    //Generate a SAS URI for a blob within the container, without a stored access policy.
	    Console.WriteLine("Blob SAS URI: " + GetBlobSasUri(container));
	    Console.WriteLine();

        //Clear any existing access policies on container.
        BlobContainerPermissions perms = container.GetPermissions();
        perms.SharedAccessPolicies.Clear();
        container.SetPermissions(perms);

        //Create a new access policy on the container, which may be optionally used to provide constraints for
        //shared access signatures on the container and the blob.
	    string sharedAccessPolicyName = "tutorialpolicy";
	    CreateSharedAccessPolicy(blobClient, container, sharedAccessPolicyName);

	    //Generate a SAS URI for the container, using a stored access policy to set constraints on the SAS.
	    Console.WriteLine("Container SAS URI using stored access policy: " + GetContainerSasUriWithPolicy(container, sharedAccessPolicyName));
	    Console.WriteLine();

	    //Generate a SAS URI for a blob within the container, using a stored access policy to set constraints on the SAS.
	    Console.WriteLine("Blob SAS URI using stored access policy: " + GetBlobSasUriWithPolicy(container, sharedAccessPolicyName));
	    Console.WriteLine();

	    Console.ReadLine();
    }

运行 GenerateSharedAccessSignatures 控制台应用程序时，你将会在控制台窗口中看到如下输出。它们是你在本教程的第 2 部分中将使用的共享访问签名。

![sas-console-output-1][sas-console-output-1]

## 第 2 部分：创建控制台应用程序来测试共享访问签名

为了测试在之前的示例中创建的共享访问签名，我们将创建第二个控制台应用程序，该应用程序将使用这些签名在容器和 Blob 上执行操作。

> [AZURE.NOTE] 如果自你完成本教程的第一部分后超过 24 小时，则你生成的签名将不再有效。在这种情况下，你应该在第一个控制台应用程序中运行代码，生成全新的共享访问签名以供在本教程的第二部分使用。

在 Visual Studio 中，创建一个新的 Windows 控制台应用程序并将其命名为 **ConsumeSharedAccessSignatures**。像之前所做的一样添加对 **Microsoft.WindowsAzure.Configuration.dll** 和 **Microsoft.WindowsAzure.Storage.dll** 的引用：

在 Program.cs 文件的顶部，添加以下 **using** 语句：

    using System.IO;
    using Microsoft.WindowsAzure.Storage;
    using Microsoft.WindowsAzure.Storage.Blob;

在 **Main()** 方法的主体中，添加以下约束，并且将其值更新为你在本教程的第 1 部分中生成的共享访问签名。

    static void Main(string[] args)
    {
	    string containerSAS = "<your container SAS>";
	    string blobSAS = "<your blob SAS>";
	    string containerSASWithAccessPolicy = "<your container SAS with access policy>";
	    string blobSASWithAccessPolicy = "<your blob SAS with access policy>";
    }

### 添加方法以便尝试使用共享访问签名执行容器操作

接下来，我们将添加一个方法，该方法将在容器上使用共享访问签名测试一些有代表性的容器操作。请注意，使用共享访问签名返回对容器的引用，并且单独基于该签名验证对容器的访问。

将以下方法添加到 Program.cs：

    static void UseContainerSAS(string sas)
    {
        //Try performing container operations with the SAS provided.

        //Return a reference to the container using the SAS URI.
        CloudBlobContainer container = new CloudBlobContainer(new Uri(sas));

        //Create a list to store blob URIs returned by a listing operation on the container.
        List<ICloudBlob> blobList = new List<ICloudBlob>();

        //Write operation: write a new blob to the container.
        try
        {
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
            Console.WriteLine("Additional error information: " + e.Message);
            Console.WriteLine();
        }

        //List operation: List the blobs in the container.
        try
        {
            foreach (ICloudBlob blob in container.ListBlobs())
            {
                blobList.Add(blob);
            }
            Console.WriteLine("List operation succeeded for SAS " + sas);
            Console.WriteLine();
        }
        catch (StorageException e)
        {
            Console.WriteLine("List operation failed for SAS " + sas);
            Console.WriteLine("Additional error information: " + e.Message);
            Console.WriteLine();
        }

        //Read operation: Get a reference to one of the blobs in the container and read it.
        try
        {
            CloudBlockBlob blob = container.GetBlockBlobReference(blobList[0].Name);
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
            Console.WriteLine("Additional error information: " + e.Message);
            Console.WriteLine();
        }
        Console.WriteLine();

        //Delete operation: Delete a blob in the container.
        try
        {
            CloudBlockBlob blob = container.GetBlockBlobReference(blobList[0].Name);
            blob.Delete();
            Console.WriteLine("Delete operation succeeded for SAS " + sas);
            Console.WriteLine();
        }
        catch (StorageException e)
        {
            Console.WriteLine("Delete operation failed for SAS " + sas);
            Console.WriteLine("Additional error information: " + e.Message);
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

	    //Call the test methods with the shared access signatures created on the container, with and without the access policy.
	    UseContainerSAS(containerSAS);
	    UseContainerSAS(containerSASWithAccessPolicy);

	    Console.ReadLine();
	}


### 添加方法以便尝试使用共享访问签名执行 Blob 操作

最后，我们将添加一个方法，该方法将在 Blob 上使用共享访问签名测试一些有代表性的 Blob 操作。在这个例子中，我们使用在共享访问签名中传入的构造函数 **CloudBlockBlob(String)** 返回对该 Blob 的引用。无需其他身份验证；它仅基于签名。

将以下方法添加到 Program.cs：

    static void UseBlobSAS(string sas)
    {
        //Try performing blob operations using the SAS provided.

        //Return a reference to the blob using the SAS URI.
        CloudBlockBlob blob = new CloudBlockBlob(new Uri(sas));

        //Write operation: Write a new blob to the container.
        try
        {
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
            Console.WriteLine("Additional error information: " + e.Message);
            Console.WriteLine();
        }

        //Read operation: Read the contents of the blob.
        try
        {
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
            Console.WriteLine("Additional error information: " + e.Message);
            Console.WriteLine();
        }

        //Delete operation: Delete the blob.
        try
        {
            blob.Delete();
            Console.WriteLine("Delete operation succeeded for SAS " + sas);
            Console.WriteLine();
        }
        catch (StorageException e)
        {
            Console.WriteLine("Delete operation failed for SAS " + sas);
            Console.WriteLine("Additional error information: " + e.Message);
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

	    //Call the test methods with the shared access signatures created on the container, with and without the access policy.
	    UseContainerSAS(containerSAS);
	    UseContainerSAS(containerSASWithAccessPolicy);

	    //Call the test methods with the shared access signatures created on the blob, with and without the access policy.
	    UseBlobSAS(blobSAS);
	    UseBlobSAS(blobSASWithAccessPolicy);

	    Console.ReadLine();
	}

运行该控制台应用程序并观察输出，查看对各个签名允许的操作。控制台窗口中的输出将与下面的输出类似：

![sas-console-output-2][sas-console-output-2]

## 后续步骤

[共享访问签名，第 1 部分：了解 SAS 模型](/documentation/articles/storage-dotnet-shared-access-signature-part-1/)

[管理对容器和 blob 的匿名读取访问](/documentation/articles/storage-manage-access-to-resources/)

[使用共享访问签名委托访问 (REST API)](http://msdn.microsoft.com/zh-cn/library/azure/ee395415.aspx)

[介绍表和队列 SAS](http://blogs.msdn.com/b/windowsazurestorage/archive/2012/06/12/introducing-table-sas-shared-access-signature-queue-sas-and-update-to-blob-sas.aspx)

[sas-console-output-1]: ./media/storage-dotnet-shared-access-signature-part-2/sas-console-output-1.PNG
[sas-console-output-2]: ./media/storage-dotnet-shared-access-signature-part-2/sas-console-output-2.PNG


<!---HONumber=Mooncake_0405_2016-->