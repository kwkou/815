<properties
	pageTitle="预配置解决方案入门 | Azure"
	description="遵循本教程，了解如何部署 Azure IoT 套件预配置解决方案。"
	services=""
    suite="iot-suite"
	documentationCenter=""
	authors="dominicbetts"
	manager="timlt"
	editor=""/>  


<tags
     ms.service="iot-suite"
     ms.devlang="na"
     ms.topic="hero-article"
     ms.tgt_pltfrm="na"
     ms.workload="na"
     ms.date="08/16/2016"
     ms.author="dobett"
     wacn.date="10/10/2016"/>

# 教程：预配置解决方案入门

## 介绍

Azure IoT 套件[预配置解决方案][lnk-preconfigured-solutions]结合了多项 Azure IoT 服务，以提供可实现常见 IoT 商业应用场景的端到端解决方案。*远程监视*预配置解决方案将连接并监视设备。可使用解决方案分析设备发出的数据流，并让进程自动响应该数据流来提升业务绩效。

本教程演示如何预配远程监视预配置解决方案。还将逐步解说远程监视解决方案的基本功能。可以通过随预配置解决方案一起部署的解决方案仪表板来访问其中的多项功能：

![远程监控预配置解决方案仪表板][img-dashboard]  


需要有效的 Azure 订阅才能完成此教程。

> [AZURE.NOTE]  如果没有帐户，可以创建一个试用帐户，只需几分钟即可完成。有关详细信息，请参阅 [Trial][1rmb-trial]（试用）。

[AZURE.INCLUDE [iot-suite-provision-remote-monitoring](../../includes/iot-suite-provision-remote-monitoring.md)]

## 查看解决方案仪表板

该解决方案仪表板可让你管理部署的解决方案。例如，你可以查看遥测数据、添加设备以及配置规则。

1.  当预配完成且预配置解决方案的磁贴指示“就绪”时，单击“启动”以在新的选项卡中打开远程监视解决方案门户。

    ![启动预配置解决方案][img-launch-solution]  


2.  默认情况下，此解决方案门户会显示 *解决方案仪表板* 。你可以使用左侧菜单选择其他视图。

    ![远程监控预配置解决方案仪表板][img-dashboard]  


仪表板将显示以下信息：

- 地图显示每个设备连接到解决方案的位置。第一次运行解决方案时有四个模拟设备。模拟设备作为 Azure WebJobs 实现，而解决方案使用必应地图 API 在地图上绘制信息。
- “遥测历史记录”面板从所选的设备绘制近乎实时的湿度和温度遥测数据，并显示聚合数据，例如最大、最小和平均湿度。
- “警报历史记录”面板在遥测值超过阈值时显示最近的警报事件。除了预配置解决方案所创建的示例之外，还可以定义自己的警报。

## 查看设备列表

设备列表会显示解决方案中所有已注册的设备。你可以查看和编辑设备元数据、添加或删除设备，以及向设备发送命令。

1.  单击左侧菜单中的“设备”，以显示此解决方案的 *设备列表* 。

    ![仪表板中的设备列表][img-devicelist]  


2.  此设备列表显示预配过程所创建的四个模拟设备。

3.  单击设备列表中的设备，查看设备详细信息。

    ![仪表板中的设备详细信息][img-devicedetails]  


“设备详细信息”面板包含三个部分：

- “操作”部分列出可以在设备上执行的操作。设备被禁用后，不可再发送遥测数据或接收命令。如果禁用设备，之后可以重新启用它。你可以添加与设备关联的规则，以便在遥测值超出阈值时触发警报。也可以向设备发送命令。设备首次连接时，会告诉解决方案其可响应的命令。
- “设备属性”部分列出设备元数据。此元数据某些来自设备本身（例如制造商），某些由解决方案生成（例如创建时间）。你可以从此处编辑设备元数据。
- “身份验证密钥”部分列出设备可用来向解决方案进行身份验证的密钥。

## 向设备发送命令

设备详细信息窗格显示特定设备支持的所有命令，还可用于向设备发送命令。设备在第一次启动时，会向解决方案发送其支持的命令的相关信息。

1.  单击所选设备的设备详细信息窗格中的“命令”。

    ![仪表板中的设备命令][img-devicecommands]  


2.  从命令列表中选择“PingDevice”。

3.  单击“发送命令”。

4.  你可以在命令历史记录中查看命令的状态。

    ![仪表板中的命令状态][img-pingcommand]  


解决方案会跟踪其发送的每个命令的状态。结果最初为“挂起”。当设备报告它已执行命令时，结果会设置为“成功”。

## 添加新的模拟设备

当你部署预配置解决方案时，会自动预配在设备列表中可见的四个示例设备。这些设备是在 Azure Web 作业中运行的*模拟设备*。模拟设备可让你试验预配置解决方案，而不需要部署实际的物理设备。若要将实际设备连接到解决方案，请参阅[将设备连接到远程监视预配置解决方案][lnk-connect-rm]教程。

以下步骤说明了如何向解决方案添加模拟设备：

1.  导航回到设备列表。

2.  单击左下角的“+添加设备”进行添加。

    ![将设备添加到预配置解决方案][img-adddevice]  


3.  单击“模拟设备”磁贴上的“新增”。

    ![在仪表板中设置新设备详细信息][img-addnew]
    
    如果选择创建“自定义设备”，则除了创建新的模拟设备，也可以添加物理设备。若要深入了解如何将物理设备连接到解决方案，请参阅[将设备连接到 IoT 套件远程监视预配置解决方案][lnk-connect-rm]。

4.  选择“自行定义设备 ID”，然后输入唯一的设备 ID 名称，例如 **mydevice\_01**。

5.  单击“创建”。

    ![保存新设备][img-definedevice]  


6. 在“添加模拟设备”的步骤 3 中，单击“完成”回到设备列表。

7. 可以在设备列表中查看“正在运行”的设备。

    ![查看设备列表中的新设备][img-runningnew]

8. 也可以在仪表板上查看来自新设备的模拟遥测：

    ![查看新设备发出的遥测数据][img-runningnew-2]  


## 编辑设备元数据

设备首次连接到解决方案后，会向该方案发送其元数据。当你通过解决方案仪表板编辑设备元数据时，会将新的元数据值发送到设备，并将新值存储在解决方案的 DocumentDB 数据库中。有关详细信息，请参阅 [Device identity registry and DocumentDB][lnk-devicemetadata]（设备标识注册表和 DocumentDB）。

1.  导航回到设备列表。

2.  在“设备列表”中选择新设备，然后单击“编辑”以编辑“设备属性”：

    ![编辑设备元数据][img-editdevice]  


3. 向下滚动并对纬度和经度值进行更改。然后单击“将更改保存到设备注册表”。

    ![编辑设备元数据][img-editdevice2]  


4. 导航回到仪表板，地图上的设备位置已更改：

    ![编辑设备元数据][img-editdevice3]  


## 为新设备添加规则

刚才添加的新设备没有规则。在本部分中，将添加一个规则，以便在新设备报告的温度超过 47 度时触发警报。在开始之前，请注意仪表板上新设备的遥测历史记录显示设备温度绝不会超过 45 度。

1.  导航回到设备列表。

2.  在“设备列表”中选择新设备，然后单击“添加规则”来添加设备规则。

3. 创建一个规则，该规则使用“温度”作为数据字段，使用“AlarmTemp”作为温度超过 47 度时的输出：

    ![添加设备规则][img-adddevicerule]  


4. 单击“保存并查看规则”以保存更改。

5.  单击新设备的设备详细信息窗格中的“命令”。

    ![添加设备规则][img-adddevicerule2]

6.  从命令列表中选择“ChangeSetPointTemp”并将“SetPointTemp”设置为 45。然后单击“发送命令”：

    ![添加设备规则][img-adddevicerule3]

7.  导航回到解决方案仪表板。片刻之后，当新设备报告的温度超过 47 度阙值时，“警报历史记录”窗格中将显示新条目：

    ![添加设备规则][img-adddevicerule4]  


8. 可以在仪表板的“规则”页上查看和编辑所有规则：

    ![列出设备规则][img-rules]

9. 可以在仪表板的“操作”页上查看和编辑为了响应规则而可采取的所有操作：

    ![列出设备操作][img-actions]  



## 其他功能

可以使用解决方案门户搜索具有特定特征（例如型号）的设备：

![搜索设备][img-search]  


可以禁用设备，并且在已禁用之后删除：

![禁用并删除设备][img-disable]  


## 幕后

当你部署预配置解决方案时，部署过程会在你选择的 Azure 订阅中创建多个资源。你可以在 Azure [门户预览][lnk-portal]中查看这些资源。部署过程会创建一个**资源组**，其名称基于你为预配置解决方案选择的名称：

![Azure 门户中的预配置解决方案][img-portal]  


你可以查看每个资源的设置，方法是在资源组中的资源列表中选择该资源。

你也可以查看预配置解决方案的源代码。远程监视预配置解决方案源代码位于 [azure-iot-remote-monitoring][lnk-rmgithub] GitHub 存储库中：

- **DeviceAdministration** 文件夹包含仪表板的源代码。
- **Simulator** 文件夹包含模拟设备的源代码。
- **EventProcessor** 文件夹包含后端进程的源代码，可用于处理传入遥测。

完成后，可在 [azureiotsuite.cn][lnk-azureiotsuite] 站点的 Azure 订阅中删除预配置解决方案。通过该站点，可轻松删除创建预配置解决方案时预配的所有资源。

> [AZURE.NOTE] 若要确保删除与预配置解决方案相关的所有内容，请在 [azureiotsuite.cn][lnk-azureiotsuite] 站点中删除这些内容，而不只是删除门户中的资源组。

## 后续步骤

部署正常工作的预配置解决方案后，可以继续通过阅读以下文章开始使用 IoT 套件：

- [远程监视预配置解决方案演练][lnk-rm-walkthrough]
- [将设备连接到远程监视预配置解决方案][lnk-connect-rm]
- [azureiotsuite.cn 站点权限][lnk-permissions]

[img-launch-solution]: ./media/iot-suite-getstarted-preconfigured-solutions/launch.png
[img-dashboard]: ./media/iot-suite-getstarted-preconfigured-solutions/dashboard.png
[img-devicelist]: ./media/iot-suite-getstarted-preconfigured-solutions/devicelist.png
[img-devicedetails]: ./media/iot-suite-getstarted-preconfigured-solutions/devicedetails.png
[img-devicecommands]: ./media/iot-suite-getstarted-preconfigured-solutions/devicecommands.png
[img-pingcommand]: ./media/iot-suite-getstarted-preconfigured-solutions/pingcommand.png
[img-adddevice]: ./media/iot-suite-getstarted-preconfigured-solutions/adddevice.png
[img-addnew]: ./media/iot-suite-getstarted-preconfigured-solutions/addnew.png
[img-definedevice]: ./media/iot-suite-getstarted-preconfigured-solutions/definedevice.png
[img-runningnew]: ./media/iot-suite-getstarted-preconfigured-solutions/runningnew.png
[img-runningnew-2]: ./media/iot-suite-getstarted-preconfigured-solutions/runningnew2.png
[img-rules]: ./media/iot-suite-getstarted-preconfigured-solutions/rules.png
[img-editdevice]: ./media/iot-suite-getstarted-preconfigured-solutions/editdevice.png
[img-editdevice2]: ./media/iot-suite-getstarted-preconfigured-solutions/editdevice2.png
[img-editdevice3]: ./media/iot-suite-getstarted-preconfigured-solutions/editdevice3.png
[img-adddevicerule]: ./media/iot-suite-getstarted-preconfigured-solutions/addrule.png
[img-adddevicerule2]: ./media/iot-suite-getstarted-preconfigured-solutions/addrule2.png
[img-adddevicerule3]: ./media/iot-suite-getstarted-preconfigured-solutions/addrule3.png
[img-adddevicerule4]: ./media/iot-suite-getstarted-preconfigured-solutions/addrule4.png
[img-actions]: ./media/iot-suite-getstarted-preconfigured-solutions/actions.png
[img-portal]: ./media/iot-suite-getstarted-preconfigured-solutions/portal.png
[img-search]: ./media/iot-suite-getstarted-preconfigured-solutions/solutionportal_07.png
[img-disable]: ./media/iot-suite-getstarted-preconfigured-solutions/solutionportal_08.png
[1rmb-trial]: /pricing/1rmb-trial/
[lnk-preconfigured-solutions]: /documentation/articles/iot-suite-what-are-preconfigured-solutions/
[lnk-azureiotsuite]: https://www.azureiotsuite.cn
[lnk-logic-apps]: /documentation/services/app-service/logic/
[lnk-portal]: http://portal.azure.cn/
[lnk-rmgithub]: https://github.com/Azure/azure-iot-remote-monitoring
[lnk-devicemetadata]: /documentation/articles/iot-suite-what-are-preconfigured-solutions/#device-identity-registry-and-documentdb
[lnk-logicapptutorial]: /documentation/articles/iot-suite-logic-apps-tutorial/
[lnk-rm-walkthrough]: /documentation/articles/iot-suite-remote-monitoring-sample-walkthrough/
[lnk-connect-rm]: /documentation/articles/iot-suite-connecting-devices/
[lnk-permissions]: /documentation/articles/iot-suite-permissions/

<!---HONumber=Mooncake_0815_2016-->