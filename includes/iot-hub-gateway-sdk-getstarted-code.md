## 典型输出

下面通过 Hello World 示例来说明写入到日志文件的输出。添加了换行符和制表符来增强可读性：

```
[{
	"time": "Mon Apr 11 13:48:07 2016",
	"content": "Log started"
}, {
	"time": "Mon Apr 11 13:48:48 2016",
	"properties": {
		"helloWorld": "from Azure IoT Gateway SDK simple sample!"
	},
	"content": "aGVsbG8gd29ybGQ="
}, {
	"time": "Mon Apr 11 13:48:55 2016",
	"properties": {
		"helloWorld": "from Azure IoT Gateway SDK simple sample!"
	},
	"content": "aGVsbG8gd29ybGQ="
}, {
	"time": "Mon Apr 11 13:49:01 2016",
	"properties": {
		"helloWorld": "from Azure IoT Gateway SDK simple sample!"
	},
	"content": "aGVsbG8gd29ybGQ="
}, {
	"time": "Mon Apr 11 13:49:04 2016",
	"content": "Log stopped"
}]
```

## 代码段

此部分讨论 Hello World 示例中的部分重要代码。

### 创建网关

开发人员必须编写网关进程。该程序创建内部基础结构（消息总线）、加载模块，以及进行正常运行所需的所有设置。该 SDK 提供 **Gateway\_Create\_From\_JSON** 函数，允许你从 JSON 文件启动网关。若要使用 **Gateway\_Create\_From\_JSON** 函数，必须将 JSON 文件的路径传递给它，以便指定要加载的模块。

你可以在 Hello World 示例的 [main.c][lnk-main-c] 文件中找到网关进程的代码。为了增强可读性，下面的代码段显示的是简化版网关进程代码。该程序创建一个网关，在卸除该网关之前，会等待用户按 **Enter** 键。

```
int main(int argc, char** argv)
{
    GATEWAY_HANDLE gateway;
    if ((gateway = Gateway_Create_From_JSON(argv[1])) == NULL)
    {
        printf("failed to create the gateway from JSON\n");
    }
    else
    {
        printf("gateway successfully created from JSON\n");
        printf("gateway shall run until ENTER is pressed\n");
        (void)getchar();
        Gateway_LL_Destroy(gateway);
    }
	return 0;
}
```

JSON 设置文件包含要加载的模块的列表。每个模块必须指定：

- **module\_name**：模块的唯一名称。
- **module\_path**：包含模块的库的路径。在 Linux 上，这是一个 .so 文件，而在 Windows 上，这是一个 .dll 文件。
- **args**：模块所需的任何配置信息。

以下示例显示了用于在 Linux 上配置 Hello World 示例的 JSON 设置文件。模块是否需要参数取决于模块的设计。在此示例中，记录器模块使用的参数是输出文件的路径，而 Hello World 模块则不使用任何参数：

```
{
    "modules" :
    [ 
        {
            "module name" : "logger_hl",
            "module path" : "./modules/logger/liblogger_hl.so",
            "args" : {"filename":"log.txt"}
        },
        {
            "module name" : "helloworld",
            "module path" : "./modules/hello_world/libhello_world_hl.so",
			"args" : null
        }
    ]
}
```

### Hello World 模块消息发布

你可以在[“hello\_world.c”][lnk-helloworld-c]文件中找到“hello world”模块发布消息时使用的代码。以下代码段显示的是修改版，添加了更多的注释，并删除了部分处理错误的代码以增强可读性：

```
int helloWorldThread(void *param)
{
    // Create data structures used in function.
    HELLOWORLD_HANDLE_DATA* handleData = param;
    MESSAGE_CONFIG msgConfig;
    MAP_HANDLE propertiesMap = Map_Create(NULL);
    
    // Add a property named "helloWorld" with a value of "from Azure IoT
    // Gateway SDK simple sample!" to a set of message properties that
    // will be appended to the message before publishing it. 
    Map_AddOrUpdate(propertiesMap, "helloWorld", "from Azure IoT Gateway SDK simple sample!")

    // Set the content for the message
    msgConfig.size = strlen(HELLOWORLD_MESSAGE);
    msgConfig.source = HELLOWORLD_MESSAGE;

    // Set the properties for the message
    msgConfig.sourceProperties = propertiesMap;
    
    // Create a message based on the msgConfig structure
    MESSAGE_HANDLE helloWorldMessage = Message_Create(&msgConfig);

    while (1)
    {
        if (handleData->stopThread)
        {
            (void)Unlock(handleData->lockHandle);
            break; /*gets out of the thread*/
        }
        else
        {
            // Publish the message to the bus
            (void)MessageBus_Publish(handleData->busHandle, helloWorldMessage);
            (void)Unlock(handleData->lockHandle);
        }

        (void)ThreadAPI_Sleep(5000); /*every 5 seconds*/
    }

    Message_Destroy(helloWorldMessage);

    return 0;
}
```

### Hello World 模块消息处理

Hello World 模块不需处理其他模块发布到消息总线的任何消息。因此，在 Hello World 模块中实施消息回调时，使用的是不执行任何操作的函数。

```
static void HelloWorld_Receive(MODULE_HANDLE moduleHandle, MESSAGE_HANDLE messageHandle)
{
    /* No action, HelloWorld is not interested in any messages. */
}
```

### 记录器模块消息发布和处理

记录器模块接收来自消息总线的消息，并将其写入文件中。它不会将消息发布到消息总线。因此，记录器模块的代码不会调用 **MessageBus\_Publish** 函数。

[logger.c][lnk-logger-c] 文件中的 **Logger\_Recieve** 函数是消息总线发起的回调，用于将消息传递给记录器模块。以下代码段显示的是修改版，添加了更多的注释，并删除了部分处理错误的代码以增强可读性：

```
static void Logger_Receive(MODULE_HANDLE moduleHandle, MESSAGE_HANDLE messageHandle)
{

    time_t temp = time(NULL);
    struct tm* t = localtime(&temp);
    char timetemp[80] = { 0 };

    // Get the message properties from the message
    CONSTMAP_HANDLE originalProperties = Message_GetProperties(messageHandle); 
    MAP_HANDLE propertiesAsMap = ConstMap_CloneWriteable(originalProperties);

    // Convert the collection of properties into a JSON string
    STRING_HANDLE jsonProperties = Map_ToJSON(propertiesAsMap);

    //  base64 encode the message content
    const CONSTBUFFER * content = Message_GetContent(messageHandle);
    STRING_HANDLE contentAsJSON = Base64_Encode_Bytes(content->buffer, content->size);

    // Start the construction of the final string to be logged by adding
    // the timestamp
    STRING_HANDLE jsonToBeAppended = STRING_construct(",{"time":"");
    STRING_concat(jsonToBeAppended, timetemp);

    // Add the message properties
    STRING_concat(jsonToBeAppended, "","properties":"); 
    STRING_concat_with_STRING(jsonToBeAppended, jsonProperties);

    // Add the content
    STRING_concat(jsonToBeAppended, ","content":"");
    STRING_concat_with_STRING(jsonToBeAppended, contentAsJSON);
    STRING_concat(jsonToBeAppended, ""}]");

    // Write the formatted string
    LOGGER_HANDLE_DATA *handleData = (LOGGER_HANDLE_DATA *)moduleHandle;
    addJSONString(handleData->fout, STRING_c_str(jsonToBeAppended);
}
```

## 后续步骤

若要了解如何使用网关 SDK，请参阅：

- [IoT 网关 SDK – 使用 Linux 通过模拟设备发送设备至云消息][lnk-gateway-simulated]。
- GitHub 上的 [Azure IoT Gateway SDK（Azure IoT 网关 SDK）][lnk-gateway-sdk]。

有关使用 IoT 中心进行设备管理的详细信息，请参阅 [Azure IoT 中心设备管理概述][lnk-device-management]。

<!-- Links -->
[lnk-main-c]: https://github.com/Azure/azure-iot-gateway-sdk/blob/master/samples/hello_world/src/main.c
[lnk-helloworld-c]: https://github.com/Azure/azure-iot-gateway-sdk/blob/master/modules/hello_world/src/hello_world.c
[lnk-logger-c]: https://github.com/Azure/azure-iot-gateway-sdk/blob/master/modules/logger/src/logger.c
[lnk-gateway-sdk]: https://github.com/Azure/azure-iot-gateway-sdk/
[lnk-gateway-simulated]: /documentation/articles/iot-hub-linux-gateway-sdk-simulated-device/
[lnk-device-management]: /documentation/articles/iot-hub-device-management-overview/
<!---HONumber=Mooncake_0523_2016-->