<properties
   pageTitle="在 mbed 上使用 C 连接设备 | Azure"
   description="介绍如何使用在 mbed 设备上运行的以 C 编写的应用程序将设备连接到 Azure IoT 套件预配置远程监视解决方案。"
   services=""
   suite="iot-suite"
   documentationCenter="na"
   authors="dominicbetts"
   manager="timlt"
   editor=""/>  


<tags
   ms.service="iot-suite"
   ms.devlang="na"
   ms.topic="article"
   ms.tgt_pltfrm="na"
   ms.workload="na"
   ms.date="01/04/2017"
   wacn.date="01/25/2017"
   ms.author="dobett"/>


# 将设备连接到远程监视预配置解决方案 (mbed)

[AZURE.INCLUDE [iot-suite-selector-connecting](../../includes/iot-suite-selector-connecting.md)]

## 生成并运行 C 示例解决方案

以下说明介绍用于将[启用 mbed 的 Freescale FRDM-K64F][lnk-mbed-home] 设备连接到远程监视解决方案的步骤。

### 将 mbed 设备连接到网络和桌面计算机

1. 使用以太网电缆将 mbed 设备连接到网络。此步骤是必需的，因为示例应用程序需要 Internet 访问。

2. 请参阅 [mbed 入门][lnk-mbed-getstarted]以将 mbed 设备连接到桌面 PC。

3. 如果桌面 PC 在运行 Windows，请参阅 [PC 配置][lnk-mbed-pcconnect]以配置对 mbed 设备的串行端口访问。

### 创建 mbed 项目并导入示例代码

1. 在 Web 浏览器中，转到 mbed.org [开发人员站点](https://developer.mbed.org/)。如果尚未注册，则会看到一个用于创建帐户的选项（它是免费的）。否则，使用你的帐户凭据登录。然后在页面右上角单击“编译器”。通过此操作将转到“工作区”界面。

2. 请确保你使用的硬件平台出现在窗口右上角，或单击右角的图标以选择你的硬件平台。

3. 在主菜单上单击“导入”。然后单击“单击此处”从 mbed 球形徽标旁的 URL 链接导入。

    ![][6]  


4. 在弹出窗口中，输入示例代码 https://developer.mbed.org/users/AzureIoTClient/code/remote_monitoring/ 的链接，然后单击“导入”。

    ![][7]  


5. 可以在 mbed 编译器窗口中看到导入此项目也会导入各种库。一些库由 Azure IoT 团队提供并维护（azureiot\_common、[iothub\_client](https://developer.mbed.org/users/AzureIoTClient/code/iothub_client/)、[iothub\_amqp\_transport](https://developer.mbed.org/users/AzureIoTClient/code/iothub_amqp_transport/)、azure\_uamqp），而另其他库是 mbed 库目录中可用的第三方库。

    ![][8]  


6. 打开 remote\_monitoring\\remote\_monitoring.c 文件并在该文件中找到以下代码：

    ```
    static const char* deviceId = "[Device Id]";
    static const char* deviceKey = "[Device Key]";
    static const char* hubName = "[IoTHub Name]";
    static const char* hubSuffix = "[IoTHub Suffix, i.e. azure-devices.cn]";
    ```

7. 将 [Device Id] 和 [Device Key] 替换为设备数据以便使示例程序可以连接到 IoT 中心。使用 IoT 中心主机名替换 [IoTHub Name] 和 [IoTHub Suffix, i.e. azure-devices.cn] 占位符。例如，如果 IoT 中心主机名是 **contoso.azure-devices.cn**，则 **contoso** 是 **hubName**，其后的所有内容是 **hubSuffix**：

    ```
    static const char* deviceId = "mydevice";
    static const char* deviceKey = "mykey";
    static const char* hubName = "contoso";
    static const char* hubSuffix = "azure-devices.cn";
    ```

    ![][9]  


### 演练代码
如果对程序的工作原理感兴趣，此部分中介绍了示例代码的一些关键部分。若只需运行代码，请跳到[生成并运行程序](#buildandrun)。

#### 定义模型
此示例使用[序列化程序][lnk-serializer]库定义一个模型，该模型指定设备可以发送到 IoT 中心以及从 IoT 中心接收的消息。在此示例中，**Contoso** 命名空间定义的 **Thermostat** 模型指定了以下内容：

- **温度**、**ExternalTemperature** 和**湿度**遥测数据。
- 元数据，例如设备 ID、设备属性。
- 设备对其进行响应的命令：

	BEGIN_NAMESPACE(Contoso);
	
	DECLARE_STRUCT(SystemProperties,
	    ascii_char_ptr, DeviceID,
	    _Bool, Enabled
	);
	
	DECLARE_STRUCT(DeviceProperties,
	ascii_char_ptr, DeviceID,
	_Bool, HubEnabledState
	);
	
	DECLARE_MODEL(Thermostat,
	
	    /* Event data (temperature, external temperature and humidity) */
	    WITH_DATA(int, Temperature),
	    WITH_DATA(int, ExternalTemperature),
	    WITH_DATA(int, Humidity),
	    WITH_DATA(ascii_char_ptr, DeviceId),
	
	    /* Device Info - This is command metadata + some extra fields */
	    WITH_DATA(ascii_char_ptr, ObjectType),
	    WITH_DATA(_Bool, IsSimulatedDevice),
	    WITH_DATA(ascii_char_ptr, Version),
	    WITH_DATA(DeviceProperties, DeviceProperties),
	    WITH_DATA(ascii_char_ptr_no_quotes, Commands),
	
	    /* Commands implemented by the device */
	    WITH_ACTION(SetTemperature, int, temperature),
	    WITH_ACTION(SetHumidity, int, humidity)
	);
	
	END_NAMESPACE(Contoso);


与模型定义相关的是设备响应的 **SetTemperature** 和 **SetHumidity** 命令的定义：

	
	EXECUTE_COMMAND_RESULT SetTemperature(Thermostat* thermostat, int temperature)
	{
	    (void)printf("Received temperature %d\r\n", temperature);
	    thermostat->Temperature = temperature;
	    return EXECUTE_COMMAND_SUCCESS;
	}
	
	EXECUTE_COMMAND_RESULT SetHumidity(Thermostat* thermostat, int humidity)
	{
	    (void)printf("Received humidity %d\r\n", humidity);
	    thermostat->Humidity = humidity;
	    return EXECUTE_COMMAND_SUCCESS;
	}


#### 将模型连接到库

函数 **sendMessage** 和 **IoTHubMessage** 是样板代码，用于从设备发送遥测数据以及将来自 IoT 中心的消息连接到命令处理程序。

#### remote\_monitoring\_run 函数

程序的 **main** 函数会在应用程序开始作为 IoT 中心设备客户端来执行设备行为时调用 **remote\_monitoring\_run** 函数。此 **remote\_monitoring\_run** 函数主要由嵌套函数对组成：

- **platform\_init** 和 **platform\_deinit** 执行平台特定初始化和关闭操作。
- **serializer\_init** 和 **serializer\_deinit** 初始化和取消初始化序列化程序库。
- **IoTHubClient\_Create** 和 **IoTHubClient\_Destroy** 使用用于连接到 IoT 中心的设备凭据来创建客户端句柄 **iotHubClientHandle**。

在 **remote\_monitoring\_run** 函数的主要部分中，程序会使用 **iotHubClientHandle** 句柄执行以下操作：

- 创建 Contoso 恒温器模型的实例并为两个命令设置消息回调。
- 使用序列化程序库将有关设备本身的信息（包括它支持的命令）发送到 IoT 中心。当中心收到此消息时，它会在仪表板中将设备状态从“挂起”更改为“正在运行”。
- 启动一个 **while** 循环，它会每秒将温度、外部温度和湿度值发送到 IoT 中心。

下面提供了一个在启动时发送到 IoT 中心的示例 **DeviceInfo** 消息以供参考：


	{
	  "ObjectType":"DeviceInfo",
	  "Version":"1.0",
	  "IsSimulatedDevice":false,
	  "DeviceProperties":
	  {
	    "DeviceID":"mydevice01", "HubEnabledState":true
	  }, 
	  "Commands":
	  [
	    {"Name":"SetHumidity", "Parameters":[{"Name":"humidity","Type":"double"}]},
	    { "Name":"SetTemperature", "Parameters":[{"Name":"temperature","Type":"double"}]}
	  ]
	}


下面提供了一个发送到 IoT 中心的示例 **Telemetry** 消息以供参考：


	{"DeviceId":"mydevice01", "Temperature":50, "Humidity":50, "ExternalTemperature":55}


下面提供了一个从 IoT 中心接收的示例**命令**以供参考：


	{
	  "Name":"SetHumidity",
	  "MessageId":"2f3d3c75-3b77-4832-80ed-a5bb3e233391",
	  "CreatedTime":"2016-03-11T15:09:44.2231295Z",
	  "Parameters":{"humidity":23}
	}


<a id="buildandrun"></a>  

### 生成并运行程序

1. 单击“编译”以生成程序。可以安全地忽略任何警告，但是如果生成产生错误，请在更正它们之后再继续进行。

2. 如果生成成功，则 mbed 编译器网站会生成一个具有项目名称的 .bin 文件，并将它下载到本地计算机。将 .bin 文件复制到设备。将 .bin 文件保存到设备会导致设备重新启动并运行 .bin 文件中包含的程序。可以通过在 mbed 设备上按重置按钮，来随时手动重新启动程序。

3. 使用 SSH 客户端应用程序（如 PuTTY）连接到设备。可以通过检查 Windows 设备管理器来确定设备使用的串行端口。

    ![][11]  


4. 在 PuTTY 中，单击“串行”连接类型。设备通常以 9600 波特进行连接，因此在“速度”框中输入 9600。然后单击“打开”。

5. 程序开始执行。如果程序未在连接时自动启动，则可能必须重置板（按 CTRL+Break 或按板的重置按钮）。

    ![][10]  


[AZURE.INCLUDE [iot-suite-visualize-connecting](../../includes/iot-suite-visualize-connecting.md)]


[6]: ./media/iot-suite-connecting-devices-mbed/mbed1.png
[7]: ./media/iot-suite-connecting-devices-mbed/mbed2a.png
[8]: ./media/iot-suite-connecting-devices-mbed/mbed3a.png
[9]: ./media/iot-suite-connecting-devices-mbed/suite6.png
[10]: ./media/iot-suite-connecting-devices-mbed/putty.png
[11]: ./media/iot-suite-connecting-devices-mbed/mbed6.png

[lnk-mbed-home]: https://developer.mbed.org/platforms/FRDM-K64F/
[lnk-mbed-getstarted]: https://developer.mbed.org/platforms/FRDM-K64F/#getting-started-with-mbed
[lnk-mbed-pcconnect]: https://developer.mbed.org/platforms/FRDM-K64F/#pc-configuration
[lnk-serializer]: /documentation/articles/iot-hub-device-sdk-c-intro/#serializer

<!---HONumber=Mooncake_0120_2017-->
<!--Update_Description:update wording-->