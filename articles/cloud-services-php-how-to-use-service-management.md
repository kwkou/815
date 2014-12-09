<properties linkid="develop-php-how-to-guides-service-management" urlDisplayName="Service Management" pageTitle="如何使用 Azure 服务管理 API (PHP)" metaKeywords="" description="了解如何使用 Azure PHP 服务管理 API 来管理云服务和其他 Azure 应用程序。" metaCanonical="" services="" documentationCenter="PHP" title="如何通过 PHP 使用服务管理" authors="robmcm" solutions="" manager="wpickett" editor="mollybos" videoId="" scriptId="" />

# 如何通过 PHP 使用服务管理

本指南将演示如何通过 PHP 以编程方式执行常见服务管理任务。[Azure SDK for PHP][Azure SDK for PHP] 中的 [ServiceManagementRestProxy][ServiceManagementRestProxy] 类支持以编程方式访问[管理门户][管理门户]中提供的很多与服务管理相关的功能（如**创建、更新和删除云服务、部署、存储服务和地缘组**）。此功能可用于构建需要以编程方式访问服务管理的应用程序。

## 目录

-   [什么是服务管理][什么是服务管理]
-   [概念][概念]
-   [创建 PHP 应用程序][创建 PHP 应用程序]
-   [获取 Azure 客户端库][获取 Azure 客户端库]
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

## <span id="WhatIs"></span></a>什么是服务管理

利用服务管理 API，可以编程方式访问通过[管理门户][管理门户]提供的众多服务管理功能。Azure SDK for PHP 允许你管理云服务、存储帐户和地缘组。

若要使用服务管理 API，你需要[创建 Azure 帐户][创建 Azure 帐户]。

## <span id="Concepts"></span></a>概念

Azure SDK for PHP 可包装 [Azure 服务管理 API][Azure 服务管理 API]，后者是一种 REST API。所有 API 操作都是通过 SSL 执行的，并且使用 X.509 v3 证书互相进行身份验证。可以从在 Azure 中运行的服务访问管理服务，或直接通过 Internet 从可发送 HTTPS 请求和接收 HTTPS 响应的任意应用程序访问管理服务。

## <span id="CreateApplication"></span></a>创建 PHP 应用程序

创建使用 Azure 服务管理的 PHP 应用程序的唯一要求是从代码中引用 Azure SDK for PHP 中的类。你可以使用任何开发工具（包括“记事本”）创建应用程序。

在本指南中，你将使用服务功能，这些功能可在 PHP 应用程序中本地调用，或通过在 Azure 的 Web 角色、辅助角色或网站中运行的代码调用。

## <span id="GetClientLibraries"></span></a>获取 Azure 客户端库

[WACOM.INCLUDE [get-client-libraries](../includes/get-client-libraries.md)]

## <span id="Connect"></span></a>如何：连接到服务管理

若要连接到服务管理终结点，你需要 Azure 订阅 ID 和有效管理证书的路径。你可以通过[管理门户][管理门户]获取订阅 ID，并且可以采用多种方式创建管理证书。在本指南中将使用 [OpenSSL][OpenSSL]，你可以[为 Windows 下载][为 Windows 下载]并在控制台中运行它。

你实际上需要创建两个证书，一个用于服务器（`.cer` 文件），一个用于客户端（`.pem` 文件）。若要创建 `.pem` 文件，请执行以下代码：

    `openssl req -x509 -nodes -days 365 -newkey rsa:1024 -keyout mycert.pem -out mycert.pem`

若要创建 `.cer` 证书，请执行以下代码：

    `openssl x509 -inform pem -in mycert.pem -outform der -out mycert.cer`

有关 Azure 证书的详细信息，请参阅[Azure 中的证书概述][Azure 中的证书概述]。有关 OpenSSL 参数的完整说明，请参阅 <http://www.openssl.org/docs/apps/openssl.html> 上的文档。

如果已使用 [Azure 命令行工具][Azure 命令行工具]下载和导入发布设置文件，则可以使用这些工具创建的 `.pem` 文件（而不是你自己的 .pem 文件）。这些工具为你创建了一个 `.cer` 并将其上载到 Azure，然后将相应的 `.pem` 文件放在计算机上的 `.azure` 目录中（在你的用户目录中）。

创建这些文件之后，你将需要通过[管理门户][管理门户]将 `.cer` 文件上载到 Azure，并且你将需要记下 `.pem` 文件的保存位置。

获取订阅 ID，创建证书并将 `.cer` 文件上载到 Azure 之后，你可以通过创建连接字符串并将其传递到对 **ServicesBuilder** 类调用的 **createServiceManagementService** 方法来连接到 Azure 管理终结点：

    require_once 'vendor\autoload.php';

    use WindowsAzure\Common\ServicesBuilder;

    $conn_string = "SubscriptionID=<your_subscription_id>;CertificatePath=<path_to_.pem_certificate>";

    $serviceManagementRestProxy = ServicesBuilder::getInstance()->createServiceManagementService($conn_string);

在上面的示例中，`$serviceManagementRestProxy` 是一个 [ServiceManagementRestProxy][ServiceManagementRestProxy] 对象。**ServiceManagementRestProxy** 类是用于管理 Azure 服务的主类。

## <span id="ListAvailableLocations"></span></a>如何：列出可用位置

若要列出可用于托管服务的位置，请使用 **ServiceManagementRestProxy-\>listLocations** 方法：

    require_once 'vendor\autoload.php';

    use WindowsAzure\Common\ServicesBuilder;
    use WindowsAzure\Common\ServiceException;

    try{
        $serviceManagementRestProxy = ServicesBuilder::getInstance()->createServiceManagementService($conn_string);

        $result = $serviceManagementRestProxy->listLocations();

        $locations = $result->getLocations();

        foreach($locations as $location){
              echo $location->getName()."<br />";
        }
    }
    catch(ServiceException $e){
        // Handle exception based on error codes and messages.
        // Error codes and messages are here: 
        // http://msdn.microsoft.com/zh-cn/library/windowsazure/ee460801
        $code = $e->getCode();
        $error_message = $e->getMessage();
        echo $code.": ".$error_message."<br />";
    }

在创建云服务、存储服务或地缘组时，你将需要提供有效位置。**listLocations** 方法将始终返回当前可用位置的最新列表。截止到本文撰写时为止，可用位置为：

-   美国任意地区
-   欧洲任意地区
-   欧洲西部
-   亚洲任意地区
-   亚洲东南部
-   亚洲东部
-   美国中北部
-   欧洲北部
-   美国中南部
-   美国西部
-   美国东部

> [WACOM.NOTE]
> 在下面的代码示例中，位置将作为字符串传递到方法。但是，你还可以使用 `WindowsAzure\ServiceManagement\Models\Locations` 类将位置作为枚举传递。例如，你可以将 `Locations::WEST_US` 传递到接受位置的方法，而不是传递“美国西部”。

## <span id="CreateCloudService"></span></a>如何：创建云服务

当你创建应用程序以及在 Azure 中运行该应用程序时，相关代码和配置统称为 Azure [云服务][云服务]（在早期版本的 Azure 中称为*托管服务*）。利用 **createHostedServices** 方法，你可以通过提供托管服务名称（在 Azure 中必须是唯一的）、标签（base 64 编码托管服务名称）和 **CreateServiceOptions** 对象来创建新的托管服务。利用 [CreateServiceOptions][CreateServiceOptions] 对象，你可以设置位置*或* 服务的地缘组。

    require_once 'vendor\autoload.php';

    use WindowsAzure\Common\ServicesBuilder;
    use WindowsAzure\ServiceManagement\Models\CreateServiceOptions;
    use WindowsAzure\Common\ServiceException;

    try{
        // Create REST proxy.
        $serviceManagementRestProxy = ServicesBuilder::getInstance()->createServiceManagementService($conn_string);
        
        $name = "myhostedservice";
        $label = base64_encode($name);
        $options = new CreateServiceOptions();
        $options->setLocation('West US');
        // Instead of setLocation, you can use setAffinityGroup
        // to set an affinity group.

        $result = $serviceManagementRestProxy->createHostedService($name, $label, $options);
    }
    catch(ServiceException $e){
        // Handle exception based on error codes and messages.
        // Error codes and messages are here: 
        // http://msdn.microsoft.com/zh-cn/library/windowsazure/ee460801
        $code = $e->getCode();
        $error_message = $e->getMessage();
        echo $code.": ".$error_message."<br />";
    }

你可以使用 **listHostedServices** 方法列出你的订阅的所有托管方法，该方法将返回 [ListHostedServicesResult][ListHostedServicesResult] 对象。调用 **getHostedServices** 方法后，你就可以循环访问一系列 [HostedServices] 对象和检索服务属性：

    $listHostedServicesResult = $serviceManagementRestProxy->listHostedServices();

    $hosted_services = $listHostedServicesResult->getHostedServices();

    foreach($hosted_services as $hosted_service){
        echo "Service name: ".$hosted_service->getName()."<br />";
        echo "Management URL: ".$hosted_service->getUrl()."<br />";
        echo "Affinity group: ".$hosted_service->getAffinityGroup()."<br />";
        echo "Location: ".$hosted_service->getLocation()."<br />";
        echo "------<br />";
        }

如果你希望获得有关特定托管服务的信息，可以通过将托管服务名称传递给 **getHostedServiceProperties** 方法来实现此目的：

    $getHostedServicePropertiesResult = $serviceManagementRestProxy->getHostedServiceProperties("myhostedservice");
        
    $hosted_service = $getHostedServicePropertiesResult->getHostedService();
        
    echo "Service name: ".$hosted_service->getName()."<br />";
    echo "Management URL: ".$hosted_service->getUrl()."<br />";
    echo "Affinity group: ".$hosted_service->getAffinityGroup()."<br />";
    echo "Location: ".$hosted_service->getLocation()."<br />";

在创建云服务后，你可以使用 [createDeployment][如何：创建部署] 方法将代码部署到服务。

## <span id="DeleteCloudService"></span></a>如何：删除云服务

你可以通过将服务名称传递到 **deleteHostedService** 方法来删除云服务：

    $serviceManagementRestProxy->deleteHostedService("myhostedservice");

请注意，必须先删除服务的所有部署，然后才能删除服务。（有关详细信息，请参阅[如何：删除部署][如何：删除部署]。）

## <span id="CreateDeployment"></span></a>如何：创建部署

**createDeployment** 方法上载新的[服务包][服务包]并在过渡环境或生产环境中创建新部署。此方法的参数如下：

-   **$name**：托管服务的名称。
-   **$deploymentName**：部署的名称。
-   **$slot**：用于指示过渡槽或生产槽的枚举。
-   **$packageUrl**：部署包（.cspgk 文件）的 URL。包文件必须存储在 Azure Blob 存储帐户中，该帐户必须与包要上载到的托管服务属于同一个订阅。你可以使用 [Azure PowerShell cmdlet][Azure PowerShell cmdlet] 或使用 [cspack 命令行工具][cspack 命令行工具]创建部署包。
-   **$configuration**：服务配置文件（.cscfg 文件）。
-   **$label**：base 64 编码托管服务名称。

下面的示例将在名为 `myhostedservice` 的托管服务的生产槽中创建新部署：

    require_once 'vendor\autoload.php';

    use WindowsAzure\Common\ServicesBuilder;
    use WindowsAzure\ServiceManagement\Models\DeploymentSlot;
    use WindowsAzure\Common\ServiceException;

    try{
        // Create REST proxy.
        $serviceManagementRestProxy = ServicesBuilder::getInstance()->createServiceManagementService($conn_string);
        
        $name = "myhostedservice";
        $deploymentName = "v1";
        $slot = DeploymentSlot::PRODUCTION;
        $packageUrl = "URL_for_.cspkg_file";
        $configuration = base64_encode(file_get_contents('path_to_.cscfg_file'));
        $label = base64_encode($name);

        $result = $serviceManagementRestProxy->createDeployment($name,
                                                         $deploymentName,
                                                         $slot,
                                                         $packageUrl,
                                                         $configuration,
                                                         $label);
        
        $status = $serviceManagementRestProxy->getOperationStatus($result);
        echo "Operation status: ".$status->getStatus()."<br />";
    }
    catch(ServiceException $e){
        // Handle exception based on error codes and messages.
        // Error codes and messages are here: 
        // http://msdn.microsoft.com/zh-cn/library/windowsazure/ee460801
        $code = $e->getCode();
        $error_message = $e->getMessage();
        echo $code.": ".$error_message."<br />";
    }

请注意，在上面的示例中，可以通过将 **createDeployment** 返回的结果传递到 **getOperationStatus** 方法来检索 **createDeployment** 操作的状态。

你可以使用 **getDeployment** 方法访问部署属性。下面的示例通过指定 [GetDeploymentOptions][ListHostedServicesResult] 对象中的部署槽来检索部署，但你可以指定部署名称。此示例还循环访问部署的所有实例：

    $options = new GetDeploymentOptions();
    $options->setSlot(DeploymentSlot::PRODUCTION);
        
    $getDeploymentResult = $serviceManagementRestProxy->getDeployment("myhostedservice", $options);
    $deployment = $getDeploymentResult->getDeployment();

    echo "Name: ".$deployment->getName()."<br />";
    echo "Slot: ".$deployment->getSlot()."<br />";
    echo "Private ID: ".$deployment->getPrivateId()."<br />";
    echo "Status: ".$deployment->getStatus()."<br />";
    echo "Instances: <br />";
    foreach($deployment->getroleInstanceList() as $roleInstance){
        echo "Instance name: ".$roleInstance->getInstanceName()."<br />";
        echo "Instance status: ".$roleInstance->getInstanceStatus()."<br />";
        echo "Instance size: ".$roleInstance->getInstanceSize()."<br />";
    }
    echo "------<br />";

## <span id="UpdateDeployment"></span></a>如何：更新部署

可以使用 **changeDeploymentConfiguration** 方法或 **updateDeploymentStatus** 方法来更新部署。

利用 **changeDeploymentConfiguration** 方法，你可以上载新服务配置 (`.cscfg`) 文件，这将更改多项服务设置（包括部署中的实例数）中的任何设置。有关详细信息，请参阅 [Azure 服务配置架构 (.cscfg)][Azure 服务配置架构 (.cscfg)]。下面的示例演示如何上载新的服务配置文件：

    require_once 'vendor\autoload.php';

    use WindowsAzure\Common\ServicesBuilder;
    use WindowsAzure\ServiceManagement\Models\ChangeDeploymentConfigurationOptions;
    use WindowsAzure\ServiceManagement\Models\DeploymentSlot;
    use WindowsAzure\Common\ServiceException;

    try{
        // Create REST proxy.
        $serviceManagementRestProxy = ServicesBuilder::getInstance()->createServiceManagementService($conn_string);
        
        $name = "myhostedservice";
        $configuration = base64_encode(file_get_contents('path to .cscfg file'));
        $options = new ChangeDeploymentConfigurationOptions();
        $options->setSlot(DeploymentSlot::PRODUCTION);

        $result = $serviceManagementRestProxy->changeDeploymentConfiguration($name, $configuration, $options);
        
        $status = $serviceManagementRestProxy->getOperationStatus($result);
        echo "Operation status: ".$status->getStatus()."<br />";
    }
    catch(ServiceException $e){
        // Handle exception based on error codes and messages.
        // Error codes and messages are here: 
        // http://msdn.microsoft.com/zh-cn/library/windowsazure/ee460801
        $code = $e->getCode();
        $error_message = $e->getMessage();
        echo $code.": ".$error_message."<br />";
    }

请注意，在上面的示例中，可以通过将 **changeDeploymentConfiguration** 返回的结果传递到 **getOperationStatus** 方法来检索 **changeDeploymentConfiguration** 操作的状态。

**updateDeploymentStatus** 方法允许你将部署状态设置为 RUNNING 或 SUSPENDED。下面的示例演示如何将名为 `myhostedservice` 的托管服务的生产槽中的部署的状态设置为“正在运行”：

    require_once 'vendor\autoload.php';

    use WindowsAzure\Common\ServicesBuilder;
    use WindowsAzure\ServiceManagement\Models\DeploymentStatus;
    use WindowsAzure\ServiceManagement\Models\DeploymentSlot;
    use WindowsAzure\ServiceManagement\Models\GetDeploymentOptions;
    use WindowsAzure\Common\ServiceException;

    try{
        // Create REST proxy.
        $serviceManagementRestProxy = ServicesBuilder::getInstance()->createServiceManagementService($conn_string);
        
        $options = new GetDeploymentOptions();
        $options->setSlot(DeploymentSlot::PRODUCTION);
        
        $result = $serviceManagementRestProxy->updateDeploymentStatus("myhostedservice", DeploymentStatus::RUNNING, $options);
    }
    catch(ServiceException $e){
        // Handle exception based on error codes and messages.
        // Error codes and messages are here: 
        // http://msdn.microsoft.com/zh-cn/library/windowsazure/ee460801
        $code = $e->getCode();
        $error_message = $e->getMessage();
        echo $code.": ".$error_message."<br />";
    }

## <span id="MoveDeployments"></span></a>如何：在过渡环境和生产环境之间移动部署

Azure 提供两种部署环境：过渡环境和生产环境。通常，在将服务部署到生产环境之前，会将服务部署到过渡环境以进行测试。在需要将过渡环境中的服务提升到生产环境时，你可以执行此操作而无需重新部署该服务。这可通过交换部署来完成。（有关交换部署的详细信息，请参阅[在 Azure 中管理部署概述][在 Azure 中管理部署概述]。）

下面的示例演示如何使用 **swapDeployment** 方法来交换两个部署（部署名称分别为 `v1` 和 `v2`）。在该示例中，在调用 **swapDeployment** 之前，部署 `v1` 位于生产槽中，部署 `v2` 位于过渡槽中。在调用 **swapDeployment** 之后，`v2` 位于生产槽中，`v1` 位于过渡槽中。

    require_once 'vendor\autoload.php'; 

    use WindowsAzure\Common\ServicesBuilder;
    use WindowsAzure\Common\ServiceException;

    try{
        // Create REST proxy.
        $serviceManagementRestProxy = ServicesBuilder::getInstance()->createServiceManagementService($conn_string);
        
        $result = $serviceManagementRestProxy->swapDeployment("myhostedservice", "v2", "v1");
    }
    catch(ServiceException $e){
        // Handle exception based on error codes and messages.
        // Error codes and messages are here: 
        // http://msdn.microsoft.com/zh-cn/library/windowsazure/ee460801
        $code = $e->getCode();
        $error_message = $e->getMessage();
        echo $code.": ".$error_message."<br />";
    }

## <span id="DeleteDeployment"></span></a>如何：删除部署

若要删除部署，请使用 **deleteDeployment** 方法。下面的示例演示如何通过对 [GetDeploymentOptions][ListHostedServicesResult] 对象使用 **setSlot** 方法，然后将过渡环境中的部署传递到 **deleteDeployment** 来删除该部署。你可以对 [GetDepolymentOptions] 类使用 **setName** 方法来按部署名称指定部署，而不是按槽指定部署。

    require_once 'vendor\autoload.php';

    use WindowsAzure\Common\ServicesBuilder;
    use WindowsAzure\ServiceManagement\Models\GetDeploymentOptions;
    use WindowsAzure\ServiceManagement\Models\DeploymentSlot;
    use WindowsAzure\Common\ServiceException;

    try{
        // Create REST proxy.
        $serviceManagementRestProxy = ServicesBuilder::getInstance()->createServiceManagementService($conn_string);
        
        $options = new GetDeploymentOptions();
        $options->setSlot(DeploymentSlot::STAGING);
        
        $result = $serviceManagementRestProxy->deleteDeployment("myhostedservice", $options);
    }
    catch(ServiceException $e){
        // Handle exception based on error codes and messages.
        // Error codes and messages are here: 
        // http://msdn.microsoft.com/zh-cn/library/windowsazure/ee460801
        $code = $e->getCode();
        $error_message = $e->getMessage();
        echo $code.": ".$error_message."<br />";
    }

## <span id="CreateStorageService"></span></a>如何：创建存储服务

利用[存储服务][存储服务]，可以访问 Azure [Blob][Blob]、[表][表]和[队列][队列]。若要创建存储服务，你需要服务的名称（长度为 3 - 24 个小写字符且在 Azure 中是唯一的）、标签（服务的 base-64 编码名称，最多 100 个字符）以及位置或地缘组。提供服务的描述是可选的。位置、地缘组和描述在传递到 **createStorageService** 方法的 [CreateServiceOptions][CreateServiceOptions] 对象中设置。下面的示例演示如何通过指定位置来创建存储服务。如果你要使用地缘组，则必须首先创建地缘组（请参阅[如何：创建地缘组][如何：创建地缘组]），并使用 **CreateServiceOptions-\>setAffinityGroup** 方法对其进行设置。

    require_once 'vendor\autoload.php';
     
    use WindowsAzure\Common\ServicesBuilder;
    use WindowsAzure\ServiceManagement\Models\CreateServiceOptions;
    use WindowsAzure\Common\ServiceException;
     
     
    try{
        // Create REST proxy.
        $serviceManagementRestProxy = ServicesBuilder::getInstance()->createServiceManagementService($conn_string);
        
        $name = "mystorageaccount";
        $label = base64_encode($name);
        $options = new CreateServiceOptions();
        $options->setLocation('West US');
        $options->setDescription("My storage account description.");

        $result = $serviceManagementRestProxy->createStorageService($name, $label, $options);

        $status = $serviceManagementRestProxy->getOperationStatus($result);
        echo "Operation status: ".$status->getStatus()."<br />";
    }
    catch(ServiceException $e){
        // Handle exception based on error codes and messages.
        // Error codes and messages are here: 
        // http://msdn.microsoft.com/zh-cn/library/windowsazure/ee460801
        $code = $e->getCode();
        $error_message = $e->getMessage();
        echo $code.": ".$error_message."<br />";
    }

请注意，在上面的示例中，可以通过将 **createStorageService** 返回的结果传递到 **getOperationStatus** 方法来检索 **createStorageService** 操作的状态。

你可以使用 **listStorageServices** 方法来列出存储帐户及其属性：

    // Create REST proxy.
    $serviceManagementRestProxy = ServicesBuilder::getInstance()->createServiceManagementService($conn_string);
    $getStorageServicesResult = $serviceManagementRestProxy->listStorageServices();
    $storageServices = $getStorageServicesResult->getStorageServices();

    foreach($storageServices as $storageService){
        echo "Service name: ".$storageService->getName()."<br />";
        echo "Service URL: ".$storageService->getUrl()."<br />";
        echo "Affinity group: ".$storageService->getAffinityGroup()."<br />";
        echo "Location: ".$storageService->getLocation()."<br />";
        echo "------<br />";
    }

## <span id="DeleteStorageService"></span></a>如何：删除存储服务

你可以通过将存储服务名称传递到 **deleteStorageService** 方法来删除存储服务。删除存储服务会删除该服务中存储的所有数据（Blob、表和队列）。

    require_once 'vendor\autoload.php';

    use WindowsAzure\Common\ServicesBuilder;
    use WindowsAzure\Common\ServiceException;

    try{
        // Create REST proxy.
        $serviceManagementRestProxy = ServicesBuilder::getInstance()->createServiceManagementService($conn_string);
        
        $serviceManagementRestProxy->deleteStorageService("mystorageservice");
    }
    catch(ServiceException $e){
        // Handle exception based on error codes and messages.
        // Error codes and messages are here: 
        // http://msdn.microsoft.com/zh-cn/library/windowsazure/ee460801
        $code = $e->getCode();
        $error_message = $e->getMessage();
        echo $code.": ".$error_message."<br />";
    }

## <span id="CreateAffinityGroup"></span></a>如何：创建地缘组

地缘组是 Azure 服务的逻辑分组，它告知 Azure 定位服务以优化性能。例如，你可以在“美国西部”位置创建一个地缘组，然后在该地缘组中创建[云服务][如何：创建云服务]。如果你然后在同一地缘组中创建存储服务，则 Azure 知道将该服务放在“美国西部”位置，并在数据中心使用同一地缘组中的云服务进行优化以实现最佳性能。

若要创建地缘组，你需要名称、标签（base 64 编码名称）和位置。你可以选择提供说明：

    require_once 'vendor\autoload.php';

    use WindowsAzure\Common\ServicesBuilder;
    use WindowsAzure\ServiceManagement\Models\CreateAffinityGroupOptions;
    use WindowsAzure\Common\ServiceException;
     
    try{
        // Create REST proxy.
        $serviceManagementRestProxy = ServicesBuilder::getInstance()->createServiceManagementService($conn_string);
        
        $name = "myAffinityGroup";
        $label = base64_encode($name);
        $location = "West US";

        $options = new CreateAffinityGroupOptions();
        $options->setDescription = "My affinity group description.";
        
        $serviceManagementRestProxy->createAffinityGroup($name, $label, $location, $options);
    }
    catch(ServiceException $e){
        // Handle exception based on error codes and messages.
        // Error codes and messages are here: 
        // http://msdn.microsoft.com/zh-cn/library/windowsazure/ee460801
        $code = $e->getCode();
        $error_message = $e->getMessage();
        echo $code.": ".$error_message."<br />";
    }

在创建地缘组后，你可以在[创建存储服务][如何：创建存储服务]时指定该组（而非位置）。

你可以通过调用 **listAffinityGroups** 方法，然后对 [AffinityGroup][AffinityGroup] 类调用适当的方法来列出地缘组并检查其属性：

    $result = $serviceManagementRestProxy->listAffinityGroups();

    $groups = $result->getAffinityGroups();

    foreach($groups as $group){
        echo "Name: ".$group->getName()."<br />";
        echo "Description: ".$group->getDescription()."<br />";
        echo "Location: ".$group->getLocation()."<br />";
        echo "------<br />";
    }

## <span id="DeleteAffinityGroup"></span></a>如何：删除地缘组

你可以通过将组名传递到 **deleteAffinityGroup** 方法来删除地缘组。请注意，地缘组必须与任何服务解除关联（或者必须删除使用地缘组的服务），然后才能删除该地缘组。

    require_once 'vendor\autoload.php';

    use WindowsAzure\Common\ServicesBuilder;
    use WindowsAzure\Common\ServiceException;

    try{
        // Create REST proxy.
        $serviceManagementRestProxy = ServicesBuilder::getInstance()->createServiceManagementService($conn_string);
        
        // An affinity group must be disassociated from all services 
        // before it can be deleted.
        $serviceManagementRestProxy->deleteAffinityGroup("myAffinityGroup");
    }
    catch(ServiceException $e){
        // Handle exception based on error codes and messages.
        // Error codes and messages are here: 
        // http://msdn.microsoft.com/zh-cn/library/windowsazure/ee460801
        $code = $e->getCode();
        $error_message = $e->getMessage();
        echo $code.": ".$error_message."<br />";
    }

  [Azure SDK for PHP]: ../php-download-sdk/
  [ServiceManagementRestProxy]: https://github.com/WindowsAzure/azure-sdk-for-php/blob/master/WindowsAzure/ServiceManagement/ServiceManagementRestProxy.php
  [管理门户]: https://manage.windowsazure.cn/
  [什么是服务管理]: #WhatIs
  [概念]: #Concepts
  [创建 PHP 应用程序]: #CreateApplication
  [获取 Azure 客户端库]: #GetClientLibraries
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
  [创建 Azure 帐户]: /zh-cn/pricing/free-trial/
  [Azure 服务管理 API]: http://msdn.microsoft.com/zh-cn/library/windowsazure/ee460799.aspx
  [OpenSSL]: http://www.openssl.org/
  [为 Windows 下载]: http://www.openssl.org/related/binaries.html
  [Azure 中的证书概述]: http://msdn.microsoft.com/zh-cn/library/windowsazure/gg981935.aspx
  [Azure 命令行工具]: ../command-line-tools/
  [云服务]: ../cloud-services-what-is/
  [CreateServiceOptions]: https://github.com/WindowsAzure/azure-sdk-for-php/blob/master/WindowsAzure/ServiceManagement/Models/CreateServiceOptions.php
  [ListHostedServicesResult]: https://github.com/WindowsAzure/azure-sdk-for-php/blob/master/WindowsAzure/ServiceManagement/Models/GetDeploymentOptions.php
  [服务包]: http://msdn.microsoft.com/zh-cn/library/windowsazure/gg433093
  [Azure PowerShell cmdlet]: ../install-configure-powershell/
  [cspack 命令行工具]: http://msdn.microsoft.com/zh-cn/library/windowsazure/gg432988.aspx
  [Azure 服务配置架构 (.cscfg)]: http://msdn.microsoft.com/zh-cn/library/windowsazure/ee758710.aspx
  [在 Azure 中管理部署概述]: http://msdn.microsoft.com/zh-cn/library/windowsazure/hh386336.aspx
  [存储服务]: ../storage-whatis-account/
  [Blob]: ../storage-php-how-to-use-blobs/
  [表]: ../storage-php-how-to-use-table-storage/
  [队列]: ../storage-php-how-to-use-queues/
  [AffinityGroup]: https://github.com/WindowsAzure/azure-sdk-for-php/blob/master/WindowsAzure/ServiceManagement/Models/AffinityGroup.php
