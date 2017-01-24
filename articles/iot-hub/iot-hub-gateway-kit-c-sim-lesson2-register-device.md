<properties
    pageTitle="创建 Azure IoT 中心和注册设备 | Azure"
    description="服务：iot-hub"
    documentationcenter=""
    author="shizn"
    manager="timtl"
    tags=""
    keywords="azure iot 中心, 物联网云, azure iot 中心创建设备, ti sensortag, ti ble" />
<tags
    ms.assetid="23cfbe21-22c6-4fe1-ae41-63714a897f12"
    ms.service="iot-hub"
    ms.devlang="c"
    ms.topic="article"
    ms.tgt_pltfrm="na"
    ms.workload="na"
    ms.date="11/07/2016"
    wacn.date="01/23/2017"
    ms.author="xshi" />  


# 创建 Azure IoT 中心和注册设备

## 执行的操作

- 创建资源组
- 创建第一个 IoT 中心
- 使用 Azure CLI 在 IoT 中心注册设备。

在 IoT 中心注册设备时，Azure IoT 中心服务会生成设备密钥，用于向服务进行身份验证。

如果有问题，可在[故障排除页](/documentation/articles/iot-hub-gateway-kit-c-sim-troubleshooting/)上查找解决方案。

## 你要学习的知识

本课介绍以下内容：

- 如何使用 Azure CLI 创建 IoT 中心。
- 如何在 IoT 中心注册设备。

## 需要什么

- 一个有效的 Azure 订阅。如果没有 Azure 帐户，只需花费几分钟就能创建一个 [Azure 试用帐户](/pricing/1rmb-trial/)。
- 应该安装 Azure CLI。

## 创建 IoT 中心

若要创建 IoT 中心，请执行以下步骤：

1. 通过运行以下命令登录到 Azure 帐户：

   
		   az login
   

   成功登录后，会列出所有可用的订阅。

2. 运行以下命令，设置默认的需要使用的 Azure 订阅：

   
		   az account set --subscription {subscription id or name}
   

   可在 `az login` 或 `az account list` 命令的输出中找到 `subscription ID or name`。

3. 运行以下命令，注册提供程序。资源提供程序是指为应用程序提供资源的服务。必须先注册提供程序，然后才能部署该提供程序提供的 Azure 资源。

   
		   az provider register -n "Microsoft.Devices"
   

4. 运行以下命令，在“中国东部”区域创建名为 `iot-gateway` 的资源组：

   
		   az resource group create --name iot-gateway --location chinaeast
   
   
   `chinaeast` 是创建资源组所在的位置。如果想要使用其他位置，可运行 `az account list-locations -o table` 来查看 Azure 支持的所有位置。

5. 运行以下命令，在 `iot-gateway` 资源组中创建 IoT 中心：

   
		   az iot hub create --name {my hub name} --resource-group iot-gateway
   

默认情况下，该工具在免费定价层中创建 IoT 中心。有关详细信息，请参阅 [Azure IoT 中心定价](/pricing/details/iot-hub/)。

> [AZURE.NOTE]
IoT 中心的名称必须全局唯一。在 Azure 订阅下只能创建一个 F1 版的 Azure IoT 中心。

## 在 IoT 中心注册设备

每个将消息发送到 IoT 中心并从 IoT 中心接收消息的设备都必须使用唯一 ID 注册。运行以下命令，在 IoT 中心注册设备：


		az iot device create --device-id mydevice --hub-name {my hub name} --resource-group iot-gateway


## 摘要

已创建 IoT 中心并在 IoT 中心使用设备标识注册逻辑设备。已准备好学习如何配置和运行网关示例应用程序，将数据从物理设备发送到云中的 IoT 中心。

## 后续步骤
[配置和运行模拟设备云上传示例应用程序](/documentation/articles/iot-hub-gateway-kit-c-sim-lesson3-configure-simulated-device-app/)

<!---HONumber=Mooncake_0116_2017-->