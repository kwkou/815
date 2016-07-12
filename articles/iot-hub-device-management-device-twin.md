<properties
	pageTitle="IoT 中心设备管理设备克隆 | Azure"
	description="Azure IoT 中心设备管理教程，描述如何使用设备克隆。"
	services="iot-hub"
	documentationCenter=".net"
	authors="ellenfosborne"
	manager="timlt"
	editor=""/>

<tags
 ms.service="iot-hub"
 ms.date="04/29/2016"
 wacn.date="05/30/2016"/>

# 教程：如何将设备克隆用于 C#（预览版）

[AZURE.INCLUDE [iot-hub-device-management-twin-selector](../includes/iot-hub-device-management-twin-selector.md)]
## 介绍

Azure IoT 中心设备管理引入了设备克隆，它是物理设备的一种服务方表示形式。下图显示了设备克隆的各个组件。

![][img-twin]

在本教程中，我们重点关注设备属性。若要了解有关其他组件的详细信息，请参阅 [Azure IoT 中心设备管理概述][lnk-dm-overview]。

设备属性是预定义的属性字典，用于描述物理设备。物理设备是每个设备属性的主机，是每个对应值的权威存储位置。这些属性的“最终一致”表示形式存储在云中的设备克隆中。相干性和新鲜度受同步设置影响，如下所述。设备属性的一些示例包括固件版本、电池剩余电量和制造商名称。

## 设备属性同步

物理设备是设备属性的权威来源。通过 [LWM2M][lnk-lwm2m] 所述的观察/通知模式，将物理设备上的所选值同步到 IoT 中心内的设备克隆。

当物理设备连接到 IoT 中心时，该服务对所选设备属性启动观察。然后，物理设备将设备属性的更改通知 IoT 中心。为实现滞后，已将 **pmin**（通知之间的最短时间）设置为 5 分钟。这意味着对于每个属性，物理设备通知 IoT 中心的频率不会高于每 5 分钟一次，即使在此期间发生了更改。为确保新鲜度，已将 **pmax**（通知之间的最长时间）设置为 6 小时。这意味着对于每个属性，物理设备通知 IoT 中心的频率至少应为每 6 小时一次，即使在此期间未该属性更改。

当物理设备断开连接时，同步将停止。当设备重新连接到服务时，同步将重新启动。你可以随时检查属性的上次更新时间以确保新鲜度。

下面列出了自动观察到的设备属性的完整列表：

![][img-observed]


## 运行设备克隆查询示例

下面的示例扩展了 [Azure IoT 中心设备管理入门][lnk-get-started]教程的功能。从使各模拟设备运行开始，它使用设备克隆来读取和更改模拟设备上的属性。

### 先决条件 

运行此示例之前，请务必完成 [Azure IoT 中心设备管理入门][lnk-get-started]中的步骤。这意味着必须运行模拟设备。如果你以前完成了该进程，现在请重启模拟设备。

### 启动示例

若要启动该示例，则需要运行 **DeviceTwin.exe** 进程。这将从设备克隆和物理设备读取设备属性。还将更改物理设备上的设备属性。按照以下步骤启动示例：

1.  从克隆 **azure-iot-sdks** 存储库的根文件夹，导航到 **azure-iot-sdks\\csharp\\service\\samples\\bin** 文件夹。  

2.  运行 `DeviceTwin.exe <IoT Hub Connection String>`。

应可看到命令行窗口中的输出显示了设备克隆的使用情况。此示例逐步完成了以下过程：

1.  打印出设备克隆上的全部设备属性。

2.  深度读取：从物理设备读取电池剩余电量设备属性（3 次）。

3.  深度写入：在物理设备上写入 **Timezone** 设备属性。

4.  深度读取：从物理设备读取 **Timezone** 设备属性，查看其是否已更改。

### 浅度读取

浅度读取和深度读取/写入之间存在区别。浅度读取从存储在 Azure IoT 中心内的设备克隆返回所请求属性的值。它将是上一个值通知操作所得的值。无法进行浅度写入，因为物理设备是设备属性的权威来源。浅度读取仅从设备克隆读取属性：

```
device.DeviceProperties[DevicePropertyNames.BatteryLevel].Value.ToString();
```

若要确定这些值的新鲜度，可检查上次更新时间：

```
device.DeviceProperties[DevicePropertyNames.BatteryLevel].LastUpdatedTime.ToString();
```

还可以类似方式读取服务属性，它们仅存储在设备克隆内。它们不同步到设备。

### 深度读取

深度读取将启动设备作业以从物理设备读取所请求属性的值。[Azure IoT 设备管理概述][lnk-dm-overview]中对设备作业进行了简要介绍，[教程：如何使用设备作业更新设备固件][lnk-dm-jobs]中对其进行了详细描述。深度读取将给出设备属性的较新值，因为新鲜度不受通知时间间隔的限制。该作业将向物理设备发送一条消息，并使仅为设备克隆的指定属性更新最新值。它不会刷新整个设备克隆。

```
JobResponse jobResponse = await deviceJobClient.ScheduleDevicePropertyReadAsync(Guid.NewGuid().ToString(), deviceId, propertyToRead);
```

无法对服务属性或标记进行深度读取，因为它们不同步到设备。

### 深度写入

如果你想更改可写设备属性，可以通过深度写入执行此操作，它将启动一个设备作业以在物理设备上写入值。并非所有设备属性都是可写的。有关完整列表，请参阅 [Azure IoT 中心设备管理客户端库介绍][lnk-dm-library]的附录 A。

该作业将向物理设备发送一条消息，以更新指定属性。作业完成时，设备克隆不会立即更新。必须等到下一个通知时间间隔。一旦发生同步，即可通过浅度读取查看设备克隆中的更改。

```
JobResponse jobResponse = await deviceJobClient.ScheduleDevicePropertyWriteAsync(Guid.NewGuid().ToString(), deviceId, propertyToSet, setValue); TODO
```

### 设备模拟器实现详细信息

让我们研究一下要在设备方执行哪些操作才能实现观察/通知模式和深度读取/写入。

由于设备属性同步完全是通过 Azure IoT 中心 DM 客户端库来处理的，因此你只需定期调用 API 以设置设备属性（在此示例中即为电池剩余电量）即可。当该服务进行深度读取时，将返回你设置的上一个值。当该服务进行深度写入时，将调用此 set 方法。在 **iotdm\_simple\_sample.c** 中可以看到此示例：

```
int level = get_batterylevel();  // call to platform specific code 
set_device_batterylevel(0, level);
```

若不使用 set 方法，可实现一个回调。有关此选项的其他信息，请参阅 [Azure IoT 中心设备管理库介绍][lnk-dm-library]。

## 后续步骤

若要了解有关 Azure IoT 中心设备管理功能的详细信息，可学习以下教程：

- [如何使用查询查找设备克隆][lnk-tutorial-queries]

- [如何使用设备作业更新设备固件][lnk-dm-jobs]

- 设备管理客户端库提供了使用 [Intel Edison 设备][lnk-edison]的端到端示例。



<!-- images and links -->
[img-twin]: ./media/iot-hub-device-management-device-twin/image1.png
[img-observed]: ./media/iot-hub-device-management-device-twin/image2.png

[lnk-lwm2m]: http://technical.openmobilealliance.org/Technical/technical-information/release-program/current-releases/oma-lightweightm2m-v1-0
[lnk-dm-overview]: /documentation/articles/iot-hub-device-management-overview/
[lnk-dm-library]: /documentation/articles/iot-hub-device-management-library/
[lnk-get-started]: /documentation/articles/iot-hub-device-management-get-started/
[lnk-tutorial-queries]: /documentation/articles/iot-hub-device-management-device-query/
[lnk-dm-jobs]: /documentation/articles/iot-hub-device-management-device-jobs/
[lnk-edison]: https://github.com/Azure/azure-iot-sdks/tree/dmpreview/c/iotdm_client/samples/iotdm_edison_sample

<!---HONumber=Mooncake_0523_2016-->