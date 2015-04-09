<properties linkid="" urlDisplayName="" pageTitle="Architecture" metaKeywords="" description="Architecture overview that covers common design patterns" metaCanonical="" services="" documentationCenter="" videoId="" scriptId="" title="Architecture Overview" authors="robb" solutions="" manager="dongill" editor="mattshel" />

# 体系结构

了解如何在 Azure 中实施常见设计模式。

### Azure 符号/图标集

[下载 Azure 符号/图标集][下载 Azure 符号/图标集]以创建描述（或使用）Azure 的技术材料 - 例如体系结构图、培训材料、演示文稿、数据表、信息图和白皮书。你可以下载 PPT、Visio 或 PNG 格式的符号。我们想要知道你的想法，因此下载文件中包含了有关如何提供反馈的说明。

![Azure 符号/图标集][Azure 符号/图标集]

## 设计模式

### [让使用者竞争][让使用者竞争]

![让使用者竞争][1]

使多个并发使用者能够处理同一消息通道上收到的消息。此模式可让系统同时处理多个消息，以优化吞吐量、改进可扩展性和可用性，以及平衡工作负载。

### [命令和查询责任分离][命令和查询责任分离]

![命令和查询责任分离][2]

使用独立接口将读取数据的操作与更新数据的操作分离。此模式可以最大化性能、可扩展性和安全性；具有更高的灵活性，支持系统随着时间的推移而改进；防止更新命令在域级别造成合并冲突。

### [领导选拔][领导选拔]

![领导选拔][3]

协调分布式应用程序中协作的任务实例集合所执行的操作，具体的协调方式是选拔一个实例作为领导来负责管理其他实例。此模式有助于确保任务实例不会因相互冲突而导致共享资源的争用，也不会意外地影响其他任务实例执行的工作。

### [管道和筛选器][管道和筛选器]

![管道和筛选器][4]

将一个执行复杂处理的任务分解为一系列可重复使用的离散元素。此模式允许单独部署和缩放执行处理的任务元素，从而可以提高性能、可扩展性和可重用性。

### [严格保密的密钥][严格保密的密钥]

![严格保密的密钥][5]

使用一个可让客户端有限度直接访问特定资源或服务的令牌或密钥，以减轻应用程序代码的数据传输操作负担。此模式在使用云托管存储系统或队列的应用程序中特别有用，可以最大程度地降低成本并提高可扩展性和性能。

### 更多指导

有关 Azure 中其他常见设计模式的信息，请参阅[云设计模式][云设计模式]。

  [下载 Azure 符号/图标集]: http://www.microsoft.com/en-us/download/details.aspx?id=41937
  [Azure 符号/图标集]: ./media/architecture-overview/AzureSymbols.png
  [让使用者竞争]: http://msdn.microsoft.com/zh-cn/library/dn568101.aspx
  [1]: ./media/architecture-overview/CompetingConsumers.png
  [命令和查询责任分离]: http://msdn.microsoft.com/zh-cn/library/dn568103.aspx
  [2]: ./media/architecture-overview/CQRS.png
  [领导选拔]: http://msdn.microsoft.com/zh-cn/library/dn568104.aspx
  [3]: ./media/architecture-overview/LeaderElection.png
  [管道和筛选器]: http://msdn.microsoft.com/zh-cn/library/dn568100.aspx
  [4]: ./media/architecture-overview/PipesAndFilters.png
  [严格保密的密钥]: http://msdn.microsoft.com/zh-cn/library/dn568102.aspx
  [5]: ./media/architecture-overview/ValetKey.png
  [云设计模式]: http://msdn.microsoft.com/zh-cn/library/dn568099.aspx
