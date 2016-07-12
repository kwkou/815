<properties
	pageTitle="使用适用于 C 语言的 Azure IoT 设备 SDK | Azure"
	description="了解并开始使用适用于 C 语言的 Azure IoT 设备 SDK 中的示例代码。"
	services="iot-hub"
	documentationCenter=""
	authors="olivierbloch"
	manager="timlt"
	editor=""/>

<tags
     ms.service="iot-hub"
     ms.date="03/29/2016"
     wacn.date="07/04/2016"/>

# 适用于 C 语言的 Azure IoT 设备 SDK 简介

**Azure IoT 设备 SDK** 是一个库集，旨在简化从 **Azure IoT 中心**服务发送事件和接收消息的过程。有各种不同的 SDK，每个 SDK 都以特定的平台为目标，而本文说明的是**适用于 C 语言的 Azure IoT 设备 SDK**。

适用于 C 语言的 Azure IoT 设备 SDK 以 ANSI C (C99) 编写，以获得最大可移植性。如此就很适合在一些平台和设备上运行 - 尤其是在以将磁盘和内存占用量降到最低作为优先考虑的情况下。

许多平台已对 SDK 进行了测试（有关详细信息，请参阅[平台和兼容性列表](/documentation/articles/iot-hub-tested-configurations/)）。尽管本文包含的是在 Windows 平台上运行的示例代码演示，但请记住，本文所述的代码在各种支持的平台上都完全相同。

本文将介绍适用于 C 语言的 Azure IoT 设备 SDK 的体系结构。我们将演示如何初始化设备库，将事件发送到 IoT 中心，以及从 IoT 中心接收消息。本文中的信息应足以让你开始使用 SDK，但同时也提供了有关库的其他信息的链接。

>> [AZURE.NOTE] 本文不包括有关如何使用 SDK 中的 C 语言库的设备管理功能的信息。若要了解如何使用设备管理功能，请参阅 [Introducing the Azure IoT Hub device management library for C（适用于 C 语言的 Azure IoT 中心设备管理库简介）](/documentation/articles/iot-hub-device-management-library/)。

## SDK 体系结构

可以在以下 GitHub 存储库中找到**适用于 C 语言的 Azure IoT 设备 SDK**：

[azure-iot-sdks](https://github.com/Azure/azure-iot-sdks)

在此存储库的 **master** 分支中可找到最新版本的库：

  ![](./media/iot-hub-device-sdk-c-intro/01-MasterBranch.PNG)

此存储库包含整个系列的 Azure IoT 设备 SDK。不过，本文介绍的是适用于 C 语言的 Azure IoT 设备 SDK（可在 **c** 文件夹中找到）。

  ![](./media/iot-hub-device-sdk-c-intro/02-CFolder.PNG)

* 此 SDK 的核心实现可在 **iothub\_client** 文件夹中找到，此文件夹包含 SDK 的最低 API 层的实现：**IoTHubClient** 库。此 **IoTHubClient** 库包含实现原始消息传送的 API，即将消息发送到 IoT 中心以及从 IoT 中心接收消息。如果你使用此库，就需负责实现消息序列化（最终使用下面描述的序列化程序示例），但与 IoT 中心通信的其他细节则由系统为你处理。
* **serializer** 文件夹包含帮助器函数和示例代码，演示了使用客户端库向 Azure IoT 中心发送消息之前如何序列化数据。请注意使用序列化程序不是必需的，仅为了提供便利。如果你使用**序列化程序**库，首先需要定义一个模型，以指定要发送到 IoT 中心的事件以及预期要从 IoT 中心接收的消息。定义此模型后，SDK 将提供一个 API 界面，让你轻松处理事件和消息，而无需担心序列化细节。此库依赖于其他使用一些协议（AMQP、MQTT）实现传输的开放源代码库。
* **IoTHubClient** 库依赖于其他开放源代码库：
   * [Azure C 共享实用程序](https://github.com/Azure/azure-c-shared-utility)库，此库在一些 Azure 相关的 C SDK 中为所需的基本任务（如字符串、列表操作、IO 等）提供常用功能
   * [Azure uAMQP](https://github.com/Azure/azure-uamqp-c) 库，此库是针对资源约束设备的 AMQP 的客户端实现的优化。
   * [Azure uMQTT](https://github.com/Azure/azure-umqtt-c) 库，此库是实现 MQTT 协议并针对资源约束设备进行了优化的通用型库。

查看示例代码可以更轻松地了解所有这些知识。以下部分将演练 SDK 中包含的几个示例应用程序。这应可让你轻松了解 SDK 体系结构层的各种功能以及 API 工作原理的简介。

## 运行示例之前

在运行适用于 C 语言的 Azure IoT 设备 SDK 中的示例之前，必须在你的 Azure 订阅中创建一个服务实例（如果没有）并完成 2 个任务：
* 准备开发环境
* 获取设备凭据。

如果需要在你的 Azure 订阅上创建一个 Azure IoT 中心实例，请按照[此处](https://github.com/Azure/azure-iot-sdks/blob/master/doc/setup_iothub.md)的说明执行操作。

SDK 中包含的[自述文件](https://github.com/Azure/azure-iot-sdks/tree/master/c)提供了有关准备开发环境和获取设备凭据的说明。
以下部分包含有关这些说明的一些额外注释。

### 准备开发环境

为某些平台提供了程序包（例如适用于 Windows 的 NuGet 或者适用于 Debian 和 Ubuntu 的 apt\_get），并且示例可使用这些程序包（如果可用），以下说明详细介绍了如何直接从代码中构建库和示例。

首先，需要从 GitHub 获取 SDK 的副本，然后构建源。请从 [GitHub 存储库](https://github.com/Azure/azure-iot-sdks)的 **master** 分支获取源的副本。

下载源副本后，必须完成 SDK 文章[准备开发环境](https://github.com/Azure/azure-iot-sdks/blob/master/c/doc/devbox_setup.md)中所述的步骤。


以下是一些提示，可帮助你完成准备指南中所述的过程：

-   当你安装 **CMake** 实用程序时，请选择将 **CMake** 添加到**所有用户**的系统 PATH（也可以添加到**当前用户**）：

  ![](./media/iot-hub-device-sdk-c-intro/08-CMake.PNG)


-   在打开**VS2015 开发人员命令提示符**之前，请先安装 Git 命令行工具。若要安装这些工具，请完成以下步骤：

	1. 启动 **Visual Studio 2015** 安装程序（或从“程序和功能”控制面板选择“Microsoft Visual Studio 2015”，然后选择“更改”）。
	
	2. 确保在安装程序中选择“Git for Windows”功能，但也可以选中“Visual Studio 的 GitHub 扩展”选项以提供 IDE 集成：

  		![](./media/iot-hub-device-sdk-c-intro/10-GitTools.PNG)

	3. 完成安装向导以安装工具。

	4. 将 Git 工具 **bin** 目录添加到系统 **PATH** 环境变量。在 Windows 上，屏幕如下所示：

  		![](./media/iot-hub-device-sdk-c-intro/11-GitToolsPath.PNG)


当你完成[准备开发环境](https://github.com/Azure/azure-iot-sdks/blob/master/c/doc/devbox_setup.md)页面上所述的所有步骤后，就可以编译示例应用程序。

### 获取设备凭据

现在你已设置好开发环境，下一步要做的就是获取一组设备凭据。若要使设备能够访问 IoT 中心，必须先将该设备添加到 IoT 中心设备注册表。添加设备时，你将获取一组所需的设备凭据，以便设备能够连接到 IoT 中心。下一部分中所述的示例应用程序预期的凭据的格式为**设备连接字符串**。

SDK 开放源代码存储库中提供了两个工具用来帮助管理 IoT 中心。一个是称为设备资源管理器的 Windows 应用程序，另一个是称为 iothub-explorer 的基于 node.js 的跨平台 CLI 工具。你可以在[此处](https://github.com/Azure/azure-iot-sdks/blob/master/doc/manage_iot_hub.md)了解更多有关这两种工具的信息。

本文中所述的在 Windows 上运行这些示例时使用的是设备资源管理器工具。但是，如果你更喜欢 CLI 工具，也可以使用 iothub-explorer。

[设备资源管理器](https://github.com/Azure/azure-iot-sdks/tree/master/tools/DeviceExplorer)工具使用 Azure IoT 服务库在 IoT 中心执行各种功能（包括添加设备）。如果你使用设备资源管理器来添加设备，将会获得相应的连接字符串。需要此连接字符串才能运行示例应用程序。

如果你不熟悉上述过程，以下过程描述了如何使用设备资源管理器来添加设备和获取设备连接字符串。

你可以在 [SDK 发布页面](https://github.com/Azure/azure-iot-sdks/releases)中找到设备资源管理器工具的 Windows 安装程序。但是也可以直接从其代码运行该工具，方法是在 **Visual Studio 2015** 中打开 **[DeviceExplorer.sln](https://github.com/Azure/azure-iot-sdks/blob/master/tools/DeviceExplorer/DeviceExplorer.sln)**，然后生成解决方案。

运行该程序时，你将看到此界面：

  ![](./media/iot-hub-device-sdk-c-intro/03-DeviceExplorer.PNG)

在第一个字段中输入你的 **IoT 中心连接字符串**，然后单击“更新”。这将配置该工具，以便与 IoT 中心通信。

配置 IoT 中心连接字符串后，单击“管理”选项卡：

  ![](./media/iot-hub-device-sdk-c-intro/04-ManagementTab.PNG)

你将在其中管理已注册到 IoT 中心的设备。

单击“创建”按钮即可创建设备。将显示一个已预先填充一组密钥（主密钥和辅助密钥）的对话框。你只需要输入**设备 ID**，然后单击“创建”。

  ![](./media/iot-hub-device-sdk-c-intro/05-CreateDevice.PNG)

创建设备后，“设备”列表将会更新，其中包含所有已注册的设备（包括刚刚创建的设备）。如果在新设备上单击右键，将看到此菜单：

  ![](./media/iot-hub-device-sdk-c-intro/06-RightClickDevice.PNG)

如果你选择“复制所选设备的连接字符串”选项，设备的连接字符串将复制到剪贴板。请保留连接字符串的副本。在运行后续部分中所述的示例应用程序时，将要用到它。

完成上述步骤后，可以开始运行一些代码。两个示例的主源文件顶部都有一个常量，此常量可让你输入连接字符串。例如，**iothub\_client\_sample\_amqp** 应用程序中的相应行如下所示。

```
static const char* connectionString = "[device connection string]";
```

如果要继续，请在此输入设备连接字符串，重新编译解决方案，然后即可运行示例。

## IoTHubClient

azure-iot-sdks 存储库的 **iothub\_client** 文件夹中有一个 **samples** 文件夹，其中包含名为 **iothub\_client\_sample\_amqp** 的应用程序。

Windows 版本的 **iothub\_client\_sample\_ampq** 应用程序包含以下 Visual Studio 解决方案：

  ![](./media/iot-hub-device-sdk-c-intro/12-iothub-client-sample-amqp.PNG)

此解决方案只包含一个项目。值得注意的是，此解决方案中安装了四个 NuGet 包：

- Microsoft.Azure.C.SharedUtility
- Microsoft.Azure.IoTHub.AmqpTransport
- Microsoft.Azure.IoTHub.IoTHubClient
- Microsoft.Azure.uamqp

在使用 SDK 时始终需要 **Microsoft.Azure.C.SharedUtility** 包。由于此示例依赖于 AMQP，因此还必须包括 **Microsoft.Azure.uamqp** 和 **Microsoft.Azure.IoTHub.AmqpTransport** 包（HTTP 和 MQTT 有对应的包）。由于此示例使用 **IoTHubClient** 库，因此还必须在解决方案中包含 **Microsoft.Azure.IoTHub.IoTHubClient** 包。

可以在 **iothub\_client\_sample\_amqp.c** 源文件中找到示例应用程序的实现。

我们将使用此示例应用程序来演示使用 **IoTHubClient** 库时所需的项目。

### 初始化库

> [AZURE.NOTE] 在开始使用库之前，你可能需要执行一些特定于平台的初始化。例如，如果你打算在 Linux 上使用 AMQPS，则必须初始化 OpenSSL 库。[GitHub 存储库](https://github.com/Azure/azure-iot-sdks)中的示例将在客户端启动时调用实用工具函数 **platform\_init**，并在退出之前调用 **platform\_deinit** 函数。这些函数在“platform.h”标头文件中声明。你应该在[存储库](https://github.com/Azure/azure-iot-sdks)中为目标平台检查这些函数定义，以确定是否需要在客户端中包含任何平台初始化代码。

只有在分配 IoT 中心客户端句柄之后，才可以开始使用库：

```
IOTHUB_CLIENT_HANDLE iotHubClientHandle;
iotHubClientHandle = IoTHubClient_CreateFromConnectionString(connectionString, AMQP_Protocol);
```

请注意，我们要将设备连接字符串副本（从设备资源管理器获取的连接字符串）传递给此函数。我们还需指定要使用的协议。本示例使用 AMQP，但也可以选择 MQTT 和 HTTP。

获取有效的 **IOTHUB\_CLIENT\_HANDLE** 后，可以开始调用 API 来发送事件以及从 IoT 中心接收消息。接下来我们将介绍这种操作。

### 发送事件

将事件发送到 IoT 中心需要完成以下步骤：

首先创建一条消息：

```
EVENT_INSTANCE message;
sprintf_s(msgText, sizeof(msgText), "Message_%d_From_IoTHubClient_Over_AMQP", i);
message.messageHandle = IoTHubMessage_CreateFromByteArray((const unsigned char*)msgText, strlen(msgText);
```

接下来发送消息：

```
IoTHubClient_SendEventAsync(iotHubClientHandle, message.messageHandle, SendConfirmationCallback, &message);
```

每次发送消息时，指定发送数据时所调用的回调函数的引用：

```
static void SendConfirmationCallback(IOTHUB_CLIENT_CONFIRMATION_RESULT result, void* userContextCallback)
{
    EVENT_INSTANCE* eventInstance = (EVENT_INSTANCE*)userContextCallback;
    (void)printf("Confirmation[%d] received for message tracking id = %d with result = %s\r\n", callbackCounter, eventInstance->messageTrackingId, ENUM_TO_STRING(IOTHUB_CLIENT_CONFIRMATION_RESULT, result));
    /* Some device specific action code goes here... */
    callbackCounter++;
    IoTHubMessage_Destroy(eventInstance->messageHandle);
}
```

处理完消息后，请注意对 **IoTHubMessage\_Destroy** 函数的调用。必须进行此调用，才能释放在创建消息时分配的资源。

### 接收消息

接收消息是一个异步操作。首先，请注册当设备接收消息时所要调用的回调：

```
int receiveContext = 0;
IoTHubClient_SetMessageCallback(iotHubClientHandle, ReceiveMessageCallback, &receiveContext);
```

最后一个参数是指向所需对象的 void 指针。在本示例中，这是一个指向整数的指针，但也可以是指向更复杂数据结构的指针。这样，回调函数便可与此函数的调用方以共享状态运行。

当设备接收消息时，将调用注册的回调函数：

```
static IOTHUBMESSAGE_DISPOSITION_RESULT ReceiveMessageCallback(IOTHUB_MESSAGE_HANDLE message, void* userContextCallback)
{
    int* counter = (int*)userContextCallback;
    const char* buffer;
    size_t size;
    if (IoTHubMessage_GetByteArray(message, (const unsigned char**)&buffer, &size) == IOTHUB_MESSAGE_OK)
    {
        (void)printf("Received Message [%d] with Data: <<<%.*s>>> & Size=%d\r\n", *counter, (int)size, buffer, (int)size);
    }

    /* Some device specific action code goes here... */
    (*counter)++;
    return IOTHUBMESSAGE_ACCEPTED;
}
```

请注意，需使用 **IoTHubMessage\_GetByteArray** 函数来检索消息（在本示例中是一个字符串）。

### 取消初始化库

完成发送事件和接收消息后，你可以取消初始化 IoT 库。为此，请发出以下函数调用：

```
IoTHubClient_Destroy(iotHubClientHandle);
```

这将释放 **IoTHubClient\_CreateFromConnectionString** 函数以前分配的资源。

如你所见，使用 **IoTHubClient** 库可以轻松发送事件和接收消息。该库将处理与 IoT 中心通信的详细信息，包括要使用哪个协议（从开发人员的立场来看，这是一个简单的配置选项）。

在如何对设备发送到 IoT 中心的事件进行序列化方面，**IoTHubClient** 库也可提供精确的控制。在某些情况下，这是一项优点，但在其他情况下，这是你不想要看到的实现细节。如果是这样，可以考虑使用下一部分中介绍的**序列化程序**库。

## 序列化程序

从概念上讲，**序列化程序**库位于 SDK 中的 **IoTHubClient** 库之上。它使用 **IoTHubClient** 库来与 IoT 中心进行底层通信，但它添加了建模功能，消除了开发人员处理消息序列化的负担。我们将通过一个示例充分演示此库的工作原理。

azure-iot-sdks 存储库中的 **serializer** 文件夹中有一个 **samples** 文件夹，其中包含名为 **simplesample\_amqp** 的应用程序。此示例的 Windows 版本包含以下 Visual Studio 解决方案：

  ![](./media/iot-hub-device-sdk-c-intro/14-simplesample_amqp.PNG)

如同前面的示例，此示例也包含多个 NuGet 包：

- Microsoft.Azure.C.SharedUtility
- Microsoft.Azure.IoTHub.AmqpTransport
- Microsoft.Azure.IoTHub.IoTHubClient
- Microsoft.Azure.IoTHub.Serializer
- Microsoft.Azure.uamqp

其中的大多数包已在前面的示例中出现过，但 **Microsoft.Azure.IoTHub.Serializer** 是新的。在使用**序列化程序**库时将用到它。

可以在 **simplesample\_amqp.c** 文件中找到示例应用程序的实现。

以下部分将演练本示例的重要组成部分。

### 初始化库

必须先调用初始化 API，然后才可以开始使用**序列化程序**库：

```
serializer_init(NULL);

IOTHUB_CLIENT_HANDLE iotHubClientHandle = IoTHubClient_CreateFromConnectionString(connectionString, AMQP_Protocol);

ContosoAnemometer* myWeather = CREATE_MODEL_INSTANCE(WeatherStation, ContosoAnemometer);
```

对 **serializer\_init** 函数进行的调用是一次性调用，用于初始化底层库。然后，需调用 **IoTHubClient\_CreateFromConnectionString** 函数，这是 **IoTHubClient** 示例中的同一 API。此调用将设置设备连接字符串（也可用于选择要使用的协议）。请注意，本示例使用 AMQP 作为传输方式，但也可以使用 HTTP。

最后，调用 **CREATE\_MODEL\_INSTANCE** 函数。请注意，**WeatherStation** 是模型的命名空间，**ContosoAnemometer** 是模型的名称。创建模型实例后，可以使用它来开始发送事件和接收消息。但是，必须了解模型是什么。

### 定义模型

**序列化程序**库中的模型定义了设备可发送到 IoT 中心的事件以及可接收的消息（在建模语言中称为操作）。正如 **simplesample\_amqp** 示例应用程序中所示，可以使用一组 C 宏来定义模型：

```
BEGIN_NAMESPACE(WeatherStation);

DECLARE_MODEL(ContosoAnemometer,
WITH_DATA(ascii_char_ptr, DeviceId),
WITH_DATA(double, WindSpeed),
WITH_ACTION(TurnFanOn),
WITH_ACTION(TurnFanOff),
WITH_ACTION(SetAirResistance, int, Position)
);

END_NAMESPACE(WeatherStation);
```

**BEGIN\_NAMESPACE** 和 **END\_NAMESPACE** 这两个宏都以模型的命名空间作为参数。介于这两个宏之间的内容应该就是模型的定义和模型使用的数据结构。

在本示例中，有一个名为 **ContosoAnemometer** 的模型。此模型定义了设备可以发送到 IoT 中心的两个事件：**DeviceId** 和 **WindSpeed**。它还定义了设备可以接收的三个操作（消息）：**TurnFanOn**、**TurnFanOff** 和 **SetAirResistance**。每个事件都有一个类型，而每个操作都有一个名称（以及一组可选参数）。

模型中定义的事件和操作可定义 API 接口，此接口可用于将事件发送到 IoT 中心，以及响应发送到设备的消息。最好通过示例了解相关知识。

### 发送事件

模型定义了可以发送到 IoT 中心的事件。在本示例中，这是指使用 **WITH\_DATA** 宏来定义的两个事件之一。例如，如果你想要将 **WindSpeed** 事件发送到 IoT 中心，就需要执行以下几个步骤。第一个步骤是设置要发送的数据：

```
myWeather->WindSpeed = 15;
```

前面定义的模型可让你通过设置 **struct** 的成员来实现此目的。接下来，序列化想要发送的事件：

```
unsigned char* destination;
size_t destinationSize;

SERIALIZE(&destination, &destinationSize, myWeather->WindSpeed);
```

此代码将此事件序列化到缓冲区（由 **destination** 引用）。最后，使用以下代码将事件发送到 IoT 中心：

```
sendMessage(iotHubClientHandle, destination, destinationSize);
```

在示例应用程序中，这是将已序列化的事件发送到 IoT 中心的帮助器函数：

```
static void sendMessage(IOTHUB_CLIENT_HANDLE iotHubClientHandle, const unsigned char* buffer, size_t size)
{
    static unsigned int messageTrackingId;
    IOTHUB_MESSAGE_HANDLE messageHandle = IoTHubMessage_CreateFromByteArray(buffer, size);
    if (messageHandle != NULL)
    {
        if (IoTHubClient_SendEventAsync(iotHubClientHandle, messageHandle, sendCallback, (void*)(uintptr_t)messageTrackingId) != IOTHUB_CLIENT_OK)
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
    messageTrackingId++;
}
```

此代码非常类似于 **iothub\_client\_sample\_amqp** 应用程序中的代码，在此代码中我们使用某个字节数组创建了一条消息，然后使用 **IoTHubClient\_SendEventAsync** 将它发送到 IoT 中心。此后，只需释放以前分配的消息句柄和已序列化的数据缓冲区。

**IoTHubClient\_SendEventAsync** 的倒数第二个参数是对成功发送数据后所调用的回调函数的引用。下面是回调函数的示例：

```
void sendCallback(IOTHUB_CLIENT_CONFIRMATION_RESULT result, void* userContextCallback)
{
    int messageTrackingId = (intptr_t)userContextCallback;

    (void)printf("Message Id: %d Received.\r\n", messageTrackingId);

    (void)printf("Result Call Back Called! Result is: %s \r\n", ENUM_TO_STRING(IOTHUB_CLIENT_CONFIRMATION_RESULT, result));
}
```

第二个参数是指向用户上下文的指针，即我们传递给 **IoTHubClient\_SendEventAsync** 的同一个指针。在本例中，该上下文是一个简易计数器，但也可以是你想要的任何装置。

发送事件就是这么简单。最后要介绍的内容是如何接收消息。

### 接收消息

接收消息的方式类似于在 **IoTHubClient** 库中处理消息。首先，需要注册消息回调函数：

```
IoTHubClient_SetMessageCallback(iotHubClientHandle, IoTHubMessage, myWeather)
```

然后编写在接收消息时要调用的回调函数：

```
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
```

此代码是一个样板 - 对任何解决方案都是相同的。此函数将接收消息并通过调用 **EXECUTE\_COMMAND** 将它路由到相应的函数。此时调用的函数取决于模型中的操作定义。

在模型中定义操作时，需要实现当设备接收相应的消息时调用的函数。例如，如果模型定义了此操作：

```
WITH_ACTION(SetAirResistance, int, Position)
```

你必须使用此签名来定义函数：

```
EXECUTE_COMMAND_RESULT SetAirResistance(ContosoAnemometer* device, int Position)
{
    (void)device;
    (void)printf("Setting Air Resistance Position to %d.\r\n", Position);
    return EXECUTE_COMMAND_SUCCESS;
}
```

请注意，函数的名称与模型中的操作名称匹配，而函数的参数与为该操作指定的参数匹配。第一个参数始终是必需的，包含指向模型实例的指针。

当设备收到与此签名匹配的消息时，将调用相应的函数。因此，除了必须包含 **IoTHubMessage** 中的样板代码以外，接收消息所涉及的操作只是为模型中定义的每个操作定义一个简单的函数。

### 取消初始化库

完成发送数据和接收消息后，你可以取消初始化 IoT 库。

```
        DESTROY_MODEL_INSTANCE(myWeather);
    }
    IoTHubClient_Destroy(iotHubClientHandle);
}
serializer_deinit();
```

上述三个函数均符合以前所述的三个初始化函数。调用这些 API 可确保释放以前分配的资源。

## 后续步骤

本文介绍了有关使用**适用于 C 语言的 Azure IoT 设备 SDK**中的库的基本知识。其中提供了足够的信息来让你了解 SDK 中包含哪些组件及其体系结构，以及如何开始使用 Windows 示例。下一篇文章通过讲解[有关 IoTHubClient 库的详细信息](/documentation/articles/iot-hub-device-sdk-c-iothubclient/)来继续介绍该 SDK。
若要了解如何使用**适用于 C 的 Azure IoT 设备 SDK** 中的设备管理功能，请参阅[适用于 C 语言的 Azure IoT 中心设备管理库简介](/documentation/articles/iot-hub-device-management-library/)。

<!---HONumber=Mooncake_0307_2016-->