## 指定 IoT 中心设备的行为

IoT 中心序列化程序客户端库使用模型来指定设备与 IoT 中心交换的消息的格式。

1. 在 `#include` 语句之后添加以下变量声明。将占位符值 [Device Id] 和 [Device Key] 替换为在远程监视解决方案仪表板中记下的设备值。使用解决方案仪表板中的 IoT 中心主机名替换 [IoTHub Name]。例如，如果 IoT 中心主机名是 **contoso.azure-devices.cn**，则将 [IoTHub Name] 替换为 **contoso**：
   
    
	    static const char* deviceId = "[Device Id]";
	    static const char* connectionString = "HostName=[IoTHub Name].azure-devices.cn;DeviceId=[Device Id];SharedAccessKey=[Device Key]";
    

1. 添加以下代码以定义使设备可以与 IoT 中心通信的模型。此模型指定设备可执行以下操作：

   - 可将温度、外部温度、湿度和设备 ID 作为遥测数据发送。
   - 可将有关设备的元数据发送到 IoT 中心。设备在启动时发送 **DeviceInfo** 对象中的基本元数据。
   - 可以向 IoT 中心中的设备孪生发送已报告的属性。这些报告的属性划分为配置、设备和系统属性。
   - 可以接收和处理在 IoT 中心的设备孪生中设置的所需属性。
   - 可以响应通过解决方案门户调用的 **Reboot** 和 **InitiateFirmwareUpdate** 直接方法。设备使用报告的属性发送有关其支持的直接方法的信息。
   
    
    	    // Define the Model
    	    BEGIN_NAMESPACE(Contoso);
    
    	    /* Reported properties */
    	    DECLARE_STRUCT(SystemProperties,
    	      ascii_char_ptr, Manufacturer,
    	      ascii_char_ptr, FirmwareVersion,
    	      ascii_char_ptr, InstalledRAM,
    	      ascii_char_ptr, ModelNumber,
    	      ascii_char_ptr, Platform,
    	      ascii_char_ptr, Processor,
    	      ascii_char_ptr, SerialNumber
    	    );
    
    	    DECLARE_STRUCT(LocationProperties,
    	      double, Latitude,
    	      double, Longitude
    	    );
    
    	    DECLARE_STRUCT(ReportedDeviceProperties,
    	      ascii_char_ptr, DeviceState,
    	      LocationProperties, Location
    	    );
    
    	    DECLARE_MODEL(ConfigProperties,
    	      WITH_REPORTED_PROPERTY(double, TemperatureMeanValue),
    	      WITH_REPORTED_PROPERTY(uint8_t, TelemetryInterval)
    	    );
    
    	    /* Part of DeviceInfo */
    	    DECLARE_STRUCT(DeviceProperties,
    	      ascii_char_ptr, DeviceID,
    	      _Bool, HubEnabledState
    	    );
    
    	    DECLARE_DEVICETWIN_MODEL(Thermostat,
    	      /* Telemetry (temperature, external temperature and humidity) */
    	      WITH_DATA(double, Temperature),
    	      WITH_DATA(double, ExternalTemperature),
    	      WITH_DATA(double, Humidity),
    	      WITH_DATA(ascii_char_ptr, DeviceId),
    
    	      /* DeviceInfo */
    	      WITH_DATA(ascii_char_ptr, ObjectType),
    	      WITH_DATA(_Bool, IsSimulatedDevice),
    	      WITH_DATA(ascii_char_ptr, Version),
    	      WITH_DATA(DeviceProperties, DeviceProperties),
    
    	      /* Device twin properties */
    	      WITH_REPORTED_PROPERTY(ReportedDeviceProperties, Device),
    	      WITH_REPORTED_PROPERTY(ConfigProperties, Config),
    	      WITH_REPORTED_PROPERTY(SystemProperties, System),
    
    	      WITH_DESIRED_PROPERTY(double, TemperatureMeanValue, onDesiredTemperatureMeanValue),
    	      WITH_DESIRED_PROPERTY(uint8_t, TelemetryInterval, onDesiredTelemetryInterval),
    
    	      /* Direct methods implemented by the device */
    	      WITH_METHOD(Reboot),
    	      WITH_METHOD(InitiateFirmwareUpdate, ascii_char_ptr, FwPackageURI),
    
    	      /* Register direct methods with solution portal */
    	      WITH_REPORTED_PROPERTY(ascii_char_ptr_no_quotes, SupportedMethods)
    	    );
    
    	    END_NAMESPACE(Contoso);
    

## 实现设备的行为
现在添加实现模型中定义的行为的代码。

1. 添加以下函数，用于处理在解决方案仪表板中设置的所需属性。模型中定义了以下所需属性：

    
	    void onDesiredTemperatureMeanValue(void* argument)
	    {
	      /* By convention 'argument' is of the type of the MODEL */
	      Thermostat* thermostat = argument;
	      printf("Received a new desired_TemperatureMeanValue = %f\r\n", thermostat->TemperatureMeanValue);

	    }

	    void onDesiredTelemetryInterval(void* argument)
	    {
	      /* By convention 'argument' is of the type of the MODEL */
	      Thermostat* thermostat = argument;
	      printf("Received a new desired_TelemetryInterval = %d\r\n", thermostat->TelemetryInterval);
	    }
    

1. 添加以下函数，用于处理通过 IoT 中心调用的直接方法。模型中定义了以下直接方法：

    
	    /* Handlers for direct methods */
	    METHODRETURN_HANDLE Reboot(Thermostat* thermostat)
	    {
	      (void)(thermostat);

	      METHODRETURN_HANDLE result = MethodReturn_Create(201, ""Rebooting"");
	      printf("Received reboot request\r\n");
	      return result;
	    }

	    METHODRETURN_HANDLE InitiateFirmwareUpdate(Thermostat* thermostat, ascii_char_ptr FwPackageURI)
	    {
	      (void)(thermostat);

	      METHODRETURN_HANDLE result = MethodReturn_Create(201, ""Initiating Firmware Update"");
	      printf("Recieved firmware update request. Use package at: %s\r\n", FwPackageURI);
	      return result;
	    }
    

1. 添加以下函数，用于向预配置解决方案发送消息：
   
    
	    /* Send data to IoT Hub */
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
    

1. 添加以下回调处理程序，设备向预配置解决方案发送新的报告属性值后，将运行该处理程序：

    
	    /* Callback after sending reported properties */
	    void deviceTwinCallback(int status_code, void* userContextCallback)
	    {
	      (void)(userContextCallback);
	      printf("IoTHub: reported properties delivered with status_code = %u\n", status_code);
	    }
    

1. 添加以下函数，用于将设备连接到云中的预配置解决方案并交换数据。此函数执行以下步骤：

   - 初始化平台。
   - 将 Contoso 命名空间注册到序列化库。
   - 使用设备连接字符串初始化客户端。
   - 创建 **Thermostat** 模型的实例。
   - 创建并发送报告属性值。
   - 发送 **DeviceInfo** 对象。
   - 创建一个循环，以便每秒发送遥测数据。
   - 取消初始化所有资源。

      
              void remote_monitoring_run(void)
              {
                if (platform_init() != 0)
                {
                  printf("Failed to initialize the platform.\n");
                }
                else
                {
                  if (SERIALIZER_REGISTER_NAMESPACE(Contoso) == NULL)
                  {
                    printf("Unable to SERIALIZER_REGISTER_NAMESPACE\n");
                  }
                  else
                  {
                    IOTHUB_CLIENT_HANDLE iotHubClientHandle = IoTHubClient_CreateFromConnectionString(connectionString, MQTT_Protocol);
                    if (iotHubClientHandle == NULL)
                    {
                      printf("Failure in IoTHubClient_CreateFromConnectionString\n");
                    }
                    else
                    {
              #ifdef MBED_BUILD_TIMESTAMP
                      // For mbed add the certificate information
                      if (IoTHubClient_SetOption(iotHubClientHandle, "TrustedCerts", certificates) != IOTHUB_CLIENT_OK)
                      {
                          printf("Failed to set option \"TrustedCerts\"\n");
                      }
              #endif // MBED_BUILD_TIMESTAMP
                      Thermostat* thermostat = IoTHubDeviceTwin_CreateThermostat(iotHubClientHandle);
                      if (thermostat == NULL)
                      {
                        printf("Failure in IoTHubDeviceTwin_CreateThermostat\n");
                      }
                      else
                      {
                        /* Set values for reported properties */
                        thermostat->Config.TemperatureMeanValue = 55.5;
                        thermostat->Config.TelemetryInterval = 3;
                        thermostat->Device.DeviceState = "normal";
                        thermostat->Device.Location.Latitude = 47.642877;
                        thermostat->Device.Location.Longitude = -122.125497;
                        thermostat->System.Manufacturer = "Contoso Inc.";
                        thermostat->System.FirmwareVersion = "2.22";
                        thermostat->System.InstalledRAM = "8 MB";
                        thermostat->System.ModelNumber = "DB-14";
                        thermostat->System.Platform = "Plat 9.75";
                        thermostat->System.Processor = "i3-7";
                        thermostat->System.SerialNumber = "SER21";
                        /* Specify the signatures of the supported direct methods */
                        thermostat->SupportedMethods = "{\"Reboot\": \"Reboot the device\", \"InitiateFirmwareUpdate--FwPackageURI-string\": \"Updates device Firmware. Use parameter FwPackageURI to specifiy the URI of the firmware file\"}";
        
                        /* Send reported properties to IoT Hub */
                        if (IoTHubDeviceTwin_SendReportedStateThermostat(thermostat, deviceTwinCallback, NULL) != IOTHUB_CLIENT_OK)
                        {
                          printf("Failed sending serialized reported state\n");
                        }
                        else
                        {
                          printf("Send DeviceInfo object to IoT Hub at startup\n");
              
                          thermostat->ObjectType = "DeviceInfo";
                          thermostat->IsSimulatedDevice = 0;
                          thermostat->Version = "1.0";
                          thermostat->DeviceProperties.HubEnabledState = 1;
                          thermostat->DeviceProperties.DeviceID = (char*)deviceId;
        
                          unsigned char* buffer;
                          size_t bufferSize;
        
                          if (SERIALIZE(&buffer, &bufferSize, thermostat->ObjectType, thermostat->Version, thermostat->IsSimulatedDevice, thermostat->DeviceProperties) != CODEFIRST_OK)
                          {
                            (void)printf("Failed serializing DeviceInfo\n");
                          }
                          else
                          {
                            sendMessage(iotHubClientHandle, buffer, bufferSize);
                          }
        
                          /* Send telemetry */
                          thermostat->Temperature = 50;
                          thermostat->ExternalTemperature = 55;
                          thermostat->Humidity = 50;
                          thermostat->DeviceId = (char*)deviceId;
        
                          while (1)
                          {
                            unsigned char*buffer;
                            size_t bufferSize;
        
                            (void)printf("Sending sensor value Temperature = %f, Humidity = %f\n", thermostat->Temperature, thermostat->Humidity);
        
                            if (SERIALIZE(&buffer, &bufferSize, thermostat->DeviceId, thermostat->Temperature, thermostat->Humidity, thermostat->ExternalTemperature) != CODEFIRST_OK)
                            {
                              (void)printf("Failed sending sensor value\r\n");
                            }
                            else
                            {
                              sendMessage(iotHubClientHandle, buffer, bufferSize);
                            }
        
                            ThreadAPI_Sleep(1000);
                          }
        
                          IoTHubDeviceTwin_DestroyThermostat(thermostat);
                        }
                      }
                      IoTHubClient_Destroy(iotHubClientHandle);
                    }
                    serializer_deinit();
                  }
                }
                platform_deinit();
              }
    
   
    下面提供了发送到预配置解决方案的示例**遥测**消息供参考：
   
    
	    {"DeviceId":"mydevice01", "Temperature":50, "Humidity":50, "ExternalTemperature":55}
    

<!---HONumber=Mooncake_0327_2017-->