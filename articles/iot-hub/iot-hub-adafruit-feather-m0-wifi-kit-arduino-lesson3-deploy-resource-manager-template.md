<properties
    pageTitle="创建 Azure Function App 和存储帐户 | Azure"
    description="Azure 函数应用可侦听 Azure IoT 中心事件、处理传入消息以及将其写入到 Azure 表存储。"
    services="iot-hub"
    documentationcenter=""
    author="shizn"
    manager="timtl"
    tags=""
    keywords="在云中存储数据, 云中存储的数据, iot 云服务"
    translationtype="Human Translation" />
<tags
    ms.assetid="9c8f4cd1-9511-4601-ad7e-51761a986753"
    ms.service="iot-hub"
    ms.devlang="arduino"
    ms.topic="article"
    ms.tgt_pltfrm="na"
    ms.workload="na"
    ms.date="11/13/2016"
    wacn.date="02/10/2017"
    ms.author="xshi" />  


# 创建 Azure 函数应用和 Azure 存储帐户
Azure Functions 是一种解决方案，特别适合在云中运行*函数*（小块代码）。Azure 函数应用在 Azure 中托管函数的执行。

## <a name="what-will-you-do"></a>执行的操作
使用 Azure Resource Manager 模板创建 Azure 函数应用和 Azure 存储帐户。 Azure 函数应用可侦听 Azure IoT 中心事件、处理传入消息以及将其写入到 Azure 表存储。

如果遇到任何问题，请在 [Adafruit Feather M0 WiFi Arduino 开发板的故障排除页](/documentation/articles/iot-hub-adafruit-feather-m0-wifi-kit-arduino-troubleshooting/)中查找解决方法。

## <a name="what-will-you-learn"></a>学习的内容
本文介绍：

 - 如何使用 [Azure Resource Manager](/documentation/articles/resource-group-overview/) 部署 Azure 资源。
 - 如何使用 Azure 函数应用处理 IoT 中心消息并将其写入到 Azure 表存储的表中。

## <a name="what-do-you-need"></a>所需条件
用户必须已成功完成：

 - [Arduino 开发板入门][get-started]
 - [创建 Azure IoT 中心][create-iot-hub]

## <a name="open-the-sample-app"></a>打开示例应用
通过运行以下命令在 Visual Studio Code 中打开示例项目：


	cd Lesson3
	code .


![存储库结构][repo-structure]  

* `app` 子文件夹中的 `app.ino` 文件是关键源文件。 此源文件包含的代码可将一条消息发送到 IoT 中心 20 次，并且在每次发送消息时使 LED 闪烁。
* `config.json` 包含所需的配置设置。
* `arm-template.json` 文件是 Azure Resource Manager 模板，其中包含一个 Azure 函数应用和一个 Azure 存储帐户。
* `arm-template-param.json` 文件是 Azure Resource Manager 模板使用的配置文件。
* `ReceiveDeviceMessages` 子文件夹包含用于 Azure 函数的 Node.js 代码。

## <a name="configure-azure-resource-manager-templates-and-create-resources-in-azure"></a>在 Azure 中配置 Azure Resource Manager 模板并创建资源
在 Visual Studio Code 中更新 `arm-template-param.json` 文件。

![Azure Resource Manager 模板参数][arm-template-params]  

* 将 **[your IoT Hub name]** 替换为在[创建 IoT 中心并注册 Arduino 开发板][created-iot-hub-and-registered-arduino-board] 时指定的 **{my hub name}**。
* 将 **[prefix string for new resources]** 替换为你需要的任何前缀。 前缀可以确保资源名称全局唯一以避免冲突。 请勿在前缀中以短划线或数字开头。

更新 `arm-template-param.json` 文件后，请运行以下命令，将资源部署到 Azure：


	az group deployment create --template-file arm-template.json --parameters @arm-template-param.json -g iot-sample


创建这些资源约需五分钟。在创建这些资源时，用户可以阅读下一篇文章。

## <a name="summary"></a>摘要
用户已创建 Azure 函数应用，因此可以处理 IoT 中心消息并通过 Azure 存储帐户存储这些消息。 现在可在 Arduino 开发板上部署和运行示例，发送设备到云消息。

## <a name="next-steps"></a>后续步骤
[在 Arduino 开发板上运行示例应用程序，发送设备到云消息][send-device-to-cloud-messages]

<!-- Images and links -->


[get-started]: /documentation/articles/iot-hub-adafruit-feather-m0-wifi-kit-arduino-get-started/
[create-iot-hub]: /documentation/articles/iot-hub-adafruit-feather-m0-wifi-kit-arduino-lesson2-prepare-azure-iot-hub/
[repo-structure]: ./media/iot-hub-adafruit-feather-m0-wifi-lessons/lesson3/repo_structure_c.png
[arm-template-params]: ./media/iot-hub-adafruit-feather-m0-wifi-lessons/lesson3/arm_para_arduino.png
[created-iot-hub-and-registered-arduino-board]: /documentation/articles/iot-hub-adafruit-feather-m0-wifi-kit-arduino-lesson2-prepare-azure-iot-hub/
[send-device-to-cloud-messages]: /documentation/articles/iot-hub-adafruit-feather-m0-wifi-kit-arduino-lesson3-run-azure-blink/


<!--Update_Description:update wording and code-->