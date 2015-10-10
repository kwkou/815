<properties
   pageTitle="下载 Azure 备份中的保管库凭据"
   description="了解如何使用保管库凭据在备份保管库和 Azure 备份服务中对你的计算机进行身份验证"
   services="backup"
   documentationCenter=""
   authors="Jim-Parker"
   manager="shreeshd"
   editor=""/>
<tags
   ms.service="backup"
   ms.date="08/11/2015"
   wacn.date="09/15/2015"/>

# 使用保管库凭据向 Azure 备份服务进行身份验证

在本地服务器（Windows 客户端、Windows Server 或 SCDPM 服务器）将数据备份到 Azure 之前，需要在备份保管库中对服务器进行身份验证。身份验证是使用“保管库凭据”实现的。保管库凭据的概念类似于 Azure PowerShell 中使用的“发布设置”文件概念。

## 什么是保管库凭据文件？

保管库凭据文件是在门户中为每个备份保管库生成的证书。然后，门户会将公钥上载到访问控制服务 (ACS)。证书的私钥将提供给用户作为工作流的一部分，它被指定为计算机注册工作流的输入。当计算机向 Azure 备份服务中标识的保管库发送备份数据时，该证书将对计算机进行身份验证。

值得一提的是，保管库凭据只会在注册工作流中使用。用户需负责确保保管库凭据文件不会泄露。如果该文件落入恶意用户之手，他们可以使用保管库凭据文件将其他计算机注册到同一个保管库。但是，但是备份数据已使用属于客户的通行短语加密，因此现有的备份数据不会泄露。为了消除这种忧虑，保管库凭据设置为 48 小时后过期。你可以下载备份保管库的保管库凭据任意次数 - 但是，在运行注册工作流期间，只有最新的保管库凭据文件才适用。

## 下载保管库凭据文件

从 Azure 门户通过安全通道下载保管库凭据文件。Azure 备份服务并不知道证书的私钥，并且私钥不会保存在门户或服务中。使用以下步骤将保管库凭据下载到本地计算机。

1.  登录到[管理门户](https://manage.windowsazure.cn/)。
2.  在左侧导航窗格中单击“恢复服务”，然后选择您创建的备份保管库。 
3. 单击云图标转到备份保管库的“快速启动”视图。<br/>
![快速查看][1]

4.  在“快速启动”页上，单击“下载保管库凭据”。门户将生成可供下载的保管库凭据文件。<br/>
![下载][2]

5.  门户将使用保管库名称和当前日期的组合生成保管库凭据。单击“保存”将保管库凭据下载到本地帐户的 downloads 文件夹，或者从“保存”菜单中选择“另存为”以指定保管库凭据保存到的位置。

## 说明
+ 截至 2015 年 3 月，用户无法以编程方式（例如：PowerShell）下载保管库凭据。

+ 确保将保管库凭据保存可从计算机访问的位置。如果将它存储在文件共享/SMB 中，请检查访问权限。

+ 保管库凭据文件仅在注册工作流中使用。

+ 保管库凭据文件在 48 小时后过期，可从门户下载。

+ 如有工作流方面的任何问题，请参阅 Azure 备份[常见问题](/documentation/articles/backup-azure-backup-faq)。

## 后续步骤
[下载、注册和安装 Azure 备份代理](/documenatation/articles/backup-azure-backup-download-register)

<!--Image references-->
[1]: ./media/backup-azure-backup-download-vc/quickview.png
[2]: ./media/backup-azure-backup-download-vc/downloadvc.png

<!---HONumber=67-->