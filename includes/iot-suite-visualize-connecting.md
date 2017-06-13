## 在仪表板中查看设备遥测数据
使用远程监控解决方案中的仪表板可以查看设备发送到 IoT 中心的遥测数据。

1. 在浏览器中，返回到远程监控解决方案仪表板，然后单击左侧面板中的“设备”导航到“设备列表”。
2. 在“设备列表”中，应会看到设备状态为“正在运行”。如果不是，请在“设备详细信息”面板中单击“启用设备”。
   
    ![查看服务状态][18]  

3. 单击“仪表板”返回到仪表板，然后在“要查看的设备”下拉列表中选择设备查看其遥测数据。示例应用程序的遥测数据是 50 个单位的内部温度、55 个单位的外部温度，以及 50 个单位的湿度。
   
    ![查看设备遥测数据][img-telemetry]  


## 在设备上调用方法
使用远程监控解决方案中的仪表板可通过 IoT 中心在设备上调用方法。例如，在远程监控解决方案中，可以调用方法来模拟重新启动设备。

1. 在远程监控解决方案仪表板中，单击左侧面板中的“设备”导航到“设备列表”。
2. 在“设备列表”中，单击设备的“设备 ID”。
3. 在“设备详细信息”面板中，单击“方法”。
   
    ![设备方法][13]  

4. 在“方法”下拉列表中选择“InitiateFirmwareUpdate”，然后在“FWPACKAGEURI”中输入虚构的 URL。单击“调用方法”在设备上调用该方法。
   
    ![调用设备方法][14]  

   

5. 设备处理方法时，运行设备代码的控制台中会显示消息。方法的结果将添加到解决方案门户中的历史记录：

    ![查看方法历史记录][img-method-history]  


## 后续步骤
[自定义预配置解决方案][lnk-customize]一文介绍了扩展本示例的一些方法。可能的扩展包括使用真实传感器和实现其他命令。

详细了解 [azureiotsuite.cn 站点权限][lnk-permissions]。

[13]: ./media/iot-suite-visualize-connecting/suite4.png
[14]: ./media/iot-suite-visualize-connecting/suite7-1.png
[18]: ./media/iot-suite-visualize-connecting/suite10.png
[img-telemetry]: ./media/iot-suite-visualize-connecting/telemetry.png
[img-method-history]: ./media/iot-suite-visualize-connecting/history.png
[lnk-customize]: /documentation/articles/iot-suite-guidance-on-customizing-preconfigured-solutions/
[lnk-permissions]: /documentation/articles/iot-suite-permissions/

<!---HONumber=Mooncake_0327_2017-->