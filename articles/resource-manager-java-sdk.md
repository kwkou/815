<properties
   pageTitle="Resource Manager SDK for Java | Azure"
   description="概述 Resource Manager Java SDK 身份验证和用例"
   services="azure-resource-manager"
   documentationCenter="na"
   authors="navalev"
   manager=""
   editor=""/>

<tags
   ms.service="azure-resource-manager"
   ms.date="03/16/2016"
   wacn.date="04/05/2016"/>
   
# Azure Resource Manager SDK for Java
   
Azure Resource Manager (ARM) 预览版 SDK 适用于多种语言和平台。每种语言中实现可通过其生态系统包管理器和 GitHub 来使用。

其中每个 SDK 中的代码是从 [Azure RESTful API 规范](https://github.com/azure/azure-rest-api-specs)生成的。这些规范是开源代码，基于 Swagger v2 规范。SDK 代码是通过名为 [AutoRest](https://github.com/azure/autorest) 的开源项目生成的。AutoRest 将这些 RESTful API 规范转换成采用多种语言的客户端库。如果你想要改进 SDK 中生成的代码的任何方面，你可以使用开放的、免费提供的、基于广泛采用 API 规范格式的整个工具集。

Azure Resource Manager Java SDK 托管在 GitHub [Azure Java SDK 存储库](https://github.com/azure/azure-sdk-for-java)中。请注意，在撰写本文时，该 SDK 以预览版提供。提供了以下包：

* 计算管理：(azure-mgmt-compute)
* DNS 管理：(azure-mgmt-dns)
* 网络管理：(azure-mgmt-network)
* 资源管理：(azure-mgmt-resources)
* SQL 管理：(azure-mgmt-sql)
* 存储管理：(azure-mgmt-storage)
* 流量管理器管理：(azure-mgmt-traffic-manager)
* 实用程序和帮助器：(azure-mgmt-utility)
* 网站/WebApps 管理：(azure-mgmt-websites)

有关最新列表，请查看 Azure SDK 的 [Java 功能 Wiki 页](https://github.com/Azure/azure-sdk-for-java/wiki/Azure-SDK-for-Java-Features)。

## 先决条件
1. Java v1.6+
2. [maven](https://maven.apache.org/)（如果你想要在 SDK 上进行开发）

## 获取 SDK
[Maven](https://maven.apache.org/) 分布式 jar 是入门 Azure Java SDK 的推荐方法。你可以将这些依赖项添加到许多 Java 依赖性管理工具（Maven、Gradle、Ivy，等等）。有关可在 maven 中使用的库列表，请单击此[链接](http://search.maven.org/#search%7Cga%7C1%7Cg%3A%22com.microsoft.azure%22)。

或者，可以使用 git 直接从源中获取该 SDK。通过 git 获取 SDK 的源代码：

    git clone https://github.com/Azure/azure-sdk-for-java.git
    cd ./azure-sdk-for-java/

（本概述中的示例将使用 Maven 作为 SDK 包源）

SDK 包括某些方案的示例 - 身份验证、预配 VM，等等。可以在 [azure-mgmt-samples](https://github.com/Azure/azure-sdk-for-java/tree/master/azure-mgmt-samples) GitHub 存储库中找到所有示例。

## 帮助器类

SDK 包含多个主包的帮助器类。帮助器类实现于 auzre-mgmt-utility 包中：

* AuthHelper - 身份验证帮助器类
* ComputeHelper - 计算帮助器类
* NetworkHelper - 网络帮助器类
* ResourceHelper - 部署帮助器类
* StorageHelper - 存储帮助器类

**Maven 依赖性信息**

    <dependency>
      <groupId>com.microsoft.azure</groupId>
      <artifactId>azure-mgmt-utility</artifactId>
      <version>0.9.1</version>
    </dependency>

这将使用 0.9.1 版的实用程序包。

## 身份验证

对 ARM 的身份验证由 Azure Active Directory (AD) 处理。若要连接到任何 API，首先需要使用 Azure AD 进行身份验证，以接收可传递给每个请求的身份验证令牌。若要获取此令牌，首先需要创建所谓的 Azure AD 应用程序和一个用于登录的服务主体。有关分步说明，请参阅[创建 Azure AD 应用程序和服务主体](/documentation/articles/resource-group-create-service-principal-portal/)。

创建服务主体后，你应会获得：

* 客户端 ID (GUID)
* 客户端机密（字符串）
* 租户 ID (GUID) 或域名（字符串）

获取这些值后，可以获取有效期为一小时的 Active Directory 访问令牌。

提供客户端 ID、机密和租户 ID 后，Java SDK 将包含一个用于创建访问令牌的帮助器类 AuthHelper。[ServicePrincipalExample](https://github.com/Azure/azure-sdk-for-java/blob/master/azure-mgmt-samples/src/main/java/com/microsoft/azure/samples/authentication/ServicePrincipalExample.java) 类中的以下示例使用 AuthHelper *getAccessTokenFromServicePrincipalCredentials* 方法来获取访问令牌：

```java
public static Configuration createConfiguration() throws Exception {
   String baseUri = System.getenv("arm.url");

   return ManagementConfiguration.configure(
         null,
         baseUri != null ? new URI(baseUri) : null,
         System.getenv(ManagementConfiguration.SUBSCRIPTION_ID),
         AuthHelper.getAccessTokenFromServicePrincipalCredentials(
                  System.getenv(ManagementConfiguration.URI), System.getenv("arm.aad.url"),
                  System.getenv("arm.tenant"), System.getenv("arm.clientid"),
                  System.getenv("arm.clientkey"))
                  .getAccessToken());
}
```

## 创建虚拟机 
实用程序包包含一个用于创建虚拟机的帮助器类 [ComputeHelper](https://github.com/Azure/azure-sdk-for-java/blob/master/resource-management/azure-mgmt-utility/src/main/java/com/microsoft/azure/utility/ComputeHelper.java)。在 azure-mgmt-samples 包的 [compute](https://github.com/Azure/azure-sdk-for-java/tree/master/azure-mgmt-samples/src/main/java/com/microsoft/azure/samples/compute) 下面，可以找到有关处理虚拟机的若干示例。

下面是创建虚拟机的简单流程。在本示例中，帮助器类将在创建 VM 的过程中创建存储和网络：

```java
public static void main(String[] args) throws Exception {
        Configuration config = createConfiguration();
        ResourceManagementClient resourceManagementClient = ResourceManagementService.create(config);
        StorageManagementClient storageManagementClient = StorageManagementService.create(config);
        ComputeManagementClient computeManagementClient = ComputeManagementService.create(config);
        NetworkResourceProviderClient networkResourceProviderClient = NetworkResourceProviderService.create(config);

        String resourceGroupName = "javasampleresourcegroup";
        String region = "EastAsia";

        ResourceContext context = new ResourceContext(
                region, resourceGroupName, System.getenv(ManagementConfiguration.SUBSCRIPTION_ID), false);

        System.out.println("Start create vm...");

        try {
            VirtualMachine vm = ComputeHelper.createVM(
                    resourceManagementClient, computeManagementClient, networkResourceProviderClient, 
                    storageManagementClient,
                    context, vnName, vmAdminUser, vmAdminPassword)
              .getVirtualMachine();

            System.out.println(vm.getName() + " is created");
        } catch (Exception ex) {
            System.out.println(ex.toString());
        }
}
```

## 部署模板
[ResouceHelper](https://github.com/Azure/azure-sdk-for-java/blob/master/resource-management/azure-mgmt-utility/src/main/java/com/microsoft/azure/utility/ResourceHelper.java) 类旨在简化使用 Java SDK 部署 ARM 模板的过程。

```java
// create a new resource group
ResourceManagementClient resourceManagementClient = createResourceManagementClient();
ResourceContext resourceContext = new ResourceContext(
        resourceGroupLocation, resourceGroupName,
        System.getenv(ManagementConfiguration.SUBSCRIPTION_ID), false);
ComputeHelper.createOrUpdateResourceGroup(resourceManagementClient, resourceContext);

// create the template deployment
DeploymentExtended deployment = ResourceHelper.createTemplateDeploymentFromURI(
        resourceManagementClient,
        resourceGroupName,
        DeploymentMode.Incremental,
        deploymentName,
        TEMPLATE_URI,
        "1.0.0.0",
        parameters);
```
## 列出所有虚拟机
你不一定要使用帮助器类（不过它可以提供方便），而可以改为对每个资源提供程序直接使用服务类。在本示例中，我们将列出已经过身份验证的订阅下的某些资源 - 对于每个资源组，我们将查找虚拟机，然后查找与其关联的 IP。

```java
// authenticate and get access token
Configuration config = createConfiguration();
ResourceManagementClient resourceManagementClient = ResourceManagementService.create(config);
ComputeManagementClient computeManagementClient = ComputeManagementService.create(config);
NetworkResourceProviderClient networkResourceProviderClient = NetworkResourceProviderService.create(config);

// list all resource groups     
ArrayList<ResourceGroupExtended> resourceGroups = resourceManagementClient.getResourceGroupsOperations().list(null).getResourceGroups();
for (ResourceGroupExtended resourcesGroup : resourceGroups) {
   String rgName = resourcesGroup.getName();
   System.out.println("Resource Group: " + rgName);
   
   // list all virtual machines
   ArrayList<VirtualMachine> vms = computeManagementClient.getVirtualMachinesOperations().list(rgName).getVirtualMachines();
   for (VirtualMachine vm : vms) {
      System.out.println("    VM: " + vm.getName());
      // list all nics
      ArrayList<NetworkInterfaceReference> nics = vm.getNetworkProfile().getNetworkInterfaces();
      for (NetworkInterfaceReference nicReference : nics) {
         String[] nicURI = nicReference.getReferenceUri().split("/");
         NetworkInterface nic = networkResourceProviderClient.getNetworkInterfacesOperations().get(rgName, nicURI[nicURI.length - 1]).getNetworkInterface();
         System.out.println("        NIC: " + nic.getName());
         System.out.println("        Is primary: " + nic.isPrimary());
         ArrayList<NetworkInterfaceIpConfiguration> ips = nic.getIpConfigurations();

         // find public ip address
         for (NetworkInterfaceIpConfiguration ipConfiguration : ips) {
               System.out.println("        Private IP address: " + ipConfiguration.getPrivateIpAddress());
               String[] pipID = ipConfiguration.getPublicIpAddress().getId().split("/");
               PublicIpAddress pip = networkResourceProviderClient.getPublicIpAddressesOperations().get(rgName, pipID[pipID.length - 1]).getPublicIpAddress();
               System.out.println("        Public IP address: " + pip.getIpAddress());
         }
      }
}  
```

可以在示例包的 [templatedeployments](https://github.com/Azure/azure-sdk-for-java/tree/master/azure-mgmt-samples/src/main/java/com/microsoft/azure/samples/templatedeployments) 下面找到更多示例。

## 其他阅读材料和帮助
Azure SDK for Java 文档：[Java 文档](http://azure.github.io/azure-sdk-for-java/)

如果你在使用该 SDK 时遇到任何 bug，请通过[问题](https://github.com/Azure/azure-sdk-for-java/issues)报告你的问题，或查看 [Azure Java SDK 堆栈溢出](http://stackoverflow.com/questions/tagged/azure-java-sdk)。

<!---HONumber=Mooncake_0328_2016-->