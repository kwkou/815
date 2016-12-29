<properties
                pageTitle="百度推送服务绑定关系未找到或不存在"
                description="通过调用 Baidu Push API 修复 Bind Relation Not Found 错误"
                services="notification-hubs"
                documentationCenter=""
                authors=""
                manager=""
                editor=""
                tags="Notification Hubs,Baidu,Bind Relation"/>

<tags
                ms.service="notification-hubs-aog"
                ms.date="12/15/2016"
                wacn.date="12/15/2016"/>

# 百度推送服务绑定关系未找到或不存在

## 问题描述

使用 Azure 通知中心通过百度推送服务时反馈绑定关系未找到或不存在。

## 现象描述

Azure 通知中心通过百度推送时，服务端显示如下错误信息：

	Response:[{"request_id":**********,"error_code":30608,"error_msg":"Bind Relation Not Found"}]

## 解决方法

1.	通常遇到这种问题是因为绑定关系解除或者过期导致的，客户需要在对应的设备上重新调用一次 `Baidu Push` 的 API 方法：

		PushManager.startWork(context, loginType, loginValue)

2.	参考链接：[百度云推送服务端错误码](http://push.baidu.com/doc/restapi/error_code)。


