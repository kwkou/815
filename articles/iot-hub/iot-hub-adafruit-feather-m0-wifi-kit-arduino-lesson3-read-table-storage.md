<properties
    pageTitle="读取保存在 Azure 存储中的消息 | Azure"
    description="在将从设备到云的消息写入 Azure 表存储时，对其进行监视。"
    services="iot-hub"
    documentationcenter=""
    author="shizn"
    manager="timtl"
    tags=""
    keywords="云中的数据, 云数据收集, iot 云服务, iot 数据"
    translationtype="Human Translation" />
<tags
    ms.assetid="386083e0-0dbb-48c0-9ac2-4f8fb4590772"
    ms.service="iot-hub"
    ms.devlang="arduino"
    ms.topic="article"
    ms.tgt_pltfrm="na"
    ms.workload="na"
    ms.date="11/13/2016"
    wacn.date="05/08/2017"
    ms.author="xshi"
    ms.sourcegitcommit="2c4ee90387d280f15b2f2ed656f7d4862ad80901"
    ms.openlocfilehash="9b9159a410164b0be46db5342cd847806f9bf225"
    ms.lasthandoff="04/28/2017" />

# <a name="read-messages-persisted-in-azure-storage"></a>读取保存在 Azure 存储中的消息
## <a name="what-you-will-do"></a>执行的操作
对于从 Adafruit Feather M0 WiFi Arduino 开发板发送到 IoT 中心的设备到云消息，在将其写入 Azure 表存储时对其进行监视。

如果有问题，可在[故障排除页][troubleshooting]上查找解决方案。

## <a name="what-you-will-learn"></a>你要学习的知识
在本文中，用户将学习如何通过 gulp 的 read-message 任务读取保存在 Azure 表存储中的消息。

## <a name="what-you-need"></a>需要什么
在开始此过程之前，必须已成功完成 [在 Arduino 开发板上运行 Azure blink 示例应用程序][run-blink-application]。

## <a name="read-new-messages-from-your-storage-account"></a>从存储帐户中读取新消息
在之前的文章中，已在 Arduino 开发板上运行示例应用程序。 示例应用程序发送消息到 Azure IoT 中心。 发送到 IoT 中心的消息通过 Azure 函数应用存储在 Azure 表存储中。 需要使用 Azure 存储连接字符串读取 Azure 表存储中的消息。

若要读取存储在 Azure 表存储中的消息，请执行以下步骤：

1. 运行以下命令，获取连接字符串：

   
		   az storage account list -g iot-sample --query [].name
		   az storage account show-connection-string -g iot-sample -n {storage name}
   

    第一个命令检索第二个命令中使用的 `storage name` 来获取连接字符串。 使用 `iot-sample` 作为 `{resource group name}` 的值（如果未更改该值）。
    
2. 通过运行以下命令在 Visual Studio Code 中打开配置文件 `config-arduino.json`：

   
		   # For Windows command prompt
		   code %USERPROFILE%\.iot-hub-getting-started\config-arduino.json

		   # For MacOS or Ubuntu
		   code ~/.iot-hub-getting-started/config-arduino.json
   
3. 将 `[Azure storage connection string]` 替换为在步骤 1 中获取的连接字符串。
4. 保存 `config-arduino.json` 文件。
5. 运行以下命令，再次发送消息并从 Azure 表存储中读取这些消息：

   
		   gulp run --read-storage

		   # You can monitor the serial port by running listen task:
		   gulp listen

		   # Or you can combine above two gulp tasks into one:
		   gulp run --read-storage --listen
   

    从 Azure 表存储进行读取的逻辑位于 `azure-table.js` 文件中。

    ![gulp run --read-storage][gulp-run]  

## <a name="summary"></a>摘要
已成功将 Arduino 开发板连接到云中的 IoT 中心，并已使用 blink 示例应用程序发送设备到云消息。 用户还使用 Azure 函数应用将传入的 IoT 中心消息存储到 Azure 表存储中。 现在可以将云到设备消息从 IoT 中心发送到 Arduino 开发板。

## <a name="next-steps"></a>后续步骤
[发送云到设备的消息][send-cloud-to-device-messages]
<!-- Images and links -->


[troubleshooting]: /documentation/articles/iot-hub-adafruit-feather-m0-wifi-kit-arduino-troubleshooting/
[run-blink-application]: /documentation/articles/iot-hub-adafruit-feather-m0-wifi-kit-arduino-lesson3-run-azure-blink/
[gulp-run]: ./media/iot-hub-adafruit-feather-m0-wifi-lessons/lesson3/gulp_read_message_arduino.png
[send-cloud-to-device-messages]: /documentation/articles/iot-hub-adafruit-feather-m0-wifi-kit-arduino-lesson4-send-cloud-to-device-messages/

<!---HONumber=Mooncake_0116_2017-->