## <a name="view-the-solution-dashboard"></a>查看解决方案仪表板

该解决方案仪表板可让你管理部署的解决方案。 例如，可以查看遥测数据、添加设备以及调用方法。

1. 当预配完成且预配置解决方案的磁贴指示“就绪”时，请选择“启动”，在新的选项卡中打开远程监控解决方案门户。

    ![启动预配置解决方案][img-launch-solution]

1. 默认情况下，此解决方案门户会显示 *仪表板*。 可以通过页面左侧的菜单导航到解决方案门户中的其他区域。

    ![远程监控预配置解决方案仪表板][img-menu]

## <a name="add-a-device"></a>添加设备

连接到预配置解决方案的设备必须能够使用有效凭据对 IoT 中心识别自身。 用户可从解决方案仪表板中检索设备凭据。 本教程后文中的客户端应用程序要采用该设备凭据。

如果尚未这样做，请向远程监控解决方案添加自定义设备。 在解决方案仪表板中完成以下步骤：

1. 在仪表板左下角，单击“添加设备” 。

    ![添加设备][1]

1. 在“自定义设备”面板中，单击“新增”。

    ![添加自定义设备][2]

1. 选择“让我定义自己的设备 ID”。 输入设备 ID（例如“rasppi”），单击“检查 ID”验证该名称是否尚未在解决方案中使用，然后单击“创建”预配设备。

    ![添加设备 ID][3]

1. 记下设备凭据（“设备 ID”、“IoT 中心主机名”和“设备密钥”）。 Raspberry Pi 上的客户端应用程序需要这些值才能连接到远程监控解决方案。 然后单击“完成”。

    ![查看设备凭据][4]

1. 在解决方案仪表板上的设备列表中选择设备。 然后，在“设备详细信息”面板中，单击“启用设备”。 设备状态现在为“正在运行”。 远程监控解决方案现在可以从设备接收遥测数据，并在设备上调用方法。

[img-launch-solution]: ./media/iot-suite-raspberry-pi-kit-view-solution/launch.png
[img-menu]: ./media/iot-suite-raspberry-pi-kit-view-solution/menu.png
[1]: ./media/iot-suite-raspberry-pi-kit-view-solution/suite0.png
[2]: ./media/iot-suite-raspberry-pi-kit-view-solution/suite1.png
[3]: ./media/iot-suite-raspberry-pi-kit-view-solution/suite2.png
[4]: ./media/iot-suite-raspberry-pi-kit-view-solution/suite3.png