## <a name="view-the-telemetry"></a>查看遥测数据

Raspberry Pi 现在正将遥测数据发送到远程监控解决方案。 可以在解决方案仪表板上查看遥测数据。 也可以从解决方案仪表板向 Raspberry Pi 发送消息。

- 导航到解决方案仪表板。
- 在“要查看的设备”下拉列表中选择设备。
- 来自 Raspberry Pi 的遥测数据将显示在仪表板上。

![显示来自 Raspberry Pi 的遥测数据][img-telemetry-display]

## <a name="act-on-the-device"></a>在设备上操作

可以通过解决方案仪表板调用 Raspberry Pi 上的方法。 Raspberry Pi 在连接到远程监控解决方案时，会发送所支持的方法的相关信息。

- 在解决方案仪表板中，单击“设备”即可访问“设备”页。 在“设备列表”中选择 Raspberry Pi。 然后选择“方法”：

    ![在仪表板中列出设备][img-list-devices]

- 在“调用方法”页的“方法”下拉列表中，选择“LightBlink”。

- 选择“InvokeMethod”。 模拟器会在 Raspberry Pi 的控制台中输出消息。 Raspberry Pi 上的应用会将确认发送回解决方案仪表板：

    ![显示方法历史记录][img-method-history]

- 可以使用 **ChangeLightStatus** 方法来开关 LED，**LightStatusValue** 设置为 **1** 时表示开，设置为 **0** 时表示关。

> [AZURE.WARNING]
> 如果让远程监控解决方案在 Azure 帐户中保持运行状态，系统会按其运行时间计费。 若要详细了解如何在远程监控解决方案运行时减少消耗，请参阅[出于演示目的配置 Azure IoT 套件预配置解决方案][lnk-demo-config]。 请在用完预配置的解决方案后将其从 Azure 帐户中删除。


[img-telemetry-display]: ./media/iot-suite-raspberry-pi-kit-view-telemetry-simulator/telemetry.png
[img-list-devices]: ./media/iot-suite-raspberry-pi-kit-view-telemetry-simulator/listdevices.png
[img-method-history]: ./media/iot-suite-raspberry-pi-kit-view-telemetry-simulator/methodhistory.png

[lnk-demo-config]: https://github.com/Azure/azure-iot-remote-monitoring/blob/master/Docs/configure-preconfigured-demo.md