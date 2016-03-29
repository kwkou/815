<properties
	pageTitle="OS 平台和硬件兼容性 | Microsoft Azure"
	description="总结 IoT 设备 SDK 与 OS 平台和设备硬件的兼容性。"
	services="iot-hub"
	documentationCenter=""
	authors="hegate"
	manager="timlt"
	editor=""/>

<tags
     ms.service="iot-hub"
     ms.date="02/03/2016"
     wacn.date="03/28/2016"/>

# OS 平台和硬件与设备 SDK 的兼容性

本文档介绍 SDK 与不同 OS 平台的兼容性，以及 [Microsoft Azure IoT 认证计划](#microsoft-azure-certified-for-iot)中包含的特定设备配置。如果你已经有一个设备，请查看该计划中包含的设备的列表以找到有关兼容性的特定于设备的信息。如果你不确定要使用的设备，请查看 [OS 平台和库](#os-platforms)兼容性部分。


## OS 平台

Azure IoT 库在以下操作系统平台上进行了测试：


|Linux/Unix OS 平台 | 版本|
|:---------------|:------------:|
|Debian Linux| 7\.5|
|Fedora Linux|20|
|Raspbian Linux| 3\.18 |
|Ubuntu Linux| 14\.04 |
|Yocto Linux|2\.1 |

|Windows OS 平台 | 版本|
|:---------------|:------------:|
|Windows 桌面| 7、8、10 |
|Windows IoT Core| 10 |
|Windows Server| 2012 R2|

|其他平台 | 版本|
|:---------------|:------------:|
|mbed | 2\.0 |
|TI-RTOS | 2\.x |



## C 库

[适用于 C 的 Microsoft Azure IoT 设备 SDK](https://github.com/Azure/azure-iot-sdks/blob/master/c/readme.md) 在以下配置上进行了测试：

|OS 平台| 版本|协议|
|:---------|:----------:|:----------:|
|Debian Linux| 7\.5 | HTTPS、AMQP、MQTT |
|Fedora Linux| 20 | HTTPS、AMQP、MQTT |
|mbed OS| 2\.0 | HTTPS、AMQP |
|TI-RTOS| 2\.x | HTTPS |
|Ubuntu Linux| 14\.04 | HTTPS、AMQP、MQTT |
|Windows 桌面| 7、8、10 | HTTPS、AMPQ、MQTT |
|Yocto Linux|2\.1 | HTTPS、AMQP|



## Node.js 库

[适用于 Node.js 的 Microsoft Azure IoT 设备 SDK](https://github.com/Azure/azure-iot-sdks/blob/master/node/device/readme.md) 在以下配置上进行了测试：


|运行时| 版本|协议|
|:---------|:----------:|:----:|
|Node.js| 4\.1.0 | HTTPS|



## Java 库

[适用于 Java 的 Microsoft Azure IoT 设备 SDK](https://github.com/Azure/azure-iot-sdks/blob/master/java/device/readme.md) 在以下配置上进行了测试：

|运行时| 版本|协议|
|:---------|:----------:|----|
|Java SE (Windows)| 1\.7 | HTTPS、AMQP |
|Java SE (Linux)| 1\.8 | HTTPS、AMQP|

适用于 Java 的 Microsoft Azure IoT 服务 SDK 在以下配置上进行了测试：

|运行时| 版本|协议|
|:---------|:----------:|:-----|
|Java SE| 1\.8 | HTTPS、AMQP |


## CSharp

[适用于 .NET 的 Microsoft Azure IoT 设备 SDK](https://github.com/Azure/azure-iot-sdks/blob/master/csharp/device/readme.md) 在以下配置上进行了测试：

|OS 平台| 版本|协议|
|:---------|:----------:|:----------:|
|Windows 桌面| 7、8、10 | HTTPS、AMPQ|
|Windows IoT Core|10 | HTTPS|

托管代理代码需要 Microsoft .NET Framework 4.5


## Microsoft Azure IoT 认证

**Microsoft Azure IoT 认证**是一种合作伙伴计划，该计划将更广泛的 IoT 生态系统与 Microsoft Azure 相连接，以便开发人员和架构师了解兼容性方案。具体而言，它提供 OS/设备组合的受信任列表来帮助快速开始 IoT 项目 — 无论你是处于概念证明还是试验阶段。借助认证的设备和操作系统组合，IoT 项目可以快速开始，只需较少的工作和自定义便可确保设备与 [Azure IoT 套件][lnk-iot-suite] 和 Azure IoT 中心兼容。

## IoT 认证设备

**IoT 认证**设备测试了与 Azure IoT SDK 的兼容性，已准备好在 IoT 应用程序中使用。具体而言，我们基于操作系统平台和代码语言确定兼容性。

#### 设备列表

每个设备都进行了认证，可在设备制造商所选择的 OS 和语言中与我们的 SDK 结合使用。例如，BeagleBone Black 可使用我们的 C、Javascript 和 Java 语言在 Debian 上工作。这意味着开发人员可以在特定设备上采用其中任何语言和 OS 组合构建应用程序。

|设备| 经测试的 OS |语言|
|:---------|:----------|:----------|
|[AAEON ACP-1104](http://www.aaeon.com/en/p/infotainment-multi-touch-panel-solutions-1104/) |Windows 10 | C#|
|[AAEON GENE-BT05](http://www.aaeon.com/en/p/3-and-half-inches-subcompact-boards-gene-bt05/) |Windows 10 | C#|
|[AAEON PICO-BT01](http://www.aaeon.com/en/p/pico-itx-boards-pico-bt01) |Windows 10 | C#|
|[AAEON UP](http://www.up-board.org/) |Windows 10 | C#|
|[Acme Systems Arietta G25](http://www.acmesystems.it/arietta) |Debian | C|
|[ADLINK MXE-202i](http://www.adlinktech.com/PD/web/PD_detail.php?pid=1589) |Wind River | Javascript|
|[ADLINK MXE-5400](http://www.adlinktech.com/PD/web/PD_detail.php?pid=1318) |Windows 10 | C#|
|[Advantech Co.，ARK-2121L](http://www.advantech.com/products/ark-2000_series_embedded_box_pcs/ark-2121l/mod_dd092808-0832-44bc-b38a-945eb7e016bd) |Windows 10 | C#|
|[Advantech Co.，ARK-1123C](http://www.advantech.com/products/92d96fda-cdd3-409d-aae5-2e516c0f1b01/ark-1123c/mod_0b91165c-aa8c-485d-8d25-fde6f88f4873) |Windows 10 | C#|
|[Advantech Co.，LTD UNO-1372G](http://www.advantech.com/products/gf-bvl2/uno-1372g/mod_8e63b3c9-b606-4725-a1af-94fccb98bb1a) |[Windows 10 | C#|
|[Advantech Co.，TREK-674](http://www.advantech.com/products/1-2jsj5t/trek-674/mod_88a737dd-819b-4c8e-8f2e-2bb75b04619b) |Windows 10 | C#|
|[Advantech Co.，UTX-3115](http://www.advantech.com/products/bda911fe-28bc-4171-aed3-67f76f6a12c8/utx-3115/mod_fa00d5cd-7d2b-430b-8983-c232bfb9f315) |Windows 10 | C#|
|[Arduino MKR1000](https://www.arduino.cc/en/Main/ArduinoMKR1000) |Arduino IDE | C|
|[Arduino Zero](https://www.arduino.cc/en/Guide/ArduinoZero) |Arduino IDE | C|
|[Arrow DragonBoard 410c](http://partners.arrow.com/campaigns-na/qualcomm/dragonboard-410c/) |Windows 10 IoT Core | C#|
|[Axiomtek ICO300](http://www.axiomtek.com/Default.aspx?MenuId=Products&FunctionId=ProductView&ItemId=1151) |Windows 10 | C#|
|[BeagleBone Black](http://beagleboard.org/black) | Debian | C、Javascript、Java|
|[BeagleBone Green](http://beagleboard.org/green) |Debian | C、Javascript、Java|
|[ComponentSoft RFID Tunnel](http://www.componentsoft.com/) |Windows 10 | C#|
|[Dell 边缘网关 5000 系列](http://www.dell.com/IoTgateway) |Ubuntu | Java|
|[e-con Systems Almach](http://www.e-consystems.com/DM3730-development-board.asp) |Linux Yocto | C|
|[e-con Systems Ankaa](http://www.e-consystems.com/iMX6-development-board.asp) |Ubuntu | C|
|[嵌入式系统 LogicMachine 系列](http://openrb.com/products/) |自定义 Linux | C|
|[Freescale FRDM K64](http://www.freescale.com/products/arm-processors/kinetis-cortex-m/k-series/k6x-ethernet-mcus/freescale-freedom-development-platform-for-kinetis-k64-k63-and-k24-mcus:FRDM-K64F) |mbed 2.0 | C|
|[HPE Edgeline EL10](http://www8.hp.com/h20195/v2/GetPDF.aspx/c04884747.pdf) |Windows 10 | C#|
|[HPE Edgeline EL20](http://www8.hp.com/h20195/v2/GetPDF.aspx/c04884769.pdf) |Windows 10 | C#|
|[IEI ICECARE-10W](http://www.ieimobile.com/index.php?option=com_content&view=article&id=222&Itemid=21) |Windows 10 | C#|
|[IEI DRPC-120](http://www.ieiworld.com/product_groups/industrial/content.aspx?gid=09049552811981014603&cid=0D182494345754583862&id=0E318374091597499543#.VqW3Q_l97Dd) |Windows 10 | C#|
|[IEI IVS-100-BT](http://tw.ieiworld.com/product_groups/industrial/content.aspx?gid=09049552811981014603&cid=0F202412454715193114&id=0F202496627608256517#.VqH1hvl97Dc) |Windows 10 | C#|
|[Ilevia Eve Raspberry](http://www.ilevia.com/overview/) |Debian | C|
|[Intel Edison](http://www.intel.com/content/www/us/en/do-it-yourself/edison.html) |Yocto | C、Javascript|
|[Libelium Meshlium Xtreme](http://www.libelium.com/products/meshlium/) |Debian | Java|
|[Minnowboard Max](http://www.minnowboard.org/meet-minnowboard-max/) |Windows 7、8、10 | C#|
|[NEXCOM NISE 50C](http://www.nexcom.com/Products/industrial-computing-solutions/industrial-fanless-computer/atom-compact/fanless-computer-nise-50c) |Windows 10 IoT Core | C#|
|[Raspberry Pi 2](https://www.raspberrypi.org/products/raspberry-pi-2-model-b/) | Raspbian | C、Javascript、Java |
|[Raspberry Pi 2](https://www.raspberrypi.org/products/raspberry-pi-2-model-b/) | Windows 10 IoT Core| C、Javascript、C#|
|[Samsung ARTIK](http://developer.samsung.com/artik) |Fedora | C|
|[SOTEC CloudPlug](http://cloudplug.info/) |YOCTO | C|
|[TI CC3200](http://www.ti.com/product/cc3200) |TI-RTOS 2.x | C|
|[Toradex Apalis iMX6](https://www.toradex.com/computer-on-modules/apalis-arm-family/freescale-imx-6) |Linux Angstrom(Yocto) | Javascript、Java|
|[Toradex Apalis T30](https://www.toradex.com/computer-on-modules/apalis-arm-family/nvidia-tegra-3) |Linux Angstrom(Yocto) | Javascript、Java|
|[Toradex Colibri iMX6](https://www.toradex.com/computer-on-modules/colibri-arm-family/freescale-imx6) |Linux Angstrom(Yocto) | Javascript、Java|
|[Toradex Colibri T20](https://www.toradex.com/computer-on-modules/colibri-arm-family/nvidia-tegra-2) |Linux Angstrom(Yocto) | Java|
|[Toradex Colibri T30](https://www.toradex.com/computer-on-modules/colibri-arm-family/nvidia-tegra-3) |Windows 10 IoT Core | C#|
|[Toradex Colibri VF61](https://www.toradex.com/computer-on-modules/colibri-arm-family/freescale-vybrid-vf6xx) |Linux Angstrom(Yocto) | Javascript、Java|
|[Trex NGP](http://www.trex.com.tr/en/donanim_dcasngp8739_73.php) |Windows 10 | C#|
|[Trueverit V4](http://www.trueverit.com/) |自定义 Linux | C|
|[USISH EDA8909](http://www.usish.com/) |Windows 10 | C#|

[开始使用这些设备](/develop/iot/get-started/)或访问我们的 GitHub [存储库](https://github.com/Azure/azure-iot-sdks)并按照语言搜索设备文档。

## 后续步骤

了解有关使用 [IoT 认证设备](http://azure.com/iotdev)开发解决方案的详细信息。


[lnk-iot-suite]: /documentation/suites/iot-suite/

<!---HONumber=Mooncake_0321_2016-->