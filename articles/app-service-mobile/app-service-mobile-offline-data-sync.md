<properties
	pageTitle="Azure 移动应用中的脱机数据同步 | Azure"
	description="Azure 移动应用脱机数据同步功能的概念参考和概述"
	documentationCenter="windows"
	authors="adrianhall"
	manager="dwrede"
	editor=""
	services="app-service\mobile"/>

<tags
	ms.service="app-service-mobile"
	ms.workload="mobile"
	ms.tgt_pltfrm="na"
	ms.devlang="multiple"
	ms.topic="article"
	ms.date="10/30/2016"
	wacn.date="01/23/2017"
	ms.author="adrianha"/>

# Azure 移动应用中的脱机数据同步

## 什么是脱机数据同步？

脱机数据同步是 Azure 移动应用的客户端和服务器 SDK 功能，可让开发人员创建不需要网络连接就能正常运行的应用。

应用处于脱机模式时，你仍可创建和修改数据，并且数据将保存到本地存储。当应用重新联机时，即可将本地更改与 Azure 移动应用后端同步。此功能还支持冲突检测（当同一条记录同时在客户端和后端发生更改时，即会发生冲突）。可以在服务器或客户端上处理冲突。

脱机同步具有几个优点：

* 通过缓存在设备上的本地服务器数据来提高应用程序响应能力
* 创建当网络出问题时仍可使用的稳健应用
* 允许最终用户创建和修改数据，甚至在没有网络访问权限，并支持方案具有很少或没有连接时
* 跨多个设备同步数据和同一个记录修改由两个设备时检测冲突
* 在高延迟状态下或者在流量计费网络中限制网络使用

以下教程说明如何使用 Azure 移动应用将脱机同步添加到移动客户端：

* [Android: Enable offline sync]（Android：启用脱机同步）
* [Apache Cordova：启用脱机同步](/documentation/articles/app-service-mobile-cordova-get-started-offline-data/)
* [iOS: Enable offline sync]（iOS：启用脱机同步）
* [Xamarin iOS: Enable offline sync]（Xamarin iOS：启用脱机同步）
* [Xamarin Android: Enable offline sync]（Xamarin Android：启用脱机同步）
* [Xamarin.Forms: Enable offline sync（Xamarin.Forms：启用脱机同步）](/documentation/articles/app-service-mobile-xamarin-forms-get-started-offline-data/)
* [通用 Windows 平台：启用脱机同步]

## 什么是同步表？
为了访问 "/tables" 终结点，Azure 移动客户端 SDK 提供 `IMobileServiceTable`（.NET 客户端 SDK）或 `MSTable`（iOS 客户端）等接口。这些 API 直接连接到 Azure 移动应用后端，如果客户端设备没有网络连接，则会失败。

若要支持脱机使用，应用应该改用*同步表* API，例如 `IMobileServiceSyncTable`（.NET 客户端 SDK）或 `MSSyncTable`（iOS 客户端）。所有相同的 CRUD 操作（Create、Read、Update、Delete）都适用于同步表 API，不过它们现在从本地存储读取或者向其写入数据。在执行任何同步表操作之前，必须先初始化本地存储。

## 什么是本地存储？
本地存储是客户端设备上的数据持久层。Azure 移动应用客户端 SDK 提供默认的本地存储实现。在 Windows、Xamarin 和 Android 上，它基于 SQLite。在 iOS 上，它基于核心数据。

若要在 Windows Phone 或 Windows 应用商店 8.1 中使用基于 SQLite 的实现，需要安装 SQLite 扩展。有关详细信息，请参阅[通用 Windows 平台：启用脱机同步]。Android 和 iOS 设备的操作系统本身包含 SQLite 版本，因此不需要引用自己的 SQLite 版本。

开发人员也可以实现自己的本地存储。例如，如果希望将数据以加密格式存储在移动客户端上，可以定义使用 SQLCipher 进行加密的本地存储。

## 什么是同步上下文？
*同步上下文*与移动客户端对象（例如 `IMobileServiceClient` 或 `MSClient`）关联，跟踪对同步表所做的更改。同步上下文维护操作队列，其中保留了 CUD 操作（Create、Update、Delete）的顺序列表，该列表稍后将发送到服务器。

本地存储使用初始化方法（例如 [.NET 客户端 SDK] 中的 `IMobileServicesSyncContext.InitializeAsync(localstore)`）来与同步上下文关联。

## <a name="how-sync-works"></a>脱机同步的工作原理
使用同步表时，客户端代码将控制本地更改与 Azure 移动应用后端同步的时机。在发生*推送*本地更改的调用之前，不会向后端发送任何内容。同样，仅当发生了*提取*数据的调用时，才在本地存储中填充新数据。

* **推送**：推送是对同步上下文的操作，发送自上一次推送之后的所有 CUD 更改。请注意，无法做到只发送单个表的更改，否则操作发送顺序可能出错。推送对 Azure 移动应用后端执行一系列 REST 调用，而这会修改服务器数据库。
* **提取**：提取是根据每个表执行的，可以使用查询来自定义，以便只检索服务器数据的子集。然后，Azure 移动客户端 SDK 会将最终数据插入本地存储。
* **隐式推送**：如果提取是针对包含挂起本地更新的表执行的，则提取操作先对同步上下文执行 `push()`。此推送有助于最小化已排队的更改与服务器中新数据之间的冲突。
* **增量同步**：提取操作的第一个参数是*查询名称*，此参数只在客户端上使用。如果使用非 null 查询名称，Azure Mobile SDK 将执行增量同步。每当提取操作返回结果集，该结果集中最新的 `updatedAt` 时间戳将存储在 SDK 本地系统表中。后续提取操作只检索该时间戳以后的记录。

  若要使用增量同步，服务器必须返回有意义的 `updatedAt` 值，并且必须支持按此字段排序。但是，由于 SDK 在 updatedAt 字段中添加了自身的排序，因此无法使用本身具有 `orderBy` 子句的提取查询。

  查询名称可以是所选的任何字符串，但是对应用中的每个逻辑查询必须保持唯一。否则，不同的提取操作可能覆盖相同的增量同步时间戳，查询可能因此返回错误的结果。

  如果查询具有参数，创建唯一查询名称的方法之一是包含该参数值。例如，如果要按 userid 筛选，可以使用如下所示的查询名称（在 C# 中）：

        await todoTable.PullAsync("todoItems" + userid,
            syncTable.Where(u => u.UserId == userid));

  如果想要选择退出增量同步，请将 `null` 作为查询 ID 传递。在此情况下，在对 `PullAsync` 的每次调用中将检索所有记录，从而可能会降低效率。
* **清除**：可以使用 `IMobileServiceSyncTable.PurgeAsync` 清除本地存储的内容。如果客户端数据库包含陈旧数据，或者想要丢弃所有挂起的更改，可能需要执行清除操作。

  清除操作将从本地存储中清除表。如果有操作等待与服务器数据库进行同步，则清除将引发异常，除非设置了 force purge 参数。

  客户端包含陈旧数据的示例：假设在“待办事项列表”示例中，Device1 只提取未完成的项。“购买牛奶”待办事项通过其他设备在服务器上标记为已完成。但是，Device1 在本地存储中仍有“购买牛奶”待办事项，因为它只提取未标记为完成的项。清除操作会清除这条陈旧项。

## 后续步骤

* [iOS: Enable offline sync]（iOS：启用脱机同步）
* [Xamarin iOS: Enable offline sync]（Xamarin iOS：启用脱机同步）
* [Xamarin Android: Enable offline sync]（Xamarin Android：启用脱机同步）
* [通用 Windows 平台：启用脱机同步]

<!-- Links -->
[.NET 客户端 SDK]: /documentation/articles/app-service-mobile-dotnet-how-to-use-client-library/
[Android: Enable offline sync]: /documentation/articles/app-service-mobile-android-get-started-offline-data/
[iOS: Enable offline sync]: /documentation/articles/app-service-mobile-ios-get-started-offline-data/
[Xamarin iOS: Enable offline sync]: /documentation/articles/app-service-mobile-xamarin-ios-get-started-offline-data/
[Xamarin Android: Enable offline sync]: /documentation/articles/app-service-mobile-xamarin-ios-get-started-offline-data/
[通用 Windows 平台：启用脱机同步]: /documentation/articles/app-service-mobile-windows-store-dotnet-get-started-offline-data/

<!---HONumber=Mooncake_0116_2017-->
<!--Update_Description:update wording-->
