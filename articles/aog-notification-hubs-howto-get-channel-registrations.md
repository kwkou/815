<properties
	pageTitle="如何根据通道信息获取通知中心中的注册信息"
	description="如何根据通道信息获取通知中心中的注册信息。"
	services="notification-hubs"
	documentationCenter=""
	authors=""
	manager=""
	editor=""
	tags=""/>

<tags
	ms.service="notification-hubs-aog"
	ms.date="12/05/2016"
	wacn.date="12/05/2016"/>
# 如何根据通道信息获取通知中心中的注册信息 #

在中国市场，对于安卓系统的手机，不一定内置支持 GCM，但是支持百度推送服务。因此，为了更好的服务客户，由世纪互联运营的 Microsoft Azure 的通知中心也添加了对百度推送服务的支持，并推出了相应的 Java SDK 来给开发者使用。相关文档和示例可以参阅[这篇入门文章](/documentation/articles/notification-hubs-baidu-china-android-notifications-get-started/)。

Azure 的通知中心提供的是跨平台设备的支持，以及超大吞吐量和可伸缩性，它本身不会直接推送消息给手机端，而是通过具体推送服务的通道发送到手机客户端，比如中国安卓手机上的百度推送服务，苹果手机上的 APNS 等等，所以需要客户端提供具体推送服务的通道信息。那么这个客户端提供消息推送通道信息给 Azure 的通知中心的过程就叫注册 （Registration）。一个注册一般包括注册 ID，通道信息，标签等等信息。

当需要查阅一批客户端注册信息的时候，一般都可以通过标签来查询。那么当需要查阅某一特定客户端注册信息呢？当然我们也可以把通道信息作为标签信息来查询，但其实有更直接的办法，就是直接通过通道信息来获取。

在官方提供的 [Java SDK](https://github.com/Azure/azure-notificationhubs-java-backend) 里面，有个方法是 GetRegistrationsByChannel，但是这个方法其实只适用于微软平台，因为它发出的 http 请求用的通道过滤条件名称是 ChannelUri。而不同平台提供的通道信息名称都是不一样的，分别如下：

<table><tr><th> 平台 </th><th> 通道名称 </th></tr>
<tr><td> 微软（WNS, MPNS）</td><td> ChannelUri </td></tr>
<tr><td> 苹果（APNS） </td><td> DeviceToken </td></tr>
<tr><td> 谷歌（GCM） </td><td> GcmRegistrationId </td></tr>
<tr><td> 百度（Baidu） </td><td> BaiduUserId-BaiduChannelId </td></tr></table>

因此，需要根据不同的平台发出不同的请求。SDK 相关方法的缺失已经[提交请求](https://github.com/Azure/azure-notificationhubs-java-backend/issues/19)。另外我自己根据通知中心的 [REST API](https://msdn.microsoft.com/zh-cn/library/azure/dn223271.aspx) 实现了使用百度通道信息的查询。在 SDK 提供相关方法之前，大家可以修改 SDK 按照 GetRegistrationsByChannel 方法实现不同平台的查询，也可以模拟我的方法来实现，具体代码可在 [MSDN](https://code.msdn.microsoft.com/How-to-get-registrations-ca498761#content) 上下载。
