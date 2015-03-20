
<properties linkid="manage-windows-common-task-upload-vhd" urlDisplayName="Upload a VHD" pageTitle="创建 Windows Server VHD 并将其上载到 Azure" metaKeywords="Azure VHD, uploading VHD" description="了解如何在 Azure 中创建和上载包含 Windows Server 操作系统的虚拟硬盘 (VHD)。" metaCanonical="" services="virtual-machines" documentationCenter="" title="Creating and Uploading a Virtual Hard Disk that Contains the Windows Server Operating System" authors="kathydav" solutions="" manager="dongill" editor="tysonn" />




#创建 Windows Server VHD 并将其上载到 Azure#

本文介绍了如何使用操作系统上载虚拟硬盘 (VHD)，以便将其用作映像，从而创建基于该映像的虚拟机。有关 Windows Azure 中的磁盘和映像的详细信息，请参阅[关于 Azure 中的磁盘和映像](http://msdn.microsoft.com/zh-cn/library/windowsazure/jj672979.aspx)。

**注意**：创建虚拟机时，您可以自定义操作系统设置以快速运行您的应用程序。您设置的配置存储在该虚拟机的磁盘上。有关说明，请参阅[如何创建自定义虚拟机](/zh-cn/documentation/articles/virtual-machines-windows-tutorial/)。

##先决条件##
本文假定您拥有以下项目：

**Azure 订阅** - 如果没有帐户，只需花费几分钟就能创建一个免费试用帐户有关详细信息，请参阅[创建 Azure 帐户](/zh-cn/develop/php/tutorials/create-a-windows-azure-account/)。  

**Windows Azure PowerShell** - 您已经安装了 Windows Azure PowerShell 模块。若要下载该模块，请参阅 [Windows Azure 下载](/zh-cn/downloads/)。在[此处](/zh-cn/documentation/articles/install-configure-powershell/)可找到使用 Azure 订阅来安装和配置 PowerShell 的教程。

- 若要上载 VHD，必须使用 [Add-AzureVHD](http://msdn.microsoft.com/zh-cn/library/azure/dn205185.aspx) cmdlet，它是 Windows Azure PowerShell 模块的一部分。

**存储在 .vhd 文件中的受支持的 Windows 操作系统** - 您已将受支持的 Windows Server 操作系统安装到虚拟硬盘。可使用多个工具创建 .vhd 文件。可使用虚拟化解决方案（例如 Hyper-V）创建 .vhd 文件并安装操作系统。有关说明，请参阅[安装 Hyper-V 角色和配置虚拟机](http://technet.microsoft.com/zh-cn/library/hh846766.aspx)。

**重要说明**：Windows Azure 不支持 VHDX 格式。可使用 Hyper-V 管理器或 [Convert-VHD cmdlet](http://technet.microsoft.com/zh-cn/library/hh848454.aspx) 将磁盘转换为 VHD 格式。在[此处](http://blogs.msdn.com/b/virtual_pc_guy/archive/2012/10/03/using-powershell-to-convert-a-vhd-to-a-vhdx.aspx)可找到教程。
 
**Window Server 操作系统媒体。**此任务需要包含 Windows Server 操作系统的 .iso 文件。支持以下 Windows Server 版本：
<P>
  <TABLE BORDER="1" WIDTH="600">
  <TR BGCOLOR="#E9E7E7">
    <TH>OS</TH>
    <TH>SKU</TH>
    <TH>Service Pack</TH>
    <TH>Architecture</TH>
  </TR>
  <TR>
    <TD>Windows Server 2012 R2</TD>
    <TD>All editions</TD>
    <TD>N/A</TD>
    <TD>x64</TD>
  </TR>
  <TR>
    <TD>Windows Server 2012</TD>
    <TD>All editions</TD>
    <TD>N/A</TD>
    <TD>x64</TD>
  </TR>
  <TR>
    <TD>Windows Server 2008 R2</TD>
    <TD>All editions</TD>
    <TD>SP1</TD>
    <TD>x64</TD>
  </TR>
  </TABLE>
</P>


此任务包括以下步骤：

- [步骤 1：准备要上载的映像] []
- [步骤 2：在 Azure 中创建存储帐户] []
- [步骤 3：准备连接到 Azure] []
- [步骤 4：上载 .vhd 文件] []

## <a id="prepimage"> </a>步骤 1：准备要上载的映像 ##


在将映像上载到 Azure 之前，必须先使用 Sysprep 命令通用化。有关使用 Sysprep 的详细信息，请参阅[如何使用 Sysprep：简介](http://technet.microsoft.com/zh-cn/library/bb457073.aspx)。

请在您刚创建的虚拟机中，完成以下过程：

1. 登录到操作系统。

2. 以管理员身份打开"命令提示符"窗口。将目录更改为 **%windir%\system32\sysprep**，然后运行"sysprep.exe"。

	![Open Command Prompt window](./media/virtual-machines-create-upload-vhd-windows-server/sysprep_commandprompt.png)

3.	将显示"系统准备工具"****对话框。

	![Start Sysprep](./media/virtual-machines-create-upload-vhd-windows-server/sysprepgeneral.png)
4.  在"系统准备工具"****中，选择"进入系统全新体验 (OOBE)"****并确保选中"通用化"。
5.  在"关机选项"****中，选择"关机"****。
6.  单击"确定"****。 




## <a id="createstorage"> </a>步骤 2：在 Azure 中创建存储帐户 ##

存储帐户表示用于访问存储服务的最高级别的命名空间，并且与您的 Azure 订阅相关联。您需要在 Azure 中具有存储帐户才能将 .vhd 文件上载到可用于创建虚拟机的 Azure。可使用 Azure 管理门户创建存储帐户。

1. 登录到 Azure 管理门户。

2. 在命令栏上，单击"新建"****。

3. 单击"存储帐户"****，然后单击"快速创建"****。

	![Quick create a storage account](./media/virtual-machines-create-upload-vhd-windows-server/Storage-quick-create.png)

4. 如下所示填写字段：

	
	
- 在"URL"****下，键入要在存储帐户的 URL 中使用的子域名称。输入的名称可包含 3-24 个小写字母和数字。此名称将成为用于对订阅的 Blob、队列或表资源进行寻址的 URL 中的主机名。
			
- 选择存储帐户的**位置或地缘组**。通过指定地缘组，可将同一数据中心内的云服务与您的存储放置在一起。
		 
- 决定是否对存储帐户使用**地域复制**。默认情况下启用地域复制。此选项会将您的数据免费复制到辅助位置，以便在发生无法在主要位置进行处理的严重故障时将您的存储故障转移到辅助位置。将自动分配辅助位置，并且无法对其进行更改。如果法律要求或组织政策要求更加严格地控制基于云的存储的位置，则您可以关闭地域复制。但是，请注意，如果稍后您打开地域复制，则将现有数据复制到辅助位置时将向您收取一次性数据传输费用。不具有地域复制的存储服务将以优惠价提供。有关管理存储帐户的地域复制的详细信息，请参阅：[创建、管理或删除存储帐户](../storage-create-storage-account/#replication-options)。

	![Enter storage account details](./media/virtual-machines-create-upload-vhd-windows-server/Storage-create-account.png)

5. 单击"创建存储帐户"****。

	该帐户现在会出现在"存储帐户"****下。

	![Storage account successfully created](./media/virtual-machines-create-upload-vhd-windows-server/Storagenewaccount.png)

6. 接下来，为您上载的 VHD 创建一个容器。单击"存储帐户名称"****，然后单击"容器"****。

	![Storage account detail](./media/virtual-machines-create-upload-vhd-windows-server/storageaccount_detail.png)

7. 单击"创建容器"****。

	![Storage account detail](./media/virtual-machines-create-upload-vhd-windows-server/storageaccount_container.png)

8. 为您的容器键入"名称"****并选择"访问策略"****。

	![Container name](./media/virtual-machines-create-upload-vhd-windows-server/storageaccount_containervalues.png)

	> [WACOM.NOTE] 默认情况下，容器是私有的，并且只能由帐户所有者访问。若要允许对容器中的 Blob 进行公共读取访问，但不允许对容器属性和元数据进行公共读取访问，请使用"公共 Blob"选项。若要允许对容器和 Blob 进行完全公共读取访问，请使用"公共容器"选项。

## <a id="PrepAzure"> </a>步骤 3：准备连接到 Windows Azure ##

您首先需要在计算机和 Windows Azure 中的订阅之间建立一个安全连接，然后才能上载 .vhd 文件。您可以使用 Windows Azure Active Directory 方法或证书方法来执行此操作。

<h3>使用 Windows Azure AD 方法</h3>

1. 打开 Windows Azure PowerShell 控制台，请按照[如何：安装 Windows Azure PowerShell](#Install) 中的说明操作。

2. 键入以下命令：  
	`Add-AzureAccount`
	
	此命令将打开一个登录窗口，以便您可以使用工作或学校帐户登录。

	![PowerShell Window](./media/virtual-machines-create-upload-vhd-windows-server/add_azureaccount.png)

3. Windows Azure 将对凭据信息进行身份验证和保存，然后关闭该窗口。

<h3>使用证书方法</h3> 

1. 打开 Windows Azure PowerShell 窗口。 

2.	类型： 
	`Get-AzurePublishSettingsFile`.


3. 浏览器窗口将打开并提示您下载 .publishsettings 文件。它包含您的 Windows Azure 订阅的信息以及证书。

	![Browser download page](./media/virtual-machines-create-upload-vhd-windows-server/Browser_download_GetPublishSettingsFile.png)

3. 保存 .publishsettings 文件。 

4. 类型： 
	`Import-AzurePublishSettingsFile <PathToFile>`

	其中，"<PathToFile>"是 .publishsettings 文件的完整路径。 


	有关详细信息，请参阅 [Windows Azure Cmdlet 入门](http://msdn.microsoft.com/zh-cn/library/azure/jj554332.aspx) 
	
	有关安装和配置 PowerShell 的详细信息，请参阅[如何安装和配置 Windows Azure PowerShell](/zh-cn/documentation/articles/install-configure-powershell/) 


## <a id="upload"> </a>步骤 4：上载 .vhd 文件 ##

在上载 .vhd 文件时，您可以将 .vhd 文件放置在 Blob 存储中的任何位置。在以下命令示例中，**BlobStorageURL** 是您在步骤 2 中创建的存储帐户的 URL，**YourImagesFolder** 是要在其中存储映像的 Blob 存储中的容器。**VHDName** 是显示在管理门户中用于标识虚拟硬盘的标签。**PathToVHDFile** 是 .vhd 文件的完整路径和名称。 


1. 从您在上一步中使用的 Windows Azure PowerShell 窗口中，键入：

	`Add-AzureVhd -Destination "<BlobStorageURL>/<YourImagesFolder>/<VHDName>.vhd" -LocalFilePath <PathToVHDFile>`
	
	![PowerShell Add-AzureVHD](./media/virtual-machines-create-upload-vhd-windows-server/powershell_upload_vhd.png)

	有关 Add-AzureVhd cmdlet 的详细信息，请参阅 [Add-AzureVhd](http://msdn.microsoft.com/zh-cn/library/dn495173.aspx)。

##将映像添加到自定义映像列表 ##
上载 .vhd 后，将其作为映像添加到与订阅关联的自定义映像列表。

1. 从管理门户中的"所有项目"****下，单击"虚拟机"****。

2. 在"虚拟机"下，单击"映像"****。

3. 然后单击"创建映像"****。

	![PowerShell Add-AzureVHD](./media/virtual-machines-create-upload-vhd-windows-server/Create_Image.png)

4. 在"从 VHD 创建映像"****中，执行以下操作：
 	
	- 指定"名称"****
	- 指定"说明"****
	- 若要指定"VHD 的 URL"****，请单击文件夹按钮，以启动下面的对话框 
	![Select VHD](./media/virtual-machines-create-upload-vhd-windows-server/Select_VHD.png)
	- 选中您的 VHD 所在的存储帐户并单击"打开"****。这样您会回到"从 VHD 创建映像"****窗口。
	- 返回到"从 VHD 创建映像"****窗口后，请选择"操作系统系列"。
	- 选中"我已在虚拟机上运行与此 VHD 关联的 Sysprep"****来确认您已在步骤 1 中对操作系统进行通用化，然后单击"确定"****。 

	![Add Image](./media/virtual-machines-create-upload-vhd-windows-server/Create_Image_From_VHD.png)

5. **可选：**您也可以使用 Azure PowerShell 的 Add-AzureVMImage cmdlet 将 VHD 添加为映像。

	从 Windows Azure PowerShell 窗口中，键入：

	`Add-AzureVMImage -ImageName <Your Image's Name> -MediaLocation <location of the VHD> -OS <Type of the OS on the VHD>`
	
	![PowerShell Add-AzureVMImage](./media/virtual-machines-create-upload-vhd-windows-server/add_azureimage_powershell.png)

6. 完成前面的步骤后，当您选择"映像"****选项卡时，新的映像将列出。 


	![custom image](./media/virtual-machines-create-upload-vhd-windows-server/vm_custom_image.png)

	创建新虚拟机后，现在您可以使用此新映像。选择"我的映像"****以显示新映像。有关说明，请参阅[创建运行 Windows Server 的虚拟机](/zh-cn/documentation/articles/virtual-machines-windows-tutorial/)。

	![create VM from custom image](./media/virtual-machines-create-upload-vhd-windows-server/create_vm_custom_image.png)

## 后续步骤 ##
 

创建虚拟机后，尝试创建 SQL Server 虚拟机。有关说明，请参阅[在 Windows Azure 上设置 SQL Server 虚拟机](/zh-cn/documentation/articles/virtual-machines-provision-sql-server/)。 

[步骤 1：准备要上载的映像]: #prepimage
[步骤 2：在 Azure 中创建存储帐户]: #createstorage
[步骤 3：准备连接到 Azure]: #prepAzure
[步骤 4：上载 .vhd 文件]: #upload
