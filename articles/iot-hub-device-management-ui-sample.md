<properties
 pageTitle="使用 IoT 中心设备管理 UI |Azure"
 description="使用 Azure IoT 中心设备管理 UI 的演练"
 services="iot-hub"
 documentationCenter=""
 authors="dominicbetts"
 manager="timlt"
 editor=""/>

<tags
 ms.service="iot-hub"
 ms.date="06/08/2016"
 wacn.date="07/04/2016"/>

# 使用示例性 UI 探索 Azure IoT 中心设备管理

与示例性设备管理 UI 交互将有助于你巩固 Azure IoT 中心设备管理[概述][lnk-dm-overview]和[入门][lnk-get-started]文章中涵盖的概念和功能。本文将指导你完成所有三个主要设备管理概念 –“设备克隆”、“设备查询”和“设备作业”，如示例性设备管理 Web UI 中表示的那样。

那些寻求构建他们自己的设备管理交互式体验的开发人员可以分叉示例性 UI 代码库来用作自定义项目的基础。你可以在 [Azure IoT 设备管理 UI][lnk-dm-github] GitHub 存储库中查看完整项目代码和详细介绍其他开发工具功能的自述文档。

## 先决条件

开始本教程之前，应完成 [Azure IoT 中心设备管理入门][lnk-get-started]文章中的步骤。如果尚未执行，请返回并完成本文中的所有步骤，然后再继续。

已完成“入门”教程后，测试系统上会运行以下设备：

- 六个“iotdm\_simple\_sample”模拟设备（在控制台/终端窗口中运行），每个都显示有成功的“已注册”消息。

- 在 <http://127.0.0.1:3003> 构建和本地运行的设备管理示例性 UI 应用程序。

## 默认设备视图

设备管理示例性 UI 的默认主屏幕是“设备”视图，该视图包括以下 5 个组件：

![][1]

1.  导航按钮：“设备”视图（选定）、“作业历史记录”视图和“添加设备”视图。

2. 设备搜索：通过设备 ID 查找和编辑单个设备。

3.  设备操作：“编辑”操作、“删除”操作和“导出”操作。

4.  设备作业：“重新启动”设备、“固件更新”和“出厂重置”。

5.  设备网格筛选器：用于构建和保存设备网格的自定义视图的筛选器编辑器。

6.  设备网格：查看你向 IoT 中心实例注册的所有设备，该实例包括默认属性（“设备 ID”、“状态”和“标记”）。

[设备管理概述][lnk-dm-overview]引入了“设备克隆”概念，它表示 Azure IoT 中心中的物理（或模拟）设备。从设备网格可以选择任何已注册的设备，从设备列表可以查看和编辑该设备的设备克隆。

通过选择相应的设备行，然后单击“编辑”按钮（也可以双击该行或在搜索框中输入设备 ID），在第一台模拟设备“Device11 7ce4a850”中输入此详细视图。

你现在查看到的是设备克隆组件的完整表示形式，可在其中更新可写属性并运行如下所述的其他设备操作：

![][2]

1.  编辑设备克隆：这包括设备的“启用/禁用”切换。

2.  服务属性：这包括设备“标记”。

3.  设备属性：单击此项可展开该部分。

4.  “刷新”按钮：检索最新设备克隆属性和值。

若要继续，请单击此视图右下角的“取消”以返回到默认设备列表页面。

## 使用设备查询来筛选设备视图

设备查询是一种快速查找具有特定属性（如特定标记）的一个设备或一组设备的强大方法。示例性 UI 使用设备查询填充默认设备列表页面上列出的设备。示例性 UI 使你能够在其中某些属性上的表和筛选器中添加和删除服务属性。

执行以下步骤以创建“bacon”服务属性标记的客户筛选器：

1.  单击漏斗图标以打开设备查询编辑视图：

    ![][3]

2.  单击“+ 添加筛选器”以展开编辑器。从属性下拉列表中选择“标记”并在文本字段中输入“bacon”，然后单击“应用”。设备列表现在仅显示具有“bacon”标记的三个设备：

    ![][4]

3.  在查询标题字段中（漏斗图标旁边），将查询命名为“仅 Bacon”，然后单击“保存”：

    ![][5]

4.  单击漏斗图标可退出设备查询编辑器。

## 使用设备作业模拟设备重启 

你在设备管理概述中已了解到，设备作业可用于组织一个或多个物理设备上的简单或复杂操作。本节将介绍如何在示例性 UI 中创建设备作业，以在具有“bacon”标记的所有模拟设备上执行重启操作：

1.  在“仅 Bacon”设备查询列表中，单击每个设备行以对其进行标记，以便执行重启作业操作：

    ![][6]

2.  单击设备作业工具栏中的“重启”按钮以创建重启作业。在确认“是”后，单击生成的“设备作业已开始”对话框中的“作业历史记录”超链接，以导航到“设备作业”视图。

    ![][7]

现在你已创建了一个父作业，该父作业生成了三个子作业，每个子作业在三个具有“bacon”标记其中一个设备上执行重启操作：

![][8]

几分钟后刷新此屏幕会将父作业和三个子作业的状态更改为“已完成”，这表示重启操作已成功并通过模拟设备确认。使用“设备 ID”列确定哪些作业与哪些设备相关联。


> [AZURE.NOTE] 如果子作业返回“失败”状态，请检查模拟设备是否仍在你的测试系统上运行。如果没有，再次运行 simulate.bat 或 simulate.sh 脚本并重复上面的重启设备作业步骤。

## 后续步骤

你现在已使用示例性设备管理 UI 体验完成了设备管理概念的引导式探索。如果想要深入了解设备管理 API 并尝试一些代码示例，请访问以下开发人员教程：

- [如何使用设备克隆][lnk-tutorial-twin]
- [如何使用查询查找设备克隆][lnk-tutorial-queries]
- [如何使用设备作业更新设备固件][lnk-tutorial-jobs]
- [Azure IoT 中心设备管理客户端库介绍][lnk-library-c]

[1]: ./media/iot-hub-device-management-ui-sample/image1.png
[2]: ./media/iot-hub-device-management-ui-sample/image2.png
[3]: ./media/iot-hub-device-management-ui-sample/image3.png
[4]: ./media/iot-hub-device-management-ui-sample/image4.png
[5]: ./media/iot-hub-device-management-ui-sample/image5.png
[6]: ./media/iot-hub-device-management-ui-sample/image6.png
[7]: ./media/iot-hub-device-management-ui-sample/image7.png
[8]: ./media/iot-hub-device-management-ui-sample/image8.png

[lnk-dm-overview]: /documentation/articles/iot-hub-device-management-overview/
[lnk-get-started]: /documentation/articles/iot-hub-device-management-get-started/
[lnk-dm-github]: https://github.com/Azure/azure-iot-device-management/
[lnk-library-c]: /documentation/articles/iot-hub-device-management-library/
[lnk-tutorial-twin]: /documentation/articles/iot-hub-device-management-device-twin/
[lnk-tutorial-queries]: /documentation/articles/iot-hub-device-management-device-query/
[lnk-tutorial-jobs]: /documentation/articles/iot-hub-device-management-device-jobs/

<!---HONumber=Mooncake_0627_2016-->