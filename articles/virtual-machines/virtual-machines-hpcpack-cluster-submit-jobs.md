<properties
 pageTitle="向 Azure 中的 HPC Pack 群集提交作业 | Azure"
 description="了解如何设置本地计算机，以将作业提交到 Azure 中的 HPC Pack 群集"
 services="virtual-machines-windows"
 documentationCenter=""
 authors="dlepow"
 manager="timlt"
 editor=""
 tags="azure-resource-manager,azure-service-management,hpc-pack"/>
<tags
	ms.service="virtual-machines-windows"
	ms.date="04/14/2016"
	wacn.date="05/24/2016"/>

# 将 HPC 作业从本地计算机提交到部署在 Azure 中的 HPC Pack 群集

[AZURE.INCLUDE [了解部署模型](../includes/learn-about-deployment-models-both-include.md)]

配置运行 Windows 的本地客户端计算机，使其运行 HPC Pack 作业提交工具，以通过 HTTPS 来与 Azure 中的 HPC Pack 群集通信。这为各种群集用户提供了一种直截了当的灵活方法来向基于云的 HPC Pack 群集提交作业，不需要直接连接到头节点 VM 或访问 Azure 订阅来运行作业提交工具。

![向 Azure 中的群集提交作业][jobsubmit]

## 先决条件

* **部署在 Azure VM 中的 HPC Pack 头节点** - 建议你使用自动化工具（例如 [Azure PowerShell script](/documentation/articles/virtual-machines-windows-classic-hpcpack-cluster-powershell-script/)）来部署头节点和群集。需要获得头节点的 DNS 名称和群集管理员的凭据才能完成本文中的步骤。

* **HPC Pack 安装媒体** - HPC Pack (HPC Pack 2012 R2) 最新版本的免费安装包可在 [Microsoft 下载中心](http://go.microsoft.com/fwlink/?LinkId=328024)找到。确保下载与头节点 VM 上安装的版本相同的 HPC Pack 版本。

* **客户端计算机** - 需要有可运行 HPC Pack 客户端实用工具的 Windows 或 Windows Server 客户端计算机（请参阅[系统要求](https://technet.microsoft.com/zh-cn/library/dn535781.aspx)）。如果你只想要使用 HPC Pack Web 门户或 REST API 来提交作业，则可以使用自选的客户端计算机。


##<a name="step-1:-install-and-configure-the-web-components-on-the-head-node"></a> 步骤 1：在头节点上安装并配置 Web 组件

若要启用通过 HTTPS 以 REST 接口将作业提交到群集的功能，请在 HPC Pack 头节点上安装并配置 HPC Pack Web 组件（如果尚未配置）。必须先运行 HpcWebComponents.msi 安装文件，以安装 Web 组件。然后，通过运行 HPC PowerShell 脚本 **Set-HPCWebComponents.ps1** 来配置组件。

有关详细过程，请参阅[安装 Microsoft HPC Pack Web 组件](http://technet.microsoft.com/zh-cn/library/hh314627.aspx)。

>[AZURE.TIP] 特定的 HPC Pack 模板会自动安装和配置 Web 组件。如果使用 [HPC Pack IaaS 部署脚本](/documentation/articles/virtual-machines-windows-classic-hpcpack-cluster-powershell-script/)创建群集，你可以在部署期间选择安装并配置 Web 组件。

**安装 Web 组件**

1. 使用群集管理员的凭据连接到头节点 VM。

2. 在头节点上从 HPC Pack 安装程序文件夹中运行 HpcWebComponents.msi。

3. 按照向导中的步骤安装 Web 组件。

**配置 Web 组件**

1. 在头节点上，以管理员身份启动 HPC PowerShell。

2. 若要将目录切换到配置脚本所在的位置，请键入以下命令：

	
	    cd $env:CCP_HOME\bin
	    
3. 若要配置 REST 接口并启动 HPC Web 服务，请键入以下命令：

	
	    .\Set-HPCWebComponents.ps1 -Service REST -enable
	    

4. 在系统提示你选择证书时，请选择与头节点的公共 DNS 名称对应的证书。例如，如果使用 HPC Pack IaaS 部署脚本来创建群集，则证书名称采用 CN=&lt;*HeadNodeDnsName*&gt;.chinacloudapp.cn 形式。

    >[AZURE.NOTE] 你必须选择此证书才能在稍后将作业从本地计算机提交到头节点。不要选择或配置与 Active Directory 域中头节点的计算机名称对应的证书（例如 CN=*MyHPCHeadNode.HpcAzure.local*）。

5. 若要配置用于作业提交的 Web 门户，请键入以下命令：

	
	    .\Set-HPCWebComponents.ps1 -Service Portal -enable
	    
6. 脚本完成后，请通过键入以下命令停止并重新启动 HPC 作业计划程序服务：

	
	    net stop hpcscheduler
	    net start hpcscheduler
	    

## 步骤 2：在本地计算机上安装 HPC Pack 客户端实用工具

如果你要安装 HPC Pack 客户端，将 HPC Pack 安装程序文件（完整安装）从 [Microsoft 下载中心](http://go.microsoft.com/fwlink/?LinkId=328024)下载到客户端计算机。开始安装时，请选择针对 HPC Pack 客户端实用程序的安装选项。

若要使用 HPC Pack 客户端工具向头节点 VM 提交作业，你必须从头节点中导出证书并将其安装在客户端计算机上。该证书需采用 .CER 格式。

**从头节点中导出证书**

1. 在头节点上，向 Microsoft 管理控制台中添加用于“本地计算机”帐户的证书管理单元。有关添加此管理单元的步骤，请参阅[向 MMC 中添加证书管理单元](https://technet.microsoft.com/zh-cn/library/cc754431.aspx)。

2. 在控制台树中，依次展开“证书 - 本地计算机”和“个人”，然后单击“证书”。

3. 找到你在[步骤 1：在头节点上安装并配置 Web 组件](#step-1:-install-and-configure-the-web-components-on-the-head-node)中为 HPC Pack Web 组件配置的证书（例如，CN=&lt;HeadNodeDnsName&gt;.chinacloudapp.cn）。

4. 右键单击该证书，单击“所有任务”，然后单击“导出”。

5. 在证书导出向导中，单击“下一步”并确保选中“否，不导出私钥”。

6. 执行此向导中的其余步骤，以“DER 编码二进制 X.509 (.CER)”格式导出证书。


**在客户端计算机上导入证书**


1. 将你从头节点中导出的证书复制到客户端计算机上的某个文件夹中。

2. 在客户端计算机上，运行 certmgr.msc。

3. 在证书管理器中，依次展开“证书 - 当前用户”和“受信任的根证书颁发机构”，右键单击“证书”，单击“所有任务”，然后单击 “导入”。

4. 在证书导入向导中单击“下一步”，然后按照步骤将你从头节点中导出的证书导入“受信任的根证书颁发机构”存储。



>[AZURE.SECURITY] 由于客户端计算机无法识别头节点上的证书颁发机构，因此你可能会看到安全警告。出于测试目的，你可以忽略此警告并完成证书导入。

## 步骤 3：在群集上运行测试作业

若要验证你的配置，可以尝试使用运行 HPC Pack 客户端实用工具的本地计算机在 Azure 中的群集上运行作业。例如，可以使用 HPC Pack GUI 工具或 HPC Pack 命令行命令向群集提交作业。也可以使用基于 Web 的门户来提交作业。


**在客户端计算机上运行作业提交命令**


1. 在客户端计算机上启动命令提示符。

2. 键入示例命令。例如，若要列出群集中的所有作业，可键入如下所示的某个命令，具体取决于头节点的完整 DNS 名称：

	
	    job list /scheduler:https://<HeadNodeDnsName>.chinacloudapp.cn /all
	
	    job list /scheduler:https://<HeadNodeDnsName>.<region>.chinacloudapp.cn /all
	    

    >[AZURE.TIP] 请在计划程序 URL 中使用头节点的完整 DNS 名称，而不是 IP 地址。如果你指定 IP 地址，将会看到类似于下面的错误：“服务器证书必须具有有效的信任链，或放置在受信任的根存储区中”。

3. 出现提示时，请键入 HPC 群集管理员或你配置的另一群集用户的用户名（格式为 &lt;DomainName&gt;\\&lt;UserName&gt;）和密码。你可以选择在本地存储凭据以执行更多作业操作。

    将显示作业列表。


**在客户端计算机上使用 HPC 作业管理器**

1. 如果以前没有在客户端计算机上存储群集用户的域凭据，在提交作业时可以在凭据管理器中添加凭据。

    a.在客户端计算机上的控制面板中，启动凭据管理器。

    b.单击“Windows 凭据”，然后单击“添加普通凭据”。

    c.指定 Internet 地址（例如 https://&lt;HeadNodeDnsName&gt;.chinacloudapp.cn/HpcScheduler 或 https://&lt;HeadNodeDnsName&gt;.&lt;region&gt;.chinacloudapp.cn/HpcScheduler)，然后提供 HPC 群集管理员或你配置的另一群集用户的用户名（格式为 &lt;DomainName&gt;\\&lt;UserName&gt;）和密码。

2. 在客户端计算机上启动 HPC 作业管理器。

3. 在“选择头节点”对话框中，键入指向 Azure 中的头节点的 URL（例如 https://&lt;HeadNodeDnsName&gt;.chinacloudapp.cn 或 https://&lt;HeadNodeDnsName&gt;.&lt;region&gt;.chinacloudapp.cn)。

    HPC 作业管理器将会打开并显示头节点上的作业列表。

**使用在头节点上运行的 Web 门户**

1. 在客户端计算机上启动 Web 浏览器，键入以下内容之一，具体取决于头节点的完整 DNS 名称：

	
	    https://<HeadNodeDnsName>.chinacloudapp.cn/HpcPortal
	
	    https://<HeadNodeDnsName>.<region>.chinacloudapp.cn/HpcPortal
	    
2. 在出现的安全性对话框中，键入 HPC 群集管理员的域凭据。（你还可以添加具有不同角色的其他群集用户。请参阅[管理群集用户](https://technet.microsoft.com/zh-cn/library/ff919335.aspx)。）

    Web 门户将会打开并显示作业列表视图。

3. 若要从群集中提交返回“Hello World”字符串的示例作业，请在左侧导航区域中单击“新建作业”。

4. 在“新建作业”页面上的“从提交页面”下，单击“HelloWorld”。此时将显示作业提交页面。

5. 单击“提交”。出现提示时，请提供 HPC 群集管理员的域凭据。作业已提交，作业 ID 出现在“我的作业”页面上。

6. 若要查看你提交的作业的结果，请单击作业 ID，然后单击“查看任务”查看命令输出（在“输出”下方）。

## 后续步骤

* 你还可以使用 [HPC Pack REST API](http://social.technet.microsoft.com/wiki/contents/articles/7737.creating-and-submitting-jobs-by-using-the-rest-api-in-microsoft-hpc-pack-windows-hpc-server.aspx) 将作业提交到 Azure 群集。

* 若要从 Linux 客户端提交群集作业，请参阅 [HPC Pack 2012 R2 SDK 和示例代码](https://www.microsoft.com/download/details.aspx?id=41633)中的 Python 示例。


<!--Image references-->
[jobsubmit]: ./media/virtual-machines-windows-hpcpack-cluster-submit-jobs/jobsubmit.png

<!---HONumber=Mooncake_0314_2016-->