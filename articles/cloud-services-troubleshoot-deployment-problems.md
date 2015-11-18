<properties 
 pageTitle="解决云服务部署问题 | Windows Azure" 
 description="将云服务部署到 Azure 时，你可能会遇到几个常见问题。本文提供了部分问题的解决方案。" 
   services="cloud-services"
   documentationCenter=""
   authors="Thraka"
   manager="msmets"
   editor=""
   tags="top-support-issue"/>
<tags 
   ms.service="cloud-services"
   ms.date="10/14/2015"
   wacn.date="11/12/2015" />

# 如何解决可能会遇到的云服务部署问题

将云服务应用程序包部署到 Azure 时，可通过管理门户中的“属性”窗格获取有关部署的信息。你可以使用此窗格中的详细信息来帮助你解决云服务的问题，还可以在提交新的支持请求时将此信息提供给 Azure 支持人员。

> [AZURE.NOTE]你可以通过单击“属性”窗格右上角的图标将该窗格的内容复制到剪贴板。

## 与 Azure 客户支持联系

如果你对本文中的任何观点存在疑问，可以联系 [MSDN Azure 和堆栈溢出论坛](/support/forums/)上的 Azure 专家。

或者，你也可以提出 Azure 支持事件。转至 [Azure 支持站点](/support/options/)并单击“获取支持”。有关使用 Azure 支持的信息，请阅读 [Windows Azure 支持常见问题](/support/faq/)。



## 问题：虽然我的部署已启动且所有角色实例都已就绪，但我仍然无法访问我的网站

在门户中显示的网站 URL 链接不包括该端口。网站的默认端口为 80。但是，如果你的应用程序已配置为在其他端口中运行，则必须在访问网站时向 URL 添加正确的端口。

1. 在管理门户中，单击云服务的部署。
2. 在管理门户的“属性”窗格中，查看角色实例（位于“输入终结点”下）的端口。
3. 如果端口不是 *80*，则请在访问应用程序时将正确的端口值添加到 URL。若要指定非默认端口，请键入 URL，后跟冒号 (:) 以及端口号（无空格）。

## 问题：我的角色实例在我不执行任何操作的情况下重新启动

当 Azure 检测到问题节点并将角色实例移到新节点时，系统会自动进行服务修复。当发生这种情况时，你可能会看到角色实例自动重新启动。若要查看是否进行了服务修复，请执行以下操作：

1. 在管理门户中，单击云服务的部署。
2. 在管理门户的“属性”窗格中查看相关信息，确定在你观察到角色重新启动期间是否进行了服务修复。

在主机 OS 和来宾 OS 升级期间，角色也会差不多每月重新启动一次。有关详细信息，请参阅博客文章[重新启动角色实例进行 OS 升级](http://blogs.msdn.com/b/jarrettr/archive/2012/09/19/role-instance-restarts-due-to-os-upgrades.aspx)

## 问题：无法进行 VIP 交换并收到错误

如果部署更新正在进行，则不能进行 VIP 交换。出现以下情况时，部署更新可能会自动进行：

* 新的来宾操作系统已发布，而你已配置为自动进行更新
* 进行了服务修复

若要查看是否是自动升级阻止你执行 VIP 交换，请执行以下操作：

1. 在管理门户中，单击云服务的部署。
2. 在管理门户的“属性”窗格中，查看“状态”的值。如果状态为“就绪”，则请查看“上次操作”，看最近是否进行了升级，因为升级可能会阻止 VIP 交换的进行。
3. 重复进行生产部署所需的步骤 1 和步骤 2。
4. 如果自动更新正在进行，则请等待其完成，然后再尝试进行 VIP 交换。

## 问题：角色实例在“已启动”、“正在初始化”、“忙碌”和“已停止”这几种状态之间循环往复

这种情况可能表示你的应用程序代码、包或配置文件存在问题。如果确实是这样，你会看到状态每隔数分钟改变一次，而 Azure 管理门户则显示“正在回收”、“忙”或“正在初始化”之类的内容。这表示应用程序存在问题，导致角色实例无法运行。

有关如何解决此问题的详细信息，请参阅博客文章 [Azure PaaS 计算诊断数据]和[导致角色回收的常见问题](/documentation/articles/cloud-services-troubleshoot-common-issues-which-cause-roles-recycle)

## 问题：我的应用程序停止工作

1. 在管理门户中，单击角色实例。
2. 在管理门户的“属性”窗格中，考虑是否存在以下情况，这些情况将有助于解决你的问题：
   * 如果角色实例最近停止过（可查看“中止计数”的值），则可能是因为部署正在进行更新。等待，看角色实例是否会自行恢复运行。
   * 如果角色实例处于“忙碌”状态，请检查应用程序代码，看是否已处理 [StatusCheck](https://msdn.microsoft.com/zh-cn/library/microsoft.windowsazure.serviceruntime.roleenvironment.statuscheck) 事件。你可能需要添加或修复处理此事件的某些代码。
   * 浏览博客文章 [Azure PaaS 计算诊断数据]中的诊断数据和故障排除方案。

>[AZURE.WARNING]如果你重新启动云服务，请重置部署的属性，以便有效擦除有关原始问题的信息。

## 后续步骤

查看更多针对云服务的[故障排除文章](https://azure.microsoft.com/zh-cn/documentation/articles/?tag=top-support-issue&service=cloud-services?tag=top-support-issue&service=cloud-services)。


[Azure PaaS 计算诊断数据]: http://blogs.msdn.com/b/kwill/archive/2013/08/09/windows-azure-paas-compute-diagnostics-data.aspx

<!---HONumber=79-->