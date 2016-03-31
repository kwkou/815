<properties 
   pageTitle="角色的默认 TEMP 文件夹大小太小 | Azure"
   description="云服务角色的 TEMP 文件夹的空间有限。本文针对如何避免磁盘空间不足的问题提供了一些建议。"
   services="cloud-services"
   documentationCenter=""
   authors="Thraka"
   manager="msmets"
   editor=""
   tags="top-support-issue"/>
<tags 
   ms.service="cloud-services"
   ms.date="10/14/2015"
   wacn.date="11/12/2015" />

# 云服务 Web 角色/辅助角色的默认 TEMP 文件夹大小太小。

云服务辅助角色或 Web 角色的默认临时目录的最大大小为 100 MB，可能会导致该目录在某个时候被填满。本文介绍如何避免临时目录空间不足。

>[AZURE.NOTE]这仅适用于使用 Web 角色和辅助角色的 Azure SDK 1.0 到 SDK 1.4。

## 与 Azure 客户支持联系

如果你对本文中的任何观点存在疑问，可以联系 [MSDN Azure 和CSDN论坛](/support/forums/)上的 Azure 专家。

或者，你也可以提出 Azure 支持事件。转至 [Azure 支持站点](/support/)并单击“获取支持”。有关使用 Azure 支持的信息，请阅读 [Azure 支持常见问题](/support/faq/)。


## 为什么空间会不足？
标准 Windows 环境变量 TEMP 和 TMP 可供在应用程序中运行的代码使用。TEMP 和 TMP 都指向一个最大大小为 100 MB 的目录。此目录中存储的任何数据都不会贯穿托管服务的整个生命周期；如果某一托管服务中的角色实例被回收，则会清除此目录。

## 解决此问题的建议
实现以下备选方案之一：

- 配置本地存储资源，直接对其进行访问而不使用 **TEMP** 或 **TMP**。若要通过应用程序内运行的代码访问本地存储资源，请调用 [RoleEnvironment.GetLocalResource](https://msdn.microsoft.com/zh-cn/library/microsoft.windowsazure.serviceruntime.roleenvironment.getlocalresource.aspx) 方法。有关如何设置本地存储资源的详细信息，请参阅[配置本地存储资源](/documentation/articles/cloud-services-configure-local-storage-resources)。

- 配置本地存储资源，将 TEMP 和 TMP 目录指向本地存储资源的路径。应在 [RoleEntryPoint.OnStart](https://msdn.microsoft.com/zh-cn/library/microsoft.windowsazure.serviceruntime.roleentrypoint.onstart.aspx) 方法中进行这种修改。

下面的代码示例演示了如何在 OnStart 方法中修改 TEMP 和 TMP 的目标目录：


```csharp
using System;
using Microsoft.WindowsAzure.ServiceRuntime;

namespace WorkerRole1
{
    public class WorkerRole : RoleEntryPoint
    {
        public override bool OnStart()
        {
            // The local resource declaration must have been added to the
            // service definition file for the role named WorkerRole1:
            //
            // <LocalResources>
            //    <LocalStorage name="CustomTempLocalStore" 
            //                  cleanOnRoleRecycle="false" 
            //                  sizeInMB="1024" />
            // </LocalResources>
            
            string customTempLocalResourcePath = 
            RoleEnvironment.GetLocalResource("CustomTempLocalStore").RootPath;
            Environment.SetEnvironmentVariable("TMP", customTempLocalResourcePath);
            Environment.SetEnvironmentVariable("TEMP", customTempLocalResourcePath);
            
            // The rest of your startup code goes here…
            
            return base.OnStart();
        }
    }
}
```

## 后续步骤

查看更多针对云服务的[故障排除文章](https://azure.microsoft.com/zh-cn/documentation/articles/?tag=top-support-issue&service=cloud-services?tag=top-support-issue&service=cloud-services)。

<!---HONumber=79-->