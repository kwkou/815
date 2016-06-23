<properties
    pageTitle="使用 Azure Toolkit for Eclipse 启用会话相关性"
    description="了解如何使用 Azure Toolkit for Eclipse 启用会话相关性。"
    services=""
    documentationCenter="java"
    authors="rmcmurray"
    manager="wpickett"
    editor=""/>

<tags
    ms.service="multiple"
    ms.date="05/04/2016" 
    wacn.date="06/20/2016"/>

<!-- Legacy MSDN URL = https://msdn.microsoft.com/library/azure/hh690950.aspx -->

# 启用会话相关性 #

在 Azure Toolkit for Eclipse 中，你可以为角色启用 HTTP 会话相关性或“粘性会话”。下图显示了用于启用会话相关性功能的“负载平衡”属性对话框：

![][ic719492]

## 为角色启用会话相关性 ##

1. 在 Eclipse 的项目资源管理器中右键单击角色，单击“Azure”，然后单击“负载平衡”。
1. 在“WorkerRole1 负载平衡属性”对话框中：
    1. 选中“为此角色启用 HTTP 会话相关性(粘性会话)”。
    1. 对于“要使用的输入终结点”，请选择要使用的输入终结点，例如“http (public:80, private:8080)”。你的应用程序必须使用此终结点作为其 HTTP 终结点。你可以为角色启用多个终结点，但只能选择其中一个来支持粘性会话。
    1. 重新生成你的应用程序。

启用后，如果你有多个角色实例，则来自特定客户端的 HTTP 请求将继续由同一角色实例处理。

Eclipse 工具包是通过在角色实例中安装名为应用程序请求路由 (ARR) 的特殊 IIS 模块来做到这一点的。ARR 将 HTTP 请求重新路由到相应的角色实例。该工具包将自动重新配置选定的终结点，使传入的 HTTP 流量先路由到 ARR 软件。该工具包还会创建 Java 服务器配置为侦听的新内部终结点。ARR 正是使用此终结点将 HTTP 流量重新路由到相应的角色实例。这样，多实例部署中的每个角色实例将充当所有其他实例的反向代理，从而实现了粘性会话。

## 有关会话相关性的说明 ##

* 会话相关性在计算模拟器中不起作用。可以在计算模拟器中应用这些设置而不会干扰你的生成过程或计算模拟器执行，但该功能本身在计算模拟器中不能正常运行。
* 启用会话相关性会导致部署在 Azure 中占用更多的磁盘空间量，因为当你的服务在 Azure 云中启动时，会在角色实例中下载并安装其他软件。
* 初始化每个角色需要更长的时间。
* 将会添加一个充当上述流量路由器的内部终结点。

有关如何在启用会话相关性的情况下维护会话数据的示例，请参阅[如何使用会话相关性来维护会话数据][]。

## 另请参阅 ##

[适用于 Eclipse 的 Azure 工具包][]

[在 Eclipse 中为 Azure 创建 Hello World 应用程序][]

[安装 Azure Toolkit for Eclipse][]

[如何使用会话相关性来维护会话数据][]

有关将 Azure 与 Java 配合使用的详细信息，请参阅 [Azure Java 开发人员中心][]。

<!-- URL List -->

[Azure Java 开发人员中心]: /develop/java/
[适用于 Eclipse 的 Azure 工具包]: /documentation/articles/azure-toolkit-for-eclipse/
[在 Eclipse 中为 Azure 创建 Hello World 应用程序]: /documentation/articles/azure-toolkit-for-eclipse-creating-a-hello-world-application/
[如何使用会话相关性来维护会话数据]: /develop/java/
[安装 Azure Toolkit for Eclipse]: /documentation/articles/azure-toolkit-for-eclipse-installation/

<!-- IMG List -->

[ic719492]: ./media/azure-toolkit-for-eclipse-enable-session-affinity/ic719492.png

<!---HONumber=Mooncake_0215_2016-->