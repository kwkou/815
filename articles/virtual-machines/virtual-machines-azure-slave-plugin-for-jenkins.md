<!-- need to be verified -->

<properties
    pageTitle="如何将 Azure 从属插件用于 Jenkins 持续集成 | Azure"
    description="介绍如何将 Azure 从属插件用于 Jenkins 持续集成。"
    services="virtual-machines-linux"
    documentationcenter=""
    author="rmcmurray"
    manager="erikre"
    editor="" />
<tags 
    ms.assetid="09233bd4-957d-41bf-bccc-9dd2355ed1bf"
    ms.service="virtual-machines-linux"
    ms.workload="infrastructure-services"
    ms.tgt_pltfrm="vm-multiple"
    ms.devlang="java"
    ms.topic="article"
    ms.date="10/19/2016"
    wacn.date="12/20/2016"
    ms.author="robmcm" />

# 如何将 Azure 从属插件用于 Jenkins 持续集成
运行分布式生成时，可以使用适用于 Jenkins 的 Azure 从属插件在 Azure 上轻松预配从属节点，此外，该插件支持以下操作：

* 使用 SSH 和 Java 网络启动协议 (JNLP) 在 Azure 云中创建 Windows 从属节点
  
  * 若要通过 SSH 启动某个 Windows 映像，需要使用 SSH 预先配置该映像。
  * 有关准备自定义 Windows 映像的信息，请参阅[如何在 Resource Manager 部署模型中捕获 Windows 虚拟机][windows-image-capture]。
* 使用 SSH 在 Azure 云中创建 Linux 从属节点
  
  * 有关准备自定义 Linux 映像的信息，请参阅[如何捕获 Linux 虚拟机以用作 Resource Manager 模板][linux-image-capture]。

### 先决条件
执行本文中的步骤之前，需要注册和授权客户端应用程序，然后检索客户端 ID 和客户端密钥，在身份验证期间会将这些信息发送到 Azure Active Directory。有关这些先决条件的详细信息，请参阅以下文章：

* [将应用程序与 Azure Active Directory 集成][integrate-apps-with-AAD]
* [注册客户端应用][register-client-app]

此外，若要完成本文[创建在 Azure 上的从属节点中运行的 Jenkins 作业](#create-jenkins-project)部分中的步骤，需要在 GitHub 中设置一个项目。

## <a name="install-azure-slave-plugin"></a>安装 Azure 从属插件
1. 在 Jenkins 仪表板中，单击“管理 Jenkins”。
   
    ![管理 Jenkins][jenkins-dashboard-manage]  

2. 在“管理 Jenkins”页中，单击“管理插件”。
   
    ![管理插件][jenkins-manage-plugins]  

3. 单击“可用”选项卡，输入“Azure”作为筛选器，然后选择“Azure 从属插件”。
   
    ![Azure 从属插件][search-plugins]  

   
    如果滚动浏览可用插件列表，可在“群集管理和分布式生成”部分下面看到列出的 Azure 从属插件。
4. 单击“安装而不重新启动”或“立即下载并在重新启动后安装”。
   
    ![安装插件][install-plugin]  


安装插件后，接下来的步骤是使用 Azure 订阅配置文件配置插件并创建一个在为从属节点创建虚拟机时要使用的模板。

## 将 Azure 从属插件配置为使用你的订阅配置文件
订阅配置文件也称为发布设置，是一个 XML 文件，包含在你的开发环境中使用 Azure 时需要的安全凭据以及一些附加信息。若要配置 Azure 从属插件，需要提供以下各项：

* 订阅 ID
* 订阅的管理证书

这些信息可在[订阅配置文件]中找到，如以下示例所示：

        <?xml version="1.0" encoding="utf-8"?>
        <PublishData>
            <PublishProfile SchemaVersion="2.0" PublishMethod="AzureServiceManagementAPI">
                <Subscription
                    ServiceManagementUrl="https://management.core.chinacloudapi.cn"
                    Id="<Subscription ID value>"
                    Name="Pay-As-You-Go"
                    ManagementCertificate="<Management certificate value>" />
            </PublishProfile>
        </PublishData>

获取订阅配置文件后，请遵循以下步骤配置 Azure 从属插件：

1. 在 Jenkins 仪表板中，单击“管理 Jenkins”。
   
    ![管理 Jenkins][jenkins-dashboard-manage]  

2. 单击“配置系统”。
   
    ![配置系统][jenkins-configure-system]  

3. 向下滚动页面以找到“云”部分。
4. 单击“添加新云”，然后选择“Azure”。
   
    ![添加新云][cloud-section]  

    此时将显示需要在其中输入订阅详细信息的字段。
   
    ![订阅配置][subscription-configuration]  

    >[AZURE.NOTE] 若要使用“Azure 中国区”，需要单击“...高级”，并将 “https://management.core.windows.net” 替换为 “https://management.core.chinacloudapi.cn” 。

5. 从你的订阅配置文件复制订阅 ID 和管理证书值，然后将其粘贴到相应的字段中。
   
    在复制订阅 ID 和管理证书时，不要包含将值括起来的引号。
6. 单击“验证配置”检查指定的参数是否有效，然后单击“保存”。

## 为 Azure 从属插件设置虚拟机模板
在本部分，我们将添加一个虚拟机模板，用于定义 Azure 从属插件在 Azure 上创建从属节点时使用的参数。以下步骤将为 Ubuntu 虚拟机创建模板。

1. 在 Jenkins 仪表板中，单击“管理 Jenkins”。
   
    ![管理 Jenkins][jenkins-dashboard-manage]  

2. 单击“配置系统”。
   
    ![配置系统][jenkins-configure-system]  

3. 向下滚动页面以找到“云”部分。
4. 在“云”部分中，找到在上一部分中添加的云所对应的“添加 Azure 虚拟机模板”，然后单击“添加”。
   
    ![添加 Azure 虚拟机模板][add-vm-template]  

    Jenkins 将显示一个表单，可在该表单的字段中输入有关所要创建的模板的详细信息。
   
    ![空白常规配置][blank-general-configuration]  

5. 输入模板的“常规配置”信息：
   
   1. 在“名称”框中，输入新模板的名称；该名称仅供你自己使用，而不是会在预配虚拟机时使用。
   2. 在“说明”框中，输入用于说明所创建模板的文本。此信息供仅你参考，而不会在预配虚拟机时使用。
   3. “标签”框用于标识所创建的模板，随后在创建 Jenkins 作业时用于引用该模板。对于本示例，请在此框中输入 **linux**。
   4. 在“区域”列表中，单击要在其中创建虚拟机的 Azure 区域。
   5. 在“虚拟机大小”列表中，通过下拉菜单选择适当的大小。
   6. 在“存储帐户名称”框中，指定要在其中创建虚拟机的存储帐户。如果希望 Jenkins 使用默认值“jenkinsarmst”创建存储帐户，可以将此框留空。
   7. “保留时间”指定 Jenkins 在经过多少分钟后删除空闲从属节点；请在此字段中保留默认值 60。
      
      * 还可以选择在从属节点处于空闲状态时将其关闭而不是将其删除。为此，请选中“仅限在保留时间之后关闭(不删除)”复选框。
      * 如果不希望自动删除空闲从属节点，也可以指定 0。
   8. 在“用法”列表中，可从以下选项中选择：
      
      * **尽量使用此节点** - 只要从属节点可用，Jenkins 就可以在该节点上运行任何作业。
      * **仅将此节点保留给绑定的作业** - 仅当某个项目已专门绑定到此节点时，Jenkins 才在该节点上生成项目（或作业）；这样，便可以将从属节点保留给特定类型的作业使用。
      
      对于本教程，请单击“尽量使用此节点”。
      
      此时，窗体应类似于：
      
      ![常规模板配置][general-template-config]  

6. 为模板提供**映像配置**：
   
   1. 对于映像系列，可以使用两个选项：
      
      * **自定义用户映像** - 此选项要求提供要在其中创建从属节点的同一存储帐户中的自定义图像的 URL。
      * **映像引用** - 此选项要求指定映像的“发布者”、“产品”和“SKU”，可以在 Azure 虚拟机应用商店中找到这些信息。
        
        对于本教程，请选择“映像引用”并使用以下值：
      * **映像发布者**：Canonical
      * **映像产品**：UbuntuServer
      * **映像 SKU**：14.04.3-LTS
   2. 对于“启动方法”列表，可以使用两个选项：“SSH”或“JNLP”。对于本教程，请选择“SSH”。但是，在选择启动方法时，需要注意以下几个事项：
      
      * Linux 从属节点只能使用 SSH 启动。
      * Windows 从属节点可以使用 SSH 或 JNLP，但将 SSH 用于 Windows 从属节点时，必须使用 SSH 服务器对映像进行自定义准备。
        
        如果使用 JNLP 作为启动方法，需确保配置以下各项：
      * Azure 从属节点需要能够访问 Jenkins URL，因此请相应地配置所有防火墙。依次单击“管理 Jenkins”、“配置系统”，然后查看“Jenkins 位置”部分即可找到 Jenkins URL。
   3. 使用 JNLP 启动的 Azure 从属节点需要能够访问 Jenkins TCP 端口；因此，建议使用固定的 TCP 端口，以便可以相应地配置所有防火墙。若要指定 Jenkins TCP 端口，可以依次单击“管理 Jenkins”、“配置全局安全性”，选中“启用安全性”对应的选项并并用一个**固定**端口，然后指定要使用的端口。
   4. 对于 **Init 脚本**，需要提供创建虚拟机后所要执行的初始化脚本，同时请注意以下事项：
      
      * 该脚本最起码要安装 Java。
      * 如果使用 JNLP，必须在 PowerShell 中编写初始化脚本。
      * 如果执行初始化脚本预计需要花费较长的时间，（例如，要安装大量的包），则我们建议准备一个自定义映像并在其中预装所有必要的软件，而不要通过初始化脚本安装这些软件。
        
        对于本教程，请复制以下脚本并将其粘贴到“Init 脚本”框中：
        
              # Install Java
              sudo apt-get -y update
              sudo apt-get install -y openjdk-7-jdk
              sudo apt-get -y update --fix-missing
              sudo apt-get install -y openjdk-7-jdk
              # Install git
              sudo apt-get install -y git
              # Install ant
              sudo apt-get install -y ant
              sudo apt-get -y update --fix-missing
              sudo apt-get install -y ant
        
        在上面的示例中，初始化脚本将安装 *Java* 、 *Git* 和 *ant*。
   5. 在“用户名”和“密码”框中，为要在虚拟机中创建的管理员帐户创建首选值。
      
      此时，窗体应类似于：
      
      ![Jenkins 映像配置][jenkins-image-configuration]  

7. 单击“验证模板”检查指定的参数是否有效，然后单击“保存”。
   
    ![Jenkins - 保存模板][jenkins-save-template]  


## <a name="create-jenkins-project"></a>创建在 Azure 中的从属节点上运行的 Jenkins 作业
在这一部分，将创建一个在 Azure 上的从属节点上运行的 Jenkins 任务。若要完成这些步骤，需在 GitHub 中设置一个项目。

1. 在 Jenkins 仪表板中，单击“新建项目”。
   
    ![Jenkins 仪表板 - 新建项][jenkins-dashboard-new-item]  

2. 输入所要创建的任务的名称，选择“自由风格项目”作为项目类型，然后单击“确定”。
   
    ![Jenkins - 创建新项][jenkins-create-new-item]  

3. 在任务配置页中，选择“限制可以运行此项目的位置”，在“标签表达式”框中输入 **linux**；此值与在上一部分中创建的从属模板的标签匹配。
   
    ![Jenkins - 新项限制][jenkins-new-item-restrict]  

4. 在“构建”部分，单击“添加构建步骤”，然后选择“执行 shell”。
   
    ![Jenkins - 新项生成][jenkins-new-item-build]  

5. 编辑以下脚本，用相应的值替换 **(your GitHub account name)**、**(your project name)** 和 **(your project directory)**，然后将编辑后的脚本粘贴到显示的文本区域中。
   
            # Clone from git repo
            currentDir="$PWD"
            if [ -e (your project directory) ]; then
                cd (your project directory)
                git pull origin master
            else
                git clone https://github.com/(your GitHub account name)/(your project name).git
            fi
            # change directory to project
            cd $currentDir/(your project directory)
            #Execute build task
            ant
   
    表单应类似于以下示例：
   
    ![Jenkins - 新项脚本][jenkins-new-item-script]  

6. 完成所有配置步骤后，单击“保存”。
   
    ![Jenkins - 新项保存][jenkins-new-item-save]  

7. 在 Jenkins 仪表板中，单击刚刚创建的项目旁边的下拉箭头，然后单击“立即生成”。
   
    ![Jenkins - 立即生成][jenkins-build-now]  


Jenkins 将使用在上一部分中创建的模板创建一个从属节点，然后执行在此任务的生成步骤中指定的脚本。

## <a name="image-template-considerations"></a>使用映像模板时的注意事项
以下部分包含有关配置各种模板的一些有用信息。

### 使用 Ubuntu 映像模板时
* 如果执行初始化脚本预计需要花费较长的时间，（例如，要安装大量的包），则我们建议准备一个自定义 Ubuntu 映像并在其中预装所有必要的软件，而不要通过初始化脚本安装这些软件。
* 该脚本最起码要安装 Java。如前面的示例所示，可以使用以下脚本来安装 Java、Git 和 ant。
  
        # Install Java
        sudo apt-get -y update
        sudo apt-get install -y openjdk-7-jdk
        sudo apt-get -y update --fix-missing
        sudo apt-get install -y openjdk-7-jdk
        # Install git
        sudo apt-get install -y git
        # Install ant
        sudo apt-get install -y ant
        sudo apt-get -y update --fix-missing
        sudo apt-get install -y ant

### 使用 Windows 映像模板和 JNLP 启动方法时
* 如果 Jenkins 主节点中未配置安全性：
  
  * 将“Init 脚本”字段留空，以便在从属节点上执行默认脚本。
* 如果 Jenkins 主节点中已配置安全性：
  
  * 复制 [Windows 从属节点设置][windows-slaves-setup]中的脚本，然后使用你的 Jenkins 凭据修改该脚本。
  * 最起码需要使用 Jenkins 用户 ID 和 API 令牌来修改该脚本。若要检索用户的 API 令牌，请执行以下步骤：
    
    1. 在 Jenkins 仪表板中，单击“人员”。
    2. 单击相应的用户帐户。
    3. 在左侧菜单中单击“配置”。
    4. 单击“显示 API 令牌...”。

## <a name="see-also"></a> 另请参阅
有关将 Azure 与 Java 配合使用的详细信息，请参阅 [Azure Java 开发人员中心]。

有关适用于 Jenkins 的 Azure 从属插件的更多信息，请参阅 GitHub 上的 [Azure 从属插件]项目。

<!-- URL List -->


[Azure Java 开发人员中心]: /develop/java/
[Azure 从属插件]: https://github.com/jenkinsci/azure-slave-plugin/
[订阅配置文件]: https://manage.windowsazure.cn/publishsettings/Index?SchemaVersion=2.0
[integrate-apps-with-AAD]: /documentation/articles/active-directory-integrating-applications/
[register-client-app]: http://msdn.microsoft.com/dn877542.aspx
[windows-slaves-setup]: https://gist.github.com/snallami/5aa9ea2c57836a3b3635

<!-- IMG List -->


[jenkins-dashboard-manage]: ./media/virtual-machines-azure-slave-plugin-for-jenkins/jenkins-dashboard-manage.png
[jenkins-manage-plugins]: ./media/virtual-machines-azure-slave-plugin-for-jenkins/jenkins-manage-plugins.png
[jenkins-configure-system]: ./media/virtual-machines-azure-slave-plugin-for-jenkins/jenkins-configure-system.png
[jenkins-dashboard-new-item]: ./media/virtual-machines-azure-slave-plugin-for-jenkins/jenkins-dashboard-new-item.png
[search-plugins]: ./media/virtual-machines-azure-slave-plugin-for-jenkins/search-for-azure-plugin.png
[install-plugin]: ./media/virtual-machines-azure-slave-plugin-for-jenkins/install-plugin.png
[jenkins-create-new-item]: ./media/virtual-machines-azure-slave-plugin-for-jenkins/jenkins-create-new-item.png
[cloud-section]: ./media/virtual-machines-azure-slave-plugin-for-jenkins/jenkins-cloud-section.png
[subscription-configuration]: ./media/virtual-machines-azure-slave-plugin-for-jenkins/jenkins-account-configuration-fields.png
[add-vm-template]: ./media/virtual-machines-azure-slave-plugin-for-jenkins/jenkins-add-vm-template.png
[blank-general-configuration]: ./media/virtual-machines-azure-slave-plugin-for-jenkins/jenkins-slave-template-general-configuration-blank.png
[general-template-config]: ./media/virtual-machines-azure-slave-plugin-for-jenkins/jenkins-slave-template-general-configuration.png
[jenkins-new-item-restrict]: ./media/virtual-machines-azure-slave-plugin-for-jenkins/jenkins-new-item-restrict.png
[jenkins-new-item-build]: ./media/virtual-machines-azure-slave-plugin-for-jenkins/jenkins-new-item-build.png
[jenkins-new-item-script]: ./media/virtual-machines-azure-slave-plugin-for-jenkins/jenkins-new-item-script.png
[jenkins-new-item-save]: ./media/virtual-machines-azure-slave-plugin-for-jenkins/jenkins-new-item-save.png
[jenkins-build-now]: ./media/virtual-machines-azure-slave-plugin-for-jenkins/jenkins-build-now.png
[jenkins-image-configuration]: ./media/virtual-machines-azure-slave-plugin-for-jenkins/jenkins-image-configuration.png
[jenkins-save-template]: ./media/virtual-machines-azure-slave-plugin-for-jenkins/jenkins-save-template.png
[windows-image-capture]: /documentation/articles/virtual-machines-windows-classic-capture-image/
[linux-image-capture]: /documentation/articles/virtual-machines-linux-capture-image/

<!---HONumber=Mooncake_1212_2016-->