<properties
   pageTitle="重试服务指南 | Microsoft Azure"
   description="设置重试机制的服务指南。"
   services=""
   documentationCenter="na"
   authors="dragon119"
   manager="masimms"
   editor=""
   tags=""/>

<tags
   ms.service="best-practice"
   ms.date="04/28/2015"
   wacn.date="10/3/2015"/>

# 重试服务指南

## 概述

大多数 Azure 服务和客户端 SDK 都包括重试机制。不过，这些重试机制各不相同，这是因为每个服务都有不同的特征和要求，这样一来，各个重试机制都会针对特定服务进行优化。本指南汇总了大多数 Azure 服务的重试机制功能，并介绍了如何使用、适应或扩展相应服务的重试机制。

有关如何处理临时故障、针对服务和资源重试连接和操作的一般指南，请参阅[重试指南](/documentation/articles/best-practices-retry-general)。

下表总结了本指南中介绍的 Azure 服务重试功能。

| **服务** | **重试功能** | **策略配置** | **范围** | **遥测功能** |
|---------------------------------------|-----------------------------------------|------------------------------|--------------------------------------------------|------------------------
| **[AzureStorage](#azure-storage-retry-guidelines)** | 客户端原生 | 编程 | 客户端和各项操作 | TraceSource |
| **[使用 Entity Framework 的 SQL 数据库](#sql-database-using-entity-framework-6-retry-guidelines)** | 客户端原生 | 编程 | 每个应用域均为全局 | 无 |
| **[使用 ADO.NET 的 SQL 数据库](#sql-database-using-ado-net-retry-guidelines)** | Topaz* | 声明性和编程 | 各个语句或代码块 | “自定义” |
| **[服务总线](#service-bus-retry-guidelines)** | 客户端原生 | 编程 | 命名空间管理器、消息工厂和客户端 | ETW |
| **[高速缓存](#cache-redis-retry-guidelines)** | 客户端原生 | 编程 | 客户端 | TextWriter |
| **[DocumentDB](#documentdb-pre-release-retry-guidelines)** | 服务原生 | 不可配置 | 全局 | TraceSource |
| **[搜索](#search-retry-guidelines)** | Topaz*（使用自定义检测策略） | 声明性和编程 | 代码块 | “自定义” |
| **[Active Directory](#azure-active-directory-retry-guidelines)** | Topaz*（使用自定义检测策略） | 声明性和编程 | 代码块 | “自定义” |
*Topaz 是<a href="http://msdn.microsoft.com/library/dn440719.aspx">企业库 6.0</a> 中包含的临时故障处理应用程序块的易记名称。对于大多数类型的服务，您可以结合使用自定义检测策略和 Topaz，如本指南所述。本指南末尾的[临时故障处理应用程序块 (Topaz) 策略](#transient-fault-handling-application-block-topaz-strategies)部分中介绍了 Topaz 默认策略。请注意，临时故障处理应用程序块现在是一个开放源代码框架，不受 Microsoft 直接支持。

> [AZURE.NOTE]对于大多数 Azure 内置重试机制，目前尚无方法针对不同类型的错误或异常（不局限于重试策略功能）应用不同的重试策略。因此，根据指南，最好在编写时配置可提供最佳平均性能和可用性的策略。微调策略的一种方法是分析日志文件，以确定发生的临时故障的类型。例如，如果大部分错误都与网络连接问题相关，那么您可以立即尝试重试，而不是等待很长一段时间才进行首次重试。

## Azure 存储空间重试指南

Azure 存储空间服务包括表和 blob 存储、文件以及存储队列。

### 重试机制

重试在单独的 REST 操作一级发生，是客户端 API 实现的主要组成部分。客户端存储 SDK 使用可实现 [IExtendedRetryPolicy 接口](http://msdn.microsoft.com/zh-cn/library/microsoft.windowsazure.storage.retrypolicies.iextendedretrypolicy.aspx)的类。

接口的实现各不相同。存储客户端可以从专门用于访问表、blob 和队列的策略中进行选择。每个实现使用不同的重试策略，此策略基本上定义了重试间隔和其他详细信息。

内置类支持线性（固定延迟）和指数随机化重试间隔。当其他进程在较高级别处理重试时，还有不重试策略可供使用。不过，如果您具有内置类未规定的特定要求，则可以实现您自己的重试类。

如果您使用的是读取访问异地冗余存储 (RA-GRS)，且请求的结果是可重试错误，则备用重试会在主要和辅助存储服务位置之间切换。有关详细信息，请参阅 [Azure 存储空间冗余选项](http://msdn.microsoft.com/zh-cn/library/azure/dn727290.aspx)。

### 策略配置（Azure 存储空间）

重试策略是以编程方式进行配置。典型的过程是创建并填充 **TableRequestOptions**、**BlobRequestOptions**、**FileRequestOptions** 或 **QueueRequestOptions** 实例。

```csharp
TableRequestOptions interactiveRequestOption = new TableRequestOptions()
{
  RetryPolicy = new LinearRetry(TimeSpan.FromMilliseconds(500), 3),
  // For Read-access geo-redundant storage, use PrimaryThenSecondary.
  // Otherwise set this to PrimaryOnly.
  LocationMode = LocationMode.PrimaryThenSecondary,
  // Maximum execution time based on the business use case. Maximum value up to 10 seconds.
  MaximumExecutionTime = TimeSpan.FromSeconds(2)
};
```

然后，可以在客户端上设置请求选项实例，客户端的所有操作都会使用指定的请求选项。

```csharp
client.DefaultRequestOptions = interactiveRequestOption;
var stats = await client.GetServiceStatsAsync();
```

您可以将填充的请求选项类实例作为参数传递到操作方法，以此来替代客户端请求选项。

```csharp
var stats = await client.GetServiceStatsAsync(interactiveRequestOption, operationContext: null);
```

使用 **OperationContext** 实例可以指定在发生重试时以及在操作完成时要执行的代码。此代码可以收集供日志和遥测使用的操作的相关信息。

	// Set up notifications for an operation
	var context = new OperationContext();
	context.ClientRequestID = "some request id";
	context.Retrying += (sender, args) =>
	{
	  /* Collect retry information */
	};
	context.RequestCompleted += (sender, args) =>
	{
	  /* Collect operation completion information */
	};
	var stats = await client.GetServiceStatsAsync(null, context);

除了指明故障是否适合重试，重试扩展策略还会返回 **RetryContext** 对象，用于指明重试次数、上次请求结果、下次重试是在主要位置还是在辅助位置上发生（有关详细信息，请参阅下表）。**RetryContext** 对象的属性可用于确定是否以及何时尝试进行重试。如需了解更多详情，请参阅 [IExtendedRetryPolicy.Evaluate 方法](http://msdn.microsoft.com/zh-cn/library/microsoft.windowsazure.storage.retrypolicies.iextendedretrypolicy.evaluate.aspx)。

下表展示了内置重试策略的默认设置。

| **上下文** | **设置** | **默认值** | **含义** |
|--------------------------|-------------------------------------------------------------|------------------------------------|-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| 表/Blob/文件<br />QueueRequestOptions | MaximumExecutionTime<br /><br />ServerTimeout<br /><br /><br /><br /><br />LocationMode<br /><br /><br /><br /><br /><br /><br />RetryPolicy | 120 秒<br /><br />无<br /><br /><br /><br /><br /><br /><br /><br /><br /><br /><br /><br />ExponentialPolicy | 请求的最长执行时间（包括所有可能的重试尝试）。<br />请求的服务器超时间隔（值四舍五入到秒）。如果未指定，则会使用所有服务器请求的默认值。通常，最好是忽略此设置，以便使用服务器默认值。<br />如果创建的存储帐户设置了读取访问异地冗余存储 (RA-GRS) 复制选项，则您可以使用位置模式来指明哪个位置应接收请求。例如，如果指定的是 **PrimaryThenSecondary**，则请求总是会先发送到主要位置。如果请求失败，则会发送到辅助位置。<br />有关每个选项的详细信息，请参阅下文。 |
| 指数策略 | maxAttempt<br />deltaBackoff<br /><br /><br />MinBackoff<br /><br />MaxBackoff | 3<br />4 秒<br /><br /><br />3 秒<br /><br />30 秒 | 重试尝试次数。<br />不同重试之间的回退间隔。系统会对后续重试尝试使用此时间跨度的倍数（包括随机元素）。<br />添加到所有通过 deltaBackoff 计算得出的重试间隔。不能更改此值。<br />如果计算的重试间隔大于 MaxBackoff，则会使用 MaxBackoff。不能更改此值。 |
| 线性策略 | maxAttempt<br />deltaBackoff | 3<br />30 秒 | 重试尝试次数。<br />不同重试之间的回退间隔。 |

### 重试使用指南
在使用存储客户端 API 访问 Azure 存储空间服务时，请注意以下指南：

* 使用 Microsoft.WindowsAzure.Storage.RetryPolicies 命名空间中符合您要求的内置重试策略。在大多数情况下，这些策略就够用了。
* 对批处理操作、后台任务或非交互式方案使用 **ExponentialRetry** 策略。在这种情况下，服务通常可以有更多的恢复时间。结果就是，提高了操作最终成功的可能性。
* 考虑指定 **RequestOptions** 参数的 **MaximumExecutionTime** 属性，以限制总执行时间；但在选择超时值时，请将操作的类型和大小考虑进去。
* 如果您需要实现自定义重试，请避免创建存储客户端类的包装器。应使用这些功能通过 **IExtendedRetryPolicy** 接口扩展现有策略。
* 如果您使用的是读取访问异地冗余存储 (RA-GRS)，则可以使用 **LocationMode** 指定在主要访问失败时重试尝试会访问存储空间的只读辅助副本。不过，使用此选项时，您必须确保在尚未完成从主要存储空间进行复制的情况下，您的应用程序可以成功使用可能已过时的数据。

请考虑从下列重试操作设置入手。这些都是通用设置，您应监视操作，并对值进行微调以适应您自己的方案。

| **上下文** | **示例目标 E2E<br />最长延迟** | **重试策略** | **设置** | **值** | **工作原理** |
|----------------------|-----------------------------------|------------------|-------------------------|-------------|-----------------------------------------------------------------------------|
| 交互式、UI<br />或前台 | 2 秒 | 线性 | maxAttempt<br />deltaBackoff | 3<br />500 毫秒 | 第 1 次尝试 - 延迟 500 毫秒<br />第 2 次尝试 - 延迟 500 毫秒<br />第 3 次尝试 - 延迟 500 毫秒 |
| 后台<br />或批处理 | 30 秒 | 指数 | maxAttempt<br />deltaBackoff | 5<br />4 秒 | 第 1 次尝试 - 延迟约 3 秒<br />第 2 次尝试 - 延迟约 7 秒<br />第 3 次尝试 - 延迟约 15 秒 |

## 遥测

重试尝试是记录到 **TraceSource** 中。您必须配置 **TraceListener**，才能捕获事件并将这些事件写入合适的目标日志。您可以使用 **TextWriterTraceListener** 或 **XmlWriterTraceListener** 将数据写入日志文件、使用 **EventLogTraceListener** 将数据写入 Windows 事件日志，或使用 **EventProviderTraceListener** 将跟踪数据写入 ETW 子系统。此外，您还可以配置缓冲区的自动刷新和即将记录的事件详细信息（例如，错误、警告、信息和详细内容）。有关详细信息，请参阅[使用.NET 存储客户端库的客户端日志记录](http://msdn.microsoft.com/zh-cn/library/azure/dn782839.aspx)。

操作可以接收 **OperationContext** 实例，这会公开可用于附加自定义遥测逻辑的 **Retrying** 事件。有关详细信息，请参阅 [OperationContext.Retrying 事件](http://msdn.microsoft.com/zh-cn/library/microsoft.windowsazure.storage.operationcontext.retrying.aspx)。

## 示例（Azure 存储空间）

以下代码示例展示了如何创建两个具有不同重试设置的 **TableRequestOptions** 实例；一个用于交互式请求，另一个用于后台请求。然后，此示例在客户端上设置这两个重试策略，以便它们可以针对所有请求进行应用。此示例还会对特定请求设置交互式策略，以便替代应用于客户端的默认设置。

```csharp
using System;
using System.Threading.Tasks;
using Microsoft.WindowsAzure.Storage;
using Microsoft.WindowsAzure.Storage.RetryPolicies;
using Microsoft.WindowsAzure.Storage.Table;

namespace RetryCodeSamples
{
    class AzureStorageCodeSamples
    {
        private const string connectionString = "UseDevelopmentStorage=true";

        public async static Task Samples()
        {
            var storageAccount = CloudStorageAccount.Parse(connectionString);

            TableRequestOptions interactiveRequestOption = new TableRequestOptions()
            {
                RetryPolicy = new LinearRetry(TimeSpan.FromMilliseconds(500), 3),
                // For Read-access geo-redundant storage, use PrimaryThenSecondary.
                // Otherwise set this to PrimaryOnly.
                LocationMode = LocationMode.PrimaryThenSecondary,
                // Maximum execution time based on the business use case. Maximum value up to 10 seconds.
                MaximumExecutionTime = TimeSpan.FromSeconds(2)
            };

            TableRequestOptions backgroundRequestOption = new TableRequestOptions()
            {
                // Client has a default exponential retry policy with 4 sec delay and 3 retry attempts
                // Retry delays will be approximately 3 sec, 7 sec, and 15 sec
                MaximumExecutionTime = TimeSpan.FromSeconds(30),
                // PrimaryThenSecondary in case of Read-access geo-redundant storage, else set this to PrimaryOnly
                LocationMode = LocationMode.PrimaryThenSecondary
            };

            var client = storageAccount.CreateCloudTableClient();
            // Client has a default exponential retry policy with 4 sec delay and 3 retry attempts
            // Retry delays will be approximately 3 sec, 7 sec, and 15 sec
            // ServerTimeout and MaximumExecutionTime are not set

            {
                // Set properties for the client (used on all requests unless overridden)
                // Different exponential policy parameters for background scenarios
                client.DefaultRequestOptions = backgroundRequestOption;
                // Linear policy for interactive scenarios
                client.DefaultRequestOptions = interactiveRequestOption;
            }

            {
                // set properties for a specific request
                var stats = await client.GetServiceStatsAsync(interactiveRequestOption, operationContext: null);
            }

            {
                // Set up notifications for an operation
                var context = new OperationContext();
                context.ClientRequestID = "some request id";
                context.Retrying += (sender, args) =>
                {
                    /* Collect retry information */
                };
                context.RequestCompleted += (sender, args) =>
                {
                    /* Collect operation completion information */
                };
                var stats = await client.GetServiceStatsAsync(null, context);
            }
        }
    }
}
```

## 详细信息

- [Azure 存储空间客户端库重试策略建议](http://azure.microsoft.com/blog/2014/05/22/azure-storage-client-library-retry-policy-recommendations/)
- [存储客户端库 2.0 - 实现重试策略](http://gauravmantri.com/2012/12/30/storage-client-library-2-0-implementing-retry-policies/)

## 使用 Entity Framework 6 的 SQL 数据库重试指南

SQL 数据库是一种托管的 SQL 数据库，具有各种大小，可作为标准（共享）和高级（非共享）服务提供。Entity Framework 是一种对象关系映射程序，可方便 .NET 开发人员使用域特定对象处理关系数据。开发人员无需再像往常一样编写大部分数据访问代码。

## 重试机制

在通过名为[连接复原/重试逻辑](http://msdn.microsoft.com/data/dn456835.aspx)的机制访问使用 Entity Framework 6.0 及更高版本的 SQL 数据库时，提供重试支持。有关完整规范，请参阅 Codeplex 上的 [.NET Entity Framework wiki](https://entityframework.codeplex.com/wikipage?title=Connection%20Resiliency%20Spec) 文章。重试机制的主要功能包括：

* 主要抽象是 **IDbExecutionStrategy** 接口。此接口：
  * 定义同步和异步 **Execute*** 方法。
  * 定义可直接使用或可在数据库上下文中配置为默认策略的类（映射到提供程序名称或提供程序名称和服务器名称）。如果已在上下文中进行配置，则重试会在各个数据库操作级别发生，其中一些可能适用于给定的上下文操作。
  * 定义何时以及如何重试失败的连接。
* 它包括 **IDbExecutionStrategy** 接口的多个内置实现：
  * 默认 - 不重试。
  * SQL 数据库默认 - 不重试（自动），但检查异常，并根据建议包装它们以使用 SQL 数据库策略。
  * SQL 数据库默认 - 指数（从基类继承），外加 SQL 数据库检测逻辑。
* 它实现了包含随机化的指数回退策略。
* 内置重试类有状态，但非线程安全。不过，可以在当前操作完成后重用它们。
* 如果超出指定的重试计数，则结果会在新的异常中进行包装。它不会加剧当前异常。

## 策略配置（使用 Entity Framework 6 的 SQL 数据库）

在访问使用 Entity Framework 6.0 及更高版本的 SQL 数据库时，提供重试支持。重试策略是以编程方式进行配置。不能逐个操作更改配置。

在上下文中将策略配置为默认值时，您需要指定一个用于根据需要新建策略的函数。以下代码展示了如何创建重试配置类来扩展 **DbConfiguration** 基类。

```csharp
public class BloggingContextConfiguration : DbConfiguration
{
  public BlogConfiguration()
  {
    // Set up the execution strategy for SQL ??? (exponential) with 5 retries and 4 sec delay
    this.SetExecutionStrategy(
         "System.Data.SqlClient", () => new SqlAzureExecutionStrategy(5, TimeSpan.FromSeconds(4)));
  }
}
```

然后，您可以将此指定为应用程序启动时，所有使用 **DbConfiguration** 实例的 **SetConfiguration** 方法的操作的默认重试策略。默认情况下，EF 会自动发现和使用配置类。

	DbConfiguration.SetConfiguration(new BloggingContextConfiguration());

您可以使用 **DbConfigurationType** 属性为上下文类添加批注，从而为上下文指定重试配置类。不过，如果您只有一个配置类，则 EF 会使用它，而无需为上下文添加批注。

	[DbConfigurationType(typeof(BloggingContextConfiguration))]
	public class BloggingContext : DbContext
	{ ...

如果您需要对特定操作使用不同的重试策略，或对特定操作禁用重试，则可以创建配置类，以便您能够在 **CallContext** 中设置标志，从而挂起或交换策略。配置类可以使用此标志切换策略，或禁用您提供的策略并使用默认策略。有关详细信息，请参阅“重试执行策略限制（EF6 及更高版本）”页面中的[挂起执行策略](http://msdn.microsoft.com/dn307226#transactions_workarounds)。

对各个操作使用特定重试策略的另一种方法是，创建相应策略类的实例，并通过参数提供所需的设置。然后，您需要调用它的 **ExecuteAsync** 方法。

	var executionStrategy = new SqlAzureExecutionStrategy(5, TimeSpan.FromSeconds(4));
	var blogs = await executionStrategy.ExecuteAsync(
	    async () =>
	    {
	        using (var db = new BloggingContext("Blogs"))
	        {
	            // Acquire some values asynchronously and return them
	        }
	    },
	    new CancellationToken()
	);

使用 **DbConfiguration** 类的最简单方法是在与 **DbContext** 类相同的程序集中中找到它。不过，如果需要在不同的方案（如使用不同的交互式重试策略和后台重试策略）中使用相同的上下文，则这样做并不适合。如果不同的上下文在各个应用域中执行，则您可以通过内置的支持在配置文件中指定配置类，也可以使用代码对它进行明确设置。如果不同的上下文必须在同一应用域中执行，则需要使用自定义解决方案。

有关详细信息，请参阅[基于代码的配置（EF6 及更高版本）](http://msdn.microsoft.com/data/jj680699.aspx)。

下表显示了使用 EF6 时的内置重试策略的默认设置。

![](media/best-practices-retry-service-specific/RetryServiceSpecificGuidanceTable4.png)
## 重试使用指南

访问使用 EF6 的 SQL 数据库时，请注意以下指南：

* 选择适当的服务选项（共享或高级）。鉴于共享服务器的其他租户的使用情况，共享实例的连接延迟和限制可能比平常要长。如果需要可预测的性能和可靠的低延迟操作，请考虑选择高级选项。
* 不建议结合使用固定间隔策略和 Azure SQL 数据库。请改用指数回退策略，因为服务可能已过载，更长的延迟则可以让服务有更长时间进行恢复。
* 定义连接时，为连接和命令超时选择合适的值。根据业务逻辑设计和全面测试来选择超时值。您可能需要随着数据量或业务流程的变化，陆续修改此值。当数据库忙碌时，超时太短可能会导致连接过早失败。超时太长则会导致在检测到失败的连接前等待太久，进而可能会妨碍重试逻辑正常运行。超时值是端到端延迟的组成部分，尽管您无法轻松确定在保存上下文时将执行多少命令。您可以设置 **DbContext** 实例的 **CommandTimeout** 属性，以此来更改默认超时值。
* Entity Framework 支持在配置文件中定义的重试配置。不过，为了实现 Azure 上的最大灵活性，您应考虑在应用程序内以编程方式创建配置。重试策略的特定参数（如重试次数和重试间隔）可以存储在服务配置文件中，并在运行时用于创建相应的策略。这样一来，您无需重启应用程序，即可更改设置。

请考虑从下列重试操作设置入手。您无法指定重试尝试之间的延迟（它是固定的，且生成为指数序列）。您只能指定最大值（如下所示）；除非您创建自定义重试策略。这些都是通用设置，您应监视操作，并对值进行微调以适应您自己的方案。

| **上下文** | **示例目标 E2E<br />最长延迟** | **重试策略** | **设置** | **值** | **工作原理** |
|----------------------|-----------------------------------|--------------------|------------------------|--------------|-----------------------------------------------------------------------------------------------------------------------------|
| 交互式、UI<br />或前台 | 2 秒 | 指数 | MaxRetryCount<br />MaxDelay | 3<br />750 毫秒 | 第 1 次尝试 - 延迟 0 秒<br />第 2 次尝试 - 延迟 750 毫秒<br />第 3 次尝试 - 延迟 750 毫秒 |
| 后台<br />或批处理 | 30 秒 | 指数 | MaxRetryCount<br />MaxDelay | 5<br />12 秒 | 第 1 次尝试 - 延迟 0 秒<br />第 2 次尝试 - 延迟约 1 秒<br />第 3 次尝试 - 延迟约 3 秒<br />第 4 次尝试 - 延迟约 7 秒<br />第 5 次尝试 - 延迟 12 秒 |

> [AZURE.NOTE]端到端延迟目标假设采用服务连接的默认超时。如果您指定更长的连接超时，则每次重试尝试都会将端到端延迟延长这一附加时间。

## 示例（使用 Entity Framework 6 的 SQL 数据库）

以下代码示例定义了使用 Entity Framework 的简单数据访问解决方案。它通过定义可扩展 **DbConfiguration** 的 **BlogConfiguration** 类的实例来指定重试策略。

```csharp
using System;
using System.Collections.Generic;
using System.Data.Entity;
using System.Data.Entity.SqlServer;
using System.Threading.Tasks;

namespace RetryCodeSamples
{
	public class BlogConfiguration : DbConfiguration
	{
	    public BlogConfiguration()
	    {
	        // Set up the execution strategy for SQL ??? (exponential) with 5 retries and 12 sec delay.
	        // These values could be loaded from configuration rather than being hard-coded.
	        this.SetExecutionStrategy(
	                "System.Data.SqlClient", () => new SqlAzureExecutionStrategy(5, TimeSpan.FromSeconds(12)));
	    }
	}

	// Specify the configuration type if more than one has been defined.
	// [DbConfigurationType(typeof(BlogConfiguration))]
	public class BloggingContext : DbContext
	{
	    // Definition of content goes here.
	}

	class EF6CodeSamples
	{
	    public async static Task Samples()
	    {
	        // Execution strategy configured by DbConfiguration subclass, discovered automatically or
	        // or explicitly indicated through configuration or with an attribute. Default is no retries.
	        using (var db = new BloggingContext("Blogs"))
	        {
	            // Add, edit, delete blog items here, then:
	            await db.SaveChangesAsync();
	        }
	    }
	}
}
```

有关使用 Entity Framework 重试机制的更多示例，请参阅[连接复原/重试逻辑](http://msdn.microsoft.com/data/dn456835.aspx)。

## 详细信息

* [Azure SQL 数据库性能和弹性指南](http://social.technet.microsoft.com/wiki/contents/articles/3507.windows-azure-sql-database-performance-and-elasticity-guide.aspx)

## 使用 ADO.NET 的 SQL 数据库重试指南

SQL 数据库是一种托管的 SQL 数据库，具有各种大小，可作为标准（共享）和高级（非共享）服务提供。

### 重试机制

访问使用 ADO.NET 的 SQL 数据库时，其中没有内置重试支持。不过，请求的返回代码可用于确定请求失败原因。[Azure SQL 数据库限制](http://msdn.microsoft.com/zh-cn/library/dn338079.aspx)页面解释了限制如何阻止连接、特定情况的返回代码以及如何处理这些情况和重试操作。

您可以结合使用临时故障处理应用程序块 (Topaz) 和 Nuget 包 EnterpriseLibrary.TransientFaultHandling.Data（类 **SqlAzureTransientErrorDetectionStrategy**），从而实现 SQL 数据库的重试机制。

临时故障处理应用程序块还提供 **ReliableSqlConnection** 类，此类可以实现旧版 ADO.NET 1.0 API（**IDbConnection**，而不是 **DbConnection**），并在内部执行重试和连接管理。虽然操作起来很方便，但您需要使用不同的方法集通过重试调用操作，因此不能直接替代。它不支持异步执行，而我们建议您在实现和使用 Azure 服务时进行异步执行。此外，由于此类使用 ADO.NET 1.0，因此它无法受益于 ADO.NET 的最新改进和更新。

### 策略配置（使用 ADO.NET 的 SQL 数据库）

临时故障处理应用程序块既支持基于文件的配置，也支持以编程方式进行配置。通常情况下，为了实现最大灵活性，应以编程方式进行配置（有关详细信息，请参阅以下部分的备注）。以下代码（在应用程序启动时会执行一次）使用适合与 Azure SQL 数据库结合使用的四个重试策略的列表，创建并填充 **RetryManager**。它还设置 **RetryManager** 的默认策略。这些是在未指定替代策略（在创建连接或命令时）的情况下，用于连接或命令的策略。

```csharp
RetryManager.SetDefault(new RetryManager(
	new List<RetryStrategy> { new ExponentialBackoff(name: "default", retryCount: 3,
	                                                minBackoff: 	TimeSpan.FromMilliseconds(100),
	                                                maxBackoff: 	TimeSpan.FromSeconds(30),
	                                                deltaBackoff: 	TimeSpan.FromSeconds(1),
	                                                firstFastRetry: true),
	                        new ExponentialBackoff(name: "default sql connection", retryCount: 3,
	                                                minBackoff: 	TimeSpan.FromMilliseconds(100),
	                                                maxBackoff: 	TimeSpan.FromSeconds(30),
	                                                deltaBackoff: 	TimeSpan.FromSeconds(1),
	                                                firstFastRetry: true),
	                        new ExponentialBackoff(name: "default sql command", retryCount: 3,
	                                                minBackoff: 	TimeSpan.FromMilliseconds(100),
	                                                maxBackoff: 	TimeSpan.FromSeconds(30),
	                                                deltaBackoff: 	TimeSpan.FromSeconds(1),
	                                                firstFastRetry: true),
	                        new ExponentialBackoff(name: "alt sql", retryCount: 5,
	                                                minBackoff: 	TimeSpan.FromMilliseconds(100),
	                                                maxBackoff: 	TimeSpan.FromSeconds(30),
	                                                deltaBackoff: 	TimeSpan.FromSeconds(1),
	                                                firstFastRetry: true), },
	"default",
	new Dictionary<string, string> {
	    {
	    RetryManagerSqlExtensions.DefaultStrategyConnectionTechnologyName, "default sql connection"
	    },
	    {
	    RetryManagerSqlExtensions.DefaultStrategyCommandTechnologyName, "default sql command"}
	    }));
```

若要了解如何在访问 Azure SQL 数据库时使用已配置的重试策略，请参阅以下[示例](#examples-sql-database-using-ado-net-)部分。

本指南末尾的[临时故障处理应用程序块 (Topaz) 策略](#transient-fault-handling-application-block-topaz-strategies)部分中介绍了临时故障处理应用程序块的默认策略。

### 重试使用指南

访问使用 ADO.NET 的 SQL 数据库时，请注意以下指南：

* 选择适当的服务选项（共享或高级）。鉴于共享服务器的其他租户的使用情况，共享实例的连接延迟和限制可能比平常要长。如果需要更加可预测的性能和可靠的低延迟操作，请考虑选择高级选项。
* 请务必在适当的级别或作用域执行重试，以避免出现导致数据不一致的非幂等操作。理想情况下，所有操作都应是幂等操作，这样就可以重复执行，而不会导致不一致。如果并非如此，应在符合以下条件的级别或作用域执行重试：在一个操作失败时允许撤消所有相关更改；例如，从事务作用域内。有关详细信息，请参阅[云服务基础数据访问层 - 临时故障处理](http://social.technet.microsoft.com/wiki/contents/articles/18665.cloud-service-fundamentals-data-access-layer-transient-fault-handling.aspx#Idempotent_Guarantee)。
* 不建议结合使用固定间隔策略和 Azure SQL 数据库（交互式方案除外，在此类方案中，仅有几次重试的间隔非常短）。对于大多数方案，请考虑使用指数回退策略。
* 定义连接时，为连接和命令超时选择合适的值。当数据库忙碌时，超时太短可能会导致连接过早失败。超时太长则会导致在检测到失败的连接前等待太久，进而可能会妨碍重试逻辑正常运行。超时值是端到端延迟的组成部分；它可以有效地增加每次重试尝试的重试策略中指定的重试延迟。
* 重试一定次数后关闭连接（即使使用指数回退重试逻辑，也是如此），然后在新连接上重试操作。在同一个连接上多次重试相同的操作可能是导致连接问题出现的一个因素。有关此技术的示例，请参阅[云服务基础数据访问层 - 临时故障处理](http://social.technet.microsoft.com/wiki/contents/articles/18665.cloud-service-fundamentals-data-access-layer-transient-fault-handling.aspx)。
* 当连接池正在使用中（默认情况）时，可能会从池中选择相同的连接，即使在关闭并重新打开连接后，也可能会出现这样的情况。如果遇到这种情况，解决方法是调用 **SqlConnection** 类的 **ClearPool** 方法，将连接标记为不可重用。不过，仅在几次连接尝试均失败，且遇到与失败的连接相关的某类临时故障（如 SQL 超时（错误代码 -2））时，才能这样做。
* 如果数据访问代码使用作为 **TransactionScope** 实例启动的事务，则重试逻辑应重新打开连接，并启动新的事务作用域。因此，可重试代码块应包含事务的整个作用域。
* 临时故障处理应用程序块支持完全在配置文件中定义重试配置。不过，为了实现 Azure 上的最大灵活性，您应考虑在应用程序内以编程方式创建配置。重试策略的特定参数（如重试次数和重试间隔）可以存储在服务配置文件中，并在运行时用于创建相应的策略。这样一来，您无需重启应用程序，即可更改设置。

请考虑从下列重试操作设置入手。这些都是通用设置，您应监视操作，并对值进行微调以适应您自己的方案。

| **上下文** | **示例目标 E2E<br />最长延迟** | **重试策略** | **设置** | **值** | **工作原理** |
|----------------------|-----------------------------------|--------------------|-----------------------------------------------------------------------|----------------------------|-------------------------------------------------------------------------------------------------------------------------------|
| 交互式、UI<br />或前台 | 2 秒 | FixedInterval | 重试计数<br />重试间隔<br />首次快速重试 | 3<br />500 毫秒<br />true | 第 1 次尝试 - 延迟 0 秒<br />第 2 次尝试 - 延迟 500 毫秒<br />第 3 次尝试 - 延迟 500 毫秒 |
| 后台<br />或批处理 | 30 秒 | ExponentialBackoff | 重试计数<br />最小回退<br />最大回退<br />增量回退<br />首次快速重试 | 5<br />0 秒<br />60 秒<br />2 秒<br />false | 第 1 次尝试 - 延迟 0 秒<br />第 2 次尝试 - 延迟约 2 秒<br />第 3 次尝试 - 延迟约 6 秒<br />第 4 次尝试 - 延迟约 14 秒<br />第 5 次尝试 - 延迟约 30 秒 |

> [AZURE.NOTE]端到端延迟目标假设采用服务连接的默认超时。如果您指定更长的连接超时，则每次重试尝试都会将端到端延迟延长这一附加时间。

### 示例（使用 ADO.NET 的 SQL 数据库）

本部分介绍了如何使用临时故障处理应用程序块通过 **RetryManager** 中已配置的一组重试策略来访问 Azure SQL 数据库（如上一部分中的[策略配置](#policy-configuration-sql-database-using-ado-net-)所述）。 使用临时故障处理应用程序块的最简单方法是通过 **ReliableSqlConnection** 类，或通过在连接上调用扩展方法（如 **OpenWithRetry**）。有关详细信息，请参阅[临时故障处理应用程序块](http://msdn.microsoft.com/zh-cn/library/hh680934.aspx)。

不过，在临时故障处理应用程序块的当前版本中，这些方法本身并不支持针对 SQL 数据库的异步操作。最好仅使用异步技术访问 Azure 服务（如 SQL 数据库）。因此，您应考虑采用以下技术，结合使用临时故障处理应用程序块和 SQL 数据库。

您可以借助 C# 语言第 5 版中的简化异步支持，创建由临时故障处理应用程序块提供的方法的异步版本。例如，以下代码展示了如何创建 **ExecuteReaderWithRetry** 扩展方法的异步版本。原始代码的更改和添加内容进行了突出显示。可从 GitHub 上的[临时故障处理应用程序块 (Topaz)](http://topaz.codeplex.com/SourceControl/latest) 获取 Topaz 的源代码。

```csharp
public async static Task<SqlDataReader> ExecuteReaderWithRetryAsync(this SqlCommand command, RetryPolicy cmdRetryPolicy,
RetryPolicy conRetryPolicy)
{
	GuardConnectionIsNotNull(command);

	// Check if retry policy was specified, if not, use the default retry policy.
	return await (cmdRetryPolicy ?? RetryPolicy.NoRetry).ExecuteAsync(async () =>
	{
	    var hasOpenConnection = await EnsureValidConnectionAsync(command, conRetryPolicy).ConfigureAwait(false);

	    try
	    {
	        return await command.ExecuteReaderAsync().ConfigureAwait(false);
	    }
	    catch (Exception)
	    {
	        if (hasOpenConnection && command.Connection != null && command.Connection.State == ConnectionState.Open)
	        {
	            command.Connection.Close();
	        }

	        throw;
	    }
	}).ConfigureAwait(false);
}
```

这一新建的异步扩展方法的使用方法与临时故障处理应用程序块中包含的同步版本相同。

```csharp
var sqlCommand = sqlConnection.CreateCommand();
sqlCommand.CommandText = "[some query]";

var retryPolicy =
	RetryManager.Instance.GetRetryPolicy<SqlDatabaseTransientErrorDetectionStrategy>("alt sql");
using (var reader = await sqlCommand.ExecuteReaderWithRetryAsync(retryPolicy))
{
	// Do something with the values
}
```

不过，此方法仅处理各个操作或命令，不处理包含正确定义的事务边界的语句块。另外，此方法也无法从连接池中删除失败的连接，因此后续尝试不会选择它们。有关解决这些问题的同步示例，请访问[云服务基础数据访问层 - 临时故障处理](http://social.technet.microsoft.com/wiki/contents/articles/18665.cloud-service-fundamentals-data-access-layer-transient-fault-handling.aspx#Timeouts_amp_Connection_Management)。除了重试数据库指令的任意序列，它还会清除连接池以删除无效的连接，并检测整个过程。虽然此示例中展示的代码是同步代码，但您可以相对较容易地将它转换为异步代码。

### 详细信息

若要详细了解如何使用临时故障处理应用程序块，请参阅：

* [结合使用临时故障处理应用程序块和 SQL Azure](http://msdn.microsoft.com/zh-cn/library/hh680899.aspx)
* [坚持不懈是一切成功的秘密：使用临时故障处理应用程序块](http://msdn.microsoft.com/zh-cn/library/dn440719.aspx)
* [云服务基础数据访问层 - 临时故障处理](http://social.technet.microsoft.com/wiki/contents/articles/18665.cloud-service-fundamentals-data-access-layer-transient-fault-handling.aspx)

有关充分利用 SQL 数据库的一般指南，请参阅：

* [Azure SQL 数据库性能和弹性指南](http://social.technet.microsoft.com/wiki/contents/articles/3507.windows-azure-sql-database-performance-and-elasticity-guide.aspx)
* [最大限度地减少 SQL Azure 中的连接池错误](http://blogs.msdn.com/b/adonet/archive/2011/11/05/minimizing-connection-pool-errors-in-sql-azure.aspx)

## 服务总线重试指南

服务总线是云消息传送平台，可提供松散耦合的消息交换，同时提升应用程序各组件（无论是托管在云中还是在本地）的规模和复原能力。

### 重试机制

服务总线通过 [RetryPolicy](http://msdn.microsoft.com/zh-cn/library/microsoft.servicebus.retrypolicy.aspx) 基类的实现来实现重试。所有服务总线客户端均公开 **RetryPolicy** 属性，此属性可以设置为 **RetryPolicy** 基类的一个实现。内置实现为：

* [RetryExponential 类](http://msdn.microsoft.com/zh-cn/library/microsoft.servicebus.retryexponential.aspx)：这会公开用于控制回退间隔和重试计数的属性，以及用于限制操作完成总时间的 **TerminationTimeBuffer** 属性。
* [NoRetry 类](http://msdn.microsoft.com/zh-cn/library/microsoft.servicebus.noretry.aspx)：这适用于不需要在服务总线 API 一级进行重试的情况，如其他进程将重试作为批处理或多步操作的一部分进行管理的情况。

服务总线操作可以返回一系列异常，如[附录：消息传送异常](http://msdn.microsoft.com/zh-cn/library/hh418082.aspx)中所列。此列表提供了一些信息来指明重试操作是否合适。例如，[ServerBusyException](http://msdn.microsoft.com/zh-cn/library/microsoft.servicebus.messaging.serverbusyexception.aspx) 指明客户端应等待一段时间，然后重试操作。**ServerBusyException** 的出现还会导致服务总线切换到不同模式，这会令计算的重试延迟额外延迟 10 秒。在一段很短的时间后，此模式会重置。

从服务总线返回的异常会公开 **IsTransient** 属性，此属性指明客户端是否应重试操作。内置的 **RetryExponential** 策略依赖于 **MessagingException** 类（所有服务总线异常的基类）中的 **IsTransient** 属性。如果您创建 **RetryPolicy** 基类的自定义实现，则可以结合使用异常类型和 **IsTransient** 属性来更精细地控制重试操作。例如，您可以检测 **QuotaExceededException**，并采取措施以在重试向其发送消息之前清空队列。

### 策略配置（服务总线）

重试策略是以编程方式进行设置，可以设置为 **NamespaceManager** 和 **MessagingFactory** 的默认策略，也可以针对每个消息客户端进行单独设置。若要设置消息会话的默认重试策略 ，请设置 **NamespaceManager** 的 **RetryPolicy**。

	namespaceManager.Settings.RetryPolicy = new RetryExponential(minBackoff: TimeSpan.FromSeconds(0.1),
	                                                             maxBackoff: TimeSpan.FromSeconds(30),
	                                                             deltaBackoff: TimeSpan.FromSeconds(2),
	                                                             terminationTimeBuffer: TimeSpan.FromSeconds(5),
	                                                             maxRetryCount: 3);

请注意，为了清楚起见，此代码使用命名参数。或者，您可以省略名称，因为没有参数是可选的。

	namespaceManager.Settings.RetryPolicy = new RetryExponential(TimeSpan.FromSeconds(0.1),
	                 TimeSpan.FromSeconds(30), TimeSpan.FromSeconds(2), TimeSpan.FromSeconds(5), 3);

若要为通过消息工厂创建的所有客户端设置默认重试策略，请设置 **MessagingFactory** 的 **RetryPolicy**。

	messagingFactory.RetryPolicy = new RetryExponential(minBackoff: TimeSpan.FromSeconds(0.1),
	                                                    maxBackoff: TimeSpan.FromSeconds(30),
	                                                    deltaBackoff: TimeSpan.FromSeconds(2),
	                                                    terminationTimeBuffer: TimeSpan.FromSeconds(5),
	                                                    maxRetryCount: 3);

若要设置消息客户端的重试策略，或替代其默认策略，请使用相应策略类的实例设置其 **RetryPolicy** 属性：

```csharp
client.RetryPolicy = new RetryExponential(minBackoff: TimeSpan.FromSeconds(0.1),
	                                        maxBackoff: TimeSpan.FromSeconds(30),
	                                        deltaBackoff: TimeSpan.FromSeconds(2),
	                                        terminationTimeBuffer: TimeSpan.FromSeconds(5),
	                                        maxRetryCount: 3);
```

不能在各个操作级别设置重试策略。它适用于消息客户端的所有操作。下表显示了内置重试策略的默认设置。

![](media/best-practices-retry-service-specific/RetryServiceSpecificGuidanceTable7.png)

### 重试使用指南

使用服务总线时，请注意以下指南：

* 使用内置 **RetryExponential** 实现时，请勿将回退操作实现为响应服务器忙异常和自动切换到适合重试模式的策略。
* 服务总线支持配对的命名空间这一功能。如果主要命名空间中的队列失败，则此功能会实现自动故障转移到单独命名空间中的备份队列。在主要队列恢复后，来自辅助队列的消息可以发送回主要队列。此功能有助于解决临时故障。有关详细信息，请参阅[异步消息模式和高可用性](http://msdn.microsoft.com/zh-cn/library/azure/dn292562.aspx)。

请考虑从下列重试操作设置入手。这些都是通用设置，您应监视操作，并对值进行微调以适应您自己的方案。


![](media/best-practices-retry-service-specific/RetryServiceSpecificGuidanceTable8.png)

### 遥测

服务总线使用 **EventSource** 将重试记录为 ETW 事件。必须将 **EventListener** 附加到事件源，才能捕获事件并在性能查看器中查看这些事件，或将这些事件写入合适的目标日志。为此，您可以使用[语义式日志记录应用程序块](http://msdn.microsoft.com/zh-cn/library/dn775006.aspx)。重试事件具有以下形式：

```text
Microsoft-ServiceBus-Client/RetryPolicyIteration
ThreadID="14,500"
FormattedMessage="[TrackingId:] RetryExponential: Operation Get:https://retry-guidance-tests.servicebus.windows.net/TestQueue/?api-version=2014-05 at iteration 0 is retrying after 00:00:00.1000000 sleep because of Microsoft.ServiceBus.Messaging.MessagingCommunicationException: The remote name could not be resolved: 'retry-guidance-tests.servicebus.windows.net'.TrackingId:6a26f99c-dc6d-422e-8565-f89fdd0d4fe3, TimeStamp:9/5/2014 10:00:13 PM."
trackingId=""
policyType="RetryExponential"
operation="Get:https://retry-guidance-tests.servicebus.windows.net/TestQueue/?api-version=2014-05"
iteration="0"
iterationSleep="00:00:00.1000000"
lastExceptionType="Microsoft.ServiceBus.Messaging.MessagingCommunicationException"
exceptionMessage="The remote name could not be resolved: 'retry-guidance-tests.servicebus.windows.net'.TrackingId:6a26f99c-dc6d-422e-8565-f89fdd0d4fe3,TimeStamp:9/5/2014 10:00:13 PM"
```

### 示例（服务总线）

以下代码示例展示了如何为下列对象设置重试策略：

* 命名空间管理器：重试策略适用于此管理器上的所有操作，不能针对各个操作进行替代。
* 消息工厂：重试策略适用于通过此工厂创建的所有客户端，不能在创建各个客户端时进行替代。
* 单个消息客户端：在创建客户端后，您可以为此客户端设置重试策略。重试策略适用于此客户端上的所有操作。

```csharp
using System;
using System.Threading.Tasks;
using Microsoft.ServiceBus;
using Microsoft.ServiceBus.Messaging;

namespace RetryCodeSamples
{
	class ServiceBusCodeSamples
	{
		private const string connectionString =
		    @"Endpoint=sb://[my-namespace].servicebus.windows.net/;
		        SharedAccessKeyName=RootManageSharedAccessKey;
		        SharedAccessKey=C99..........Mk=";

		public async static Task Samples()
		{
		    const string QueueName = "TestQueue";

		    ServiceBusEnvironment.SystemConnectivity.Mode = ConnectivityMode.Http;

		    var namespaceManager = NamespaceManager.CreateFromConnectionString(connectionString);

		    // The namespace manager will have a default exponential policy with 10 retry attempts
		    // and a 3 second delay delta.
		    // Retry delays will be approximately 0 sec, 3 sec, 9 sec, 25 sec and the fixed 30 sec,
		    // with an extra 10 sec added when receiving a ServiceBusyException.

		    {
		        // Set different values for the retry policy, used for all operations on the namespace manager.
		        namespaceManager.Settings.RetryPolicy =
		            new RetryExponential(
		                minBackoff: TimeSpan.FromSeconds(0),
		                maxBackoff: TimeSpan.FromSeconds(30),
		                deltaBackoff: TimeSpan.FromSeconds(1.75),
		                terminationTimeBuffer: TimeSpan.FromSeconds(4),
		                maxRetryCount: 3);

		        // Policies cannot be specified on a per-operation basis.
		        if (!await namespaceManager.QueueExistsAsync(QueueName))
		        {
		            await namespaceManager.CreateQueueAsync(QueueName);
		        }
		    }


		    var messagingFactory = MessagingFactory.Create(
		        namespaceManager.Address, namespaceManager.Settings.TokenProvider);
		    // The messaging factory will have a default exponential policy with 10 retry attempts
		    // and a 3 second delay delta.
		    // Retry delays will be approximately 0 sec, 3 sec, 9 sec, 25 sec and the fixed 30 sec,
		    // with an extra 10 sec added when receiving a ServiceBusyException.

		    {
		        // Set different values for the retry policy, used for clients created from it.
		        messagingFactory.RetryPolicy =
		            new RetryExponential(
		                minBackoff: TimeSpan.FromSeconds(1),
		                maxBackoff: TimeSpan.FromSeconds(30),
		                deltaBackoff: TimeSpan.FromSeconds(2),
		                terminationTimeBuffer: TimeSpan.FromSeconds(5),
		                maxRetryCount: 3);


		        // Policies cannot be specified on a per-operation basis.
		        var session = await messagingFactory.AcceptMessageSessionAsync();
		    }


		    {
		        var client = messagingFactory.CreateQueueClient(QueueName);
		        // The client inherits the policy from the factory that created it.


		        // Set different values for the retry policy on the client.
		        client.RetryPolicy =
		            new RetryExponential(
		                minBackoff: TimeSpan.FromSeconds(0.1),
		                maxBackoff: TimeSpan.FromSeconds(30),
		                deltaBackoff: TimeSpan.FromSeconds(2),
		                terminationTimeBuffer: TimeSpan.FromSeconds(5),
		                maxRetryCount: 3);


		        // Policies cannot be specified on a per-operation basis.
		        var session = await client.AcceptMessageSessionAsync();
		    }
		}
	}
}
```

## 详细信息

* [异步消息模式和高可用性](http://msdn.microsoft.com/zh-cn/library/azure/dn292562.aspx)

## 高速缓存 (Redis) 重试指南

Azure Redis 高速缓存是一项快速数据访问和低延迟高速缓存服务，基于常用的开放源代码 Redis 高速缓存。它是由 Microsoft 管理的安全服务，可通过 Azure 中的任意应用程序进行访问。

本部分中的指南假设您使用 StackExchange.Redis 客户端访问高速缓存。[Redis 网站](http://redis.io/clients)上列出了其他合适的客户端，这些客户端可能具有不同的重试机制。

请注意，StackExchange.Redis 客户端通过单个连接使用多路复用。建议的用法是，在应用程序启动时创建客户端实例，然后对针对高速缓存执行的所有操作使用此实例。因此，高速缓存连接只建立一次，且本部分中的所有指南均与此启动连接（而不是访问高速缓存的每个操作）的重试策略相关。

### 重试机制

StackExchange.Redis 客户端使用连接管理器类，此类通过一组选项进行配置。这些选项包括 **ConnectRetry** 属性，此属性指定了失败的高速缓存连接的重试次数。不过，重试策略仅用于初始连接操作，重试之间没有等待间隔。

### 策略配置（Azure Redis 高速缓存）

重试策略是以编程方式进行配置，方法为在连接到高速缓存前设置客户端的选项。为此，您可以创建 **ConfigurationOptions** 类的实例，填充其属性，然后将它传递到 **Connect** 方法。

```csharp
var options = new ConfigurationOptions { EndPoints = { "localhost" },
	                                        ConnectRetry = 3,
	                                        ConnectTimeout = 2000 };
ConnectionMultiplexer redis = ConnectionMultiplexer.Connect(options, writer);
```

请注意，**ConnectTimeout** 属性指定的是最长等待时间（以毫秒为单位），而不是重试之间的延迟。

或者，您可以将选项指定为字符串，然后将其传递到 **Connect** 方法。

```csharp
	var options = "localhost,connectRetry=3,connectTimeout=2000";
	ConnectionMultiplexer redis = ConnectionMultiplexer.Connect(options, writer);
```

您也可以在连接到高速缓存时，直接指定选项。

```csharp
var conn = ConnectionMultiplexer.Connect("redis0:6380,redis1:6380,connectRetry=3");
```

下表显示了内置重试策略的默认设置。

| **上下文** | **设置** | **默认值**<br />(v 1.0.331) | **含义** |
|----------------------|-----------------------------------------|-----------------------------------------|---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| 配置选项 | ConnectRetry<br /><br />ConnectTimeout<br /><br />SyncTimeout | 3<br /><br />最长 5000 毫秒，外加 SyncTimeout<br />1000 | 初始连接操作期间，连接尝试的重复次数。<br />连接操作的超时（以毫秒为单位）。不是重试尝试之间的延迟。<br />同步操作时间（以毫秒为单位）。 |

> [AZURE.NOTE]SyncTimeout 对操作的端到端延迟有推高效果。不过，通常情况下，不建议使用同步操作。有关详细信息，请参阅[管道和多路复用器](http://github.com/StackExchange/StackExchange.Redis/blob/master/Docs/PipelinesMultiplexers.md)。

## 重试使用指南

使用 Azure Redis 高速缓存时，请注意以下指南：

* StackExchange Redis 客户端管理自己的重试，但仅限在应用程序首次启动时建立的高速缓存连接。您可以配置连接超时和建立此连接的重试尝试次数，但重试策略不适用于针对高速缓存执行的操作。
* 重试机制在重试尝试之间没有延迟。它仅在指定的连接超时到期后对失败的连接进行指定次数的重试。
* 请考虑改为访问原始数据源进行回退，而不是使用大量重试尝试。

## 遥测

您可以使用 **TextWriter** 收集连接（而不是其他操作）的相关信息。

```csharp
var writer = new StringWriter();
...
ConnectionMultiplexer redis = ConnectionMultiplexer.Connect(options, writer);
```

生成的输出示例如下所示。

```text
localhost:6379,connectTimeout=2000,connectRetry=3
1 unique nodes specified
Requesting tie-break from localhost:6379 > __Booksleeve_TieBreak...
Allowing endpoints 00:00:02 to respond...
localhost:6379 faulted: SocketFailure on PING
localhost:6379 failed to nominate (Faulted)
> UnableToResolvePhysicalConnection on GET
No masters detected
localhost:6379: Standalone v2.0.0, master; keep-alive: 00:01:00; int: Connecting; sub: Connecting; not in use: DidNotRespond
localhost:6379: int ops=0, qu=0, qs=0, qc=1, wr=0, sync=1, socks=2; sub ops=0, qu=0, qs=0, qc=0, wr=0, socks=2
Circular op-count snapshot; int: 0 (0.00 ops/s; spans 10s); sub: 0 (0.00 ops/s; spans 10s)
Sync timeouts: 0; fire and forget: 0; last heartbeat: -1s ago
resetting failing connections to retry...
retrying; attempts left: 2...
...
```

## 示例（Azure Redis 高速缓存）

以下代码示例展示了如何在初始化 StackExchange.Redis 客户端以在应用程序启动时访问 Azure Redis 高速缓存时，配置连接超时设置和重试次数。请注意，连接超时是指您愿意等待的高速缓存连接时间段；不是指重试尝试之间的延迟。

此示例展示了如何使用 **ConfigurationOptions** 实例设置配置。

```csharp
using System;
using System.Collections.Generic;
using System.IO;
using System.Linq;
using System.Text;
using System.Threading.Tasks;
using StackExchange.Redis;

namespace RetryCodeSamples
{
	class CacheRedisCodeSamples
	{
	    public async static Task Samples()
	    {
	        var writer = new StringWriter();

	        {
	            try
	            {
	                // Using object-based configuration.
	                var options = new ConfigurationOptions
	                                    {
	                                        EndPoints = { "localhost" },
	                                        ConnectRetry = 3,
	                                        ConnectTimeout = 2000  // The maximum waiting time (ms), not the delay for retries.
	                                    };
	                ConnectionMultiplexer redis = ConnectionMultiplexer.Connect(options, writer);

	                // Store a reference to the multiplexer for use in the application.
	            }
	            catch
	            {
	                Console.WriteLine(writer.ToString());
	                throw;
	            }
	        }
	    }
	}
}
```

此示例展示了如何通过将选项指定为字符串来设置配置。

```csharp
using System.Collections.Generic;
using System.IO;
using System.Linq;
using System.Text;
using System.Threading.Tasks;
using StackExchange.Redis;

namespace RetryCodeSamples
{
	class CacheRedisCodeSamples
	{
	    public async static Task Samples()
	    {
	        var writer = new StringWriter();

	        {
	            try
	            {
	                // Using string-based configuration.
	                var options = "localhost,connectRetry=3,connectTimeout=2000";
	                ConnectionMultiplexer redis = ConnectionMultiplexer.Connect(options, writer);

	                // Store a reference to the multiplexer for use in the application.
	            }
	            catch
	            {
	                Console.WriteLine(writer.ToString());
	                throw;
	            }
	        }
	    }
	}
}
```

有关更多示例，请参阅项目网站上的[配置](http://github.com/StackExchange/StackExchange.Redis/blob/master/Docs/Configuration.md#configuration)。

## 详细信息

* [Redis 网站](http://redis.io/)

## DocumentDB（预发行）重试指南

DocumentDB 是一项完全托管的文档数据库即服务，可以对无架构的 JSON 数据模型进行大量查询和索引。它提供可配置的可靠性能、本机 JavaScript 事务处理，专为具有弹性延展能力的云构建而成。

## 重试机制

DocumentDB 客户端的预发行版本包括内部重试机制和不可配置的重试机制（这可能会在后续版本中有所变化）。默认设置因所使用的上下文而异。一些操作使用指数回退策略（包含硬编码的参数）。另一些操作仅指定应尝试的重试次数，并在从服务返回的 [DocumentClientException](http://msdn.microsoft.com/zh-cn/library/microsoft.azure.documents.documentclientexception.retryafter.aspx) 实例中使用重试延迟。如果未指定延迟，则使用 5 秒延迟。

## 策略配置 (DocumentDB)

无。所有用来实现重试的类都是内部的。重试参数是常量或设置为类构造函数（使用参数）。

下表显示了内置重试策略的默认设置。

| **上下文** | **设置** | **值** | **工作原理** |
|------------------------|---------------------------------------------------|------------|----------------------------------------------------------------------------------------------------------------------------------------------------------------|
| RetryPolicy（内部） | MaxRetryAttemptsOnQuery<br /><br />MaxRetryAttemptsOnRequest | 3<br /><br />0 | 文档查询的重试尝试次数。不能更改此值。<br />其他请求的重试尝试次数。不能更改此值。 |

## 重试使用指南

使用 DocumentDB 时，请注意以下指南：

* 不能更改默认重试策略。
* 有关默认设置的详细信息，请参阅 [TBD]。

## 遥测

重试尝试通过 .NET **TraceSource** 记录为未结构化的跟踪消息。您必须配置 **TraceListener**，才能捕获事件并将这些事件写入合适的目标日志。

## 搜索重试指南

Azure 搜索可用于向网站或应用程序添加功能强大且复杂的搜索功能、快速轻松地优化搜索结果，并构造大量经过微调的排名模型。

### 重试机制

没有适用于搜索的内置重试机制，因为典型用法是通过 HTTP 请求。若要实现重试，您可以使用 REST 客户端的一般实现，并根据服务响应来决定何时以及是否重试操作。有关详细信息，请参阅本指南后面的[常规 REST 和重试指南](#general-rest-and-retry-guidelines)部分。

### 重试使用指南

使用 Azure 搜索时，请注意以下指南：

* 使用服务返回的状态代码来确定故障类型。[HTTP 状态代码（Azure 搜索）](http://msdn.microsoft.com/zh-cn/library/dn798925.aspx)中定义了状态代码。状态代码 503（服务不可用）表明服务过载，无法立即处理请求。相应的措施是，仅在允许的服务恢复时间过后才重试操作。如果在太短的延迟间隔后重试，则可能会延长不可用时间。
* 有关重试 REST 操作的一般信息，请参阅本指南后面的[常规 REST 和重试指南](#general-rest-and-retry-guidelines)部分。

## 详细信息

* [Azure 搜索 REST API](http://msdn.microsoft.com/zh-cn/library/dn798935.aspx)

## Azure Active Directory 重试指南

Azure Active Directory (AD) 是一项全面的标识和访问管理云解决方案，集成了核心目录服务、高级标识监管、安全性和应用程序访问管理等各种功能。Microsoft Azure AD 还为开发人员提供了身份管理平台，以便他们可以根据集中的策略和规则，控制应用程序访问情况。

### 重试机制

在 Active Directory Authentication Library (ADAL) 中，没有适用于 Azure Active Directory 的内置重试机制。您可以使用临时故障处理应用程序块实现重试策略，其中包含适用于 Active Directory 返回的异常的自定义检测机制。

### 策略配置 (Azure Active Directory)

结合使用临时故障处理应用程序块和 Azure Active Directory 时，您需要根据定义了要使用的检测策略的类来创建 **RetryPolicy** 实例。

```csharp
var policy = new RetryPolicy<AdalDetectionStrategy>(new ExponentialBackoff(retryCount: 5,
	                                                                 minBackoff: TimeSpan.FromSeconds(0),
	                                                                 maxBackoff: TimeSpan.FromSeconds(60),
	                                                                 deltaBackoff: TimeSpan.FromSeconds(2)));
```

然后，调用重试策略的 **ExecuteAction** 或 **ExecuteAsync** 方法，同时传入要执行的操作。

```csharp
var result = await policy.ExecuteAsync(() => authContext.AcquireTokenAsync(resourceId, clientId, uc));
```

故障发生时，检测策略类会收到异常，且必须检测这可能是临时故障还是永久性故障。通常情况下，它会通过检查异常类型和状态代码来执行此操作。例如，“服务不可用”响应表明应进行重试尝试。临时故障处理应用程序块不包括适合与 ADAL 客户端结合使用的检测策略类，但以下[示例](#examples-azure-active-directory-)部分展示了自定义检测策略的示例。使用自定义检测策略与使用临时故障处理应用程序块提供的策略无异。

本指南末尾的[临时故障处理应用程序块 (Topaz) 策略](#transient-fault-handling-application-block-topaz-strategies)部分中介绍了临时故障处理应用程序块的默认策略。

## 重试使用指南

使用 Azure Active Directory 时，请注意以下指南：

* 如果您使用的是适用于 Azure Active Directory 的 REST API，只有当结果是 5xx 范围内的错误（如 500 内部服务器错误、502 错误的网关、503 服务不可用和 504 网关超时）时，才应重试操作。请勿针对其他任何错误重试操作。
* 如果您使用的是 Active Directory Authentication Library (ADAL)，则无法轻松访问 HTTP 代码。您需要创建自定义检测策略，其中包含用于检查 ADAL 异常属性的逻辑。请参阅以下[示例](#examples-azure-active-directory-)部分。
* 建议将指数回退策略用于 Azure Active Directory 的批处理方案。

请考虑从下列重试操作设置入手。这些都是通用设置，您应监视操作，并对值进行微调以适应您自己的方案。


| **上下文** | **示例目标 E2E<br />最长延迟** | **重试策略** | **设置** | **值** | **工作原理** |
|----------------------|----------------------------------------------|--------------------|-----------------------------------------------------------------------|----------------------------|-------------------------------------------------------------------------------------------------------------------------------|
| 交互式、UI<br />或前台 | 2 秒 | FixedInterval | 重试计数<br />重试间隔<br />首次快速重试 | 3<br />500 毫秒<br />true | 第 1 次尝试 - 延迟 0 秒<br />第 2 次尝试 - 延迟 500 毫秒<br />第 3 次尝试 - 延迟 500 毫秒 |
| 后台<br />或批处理 | 60 秒 | ExponentialBackoff | 重试计数<br />最小回退<br />最大回退<br />增量回退<br />首次快速重试 | 5<br />0 秒<br />60 秒<br />2 秒<br />false | 第 1 次尝试 - 延迟 0 秒<br />第 2 次尝试 - 延迟约 2 秒<br />第 3 次尝试 - 延迟约 6 秒<br />第 4 次尝试 - 延迟约 14 秒<br />第 5 次尝试 - 延迟约 30 秒 |

## 示例 (Azure Active Directory)

以下代码示例展示了如何使用临时故障处理应用程序块 (Topaz)，定义适合与 ADAL 客户端结合使用的自定义临时错误检测策略。此代码会根据类型 **AdalDetectionStrategy** 的自定义检测策略新建一个 **RetryPolicy** 实例，如以下代码清单所定义。Topaz 的自定义检测策略实现 **ITransientErrorDetectionStrategy** 接口。如果应尝试重试，则返回 true；如果故障似乎是非临时故障且不应尝试重试，则返回 **false**。

	using System;
	using System.Collections.Generic;
	using System.Linq;
	using System.Net;
	using System.Text;
	using System.Threading.Tasks;
	using Microsoft.IdentityModel.Clients.ActiveDirectory;
	using Microsoft.Practices.EnterpriseLibrary.TransientFaultHandling;

	namespace RetryCodeSamples
	{
	    class ActiveDirectoryCodeSamples
	    {
	        public async static Task Samples()
	        {
	            var authority = "[some authority]";
	            var resourceId = “[some resource id]”;
	            var clientId = “[some client id]”;

	            var authContext = new AuthenticationContext(authority);

	            var uc = new UserCredential(“[user]", "[password]");

	            // Use Topaz with a custom detection strategy to manage retries.
	            var policy =
	                new RetryPolicy<AdalDetectionStrategy>(
	                    new ExponentialBackoff(
	                        retryCount: 5,
	                        minBackoff: TimeSpan.FromSeconds(0),
	                        maxBackoff: TimeSpan.FromSeconds(60),
	                        deltaBackoff: TimeSpan.FromSeconds(2)));

	            var result = await policy.ExecuteAsync(() => authContext.AcquireTokenAsync(resourceId, clientId, uc));

	            // Get the access token
	            var accessToken = result.AccessToken;

	            // Use the result, probably to authorize an API call.
	        }
	    }

	    // TODO: This is sample code that needs validation from the WAAD team!
	    // based on existing detection strategies
	    public class AdalDetectionStrategy : ITransientErrorDetectionStrategy
	    {
	        private static readonly WebExceptionStatus[] webExceptionStatus =
	            new[]
	            {
	                WebExceptionStatus.ConnectionClosed,
	                WebExceptionStatus.Timeout,
	                WebExceptionStatus.RequestCanceled
	            };

	        private static readonly HttpStatusCode[] httpStatusCodes =
	            new[]
	            {
	                HttpStatusCode.InternalServerError,
	                HttpStatusCode.GatewayTimeout,
	                HttpStatusCode.ServiceUnavailable,
	                HttpStatusCode.RequestTimeout
	            };

	        public bool IsTransient(Exception ex)
	        {
	            var adalException = ex as AdalException;
	            if (adalException == null)
	            {
	                return false;
	            }

	            if (adalException.ErrorCode == AdalError.ServiceUnavailable)
	            {
	                return true;
	            }

	            var innerWebException = adalException.InnerException as WebException;
	            if (innerWebException != null)
	            {
	                if (webExceptionStatus.Contains(innerWebException.Status))
	                {
	                    return true;
	                }

	                if (innerWebException.Status == WebExceptionStatus.ProtocolError)
	                {
	                    var response = innerWebException.Response as HttpWebResponse;
	                    return response != null && httpStatusCodes.Contains(response.StatusCode);
	                }
	            }

	            return false;
	        }
	    }
	}

若要了解如何重试 Active Directory Graph API 操作和返回的错误代码，请参阅：

* [代码示例：重试逻辑](http://msdn.microsoft.com/zh-cn/library/azure/dn448547.aspx)
* [Azure AD Graph 错误代码](http://msdn.microsoft.com/zh-cn/library/azure/hh974480.aspx)

## 详细信息

* [实现自定义检测策略](http://msdn.microsoft.com/zh-cn/library/hh680940.aspx) (Topaz)
* [实现自定义重试策略](http://msdn.microsoft.com/zh-cn/library/hh680943.aspx) (Topaz)
* [令牌颁发和重试指南](http://msdn.microsoft.com/zh-cn/library/azure/dn168916.aspx)

## 常规 REST 和重试指南

访问 Azure 或第三方服务时，请注意以下指南：

* 使用管理重试的系统化方法（可能作为可重用代码），以便您可以跨所有客户端和解决方案应用一致的方法。
* 如果目标服务或客户端没有内置的重试机制，请考虑使用重试框架（如临时故障处理应用程序块）来管理重试。这会帮助您实现一致的重试行为，并可能会为目标服务提供合适的默认重试策略。不过，您可能需要为具有非标准行为的服务创建自定义重试代码，无需根据异常来指明临时故障；或者，您可能需要使用 **Retry-Response** 答复来管理重试行为。
* 临时检测逻辑视用于调用 REST 调用的实际客户端 API 而定。某些客户端（如较新的 **HttpClient** 类）不会引发包含失败 HTTP 状态代码的已完成请求的异常。这可以提升性能，但会阻止使用临时故障处理应用程序块。在这种情况下，您可以使用针对失败 HTTP 状态代码引发异常的代码，包装 REST API 调用，然后可以由临时故障处理应用程序块进行处理。或者，您可以使用另一种机制来驱动重试。
* 从服务返回的 HTTP 状态代码可以帮助指明故障是否是临时故障。您可能需要检查客户端生成的异常或重试框架，以访问状态代码或确定对等的异常类型。以下 HTTP 代码通常可指明重试是否合适：
  * 408 请求超时
  * 500 内部服务器错误
  * 502 错误的网关
  * 503 服务不可用
  * 504 网关超时
* 如果您根据异常构建重试逻辑，则以下代码通常可指明出现了无法建立连接的临时故障：
  * WebExceptionStatus.ConnectionClosed
  * WebExceptionStatus.ConnectFailure
  * WebExceptionStatus.Timeout
  * WebExceptionStatus.RequestCanceled
* 如果服务处于不可用状态，则服务可以指明在 **Retry-After** 响应头或其他自定义头中进行重试前的相应延迟（如 DocumentDB 服务所示）。服务还可以发送其他信息，既能以自定义头的形式，也能嵌入响应内容中。临时故障处理应用程序块不能使用标准头或任何自定义“retry-after”头。
* 请勿针对表示客户端错误的状态代码（4xx 范围内的错误）进行重试，“408 请求超时”除外。
* 在各种条件（如不同的网络状态和系统负载）下全面测试重试策略和机制。

## 重试策略

以下是典型的重试策略间隔类型：

* **指数**：此重试策略执行指定次数的重试，并使用随机指数回退方法来确定重试间隔。例如：

		var random = new Random();

		var delta = (int)((Math.Pow(2.0, currentRetryCount) - 1.0) *
		            random.Next((int)(this.deltaBackoff.TotalMilliseconds * 0.8),
		            (int)(this.deltaBackoff.TotalMilliseconds * 1.2)));
		var interval = (int)Math.Min(checked(this.minBackoff.TotalMilliseconds + delta),
		               this.maxBackoff.TotalMilliseconds);
		retryInterval = TimeSpan.FromMilliseconds(interval);

* **增量**：此重试策略具有指定的重试尝试次数以及增量重试间隔。例如：

		retryInterval = TimeSpan.FromMilliseconds(this.initialInterval.TotalMilliseconds +
		               (this.increment.TotalMilliseconds * currentRetryCount));

* **线性重试**：此重试策略执行指定次数的重试，并使用指定的固定重试间隔。例如：

		retryInterval = this.deltaBackoff;

## 详细信息

* [断路器策略](http://msdn.microsoft.com/zh-cn/library/dn589784.aspx)

## 临时故障处理应用程序块 (Topaz) 策略

临时故障处理应用程序块具有以下默认策略。

| **策略** | **设置** | **默认值** | **含义** |
|-------------------------|-----------------------------------------------------|-----------------------------|---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| **指数** | retryCount<br />minBackoff<br /><br />maxBackoff<br /><br />deltaBackoff<br /><br />fastFirstRetry | 10<br />1 秒<br /><br />30 秒<br /><br />10 秒<br /><br />true | 重试尝试次数。<br />最短回退时间。此值与计算的回退中的较高者将用作重试延迟。<br />最短回退时间。此值与计算的回退中的较低者将用作重试延迟。<br />此值用于计算重试之间指数延迟的随机增量。<br />是否立即进行首次重试尝试。 |
| **增量** | retryCount<br />initialInterval<br />increment<br /><br />fastFirstRetry<br />| 10<br />1 秒<br />1 秒<br /><br />true | 重试尝试次数。<br />适用于首次重试的初始间隔。<br />用于计算重试之间渐进式延迟的增量时间值。<br />是否立即进行首次重试尝试。 |
| **线性（固定间隔）** | retryCount<br />retryInterval<br />fastFirstRetry<br /> | 10<br />1 秒<br />true | 重试尝试次数。<br />重试之间延迟。<br />是否立即进行首次重试尝试。 |
有关使用临时故障处理应用程序块的示例，请参阅本指南中前面与使用 ADO.NET 的 Azure SQL 数据库和 Azure Active Directory 有关的示例部分。

<!---HONumber=71-->