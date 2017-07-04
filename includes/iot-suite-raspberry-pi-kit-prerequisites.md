## <a name="prerequisites"></a>先决条件

需要有效的 Azure 订阅才能完成此教程。

> [AZURE.NOTE]
> 如果没有帐户，只需花费几分钟就能创建一个试用帐户。 有关详细信息，请参阅 [Azure 试用][lnk-free-trial]。

### <a name="required-software"></a>所需软件

需要在台式机上安装 SSH 客户端，然后才能远程访问 Raspberry Pi 上的命令行。

- Windows 不包括 SSH 客户端。 建议使用 [PuTTY](http://www.putty.org/)。
- 大多数 Linux 发行版和 Mac OS 包括命令行 SSH 实用工具。 有关详细信息，请参阅 [SSH Using Linux or Mac OS](https://www.raspberrypi.org/documentation/remote-access/ssh/unix.md)（使用 Linux 或 Mac OS 的 SSH）。

### <a name="required-hardware"></a>所需硬件

一个台式机，用于通过远程方式连接到 Raspberry Pi 上的命令行。

[适用于 Raspberry Pi 3 的 Microsoft IoT 初学者工具包][lnk-starter-kits]或等效组件。 本教程使用工具包中的以下项目：

- Raspberry Pi 3
- MicroSD 卡（带 NOOBS）
- USB 迷你电缆
- 以太网电缆
- BME280 传感器
- 试验板
- 跳线
- 电阻
- LED

[lnk-starter-kits]: /develop/iot/iot-starter-kits/
[lnk-free-trial]: /pricing/1rmb-trial/