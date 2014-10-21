<properties linkid="develop-python-service-management" urlDisplayName="Service Management" pageTitle="How to use the service management API (Python) - feature guide" metaKeywords="" description="Learn how to programmatically perform common service management tasks from Python." metaCanonical="" services="cloud-services" documentationCenter="Python" title="How to use Service Management from Python" authors="" solutions="" manager="" editor="" />

# 如何从 Python 使用服务管理

本指南说明如何以编程方式从 Python 执行常见服务管理任务。[Azure SDK for Python][Azure SDK for Python] 中的 **ServiceManagementService** 类支持以编程方式访问[管理门户][管理门户]中提供的众多与服务管理相关的功能（例如**创建、更新和删除云服务、部署、数据管理服务、虚拟机以及地缘组**）。此功能可用于构建需要以编程方式访问服务管理的应用程序。

## 目录

-   [什么是服务管理][什么是服务管理]
-   [概念][概念]
-   [如何：连接到服务管理][如何：连接到服务管理]
-   [如何：列出可用位置][如何：列出可用位置]
-   [如何：创建云服务][如何：创建云服务]
-   [如何：删除云服务][如何：删除云服务]
-   [如何：创建部署][如何：创建部署]
-   [如何：更新部署][如何：更新部署]
-   [如何：在过渡环境和生产环境之间移动部署][如何：在过渡环境和生产环境之间移动部署]
-   [如何：删除部署][如何：删除部署]
-   [如何：创建存储服务][如何：创建存储服务]
-   [如何：删除存储服务][如何：删除存储服务]
-   [如何：创建地缘组][如何：创建地缘组]
-   [如何：删除地缘组][如何：删除地缘组]
-   [如何：列出可用操作系统][如何：列出可用操作系统]
-   [如何：创建操作系统映像][如何：创建操作系统映像]
-   [如何：删除操作系统映像][如何：删除操作系统映像]
-   [如何：创建虚拟机][如何：创建虚拟机]
-   [如何：删除虚拟机][如何：删除虚拟机]
-   [后续步骤][后续步骤]

## <a name="WhatIs"> </a>什么是服务管理

利用服务管理 API，可以编程方式访问通过[管理门户][管理门户]提供的众多服务管理功能。Azure SDK for Python 允许你管理云服务、存储帐户和地缘组。

若要使用服务管理 API，你需要[创建 Azure 帐户][创建 Azure 帐户]。

## <a name="Concepts"> </a>概念

Azure SDK for Python 可包装 [Azure 服务管理 API][Azure 服务管理 API]，后者是一种 REST API。所有 API 操作都是通过 SSL 执行的，并且使用 X.509 v3 证书互相进行身份验证。可以从在 Azure 中运行的服务访问管理服务，或直接通过 Internet 从可发送 HTTPS 请求和接收 HTTPS 响应的任意应用程序访问管理服务。

## <a name="Connect"> </a>如何：连接到服务管理

若要连接到服务管理终结点，你需要 Azure 订阅 ID 和有效管理证书。你可以通过[管理门户][管理门户]获取订阅 ID。

### Windows 上的管理证书

你可以使用以下程序在你的计算机上创建自签名管理证书：`makecert.exe`. 以**管理员**身份打开 **Visual Studio 命令提示符**并且使用以下命令，用你要使用的证书名称替换 *AzureCertificate*。

    makecert -sky exchange -r -n "CN=AzureCertificate" -pe -a sha1 -len 2048 -ss My "AzureCertificate.cer"

该命令将创建`.cer` 文件，并将它安装到“个人”证书存储中。有关详细信息，请参阅[创建并上载 Azure 管理证书][创建并上载 Azure 管理证书]。

在创建证书后，需要将 `.cer` 文件上载到 Azure（通过[管理门户][管理门户]的“设置”选项卡中的“上载”操作）。

在获取订阅 ID、创建证书并且将 `.cer` 文件上载到 Azure 后，可以通过将订阅 ID 以及你的“个人”证书存储中的证书位置传递到 **ServiceManagementService**（同样，用你的证书名称替换 *AzureCertificate*），连接到 Azure 管理终结点：

    from azure import *
    from azure.servicemanagement import *

    subscription_id = '<your_subscription_id>'
    certificate_path = 'CURRENT_USER\\my\\AzureCertificate'

    sms = ServiceManagementService(subscription_id, certificate_path)

在以上示例中，`sms` 是一个 **ServiceManagementService** 对象。**ServiceManagementService** 类是用于管理 Azure 服务的主类。

### Mac/Linux 上的管理证书

你可以使用 [OpenSSL][OpenSSL] 创建管理证书。你实际上需要创建两个证书，一个用于服务器（`.cer` 文件），一个用于客户端（`.pem` 文件）。若要创建 `.pem` 文件，请执行以下命令：

    `openssl req -x509 -nodes -days 365 -newkey rsa:1024 -keyout mycert.pem -out mycert.pem`

若要创建 `.cer` 证书，请执行以下命令：

    `openssl x509 -inform pem -in mycert.pem -outform der -out mycert.cer`

有关 Azure 证书的详细信息，请参阅[在 Azure 中管理证书][在 Azure 中管理证书]。有关 OpenSSL 参数的完整说明，请参阅 [][]<http://www.openssl.org/docs/apps/openssl.html></a> 上的文档。

创建这些文件后，需要将 `.cer` 文件上载到 Azure（通过[管理门户][管理门户]的“设置”选项卡中的“上载”操作），并且需要记下`.pem` 文件的保存位置。

在获取订阅 ID、创建证书并且将 `.cer` 件上载到 Azure 后，你可以通过将订阅 ID 和 `.pem` 文件的路径传递给 **ServiceManagementService** 来连接到 Azure 管理终结点：

    from azure import *
    from azure.servicemanagement import *

    subscription_id = '<your_subscription_id>'
    certificate_path = '<path_to_.pem_certificate>'

    sms = ServiceManagementService(subscription_id, certificate_path)

在以上示例中，`sms` 是一个 **ServiceManagementService** 对象。**ServiceManagementService** 类是用于管理 Azure 服务的主类。

## <a name="ListAvailableLocations"> </a>如何：列出可用位置

若要列出可用于托管服务的位置，请使用 **list\_locations** 方法：

    from azure import *
    from azure.servicemanagement import *

    sms = ServiceManagementService(subscription_id, certificate_path)

    result = sms.list_locations()
    for location in result:
        print(location.name)

在创建云服务、存储服务或地缘组时，你将需要提供有效位置。**list\_locations** 方法将始终返回当前可用位置的最新列表。截止到本文撰写时为止，可用位置为：

-   欧洲西部
-   亚洲东南部
-   亚洲东部
-   美国中北部
-   欧洲北部
-   美国中南部
-   美国西部
-   美国东部

## <a name="CreateCloudService"> </a>如何：创建云服务

当你创建应用程序以及在 Azure 中运行该应用程序时，相关代码和配置统称为 Azure [云服务][云服务]（在早期版本的 Azure 中称为*托管服务*）。**create\_hosted\_service** 方法允许你通过提供托管服务名称（它在 Azure 中必须是唯一的）、标签（自动编码为 base64）、说明和位置来创建新的托管服务。你可以为服务指定地缘组而非位置。

    from azure import *
    from azure.servicemanagement import *

    sms = ServiceManagementService(subscription_id, certificate_path)

    name = 'myhostedservice'
    label = 'myhostedservice'
    desc = 'my hosted service'
    location = 'West US'

    # You can either set the location or an affinity_group
    sms.create_hosted_service(name, label, desc, location)

你可以使用 **list\_hosted\_services** 方法列出你的订阅的所有托管服务：

    result = sms.list_hosted_services()

    for hosted_service in result:
        print('Service name: ' + hosted_service.service_name)
        print('Management URL: ' + hosted_service.url)
        print('Affinity group: ' + hosted_service.hosted_service_properties.affinity_group)
        print('Location: ' + hosted_service.hosted_service_properties.location)
        print('')

如果你希望获得有关特定托管服务的信息，可以通过将托管服务名称传递给 **get\_hosted\_service\_properties** 方法来实现此目的：

    hosted_service = sms.get_hosted_service_properties('myhostedservice')

    print('Service name: ' + hosted_service.service_name)
    print('Management URL: ' + hosted_service.url)
    print('Affinity group: ' + hosted_service.hosted_service_properties.affinity_group)
    print('Location: ' + hosted_service.hosted_service_properties.location)
            

在创建云服务后，你可以使用 **create\_deployment** 方法将代码部署到服务。

## <a name="DeleteCloudService"> </a>如何：删除云服务

你可以通过将服务名称传递给 **delete\_hosted\_service** 方法来删除云服务：

    sms.delete_hosted_service('myhostedservice')

请注意，必须先删除服务的所有部署，然后才能删除服务。（有关详细信息，请参阅[如何：删除部署][如何：删除部署]。）

## <a name="CreateDeployment"> </a>如何：创建部署

**create\_deployment** 方法上载新的[服务包][服务包]并在过渡环境或生产环境中创建新部署。此方法的参数如下：

-   **name**：托管服务的名称。
-   **deployment\_name**：部署的名称。
-   **slot**：指示`staging` 或`production` 槽的字符串。
-   **package\_url**：部署包（.cspgk 文件）的 URL。包文件必须存储在 Azure Blob 存储帐户中，该帐户必须与包要上载到的托管服务属于同一个订阅。你可以使用 [Azure PowerShell cmdlet][Azure PowerShell cmdlet] 或使用 [cspack 命令行工具][cspack 命令行工具]创建部署包。
-   **configuration**：编码为 base64 的服务配置文件（.cscfg 文件）。
-   **label**：托管服务名称的标签（自动编码为 base64）。

以下示例将创建新部署`v1` （针对的托管服务名为 `myhostedservice`:

    from azure import *
    from azure.servicemanagement import *
    import base64

    sms = ServiceManagementService(subscription_id, certificate_path)

    name = 'myhostedservice'
    deployment_name = 'v1'
    slot = 'production'
    package_url = 'URL_for_.cspkg_file'
    configuration = base64.b64encode(open('path_to_cscfg', 'rb'))
    label = 'myhostedservice'

    result = sms.create_deployment(name, slot, deployment_name, package_url, label, configuration)

    operation_result = sms.get_operation_status(result.request_id)
    print('Operation status: ' + operation_result.status)

请注意，在上面的示例中，可以通过将 **create\_deployment** 返回的结果传递给 **get\_operation\_status** 方法来检索 **create\_deployment** 操作的状态。

你可以使用 **get\_deployment\_by\_slot** 或 **get\_deployment\_by\_name** 方法访问部署属性。下面的示例通过指定部署槽来检索部署。此示例还循环访问部署的所有实例：

    result = sms.get_deployment_by_slot('myhostedservice', 'production')

    print('Name: ' + result.name)
    print('Slot: ' + result.deployment_slot)
    print('Private ID: ' + result.private_id)
    print('Status: ' + result.status)
    print('Instances:')
    for instance in result.role_instance_list:
        print('Instance name: ' + instance.instance_name)
        print('Instance status: ' + instance.instance_status)
        print('Instance size: ' + instance.instance_size)

## <a name="UpdateDeployment"> </a>如何：更新部署

可以通过使用 **change\_deployment\_configuration** 方法或 **update\_deployment\_status** 方法来更新部署。

利用 **change\_deployment\_configuration** 方法，你可以上载新服务配置 (`.cscfg`) 文件，这将更改多项服务设置（包括部署中的实例数）中的任何设置。有关详细信息，请参阅 [Azure 服务配置架构 (.cscfg)][Azure 服务配置架构 (.cscfg)]。下面的示例演示如何上载新的服务配置文件：

    from azure import *
    from azure.servicemanagement import *
    import base64

    sms = ServiceManagementService(subscription_id, certificate_path)

    name = 'myhostedservice'
    deployment_name = 'v1'
    configuration = base64.b64encode(open('path_to_cscfg', 'rb'))

    result = sms.change_deployment_configuration(name, deployment_name, configuration)

    operation_result = sms.get_operation_status(result.request_id)
    print('Operation status: ' + operation_result.status)

请注意，在上面的示例中，可以通过将 **change\_deployment\_configuration** 返回的结果传递给 **get\_operation\_status** 方法来检索 **change\_deployment\_configuration** 操作的状态。

**update\_deployment\_status** 方法允许你将部署状态设置为 RUNNING 或 SUSPENDED。以下示例演示了如何将名为 myhostedservice 的托管服务的名为 v1 的部署的状态设置为 RUNNING：

    from azure import *
    from azure.servicemanagement import *

    sms = ServiceManagementService(subscription_id, certificate_path)

    name = 'myhostedservice'
    deployment_name = 'v1'

    result = update_deployment_status(name, deployment_name, 'Running')

## <a name="MoveDeployments"> </a>如何：在过渡环境和生产环境之间移动部署

Azure 提供两种部署环境：过渡环境和生产环境。通常，在将服务部署到生产环境之前，会将服务部署到过渡环境以进行测试。在需要将过渡环境中的服务提升到生产环境时，你可以执行此操作而无需重新部署该服务。这可通过交换部署来完成。（有关交换部署的详细信息，请参阅[部署 Azure 服务][部署 Azure 服务]。）

以下示例演示了如何使用 **swap\_deployment** 方法来交换两个部署（部署名称分别为`v1` 和 `v2`). 在该示例中，调用 **swap\_deployment** 之前，部署`v1` 位于生产槽中，部署 `v2` 位于过渡槽中。调用 **swap\_deployment** 之后，`v2` 位于生产槽中，`v1` 位于过渡槽中。

    from azure import *
    from azure.servicemanagement import *

    sms = ServiceManagementService(subscription_id, certificate_path)

    result = sms.swap_deployment('myhostedservice', 'v1', 'v2')

## <a name="DeleteDeployment"> </a>如何：删除部署

若要删除部署，请使用 **delete\_deployment** 方法。以下示例演示了如何删除名为 v1 的部署。

    from azure import *
    from azure.servicemanagement import *

    sms = ServiceManagementService(subscription_id, certificate_path)

    sms.delete_deployment('myhostedservice', 'v1')

## <a name="CreateStorageService"> </a>如何：创建存储服务

利用[存储服务][存储服务]，可以访问 Azure [Blob][Blob]、[表][表]和[队列][队列]。若要创建存储服务，你需要服务名称（3 至 24 个小写字符且在 Azure 中是唯一的）、说明、标签（最多 100 个字符，自动编码为 base64）以及位置或地缘组。下面的示例演示如何通过指定位置来创建存储服务。如果你要使用地缘组，则必须首先创建地缘组（请参阅[如何：创建地缘组][如何：创建地缘组]）并使用 **affinity\_group** 参数设置它。

    from azure import *
    from azure.servicemanagement import *

    sms = ServiceManagementService(subscription_id, certificate_path)

    name = 'mystorageaccount'
    label = 'mystorageaccount'
    location = 'West US'
    desc = 'My storage account description.'

    result = sms.create_storage_account(name, desc, label, location=location)

    operation_result = sms.get_operation_status(result.request_id)
    print('Operation status: ' + operation_result.status)

请注意，在上面的示例中，可以通过将 **create\_storage\_account** 返回的结果传递给 **get\_operation\_status** 方法来检索 **create\_storage\_account** 操作的状态。

你可以使用 **list\_storage\_accounts** 方法列出存储帐户及其属性：

    from azure import *
    from azure.servicemanagement import *

    sms = ServiceManagementService(subscription_id, certificate_path)

    result = sms.list_storage_accounts()
    for account in result:
        print('Service name: ' + account.service_name)
        print('Affinity group: ' + account.storage_service_properties.affinity_group)
        print('Location: ' + account.storage_service_properties.location)
        print('')

## <a name="DeleteStorageService"> </a>如何：删除存储服务

你可以通过将存储服务名称传递给 **delete\_storage\_account** 方法来删除存储服务。删除存储服务会删除该服务中存储的所有数据（Blob、表和队列）。

    from azure import *
    from azure.servicemanagement import *

    sms = ServiceManagementService(subscription_id, certificate_path)

    sms.delete_storage_account('mystorageaccount')

## <a name="CreateAffinityGroup"> </a>如何：创建地缘组

地缘组是 Azure 服务的逻辑分组，它告知 Azure 定位服务以优化性能。例如，你可以在“美国西部”位置创建一个地缘组，然后在该地缘组中创建[云服务][如何：创建云服务]。如果你然后在同一地缘组中创建存储服务，则 Azure 知道将该服务放在“美国西部”位置，并在数据中心使用同一地缘组中的云服务进行优化以实现最佳性能。

若要创建地缘组，你需要名称、标签（自动编码为 base64）和位置。你可以选择提供说明：

    from azure import *
    from azure.servicemanagement import *

    sms = ServiceManagementService(subscription_id, certificate_path)

    name = 'myAffinityGroup'
    label = 'myAffinityGroup'
    location = 'West US'
    desc = 'my affinity group'

    sms.create_affinity_group(name, label, location, desc)

在创建地缘组后，你可以在[创建存储服务][如何：创建存储服务]时指定该组（而非位置）。

你可以通过调用 **list\_affinity\_groups** 方法来列出地缘组并检查其属性：

    result = sms.list_affinity_groups()

    for group in result:
        print('Name: ' + group.name)
        print('Description: ' + group.description)
        print('Location: ' + group.location)
        print('')

## <a name="DeleteAffinityGroup"> </a>如何：删除地缘组

你可以通过将组名传递给 **delete\_affinity\_group** 方法来删除地缘组。请注意，地缘组必须与任何服务解除关联（或者必须删除使用地缘组的服务），然后才能删除该地缘组。

    from azure import *
    from azure.servicemanagement import *

    sms = ServiceManagementService(subscription_id, certificate_path)

    sms.delete_affinity_group('myAffinityGroup')

## <a name="ListOperatingSystems"> </a>如何：列出可用操作系统

若要列出可用于托管服务的操作系统，请使用 **list\_operating\_systems** 方法：

    from azure import *
    from azure.servicemanagement import *

    sms = ServiceManagementService(subscription_id, certificate_path)

    result = sms.list_operating_systems()

    for os in result:
        print('OS: ' + os.label)
        print('Family: ' + os.family_label)
        print('Active: ' + str(os.is_active))

或者，你可以使用 **list\_operating\_system\_families** 方法，按系列对操作系统进行分组：

    result = sms.list_operating_system_families()

    for family in result:
        print('Family: ' + family.label)
        for os in family.operating_systems:
            if os.is_active:
                print('OS: ' + os.label)
                print('Version: ' + os.version)
        print('')

## <a name="CreateVMImage"> </a>如何：创建操作系统映像

若要将操作系统映像添加到映像存储库中，请使用 **add\_os\_image** 方法：

    from azure import *
    from azure.servicemanagement import *

    sms = ServiceManagementService(subscription_id, certificate_path)

    name = 'mycentos'
    label = 'mycentos'
    os = 'Linux' # Linux or Windows
    media_link = 'url_to_storage_blob_for_source_image_vhd'

    result = sms.add_os_image(label, media_link, name, os)

    operation_result = sms.get_operation_status(result.request_id)
    print('Operation status: ' + operation_result.status)

若要列出可用的操作系统映像，请使用 **list\_os\_images** 方法。这包括所有平台映像和用户映像：

    result = sms.list_os_images()

    for image in result:
        print('Name: ' + image.name)
        print('Label: ' + image.label)
        print('OS: ' + image.os)
        print('Category: ' + image.category)
        print('Description: ' + image.description)
        print('Location: ' + image.location)
        print('Affinity group: ' + image.affinity_group)
        print('Media link: ' + image.media_link)
        print('')

## <a name="DeleteVMImage"> </a>如何：删除操作系统映像

若要删除用户映像，请使用 **delete\_os\_image** 方法：

    from azure import *
    from azure.servicemanagement import *

    sms = ServiceManagementService(subscription_id, certificate_path)

    result = sms.delete_os_image('mycentos')

    operation_result = sms.get_operation_status(result.request_id)
    print('Operation status: ' + operation_result.status)

## <a name="CreateVM"> </a>如何：创建虚拟机

若要创建虚拟机，你首先需要创建[云服务][如何：创建云服务]。然后使用 **create\_virtual\_machine\_deployment** 方法来创建虚拟机部署：

    from azure import *
    from azure.servicemanagement import *

    sms = ServiceManagementService(subscription_id, certificate_path)

    name = 'myvm'
    location = 'West US'

    # You can either set the location or an affinity_group
    sms.create_hosted_service(service_name=name,
        label=name,
        location=location)

    # Name of an os image as returned by list_os_images
    image_name = 'OpenLogic__OpenLogic-CentOS-62-20120531-zh-cn-30GB.vhd'

    # Destination storage account container/blob where the VM disk
    # will be created
    media_link = 'url_to_target_storage_blob_for_vm_hd'

    # Linux VM configuration, you can use WindowsConfigurationSet
    # for a Windows VM instead
    linux_config = LinuxConfigurationSet('myhostname', 'myuser', 'mypassword', True)

    os_hd = OSVirtualHardDisk(image_name, media_link)

    sms.create_virtual_machine_deployment(service_name=name,
        deployment_name=name,
        deployment_slot='production',
        label=name,
        role_name=name,
        system_config=linux_config,
        os_virtual_hard_disk=os_hd,
        role_size='Small')

## <a name="DeleteVM"> </a>如何：删除虚拟机

若要删除虚拟机，请首先使用 **delete\_deployment** 方法来删除部署：

    from azure import *
    from azure.servicemanagement import *

    sms = ServiceManagementService(subscription_id, certificate_path)

    sms.delete_deployment(service_name='myvm',
        deployment_name='myvm')

然后可以使用 **delete\_hosted\_service** 方法来删除云服务：

    sms.delete_hosted_service(service_name='myvm')

## <a name="NextSteps"> </a> 后续步骤

现在，你已了解有关服务管理的基础知识，单击下面的链接可执行更复杂的任务。

-   查看 MSDN 参考：[云服务][服务包]
-   查看 MSDN 参考：[虚拟机][虚拟机]

  [Azure SDK for Python]: https://www.windowsazure.com/zh-cn/develop/python/common-tasks/install-python/
  [管理门户]: https://manage.windowsazure.cn/
  [什么是服务管理]: #WhatIs
  [概念]: #Concepts
  [如何：连接到服务管理]: #Connect
  [如何：列出可用位置]: #ListAvailableLocations
  [如何：创建云服务]: #CreateCloudService
  [如何：删除云服务]: #DeleteCloudService
  [如何：创建部署]: #CreateDeployment
  [如何：更新部署]: #UpdateDeployment
  [如何：在过渡环境和生产环境之间移动部署]: #MoveDeployments
  [如何：删除部署]: #DeleteDeployment
  [如何：创建存储服务]: #CreateStorageService
  [如何：删除存储服务]: #DeleteStorageService
  [如何：创建地缘组]: #CreateAffinityGroup
  [如何：删除地缘组]: #DeleteAffinityGroup
  [如何：列出可用操作系统]: #ListOperatingSystems
  [如何：创建操作系统映像]: #CreateVMImage
  [如何：删除操作系统映像]: #DeleteVMImage
  [如何：创建虚拟机]: #CreateVM
  [如何：删除虚拟机]: #DeleteVM
  [后续步骤]: #NextSteps
  [创建 Azure 帐户]: http://www.windowsazure.com/zh-cn/pricing/free-trial/
  [Azure 服务管理 API]: http://msdn.microsoft.com/zh-cn/library/windowsazure/ee460799.aspx
  [创建并上载 Azure 管理证书]: http://msdn.microsoft.com/zh-cn/library/windowsazure/gg551722.aspx
  [OpenSSL]: http://www.openssl.org/
  [在 Azure 中管理证书]: http://msdn.microsoft.com/zh-cn/library/windowsazure/gg981929.aspx
  []: http://www.openssl.org/docs/apps/openssl.html
  [云服务]: http://windowsazure.com/zh-cn/documentation/articles/cloud-services-what-is
  [服务包]: http://msdn.microsoft.com/zh-cn/library/windowsazure/jj155995.aspx
  [Azure PowerShell cmdlet]: https://www.windowsazure.com/zh-cn/develop/php/how-to-guides/powershell-cmdlets/
  [cspack 命令行工具]: http://msdn.microsoft.com/zh-cn/library/windowsazure/gg432988.aspx
  [Azure 服务配置架构 (.cscfg)]: http://msdn.microsoft.com/zh-cn/library/windowsazure/ee758710.aspx
  [部署 Azure 服务]: http://msdn.microsoft.com/zh-cn/library/windowsazure/gg433027.aspx
  [存储服务]: https://www.windowsazure.com/zh-cn/manage/services/storage/what-is-a-storage-account/
  [Blob]: https://www.windowsazure.com/zh-cn/develop/python/how-to-guides/blob-service/
  [表]: https://www.windowsazure.com/zh-cn/develop/python/how-to-guides/table-service/
  [队列]: https://www.windowsazure.com/zh-cn/develop/python/how-to-guides/queue-service/
  [虚拟机]: http://msdn.microsoft.com/zh-cn/library/windowsazure/jj156003.aspx
