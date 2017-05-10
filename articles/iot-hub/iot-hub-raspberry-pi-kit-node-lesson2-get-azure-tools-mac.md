<properties
    pageTitle="获取 Azure 工具 (macOS 10.10) | Azure"
    description="在 macOS 上安装 Python 和 Azure 命令行接口 (Azure CLI)。"
    services="iot-hub"
    documentationcenter=""
    author="shizn"
    manager="timlt"
    tags=""
    keywords="iot 云服务, azure cli" />
<tags
    ms.assetid="1814b703-2d81-45db-aff0-eb338c97f120"
    ms.service="iot-hub"
    ms.devlang="node"
    ms.topic="article"
    ms.tgt_pltfrm="na"
    ms.workload="na"
    ms.date="3/21/2017"
    wacn.date="05/08/2017"
    ms.author="xshi" />  


# 获取 Azure 工具 \(macOS 10.10\)
> [AZURE.SELECTOR]
- [Windows 7 及更高版本](/documentation/articles/iot-hub-raspberry-pi-kit-node-lesson2-get-azure-tools-win32/)
- [Ubuntu 16.04](/documentation/articles/iot-hub-raspberry-pi-kit-node-lesson2-get-azure-tools-ubuntu/)
- [macOS 10.10](/documentation/articles/iot-hub-raspberry-pi-kit-node-lesson2-get-azure-tools-mac/)

## 执行的操作
安装 Azure 命令行接口 \(Azure CLI\)。如果有问题，可在[故障排除页](/documentation/articles/iot-hub-raspberry-pi-kit-node-troubleshooting/)上查找解决方案。

## 你要学习的知识
本文介绍：
* 如何安装 Azure CLI。
* 如何添加 Azure CLI 的 IoT 子组。

## 需要什么
* 启用 Internet 连接的 Mac。
* 一个有效的 Azure 订阅。如果没有 Azure 帐户，只需几分钟即可创建一个[帐户](/pricing/1rmb-trial/)。

## 安装 Python
虽然 macOS 随附现成的 Python 2.7，但建议通过 Homebrew 安装 Python。请参阅 [Installing Python on macOS](http://docs.python-guide.org/en/latest/starting/install/osx/)（在 macOS 上安装 Python）。

通过运行以下命令安装 Python 和 pip：


        brew install python


## 安装 Azure CLI
Azure CLI 提供适用于 Azure 的多平台命令行体验。可以直接通过命令行预配和管理资源。

若要安装最新 Azure CLI，请执行以下步骤：

1. 在终端窗口运行以下命令。安装 Azure CLI 可能需要五分钟。
   

	   pip install --upgrade azure-cli
	   pip install --upgrade azure-cli-iot
2. 运行以下命令，对安装进行验证：
   

        az iot -h


如果安装成功，则会看到以下输出。

![指示成功的输出](./media/iot-hub-raspberry-pi-lessons/lesson2/az_iot_help_osx.png)  


## 摘要
已安装 Azure CLI。下一任务是使用 Azure CLI 创建 Azure IoT 中心和设备标识。

## 后续步骤
[创建 IoT 中心并注册 Raspberry Pi 3](/documentation/articles/iot-hub-raspberry-pi-kit-node-lesson2-prepare-azure-iot-hub/)

<!---HONumber=Mooncake_0109_2017-->
<!--Update_Description:update wording and code-->