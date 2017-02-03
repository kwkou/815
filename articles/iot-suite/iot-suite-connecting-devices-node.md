<properties
   pageTitle="使用 Node.js 连接设备 | Azure"
   description="介绍如何使用以 Node.js 编写的应用程序将设备连接到 Azure IoT 套件预配置远程监视解决方案。"
   services=""
   suite="iot-suite"
   documentationCenter="na"
   authors="dominicbetts"
   manager="timlt"
   editor=""/>  


<tags
   ms.service="iot-suite"
   ms.devlang="na"
   ms.topic="article"
   ms.tgt_pltfrm="na"
   ms.workload="na"
   ms.date="01/04/2017"
   wacn.date="01/25/2017"
   ms.author="dobett"/>  



# 将设备连接到远程监视预配置解决方案 (Node.js)

[AZURE.INCLUDE [iot-suite-selector-connecting](../../includes/iot-suite-selector-connecting.md)]

## 生成并运行 node.js 示例解决方案
1. 若要克隆*适用于 Node.j 的 Azure IoT SDK* GitHub 存储库并在桌面环境中安装，请按照[准备开发环境][lnk-github-prepare]中的说明进行操作。
2. 在 [azure-iot-sdk-node][lnk-github-repo] 存储库的本地副本中，将以下两个文件从 device/samples 文件夹复制到设备上的文件夹：
   
   * package.json
   * remote\_monitoring.js
3. 打开 remote\_monitoring.js 文件并查找以下变量：

    
		var connectionString = "[IoT Hub device connection string]";
    

4. 将 **[IoT Hub device connection string]** 替换为设备连接字符串。可以在远程监视解决方案仪表板中找到 IoT 中心主机名、设备 ID 和设备密钥的值。设备连接字符串具有以下格式：

    
		HostName={your IoT Hub hostname};DeviceId={your device id};SharedAccessKey={your device key}
    
   
    如果 IoT 中心主机名是 **contoso**，而设备 ID 为 **mydevice**，则连接字符串如下所示：
   
    
		var connectionString = "HostName=contoso.azure-devices.cn;DeviceId=mydevice;SharedAccessKey=2s ... =="
    

5. 保存文件。在包含这些文件的文件夹中的命令提示符处运行以下命令，以安装所需包，然后运行示例应用程序：

    
		npm install --save
		node remote_monitoring.js
    

[AZURE.INCLUDE [iot-suite-visualize-connecting](../../includes/iot-suite-visualize-connecting.md)]

[lnk-github-repo]: https://github.com/azure/azure-iot-sdk-node
[lnk-github-prepare]: https://github.com/Azure/azure-iot-sdk-node/blob/master/doc/node-devbox-setup.md

<!---HONumber=Mooncake_1226_2016-->