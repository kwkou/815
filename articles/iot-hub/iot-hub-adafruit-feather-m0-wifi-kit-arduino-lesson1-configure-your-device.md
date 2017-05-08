<properties
    pageTitle="配置 Azure IoT 初学者工具包 | Azure"
    description="配置 Adafruit Feather M0 WiFi 以便首次使用。"
    services="iot-hub"
    documentationcenter=""
    author="shizn"
    manager="timtl"
    tags=""
    keywords="arduino 安装, 将 arduino 连接到电脑, 安装 arduino, arduino 开发板" />
<tags
    ms.assetid="f5b334f0-a148-41aa-b374-ce7b9f5b305a"
    ms.service="iot-hub"
    ms.devlang="arduino"
    ms.topic="article"
    ms.tgt_pltfrm="na"
    ms.workload="na"
    ms.date: 3/21/2017
    wacn.date="05/08/2017"
    ms.author="xshi" />  


# 配置设备
## 执行的操作
通过组装开发板并通电打开，配置 Adafruit Feather M0 WiFi Arduino 开发板以便首次使用。如果有问题，可在[故障排除页](/documentation/articles/iot-hub-adafruit-feather-m0-wifi-kit-arduino-troubleshooting/)上查找解决方案。

## 需要什么
若要完成此操作，需要使用 Adafruit Feather M0 WiFi 初学者工具包中的以下部件：

* Adafruit Feather M0 WiFi 开发板
* Micro B - Type A USB 线缆

![工具包][kit]  


用户还需要：

* 运行 Windows、Mac 或 Linux 的计算机。
* 适合 Arduino 开发板连接的无线连接。
* Internet 访问权限（用于下载配置工具）。

## 你要学习的知识
本文介绍：

* 如何组装 Arduino 开发板并将其通电，以供接下来的课程使用。
* 如何在 Ubuntu 上添加串行端口权限。

## 将 Arduino 开发板连接到计算机

1. 将 micro USB 线缆插入顶部的 micro USB 端口。

    ![顶部的 micro USB 端口][top-micro-usb-port]  


2. 将 USB 线缆的另一端插入计算机。

    ![计算机 USB][computer-usb]  


## 在 Ubuntu 上添加串行端口权限

如果使用 Windows 或 macOS 系统，可跳过本节。如果使用 Ubuntu 系统，则需执行以下步骤，确保普通 linux 用户有权在 Arduino 开发板的 USB 端口上进行操作。

1. 现在，作为终端的普通用户：

   
		   ls -l /dev/ttyUSB*
		   # Or
		   ls -l /dev/ttyACM*
   

    将看到类似于下面的内容：

   
		   crw-rw---- 1 root uucp 188, 0 5 apr 23.01 ttyUSB0
		   # Or
		   crw-rw---- 1 root dialout 188, 0 5 apr 23.01 ttyACM0
   

    “0”可能是其他数字，或可能返回多个条目。在第一种情况下，需要 `uucp` 数据；第二种情况下，则需要 `dialout` 数据，这是文件的组所有者。

2. 将用户添加到组：

   
		sudo usermod -a -G group-name username
   

    其中，`group-name` 是在第一个步骤中找到的数据，`username` 是 linux 用户名。

3. 需先注销，然后再次登录，方可使此更改生效。

## 摘要
本文介绍了如何配置 Arduino 开发板。下一任务是安装必要的工具和软件，准备在 Arduino 开发板上运行示例应用程序。

![硬件准备就绪][hardware-is-ready]  


## 后续步骤
[获取工具][get-the-tools]
<!-- Images and links -->


[kit]: ./media/iot-hub-adafruit-feather-m0-wifi-lessons/lesson1/kit.png
[top-micro-usb-port]: ./media/iot-hub-adafruit-feather-m0-wifi-lessons/lesson1/top_usbport.jpg
[computer-usb]: ./media/iot-hub-adafruit-feather-m0-wifi-lessons/lesson1/computer_usb.jpg
[hardware-is-ready]: ./media/iot-hub-adafruit-feather-m0-wifi-lessons/lesson1/hardware_ready.jpg
[get-the-tools]: /documentation/articles/iot-hub-adafruit-feather-m0-wifi-kit-arduino-lesson1-get-the-tools-win32/

<!---HONumber=Mooncake_0116_2017-->