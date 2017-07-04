## <a name="view-the-telemetry"></a>查看遥测数据

Raspberry Pi 现在正将遥测数据发送到远程监控解决方案。 可以在解决方案仪表板上查看遥测数据。 也可以从解决方案仪表板向 Raspberry Pi 发送消息。

- 导航到解决方案仪表板。
- 在“要查看的设备”下拉列表中选择设备。
- 来自 Raspberry Pi 的遥测数据将显示在仪表板上。

![显示来自 Raspberry Pi 的遥测数据][img-telemetry-display]

## <a name="initiate-the-firmware-update"></a>启动固件更新

固件更新过程会下载更新版的设备客户端应用程序并将其安装在 Raspberry Pi 上。 若要详细了解固件更新过程，请参阅[使用 IoT 中心进行设备管理的概述][lnk-update-pattern]一文中对固件更新模式的说明。

在设备上调用相应的方法即可启动固件更新过程。 该方法是异步方法，一旦更新过程开始就会返回。 设备使用报告的属性，将更新进展通知给解决方案。

可以通过解决方案仪表板调用 Raspberry Pi 上的方法。 Raspberry Pi 在首次连接到远程监控解决方案时，会发送所支持的方法的相关信息。 

[img-telemetry-display]: ./media/iot-suite-raspberry-pi-kit-view-telemetry-advanced/telemetry.png
[lnk-update-pattern]: /documentation/articles/iot-hub-device-management-overview/