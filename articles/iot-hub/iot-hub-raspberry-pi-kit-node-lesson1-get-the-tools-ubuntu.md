<properties
    pageTitle="获取工具 (Ubuntu 16.04) | Azure"
    description="下载并安装适用于 Ubuntu 上 Pi 的第一个示例应用程序的必需工具和软件。"
    services="iot-hub"
    documentationcenter=""
    author="shizn"
    manager="timlt"
    tags=""
    keywords="iot 开发, iot 软件, 物联网软件, 在 ubuntu 上安装 git, gulp 运行, 安装 node js ubuntu" />
<tags
    ms.assetid="4d5e45c0-1db9-4662-a039-99ba26333085"
    ms.service="iot-hub"
    ms.devlang="node"
    ms.topic="article"
    ms.tgt_pltfrm="na"
    ms.workload="na"
    ms.date="3/21/2017"
    wacn.date="05/08/2017"
    ms.author="xshi" />  

# 获取工具 (Ubuntu 16.04)
> [AZURE.SELECTOR]
- [Windows 7 或更高版本](/documentation/articles/iot-hub-raspberry-pi-kit-node-lesson1-get-the-tools-win32/)
- [Ubuntu 16.04](/documentation/articles/iot-hub-raspberry-pi-kit-node-lesson1-get-the-tools-ubuntu/)
- [macOS 10.10](/documentation/articles/iot-hub-raspberry-pi-kit-node-lesson1-get-the-tools-mac/)

## 执行的操作
下载适用于 Raspberry Pi 3 的第一个示例应用程序的开发工具和软件。如果有问题，可在[故障排除页](/documentation/articles/iot-hub-raspberry-pi-kit-node-troubleshooting/)上查找解决方案。

## 你要学习的知识
本文介绍：

* 如何安装 Git 和 Node.js。
  * [Git](https://git-scm.com) 是一个开源分布式版本控制系统。本文的示例应用程序存储在 Git 中。
  * [Node.js](https://nodejs.org/en/) 是一个包生态系统很丰富的 JavaScript 运行时。
* 如何使用 NPM 安装其他 Node.js 开发工具。
  * 所需 Node.js 版本最低为 4.5 LTS。
  * [NPM](https://www.npmjs.com) 是适用于 Node.js 的一个包管理器。

## 所需条件
若要完成此操作，需具备：

* Internet 连接，用于下载开发工具和软件。
* 运行 Ubuntu 16.04 或更高版本的计算机。

## 安装 Git、Node.js 和 NPM
使用键盘快捷方式 `Ctrl + Alt + T` 打开一个终端并运行以下命令：

        sudo apt-get update
        curl -sL https://deb.nodesource.com/setup_6.x | sudo -E bash -
        sudo apt-get install -y nodejs
        sudo apt-get install git


## 安装其他 Node.js 开发工具
使用 [gulp.js](http://gulpjs.com) 将示例应用程序自动部署到 Pi。也可使用 [device-discovery-cli](https://github.com/Azure/device-discovery-cli) 检索有关 IoT 设备的网络信息。

在终端运行以下命令，安装 `gulp` 和 `device-discovery-cli`：

        sudo npm install -g device-discovery-cli gulp

如果无法在 Ubuntu 上安装 Node.js 和这些额外的开发工具，请参阅[故障排除指南](/documentation/articles/iot-hub-raspberry-pi-kit-node-troubleshooting/)，了解常见问题的解决方案。

## 安装 Visual Studio Code
[下载](https://code.visualstudio.com/docs/setup/linux)并安装 Visual Studio Code。Visual Studio Code 是一种轻型但却功能强大的源代码编辑器，适用于 Windows、Linux 和 macOS。本教程后面需使用此编辑器编辑示例代码。

## 摘要
用户已为第一个示例应用程序安装所需的开发工具和软件。下一任务是在 Pi 上创建、部署和运行示例应用程序。

## 后续步骤
[创建和部署 blink 示例应用程序](/documentation/articles/iot-hub-raspberry-pi-kit-node-lesson1-deploy-blink-app/)

<!---HONumber=Mooncake_0109_2017-->
<!--Update_Description:update meta properties-->