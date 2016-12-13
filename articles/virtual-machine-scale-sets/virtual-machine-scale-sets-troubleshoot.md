<properties
    pageTitle="疑难解答使用虚拟机规模集的自动缩放问题 | Azure"
    description="疑难解答使用虚拟机规模集的自动缩放问题。了解遇到的典型问题以及如何解决这些问题。"
    services="virtual-machine-scale-sets"
    documentationcenter=""
    author="gbowerman"
    manager="timlt"
    editor=""
    tags="azure-resource-manager" />  

<tags
    ms.assetid="c7d87b72-ee24-4e52-9377-a42f337f76fa"
    ms.service="virtual-machine-scale-sets"
    ms.workload="na"
    ms.tgt_pltfrm="windows"
    ms.devlang="na"
    ms.topic="article"
    ms.date="10/28/2016"
    wacn.date="12/12/2016"
    ms.author="guybo" />

# 疑难解答使用虚拟机规模集的自动缩放问题
**问题** - 您已使用 VM 规模集在 Azure Resource Manager 中创建自动缩放基础结构 - 例如，通过部署与此类似的模板：https://github.com/Azure/azure-quickstart-templates/tree/master/201-vmss-bottle-autoscale - 您定义了缩放规则，其效果良好，只不过无论在 VM 中施放多少负载，它都不会自动缩放。

## 疑难解答步骤
应考虑的一些事项包括：

* 每个 VM 有多少个核心？您加载了所有核心吗？ 上面的 Azure 快速入门模板示例具有 do\_work.php 脚本，它加载单个核心。如果使用的 VM 超过单核 VM 的大小（例如超过 Standard\_A1 或 D1），需要多次运行此负载。通过查看 [Azure 中 Windows 虚拟机的大小](/documentation/articles/virtual-machines-windows-sizes/)检查 VM 中有多少个核心
* VM 规模集中有多少个 VM？您正在处理每个 VM 吗？
  
    仅当规模集中**所有** VM 的平均 CPU 在自动缩放规则中内部定义的时间之内超出阈值时，才会发生扩大事件。
* 是否遗漏任何缩放事件？
  
    检查 Azure 门户中的审核日志以查找缩放事件。或许遗漏了一个增加事件和一个减少事件。可以通过“缩放”筛选...
  
    ![审核日志][audit]
* 您的缩小和扩大阈值的差值是否足够大？
  
    假设您设置了一个规则，该规则指示当平均 CPU 在 5 分钟内超过 50% 时扩大，当平均 CPU 低于 50% 则缩小。在 CPU 使用率接近此阈值时，这将导致“摇摆”问题，缩放操作不断地增加和减少集的大小。因此，自动缩放服务尝试防止“摇摆”的发生，这可能表现为不缩放。因此请确保扩大和缩小阈值的差值足够大，以允许在缩放之间存在一些空间。
* 您是否编写过自己的 JSON 模板？
  
    编写时很容易犯错，因此可使用如上述的久经验证的模板来开始编写，然后进行微小的增量更改。
* 您可以手动缩小或扩大吗？
  
    请尝试使用不同的“容量”设置重新部署 VM 规模集资源，以手动更改 VM 的数目。此处是一个用于执行此操作的示例模板：https://github.com/Azure/azure-quickstart-templates/tree/master/201-vmss-scale-existing - 可能需要编辑模板以确保其虚拟机大小和您的规模集正在使用的相同。如果成功手动更改 VM 数目，则可知该问题与自动缩放无关。

* 诊断扩展运行正常且可发出性能数据吗？
  
    **更新：**Azure 自动缩放已得到增强，使用基于主机的指标管道，不再需要安装诊断扩展。这意味着，如果使用新管道创建自动缩放的应用程序，则下面的几个段落不再适用。改为使用主机管道的 Azure 模板示例包括：https://github.com/Azure/azure-quickstart-templates/tree/master/201-vmss-bottle-autoscale、https://github.com/Azure/azure-quickstart-templates/tree/master/201-vmss-lapstack-autoscale。
  
    使用基于主机的指标进行自动缩放更有利，原因如下：
  
  * 由于不需要安装诊断扩展，因为移动的部件更少。
  * 模板更简单。只需将见解自动缩放规则添加到现有规模集模板即可。
  * 报告更可靠，新 VM 的启动速度更快。
    
    继续使用诊断扩展的唯一原因是需要内存诊断报告/缩放。基于主机的指标不会报告内存。
    
    出于上述原因，仅当仍要使用诊断扩展进行自动缩放时，才遵循本文的余下内容。
    
    Azure Resource Manager 中的自动缩放可以通过一个名为“诊断扩展”的 VM 扩展来工作（但现在不再需要此扩展）。它会向模板中定义的存储帐户发出性能数据。然后此数据由 Azure 监视器服务聚合。
    
    如果 Insights 服务无法从 VM 中读取数据，它会发送给您一封电子邮件 - 例如，如果 VM 已关闭，那么应查看您的电子邮件（创建 Azure 帐户时指定的电子邮件）。
    
    此外，还可以自己前往并查看数据。使用云资源管理器查看 Azure 存储帐户。例如使用 [Visual Studio Cloud Explorer](https://visualstudiogallery.msdn.microsoft.com/aaef6e67-4d99-40bc-aacf-662237db85a2)，登录并选取正在使用的 Azure 订阅以及部署模板中诊断扩展定义中引用的诊断存储帐户名称...
    
    ![云资源管理器][explorer]
    
    将在此处看到许多表，其中存储着每个 VM 的数据。以 Linux 和 CPU 指标为例，查看最近的行。Visual Studio Cloud Explorer 支持查询语言，因此可以运行类似“Timestamp gt datetime'2016-02-02T21:20:00Z'”的查询以确保获取最近的事件（假设时间采用 UTC 格式）。您在此处看到的数据是否符合您设置的缩放规则？ 在下面的示例中，虚拟机 20 的 CPU 在过去 5 分钟内开始增加到 100%...
    
    ![存储表][tables]
    
    如果数据不存在，则意味着问题与在 VM 中运行的诊断扩展相关。如果数据存在，则意味着问题与缩放规则或 Insights 服务相关。检查 [Azure 状态](/support/service-dashboard/)。

    后完这些步骤成，如果仍旧遇到自动缩放的问题，可以尝试访问 [MSDN](https://social.msdn.microsoft.com/forums/azure/home?category=windowsazureplatform%2Cazuremarketplace%2Cwindowsazureplatformctp) 上的论坛、[CSDN Azure](http://azure.csdn.net/)，或者登记电话支持。准备共享模板和性能数据的视图。

[audit]: ./media/virtual-machine-scale-sets-troubleshoot/image3.png
[explorer]: ./media/virtual-machine-scale-sets-troubleshoot/image1.png
[tables]: ./media/virtual-machine-scale-sets-troubleshoot/image4.png

<!---HONumber=Mooncake_1205_2016-->