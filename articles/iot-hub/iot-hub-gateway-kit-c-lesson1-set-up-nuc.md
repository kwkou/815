<properties
    pageTitle="将 Intel NUC 设置为 Azure IoT 网关 | Azure"
    description="将 Intel NUC 设置为传感器和 Azure IoT 中心之间的 IoT 网关，用于收集传感器信息并将其发送到 IoT 中心。"
    services="iot-hub"
    documentationcenter=""
    author="shizn"
    manager="yjianfeng"
    tags=""
    keywords="iot 网关, intel nuc, nuc 计算机, DE3815TYKE" />
<tags
    ms.assetid="917090d6-35c2-495b-a620-ca6f9c02b317"
    ms.service="iot-hub"
    ms.devlang="c"
    ms.topic="article"
    ms.tgt_pltfrm="na"
    ms.workload="na"
    ms.date="10/28/2016"
    wacn.date="01/23/2017"
    ms.author="xshi" />  


# 将 Intel NUC 设置为 IoT 网关

## 执行的操作

- 将 Intel NUC 设置为 IoT 网关。
- 在 Intel NUC 上安装 Azure IoT 网关 SDK 包。
- 在 Intel NUC 上运行“hello\_world”示例应用程序，验证网关功能。
如果有问题，可在[故障排除页](/documentation/articles/iot-hub-gateway-kit-c-troubleshooting/)上查找解决方案。

## 你要学习的知识

本课介绍以下内容：

- 如何将 Intel NUC 与外围设备连接。
- 如何使用智能包管理器在 Intel NUC 上安装和更新所需的包。
- 如何运行“hello\_world”示例应用程序来验证网关功能。

## 需要什么

- 预安装 Intel IoT 网关软件套件 (Wind River Linux *7.0.0.13) 的 Intel NUC 工具包 DE3815TYKE。
- 以太网电缆。
- 键盘。
- HDMI 或 VGA 电缆。
- 带有 HDMI 或 VGA 端口的监视器。

![网关工具包](./media/iot-hub-gateway-kit-lessons/lesson1/kit.png)  


## 将 Intel NUC 与外设连接

下图是已连接到各种外围设备的 Intel NUC 示例:

1. 连接到键盘。
2. 通过 VGA 电缆或 HDMI 电缆连接到监视器。
3. 通过以太网电缆连接到有线网络。
4. 通过电源线连接到电源。

    ![连接到外围设备的 Intel NUC](./media/iot-hub-gateway-kit-lessons/lesson1/nuc.png)  


## 通过安全外壳 (SSH) 从主计算机连接到 Intel NUC 系统

此时需要键盘和监视器才能获取 NUC 设备的 IP 地址。如果已知道 IP 地址，则可跳到本部分的步骤 3。

1. 按电源按钮打开 Intel NUC，登录系统。

    默认用户名和密码都是 `root`。

2. 运行 `ifconfig` 命令，获取 NUC 的 IP 地址。在 NUC 设备上完成此步骤。

    以下是命令输出的示例。

    ![显示 NUC IP 的 ifconfig 输出](./media/iot-hub-gateway-kit-lessons/lesson1/ifconfig.png)  


    在此示例中，`inet addr:` 后面的值是计划从主计算机远程连接到 Intel NUC 时所需的 IP 地址。

3. 使用主计算机的以下任一 SSH 客户端连接到 Intel NUC。

   - 适用于 Windows 的 [PuTTY](http://www.putty.org/)。
   - Ubuntu 或 macOS 上的内置 SSH 客户端。

    通过主计算机在 Intel NUC 上执行操作更高效。需要 IP 地址、用户名和密码才能通过 SSH 客户端连接 NUC。下面是在 macOS 上使用 SSH 客户端的示例。
    
    ![在 macOS 上运行的 SSH 客户端](./media/iot-hub-gateway-kit-lessons/lesson1/ssh.png)

## 安装 Azure IoT 网关 SDK 包

Azure IoT 网关 SDK 包中包含 SDK 及其依赖项的预编译二进制文件。这些二进制文件包括 Azure IoT 网关 SDK、Azure IoT SDK 和相应的工具。该包还包含用于验证网关功能的“hello\_world”示例应用程序。SDK 是网关的核心部分。若要安装包，请执行以下步骤：

1. 在终端窗口中运行以下命令，添加 IoT 云存储库：

   
		rpm --import http://iotdk.intel.com/misc/iot_pub.key
		smart channel --add IoT_Cloud type=rpm-md name="IoT_Cloud" baseurl=http://iotdk.intel.com/repos/iot-cloud/wrlinux7/rcpl13/ -y
   

    `rpm` 命令导入 rpm 密钥。`smart channel` 命令将 rpm 通道添加到智能包管理器。运行 `smart update` 命令前，将看到如下输出。

    ![rpm 和智能通道命令输出](./media/iot-hub-gateway-kit-lessons/lesson1/rpm_smart_channel.png)  


   
		smart update
   

2. 运行以下命令来安装包：

   
		smart install packagegroup-cloud-azure -y
   

    `packagegroup-cloud-azure` 是包的名称。`smart install` 命令用于安装包。

    安装此包后，Intel NUC 应可用作网关。

## 运行 Azure IoT 网关 SDK“hello\_world”示例应用程序

转到 `azureiotgatewaysdk/samples` 并运行“hello\_world”示例应用程序。此示例应用程序通过 `hello_world.json` 文件创建网关，并使用 Azure IoT 网关 SDK 体系结构的基本组件每隔 5 秒将“hello world”消息记录到文件。

运行以下命令，运行“hello\_world”示例应用程序：


		cd /usr/share/azureiotgatewaysdk/samples/hello_world/
		./hello_world hello_world.json


如果网关功能正常工作，示例应用程序生成以下输出：

![应用程序输出](./media/iot-hub-gateway-kit-lessons/lesson1/hello_world.png)  


如果有问题，可在[故障排除页](/documentation/articles/iot-hub-gateway-kit-c-troubleshooting/)上查找解决方案。

## 摘要

祝贺你！ 你已将 Intel NUC 设置为网关。现可进入下一课，了解如何设置主机、创建 Azure IoT 中心以及注册 Azure IoT 中心逻辑设备。

## 后续步骤
[准备好主计算机和 Azure IoT 中心](/documentation/articles/iot-hub-gateway-kit-c-lesson2-get-the-tools-win32/)

<!---HONumber=Mooncake_0116_2017-->