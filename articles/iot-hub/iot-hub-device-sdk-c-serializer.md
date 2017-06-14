<properties
    pageTitle="适用于 C 语言的 Azure IoT 设备 SDK - 序列化程序 | Azure"
    description="如何使用 Azure IoT 设备 SDK 中面向 C 语言的序列化程序库创建与 IoT 中心通信的设备应用。"
    services="iot-hub"
    documentationcenter=""
    author="olivierbloch"
    manager="timlt"
    editor="" />
<tags
    ms.assetid="defbed34-de73-429c-8592-cd863a38e4dd"
    ms.service="iot-hub"
    ms.devlang="cpp"
    ms.topic="article"
    ms.tgt_pltfrm="na"
    ms.workload="na"
    ms.date="09/06/2016"
    wacn.date="06/05/2017"
    ms.author="v-yiso" />

# <a name="azure-iot-device-sdk-for-c--more-about-serializer"></a>适用于 C 语言的 Azure IoT 设备 SDK - 有关序列化程序的详细信息

此系列中的[第一篇文章](/documentation/articles/iot-hub-device-sdk-c-intro/)介绍了**适用于 C 语言的 Azure IoT 设备 SDK**。下一篇文章中提供了 [**IoTHubClient**](/documentation/articles/iot-hub-device-sdk-c-iothubclient/) 的更详细说明。 本文最后的部分将提供该 SDK 的剩余组件 **序列化程序** 库的更详细说明。

本简介文章介绍如何使用**序列化程序**库将事件发送到 IoT 中心，以及接收来自 IoT 中心的消息。本文中将延伸该讨论，更完整地阐释如何使用**序列化程序**宏语言来创建数据模型。本文还包含更多有关该库如何序列化消息（以及在某些情况下，如何控制序列化行为）的详细信息。此外，将介绍可以修改以判断所要创建的模型大小的某些参数。

最后，本文将回顾前面文章中讲到的一些主题，例如消息和属性处理。 正如我们了解的那样，这些功能使用**序列化程序**库的方式与使用 **IoTHubClient** 库一样。

本文中所述的所有内容都基于 **序列化程序** SDK 示例。 如果想要继续，请参阅适用于 C 语言的 Azure IoT 设备 SDK 中包含的 **simplesample\_amqp** 和 **simplesample\_http** 应用程序。

可在 GitHub 存储库中找到[**适用于 C 语言的 Azure IoT 设备 SDK**](https://github.com/Azure/azure-iot-sdk-c)，还可在 [C API 参考](https://azure.github.io/azure-iot-sdk-c/index.html)中查看 API 的详细信息。

## <a name="the-modeling-language"></a>建模语言

此系列中的[介绍性文章](/documentation/articles/iot-hub-device-sdk-c-intro/)通过 **simplesample\_amqp** 应用程序中提供的示例，介绍了**适用于 C 语言的 Azure IoT 设备 SDK** 建模语言：


        BEGIN_NAMESPACE(WeatherStation);
        
        DECLARE_MODEL(ContosoAnemometer,
        WITH_DATA(ascii_char_ptr, DeviceId),
        WITH_DATA(double, WindSpeed),
        WITH_ACTION(TurnFanOn),
        WITH_ACTION(TurnFanOff),
        WITH_ACTION(SetAirResistance, int, Position)
        );
        
        END_NAMESPACE(WeatherStation);

如你所见，建模语言基于 C 宏。 定义请始终以 **BEGIN\_NAMESPACE** 开头，始终以 **END\_NAMESPACE** 结尾。 我们通常要为公司的命名空间命名，或者如同本示例一样，为正在处理的项目命名。

在命名空间内部运行的是模型定义。在本例中，有一个风速计模型。同样，你可以任意命名此模型，但通常会针对想要与 IoT 中心交换的设备或数据类型来命名。

模型包含可发送到 IoT 中心的事件（*数据*）以及可从 IoT 中心接收的消息（*操作*）的定义。 如同在示例中所见，事件具有类型和名称；操作具有一个名称和可选参数（各有一个类型）。

本示例并未演示 SDK 支持的其他数据类型。我们将在稍后讨论。

> [AZURE.NOTE]
> IoT 中心将设备发送到它的数据作为*事件*，而建模语言将其作为*数据*（使用 **WITH_DATA** 进行定义）。 同样，IoT 中心将你发送到设备的数据作为*消息*，而建模语言将其作为*操作*（使用 **WITH_ACTION** 进行定义）。 请注意，本文中可能会换用这些术语。
> 
> 

### <a name="supported-data-types"></a>支持的数据类型

利用**序列化程序**库创建的模型支持以下数据类型：

| 类型 | 说明 |
| --- | --- |
| double |双精度浮点数 |
| int |32 位整数 |
| float |单精度浮点数 |
| long |长整数 |
| int8\_t |8 位整数 |
| int16\_t |16 位整数 |
| int32\_t |32 位整数 |
| int64\_t |64 位整数 |
| bool |布尔值 |
| ascii\_char\_ptr |ASCII 字符串 |
| EDM\_DATE\_TIME\_OFFSET |日期时间偏移 |
| EDM\_GUID |GUID |
| EDM\_BINARY |binary |
| DECLARE\_STRUCT |复杂数据类型 |

我们从最后一种数据类型开始。 **DECLARE\_STRUCT** 可用来定义作为其他基元类型的分组的复杂数据类型。 这些分组可让我们定义如下所示的模型：

        DECLARE_STRUCT(TestType,
        double, aDouble,
        int, aInt,
        float, aFloat,
        long, aLong,
        int8_t, aInt8,
        uint8_t, auInt8,
        int16_t, aInt16,
        int32_t, aInt32,
        int64_t, aInt64,
        bool, aBool,
        ascii_char_ptr, aAsciiCharPtr,
        EDM_DATE_TIME_OFFSET, aDateTimeOffset,
        EDM_GUID, aGuid,
        EDM_BINARY, aBinary
        );
        
        DECLARE_MODEL(TestModel,
        WITH_DATA(TestType, Test)
        );

我们的模型包含 **TestType**类型的单个数据事件。 **TestType** 是包含多个成员的复杂类型，它们共同演示了**序列化程序**建模语言支持的基元类型。

使用类似于这样的模型，我们可以编写代码，以将数据发送到 IoT 中心，如下所示：


        TestModel* testModel = CREATE_MODEL_INSTANCE(MyThermostat, TestModel);
        
        testModel->Test.aDouble = 1.1;
        testModel->Test.aInt = 2;
        testModel->Test.aFloat = 3.0f;
        testModel->Test.aLong = 4;
        testModel->Test.aInt8 = 5;
        testModel->Test.auInt8 = 6;
        testModel->Test.aInt16 = 7;
        testModel->Test.aInt32 = 8;
        testModel->Test.aInt64 = 9;
        testModel->Test.aBool = true;
        testModel->Test.aAsciiCharPtr = "ascii string 1";
        
        time_t now;
        time(&now);
        testModel->Test.aDateTimeOffset = GetDateTimeOffset(now);
        
        EDM_GUID guid = { { 0x00, 0x01, 0x02, 0x03, 0x04, 0x05, 0x06, 0x07, 0x08, 0x09, 0x0A, 0x0B, 0x0C, 0x0D, 0x0E, 0x0F } };
        testModel->Test.aGuid = guid;
        
        unsigned char binaryArray[3] = { 0x01, 0x02, 0x03 };
        EDM_BINARY binaryData = { sizeof(binaryArray), &binaryArray };
        testModel->Test.aBinary = binaryData;
        
        SendAsync(iotHubClientHandle, (const void*)&(testModel->Test));

基本而言，我们要将值赋给 **Test** 结构的每个成员，然后调用 **SendAsync** 以将 **Test** 数据事件发送到云。 **SendAsync** 是将单个数据事件发送到 IoT 中心的帮助器函数：

            void SendAsync(IOTHUB_CLIENT_LL_HANDLE iotHubClientHandle, const void *dataEvent)
            {
            	unsigned char* destination;
            	size_t destinationSize;
            	if (SERIALIZE(&destination, &destinationSize, *(const unsigned char*)dataEvent) ==
            	{
            		// null terminate the string
            		char* destinationAsString = (char*)malloc(destinationSize + 1);
            		if (destinationAsString != NULL)
            		{
            			memcpy(destinationAsString, destination, destinationSize);
            			destinationAsString[destinationSize] = '\0';
            			IOTHUB_MESSAGE_HANDLE messageHandle = IoTHubMessage_CreateFromString(destinationAsString);
            			if (messageHandle != NULL)
            			{
            				IoTHubClient_SendEventAsync(iotHubClientHandle, messageHandle, sendCallback, (void*)0);
            				IoTHubMessage_Destroy(messageHandle);
            			}
            			free(destinationAsString);
            		}
            		free(destination);
            	}
            }

此函数序列化给定的数据事件，并使用 **IoTHubClient\_SendEventAsync** 将其发送到 IoT 中心。 这是在先前的文章中讨论的相同代码（**SendAsync** 将逻辑封装到一个方便访问的函数）。

在以前的代码中使用的另一个帮助器函数是 **GetDateTimeOffset**。 此函数将给定的时间转换为 **EDM\_DATE\_TIME\_OFFSET** 类型的值：

        EDM_DATE_TIME_OFFSET GetDateTimeOffset(time_t time)
        {
        	struct tm newTime;
        	gmtime_s(&newTime, &time);
        	EDM_DATE_TIME_OFFSET dateTimeOffset;
        	dateTimeOffset.dateTime = newTime;
        	dateTimeOffset.fractionalSecond = 0;
        	dateTimeOffset.hasFractionalSecond = 0;
        	dateTimeOffset.hasTimeZone = 0;
        	dateTimeOffset.timeZoneHour = 0;
        	dateTimeOffset.timeZoneMinute = 0;
        	return dateTimeOffset;
        }


如果你运行此代码，就会将以下消息发送到 IoT 中心：


        {"aDouble":1.100000000000000, "aInt":2, "aFloat":3.000000, "aLong":4, "aInt8":5, "auInt8":6, "aInt16":7, "aInt32":8, "aInt64":9, "aBool":true, "aAsciiCharPtr":"ascii string 1", "aDateTimeOffset":"2015-09-14T21:18:21Z", "aGuid":"00010203-0405-0607-0809-0A0B0C0D0E0F", "aBinary":"AQID"}


请注意，序列化为 JSON，这是**序列化程序**库生成的格式。另请注意，序列化 JSON 对象的每个成员都与模型中定义的 **TestType** 成员匹配。值也与代码中使用的值完全匹配。不过请注意，二进制数据采用 base64 编码：“AQID”是 {0x01, 0x02, 0x03} 的 base64 编码。

此示例演示使用**序列化程序**库的优点 -- 它可让我们将 JSON 发送到云，而不需要在应用程序中显式处理序列化。我们只需考虑如何在模型中设置数据事件的值，然后调用简单的 API 将这些事件发送到云。

有了此信息，我们便可以定义包含受支持数据类型范围的模型，这些数据类型包括复杂类型（甚至可以包含其他复杂类型内的复杂类型）。不过，上述示例生成的序列化 JSON 突显了一个重点。*如何*利用**序列化程序**库发送数据完全决定了 JSON 的构成形式。此特定要点就是接下来要讨论的内容。

## <a name="more-about-serialization"></a>有关序列化的详细信息

上一部分重点讲述了**序列化程序**库生成的输出示例。在本部分，我们将说明该库如何将数据序列化，以及如何使用序列化 API 来控制该行为。

为了进一步讨论序列化，我们将使用一个基于恒温器的新模型。首先，让我们针对所要尝试处理的方案提供一些背景信息。

我们想要为一个可测量温度和湿度的恒温器建模。每个数据片段将以不同的方式发送到 IoT 中心。默认情况下，该恒温器每隔 2 分钟引入温度事件一次，每隔 15 分钟引入湿度事件一次。引入任一事件时，必须包含显示相应温度或湿度测量时间的时间戳。

在此方案中，我们将演示两种不同的数据建模方式，并将说明该建模对序列化输出的影响。

### <a name="model-1"></a>模型 1

以下是支持前述方案的第一个模型版本：


        BEGIN_NAMESPACE(Contoso);
        
        DECLARE_STRUCT(TemperatureEvent,
        int, Temperature,
        EDM_DATE_TIME_OFFSET, Time);
        
        DECLARE_STRUCT(HumidityEvent,
        int, Humidity,
        EDM_DATE_TIME_OFFSET, Time);
        
        DECLARE_MODEL(Thermostat,
        WITH_DATA(TemperatureEvent, Temperature),
        WITH_DATA(HumidityEvent, Humidity)
        );
        
        END_NAMESPACE(Contoso);

请注意，该模型包含两个数据事件：**Temperature** 和 **Humidity**。 与上述示例不同，每个事件的类型是使用 **DECLARE\_STRUCT** 定义的结构。 **TemperatureEvent** 包括温度测量和时间戳；**HumidityEvent** 包含湿度度量和时间戳。 此模型可让我们以自然的方式为上述方案的数据建模。 将事件发送到云时，将发送温度/时间戳对或湿度/时间戳对。

可以使用类似于下面的代码将温度事件发送到云：


        time_t now;
        time(&now);
        thermostat->Temperature.Temperature = 75;
        thermostat->Temperature.Time = GetDateTimeOffset(now);
        
        unsigned char* destination;
        size_t destinationSize;
        if (SERIALIZE(&destination, &destinationSize, thermostat->Temperature) == IOT_AGENT_OK)
        {
            sendMessage(iotHubClientHandle, destination, destinationSize);
        }


在示例代码中，我们将针对温度和湿度使用硬编码值，但请想象成实际上是从恒温器上相应的传感器采样来检索这些值。

上述代码使用前面介绍的帮助器 **GetDateTimeOffset** 。 此代码将序列化与发送事件的任务明确区分，原因在稍后将渐趋明朗。 前面的代码将温度事件以序列化方式发送到缓冲区。 **sendMessage** 是将事件发送到 IoT 中心的帮助器函数（包括在 **simplesample\_amqp** 中）：


        static void sendMessage(IOTHUB_CLIENT_HANDLE iotHubClientHandle, const unsigned char* buffer, size_t size)
        {
            static unsigned int messageTrackingId;
            IOTHUB_MESSAGE_HANDLE messageHandle = IoTHubMessage_CreateFromByteArray(buffer, size);
            if (messageHandle != NULL)
            {
                IoTHubClient_SendEventAsync(iotHubClientHandle, messageHandle, sendCallback, (void*)(uintptr_t)messageTrackingId);
        
                IoTHubMessage_Destroy(messageHandle);
            }
            free((void*)buffer);
        }


此代码是前一部分所述的 **SendAsync** 帮助器的子集，因此不在此赘述。

运行前面的代码发送温度事件时，事件的序列化形式将发送到 IoT 中心：


        {"Temperature":75, "Time":"2015-09-17T18:45:56Z"}

我们发送类型为 **TemperatureEvent** 的温度，该构造包含 **Temperature** 和 **Time** 成员。 这直接反映在序列化数据中。

同样，我们可以使用以下代码发送湿度事件：


        thermostat->Humidity.Humidity = 45;
        thermostat->Humidity.Time = GetDateTimeOffset(now);
        if (SERIALIZE(&destination, &destinationSize, thermostat->Humidity) == IOT_AGENT_OK)
        {
            sendMessage(iotHubClientHandle, destination, destinationSize);
        }


发送到 IoT 中心的序列化形式如下所示：


        {"Humidity":45, "Time":"2015-09-17T18:45:56Z"}


同样，一切都按预期进行。

可以使用此模型来想象如何轻松地添加其他事件。 可以使用 **DECLARE\_STRUCT** 定义更多结构，使用 **WITH\_DATA** 在模型中包含相应的事件。

现在，让我们修改模型，使它包含相同的数据，但具有不同的结构。

### <a name="model-2"></a>模型 2

请考虑上述模型的替代模型：


        DECLARE_MODEL(Thermostat,
        WITH_DATA(int, Temperature),
        WITH_DATA(int, Humidity),
        WITH_DATA(EDM_DATE_TIME_OFFSET, Time)
        );

在此例中，我们已取消 **DECLARE\_STRUCT** 宏，只需使用建模语言中的简单类型来定义我们的方案中的数据项。

现在让我们忽略**时间**事件。忽略该事件后，以下是要引入**温度**的代码：


        time_t now;
        time(&now);
        thermostat->Temperature = 75;
        
        unsigned char* destination;
        size_t destinationSize;
        if (SERIALIZE(&destination, &destinationSize, thermostat->Temperature) == IOT_AGENT_OK)
        {
            sendMessage(iotHubClientHandle, destination, destinationSize);
        }


此代码将以下序列化事件发送到 IoT 中心：


        {"Temperature":75}


发送 Humidity 事件的代码如下所示：


        thermostat->Humidity = 45;
        if (SERIALIZE(&destination, &destinationSize, thermostat->Humidity) == IOT_AGENT_OK)
        {
            sendMessage(iotHubClientHandle, destination, destinationSize);
        }


此代码将该事件发送到 IoT 中心：


        {"Humidity":45}


到目前为止，一切都很正常。现在，让我们更改 SERIALIZE 宏的用法。

**SERIALIZE** 宏可将多个数据事件视为参数。 这使我们能够将 **Temperature** 和 **Humidity** 事件一起序列化，并在一次调用中将它们发送到 IoT 中心：


        if (SERIALIZE(&destination, &destinationSize, thermostat->Temperature, thermostat->Humidity) == IOT_AGENT_OK)
        {
            sendMessage(iotHubClientHandle, destination, destinationSize);
        }


或许你已猜到，此代码的结果是两个数据事件都发送到 IoT 中心：

        [
        
        {"Temperature":75},
        
        {"Humidity":45}
        
        ]

换而言之，你可能预料到此代码与分别发送 **Temperature** 和 **Humidity** 相同， 这样就可以方便地将两个事件在同一个调用中传递到 **SERIALIZE** 。 不过，事实并非如此。 上述代码会将此单个数据事件发送到 IoT 中心：

        {"Temperature":75, "Humidity":45}

这看起来很奇怪，因为我们的模型将 **Temperature** 和 **Humidity** 定义为两个*单独*事件：


        DECLARE_MODEL(Thermostat,
        WITH_DATA(int, Temperature),
        WITH_DATA(int, Humidity),
        WITH_DATA(EDM_DATE_TIME_OFFSET, Time)
        );

具体而言，我们没有对这些 **Temperature** 和 **Humidity** 处于相同结构的事件进行建模：


        DECLARE_STRUCT(TemperatureAndHumidityEvent,
        int, Temperature,
        int, Humidity,
        );
        
        DECLARE_MODEL(Thermostat,
        WITH_DATA(TemperatureAndHumidityEvent, TemperatureAndHumidity),
        );

如果我们使用此模型，则可以轻松地了解如何在同一序列化消息中发送 **Temperature** 和 **Humidity**。 不过，如果使用模型 2 将两个数据事件都传递到 **SERIALIZE** ，可能就无法突显这种工作方式的原因。

如果你知道**序列化程序**库所做的假设，就更容易了解这种行为。若要了解这一点，让我们返回到模型：


        DECLARE_MODEL(Thermostat,
        WITH_DATA(int, Temperature),
        WITH_DATA(int, Humidity),
        WITH_DATA(EDM_DATE_TIME_OFFSET, Time)
        );

请以对象定向的观点思考此模型。 在此例中，我们要对物理设备（调温器）进行建模，该设备包括 **Temperature** 和 **Humidity** 之类的属性。

可以利用类似于下面的代码来发送模型的整个状态：


        if (SERIALIZE(&destination, &destinationSize, thermostat->Temperature, thermostat->Humidity, thermostat->Time) == IOT_AGENT_OK)
        {
            sendMessage(iotHubClientHandle, destination, destinationSize);
        }


假设已设置温度、湿度和时间值，我们会发现有这样的事件发送到 IoT 中心：


        {"Temperature":75, "Humidity":45, "Time":"2015-09-17T18:45:56Z"}

有时，你可能只想将模型的*某些*属性发送到云（特别是当模型包含大量数据事件时）。 这时，只发送数据事件的子集相当有用，就像前面的示例一样：


        {"Temperature":75, "Time":"2015-09-17T18:45:56Z"}

这将生成完全相同的序列化的事件，就像我们在模型 1 中定义带有 **Temperature** 和 **Time** 成员的 **TemperatureEvent** 一样。 在本例中，我们可以使用不同的模型（模型 2）来生成完全相同的序列化事件，因为我们以不同的方式调用了 **SERIALIZE**。

重点是，如果将多个数据事件传递给 **SERIALIZE**，则它会假设每个事件都是单个 JSON 对象中的一个属性。

最佳方法取决于你自己以及对模型的思考方式。如果要将“事件”发送到云，且每个事件都包含一组已定义的属性，则第一种方法较为适合。在此情况下，使用 **DECLARE\_STRUCT** 定义每个事件的结构，并使用 **WITH\_DATA** 宏将它们包含在模型中。然后根据上述第一个示例中使用的方法来发送每个事件。采用此方法时，只会将单个数据事件传递给 **SERIALIZER**。

如果你以对象定向的方式思考模型，则第二种方法可能比较适合。在本例中，使用 **WITH\_DATA** 定义的元素是对象的“属性”。你可以根据想要发送到云的“对象”状态详细程度，将事件的任何子集传递给 **SERIALIZE**。

没有绝对正确或错误的方法。你只需了解**序列化程序**库的工作原理，并挑选最符合需求的建模方法。

## <a name="message-handling"></a>消息处理

到目前为止，本文只讨论了如何将事件发送到 IoT 中心，而尚未涉及到消息接收。 这是因为有关接收消息的知识大多在[以前的文章](/documentation/articles/iot-hub-device-sdk-c-intro/)中都已涵盖。 回顾那篇文章，我们知道是通过注册消息回调函数来处理消息的：


        IoTHubClient_SetMessageCallback(iotHubClientHandle, IoTHubMessage, myWeather)


然后编写在接收消息时要调用的回调函数：


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


**IoTHubMessage** 的这种实现将针对模型中的每个操作调用特定的函数。例如，如果模型定义了此操作：


        WITH_ACTION(SetAirResistance, int, Position)


你必须使用此签名来定义函数：


        EXECUTE_COMMAND_RESULT SetAirResistance(ContosoAnemometer* device, int Position)
        {
            (void)device;
            (void)printf("Setting Air Resistance Position to %d.\r\n", Position);
            return EXECUTE_COMMAND_SUCCESS;
        }

**SetAirResistance** 。

我们还没有说明消息序列化版本的外观。换而言之，如果要将 **SetAirResistance** 消息发送到设备，该消息的外观是怎样的？

如果要将消息发送给设备，可以通过 Azure IoT 服务 SDK 来完成。你仍需要知道要发送哪个字符串才能调用特定操作。用于发送消息的常规格式如下所示：


        {"Name" : "", "Parameters" : "" }


将使用两个属性来发送序列化的 JSON 对象：**Name** 是操作（消息）的名称，**Parameters** 包含该操作的参数。

例如，若要调用 **SetAirResistance**，可以将以下消息发送给设备：


        {"Name" : "SetAirResistance", "Parameters" : { "Position" : 5 }}


操作名称必须完全与模型中定义的操作匹配。参数名称也必须匹配。另请注意大小写。**Name** 和 **Parameters** 始终大写。请务必与模型中操作名称和参数的大小写匹配。在本示例中，操作名称是“SetAirResistance”，而不是“setairresistance”。

将这些消息发送到设备可以调用其他两个操作 **TurnFanOn** 和 **TurnFanOff**：


	{"Name" : "TurnFanOn", "Parameters" : {}}
	{"Name" : "TurnFanOff", "Parameters" : {}}


本部分说明了使用**序列化程序**库发送事件和接收消息时的所有要点。在继续讨论之前，让我们先介绍一些可以配置以控制模型大小的参数。

## <a name="macro-configuration"></a>宏配置
如果使用的是 **序列化程序** 库，那么可在 azure-c-shared-utility 库中找到要注意的 SDK 的重要组成部分。
如果使用 --recursive 选项从 GitHub 中克隆了 Azure-iot-sdk-c 存储库，那么可在此处找到此共享的实用程序库：


	.\\c-utility


如果没有克隆此库，则可以在[此处](https://github.com/Azure/azure-c-shared-utility)找到它。

在此共享的实用程序库中，可找到以下文件夹：


    azure-c-shared-utility\\macro\_utils\_h\_generator.


此文件夹包含名为 **macro\_utils\_h\_generator.sln** 的 Visual Studio 解决方案：

  ![](./media/iot-hub-device-sdk-c-serializer/01-macro_utils_h_generator.PNG)

此解决方案中的程序将生成 **macro\_utils.h** 文件。 SDK 附带了一个默认的 macro\_utils.h 文件。 此解决方案可让你修改某些参数，然后根据这些参数重新创建标头文件。

要注意两个重要参数：**nArithmetic** 和 **nMacroParameters**，这些参数在 macro\_utils.tt 的以下两行中定义：


        <#int nArithmetic=1024;#>
        <#int nMacroParameters=124;/*127 parameters in one macro deﬁnition in C99 in chapter 5.2.4.1 Translation limits*/#>



这些值是 SDK 随附的默认参数。每个参数的含义如下：

-   nMacroParameters – 控制可以在一个 DECLARE\_MODEL 宏定义中指定的参数数目。

-   nArithmetic – 控制模型中允许的成员总数。

这些参数之所以重要，是因为它们控制模型的大小。例如，假设有以下模型定义：


        DECLARE_MODEL(MyModel,
        WITH_DATA(int, MyData)
        );

如前所述，**DECLARE\_MODEL** 只是一个 C 宏。 模型的名称和 **WITH\_DATA** 语句（也即另一个宏）是 **DECLARE\_MODEL** 的参数。 **nMacroParameters** 定义了 **DECLARE\_MODEL** 中可以包含的参数数目。 实际上，这定义了你可以指定的数据事件和操作声明数目。 因此，使用默认限制 124 时，你可以定义由大约 60 个操作和事件数据组成的模型。 如果你试图超过此限制，将收到如下所示的编译器错误：

  ![](./media/iot-hub-device-sdk-c-serializer/02-nMacroParametersCompilerErrors.PNG)

**nArithmetic** 参数主要与宏语言的内部工作有关，而与应用程序没有太大的关系。  该参数控制可以在模型中（包括 **DECLARE_STRUCT** 宏）指定的成员总数。 如果你开始看到这样的编译器错误，应该尝试增大 **nArithmetic**的值：

   ![](./media/iot-hub-device-sdk-c-serializer/03-nArithmeticCompilerErrors.PNG)

如果想要更改这些参数，请修改 macro\_utils.tt 文件中的值，重新编译 macro\_utils\_h\_generator.sln 解决方案并运行已编译的程序。 当你这么做时，将生成一个新的 macro\_utils.h 文件，该文件置于 .\\common\\inc 目录中。

为了使用新版本的 macro\_utils.h，请从你的解决方案中删除**序列化程序** NuGet 包，并在其位置放置**序列化程序** Visual Studio 项目。 这样，便可以让代码针对序列化程序库的源代码进行编译。 这包括更新的 macro\_utils.h。 如果想要针对 **simplesample\_amqp** 执行此操作，首先从解决方案中删除序列化程序库的 NuGet 包：

   ![](./media/iot-hub-device-sdk-c-serializer/04-serializer-github-package.PNG)

然后将此项目添加到 Visual Studio 解决方案：

> .\\c\\serializer\\build\\windows\\serializer.vcxproj
> 
> 

完成后，解决方案应该如下所示：

   ![](./media/iot-hub-device-sdk-c-serializer/05-serializer-project.PNG)

现在当你编译解决方案时，二进制文件中包含已更新的 macro\_utils.h。

请注意，将这些值增大到足够高的数目可能会超出编译器限制。 对于这一点， **nMacroParameters** 是要考虑的主要参数。 C99 规范规定，宏定义中至少允许 127 个参数。 Microsoft 编译器完全遵循规范（个数限制为 127），因此不能将 **nMacroParameters** 提高到超出默认值。 其他编译器可能允许这么做（例如 GNU 编译器支持更高的限制）。

到目前为止，我们介绍了你在使用**序列化程序**库编写代码时需要知道的几乎所有内容。结束前，来回顾下前面文章中可能感兴趣的一些主题。

## <a name="the-lower-level-apis"></a>较低级别 API

这篇文章重点介绍的示例应用程序是 **simplesample\_amqp**。 此示例使用较高级别的（非“LL”）API 来发送事件和接收消息。 如果你使用这些 API，将运行后台线程来处理事件发送和消息接收。 不过，可以使用较低级别 (LL) API 以取消此后台线程，并在发送事件或接收来自云的消息时接管显式控制。

如[前一篇文章](/documentation/articles/iot-hub-device-sdk-c-iothubclient/)中所述，有一组由较高级别 API 构成的函数：

-   IoTHubClient\_CreateFromConnectionString

-   IoTHubClient\_SendEventAsync

-   IoTHubClient\_SetMessageCallback

-   IoTHubClient\_Destroy

**simplesample\_amqp** 中演示了这些 API。

还有一组类似但级别更低的 API。

-   IoTHubClient\_LL\_CreateFromConnectionString

-   IoTHubClient\_LL\_SendEventAsync

-   IoTHubClient\_LL\_SetMessageCallback

-   IoTHubClient\_LL\_Destroy

请注意，较低级别的 API 的工作原理与前面文章中所述的完全相同。如果你想要使用后台线程来处理事件发送和消息接收，可以使用第一组 API。如果你想要掌握与 IoT 中心之间发送和接收数据时的明确控制权，可以使用第二组 API。上述任何一组 API 都可以很好地配合使用**序列化程序**库。

若要通过示例了解较低级别的 API 如何与**序列化程序**库配合使用，请参阅 **simplesample\_http** 应用程序。

## <a name="additional-topics"></a>其他主题

值得一提的其他几个主题包括属性处理、使用替代设备凭据和配置选项。 这些主题均涵盖在 [前一篇文章](/documentation/articles/iot-hub-device-sdk-c-iothubclient/)中。 重点在于，所有这些功能与**序列化程序**库配合使用的方式与和 **IoTHubClient** 库配合使用的方式相同。 例如，如果想要从模型将属性附加到事件，需要以前面所述的相同方式，使用 **IoTHubMessage\_Properties** 和 **Map**\_**AddorUpdate**：


    MAP_HANDLE propMap = IoTHubMessage_Properties(message.messageHandle);
    sprintf_s(propText, sizeof(propText), "%d", i);
    Map_AddOrUpdate(propMap, "SequenceNumber", propText);

至于事件是从**序列化程序**库生成，还是使用 **IoTHubClient** 库手动创建，并不重要。

就替代设备凭据而言，使用 **IoTHubClient\_LL\_Create** 分配 **IOTHUB\_CLIENT\_HANDLE** 的效果和使用 **IoTHubClient\_CreateFromConnectionString** 一样好。

最后，如果使用**序列化程序**库，则可以使用 **IoTHubClient\_LL\_SetOption** 来设置配置选项，就像使用 **IoTHubClient** 库时一样。

**序列化程序** 库所具有的一个独特功能为初始化 API。 在开始使用库之前，必须调用 **serializer\_init**：


    serializer_init(NULL);


此操作必须在调用 **IoTHubClient\_CreateFromConnectionString** 之前完成。

同样，当使用完该库时，最后调用的对象是 **serializer\_deinit**：


    serializer_deinit();


除此之外，上面列出的所有其他功能在**序列化程序**库中的运行方式均与在 **IoTHubClient** 库中的运行方式相同。有关这些主题中任何一个主题的详细信息，请参阅本系列教程中的[前一篇文章](/documentation/articles/iot-hub-device-sdk-c-iothubclient/)。

## <a name="next-steps"></a>后续步骤

本文详细介绍了**适用于 C 语言的 Azure IoT 设备 SDK** 中包含的**序列化程序**库的独特方面。通过文中提供的信息，你应该能充分了解如何使用模型来发送事件和接收来自 IoT 中心的消息。

本文也是通过**适用于 C 语言的 Azure IoT 设备 SDK** 开发应用程序这一系列教程（由三部分组成）的最后一部分。这些信息应该不仅足以让你入门，还能让你彻底了解 API 的工作原理。请了解其他信息，因为还有一些 SDK 中的示例未涵盖在本文中。除此之外，[SDK 文档](https://github.com/Azure/azure-iot-sdk-c)是获取其他信息的绝佳资源。

若要详细了解如何针对 IoT 中心进行开发，请参阅 [Azure IoT SDK][lnk-sdks]。

若要进一步探索 IoT 中心的功能，请参阅：

* [使用 Azure IoT Edge 模拟设备][lnk-gateway]

[lnk-sdks]: /documentation/articles/iot-hub-devguide-sdks/

[lnk-gateway]: /documentation/articles/iot-hub-linux-gateway-sdk-simulated-device/
