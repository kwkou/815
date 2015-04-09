<properties title="如何创建 RemoteApp 的混合部署" pageTitle="如何创建 RemoteApp 的混合部署" description="了解如何创建连接到您的内部网络的 RemoteApp 的部署。" metaKeywords="" services="" solutions="" documentationCenter="" authors="elizapo"  />

# 如何创建 RemoteApp 的云部署

有两种类型的 RemoteApp 部署：

-   云：完全位于 Azure 中并使用 Azure 管理门户中的“快速创建”选项进行创建。
-   混合：包括用于本地访问的虚拟网络，并使用管理门户中的“使用 VPN 创建”选项进行创建。

本教程将指导您完成创建云部署的过程。有四个步骤：

1.  创建 RemoteApp 服务。
2.  （可选）配置目录同步。RemoteApp 需要此操作来将用户、组、联系人和密码从本地 Active Directory 同步到 Azure Active Directory 租户。
3.  发布 RemoteApp 程序。
4.  配置用户访问权限。

**开始之前**

您需要在创建服务之前执行以下操作：

-   收集有关您想要授予访问权限的用户和组的信息。这可以是用户或组的 Microsoft 帐户信息，也可以是 Active Directory 组织的帐户信息。

## **步骤 1：创建 RemoteApp 服务**

1.  在 [Windows Azure 管理门户][Windows Azure 管理门户]中，请转到 RemoteApp 页上。
2.  单击“新建”\>“快速创建”。

3.  输入您的服务的名称并选择您所在的区域。
4.  选择您想要用于创建此服务的订阅。
5.  选择要用于此服务的模板。

    **提示：**您的 RemoteApp 的订阅附带了一个包含 Office 2013 程序、一些已发布的模板映像（如 Word）和其他已准备好发布的模板映像。

6.  单击“创建 RemoteApp 服务”。

    **重要说明：**它可能需要 30 分钟来设置您的服务。

创建 RemoteApp 服务后，请转到 RemoteApp 的“快速启动”页面以继续执行设置步骤。

## **步骤 2：配置 Active Directory 目录同步（可选）**

如果您想要使用 Active Directory，RemoteApp 会要求 Azure Active Directory 和本地 Active Directory 之间的目录同步，以便将用户、组、联系人和密码同步到 Azure Active Directory 租户。有关规划信息和详细步骤，请参阅[目录同步路线图][目录同步路线图]。

## **步骤 3：发布 RemoteApp 程序**

RemoteApp 程序是您为用户提供的应用程序或程序。它位于您为该服务上载的模板映像中。当用户访问 RemoteApp 程序时，该程序就像是在它们的本地环境中运行，但实际上它是在 Azure 中运行。

在您的用户可以访问 RemoteApp 程序之前，您需要将它们发布到最终用户源 – 用户通过 Azure 门户访问的可用程序的列表。

您可以将多个程序发布到您的 RemoteApp 服务。在 RemoteApp 程序页中，单击“发布”以添加程序。您可以从模板映像的“开始”菜单进行发布，也可以通过在模板映像上为程序指定路径进行发布。如果您选择从“开始”菜单添加，则选择要发布的程序。如果您选择提供程序的路径，请在模板映像上提供程序的名称以及要安装该程序的路径。

## **步骤 4：配置用户访问权限**

由于您已创建了 RemoteApp 服务，您需要添加您想要使其能够使用远程资源的用户和组。如果使用 Active Directory，则您提供访问权限的用户或组需要在 Active Directory 租户中存在，此租户与您用来创建此 RemoteApp 服务的订阅相关联。

1.  在“快速启动”页上，单击“配置用户访问权限”。
2.  输入您想要授予访问权限的组织帐户或组的名称（来自 Active Directory）或 Microsoft 帐户。

    对于用户，请确保使用“user@domain.com”格式。对于组，请输入该组的名称。

3.  对用户或组进行验证后，请单击“保存”。

## 后续步骤

全部完成 - 您已成功创建并部署了 RemoteApp 云部署。下一步是让您的用户下载并安装远程桌面客户端。您可以在 RemoteApp 的“快速启动”页上找到客户端的 URL。然后，让用户登录到 Azure 并访问您发布的 RemoteApp 程序。

  [Windows Azure 管理门户]: http://manage.windowsazure.cn
  [目录同步路线图]: http://msdn.microsoft.com/zh-cn/library/azure/hh967642.aspx
