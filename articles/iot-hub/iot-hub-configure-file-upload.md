<properties
	 pageTitle="使用 Azure 门户配置文件上载 | Azure"
	 description="使用 Azure 门户配置文件上载概述"
	 services="iot-hub"
	 documentationCenter=""
	 authors="dominicbetts"
	 manager="timlt"
	 editor=""/>  


<tags
	 ms.service="iot-hub"
	 ms.devlang="na"
	 ms.topic="article"
	 ms.tgt_pltfrm="na"
	 ms.workload="na"
	 ms.date="09/30/2016"
	 wacn.date="12/12/2016"
	 ms.author="dobett"/>  


# 通过 Azure 门户配置文件上载

## 文件上载

若要使用 [IoT 中心的文件上载功能][lnk-upload]，必须先将 Azure 存储帐户与中心关联。选择“文件上传”设置，即可显示正在修改的 IoT 中心的文件上传属性列表。

![][13]  


**存储容器**：使用 Azure 门户在当前 Azure 订阅中选择 Azure 存储帐户中的 Blob 容器，以便与 IoT 中心关联。如有必要，可在“存储帐户”边栏选项卡上创建 Azure 存储帐户，并在“容器”边栏选项卡上创建 Blob 容器。IoT 中心会自动生成对此 Blob 容器具有写入权限的 SAS URI，以供设备上传文件时使用。

![][14]

**接收已上传文件的通知**：通过切换来启用或禁用文件上传通知。

**SAS TTL**：此设置是 IoT 中心返回给设备的 SAS URI 生存时间。默认设置为一小时，但可以使用滑块自定义为其他值。

**文件通知设置默认 TTL**：文件上传通知到期前的生存时间。默认设置为一天，但可以使用滑块自定义为其他值。

**文件通知最大传送数**：IoT 中心将尝试传送文件上载通知的次数。默认设置为 10，但可以使用滑块自定义为其他值。

![][15]  


## 后续步骤
如需详细了解 IoT 中心的文件上载功能，请参阅开发人员指南中的 [Upload files from a device][lnk-upload]（从设备上载文件）。

若要了解有关如何管理 Azure IoT 中心的详细信息，请参阅以下链接：

- [批量管理 IoT 设备][lnk-bulk]
- [使用指标][lnk-metrics]
- [操作监视][lnk-monitor]

若要进一步探索 IoT 中心的功能，请参阅：

- [开发人员指南][lnk-devguide]
- [使用 IoT 网关 SDK 模拟设备][lnk-gateway]


  [13]: ./media/iot-hub-configure-file-upload/file-upload-settings.png
  [14]: ./media/iot-hub-configure-file-upload/file-upload-container-selection.png
  [15]: ./media/iot-hub-configure-file-upload/file-upload-selected-container.png

[lnk-upload]: /documentation/articles/iot-hub-devguide-file-upload/

[lnk-bulk]: /documentation/articles/iot-hub-bulk-identity-mgmt/
[lnk-metrics]: /documentation/articles/iot-hub-metrics/
[lnk-monitor]: /documentation/articles/iot-hub-operations-monitoring/

[lnk-devguide]: /documentation/articles/iot-hub-devguide/
[lnk-gateway]: /documentation/articles/iot-hub-linux-gateway-sdk-simulated-device/

<!---HONumber=Mooncake_1205_2016-->