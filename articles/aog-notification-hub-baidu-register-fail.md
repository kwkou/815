
<properties
	pageTitle="百度推送通过获取注册信息失败"
	description="介绍如何解决百度推送过程中的注册问题。"
	services="notification-hubs"
	documentationCenter=""
	authors=""
	manager=""
	editor=""
	tags="通知中心,百度推送,注册信息"/>

<tags
	ms.service="notification-hubs-aog"
	ms.date="10/28/2016"
	wacn.date="10/28/2016"/>


# 百度推送通过参数获取注册信息失败 #


**问题：** 在尝试使用参数 ChanneId 获取注册信息时失败

**原因：** `getRegistrationsByChannel()` 方法使用的不当。在 notificationHub 的包中关于百度推送的获取方法只支持通过 Tags 标签的形式获取注册信息；使用 channelId 参数的方法可以调用，但是无法返回注册结果集。因为百度本身的句柄是由 channelId 和 UserId 共同组成，故而无法仅仅通过 channelId 获取注册结果集。

**解决方法：**

方案一：直接使用 Tags 获取注册信息

		CollectionResult result = hub.getRegistrationsByTag("myTestTag111");
		for(Registration item:result.getRegistrations())
		{
		    System.out.println(item.getXml());
		}

方案二：直接修改源码通过 ChannelID 和 UserId 两个参数获取注册信息

		public void getRegistrationsByBaiduChannelIdAsync(String BaiduUserId, String BaiduChannelId, String continuationToken, final FutureCallback<CollectionResult> callback) {
		String endpoint = "https://baidutestns.servicebus.chinacloudapi.cn/";
		String hubPath = "baidutest";//your notificationhub name
		String APIVERSION = "?api-version=2015-01";
		
		String queryUri = null;
		try {
		String channelQuery = URLEncoder.encode("BaiduUserId-BaiduChannelId eq '" + BaiduUserId + "-" + BaiduChannelId + "'", "UTF-8");
		
		queryUri = endpoint + hubPath + "/registrations" + APIVERSION
		           + "&$filter=" + channelQuery
		           + getQueryString(0, continuationToken);
			} catch (UnsupportedEncodingException e) {
				throw new RuntimeException(e);
			}
			retrieveRegistrationCollectionAsync(queryUri, callback);
		}

