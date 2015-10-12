<properties 
	pageTitle="Azure 云服务 - 服务定义和服务配置 - XML 证书" 
	description="了解如何在服务定义和服务配置文件中配置证书。" 
	services="cloud-services" 
	documentationCenter=".net" 
	authors="Thraka" 
	manager="timlt" 
	editor=""/>

<tags 
	ms.service="cloud-services" 
	ms.date="07/24/2015"
	wacn.date="10/03/2015"/>



# 配置证书的服务定义和配置

运行 Web 角色或辅助角色的虚拟机可以具备与之关联的证书。将证书上载到门户中后，你需要配置证书的服务定义 (.csdef) 和服务配置 (.cscfg) 文件。

安装后，虚拟机可以访问该证书的私钥。因此，你可能想要限制对具有提升的权限的进程的访问。

## 服务定义示例

下面是在服务定义中定义的证书的示例。

```xml
<ServiceDefinition name="WindowsAzureProject4" xmlns="http://schemas.microsoft.com/ServiceHosting/2008/10/ServiceDefinition">
  <WorkerRole name="MyWokerRole"> <!-- or <WebRole name="MyWebRole" vmsize="Small"> -->
    <ConfigurationSettings>
      ...
    </ConfigurationSettings>
    <Certificates>
      <Certificate name="MySSLCert" storeLocation="LocalMachine" storeName="My" permissionLevel="elevated" />
    </Certificates>
  </WorkerRole>
</ServiceDefinition>
```

### 权限
权限（`permisionLevel` 属性）可以设置为以下值之一：

| 权限值 | 说明 |
| ----------------  | ----------- |
| limitedOrElevated | **（默认）**所有角色进程都可以访问该私钥。 |
| 提升的 | 仅提升的进程可以访问该私钥。|

## 服务配置示例

下面是在服务配置中定义的证书的示例。

```xml
<Role name="MyWokerRole">
...
    <Certificates>
        <Certificate name="MySSLCert" 
            thumbprint="9427befa18ec6865a9ebdc79d4c38de50e6316ff" 
            thumbprintAlgorithm="sha1" />
    </Certificates>
...
</Role>
```

**请注意**匹配的 `name` 属性。

## 后续步骤
查看[服务定义 XML](https://msdn.microsoft.com/zh-cn/library/azure/ee758711.aspx) 架构和[服务配置 XML](https://msdn.microsoft.com/zh-cn/library/azure/ee758710.aspx) 架构。

<!---HONumber=71-->