<properties
   pageTitle="从服务器资源管理器访问 Azure 虚拟机 | Azure"
   description="概述如何在 Visual Studio 的服务器资源管理器中查看、创建和管理 Azure 虚拟机 (VM)。"
   services="visual-studio-online"
   documentationCenter="na"
   authors="TomArcher"
   manager="douge"
   editor="" />

<tags
    ms.assetid="eb3afde6-ba90-4308-9ac1-3cc29da4ede0"
    ms.service="multiple"
    ms.devlang="dotnet"
    ms.topic="article"
    ms.tgt_pltfrm="na"
    ms.workload="multiple"
    ms.date="11/18/2016"
    wacn.date="02/04/2017"
    ms.author="tarcher" />

# 从服务器资源管理器访问 Azure 虚拟机

通过使用 Visual Studio 中的服务器资源管理器可以显示有关 Azure 托管的虚拟机的信息。

## 在服务器资源管理器中访问 Azure 虚拟机

如果你有 Azure 托管的虚拟机，可以在服务器资源管理器中访问它们。必须先登录 Azure 订阅才能查看移动服务。若要登录，请在服务器资源管理器中打开“Azure”节点的快捷菜单，并选择“连接到 Azure”。

### 获取有关虚拟机的信息

1. 在服务器资源管理器中选择虚拟机，然后选择 F4 键显示其属性窗口。

    下表显示了可用的属性，但这些属性都是只读的。若要更改属性，请使用 [Azure 经典管理门户](http://manage.windowsazure.cn)。

  	|属性|说明|
  	|---|---|
  	|DNS 名称|包含虚拟机 Internet 地址的 URL。|
  	|环境|对于虚拟机，此属性的值始终为“生产”。|
  	|名称|虚拟机的名称。|
  	|大小|虚拟机的大小，此值反映可用的内存和磁盘空间量。有关详细信息，请参阅“How To: Configure Virtual Machine Sizes”（如何：配置虚拟机大小）。|
  	|状态|值包括“正在启动”、“已启动”、“正在停止”、“已停止”和“正在检索状态”。如果出现“正在检索状态”，则表示当前状态未知。此属性的值不同于 [Azure 经典管理门户](http://manage.windowsazure.cn)上使用的值。|
  	|SubscriptionID|你的 Azure 帐户的订阅 ID。可以通过在 [Azure 经典管理门户](http://manage.windowsazure.cn)上查看订阅的属性来显示此信息。|

1. 选择一个终结点节点，然后查看“属性”窗口。

1. 下表描述了可用的终结点属性，但这些属性都是只读的。若要添加或编辑虚拟机的终结点，请使用 [Azure 经典管理门户](http://manage.windowsazure.cn)。

  	|属性|说明|
  	|---|---|
  	|名称|终结点的标识符。|
  	|专用端口|应用程序的内部网络访问端口。|
  	|协议|此终结点的传输层使用的协议：TCP 或 UDP。|
  	|公用端口|用于公开访问应用程序的端口。|



<!---HONumber=Mooncake_0620_2016-->