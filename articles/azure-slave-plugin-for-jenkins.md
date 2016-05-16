<properties
    pageTitle="如何在 Jenkins 持续集成中使用 Azure Slave 插件 | Azure"
    description="介绍如何在 Jenkins 持续集成中使用 Azure Slave 插件。"
	services="storage" 
	documentationCenter="java" 
	authors="rmcmurray" 
	manager="wpickett" 
	editor="jimbe" />

<tags
	ms.service="storage" 
	ms.date="04/08/2016"
	wacn.date="05/16/2016"/>

# 如何在 Jenkins 持续集成中使用 Azure Slave 插件

你可以使用适用于 Jenkins 的 Azure Slave 插件在运行分布式构建系统时，设置 Azure 上的从属节点。

## 安装 Azure Slave 插件
1. 在 Jenkins 仪表板中，单击“管理 Jenkins”。
2. 在“管理 Jenkins”页中，单击“管理插件”。
3. 单击“可用”选项卡。
4. 在可用插件列表上方的筛选器字段中，键入 **Azure** 将列表限制到相关插件。

	如果选择滚动可用插件列表，你会在“群集管理和分布式构建”部分找到 Azure Slave 插件。

5. 选中“Azure Slave 插件”复选框。
6. 单击“安装而不重新启动”或“立即下载并在重新启动后安装”。

现在插件安装完毕，接下来的步骤会是用你的 Azure 订阅配置文件配置插件并创建一个在为从属节点创建虚拟机时要使用的模板。


## 使用订阅配置文件配置 Azure Slave 插件

订阅配置文件也称为发布设置，是一个 XML 文件，包含在你的开发环境中使用 Azure 时需要的安全凭据以及一些附加信息。若要配置 Azure Slave 插件，你需要：

* 你的订阅 ID
* 用于你的订阅的管理证书

可以在你的[订阅配置文件](https://manage.windowsazure.cn/publishsettings/Index?SchemaVersion=2.0)中找到这些信息。以下是订阅配置文件的一个示例。

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

获取订阅配置文件后，请按照以下步骤配置 Azure Slave 插件：

1. 在 Jenkins 仪表板中，单击“管理 Jenkins”。
2. 单击“配置系统”。
3. 向下滚动页面以找到“云”部分。
4. 单击“添加新的云”> Microsoft Azure。

	![云部分](./media/azure-slave-plugin-for-jenkins/jenkins-cloud-section.png)

	这将显示你需要输入你的订阅详细信息的所在字段。

	![订阅配置](./media/azure-slave-plugin-for-jenkins/jenkins-account-configuration-fields.png)

5. 从你的订阅配置文件复制订阅 ID 和管理证书值，然后将其粘贴到相应的字段中。

	在复制订阅 ID 和管理证书时，不要包含将值括起来的引号。

6. 单击“验证配置”。
7. 配置验证正确后，单击“保存”。

## 为 Azure Slave 插件设置虚拟机模板

虚拟机模板定义插件在 Azure 上创建从属节点时使用的参数。在以下步骤中，我们将为 Ubuntu 虚拟机创建模板。

1. 在 Jenkins 仪表板中，单击“管理 Jenkins”。
2. 单击“配置系统”。
3. 向下滚动页面以找到“云”部分。

4. 在“云”部分中，找到“添加 Azure 虚拟机模板”，然后单击“添加”。

	![添加 VM 模板](./media/azure-slave-plugin-for-jenkins/jenkins-add-vm-template.png)

	这将显示你为所创建模板输入相关详细信息的所在字段。

	![空白的常规配置](./media/azure-slave-plugin-for-jenkins/jenkins-slave-template-general-configuration-blank.png)

5. 在“名称”框中输入 Azure 云服务的名称。如果你输入的名称引用一个现有的云服务，则将在该服务中设置虚拟机。否则，Azure 将创建一个新服务。

6. 在“说明”框中，输入描述所创建模板的文本。此信息供仅你参考，而不会在预配虚拟机时使用。
7. “标签”框用于标识所创建的模板，随后在创建 Jenkins 作业时用于引用该模板。对于本例，请在此框中输入 **linux**。
8. 在“区域”列表中，单击要在其中创建虚拟机的区域。
9. 在“虚拟机大小”列表中，单击适当的大小。
10. 在“存储帐户名称”框中，指定要在其中创建虚拟机的存储帐户。确保其所在区域与你要使用的云服务相同。如果你想要创建新的存储空间，可以将此框留空。
11. 保留时间指定 Jenkins 删除空闲从属节点之前所需要的分钟数。其保留默认值为 60。还可以选择在从属节点处于空闲状态时将其关闭而不是将其删除。为此，请选中“仅限在保留时间之后关闭(不删除)”复选框。
12. 在“使用情况”列表，选择使用此从属节点时的适当条件。现在，单击“尽可能多地使用此节点”。

	此时，你的窗体应类似于：

	![检查点常规模板配置](./media/azure-slave-plugin-for-jenkins/jenkins-slave-template-general-configuration.png)

	下一步是就你要创建从属节点所在的操作系统映像提供相关详细信息。

13. 在“映像系列或 ID”框中，你必须指定要在你的虚拟机中安装的系统映像。可以从映像系列的列表中选择，也可以指定一个自定义映像。

	如果要从映像系列的列表中选择，请输入映像系列名称的首字符（区分大小写）。例如，键入 **U** 将显示一个 Ubuntu Server 系列列表。从列表中选择后，Jenkins 将在预配虚拟机时使用该系列的最新版系统映像。

	![OS 映像列表示例](./media/azure-slave-plugin-for-jenkins/jenkins-os-family-list-sample.png)

	如果你有一个自定义映像，想要改用该映像，则输入该自定义映像的名称。自定义映像名称不显示在列表中，因此你必须确保名称输入正确。

	对于本教程，键入 **U** 显示一个 Ubuntu 映像列表，然后单击“Ubuntu Server 14.04 LTS”。

14. 在“启动方法”列表中，单击“SSH”。
15. 复制下面的脚本并粘贴到“Init 脚本”框中。

		# Install Java
		sudo apt-get -y update

		sudo apt-get install -y openjdk-7-jdk

		sudo apt-get -y update --fix-missing

		sudo apt-get install -y openjdk-7-jdk



		# Install git

		sudo apt-get install -y git



		#Install ant

		sudo apt-get install -y ant

		sudo apt-get -y update --fix-missing

		sudo apt-get install -y ant

	将在创建虚拟机后执行该 init 脚本。在本示例中，脚本将安装 Java、Git 和 ant。

16. 在“用户名”和“密码”框中，为要在你的虚拟机中创建的管理员帐户创建你的偏好值。
17. 单击“验证模板”以检查你指定的参数是否有效。
18. 单击“保存”。


## 创建在 Azure 的从属节点上运行的 Jenkins 作业

在这一部分，将创建一个在 Azure 上的从属节点上运行的 Jenkins 任务。你需要在 GitHub 上有自己的项目以便跟进。

1. 在 Jenkins 仪表板中，单击“新建项目”。
2. 为你正在创建的任务输入一个名称。
3. 对于项目类型，请单击“自由格式项目”。
4. 单击“确定”。
5. 在任务配置页中，选择“限制可以运行此项目的位置”。
6. 在“标签表达式”框中，输入 **linux**。在前一部分，我们创建了一个名为 **linux** 的从属模板，其名称与我们在这里指定的相同。
7. 在“构建”部分，单击“添加构建步骤”，然后选择“执行 shell”。
8. 编辑以下脚本，用相应的值替换 **(your GitHub account name)**、**(your project name)** 和 **(your project directory)**，然后将编辑后的脚本粘贴到显示的文本区域中。

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

9. 单击“保存”。
10. 在 Jenkins 仪表板中，将鼠标移到刚创建的任务上，然后单击下拉箭头以显示任务选项。
11. 单击“立即构建”。

之后，Jenkins 将使用在上一部分中创建的模板创建一个从属节点，并执行你在此任务的构建步骤中指定的脚本。

<!---HONumber=Mooncake_0509_2016-->