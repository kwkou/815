<properties
	pageTitle="使用存储资源管理器（预览版）管理 Azure Blob 存储资源 | Azure"
	description="使用存储资源管理器（预览版）管理 Azure Blob 容器和 Blob"
	services="storage"
	documentationCenter="na"
	authors="TomArcher"
	manager="douge"
	editor="" />  

<tags
    ms.assetid="2f09e545-ec94-4d89-b96c-14783cc9d7a9"
    ms.service="storage"
    ms.devlang="multiple"
    ms.topic="article"
    ms.tgt_pltfrm="na"
    ms.workload="na"
    ms.date="11/18/2016"
    wacn.date="02/04/2017"
    ms.author="tarcher" />

# 使用存储资源管理器（预览版）管理 Azure Blob 存储资源

## 概述

[Azure Blob 存储](/documentation/articles/storage-dotnet-how-to-use-blobs/)是用于存储大量非结构化数据（例如文本或二进制数据）的服务，这些数据可通过 HTTP 或 HTTPS 从世界各地进行访问。
你可以使用 Blob 存储向外公开数据，或者私下存储应用程序数据。在本文中，你将了解如何使用适合 Blob 容器和 Blob 的存储资源管理器（预览版）。

## 先决条件

若要完成本文中的步骤，你需要满足以下先决条件：

- [下载并安装存储资源管理器（预览版）](http://www.storageexplorer.com)
- 连接到 Azure 存储帐户或服务

## 创建 Blob 容器

所有 Blob 都必须驻留在 Blob 容器中。简单说来，该容器就是对 Blob 进行逻辑分组。一个帐户可以包含无限数量的容器，一个容器可以存储无限数量的 Blob。

以下步骤演示了如何在存储资源管理器（预览版）中创建 Blob 容器。

1.	打开存储资源管理器（预览版）。
1.	在左窗格中，展开需要在其中创建 Blob 容器的存储帐户。
1.	右键单击“Blob 容器”，然后从上下文菜单中选择“创建 Blob 容器”。

	![“创建 Blob 容器”上下文菜单][0]

1.	此时会在“Blob 容器”文件夹下显示一个文本框。输入你的 Blob 容器的名称。如需 Blob 容器命名规则和限制的列表，请参[容器命名规则](/documentation/articles/storage-dotnet-how-to-use-blobs/)部分。

	![“创建 Blob 容器”文本框][1]

1.	完成时按 **Enter** 可创建 Blob 容器，或者按 **Esc** 取消相关操作。成功创建 Blob 容器以后，就会在所选存储帐户的“Blob 容器”文件夹下显示该容器。

	![已创建的 Blob 容器][2]

## 查看 Blob 容器的内容

Blob 容器包含 Blob 和文件夹（其中也可能包含 Blob）。

以下步骤演示了如何在存储资源管理器（预览版）中查看 Blob 容器的内容：

1.	打开存储资源管理器（预览版）。
1.	在左窗格中，展开包含你想要查看的 Blob 容器的存储帐户。
1.	展开存储帐户的“Blob 容器”。
1.	右键单击你想要查看的 Blob 容器，然后从上下文菜单中选择“打开 Blob 容器编辑器”。也可双击你想要查看的 Blob 容器。

	![“打开 Blob 容器编辑器”上下文菜单][19]

1.	主窗格将显示 Blob 容器的内容。

	![Blob 容器编辑器][3]

## 删除 Blob 容器

你可以根据需要轻松地创建和删除 Blob 容器。

以下步骤演示了如何在存储资源管理器（预览版）中删除 Blob 容器：

1.	打开存储资源管理器（预览版）。
1.	在左窗格中，展开包含你想要查看的 Blob 容器的存储帐户。
1.	展开存储帐户的“Blob 容器”。
1.	右键单击你想要删除的 Blob 容器，然后从上下文菜单中选择“删除”。也可通过按“删除”来删除当前选定的 Blob 容器。

	![“删除 Blob 容器”上下文菜单][4]

1.	出现确认对话框时，选择“是”。

	![“删除 Blob 容器”确认][5]

## 复制 Blob 容器

你可以通过存储资源管理器（预览版）将 Blob 容器复制到剪贴板，然后再将该 Blob 容器粘贴到另一存储帐户中。

以下步骤演示了如何将 Blob 容器从一个存储帐户复制到另一个存储帐户。

1.	打开存储资源管理器（预览版）。
1.	在左窗格中，展开包含你想要复制的 Blob 容器的存储帐户。
1.	展开存储帐户的“Blob 容器”。
1.	右键单击你想要复制的 Blob 容器，然后从上下文菜单中选择“复制 Blob 容器”。

	![“复制 Blob 容器”上下文菜单][6]

1.	右键单击需要将 Blob 容器粘贴到其中的“目标”存储帐户，然后从上下文菜单中选择“粘贴 Blob 容器”。

	![“粘贴 Blob 容器”上下文菜单][7]

## 获取 Blob 容器的 SAS

[共享访问签名 (SAS)](/documentation/articles/storage-dotnet-shared-access-signature-part-1/) 用于对存储帐户中的资源进行委托访问。
这意味着你可以授权客户端在指定时间段内，以一组指定权限有限地访问你的存储帐户中的对象，而不必共享你的帐户访问密钥。

以下步骤演示了如何为 Blob 容器创建 SAS：

1.	打开存储资源管理器（预览版）。
1.	在左窗格中展开存储帐户，其中包含你想要获取其 SAS 的 Blob 容器。
1.	展开存储帐户的“Blob 容器”。
1.	右键单击所需 Blob 容器，然后从上下文菜单中选择“获取共享访问签名”。

	![“获取 SAS”上下文菜单][8]

1.	在“共享访问签名”对话框中，根据需要为资源指定策略、开始和过期日期、时区以及访问级别。

	![“获取 SAS”选项][9]

1.	指定完 SAS 选项以后，选择“创建”。

1.	然后会显示第二个“共享访问签名”对话框，其中列出了可用来访问存储资源的 Blob 容器以及 URL 和 QueryString。选择要复制到剪贴板的 URL 旁边的“复制”。

	![复制 SAS URL][10]

1.	完成后，选择“关闭”。

## 管理 Blob 容器的访问策略

以下步骤演示了如何管理（添加和删除）Blob 容器的访问策略：

1.	打开存储资源管理器（预览版）。
1.	在左窗格中展开存储帐户，其中包含你想要管理其访问策略的 Blob 容器。
1.	展开存储帐户的“Blob 容器”。
1.	选择所需 Blob 容器，然后从上下文菜单中选择“管理访问策略”。

	![“管理访问策略”上下文菜单][11]

1.	“访问策略”对话框将列出为所选 Blob 容器创建的任何访问策略。

	![“访问策略”选项][12]

1.	根据访问策略管理任务完成以下步骤：

	- **添加新的访问策略** - 选择“添加”。生成后，“访问策略”对话框将显示新添加的访问策略（以及默认设置）。
	- **编辑访问策略** - 进行需要的编辑，然后选择“保存”。
	- **删除访问策略** - 在要删除的访问策略旁边选择“删除”。

## 为 Blob 容器设置公共访问级别

默认情况下，每个 Blob 容器都设置为“无公共访问权限”。

以下步骤演示了如何指定 Blob 容器的公共访问级别。

1.	打开存储资源管理器（预览版）。
1.	在左窗格中展开存储帐户，其中包含你想要管理其访问策略的 Blob 容器。
1.	展开存储帐户的“Blob 容器”。
1.	选择所需 Blob 容器，然后从上下文菜单中选择“设置公共访问级别”。

	![“设置公共访问级别”上下文菜单][13]

1.	在“设置容器公共访问级别”对话框中，指定所需的访问级别。

	![“设置公共访问级别”选项][14]

1.	选择“应用”。

## 管理 Blob 容器中的 Blob

创建 Blob 容器以后，即可将 Blob 上载到该 Blob 容器、将 Blob 下载到本地计算机、在本地计算机上打开 Blob，等等。

以下步骤演示如何管理 Blob 容器中的 Blob（和文件夹）。

1.	打开存储资源管理器（预览版）。
1.	在左窗格中，展开包含你想要管理的 Blob 容器的存储帐户。
1.	展开存储帐户的“Blob 容器”。
1.	双击你想要查看的 Blob 容器。
1.	主窗格将显示 Blob 容器的内容。

	![查看 Blob 容器][3]

1.	主窗格将显示 Blob 容器的内容。

1.	根据所要执行的任务完成以下步骤：

	- **将文件上载到 Blob 容器**

		1.	在主窗格的工具栏上选择“上载”，然后通过下拉菜单“上载文件”。

			![“上载文件”菜单][15]

		1.	在“上载文件”对话框中，选择“文件”文本框右侧的省略号 (**…**) 按钮，以选择要上载的文件。

			![“上载文件”选项][16]

		1.	将类型指定为“Blob 类型”。[通过 .NET 开始使用 Azure Blob 存储](/documentation/articles/storage-dotnet-how-to-use-blobs/)一文说明了不同 Blob 类型的区别。

		1.	（可选）指定要将选定文件上载到其中的目标文件夹。如果目标文件夹不存在，系统将会创建一个。

		1.	选择“上载”。

	- **将文件夹上载到 Blob 容器**

		1.	在主窗格的工具栏上选择“上载”，然后通过下拉菜单“上载文件夹”。

			![“上载文件夹”菜单][17]

		1.	在“上载文件夹”对话框中，选择“文件夹”文本框右侧的省略号 (**…**) 按钮，以选择要上载其内容的文件夹。

			![“上载文件夹”选项][18]

		1.	将类型指定为“Blob 类型”。[通过 .NET 开始使用 Azure Blob 存储](/documentation/articles/storage-dotnet-how-to-use-blobs/#blob-service-concepts)一文说明了不同 Blob 类型的区别。

		1.	（可选）指定要将选定文件夹的内容上载到其中的目标文件夹。如果目标文件夹不存在，系统将会创建一个。

		1.	选择“上载”。

	- **将 Blob 下载到本地计算机**

		1.	选择要下载的 Blob。

		1.	在主窗格的工具栏上，选择“下载”。

		1.	在“指定已下载 Blob 的保存位置”对话框中，指定要将 Blob 下载到其中的位置，以及要为 Blob 提供的名称。

		1.	选择“保存”。

	- **在本地计算机上打开 Blob**

		1.	选择要打开的 Blob。

		1.	在主窗格的工具栏上，选择“打开”。

		1.	将使用与 Blob 的基础文件类型相关联的应用程序下载和打开 Blob。

	- **将 Blob 复制到剪贴板**

		1.	选择要复制的 Blob。

		1.	在主窗格的工具栏上，选择“复制”。

		1.	在左窗格中导航到另一 Blob 容器，然后通过双击在主窗格中查看它。

		1.	在主窗格的工具栏上选择“粘贴”，以便创建 Blob 的副本。

	- **删除 Blob**

		1.	选择要删除的 Blob。

		1.	在主窗格的工具栏上，选择“删除”。

		1.	出现确认对话框时，选择“是”。

## 后续步骤

- 查看[最新的存储资源管理器（预览版）发行说明和视频](http://www.storageexplorer.com)。
- 了解如何[使用 Azure Blob、表、队列和文件创建应用程序](/documentation/services/storage/)。

[0]: ./media/vs-azure-tools-storage-explorer-blobs/blob-containers-create-context-menu.png
[1]: ./media/vs-azure-tools-storage-explorer-blobs/blob-container-create.png
[2]: ./media/vs-azure-tools-storage-explorer-blobs/blob-container-create-done.png
[3]: ./media/vs-azure-tools-storage-explorer-blobs/blob-container-editor.png
[4]: ./media/vs-azure-tools-storage-explorer-blobs/blob-container-delete-context-menu.png
[5]: ./media/vs-azure-tools-storage-explorer-blobs/blob-container-delete-confirmation.png
[6]: ./media/vs-azure-tools-storage-explorer-blobs/blob-container-copy-context-menu.png
[7]: ./media/vs-azure-tools-storage-explorer-blobs/blob-containers-paste-context-menu.png
[8]: ./media/vs-azure-tools-storage-explorer-blobs/blob-container-get-sas-context-menu.png
[9]: ./media/vs-azure-tools-storage-explorer-blobs/blob-container-get-sas-options.png
[10]: ./media/vs-azure-tools-storage-explorer-blobs/blob-container-get-sas-urls.png
[11]: ./media/vs-azure-tools-storage-explorer-blobs/blob-container-manage-access-policies-context-menu.png
[12]: ./media/vs-azure-tools-storage-explorer-blobs/blob-container-manage-access-policies-options.png
[13]: ./media/vs-azure-tools-storage-explorer-blobs/blob-container-set-public-access-level-context-menu.png
[14]: ./media/vs-azure-tools-storage-explorer-blobs/blob-container-set-public-access-level-options.png
[15]: ./media/vs-azure-tools-storage-explorer-blobs/blob-upload-files-menu.png
[16]: ./media/vs-azure-tools-storage-explorer-blobs/blob-upload-files-options.png
[17]: ./media/vs-azure-tools-storage-explorer-blobs/blob-upload-folder-menu.png
[18]: ./media/vs-azure-tools-storage-explorer-blobs/blob-upload-folder-options.png
[19]: ./media/vs-azure-tools-storage-explorer-blobs/blob-container-open-editor-context-menu.png

<!---HONumber=Mooncake_1017_2016-->