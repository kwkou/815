<properties 
	pageTitle="媒体服务 PlayReady 许可证模板概述" 
	description="本主题概述了用于配置 PlayReady 许可证的 PlayReady 许可证模板。" 
	authors="juliako" 
	manager="dwrede" 
	editor="" 
	services="media-services" 
	documentationCenter=""/>

<tags
	ms.service="media-services"
 	ms.date="04/18/2016"  
	wacn.date="06/27/2016"/>

#媒体服务 PlayReady 许可证模板概述

Azure 媒体服务现在提供有用于传送 Microsoft PlayReady 许可证的服务。当最终用户播放器（例如 Silverlight）尝试播放受 PlayReady 保护的内容时，将向许可证交付服务发送请求以获取许可证。如果许可证服务批准了该请求，则会颁发该许可证，该许可证将发送到客户端，并可用于解密和播放指定的内容。

媒体服务还提供有可让你配置 PlayReady 许可证的 API。许可证包含在用户尝试播放受保护的内容时要由 PlayReady DRM 运行时强制实施的权限和限制。以下是你可以指定的 PlayReady 许可证限制的一些示例：

- 该许可证开始生效的日期/时间。
- 许可证过期时的日期/时间值。 
- 要在客户端的永久性存储区保存的许可证。永久性许可证通常用于允许脱机播放内容。
- 播放器必须具有的要播放你的内容的最低安全级别。 
- 音频\\视频内容的输入控件的输出保护级别。 
- 有关详细信息，请参阅 [PlayReady 符合性规则](https://www.microsoft.com/playready/licensing/compliance/)文档中的输出控件部分 (3.5)。

>[AZURE.NOTE]目前，只能配置 PlayReady 许可证（此权限是必需的）的 PlayRight。PlayRight 赋予客户端播放内容的权限。PlayRight 还允许配置特定于播放的限制。有关详细信息，请参阅 [PlayReadyPlayRight](/documentation/articles/media-services-playready-license-template-overview/#PlayReadyPlayRight)。

若要使用媒体服务配置 PlayReady 许可证，必须配置媒体服务 PlayReady 许可证模板。该模板在 XML 中定义。

下例演示了配置基本的流式处理许可证的最简单（也是最常见的）模板。使用此许可证，你的客户端便可以播放受 PlayReady 保护的内容。
	
	<?xml version="1.0" encoding="utf-8"?>
	<PlayReadyLicenseResponseTemplate xmlns:i="http://www.w3.org/2001/XMLSchema-instance" 
	                                  xmlns="http://schemas.microsoft.com/Azure/MediaServices/KeyDelivery/PlayReadyTemplate/v1">
	  <LicenseTemplates>
	    <PlayReadyLicenseTemplate>
	      <ContentKey i:type="ContentEncryptionKeyFromHeader" />
	      <PlayRight />
	    </PlayReadyLicenseTemplate>
	  </LicenseTemplates>
	</PlayReadyLicenseResponseTemplate>

XML 符合 PlayReady 许可证模板 XML 架构部分中定义的 PlayReady 许可证模板 XML 架构。

媒体服务还可定义一组可用于序列化到 XML 和从 XML 反序列化的 .NET 类。有关主类的描述，请参阅用于配置许可证模板的[媒体服务 .NET 类](/documentation/articles/media-services-playready-license-template-overview/#classes)。

有关使用 .NET 类来配置 PlayReady 许可证模板的端到端示例，请参阅[使用 PlayReady 动态加密和许可证传送服务](/documentation/articles/media-services-protect-with-drm/)。

##<a id="classes"></a>用于配置许可证模板的媒体服务 .NET 类

以下是用于配置媒体服务 PlayReady 许可证模板的主要 .NET 类。这些类映射到 [PlayReady 许可证模板 XML 架构](/documentation/articles/media-services-playready-license-template-overview/#schema)中定义的类型。

[MediaServicesLicenseTemplateSerializer](https://msdn.microsoft.com/zh-cn/library/azure/microsoft.windowsazure.mediaservices.client.contentkeyauthorization.mediaserviceslicensetemplateserializer.aspx) 类用于序列化到媒体服务许可证模板 XML 序列化和从该 XML 进行反序列化。

###PlayReadyLicenseResponseTemplate

[PlayReadyLicenseResponseTemplate](https://msdn.microsoft.com/zh-cn/library/azure/microsoft.windowsazure.mediaservices.client.contentkeyauthorization.playreadylicenseresponsetemplate.aspx) - 此类表示发送回最终用户的响应的模板。它包含一个用于许可证服务器和应用程序（可能用于自定义应用逻辑）以及一个或多个许可证模板的列表之间的自定义数据字符串的字段。

这是模板层次结构中的“顶层”类。意即响应模板包括许可证模板列表，而且这些许可证模板（直接或间接）包括所有构成要序列化的模板数据的其他类。


###PlayReadyLicenseTemplate

[PlayReadyLicenseTemplate](https://msdn.microsoft.com/zh-cn/library/azure/microsoft.windowsazure.mediaservices.client.contentkeyauthorization.playreadylicensetemplate.aspx) - 该类表示用于创建要返回给最终用户的 PlayReady 许可证的许可证模板。它包含许可证中内容密钥上的数据以及使用内容密钥时由 PlayReady DRM 运行时强制执行的任何权限或限制。


###<a id="PlayReadyPlayRight"></a>PlayReadyPlayRight

[PlayReadyPlayRight](https://msdn.microsoft.com/zh-cn/library/azure/microsoft.windowsazure.mediaservices.client.contentkeyauthorization.playreadyplayright.aspx) - 此类表示 PlayReady 许可证的 PlayRight。它会授予用户播放许可证中和 PlayRight 本身（用于播放特定策略）配置的零个或多个限制的制约内容的权限。PlayRight 上的很多策略都与输出限制有关，输出限制用于控制可以播放的内容的输出类型和使用给定输出时必须使用的任何限制。例如，如果启用了 DigitalVideoOnlyContentRestriction，DRM 运行时将只允许通过数字输出显示视频（将不允许模拟视频输出传递内容）。

>[AZURE.IMPORTANT]这些限制类型可以非常强大，但也会影响使用者体验。如果输出保护配置了太多限制，内容可能会无法在某些客户端上播放。有关详细信息，请参阅 PlayReady 符合性规则文档。

有关 Silverlight 支持的保护级别的示例，请参阅：[Silverlight 支持的输出保护](https://msdn.microsoft.com/zh-cn/library/cc838192(v=VS.95).aspx#Silverlight_Support_for_Output_Protection)。

##<a id="schema"></a>PlayReady 许可证模板 XML 架构
	
	<?xml version="1.0" encoding="utf-8"?>
	<xs:schema xmlns:tns="http://schemas.microsoft.com/Azure/MediaServices/KeyDelivery/PlayReadyTemplate/v1" xmlns:ser="http://schemas.microsoft.com/2003/10/Serialization/" elementFormDefault="qualified" targetNamespace="http://schemas.microsoft.com/Azure/MediaServices/KeyDelivery/PlayReadyTemplate/v1" xmlns:xs="http://www.w3.org/2001/XMLSchema">
	  <xs:import namespace="http://schemas.microsoft.com/2003/10/Serialization/" />
	  <xs:complexType name="AgcAndColorStripeRestriction">
	    <xs:sequence>
	      <xs:element minOccurs="0" name="ConfigurationData" type="xs:unsignedByte" />
	    </xs:sequence>
	  </xs:complexType>
	  <xs:element name="AgcAndColorStripeRestriction" nillable="true" type="tns:AgcAndColorStripeRestriction" />
	  <xs:simpleType name="ContentType">
	    <xs:restriction base="xs:string">
	      <xs:enumeration value="Unspecified" />
	      <xs:enumeration value="UltravioletDownload" />
	      <xs:enumeration value="UltravioletStreaming" />
	    </xs:restriction>
	  </xs:simpleType>
	  <xs:element name="ContentType" nillable="true" type="tns:ContentType" />
	  <xs:complexType name="ExplicitAnalogTelevisionRestriction">
	    <xs:sequence>
	      <xs:element minOccurs="0" name="BestEffort" type="xs:boolean" />
	      <xs:element minOccurs="0" name="ConfigurationData" type="xs:unsignedByte" />
	    </xs:sequence>
	  </xs:complexType>
	  <xs:element name="ExplicitAnalogTelevisionRestriction" nillable="true" type="tns:ExplicitAnalogTelevisionRestriction" />
	  <xs:complexType name="PlayReadyContentKey">
	    <xs:sequence />
	  </xs:complexType>
	  <xs:element name="PlayReadyContentKey" nillable="true" type="tns:PlayReadyContentKey" />
	  <xs:complexType name="ContentEncryptionKeyFromHeader">
	    <xs:complexContent mixed="false">
	      <xs:extension base="tns:PlayReadyContentKey">
	        <xs:sequence />
	      </xs:extension>
	    </xs:complexContent>
	  </xs:complexType>
	  <xs:element name="ContentEncryptionKeyFromHeader" nillable="true" type="tns:ContentEncryptionKeyFromHeader" />
	  <xs:complexType name="ContentEncryptionKeyFromKeyIdentifier">
	    <xs:complexContent mixed="false">
	      <xs:extension base="tns:PlayReadyContentKey">
	        <xs:sequence>
	          <xs:element minOccurs="0" name="KeyIdentifier" type="ser:guid" />
	        </xs:sequence>
	      </xs:extension>
	    </xs:complexContent>
	  </xs:complexType>
	  <xs:element name="ContentEncryptionKeyFromKeyIdentifier" nillable="true" type="tns:ContentEncryptionKeyFromKeyIdentifier" />
	  <xs:complexType name="PlayReadyLicenseResponseTemplate">
	    <xs:sequence>
	      <xs:element name="LicenseTemplates" nillable="true" type="tns:ArrayOfPlayReadyLicenseTemplate" />
	      <xs:element minOccurs="0" name="ResponseCustomData" nillable="true" type="xs:string">
	        <xs:annotation>
	          <xs:appinfo>
	            <DefaultValue EmitDefaultValue="false" xmlns="http://schemas.microsoft.com/2003/10/Serialization/" />
	          </xs:appinfo>
	        </xs:annotation>
	      </xs:element>
	    </xs:sequence>
	  </xs:complexType>
	  <xs:element name="PlayReadyLicenseResponseTemplate" nillable="true" type="tns:PlayReadyLicenseResponseTemplate" />
	  <xs:complexType name="ArrayOfPlayReadyLicenseTemplate">
	    <xs:sequence>
	      <xs:element minOccurs="0" maxOccurs="unbounded" name="PlayReadyLicenseTemplate" nillable="true" type="tns:PlayReadyLicenseTemplate" />
	    </xs:sequence>
	  </xs:complexType>
	  <xs:element name="ArrayOfPlayReadyLicenseTemplate" nillable="true" type="tns:ArrayOfPlayReadyLicenseTemplate" />
	  <xs:complexType name="PlayReadyLicenseTemplate">
	    <xs:sequence>
	      <xs:element minOccurs="0" name="AllowTestDevices" type="xs:boolean" />
	      <xs:element minOccurs="0" name="BeginDate" nillable="true" type="xs:dateTime">
	        <xs:annotation>
	          <xs:appinfo>
	            <DefaultValue EmitDefaultValue="false" xmlns="http://schemas.microsoft.com/2003/10/Serialization/" />
	          </xs:appinfo>
	        </xs:annotation>
	      </xs:element>
	      <xs:element name="ContentKey" nillable="true" type="tns:PlayReadyContentKey" />
	      <xs:element minOccurs="0" name="ContentType" type="tns:ContentType">
	        <xs:annotation>
	          <xs:appinfo>
	            <DefaultValue EmitDefaultValue="false" xmlns="http://schemas.microsoft.com/2003/10/Serialization/" />
	          </xs:appinfo>
	        </xs:annotation>
	      </xs:element>
	      <xs:element minOccurs="0" name="ExpirationDate" nillable="true" type="xs:dateTime">
	        <xs:annotation>
	          <xs:appinfo>
	            <DefaultValue EmitDefaultValue="false" xmlns="http://schemas.microsoft.com/2003/10/Serialization/" />
	          </xs:appinfo>
	        </xs:annotation>
	      </xs:element>
	      <xs:element minOccurs="0" name="GracePeriod" nillable="true" type="ser:duration">
	        <xs:annotation>
	          <xs:appinfo>
	            <DefaultValue EmitDefaultValue="false" xmlns="http://schemas.microsoft.com/2003/10/Serialization/" />
	          </xs:appinfo>
	        </xs:annotation>
	      </xs:element>
	      <xs:element minOccurs="0" name="LicenseType" type="tns:PlayReadyLicenseType" />
	      <xs:element minOccurs="0" name="PlayRight" nillable="true" type="tns:PlayReadyPlayRight" />
	    </xs:sequence>
	  </xs:complexType>
	  <xs:element name="PlayReadyLicenseTemplate" nillable="true" type="tns:PlayReadyLicenseTemplate" />
	  <xs:simpleType name="PlayReadyLicenseType">
	    <xs:restriction base="xs:string">
	      <xs:enumeration value="Nonpersistent" />
	      <xs:enumeration value="Persistent" />
	    </xs:restriction>
	  </xs:simpleType>
	  <xs:element name="PlayReadyLicenseType" nillable="true" type="tns:PlayReadyLicenseType" />
	  <xs:complexType name="PlayReadyPlayRight">
	    <xs:sequence>
	      <xs:element minOccurs="0" name="AgcAndColorStripeRestriction" nillable="true" type="tns:AgcAndColorStripeRestriction">
	        <xs:annotation>
	          <xs:appinfo>
	            <DefaultValue EmitDefaultValue="false" xmlns="http://schemas.microsoft.com/2003/10/Serialization/" />
	          </xs:appinfo>
	        </xs:annotation>
	      </xs:element>
	      <xs:element minOccurs="0" name="AllowPassingVideoContentToUnknownOutput" type="tns:UnknownOutputPassingOption">
	        <xs:annotation>
	          <xs:appinfo>
	            <DefaultValue EmitDefaultValue="false" xmlns="http://schemas.microsoft.com/2003/10/Serialization/" />
	          </xs:appinfo>
	        </xs:annotation>
	      </xs:element>
	      <xs:element minOccurs="0" name="AnalogVideoOpl" nillable="true" type="xs:int">
	        <xs:annotation>
	          <xs:appinfo>
	            <DefaultValue EmitDefaultValue="false" xmlns="http://schemas.microsoft.com/2003/10/Serialization/" />
	          </xs:appinfo>
	        </xs:annotation>
	      </xs:element>
	      <xs:element minOccurs="0" name="CompressedDigitalAudioOpl" nillable="true" type="xs:int">
	        <xs:annotation>
	          <xs:appinfo>
	            <DefaultValue EmitDefaultValue="false" xmlns="http://schemas.microsoft.com/2003/10/Serialization/" />
	          </xs:appinfo>
	        </xs:annotation>
	      </xs:element>
	      <xs:element minOccurs="0" name="CompressedDigitalVideoOpl" nillable="true" type="xs:int">
	        <xs:annotation>
	          <xs:appinfo>
	            <DefaultValue EmitDefaultValue="false" xmlns="http://schemas.microsoft.com/2003/10/Serialization/" />
	          </xs:appinfo>
	        </xs:annotation>
	      </xs:element>
	      <xs:element minOccurs="0" name="DigitalVideoOnlyContentRestriction" type="xs:boolean">
	        <xs:annotation>
	          <xs:appinfo>
	            <DefaultValue EmitDefaultValue="false" xmlns="http://schemas.microsoft.com/2003/10/Serialization/" />
	          </xs:appinfo>
	        </xs:annotation>
	      </xs:element>
	      <xs:element minOccurs="0" name="ExplicitAnalogTelevisionOutputRestriction" nillable="true" type="tns:ExplicitAnalogTelevisionRestriction">
	        <xs:annotation>
	          <xs:appinfo>
	            <DefaultValue EmitDefaultValue="false" xmlns="http://schemas.microsoft.com/2003/10/Serialization/" />
	          </xs:appinfo>
	        </xs:annotation>
	      </xs:element>
	      <xs:element minOccurs="0" name="FirstPlayExpiration" nillable="true" type="ser:duration">
	        <xs:annotation>
	          <xs:appinfo>
	            <DefaultValue EmitDefaultValue="false" xmlns="http://schemas.microsoft.com/2003/10/Serialization/" />
	          </xs:appinfo>
	        </xs:annotation>
	      </xs:element>
	      <xs:element minOccurs="0" name="ImageConstraintForAnalogComponentVideoRestriction" type="xs:boolean">
	        <xs:annotation>
	          <xs:appinfo>
	            <DefaultValue EmitDefaultValue="false" xmlns="http://schemas.microsoft.com/2003/10/Serialization/" />
	          </xs:appinfo>
	        </xs:annotation>
	      </xs:element>
	      <xs:element minOccurs="0" name="ImageConstraintForAnalogComputerMonitorRestriction" type="xs:boolean">
	        <xs:annotation>
	          <xs:appinfo>
	            <DefaultValue EmitDefaultValue="false" xmlns="http://schemas.microsoft.com/2003/10/Serialization/" />
	          </xs:appinfo>
	        </xs:annotation>
	      </xs:element>
	      <xs:element minOccurs="0" name="ScmsRestriction" nillable="true" type="tns:ScmsRestriction">
	        <xs:annotation>
	          <xs:appinfo>
	            <DefaultValue EmitDefaultValue="false" xmlns="http://schemas.microsoft.com/2003/10/Serialization/" />
	          </xs:appinfo>
	        </xs:annotation>
	      </xs:element>
	      <xs:element minOccurs="0" name="UncompressedDigitalAudioOpl" nillable="true" type="xs:int">
	        <xs:annotation>
	          <xs:appinfo>
	            <DefaultValue EmitDefaultValue="false" xmlns="http://schemas.microsoft.com/2003/10/Serialization/" />
	          </xs:appinfo>
	        </xs:annotation>
	      </xs:element>
	      <xs:element minOccurs="0" name="UncompressedDigitalVideoOpl" nillable="true" type="xs:int">
	        <xs:annotation>
	          <xs:appinfo>
	            <DefaultValue EmitDefaultValue="false" xmlns="http://schemas.microsoft.com/2003/10/Serialization/" />
	          </xs:appinfo>
	        </xs:annotation>
	      </xs:element>
	    </xs:sequence>
	  </xs:complexType>
	  <xs:element name="PlayReadyPlayRight" nillable="true" type="tns:PlayReadyPlayRight" />
	  <xs:simpleType name="UnknownOutputPassingOption">
	    <xs:restriction base="xs:string">
	      <xs:enumeration value="NotAllowed" />
	      <xs:enumeration value="Allowed" />
	      <xs:enumeration value="AllowedWithVideoConstriction" />
	    </xs:restriction>
	  </xs:simpleType>
	  <xs:element name="UnknownOutputPassingOption" nillable="true" type="tns:UnknownOutputPassingOption" />
	  <xs:complexType name="ScmsRestriction">
	    <xs:sequence>
	      <xs:element minOccurs="0" name="ConfigurationData" type="xs:unsignedByte" />
	    </xs:sequence>
	  </xs:complexType>
	  <xs:element name="ScmsRestriction" nillable="true" type="tns:ScmsRestriction" />
	</xs:schema>



<!---HONumber=Mooncake_0620_2016-->