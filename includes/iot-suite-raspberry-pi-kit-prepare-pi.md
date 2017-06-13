## <a name="prepare-your-raspberry-pi"></a>准备 Raspberry Pi

### <a name="install-raspbian"></a>安装 Raspbian

如果这是首次使用 Raspberry Pi，需使用工具包中随附的 SD 卡上的 NOOBS 来安装 Raspbian 操作系统。 [Raspberry Pi 软件指南][lnk-install-raspbian]介绍了如何在 Raspberry Pi 上安装操作系统。 本教程假定已在 Raspberry Pi 上安装 Raspbian 操作系统。

> [AZURE.NOTE]
> [适用于 Raspberry Pi 3 的 Microsoft Azure IoT 初学者工具包][lnk-starter-kits]随附的 SD 卡已安装 NOOBS。 可以从该卡启动 Raspberry Pi，然后选择安装 Raspbian OS。

### <a name="set-up-the-hardware"></a>安装硬件

本教程使用[适用于 Raspberry Pi 3 的 Microsoft Azure IoT 初学者工具包][lnk-starter-kits]中随附的 BME280 传感器生成遥测数据。 它使用 LED 来指示 Raspberry Pi 何时处理解决方案仪表板中的方法调用。

试验板上的组件包括：

- 红色 LED
- 220 欧电阻器（红色、红色、棕色）
- BME280 传感器

下图显示了如何连接硬件：

![Raspberry Pi 的硬件安装][img-connection-diagram]

下表总结了如何从 Raspberry Pi 连接到试验板上的组件：

| Raspberry Pi            | 试验板             |颜色         |
| ----------------------- | ---------------------- | ------------- |
| GND（引脚 14）            | LED - ve 引脚 (18A)      | 紫色          |
| GPCLK0（引脚 7）          | 电阻器 (25A)         | 橙色          |
| SPI_CE0（引脚 24）        | CS (39A)               | 蓝色          |
| SPI_SCLK（引脚 23）       | SCK (36A)              | 黄色        |
| SPI_MISO（引脚 21）       | SDO (37A)              | 白色         |
| SPI_MOSI（引脚 19）       | SDI (38A)              | 绿色         |
| GND（引脚 6）             | GND (35A)              | 黑色         |
| 3.3 V（引脚 1）           | 3Vo (34A)              | 红色           |

若要完成硬件设置，需执行以下操作：

- 将 Raspberry Pi 连接到工具包中提供的电源。
- 使用工具包中提供的以太网电缆将 Raspberry Pi 连接到网络。 也可为 Raspberry Pi 设置[无线连接][lnk-pi-wireless]。

现在已经完成 Raspberry Pi 的硬件设置。

### <a name="sign-in-and-access-the-terminal"></a>登录并访问终端

可以通过两个选项来访问 Raspberry Pi 上的终端环境：

- 如果已将键盘和显示器连接到 Raspberry Pi，则可使用 Raspbian GUI 访问终端窗口。

- 使用台式机中的 SSH 访问 Raspberry Pi 上的命令行。

#### <a name="use-a-terminal-window-in-the-gui"></a>使用 GUI 中的终端窗口

Raspbian 的默认凭据为用户名 **pi** 和密码 **raspberry**。 在 GUI 的任务栏中，可以使用外观像显示器的图标启动 **Terminal** 实用工具。

#### <a name="sign-in-with-ssh"></a>使用 SSH 登录

可以使用 SSH 对 Raspberry Pi 进行命令行访问。 [安全外壳][lnk-pi-ssh]一文介绍了如何在 Raspberry Pi 上配置 SSH，以及如何从 [Windows][lnk-ssh-windows] 或 [Linux 和 Mac OS][lnk-ssh-linux] 进行连接。

使用用户名 **pi** 和密码 **raspberry** 登录。

#### <a name="optional-share-a-folder-on-your-raspberry-pi"></a>可选：共享 Raspberry Pi 上的文件夹

（可选）可以将 Raspberry Pi 上的文件夹与桌面环境共享。 共享文件夹之后，即可使用偏好的桌面文本编辑器（例如 [Visual Studio Code](https://code.visualstudio.com/) 或 [Sublime Text](http://www.sublimetext.com/)）在 Raspberry Pi 上编辑文件，而不必使用 `nano` 或 `vi`。

若要与 Windows 共享文件夹，请在 Raspberry Pi 上配置 Samba 服务器。 也可将内置的 [SFTP](https://www.raspberrypi.org/documentation/remote-access/) 服务器与桌面上的 SFTP 客户端配合使用。

### <a name="enable-spi"></a>启用 SPI

在运行示例应用程序之前，必须在 Raspberry Pi 上启用串行外围设备接口 (SPI) 总线。 Raspberry Pi 通过 SPI 总线与 BME280 传感器设备通信。 使用以下命令编辑配置文件：

`sudo nano /boot/config.txt`

查找行：

    #dtparam=spi=on

- 若要取消注释行，请删除开头的 `#`。
- 保存所做的更改（按 **Ctrl-O**，然后按 **Enter**），然后退出编辑器（按 **Ctrl-X**）。
- 若要启用 SPI，请重新启动 Raspberry Pi。 重新启动会断开终端的连接，需要在 Raspberry Pi 重新启动时重新登录：

  `sudo reboot`


[img-connection-diagram]: ./media/iot-suite-raspberry-pi-kit-prepare-pi/rpi2_remote_monitoring.png

[lnk-install-raspbian]: https://www.raspberrypi.org/learning/software-guide/quickstart/
[lnk-pi-wireless]: https://www.raspberrypi.org/documentation/configuration/wireless/README.md
[lnk-pi-ssh]: https://www.raspberrypi.org/documentation/remote-access/ssh/README.md
[lnk-ssh-windows]: https://www.raspberrypi.org/documentation/remote-access/ssh/windows.md
[lnk-ssh-linux]: https://www.raspberrypi.org/documentation/remote-access/ssh/unix.md
[lnk-starter-kits]: /develop/iot/iot-starter-kits/