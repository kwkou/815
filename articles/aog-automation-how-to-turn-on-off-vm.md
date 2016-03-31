<properties 
	pageTitle="使用Azure Automation实现自动开关虚拟机的操作" 
	description="本页介绍如何使用Azure Automation实现自动开关虚拟机的操作。" 
	services="automation" 
	documentationCenter="" 
	authors=""
	manager="" 
	editor=""/>
<tags ms.service="automation" ms.date="" wacn.date="12/29/2015"/>

#使用Azure Automation实现自动开关虚拟机的操作
 
###本文包含以下内容

- [如何创建自动化账号和Runbook](#create)
- [如何创建凭据](#certification)
- [如何添加作业及执行计划](#scheduler)
- [更多参考资料](#resource)
 
## <a id="create"></a>如何创建自动化账号和Runbook
 
关于Azure Automation功能的使用请参考链接：
https://www.azure.cn/documentation/articles/automation-create-runbook-from-samples/
 
首先，我们需要创建一个自动化账号（Automation Account）：

![](./media/aog-automation-how-to-turn-on-off-vm/create-automation-account.jpg) 

在弹出的界面中填写自动化账号名称（用户随便定义一个即可），选择区域以及订阅：

![](./media/aog-automation-how-to-turn-on-off-vm/create-automation-account-step2.jpg) 

完成后就能够在列表中看到这个自动化账号，接着点击左下角“创建”按钮创建一个Runbook：

![](./media/aog-automation-how-to-turn-on-off-vm/create-autionmation-runbook.jpg)  

创建完成后点击进入这个账号：

![](./media/aog-automation-how-to-turn-on-off-vm/entry-automation.jpg) 

可以看到刚刚创建的Runbook：

![](./media/aog-automation-how-to-turn-on-off-vm/runbook-list.jpg) 

## <a id="certification"></a>如何创建凭据
 
接着我们在ASSETS选项中添加一个Credential：

![](./media/aog-automation-how-to-turn-on-off-vm/create-credential.jpg) 

在弹出的界面中选择ADD CREDENTIAL：

![](./media/aog-automation-how-to-turn-on-off-vm/select-credential-type.jpg)  

填写Credential的类型以及名称：

![](./media/aog-automation-how-to-turn-on-off-vm/define-credential.jpg)  

填写用到的用户名和密码（这里是管理Azure的用户名和密码）：

![](./media/aog-automation-how-to-turn-on-off-vm/define-credential-name-password.jpg)

**注意**： 这个步骤中的用户是通过AAD创建出来的，我们可以参考[这个链接](https://msdn.microsoft.com/zh-cn/library/azure/hh967632.aspx)来了解如何创建AAD用户。本例子是使用管理账号来做的， 

完成后保存。
 
## <a id="scheduler"></a>如何添加作业及执行计划
 
接着进入之前创建的Runbook中，切换到AUTHOR选项卡，点击EDIT RUNBOOK：

![](./media/aog-automation-how-to-turn-on-off-vm/edit-runbook.jpg)   

在弹出的编辑界面中输入下面的代码：

	workflow DanRunbook
	{
	    $Cred = Get-AutomationPSCredential -Name "DanCredential"; 
	    Add-AzureAccount -Credential $Cred -Environment AzureChinaCloud;
	    Select-AzureSubscription -SubscriptionName "Internal-002";    
	    Start-AzureVM -ServiceName "DanEastCS" -Name "Dan08Test";
	}

workflow后面的DanRunbook需要与你的Runbook的名称一致，请按照实际情况修改。
这里面第一行里面的DanCredential这个名字就是我们前面创建的Credential的名称，需要根据实际创建进行替换。Internal-002是订阅名称DanEastCS是云服务的名称，Dan08Test是虚拟机的名称，也都需要根据具体的情况进行替换。
 
创建完成后我们点击PUBLISH，将这段脚本发布为正式版本：

![](./media/aog-automation-how-to-turn-on-off-vm/publish-runbook.jpg)
 
注：这里也可以点击TEST（测试）先测试一下脚本的执行情况，确认无误后再点击PUBLISH。

发布完成后，我们在PUBLISHED选项卡中看到发布的正式脚本，可以点击START（开始）执行这个脚本：
 
![](./media/aog-automation-how-to-turn-on-off-vm/start-runbook.jpg)

执行完成后，会对应的生成一个JOB（作业），这里我们脚本测试了两次，所以有两条JOB记录：

![](./media/aog-automation-how-to-turn-on-off-vm/runbook-result.jpg)

点击JOB右侧的白色箭头可以进入到JOB中查看具体的执行情况和输出结果。
JOB执行结束后，可以看到我们的虚拟机已经成功启动了。
 
接着我们为RUNBOOK添加一个执行计划：

![](./media/aog-automation-how-to-turn-on-off-vm/create-shedule.jpg)
 
选择LINK TO A NEW SCHEDULE，填写一个计划名称：

![](./media/aog-automation-how-to-turn-on-off-vm/config-schedule.jpg) 

设置一个执行周期：

![](./media/aog-automation-how-to-turn-on-off-vm/config-shedule-detail.jpg)

例如这里根据上面截图中的设置，这个RUNBOOK会在2015年8月25日到2015年10月1日每天的12:00执行，如果不希望设置过期时间，可以取消SCHEDULE EXPIRES ON的勾选。
 
这样我们的开机脚本就设置完成了，同样的原理，可以使用下面的脚本配置一个关机脚本：

	workflow StopVMRunbook
	{
	    $Cred = Get-AutomationPSCredential -Name "DanCredential"; 
	    Add-AzureAccount -Credential $Cred -Environment AzureChinaCloud;
	    Select-AzureSubscription -SubscriptionName "Internal-002";	    
	    Stop-AzureVM -ServiceName "DanEastCS" -Name "Dan08Test" -Force;
	}
这个脚本创建在名为StopVMRunbook的Runbook中。
需要注意的是，这里Stop-AzureVM这个命令一定要添加-Force参数，不然在命令执行的时候会停在确认是否要关闭虚拟机的界面而无法完成关闭操作。
后面的配置操作与前面Start的类似，这里就不赘述了。
 
## <a id="resource"></a>更多参考资料

关于ASSETS的说明

- [凭据（Credential）](https://technet.microsoft.com/zh-cn/library/dn919926.aspx)
- [连接（Connection）](https://technet.microsoft.com/zh-cn/library/dn919922.aspx)
- [变量（Variable）](https://technet.microsoft.com/zh-cn/library/dn919925.aspx)
- [计划（Schedule）](https://technet.microsoft.com/zh-cn/library/dn919914.aspx)
- [创作自动化 Runbook](https://technet.microsoft.com/zh-cn/library/dn469262.aspx)
- [Getting Started with NEW Azure Automation preview feature](http://blogs.technet.com/b/keithmayer/archive/2014/04/04/step-by-step-getting-started-with-windows-azure-automation.aspx)
- [面向 IT 专业人员的脚本资源](https://gallery.technet.microsoft.com/scriptcenter/site/search?f%5B0%5D.Type=User&f%5B0%5D.Value=SC%20Automation%20Product%20Team&f%5B0%5D.Text=SC%20Automation%20Product%20Team&f%5B1%5D.Type=RootCategory&f%5B1%5D.Value=WindowsAzure&f%5B1%5D.Text=Windows%20Azure)
