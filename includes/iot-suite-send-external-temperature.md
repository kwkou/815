## 配置 Node.js 模拟设备
1. 在远程监视仪表板上单击“+ 添加设备”，然后添加*自定义设备*。记下 IoT 中心主机名、设备 ID 和设备密钥。在本教程稍后准备 remote\_monitoring.js 设备客户端应用程序时，需要使用它们。
2. 请确保已在开发计算机上安装 Node.js 0.12.x 或更高版本。在命令提示符或 shell 中运行 `node --version` 以检查版本。有关使用包管理器在 Linux 上安装 Node.js 的信息，请参阅 [Installing Node.js via package manager][node-linux]（通过包管理器安装 Node.js）。
3. 安装 Node.js 之后，请将最新版本的 [azure-iot-sdk-node][lnk-github-repo] 存储库复制到开发计算机。始终对最新版的库和示例使用**主**分支。
4. 在 [azure-iot-sdk-node][lnk-github-repo] 存储库的本地副本中，将以下两个文件从 node/device/samples 文件夹复制到开发计算机上的某个空文件夹：
   
   - packages.json
   - remote\_monitoring.js


5. 打开 remote\_monitoring.js 文件并查找以下变量定义：
   
    
        var connectionString = "[IoT Hub device connection string]";
    
6. 将 **[IoT Hub device connection string]** 替换为设备连接字符串。使用在步骤 1 中记下的 IoT 中心主机名、设备 ID 和设备密钥的值。设备连接字符串具有以下格式：
   
    
        HostName={your IoT Hub hostname};DeviceId={your device id};SharedAccessKey={your device key}
    
   
    如果 IoT 中心主机名是 **contoso**，而设备 ID 为 **mydevice**，则连接字符串如下代码片段所示：
   
    
        var connectionString = "HostName=contoso.azure-devices.cn;DeviceId=mydevice;SharedAccessKey=2s ... =="
    
7. 保存文件。在包含这些文件的文件夹中的 shell 或命令提示符处运行以下命令，以安装所需包，然后运行示例应用程序：
   
    
        npm install
        node remote_monitoring.js
    

## 观察动态遥测的工作动态
仪表板将显示现有模拟设备的温度与湿度遥测数据：

![默认仪表板][image1]

如果你选择在上一部分中运行的 Node.js 模拟设备，将看到温度、湿度与外部温度遥测数据：

![将外部温度添加到仪表板][image2]

远程监视解决方案将自动检测其他外部温度遥测类型，并将其添加到仪表板上的图表中。

[node-linux]: https://github.com/nodejs/node-v0.x-archive/wiki/Installing-Node.js-via-package-manager
[lnk-github-repo]: https://github.com/Azure/azure-iot-sdk-node
[image1]: ./media/iot-suite-send-external-temperature/image1.png
[image2]: ./media/iot-suite-send-external-temperature/image2.png

<!---HONumber=Mooncake_1226_2016-->