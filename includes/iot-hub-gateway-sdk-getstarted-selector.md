> [AZURE.SELECTOR]
- [Linux](/documentation/articles/iot-hub-linux-gateway-sdk-get-started/)
- [Windows](/documentation/articles/iot-hub-windows-gateway-sdk-get-started/)

本文详细介绍了 [Hello World sample code（Hello World 示例代码）][lnk-helloworld-sample]，说明了 [Azure IoT Gateway SDK（Azure IoT 网关 SDK）][lnk-gateway-sdk]体系结构的基本组件。该示例使用网关 SDK 生成一个简单的网关，每隔 5 秒将“hello world”消息记录到文件中。

本文介绍的内容包括：

- **概念**：通过概念来大致了解相关组件，这些组件组成你使用网关 SDK 创建的网关。  
- **Hello World 示例体系结构**：说明如何将概念应用到 Hello World 示例，以及如何将这些组件组合到一起。
- **如何生成示例**：生成示例所需的步骤。
- **如何运行示例**：运行示例所需的步骤。 
- **典型输出**：运行示例时预期获得的输出的示例。
- **代码段**：代码段的集合，显示 Hello World 示例如何实现重要的网关组件。

## 网关 SDK 概念

在查看示例代码或使用网关 SDK 创建你自己的现场网关之前，你应该了解支撑 SDK 体系结构的重要概念。

### 模块

使用 Azure IoT 网关 SDK 生成网关时，需创建*模块* 并对其进行组合。模块通过*消息* 来相互交换数据。一个模块收到一条消息，对其执行某些操作，选择性地将其转换为新的消息，然后将其发布，供其他模块处理。某些模块可能只生成新消息，不处理传入消息。一系列的模块会生成一个数据处理管道，每个模块在该管道的某个点对数据进行转换。

![][1]
 
该 SDK 包含：

- 预先编写的模块，执行常见的网关功能。
- 供开发人员编写自定义模块的接口。
- 部署和运行一组模块所需的基础结构。

该 SDK 提供一个抽象层，你可以利用该抽象层生成在各种操作系统和平台上运行的网关。

![][2]

### 消息

虽然可以通过思考模块互相传递消息的方式来了解网关运行过程中的概念，但这种思考过程并不能准确地反映实际发生的情况。模块使用消息总线来互相通信，它们会将消息发布到总线，然后总线就会将这些消息向连接到该总线的所有模块广播。

模块使用 **MessageBus\_Publish** 函数将消息发布到消息总线。消息总线通过调用回调函数将消息传递给模块。一条消息由一组键/值属性以及作为内存块传递的内容组成。

![][3]

由各个模块负责筛选消息，因为消息总线使用广播机制将每条消息传递给连接到该总线的所有模块。模块只会对应该发送给它的消息进行操作。消息筛选实际上是生成消息管道。模块通常会根据消息属性对接收到的消息进行筛选，以确定哪些消息应该处理。

## Hello World 示例体系结构

Hello World 示例体现了上一部分所述概念。Hello World 示例所实现的网关具有一个管道，该管道包含两个模块：

-	hello world 模块每 5 秒创建一条消息，并将该消息传递给记录器模块。
-	记录器 模块将接收的消息写入文件。

![][4]

如前一部分所述，Hello World 模块并不是每隔 5 秒就直接将消息传递给记录器模块，而是每隔 5 秒将消息发布到消息总线。

记录器模块从消息总线接收消息，在筛选器中检查其属性。如果记录器模块确定它应处理该消息，则会将消息内容写入文件。

记录器模块只从消息总线接收消息，而不将新消息发布到总线。

![][5]

上图显示了 Hello World 示例的体系结构，同时显示了在 [repository（存储库）][lnk-gateway-sdk]中对示例不同部分进行实施的源文件的相对路径。请自行浏览代码，或使用下面的代码段作为指导。

<!-- Images -->
[1]: ./media/iot-hub-gateway-sdk-getstarted-selector/modules.png
[2]: ./media/iot-hub-gateway-sdk-getstarted-selector/modules_2.png
[3]: ./media/iot-hub-gateway-sdk-getstarted-selector/messages_1.png
[4]: ./media/iot-hub-gateway-sdk-getstarted-selector/high_level_architecture.png
[5]: ./media/iot-hub-gateway-sdk-getstarted-selector/detailed_architecture.png

<!-- Links -->
[lnk-helloworld-sample]: https://github.com/Azure/azure-iot-gateway-sdk/tree/master/samples/hello_world
[lnk-gateway-sdk]: https://github.com/Azure/azure-iot-gateway-sdk
<!---HONumber=Mooncake_0523_2016-->