## <a name="create-an-iot-hub"></a>创建 IoT 中心
创建模拟设备应用要连接到的 IoT 中心。 以下步骤说明如何使用 Azure 门户来完成此任务。

1. 登录到 [Azure 门户][lnk-portal]。

2. 在“跳转栏”中，依次单击“新建” > “物联网” > “IoT 中心”。

    ![Azure 门户跳转栏][1]

3. 在“IoT 中心”边栏选项卡中，选择 IoT 中心的配置。

    ![IoT 中心边栏选项卡][2]

   * 在“名称”框中，输入 IoT 中心的名称。 如果该“名称”有效且可用，“名称”框中会出现绿色的勾选标记。
   * 选择 [定价和缩放层][lnk-pricing]。 本教程不需要特定的层。 对于本教程，请使用免费 F1 层。
   * 在“资源组”中，创建资源组或选择现有的资源组。 有关详细信息，请参阅[使用资源组管理 Azure 资源][lnk-resource-groups]。
   * 在“位置”中，选择托管 IoT 中心的位置。 对于本教程，请选择最近位置。
4. 选择 IoT 中心配置选项后，单击“创建”。  Azure 可能需要几分钟时间来创建 IoT 中心。 若要检查状态，可以在“启动板”或“通知”面板中监视进度。

    ![新的 IoT 中心状态][3]
5. 成功创建 IoT 中心后，请在 Azure 门户中单击 IoT 中心对应的新磁贴，以打开新 IoT 中心的边栏选项卡。 记下“主机名”，然后单击“共享访问策略”。

    ![新的 IoT 中心边栏选项卡][4]
6. 在“共享访问策略”边栏选项卡中，单击“iothubowner”策略，然后复制并记下“iothubowner”边栏选项卡中的 IoT 中心连接字符串。 有关详细信息，请参阅“IoT 中心开发人员指南”中的[访问控制][lnk-access-control]。

    ![共享访问策略边栏选项卡][5]  



<!-- Images. -->
[1]: ./media/iot-hub-get-started-create-hub/create-iot-hub1.png
[2]: ./media/iot-hub-get-started-create-hub/create-iot-hub2.png
[3]: ./media/iot-hub-get-started-create-hub/create-iot-hub3.png
[4]: ./media/iot-hub-get-started-create-hub/create-iot-hub4.png
[5]: ./media/iot-hub-get-started-create-hub/create-iot-hub5.png

<!-- Links -->
[lnk-resource-groups]: /documentation/articles/resource-group-portal/
[lnk-portal]: https://portal.azure.cn/
[lnk-pricing]: /pricing/details/iot-hub/
[lnk-access-control]: /documentation/articles/iot-hub-devguide-security/