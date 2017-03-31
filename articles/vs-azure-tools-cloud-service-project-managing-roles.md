<properties
    pageTitle="使用 Visual Studio 在 Azure 云服务项目中管理角色 | Azure"
    description="了解如何使用 Visual Studio 向 Azure 云服务项目中添加新的角色或从中删除现有角色。"
    services="visual-studio-online"
    documentationcenter="na"
    author="TomArcher"
    manager="douge"
    editor="" />
<tags
    ms.assetid="5ec9ae2e-8579-4e5d-999e-8ae05b629bd1"
    ms.service="multiple"
    ms.devlang="dotnet"
    ms.topic="article"
    ms.tgt_pltfrm="na"
    ms.workload="multiple"
    ms.date="11/11/2016"
    wacn.date="03/30/2017"
    ms.author="tarcher" />  


# 使用 Visual Studio 在 Azure 云服务项目中管理角色
创建 Azure 云服务项目后，即可向其添加新角色或从中删除现有角色。你也可以导入现有项目并将其转换为角色。例如，你可以导入 ASP.NET Web 应用程序并将其指定为 Web 角色。

## 添加或删除角色
**添加角色**

在“解决方案资源管理器”中，打开你的云服务项目中“角色”节点的快捷菜单，然后选择“添加”。你可以从当前解决方案中选择现有 Web 角色或辅助角色，或创建新的 Web 或辅助角色项目。也可以选择适当的项目（如 ASP.NET Web 应用程序项目），然后将其与角色项目相关联。

**删除角色关联**

在“解决方案资源管理器”中云服务项目的“角色”节点中，打开要删除的角色的快捷菜单，然后选择“删除”。

## 在云服务中删除和添加角色
如果你从云服务项目中删除了角色，但后来又决定将该角色添加回项目中，则只会添加角色声明和基本特性，如终结点和诊断信息。其他资源或引用不会添加到 ServiceDefinition.csdef 文件或 ServiceConfiguration.cscfg 文件中。如果你想添加此类信息，则必须将其手动添加回这些文件中。

例如，你可能删除了 Web 服务角色，但后来又决定将此角色添加回解决方案中。如果你这样做，则会出现错误。为了防止出现此错误，必须将以下 XML 中显示的 `<LocalResources>` 元素添加回 ServiceDefinition.csdef 文件中。使用添加回项目中的 Web 服务角色的名称作为 **<LocalStorage>** 元素的名称属性的一部分。在此示例中，Web 服务角色的名称为 **WCFServiceWebRole1**。

    <WebRole name="WCFServiceWebRole1">
        <Sites>
          <Site name="Web">
            <Bindings>
              <Binding name="Endpoint1" endpointName="Endpoint1" />
            </Bindings>
          </Site>
        </Sites>
        <Endpoints>
          <InputEndpoint name="Endpoint1" protocol="http" port="80" />
        </Endpoints>
        <Imports>
          <Import moduleName="Diagnostics" />
        </Imports>
       <LocalResources>
          <LocalStorage name="WCFServiceWebRole1.svclog" sizeInMB="1000" cleanOnRoleRecycle="false" />
       </LocalResources>
    </WebRole>

## 后续步骤
有关如何在 Visual Studio 中配置角色的详细信息，请参阅[使用 Visual Studio 配置 Azure 云服务的角色](/documentation/articles/vs-azure-tools-configure-roles-for-cloud-service/)。

<!---HONumber=Mooncake_0320_2017-->
<!-- Update_Description: wording update -->