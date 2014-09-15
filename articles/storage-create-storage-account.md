<properties linkid="manage-services-how-to-create-a-storage-account" urlDisplayName="How to create" pageTitle="How to create a storage account | Azure" metaKeywords="" description="Learn how to create a storage account in the Azure management portal." metaCanonical="" services="storage" documentationCenter="" title="How To Create a Storage Account" solutions="" authors="tamram" manager="mbaldwin" editor="cgronlun" />

# 如何创建存储帐户

若要将 Blob、表和队列服务中的文件和数据存储在 Azure 中，你必须在要存储数据的地理区域中创建存储帐户。一个存储帐户最多可以包含 200 TB 的数据，你最多可以为每个 Azure 订阅创建二十个存储帐户。有关详细信息，请参阅 [Azure 存储的可伸缩性和性能目标][]。

本主题介绍了如何在 Azure 管理门户中创建存储帐户。

**说明**

对于 Azure 虚拟机，如果你在部署位置中还没有存储帐户，则会在该位置自动创建一个存储帐户。存储帐户名称将基于虚拟机名称。

## 目录

-   [如何：创建存储帐户][]
-   [后续步骤][]

## 如何：创建存储帐户

1.  登录到[管理门户][]。

2.  依次单击**“新建”**、**“存储”**和**“快速创建”**。

    ![新建存储帐户][]

3.  在**“URL”**中，输入要在存储帐户 URL 中使用的子域名称。若要访问存储中的对象，请将该对象的位置附加到终结点。例如，用于访问 Blob 的 URL 可以是 <http://*myaccount>*.blob.core.chinacloudapi.cn/*mycontainer*/*myblob\*。

4.  在**“区域/地缘组”**中，为存储选择区域或地缘组。如果你希望存储服务与你所使用的其他 Azure 服务位于同一数据中心，请选择一个地缘组而不是区域。这可以提高性能，且不会对传出收费。

    > [WACOM.NOTE]
    >  \> 若要创建地缘组，请打开管理门户的**“网络”**区域，单击**“地缘组”**，然后单击**“创建新的地缘组”**或**“创建”**。可以使用你在以前的管理门户中创建的地缘组。若要打开其他门户，请单击标题栏上的**“预览”**，然后单击**“转到以前的门户”**。（若要返回此门户，请单击门户底部的**“查看预览门户”**。）你也可以使用 Azure 服务管理 API 创建和管理地缘组。有关详细信息，请参阅[对地缘组的操作][]。

5.  如果你有多个 Azure 订阅，则会显示**“订阅”**字段。在**“订阅”**中，输入要使用存储帐户的 Azure 订阅。你最多可以为一个订阅创建五个存储帐户。

6.  在**“复制”**中，选择要用于存储帐户的复制级别。

    默认情况下，复制设置为**“地域冗余”**。使用地域冗余复制，你的存储帐户及其中的所有数据在主位置发生重大灾难时就能够故障转移到辅助位置。Azure 将分配同一区域中的辅助位置，并且无法对其进行更改。在进行故障转移后，辅助位置将成为存储帐户的主位置，并且会将你的数据复制到新的辅助位置。

    如果希望能够从辅助位置读取数据，可以选择**“读取访问地域冗余”**复制。此选项提供了地域冗余复制并启用了对辅助位置中复制数据的只读访问。当主位置或辅助位置中的某个位置变得不可用时，读取访问地域冗余复制允许你从另一个位置访问你的数据。

    第三个复制选项**“读取访问地域冗余”**当前处于预览状态。此选项启用了对辅助位置中复制数据的只读访问。当主位置或辅助位置中的某个位置变得不可用时，读取访问地域冗余复制允许你从另一个位置访问你的数据。

    > [WACOM.NOTE]
    >  \> 若要在读取访问地域冗余复制处于预览状态时使用它，必须手动请求为你的订阅启用该功能。可以访问 [Azure 预览功能][]页面来为你的订阅请求读取访问地域冗余复制。有关读取访问地域冗余复制的更多详细信息，请参阅 [Azure 存储空间团队博客][]。
    > 如果在你的订阅上没有将读取访问地域冗余复制启用为预览功能，则用于为你的存储帐户选择该功能的选项将处于禁用状态。

    有关存储帐户复制的定价信息，请参阅[存储定价详细信息][]。

7.  单击**“创建存储帐户”**。

    创建存储帐户可能需要花费几分钟的时间。若要检查状态，你可以监视门户底部的通知。创建存储帐户后，你的新存储帐户将处于**“联机”**状态并且随时可供使用。

    ![存储页面][]

## 后续步骤

-   若要更详细地了解 Azure 存储服务，请参阅[了解云存储][]和 [Blob、队列和表][]。

-   访问 [Azure 存储空间团队博客][1]。

-   对你的应用程序进行配置以使用 Azure Blob、表和队列服务。[Azure 开发人员中心][]提供了将 Blob、表和队列存储服务与 .NET、Node.js、Java 和 PHP 应用程序结合使用的操作方法指南。有关特定于某种编程语言的说明，请参阅该语言对应的操作方法指南。

  [Azure 存储的可伸缩性和性能目标]: http://msdn.microsoft.com/zh-cn/library/dn249410.aspx
  [如何：创建存储帐户]: #create
  [后续步骤]: #next
  [管理门户]: https://manage.windowsazure.cn
  [新建存储帐户]: ./media/storage-create-storage-account/storage_NewStorageAccount.png
  [对地缘组的操作]: http://msdn.microsoft.com/zh-cn/library/azure/ee460798.aspx
  [Azure 预览功能]: https://account.windowsazure.com/PreviewFeatures
  [Azure 存储空间团队博客]: http://blogs.msdn.com/b/windowsazurestorage/archive/2013/12/04/introducing-read-access-geo-replicated-storage-ra-grs-for-windows-azure-storage.aspx
  [存储定价详细信息]: http://www.windowsazure.cn/zh-cn/pricing/overview/#storage
  [存储页面]: ./media/storage-create-storage-account/Storage_StoragePage.png
  [了解云存储]: http://azure.microsoft.com/zh-cn/documentation/articles/storage-introduction/
  [Blob、队列和表]: http://msdn.microsoft.com/zh-cn/library/gg433040.aspx
  [1]: http://blogs.msdn.com/b/windowsazurestorage/
  [Azure 开发人员中心]: http://azure.microsoft.com/zh-cn/documentation/
