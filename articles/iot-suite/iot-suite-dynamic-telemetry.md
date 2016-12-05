<properties
	pageTitle="使用动态遥测 | Azure"
	description="遵循本教程来了解如何配合使用动态遥测和远程监视预配置解决方案。"
	services=""
    suite="iot-suite"
	documentationCenter=""
	authors="dominicbetts"
	manager="timlt"
	editor=""/>  


<tags
     ms.service="iot-suite"
     ms.devlang="na"
     ms.topic="article"
     ms.tgt_pltfrm="na"
     ms.workload="na"
     ms.date="11/16/2016"
     wacn.date="12/05/2016"
     ms.author="dobett"/>  


# 配合使用动态遥测和远程监视预配置解决方案

## 介绍

动态遥测可让你将发送到远程监视预配置解决方案的任何遥测数据可视化。部署了预配置解决方案的模拟设备会发送温度和湿度遥测，可在仪表板上直观显示这些数据。如果你自定义现有的模拟设备、创建新的模拟设备或者将物理设备连接到预配置解决方案，则可以发送其他遥测值，例如外部温度、RPM 或风速。然后，可以在仪表板上可视化这些附加的遥测数据。

本教程使用一个简单的 Node.js 模拟设备，你可以轻松对它进行修改，以体验动态遥测。

若要完成本教程，需要：

- 一个有效的 Azure 订阅。如果没有帐户，可以创建一个试用帐户，只需几分钟即可完成。有关详细信息，请参阅 [Azure 试用][lnk_free_trial]。
- [Node.js][lnk-node] 0.12.x 或更高版本。

可以在任何操作系统上完成本教程，例如 Windows 或 Linux，只要能够在其中安装 Node.js 即可。

[AZURE.INCLUDE [iot-suite-provision-remote-monitoring](../../includes/iot-suite-provision-remote-monitoring.md)]

## 配置 Node.js 模拟设备

1. 在远程监视仪表板上，单击“+ 添加设备”，再添加自定义设备。记下 IoT 中心主机名、设备 ID 和设备密钥。在本教程稍后准备 remote\_monitoring.js 设备客户端应用程序时，需要使用它们。

2. 请确保已在开发计算机上安装 Node.js 0.12.x 或更高版本。在命令提示符或 shell 中运行 `node --version` 以检查版本。有关使用包管理器在 Linux 上安装 Node.js 的信息，请参阅 [Installing Node.js via package manager][node-linux]（通过包管理器安装 Node.js）。

3. 安装 Node.js 之后，请将最新版本的 [azure-iot-sdks][lnk-github-repo] 存储库复制到开发计算机。始终对最新版的库和示例使用**主**分支。

4. 从 [azure-iot-sdks][lnk-github-repo] 存储库的本地副本中，将以下两个文件从 node/device/samples 文件夹复制到开发计算机上的某个空文件夹：

  - packages.json
  - remote\_monitoring.js

5. 打开 remote\_monitoring.js 文件并查找以下变量定义：

    ```
    var connectionString = "[IoT Hub device connection string]";
    ```

6. 将 **[IoT Hub device connection string]** 替换为设备连接字符串。使用在步骤 1 中记下的 IoT 中心主机名、设备 ID 和设备密钥的值。设备连接字符串具有以下格式：

    ```
    HostName={your IoT Hub hostname};DeviceId={your device id};SharedAccessKey={your device key}
    ```

    如果 IoT 中心主机名是 **contoso**，而设备 ID 为 **mydevice**，则连接字符串如下所示：

    ```
    var connectionString = "HostName=contoso.azure-devices.cn;DeviceId=mydevice;SharedAccessKey=2s ... =="
    ```

7. 保存文件。在包含这些文件的文件夹中的 shell 或命令提示符处运行以下命令，以安装所需包，然后运行示例应用程序：

    ```
    npm install
    node remote_monitoring.js
    ```

## 观察动态遥测的工作动态

仪表板将显示现有模拟设备的温度与湿度遥测数据：

![默认仪表板][image1]

如果你选择在上一部分中运行的 Node.js 模拟设备，将看到温度、湿度与外部温度遥测数据：

![将外部温度添加到仪表板][image2]

远程监视解决方案将自动检测其他外部温度遥测类型，并将其添加到仪表板上的图表中。

## 添加遥测类型

下一步是将 Node.js 模拟设备生成的遥测数据替换为一组新值：

1. 在命令提示符或 shell 中键入 **Ctrl+C** 以停止 Node.js 模拟设备。

2. 在 remote\_monitoring.js 文件中，你可以查看现有温度、湿度和外部温度遥测的基本数据值。如下添加 **rpm** 的基本数据值：

    ```
    // Sensors data
    var temperature = 50;
    var humidity = 50;
    var externalTemperature = 55;
    var rpm = 200;
    ```

3. Node.js 模拟设备使用 remote\_monitoring.js 文件中的 **generateRandomIncrement** 函数，向基本数据值添加随机增量。在现有随机化后面添加一行代码，以将 **rpm** 的值随机化，如下所示：

    ```
    temperature += generateRandomIncrement();
    externalTemperature += generateRandomIncrement();
    humidity += generateRandomIncrement();
    rpm += generateRandomIncrement();
    ```

4. 将新的 rpm 值添加到设备发送给 IoT 中心的 JSON 有效负载：

    ```
    var data = JSON.stringify({
      'DeviceID': deviceId,
      'Temperature': temperature,
      'Humidity': humidity,
      'ExternalTemperature': externalTemperature,
      'RPM': rpm
    });
    ```

5. 使用以下命令运行 Node.js 模拟设备：

    ```
    node remote_monitoring.js
    ```

6. 观察仪表板中图表上显示的新 RPM 遥测类型：

![将 RPM 添加到仪表板][image3]  


> [AZURE.NOTE] 可能需要在仪表板中的“设备”页上禁用 Node.js 设备然后重新将它启用，才能立即查看更改。

## 自定义仪表板显示内容

**Device-Info** 消息可以包含设备可发送给 IoT 中心的遥测数据的相关元数据。此元数据可指定设备发送的遥测类型。修改 remote\_monitoring.js 文件中的 **deviceMetaData** 值，在 **Commands** 定义后附加 **Telemetry** 定义。以下代码片段显示 **Commands** 定义（务必在 **Commands** 定义后添加 `,`）：

```
'Commands': [{
  'Name': 'SetTemperature',
  'Parameters': [{
    'Name': 'Temperature',
    'Type': 'double'
  }]
},
{
  'Name': 'SetHumidity',
  'Parameters': [{
    'Name': 'Humidity',
    'Type': 'double'
  }]
}],
'Telemetry': [{
  'Name': 'Temperature',
  'Type': 'double'
},
{
  'Name': 'Humidity',
  'Type': 'double'
},
{
  'Name': 'ExternalTemperature',
  'Type': 'double'
}]
```

> [AZURE.NOTE] 远程监视解决方案会比较元数据定义和遥测流中的数据并区分大小写。

按以上代码片段中所述添加 **Telemetry** 定义不会影响仪表板的行为。但是，元数据也可以包含 **DisplayName** 属性来自定义仪表板中的显示内容。如以下片段中所述更新 **Telemetry** 元数据定义：

```
'Telemetry': [
{
  'Name': 'Temperature',
  'Type': 'double',
  'DisplayName': 'Temperature (C*)'
},
{
  'Name': 'Humidity',
  'Type': 'double',
  'DisplayName': 'Humidity (relative)'
},
{
  'Name': 'ExternalTemperature',
  'Type': 'double',
  'DisplayName': 'Outdoor Temperature (C*)'
}
]
```

以下屏幕截图演示了此项更改如何修改仪表板上的图表图例：

![自定义图表图例][image4]  


> [AZURE.NOTE] 可能需要在仪表板中的“设备”页上禁用 Node.js 设备然后重新将它启用，才能立即查看更改。

## 筛选遥测类型

默认情况下，仪表板上的图表显示遥测流中的每个数据系列。你可以使用 **Device-Info** 元数据来隐藏图表中的特定遥测类型。

若要使图表只显示温度和湿度遥测数据，请省略 **Device-Info** **Telemetry** 元数据中的 **ExternalTemperature**，如下所示：

```
'Telemetry': [
{
  'Name': 'Temperature',
  'Type': 'double',
  'DisplayName': 'Temperature (C*)'
},
{
  'Name': 'Humidity',
  'Type': 'double',
  'DisplayName': 'Humidity (relative)'
},
//{
//  'Name': 'ExternalTemperature',
//  'Type': 'double',
//  'DisplayName': 'Outdoor Temperature (C*)'
//}
]
```

**室外温度**不再显示在图表上：

![筛选仪表板上的遥测数据][image5]  

此更改仅影响图表的显示效果。仍会存储 **ExternalTemperature** 数据值，并可用于任何后端处理。

> [AZURE.NOTE] 可能需要在仪表板中的“设备”页上禁用 Node.js 设备然后重新将它启用，才能立即查看更改。

## 处理错误

要使某个数据流显示在图表上，其在 **Device-Info** 元数据中的 **Type** 必须与遥测值的数据类型匹配。例如，如果元数据指定湿度数据的 **Type** 必须为 **int**，而在遥测流中找到 **double**，则湿度遥测将不会显示在图表上。但是，**湿度**值仍会存储，并可供任何后端处理使用。

## 后续步骤

熟悉了动态遥测的使用方式，现在可深入了解预配置的解决方案如何使用设备信息：[远程监视预配置解决方案中的设备信息元数据][lnk-devinfo]。

[lnk-devinfo]: /documentation/articles/iot-suite-remote-monitoring-device-info/
[image1]: ./media/iot-suite-dynamic-telemetry/image1.png
[image2]: ./media/iot-suite-dynamic-telemetry/image2.png
[image3]: ./media/iot-suite-dynamic-telemetry/image3.png
[image4]: ./media/iot-suite-dynamic-telemetry/image4.png
[image5]: ./media/iot-suite-dynamic-telemetry/image5.png

[lnk_free_trial]: /pricing/1rmb-trial/
[lnk-node]: http://nodejs.org
[node-linux]: https://github.com/nodejs/node-v0.x-archive/wiki/Installing-Node.js-via-package-manager
[lnk-github-repo]: https://github.com/Azure/azure-iot-sdks

<!---HONumber=Mooncake_0815_2016-->