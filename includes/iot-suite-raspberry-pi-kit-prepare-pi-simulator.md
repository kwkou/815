## <a name="prepare-your-raspberry-pi"></a>准备 Raspberry Pi

### <a name="install-raspbian"></a>安装 Raspbian

如果这是首次使用 Raspberry Pi，需使用工具包中随附的 SD 卡上的 NOOBS 来安装 Raspbian 操作系统。 [Raspberry Pi 软件指南][lnk-install-raspbian]介绍了如何在 Raspberry Pi 上安装操作系统。 本教程假定已在 Raspberry Pi 上安装 Raspbian 操作系统。

> [AZURE.NOTE]
> [适用于 Raspberry Pi 3 的 Microsoft Azure IoT 初学者工具包][lnk-starter-kits]随附的 SD 卡已安装 NOOBS。 可以从该卡启动 Raspberry Pi，然后选择安装 Raspbian OS。

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

[lnk-install-raspbian]: https://www.raspberrypi.org/learning/software-guide/quickstart/
[lnk-pi-wireless]: https://www.raspberrypi.org/documentation/configuration/wireless/README.md
[lnk-pi-ssh]: https://www.raspberrypi.org/documentation/remote-access/ssh/README.md
[lnk-ssh-windows]: https://www.raspberrypi.org/documentation/remote-access/ssh/windows.md
[lnk-ssh-linux]: https://www.raspberrypi.org/documentation/remote-access/ssh/unix.md
[lnk-starter-kits]: /develop/iot/iot-starter-kits/