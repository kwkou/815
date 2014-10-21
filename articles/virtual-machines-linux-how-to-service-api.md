<properties linkid="manage-linux-howto-service-management-api" urlDisplayName="Service Management API" pageTitle="How to use the service management API for VMs - Azure" metaKeywords="" description="Learn how to use the Azure Service Management API for a Linux virtual machine." metaCanonical="" services="virtual-machines" documentationCenter="" title="How to Use the Service Management API" authors="" solutions="" manager="" editor="" />

# 如何使用服务管理 API

## 初始化

若要从 NodeJS 调用 Azure IaaS 服务管理 API，可使用`azure` 模块。

    var azure = require('azure');

首先，创建 ServiceManagementService 的实例。对该 API 的所有调用都将使用此对象。此时会创建订阅 ID、凭据和其他连接选项。若要管理多个订阅，请创建多个 ServiceManagementService。

    var iaasClient = azure.createServiceManagementService(subscriptionid, auth, options);

-   Subscriptionid 是必需字符串。它应是将访问的帐户的订阅 ID。
-   Auth 是一个可选对象，该对象指定要与该帐户一起使用的私钥和公用证书。

    -   keyfile - 具有私钥的 .pem 文件的文件路径。在指定 keyvalue 时将被忽略。
    -   keyvalue - 存储在 .pem 文件中的私钥的实际值。
    -   certfile - 具有公用证书的 .pem 文件的文件路径。在指定 cervalue 时将被忽略。
    -   certvalue - 存储在 .pem 文件中的公用证书的实际值。
    -   如果未指定上述值，则将读取并使用进程环境变量值`CLIENT_AUTH_KEYFILE` 和 `CLIENT_AUTH_CERTFILE` 。如果未设置这些值，则会尝试文件的默认值：priv.pem 和 pub.pem。
    -   如果无法加载私钥和公用证书，则将引发错误。
-   Options 是一个可选对象，该对象可用于控制客户端所使用的属性。

    -   host - Azure 管理服务器的主机名。如果未指定此属性，则使用默认主机。
    -   apiversion - 用于 HTTP 标头的版本字符串。如果未指定此属性，则使用默认版本。
    -   serializetype - 可以为 XML 或 JSON。如果未指定此属性，则使用默认序列化。

（可选）可使用隧道代理以使 HTTPS 请求能够通过代理。创建 IaasClient 后，它将处理环境变量`HTTPS_PROXY`。如果将其设置为有效 URL，则可从 URL 解析主机名和端口，并且可在后续请求中使用它们来标识代理。

        iaasClient.SetProxyUrl(proxyurl)

可调用 SetProxyUrl 来显式设置代理主机和端口。它的效果与设置`HTTPS_PROXY` 环境变量的效果相同，并且将覆盖环境设置。

## 回调

所有 API 都具有一个必需的回调参数。通过调用已传入回调中的函数来发出请求已完成的信号。

    callback(rsp)

-   回调具有一个作为响应对象的参数。
-   isSuccessful - true 或 false
-   statusCode - 响应中的 HTTP 状态代码
-   response - 解析为 javascript 对象的响应。如果 isSuccessful 为 true，则设置此参数。
-   error - 一个包含错误信息的 javascript 对象。如果 isSuccessful 为 false，则设置此参数。
-   body - 字符串形式的响应的实际主体
-   headers - 响应的实际 HTTP 标头
-   reqopts - 用于发出请求的节点 HTTP 请求选项。

请注意，在某些情况下，完成只能指示已接受请求。在此情况下，可使用 **GetOperationStatus** 获取最终状态。

## API

**iaasClient.GetOperationStatus(requested, callback)**

-   许多管理调用会在操作完成之前返回。这些调用将返回值“202 已接受”，并在 ms-request-id HTPP 标头中放置请求的值。若要轮询请求的完成情况，请使用此 API 并传入请求的值。
-   callback 是必需的。

**iaasClient.GetOSImage(imagename, callback)**

-   imagename 是映像的必需的字符串名称。
-   callback 是必需的。
-   如果操作成功，则响应对象将包含已命名映像的属性。

**iaasClient.ListOSImages(callback)**

-   callback 是必需的。
-   如果操作成功，则响应对象将包含映像对象的数组。

**iaasClient.CreateOSImage(imageName, mediaLink, imageOptions, callback)**

-   imageName 是映像的必需的字符串名称。
-   mediaLink 是要使用的 mediaLink 的必需的字符串名称。
-   imageOptions 是一个可选对象。它可包含下列属性

    -   类别
    -   Label - 如果未设置，则默认为 imageName。
    -   位置
    -   RoleSize
-   callback 是必需的。（如果未设置 imageOptions，则可能为第三个参数。）
-   如果操作成功，则响应对象将包含已创建映像的属性。

**iaasClient.ListHostedServices(callback)**

-   callback 是必需的。
-   如果操作成功，则响应对象将包含托管服务对象的数组。

**iaasClient.GetHostedService(serviceName, callback)**

-   serviceName 是托管服务的必需的字符串名称。
-   callback 是必需的。
-   如果操作成功，则响应对象将包含托管服务的属性。

**iaasClient.CreateHostedService(serviceName, serviceOptions, callback)**

-   serviceName 是托管服务的必需的字符串名称。
-   serviceOptions 是一个可选对象。它可包含下列属性

    -   Description - 默认为“Service host”
    -   Label - 如果未设置，则默认为 serviceName。
    -   Location - 默认为“Azure Preview” - 发布后需要更改。
-   callback 是必需的。

**iaasClient.GetStorageAccountKeys(serviceName, callback)**

-   serviceName 是托管服务的必需的字符串名称。
-   callback 是必需的。
-   如果操作成功，则响应对象将包含存储访问密钥。

**iaasClient.GetDeployment(serviceName, deploymentName, callback)**

-   serviceName 是托管服务的必需的字符串名称。
-   deploymentName 是部署的必需的字符串名称。
-   callback 是必需的。
-   如果操作成功，则响应对象将包含已命名部署的属性。

**iaasClient.GetDeploymentBySlot(serviceName, deploymentSlot, callback)**

-   serviceName 是托管服务的必需的字符串名称。
-   deploymentSlot 是槽的必需的字符串名称（过渡或生产）。
-   callback 是必需的。
-   如果操作成功，则响应对象将包含槽中部署的属性。

**iaasClient.CreateDeployment(serviceName, deploymentName, VMRole, deployOptions, callback)**

-   serviceName 是托管服务的必需的字符串名称。
-   deploymentName 是部署的必需的字符串名称。
-   VMRole 是必需的对象，该对象具有要为部署创建的角色的属性。
-   deployOptions 是一个可选对象。它可包含下列属性

    -   DeploymentSlot - 默认为“Production”
    -   Label - 如果未设置，则默认为 deploymentName。
    -   UpgradeDomainCount - 无默认值
-   callback 是必需的。

**iaasClient.GetRole(serviceName, deploymentName, roleName, callback)**

-   serviceName 是托管服务的必需的字符串名称。
-   deploymentName 是部署的必需的字符串名称。
-   roleName 是角色的必需的字符串名称。
-   callback 是必需的。
-   如果操作成功，则响应对象将包含已命名角色的属性。

**iaasClient.AddRole(serviceName, deploymentName, VMRole, callback)**

-   serviceName 是托管服务的必需的字符串名称。
-   deploymentName 是部署的必需的字符串名称。
-   VMRole 是必需的对象，该对象具有要添加到部署的角色的属性。
-   callback 是必需的。

**iaasClient.ModifyRole(serviceName, deploymentName, roleName, VMRole, callback)**

-   serviceName 是托管服务的必需的字符串名称。
-   deploymentName 是部署的必需的字符串名称。
-   roleName 是角色的必需的字符串名称。
-   VMRole 是必需的对象，该对象具有要在角色中修改的属性。
-   callback 是必需的。

**iaasClient.DeleteRole(serviceName, deploymentName, roleName, callback)**

-   serviceName 是托管服务的必需的字符串名称。
-   deploymentName 是部署的必需的字符串名称。
-   roleName 是角色的必需的字符串名称。
-   callback 是必需的。

**iaasClient.AddDataDisk(serviceName, deploymentName, roleName, datadisk, callback)**

-   serviceName 是托管服务的必需的字符串名称。
-   deploymentName 是部署的必需的字符串名称。
-   roleName 是角色的必需的字符串名称。
-   Datadisk 是必需的对象，该对象用于指定将创建数据磁盘的方式。
-   callback 是必需的。

**iaasClient.ModifyDataDisk(serviceName, deploymentName, roleName, LUN, datadisk, callback)**

-   serviceName 是托管服务的必需的字符串名称。
-   deploymentName 是部署的必需的字符串名称。
-   roleName 是角色的必需的字符串名称。
-   LUN 是标识数据磁盘的数字
-   Datadisk 是必需的对象，该对象用于指定将修改数据磁盘的方式。
-   callback 是必需的。

**iaasClient.RemoveDataDisk(serviceName, deploymentName, roleName, LUN, callback)**

-   serviceName 是托管服务的必需的字符串名称。
-   deploymentName 是部署的必需的字符串名称。
-   roleName 是角色的必需的字符串名称。
-   LUN 是标识数据磁盘的数字
-   callback 是必需的。

**iaasClient.ShutdownRoleInstance(serviceName, deploymentName, roleInstance, callback)**

-   serviceName 是托管服务的必需的字符串名称。
-   deploymentName 是部署的必需的字符串名称。
-   roleInstance 是角色实例的必需的字符串名称。
-   callback 是必需的。

**iaasClient.RestartRoleInstance(serviceName, deploymentName, roleInstance, callback)**

-   serviceName 是托管服务的必需的字符串名称。
-   deploymentName 是部署的必需的字符串名称。
-   roleInstance 是角色实例的必需的字符串名称。
-   callback 是必需的。

**iaasClient.CaptureRoleInstance(serviceName, deploymentName, roleInstance, captOptions, callback)**

-   serviceName 是托管服务的必需的字符串名称。
-   deploymentName 是部署的必需的字符串名称。
-   roleInstance 是角色实例的必需的字符串名称。
-   captOptions 是必需的对象，该对象定义捕获操作

    -   PostCaptureActions
    -   ProvisioningConfiguration
    -   SupportsStatelessDeployment
    -   TargetImageLabel
    -   TargetImageName
-   callback 是必需的。

## 数据对象

在创建或修改部署、角色或磁盘时，API 会将对象作为输入。其他 API 将在执行 Get 或 List 操作时返回类似对象。
本节概要介绍对象属性。
部署

-   Name - 字符串
-   DeploymentSlot -“过渡”或“生产”
-   Status - 字符串 - 仅输出
-   PrivateID - 字符串 - 仅输出
-   Label - 字符串
-   UpgradeDomainCount - 数字
-   SdkVersion - 字符串
-   Locked - true 或 false
-   RollbackAllowed - true 或 false
-   RoleInstance - 对象 - 仅输出
-   Role - VMRole 对象
-   InputEndpointList - InputEndpoint 数组

**VMRole**

-   RoleName - 字符串。创建时必需的。
-   RoleSize - 字符串。创建时可选的。
-   RoleType - 字符串。如果未在创建中指定，则默认为“PersistentVMRole”。
-   OSDisk - 对象。创建时必需的
-   DataDisks - 对象的数组。创建时可选的。
-   ConfigurationSets - 配置对象的数组。

**RoleInstance**

-   RoleName - 字符串
-   InstanceName - 字符串
-   InstanceStatus - 字符串
-   InstanceUpgradeDomain - 数字
-   InstanceFaultDomain - 数字
-   InstanceSize - 字符串
-   IpAddress - 字符串

**OSDisk**

-   SourceImageName - 字符串。创建时必需的
-   DisableWriteCache - true 或 false
-   DiskName - 字符串。
-   MediaLink - 字符串

**DataDisk**

-   DisableReadCache - true 或 false
-   EnableWriteCache - true 或 false
-   DiskName - 字符串
-   MediaLink - 字符串
-   LUN - 数字 (0-15)
-   LogicalDiskSizeInGB - 数字
-   SourceMediaLink - 字符串
-   在创建过程中，可通过 3 种方法指定磁盘 - 按名称、按介质或按大小。指定的选项将确定其工作方式。有关详细信息，请参阅 RDFE API 文档。

**ProvisioningConfiguration**

-   ConfigurationSetType -“ProvisioningConfiguration”
-   AdminPassword - 字符串
-   MachineName - 字符串
-   ResetPasswordOnFirstLogon - true 或 false

**NetworkConfiguration**

-   ConfigurationSetType -“NetworkConfiguration”
-   InputEndpoints - ExternalEndpoints 的数组

**InputEndpoint**

-   RoleName
-   Vip
-   Port

**ExternalEndpoint**

-   LocalPort
-   名称
-   Port
-   协议

## 代码示例

以下是一个示例 javascript 代码，此代码创建托管服务和部署，然后轮询部署的完成状态。

    var azure = require('azure');

    // specify where certificate files are located
    var auth = {
      keyfile : '../certs/priv.pem',
      certfile : '../certs/pub.pem'
    }

    // names and IDs for subscription, service, deployment
    var subscriptionId = '167a0c69-cb6f-4522-ba3e-d3bdc9c504e1';
    var serviceName = 'sampleService2';
    var deploymentName = 'sampleDeployment';

    // poll for completion every 10 seconds
    var pollPeriod = 10000;

    // data used to define deployment and role

    var deploymentOptions = {
      DeploymentSlot: 'Staging',
      Label: 'Deployment Label'
    }

    var osDisk = {
      SourceImageName : 'Win2K8SP1.110809-2000.201108-01.en.us.30GB.vhd',
    };

    var dataDisk1 = {
      LogicalDiskSizeInGB : 10,
      LUN : '0'
    };

    var provisioningConfigurationSet = {
      ConfigurationSetType: 'ProvisioningConfiguration',
      AdminPassword: 'myAdminPwd1',
      MachineName: 'sampleMach1',
      ResetPasswordOnFirstLogon: false
    };

    var externalEndpoint1 = {
      Name: 'endpname1',
      Protocol: 'tcp',
      Port: '59919',
      LocalPort: '3395'
    };

    var networkConfigurationSet = {
      ConfigurationSetType: 'NetworkConfiguration',
      InputEndpoints: [externalEndpoint1]
    };

    var VMRole = {
      RoleName: 'sampleRole',
      RoleSize: 'Small',
      OSDisk: osDisk,
      DataDisks: [dataDisk1],
      ConfigurationSets: [provisioningConfigurationSet, networkConfigurationSet]
    }


    // function to show error messages if failed
    function showErrorResponse(rsp) {
      console.log('There was an error response from the service');
      console.log('status code=' + rsp.statusCode);
      console.log('Error Code=' + rsp.error.Code);
      console.log('Error Message=' + rsp.error.Message);
    }

    // polling for completion
    function PollComplete(reqid) {
      iaasCli.GetOperationStatus(reqid, function(rspobj) {
        if (rspobj.isSuccessful && rspobj.response) {
          var rsp = rspobj.response;
          if (rsp.Status == 'InProgress') {
            setTimeout(PollComplete(reqid), pollPeriod);
            process.stdout.write('.');
          } else {
            console.log(rsp.Status);
            if (rsp.HttpStatusCode) console.log('HTTP Status: ' + rsp.HttpStatusCode);
            if (rsp.Error) {      
              console.log('Error code: ' + rsp.Error.Code);
              console.log('Error Message: ' + rsp.Error.Message);
            }
          }
        } else {
          showErrorResponse(rspobj);
        }
      });
    }


    // create the client object
    var iaasCli = azure.createServiceManagementService(subscriptionId, auth);

    // create a hosted service.
    // if successful, create deployment
    // if accepted, poll for completion
    iaasCli.CreateHostedService(serviceName, function(rspobj) {
      if (rspobj.isSuccessful) {
        iaasCli.CreateDeployment(serviceName, 
                                 deploymentName,
                                 VMRole, deploymentOptions,
                                  function(rspobj) {
          if (rspobj.isSuccessful) {
          // get request id, and start polling for completion
            var reqid = rspobj.headers['x-ms-request-id'];
            process.stdout.write('Polling');
            setTimeout(PollComplete(reqid), pollPeriod);
          } else {
            showErrorResponse(rspobj);
          }
        });
      } else {
        showErrorResponse(rspobj);
      }
    });
