<properties
	pageTitle="如何发送预定通知 | Azure"
	description="本主题介绍如何使用 Azure 通知中心发送预定通知。"
	services="notification-hubs"
	documentationCenter=".net"
	keywords="推送通知,push notification,计划推送通知"
	authors="wesmc7777"
	manager="erikre"
	editor=""/>
<tags
	ms.service="notification-hubs"
	ms.date="04/11/2016"
	wacn.date="05/18/2016"/>

# 如何：发送预定通知


##概述

如果在你的方案中，你需要在将来的某个时间点发送通知，但却无法轻松地唤醒后端代码来发送该通知。标准层通知中心支持一种功能，可让你提前最多 7 天计划好通知。

发送通知时，只需如以下示例中所示，使用通知中心 SDK 中的 [ScheduledNotification](https://msdn.microsoft.com/library/microsoft.azure.notificationhubs.schedulednotification.aspx) 类：

	Notification notification = new AppleNotification("{"aps":{"alert":"Happy birthday!"}}");
	var scheduled = await hub.ScheduleNotificationAsync(notification, new DateTime(2014, 7, 19, 0, 0, 0));

此外，你可以使用其 notificationId 取消以前计划的通知：

	await hub.CancelNotificationAsync(scheduled.ScheduledNotificationId);

可以发送的预定通知数没有限制。
<!---HONumber=Mooncake_0503_2016-->