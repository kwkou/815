
若要诊断 Azure 云服务的问题，需要在问题发生时收集虚拟机上该服务的日志文件。可以使用 AzureLogCollector 扩展按需从一个或多个云服务 VM（通过 Web 角色和辅助角色）执行一次性日志收集，并将收集到的文件传输到 Azure 存储帐户 - 所有这些操作都无需远程登录到任何 VM。
> [AZURE.NOTE] 大多数日志信息的说明可以在 http://blogs.msdn.com/b/kwill/archive/2013/08/09/windows-azure-paas-compute-diagnostics-data.asp 中找到。

有两种模式的收集，其具体使用取决于要收集的文件的类型。
- 仅 Azure 来宾代理日志 (GA)。此收集模式包括与 Azure 来宾代理以及其他 Azure 组件相关的所有日志。
- 所有日志（完整）。此收集模式将收集 GA 模式下的所有文件以及：

  - 系统和应用程序事件日志
  
  - HTTP 错误日志
  
  - IIS Logs
  
  - 安装日志
  
  - 其他系统日志

在两种收集模式下，均可使用以下结构的集合来指定额外的数据收集文件夹：

- **Name**：集合的名称，将用作要收集的 zip 文件中子文件夹的名称。

- **Location**：虚拟机中用于收集文件的文件夹的路径。

- **SearchPattern**：要收集的文件名的样式。默认值为“*”

- **Recursive**：如果要在文件夹中以递归方式收集文件。
