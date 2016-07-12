> [AZURE.SELECTOR]
- [Linux](/documentation/articles/iot-hub-linux-gateway-sdk-simulated-device/)
- [Windows](/documentation/articles/iot-hub-windows-gateway-sdk-simulated-device/)

本文演示了 [Simulated Device Cloud Upload sample（模拟设备云上载示例）]，说明了如何使用 [Azure IoT Gateway SDK（Azure IoT 网关 SDK）][lnk-sdk]从模拟设备将设备到云的遥测发送到 IoT 中心。

本文介绍的内容包括：

1. **体系结构**：与模拟设备云上载示例有关的重要的体系结构信息。

2. **生成并运行**：生成并运行示例所需的步骤。

## 体系结构

模拟设备云上载示例说明了如何使用 SDK 来创建网关，以便将遥测从模拟设备发送到 IoT 中心。模拟设备无法直接连接到 IoT 中心，因为：

- 模拟设备使用的通信协议 IoT 中心无法理解。
- 模拟设备不够智能，无法记住 IoT 中心分配给它们的标识。

网关通过以下方式解决模拟设备的这些问题：

- 网关理解模拟设备所使用的协议，因此在从这些设备接收设备到云遥测后，会使用 IoT 中心理解的协议将这些消息转发到该中心。
- 网关代表模拟设备来存储 IoT 中心标识，并在模拟设备向 IoT 中心发送消息时充当代理。

下图显示了示例的主要组件，包括网关模块：

![][1]


> [AZURE.NOTE] 模块彼此之间不直接传递消息。模块将消息发布给内部消息总线，该总线通过订阅机制将消息传递给其他模块，如下图所示。有关详细信息，请参阅[网关 SDK 入门][lnk-gw-getstarted]。

### 协议引入模块

数据从设备获取后，将以此模块为起点，通过网关传递到云中。在示例中，该模块执行四项任务：

1.  创建模拟温度数据。
    
    注意：如果你使用真实的设备，该模块会从这些物理设备读取数据。

2.  将模拟温度数据置于消息内容中。

3.  将带有虚设 MAC 地址的属性添加到包含模拟温度数据的消息。

4.  向链中的下一模块提供该消息。

> [AZURE.NOTE] 在上图中称为“协议 X 引入”的模块在源代码中称为“模拟设备”。

### MAC &lt;-&gt; IoT 中心 ID 模块

此模块以扫描方式查找的消息包括一个属性，其中包含通过协议引入模块添加的模拟设备 MAC 地址。如果找到了这种属性，该模块就会将带有 IoT 中心设备密钥的另一属性添加到消息，并将该消息提供给链中的下一模块。这就是示例将 IoT 中心设备标识与模拟设备进行关联的机制。开发人员会在模块配置过程中手动设置 MAC 地址与 IoT 中心标识之间的映射。

> [AZURE.NOTE]  此示例使用 MAC 地址作为唯一的设备标识符，将其与 IoT 中心设备标识相关联。不过，你可以编写你自己的模块，让其使用其他的唯一标识符。例如，设备的唯一序列号或遥测数据中可能嵌入了唯一的设备名，你可以使用该名称来确定 IoT 中心设备标识。

### IoT 中心通信模块

此模块接收的消息带有前一模块分配的 IoT 中心设备标识，它会通过 HTTPS 将消息内容发送给 IoT 中心。HTTPS 是 IoT 中心理解的三大协议之一。

此模块不会为每个模拟设备打开一个到 IoT 中心的连接，而是只打开一个从网关到 IoT 中心的 HTTP 连接，通过该连接对发自所有模拟设备的连接进行多路复用。与为每个设备打开一个唯一的连接相比，这种方式可以让单个网关连接更多的设备，不管是模拟设备还是其他设备。

![][2]


<!-- Images -->
[1]: ./media/iot-hub-gateway-sdk-simulated-selector/image1.png
[2]: ./media/iot-hub-gateway-sdk-simulated-selector/image2.png

<!-- Links -->
[Simulated Device Cloud Upload sample（模拟设备云上载示例）]: https://github.com/Azure/azure-iot-gateway-sdk/blob/master/doc/sample_simulated_device_cloud_upload.md
[lnk-sdk]: https://github.com/Azure/azure-iot-gateway-sdk
[lnk-gw-getstarted]: /documentation/articles/iot-hub-linux-gateway-sdk-get-started/
<!---HONumber=Mooncake_0523_2016-->