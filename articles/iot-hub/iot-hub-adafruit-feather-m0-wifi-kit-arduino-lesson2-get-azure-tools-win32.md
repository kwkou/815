<properties
    pageTitle="获取用于 Azure IoT 初学者工具包（Windows 7 和更高版本）的 Azure 工具 | Azure"
    description="在 Windows 7 及更高版本上安装 Python 和 Azure 命令行接口 (Azure CLI)。"
    services="iot-hub"
    documentationcenter=""
    author="shizn"
    manager="timtl"
    tags=""
    keywords="azure cli, iot 云服务, arduino 云" />
<tags
    ms.assetid="70dfff14-4be1-468d-9919-e40f4bead308"
    ms.service="iot-hub"
    ms.devlang="arduino"
    ms.topic="article"
    ms.tgt_pltfrm="na"
    ms.workload="na"
    ms.date="11/13/2016"
    wacn.date="01/23/2017"
    ms.author="xshi" />  


# 获取 Azure 工具（Windows 7 及更高版本）

> [!div class="op_single_selector"]
> * [Windows 7 或更高版本][windows]
> * [Ubuntu 16.04][ubuntu]
> * [macOS 10.10][macos]

## 执行的操作

安装 Python 和 Azure 命令行接口 (Azure CLI)。如果有任何问题，请在 Adafruit Feather M0 WiFi Arduino 开发板的[故障排除页](/documentation/articles/iot-hub-adafruit-feather-m0-wifi-kit-arduino-troubleshooting/)查找解决方案。

## 你要学习的知识
本文介绍：
* 如何安装 Python。
* 如何安装 Azure CLI。

## 需要什么
* 启用 Internet 连接的 Windows 计算机。
* 一个有效的 Azure 订阅。如果没有 Azure 帐户，只需花费几分钟就能创建一个 [Azure 试用帐户](/pricing/1rmb-trial/)。

## 安装 Python
在 Windows 计算机上[安装 Python](https://www.python.org/downloads/)。可安装 Python 2.7、3.4 或 3.5。本教程基于 Python 2.7。如果已安装 Python，请转到下一部分，然后安装 Azure CLI。

此外还需添加文件夹的路径，以便通过该路径将 python.exe 和 pip.exe 安装到系统的 `PATH` 环境变量中。默认情况下，python.exe 安装在 `C:\Python27` 中，pip.exe 安装在 `C:\Python27\Scripts` 中。

## 安装 Azure CLI
Azure CLI 提供适用于 Azure 的多平台命令行体验。可以直接通过命令行预配和管理资源。

若要安装 Azure CLI，请执行以下步骤：

1. 以管理员身份打开“命令提示符”窗口。
2. 运行以下命令：

   
		   pip install --upgrade azure-cli
		   pip install --upgrade azure-cli-iot
   
3. 运行以下命令，对安装进行验证：

   
		az iot -h
   

如果安装成功，则会看到以下输出。

![指示成功的输出][output]  


## 摘要
已安装 Azure CLI。下一任务是使用 Azure CLI 创建 Azure IoT 中心和设备标识。

## 后续步骤
[创建 IoT 中心并注册 Arduino 开发板][create-your-iot-hub-and-register-your-arduino-board]


<!-- Images and links -->


[windows]: /documentation/articles/iot-hub-adafruit-feather-m0-wifi-kit-arduino-lesson2-get-azure-tools-win32/
[ubuntu]: /documentation/articles/iot-hub-adafruit-feather-m0-wifi-kit-arduino-lesson2-get-azure-tools-ubuntu/
[macos]: /documentation/articles/iot-hub-adafruit-feather-m0-wifi-kit-arduino-lesson2-get-azure-tools-mac/
[output]: ./media/iot-hub-adafruit-feather-m0-wifi-lessons/lesson2/az_iot_help_win.png
[create-your-iot-hub-and-register-your-arduino-board]: /documentation/articles/iot-hub-adafruit-feather-m0-wifi-kit-arduino-lesson2-prepare-azure-iot-hub/

<!---HONumber=Mooncake_0116_2017-->