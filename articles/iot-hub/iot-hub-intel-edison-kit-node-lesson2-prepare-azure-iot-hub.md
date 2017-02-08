<properties
    pageTitle="创建 Azure IoT 中心并注册 Intel Edison | Azure"
    description="使用 Azure CLI 创建资源组、创建 Azure IoT 中心，以及在 Azure IoT 中心注册 Edison。"
    services="iot-hub"
    documentationcenter=""
    author="shizn"
    manager="timtl"
    tags=""
    keywords="" />
<tags
    ms.assetid="c1465cc2-d0d9-4326-967c-64d95b61dd54"
    ms.service="iot-hub"
    ms.devlang="nodejs"
    ms.topic="article"
    ms.tgt_pltfrm="na"
    ms.workload="na"
    ms.date="11/8/2016"
    wacn.date="01/06/2017"
    ms.author="xshi" />  


# 创建 IoT 中心并注册 Intel Edison
## 执行的操作
* 创建资源组。
* 在资源组中创建 Azure IoT 中心。
* 使用 Azure 命令行接口 (Azure CLI) 将 Intel Edison 添加到 Azure IoT 中心。

使用 Azure CLI 将 Edison 添加到 IoT 中心时，服务会为 Edison 生成一个密钥，用于向服务进行身份验证。如果有问题，可在[故障排除页][troubleshooting]上查找解决方案。

## 你要学习的知识
本文介绍：
 - 如何使用 Azure CLI 创建 IoT 中心。
 - 如何在 IoT 中心为 Edison 创建设备标识。

## 需要什么
* 一个 Azure 帐户。如果没有 Azure 帐户，只需花费几分钟就能创建一个 [Azure 试用帐户](/pricing/1rmb-trial/)。
* 应该安装 Azure CLI。

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

   
		az resource group create --name iot-sample --location chinaeast
   

    `chinaeast` 是创建资源组所在的位置。如果想要使用其他位置，可运行 `az account list-locations -o table` 来查看 Azure 支持的所有位置。

5. 运行以下命令，在 iot-sample 资源组中创建 IoT 中心：

   
		az iot hub create --name {my hub name} --resource-group iot-sample
   

默认情况下，该工具在免费定价层中创建 IoT 中心。有关详细信息，请参阅 [Azure IoT 中心定价](/pricing/details/iot-hub/)。

> [AZURE.NOTE] 
IoT 中心的名称必须全局唯一。在 Azure 订阅下只能创建一个 F1 版的 Azure IoT 中心。


## 在 IoT 中心注册 Edison
每个将消息发送到 IoT 中心并从 IoT 中心接收消息的设备都必须使用唯一 ID 注册。

运行以下命令，在 IoT 中心注册 Edison：


	az iot device create --device-id myinteledison --hub-name {my hub name}


## 摘要
已创建 IoT 中心并已在 IoT 中心使用设备标识注册 Edison。已准备好学习如何将消息从 Edison 发送到 IoT 中心。

## 后续步骤
[创建 Azure Function App 和 Azure 存储帐户，以便处理和存储 IoT 中心消息][process-and-store-iot-hub-messages]。


<!-- Images and links -->


[troubleshooting]: /documentation/articles/iot-hub-intel-edison-kit-node-troubleshooting/
[process-and-store-iot-hub-messages]: /documentation/articles/iot-hub-intel-edison-kit-node-lesson3-deploy-resource-manager-template/

<!---HONumber=Mooncake_0103_2017-->