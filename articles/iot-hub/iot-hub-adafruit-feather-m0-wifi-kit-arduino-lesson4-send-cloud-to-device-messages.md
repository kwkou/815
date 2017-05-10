<properties
    pageTitle="运行示例应用程序，接收来自 Azure IoT 中心的云到设备消息 | Azure"
    description="示例应用程序在 Adafruit Feather M0 WiFi 上运行，并监视来自 IoT 中心的传入消息。新的 gulp 任务将消息从 IoT 中心发送到 Adafruit Feather M0 WiFi，使 LED 闪烁。"
    services="iot-hub"
    documentationcenter=""
    author="shizn"
    manager="timtl"
    tags=""
    keywords="arduino 从 web 控制 led, arduino 通过 web 控制 led" />
<tags
    ms.assetid="a0bf53fb-29fb-485f-ba4a-6c715057b1a2"
    ms.service="iot-hub"
    ms.devlang="arduino"
    ms.topic="article"
    ms.tgt_pltfrm="na"
    ms.workload="na"
    ms.date="3/21/2017"
    wacn.date="05/08/2017"
    ms.author="xshi" />  


# 运行示例应用程序，接收云到设备消息
在本文中，需在 Adafruit Feather M0 WiFi Arduino 开发板上部署示例应用程序。

示例应用程序监视来自 IoT 中心的传入消息。此外还需在计算机上运行 gulp 任务，将消息从 IoT 中心发送到 Arduino 开发板。示例应用程序在收到这些消息后，就会让 LED 闪烁。如果有问题，可在[故障排除页][troubleshooting]上查找解决方案。

## 执行的操作
* 将示例应用程序连接到 IoT 中心。
* 部署并运行示例应用程序。
* 将消息从 IoT 中心发送到 Arduino 开发板，使 LED 闪烁。

## 你要学习的知识
本文介绍：

 - 如何监视来自 IoT 中心的传入消息。
 - 如何将云到设备消息从 IoT 中心发送到 Arduino 开发板。

## 需要什么

 - 设置 Arduino 开发板以供使用。若要了解如何设置 Arduino 开发板，请参阅[配置设备][configure-your-device]。
 - 一个 IoT 中心，已在 Azure 订阅中创建。若要了解如何创建 IoT 中心，请参阅[创建 Azure IoT 中心][create-your-azure-iot-hub]。

## 将示例应用程序连接到 IoT 中心

1. 确保位于存储库文件夹 `iot-hub-c-feather-m0-getting-started` 中。

    通过运行以下命令在 Visual Studio Code 中打开示例应用程序：

   
		cd Lesson4
		code .
   

    请注意 `app` 子文件夹中的 `app.ino` 文件。`app.ino` 文件是关键的源文件，其中包含的代码用于监视 IoT 中心发出的传入消息。`blinkLED` 函数可使 LED 闪烁。

    ![示例应用程序中的存储库结构][repo-structure]  


2. 使用设备发现 CLI 获取设备的串行端口：

   
		devdisco list --usb
   

    应看到类似如下的输出，并找到 Arduino 开发板的 USB COM 端口：

    ![设备发现][device-discovery]  


3. 打开课程文件夹中的 `config.json` 文件，并输入找到的 COM 端口号的值：

		{
		    "device_port" : "COM1"
		}
   

    ![config.json][config-json]  


    > [AZURE.NOTE]
    > 在 Windows 平台上，COM 端口的格式为：`COM1, COM2, ...`。在 macOS 或 Ubuntu 上，其以 `/dev/` 开头。

4. 运行以下命令初始化配置文件：

   
		# For Windows command prompt
		npm install
		gulp init
		gulp install-tools
   

5. 在 `config-arduino.json` 文件中进行以下替换：

    如果已在此计算机上完成[创建 Azure Function App 和存储帐户][create-an-azure-function-app-and-storage-account]中的步骤，并继承了所有配置，则可跳过该步骤，转到部署并运行示例应用程序的任务。如果用户在另一计算机上完成了[创建 Azure 函数应用和存储帐户][create-an-azure-function-app-and-storage-account]中的步骤，则需替换 `config-arduino.json` 文件中的占位符。`config-arduino.json` 文件位于主文件夹的子文件夹中。

    ![config-arduino.json 文件的内容][config-arduino-json]  


   * 将 **[Wi-Fi SSID]** 替换为连接到 Internet 的 Wi-Fi SSID。
   * 将 **[Wi-Fi 密码]** 替换为你的 Wi-Fi 密码。如果 Wi-Fi 不需要密码，请删除字符串。
   * 将 **[IoT 设备连接字符串]** 替换为通过运行 `az iot device show-connection-string --hub-name {my hub name} --device-id {device id}` 命令获取的设备连接字符串。
   * 将 **[IoT 中心连接字符串]** 替换为通过运行 `az iot hub show-connection-string --name {my hub name}` 命令获取的 IoT 中心连接字符串。

## 部署并运行示例应用程序
运行以下命令，在 Arduino 开发板上部署并运行示例应用程序：


		gulp run
		# You can monitor the serial port by running listen task:
		gulp listen

		# Or you can combine above two gulp tasks into one:
		gulp run --listen


gulp 命令将示例应用程序部署到 Arduino 开发板。然后，它会在 Arduino 开发板上运行该应用程序，并在主机上运行单独的任务，将 20 条闪烁消息从 IoT 中心发送到 Arduino 开发板。

在示例应用程序运行以后，它会开始侦听 IoT 中心发出的消息。同时，gulp 任务会将多个“闪烁”消息从 IoT 中心发送到 Arduino 开发板。开发板每收到一条闪烁消息，示例应用程序都会调用 `blinkLED` 函数，使 LED 闪烁。

当 gulp 任务将 20 条消息从 IoT 中心发送到 Arduino 开发板时，应看到 LED 每隔两秒闪烁一次。最后一条为“停止”消息，会停止示例应用程序的运行。

![使用 gulp 命令和闪烁消息的示例应用程序][sample-application]  


## 摘要
已成功将消息从 IoT 中心发送到 Arduino 开发板，使 LED 闪烁。下一任务为可选任务：更改 LED 的开关行为。

## 后续步骤
[更改 LED 的开关行为][change-the-on-and-off-led-behavior]


<!-- Images and links -->


[troubleshooting]: /documentation/articles/iot-hub-adafruit-feather-m0-wifi-kit-arduino-troubleshooting/
[configure-your-device]: /documentation/articles/iot-hub-adafruit-feather-m0-wifi-kit-arduino-lesson1-configure-your-device/
[create-your-azure-iot-hub]: /documentation/articles/iot-hub-adafruit-feather-m0-wifi-kit-arduino-lesson2-prepare-azure-iot-hub/
[repo-structure]: ./media/iot-hub-adafruit-feather-m0-wifi-lessons/lesson4/repo_structure_arduino.png
[device-discovery]: ./media/iot-hub-adafruit-feather-m0-wifi-lessons/lesson1/device_discovery.png
[config-json]: ./media/iot-hub-adafruit-feather-m0-wifi-lessons/lesson1/vscode-config-mac.png
[create-an-azure-function-app-and-storage-account]: /documentation/articles/iot-hub-adafruit-feather-m0-wifi-kit-arduino-lesson3-deploy-resource-manager-template/
[config-arduino-json]: ./media/iot-hub-adafruit-feather-m0-wifi-lessons/lesson4/config-arduino.png
[sample-application]: ./media/iot-hub-adafruit-feather-m0-wifi-lessons/lesson4/gulp_blink_arduino.png
[change-the-on-and-off-led-behavior]: /documentation/articles/iot-hub-adafruit-feather-m0-wifi-kit-arduino-lesson4-change-led-behavior/

<!---HONumber=Mooncake_0116_2017-->