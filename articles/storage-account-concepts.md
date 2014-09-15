<properties umbracoNaviHide="0" pageTitle="Storage Account Concepts | Azure" metaKeywords="Azure storage, storage service, service, storage account, account, create storage account, create account" description="Learn about storage account concepts." linkid="manage-windows-how-to-guide-storage-accounts" urlDisplayName="How to: storage accounts" headerExpose="" footerExpose="" disqusComments="1" title="Storage Account Concepts" services="storage" authors="tamram" manager="mbaldwin" editor="cgronlun" />

# 存储帐户概念

-   **地域冗余存储 (GRS)**：地域冗余存储通过将你的数据无缝复制到同一区域内的辅助位置来提供最高级别的持久存储。这样就能够在主要位置发生严重故障时进行故障转移。辅助位置距离主要位置数百英里。可通过名为“地域复制”**的功能实现 GRS，存储帐户的该功能默认情况下处于打开状态，但是，如果你不希望使用它（例如，公司政策禁止使用），则可以将其关闭。有关详细信息，请参阅 [Windows Azure 的地域复制简介][]。

-   **本地冗余存储 (LRS)**：本地冗余存储在单个位置提供高度持久且高度可用的存储服务。使用本地冗余存储时，将在同一数据中心复制帐户数据三次。Azure 中的所有存储都是本地冗余的。为增加持久性，你可以打开地域复制。本地冗余存储将以优惠价提供。有关定价信息，请参阅[Azure 定价概述][]。

-   **地缘组**：**“地缘组”是你的云服务部署和存储帐户在 Azure 中的地理分组。通过定位同一数据中心或靠近目标用户受众的计算机工作负载，地缘组可提高服务性能。此外，不会对出口带宽收费。

-   **存储帐户终结点**：**存储帐户的“终结点”表示用于访问 Blob、表或队列的最高级别的命名空间。存储帐户的默认终结点具有以下几种格式：

    -   Blob 服务：<http://*mystorageaccount>\*.blob.core.chinacloudapi.cn

    -   表服务：<http://*mystorageaccount>\*.table.core.chinacloudapi.cn

    -   队列服务：<http://*mystorageaccount>\*.queue.core.chinacloudapi.cn

-   **存储帐户 URL**：用于访问存储帐户中某个对象的 URL 是通过将存储帐户中对象的位置附加到终结点而构建的。例如，Blob 地址可能具有以下格式：<http://*mystorageaccount>*.blob.core.chinacloudapi.cn/*mycontainer*/*myblob\*。

-   **存储访问密钥**：当你创建存储帐户时，Azure 将生成两个 512 位存储访问密钥，用于在用户访问该存储帐户时对其进行身份验证。通过提供两个存储访问密钥，Azure 使你能够在不中断存储服务的情况下重新生成用于访问该服务的密钥。

-   **最少监视与详细监视度量值**：你可以在监视设置中为存储帐户配置最少监视或详细监视度量值。**“最少监视度量值”收集经过汇总的有关 Blob、表和队列服务的入口/出口、可用性、延迟及成功百分比等数据的度量值。**“详细监视度量值”针对相同度量值收集除服务级别汇总之外的操作级别详细信息。通过详细监视度量值可对应用程序运行期间出现的问题进行进一步分析。有关可用度量值的完整列表，请参阅[存储分析度量值表架构][]。有关存储监视的详细信息，请参阅[关于存储分析指标][]。

-   **日志记录**：日志记录是存储帐户的一项可配置功能，用于记录读、写以及删除 Blob、表和队列的请求。你可以在 Azure 管理门户中配置日志记录，但无法在管理门户中查看日志。这些日志存储在存储帐户的 \$logs 容器中，并可对其进行访问。有关详细信息，请参阅[存储分析概述][]。

  [Windows Azure 的地域复制简介]: http://blogs.msdn.com/b/windowsazurestorage/archive/2011/09/15/introducing-geo-replication-for-windows-azure-storage.aspx
  [Azure 定价概述]: http://www.windowsazure.cn/zh-cn/pricing/overview/
  [存储分析度量值表架构]: http://msdn.microsoft.com/zh-cn/library/azure/hh343264.aspx
  [关于存储分析指标]: http://msdn.microsoft.com/zh-cn/library/azure/hh343258.aspx
  [存储分析概述]: http://msdn.microsoft.com/zh-cn/library/azure/hh343268.aspx
