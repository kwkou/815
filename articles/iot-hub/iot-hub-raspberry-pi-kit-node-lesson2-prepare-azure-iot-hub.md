<properties
	title="创建中心并注册 Raspberry Pi 3"
	description="使用 Azure CLI 创建资源组、创建 Azure IoT 中心，以及在 IoT 中心注册 Pi。"
	services="iot-hub"
	documentationcenter=""
	author="shizn"
	manager="timlt"
	tags=""
	keywords=""/>  


<tags
	ms.service="iot-hub"
	ms.date="10/21/2016"
	wacn.date="12/19/2016"/>  


# 创建 IoT 中心并注册 Raspberry Pi 3
## 执行的操作
* 创建资源组。
* 在资源组中创建 Azure IoT 中心。
* 使用 Azure 命令行接口 (Azure CLI) 将 Raspberry Pi 3 添加到 Azure IoT 中心。

使用 Azure CLI 将 Pi 添加到 IoT 中心时，服务会为 Pi 生成一个密钥，以便通过服务进行身份验证。如果有问题，请在[故障排除页](/documentation/articles/iot-hub-raspberry-pi-kit-node-troubleshooting/)上查找解决方案。

## 你要学习的知识
* 如何使用 Azure CLI 创建 IoT 中心
* 如何在 IoT 中心为 Pi 创建设备标识

## 需要什么
* 一个 Azure 帐户
* 已安装 Azure CLI 的 Mac 或 Windows 计算机

## 创建 IoT 中心
Azure IoT 中心用于连接、监视并管理数百万 IoT 资产。若要创建 IoT 中心，请执行以下步骤：

1. 通过运行以下命令登录到 Azure 帐户：
   
    ```bash
    az login
    ```
   
    成功登录后，将会列出所有可用的 Azure 订阅。
2. 运行以下命令，设置默认的需要使用的 Azure 订阅：
   
    ```bash
    az account set -n {subscription id or name}
    ```
   
    可以在 `az login` 的输出中找到订阅 ID 或名称。
3. 通过运行以下命令注册提供程序：
   
    ```bash
    az resource provider register -n "Microsoft.Devices"
    ```
   
    必须先注册提供程序，然后才能部署该提供程序提供的 Azure 资源。
   
   > [AZURE.NOTE] Azure 门户或正在使用的 Azure CLI 会自动注册大多数提供程序，但非全部。有关提供程序的详细信息，请参阅[排查使用 Azure Resource Manager 时的常见 Azure 部署错误](/documentation/articles/resource-manager-common-deployment-errors/)。
   > 
   > 
4. 运行以下命令，在“中国东部”区域创建名为 iot-sample 的资源组：
   
    ```bash
    az resource group create --name iot-sample --location chinaeast
    ```
5. 运行以下命令，在 iot-sample 资源组中创建 IoT 中心：
   
    ```bash
    az iot hub create --name {my hub name} --resource-group iot-sample
    ```
   
    用户创建的 IoT 中心的默认版本为 F1，免费。有关详细信息，请参阅 [Azure IoT 中心定价](/pricing/details/iot-hub/)。

> [AZURE.NOTE] IoT 中心的名称必须全局唯一。
> 
> 在 Azure 订阅下只能创建一个 F1 版的 IoT 中心。
> 
> 

## 在 IoT 中心注册 Pi
每个将消息发送到 IoT 中心并从 IoT 中心接收消息的设备都必须使用唯一 ID 注册。

运行以下命令，在中心注册 Pi：

```bash
az iot device create --device-id myraspberrypi --hub {my hub name} --resource-group iot-sample
```

## 摘要
用户已创建 Azure IoT 中心并已在 IoT 中心使用设备标识注册 Pi。用户已准备学习如何将消息从 Pi 发送到 IoT 中心。

## 后续步骤
[创建 Azure 函数应用和 Azure 存储帐户，以便处理和存储 IoT 中心消息](/documentation/articles/iot-hub-raspberry-pi-kit-node-lesson3-deploy-resource-manager-template/)

<!---HONumber=Mooncake_1212_2016-->