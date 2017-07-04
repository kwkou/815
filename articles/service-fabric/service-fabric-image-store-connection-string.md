<properties
    pageTitle="Azure Service Fabric 映像存储区连接字符串 | Azure"
    description="了解映像存储区连接字符串"
    services="service-fabric"
    documentationcenter=".net"
    author="alexwun"
    manager="timlt"
    editor="" />
<tags
    ms.assetid="00f8059d-9d53-4cb8-b44a-b25149de3030"
    ms.service="service-fabric"
    ms.devlang="dotnet"
    ms.topic="article"
    ms.tgt_pltfrm="NA"
    ms.workload="NA"
    ms.date="02/08/2017"
    wacn.date="03/03/2017"
    ms.author="alexwun" />  


# 了解 ImageStoreConnectionString 设置

在某些文档中，我们简单提及存在“ImageStoreConnectionString”参数，但没有介绍它实际意味着什么。阅读[使用 PowerShell 部署和删除应用程序][10]一类的文章后，会发现似乎用户只需复制/粘贴在目标群集的群集清单中显示的值即可。因此，每个群集的设置都必须是可配置的，但用户通过 [Azure 门户][11]创建群集时，没有用于配置此设置的选项，它始终是“fabric:ImageStore”。那么，此设置的存在有何作用？

![群集清单][img_cm]  


Service Fabric 一开始被许多不同的团队用作 Microsoft 内部消耗平台，因此，其某些方面是高度可自定义的，“映像存储区 (Image Store)”就是这样的存在。映像存储区实质上是用于存储应用程序包的可插入存储库。应用程序部署到群集中的节点时，该节点会从映像存储区下载应用程序包的内容。ImageStoreConnectionString 是一项设置，它包含客户端和节点用于查找给定群集的正确映像存储区的所有必要信息。

目前，有三种可能的映像存储区提供程序类型，这些类型及其对应的连接字符串如下所示：

1. 映像存储区服务：“fabric:ImageStore”

2. 文件系统：“file:[file system path]”

3. Azure 存储：“xstore:DefaultEndpointsProtocol=https;AccountName=[...];AccountKey=[...];Container=[...];EndpointSuffix=core.chinacloudapi.cn”

在生产中使用的提供程序类型为映像存储区服务，它是可通过 Service Fabric Explorer 查看的有状态持久化系统服务。

![映像存储区服务][img_is]  


在群集内的系统服务中托管映像存储区可清除包存储库的外部依赖项，并让我们能够更好地控制存储的位置。将来的映像存储区增强功能就算没有专门针对映像存储区提供程序，也很可能以它为优先目标。客户端已连接到目标群集，因此，映像存储区服务提供程序的连接字符串不具有任何唯一的信息。客户端只需知道应使用面向系统服务的协议即可。

在开发过程中，本地单机群集使用文件系统提供程序，而不使用映像存储区服务，目的在于让群集的 bootstrap 操作速度略微提升。差异通常较小，但在开发期间，它对大多数人员而言是有用的优化。尽管可部署其他存储提供程序类型的本地单机群集，但通常没有理由这么做，因为无论使用哪种提供程序，开发/测试工作流都保持不变。除这种用法外，文件系统和 Azure 存储提供程序仅为提供旧版支持而存在。

因此，虽然 ImageStoreConnectionString 是可配置的，但用户通常只使用默认设置。通过 [Visual Studio][12] 发布到 Azure 时，会相应地自动为用户设置该参数。对于在 Azure 中托管的群集的编程部署，连接字符串始终是“fabric:ImageStore”。如果有疑问，始终可通过 [PowerShell](https://docs.microsoft.com/powershell/servicefabric/vlatest/get-servicefabricclustermanifest)、[.NET](https://msdn.microsoft.com/zh-cn/library/azure/mt161375.aspx) 或 [REST](https://docs.microsoft.com/rest/api/servicefabric/get-a-cluster-manifest) 检索群集清单来验证其值。还应始终将本地测试和生产群集配置为使用映像存储区服务提供程序。

### 后续步骤
[使用 PowerShell 部署和删除应用程序][10]

<!--Image references-->

[img_is]: ./media/service-fabric-image-store-connection-string/image_store_service.png
[img_cm]: ./media/service-fabric-image-store-connection-string/cluster_manifest.png

[10]: /documentation/articles/service-fabric-deploy-remove-applications/
[11]: /documentation/articles/service-fabric-cluster-creation-via-portal/
[12]: /documentation/articles/service-fabric-publish-app-remote-cluster/

<!---HONumber=Mooncake_0227_2017-->