## 在仪表板中查看设备遥测数据

远程监视解决方案中的仪表板可让你查看设备发送到 IoT 中心的遥测数据。

1. 在浏览器中，返回到远程监视解决方案仪表板，然后单击左侧面板中的“设备”导航到“设备列表”。

2. 在“设备列表”中，你应会看到设备状态现在为“正在运行”。

    ![][18]

3. 单击“仪表板”返回到仪表板，然后在“要查看的设备”下拉列表中选择设备，以查看其遥测数据。示例应用程序的遥测数据是 50 个单位的内部温度、55 个单位的外部温度，以及 50 个单位的湿度。请注意，仪表板默认只显示温度和湿度的值。

    ![][img-telemetry]

## 将命令发送到设备

远程监视解决方案中的仪表板可让你请求 IoT 中心向设备发送命令。例如，在远程监视解决方案中，你可以发送命令来设置设备的内部温度。

1. 在远程监视解决方案仪表板中，单击左侧面板中的“设备”导航到“设备列表”。

2. 在“设备列表”中，单击设备的“设备 ID”。

3. 在“设备详细信息”面板中，单击“命令”。

    ![][13]

4. 在“命令”下拉列表中选择“SetTemperature”，然后在“温度”中输入新的温度值。单击“发送命令”以将命令发送到设备。

    ![][14]

    > [AZURE.NOTE] 命令历史记录一开始将命令状态显示为“挂起”。设备确认命令后，状态将更改为“成功”。

5. 在仪表板上检查该设备现在是否发送 75 作为新的温度值。

## 后续步骤

[自定义预配置解决方案][lnk-customize]一文介绍了扩展本示例的一些方法。可能的扩展包括使用真实传感器和实现其他命令。

[13]: ./media/iot-suite-visualize-connecting/suite4.png
[14]: ./media/iot-suite-visualize-connecting/suite7-1.png
[18]: ./media/iot-suite-visualize-connecting/suite10.png
[img-telemetry]: ./media/iot-suite-visualize-connecting/telemetry.png
[lnk-customize]: /documentation/articles/iot-suite-guidance-on-customizing-preconfigured-solutions/
[lnk-dev-messaging]: /documentation/articles/iot-hub-devguide/#messaging

<!---HONumber=Mooncake_0523_2016-->