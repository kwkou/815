<properties
	pageTitle="MyDriving Azure IoT 示例：快速入门 | Azure"
	description="开始使用一个应用，该应用全面演示如何通过使用 Azure（包括流分析、机器学习和事件中心）构建 IoT 系统。"
	services=""
    	documentationCenter=".net"
 	suite=""
	authors="harikmenon"
	manager="douge"/>  


<tags
	ms.service="iot-suite"
	ms.date="03/25/2016"
	wacn.date="08/22/2016"/>

# MyDriving IoT 系统：快速入门

MyDriving 系统用于演示典型[物联网](/documentation/articles/iot-suite-overview/) (IoT) 解决方案的设计和实现，物联网解决方案可从设备收集遥测、在云中处理收集的数据，并应用机器学习来提供自适应响应。通过使用你的移动电话中的数据和从你的汽车控制系统中收集信息的适配器中的数据，此演示记录有关你的汽车行程的数据。它使用此数据来提供有关你相较于其他用户的驾驶方式的反馈。

MyDriving 的真正目的是帮助你开始创建你自己的 IoT 解决方案。但在此之前，让我们帮助你以我们的测试用户小组成员的身份开始使用 MyDriving 应用。在你深入了解体系结构之前，这将使你以消费者的身份体验该应用及其背后的系统。它还会向你介绍 HockeyApp，一种用于对将你的应用的 alpha 和 beta 版分发给测试用户进行管理的绝佳方法。

## 使用移动体验

如果你有 Android、iOS 或 Windows 10 设备，则可以使用 MyDriving 应用。

### Android 和 Windows 10 移动版安装

在设备上：

1.  允许开发应用：

    -   Android：在“设置”>“安全”中，允许来自“未知源”的应用。

    -   Windows 10：在“设置”>“更新”>“面向开发人员”中，设置“开发人员模式”。

2.  通过注册或登录到[HockeyApp](https://rink.hockeyapp.net)即可加入我们的 beta 版测试小组。HockeyApp 使你可轻松地将应用的早期版本分发给测试用户。

    如果你在使用 Windows 10，请使用 Edge 浏览器。

    如果你是 Build 2016 的参与者，请通过一个 Microsoft，使用你用于注册会议的同一 Microsoft 帐户电子邮件登录。你已经注册了 HockeyApp。

    ![HockeyApp 登录屏幕](./media/iot-solution-get-started/image1.png)

3.  从此处下载并安装应用：

    -   [Android](http://rink.io/spMyDrivingAndroid)

    -   [Windows 10](http://rink.io/spMyDrivingUWP)

    有两个项目。在“受信任人”中安装证书。然后安装应用。

在 Windows 10 移动版上启动应用时遇到了问题？ 你的手机可能落后了一次或两次更新。请确保你已获得最新的更新，或安装：

 - [Microsoft.NET.Native.Framework.1.2.appx](https://download.hockeyapp.net/packages/win10/Microsoft.NET.Native.Framework.1.2.appx)

 - [Microsoft.NET.Native.Runtime.1.1.appx](https://download.hockeyapp.net/packages/win10/Microsoft.NET.Native.Runtime.1.1.appx)

 - [Microsoft.VCLibs.ARM.14.00.appx](https://download.hockeyapp.net/packages/win10/Microsoft.VCLibs.ARM.14.00.appx)


### iOS 安装

如果你未参与 Build 2016，请作为 HockeyApp 上我们的测试团队的成员下载应用：

1.  在你的 iOS 设备上登录到 [HockeyApp](https://rink.hockeyapp.net)。
    使用 Microsoft 登录按钮之一，并使用你注册会议时使用的同一 Microsoft 帐户电子邮件登录。（请勿使用电子邮件和密码字段。）

    ![HockeyApp 登录屏幕](./media/iot-solution-get-started/image1.png)

2.  在 HockeyApp 仪表板中，选择 MyDriving 并下载。

3.  从 HockeyApp 为 beta 版本授权：

    a.转到“设置”>“常规”>“配置文件和设备管理”。

    b.信任 **Bit Stadium GmbH** 证书。

如果未参与 Build 2016，你可以自己生成并部署应用：

1.   [从 GitHub] 下载代码。

2.   [使用 Xamarin] 进行生成和部署。

有关详细信息，请参阅 [MyDriving Reference Guide（MyDriving 参考指南）](http://aka.ms/mydrivingdocs)。

## 获取 OBD 适配器（可选）

这是使其成为真正的物联网系统的一部分！ 你可以使用不包含该适配器的应用，但使用真实的适配器将更有趣，而且价格并不贵。

车载诊断系统 (OBD) 是你的汽车的一项功能，汽车修理厂可使用此功能检修你的汽车并诊断奇怪噪声和警告灯。除非你的车年代非常久，否则你都讲在驾驶室的某个位置（通常在仪表板下翼片后面）找到一个插槽。使用正确的连接器，可以获取该引擎的性能指标并进行某些调整。OBD 连接器价格便宜，可以从常规的地点买到。它可以通过使用蓝牙或 Wi-Fi 连接到手机上的应用。

在这种情况下，我们会将你的汽车连接到云。OBD 可直接与你的手机连接，但我们的应用可作为中继。你的车载遥测会直接发送到 MyDriving IoT 中心，在这里会对遥测进行处理以记录你的公路行程并评估你的驾驶方式。

连接 OBD 设备：

1.  检查你的汽车是否有 OBD。

2.  获取 OBD 适配器：

    -   如果你正在使用 Android 或 Windows 手机，你将需要启用蓝牙的 OBD II 适配器。我们使用 [BAFX Products 34t5 Bluetooth OBDII Scan Tool]（BAFX Products 34t5 Bluetooth OBDII 扫描工具）。

    -   如果你使用的是 iOS 手机，则需要启用 Wi-Fi 的 OBD 适配器。我们使用 [ScanTool OBDLink MX Wi-Fi: OBD Adapter/Diagnostic Scanner]（ScanTool OBDLink MX Wi-Fi：OBD 适配器/诊断扫描仪）。

3.  按照随 OBD 适配器一起提供的说明将该适配器连接到你的手机。请记住以下几点：

    -   必须在“设置”页上将蓝牙适配器与手机配对。

    -   Wi-Fi 适配器必须具有 192.168.xxx.xxx 范围内的地址。

4.  如果你有多辆汽车，则可为每辆车获取一个单独的适配器（最多三个）。

如果你没有 OBD 适配器，该应用仍会从手机的 GPS 接收器将位置和速度数据发送到后端，并询问是否要模拟 OBD。

有关应用如何使用 OBD 适配器中的数据以及有关在 2.1 节“IoT 设备”中描述的用于创建你自己的 OBD 设备的选项的详细信息，请参阅 [MyDriving Reference Guide（MyDriving 参考指南）](http://aka.ms/mydrivingdocs)。

## 使用应用

启动应用。有一个初始快速入门可帮助你了解其工作原理。

### 跟踪你的行程

点击“记录”按钮（屏幕底部的红色大圆圈）可开始一段行程，再次点击即可结束。

![用于跟踪行程的“记录”按钮的示意图](./media/iot-solution-get-started/image2.png)

每次启动一段行程时，如果没有 OBD 设备，系统将会询问是否想要使用模拟器。

在一段行程结束时，点击“停止”按钮即刻获取摘要。

![行程摘要示例](./media/iot-solution-get-started/image3.png)

### 回顾你的行程

![过去的行程示例](./media/iot-solution-get-started/image4.png)

### 回顾你的配置文件

![驾驶方式配置文件示例](./media/iot-solution-get-started/image5.png)

## 向我们发送你的测试反馈

因为我们已经创建了 MyDriving 来帮助你快速开始创建自己的 IoT 系统，我们非常希望收到你对它的工作情况的反馈。如果出现以下情况，请告知我们：

- 遇到困难或难题。

- 存在更适合你的方案的扩展点。

- 你发现了完成某些需求的更高效方法。

- 你有改进 MyDriving 或此文档的任何其他建议。

在 MyDriving 应用内部，你可以使用内置的 HockeyApp 反馈机制：只需摇动你的 iOS 和 Android 手机或使用“反馈”菜单命令即可。这将自动附加屏幕截图，这样我们便能了解你想说明的内容。如果不幸出现了故障，HockeyApp 将收集故障日志并告知我们。你也可以通过 [HockeyApp 门户] 提供反馈。

你也可以在 [GitHub 上提出问题]，或者在下面留言（en-us 版）。

我们期待收到你的反馈！

## 后续步骤

-   浏览 [MyDriving Reference Guide（MyDriving 参考指南）](http://aka.ms/mydrivingdocs)，了解我们如何设计并生成整个 MyDriving 系统。

-   使用我们的 Azure Resource Manager 脚本[创建和部署你自己的系统](/documentation/articles/iot-solution-build-system/)。[MyDriving Reference Guide（MyDriving 参考指南）](http://aka.ms/mydrivingdocs)还将指导你了解你将进行最多自定义操作的区域。

  [从 GitHub]: https://github.com/Azure-Samples/MyDriving
  [使用 Xamarin]: https://developer.xamarin.com/guides/ios/getting_started/installation/
  [BAFX Products 34t5 Bluetooth OBDII Scan Tool]: http://www.amazon.com/gp/product/B005NLQAHS
  [ScanTool OBDLink MX Wi-Fi: OBD Adapter/Diagnostic Scanner]: http://www.amazon.com/gp/product/B00OCYXTYY/ref=s9_simh_gw_g263_i1_r?pf_rd_m=ATVPDKIKX0DER&pf_rd_s=desktop-2&pf_rd_r=1MWRMKXK4KK9VYMJ44MP
  [HockeyApp 门户]: https://rink.hockeyapp.org
  [GitHub 上提出问题]: https://github.com/Azure-Samples/MyDriving/issues

<!---HONumber=Mooncake_0815_2016-->