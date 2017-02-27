<properties
    pageTitle="Azure 中 Linux 虚拟机上的 Microsoft R Server 2016 (9.0.1)"
    description="Azure 中 Linux 虚拟机上的 Microsoft R Server 2016 (9.0.1)"
    keywords="Microsoft R Server"
    services="virtual-machines-linux"
    documentationcenter=""
    tags=""
    author="j-martens"
    manager=""
    editor="j-martens" />
<tags
    ms.service="virtual-machines-linux"
    ms.workload=""
    ms.tgt_pltfrm="na"
    ms.devlang="na"
    ms.topic=""
    ms.date="01/14/2017"
    wacn.date="02/27/2017"
    ms.author="j-martens" />  


# Azure 中 Linux 虚拟机上的 Microsoft R Server 2016 \(9.0.1\)

Microsoft R Server 是目前适用于 R 的可部署范围最广的企业级分析平台。此虚拟机 \(VM\) 包含适用于 Linux（CentOS 版本 7.2 或 Ubuntu 版本 16.04）的 Microsoft R Server 2016（9.0.1 版）。以下网页提供了有关此版本的详细信息：[https://msdn.microsoft.com/zh-cn/microsoft-r/rserver-install-linux-server](https://msdn.microsoft.com/zh-cn/microsoft-r/rserver-install-linux-server)。

此 VM 还包含：

* **mrsdeploy**：用于在应用程序和后端系统中部署 R 分析的新 R 包。[了解详细信息...](https://msdn.microsoft.com/zh-cn/microsoft-r/mrsdeploy/mrsdeploy)
* **MicrosoftML**：新的“Microsoft 机器学习算法”R 包，其中包含的函数可将机器学习整合到在 R Server for Windows、R Client for Windows 和 SQL Server R Services 上执行的 R 代码或脚本。适用于 Linux、Hadoop 和 Azure HDInsight 的版本已计划在 2017 年第一季度推出。[了解详细信息...](https://msdn.microsoft.com/zh-cn/microsoft-r/microsoftml/microsoftml)

## 预配 R Server 虚拟机

如果你是 Azure VM 的新手，我们建议查看[此文](/documentation/services/virtual-machines/linux/)，了解有关使用门户和配置虚拟机的详细信息。

### 若要在 Linux VM 上创建 Microsoft R Server，请执行以下操作：

1. 转到 Azure 应用商店。
2. 在搜索框中键入“Linux 上的 Microsoft R Server”。
3. 选择“Linux 上的 Microsoft R Server”。
4. 单击“创建虚拟机”。
5. 接受条款，然后单击“创建”开始创建。
6. 使用屏幕上的提示和字段配置 R Server VM。

    >[AZURE.NOTE]
    需要使用一个 Azure 订阅来创建 VM。
    <p>如果你不熟悉 Azure 上的虚拟机，请[在此处了解相关过程](/documentation/services/virtual-machines/linux/)。

7. 部署并运行 VM 后，请[连接](#connect)到该 VM，开始与 R Server 交互。
8. 此时，还可以：
    * [安装 R IDE](#ride)
    * 配置 R Server 的[操作化](#o16n)，使其能够充当部署服务器和主机分析 Web 服务。

## <a name="connect"></a>连接到虚拟机

很多人都可使用开源 SSH 客户端成功连接。

在 Azure 门户上使用 VM 属性页中列出的公共 IP 地址即可建立连接。

启动后，将会转到终端窗口中的根目录。

## 启动 Microsoft R Server

若要启动 Microsoft R Server，只需在命令提示符下键入 `R`。此时将启动 R Server 控制台会话。

## <a name="ride"></a>配置 R IDE

安装 Microsoft R Server 后，可将偏好的 R 集成开发环境 \(IDE\) 配置为指向 Microsoft R Server R 可执行文件。这样，始终都可以使用 Microsoft R Server 执行 R 代码，享受其专属包的优势。R Server 也适用于流行的 IDE，例如 [RStudio](https://www.rstudio.com/) Desktop 或 Server。

### 配置 Microsoft R Server 的 RStudio

1. 启动 RStudio。
2. [将路径更新为 R](https://support.rstudio.com/hc/zh-cn/articles/200486138-Using-Different-Versions-of-R)。
3. 启动 RStudio 时，Microsoft R Server 将成为默认 R 引擎。

### 打开所需的端口以使用 RStudio Server

RStudio Server 使用端口 8787。Azure VM 的默认配置不会打开此端口。若要打开此端口，必须转到 Azure 门户并选择适当的网络安全组。选择“所有设置”选项，然后选择“入站安全规则”。为 RStudio 添加一个新规则。为规则命名，为“协议”选择“任何”，然后将端口 8787 添加到目标端口范围。单击“确定”以保存你的更改。现在应该能够使用浏览器访问 RStudio。

### 向 VM 分配完全限定的域名以访问 RStudio Server

系统不会创建云服务来包含 VM 的公共资源，因此，默认情况下，没有向动态公共 IP 分配完全限定的域名。可以创建一个完全限定的域名，并在部署后使用 Azure PowerShell 将其添加到映像。主机名的格式为 `domainnamelabel; region;.cloudapp.azure.com`。

例如，使用 PowerShell 为包含资源组 `rservercloudrg` 和所需主机名 `rservercloud` 的、名为 `rservercloudvm` 的 VM 添加公共主机名。

```powershell
PS C:\\Users\\juser> Select-AzureSubscription -SubscriptionName "Visual Studio Ultimate with MSDN" –Current

PS C:\\Users\\juser> Switch-AzureMode -Name AzureResourceManager

PS C:\\Users\\juser> New-AzurePublicIpAddress -Name rservercloudvm -ResourceGroupName rservercloudrg -Location "China East" -DomainNameLabel rservercloud -AllocationMethod Dynamic
```

在入站安全规则中添加对端口 TCP/8787 的访问权限后，可通过 http://rservercloud.chinaeast.chinacloudapp.cn:8787/ 访问 RStudio Server

相关文章包括：

* [Azure Resource Manager 部署模型中适用于 Windows 应用程序的 Azure 计算、网络和存储提供程序](/documentation/articles/resource-manager-deployment-model/)
* [使用 ARM PowerShell cmdlet 创建 Azure VM](http://blogs.msdn.com/b/cloud_solution_architect/archive/2015/05/05/creating-azure-vms-with-arm-powershell-cmdlets.aspx)

## <a name="o16n"></a>在 VM 上配合 R Server 操作 R Analytics

若要利用 Microsoft R Server 的部署和操作化功能，可在安装后[配置 R Server 的操作化](https://msdn.microsoft.com/zh-cn/microsoft-r/operationalize/configuration-initial)，使其能够充当部署服务器和主机分析 Web 服务。

## 访问 Azure 存储帐户中的数据

需要使用 Azure 存储帐户中的数据时，可通过多个选项来访问或移动数据：

* 使用 [AzCopy](/documentation/articles/storage-use-azcopy/) 等实用工具将数据从存储帐户复制到本地文件系统。
* 将文件添加到存储帐户中的某个文件共享，然后将该文件共享装载为 VM 上的网络驱动器。有关详细信息，请参阅[装载 Azure 文件](/documentation/articles/storage-how-to-use-files-linux/)。

## 使用 Azure Data Lake Storage \(ADLS\) 帐户中的数据
可以使用 `RevoscaleR` 包从 ADLS 存储中读取数据。只需像使用 `webHDFS` 引用 HDFS 文件系统一样引用存储帐户。有关详细信息，请参阅此[设置指南](http://go.microsoft.com/fwlink/?LinkId=723452)。

## 文档和资源

可以使用左侧的目录，在本 MSDN 库中找到有关 Microsoft R 的其他文档。

请参阅以下附加资源，了解有关 R 的一般信息：

* [DataCamp](http://www.datacamp.com/)：有关 R 的免费临时简介课程，此外还包括一套有关使用 Revolution R 处理大数据的课程
* [交叉验证](https://stats.stackexchange.com/)：可在该站点上针对机器学习中的统计问题提问
* [R 帮助邮件列表](https://www.r-project.org/mail.html)：该列表及其存档提供丰富的历史信息资源
* [MRAN 网站](https://mran.microsoft.com/documents/getting-started/)：其他许多 R 资源。

<!---HONumber=Mooncake_0213_2017-->