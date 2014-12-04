<properties linkid="Contact - Support" urlDisplayName="Caching" pageTitle="How to use In-Role Cache (.NET) - Azure feature guide" metaKeywords="Azure cache, Azure caching, Azure cache, Azure caching, Azure store session state, Azure cache .NET, Azure cache C#" description="Learn how to use Azure In-Role Cache. The samples are written in C# code and use the .NET API." metaCanonical="" services="cache" documentationCenter=".NET" title="How to Use In-Role Cache for Azure Cache" authors="" solutions="" manager="" editor="" />

# 如何使用适用于 Azure Cache 的角色中缓存

本指南说明如何开始使用**适用于 Azure Cache 的角色中缓存**。相关示例用 C# 代码编写且使用 .NET API。涉及的应用场景包括**配置缓存群集**、**配置缓存客户端**、**在缓存中添加和删除对象、在缓存中存储 ASP.NET 会话状态**以及**使用缓存启用 ASP.NET 页面输出缓存**。有关使用角色中缓存的详细信息，请参阅[后续步骤][后续步骤]部分。

## 目录

-   [什么是角色中缓存？][什么是角色中缓存？]
-   [角色中缓存入门][角色中缓存入门]

    -   [配置缓存群集][配置缓存群集]
    -   [配置缓存客户端][配置缓存客户端]
-   [使用缓存][使用缓存]

    -   [如何：创建 DataCache 对象][如何：创建 DataCache 对象]
    -   [如何：在缓存中添加和检索对象][如何：在缓存中添加和检索对象]
    -   [如何：在缓存中指定对象的有效期][如何：在缓存中指定对象的有效期]
    -   [如何：在缓存中存储 ASP.NET 会话状态][如何：在缓存中存储 ASP.NET 会话状态]
    -   [如何：在缓存中存储 ASP.NET 页面输出缓存][如何：在缓存中存储 ASP.NET 页面输出缓存]
-   [后续步骤][后续步骤]

<a name="what-is"></a>

## 什么是角色中缓存？

角色中缓存为你的 Azure 应用程序提供了缓存层。Caching 通过将来自其他后端源的信息暂时存储在内存中来提高性能，并且可以降低云中与数据库事务相关的成本。角色中缓存包含以下功能：

-   为会话状态和页面输出缓存预生成 ASP.NET 提供
    程序，这样无需修改应用程序代码即可
    启用 Web 应用程序加速。
-   缓存任何可序列化的托管对象 - 例如：CLR 对象、行、XML、
    二进制数据。
-   跨 Azure 和 Windows Server AppFabric 实现
    一致的开发模型。

角色中缓存通过使用在 Azure 云服务（又称为托管服务）中托管角色实例的虚拟机的一部分内存，引入了执行缓存的新方法。你将可以采用更加灵活的部署方式，缓存的大小会非常大，并且没有特定于缓存的配额限制。

角色实例中的缓存具有以下优势：

-   无需为缓存付费。你只需为托管缓存的计算资源付费。
-   消除了缓存配额和限制。
-   提供了更强的控制和隔离措施。
-   提高了性能。
-   在向内或向外扩展角色时，会自动调整缓存大小。在添加或删除角色实例时，有效地向上或向下扩展可用于缓存的内存。
-   提供了完全无损的开发时调试。
-   支持 memcache 协议。

另外，角色实例中的缓存还提供以下可配置选项：

-   为缓存配置专用角色，或使缓存在现有角色中共存。
-   使缓存可供同一云服务部署中的多个客户端使用。
-   创建多个具有不同属性的命名缓存。
-   可以选择在单个缓存上配置高可用性。
-   使用扩展的缓存功能，例如区域、标记和通知。

本指南提供了角色中缓存的入门概述。有关超出本入门指南范围的那些功能的更多详细信息，请参阅[角色中缓存概述][角色中缓存概述]。

<a name="getting-started-cache-role-instance"></a>

## 角色中缓存入门

角色中缓存提供了一种使用托管角色实例的虚拟机上的内存来进行缓存的方法。托管缓存的角色实例称为**缓存群集**。有两种部署拓扑可用于角色实例中的缓存：

-   **专用角色**缓存 - 这些角色实例专用于缓存。
-   **并置角色**缓存 - 这些缓存与应用程序共享 VM 资源（带宽、CPU 和内存）。

若要在角色实例中使用缓存，你需要配置缓存群集，然后配置缓存客户端以便它们可以访问缓存群集。

-   [配置缓存群集][配置缓存群集]
-   [配置缓存客户端][配置缓存客户端]

<a name="enable-caching"></a>

## 配置缓存群集

若要配置**并置角色**缓存群集，请选择希望在其中托管缓存群集的角色。在“解决方案资源管理器”中，右键单击角色属性，然后选择“属性”。

![RoleCache1][RoleCache1]

切换到“缓存”选项卡，选中“启用缓存”复选框，然后指定所需缓存选项。在**辅助角色**或 **ASP.NET Web 角色**中启用缓存后，默认配置是为缓存分配了角色实例的 30% 内存的**并置角色**缓存。默认缓存是自动配置的，如果需要，可以创建其他命名缓存，并且这些缓存将共享已分配的内存。

![RoleCache2][RoleCache2]

若要配置**专用角色**缓存群集，请向项目中添加**缓存辅助角色**。

![RoleCache7][RoleCache7]

在向项目中添加**缓存辅助角色**后，默认配置是**专用角色**缓存。

![RoleCache8][RoleCache8]

在启用缓存后，可以配置缓存群集存储帐户。角色中缓存需要 Azure 存储帐户。此存储帐户用于保存从组成缓存群集的所有虚拟机访问的缓存群集的相关配置数据。此存储帐户在缓存群集角色属性页的“缓存”选项卡上的“命名缓存设置”上方指定。

![RoleCache10][RoleCache10]

> 如果没有配置此存储帐户，则角色将无法启动。

缓存的大小由角色的 VM 大小、角色的实例计数以及缓存群集是配置为专用角色还是并置角色缓存群集共同决定。

> 本部分提供了有关配置缓存大小的简单概述。有关缓存大小及其他容量规划注意事项的详细信息，请参阅[角色中缓存容量规划注意事项][角色中缓存容量规划注意事项]。

若要配置虚拟机大小和角色实例数，请在“解决方案资源管理器”中右键单击角色属性，然后选择“属性”。

![RoleCache1][RoleCache1]

切换到“配置”选项卡。默认的“实例计数”为 1，默认的“VM 大小”为“小型”。

![RoleCache3][RoleCache3]

VM 大小的总内存如下：

-   **小型**：1.75 GB
-   **中型**：3.5 GB
-   **大型**：7 GB
-   **超大型**：14 GB

> 这些内存大小表示可用于跨 OS、缓存进程、缓存数据和应用程序共享的 VM 的内存总量。有关配置虚拟机大小的更多信息，请参见[如何配置虚拟机大小][如何配置虚拟机大小]。请注意，**特小型** VM 大小不支持缓存。

在指定**并置角色**缓存后，通过获取指定百分比的虚拟机内存可确定缓存大小。在指定**专用角色**缓存后，虚拟机的所有可用内存均用于缓存。如果配置了两个角色实例，将使用虚拟机的组合内存。这构成了缓存群集，其中的可用缓存内存分布在多个角色实例上，但对缓存的客户端显示为单个资源。配置其他角色实例会以相同方式增加缓存大小。若要确定设置所需大小的缓存所需的设置，你可以使用[角色中缓存容量规划注意事项][角色中缓存容量规划注意事项]中的容量规划电子表格。

在配置缓存群集后，可以配置缓存客户端以允许访问缓存。

<a name="NuGet"></a>

## 配置缓存客户端

若要访问角色中缓存，客户端必须位于同一部署中。如果缓存群集是专用角色缓存群集，则客户端是部署中的其他角色。如果缓存群集是并置角色缓存群集，则客户端可以是部署中的其他角色或托管缓存群集的角色本身。提供了 NuGet 包，它可用于配置访问缓存的每个客户端角色。若要使用 Caching NuGet 包配置角色以访问缓存群集，请在“解决方案资源管理器”中右键单击角色项目，然后选择“管理 NuGet 包”。

![RoleCache4][RoleCache4]

选择“角色中缓存”，单击“安装”，然后单击“我接受”。

> 如果“角色中缓存”没有显示在列表中，请在“联机搜索”文本框中键入 **WindowsAzure.Caching**，然后从结果中选择它。

![RoleCache5][RoleCache5]

NuGet 包可执行多项操作：它将所需配置添加到角色的配置文件中，将缓存客户端诊断级别设置添加到 Azure 应用程序的 ServiceConfiguration.cscfg 文件中，并添加所需的程序集引用。

> 对于 ASP.NET Web 角色，Caching NuGet 包还将两个注释掉的节添加到 web.config 中。第一个节允许会话状态存储在缓存中，第二个节启用 ASP.NET 页面输出缓存。有关更多信息，请参见[如何：在缓存中存储 ASP.NET 会话状态][如何：在缓存中存储 ASP.NET 会话状态]和[如何：在缓存中存储 ASP.NET 页面输出缓存][如何：在缓存中存储 ASP.NET 页面输出缓存]。

NuGet 包将以下配置元素添加到角色的 web.config 或 app.config 中。将 **dataCacheClients** 节和 **cacheDiagnostics** 节添加到 **configSections** 元素之下。如果 **configSections** 元素不存在，则会创建一个作为 **configuration** 元素的子级。

    <configSections>
      <!-- Existing sections omitted for clarity. -->

      <section name="dataCacheClients" 
               type="Microsoft.ApplicationServer.Caching.DataCacheClientsSection, Microsoft.ApplicationServer.Caching.Core" 
               allowLocation="true" 
               allowDefinition="Everywhere" />

      <section name="cacheDiagnostics" 
               type="Microsoft.ApplicationServer.Caching.AzureCommon.DiagnosticsConfigurationSection, Microsoft.ApplicationServer.Caching.AzureCommon" 
               allowLocation="true" 
               allowDefinition="Everywhere" />
    </configSections>

这些新节包括对 **dataCacheClients** 元素和 **cacheDiagnostics** 元素的引用。这些元素还添加到 **configuration** 元素中。

    <dataCacheClients>
      <dataCacheClient name="default">
        <autoDiscover isEnabled="true" identifier="[cache cluster role name]" />
        <!--<localCache isEnabled="true" sync="TimeoutBased" objectCount="100000" ttlValue="300" />-->
      </dataCacheClient>
    </dataCacheClients>
    <cacheDiagnostics>
      <crashDump dumpLevel="Off" dumpStorageQuotaInMB="100" />
    </cacheDiagnostics>

在添加配置后，将 **[cache cluster role name]** 替换为托管缓存群集的角色的名称。

> 如果没有将 **[cache cluster role name]** 替换为托管缓存群集的角色的名称，则在访问缓存时会引发 **TargetInvocationException**，同时会引发内部 **DatacacheException** 并显示消息“不存在此类角色”。

NuGet 包还将 **ClientDiagnosticLevel** 设置添加到 ServiceConfiguration.cscfg 中的缓存客户端角色的 **ConfigurationSettings** 中。下面的示例是 ServiceConfiguration.cscfg 文件中的 **WebRole1** 节，其 **ClientDiagnosticLevel** 为 1，这是默认的 **ClientDiagnosticLevel**。

    <Role name="WebRole1">
      <Instances count="1" />
      <ConfigurationSettings>
        <!-- Existing settings omitted for clarity. -->
        <Setting name="Microsoft.WindowsAzure.Plugins.Caching.ClientDiagnosticLevel" 
                 value="1" />
      </ConfigurationSettings>
    </Role>

> 角色中缓存同时提供了缓存服务器和缓存客户端诊断级别。诊断级别是配置为缓存收集的诊断信息级别的单个设置。有关更多信息，请参见[解决和诊断角色中缓存问题][解决和诊断角色中缓存问题]

NuGet 包还添加对以下程序集的引用：

-   Microsoft.ApplicationServer.Caching.Client.dll
-   Microsoft.ApplicationServer.Caching.Core.dll
-   Microsoft.WindowsFabric.Common.dll
-   Microsoft.WindowsFabric.Data.Common.dll
-   Microsoft.ApplicationServer.Caching.AzureCommon.dll
-   Microsoft.ApplicationServer.Caching.AzureClientHelper.dll

如果你的角色是 ASP.NET Web 角色，则还添加以下程序集引用：

-   Microsoft.Web.DistributedCache.dll。

> 这些程序集位于 C:\\Program Files\\Microsoft SDKs\\Windows Azure\\.NET SDK\\2012-10\\ref\\Caching\\ 文件夹中。

在配置了你的客户端项目的缓存后，你可以使用以下各节中介绍的方法来使用你的缓存。

<a name="working-with-caches"></a>

## 使用缓存

本节中的步骤介绍如何使用缓存执行常见任务。

-   [如何：创建 DataCache 对象][如何：创建 DataCache 对象]
-   [如何：在缓存中添加和检索对象][如何：在缓存中添加和检索对象]
-   [如何：在缓存中指定对象的有效期][如何：在缓存中指定对象的有效期]
-   [如何：在缓存中存储 ASP.NET 会话状态][如何：在缓存中存储 ASP.NET 会话状态]
-   [如何：在缓存中存储 ASP.NET 页面输出缓存][如何：在缓存中存储 ASP.NET 页面输出缓存]

<a name="create-cache-object"></a>

## 如何：创建 DataCache 对象

若要以编程方式使用缓存，你需要引用该缓存。将以下代码添加到要从中使用角色中缓存的任何文件的顶部：

    using Microsoft.ApplicationServer.Caching;

> 如果在安装了添加必要引用的 Caching NuGet 包后， Visual Studio 仍不能识别 using 语句中的类型，请确保项目的目标配置文件是 .NET Framework 4.0 或 更高版本，并确保选择没有指定**客户端配置文件**的配置文件之一。有关配置缓存客户端的说明，请参阅[配置缓存客户端][配置缓存客户端]。

创建 **DataCache** 对象有两种方法。第一种方法是仅创建 **DataCache**，并传入所需缓存的名称。

    DataCache cache = new DataCache("default");

在实例化 **DataCache** 后，你可以使用它来与缓存交互，如以下各部分中所述。

若要使用第二种方法，请在你的应用程序中使用默认的构造函数创建新的 **DataCacheFactory** 对象。这会导致缓存客户端使用配置文件中的设置。调用新的 **DataCacheFactory** 实例的 **GetDefaultCache** 方法，该方法返回 **DataCache** 对象，或调用 **GetCache** 方法并传入你的缓存的名称。这些方法返回以后可用于以编程方式访问缓存的 **DataCache** 对象。

    // Cache client configured by settings in application configuration file.
    DataCacheFactory cacheFactory = new DataCacheFactory();
    DataCache cache = cacheFactory.GetDefaultCache();
    // Or DataCache cache = cacheFactory.GetCache("MyCache");
    // cache can now be used to add and retrieve items. 

<a name="add-object"></a>

## 如何：在缓存中添加和检索对象

若要向缓存中添加项，可以使用 **Add** 或 **Put**方法。**Add** 方法将指定的对象添加到缓存中，并按键参数的值进行键控。

    // Add the string "value" to the cache, keyed by "item"
    cache.Add("item", "value");

如果缓存中已存在具有相同键的对象，将引发**DataCacheException** 并显示以下消息：

> ErrorCode:SubStatus:正在尝试使用缓存中已存在的键创建对象。Caching 只会接受对象的唯一键值。

若要检索具有特定键的对象，可以使用 **Get** 方法。如果对象存在，则返回它，如果对象不存在，则返回 null。

    // Add the string "value" to the cache, keyed by "key"
    object result = cache.Get("Item");
    if (result == null)
    {
        // "Item" not in cache. Obtain it from specified data source
        // and add it.
        string value = GetItemValue(...);
        cache.Add("item", value);
    }
    else
    {
        // "Item" is in cache, cast result to correct type.
    }

如果具有指定键的对象不存在，则 **Put** 方法将该对象添加到缓存中，如果该对象存在，则替换它。

    // Add the string "value" to the cache, keyed by "item". If it exists,
    // replace it.
    cache.Put("item", "value");

<a name="specify-expiration"></a>

## 如何：在缓存中指定对象的有效期

默认情况下，缓存中的项在放入缓存中 10 分钟后到期。这可在托管缓存群集的角色的角色属性中的“生存时间(分钟)”设置中进行配置。

![RoleCache6][RoleCache6]

有三种类型的“过期类型”：“无”、“绝对”和“可调窗口”。这些类型配置如何使用“生存时间(分钟)”来确定有效期。默认的“过期类型”为“绝对”，这意味着在将项放入缓存中时，记录该项有效期的倒计时器即会启动。在项经过指定的时间后，该项过期。如果指定了“可调窗口”，则在每次访问缓存中的项时，会重置该项的有效期倒计时，并且仅在自上次访问该项后经过指定的一段时间后，该项才会过期。如果指定了“无”，则“生存时间(分钟)”必须设置为 **0**，并且项不会过期，只要它们在缓存中就会保持有效。

如果需要比在角色属性中配置的时间更长或更短的超时时间间隔，则可以在缓存中添加或更新项时，使用采用 **TimeSpan** 参数的 **Add** 和 **Put** 的重载来指定具体的持续时间。以下示例将字符串 **value** 添加到缓存中并按**item** 进行键控，超时为 30 分钟。

    // Add the string "value" to the cache, keyed by "item"
    cache.Add("item", "value", TimeSpan.FromMinutes(30));

若要查看缓存中的项的剩余超时时间间隔，可以使用**GetCacheItem** 方法来检索 **DataCacheItem**对象，该对象包含有关缓存中项的信息，其中包括剩余超时时间间隔。

    // Get a DataCacheItem object that contains information about
    // "item" in the cache. If there is no object keyed by "item" null
    // is returned. 
    DataCacheItem item = cache.GetCacheItem("item");
    TimeSpan timeRemaining = item.Timeout;

<a name="store-session"></a>

## 如何：在缓存中存储 ASP.NET 会话状态

用于角色中缓存的会话状态提供程序是适用于 ASP.NET 应用程序的进程外存储机制。此提供程序允许你将会话状态存储在 Azure 缓存中而非内存或 SQL Server 数据库中。若要使用 Caching 会话状态提供程序，请首先配置缓存群集，然后使用 Caching NuGet 包配置 ASP.NET 应用程序的缓存，如[角色中缓存入门][角色中缓存入门]中所述。在安装 Caching NuGet 包时，它会在 web.config 中添加一个包含 ASP.NET 应用程序所需配置的注释掉的节，以使用用于角色中缓存的会话状态提供程序。

    <!--Uncomment this section to use In-Role Cache for session state caching
    <system.web>
      <sessionState mode="Custom" customProvider="AFCacheSessionStateProvider">
        <providers>
          <add name="AFCacheSessionStateProvider" 
            type="Microsoft.Web.DistributedCache.DistributedCacheSessionStateStoreProvider,
                  Microsoft.Web.DistributedCache" 
            cacheName="default" 
            dataCacheClientName="default"/>
        </providers>
      </sessionState>
    </system.web>-->

> 如果在安装 Caching NuGet 包后 web.config 没有包含此注释掉的节，请确保从 [NuGet 包管理器安装][NuGet 包管理器安装]安装了最新的 NuGet 包管理器，然后卸载并重新安装该包。

若要启用用于角色中缓存的会话状态提供程序，请取消注释指定的节。在提供的代码段中指定了默认缓存。若要使用不同的缓存，请在 **cacheName** 属性中指定所需缓存。

有关使用 Caching 服务会话状态提供程序的详细信息，请参阅[用于角色中缓存的会话状态提供程序][用于角色中缓存的会话状态提供程序]。

<a name="store-page"></a>

## 如何：在缓存中存储 ASP.NET 页面输出缓存

用于角色中缓存的输出缓存提供程序是适用于输出缓存数据的进程外存储机制。此数据专门用于完整 HTTP响应（页面输出缓存）。此提供程序插入 ASP.NET 4 中引入的新输出缓存提供程序扩展点。若要使用该输出缓存提供程序，请首先配置缓存群集，然后使用 Caching NuGet 包配置 ASP.NET 应用程序的缓存，如[角色中缓存入门][角色中缓存入门]中所述。在安装 Caching NuGet 包时，它会在 web.config 中添加以下包含 ASP.NET 应用程序所需配置的注释掉的节，以使用用于角色中缓存的输出缓存提供程序。

    <!--Uncomment this section to use In-Role Cache for output caching
    <caching>
      <outputCache defaultProvider="AFCacheOutputCacheProvider">
        <providers>
          <add name="AFCacheOutputCacheProvider" 
            type="Microsoft.Web.DistributedCache.DistributedCacheOutputCacheProvider,
                  Microsoft.Web.DistributedCache" 
            cacheName="default" 
            dataCacheClientName="default" />
        </providers>
      </outputCache>
    </caching>-->

> 如果在安装 Caching NuGet 包后 web.config 没有包含此注释掉的节，请确保从 [NuGet 包管理器安装][NuGet 包管理器安装]安装了最新的 NuGet 包管理器，然后卸载并重新安装该包。

若要启用用于角色中缓存的输出缓存提供程序，请取消注释指定的节。在提供的代码段中指定了默认缓存。若要使用不同的缓存，请在 **cacheName** 属性中指定所需缓存。

将 **OutputCache** 指令添加到希望为其缓存输出的每个页面中。

    <%@ OutputCache Duration="60" VaryByParam="*" %>

在此示例中，缓存的页面数据将保留在缓存中 60 秒，并且将为每个参数组合缓存不同版本的页面。有关可用选项的详细信息，请参阅 [OutputCache 指令][OutputCache 指令]。

有关使用用于角色中缓存的输出缓存提供程序的详细信息，请参阅[用于角色中缓存的输出缓存提供程序][用于角色中缓存的输出缓存提供程序]。

<a name="next-steps"></a>

## 后续步骤

现在，你已了解角色中缓存的基础知识，请单击下面的链接了解如何执行更复杂的缓存任务。

-   查看 MSDN 参考：[角色中缓存][角色中缓存]
-   了解如何迁移到角色中缓存：[迁移到角色中缓存][迁移到角色中缓存]
-   查看示例：[角色中缓存示例][角色中缓存示例]
-   观看 TechEd 2013 上针对角色中缓存提供的[最高性能：使用 Azure Caching 让你的云服务应用程序提速][最高性能：使用 Azure Caching 让你的云服务应用程序提速]课程。

<!-- INTRA-TOPIC LINKS -->
<!-- IMAGES --> 
<!-- LINKS -->

  [后续步骤]: #next-steps
  [什么是角色中缓存？]: #what-is
  [角色中缓存入门]: #getting-started-cache-role-instance
  [配置缓存群集]: #enable-caching
  [配置缓存客户端]: #NuGet
  [使用缓存]: #working-with-caches
  [如何：创建 DataCache 对象]: #create-cache-object
  [如何：在缓存中添加和检索对象]: #add-object
  [如何：在缓存中指定对象的有效期]: #specify-expiration
  [如何：在缓存中存储 ASP.NET 会话状态]: #store-session
  [如何：在缓存中存储 ASP.NET 页面输出缓存]: #store-page
  [角色中缓存概述]: http://go.microsoft.com/fwlink/?LinkId=254172
  [RoleCache1]: ./media/cache-dotnet-how-to-use-in-role/cache8.png
  [RoleCache2]: ./media/cache-dotnet-how-to-use-in-role/cache9.png
  [RoleCache7]: ./media/cache-dotnet-how-to-use-in-role/cache14.png
  [RoleCache8]: ./media/cache-dotnet-how-to-use-in-role/cache15.png
  [RoleCache10]: ./media/cache-dotnet-how-to-use-in-role/cache17.png
  [角色中缓存容量规划注意事项]: http://go.microsoft.com/fwlink/?LinkId=252651
  [RoleCache3]: ./media/cache-dotnet-how-to-use-in-role/cache10.png
  [如何配置虚拟机大小]: http://go.microsoft.com/fwlink/?LinkId=164387
  [RoleCache4]: ./media/cache-dotnet-how-to-use-in-role/cache11.png
  [RoleCache5]: ./media/cache-dotnet-how-to-use-in-role/cache12.png
  [解决和诊断角色中缓存问题]: http://msdn.microsoft.com/en-us/library/windowsazure/hh914135.aspx
  [RoleCache6]: ./media/cache-dotnet-how-to-use-in-role/cache13.png
  [NuGet 包管理器安装]: http://go.microsoft.com/fwlink/?LinkId=240311
  [用于角色中缓存的会话状态提供程序]: http://msdn.microsoft.com/en-us/library/windowsazure/gg185668.aspx
  [OutputCache 指令]: http://go.microsoft.com/fwlink/?LinkId=251979
  [用于角色中缓存的输出缓存提供程序]: http://msdn.microsoft.com/en-us/library/windowsazure/gg185662.aspx
  [角色中缓存]: http://www.microsoft.com/en-us/showcase/Search.aspx?phrase=azure+caching
  [迁移到角色中缓存]: http://msdn.microsoft.com/en-us/library/hh914163.aspx
  [角色中缓存示例]: http://msdn.microsoft.com/en-us/library/jj189876.aspx
  [最高性能：使用 Azure Caching 让你的云服务应用程序提速]: http://channel9.msdn.com/Events/TechEd/NorthAmerica/2013/WAD-B326#fbid=kmrzkRxQ6gU
