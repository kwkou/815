<properties
   pageTitle="渗透测试 | Microsoft Azure"
   description="本文概述了渗透测试（简称 pentest）过程，以及如何对运行在 Azure 基础结构中的应用进行渗透测试。"
   services="security"
   documentationCenter="na"
   authors="YuriDio"
   manager="swadhwa"
   editor="TomSh"/>  


<tags
   ms.service="security"
   ms.devlang="na"
   ms.topic="article"
   ms.tgt_pltfrm="na"
   ms.workload="na"
   ms.date="08/09/2016"
   wacn.date="10/31/2016"
   ms.author="yurid"/>  


# 渗透测试

使用 Microsoft Azure 进行应用程序测试和部署的一大好处是，用户不需设置本地基础结构即可开发、测试和部署应用程序。所有基础结构工作均由 Microsoft Azure 平台服务完成。用户不需担心如何请求、获取和安装自己的本地硬件。

此功能很有用，但用户仍需确保执行正常的安全防护措施。用户需执行的一项操作是对部署在 Azure 中的应用程序进行渗透测试。

用户可能已经知道，21Vianet 执行[对 Azure 环境的渗透测试](https://gallery.technet.microsoft.com/Cloud-Red-Teaming-b837392e)。该测试有助于改进我们的平台，指导我们在以下方面的行动：改进安全控制、引入新的安全控制，以及改进安全流程。

我们不会为用户进行应用程序渗透测试，但也理解用户想要对自己的应用程序进行渗透测试的心情，这确实也是必需的。这是好事，因为改进用户应用程序的安全性可以加强整个 Azure 生态系统的安全性。

当用户对应用程序进行渗透测试时，看起来就像是对我们进行一次攻击。我们会[持续监视](http://blogs.msdn.com/b/azuresecurity/archive/2015/07/05/best-practices-to-protect-your-azure-deployment-against-cloud-drive-by-attacks.aspx)各种攻击模式，并且会根据需要发起事件响应过程。如果用户进行正当的渗透测试我们就触发事件响应，这对用户和我们都没有好处。

怎么办？

用户在准备对 Azure 托管的应用程序进行渗透测试时，需让我们知道。我们知道用户要进行特定的测试后，就不会无意中将用户关闭（例如阻止用户在测试时需要使用的 IP 地址），但前提是用户的测试遵循 Azure 的渗透测试条款和条件。

用户不能执行的一类测试是任何类型的[拒绝服务 (DoS)](https://en.wikipedia.org/wiki/Denial-of-service_attack) 攻击。其中包括：发起 DoS 攻击，或者执行相关的测试，以便确定、演示或模拟任何类型的 DoS 攻击。

做好对 Microsoft Azure 中托管的应用程序进行渗透测试的准备了吗？ 如果已做好准备，则可转到 [Penetration Test Overview](https://www.trustcenter.cn/zh-cn/security/penetrationtesting.html)（渗透测试概述）页，单击页面底部的“创建测试请求”按钮。用户还会发现有关渗透测试条款和条件的详细信息，并可通过各种有用的链接报告 Azure 或其他 Microsoft 服务的相关安全缺陷。

<!---HONumber=Mooncake_1024_2016-->