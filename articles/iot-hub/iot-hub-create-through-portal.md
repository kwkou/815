<properties
    pageTitle="使用 Azure 门户创建 IoT 中心 | Azure"
    description="如何通过 Azure 门户创建、管理和删除 Azure IoT 中心。包括有关定价层、缩放、安全性和消息传递配置的信息。"
    services="iot-hub"
    documentationcenter=""
    author="dominicbetts"
    manager="timlt"
    editor="" />
<tags
    ms.assetid="0909cd2b-4c1e-49e0-b68a-75532caf0a6a"
    ms.service="iot-hub"
    ms.devlang="na"
    ms.topic="article"
    ms.tgt_pltfrm="na"
    ms.workload="na"
    ms.date="01/31/2017"
    wacn.date="03/10/2017"
    ms.author="dobett" />  


# 使用 Azure 门户预览创建 IoT 中心

[AZURE.INCLUDE [iot-hub-resource-manager-selector](../../includes/iot-hub-resource-manager-selector.md)]

## 介绍
本文介绍如何在 Azure 门户预览中查找 IoT 中心服务，以及如何创建和管理 IoT 中心。

## 在哪里可以找到 IoT 中心
可在多个位置找到 IoT 中心。

1. **+ 新建**：**Azure IoT 中心**是一种 IoT 服务，类似于其他服务，可以在“+ 新建”下面的“物联网”类别中找到它。
2. 也可以通过应用商店访问 IoT 中心，它已列为“物联网”下面的主流服务。

## 创建 IoT 中心
可以使用以下方法创建 IoT 中心：

* 通过“+ 新建”选项创建 IoT 中心时，会出现以下屏幕截图所示的边栏选项卡。通过此方法和通过应用商店创建 IoT 中心的步骤完全相同。
* 通过应用商店创建 IoT 中心：单击“创建”打开边栏选项卡，与前面使用“+ 新建”时显示的边栏选项卡完全相同。后续部分列出了涉及创建 IoT 中心的几个步骤。

### 选择 IoT 中心的名称
若要创建 IoT 中心，必须为 IoT 中心命名。此名称必须在所有 IoT 中心保持唯一。解决方案后端不允许中心重复，因此，中心的名称越独特越好。

### 选择定价层
可以从四个层中做选择：**免费**、**标准 1**、**标准 2** 和**标准 S3**。免费层只允许 500 台设备连接到 IoT 中心，并且每天最多传输 8,000 条信息。

**标准 S1**：IoT 中心 S1 版专为具有大量设备，但每台设备生成的数据量相对较少的 IoT 解决方案而设计。S1 版的每个计价单位让你能够在所有已连接设备间每天传输最多 400,000 条消息。

**标准 S2**：IoT 中心 S2 版专为设备生成大量数据的 IoT 解决方案而设计。S2 版的每个计价单位让你能够在所有已连接设备间每天传输最多 600 万条消息。

**标准 S3**：IoT 中心 S3 版专为生成大量数据的 IoT 解决方案而设计。S3 版的每个计价单位让你能够在所有已连接设备间每天传输最多 3 亿条消息。

![][4]  


> [AZURE.NOTE]
> IoT 中心只允许每个 Azure 订阅有一个免费中心。
> 
> 

### IoT 中心单位
每个单位每日允许的消息数取决于中心的定价层。例如，如果希望 IoT 中心支持 700,000 条消息输入，则选择两个 S1 层单位。

### 设备到云分区和资源组
可以更改 IoT 中心的分区数目。默认的分区设置为 4 个。但是，你可以从下拉列表中选择不同的分区数目。

对于资源组，你不需要显式创建空资源组。在创建资源时，可以选择创建新的资源组，或使用现有的资源组。

![][5]  


### 选择订阅
Azure IoT 中心自动显示用户帐户所链接的 Azure 订阅列表。可在此处选择其中一个选项，将 IoT 中心与该 Azure 订阅关联。

### 选择位置

位置选项提供可在其中使用 IoT 中心的区域列表。IoT 中心可以部署在以下位置：中国东部、中国北部。

### 创建 IoT 中心
完成上述所有步骤后，可以创建 IoT 中心。单击“创建”启动使用特定选项创建此 IoT 中心的后端进程，并将 IoT 中心部署到指定的位置。

创建 IoT 中心可能需要几分钟时间，因为在适当的位置服务器中进行后端部署需要花费时间。

## 更改 IoT 中心的设置
可以在通过“IoT 中心”边栏选项卡创建 IoT 中心后更改现有 IoT 中心的设置。

![][8]  


**共享访问策略**：这些策略定义设备与服务连接到 IoT 中心所需的权限。可单击“常规”下面的“共享访问策略”来访问这些策略。在此边栏选项卡中，可以修改现有的策略或添加新策略。

### 创建策略
* 单击“添加”打开边栏选项卡。可在此处输入新的策略名称以及想要与此策略关联的权限，如下图所示：
  
    有许多权限可与这些共享策略相关联。前两个策略（**注册表读取**和**注册表写入**）用于向设备标识存储或标识注册表授予读取和写入访问权限。选择写入选项会自动选择读取选项。
  
     “服务连接”策略向连接到 IoT 中心的服务授予访问云端终结点（例如使用者组）的权限。“设备连接”策略授予在 IoT 中心的设备端终结点上发送和接收消息的权限。
* 单击“创建”将此新建策略添加到现有列表。

![][10]  


## 终结点
单击“终结点”可显示要修改的 IoT 中心的终结点列表。有两种类型的终结点：已内置到 IoT 中心的终结点，以及在创建后添加到 IoT 中心的终结点。

### 内置终结点
有两个内置终结点：**云到设备反馈**和**事件**。

* **云到设备反馈**设置：此设置有两项子设置：消息的**云到设备 TTL**（生存时间）和**保留时间**（以小时为单位）。首次创建 IoT 中心时，这两项设置具有一小时的默认值。若要调整这些设置，请使用滑块或键入值。
* **事件**设置：此设置具有多个子设置，其中一些子设置是只读的。以下列表说明了这些设置：

    * **分区**：创建 IoT 中心时，将设置默认值。可以通过此设置更改分区数。

    * **与事件中心兼容的名称和终结点**：创建 IoT 中心时，会在内部创建事件中心，供用户在特定情况下根据需要访问。无法自定义与事件中心兼容的名称和终结点值，但可以通过单击“复制”复制它们。

    * **保留时间**：默认情况下设置为一天，但可以使用下拉列表更改它。对于设备到云的设置，此值以天为单位。

    * **使用者组**：使用者组是一种类似于其他消息系统的设置，可用于通过特定方式拉取数据，以将其他应用程序或服务连接到 IoT 中心。创建的每个 IoT 中心都包含一个默认使用者组。但是，可以使用此设置在 IoT 中心添加或删除使用者组。

> [AZURE.NOTE]
> 无法编辑或删除默认使用者组。
> 
> 

![][11]  


### 自定义终结点
可通过门户将自定义终结点添加到 IoT 中心。在“终结点”边栏选项卡中，单击顶部的“添加”，打开“添加终结点”边栏选项卡。输入所需的信息，然后单击“确定”。现在，自定义终结点将在主要“终结点”边栏选项卡中列出。

![][13]  


有关自定义终结点的详细信息，请阅读[参考 - IoT 中心终结点][lnk-devguide-endpoints]。

## 路由
单击“路由”，管理 IoT 中心发送设备到云消息的方式。

![][14]  


单击“路由”边栏选项卡顶部的“添加”，*输入所需的信息，然后单击“确定”**，即可将路由添加到 IoT 中心。然后，路由将在主要“路由”**边栏选项卡中列出。可通过在路由列表中单击路由来编辑路由。若要启用路由，请在路由列表中单击它，然后将“启用”**切换到“关”**。单击边栏选项卡底部的“确定”**，保存更改。

![][15]  


## 定价和缩放
现有 IoT 中心的定价可通过“定价”设置来更改，但存在以下例外情况：

- 在当前的实现中，使用免费 SKU 的 IoT 中心无法更改为付费型 SKU 层，反之亦然。
- Azure 订阅中只能有一个免费层 IoT 中心。

![][12]  


只有在当天发送的消息数目不冲突时，才允许从较高层（S2 或 S3）转到较低层（S1 或 S2）。例如，如果每天的消息数目超过 400,000，则可更改 IoT 中心的层。但是，如果更改为 S1 层，则会在当天对 IoT 中心进行限制。

## 删除 IoT 中心
你可以浏览到想要删除的 IoT 中心，方法是单击“浏览”，然后选择要删除的相应中心。单击 IoT 中心名称下面的“删除”按钮即可删除该 IoT 中心。

## 后续步骤

若要了解有关如何管理 Azure IoT 中心的详细信息，请参阅以下链接：

- [批量管理 IoT 设备][lnk-bulk]
- [IoT 中心指标][lnk-metrics]
- [操作监视][lnk-monitor]

若要进一步探索 IoT 中心的功能，请参阅：

- [IoT 中心开发人员指南][lnk-devguide]
- [使用 IoT 网关 SDK 模拟设备][lnk-gateway]


  [4]: ./media/iot-hub-create-through-portal/create-iothub.png
  [5]: ./media/iot-hub-create-through-portal/location1.png
  [8]: ./media/iot-hub-create-through-portal/portal-settings.png
  [10]: ./media/iot-hub-create-through-portal/shared-access-policies.png
  [11]: ./media/iot-hub-create-through-portal/messaging-settings.png
  [12]: ./media/iot-hub-create-through-portal/pricing-error.png
[13]: ./media/iot-hub-create-through-portal/endpoint-creation.png
[14]: ./media/iot-hub-create-through-portal/routes-list.png
[15]: ./media/iot-hub-create-through-portal/route-edit.png

[lnk-bulk]: /documentation/articles/iot-hub-bulk-identity-mgmt/
[lnk-metrics]: /documentation/articles/iot-hub-metrics/
[lnk-monitor]: /documentation/articles/iot-hub-operations-monitoring/

[lnk-devguide]: /documentation/articles/iot-hub-devguide/
[lnk-gateway]: /documentation/articles/iot-hub-linux-gateway-sdk-simulated-device/
[lnk-devguide-endpoints]: /documentation/articles/iot-hub-devguide-endpoints/

<!---HONumber=Mooncake_0306_2017-->
<!--Update_Description:update wording-->