> [AZURE.SELECTOR]
- [Windows 上的 C](/documentation/articles/iot-suite-connecting-devices/)
- [Linux 上的 C](/documentation/articles/iot-suite-connecting-devices-linux/)
- [mbed 上的 C](/documentation/articles/iot-suite-connecting-devices-mbed/)
- [Node.js](/documentation/articles/iot-suite-connecting-devices-node/)

## 方案概述

在此方案中，你要创建会将以下遥测数据发送到远程监视[预配置解决方案][lnk-what-are-preconfig-solutions]的设备：

- 外部温度
- 内部温度
- 湿度

为简单起见，设备上的代码将生成示例值，但我们建议你通过将实际传感器连接到设备并发送实际的遥测数据来扩展此示例。

若要完成此教程，你需要一个有效的 Azure 帐户。如果你没有帐户，可以创建一个试用帐户，只需几分钟即可完成。有关详细信息，请参阅 [Azure 试用][lnk-1rmb-trial]。

## 开始之前

在为设备编写任何代码之前，必须先预配远程监视预配置解决方案，然后在该解决方案中预配设备。

### 预配远程监视预配置解决方案

创建的设备将发送数据到[远程监视][lnk-remote-monitoring]预配置解决方案的实例。请访问 [Azure IoT 套件入门][lnk-getstarted]，以在 Azure 帐户中预配远程监视预配置解决方案：

1. 在 https://www.azureiotsuite.cn/ 页面上，单击“+”创建新解决方案。

2. 单击“远程监视”面板上的“选择”以创建新解决方案。

3. 在“创建‘远程监视’解决方案”页面上，输入所选的“解决方案名称”，选择要在其中进行部署的“区域”，然后选择想要使用的 Azure 订阅。单击“创建解决方案”。

4. 等待预配过程完成。

> [AZURE.WARNING] 预配置的解决方案使用可计费的 Azure 服务。当使用完预配置的解决方案之后，请务必将它从订阅中删除，以避免产生任何不必要的费用。只需访问 https://www.azureiotsuite.cn/ 页面，即可将预配置的解决方案从订阅中完全删除。

预配好远程监视解决方案之后，请单击“启动”，在浏览器中打开解决方案仪表板。

![][img-dashboard]

### 在远程监视方案中预配设备

> [AZURE.NOTE] 如果你已在解决方案中预配了设备，则可以跳过此步骤。创建客户端应用程序时需要知道设备凭据。

对于连接到预配置解决方案的设备，该设备必须能够使用有效的凭据识别自身。你可以从解决方案仪表板获取设备凭据，然后将它们包含在客户端应用程序中。

若要在远程监视解决方案中添加设备，请在解决方案仪表板中完成以下步骤：

1.  在仪表板左下角，单击“添加设备”。

    ![][1]

2.  在“自定义设备”面板中单击“添加”。

    ![][2]

3.  选择“让我自行定义设备 ID”，输入设备 ID（例如 “mydevice”），单击“检查 ID”以验证该名称是否未被使用，然后单击“创建”以预配设备。

    ![][3]

5. 记下设备的凭据（设备 ID、IoT 中心主机名和设备密钥），客户端应用程序需要这些信息才能将设备连接到远程监视解决方案。然后单击“Done”（完成）。

    ![][4]

6. 确定设备在设备部署中正确显示。在设备与远程监视解决方案创建连接之前，状态一直是“挂起”。

    ![][5]

[img-dashboard]: ./media/iot-suite-selector-connecting/dashboard.png
[1]: ./media/iot-suite-selector-connecting/suite0.png
[2]: ./media/iot-suite-selector-connecting/suite1.png
[3]: ./media/iot-suite-selector-connecting/suite2.png
[4]: ./media/iot-suite-selector-connecting/suite3.png
[5]: ./media/iot-suite-selector-connecting/suite5.png

[lnk-getstarted]: https://www.azureiotsuite.cn/
[lnk-what-are-preconfig-solutions]: /documentation/articles/iot-suite-what-are-preconfigured-solutions/
[lnk-remote-monitoring]: /documentation/articles/iot-suite-remote-monitoring-sample-walkthrough/
[lnk-1rmb-trial]: /pricing/1rmb-trial/
<!---HONumber=Mooncake_0523_2016-->