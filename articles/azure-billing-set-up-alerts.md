<properties 
	pageTitle="为您的 Microsoft Azure 订阅设置帐单警报" 
	description="介绍如何对 Azure 帐单设置警报，以免帐单出乎你的意料。" 
	services="" 
	documentationCenter="" 
	authors="vikdesai" 
	manager="msmbaldwin" 
	editor=""/>

<tags 
	ms.service="multiple" 
	ms.date="06/01/2015" 
	wacn.date="10/3/2015"/>

# 为您的 Microsoft Azure 订阅设置帐单警报

你在乎 Azure 订阅每个月花费多少钱吗？ 如果你是 Azure 订阅的帐户管理员，你可以使用 Azure 计费警报服务来创建自定义的计费警报，以帮助你监控和管理 Azure 帐户的计费活动。

此服务是一项预览版服务，因此您需要做的第一件事就是注册 - 请访问 Azure 帐户管理门户中的<a href="https://account.windowsazure.com/PreviewFeatures">预览版功能页面</a>来执行此操作。

## 设置警报阈值和电子邮件收件人

在收到已为您的订阅启用了计费服务的确认电子邮件后，请访问帐户门户中的<a href="https://account.windowsazure.com/Subscriptions">订阅页面</a>。单击您想要监控的订阅，然后单击“警报”。

![][Image1]

接下来，单击“添加警报”创建您的第一个警报 - 对于每个订阅，总共可以设置五个帐单警报，每个警报有一个不同的阈值和最多两个电子邮件收件人。

![][Image2]

添加警报时，你需要为其指定唯一的名称，选择费用阈值，并选择要向其发送警报的电子邮件地址。设置阈值时，可以从“警报条件”列表中选择“帐单合计”或“资金信用”。对于帐单合计，当订阅费用超出阈值时会发送警报。对于资金信用，当其降至限制以下时会发送警报。资金信用通常适用于免费试用以及与 MSDN 帐户关联的订阅。

![][Image3]

Azure 支持任何电子邮件地址并且不验证电子邮件地址是否可正常使用，因此请仔细检查以避免拼写错误。

## 检查警报

在设置警报后，帐户中心会列出它们并显示你还可以设置多少警报。对于每个警报，你可以看到其发送日期和时间，是针对帐单合计的警报还是针对资金信用的警报，以及你设置的限制。日期和时间格式为 24 小时制通用协调时间 (UTC)，日期为 yyyy-mm-dd 格式。单击列表中某个警报的加号可对其进行编辑，单击垃圾桶可将其删除。

[Image1]: ./media/azure-billing-set-up-alerts/billingalert1.png
[Image2]: ./media/azure-billing-set-up-alerts/billingalert2.png
[Image3]: ./media/azure-billing-set-up-alerts/billingalerts3.png

<!---HONumber=71-->