<properties linkid="manage-services-what-is-a-storage-account" urlDisplayName="What is a Storage Account" pageTitle="What is a storage account? | Microsoft Azure" metaKeywords="" description="Learn about the different types of storage accounts available in Azure, and get definitions for key storage terms." metaCanonical="" services="storage" documentationCenter="" title="What is a Storage Account?" authors="tamram" solutions="" manager="mbaldwin" editor="cgronlun" />

# 什么是存储帐户？

Azure 存储空间包括三项服务：Blob 存储、表存储和队列存储。每个存储帐户中都包括这三项服务。存储帐户提供了用于操作 Blob、队列和表的唯一命名空间。

存储帐户最多可包含 200TB 的 Blob、队列和表数据，前提是它是在 2012 年 6 月 8 日或之后创建的；对于在此日期之前创建的存储帐户，总容量是 100TB。通过单个订阅，你最多可以创建 20 个唯一的命名存储帐户。有关存储帐户的详细信息，请参阅 [Azure 存储空间可伸缩性和性能目标][]。

关于你的存储帐户的所有信息（包括它是何时创建的）都位于管理门户中**存储**的**“仪表板”**页上。

存储成本取决于以下四个因素：存储容量、复制方案、存储事务和数据流出量。存储容量指的是存储帐户中用来存储数据的配额。对数据进行简单存储时，其成本取决于存储的数据量和数据复制方式。事务指的是对 Azure 存储空间的所有读取和写入操作。数据流出量指的是传出某个 Azure 区域的数据。当不在同一区域中的应用程序访问你的存储帐户中的数据时，无论该应用程序是云服务还是某个其他类型的应用程序，都将会针对数据流出量向你收费。（对于 Azure 服务，你可以采取措施将你的数据和服务通过分组分到相同的数据中心内，从而降低或避免数据流出量费用。）

[存储定价详细信息][]页提供了针对存储容量、复制和事务的详细定价信息。[数据传输定价详细信息][]提供了针对数据流出量的详细定价信息。

## 概念

-   **本地冗余存储 (LRS)**：本地冗余存储在单个位置提供高度持久且高度可用的存储服务。使用本地冗余存储时，将在同一数据中心复制帐户数据三次。Azure 中的所有存储都是本地冗余的。为增加持久性，你可以打开地域复制。本地冗余存储将以优惠价提供。有关定价信息，请参阅[定价详细信息][存储定价详细信息]。

-   **地缘组**：**“地缘组”是你的云服务部署和存储帐户在 Azure 中的地理分组。通过定位同一数据中心或靠近目标用户受众的计算机工作负载，地缘组可提高服务性能。另外，当在同一地缘组中运行的服务访问你的存储帐户中的数据时，不会产生数据流出量费用。

-   **存储帐户终结点**：**存储帐户的“终结点”表示用于访问 Blob、表或队列的最高级别的命名空间。存储帐户的默认终结点具有以下几种格式：

    -   Blob 服务：<http://*mystorageaccount>\*.blob.core.chinacloudapi.cn

    -   表服务：<http://*mystorageaccount>\*.table.core.chinacloudapi.cn

    -   队列服务：<http://*mystorageaccount>\*.queue.core.chinacloudapi.cn

-   **存储帐户 URL**：用于访问存储帐户中某个对象的 URL 是通过将存储帐户中对象的位置附加到终结点而构建的。例如，Blob 地址可能具有以下格式：<http://*mystorageaccount>*.blob.core.chinacloudapi.cn/*mycontainer*/*myblob\*。

-   **存储访问密钥**：当你创建存储帐户时，Azure 将生成两个 512 位存储访问密钥，用于在用户访问该存储帐户时对其进行身份验证。通过提供两个存储访问密钥，Azure 使你能够在不中断存储服务的情况下重新生成用于访问该服务的密钥。

-   **最少监视与详细监视度量值**：你可以在监视设置中为存储帐户配置最少监视或详细监视度量值。**“最少监视度量值”收集经过汇总的有关 Blob、表和队列服务的入口/出口、可用性、延迟及成功百分比等数据的度量值。**“详细监视度量值”针对相同度量值收集除服务级别汇总之外的操作级别详细信息。通过详细监视度量值可对应用程序运行期间出现的问题进行进一步分析。有关可用度量值的完整列表，请参阅[存储分析度量值表架构][]。有关存储监视的详细信息，请参阅[关于存储分析指标][]。

-   **日志记录**：日志记录是存储帐户的一项可配置功能，用于记录读、写以及删除 Blob、表和队列的请求。你可以在 Azure 管理门户中配置日志记录，但无法在管理门户中查看日志。这些日志存储在存储帐户的 \$logs 容器中，并可对其进行访问。有关详细信息，请参阅[关于存储分析日志记录][]。

  [Azure 存储空间可伸缩性和性能目标]: http://msdn.microsoft.com/zh-cn/library/dn249410.aspx
  [存储定价详细信息]: http://www.windowsazure.cn/zh-cn/pricing/overview/#storage
  [数据传输定价详细信息]: http://www.windowsazure.cn/zh-cn/pricing/overview/#data_transfer
  [存储分析度量值表架构]: http://msdn.microsoft.com/zh-cn/library/azure/hh343264.aspx
  [关于存储分析指标]: http://msdn.microsoft.com/zh-cn/library/azure/hh343258.aspx
  [关于存储分析日志记录]: http://msdn.microsoft.com/zh-cn/library/azure/hh343262.aspx
