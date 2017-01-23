<properties
    pageTitle="Azure IoT 中心 IP 筛选器 | Azure"
    description="如何使用 IP 筛选阻止特定 IP 地址到 Azure IoT 中心的连接。可阻止来自单独 IP 地址或 IP 地址范围的连接。"
    services="iot-hub"
    documentationcenter=""
    author="BeatriceOltean"
    manager="timlt"
    editor="" />
<tags
    ms.assetid="f833eac3-5b5f-46a7-a47b-f4f6fc927f3f"
    ms.service="iot-hub"
    ms.devlang="na"
    ms.topic="article"
    ms.tgt_pltfrm="na"
    ms.workload="na"
    ms.date="11/14/2016"
    wacn.date="01/13/2017"
    ms.author="boltean" />  


# 使用 IP 筛选器

安全是任何基于 Azure IoT 中心的 IoT 解决方案的重要部分。有时候，在进行安全配置时，需将某些 IP 地址列入方块列表或允许列表。_IP 筛选器_ 功能用于配置相关规则，以便拒绝或接受来自特定 IPv4 地址的流量。

## 何时使用

两种特定情况下，可以使用它来阻止特定 IP 地址的 IoT 中心终结点：

- IoT 中心只应接收来自指定范围的 IP 地址的流量，拒绝其他地址的流量。例如，将 IoT 中心用于 [Azure ExpressRoute]，以便在 IoT 中心和本地基础结构之间创建专用连接。
- 需要拒绝特定 IP 地址的流量，这些地址已被 IoT 中心管理员确定为可疑地址。

## 如何应用筛选器规则

在 IoT 中心服务级别应用 IP 筛选器规则。因此，IP 筛选器规则适用于使用任意受支持协议和从设备和后端应用发出的所有连接。

与 IoT 中心的拒绝 IP 规则匹配的 IP 地址发出的任何连接尝试都会收到“未授权”401 状态代码和说明。响应消息不提及 IP 规则。

## 默认设置
默认情况下，门户中对应于 IoT 中心的“IP 筛选器”网格为空。此默认设置意味着用户的中心接受任何 IP 地址的连接。此默认设置相当于某项规则接受 0.0.0.0/0 IP 地址范围。

![][img-ip-filter-default]  


## 添加或编辑 IP 筛选器规则

添加 IP 筛选器规则时，系统会提示用户提供以下值：

- **IP 筛选器规则名称**，必须是一个唯一的、不区分大小写的字母数字字符串，最长 128 个字符。只接受 ASCII 7 位字母数字字符以及 `{'-', ':', '/', '\', '.', '+', '%', '_', '#', '*', '?', '!', '(', ')', ',', '=', '@', ';', '''}`。
- 选择“拒绝”或“接受”作为 IP 筛选器规则的“操作”。
- 在 CIDR 表示法中提供单个 IPv4 地址或 IP 地址块。例如，在 CIDR 表示法中，192.168.100.0/22 表示 1024 个从 192.168.100.0 到 192.168.103.255 的 IPv4 地址。

![][img-ip-filter-add-rule]  


保存规则后，将会出现一个提醒，通知你更新正在进行。

![][img-ip-filter-save-new-rule]  


在用户达到 10 个 IP 筛选器规则这一最大限制以后，系统会禁用“添加”选项。

可以通过双击包含规则的行来编辑现有规则。

## 删除 IP 筛选器规则

若要删除 IP 筛选器规则，请在网格中选择一条或多条规则，然后单击“删除”。

![][img-ip-filter-delete-rule]  


## IP 筛选器规则评估

IP 筛选器规则按顺序应用，与 IP 地址匹配的第一条规则决定了是采取接受操作还是拒绝操作。

例如，若要接受 192.168.100.0/22 范围内的地址并拒绝所有其他地址，则网格中的第一条规则应接受 192.168.100.0/22 这一地址范围。下一规则应使用 0.0.0.0/0 这一范围拒绝所有地址。如果在添加最后一条规则时拒绝了 0.0.0.0/0 这一范围，则相当于将默认行为更改为启用允许列表。

可以更改网格中 IP 筛选器规则的顺序，只需单击一行开始处的三个垂直点并使用拖放操作即可。

若要保存新的 IP 筛选器规则顺序，请单击“保存”。

![][img-ip-filter-rule-order]  


## 后续步骤

若要进一步探索 IoT 中心的功能，请参阅：

* [操作监视][lnk-monitor]
* [IoT 中心指标][lnk-metrics]

<!-- Images -->

[img-ip-filter-default]: ./media/iot-hub-ip-filtering/ip-filter-default.png
[img-ip-filter-add-rule]: ./media/iot-hub-ip-filtering/ip-filter-add-rule.png
[img-ip-filter-save-new-rule]: ./media/iot-hub-ip-filtering/ip-filter-save-new-rule.png
[img-ip-filter-delete-rule]: ./media/iot-hub-ip-filtering/ip-filter-delete-rule.png
[img-ip-filter-rule-order]: ./media/iot-hub-ip-filtering/ip-filter-rule-order.png


<!-- Links -->


[IoT Hub Developer Guide]: /documentation/articles/iot-hub-devguide
[Azure ExpressRoute]: /documentation/articles/expressroute-faqs/#supported-services

[lnk-monitor]: /documentation/articles/iot-hub-operations-monitoring/
[lnk-metrics]: /documentation/articles/iot-hub-metrics/

<!---HONumber=Mooncake_0109_2017-->
<!--Update_Description:update wording-->