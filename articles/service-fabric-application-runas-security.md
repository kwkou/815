<properties
   pageTitle="了解 Service Fabric 应用程序 RunAs 安全性策略 | Azure"
   description="有关如何使用系统帐户和本地安全帐户运行 Service Fabric 应用程序概述，包括应用程序在启动之前，需要在其中执行某些特权操作的 SetupEntry 点"
   services="service-fabric"
   documentationCenter=".net"
   authors="msfussell"
   manager="timlt"
   editor="bscholl"/>

<tags
   ms.service="service-fabric"
   ms.date="03/24/2016"
   wacn.date="07/04/2016"/>

# RunAs：使用不同的安全权限运行 Service Fabric 应用程序
Azure Service Fabric 能够保护群集中以不同用户帐户（名为 **RunAs**）运行的应用程序。Service Fabric 还使用此用户帐户来保护应用程序所用的资源，例如文件、目录和证书。

默认情况下，Service Fabric 应用程序在运行 Fabric.exe 程序的帐户之下运行。Service Fabric 还可让你使用应用程序清单中指定的本地用户帐户或本地系统帐户运行应用程序。RunAs 支持的本地系统帐户类型为 **LocalUser**、**NetworkService**、**LocalService** 和 **LocalSystem**。

> [AZURE.NOTE] 可在其中使用 Azure Active Directory 的 Windows Server 部署支持域帐户。

可以定义和创建用户组，以便将一个或多个要统一管理的用户添加到每个组。如果不同的服务入口点有多个用户，而且这些用户需要拥有可在组级别使用的某些常用权限，则这种做法特别有用。

## 为 SetupEntryPoint 设置 RunAs 策略

如[应用程序模型](/documentation/articles/service-fabric-application-model/)中所述，**SetupEntryPoint** 是特权入口点，以与 Service Fabric 相同的凭据（通常是 NetworkService 帐户）先于任何其他入口点运行。由 **EntryPoint** 指定的可执行文件通常是长时间运行的服务主机，因此具有单独的安装入口点，不必长时间运行具有高特权的服务主机可执行文件。由 **EntryPoint** 指定的可执行文件在 **SetupEntryPoint** 成功退出后运行。如果总是终止或出现故障，则将监视并重启所产生的过程（再次从 **SetupEntryPoint** 开始）。

下面是一个简单的服务清单示例，其中显示服务的 SetupEntryPoint 和主要 EntryPoint。

~~~
<?xml version="1.0" encoding="utf-8" ?>
<ServiceManifest Name="MyServiceManifest" Version="SvcManifestVersion1" xmlns="http://schemas.microsoft.com/2011/01/fabric" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance">
  <Description>An example service manifest</Description>
  <ServiceTypes>
    <StatelessServiceType ServiceTypeName="MyServiceType" />
  </ServiceTypes>
  <CodePackage Name="Code" Version="1.0.0">
    <SetupEntryPoint>
      <ExeHost>
        <Program>MySetup.bat</Program>
        <WorkingFolder>CodePackage</WorkingFolder>
      </ExeHost>
    </SetupEntryPoint>
    <EntryPoint>
      <ExeHost>
        <Program>MyServiceHost.exe</Program>
      </ExeHost>
    </EntryPoint>
  </CodePackage>
  <ConfigPackage Name="Config" Version="1.0.0" />
</ServiceManifest>
~~~

### 使用本地帐户配置 RunAs 策略

在配置服务以获取设置入口点之后，你可以在应用程序清单中更改用于运行服务的安全权限。以下示例演示如何将服务配置为以用户管理员帐户特权运行。

~~~
<?xml version="1.0" encoding="utf-8"?>
<ApplicationManifest xmlns:xsd="http://www.w3.org/2001/XMLSchema" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" ApplicationTypeName="MyApplicationType" ApplicationTypeVersion="1.0.0" xmlns="http://schemas.microsoft.com/2011/01/fabric">
   <ServiceManifestImport>
      <ServiceManifestRef ServiceManifestName="MyServiceTypePkg" ServiceManifestVersion="1.0.0" />
      <ConfigOverrides />
      <Policies>
         <RunAsPolicy CodePackageRef="Code" UserRef="SetupAdminUser" EntryPointType="Setup" />
      </Policies>
   </ServiceManifestImport>
   <Principals>
      <Users>
         <User Name="SetupAdminUser">
            <MemberOf>
               <SystemGroup Name="Administrators" />
            </MemberOf>
         </User>
      </Users>
   </Principals>
</ApplicationManifest>
~~~

首先，以用户名（例如 SetupAdminUser）创建 **Principals** 节。这指明用户是管理员系统组的成员。

接下来，在 **ServiceManifestImport** 节下面配置策略，以将此主体应用到 **SetupEntryPoint**。这会告知 Service Fabric，当 **MySetup.bat** 文件运行时，它应该是具有管理员特权的 RunAs。假设你*尚未*将策略应用到主入口点，则 **MyServiceHost.exe** 中的代码将以系统 **NetworkService** 帐户运行。这是用于运行所有服务入口点的默认帐户。

现在让我们将 MySetup.bat 文件添加 Visual Studio 项目，以测试管理员特权。在 Visual Studio 中，右键单击服务项目，然后添加名为 MySetup.bat 的新文件。

接下来，确保 MySetup.bat 文件已包含在服务包中。默认情况下未包含该文件。选择该文件，单击右键打开上下文菜单，然后选择“属性”。在“属性”对话框中，确保已将“复制到输出目录”设置为“有更新时才复制”。如以下屏幕截图所示。

![SetupEntryPoint 批处理文件的 Visual Studio CopyToOutput][image1]

现在打开 MySetup.bat 文件并添加以下命令：

~~~
REM Set a system environment variable. This requires administrator privilege
setx -m TestVariable "MyValue"
echo System TestVariable set to > out.txt
echo %TestVariable% >> out.txt

REM To delete this system variable us
REM REG delete "HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\Session Manager\Environment" /v TestVariable /f
~~~

接下来，构建解决方案并将其部署到本地开发群集。启动该服务之后（如 Service Fabric 资源管理器中所示），你可以看到 MySetup.bat 在两个方面都已成功。打开 PowerShell 命令提示符并键入：

~~~
PS C:\ [Environment]::GetEnvironmentVariable("TestVariable","Machine")
MyValue
~~~

接下来，记下已在 Service Fabric 资源管理器中部署并启动服务的节点名称，例如“节点 2”。接下来，导航到应用程序实例工作文件夹，找到显示 **TestVariable** 值的 out.txt 文件。例如，如果此服务已部署到节点 2，则你可以转到 **MyApplicationType** 的此路径：

~~~
C:\SfDevCluster\Data\_App\Node.2\MyApplicationType_App\work\out.txt
~~~

###  使用本地系统帐户配置 RunAs 策略
如上所示，通常建议使用本地系统帐户，而不是管理员帐户运行启动脚本。用管理员帐户运行 RunAs 策略通常效果不佳，因为计算机在默认情况下启用用户访问控制 (UAC)。在这种情况下，**建议将 SetupEntryPoint 作为 LocalSystem 运行，而不要作为添加到管理员组的本地用户来运行**。以下示例演示如何将 SetupEntryPoint 设置为作为 LocalSystem 运行。

~~~
<?xml version="1.0" encoding="utf-8"?>
<ApplicationManifest xmlns:xsd="http://www.w3.org/2001/XMLSchema" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" ApplicationTypeName="MyApplicationType" ApplicationTypeVersion="1.0.0" xmlns="http://schemas.microsoft.com/2011/01/fabric">
   <ServiceManifestImport>
      <ServiceManifestRef ServiceManifestName="MyServiceTypePkg" ServiceManifestVersion="1.0.0" />
      <ConfigOverrides />
      <Policies>
         <RunAsPolicy CodePackageRef="Code" UserRef="SetupLocalSystem" EntryPointType="Setup" />
      </Policies>
   </ServiceManifestImport>
   <Principals>
      <Users>
         <User Name="SetupLocalSystem" AccountType="LocalSystem" />
      </Users>
   </Principals>
</ApplicationManifest>
~~~

##  从 SetupEntryPoint 启动 PowerShell 命令
若要从 **SetupEntryPoint** 点运行 PowerShell，可以在指向 PowerShell 文件的批处理文件中运行 **PowerShell.exe**。首先，将 PowerShell 文件添加到服务项目（例如 **MySetup.ps1**）。请记得设置“如果较新则复制”属性，使该文件也包含在服务包中。以下示例显示的示例批处理文件可启动名为 MySetup.ps1 的 PowerShell 文件，用于设置名为 **TestVariable** 的系统环境变量。


MySetup.bat 可启动 PowerShell 文件。

~~~
powershell.exe -ExecutionPolicy Bypass -Command ".\MySetup.ps1"
~~~

在 PowerShell 文件中，添加以下内容来设置系统环境变量：

```
[Environment]::SetEnvironmentVariable("TestVariable", "MyValue", "Machine")
[Environment]::GetEnvironmentVariable("TestVariable","Machine") > out.txt
```
**注意：**默认情况下，批处理文件运行时会在名为 **work** 的应用程序文件夹中查找文件。此示例中，当 MySetup.bat 运行时，我们想在相同的文件夹（即应用程序的 **code package** 文件夹）中查找 MySetup.ps1。若要更改此文件夹，请如下所示设置工作文件夹。
    
~~~
<SetupEntryPoint>
    <ExeHost>
    <Program>MySetup.bat</Program>
    <WorkingFolder>CodePackage</WorkingFolder>
    </ExeHost>
</SetupEntryPoint> 
~~~

## 使用控制台重定向策略对入口点进行本地调试
有时通过运行脚本进行调试来查看控制台输出很有用。为此你可以设置一个控制台重定向策略，用它将输出写入文件。文件输出被写入名为 **log** 的应用程序文件夹，此文件夹位于部署和运行应用程序的节点上（请参阅上述示例查看如何找到此节点）。

**注意：**一定不要在部署到生产环境中的应用程序中使用控制台重定向策略，因为这样会影响应用程序的故障转移。**只能**使用此策略来进行本地开发和调试。

以下示例演示如何使用 FileRetentionCount 值设置控制台重定向。

~~~
<SetupEntryPoint>
    <ExeHost>
    <Program>MySetup.bat</Program>
    <WorkingFolder>CodePackage</WorkingFolder>
    <ConsoleRedirection FileRetentionCount="10"/>
    </ExeHost>
</SetupEntryPoint> 
~~~

如果现在将 MySetup.ps1 文件改为写入 **Echo** 命令，则策略将写入输出文件以进行调试。

~~~
Echo "Test console redirection which writes to the application log folder on the node that the application is deployed to" 
~~~

**调试脚本之后，应立即删除此控制台重定向策略**

## 将 RunAsPolicy 应用到服务
在上述步骤中，你已了解如何将 RunAs 策略应用到 SetupEntryPoint。让我们深入了解如何创建可作为服务策略应用的不同主体。

### 创建本地用户组
可以定义和创建用户组，然后将一个或多个用户添加到组。如果不同的服务入口点有多个用户，而且这些用户需要拥有可在组级别使用的某些常用权限，则这种做法特别有用。以下示例展示名为 **LocalAdminGroup** 且具有管理员特权的本地组。Customer1 和 Customer2 这两个用户已成为此本地组的成员。

~~~
<Principals>
 <Groups>
   <Group Name="LocalAdminGroup">
     <Membership>
       <SystemGroup Name="Administrators"/>
     </Membership>
   </Group>
 </Groups>
  <Users>
     <User Name="Customer1">
        <MemberOf>
           <Group NameRef="LocalAdminGroup" />
        </MemberOf>
     </User>
    <User Name="Customer2">
      <MemberOf>
        <Group NameRef="LocalAdminGroup" />
      </MemberOf>
    </User>
  </Users>
</Principals>
~~~

### 创建本地用户
你可以创建一个本地用户，用于保护应用程序中的服务。在应用程序清单的 principals 部分指定 **LocalUser** 帐户类型时，Service Fabric 将在部署应用程序的计算机上创建本地用户帐户。默认情况下，这些帐户的名称不与应用程序清单中指定的名称相同（例如，以下示例中的 Customer3）。相反，它们是动态生成的并带有随机密码。

~~~
<Principals>
  <Users>
     <User Name="Customer3" AccountType="LocalUser" />
  </Users>
</Principals>
~~~

<!-- If an application requires that the user account and password be same on all machines (for example, to enable NTLM authentication), the cluster manifest must set NTLMAuthenticationEnabled to true. The cluster manifest must also specify an NTLMAuthenticationPasswordSecret that will be used to generate the same password across all machines.

<Section Name="Hosting">
      <Parameter Name="EndpointProviderEnabled" Value="true"/>
      <Parameter Name="NTLMAuthenticationEnabled" Value="true"/>
      <Parameter Name="NTLMAuthenticationPassworkSecret" Value="******" IsEncrypted="true"/>
 </Section>
-->

## 将策略分配到服务代码包
**ServiceManifestImport** 的 **RunAsPolicy** 部分指定 principals 部分中应该用来运行代码包的帐户。它还将服务清单中的代码包关联到 principals 节中的用户帐户。可以为设置入口点或主入口点指定此策略，或者指定 All 以将其应用到两者。以下示例演示了应用不同的策略：

~~~
<Policies>
<RunAsPolicy CodePackageRef="Code" UserRef="LocalAdmin" EntryPointType="Setup"/>
<RunAsPolicy CodePackageRef="Code" UserRef="Customer3" EntryPointType="Main"/>
</Policies>
~~~

如果未指定 **EntryPointType**，则 EntryPointType 的默认值设置为“Main”。如果你想要以系统帐户运行某些高特权设置操作，则指定 **SetupEntryPoint** 将特别有用。实际服务代码可使用较低特权的帐户来运行。

### 将默认策略应用到所有服务代码包
**DefaultRunAsPolicy** 部分用于针对未定义特定 **RunAsPolicy** 的所有代码包指定默认用户帐户。如果在应用程序所用的服务清单中指定的大多数代码包必须以同一 RunAs 用户运行，则应用程序可以只定义该用户帐户的默认 RunAs 策略，而无需针对每个代码包指定 **RunAsPolicy**。以下示例指定如果代码包未指定 **RunAsPolicy**，则此代码包应该以 principals 部分指定的 **MyDefaultAccount** 运行。

~~~
<Policies>
  <DefaultRunAsPolicy UserRef="MyDefaultAccount"/>
</Policies>
~~~

## 为 HTTP 和 HTTPS 终结点分配 SecurityAccessPolicy
如果向服务应用 RunAs 策略务，而服务清单声明具有 HTTP 协议的终结点资源，则必须指定 **SecurityAccessPolicy**，以确保分配给这些终结点的端口都已针对用来运行服务的 RunAs 用户帐户正确列入访问控制列表中。否则，**http.sys** 将无权访问服务，并且将无法从客户端调用。以下示例将 Customer3 帐户应用到名为 **ServiceEndpointName** 的终结点，并向它授予完全访问权限。

~~~
<Policies>
   <RunAsPolicy CodePackageRef="Code" UserRef="Customer1" />
   <!--SecurityAccessPolicy is needed if RunAsPolicy is defined and the Endpoint is http -->
   <SecurityAccessPolicy ResourceRef="EndpointName" PrincipalRef="Customer1" />
</Policies>
~~~

针对 HTTPS 终结点，你还必须指出要返回给客户端的证书名称。可以使用 **EndpointBindingPolicy** 结合应用程序清单的 certificates 部分定义的证书，来实现此目的。

~~~
<Policies>
   <RunAsPolicy CodePackageRef="Code" UserRef="Customer1" />
  <!--SecurityAccessPolicy is needed if RunAsPolicy is defined and the Endpoint is http -->
   <SecurityAccessPolicy ResourceRef="EndpointName" PrincipalRef="Customer1" />
  <!--EndpointBindingPolicy is needed if the EndpointName is secured with https -->
  <EndpointBindingPolicy EndpointRef="EndpointName" CertificateRef="Cert1" />
</Policies
~~~


## 完整应用程序清单的示例
以下应用程序清单显示了上述许多不同的设置：

~~~
<?xml version="1.0" encoding="utf-8"?>
<ApplicationManifest xmlns:xsd="http://www.w3.org/2001/XMLSchema" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" ApplicationTypeName="Application3Type" ApplicationTypeVersion="1.0.0" xmlns="http://schemas.microsoft.com/2011/01/fabric">
   <Parameters>
      <Parameter Name="Stateless1_InstanceCount" DefaultValue="-1" />
   </Parameters>
   <ServiceManifestImport>
      <ServiceManifestRef ServiceManifestName="Stateless1Pkg" ServiceManifestVersion="1.0.0" />
      <ConfigOverrides />
      <Policies>
         <RunAsPolicy CodePackageRef="Code" UserRef="Customer1" />
         <RunAsPolicy CodePackageRef="Code" UserRef="LocalAdmin" EntryPointType="Setup" />
        <!--SecurityAccessPolicy is needed if RunAsPolicy is defined and the Endpoint is http -->
         <SecurityAccessPolicy ResourceRef="EndpointName" PrincipalRef="Customer1" />
        <!--EndpointBindingPolicy is needed the EndpointName is secured with https -->
        <EndpointBindingPolicy EndpointRef="EndpointName" CertificateRef="Cert1" />
     </Policies>
   </ServiceManifestImport>
   <DefaultServices>
      <Service Name="Stateless1">
         <StatelessService ServiceTypeName="Stateless1Type" InstanceCount="[Stateless1_InstanceCount]">
            <SingletonPartition />
         </StatelessService>
      </Service>
   </DefaultServices>
   <Principals>
      <Groups>
         <Group Name="LocalAdminGroup">
            <Membership>
               <SystemGroup Name="Administrators" />
            </Membership>
         </Group>
      </Groups>
      <Users>
         <User Name="LocalAdmin">
            <MemberOf>
               <Group NameRef="LocalAdminGroup" />
            </MemberOf>
         </User>
         <!--Customer1 below create a local account that this service runs under -->
         <User Name="Customer1" />
         <User Name="MyDefaultAccount" AccountType="NetworkService" />
      </Users>
   </Principals>
   <Policies>
      <DefaultRunAsPolicy UserRef="LocalAdmin" />
   </Policies>
   <Certificates>
	 <EndpointCertificate Name="Cert1" X509FindValue="FF EE E0 TT JJ DD JJ EE EE XX 23 4T 66 "/>
  </Certificates>
</ApplicationManifest>
~~~


<!--Every topic should have next steps and links to the next logical set of content to keep the customer engaged-->
## 后续步骤

* [了解应用程序模型](/documentation/articles/service-fabric-application-model/)
* [在服务清单中指定资源](/documentation/articles/service-fabric-service-manifest-resources/)
* [部署应用程序](/documentation/articles/service-fabric-deploy-remove-applications/)

[image1]: ./media/service-fabric-application-runas-security/copy-to-output.png

<!---HONumber=Mooncake_0425_2016-->