<properties
  pageTitle="Azure IoT 套件和 Azure Active Directory | Azure"
  description="介绍 Azure IoT 套件如何使用 Azure Active Directory 管理权限。"
  services=""
  suite="iot-suite"
  documentationCenter=""
  author="dominicbetts"
  manager="timlt"
  editor=""/>  


<tags
  ms.service="iot-suite"
  ms.devlang="na"
  ms.topic="article"
  ms.tgt_pltfrm="na"
  ms.workload="na"
  ms.date="02/08/2017"
  wacn.date="03/03/2017"
  ms.author="dominicbetts"/>
  
# azureiotsuite.cn 站点权限

## 登录时发生的情况

在 [azureiotsuite.cn][lnk-azureiotsuite] 上首次登录时，该站点会基于当前所选 Azure Active Directory (AAD) 租户和 Azure 订阅来确定用户拥有的权限级别。

1. 首先，为了填充用户名旁边显示的租户列表，该站点会从 Azure 中找到用户所属的 AAD 租户。当前，该站点一次只能获取一个租户的用户令牌。因此，当用户使用右上角的下拉列表切换租户时，该站点会将用户登录到该租户，以获取该租户的令牌。
2. 接下来，站点从 Azure 查明你所具有的与所选租户关联的订阅。创建新的预配置解决方案时，你会看到可用订阅。
3. 最后，站点检索标记为预配置解决方案的订阅和资源组中的所有资源，并填充主页上的磁贴。

下列各部分介绍用于控制对预配置解决方案的访问的角色。

## AAD 角色
AAD 角色可控制设置预配置解决方案以及在预配置解决方案中管理用户的能力。

有关 AAD 中的管理员角色的详细信息，请参阅[在 Azure AD 中分配管理员角色][lnk-aad-admin]。本文重点介绍预配置解决方案使用的**全局管理员**和**域用户/成员**角色。

**全局管理员：**对于每个 AAD 租户，可以存在许多个全局管理员。你在创建某个 AAD 租户时，默认情况下会成为该租户的全局管理员。全局管理员可以预配预配置解决方案，并被分配了其 AAD 租户内的应用程序的 **ADMINISTRATOR** 角色。但是，如果相同 AAD 租户中的其他用户创建应用程序，则向全局管理员授予的默认角色是 **IMPLICIT READ ONLY**。全局管理员可以使用 [Azure 经典管理门户][lnk-classic-portal]为应用程序分配角色。

**域用户/组成员：**对于每个 AAD 租户，可以存在许多个域用户/成员。域用户可以通过 [azureiotsuite.cn][lnk-azureiotsuite] 站点设置预配置解决方案。对于他们设置的应用程序，向他们授予的默认角色是 **ADMINISTRATOR**。他们可以使用 build.cmd 脚本在 [azure-iot-remote-monitoring][lnk-rm-github-repo] 或 [azure-iot-predictive-maintenance][lnk-pm-github-repo] 存储库中创建应用程序。但是，向他们授予的默认角色是 **IMPLICIT READONLY**，因为它们无权分配角色。如果 AAD 租户中的其他用户创建应用程序，则默认情况下对于该应用程序，会向他们分配 **IMPLICIT READONLY** 角色。他们无法为应用程序分配角色；因此无法为应用程序添加用户或用户的角色（即使是他们预配的应用程序）。

**来宾用户/来宾：**对于每个 AAD 租户，可以存在许多个来宾用户/来宾。来宾用户在 AAD 租户中拥有有限的权利集。因此，来宾用户无法在 AAD 租户中设置预配置解决方案。

有关详细信息，请参阅以下资源：

- [在 Azure AD 中创建或编辑用户][lnk-create-edit-users]
- [在 AAD 中分配应用角色][lnk-assign-app-roles]

## Azure 订阅管理员角色
Azure 管理员角色可控制将 Azure 订阅映射到 AD 租户的能力。

## 应用程序角色
应用程序角色可在预配置解决方案中控制对设备的访问。

在设置预配置解决方案时创建的应用程序中定义了的两个明确角色和一个隐式角色。

-   **ADMINISTRATOR：**拥有添加、管理和删除设备的完全控制权限

-   **READ ONLY：**可以查看设备

-   **IMPLICIT READ ONLY：**此角色与 Read Only 相同，但是会授予给 AAD 租户的所有用户。创建此角色是为了在开发过程中方便操作。可以通过修改 [RolePermissions.cs][lnk-resource-cs] 源文件来删除此角色。

### 更改用户的应用程序角色
可以使用下面的过程在 Active Directory 中使用户成为预配置解决方案的管理员。

你必须是 AAD 全局管理员才能更改用户的角色：

1. 转到 [Azure 经典管理门户][lnk-classic-portal]。
2. 选择“Active Directory”。
3. 单击 AAD 租户的名称（在预配解决方案时在 azureiotsuite.cn 上选择的目录）。
4. 单击“应用程序”。
5. 单击与预配置解决方案名称匹配的应用程序名称。如果在列表中看不到你的应用程序，请将“显示”下拉列表切换到“我公司拥有的应用程序”，然后单击复选标记。
6. 单击“用户”。
7. 选择要切换角色的用户。
8. 单击“分配”并选择要分配给用户的角色（如“管理员”），单击复选标记。

## 常见问题
### 我是服务管理员，要更改我的订阅与特定 AAD 租户之间的目录映射。如何完成此任务？
1. 转到 [Azure 经典管理门户][lnk-classic-portal]，在左侧的服务列表中单击“设置”。
2. 选择要将目录映射更改为的订阅。
3. 单击“编辑目录”。
4. 在下拉列表中选择要使用的“目录”。单击向前箭头。
5. 确认目录映射和受影响的协同管理员。请注意，如果你在从另一个目录进行移动，则会删除原始目录中的所有协同管理员。

### 我是 AAD 租户上的域用户/成员，我创建了一个预配置解决方案。如何针对我的应用程序向我分配角色？
请全局管理员将你分配为 AAD 租户中的全局管理员以获得你自己向用户分配角色的权限，或是请全局管理员向你分配角色。如果要更改预配置解决方案部署到的 AAD 租户，请参阅下一个问题。

### 如何切换将我的远程监控预配置解决方案和应用程序分配到的 AAD 租户？
可以从 <https://github.com/Azure/azure-iot-remote-monitoring> 运行云部署并使用新创建的 AAD 租户重新部署。由于你在创建 AAD 租户时默认情况下会成为全局管理员，因此拥有添加用户以及向这些用户分配角色的权限。

1. 在 [Azure 经典管理门户][lnk-classic-portal]中创建 AAD 目录。

2. 转到 <https://github.com/Azure/azure-iot-remote-monitoring>。

3. 运行 `build.cmd cloud [debug | release] {name of previously deployed remote monitoring solution}`（例如 `build.cmd cloud debug myRMSolution`）

4. 出现提示时，将 **tenantid** 设置为新创建的租户，而不是以前的租户。


### 为何会出现以下错误？ “你的帐户没有创建解决方案的正确权限。请咨询帐户管理员或使用其他帐户进行尝试。”
请查看下图以获得指南：

![][img-flowchart]  

> [AZURE.NOTE] 如果在验证你是 AAD 租户的全局管理员和订阅的共同管理员后，继续看到此错误，请让帐户管理员删除该用户，并按以下顺序重新分配必要的权限。首先，将用户添加为全局管理员，然后将用户添加为 Azure 订阅的共同管理员。如果问题仍然存在，请[在线申请支持][lnk-help-support]。

**为何在我具有 Azure 订阅时会出现以下错误？** *创建预配置解决方案需要 Azure 订阅。只需几分钟即可创建一个帐户。*

如果你确定具有 Azure 订阅，请验证订阅的租户映射，并确保在下拉列表中选择正确租户。如果验证了所需租户是正确的，请按照上图验证订阅和此 AAD 租户的映射。

## 后续步骤

若要继续了解 IoT 套件，请参阅如何[自定义预配置解决方案][lnk-customize]。

[img-flowchart]: ./media/iot-suite-permissions/flowchart.png

[lnk-azureiotsuite]: https://www.azureiotsuite.cn/
[lnk-rm-github-repo]: https://github.com/Azure/azure-iot-remote-monitoring
[lnk-pm-github-repo]: https://github.com/Azure/azure-iot-predictive-maintenance
[lnk-aad-admin]: /documentation/articles/active-directory-assign-admin-roles/
[lnk-classic-portal]: https://manage.windowsazure.cn/
[lnk-create-edit-users]: /documentation/articles/active-directory-create-users/
[lnk-assign-app-roles]: /documentation/articles/active-directory-application-manifest/
[lnk-resource-cs]: https://github.com/Azure/azure-iot-remote-monitoring/blob/master/DeviceAdministration/Web/Security/RolePermissions.cs
[lnk-help-support]: https://www.azure.cn/support/support-ticket-form/?l=zh-cn
[lnk-customize]: /documentation/articles/iot-suite-guidance-on-customizing-preconfigured-solutions/

<!---HONumber=Mooncake_0227_2017-->
<!--Update_Description:update wording and link references-->