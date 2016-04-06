<properties 
	pageTitle="授权 Microsoft® 平滑流式处理客户端移植工具包" 
	description="了解如何为 Microsoft® 平滑流式处理客户端移植工具包授权。" 
	services="media-services" 
	documentationCenter="" 
	authors="xpouyat,vsood" 
	manager="dwrede" 
	editor=""/>

<tags
	ms.service="media-services"
	ms.date="03/07/2016"  
	wacn.date="04/05/2016"/>

#授权 Microsoft® 平滑流式处理客户端移植工具包

##概述

Microsoft 平滑流式处理客户端移植工具包（简称 **SSPK**）是经过优化的平滑流式处理客户端实现，可帮助嵌入式设备制造商、有线和移动运营商、内容服务提供商、手持设备制造商、独立软件供应商 (ISV) 和解决方案提供商创建产品和服务，用于流式传输平滑流式处理格式的自适应流式处理内容。SSPK 是平滑流客户端的与设备和平台无关的实现，许可接受方可将它移植到任何设备和平台。

下面是一个高级体系结构。IIS 平滑流式处理移植工具包框架是 Microsoft 提供的平滑流式处理客户端实现，包含用于播放平滑流内容的所有核心逻辑。特定设备或平台的合作伙伴可以通过实现相应的接口来移植该框架。

![SSPK](./media/media-services-sspk/sspk-arch.png)

##说明

经过授权的 SSPK 可以提供优异的商业价值。SSPK 许可证为行业提供：

- 采用 C++ 的平滑流式处理移植工具包源代码 
  - 实现平滑流式处理客户端功能
  - 添加格式解析、启发、缓冲逻辑，等等
- 播放器应用程序 API 
  -	可与媒体播放器应用程序交互的编程接口
- 平台抽象层 (PAL) 接口 
  -	可与操作系统（线程、套接字）交互的编程接口
- 硬件抽象层 (HAL) 接口 
  -	可与硬件 A/V 解码器（解码、绘制）交互的编程接口
- 数字权限管理 (DRM) 接口 
  -	可通过 DRM 抽象层 (DAL) 处理 DRM 的编程接口
  -	Microsoft PlayReady 移植工具包是单独发售的，但可通过此接口集成。有关 Microsoft PlayReady 设备许可的详细信息，请单击[此处](http://www.microsoft.com/playready/licensing/device_technology.mspx#pddipdl)。
- 实现示例 
  -	适用于 Linux 的 PAL 实现示例
  -	适用于 GStreamer 的 HAL 实现示例

##许可选项

Microsoft 平滑流式处理客户端移植工具包根据两份不同的许可协议提供给受证人：一份用于开发平滑流式处理客户端中期产品，另一份用于将平滑流式处理客户端最终产品分发给最终用户。
 
- 需要使用源代码移植工具包开发中期产品的芯片组制造商、系统集成商或独立软件供应商 (ISV)，应该执行 Microsoft 平滑流式处理客户端移植工具包**中期产品许可证**。
- 需要拥有对最终用户分发平滑流式处理客户端最终产品的权限的设备制造商或 ISV 应该执行 Microsoft 平滑流式处理客户端移植工具包**最终产品许可证**。

###Microsoft 平滑流式处理客户端移植工具包中期产品许可证

Microsoft 根据此许可证提供平滑流式处理客户端移植工具包和所需的知识产权，使客户能够开发平滑流式处理客户端中期产品并分发给其他可分发平滑流式处理客户端最终产品的平滑流式处理客户端移植工具包设备受证者。

####费用结构

支付 50,000 美元的一次性许可费即可访问平滑流式处理客户端移植工具包。

###Microsoft 平滑流式处理客户端移植工具包最终产品许可证

Microsoft 根据此许可证提供全部所需的知识产权，以便从其他平滑流式处理客户端移植工具包受证者接收平滑流式处理客户端中期产品，并将公司自有品牌的平滑流式处理客户端最终产品分发给最终用户。

####费用结构

平滑流式处理客户端最终产品根据如下所述的特许权使用费模式提供：

- 交付的每个设备实现支付 0.10 美元
- 每年的特许权使用费上限为 50,000 美元
- 每年的前 10,000 个设备实现无需支付特许权使用费 

##许可过程和 SSPK 访问权限

若要咨询许可事宜，请向 [sspkinfo@microsoft.com](mailto:sspkinfo@microsoft.com) 发送电子邮件。

已注册的中期受证者可以访问 [SSPK 分发门户](https://microsoft.sharepoint.com/teams/SSPKDOWNLOAD/)。

中期和最终 SSPK 受证者可以将技术问题提交到 [smoothpk@microsoft.com](mailto:smoothpk@microsoft.com)。

##Microsoft 平滑流式处理客户端中期产品协议受证人

- Adroit Business Solutions, Inc
- Advanced Digital Broadcast SA
- AirTies Kablosuz Iletism Sanayive Dis Ticaret A.S.
- Albis Technologies Ltd.
- Alticast Corporation
- Amazon Digital Services, Inc.
- Amlogic, Co., Ltd.
- AVC Multimedia Software Co., Ltd.
- EchoStar Purchasing Corporation
- Enseo, Inc.
- Guangdong OPPO Mobile Telecommunications Corp., Ltd.
- HANDAN BroadInfoCom Co., Ltd.
- Infomir GMBH
- Inside Secure
- Irdeto USA Inc.
- Liberty Global Services BV
- MediaTek Inc.
- MStar Co, Ltd
- Nintendo Co., Ltd.
- OpenTV, Inc.
- Research In Motion Limited
- Saffron Digital Limited
- Sichuan Changhong Electric Co., Ltd
- Tatung Technology Inc.
- Vestel Elektronik Sanayi ve Ticaret A.S.
- VisualOn, Inc.
- ZTE Corporation

##Microsoft 平滑流式处理客户端最终产品协议受证人

- Advanced Digital Broadcast SA
- AirTies Kablosuz Iletism Sanayive Dis Ticaret A.S.
- Albis Technologies Ltd.
- Amazon Digital Services, Inc.
- AmTRAN Technology Co., Ltd.
- Arcadyan Technology Corporation
- ATMACA ELEKTRONİK SAN.VE TİC.A.Ş
- British Sky Broadcasting Limited
- CastPal Technology Inc., Shenzhen
- Compal Electronics, Inc.
- Dongguan Digital AV Technology Corp., Ltd.
- EchoStar Purchasing Corporation
- Enseo, Inc.
- Filmflex Movies Limited
- Guangdong OPPO Mobile Telecommunications Corp., Ltd.
- HANDAN BroadInfoCom Co., Ltd.
- Hisense International Co., Ltd
- Homecast Co.,Ltd
- Hon Hai Precision Industry Co., Ltd.
- Infomir GMBH
- Kaonmedia Co., Ltd.
- KDDI Corporation
- Nintendo Co., Ltd.
- Orange SA
- PIXELA Corporation
- Saffron Digital Limited
- Sagemcom Broadband SAS
- Sharp Corporation
- Shenzhen Coship Electronics CO., LTD
- Shenzhen Jiuzhou Electric Co.,Ltd
- Shenzhen Skyworth Digital Technology Co., Ltd
- Sichuan Changhong Electric Co., Ltd.
- Sky Deutschland Fernsehen GmbH & Co. KG
- SmarDTV S.A.
- SoftAtHome
- TCL Overseas Marketing (Macao Commercial Offshore) Limited
- Technicolor Delivery Technologies, SAS
- Toshiba Lifestyle Products & Services Corporation
- Virgin Media Limited
- VIZIO, Inc.
- Wistron Corporation
- WOOX Innovations Limited
- ZTE Corporation

<!---HONumber=Mooncake_0328_2016-->