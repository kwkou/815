<properties
 pageTitle="排查云服务部署问题 | Azure"
 description="将云服务部署到 Azure 时，你可能会遇到几个常见问题。本文提供了部分问题的解决方案。"
   services="cloud-services"
   documentationCenter=""
   authors="dalechen"
   manager="felixwu"
   editor=""
   tags="top-support-issue"/>
<tags
   ms.service="cloud-services"
   ms.date="04/20/2016"
   wacn.date="05/31/2016" />

# 排查云服务部署问题

将云服务应用程序包部署到 Azure 时，可通过 Azure 门户中的“属性”窗格获取有关部署的信息。你可以使用此窗格中的详细信息来帮助你解决云服务的问题，还可以在提交新的支持请求时将此信息提供给 Azure 支持人员。

可按如下所述找到“属性”窗格：

* 在 Azure 门户中，依次单击云服务的部署、“所有设置”、“属性”。
* 在 Azure 管理门户中，依次单击云服务的部署、“仪表板”，然后定位到页面右下角（在“速览”下面）。请注意，此窗格中没有“属性”标签。

> [AZURE.NOTE] 你可以通过单击“属性”窗格右上角的图标将该窗格的内容复制到剪贴板。

## 与 Azure 客户支持联系

如果你对本文中的任何点需要更多帮助，可以联系 [MSDN Azure 和堆栈溢出论坛](/support/forums)上的 Azure 专家。

或者，你也可以提出 Azure 支持事件。请转到 [Azure 支持站点](/support/contact)并单击“获取支持”。有关使用 Azure 支持的信息，请阅读 [Azure 支持常见问题](/support/faq)。

## 问题：我无法访问我的网站，但我的部署已启动且所有角色实例均已就绪

在门户中显示的网站 URL 链接不包括该端口。网站的默认端口为 80。如果你的应用程序已配置为在其他端口中运行，则必须在访问网站时向 URL 添加正确的端口号。

1. 在 Azure 门户中，单击云服务的部署。
2. 在 Azure 门户的“属性”窗格中，查看角色实例（位于“输入终结点”下）的端口。
3. 如果端口不是 80，请在访问应用程序时将正确的端口值添加到 URL。若要指定非默认端口，请键入 URL，后跟冒号 (:) 以及端口号（无空格）。

## 问题：我的角色实例在我未执行任何操作的情况下被回收

当 Azure 检测到问题节点并因此将角色实例移到新节点时，系统会自动进行服务修复。当发生这种情况时，你可能会看到角色实例自动回收。若要查看是否进行了服务修复，请执行以下操作：

1. 在 Azure 门户中，单击云服务的部署。
2. 在 Azure 门户的“属性”窗格中查看相关信息，确定在你观察到角色回收期间是否进行了服务修复。

在主机 OS 和来宾 OS 更新期间，角色也会差不多每月回收一次。  
有关详细信息，请参阅博客文章[重新启动角色实例进行 OS 升级](http://blogs.msdn.com/b/kwill/archive/2012/09/19/role-instance-restarts-due-to-os-upgrades.aspx)

## 问题：无法进行 VIP 交换并收到错误

如果部署更新正在进行，则不能进行 VIP 交换。出现以下情况时，部署更新可能会自动进行：

* 新的来宾操作系统已发布，而你已配置为自动进行更新。
* 进行了服务修复。

若要了解是否是自动更新阻止你执行 VIP 交换，请执行以下操作：

1. 在 Azure 门户中，单击云服务的部署。
2. 在 Azure 门户的“属性”窗格中，查看“状态”的值。如果状态为“就绪”，则请查看“上次操作”，看最近是否进行了升级，因为升级可能会阻止 VIP 交换的进行。
3. 重复进行生产部署所需的步骤 1 和步骤 2。
4. 如果自动更新正在进行，则请等待其完成，然后再尝试进行 VIP 交换。

## 问题：角色实例在“已启动”、“正在初始化”、“忙碌”和“已停止”这几种状态之间循环往复

这种情况可能指示你的应用程序代码、程序包或配置文件存在问题。在这种情况下，你应能看到状态每隔几分钟更改一次，而 Azure 门户则可能会显示“正在回收”、“忙”或“正在初始化”之类的内容。这表示应用程序存在问题，导致角色实例无法运行。

有关如何解决此问题的详细信息，请参阅博客文章 [Azure PaaS 计算诊断数据](http://blogs.msdn.com/b/kwill/archive/2013/08/09/windows-azure-paas-compute-diagnostics-data.aspx)和[导致角色回收的常见问题](/documentation/articles/cloud-services-troubleshoot-common-issues-which-cause-roles-recycle/)。

## 问题：我的应用程序停止工作

1. 在 Azure 门户中，单击角色实例。
2. 在 Azure 门户的“属性”窗格中，考虑是否存在以下情况，以便解决你的问题：
   * 如果角色实例最近停止过（可查看“中止计数”的值），则可能是因为部署正在进行更新。等待，看角色实例是否会自行恢复运行。
   * 如果角色实例处于“忙”状态，请检查应用程序代码，看是否已处理 [StatusCheck](https://msdn.microsoft.com/zh-cn/library/microsoft.windowsazure.serviceruntime.roleenvironment.statuscheck) 事件。你可能需要添加或修复处理此事件的某些代码。
   * 浏览博客文章 [Azure PaaS 计算诊断数据](http://blogs.msdn.com/b/kwill/archive/2013/08/09/windows-azure-paas-compute-diagnostics-data.aspx)中的诊断数据和故障排除方案。

>[AZURE.WARNING] 如果你回收云服务，请重置部署的属性，以便有效清除有关原始问题的信息。

## 后续步骤

查看更多针对云服务的[故障排除文章](..\?tag=top-support-issue&service=cloud-services)。

若要了解如何使用 Azure PaaS 计算机诊断数据对云服务角色问题进行故障排除，请参阅 [Kevin Williamson 博客系列](http://blogs.msdn.com/b/kwill/archive/2013/08/09/windows-azure-paas-compute-diagnostics-data.aspx)。

<!---HONumber=Mooncake_0523_2016-->
