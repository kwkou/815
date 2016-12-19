<properties
	title="运行示例应用程序，接收从云到设备的消息"
	description="示例应用程序在 Pi 上运行，用于监视来自 IoT 中心的传入消息。新的 gulp 任务会将消息从 IoT 中心发送到 Pi，使 LED 闪烁。"
	services="iot-hub"
	documentationcenter=""
	author="shizn"
	manager="timlt"
	tags=""
	keywords=""/>  


<tags
	ms.service: iot-hub"
	ms.date: 10/21/2016"
	wacn.date="12/19/2016"/>


# 运行示例应用程序，接收从云到设备的消息
在本文中，用户需在 Raspberry Pi 3 上部署示例应用程序。示例应用程序监视来自 IoT 中心的传入消息。此外还需在计算机上运行一项 gulp 任务，将消息从 IoT 中心发送到 Pi。示例应用程序在收到这些消息后，就会让 LED 闪烁。如果有问题，请在[故障排除页](/documentation/articles/iot-hub-raspberry-pi-kit-node-troubleshooting/)上查找解决方案。

## 执行的操作
* 将示例应用程序连接到 IoT 中心。
* 部署并运行示例应用程序。
* 将消息从 IoT 中心发送到 Pi，使 LED 闪烁。

## 你要学习的知识
* 如何监视来自 IoT 中心的传入消息
* 如何将从云到设备的消息从 IoT 中心发送到 Pi。

## 需要什么
* Raspberry Pi 3，已完成使用设置。若要了解如何设置 Pi，请参阅[配置设备](/documentation/articles/iot-hub-raspberry-pi-kit-node-lesson1-configure-your-device/)。
* 一个 IoT 中心，已在 Azure 订阅中创建。若要了解如何创建 IoT 中心，请参阅[创建 IoT 中心并注册 Raspberry Pi 3](/documentation/articles/iot-hub-raspberry-pi-kit-node-lesson2-prepare-azure-iot-hub/)。

## 将示例应用程序连接到 IoT 中心
1. 确保位于存储库文件夹 `iot-hub-node-raspberrypi-getting-started` 中。通过运行以下命令在 Visual Studio Code 中打开示例应用程序：
   
    ```bash
    cd Lesson4
    code .
    ```
   
    请注意 `app` 子文件夹中的 `app.js` 文件。`app.js` 文件是关键的源文件，其中包含的代码用于监视 IoT 中心发出的传入消息。`blinkLED` 函数可使 LED 闪烁。
   
    ![示例应用程序中的存储库结构](./media/iot-hub-raspberry-pi-lessons/lesson4/repo_structure.png)  

2. 使用以下命令初始化配置文件：
   
    ```bash
    npm install
    gulp init
    ```
   
    如果用户已在此计算机上完成[创建 Azure 函数应用和存储帐户](/documentation/articles/iot-hub-raspberry-pi-kit-node-lesson3-deploy-resource-manager-template/)中的步骤，继承了所有配置，则可跳到用于部署并运行示例应用程序的任务。如果用户在另一计算机上完成了[创建 Azure 函数应用和存储帐户](/documentation/articles/iot-hub-raspberry-pi-kit-node-lesson3-deploy-resource-manager-template/)中的步骤，则需替换 `config-raspberrypi.json` 文件中的占位符。`config-raspberrypi.json` 文件位于主文件夹的子文件夹中。
   
    ![config-raspberrypi.json 文件的内容](./media/iot-hub-raspberry-pi-lessons/lesson4/config_raspberrypi.png)  


* 将 **[设备主机名或 IP 地址]** 替换为通过运行 `devdisco list --eth` 命令获取的 Pi 的 IP 地址或主机名。
* 将 **[IoT 设备连接字符串]** 替换为通过运行 `az iot hub show-connection-string --name {my hub name} --resource-group {resource group name}` 命令获取的设备连接字符串。
* 将 **[IoT 中心连接字符串]** 替换为通过运行 `az iot device show-connection-string --hub {my hub name} --device-id {device id} --resource-group {resource group name}` 命令获取的 IoT 中心连接字符串。

## 部署并运行示例应用程序
运行以下命令，在 Pi 上部署并运行示例应用程序：

```
gulp
```

Gulp 命令首先运行 install-tools 任务，然后将示例应用程序部署到 Pi。最后，它会在 Pi 上运行该应用程序，并在主机上运行单独的任务，以便从 IoT 中心将 20 条闪烁消息发送到 Pi。

在示例应用程序运行以后，它会开始侦听 IoT 中心发出的消息。同时，gulp 任务会将多个“闪烁”消息从 IoT 中心发送到 Pi。Pi 每收到一条闪烁消息，示例应用程序都会调用 `blinkLED` 函数，使 LED 闪烁。

当 gulp 任务将 20 条消息从 IoT 中心发送到 Pi 时，用户会看到 LED 每隔两秒闪烁一次。最后一条为“停止”消息，告知应用程序停止运行。

![使用 gulp 命令和闪烁消息的示例应用程序](./media/iot-hub-raspberry-pi-lessons/lesson4/gulp_blink.png)  


## 摘要
用户已成功地将消息从 IoT 中心发送到 Pi，使 LED 闪烁。下一任务为可选任务：更改 LED 的开关行为。

## 后续步骤
[更改 LED 的开关行为](/documentation/articles/iot-hub-raspberry-pi-kit-node-lesson4-change-led-behavior/)

<!---HONumber=Mooncake_1212_2016-->