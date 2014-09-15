<properties linkid="dev-java-how-to-on-premise-application-with-blob-storage" urlDisplayName="Image Gallery w/ Storage" pageTitle="On-premises application with blob storage (Java) | Microsoft Azure" metaKeywords="Azure blob storage, Azure blob Java, Azure blob example, Azure blob tutorial" description="Learn how to create a console application that uploads an image to Azure, and then displays the image in your browser. Code samples in Java." metaCanonical="" services="storage" documentationCenter="Java" title="On-Premises Application with Blob Storage" authors="robmcm" solutions="" manager="wpickett" editor="mollybos" scriptId="" videoId="" />

# 使用 Blob 存储的本地应用程序

以下示例将演示如何使用 Azure 存储空间在 Azure 中存储图像。
以下代码是一个控制台应用程序的代码，
该应用程序将一个图像上载到 Azure，然后创建用于
在浏览器中显示该图像的 HTML 文件。

## 目录

-   [先决条件][]
-   [使用 Azure Blob 存储上载文件][]
-   [删除容器][]

## 先决条件

1.  已安装 Java 开发人员工具包 (JDK) 1.6 或更高版本。
2.  已安装 Azure SDK。
3.  适用于 Azure Libraries for Java 的 JAR 以及任何
    适用的依赖项 JAR 已安装并且位于 Java 编译器使用
    的生成路径中。有关安装 Azure Libraries for Java 的信息，
    请参阅[下载 Azure SDK for Java][]。
4.  已设置了一个 Azure 存储帐户。以下代码
    将使用存储帐户的帐户名称和帐户密钥。
    有关创建存储帐户的信息，请参阅[如何创建存储帐户][]；
    有关检索帐户密钥的信息，请参阅
    [如何管理存储帐户][]。
5.  你已创建了存储在路径 c:\\myimages\\image1.jpg 处的
    已命名本地图像文件。或者，在示例中修改
    **FileInputStream** 构造函数以使用其他图像路径和文件名。

[WACOM.INCLUDE [create-account-note][]]

## 使用 Azure Blob 存储上载文件

此处提供了分步过程；如果你要跳过这些过程，可在本主题的下文中找到
完整的代码。

在代码的开头请包括对 Azure 核心存储类、Azure Blob
客户端类、Java IO 类和 **URISyntaxException** 类
的导入：

    import com.microsoft.windowsazure.services.core.storage.*;
    import com.microsoft.windowsazure.services.blob.client.*;
    import java.io.*;
    import java.net.URISyntaxException;

声明一个名为 **StorageSample** 的类，包含左大括号
**{**。

    public class StorageSample {

在 **StorageSample** 类中，声明一个将包含
默认终结点协议、你的存储帐户名称和存储访问密钥
（在你的 Azure 存储帐户中指定）
的字符串变量。将占位符值 **your\_account\_name** 和
**your\_account\_key** 分别替换为你自己的帐户名称
和帐户密钥。

    public static final String storageConnectionString = 
    "DefaultEndpointsProtocol=http;" + 
    "AccountName=your_account_name;" + 
    "AccountKey=your_account_name"; 

添加对 **main** 的声明，包括一个 **try** 块并包括必需的
左大括号 **{**。

    public static void main(String[] args) 
    {
    try
        {

声明以下类型的变量（下面说明了在此示例中如何使用这些变量）：

-   **CloudStorageAccount**：用于通过 Azure 存储帐户名称和
    密钥初始化帐户对象，以及用于创建 Blob 客户端
    对象。
-   **CloudBlobClient**：用于访问 Blob 服务。
-   **CloudBlobContainer**：用于创建 Blob 容器、列出容器中
    的 Blob 以及删除容器。
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

为 **container** 变量赋值。我们将获取对
名为 **gettingstarted** 的容器的引用。

    // 容器名称必须小写。
    container = serviceClient.getContainerReference("gettingstarted");

创建该容器。如果该容器不存在，此方法将创建该容器
（并返回 **true**）。如果该容器存在，则此方法将返回
**false**。**createIfNotExist** 的一个替代方法是 **create**
方法（如果该容器已存在，该方法将返回错误）。

    container.createIfNotExist();

为容器设置匿名访问。

    // 在容器上设置匿名访问。
    BlobContainerPermissions containerPermissions;
    containerPermissions = new BlobContainerPermissions();
    containerPermissions.setPublicAccess(BlobContainerPublicAccessType.CONTAINER);
    container.uploadPermissions(containerPermissions);

获取对块 Blob 的引用，它将表示 Azure 存储空间中的 Blob。

    blob = container.getBlockBlobReference("image1.jpg");

使用 **File** 构造函数获取对你将上载的在本地创建的文件的引用。
（确保在运行代码之前已创建了此文件。）

    File fileReference = new File ("c:\\myimages\\image1.jpg");

通过调用 **CloudBlockBlob.upload** 方法上载该本地文件。
**CloudBlockBlob.upload** 方法的第一个参数是一个
**FileInputStream** 对象，它表示将上载到 Azure 存储空间
的本地文件。第二个参数是该文件的大小
（以字节为单位）。

    blob.upload(new FileInputStream(fileReference), fileReference.length());

调用一个名为 **MakeHTMLPage** 的帮助器函数来生成一个
包含 **\<image\>** 元素的基本 HTML 页面，并将元素中的源设置为现在位于
你的 Azure 存储帐户中的 Blob。（本主题下文中
将讨论 **MakeHTMLPage** 的代码。）

    MakeHTMLPage(container);

打印输出有关创建的 HTML 页的状态消息和信息。

    System.out.println("Processing complete.");
    System.out.println("Open index.html to see the images stored in your storage account.");

通过插入右大括号结束 **try** 块： **}**

处理下列异常：

-   **FileNotFoundException**：可由 **FileInputStream**
     或 **FileOutputStream**
     构造函数引发。
-   **StorageException**：可由 Azure 客户端
    存储库引发。
-   **URISyntaxException**：可由 **ListBlobItem.getUri**
     方法引发。
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

通过插入右大括号结束 **main**： **}**

创建一个名为 **MakeHTMLPage** 的方法来创建一个基本的 HTML 页面。此方法
具有一个 **CloudBlobContainer** 类型的参数，该参数将用于循环访问
已上载 Blob 的列表。此方法将
引发 **FileNotFoundException** 类型的异常（可由 **FileOutputStream**
构造函数引发）以及 **URISyntaxException** 类型的异常（可由
**ListBlobItem.getUri** 方法引发）。包括
左大括号 **{**。

    public static void MakeHTMLPage(CloudBlobContainer container) throws FileNotFoundException, URISyntaxException
    {

创建一个名为 **index.html** 的本地文件。

    PrintStream stream;
    stream = new PrintStream(new FileOutputStream("index.html"));

写入到本地文件，添加 **\<html\>**、**\<header\>** 和
**\<body\>** 元素。

    stream.println("<html>");
    stream.println("<header/>");
    stream.println("<body>");

循环访问已上载 Blob 的列表。对于每个 Blob，在 HTML 页
中创建一个 **\<img\>** 元素，并将该元素的 **src** 属性
发送到 Blob 的 URI（如同它存在于你的 Azure 存储帐户中一样）。
虽然在此示例中你只添加了一个图像，但如果你添加多个图像，
此代码将循环访问所有这些图像。

为简单起见，此示例假定上载的每个 Blob 都是一个图像。如果
你更新了不是图像的 Blob，或者更新了页面 Blob 而不是块 Blob，
请根据需要调整代码。

    // 枚举已上载的 Blob。
    for (ListBlobItem blobItem :container.listBlobs()) {
    // 将每个 Blob 在 HTML 正文中作为 <img> 元素列出。
    stream.println("<img src='" + blobItem.getUri() + "'/><br/>");
    }

结束 **\<body\>** 元素和 **\<html\>** 元素。

    stream.println("</body>");
    stream.println("</html>");

结束本地文件。

    stream.close();

通过插入右大括号结束 **MakeHTMLPage**： **}**

通过插入右大括号结束 **StorageSample**： **}**

以下是此示例的完整代码。请记得修改占位符值
**your\_account\_name** 和 **your\_account\_key** 以
分别使用你自己的帐户名称和
帐户密钥。

    import com.microsoft.windowsazure.services.core.storage.*;
    import com.microsoft.windowsazure.services.blob.client.*;
    import java.io.*;
    import java.net.URISyntaxException;

    // 在运行此示例之前创建图像 c:\myimages\image1.jpg。
    // 此外，也可以更改 FileInputStream 构造函数使用的值 
    // 以使用你已创建的其他图像路径和文件。
    public class StorageSample {

    public static final String storageConnectionString = 
    "DefaultEndpointsProtocol=http;" + 
    "AccountName=your_account_name;" + 
    "AccountKey=your_account_name"; 

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
    // 容器名称必须小写。
    container = serviceClient.getContainerReference("gettingstarted");
    container.createIfNotExist();
                
    // 在容器上设置匿名访问。
    BlobContainerPermissions containerPermissions;
    containerPermissions = new BlobContainerPermissions();
    containerPermissions.setPublicAccess(BlobContainerPublicAccessType.CONTAINER);
    container.uploadPermissions(containerPermissions);

    // 上载图像文件。
    blob = container.getBlockBlobReference("image1.jpg");
    File fileReference = new File ("c:\\myimages\\image1.jpg");
    blob.upload(new FileInputStream(fileReference), fileReference.length());

    // 此时已上载了图像。
    // 接下来，创建一个 HTML 页面以列出所有已上载的图像。
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

    // 创建一个可用来显示已上载图像的 HTML 页面。
    // 此示例假定所有 Blob 都是用于图像的。
    public static void MakeHTMLPage(CloudBlobContainer container) throws FileNotFoundException, URISyntaxException
    {
    PrintStream stream;
    stream = new PrintStream(new FileOutputStream("index.html"));

    // 创建起始 <html>、<header> 和 <body> 元素。
    stream.println("<html>");
    stream.println("<header/>");
    stream.println("<body>");

    // 枚举已上载的 Blob。
    for (ListBlobItem blobItem :container.listBlobs()) {
    // 将每个 Blob 在 HTML 正文中作为 <img> 元素列出。
    stream.println("<img src='" + blobItem.getUri() + "'/><br/>");
            }

    stream.println("</body>");

    // 完成 <html> 元素并关闭文件。
    stream.println("</html>");
    stream.close();
        }
    }

除了将本地图像文件上载到 Azure 存储空间之外，此示例代码
还将创建本地文件 namedindex.html，你可在浏览器中
打开该文件以查看上载的图像。

由于代码包含你的帐户名称和帐户密钥，
因此请确保你的源代码是安全的。

## 删除容器

由于你的存储是收费的，因此你可能希望在通过此示例完成试验后
删除 **gettingstarted** 容器。
若要删除容器，请使用 **CloudBlobContainer.delete**
方法：

    container = serviceClient.getContainerReference("gettingstarted");
    container.delete();

在调用 **CloudBlobContainer.delete** 方法时，初始化
**CloudStorageAccount**、**ClodBlobClient**、**CloudBlobContainer** 对象
的过程与为 **createIfNotExist** 方法
演示的过程相同。以下是删除名为 **gettingstarted** 的
容器的完整示例。

    import com.microsoft.windowsazure.services.core.storage.*;
    import com.microsoft.windowsazure.services.blob.client.*;

    public class DeleteContainer {

    public static final String storageConnectionString = 
    "DefaultEndpointsProtocol=http;" + 
    "AccountName=your_account_name;" + 
    "AccountKey=your_account_key"; 

    public static void main(String[] args) 
        {
    try
            {
    CloudStorageAccount account;
    CloudBlobClient serviceClient;
    CloudBlobContainer container;
                
    account = CloudStorageAccount.parse(storageConnectionString);
    serviceClient = account.createCloudBlobClient();
    // 容器名称必须小写。
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

有关其他 Blob 存储类和方法的概述，请参阅
[如何通过 Java 使用 Blob 存储服务][]。

  [先决条件]: #bkmk_prerequisites
  [使用 Azure Blob 存储上载文件]: #bkmk_uploadfile
  [删除容器]: #bkmk_deletecontainer
  [下载 Azure SDK for Java]: http://azure.microsoft.com/zh-cn/develop/java/
  [如何创建存储帐户]: http://azure.microsoft.com/zh-cn/documentation/articles/storage-create-storage-account/
  [如何管理存储帐户]: http://azure.microsoft.com/zh-cn/documentation/articles/storage-manage-storage-account/
  [create-account-note]: ../includes/create-account-note.md
  [如何通过 Java 使用 Blob 存储服务]: http://azure.microsoft.com/zh-cn/documentation/articles/storage-java-how-to-use-blob-storage/
