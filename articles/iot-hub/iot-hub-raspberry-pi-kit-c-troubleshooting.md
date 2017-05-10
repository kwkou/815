<properties
    pageTitle="故障排除 | Azure"
    description="Raspberry Pi Node.js 体验的故障排除页"
    services="iot-hub"
    documentationcenter=""
    author="shizn"
    manager="timtl"
    tags=""
    keywords="iot 问题, 物联网问题" />
<tags
    ms.assetid="3653c79b-8a0c-41d4-b0bf-f6b4edb4d233"
    ms.service="iot-hub"
    ms.devlang="c"
    ms.topic="article"
    ms.tgt_pltfrm="na"
    ms.workload="na"
    ms.date="3/21/2017"
    wacn.date="05/08/2017"
    ms.author="xshi" />  


# 故障排除
## 硬件问题
### 应用程序运行正常，但 LED 不闪烁
此问题总是与硬件线路连接有关。使用以下步骤确定问题。

1. 查看是否在板上选择了正确的 **GPIO**。两个端口应该为 **GPIO GND（管脚 6）**和 **GPIO 04（管脚 7）**。
2. 查看 LED 的极性是否正确。长脚为**正**，即阳极管脚。
3. 在 Raspberry Pi 3 上使用 **3.3V 管脚**和 **GND 管脚**。将 Pi 视为直流电源。查看 LED 是否正常。

![LED 规格](./media/iot-hub-raspberry-pi-lessons/troubleshooting/led_spec.png)  


### 其他硬件问题
若要了解如何解决 Raspberry Pi 3 的常见问题，请参阅[官方的故障排除页](http://elinux.org/R-Pi_Troubleshooting)。

## Node.js 包问题
### 在 Gulp 任务期间没有响应
如果在运行 gulp 任务时遇到问题，可添加 `--verbose` 选项进行调试。请尝试使用 `Ctrl + C` 终止当前 gulp 任务，然后在控制台窗口中运行以下命令，以便查看调试消息。可以在控制台输出中查看详细的错误消息。


	gulp --verbose


### 设备发现问题
如需使用 `devdisco` 命令时的常见问题的疑难解答帮助，请查看[自述文件](https://github.com/Azure/device-discovery-cli/blob/develop/readme.md)。

### NPM 问题
请尝试使用以下命令更新 NPM 包：


	npm install -g npm


如果问题仍然存在，请在本文末尾留下你的评论，或者在[示例存储库](https://github.com/Azure-Samples/iot-hub-c-raspberrypi-getting-started)中创建一个 GitHub 问题

## 远程调试

即将推出在 Visual Studio 代码C/C++ 扩展中可用的远程调试。

同时，你可以通过最喜爱的 SSH 终端使用 GDB：


	cd c-pi-lesson-x
	sudo gdb app


## Azure-CLI 问题
Azure 命令行接口 (Azure CLI) 为预览版。

如果在使用此工具时遇到 Bug，请在 GitHub 存储库的“问题”部分提交[问题](https://github.com/Azure/azure-cli/issues)。

如果遇到“找不到满足需求的版本”，请运行以下命令，将 pip 升级到最新版本。


	python -m pip install --upgrade pip


## Python 安装问题
### 旧版安装问题 (macOS)
安装 **pip** 时，如果使用 **su** 权限安装的包较旧，则会引发权限错误。之所以会出现这种情况，是因为未彻底卸载以前使用 brew (macOS) 安装的 Python。之前安装的某些 **pip** 包由根权限创建，会导致权限错误。删除这些通过根权限安装的包即可解决问题。通过以下步骤完成该任务：

1. 转到：/usr/local/lib/python2.7/site-packages
2. 列出由根权限创建的包：`ls -l | grep root`
3. 卸载步骤 2 中的包：`sudo rm -rf {package name}`
4. 重新安装 Python。

## Azure IoT 中心问题
如果已通过 `azure-cli` 成功预配 Azure IoT 中心，且需使用工具管理连接到 IoT 中心的设备，可尝试以下工具：

### 设备资源管理器
[设备资源管理器](https://github.com/Azure/azure-iot-sdk-csharp/blob/master/tools/DeviceExplorer)在 Windows 本地计算机上运行，并连接到 Azure 中的 IoT 中心。它与以下 [IoT 中心终结点](/documentation/articles/iot-hub-devguide/)通信：

* *设备标识管理*：用于预配和管理注册到 IoT 中心的设备。
* *接收从设备到云的消息*：用于监视从设备发送到 IoT 中心的消息。
* *发送从云到设备的消息*：用于将消息从 IoT 中心发送到设备。

在此工具中配置 `IoT hub connection string`，以便使用其所有功能。

### IoT 中心资源管理器
[IoT 中心资源管理器](https://github.com/Azure/iothub-explorer)是示例多平台 CLI 工具，可用于管理设备客户端。可以使用该工具在标识注册表中管理设备、监视从设备到云的消息，以及发送从云到设备的命令。

若要安装最新（预发行）版的 iothub-explorer 工具，请在命令行环境中运行以下命令：


	npm install -g iothub-explorer@latest


可以使用以下命令获取所有 iothub-explorer 命令及其参数的更多帮助：


	iothub-explorer help


### Azure 门户预览
完整的 CLI 体验有助于用户创建和管理其所有 Azure 资源。还可能需要借助 [Azure 门户预览](/documentation/articles/azure-portal-overview/)对 Azure 资源进行预配、管理和调试。

## Azure 存储问题
[Microsoft Azure 存储资源管理器（预览版）](http://storageexplorer.com)是 Microsoft 推出的一款独立的应用，可用于在 Windows、OS X 和 Linux 上处理 Azure 存储数据。可以使用此工具连接到表并查看其中的数据。可以使用此工具排查 Azure 存储问题。

<!---HONumber=Mooncake_0206_2017-->
<!--Update_Description:update link references-->