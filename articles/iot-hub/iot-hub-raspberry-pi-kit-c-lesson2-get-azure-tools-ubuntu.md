<properties
    pageTitle="获取 Azure 工具 (Ubuntu 16.04) | Azure"
    description="在 Ubuntu 上安装 Python 和 Azure 命令行接口 (Azure CLI)。"
    services="iot-hub"
    documentationcenter=""
    author="shizn"
    manager="timtl"
    tags=""
    keywords="iot 云服务, azure cli" />
<tags
    ms.assetid="a03512f2-fabe-40c5-8505-b93eef8e5bec"
    ms.service="iot-hub"
    ms.devlang="c"
    ms.topic="article"
    ms.tgt_pltfrm="na"
    ms.workload="na"
    ms.date="11/28/2016"
    wacn.date="01/06/2017"
    ms.author="xshi" />  


# 获取 Azure 工具 (Ubuntu 16.04)
>[AZURE.SELECTOR]
[Windows 7 and later](/documentation/articles/iot-hub-raspberry-pi-kit-c-lesson2-get-azure-tools-win32/)
[Ubuntu 16.04](/documentation/articles/iot-hub-raspberry-pi-kit-c-lesson2-get-azure-tools-ubuntu/)
[macOS 10.10](/documentation/articles/iot-hub-raspberry-pi-kit-c-lesson2-get-azure-tools-mac/)

## 执行的操作
安装 Azure 命令行接口 (Azure CLI)。如果有问题，可在[故障排除页](/documentation/articles/iot-hub-raspberry-pi-kit-c-troubleshooting/)上查找解决方案。

## 你要学习的知识
本文介绍：
* 如何安装 Azure CLI。
* 如何添加 Azure CLI 的 IoT 子组。

## 需要什么
* 启用 Internet 连接的 Ubuntu 计算机。
* 一个有效的 Azure 订阅。如果没有帐户，只需花费几分钟就能创建一个[试用帐户](/pricing/1rmb-trial/)。

## 安装 Azure CLI
Azure CLI 提供适用于 Azure 的多平台命令行体验，让用户能够直接通过命令行完成资源的预配和管理。

若要安装最新 Azure CLI，请执行以下步骤：

1. 在终端窗口运行以下命令。安装 Azure CLI 可能需要五分钟。

   
		   sudo apt-get update
		   sudo apt-get install -y libssl-dev libffi-dev
		   sudo apt-get install -y python-dev
		   sudo apt-get install -y build-essential
		   sudo apt-get install -y python-pip
		   sudo pip install --upgrade azure-cli
		   sudo pip install --upgrade azure-cli-iot
   
2. 运行以下命令，对安装进行验证：

   
		   az iot -h
   

如果安装成功，则会看到以下输出。

![指示成功的输出](./media/iot-hub-raspberry-pi-lessons/lesson2/az_iot_help_ubuntu.png)  


## 摘要
已安装 Azure CLI。下一任务是使用 Azure CLI 创建 Azure IoT 中心和设备标识。

## 后续步骤
[创建 IoT 中心并注册 Raspberry Pi 3](/documentation/articles/iot-hub-raspberry-pi-kit-c-lesson2-prepare-azure-iot-hub/)

<!---HONumber=Mooncake_0103_2017-->