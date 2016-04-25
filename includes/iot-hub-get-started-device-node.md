## 创建模拟设备应用程序

在本部分中，你将创建一个 Node.js 控制台应用程序，用于模拟向 IoT 中心发送设备到云消息的设备。

1. 新建名为 **simulateddevice** 的空文件夹。在 **simulateddevice** 文件夹中，于命令提示符下使用以下命令创建新的 package.json 文件。接受所有默认值：

    ```
    npm init
    ```

2. 在 **simulateddevice** 文件夹中的命令提示符下，运行以下命令安装 **azure-iot-device-amqp** 包：

    ```
    npm install azure-iot-device-amqp --save
    ```

3. 使用文本编辑器在 **simulateddevice** 文件夹中创建一个新的 **SimulatedDevice.js** 文件。

4. 在 **SimulatedDevice.js** 文件的开头添加以下 `require` 语句：

    ```
    'use strict';

    var clientFromConnectionString = require('azure-iot-device-amqp').clientFromConnectionString;
    var Message = require('azure-iot-device').Message;
    ```

5. 添加 **connectionString** 变量并使用它来创建设备客户端。将 **{youriothubname}** 替换为你的 IoT 中心名称，将 **{yourdeviceid}** 和 **{yourdevicekey}** 替换为创建设备标识部分中生成的设备值：

    ```
    var connectionString = 'HostName={youriothubname}.azure-devices.net;DeviceId={yourdeviceid};SharedAccessKey={yourdevicekey}';
    
    var client = clientFromConnectionString(connectionString);
    ```

6. 添加以下函数以显示应用程序的输出：

    ```
    function printResultFor(op) {
      return function printResult(err, res) {
        if (err) console.log(op + ' error: ' + err.toString());
        if (res) console.log(op + ' status: ' + res.constructor.name);
      };
    }
    ```

7. 创建回调并使用 **setInterval** 函数每隔一秒将新消息发送到 IoT 中心：

    ```
    var connectCallback = function (err) {
      if (err) {
        console.log('Could not connect: ' + err);
      } else {
        console.log('Client connected');

        // Create a message and send it to the IoT Hub every second
        setInterval(function(){
            var windSpeed = 10 + (Math.random() * 4);
            var data = JSON.stringify({ deviceId: 'mydevice', windSpeed: windSpeed });
            var message = new Message(data);
            console.log("Sending message: " + message.getData());
            client.sendEvent(message, printResultFor('send'));
        }, 2000);
      }
    };
    ```

8. 与 IoT 中心建立连接并开始发送消息：

    ```
    client.open(connectCallback);
    ```

9. 保存并关闭 **SimulatedDevice.js** 文件。

> [AZURE.NOTE] 为简单起见，本教程不实现任何重试策略。在生产代码中，应该按 MSDN 文章[暂时性故障处理][lnk-transient-faults]中所述实现重试策略（例如指数退让）。

<!-- Links -->
[lnk-transient-faults]: https://msdn.microsoft.com/zh-cn/library/hh680901(v=pandp.50).aspx

<!---HONumber=Mooncake_0321_2016-->