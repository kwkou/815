<properties
	pageTitle="使用 Blob 存储的本地应用程序 (Java) | Azure"
	description="了解如何创建将图像上载到 Azure 并在浏览器中显示图像的控制台应用程序。使用 Java 的代码示例。"
	services="storage"
	documentationCenter="java"
	authors="rmcmurray"
	manager="wpickett"
	editor="jimbe"/>

<tags
	ms.service="storage"
	ms.date="04/08/2016"
	wacn.date="05/23/2016"/>

# 使用 Blob 存储的本地应用程序

## 概述

以下示例将演示如何使用 Azure 存储空间在 Azure 中存储图像。本文中的代码是一个控制台应用程序的代码，该应用程序将一个图像上载到 Azure，然后创建用于在浏览器中显示该图像的 HTML 文件。

## 先决条件

- 已安装 Java 开发人员工具包 (JDK) 版本1.6 或更高版本。
- 已安装 Azure SDK。
- 适用于 Azure Libraries for Java 的 JAR 以及任何适用的依赖项 JAR 已安装并且位于 Java 编译器使用的生成路径中。有关安装 Azure Libraries for Java 的信息，请参阅[下载 Azure SDK for Java](/documentation/articles/java-download-azure-sdk/)。
- 已设置了一个 Azure 存储帐户。本文中的代码将使用存储帐户的帐户名称和帐户密钥。有关创建存储帐户的信息，请参阅[如何创建存储帐户](/documentation/articles/storage-create-storage-account/#create-a-storage-account)；有关检索帐户密钥的信息，请参阅[查看并复制存储访问密钥](/documentation/articles/storage-create-storage-account/#view-and-copy-storage-access-keys)。

- 您已创建存储在路径 c:\\myimages\\image1.jpg 处的已命名本地图像文件。或者，在示例中修改 **FileInputStream** 构造函数以使用其他图像路径和文件名。

[AZURE.INCLUDE [create-account-note](../includes/create-account-note.md)]

## 使用 Azure Blob 存储上载文件

此处提供分步过程。如果你要跳过这些过程，则可在本文后面找到完整的代码。

在代码的开头请包括对 Azure 核心存储类、Azure Blob 客户端类、Java IO 类和 **URISyntaxException** 类的导入：

    import com.microsoft.azure.storage.*;
    import com.microsoft.azure.storage.blob.*;
    import java.io.*;
    import java.net.URISyntaxException;

声明一个名为 **StorageSample** 的类，包含左大括号 **{**。

    public class StorageSample {

在 StorageSample 类中，声明一个将包含**默认终结点协议、你的存储帐户名称和存储访问密钥**（在你的 Azure 存储帐户中指定）的字符串变量。将占位符值 **your_account_name** 和
**your_account_key** 分别替换为你自己的帐户名称和帐户密钥。

    public static final String storageConnectionString =
           "DefaultEndpointsProtocol=http;" +
               "AccountName=your_account_name;" +
               "AccountKey=your_account_name;" +
	       "EndpointSuffix=core.chinacloudapi.cn";

添加对 **main** 的声明，包括 **try** 块并包括必需的左大括号 **{**。

    public static void main(String[] args)
    {
        try
        {

声明以下类型的变量（说明针对的是如何在此示例中使用这些变量）：

-   **CloudStorageAccount**：用于通过 Azure 存储帐户名称和密钥初始化帐户对象，以及用于创建 Blob 客户端对象。
-   **CloudBlobClient**：用于访问 Blob 服务。
-   **CloudBlobContainer**：用于创建 Blob 容器、列出容器中的 Blob 以及删除容器。
-   **CloudBlockBlob**：用于将本地图像文件上载到容器。

<!-- -->

    CloudStorageAccount account;
    CloudBlobClient serviceClient;
    CloudBlobContainer container;
    CloudBlockBlob blob;

为 **account** 变量赋值。

    account = CloudStorageAccount.parse(storageConnectionString);

为 **serviceClient** 变量赋值。

    serviceClient = account.createCloudBlobClient();

为 **container** 变量赋值。我们将获取对名为 **gettingstarted** 的容器的引用。

    // Container name must be lower case.
    container = serviceClient.getContainerReference("gettingstarted");

创建该容器。如果该容器不存在，此方法将创建该容器（并返回 true）。如果该容器存在，则此方法将返回
**false**。createIfNotExist 的一个替代方法是 create 方法（如果该容器已存在，该方法将返回错误）。

    container.createIfNotExists();

为容器设置匿名访问。

    // Set anonymous access on the container.
    BlobContainerPermissions containerPermissions;
    containerPermissions = new BlobContainerPermissions();
    containerPermissions.setPublicAccess(BlobContainerPublicAccessType.CONTAINER);
    container.uploadPermissions(containerPermissions);

获取对块 Blob 的引用，它将表示
Azure 存储空间中的 Blob。

    blob = container.getBlockBlobReference("image1.jpg");

使用 **File** 构造函数获取对你将上载的在本地创建的文件的引用。确保在运行代码之前已创建此文件。

    File fileReference = new File ("c:\\myimages\\image1.jpg");

通过调用 CloudBlockBlob.upload 方法上载该本地文件。**CloudBlockBlob.upload** 方法的第一个参数为
**FileInputStream**对象，它表示将上载到 Azure 存储空间的本地文件。第二个参数是此文件的大小（以字节为单位）。

    blob.upload(new FileInputStream(fileReference), fileReference.length());

调用一个名为 **MakeHTMLPage** 的帮助器函数来生成一个包含 **&lt;image&gt;** 元素的基本 HTML 页面，并将元素中的源设置为现在位于你的 Azure 存储帐户中的 Blob。本文后面将讨论 **MakeHTMLPage** 的代码。

    MakeHTMLPage(container);

打印输出有关创建的 HTML 页的状态消息和信息。

    System.out.println("Processing complete.");
    System.out.println("Open index.html to see the images stored in your storage account.");

结束 **try** 块的方法是插入右大括号：**}**

处理下列异常：

-   **FileNotFoundException**：可由 **FileInputStream**
    或 **FileOutputStream** 构造函数引发。
-   **StorageException**：可由 Azure 客户端存储库引发。
-   **URISyntaxException**：可由 **ListBlobItem.getUri** 方法引发。
-   **Exception**：一般异常处理。

<!-- -->

    catch (FileNotFoundException fileNotFoundException)
    {
        System.out.print("FileNotFoundException encountered: ");
        System.out.println(fileNotFoundException.getMessage());
        System.exit(-1);
    }
    catch (StorageException storageException)
    {
        System.out.print("StorageException encountered: ");
        System.out.println(storageException.getMessage());
        System.exit(-1);
    }
    catch (URISyntaxException uriSyntaxException)
    {
        System.out.print("URISyntaxException encountered: ");
        System.out.println(uriSyntaxException.getMessage());
        System.exit(-1);
    }
    catch (Exception e)
    {
        System.out.print("Exception encountered: ");
        System.out.println(e.getMessage());
        System.exit(-1);
    }

结束 **main** 的方法是插入右大括号：**}**

创建一个名为 **MakeHTMLPage** 的方法来创建一个基本的 HTML 页面。此方法具有一个 **CloudBlobContainer** 类型的参数，该参数将用于循环访问已上载 Blob 的列表。此方法将引发 **FileNotFoundException** 类型的异常（可由 **FileOutputStream** 构造函数引发）以及 **URISyntaxException**（可由 **ListBlobItem.getUri**.getUri 方法引发）。包括左大括号 **{**。

    public static void MakeHTMLPage(CloudBlobContainer container) throws FileNotFoundException, URISyntaxException
    {

创建一个名为 **index.html** 的本地文件。

    PrintStream stream;
    stream = new PrintStream(new FileOutputStream("index.html"));

写入本地文件，添加 **&lt;html&gt;**、**&lt;header&gt;** 和 **&lt;body&gt;** 元素。

    stream.println("<html>");
    stream.println("<header/>");
    stream.println("<body>");

循环访问已上载 Blob 的列表。对于 HTML 页中的每个 Blob，创建一个 **&lt;img&gt;** 元素，并将该元素的 **src** 属性发送到 Blob 的 URI（如同它存在于你的 Azure 存储帐户中一样）。
虽然你仅在此示例中添加了一个图像，但如果添加更多图像，此代码将循环访问所有这些图像。

为简单起见，此示例假定上载的每个 Blob 都是一个图像。如果你更新了图像之外的 Blob，或者更新了页面 Blob 而不是块 Blob，则请根据需要调整代码。

    // Enumerate the uploaded blobs.
    for (ListBlobItem blobItem : container.listBlobs()) {
    // List each blob as an <img> element in the HTML body.
    stream.println("<img src='" + blobItem.getUri() + "'/><br/>");
    }

关闭 **&lt;body&gt;** 元素和 **&lt;html&gt;** 元素。

    stream.println("</body>");
    stream.println("</html>");

结束本地文件。

    stream.close();

结束 **MakeHTMLPage** 的方法是插入右大括号：**}**

结束 **StorageSample** 的方法是插入右大括号：**}**

以下是此示例的完整代码。请记得修改占位符值 **your_account_name** 和
**your_account_key**，这样才能分别使用你的帐户名和帐户密钥。

    import com.microsoft.azure.storage.*;
    import com.microsoft.azure.storage.blob.*;
    import java.io.*;
    import java.net.URISyntaxException;

    // Create an image, c:\myimages\image1.jpg, prior to running this sample.
    // Alternatively, change the value used by the FileInputStream constructor
    // to use a different image path and file that you have already created.
    public class StorageSample {

        public static final String storageConnectionString =
                "DefaultEndpointsProtocol=http;" +
                       "AccountName=your_account_name;" +
                       "AccountKey=your_account_name;" +
		       "EndpointSuffix=core.chinacloudapi.cn";

        public static void main(String[] args) 
        {
            try
            {
                CloudStorageAccount account;
                CloudBlobClient serviceClient;
                CloudBlobContainer container;
                CloudBlockBlob blob;

                account = CloudStorageAccount.parse(storageConnectionString);
                serviceClient = account.createCloudBlobClient();
                // Container name must be lower case.
                container = serviceClient.getContainerReference("gettingstarted");
                container.createIfNotExists();

                // Set anonymous access on the container.
                BlobContainerPermissions containerPermissions;
                containerPermissions = new BlobContainerPermissions();
                containerPermissions.setPublicAccess(BlobContainerPublicAccessType.CONTAINER);
                container.uploadPermissions(containerPermissions);

                // Upload an image file.
                blob = container.getBlockBlobReference("image1.jpg");

                File fileReference = new File("c:\\myimages\\image1.jpg");
                blob.upload(new FileInputStream(fileReference), fileReference.length());

                // At this point the image is uploaded.
                // Next, create an HTML page that lists all of the uploaded images.
                MakeHTMLPage(container);

                System.out.println("Processing complete.");
                System.out.println("Open index.html to see the images stored in your storage account.");

            }
            catch (FileNotFoundException fileNotFoundException)
            {
                System.out.print("FileNotFoundException encountered: ");
                System.out.println(fileNotFoundException.getMessage());
                System.exit(-1);
            }
            catch (StorageException storageException)
            {
                System.out.print("StorageException encountered: ");
                System.out.println(storageException.getMessage());
                System.exit(-1);
            }
            catch (URISyntaxException uriSyntaxException)
            {
                System.out.print("URISyntaxException encountered: ");
                System.out.println(uriSyntaxException.getMessage());
                System.exit(-1);
            }
            catch (Exception e)
            {
                System.out.print("Exception encountered: ");
                System.out.println(e.getMessage());
                System.exit(-1);
            }
        }

        // Create an HTML page that can be used to display the uploaded images.
        // This example assumes all of the blobs are for images.
        public static void MakeHTMLPage(CloudBlobContainer container) throws FileNotFoundException, URISyntaxException
        {
            PrintStream stream;
            stream = new PrintStream(new FileOutputStream("index.html"));

            // Create the opening <html>, <header>, and <body> elements.
            stream.println("<html>");
            stream.println("<header/>");
            stream.println("<body>");

            // Enumerate the uploaded blobs.
            for (ListBlobItem blobItem : container.listBlobs()) {
                // List each blob as an <img> element in the HTML body.
                stream.println("<img src='" + blobItem.getUri() + "'/><br/>");
            }

            stream.println("</body>");

            // Complete the <html> element and close the file.
            stream.println("</html>");
            stream.close();
        }
    }

除了将本地图像文件上载到 Azure 存储空间之外，此示例代码还将创建本地文件 namedindex.html，你可在浏览器中打开该文件以查看上载的图像。

由于代码包含您的帐户名称和帐户密钥，因此请确保您的源代码是安全的。

## <a name="bkmk_deletecontainer"> </a>删除容器


由于你的存储是收费的，因此你可能希望在完成对此示例的试验后删除 **gettingstarted** 容器。若要删除容器，请使用 **CloudBlobContainer.delete** 方法：

    container = serviceClient.getContainerReference("gettingstarted");
    container.delete();

若要调用 **CloudBlobContainer.delete** 方法，初始化 **CloudStorageAccount**、**ClodBlobClient**、**CloudBlobContainer** 对象的过程与为 **createIfNotExist** 方法演示的过程相同。以下是删除名为 **gettingstarted** 的容器的完整示例。

    import com.microsoft.azure.storage.*;
    import com.microsoft.azure.storage.blob.*;

    public class DeleteContainer {

        public static final String storageConnectionString =
                "DefaultEndpointsProtocol=http;" +
                   "AccountName=your_account_name;" +
                   "AccountKey=your_account_key;" +
		   "EndpointSuffix=core.chinacloudapi.cn";

        public static void main(String[] args)
        {
            try
            {
                CloudStorageAccount account;
                CloudBlobClient serviceClient;
                CloudBlobContainer container;

                account = CloudStorageAccount.parse(storageConnectionString);
                serviceClient = account.createCloudBlobClient();
                // Container name must be lower case.
                container = serviceClient.getContainerReference("gettingstarted");
                container.delete();

                System.out.println("Container deleted.");

            }
            catch (StorageException storageException)
            {
                System.out.print("StorageException encountered: ");
                System.out.println(storageException.getMessage());
                System.exit(-1);
            }
            catch (Exception e)
            {
                System.out.print("Exception encountered: ");
                System.out.println(e.getMessage());
                System.exit(-1);
            }
        }
    }

有关其他 Blob 存储类和方法的概述，请参阅[如何通过 Java 使用 Blob 存储](/documentation/articles/storage-java-how-to-use-blob-storage/)。

## 后续步骤

请访问下面的链接了解有关更复杂的存储任务的详细信息。

- [Azure Storage SDK for Java](https://github.com/azure/azure-storage-java)
- [Azure 存储客户端 SDK 参考](http://azure.github.io/azure-storage-java/)
- [Azure 存储空间服务 REST API](https://msdn.microsoft.com/zh-cn/library/azure/dd179355.aspx)
- [Azure 存储团队博客](http://blogs.msdn.com/b/windowsazurestorage/)


<!---HONumber=Mooncake_0516_2016-->