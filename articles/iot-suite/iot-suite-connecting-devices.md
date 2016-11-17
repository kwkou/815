<properties
   pageTitle="在 Windows 上使用 C 连接设备 | Azure"
   description="介绍如何使用在 Windows 上运行的以 C 编写的应用程序将设备连接到 Azure IoT 套件预配置远程监视解决方案。"
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
   ms.date="10/05/2016"
   wacn.date="10/31/2016"
   ms.author="dobett"/>  



# 将设备连接到远程监视预配置解决方案 (Windows)

[AZURE.INCLUDE [iot-suite-selector-connecting](../../includes/iot-suite-selector-connecting.md)]

## 在 Windows 上创建 C 示例解决方案

以下步骤演示如何使用 Visual Studio 创建一个以 C 编写的客户端应用程序，用来与远程监视预配置解决方案通信。

在 Visual Studio 2015 中创建初学者项目并添加 IoT 中心设备客户端 NuGet 包：

1. 在 Visual Studio 2015 中，使用 Visual C++ **Win32 控制台应用程序**模板创建一个 C 控制台应用程序。将该项目命名为 **RMDevice**。

2. 在“Win32 应用程序向导”中的“应用程序设置”页上，确保选中“控制台应用程序”，并取消选中“预编译头”和“安全开发生命周期(SDL)检查”。

3. 在“解决方案资源管理器”中，删除文件 stdafx.h、targetver.h 和 stdafx.cpp。

4. 在“解决方案资源管理器”中，将文件 RMDevice.cpp 重命名为 RMDevice.c。

5. 在“解决方案资源管理器”中，右键单击“RMDevice”项目，然后单击“管理 NuGet 包”。单击“浏览”，然后搜索以下 NuGet 包并将它们安装到项目中：

    - Microsoft.Azure.IoTHub.Serializer
    - Microsoft.Azure.IoTHub.IoTHubClient
    - Microsoft.Azure.IoTHub.HttpTransport

6. 在“解决方案资源管理器”中，右键单击“RMDevice”项目，然后单击“属性”打开该项目的“属性页”对话框。有关详细信息，请参阅 [Setting Visual C++ Project Properties][lnk-c-project-properties]（设置 Visual C++ 项目属性）。

7. 单击“Linker”文件夹，然后单击“输入”属性页。

8. 将 **crypt32.lib** 添加到“其他依赖项”属性。单击“确定”，然后再次单击“确定”以保存项目属性值。

## 指定 IoT 中心设备的行为

IoT 中心客户端库使用一个模型来指定设备发送到 IoT 中心的消息的格式以及从 IoT 中心接收的命令的格式。

1. 在 Visual Studio 中，打开 RMDevice.c 文件。将现有 `#include` 语句替换为以下代码：

    ```
    #include "iothubtransporthttp.h"
    #include "schemalib.h"
    #include "iothub_client.h"
    #include "serializer.h"
    #include "schemaserializer.h"
    #include "azure_c_shared_utility/threadapi.h"
    #include "azure_c_shared_utility/platform.h"
    ```

2. 在 `#include` 语句之后添加以下变量声明。将占位符值 [Device Id] 和 [Device Key] 替换为来自远程监视解决方案仪表板的设备值。使用来自仪表板的 IoT 中心主机名替换 [IoTHub Name]。例如，如果 IoT 中心主机名是 **contoso.azure-devices.cn**，则将 [IoTHub Name] 替换为 **contoso**：


	    static const char* deviceId = "[Device Id]";
	    static const char* deviceKey = "[Device Key]";
	    static const char* hubName = "[IoTHub Name]";
	    static const char* hubSuffix = "azure-devices.cn";


3. 添加以下代码以定义使设备可以与 IoT 中心通信的模型。此模型指定设备将温度、外部温度、湿度和设备 ID 作为遥测进行发送。设备还将有关设备的元数据发送到 IoT 中心（包括设备支持的命令的列表）。此设备响应命令 **SetTemperature** 和 **SetHumidity**：


	    // Define the Model
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


## 实现设备的行为

现在添加实现模型中定义的行为的代码。

1. 添加在设备从 IoT 中心接收 **SetTemperature** 和 **SetHumidity** 命令时执行的以下函数：


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


2. 添加将消息发送到 IoT 中心的以下函数：

	    static void sendMessage(IOTHUB_CLIENT_HANDLE iotHubClientHandle, const unsigned char* buffer, size_t size)
	    {
	      IOTHUB_MESSAGE_HANDLE messageHandle = IoTHubMessage_CreateFromByteArray(buffer, size);
	      if (messageHandle == NULL)
	      {
	        printf("unable to create a new IoTHubMessage\r\n");
	      }
	      else
	      {
	        if (IoTHubClient_SendEventAsync(iotHubClientHandle, messageHandle, NULL, NULL) != IOTHUB_CLIENT_OK)
	        {
	          printf("failed to hand over the message to IoTHubClient");
	        }
	        else
	        {
	          printf("IoTHubClient accepted the message for delivery\r\n");
	        }
	
	        IoTHubMessage_Destroy(messageHandle);
	      }
	      free((void*)buffer);
	    }


3. 添加挂接 SDK 中的序列化库的以下函数：

	
	    static IOTHUBMESSAGE_DISPOSITION_RESULT IoTHubMessage(IOTHUB_MESSAGE_HANDLE message, void* userContextCallback)
	    {
	      IOTHUBMESSAGE_DISPOSITION_RESULT result;
	      const unsigned char* buffer;
	      size_t size;
	      if (IoTHubMessage_GetByteArray(message, &buffer, &size) != IOTHUB_MESSAGE_OK)
	      {
	        printf("unable to IoTHubMessage_GetByteArray\r\n");
	        result = EXECUTE_COMMAND_ERROR;
	      }
	      else
	      {
	        /*buffer is not zero terminated*/
	        char* temp = malloc(size + 1);
	        if (temp == NULL)
	        {
	          printf("failed to malloc\r\n");
	          result = EXECUTE_COMMAND_ERROR;
	        }
	        else
	        {
	          memcpy(temp, buffer, size);
	          temp[size] = '\0';
	          EXECUTE_COMMAND_RESULT executeCommandResult = EXECUTE_COMMAND(userContextCallback, temp);
	          result =
	            (executeCommandResult == EXECUTE_COMMAND_ERROR) ? IOTHUBMESSAGE_ABANDONED :
	            (executeCommandResult == EXECUTE_COMMAND_SUCCESS) ? IOTHUBMESSAGE_ACCEPTED :
	            IOTHUBMESSAGE_REJECTED;
	          free(temp);
	        }
	      }
	      return result;
	    }


4. 添加以下函数以连接到 IoT 中心、发送和接收消息以及从中心断开连接。请注意设备如何在它连接之后立即将有关自身的元数据（包括它支持的命令）发送到 IoT 中心；这使解决方案可以在仪表板上将设备的状态更新为“正在运行”：


	    void remote_monitoring_run(void)
	    {
	      if (serializer_init(NULL) != SERIALIZER_OK)
	      {
	        printf("Failed on serializer_init\r\n");
	      }
	      else
	      {
	        IOTHUB_CLIENT_CONFIG config;
	        IOTHUB_CLIENT_HANDLE iotHubClientHandle;
	
	        config.deviceId = deviceId;
	        config.deviceKey = deviceKey;
	        config.iotHubName = hubName;
	        config.iotHubSuffix = hubSuffix;
	        config.protocol = HTTP_Protocol;
	        config.deviceSasToken = NULL;
	        config.protocolGatewayHostName = NULL;
	        iotHubClientHandle = IoTHubClient_Create(&config);
	        if (iotHubClientHandle == NULL)
	        {
	          (void)printf("Failed on IoTHubClient_CreateFromConnectionString\r\n");
	        }
	        else
	        {
	          Thermostat* thermostat = CREATE_MODEL_INSTANCE(Contoso, Thermostat);
	          if (thermostat == NULL)
	          {
	            (void)printf("Failed on CREATE_MODEL_INSTANCE\r\n");
	          }
	          else
	          {
	            STRING_HANDLE commandsMetadata;
	
	            if (IoTHubClient_SetMessageCallback(iotHubClientHandle, IoTHubMessage, thermostat) != IOTHUB_CLIENT_OK)
	            {
	              printf("unable to IoTHubClient_SetMessageCallback\r\n");
	            }
	            else
	            {
	
	              /* send the device info upon startup so that the cloud app knows
	              what commands are available and the fact that the device is up */
	              thermostat->ObjectType = "DeviceInfo";
	              thermostat->IsSimulatedDevice = false;
	              thermostat->Version = "1.0";
	              thermostat->DeviceProperties.HubEnabledState = true;
	              thermostat->DeviceProperties.DeviceID = (char*)deviceId;
	
	              commandsMetadata = STRING_new();
	              if (commandsMetadata == NULL)
	              {
	                (void)printf("Failed on creating string for commands metadata\r\n");
	              }
	              else
	              {
	                /* Serialize the commands metadata as a JSON string before sending */
	                if (SchemaSerializer_SerializeCommandMetadata(GET_MODEL_HANDLE(Contoso, Thermostat), commandsMetadata) != SCHEMA_SERIALIZER_OK)
	                {
	                  (void)printf("Failed serializing commands metadata\r\n");
	                }
	                else
	                {
	                  unsigned char* buffer;
	                  size_t bufferSize;
	                  thermostat->Commands = (char*)STRING_c_str(commandsMetadata);
	
	                  /* Here is the actual send of the Device Info */
	                  if (SERIALIZE(&buffer, &bufferSize, thermostat->ObjectType, thermostat->Version, thermostat->IsSimulatedDevice, thermostat->DeviceProperties, thermostat->Commands) != IOT_AGENT_OK)
	                  {
	                    (void)printf("Failed serializing\r\n");
	                  }
	                  else
	                  {
	                    sendMessage(iotHubClientHandle, buffer, bufferSize);
	                  }
	
	                }
	
	                STRING_delete(commandsMetadata);
	              }
	
	              thermostat->Temperature = 50;
	              thermostat->ExternalTemperature = 55;
	              thermostat->Humidity = 50;
	              thermostat->DeviceId = (char*)deviceId;
	
	              while (1)
	              {
	                unsigned char*buffer;
	                size_t bufferSize;
	
	                (void)printf("Sending sensor value Temperature = %d, Humidity = %d\r\n", thermostat->Temperature, thermostat->Humidity);
	
	                if (SERIALIZE(&buffer, &bufferSize, thermostat->DeviceId, thermostat->Temperature, thermostat->Humidity, thermostat->ExternalTemperature) != IOT_AGENT_OK)
	                {
	                  (void)printf("Failed sending sensor value\r\n");
	                }
	                else
	                {
	                  sendMessage(iotHubClientHandle, buffer, bufferSize);
	                }
	
	                ThreadAPI_Sleep(1000);
	              }
	            }
	
	            DESTROY_MODEL_INSTANCE(thermostat);
	          }
	          IoTHubClient_Destroy(iotHubClientHandle);
	        }
	        serializer_deinit();
	      }
	    }

    
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


5. 将 **main** 函数替换为以下代码以调用 **remote\_monitoring\_run** 函数：


	    int main()
	    {
	      remote_monitoring_run();
	      return 0;
	    }


6. 单击“生成”，然后单击“生成解决方案”以生成设备应用程序。

7. 在“解决方案资源管理器”中，右键单击“RMDevice”项目，单击“调试”，然后单击“启动新实例”以运行示例。在应用程序将示例遥测发送到 IoT 中心和接收命令时，控制台会显示消息。

[AZURE.INCLUDE [iot-suite-visualize-connecting](../../includes/iot-suite-visualize-connecting.md)]


[lnk-c-project-properties]: https://msdn.microsoft.com/zh-cn/library/669zx6zc.aspx

<!---HONumber=Mooncake_0815_2016-->