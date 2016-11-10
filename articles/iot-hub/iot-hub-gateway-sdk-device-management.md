<properties
	pageTitle="使用网关 SDK 进行设备管理 | Azure"
	description="本 Azure IoT 中心网关 SDK 演练演示如何在使用网关 SDK 时实施设备管理"
	services="iot-hub"
	documentationCenter=""
	authors="dominicbetts"
	manager="timlt"
	editor=""/>

<tags
     ms.service="iot-hub"
     ms.devlang="cpp"
     ms.topic="article"
     ms.tgt_pltfrm="na"
     ms.workload="na"
     ms.date="09/12/2016"
     ms.author="dobett"
     wacn.date="10/10/2016"/>


# IoT 网关 SDK (beta) – 使用网关 SDK 进行设备管理

本教程说明如何将 IoT 中心的[设备管理][lnk-device-management]功能与 [Azure IoT 中心网关 SDK][lnk-gateway-sdk] 配合使用。本教程使用 Intel Edison 计算模块来托管网关，该网关中的模拟设备可将遥测数据发送到 IoT 中心。你将使用 IoT 中心的设备管理功能，在 Edison 主板上执行远程固件更新。

本教程的内容包括：

- **体系结构**：有关设备管理示例的重要体系结构信息。

- **生成并运行**：生成并运行示例所需的步骤。

## 体系结构

在本教程中，你将预配一块 Intel Edison 主板，充当已连接到 IoT 中心的 IoT 设备网关。你还将配置该 Edison 主板，以便通过 IoT 中心对它进行管理。下图显示了要在本教程中生成的解决方案的关键元素：

![网关和设备管理体系结构][img-architecture]

### 设备

有三个 IoT 设备将连接到本解决方案中的 IoT 中心：

- Edison 主板是名为 **GW-device** 的设备。在本教程中，可以使用 IoT 中心的设备管理功能来更新此物理设备的固件。

- 两个模拟设备（**GW-ble1-demo** 和 **GW-ble2-demo**）。这些设备模拟蓝牙低功耗设备，而这些蓝牙设备将通过网关连接到 IoT 中心，并将温度遥测数据发送到中心。

### 网关软件

网关软件在 Edison 主板上以服务形式运行。两个模拟设备将生成温度遥测数据。映射模块将这些模拟设备映射到已在 IoT 中心注册的设备，HTTP 模块处理与 IoT 中心终结点的通信。[IoT Gateway SDK – send device-to-cloud messages with a simulated device][lnk-gateway-scenario]（IoT 网关 SDK – 使用模拟设备发送设备到云的消息）一文详细介绍了此方案。

### 设备管理客户端

[设备管理客户端][lnk-device-management]在 Edison 主板上以服务形式运行。这样，你便可以在 Edison 主板上运行远程作业，例如[固件更新][lnk-dm-jobs]、重新启动和配置更新，以及查询设备属性。在本教程中，设备管理客户端将接收并处理在 Edison 主板上执行固件更新的请求。

### IoT 中心

[Azure IoT 中心][lnk-iot-hub]是一项完全托管的服务，可在数百万个 IoT 设备和一个解决方案后端之间实现安全可靠的双向通信。在本教程中，IoT 中心：

- 可让三个设备连接到云。
- 接收网关中模拟设备生成的遥测数据。
- 可让你将固件更新请求发送到 Edison 主板。
- 监视固件更新作业的进度。

### 设备管理示例 UI

[设备管理示例 UI][lnk-dm-sample-ui] 是一个示例 Web 应用程序，可让你在交互式 Web UI 中使用 IoT 中心的设备管理功能。例如，你可以使用示例 UI 来查询已连接到 IoT 中心的设备，并将固件更新作业提交到设备。在本教程中，可以使用示例 UI 将固件更新作业提交到 Edison 主板，并监视该作业的完成进度。

## 生成并运行

若要运行本示例，必须生成 Edison 主板的自定义映像，其中包含 IoT 中心网关软件和设备管理客户端。

在开始之前，应确保可以将 Edison 主板连接到无线网络。若要设置 Edison 主板，需要将其连接到主计算机。然后，使用主计算机在 Edison 主板中刷入你创建的自定义映像。Intel 提供了一系列的入门指南，其中包括适用于以下操作系统的指南：

- [Get Started with the Intel Edison Development Board on Windows 64-bit][lnk-setup-win64]（Windows 64 位上的 Intel Edison 开发板入门）。
- [Get Started with the Intel Edison Development Board on Windows 32-bit][lnk-setup-win32]（Windows 32 位上的 Intel Edison 开发板入门）。
- [Getting Started with the Intel® Edison Board on Linux][lnk-setup-linux]（Linux 上的 Intel® Edison 主板入门）。

若要设置 Edison 主板并熟悉它，应完成所有这些“入门”文章中除最后一步外的所有步骤。

- 刷入最新的固件。你将在学习本教程的过程中更新固件，因此暂时不需要完成此步骤。
- 最后一个步骤“选择 IDE”，在当前教程中不需要这样做。

设置 Edison 主板并在主计算机上安装必要的驱动程序后，应确保可以使用串行终端连接到 Edison 主板。Intel 网站上 [Setting up a serial terminal][lnk-serial-connection]（设置串行终端）页中的链接提供了有关如何设置主机操作系统（例如 Windows 和 Linux）的说明。

此外，需要完成以下任务

- 在 Azure 订阅中[创建 IoT 中心][lnk-create-hub]。完成本教程需要用到中心的名称。如果你还没有 Azure 订阅，可以获取一个[免费帐户][lnk-free-trial]。
- 将三个设备（**GW-ble1-demo**、**GW-ble2-demo** 和 **GW-device**）添加到 IoT 中心，并记下其 ID 和设备密钥。可以使用[设备资源管理器或 iothub-explorer][lnk-explorer-tools] 工具，将这些设备添加到上一步骤中创建的 IoT 中心，并检索其密钥。可以使用其中两个设备（**GW-ble1-demo** 和 **GW-ble2-demo**）作为已连接到网关的 BLE 模拟设备，并使用另一个设备（**GW-device**）将 Edison 网关设备识别为可从 IoT 中心管理的设备管理客户端。

### 准备你的生成环境，并确认可以创建自定义映像

若要创建 Edison 主板的自定义映像，需要一个 Linux 环境。这些步骤假设你使用 Ubuntu 14.04，这是编写本文时建议使用的平台。主分区上至少要有 40 GB 的可用空间。

> [AZURE.NOTE] 生成自定义映像的脚本最多可能需要在有 4 个 CPU 核心的计算机上运行 6 小时。你可以使用有更多 CPU 核心的更强大计算机，以减少运行时间。

对于本部分中的步骤，我们参阅了以下文章：[Intel Edison Board Support Package][lnk-inteledison-bsp]（Intel Edison 主板支持包）、[Manually Building Yocto Images for the Intel Edison Board from Source][lnk-hackgnar]（从源手动生成 Intel Edison 主板的 Yocto 映像）和 [Creating a Custom Linux Kernel for the Edison (release 2.1)][lnk-shawnhymel]（创建 Edison（2.1 版）的自定义 Linux 内核）。

1. 登录到 Ubuntu 14.04 计算机并在主文件夹中执行以下命令，以下载 Edison 源代码包：
    
    ```
    curl -O http://downloadmirror.intel.com/25028/eng/edison-src-ww25.5-15.tgz
    ```

2. 解压缩源代码包，然后使用以下命令在主文件夹中创建 **edison-src** 文件夹，并导航到新的 **edison-src** 文件夹：
    
    ```
    tar xfvz edison-src-ww25.5-15.tgz
    cd edison-src/
    ```

3. 安装必备组件包：
    
    ```
    sudo apt-get -y install build-essential git diffstat gawk chrpath texinfo libtool gcc-multilib libsdl1.2-dev u-boot-tools
    ```

4. 在 **edison-src** 文件夹中创建两个新目录：
    
    ```
    mkdir bitbake_download_dir
    mkdir bitbake_sstate_dir 
    ```

5. 运行以下脚本，以下载生成环境。如果有 4 个以上的 CPU 核心，请将 **--parallel_make** 和 **--bb\_number\_thread** 脚本参数设置为可用的核心数：
    
    ```
    ./meta-intel-edison/setup.sh --dl_dir=bitbake_download_dir  --sstate_dir=bitbake_sstate_dir 
    ```

6. 更新 Paho git 存储库的位置。编辑 **~/edison-src/out/linux64/poky/meta-intel-iot-middleware/recipes-connectivity/paho-mqtt/paho-mqtt\_3.1.bb** 文件并将 URL `git://git.eclipse.org/gitroot/paho/org.eclipse.paho.mqtt.c.git/` 替换为 `git://github.com/eclipse/paho.mqtt.c.git`。

7. 使用以下命令初始化你的生成环境：
    
    ```
    cd out/linux64/
    source poky/oe-init-build-env
    ```

8. 使用以下命令生成自定义映像。在有 4 个 CPU 核心的计算机上，首次运行此命令最多可能需要 6 小时。添加自己的自定义项之后，后续重新生成会快很多：
    
    ```
    bitbake edison-image
    ```

9. 运行以下命令来完成生成：
    
    ```
    cd ~/edison-src/
    ./meta-intel-edison/utils/flash/postBuild.sh ./out/linux64/build/
    ```

需要刷入 Edison 主板的文件现在出现在 **~/edison-src/out/linux64/build/toFlash/** 文件夹中。

### 将网关 SDK 添加到自定义映像

本部分中的步骤将引导完成将网关 SDK 添加到自定义映像的过程，以便 Edison 在启动时可充当 IoT 网关。在本教程中，网关包含可模拟两个蓝牙低功耗 (BLE) 设备的模块，这些设备将生成要转发到 IoT 中心的网关温度遥测数据。

在上一部分中用于生成 Edison 映像的同一台 Ubuntu 14.04 计算机上完成以下步骤。

1. 将网关 SDK 克隆到主文件夹：
    
    ```
    cd ~
    git clone https://github.com/Azure/azure-iot-gateway-sdk.git --recursive
    ```

2. 配置该网关，以连接到 IoT 中心并模拟两个设备。编辑 **~/azure-iot-gateway-sdk/samples/simulated\_device\_cloud\_upload/src/simulated\_device\_cloud\_upload\_intel\_edison.json** 文件，并将 **IoTHubName**、**IoTHubSuffix**、**deviceID** 和 **deviceKey** 占位符替换为配置 IoT 中心时所记下的值。使用前面创建的 **GW-ble1-demo** 和 **GW-ble2-demo** 设备。

3. 创建一个文件夹来包含新脚本：
    
    ```
    mkdir ~/edison-src/meta-intel-edison/meta-intel-edison-distro/recipes-support/azure-iot-field-gateway-sdk
    ```

4. 将名为 **azure-iot-field-gateway-sdk.bb** 的脚本文件从 **~/azure-iot-gateway-sdk/samples/simulated\_device\_cloud\_upload/src/** 文件夹复制到刚创建的 **~/edison-src/meta-intel-edison/meta-intel-edison-distro/recipes-support/azure-iot-field-gateway-sdk/** 文件夹。编辑该文件，并将两处 `<<userName>>` 替换为当前用户名。

5. 编辑 **~/edison-src/meta-intel-edison/meta-intel-edison-distro/recipes-core/images/edison-image.bb**，为新脚本添加一个条目。在该文件的末尾添加以下行：
    
    ```
    IMAGE_INSTALL += "azure-iot-field-gateway-sdk"
    ```

6. 现在，可以通过运行 **bitbake** 命令来创建新的脚本以测试更改：
    
    ```
    cd ~/edison-src/out/linux64/
    source poky/oe-init-build-env
    bitbake azure-iot-field-gateway-sdk
    ```

### 将 Azure IoT 中心设备管理客户端添加到自定义映像

本部分中的步骤引导你完成将 IoT 中心设备管理客户端添加到自定义映像的过程，以便从 IoT 中心管理 Edison 网关设备。在本教程中，设备管理客户端包含用于在 Edison 网关设备上启用固件更新的示例代码。

在上一部分中所用的同一台 Ubuntu 14.04 计算机上完成以下步骤，将 Edison 网关添加到 Edison 映像。

1. 将 IoT 中心 SDK 存储库的 **dmpreview** 分支克隆到主文件夹：
    
    ```
    cd ~
    git clone https://github.com/Azure/azure-iot-sdks -b dmpreview --recursive
    ```

2. 创建以下文件夹以包含新脚本：
    
    ```
    mkdir ~/edison-src/meta-intel-edison/meta-intel-edison-distro/recipes-support/iotdm-edison-sample
    mkdir ~/edison-src/meta-intel-edison/meta-intel-edison-distro/recipes-support/iotdm-edison-sample/files
    ```

3. 将 **iotdm-edison-sample.bb** 文件从 **~/azure-iot-sdks/c/iotdm\_client/samples/iotdm\_edison\_sample/bitbake/** 文件夹复制到 **~/edison-src/meta-intel-edison/meta-intel-edison-distro/recipes-support/iotdm-edison-sample** 文件夹。

4. 编辑文件 **~/edison-src/meta-intel-edison/meta-intel-edison-distro/recipes-support/iotdm-edison-sample/iotdm-edison-sample.bb**，并将 `-Duse_http:BOOL=OFF` 替换为 `-Duse_http:BOOL=ON`。
4. 将文件 **iotdm\_edison\_sample.service** 从 **~/azure-iot-sdks/c/iotdm\_client/samples/iotdm\_edison\_sample/bitbake/** 文件夹复制到 **~/edison-src/meta-intel-edison/meta-intel-edison-distro/recipes-support/iotdm-edison-sample/files** 文件夹。

5. 编辑 **~/edison-src/meta-intel-edison/meta-intel-edison-distro/recipes-core/images/edison-image.bb** 文件，为新脚本添加一个条目。在该文件的末尾添加以下行：
    
    ```
    IMAGE_INSTALL += "iotdm-edison-sample"
    ```

6. 网关 SDK 和设备管理客户端共享一些库，因此需编辑 **~/edison-src/out/linux64/poky/meta/classes/sstate.bbclass** 文件。在此文件的末尾添加以下行。请务必将 `<your user>` 替换为当前用户名：
    
    ```
    SSTATE_DUPWHITELIST += "/home/<your user>/edison-src/out/linux64/build/tmp/sysroots/edison/usr/lib/libaziotsharedutil.a"
    SSTATE_DUPWHITELIST += "/home/<your user>/edison-src/out/linux64/build/tmp/sysroots/edison/usr/include/azureiot"
    ```

7. 编辑 **~/edison-src/meta-intel-edison/meta-intel-edison-distro/recipes-connectivity/wpa\_supplicant/wpa-supplicant/wpa\_supplicant.conf-sane** 文件并在文件末尾添加以下行，配置为在 Edison 主板上自动启动 WiFi。请务必将 `<your wifi ssid>` 和 `<your wifi password>` 替换为 WiFi 网络的正确值：
    
    ```
    network={
      ssid="<your wifi ssid>"
      key_mgmt=WPA-PSK
      pairwise=CCMP TKIP
      group=CCMP TKIP WEP104 WEP40
      eap=TTLS PEAP TLS
      psk="<your wifi password>"
    }
    ```

8. 现在可以创建 Edison 主板的映像，其中包含网关 SDK 和设备管理客户端。**bitbake** 命令的运行速度大幅提升，因为它只需创建新脚本并将其添加到映像：
    
    ```
    cd ~/edison-src/out/linux64/
    source poky/oe-init-build-env
    bitbake edison-image
    ```

9. 运行以下命令来完成生成：
  
    ```
    cd ~/edison-src/
    ./meta-intel-edison/utils/flash/postBuild.sh ./out/linux64/build/
    ```

刷新 Edison 主板所需的文件现位于 **~/edison-src/out/linux64/build/toFlash/** 文件夹中。

### 使用自定义映像刷新 Intel Edison

现在可以使用包含 IoT 中心设备管理客户端和网关软件的自定义映像来刷新 Intel Edison。

在构建自定义映像所用的 Ubuntu 计算机上，将 **toFlash** 文件夹中的文件复制到用 USB 电缆连接到 Edison 主板的计算机上。

如果在 Windows 计算机上通过 USB 电缆连接到 Edison，则应运行 **flashall.bat** 脚本来刷新 Edison。如果在 Linux 计算机上通过 USB 电缆连接到 Edison，则应运行 **flashall.bat** 脚本来刷新。

刷新完成后，请在主机上使用[串行连接][lnk-serial-connection]来连接到 Edison，并以 **root** 身份登录。应该验证你的 Edison 主板现在是否已连接到 WiFi 网络。

### 运行示例

必须在 Edison 主板上配置设备管理客户端，才能作为 **GW-device** 设备连接到 IoT 中心。使用文本编辑器（如 **vi** 或 **nano**），在 Edison 的 /home/root 文件夹中创建名为 **.cs** 的文件。此文件只能包含 **GW-device** 的连接字符串。如果先前未记下此连接字符串，可使用[设备资源管理器或 iothub-explorer][lnk-explorer-tools] 工具，在 IoT 中心设备注册表中检索此设备连接字符串。

创建 **.cs** 文件后，请使用以下命令重启 Edison 主板：

```
reboot -h now
```

重启 Edison 后，检查设备管理和网关服务是否以“正常”状态启动：

```
[  OK  ] Started Daemon to receive the wpa_supplicant event.
         Starting PulseAudio Sound System...
[  OK  ] Started WPA supplicant service.
[  OK  ] Started Azure IoT DM as a service..
[  OK  ] Started Azure Iot Gateway as a service..
         Stopping Daemon to receive the wpa_supplicant event...
[  OK  ] Stopped Daemon to receive the wpa_supplicant event.

```

若要排查 IoT 网关服务的任何问题，请在 Edison 上查看 **/deviceCloudUploadGatewaylog.log** 文件中的条目。也可以使用以下命令来查看此服务的任何输出：

```
systemctl status simulated_device_cloud_upload.service
```

若要排查 IoT 设备管理服务的任何问题，请使用以下命令查看任何输出：

```
systemctl status iotdm_edison_sample.service
```

### 启动固件更新作业

IoT 设备管理服务请求的 Edison 固件更新通常会从某个 URL 下载一个包含固件的 zip 文件。若要简化本教程，可将 zip 文件手动复制到 Edison 主板，然后在请求更新时使用 **file://** URL 而不是 **http://** URL。

同样地，为了简化本教程，固件更新将重新应用相同的固件映像，而不是使用全新的映像。你可以看到此映像正在应用到 Edison 主板。

创建名为 **edison.zip** 的 zip 文件，其中包含创建自定义映像所用的 Ubuntu 计算机上 **toFlash** 文件夹中的所有文件和子文件夹。确保 **toFlash** 文件夹的文件位于 zip 文件的根目录中。使用 SCP（若使用 Putty，则为 PSCP）等工具将 **edison.zip** 文件复制到 Edison 主板的 /home/root 文件夹中。

若要提交固件更新作业并监视进度，请使用 Node.js [设备管理示例 UI][lnk-dm-sample-ui]。可在 Windows 或 Linux 上运行本示例，它需要 [Node.js][lnk-nodejs] 6.1.0 或更高版本。若要在台式机上检索、生成和运行设备管理示例 UI，请遵循以下步骤：

1. 打开**命令提示符**。

2. 键入 `node --version`，确认已安装 Node.js 6.1.0 或更高版本。

3. 通过运行以下命令来克隆 Azure IoT 设备管理 UI GitHub 存储库：

    ```
    git clone https://github.com/Azure/azure-iot-device-management.git
    ```
    
4. 在 Azure IoT 设备管理 UI 存储库的克隆副本的根文件夹中，运行以下命令来检索相关程序包：

    ```
    npm install
    ```

5. npm install 命令完成后，运行以下命令以生成代码：

    ```
    npm run build
    ```

6. 使用文本编辑器在克隆文件夹的根目录中打开 user-config.json 文件。将“&lt;YOUR CONNECTION STRING HERE&gt;”文本替换为你的 IoT 中心连接字符串。Azure [门户预览][lnk-azure-portal]中提供此连接字符串。

7. 在命令提示符处，运行以下命令启动设备管理 UX 应用：

    ```
    npm run start
    ```

8. 当命令提示符报告“服务已启动”时，打开 Web 浏览器，然后通过以下 URL 导航到设备管理应用查看你的设备：http://127.0.0.1:3003。

    ![设备管理 UI][img-dm-ui]

9. 选择“GW-device”设备，在“设备作业”下拉列表中选择“固件更新”，然后单击“启动”。

10. 在“包 URI”字段中，输入 **file:///home/root/edison.zip** 以使用之前复制到 Edison 主板的映像文件。依次单击“提交”、“是”、“作业历史记录”链接，查看正在运行的新父级和子作业：

    ![作业历史记录链接][img-history-link]

11. 几分钟后，你将在连接到 Edison 主板的序列终端中，看到 Edison 重新启动并应用固件更改：

    ```
    reading ota_update.scr
    14747 bytes read in 17 ms (846.7 KiB/s)
    ## Executing script at 00100000
    === OTA update script ===
    Validating Ota package
    part find mmc 0 label:update u_part_num;
    ota drive is mmc 0:9

    Validating edison_ifwi-dbg-00-dfu.bin hash for boot0 and boot1 partitions

    fatload mmc 0:9 0x6400000 edison_ifwi-dbg-00-dfu.bin;
    reading edison_ifwi-dbg-00-dfu.bin
    ...
    ```

12. Edison 启动完毕后，请在设备管理示例 UI 中刷新页面，查看这两项固件更新作业的作业状态现在是否为“已完成”：

    ![已完成作业状态][img-job-status]

现在你已完成本教程，其中演示了如何在 Intel Edison 主板上使用 IoT 中心网关软件和设备管理客户端。在本教程中，你已：

- 创建了一个自定义 Intel Edison 映像，其中包含的 IoT 中心设备管理客户端和网关软件已设置为在 Edison 主板启动时启动。
- 配置了 Edison 主板和设备管理客户端要连接到 IoT 中心。
- 从示例 UI 启动了设备管理作业，该作业导致 Edison 主板重新启动并应用新的固件映像。

## 后续步骤

若要深入了解如何使用 IoT 中心和示例 UI 管理设备，请参阅 [Azure IoT 中心设备管理概述][lnk-device-management]一文。

如果想要深入了解网关 SDK 并体验一些代码示例，请访问 [Azure IoT Gateway SDK][lnk-gateway-sdk]（Azure IoT 网关 SDK）。

若要进一步探索 IoT 中心的功能，请参阅：

- [设计你的解决方案][lnk-design]
- [开发人员指南][lnk-devguide]
- [使用 UI 示例探索设备管理][lnk-dmui]
- [使用 Azure 门户管理 IoT 中心][lnk-portal]



<!-- Links and images -->

[img-dm-ui]: ./media/iot-hub-gateway-sdk-device-management/image1.png
[img-history-link]: ./media/iot-hub-gateway-sdk-device-management/image2.png
[img-job-status]: ./media/iot-hub-gateway-sdk-device-management/image3.png
[img-architecture]: ./media/iot-hub-gateway-sdk-device-management/architecture.png

[lnk-iot-hub]: /documentation/articles/iot-hub-what-is-iot-hub/
[lnk-device-management]: /documentation/articles/iot-hub-device-management-overview/
[lnk-gateway-sdk]: https://github.com/Azure/azure-iot-gateway-sdk/
[lnk-setup-win64]: https://software.intel.com/get-started-edison-windows
[lnk-setup-win32]: https://software.intel.com/get-started-edison-windows-32
[lnk-setup-osx]: https://software.intel.com/get-started-edison-osx
[lnk-setup-linux]: https://software.intel.com/get-started-edison-linux
[lnk-serial-connection]: https://software.intel.com/setting-up-serial-terminal-intel-edison-board
[lnk-inteledison-bsp]: http://www.intel.com/content/dam/support/us/en/documents/edison/sb/edisonbsp_ug_331188007.pdf
[lnk-hackgnar]: http://www.hackgnar.com/2016/01/manually-building-yocto-images-for.html
[lnk-shawnhymel]: http://shawnhymel.com/724/creating-a-custom-linux-kernel-for-the-edison-yocto-2-1/
[lnk-create-hub]: /documentation/articles/iot-hub-manage-through-portal/
[lnk-free-trial]: /pricing/1rmb-trial/
[lnk-explorer-tools]: https://github.com/Azure/azure-iot-sdks/blob/master/doc/manage_iot_hub.md
[lnk-nodejs]: https://nodejs.org/
[lnk-azure-portal]: https://portal.azure.cn
[lnk-dm-sample-ui]: /documentation/articles/iot-hub-device-management-ui-sample/
[lnk-gateway-physical]: /documentation/articles/iot-hub-gateway-sdk-physical-device/
[lnk-gateway-scenario]: /documentation/articles/iot-hub-linux-gateway-sdk-simulated-device/
[lnk-dm-jobs]: /documentation/articles/iot-hub-device-management-device-jobs/

[lnk-design]: /documentation/articles/iot-hub-guidance/
[lnk-devguide]: /documentation/articles/iot-hub-devguide/
[lnk-dmui]: /documentation/articles/iot-hub-device-management-ui-sample/
[lnk-portal]: /documentation/articles/iot-hub-manage-through-portal/

<!---HONumber=Mooncake_0801_2016-->