<properties 
	pageTitle="异常登录活动" 
	description="包括已由我们的机器学习算法确定为异常的登录的报告。" 
	services="active-directory" 
	documentationCenter="" 
	authors="kenhoff" 
	manager="ilanas" 
	editor=""/>

<tags
	ms.service="active-directory"
	ms.date="07/13/2015"
	wacn.date="08/29/2015"/>

# 异常登录活动

| 说明 | 报表位置 |
| :-------------     | :-------        |
| <p>此报表包括已由机器学习算法标识为“异常”的登录。将登录尝试标记为“异常”的原因包括：异常登录位置、当天的时间和位置，或这些原因的组合。这可能标名黑客已尝试使用此帐户登录。机器学习算法将事件归类为“异常”或“可疑”，其中“可疑”指示安全漏洞的可能性更高。</p><p>此报表中的结果将显示这些登录以及与每个登录关联的分类、位置和时间戳。</p><p>如果我们在 30 天内遇到 10 个或更多的异常登录事件，我们将向全局管理员发送电子邮件通知。请务必将 aad-alerts-noreply@mail.windowsazure.cn 添加到你的安全发件人列表中。</p> | “目录”>“报表”选项卡 |

<!---HONumber=67-->