<properties
    pageTitle="创建 IoT 中心并注册 Raspberry Pi 3 | Azure"
    description="使用 Azure CLI 创建资源组、创建 Azure IoT 中心，以及在 Azure IoT 中心注册 Pi。"
    services="iot-hub"
    documentationcenter=""
    author="shizn"
    manager="timtl"
    tags=""
    keywords="raspberry pi 云, pi 云连接" />
<tags
    ms.assetid="4bcfb071-b3ae-48cc-8ea5-7e7434732287"
    ms.service="iot-hub"
    ms.devlang="c"
    ms.topic="article"
    ms.tgt_pltfrm="na"
    ms.workload="na"
    ms.date: 3/21/2017
    wacn.date="05/08/2017"
    ms.author="xshi" />  


# 创建 IoT 中心并注册 Raspberry Pi 3
## 执行的操作
* 创建资源组。
* 在资源组中创建 Azure IoT 中心。
* 使用 Azure 命令行接口 (Azure CLI) 将 Raspberry Pi 3 添加到 Azure IoT 中心。

使用 Azure CLI 将 Pi 添加到 IoT 中心时，服务会为 Pi 生成一个密钥，以便通过服务进行身份验证。如果有问题，可在[故障排除页](/documentation/articles/iot-hub-raspberry-pi-kit-c-troubleshooting/)上查找解决方案。

## 你要学习的知识
本文介绍：

 - 如何使用 Azure CLI 创建 IoT 中心。
 - 如何在 IoT 中心为 Pi 创建设备标识。

## 需要什么
* 一个 Azure 帐户
* 已安装 Azure CLI 的 Mac 或 Windows 计算机

## 创建 IoT 中心
Azure IoT 中心用于连接、监视并管理数百万 IoT 资产。若要创建 IoT 中心，请执行以下步骤：

1. 通过运行以下命令登录到 Azure 帐户：

   
		az login
   

    成功登录后，会列出所有可用的订阅。
    

    [AZURE.INCLUDE [azure-cli-2-azurechinacloud-environment-parameter](../../includes/azure-cli-2-azurechinacloud-environment-parameter.md)]

2. 运行以下命令，设置想要使用的默认订阅：

   
		az account set --subscription {subscription id or name}
   

    可在 `az login` 或 `az account list` 命令的输出中找到 `subscription ID or name`。

3. 运行以下命令，注册提供程序。资源提供程序是指为应用程序提供资源的服务。必须先注册提供程序，然后才能部署该提供程序提供的 Azure 资源。

   
		az provider register -n "Microsoft.Devices"
   
4. 运行以下命令，在“中国东部”区域创建名为 iot-sample 的资源组：

   
		az group create --name iot-sample --location chinaeast
   

    `chinaeast` 是创建资源组所在的位置。如果想要使用其他位置，可运行 `az account list-locations -o table` 来查看 Azure 支持的所有位置。
 
5. 运行以下命令，在 iot-sample 资源组中创建 IoT 中心：

   
		az iot hub create --name {my hub name} --resource-group iot-sample
   

    默认情况下，该工具在免费定价层中创建 IoT 中心。有关详细信息，请参阅 [Azure IoT 中心定价](/pricing/details/iot-hub/)。

> [AZURE.NOTE]
IoT 中心的名称必须全局唯一。在 Azure 订阅下只能创建一个 F1 版的 Azure IoT 中心。

## 在 IoT 中心注册 Pi
每个将消息发送到 IoT 中心并从 IoT 中心接收消息的设备都必须使用唯一 ID 注册。

运行以下命令，在中心注册 Pi：


	az iot device create --device-id myraspberrypi --hub {my hub name} --resource-group iot-sample


## 摘要
用户已创建 IoT 中心并在 IoT 中心中使用设备标识注册 Pi。用户已准备学习如何将消息从 Pi 发送到 IoT 中心。

## 后续步骤
[创建 Azure Function App 和 Azure 存储帐户，以便处理和存储 IoT 中心消息](/documentation/articles/iot-hub-raspberry-pi-kit-c-lesson3-deploy-resource-manager-template/)。

<!---HONumber=Mooncake_0206_2017-->
<!--Update_Description:update meta properties and code-->