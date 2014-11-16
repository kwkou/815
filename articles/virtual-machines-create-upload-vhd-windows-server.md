<properties linkid="manage-windows-common-task-upload-vhd" urlDisplayName="Upload a VHD" pageTitle="Create and upload a Windows Server VHD to Azure" metaKeywords="Azure VHD, uploading VHD" description="Learn how to create and upload a virtual hard disk (VHD) in Azure that has the Windows Server operating system." metaCanonical="" services="virtual-machines" documentationCenter="" title="Creating and Uploading a Virtual Hard Disk that Contains the Windows Server Operating System" authors="kathydav" solutions="" manager="dongill" editor="tysonn" />

# 创建并上载包含 Windows Server 操作系统的虚拟硬盘

![启动 Sysprep][启动 Sysprep]

Microsoft Azure 中的虚拟机运行你在创建虚拟机时选择的操作系统。Microsoft Azure 采用 VHD 格式（.vhd 文件）将虚拟机的操作系统存储在虚拟硬盘中。已为复制准备的操作系统的 VHD 称作映像。本文说明如何利用已安装且经过一般化的操作系统上载 .vhd 文件来创建你自己的映像。有关 Microsoft Azure 中的磁盘和映像的详细信息，请参阅[管理磁盘和映像（可能为英文页面）][管理磁盘和映像（可能为英文页面）]。

**注意**：创建虚拟机时，你可以自定义操作系统设置以快速运行你的应用程序。你设置的配置存储在该虚拟机的磁盘上。有关说明，请参阅[如何创建自定义虚拟机][如何创建自定义虚拟机]。

## 先决条件

本文假定你拥有以下项目：

**Microsoft Azure PowerShell** - 你已经安装了 Microsoft Azure PowerShell 模块。若要下载该模块，请参阅 [Microsoft Azure 下载（可能为英文页面）][Microsoft Azure 下载（可能为英文页面）]。在[此处][此处]可找到使用 Azure 订阅来安装和配置 PowerShell 的教程。

-   [Add-AzureVHD][Add-AzureVHD] cmdlet，它是 Microsoft Azure PowerShell 模块的组成部分，上载 VHD 时需要。

**存储在 .vhd 文件中的受支持的 Windows 操作系统** - 你已将受支持的 Windows Server 操作系统安装到虚拟硬盘。可使用多个工具创建 .vhd 文件。可使用虚拟化解决方案（例如 Hyper-V）创建 .vhd 文件并安装操作系统。有关说明，请参阅[安装 Hyper-V 角色和配置虚拟机][安装 Hyper-V 角色和配置虚拟机]。

**重要说明**：Microsoft Azure 不支持 VHDX 格式。可使用 Hyper-V 管理器或 [Convert-VHD cmdlet][Convert-VHD cmdlet] 将磁盘转换为 VHD 格式。在[此处][1]可找到教程。

**Window Server 操作系统介质。**此任务要求有包含 Windows Server 操作系统的 .iso 文件。支持以下版本的 Windows Server：

<table border="1" width="600">

<tr bgcolor="#E9E7E7">

<th>
操作系统

</th>

<th>
SKU

</th>

<th>
Service Pack

</th>

<th>
体系结构

</th>

</tr>

<tr>

<td>
Windows Server 2012 R2

</td>

<td>
所有版本

</td>

<td>
不适用

</td>

<td>
x64

</td>

</tr>

<tr>

<td>
Windows Server 2012

</td>

<td>
所有版本

</td>

<td>
不适用

</td>

<td>
x64

</td>

</tr>

<tr>

<td>
Windows Server 2008 R2

</td>

<td>
所有版本

</td>

<td>
SP1

</td>

<td>
x64

</td>

</tr>

</table>

</p>
</p>
此任务包括下列步骤：

-   [步骤 1：准备要上载的映像][步骤 1：准备要上载的映像]
-   [步骤 2：在 Azure 中创建存储帐户][步骤 2：在 Azure 中创建存储帐户]
-   [步骤 3：准备连接到 Azure][步骤 3：准备连接到 Azure]
-   [步骤 4：上载 .vhd 文件][步骤 4：上载 .vhd 文件]

## <span id="prepimage"></span> </a>步骤 1：准备要上载的映像

在将映像上载到 Azure 之前，必须先使用 Sysprep 命令通用化。有关使用 Sysprep 的更多信息，请参阅[如何使用 Sysprep：简介][如何使用 Sysprep：简介]。

在你刚创建的虚拟机中，完成以下过程：

1.  登录到操作系统。

2.  以管理员身份打开“命令提示符”窗口。将目录更改为 **%windir%\\system32\\sysprep**，然后运行 `sysprep.exe`。

    ![打开“命令提示符”窗口][打开“命令提示符”窗口]

3.  会显示**“系统准备工具”**对话框。

    ![启动 Sysprep][2]

4.  在“系统准备工具”中，选择“进入系统全新体验(OOBE)”并确保选中“一般化”。
5.  在**“关机选项”**中选择**“关机”**。
6.  单击“确定”。

## <span id="createstorage"></span> </a>步骤 2：在 Azure 中创建存储帐户

存储帐户表示用于访问存储服务的最高级别的命名空间，并且与你的 Azure 订阅相关联。你需要在 Azure 中具有存储帐户才能将 .vhd 文件上载到可用于创建虚拟机的 Azure。可使用 Azure 管理门户创建存储帐户。

1.  登录到 Azure 管理门户。

2.  在命令栏上，单击“新建”。

3.  单击“存储帐户”，然后单击“快速创建”。

    ![快速创建存储帐户][快速创建存储帐户]

4.  如下所示填写字段：

    ![输入存储帐户详细信息][输入存储帐户详细信息]

    在 **URL** 下，键入要在存储帐户的 URL 中使用的子域名称。输入的名称可包含 3-24 个小写字母和数字。此名称将成为用于对订阅的 Blob、队列或表资源进行寻址的 URL 中的主机名。

    选择存储帐户的位置或地缘组。通过指定地缘组，可将同一数据中心内的云服务与你的存储放置在一起。

    决定是否使用存储帐户的地域复制。默认情况下启用地域复制。此选项会将你的数据免费复制到辅助位置，以便在发生无法在主要位置进行处理的严重故障时将你的存储故障转移到辅助位置。将自动分配辅助位置，并且无法对其进行更改。如果法律要求或组织政策要求更加严格地控制基于云的存储的位置，则你可以关闭地域复制。但是，请注意，如果稍后你打开地域复制，则将现有数据复制到辅助位置时将向你收取一次性数据传输费用。不具有地域复制的存储服务将以优惠价提供。有关管理存储帐户的地域复制的详细信息，请参阅：[如何管理存储帐户][如何管理存储帐户]。

5.  单击**“创建存储帐户”**。

    该帐户现在会出现在“存储帐户”下。

    ![已成功创建存储帐户][已成功创建存储帐户]

6.  现在我们要来为你上载的 VHD 创建容器。单击“存储帐户名称”，然后单击“容器”。

    ![存储帐户详细信息][存储帐户详细信息]

7.  接下来，单击“创建容器”。

    ![存储帐户详细信息][3]

8.  现在为你的容器选择“名称”，并选择该容器的“访问策略”。

    ![容器名称][容器名称]

    -   **访问策略** - 默认情况下，容器是私有的，并且只能由帐户所有者访问。若要允许对容器中的 Blob 进行公共读取访问，但不允许对容器属性和元数据进行公共读取访问，请使用“公共 Blob”选项。若要允许对容器和 Blob 进行完全公共读取访问，请使用“公共容器”选项。

## <span id="PrepAzure"></span> </a>步骤 3：准备连接到 Microsoft Azure

你首先需要在计算机和 Microsoft Azure 中的订阅之间建立一个安全连接，然后才能上载 .vhd 文件。你可以使用 Microsoft Azure Active Directory 或证书方法来进行此操作。

### 使用 Microsoft Azure AD 方法

1.  打开 Microsoft Azure PowerShell 控制台，请按照[如何：安装 Azure PowerShell][如何：安装 Azure PowerShell] 中的说明操作。

2.  输入以下命令：`Add-AzureAccount`. 这时会弹出一个窗口，你可以在其中使用 Microsoft 帐户登录到 Azure。

    ![PowerShell 窗口][PowerShell 窗口]

3.  Microsoft Azure 将对凭据信息进行身份验证和保存，然后关闭该窗口。

### 使用证书方法

1.  打开 Microsoft Azure PowerShell 窗口。键入：
    `Get-AzurePublishSettingsFile`.

2.  此命令将打开浏览器窗口，并自动下载包含信息的 .publishsettings 文件和 Microsoft Azure 订阅的证书。

    ![浏览器下载页面][浏览器下载页面]

3.  保存 .publishsettings 文件。

4.  键入：
    `Import-AzurePublishSettingsFile <PathToFile>`

    其中`<PathToFile>` 是 .publishsettings 文件的完整路径。

    有关详细信息，请参阅 [Microsoft Azure Cmdlet 入门][Microsoft Azure Cmdlet 入门]

    有关安装和配置 PowerShell 的详细信息，请参阅[如何安装和配置 Microsoft Azure PowerShell][此处]

## <span id="upload"></span> </a>步骤 4：上载 .vhd 文件

在上载 .vhd 文件时，你可以将 .vhd 文件放置在 Blob 存储中的任何位置。在以下命令示例中，**BlobStorageURL** 是你在步骤 2 中创建的存储帐户的 URL，**YourImagesFolder** 是要在其中存储映像的 Blob 存储中的容器。**VHDName** 是显示在管理门户中用于标识虚拟硬盘的标签。**PathToVHDFile** 是 .vhd 文件的完整路径和名称。

1.  从你在上一步中使用的 Microsoft Azure PowerShell 窗口中，键入：

    `Add-AzureVhd -Destination "<BlobStorageURL>/<YourImagesFolder>/<VHDName>.vhd" -LocalFilePath <PathToVHDFile>`

    ![PowerShell Add-AzureVHD][PowerShell Add-AzureVHD]

    有关 Add-AzureVhd cmdlet 的详细信息，请参阅 [Add-AzureVhd][Add-AzureVhd]。

## 将映像添加到自定义映像列表

上载 .vhd 后，将其作为映像添加到与订阅关联的自定义映像列表。

1.  从管理门户中的“所有项目”下单击“虚拟机”。

2.  在“虚拟机”下，单击“映像”。

3.  然后单击“创建映像”。

    ![PowerShell Add-AzureVHD][4]

4.  在“从 VHD 创建映像”中。
    
    ![添加映像][添加映像]

    -   指定“名称”
    -   指定“说明”
    -   若要指定“VHD 的 URL”，请单击文件夹图标，以启动下面的对话框
        
        ![选择 VHD][选择 VHD]
    -   在此选择对话框中，选中你的 VHD 所在的存储帐户并单击“打开”。这样你会回到前面的“从 VHD 创建映像”框。
    -   返回到“从 VHD 创建映像”框后，请选择“操作系统系列”。
    -   选中“我已在与此 VHD 关联的虚拟机上运行 Sysprep”，确认你的第 1 步中已通用化了操作系统，然后单击“确定”。

5.  **可选：**你也可以使用 Azure PowerShell 的 Add-AzureVMImage cmdlet 将 VHD 添加为映像。

    从 Microsoft Azure PowerShell 窗口中，键入：

    `Add-AzureVMImage -ImageName <Your Image's Name> -MediaLocation <location of the VHD> -OS <Type of the OS on the VHD>`

    ![PowerShell Add-AzureVMImage][PowerShell Add-AzureVMImage]

6.  完成上述步骤后，应该会看到“映像”选项卡中已经有了你的映像，如下面截屏所示。

    ![自定义映像][自定义映像]

    现在在尝试创建新虚拟机时，你将能够通过在库中选择“我的映像”选项卡来使用此映像，如下面截屏所示

    ![从自定义映像创建虚拟机][从自定义映像创建虚拟机]

## 后续步骤

在列表中有了映像之后，你可以使用该映像来创建虚拟机。有关说明，请参阅[创建运行 Windows Server 的虚拟机][如何创建自定义虚拟机]。

创建虚拟机后，尝试创建 SQL Server 虚拟机。有关说明，请参阅[在 Microsoft Azure 上设置 SQL Server 虚拟机][在 Microsoft Azure 上设置 SQL Server 虚拟机]。

  [启动 Sysprep]: ./media/virtual-machines-create-upload-vhd-windows-server/ImageWithDisks.png
  [管理磁盘和映像（可能为英文页面）]: http://msdn.microsoft.com/zh-cn/library/azure/jj672979.aspx
  [如何创建自定义虚拟机]: http://windowsazure.cn/zh-cn/documentation/articles/virtual-machines-windows-tutorial/
  [Microsoft Azure 下载（可能为英文页面）]: http://www.windowsazure.cn/zh-cn/downloads/#cmd-line-tools
  [此处]: http://windowsazure.cn/zh-cn/documentation/articles/install-configure-powershell/
  [Add-AzureVHD]: http://msdn.microsoft.com/zh-cn/library/azure/dn205185.aspx
  [安装 Hyper-V 角色和配置虚拟机]: http://technet.microsoft.com/zh-cn/library/hh846766.aspx
  [Convert-VHD cmdlet]: http://technet.microsoft.com/zh-cn/library/hh848454.aspx
  [1]: http://blogs.msdn.com/b/virtual_pc_guy/archive/2012/10/03/using-powershell-to-convert-a-vhd-to-a-vhdx.aspx
  [步骤 1：准备要上载的映像]: #prepimage
  [步骤 2：在 Azure 中创建存储帐户]: #createstorage
  [步骤 3：准备连接到 Azure]: #prepAzure
  [步骤 4：上载 .vhd 文件]: #upload
  [如何使用 Sysprep：简介]: http://technet.microsoft.com/zh-cn/library/bb457073.aspx
  [打开“命令提示符”窗口]: ./media/virtual-machines-create-upload-vhd-windows-server/sysprep_commandprompt.png
  [2]: ./media/virtual-machines-create-upload-vhd-windows-server/sysprepgeneral.png
  [快速创建存储帐户]: ./media/virtual-machines-create-upload-vhd-windows-server/Storage-quick-create.png
  [输入存储帐户详细信息]: ./media/virtual-machines-create-upload-vhd-windows-server/Storage-create-account.png
  [如何管理存储帐户]: http://windowsazure.cn/zh-cn/documentation/articles/storage-manage-storage-account/
  [已成功创建存储帐户]: ./media/virtual-machines-create-upload-vhd-windows-server/Storagenewaccount.png
  [存储帐户详细信息]: ./media/virtual-machines-create-upload-vhd-windows-server/storageaccount_detail.png
  [3]: ./media/virtual-machines-create-upload-vhd-windows-server/storageaccount_container.png
  [容器名称]: ./media/virtual-machines-create-upload-vhd-windows-server/storageaccount_containervalues.png
  [如何：安装 Azure PowerShell]: #Install
  [PowerShell 窗口]: ./media/virtual-machines-create-upload-vhd-windows-server/add_azureaccount.png
  [浏览器下载页面]: ./media/virtual-machines-create-upload-vhd-windows-server/Browser_download_GetPublishSettingsFile.png
  [Microsoft Azure Cmdlet 入门]: http://msdn.microsoft.com/zh-cn/library/azure/jj554332.aspx
  [PowerShell Add-AzureVHD]: ./media/virtual-machines-create-upload-vhd-windows-server/powershell_upload_vhd.png
  [Add-AzureVhd]: http://msdn.microsoft.com/zh-cn/library/dn495173.aspx
  [4]: ./media/virtual-machines-create-upload-vhd-windows-server/Create_Image.png
  [添加映像]: ./media/virtual-machines-create-upload-vhd-windows-server/Create_Image_From_VHD.png
  [选择 VHD]: ./media/virtual-machines-create-upload-vhd-windows-server/Select_VHD.png
  [PowerShell Add-AzureVMImage]: ./media/virtual-machines-create-upload-vhd-windows-server/add_azureimage_powershell.png
  [自定义映像]: ./media/virtual-machines-create-upload-vhd-windows-server/vm_custom_image.png
  [从自定义映像创建虚拟机]: ./media/virtual-machines-create-upload-vhd-windows-server/create_vm_custom_image.png
  [在 Microsoft Azure 上设置 SQL Server 虚拟机]: http://windowsazure.cn/zh-cn/documentation/articles/virtual-machines-provision-sql-server/
