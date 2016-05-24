<properties
	pageTitle="使用存储空间资源管理器（预览版）管理 Azure 存储空间资源 | Azure"
	description="介绍如何使用 Azure 存储空间资源管理器（预览版）来创建和管理 Azure 存储空间资源。"
	services="visual-studio-online"
	documentationCenter="na"
	authors="TomArcher"
	manager="douge"
	editor="" />

 <tags
	ms.service="visual-studio-online"
	ms.date="01/30/2016"
	wacn.date="" />

# 使用存储空间资源管理器（预览版）管理 Azure 存储空间资源

Azure 存储空间资源管理器（预览版）是一个独立的工具，可帮助你轻松管理 Azure 存储帐户。当你想要快速管理 Azure 门户的外部存储时（例如，你要在 Visual Studio 中开发应用），此工具十分有用。此预览版可让你轻松使用 Blob 存储。你可以创建和删除容器，上载、下载和删除 Blob，以及在所有容器和 Blob 中搜索。高级功能可让开发人员和操作员使用 SAS 密钥和策略。Windows 开发人员还可以借助 Azure 存储模拟器，使用本地开发存储帐户来测试代码。

若要查看或管理存储空间资源管理器中的存储资源，需要能够以订阅或外部存储帐户访问 Azure 存储帐户。如果你没有存储帐户，只需花费几分钟就能创建一个帐户。或者，请参阅 [1 元试用版](/pricing/1rmb-trial/)。

## 管理 Azure 帐户和订阅

若要在存储空间资源管理器中查看 Azure 存储空间资源，需要登录到包含一个或多个有效订阅的 Azure 帐户。如果你有多个 Azure 帐户，可将它们添加存储空间资源管理器，然后选择想要包含在存储空间资源管理器资源视图中的订阅。如果你未曾使用过 Azure，或尚未将必要的帐户添加到 Visual Studio，系统将提示你登录到 Azure 帐户。

### 将 Azure 帐户添加到存储空间资源管理器

1.	在“存储空间资源管理器”工具栏上选择“设置”（齿轮）图标。
1.	选择“添加帐户”链接。登录到你要浏览其存储空间资源的 Azure 帐户。帐户选择器下拉列表中应已选择刚刚添加的帐户。该帐户的所有订阅出现在帐户条目下。

	![][0]

1.	选中你要浏览的帐户订阅的复选框，然后选择“应用”按钮。

	![][1]

	所选订阅的 Azure 存储空间资源将出现在存储空间资源管理器中。

### 附加外部存储

1. 获取想要附加的存储帐户的帐户名称和密钥。
	1.	在 Azure 管理门户中，选择要附加的存储帐户。
	1.	点击底部的“管理访问密钥”, 复制“存储帐户名称”和“主访问密钥”值。


1.	在“存储空间资源管理器”的“存储帐户”节点的快捷菜单中，选择“附加外部存储”命令。

	![][3]

1. 在“帐户名称”框中输入存储帐户名称，在“帐户密钥”框中输入主访问密钥。选择“确定”按钮以继续。

	![][4]

	外部存储将出现在存储空间资源管理器中。

	![][5]

1. 若要删除外部存储，请在外部存储的快捷菜单中选择“分离”命令。

	![][6]

## 查看和浏览存储资源

若要在存储空间资源管理器导航到 Azure 存储空间资源并查看其信息，请展开存储类型，然后选择资源。所选资源的相关信息将出现在“存储空间资源管理器”底部的“操作”和“属性”选项卡上。

![][7]

-	“操作”选项卡显示可以在存储空间资源管理器中针对所选存储资源执行的操作，例如打开、复制或删除。“操作”还会显示在资源的快捷菜单中。

-	“属性”选项卡显示存储资源的属性，例如其类型、区域设置、关联的资源组和 URL。

所有存储帐户都有“在门户中打开”操作。当你选择此操作时，存储空间资源管理器将在 Azure 预览门户中显示所选的存储帐户。

根据所选的资源，有时会显示其他操作和属性值。例如，“Blob 容器”、“队列”和“表”节点都有“创建”操作。各个项（例如 Blob 容器）具有“打开”、“删除”和“获取共享访问签名”等操作。当你选择存储帐户 Blob 时，将显示用于打开 Blob 编辑器的操作。

## 搜索存储帐户和 Blob 容器

若要在 Azure 订阅中查找具有特定名称的存储帐户和 Blob 容器，请在存储空间资源管理器的“搜索”框中输入名称。

![][8]

当你在“搜索”框中输入字符时，只有与这些字符匹配的存储帐户和 Blob 容器才出现在资源树中。若要清除搜索，请在“搜索”框中选择“x”按钮。

## 编辑存储帐户

若要添加或更改存储帐户的内容，请选择该存储类型对应的“打开编辑器”命令。可以在所选项的快捷菜单中选择操作，或者在存储空间资源管理器底部的“操作”选项卡上选择操作。

![][9]

可以创建或删除 Blob 容器、队列和表。还可以通过选择“打开 Blob 容器编辑器”操作来编辑存储空间资源管理器中的 Blob。

### 编辑 Blob 容器

1.	选择“打开 Blob 容器编辑器”操作。“Blob 容器编辑器”将出现在右窗格中。

	![][10]

1.	选择“上载”按钮，然后选择“上载文件”命令。

	![][11]

	如果要上载的文件位于单个文件夹中，你可以改为选择“上载文件夹”命令。

1. 在“上载文件”对话框中，选择“文件”框右侧的省略号 (**…**) 按钮，以选择要上载的文件。然后，选择要上载的 Blob 类型（块、页或追加）。如果需要，你可以选择将文件上载到 Blob 容器中的文件夹。在“上载到文件夹(可选)”框中输入文件夹的名称。如果该文件夹不存在，系统将会创建。

	![][12]

	在以下屏幕截图中，已将三个映像文件上载到“映像”Blob 容器中名为“我的新文件”的新文件夹中。

	![][13]

	可以使用 Blob 编辑器工具栏上的按钮来选择、下载、打开、复制和删除文件，以及执行其他操作。对话框底部的“活动”窗格显示操作是否成功，并可让你从视图中仅删除成功的活动，或完全清除窗格。选择已上载文件旁边的“+”图标可以查看已上载文件的详细列表。

## 创建共享访问签名 (SAS)

对于某些操作，你可能需要 SAS 才能访问存储资源。可以使用存储空间资源管理器创建 SAS。

1.	选择要为其创建 SAS 的项，然后在“操作”窗格或项目的快捷菜单中，选择“获取共享访问签名”命令。

	![][14]

1.	在“共享访问签名”对话框中，选择策略、开始和过期日期及时区。同时，请选中所需资源访问级别对应的复选框，例如只读、读写等等。完成后，选择“创建”按钮以创建 SAS。

	![][15]

1.	“共享访问签名”对话框列出了可用来访问存储资源的容器以及 URL 和查询字符串。选择“复制”按钮以复制字符串。

	![][16]

## 管理 SAS 和权限

若要控制对 Blob 容器的访问，可以选择“管理访问控制列表”和“设置公共访问级别”命令。

-	“管理访问控制列表”可让你对所选 Blob 容器添加、编辑和删除策略（用户是否可以读取、写入等等）。
-	“设置公共访问级别”可让你确定多少个公共访问用户可以获取资源。  

-

1.	选择 Blob 容器，然后在快捷菜单或“操作”窗格中选择“管理访问控制列表”命令。

	![][17]

1.	在“访问控制列表”对话框中，选择“添加”按钮以添加访问策略。选择访问策略，然后为它选择权限。完成后，请选择“保存”按钮。

	![][18]

1.	若要设置某个 Blob 容器的访问级别，请在存储空间资源管理器中选择该容器，然后在快捷菜单或“操作”窗格中选择“设置公共访问级别”命令。

	![][19]

1.	在“设置容器公共访问级别”对话框中，选择要提供给公共用户的访问级别所对应的选项按钮，然后选择“应用”按钮。

	![][20]

## 后续步骤
阅读 [Introduction to Azure Storage](/documentation/articles/storage-introduction)（Azure 存储空间简介）中的文章，了解 Azure 存储空间服务中的功能。

[0]: ./media/vs-azure-tools-storage-manage-with-storage-explorer/AddAccount1c.png
[1]: ./media/vs-azure-tools-storage-manage-with-storage-explorer/AddAccount2c.png
[2]: ./media/vs-azure-tools-storage-manage-with-storage-explorer/External1c.png
[3]: ./media/vs-azure-tools-storage-manage-with-storage-explorer/External2c.png
[4]: ./media/vs-azure-tools-storage-manage-with-storage-explorer/External3c.png
[5]: ./media/vs-azure-tools-storage-manage-with-storage-explorer/External4c.png
[6]: ./media/vs-azure-tools-storage-manage-with-storage-explorer/External5c.png
[7]: ./media/vs-azure-tools-storage-manage-with-storage-explorer/Navigatec.png
[8]: ./media/vs-azure-tools-storage-manage-with-storage-explorer/Searchc.png
[9]: ./media/vs-azure-tools-storage-manage-with-storage-explorer/Edit1c.png
[10]: ./media/vs-azure-tools-storage-manage-with-storage-explorer/Edit2c.png
[11]: ./media/vs-azure-tools-storage-manage-with-storage-explorer/Edit3c.png
[12]: ./media/vs-azure-tools-storage-manage-with-storage-explorer/Edit4c.png
[13]: ./media/vs-azure-tools-storage-manage-with-storage-explorer/Edit5c.png
[14]: ./media/vs-azure-tools-storage-manage-with-storage-explorer/SAS1c.png
[15]: ./media/vs-azure-tools-storage-manage-with-storage-explorer/SAS2c.png
[16]: ./media/vs-azure-tools-storage-manage-with-storage-explorer/SAS3c.png
[17]: ./media/vs-azure-tools-storage-manage-with-storage-explorer/ManageSAS1c.png
[18]: ./media/vs-azure-tools-storage-manage-with-storage-explorer/ManageSAS2c.png
[19]: ./media/vs-azure-tools-storage-manage-with-storage-explorer/ManageSAS3c.png
[20]: ./media/vs-azure-tools-storage-manage-with-storage-explorer/ManageSAS4c.png

<!---HONumber=Mooncake_0516_2016-->