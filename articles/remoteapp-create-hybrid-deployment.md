<properties title="如何创建 RemoteApp 的混合部署" pageTitle="如何创建 RemoteApp 的混合部署" description="了解如何创建连接到您的内部网络的 RemoteApp 的部署。" metaKeywords="" services="" solutions="" documentationCenter="" authors="elizapo"  />

# 如何创建 RemoteApp 的混合部署

有两种类型的 RemoteApp 部署：

-   云：完全位于 Azure 中并使用 Azure 管理门户中的“快速创建”选项进行创建。
-   混合：包括用于本地访问的虚拟网络，并使用管理门户中的“使用 VPN 创建”选项进行创建。

本教程将指导您完成创建混合部署的过程。有六个步骤：

1.  创建 RemoteApp 服务。
2.  链接到虚拟网络。
3.  链接模板映像。
4.  配置目录同步。RemoteApp 需要此操作来将用户、组、联系人和密码从本地 Active Directory 同步到 Azure Active Directory 租户。
5.  发布 RemoteApp 程序。
6.  配置用户访问权限。

**开始之前**

您需要在创建服务之前执行以下操作：

-   在 Active Directory 中创建要用作 RemoteApp 服务帐户的用户帐户。限制此帐户的权限，以便它可以仅将计算机加入到域。
-   收集有关您的本地网络的信息：IP 地址信息和 VPN 设备详细信息。
-   安装 [Azure PowerShell][Azure PowerShell] 模块。
-   收集有关您想要授予访问权限的用户和组的信息。这可以是用户或组的 Microsoft 帐户信息，也可以是 Active Directory 组织的帐户信息。
-   创建模板映像以用于您的服务。该模板映像包含您想要向用户提供的程序。有关创建模板映像的详细信息，请参阅[在企业中部署 Azure RemoteApp][在企业中部署 Azure RemoteApp]。

## **步骤 1：创建 RemoteApp 服务**

1.  在 [Windows Azure 管理门户][Windows Azure 管理门户]中，请转到 RemoteApp 页上。
2.  单击“新建”\>“使用 VPN 创建”。
3.  输入您的服务的名称，然后单击“创建 RemoteApp 服务”。

创建 RemoteApp 服务后，请转到 RemoteApp 的“快速启动”页面以继续执行设置步骤。

## **步骤 2：链接到虚拟网络**

虚拟网络允许您的用户通过 RemoteApp 远程资源访问本地网络上的数据。配置虚拟网络链接需要四个步骤：

1.  在“快速启动”页上，单击“链接 remoteapp 虚拟网络”。
2.  选择是创建新的虚拟网络还是使用现有的虚拟网络。对于本教程，我们将创建一个新的网络。
3.  为新的网络提供以下信息：

    -   名称
    -   虚拟网络地址空间
    -   本地地址空间
    -   DNS 服务器 IP 地址
    -   VPN IP 地址

    有关详细信息，请参阅[在管理门户中配置站点到站点 VPN][在管理门户中配置站点到站点 VPN]。

4.  接下来，返回到“快速启动”页，单击“获取脚本”下载脚本，以便配置 VPN 设备来连接到刚创建的虚拟网络。您将需要以下有关 VPN 设备的信息：

    -   供应商
    -   平台
    -   操作系统

    保存脚本，并在 VPN 设备上运行它。

    **注意：**运行此脚本后，虚拟网络将转为“就绪”状态，这可能需要 5-7 分钟。在网络准备就绪之前，您无法使用它。

5.  最后，还是在“快速启动”页上，单击“加入本地域”。将 RemoteApp 服务帐户添加到本地 Active Directory 域。您将需要域名、组织单位、服务帐户的用户名和密码。

## **步骤 3：链接到 RemoteApp 模板映像**

RemoteApp 模板映像包含您想要与用户共享的程序。您可以上载新的模板映像或链接到现有映像（已上载到 Azure 的映像）。

如果要上载新映像，您需要输入名称，并选择映像的位置。在向导的下一页上，您将看到一组 PowerShell cmdlet - 从提升的 Azure PowerShell 提示符复制并运行这些 cmdlet 以上载指定的映像。

如果要链接到现有的模板映像，则只需指定映像名称、位置和关联的 Azure 订阅。

**注意：**您必须使用安装了远程桌面会话主机和桌面体验的 Windows Server 2012 R2 创建模板映像。有关创建模板映像的详细信息，请参阅[在企业中部署 Azure RemoteApp][在企业中部署 Azure RemoteApp]。

## **步骤 4：配置 Active Directory 目录同步**

RemoteApp 需要在 Azure Active Directory 和本地 Active Directory 之间的目录同步来将用户、组、联系人和密码同步到 Azure Active Directory 租户。有关规划信息和详细步骤，请参阅[目录同步路线图][目录同步路线图]。

## **步骤 5：发布 RemoteApp 程序**

RemoteApp 程序是您为用户提供的应用程序或程序。它位于您为该服务上载的模板映像中。当用户访问 RemoteApp 程序时，该程序就像是在它们的本地环境中运行，但实际上它是在 Azure 中运行。

在您的用户可以访问 RemoteApp 程序之前，您需要将它们发布到最终用户源 – 用户通过 Azure 门户访问的可用程序的列表。

您可以将多个程序发布到您的 RemoteApp 服务。在 RemoteApp 程序页中，单击“发布”以添加程序。您可以从模板映像的“开始”菜单进行发布，也可以通过在模板映像上为程序指定路径进行发布。如果您选择从“开始”菜单添加，则选择要发布的程序。如果您选择提供程序的路径，请在模板映像上提供程序的名称以及要安装该程序的路径。

## **步骤 6：配置用户访问权限**

由于您已创建了 RemoteApp 服务，您需要添加您想要使其能够使用远程资源的用户和组。您提供访问权限的用户或组需要在 Active Directory 租户中存在，此租户与您用来创建此 RemoteApp 服务的订阅相关联。

1.  在“快速启动”页上，单击“配置用户访问权限”。
2.  输入您想要授予访问权限的组织帐户或组的名称（来自 Active Directory）或 Microsoft 帐户。

    对于用户，请确保使用“user@domain.com”格式。对于组，请输入该组的名称。

3.  对用户或组进行验证后，请单击“保存”。

## 后续步骤

全部完成 - 您已成功创建并部署了 RemoteApp 混合部署。下一步是让您的用户下载并安装远程桌面客户端。您可以在 RemoteApp 的“快速启动”页上找到客户端的 URL。然后，让用户登录到 Azure 并访问您发布的 RemoteApp 程序。

  [Azure PowerShell]: http://azure.microsoft.com/zh-cn/documentation/articles/install-configure-powershell/
  [在企业中部署 Azure RemoteApp]: http://go.microsoft.com/fwlink/?LinkId=397721
  [Windows Azure 管理门户]: http://manage.windowsazure.cn
  [在管理门户中配置站点到站点 VPN]: http://msdn.microsoft.com/library/azure/dn133795.aspx
  [目录同步路线图]: http://msdn.microsoft.com/zh-cn/library/azure/hh967642.aspx
