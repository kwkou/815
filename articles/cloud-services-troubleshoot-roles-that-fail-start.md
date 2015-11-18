<properties 
   pageTitle="启动失败的角色的疑难解答 | Windows Azure"
   description="以下是云服务角色无法启动的部分常见原因。此外还提供了这些问题的解决方案。"
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

# 对无法启动的云服务角色进行故障诊断的常见步骤

以下是一些与无法启动的 Azure 云服务角色相关的常见问题和解决方案。

## 与 Azure 客户支持联系

如果你对本文中的任何观点存在疑问，可以联系 [MSDN Azure 和堆栈溢出论坛](/support/forums/)上的 Azure 专家。

或者，你也可以提出 Azure 支持事件。转至 [Azure 支持站点](/support/options/)并单击“获取支持”。有关使用 Azure 支持的信息，请阅读 [Windows Azure 支持常见问题](/support/faq/)。


## 缺少 DLL 或依赖项
DLL 或程序集丢失可能导致出现不响应的角色以及在“正在初始化”、“忙”、“正在停止”状态之间循环的角色。

**症状：**DLL 或程序集丢失的症状可能为：

- 角色实例在“正在初始化/忙/正在停止”之间循环。
- 角色实例已转为“就绪”状态，但在导航到 Web 应用程序时未显示相应页面

解决方案：若要调查这些问题，可采用三种推荐的方法。

## 在 Web 角色中诊断丢失 DLL 的问题

当你导航到在 Web 角色中部署的网站，且浏览器显示类似于下面的服务器错误时：

!['/' 应用程序中出现服务器错误。](./media/cloud-services-troubleshoot-roles-that-fail-start/ic503388.png)

## 通过关闭自定义错误来诊断问题

可通过将 Web 角色的 web.config 配置为将自定义错误模式设置为“关闭”并重新部署服务，来查看更完整的错误。

若要在不使用远程桌面的情况下查看更完整的错误，请执行以下操作：

1. 在 Visual Studio 中打开解决方案。

2. 在“解决方案资源管理器”中，找到 web.config 文件并将其打开。

3. 在 web.config 文件中，找到 system.web 部分并添加以下行：

    ```xml
    <customErrors mode="Off" />
    ```

4. 保存文件。

5. 重新打包并重新部署服务。

重新部署服务后，你将看到下面的错误，其中包含丢失的程序集或 DLL 的名称。

## 通过远程查看错误来诊断问题

可使用远程桌面来访问角色并远程查看更为完整的错误。请通过以下步骤使用远程桌面来查看错误：

1. 确保安装了 Azure SDK 1.3 或更高版本。

2. 在使用 Visual Studio 部署解决方案的过程中，请选择“配置远程桌面连接…”。有关配置远程桌面连接的详细信息，请参阅[将远程桌面与 Azure 角色一起使用](https://msdn.microsoft.com/zh-cn/library/gg443832.aspx)。

3. 在 Windows Azure 管理门户中，当实例显示“就绪”状态时，请单击其中一个角色实例。

4. 单击功能区“远程访问”区域中的“连接”图标。

5. 使用进行远程桌面配置时指定的凭据登录到虚拟机。

6. 打开命令提示符。

7. 键入 `IPconfig`。

8. 记录 IPV4 地址值。

9. 打开 Internet Explorer。

10. 键入 Web 应用程序的地址和名称。例如，`http://<IPV4 Address>/default.aspx`。

导航到网站将返回更明确的错误消息。

* '/' 应用程序中出现服务器错误

* 说明：执行当前 Web 请求期间，出现未处理的异常。请检查堆栈跟踪信息，以了解有关该错误以及代码中导致错误的出处的详细信息。

* 异常详细信息：System.IO.FIleNotFoundException：未能加载文件或程序集“Microsoft.WindowsAzure.StorageClient, Version=1.1.0.0, Culture=neutral, PublicKeyToken=31bf856ad364e35”或它的某一个依赖项。系统找不到指定的文件。

例如：

!['/' 应用程序中出现显式服务器错误](./media/cloud-services-troubleshoot-roles-that-fail-start/ic503389.png)

## 使用计算模拟器诊断问题

你可以使用 Azure Windows Azure 计算模拟器来诊断丢失依赖项和出现 web.config 错误的问题，并纠正这些问题。

为了在使用此诊断方法时获得最佳结果，你应使用包含 Windows 的干净安装的计算机或虚拟机。若要以最佳效果模拟 Azure 环境，应使用 Windows Server 2008 R2 x64。

1. 安装独立版本的 [Azure SDK](/downloads)

2. 在开发计算机上生成云服务项目。

3. 在 Windows 资源管理器中，导航到云服务项目的 bin\\debug 文件夹。

4. 将 .csx 文件夹和 .cscfg 文件复制到你用来调试问题的计算机。

5. 在干净的计算机上打开 Azure SDK 命令提示符并键入 `csrun.exe /devstore:start`。

6. 在命令提示符处键入 `run csrun <path to .csx folder> <path to .cscfg file> /launchBrowser`。

7. 角色启动后，你便会在 Internet Explorer 中看到详细的错误信息。你还可使用标准的 Windows 故障排除工具来进一步诊断问题。

## 使用 IntelliTrace 诊断问题

对于使用 .NET Framework 4 的辅助角色和 Web 角色，你可以使用 [Microsoft Visual Studio Ultimate](https://www.visualstudio.com/products/visual-studio-ultimate-with-MSDN-vs) 中提供的 [IntelliTrace](https://msdn.microsoft.com/zh-cn/library/dd264915.aspx)。

请按照以下步骤操作来部署启用了 IntelliTrace 的服务：

1. 确认安装了 Azure SDK 1.3 或更高版本。

2. 部署使用 Visual Studio 的解决方案。在部署期间，请选中“为 .NET 4 角色启用 IntelliTrace”复选框。

3. 实例启动后，打开“服务器资源管理器”。

4. 展开 **Azure\\Cloud Services** 节点并查找部署。

5. 展开部署，直至看到角色实例。右键单击其中一个实例。

6. 选择“查看 IntelliTrace 日志”。此时会打开“IntelliTrace 摘要”。

7. 查找摘要的异常部分。如果存在异常，则会将其标记为“异常数据”。

8. 展开“异常数据”并查找类似如下内容的 **System.IO.FileNotFoundException** 错误：

![异常数据，缺少文件或程序集](./media/cloud-services-troubleshoot-roles-that-fail-start/ic503390.png)

## 解决丢失 DLL 和程序集的问题

若要纠正丢失 DLL 和程序集错误，请按照以下步骤进行操作：

1. 在 Visual Studio 中打开解决方案。

2. 在“解决方案资源管理器”中，打开 **References** 文件夹。

3. 单击错误中标识的程序集。

4. 在“属性”窗格中，找到“复制本地”属性并将值设置为 **True**。

5. 重新部署托管服务。

一旦确认所有错误已更正，则可在不选中“为 .NET 4 角色启用 IntelliTrace”设置的情况下部署服务。

## 后续步骤

查看更多针对云服务的[故障排除文章](https://azure.microsoft.com/zh-cn/documentation/articles/?tag=top-support-issue&service=cloud-services?tag=top-support-issue&service=cloud-services)。

<!---HONumber=79-->