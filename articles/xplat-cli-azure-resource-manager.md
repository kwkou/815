<properties linkid="script-xplat-intro" urlDisplayName="Windows Azure Cross-Platform Command-Line Interface" pageTitle="结合使用 Windows Azure 跨平台命令行接口和 Resource Manager" title="结合使用 Windows Azure 跨平台命令行接口和 Resource Manager" metaKeywords="windows azure cross-platform command-line interface Resource Manager, windows azure command-line resource manager, azure command-line resource manager, azure cli resource manager" description="结合使用 Windows Azure 跨平台命令行接口和 Resource Manager" metaCanonical="http://www.windowsazure.cn/zh-cn/script/xplat-cli-intro" umbracoNaviHide="0" disqusComments="1" editor="mollybos" manager="paulettm" documentationCenter="" solutions="" authors="larryfr" services="Haifeng Liu" />
<tags ms.service="Haifeng Liu"
    ms.date="03/10/2015"
    wacn.date=""
    />

# 结合使用 Azure 跨平台命令行接口和 Resource Manager

<div class="dev-center-tutorial-selector sublanding"><a href="/zh-cn/documentation/articles/powershell-azure-resource-manager.md" title="Windows PowerShell">Windows PowerShell</a><a href="/zh-cn/documentation/articles/xplat-cli-azure-resource-manager.md" title="跨平台 CLI" class="current">跨平台 CLI</a></div>

我们最近发布了 Resource Manager 的预览版，这是一个管理 Windows Azure 的新方法。您将从本文了解到如何结合使用 Azure 跨平台命令行接口 (xplat-cli) 和 Resource Manager。

> [WACOM.NOTE] Resource Manager 当前为预览版，不能提供与 Azure 服务管理相同级别的管理功能。

> [WACOM.NOTE] 如果您尚未安装和配置 xplat-cli，有关如何安装、配置和使用 xplat-cli 的详细步骤，请参阅[安装和配置 Windows Azure 跨平台命令行接口][安装和配置 Windows Azure 跨平台命令行接口]。

## Resource Manager

通过 Resource Manager，您可以将一组*资源*（用户管理的实体，例如数据库服务器、数据库或网站）作为单个逻辑单元进行管理，或管理*资源组*。例如，资源组可能包含网站和 SQL数据库 资源。

为了支持一种用于描述资源组内的资源所做更改的更具声明性的方法，Resource Manager 使用*模板*（为 JSON 文档）。模板语言也可用于描述可以在运行命令时填充任一内联的参数，或存储在单独 JSON 文件中的参数。这样您只需提供不同参数，即可使用同一模板轻松地创建新资源。例如，创建网站的模板将包含站点名称参数、网站所在区域的参数以及其他常见参数。

> [WACOM.NOTE] 本文未描述模板语言的细节。相关文档可用时，将更新本主题以提供指向参考文档的链接。
>
> 但是您可以使用 `azure group template download` 命令来下载和修改 Microsoft 和合作伙伴从模板库提供的模板。

使用模板修改或创建组时，将创建*部署*，随后将其应用于该组。

## 身份验证

当前通过 xplat-cli 使用 Resource Manager 需要您使用组织帐户对 Windows Azure 进行身份验证。无法使用 Microsoft 帐户或通过发布设置文件安装的证书进行身份验证。

有关使用组织帐户进行身份验证的详细信息，请参阅[安装和配置 Windows Azure 跨平台命令行接口][安装和配置 Windows Azure 跨平台命令行接口]。

## 使用组和模板

1.  Resource Manager 目前为预览版，因此默认情况下未启用使用 Resource Manager 的 xplat-cli 命令。请使用以下命令来启动这些命令：

        azure config mode arm

    > [WACOM.NOTE] Resource Manager 模式和 Azure 服务管理模式互斥。即在一种模式下创建的资源不能从另一种模式进行管理。

2.  使用模板时，您可以创建自己的模板，或使用模板库中的模板。要列出库中可用的模板，请使用以下命令。

        azure group template list

    响应将列出发布者和模板名称，并显示以下内容。

        data:    Publisher               Name
        data:    ----------------------------------------------------------------------------
        data:    Microsoft               Microsoft.WebSite.0.1.0-preview1
        data:    Microsoft               Microsoft.PHPStarterKit.0.1.0-preview1
        data:    Microsoft               Microsoft.HTML5EmptySite.0.1.0-preview1
        data:    Microsoft               Microsoft.ASPNETEmptySite.0.1.0-preview1
        data:    Microsoft               Microsoft.WebSiteMySQLDatabase.0.1.0-preview1

3.  要查看用于创建 Azure 网站的模板的详细信息，请使用以下命令。

        azure group template show Microsoft.WebSiteSQLDatabase.0.1.0-preview1

    这将返回有关模板的描述性信息。

4.  选择了一个模板之后，您可以使用以下命令来下载该模板。

        azure group template download Microsoft.WebSiteSQLDatabase.0.1.0-preview1

    下载模板后，您可以对其进行自定义，以便更好地满足您的要求。例如，向模板中添加其他资源。

    > [WACOM.NOTE] 如果修改了模板，请在使用该模板创建或修改现有资源组之前，使用 `azure group template validate` 命令来验证该模板。

5.  在文本编辑器中打开模板文件。请注意靠近顶部的 **parameters** 集合。该集合包含此模板要用来创建模板所述资源的参数列表。部分参数（如 **sku**）具有默认值，而其他参数只指定了值类型（如 **siteName**）。使用模板时，您可以提供参数使之作为命令行参数的一部分，或通过指定包含参数值的文件来提供参数。无论采用哪种方式，参数都必须为 JSON 格式。

    要创建包含 Microsoft.WebSiteSQLDatabase.0.1.0-preview1 模板参数的文件，请使用以下数据，并创建一个名为 **params.json** 的文件。将以 **My** 开头的值（如 **MyWebSite**）替换为您自己的值。**siteLocation** 应指定您附近的 Azure 区域，如“华东”或“华北”。

        {
          "siteName": {
            "value": "MyWebSite"
          },
          "hostingPlanName": {
            "value": "MyHostingPlan"
          },
          "siteLocation": {
            "value": "North China"
          },
          "serverName": {
            "value": "MySQLServer"
          },
          "serverLocation": {
            "value": "North China"
          },
          "administratorLogin": {
            "value": "MySQLAdmin"
          },
          "administratorLoginPassword": {
            "value": "MySQLAdminPassword"
          },
          "databaseName": {
            "value": "MySQLDB"
          }
        }

6.  保存 **params.json** 文件后，请使用以下命令基于模板创建新的资源组。`-e` 参数指定上一步中创建的 **params.json** 文件。

        azure group create MyGroupName "MyDataCenter" -y Microsoft.WebSiteSQLDatabase.0.1.0-preview1 -d MyDeployment -e params.json

    将 **MyGroupName** 替换为您要使用的组名，将 **MyDataCenter** 替换为模板中指定的 **siteLocation** 值。

    > [WACOM.NOTE] 在部署上载后且在部署应用于组中的资源之前，此命令将返回“确定”。要检查部署的状态，请使用以下命令。
    >
    > `azure group deployment show MyGroupName MyDeployment`
    >
    > **ProvisioningState** 将显示部署的状态。
    >
    > 如果您发现配置不正确，且需要停止长时间运行的部署，请使用以下命令。
    >
    > `azure group deployment stop MyGroupName MyDeployment`
    >
    > 如果您未提供部署名称，系统将根据模板文件的名称自动创建一个。返回的名称将作为 `azure group create` 命令输出的一部分。

7.  要查看组，请使用以下命令：

        azure group show MyGroupName

    此命令将返回组中资源的相关信息。如果有多个组，可以使用 `azure group list` 命令来检索组名称的列表，然后使用 `azure group show` 来查看特定组的详细信息。

## 使用资源

尽管模板可用于声明组范围内的配置更改，但有时您只需要使用特定资源。您可以使用 `azure resource` 命令来实现此目的。

> [WACOM.NOTE] 使用除 `list` 命令以外的 `azure resource` 命令时，您必须使用 `-o` 参数指定您使用的资源的 API 版本。如果您不确定要使用的 API 版本，请查阅模板文件并查找资源的 **apiVersion** 字段。

1.  要列出组中的所有资源，请使用以下命令。

        azure resource list MyGroupName

2.  要查看组中的单个资源（例如网站），请使用以下命令。

        azure resource show MyGroupName MyWebSiteName Microsoft.Web/sites -o "2014-04-01"

    请注意 **Microsoft.Web/sites** 参数。其表示您正在请求信息的资源的类型。如果您看一下之前下载的模板文件，您会注意到此相同的值用于定义模板中所述的网站资源的类型。

    此命令将返回与网站相关的信息。例如，**hostNames** 字段应包含网站的 URL。请在您的浏览器打开此 URL 来确认网站是否正在运行。

3.  查看资源详细信息时,使用 `--json` 参数很有用，因为由于某些值为嵌套结构或集合，使用此参数可使输出更易于阅读。以下代码演示了将 show 命令的结果返回为 JSON 文档。

        azure resource show MyGroupName MyWebSite Microsoft.Web/sites -o "2014-04-01" --json

    > [WACOM.NOTE] 您可以通过使用 \> 字符将输出传输到文件，将 JSON 数据保存到文件。例如：
    >
    > `azure resource show MyGroupName MyWebSite Micrsoft.Web/sites --json > myfile.json`

4.  要删除现有资源，请使用以下命令。

        azure resource delete MyGroupName MyWebSite Microsoft.Web/sites -o "2014-04-01"

## 日志记录

要查看对组执行的操作的日志记录信息，请使用 `azure group log show` 命令。默认情况下，此命令将列出对组执行的最后一个操作。要查看所有操作，请使用可选的 `--all` 参数。对于最后一个部署，请使用 `--last-deployment`。对于特定部署，请使用 `--deployment` 并指定部署名称。以下示例将返回对组“MyGroup”执行的所有操作的日志。

    azure group log show mygroup --all

## 后续步骤

-   有关使用 Azure 跨平台命令行接口的详细信息，请参阅[安装和配置 Windows Azure 跨平台命令行接口][安装和配置 Windows Azure 跨平台命令行接口]。
-   有关通过 Windows Azure PowerShell 使用 Resource Manager 的信息，请参阅[结合使用 Windows PowerShell 和 Resource Manager 入门][结合使用 Windows PowerShell 和 Resource Manager 入门]。

  [安装和配置 Windows Azure 跨平台命令行接口]: /zh-cn/documentation/articles/xplat-cli/
  [结合使用 Windows PowerShell 和 Resource Manager 入门]: /zh-cn/documentation/articles/azure-preview-portal-using-resource-groups/
