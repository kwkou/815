<properties 
	pageTitle="以编程方式监视流分析的作业 | Azure" 
	description="了解如何以编程方式监视通过 REST API、Azure SDK 或 Powershell 创建的流分析作业。"
	keywords=".net 监视器, 作业监视器, 监视应用"
	services="stream-analytics" 
	documentationCenter="" 
	authors="jeffstokes72" 
	manager="paulettm" 
	editor="cgronlun"/>

<tags 
	ms.service="stream-analytics" 
	ms.date="05/03/2016" 
	wacn.date="06/20/2016"/>


# 以编程方式创建流分析作业监视器
 本文说明如何为流分析作业启用监视功能。通过 REST API、Azure SDK 或 Powershell 创建的流分析作业并不默认启用监视功能。你可以在 Azure 门户中手动启用此功能，只需导航到作业的“监视”页并单击“启用”按钮即可；你也可以按本文所述步骤自动化此过程。流分析作业的监视数据将显示在 Azure 门户的“监视”选项卡中。

![作业监视器作业选项卡](./media/stream-analytics-monitor-jobs/stream-analytics-monitor-jobs-tab.png)

## 先决条件
在开始阅读本文前，你必须具有：

- Visual Studio 2012 或 2013。
- 下载和安装 [Azure .NET SDK](/downloads/)。
- 需要启用监视功能的现有流分析作业。

## 设置项目

1.	创建 Visual Studio C# .Net 控制台应用程序。
2.	在程序包管理器控制台中运行以下命令以安装 NuGet 包。第一个是 Azure 流分析管理 .NET SDK。第二个是 Azure Insights SDK，用于启用监视功能。最后一个是用于进行身份验证的 Azure Active Directory 客户端。

    ```
    Install-Package Microsoft.Azure.Management.StreamAnalytics
    Install-Package Microsoft.Azure.Insights -Pre
    Install-Package Microsoft.IdentityModel.Clients.ActiveDirectory
    ```

3.	将下面的 appSettings 部分添加到 App.config 文件。

    ```
    <appSettings>
    	<!--CSM Prod related values-->
    	<add key="ResourceGroupName" value="RESOURCE GROUP NAME" />
    	<add key="JobName" value="YOUR JOB NAME" />
    	<add key="StorageAccountName" value="YOUR STORAGE ACCOUNT"/>
    	<add key="ActiveDirectoryEndpoint" value="https://login.chinacloudapi.cn/" />
    	<add key="ResourceManagerEndpoint" value="https://management.azure.com/" />
    	<add key="WindowsManagementUri" value="https://management.core.chinacloudapi.cn/" />
    	<add key="AsaClientId" value="1950a258-227b-4e31-a9cf-717495945fc2" />
    	<add key="RedirectUri" value="urn:ietf:wg:oauth:2.0:oob" />
    	<add key="SubscriptionId" value="YOUR AZURE SUBSCRIPTION ID" />
    	<add key="ActiveDirectoryTenantId" value="YOUR TENANT ID" />
    </appSettings>
	```
将 *SubscriptionId* 和 *ActiveDirectoryTenantId* 的值替换为你的 Azure 订阅和租户 ID。你可以通过运行以下 PowerShell cmdlet 来获取这些值：

    ```
    Get-AzureAccount
    ```
4.	将以下 using 语句添加到项目中的源文件 (Program.cs)。 

    ```
        using System;
        using System.Configuration;
        using System.Threading;
        using Microsoft.Azure;
        using Microsoft.Azure.Management.Insights;
        using Microsoft.Azure.Management.Insights.Models;
        using Microsoft.Azure.Management.StreamAnalytics;
        using Microsoft.Azure.Management.StreamAnalytics.Models;
        using Microsoft.IdentityModel.Clients.ActiveDirectory;
    ```
5.	添加一个身份验证帮助器方法。

        public static string GetAuthorizationHeader()
        	{
        		AuthenticationResult result = null;
        		var thread = new Thread(() =>
        		{
        			try
        			{
            			var context = new AuthenticationContext(
                			ConfigurationManager.AppSettings["ActiveDirectoryEndpoint"] +
                			ConfigurationManager.AppSettings["ActiveDirectoryTenantId"]);

            			result = context.AcquireToken(
                			resource: ConfigurationManager.AppSettings["WindowsManagementUri"],
                			clientId: ConfigurationManager.AppSettings["AsaClientId"],
                			redirectUri: new Uri(ConfigurationManager.AppSettings["RedirectUri"]),
                			promptBehavior: PromptBehavior.Always);
        			}
        			catch (Exception threadEx)
        			{
            			Console.WriteLine(threadEx.Message);
        			}
    			});

    			thread.SetApartmentState(ApartmentState.STA);
    			thread.Name = "AcquireTokenThread";
    			thread.Start();
    			thread.Join();

    			if (result != null)
    			{
        			return result.AccessToken;
    			}

    			throw new InvalidOperationException("Failed to acquire token");
        }

## 创建管理客户端
以下代码将设置必需变量和管理客户端。

    string resourceGroupName = "<YOUR AZURE RESOURCE GROUP NAME>";
    string streamAnalyticsJobName = "<YOUR STREAM ANALYTICS JOB NAME>";

    // Get authentication token
    TokenCloudCredentials aadTokenCredentials =
    	new TokenCloudCredentials(
    		ConfigurationManager.AppSettings["SubscriptionId"],
    		GetAuthorizationHeader());

    Uri resourceManagerUri = new
    Uri(ConfigurationManager.AppSettings["ResourceManagerEndpoint"]);

    // Create Stream Analytics and Insights management client
    StreamAnalyticsManagementClient streamAnalyticsClient = new
    StreamAnalyticsManagementClient(aadTokenCredentials, resourceManagerUri);
    InsightsManagementClient insightsClient = new
    InsightsManagementClient(aadTokenCredentials, resourceManagerUri);

## 为现有流分析作业启用监视功能

以下代码将为 **现有** 流分析作业启用监视功能。代码的第一部分针对流分析服务执行 GET 请求，目的是检索特定流分析作业的信息。它使用“Id”属性（从 GET 请求检索而得）作为代码第二部分中 Put 方法的参数，目的是将 PUT 请求发送到 Insights 服务以启用流分析作业的监视功能。

> [AZURE.WARNING]
> 如果你此前为其他流分析作业启用了监视功能，不管是通过 Azure 门户进行的还是通过以下代码以编程方式完成的，**我们都建议你在提供存储帐户名称时提供你此前在启用监视功能时所使用的那个相同的存储帐户名称。**
> 
> 存储帐户与你创建流分析作业时所在的区域相关联，并不特定于作业本身。
> 
> 该区域的所有流分析作业（以及所有其他的 Azure 资源）在存储监视数据时将共享这个存储帐户。如果你提供其他的存储帐户，则可能会产生意想不到的副作用，影响你监视其他流分析作业和/或其他 Azure 资源。
> 
> 用于替换下面的 ```“<YOUR STORAGE ACCOUNT NAME>”``` 的存储帐户名称所代表的存储帐户应该与要启用监视功能的流分析作业属同一订阅。

    // Get an existing Stream Analytics job
    JobGetParameters jobGetParameters = new JobGetParameters()
    {
    	PropertiesToExpand = "inputs,transformation,outputs"
    };
    JobGetResponse jobGetResponse = streamAnalyticsClient.StreamingJobs.Get(resourceGroupName, streamAnalyticsJobName, jobGetParameters);

    // Enable monitoring
    ServiceDiagnosticSettingsPutParameters insightPutParameters = new ServiceDiagnosticSettingsPutParameters()
    {
    		Properties = new ServiceDiagnosticSettings()
    		{
        		StorageAccountName = "<YOUR STORAGE ACCOUNT NAME>"
    		}
    };
    insightsClient.ServiceDiagnosticSettingsOperations.Put(jobGetResponse.Job.Id, insightPutParameters);



## 获取支持
如需进一步的帮助，请尝试我们的 [Azure 流分析论坛](https://social.msdn.microsoft.com/Forums/zh-CN/home?forum=AzureStreamAnalytics)。


## 后续步骤

- [Azure 流分析简介](/documentation/articles/stream-analytics-introduction/)
- [Azure 流分析入门](/documentation/articles/stream-analytics-get-started/)
- [缩放 Azure 流分析作业](/documentation/articles/stream-analytics-scale-jobs/)
- [Azure 流分析查询语言参考](https://msdn.microsoft.com/zh-cn/library/azure/dn834998.aspx)
- [Azure 流分析管理 REST API 参考](https://msdn.microsoft.com/zh-cn/library/azure/dn835031.aspx)
 

<!---HONumber=Mooncake_0307_2016-->