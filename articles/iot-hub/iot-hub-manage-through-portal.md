<properties
	 pageTitle="使用 Azure 门户管理 IoT 中心 | Azure"
	 description="概述如何通过 Azure 门户创建和管理 Azure IoT 中心"
	 services="iot-hub"
	 documentationCenter=""
	 authors="nasing"
	 manager="timlt"
	 editor=""/>

<tags
	 ms.service="iot-hub"
	 ms.date="08/11/2016"
	 wacn.date="08/29/2016"/>

# 通过 Azure 门户管理 IoT 中心

## 介绍

本文介绍如何通过 Azure 门户开始使用 Azure IoT 中心、如何查找 IoT 中心以及如何创建和管理 IoT 中心。

## 在哪里可以找到 IoT 中心

可在多个位置找到 IoT 中心。

1. **+ 新建**：**Azure IoT 中心**是一种 IoT 服务，类似于其他服务，你可以在“+ 新建”下面的“物联网”类别中找到它。

2. 也可以通过应用商店访问 IoT 中心，它已列为“物联网”下面的主流服务。

## 创建 IoT 中心

可以使用以下方法创建 IoT 中心。

1. 通过“+ 新建”选项创建 IoT 中心时，会出现以下屏幕截图所示的边栏选项卡。通过此方法和通过应用商店创建 IoT 中心的步骤完全相同。

2. 通过应用商店创建 IoT 中心：单击“创建”打开边栏选项卡，与前面使用“+ 新建”时显示的边栏选项卡完全相同。后续部分列出了涉及创建 IoT 中心的几个步骤。

### 选择 IoT 中心的名称

若要创建 IoT 中心，必须为中心命名。请注意，此名称必须是所有中心之间保持唯一。后端不允许重复的中心，因此，中心的名称越独特越好。

### 选择定价层

可以从四个层中做选择：**免费**、**标准 1**、**标准 2** 和**标准 S3**。免费层只允许 500 台设备连接到 IoT 中心，并且每天最多传输 8,000 条信息。

**标准 S1**：IoT 中心 S1 版专为具有大量设备，但每台设备生成的数据量相对较少的 IoT 解决方案而设计。S1 版的每个计价单位让你能够在所有已连接设备间每天传输最多 400,000 条消息。

**标准 S2**：IoT 中心 S2 版专为设备生成大量数据的 IoT 解决方案而设计。S2 版的每个计价单位让你能够在所有已连接设备间每天传输最多 600 万条消息。

**标准 S3**：IoT 中心 S3 版专为生成大量数据的 IoT 解决方案而设计。S3 版的每个计价单位让你能够在所有已连接设备间每天传输最多 3 亿条消息。

![][4]

> [AZURE.NOTE] IoT 中心只允许每个订阅有一个免费中心。

### IoT 中心单位

一个 IoT 中心单位每天包含一定数量的消息。选择 IoT 单位数目表示此中心支持的消息总数等于单位数乘以该层每天的消息数。例如，如果希望 IoT 中心支持 700,000 条消息输入，则选择两个 S1 层单位。

### 设备到云分区和资源组

可以更改 IoT 中心的分区数目。默认的分区设置为 4 个。但是，你可以从下拉列表中选择不同的分区数目。

对于资源组，你不需要显式创建空资源组。在创建新资源时，可以选择创建新的资源组，或使用现有的资源组。

![][5]

### 选择订阅

Azure IoT 中心自动显示用户帐户所链接的订阅列表。可以在此处选择其中一个选项，将 IoT 中心与该订阅关联。

### 选择位置

位置选项提供可在其中使用 IoT 中心的区域列表。可在以下位置部署 IoT 中心：中国东部、中国北部。

### 创建 IoT 中心

完成上述所有步骤后，可以创建 IoT 中心。单击“创建”启动使用特定选项创建此 IoT 中心的后端进程，并将 IoT 中心部署到指定的位置。

请注意，创建 IoT 中心可能需要几分钟时间，因为在适当的位置服务器中进行后端部署需要花费时间。

## 更改 IoT 中心的设置

可以在通过“IoT 中心”边栏选项卡创建 IoT 中心后更改现有 IoT 中心的设置。

![][8]

**共享访问策略**：这些策略定义设备与服务连接到 IoT 中心所需的权限。可以单击“常规”下面的“共享访问策略”来访问这些策略。在此边栏选项卡中，可以修改现有的策略或添加新策略。

### 创建新策略

- 单击“添加”打开边栏选项卡，可在其中输入新的策略名称以及想要与此策略关联的权限，如下图所示。

	有许多权限可与这些共享策略相关联。前两个策略（**注册表读取**和**注册表写入**）用于向设备标识存储或标识注册表授予读取和写入访问权限。请注意，选择写入选项会自动选择读取选项。

 	“服务连接”策略向连接到 IoT 中心的服务授予访问云端终结点（例如使用者组）的权限。“设备连接”策略授予在 IoT 中心的设备端终结点上发送和接收消息的权限。

- 单击“创建”将此新建策略添加到现有列表。

![][10]

## 消息传送

单击“消息传送”可显示正在修改的 IoT 中心的消息传送属性列表。可修改或复制两种主要类型的属性：“云到设备”和“设备到云”。

- “云到设备”设置：此设置两项子设置：消息的“云到设备 TTL”（生存时间）和“保留时间”。首次创建 IoT 中心时，这两项设置的默认值为一小时。若要调整这些值，请使用滑块或键入值。

- “设备到云”设置：此设置有多项子设置，其中一些已在创建 IoT 中心时命名/分配，并且只能复制到其他可自定义的子设置。下一部分列出了这些子设置。

**分区**：此值在创建 IoT 中心时设置，并可通过此设置进行更改。

**事件中心兼容的名称和终结点**：创建 IoT 中心时，事件中心将在内部创建，在某些情况下你可能需要其访问权限。此事件中心名称和终结点无法自定义，但可通过“复制”按钮使用。

**保留时间**：默认设置为一天，但可以使用下拉列表自定义为其他值。请注意，此值对于“设备到云”而言是以天为单位，而不是像“云到设备”那样以小时为单位。

**使用者组**：使用者组是一种类似于其他消息传送系统的设置，可用于通过特定方式拉取数据，以将其他应用程序或服务连接到 IoT 中心。创建的每个 IoT 中心都包含一个默认使用者组。但是，你可以在 IoT 中心添加或删除使用者组。

> [AZURE.NOTE] 无法编辑或删除默认使用者组。

![][11]

## 文件上载

若要使用 IoT 中心的文件上传功能，必须先将 Azure 存储帐户与你的中心关联。选择“文件上传”设置，即可显示正在修改的 IoT 中心的文件上传属性列表。

**存储容器**：使用门户在当前订阅中选择存储帐户中的 Blob 容器，以便与 IoT 中心关联。如有必要，可以在“存储帐户”边栏选项卡上创建新的存储帐户，并在“容器”边栏选项卡上创建新的 Blob 容器。IoT 中心会自动生成对此 Blob 容器具有写入权限的 SAS URI，以供设备上传文件时使用。

![][14]

**接收已上传文件的通知**：通过切换来启用或禁用文件上传通知。

**SAS TTL**：此设置是 IoT 中心返回给设备的 SAS URI 生存时间。默认设置为一小时，但可以使用滑块自定义为其他值。

**文件通知设置默认 TTL**：文件上传通知到期前的生存时间。默认设置为一天，但可以使用滑块自定义为其他值。

**文件通知最大传送数**：IoT 中心将尝试传送文件上载通知的次数。默认设置为 10，但可以使用滑块自定义为其他值。

![][13]

## 定价和缩放

现有 IoT 中心的定价可通过“定价”设置来更改，但存在以下例外情况：

- 在当前的实现中，使用免费 SKU 的 IoT 中心无法更改为付费型 SKU 层，反之亦然。
- 订阅中只能有一个免费层 IoT 中心。

![][12]

只有在当天发送的消息数目不冲突时，才允许从较高层（S2 或 S3）转到较低层（S1 或 S2）。例如，如果每天的消息数目超过 400,000 条，则可更改 IoT 中心的层级；但如果更改为 S1 层，中心将在当天进行限制。

## 删除 IoT 中心

你可以浏览到想要删除的 IoT 中心，方法是单击“浏览”，然后选择要删除的相应中心。单击中心名称下面的“删除”按钮即可删除该中心。

## 后续步骤

若要了解有关如何管理 Azure IoT 中心的详细信息，请参阅以下链接：

- [批量管理 IoT 设备][lnk-bulk]
- [使用指标][lnk-metrics]
- [操作监视][lnk-monitor]
- [管理 IoT 中心的访问权限][lnk-itpro]

若要进一步探索 IoT 中心的功能，请参阅：

- [设计你的解决方案][lnk-design]
- [开发人员指南][lnk-devguide]
- [使用 UI 示例探索设备管理][lnk-dmui]
- [使用网关 SDK 模拟设备][lnk-gateway]


[4]: ./media/iot-hub-manage-through-portal/create-iothub.png
[5]: ./media/iot-hub-manage-through-portal/location1.png
[8]: ./media/iot-hub-manage-through-portal/portal-settings.png
[10]: ./media/iot-hub-manage-through-portal/shared-access-policies.png
[11]: ./media/iot-hub-manage-through-portal/messaging-settings.png
[12]: ./media/iot-hub-manage-through-portal/pricing-error.png
[13]: ./media/iot-hub-manage-through-portal/file-upload-settings.png
[14]: ./media/iot-hub-manage-through-portal/file-upload-container-selection.png

[lnk-get-started]: /documentation/articles/iot-hub-csharp-csharp-getstarted/
[Azure IoT 中心是什么？]: /documentation/articles/iot-hub-what-is-iot-hub/
[lnk-bulk]: /documentation/articles/iot-hub-bulk-identity-mgmt/
[lnk-metrics]: /documentation/articles/iot-hub-metrics/
[lnk-monitor]: /documentation/articles/iot-hub-operations-monitoring/
[lnk-itpro]: /documentation/articles/iot-hub-itpro-info/

[lnk-design]: /documentation/articles/iot-hub-guidance/
[lnk-devguide]: /documentation/articles/iot-hub-devguide/
[lnk-dmui]: /documentation/articles/iot-hub-device-management-ui-sample/
[lnk-gateway]: /documentation/articles/iot-hub-linux-gateway-sdk-simulated-device/
[lnk-securing]: /documentation/articles/iot-hub-security-ground-up/
<!---HONumber=Mooncake_0418_2016-->