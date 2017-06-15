<properties
    pageTitle="为 Azure 云服务中的角色启用远程桌面连接 | Azure"
    description="如何配置 Azure 云服务应用程序以允许远程桌面连接"
    services="cloud-services"
    documentationcenter=""
    author="seanmck"
    manager="timlt"
    editor="" />  

<tags
    ms.assetid="73ea1d64-1529-4d72-b58e-f6c10499e6bb"
    ms.service="cloud-services"
    ms.workload="tbd"
    ms.tgt_pltfrm="na"
    ms.devlang="na"
    ms.topic="article"
    ms.date="11/28/2016"
    wacn.date="05/22/2017"
    ms.author="seanmck"
    ms.translationtype="Human Translation"
    ms.sourcegitcommit="8fd60f0e1095add1bff99de28a0b65a8662ce661"
    ms.openlocfilehash="8a7a234c3db5fb8f055776d6826a419b03a8d9cc"
    ms.lasthandoff="05/12/2017" />

# <a name="enable-remote-desktop-connection-for-a-role-in-azure-cloud-services"></a>为 Azure 云服务中的角色设置远程桌面连接
> [AZURE.SELECTOR]
- [Azure 门户](/documentation/articles/cloud-services-role-enable-remote-desktop-new-portal/)
- [Azure 经典管理门户](/documentation/articles/cloud-services-role-enable-remote-desktop/)
- [PowerShell](/documentation/articles/cloud-services-role-enable-remote-desktop-powershell/)
- [Visual Studio](/documentation/articles/vs-azure-tools-remote-desktop-roles/)

可以通过远程桌面访问在 Azure 中运行的角色的桌面。 也可以使用远程桌面连接，在应用程序正在运行时排查和诊断其问题。

你可以在开发过程中通过在服务定义中加入远程桌面模块来在角色中启用远程桌面连接，也可以通过远程桌面扩展选择启用远程桌面。首选方法是使用远程桌面扩展，因为即使在部署应用程序后，也能启用远程桌面，而不必重新部署应用程序。

## <a name="configure-remote-desktop-from-the-azure-portal"></a>从 Azure 门户配置远程桌面
Azure 门户使用远程桌面扩展方法，即使在部署应用程序之后，也能启用远程桌面。 使用云服务的“远程桌面”边栏选项卡，可以启用远程桌面，更改用于连接虚拟机的本地 Administrator 帐户、身份验证使用的证书，以及设置到期日期。

1. 单击“云服务”，再单击云服务的名称，然后单击“远程桌面”。

    ![云服务远程桌面](./media/cloud-services-role-enable-remote-desktop-new-portal/CloudServices_Remote_Desktop.png)  


2. 选择希望为单个角色还是所有角色启用远程桌面，然后将切换器的值更改为“已启用”。

3. 填写所需的字段：用户名、密码、到期时间和证书。

    ![云服务远程桌面](./media/cloud-services-role-enable-remote-desktop-new-portal/CloudServices_Remote_Desktop_Details.PNG)


   > [AZURE.WARNING]
   > 当首次启用远程桌面并单击“确定”（复选标记）时，所有角色实例会重新启动。 为避免重新启动，必须对于此角色安装用于对密码进行加密的证书。 若要避免重新启动，请[上载云服务的证书](/documentation/articles/cloud-services-configure-ssl-certificate/#step-3-upload-a-certificate)，然后返回到此对话框。
   >
   >
3. 在“**角色**”中，选择要更新的角色，或选择“**全部**”以选择所有角色。

4. 完成配置更新后，单击“保存”。角色实例需要一些时间才能准备好接收连接。

## <a name="remote-into-role-instances"></a>远程到角色实例
为角色启用远程桌面后，可以直接从 Azure 门户启动连接：

1. 单击“实例”，打开“实例”边栏选项卡。
2. 选择一个配置了远程桌面的角色实例。
3. 单击“连接”  下载角色实例的 RDP 文件。

    ![云服务远程桌面](./media/cloud-services-role-enable-remote-desktop-new-portal/CloudServices_Remote_Desktop_Connect.PNG)  

4. 依次单击“**打开**”和“**连接**”，以启动远程桌面连接。

>[AZURE.NOTE]
> 如果云服务位于 NSG 后，可能需要创建允许端口**3389** 和 **20000** 上的流量的规则。  远程桌面使用端口 **3389**。  云服务实例经过负载均衡，因此无法直接控制要连接到哪个实例。  RemoteForwarder 和 RemoteAccess 代理管理 RDP 流量，允许客户端发送 RDP cookie 和指定要连接到的单个实例。  RemoteForwarder 和 RemoteAccess 代理要求打开端口 **20000***（如果你具有 NSG，此端口可能已被阻止）。

## <a name="additional-resources"></a>其他资源

 - [如何配置云服务](/documentation/articles/cloud-services-how-to-configure/)
 - [云服务常见问题 - 远程桌面](/documentation/articles/cloud-services-faq/#remote-desktop)