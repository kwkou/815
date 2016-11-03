<properties
	pageTitle="安卓SDK无法识别Azure媒体服务发布的HLS视频地址"
	description="如何解决安卓SDK无法识别Azure媒体服务发布的HLS视频地址的问题。"
	services="media-service"
	documentationCenter=""
	authors=""
	manager=""
	editor=""
	tags=""/>

<tags
	ms.service="media-service-aog"
	ms.date="10/27/2016"
	wacn.date="11/03/2016"/>

#安卓 SDK 无法识别 Azure 媒体服务发布的 HLS 视频地址

**问题：**

安卓 SDK 无法识别 Azure 媒体服务发布的 HLS 视频地址

**现象：**

在媒体服务发布的URL后面添加 `format=m3u8-appl-v3` 后，通过安卓 SDK (比如 ijkplayer )无法调用

**解决方法：**

在发布 URL 后添加后缀 .m3u8 如：http://xxxxxxxx.streaming.mediaservices.chinacloudapi.cn/4254b276-aa65-463f-9d75-778999e7b36f/0dfe8107-b22a-49ca-94ce-dea54989da4c.ism/manifest(format=m3u8-aapl-v3).m3u8 
