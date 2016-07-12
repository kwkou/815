<properties
	pageTitle="使用 Powershell 创建和上载 Windows Server VHD | Azure"
	description="了解如何使用经典部署模型和 Azure Powershell 创建和上载基于 Windows Server 的虚拟硬盘 (VHD)。"
	services="virtual-machines-windows"
	documentationCenter=""
	authors="cynthn"
	manager="timlt"
	editor="tysonn"
	tags="azure-service-management"/>

<tags
	ms.service="virtual-machines-windows"
	ms.date="04/15/2016"
        wacn.date="05/24/2016"/>

# 创建 Windows Server VHD 并将其上载到 Azure

本文介绍如何上载包含操作系统的虚拟硬盘 (VHD)，以便可以使用它作为映像来基于该映像创建虚拟机。有关 Azure 中的磁盘和 VHD 的更多详细信息，请参阅[关于虚拟机的磁盘和 VHD](/documentation/articles/virtual-machines-windows-about-disks-vhds/)。

> [AZURE.IMPORTANT]Azure 具有用于创建和处理资源的两个不同的部署模型：[资源管理器和经典](/documentation/articles/resource-manager-deployment-model/)。本文介绍使用经典部署模型。Azure 建议大多数新部署使用资源管理器模型。

## 先决条件

本文假定你具备以下条件：

1. **Azure 订阅** - 如果你还没有，可以[建立一个 1 RMB 的 Azure 帐户](/pricing/1rmb-trial/?WT.mc_id=A261C142F)：获取可用来试用付费版 Azure 服务的信用额度，甚至在用完信用额度后，你仍可以保留帐户和使用 1 RMB 的 Azure 服务（如 Web 应用）。

2. **Azure PowerShell** - 已安装 Azure PowerShell 模块并将其配置为使用你的订阅。若要下载该模块，请参阅 [Azure 下载](/downloads/)。可在[此处](/documentation/articles/powershell-install-configure/)获取安装和配置该模块的教程。可使用 [Add-AzureVHD](http://msdn.microsoft.com/zh-cn/library/azure/dn495173.aspx) cmdlet 上载 VHD。

3. **存储在 .vhd 文件中并附加到虚拟机的受支持的 Windows 操作系统** - 存在可创建 .vhd 文件的多种工具。例如，你可以使用 Hyper-V 创建虚拟机并安装操作系统。有关说明，请参阅[安装 Hyper-V 角色和配置虚拟机](http://technet.microsoft.com/zh-cn/library/hh846766.aspx)。有关操作系统的详细信息，请参阅[对 Azure 虚拟机支持的 Microsoft 服务器软件](http://go.microsoft.com/fwlink/p/?LinkId=393550)。

> [AZURE.IMPORTANT]Azure 不支持 VHDX 格式。可使用 Hyper-V 管理器或 [Convert-VHD cmdlet](http://technet.microsoft.com/zh-cn/library/hh848454.aspx) 将磁盘转换为 VHD 格式。有关详细信息，请参阅此 [blogpost](http://blogs.msdn.com/b/virtual_pc_guy/archive/2012/10/03/using-powershell-to-convert-a-vhd-to-a-vhdx.aspx)。

## 步骤 1：准备 VHD ##

在将 VHD 上载到 Azure 之前，需要使用 Sysprep 工具对 VHD 进行一般化。这样就可以将 VHD 作为映像使用。有关 Sysprep 的详细信息，请参阅[如何使用 Sysprep：简介](http://technet.microsoft.com/zh-cn/library/bb457073.aspx)。

在操作系统已安装到的虚拟机中，完成以下过程：

1. 登录到操作系统。

2. 以管理员身份打开“命令提示符”窗口。将目录更改为 **%windir%\\system32\\sysprep**，然后运行 `sysprep.exe`。

	![打开“命令提示符”窗口](./media/virtual-machines-windows-classic-createupload-vhd/sysprep_commandprompt.png)

3.	此时会显示**“系统准备工具”**对话框。

	![启动 Sysprep](./media/virtual-machines-windows-classic-createupload-vhd/sysprepgeneral.png)

4.  在**“系统准备工具”**中，选择**“进入系统全新体验(OOBE)”**并确保选中**“一般化”**。

5.  在**“关机选项”**中选择**“关机”**。

6.  单击**“确定”**。

## 步骤 2：创建 Azure 存储帐户或从你的 Azure 存储帐户获取信息

你需要一个 Azure 存储帐户，以便有一个可向其上载 .vhd 文件的位置。此步骤演示了如何创建一个帐户，或如何从现有帐户获取需要的信息。

### 方法 1：创建存储帐户

1. 登录到[Azure经典管理门户](http://manage.windowsazure.cn)。

2. 在命令栏上，单击“新建”。

3. 依次单击**“数据服务”**>**“存储”**>**“快速创建”**。

	![快速创建存储帐户](./media/virtual-machines-windows-classic-createupload-vhd/Storage-quick-create.png)

4. 如下所示填写字段：

 - 在 **URL** 下，键入要在存储帐户的 URL 中使用的子域名称。该条目可能包含 3 到 24 个小写字母和数字。此名称将成为用于访问订阅的 Blob、队列或表资源的 URL 的主机名。
 - 为存储帐户选择**位置或地缘组**。地缘组让你能够将你的云服务和存储放在同一数据中心。
 - 决定是否对存储帐户使用**地域复制**。默认情况下启用地域复制。此选项会将你的数据免费复制到辅助位置，以便在主位置发生严重故障时将你的存储故障转移到该位置。将自动分配辅助位置，并且无法对其进行更改。如果你因法律要求或组织策略需要更好地控制基于云的存储的位置，可以关闭地域复制。但是，如果稍后打开异地复制，则将现有数据复制到辅助位置时将向你收取一次性数据传输费用。不具有地域复制的存储服务将以优惠价提供。有关更多详细信息，请参阅[创建、管理或删除存储帐户](/documentation/articles/storage-create-storage-account/#replication-options)。

      ![输入存储帐户详细信息](./media/virtual-machines-windows-classic-createupload-vhd/Storage-create-account.png)

5. 单击“创建存储帐户”。该帐户现在会出现在**“存储”**下。

	![已成功创建存储帐户](./media/virtual-machines-windows-classic-createupload-vhd/Storagenewaccount.png)

6. 接下来，为上载的 VHD 创建容器。单击存储帐户名称，然后单击“容器”。

	![存储帐户详细信息](./media/virtual-machines-windows-classic-createupload-vhd/storageaccount_detail.png)

7. 单击**“创建容器”**。

	![存储帐户详细信息](./media/virtual-machines-windows-classic-createupload-vhd/storageaccount_container.png)

8. 为容器键入名称，并选择“访问”策略。

	![容器名称](./media/virtual-machines-windows-classic-createupload-vhd/storageaccount_containervalues.png)

	> [AZURE.NOTE]默认情况下，该容器是专用容器，只能由帐户所有者访问。若要允许对容器中的 blob 进行公共读取访问，但不允许对容器属性和元数据进行公共读取访问，请使用“公共 Blob”选项。若要允许对容器和 blob 进行完全公共读取访问，请使用“公共容器”选项。

### 方法 2：获取存储帐户信息

1.	登录到[Azure经典管理门户](http://manage.windowsazure.cn)。

2.	在导航窗格中，单击“存储”。

3.	单击存储帐户的名称，然后单击“仪表板”。

4.	在仪表板中，在“服务”下，将鼠标悬停在 Blob URL 上，单击剪贴板图标以复制此 URL，然后粘贴并保存此 URL。在构建命令上载 VHD 时使用。

## 步骤 3：从 Azure PowerShell 连接到你的订阅

你首先需要在计算机和 Azure 中的订阅之间建立一个安全连接，然后才能上载 .vhd 文件。你可以使用 Azure Active Directory 方法或证书方法来执行此操作。

> [AZURE.TIP]若要开始使用 Azure PowerShell，请参阅[如何安装和配置 Azure PowerShell](/documentation/articles/powershell-install-configure/)。有关常规信息，请参阅 [Azure Cmdlet 入门](https://msdn.microsoft.com/zh-cn/library/azure/jj554332.aspx)。

### 方法 1：使用 Azure AD

1. 打开 Azure PowerShell 控制台。

2. 键入：`Add-AzureAccount -Environment AzureChinaCloud`

3.	在登录窗口中，键入你的工作或学校帐户的用户名和密码。

4. Azure 将对凭据信息进行身份验证和保存，然后关闭该窗口。

### 方法 2：使用证书

1. 打开 Azure PowerShell 控制台。

2.	键入：
		`Get-AzurePublishSettingsFile -Environment AzureChinaCloud`。

3. 此时将打开一个浏览器窗口，并提示你下载 .publishsettings 文件。它包含 Azure 订阅的信息和证书。

	![浏览器下载页面](./media/virtual-machines-windows-classic-createupload-vhd/Browser_download_GetPublishSettingsFile.png)

3. 保存 .publishsettings 文件。

4. 键入：
		`Import-AzurePublishSettingsFile -Environment AzureChinaCloud <PathToFile>`

	其中 `<PathToFile>` 是 .publishsettings 文件的完整路径。

## 步骤 4：上载 .vhd 文件

在上载 .vhd 文件时，你可以将 .vhd 文件放置在 Blob 存储中的任何位置。

1. 从你在上一步中使用的 Azure PowerShell 窗口中，键入一个与此类似的命令：

	`Add-AzureVhd -Destination "<BlobStorageURL>/<YourImagesFolder>/<VHDName>.vhd" -LocalFilePath <PathToVHDFile>`

	其中：
	- **BlobStorageURL** 是存储帐户的 URL
	- **YourImagesFolder** 是要在其中存储映像的 blob 存储中的容器
	- **VHDName** 是你希望在 Azure 经典管理门户中显示以识别虚拟硬盘的名称
	- **PathToVHDFile** 是 .vhd 文件的完整路径和名称

	![PowerShell Add-AzureVHD](./media/virtual-machines-windows-classic-createupload-vhd/powershell_upload_vhd.png)

有关 Add-AzureVhd cmdlet 的详细信息，请参阅 [Add-AzureVhd](http://msdn.microsoft.com/zh-cn/library/dn495173.aspx)。

## 步骤 5：将映像添加到自定义映像列表

> [AZURE.TIP]若要使用 Azure PowerShell（而不是经典管理门户）来添加映像，请使用 **Add-AzureVMImage** cmdlet。例如：

>	`Add-AzureVMImage -ImageName <ImageName> -MediaLocation <VHDLocation> -OS <OSType>`

1. 在 Azure经典管理门户中的“所有项目”下单击“虚拟机”。

2. 在“虚拟机”下，单击**“映像”**。

3. 单击**“创建映像”**。

	![PowerShell Add-AzureVHD](./media/virtual-machines-windows-classic-createupload-vhd/Create_Image.png)

4. 在“从 VHD 创建映像”窗口：

	- 指定“名称”。

	- 指定“说明”。

	- 在 VHD URL 下，单击文件夹按钮，以打开“浏览云存储”窗口。找到 .vhd 文件，然后单击“打开”。

    ![选择 VHD](./media/virtual-machines-windows-classic-createupload-vhd/Select_VHD.png)

5.	在“从 VHD 创建映像”窗口中，在“操作系统系列”下，选择你的操作系统。选中“我已在与此 VHD 关联的虚拟机上运行 Sysprep”，确认你已对操作系统进行一般化，然后单击“确定”。

    ![添加映像](./media/virtual-machines-windows-classic-createupload-vhd/Create_Image_From_VHD.png)

6. 完成前面的步骤后，当你选择**“映像”**选项卡时，将列出新映像。

	![自定义映像](./media/virtual-machines-windows-classic-createupload-vhd/vm_custom_image.png)

	现在，当你创建虚拟机时，此新映像将在**“我的映像”**下提供。

	![从自定义映像创建虚拟机](./media/virtual-machines-windows-classic-createupload-vhd/create_vm_custom_image.png)

	> [AZURE.TIP]如果在你尝试创建 VM 时，收到带有此错误消息的错误：“VHD https://XXXXX.. 具有不支持的虚拟大小(YYYY 字节)。大小必须是整数(以 MB 为单位)”，这意味着你的 VHD 不是整数个 MB，需要为固定大小的 VHD。请尝试使用 **Add-AzureVMImage** PowerShell cmdlet（而不是Azure经典管理门户）来添加映像（请参阅上面的步骤 5）。Azure cmdlet 可确保 VHD 满足 Azure 要求。

[Step 1: Prepare the image to be uploaded]: #prepimage
[Step 2: Create a storage account in Azure]: #createstorage
[Step 3: Prepare the connection to Azure]: #prepAzure
[Step 4: Upload the .vhd file]: #upload

<!---HONumber=Mooncake_1207_2015-->