<properties 
	pageTitle="上载 Azure Management API 证书 | Azure" 
	description="了解如何为 Azure 经典门户上载 Management API 证书。" 
	services="cloud-services" 
	documentationCenter=".net" 
	authors="Thraka" 
	manager="timlt" 
	editor=""/>

<tags 
	ms.service="na" 
	ms.date="04/18/2016"
	wacn.date="06/13/2016"/>


# 上载 Azure Management API 管理证书

管理证书允许你使用 Azure 提供的服务管理 API 进行身份验证。许多程序和工具（如 Visual Studio 或 Azure SDK）将使用这些证书来自动配置和部署各种 Azure 服务。**这仅适用于 Azure 经典门户**。

>[AZURE.WARNING] 请小心！ 这些类型的证书允许任何使用它们进行身份验证的人管理与它们相关联的订阅。

有关 Azure 证书（包括创建自签名证书）的详细信息，如果您需要便可[使用](/documentation/articles/cloud-services-certs-create/#what-are-management-certificates)。

您还可以使用 [Azure Active Directory](/home/features/identity/) 对客户端代码进行身份验证，以用于自动化目的。

## 上载管理证书

一旦您创建了一个管理证书（仅使用公开密钥的 .cer 文件），您可以将其上载到门户。当该证书在门户中可用时，任何拥有匹配证书（私钥）的人可以通过 Management API 连接并访问与订阅相关联的资源。

1. 登录到 [Azure 管理门户](http://manage.windowsazure.cn)。
2. 单击门户左侧的“设置”（可能需要向下滚动）。 
    
    ![设置](./media/azure-api-management-certs/settings.png)

3. 单击“管理证书”选项卡。

    ![设置](./media/azure-api-management-certs/certificates-tab.png)
    
4. 单击“上载”按钮。

    ![设置](./media/azure-api-management-certs/upload.png)
    
5. 填写对话框信息并单击完成“复选标记”。

    ![设置](./media/azure-api-management-certs/upload-dialog.png)

## 后续步骤

现在，您已经拥有与订阅关联的管理证书，您可以（在本地安装匹配的证书之后）以编程的方式连接到 [Service Management REST API](https://msdn.microsoft.com/zh-cn/library/azure/mt420159.aspx)，并使也和该订阅相关联的各种 Azure 资源自动化。

<!---HONumber=Mooncake_0606_2016-->