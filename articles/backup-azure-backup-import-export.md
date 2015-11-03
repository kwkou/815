<properties
   pageTitle="Azure 备份 - 使用 Azure 导入/导出服务进行脱机备份或初始种子设定 | Windows Azure"
   description="了解 Azure 备份如何让你使用 Azure 导入/导出服务离线发送数据。本文介绍如何使用 Azure 导入导出服务来脱机设定初始备份数据的种子。"
   services="backup"
   documentationCenter=""
   authors="aashishr"
   manager="shreeshd"
   editor=""/>
<tags
   ms.service="backup"
   ms.date="08/28/2015"
   wacn.date="11/02/2015"/>

# Azure 备份中的脱机备份工作流

Azure 备份与 Azure 导入/导出服务深度集成，使你可以快速传输初始备份数据。如果你要通过高延迟、低带宽网络传输 TB 计的初始备份数据，可以使用 Azure 导入/导出服务将一个或多个硬盘驱动器中的初始备份副本传送到 Azure 数据中心。本文概述了完成此工作流所需的步骤。

## 概述

借助 Azure 备份和 Azure 导入/导出，可以简单直接地通过磁盘将数据脱机上载到 Azure。不是通过网络传输初始的完整副本，而是将备份数据写入“暂存位置”。暂存位置可以是直接附加存储，也可以是网络共享。使用“Azure 导入/导出工具”完成初始复制后，此数据将写入 SATA 驱动器，最终传送到 Azure 数据中心。根据初始备份的大小，可能需要一个或多个 SATA 驱动器来完成此操作。用于这些方案的 Azure 导入/导出工具。将备份写入磁盘后，可将其传送到最接近的数据中心，以便上载到 Azure。然后，Azure 备份会将备份数据复制到备份保管库并计划增量备份。

## 先决条件

1. 您必须熟悉[此处](/documentation/articles/storage-import-export-service)列出的 Azure 导入导出工作流。

2. 在启动工作流之前，请确保已创建 Azure 备份保管库，已下载保管库凭据，已在 Windows Server/Windows 客户端或 System Center Data Protection Manager (SCDPM) 服务器以及已注册到 Azure 备份保管库的计算机上安装 Azure 备份代理。

3. 下载中的 Azure 发布文件设置[此处](https://manage.windowsazure.cn/publishsettings)您计划用于备份我们的数据的计算机上。

4. 准备*暂存位置*，这可能是网络共享或计算机上的其他驱动器。确保临时位置具有足够的磁盘空间来保存你的初始副本。例如，如果您尝试备份 500GB 文件服务器，请确保暂存区域至少为 500GB（尽管将使用较少的量）。暂存区域是“短时存储”并在此工作流期间暂时使用。

5. 外部 SATA 驱动器编写器和外部 3.5 英寸 SATA 驱动器。只支持对 3.5 英寸 SATA II/III 硬盘驱动器使用导入/导出服务。不支持大于 4TB 的硬盘驱动器。你可以使用 SATA II/III USB 适配器在外部将 SATA II/III 磁盘连接到大多数计算机。检查的最新的一套驱动器支持的服务的 Azure 导入/导出文档。

6. SATA 驱动器编写器连接到计算机上启用 BitLocker。

7. 从[此处](http://go.microsoft.com/fwlink/?LinkID=301900&clcid=0x409)将 Azure 导入/导出工具下载到 SATA 驱动器编写器连接到的计算机。


## 工作流
本部分中提供的信息用于完成“脱机备份工作流”，以便可以将您的数据传输到 Azure 数据中心并上载到 Azure 存储。如果您有关于导入服务或流程的任何方面的问题，请参阅[上面](/documentation/articles/storage-import-export-service)引用的导入服务概述。

### 启动脱机备份

1. 作为计划备份的一部分，你会遇到下面的屏幕（在 Windows Server 中，Windows 客户端或 SCDPM）。
  ![ImportScreen][1]

  SCDPM 中的对应屏幕如下所示。<br/>
  ![DPM 导入屏幕][8]

其中：

+ **暂存位置** - 表示写入初始备份副本的临时存储位置。这可能是网络共享上或在本地计算机上。

+ **Azure 导入作业名称** - 作为完成此工作流的一部分，你将需要在 Azure 门户中创建*导入作业*（在该文档的稍后部分中介绍）。提供了你计划更高版本在 Azure 门户中使用的输入。

+ **Azure 发布设置** - 一个 XML 文件，其中包含有关您订阅的个人资料的信息。它还包含到你的订阅关联的安全凭据。可以从[此处](https://manage.windowsazure.cn/publishsettings)下载该文件。提供发布设置文件的本地路径。

+ **Azure 订阅 ID** - 提供计划用来启动 Azure 导入作业的 Azure 订阅 ID。如果你有多个 Azure 订阅，使用与导入作业关联的 ID。

+ **Azure 存储帐户** - 输入将与此导入作业相关联的 Azure 存储帐户的名称。

+ **Azure 存储容器** - 输入将导入此作业的数据的目标存储 Blob 的名称。

2. 完成工作流，然后在 Azure 备份 mmc 中选择“立即备份”，以启动脱机备份副本。初始备份写入到临时区域作为此步骤的一部分。

  ![立即备份][2]

通过单击“保护组”并选择“创建恢复点”选项启用 SCDPM 中的相应工作流。随后选择“在线保护”选项。<br/>
  ![立即进行 DPM 备份][9]

操作完成后，将在临时位置创建 *.AIBBlob* 和 *.BaseBlob* 文件。<br/>
  ![输出][3]

### 准备 SATA 驱动器

1. 将 [Windows Azure 导入/导出工具](http://go.microsoft.com/fwlink/?linkid=301900&clcid=0x409)下载到*复制计算机*。确保可从你打算运行下一个命令集的计算机访问的临时位置（在上一步）。如果需要，请在复制计算机可以与源计算机相同。

2. 解压缩 *WAImportExport.zip* 文件。运行 *WAImportExport* 工具，它将格式化 SATA 驱动器，将备份数据写入到 SATA 驱动器并对其进行加密。在运行以下命令之前请确保在计算机上启用了 BitLocker。<br/>

*.\\WAImportExport.exe PrepImport /j:<*JournalFile*>.jrn /id: <*SessionId*> /sk:<*StorageAccountKey*> /BlobType:**PageBlob*\* /t:<*TargetDriveLetter*> /format /encrypt /srcdir:<*staging location*> /dstdir: <*DestinationBlobVirtualDirectory*>/*


| 参数 | 说明|
|-------------|-------------|
| /j:<*JournalFile*>| 日志文件的路径。每个驱动器必须正好有一个日志文件。请注意，日志文件不得驻留在目标驱动器上。日志文件的扩展名是 .jrn，并作为运行此命令的一部分创建。|
|/id:<*SessionId*> | 会话 ID 标识“复制会话”。它用于确保准确恢复中断的复制会话。复制会话中复制的文件将存储在以目标驱动器上的会话 ID 命名的目录中。|
| /sk:<*StorageAccountKey*> | 将数据导入到的存储帐户的帐户密钥。 |
| /BlobType | 指定 **PageBlob**，仅当指定 PageBlob 选项时，此工作流才会成功。这不是默认选项，并应在此命令中所述。 |
|/t:<*TargetDriveLetter*> | 当前复制会话的目标硬盘驱动器的驱动器号（不带尾随冒号）。|
|/format | 在需要格式化驱动器时指定此参数；否则，请将其忽略。在对驱动器进行格式化之前，该工具将提示你通过控制台进行确认。若不希望显示该确认，请指定 /silentmode 参数。|
|/encrypt | 在尚未使用 BitLocker 对驱动器进行加密但需要使用此工具进行加密时，指定此参数。如果已使用 BitLocker 对驱动器进行加密，则忽略此参数并指定 /bk 参数，同时还提供现有 BitLocker 密钥。如果指定 /format 参数，则还必须指定 /encrypt 参数。 |
|/srcdir:<*SourceDirectory*> | 包含要复制到目标驱动器中的文件的源目录。目录路径必须是绝对路径（而非相对路径）。|
|/dstdir:<*DestinationBlobVirtualDirectory*> | Windows Azure 存储帐户中的目标虚拟目录的路径。在指定目标虚拟目录或 Blob 时，请确保使用有效的容器名称。请记住，容器名称必须是小写的。|

  >[AZURE.NOTE]捕获整个工作流的信息的 WAImportExport 文件夹中创建日志文件。在 Azure 门户中创建导入作业时，将需要此文件。

  ![PowerShell 输出][4]
>[AZURE.NOTE]目前“导入/导出”功能在Azure门户中不可用。
### 在 Azure 门户中创建导入作业
1. 导航到[管理门户](https://manage.windowsazure.cn/)中的存储帐户，然后依次单击任务窗格上的“导入/导出”和“创建导入作业”。<br/>
![门户][5]

2. 在向导的步骤 1 中，指示你已准备好了驱动器并且具有可用的驱动器日志文件。在向导的步骤 2 中，提供负责该导入作业的人员的联系信息。

3. 在步骤 3 中，上载你在上一部分获取的驱动器日志文件。

4. 在步骤 4 中，输入导入作业的描述性名称。请注意，你输入的名称只能包含小写字母、数字、连字符和下划线，必须以字母开头并且不得包含空格。在作业进行中以及在作业完成后，你将使用所选名称来跟踪作业。

5. 接下来，从列表中选择你的数据中心区域。数据中心区域会指示必须将你的包裹运送到的数据中心和地址。<br/>
![DC][6]

6. 在步骤 5 中，从列表中选择回程承运人，并输入你的承运人帐号。当你的导入作业完成后，Microsoft 将使用此帐户寄回你的驱动器。

7. 运送磁盘，然后输入要跟踪的发货状态的跟踪号。磁盘抵达数据中心后，将其复制到存储帐户并更新状态。<br/>

![完成状态][7]

### 完成工作流
在你的存储帐户中可用的初始备份数据后，Azure 备份代理将数据的内容从该帐户复制到多租用的备份存储帐户。在下一个计划备份时间，Azure 备份代理针对初始的备份副本来执行增量备份。

## 后续步骤
+ 如有 Azure 导入/导出工作流方面的任何问题，请参阅此[文章](/documentation/articles/storage-import-export-service)。

+ 如有工作流方面的任何问题，请参阅 Azure 备份[常见问题](/documentation/articles/backup-azure-backup-faq)的“脱机备份”部分

<!--Image references-->
[1]: ./media/backup-azure-backup-import-export/importscreen.png
[2]: ./media/backup-azure-backup-import-export/backupnow.png
[3]: ./media/backup-azure-backup-import-export/opbackupnow.png
[4]: ./media/backup-azure-backup-import-export/psoutput.png
[5]: ./media/backup-azure-backup-import-export/azureportal.png
[6]: ./media/backup-azure-backup-import-export/dc.png
[7]: ./media/backup-azure-backup-import-export/complete.png
[8]: ./media/backup-azure-backup-import-export/dpmoffline.png
[9]: ./media/backup-azure-backup-import-export/dpmbackupnow.png

<!---HONumber=67-->