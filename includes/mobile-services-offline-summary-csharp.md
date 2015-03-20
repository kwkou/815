为了支持移动服务的脱机功能，我们使用了  `IMobileServiceSyncTable` 接口，并使用本地存储初始化了  `MobileServiceClient.SyncContext`。在这种情况下，本地存储是一个 SQLite 数据库。

移动服务的普通 CRUD 操作执行起来就像此应用仍处于连接状态一样，但所有操作都针对本地存储进行。

我们想要将本地存储与服务器同步时，使用  `IMobileServiceSyncTable.PullAsync` 和  `MobileServiceClient.SyncContext.PushAsync` 方法。

*  为了将更改推送到服务器中，我们调用了  `IMobileServiceSyncContext.PushAsync()`。此方法属于  `IMobileServicesSyncContext`（而不是同步表），因为它会将更改推送到所有表中。

    只有已在本地以某种方式修改（通过 CUD 操作来完成）的记录才会发送到服务器。
   
* 为了从服务器上的表中将数据拉取到应用，我们调用了  `IMobileServiceSyncTable.PullAsync`。

    拉取时始终先发出推送操作。这是为了确保本地存储中的所有表以及关系都保持一致。

    还存在  `PullAsync()` 的过载，这允许指定查询以过滤客户端上存储的数据。如果未传递查询， `PullAsync()` 将拉出对应表（或查询）中的所有行。您可以传递此查询以仅过滤您的应用需要同步的更改。

* 要启用增量同步，请将查询 ID 传递到  `PullAsync()`。查询 ID 用于存储来自上一拉动操作结果的上一更新时间戳记。查询 ID 应该是一个描述性字符串，该字符串对于您应用中的每个逻辑查询都是唯一的。如果查询具有一个参数，那么相同参数值必须是查询 ID 的一部分。

    例如，如果要对 userid 进行过滤，它需要是查询 ID 的一部分：

        await PullAsync("todoItems" + userid, syncTable.Where(u => u.UserId = userid));

    如果您想要选择退出增量同步，请将  `null` 作为查询 ID 进行传递。在此情况下，在对  `PullAsync` 的每次调用中检索所有记录，这可能效率低下。

* 在移动服务数据库中删除记录后，要从设备本地存储除去记录，应启用[软删除]。否则，您的应用应定期调用  `IMobileServiceSyncTable.PurgeAsync()` 以清除本地存储。
