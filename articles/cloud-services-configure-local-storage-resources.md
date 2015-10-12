<properties
pageTitle="在 Azure 云服务中配置本地存储资源"
description=""
services="cloud-services"
documentationCenter=""
authors="cristy"
manager="timlt"
editor=""/>
<tags
ms.service="cloud-services"
ms.date="06/11/2015"
wacn.date="10/03/2015"/>

# 配置本地存储资源

本地存储资源是角色实例在其中运行的虚拟机的文件系统中的保留目录。你可以将信息存储在虚拟机实例中，以便在实例中运行的代码需要读取或写入文件时，可以访问本地存储资源。例如，当服务在 Azure 中运行时，本地存储资源可用于缓存可能需要再次访问的数据。此外，你还可以配置本地存储资源，用于在启动时存储文件。有关配置用于启动的本地存储资源的详细信息，请参阅[在启动时使用本地存储来存储文件](https://msdn.microsoft.com/zh-cn/library/azure/hh974419.aspx)

本地存储资源在服务定义文件中声明。可以为一个角色声明任意数量的本地存储资源。将为该角色的每个实例保留每种本地存储资源。可以分配给本地存储资源的最小磁盘空间量为 1 MB。分配给任何给定本地资源的最大限额取决于为该角色指定的虚拟机的大小。每种虚拟机大小都有对应的存储分配总量，并且分配给为角色声明的所有本地存储资源的总空间不能超过分配给该虚拟机大小的最大限额。有关分配给每种虚拟机大小的本地磁盘空间最大限额的详细信息，请参阅[配置云服务大小](https://msdn.microsoft.com/zh-cn/library/azure/ee814754.aspx)。

> [AZURE.NOTE]
>
-   请注意，确保本地存储资源所需的磁盘空间量不超过分配给虚拟机的最大限额是开发人员的责任。如果你配置的本地存储资源大于允许的最大限额，则在你尝试执行超过允许限额的写入操作之前，不会出现错误。在这种情况下，写入操作将失败，并且会显示一条错误消息，指示磁盘空间不足。Azure 的处理模型是尝试/失败。在收到磁盘空间不足的错误后，你可以处理该错误并清理出一些磁盘空间。然后重新尝试该写入操作。
-   可以指定在回收实例时保留本地存储资源。但是，保存到虚拟机的本地文件系统中的数据不一定能保证持久性。如果角色需要持久性数据，建议你使用 Azure 驱动器来存储文件数据。Azure 驱动器由 Azure BLOB 服务提供支持，因此可确保其持久性。  
>


## 添加本地存储资源

若要在服务定义文件中声明本地存储资源，请将 **LocalResources** 元素作为 **WebRole** 元素或 **WorkerRole** 元素的子元素添加。然后添加 **LocalStorage** 元素来表示资源。**LocalStorage** 元素有三个属性：

-   *name*
-   *sizeInMB*：指定此本地存储资源所需的大小
-   *cleanOnRoleRecycle*：指定在回收角色实例时是否应清除本地存储资源，以及在角色的生命周期内是否应保留该资源。默认值为 **true**。

下面的服务定义文件展示为一个 Web 角色声明的两个本地存储资源：

	<?xml version="1.0" encoding="utf-8"?>
    <ServiceDefinition xmlns="http://schemas.microsoft.com/ServiceHosting/2008/10/ServiceDefinition" name="MyService">
      <WebRole name="MyService_WebRole" vmsize="Medium">
        <InputEndpoints>
          <InputEndpoint name="HttpIn" port="80" protocol="http" />
        </InputEndpoints>
        <ConfigurationSettings>
          <Setting name="SimpleConfigSetting" />
        </ConfigurationSettings>
        <LocalResources>
          <LocalStorage name="localStoreOne" sizeInMB="10" />
          <LocalStorage name="localStoreTwo" sizeInMB="10" cleanOnRoleRecycle="false" />
        </LocalResources>
      </WebRole>
    </ServiceDefinition>

有关服务定义文件的详细信息，请参阅 [Azure 服务定义架构（.csdef 文件）](https://msdn.microsoft.com/zh-cn/library/azure/ee758711.aspx)。

> [AZURE.NOTE]如果你使用 Azure Tools for Microsoft Visual Studio，则可在“属性”页中为角色定义本地存储资源。有关详细信息，请参阅[使用 Visual Studio 配置 Azure 应用程序](https://msdn.microsoft.com/zh-cn/library/ee405486.aspx)。

## 以编程方式访问本地存储资源

若要访问本地存储资源，应用程序必须从 [GetLocalResource](https://msdn.microsoft.com/zh-cn/library/azure/microsoft.windowsazure.serviceruntime.roleenvironment.getlocalresource.aspx) 方法检索路径。然后，你可以使用标准文件读取和写入操作来读取和写入本地存储资源的内容。例如，以下示例演示了如何从本地存储资源中读取名为 **MyTest.txt** 的文件的内容，并将其显示在 MVC 3 应用程序的主页上：

    using Microsoft.WindowsAzure.ServiceRuntime;
    using System;
    using System.Text;
    using System.Web.Mvc;

    namespace StartupExercise.Controllers
    {
        public class HomeController : Controller
        {
            public ActionResult Index()
            {
                string SlsPath = RoleEnvironment.GetLocalResource("StartupLocalStorage").RootPath;

                string s = System.IO.File.ReadAllText(SlsPath + "\\MyTest.txt");

                ViewBag.Message = "Contents of MyTest.txt = " + s;

                return View();
            }
        }
    }

## 在运行时访问本地存储资源

Azure 托管库提供了一些类，可用于从角色实例中运行的代码访问本地存储资源。[RoleEnvironment.GetLocalResource](https://msdn.microsoft.com/zh-cn/library/microsoft.windowsazure.serviceruntime.roleenvironment.getlocalresource.aspx) 方法返回对名为 [LocalResource](https://msdn.microsoft.com/zh-cn/library/microsoft.windowsazure.serviceruntime.localresource.aspx) 对象的引用。

由于 **LocalResource** 对象表示一个目录，你可以使用标准 .NET 文件 I/O 类对其执行读取和写入操作。若要确定本地存储资源目录的路径，请使用 [LocalResource.RootPath](https://msdn.microsoft.com/zh-cn/library/microsoft.windowsazure.serviceruntime.localresource.rootpath.aspx) 属性。此属性将返回本地存储资源的完整路径，包括命名资源目录。例如，如果服务在开发环境中运行，则将在本地文件系统中定义本地存储资源，并且 **RootPath** 属性返回的值与下面列出的类似：


    C:\Users\myaccount\AppData\Local\dftmp\s0\deployment(1)\res\deployment(1).MyService.MyService_WebRole.0\directory\localStoreOne\

在将服务部署到 Azure 时，本地存储资源的路径包含部署 ID，并且 **RootPath** 属性返回的值与下面列出的类似：


    C:\Resources\directory\f335471d5a5845aaa4e66d0359e69066.MyService_WebRole.localStoreOne\

角色实例中运行的代码在实例从联机到脱机期间可以访问为该角色定义的本地存储资源。

## 后续步骤

- [为 Azure 设置云服务](https://msdn.microsoft.com/zh-cn/library/azure/hh124108.aspx)

<!---HONumber=71-->