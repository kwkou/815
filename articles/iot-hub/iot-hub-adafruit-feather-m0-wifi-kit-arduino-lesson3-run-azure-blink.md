<properties
    pageTitle="运行示例应用程序，将设备到云消息发送到 Azure IoT 中心 | Azure"
    description="在 Adafruit Feather M0 WiFi 上部署并运行将消息发送到 IoT 中心并使 LED 闪烁的示例应用程序。"
    services="iot-hub"
    documentationcenter=""
    author="shizn"
    manager="timtl"
    tags=""
    keywords="iot 云服务, arduino 向云发送数据" />
<tags
    ms.assetid="92cce319-2b17-4c9b-889d-deac959e3e7c"
    ms.service="iot-hub"
    ms.devlang="arduino"
    ms.topic="article"
    ms.tgt_pltfrm="na"
    ms.workload="na"
    ms.date="11/13/2016"
    wacn.date="01/23/2017"
    ms.author="xshi" />  


# 运行示例应用程序，以便发送从设备到云的消息
## 执行的操作
本文说明如何在 Adafruit Feather M0 WiFi Arduino 开发板上部署和运行示例应用程序，以便将消息发送到 IoT 中心。

如果有问题，可在[故障排除页][troubleshooting]上查找解决方案。

## 你要学习的知识
了解如何使用 gulp 工具在 Arduino 开发板上部署和运行示例 Arduino 应用程序。

## 需要什么
* 在开始此任务之前，用户必须已成功完成[创建 Azure 函数应用和存储帐户，以便处理和存储 IoT 中心消息][process-and-store-iot-hub-messages]。

## 获取 IoT 中心和设备连接字符串
设备连接字符串用于将 Arduino 开发板连接到 IoT 中心。IoT 中心连接字符串用于将 IoT 中心连接到在 IoT 中心代表 Arduino 开发板的设备标识。

* 运行以下 Azure CLI 命令，列出资源组中的所有 IoT 中心：


		az iot hub list -g iot-sample --query [].name


使用 `iot-sample` 作为 `{resource group name}` 的值（如果尚未更改此值）。

* 运行以下 Azure CLI 命令，获取 IoT 中心连接字符串：


		az iot hub show-connection-string --name {my hub name}


`{my hub name}` 是在创建 IoT 中心并注册 Arduino 开发板时指定的名称。

* 运行以下命令，获取设备连接字符串：


		az iot device show-connection-string --hub-name {my hub name} --device-id mym0wifi


使用 `mym0wifi` 作为 `{device id}` 的值（如果尚未更改此值）。
## 配置设备连接
若要配置设备连接，请执行以下步骤：

1. 使用设备发现 CLI 获取设备的串行端口：

   
		devdisco list --usb
   

   应看到类似如下的输出，并找到 Arduino 开发板的 USB COM 端口：

   ![设备发现][device-discovery]  


2. 打开课程文件夹中的 `config.json` 文件，并添加找到的 COM 端口号的值：

   
		   {
		       "device_port" : "COM1"
		   }
   

   ![config.json][config-json]  


   > [AZURE.NOTE]
   在 Windows 平台上，COM 端口的格式为：`COM1, COM2, ...`。在 macOS 或 Ubuntu 上，其以 `/dev/` 开头。

3. 运行以下命令初始化配置文件：

   
		   npm install
		   gulp init
		   gulp install-tools
   
4. 运行以下命令，在 Visual Studio Code 中打开设备配置文件 `config-arduino.json`：

   
		   # For Windows command prompt
		   code %USERPROFILE%\.iot-hub-getting-started\config-arduino.json

		   # For MacOS or Ubuntu
		   code ~/.iot-hub-getting-started/config-arduino.json
   

   ![config-arduino.json][config-arduino-json]  


5. 在 `config-arduino.json` 文件中进行以下替换：

   * 将 **[Wi-Fi SSID]** 替换为连接到 Internet 的 Wi-Fi SSID。
   * 将 **[Wi-Fi 密码]** 替换为你的 Wi-Fi 密码。如果 Wi-Fi 不需要密码，请删除字符串。
   * 将 **[IoT 设备连接字符串]** 替换为获得的 `device connection string`。
   * 将 **[IoT 中心连接字符串]** 替换为获得的 `iot hub connection string`。

   > [AZURE.NOTE]
   本文中不需要 `azure_storage_connection_string`。请保留该名称。

## 部署并运行示例应用程序
运行以下命令，在 Arduino 开发板上部署并运行示例应用程序：


		gulp run
		# You can monitor the serial port by running listen task:
		gulp listen

		# Or you can combine above two gulp tasks into one:
		gulp run --listen


> [AZURE.NOTE]
默认 gulp 任务按顺序运行 `install-tools` 和 `run` 任务。[部署 blink 应用][deployed-the-blink-app]时，是分别运行这些任务的。

## 验证示例应用程序是否正常运行
应看到 GPIO #0 板载 LED 每两秒闪烁一次。每次 LED 闪烁时，示例应用程序都会将消息发送到 IoT 中心，并验证该消息是否已成功发送到 IoT 中心。此外，IoT 中心收到的每条消息都会在控制台窗口输出。示例应用程序发送 20 条消息后会自动终止。

![包含已发送和已接收消息的示例应用程序][sample-application-with-sent-and-received-messages]  


## 摘要
已在 Arduino 开发板上部署和运行新的 blink 示例应用程序，将设备到云消息发送到 IoT 中心。现可在将消息写入存储帐户时对其进行监视。

## 后续步骤
[读取保存在 Azure 存储中的消息][read-messages-persisted-in-azure-storage]
<!-- Images and links -->


[troubleshooting]: /documentation/articles/iot-hub-adafruit-feather-m0-wifi-kit-arduino-troubleshooting/
[process-and-store-iot-hub-messages]: /documentation/articles/iot-hub-adafruit-feather-m0-wifi-kit-arduino-lesson3-deploy-resource-manager-template/
[device-discovery]: ./media/iot-hub-adafruit-feather-m0-wifi-lessons/lesson1/device_discovery.png
[config-json]: ./media/iot-hub-adafruit-feather-m0-wifi-lessons/lesson1/vscode-config-mac.png
[config-arduino-json]: ./media/iot-hub-adafruit-feather-m0-wifi-lessons/lesson3/config-arduino.png
[deployed-the-blink-app]: /documentation/articles/iot-hub-adafruit-feather-m0-wifi-kit-arduino-lesson1-deploy-blink-app/
[sample-application-with-sent-and-received-messages]: ./media/iot-hub-adafruit-feather-m0-wifi-lessons/lesson3/gulp_run_arduino.png
[read-messages-persisted-in-azure-storage]: /documentation/articles/iot-hub-adafruit-feather-m0-wifi-kit-arduino-lesson3-read-table-storage/

<!---HONumber=Mooncake_0116_2017-->