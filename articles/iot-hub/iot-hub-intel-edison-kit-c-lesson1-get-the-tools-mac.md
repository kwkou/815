<properties
    pageTitle="获取用于 Azure IoT 初学者工具包 (macOS 10.10) 的工具 | Azure"
    description="下载并安装用于 macOS 上的 Edison 的第一个示例应用程序的必需工具和软件。"
    services="iot-hub"
    documentationcenter=""
    author="shizn"
    manager="timtl"
    tags=""
    keywords="arduino 开发工具, iot 开发, iot 软件, 物联网软件, 在 mac 上安装 git, 安装 node js mac" />
<tags
    ms.assetid="3f764f2e-25fa-4dde-9e8d-d278453fccfd"
    ms.service="iot-hub"
    ms.devlang="c"
    ms.topic="article"
    ms.tgt_pltfrm="na"
    ms.workload="na"
    ms.date="3/21/2017"
    wacn.date="05/08/2017"
    ms.author="xshi" />  


# 获取工具 (macOS 10.10)
> [AZURE.SELECTOR]
- [Windows 7 或更高版本][windows]
- [Ubuntu 16.04][ubuntu]
- [macOS 10.10][macos]

## 执行的操作
下载用于 Intel Edison 的第一个示例应用程序的开发工具和软件。如果有问题，可在[故障排除页][troubleshooting]上查找解决方案。

> [AZURE.NOTE]
尽管主逻辑的编程语言为 C，课程中仍使用 Node.js 工具生成和部署示例应用程序。

## 你要学习的知识
本文介绍：

* 如何安装 Git 和 Node.js。
  * [Git](https://git-scm.com) 是一个开源分布式版本控制系统。本文的示例应用程序存储在 Git 中。
  * [Node.js](https://nodejs.org/en/) 是一个包生态系统很丰富的 JavaScript 运行时。
* 如何使用 NPM 安装其他 Node.js 开发工具。
  * 所需 Node.js 版本最低为 4.5 LTS。
  * [NPM](https://www.npmjs.com) 是适用于 Node.js 的一个包管理器。

## 需要什么
若要完成此操作，需具备：

 - Internet 连接，用于下载开发工具和软件。
 - 运行 macOS Yosemite (10.10) 或更高版本的 Mac。

## 安装 Git 和 Node.js
若要安装 Git 和 Node.js，请按以下步骤使用 [Homebrew](http://brew.sh) 包管理实用程序：

1. 安装 Homebrew。若已安装 Homebrew，请转到步骤 2。

   1. 按 `Cmd + Space` 并输入 `Terminal` 即可打开终端。
   2. 运行以下命令：

      
		    /usr/bin/ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"
      
2. 运行以下命令，安装 Git 和 Node.js：

   
		brew install node git
   

## 安装其他 Node.js 开发工具
使用 [gulp.js](http://gulpjs.com) 将示例应用程序自动部署到 Edison。

在终端运行以下命令，安装 `gulp`：


	sudo npm install -g gulp


如果无法在 macOS 上安装 Node.js 和这些额外的开发工具，请参阅[故障排除指南][troubleshooting]，了解常见问题的解决方案。

## 安装 Visual Studio Code
[下载](https://code.visualstudio.com/docs/setup/osx)并安装 Visual Studio Code。Visual Studio Code 是一种轻型但却功能强大的源代码编辑器，适用于 Windows、Linux 和 macOS。本教程后面需使用此编辑器编辑示例代码。

## 摘要
用户已为第一个示例应用程序安装所需的开发工具和软件。下一任务是在 Edison 上创建、部署和运行示例应用程序。

## 后续步骤
[创建和部署 blink 应用程序][create-and-deploy-the-blink-application]
<!-- Images and links -->


[troubleshooting]: /documentation/articles/iot-hub-intel-edison-kit-c-troubleshooting/
[create-and-deploy-the-blink-application]: /documentation/articles/iot-hub-intel-edison-kit-c-lesson1-deploy-blink-app/
[windows]: /documentation/articles/iot-hub-intel-edison-kit-c-lesson1-get-the-tools-win32/
[ubuntu]: /documentation/articles/iot-hub-intel-edison-kit-c-lesson1-get-the-tools-ubuntu/
[macos]: /documentation/articles/iot-hub-intel-edison-kit-c-lesson1-get-the-tools-mac/

<!---HONumber=Mooncake_0116_2017-->