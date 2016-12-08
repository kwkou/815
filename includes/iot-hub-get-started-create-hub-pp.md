## 创建支持设备管理的 IoT 中心

IoT 中心设备管理是预览版，因此需要创建支持设备管理的 IoT 中心。IoT 中心设备管理公开上市时，本教程会进行更新。以下步骤说明如何使用 Azure 门户预览完成此任务。

1.  登录到 [Azure 门户预览]。
2.  在跳转栏中，依次单击“新建”、“物联网”和“Azure IoT 中心”。

	![][img-new-hub]

3.  在“IoT 中心”边栏选项卡中，选择 IoT 中心的配置。

	![][img-configure-hub]

  -   在“名称”框中，输入 IoT 中心的名称。如果该“名称”有效且可用，“名称”框中会出现绿色的勾选标记。
  -   选择“定价和缩放层”。本教程不需要特定的层。
  -   在“资源组”中，创建新资源组或选择现有的资源组。有关详细信息，请参阅 [Using resource groups to manage your Azure resources]（使用资源组管理 Azure 资源）。
  -   选中“启用设备管理 - 预览版”复选框。
  -   在“位置”中，选择托管 IoT 中心的位置。

    > [AZURE.NOTE]  如果不选中“启用设备管理”复选框，示例将不起作用。

4.  选择 IoT 中心配置选项后，单击“创建”。Azure 可能需要几分钟时间来创建 IoT 中心。若要检查状态，可以在“启动板”或“通知”面板中监视进度。

	![][img-monitor]  


5.  成功创建 IoT 中心后，中心的边栏选项卡会自动打开。记下“主机名”，然后单击“共享访问策略”。

	![][img-keys]  


6.  单击“iothubowner”策略，然后复制并记下“iothubowner”边栏选项卡中的连接字符串。将该连接字符串复制到稍后可访问的位置，因为完成本教程需要它。

 	> [AZURE.NOTE] 在生产方案中，切勿使用 **iothubowner** 凭据。

	![][img-connection]

现在就已经创建好支持设备管理的 IoT 中心。需要使用连接字符串来完成本教程。

## 创建设备标识

在本部分中，会使用名为 IoT 中心资源管理器的节点工具为本教程创建设备标识。

1. 在命令行环境中运行以下命令：

    npm install -g iothub-explorer@latest

2. 然后，运行以下命令登录中心，请记得使用之前复制的 IoT 中心连接字符串替换 `{service connection string}`：

    iothub-explorer login "{service connection string}"

3. 最后，以下使用命令创建名为 `myDeviceId` 的新设备标识：

    iothub-explorer create myDeviceId --connection-string

记下结果中的设备连接字符串。设备应用使用此连接字符串以设备身份连接到 IoT 中心。

![][img-identity]  


若要了解以编程方式创建设备标识的方法，请参阅 [IoT 中心入门][lnk-getstarted]。

<!-- images and links -->

[img-new-hub]: ./media/iot-hub-get-started-create-hub-pp/image1.png
[img-configure-hub]: ./media/iot-hub-get-started-create-hub-pp/image2.png
[img-monitor]: ./media/iot-hub-get-started-create-hub-pp/image3.png
[img-keys]: ./media/iot-hub-get-started-create-hub-pp/image4.png
[img-connection]: ./media/iot-hub-get-started-create-hub-pp/image5.png
[img-identity]: ./media/iot-hub-get-started-create-hub-pp/devidentity.png

[Azure 门户预览]: https://portal.azure.cn/
[iot-hub-explorer]: https://github.com/Azure/azure-iot-sdks/tree/master/tools/iothub-explorer

[lnk-getstarted]: /documentation/articles/iot-hub-csharp-csharp-getstarted/
[Using resource groups to manage your Azure resources]: /documentation/articles/resource-group-portal/

<!---HONumber=Mooncake_1031_2016-->