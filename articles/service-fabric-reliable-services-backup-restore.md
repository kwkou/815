<properties
   pageTitle="Service Fabric 备份和还原 | Azure"
   description="Service Fabric 备份和还原的概念文档"
   services="service-fabric"
   documentationCenter=".net"
   authors="mcoskun"
   manager="timlt"
   editor="subramar,jessebenson"/>

<tags
   ms.service="service-fabric"
   ms.date="05/13/2016"
   wacn.date="07/04/2016"/>

# 备份和还原 Reliable Services 及 Reliable Actors

Azure Service Fabric 是一个高可用性平台，用于复制多个节点中的状态以维护此高可用性。因此，即使群集中的一个节点出现故障，服务也将继续可用。尽管此平台提供的此内置冗余对某些情况来说可能已足够用了，但在特定情况下，仍需要服务备份数据（到外部存储）。

>[AZURE.NOTE] 请务必备份和还原你的数据（并测试它是否正常运行），以便从数据丢失情形中恢复。

例如，在以下情况下，服务可能要备份数据：

* 永久失去运行指定分区的整个 Service Fabric 群集或所有节点。

* 状态被意外删除或受损而引起管理错误。例如，当具有足够权限的管理员错误地删除了服务时可能发生此情况。

* 服务中的 bug 导致数据损坏。例如，当某个服务代码升级程序开始将错误数据写入到可靠集合中时可能发生此情况。在此情况下，代码和数据可能都必须还原到先前的状态。

* 离线数据处理。对于商业智能来说使用离线处理的数据很方便，此处理是独立于生成数据的服务进行的。

备份/还原功能允许在 Reliable Services API 上构建的服务来创建和还原备份。由平台提供的备份 API 允许在不阻止读取或写入操作的情况下备份服务分区的状态。还原 API 允许从选定的备份还原服务分区的状态。

## 备份的类型

有两个备份选项：完整和增量。完整备份是包含重新创建副本状态所需的所有数据的备份：检查点和所有日志记录。因为它具有检查点和日志，所以完整备份可以自行还原。

当检查点较大时，会出现与完整备份有关的问题。例如，具有 16 GB 状态的副本的检查点合计大约有 16 GB。如果我们的恢复点目标是 5 分钟，则副本需要每 5 分钟备份一次。每次备份时，需要复制 16 GB 的检查点以及 50 MB（可使用 **CheckpointThresholdInMB** 进行配置）的日志。

![完整备份示例。](./media/service-fabric-reliable-services-backup-restore/FullBackupExample.PNG)

此问题的解决方案是增量备份，这种备份只备份自上次备份以来的日志记录。

![增量备份示例。](./media/service-fabric-reliable-services-backup-restore/IncrementalBackupExample.PNG)

由于增量备份只在自上次备份以来更改（不包括检查点），因此它们往往更快，但无法自行还原。若要还原增量备份，需要整个备份链。备份链是从完整备份开始，随后是一些连续增量备份的链。

## 备份 Reliable Services

服务创建者可完全控制何时进行备份，以及将备份存储在何处。

若要开始备份，服务需要调用继承的成员函数 **BackupAsync**。仅可从主副本创建备份，并且需要授予这些备份写入状态。

如下所示，**BackupAsync** 采用 **BackupDescription** 对象，用户可以在其中指定完整或增量备份，以及指定在本地创建备份文件夹并准备好移出到某个外部存储时调用的回叫函数 **Func<< BackupInfo, CancellationToken, Task<bool>>>**。

```C#

BackupDescription myBackupDescription = new BackupDescription(backupOption.Incremental,this.BackupCallbackAsync);

await this.BackupAsync(myBackupDescription);

```

进行增量备份的请求可能会失败，并出现 **FabricFullBackupMissingException**，这指示副本从未进行完整备份或是自上次备份以来的一些日志记录已截断。用户可以通过修改 **CheckpointThresholdInMB** 来修改截断速率。

**BackupInfo** 提供有关备份的信息，包括运行时保存备份的文件夹的位置 (**BackupInfo.Directory**)。此回叫函数可以将 **BackupInfo.Directory** 移到外部存储或其他位置。此函数也返回一个布尔值，此值表示是否已成功将备份文件夹移动到其目标位置。

以下代码演示如何使用 **BackupCallbackAsync** 方法将备份上载到 Azure 存储空间：

```C#
private async Task<bool> BackupCallbackAsync(BackupInfo backupInfo, CancellationToken cancellationToken)
{
    var backupId = Guid.NewGuid();

    await externalBackupStore.UploadBackupFolderAsync(backupInfo.Directory, backupId, cancellationToken);

    return true;
}
```

在上面的示例中，**ExternalBackupStore** 是用于与 Azure Blob 存储进行交互的示例类，**UploadBackupFolderAsync** 是压缩文件夹并将其放置在 Azure Blob 存储中的方法。

请注意：

- 在任何给定时间，每个副本只能有一项正在进行的备份操作。一次调用多个 **BackupAsync** 将引发 **FabricBackupInProgressException**，以此将正在进行的备份限制为一个。

- 如果备份正在进行时副本发生故障转移，那么备份可能没有完成。因此，故障转移完成后，服务应负责根据需要通过调用 **BackupAsync** 来重新启动备份。

## 还原 Reliable Services

一般而言，你可能需要执行还原操作的情况可以为以下类型之一：

- 服务分区丢失数据。例如，分区的三个副本中的两个（包括主副本）的磁盘数据已损坏或被擦除。新的主副本可能需要从备份中还原数据。

- 整个服务已丢失。例如，管理员删除了整个服务，因此需要还原此服务和数据。

- 服务复制了损坏的应用程序数据（例如，由于应用程序的 bug）。在此情况下，必须升级或还原服务来消除损坏的原因，并还原未损坏的数据。

虽然有许多恢复方法，但是我们仅针对上述情形提供一些使用 **RestoreAsync** 进行恢复的示例。

## Reliable Services 中的分区数据丢失

在此情况下，运行时将自动检测数据丢失并调用 **OnDataLossAsync** API。

服务创建者需要执行下列操作来恢复：

- 重写虚拟基类方法 **OnDataLossAsync**。

- 在包含服务的备份的外部位置中查找最新备份。

- 下载最新的备份（如果备份已压缩，则将其解压缩到备份文件夹中）。

- **OnDataLossAsync** 方法提供 **RestoreContext**。在提供的 **RestoreContext** 上调用 **RestoreAsync** API。

- 如果还原成功，则返回 true。

以下是 **OnDataLossAsync** 方法的实现示例：

```C#

protected override async Task<bool> OnDataLossAsync(RestoreContext restoreCtx, CancellationToken cancellationToken)
{
    var backupFolder = await this.externalBackupStore.DownloadLastBackupAsync(cancellationToken);

    var restoreDescription = new RestoreDescription(backupFolder);

    await restoreCtx.RestoreAsync(restoreDescription);

    return true;
}
```

传入到 **RestoreContext.RestoreAsync** 调用的 **RestoreDescription** 包含一个名为 **BackupFolderPath** 的成员。
还原单个完整备份时，此 **BackupFolderPath** 应设置为包含完整备份的文件夹的本地路径。
还原一个完整备份和一些增量备份时，**BackupFolderPath** 应设置为包含完整备份以及所有增量备份的文件夹的本地路径。
如果提供的 **BackupFolderPath** 不包含完整备份，则 **RestoreAsync** 调用可能会引发 **FabricFullBackupMissingException**。
如果 **BackupFolderPath** 具有断开的增量备份链，则它还可能会引发 **ArgumentException**。
例如，如果它包含完整备份、第一个增量备份和第三个增量备份，但不包含第二个增量备份。

>[AZURE.NOTE] 默认情况下，RestorePolicy 设置为安全。这表示如果检测到备份文件夹中包含的状态早于或等于此副本中包含的状态，则 **RestoreAsync** API 将因 ArgumentException 而失败。可以使用 **RestorePolicy.Force** 来跳过此安全检查。这会指定为 **RestoreDescription** 的一部分。

## 删除或丢失服务

如果删除了一个服务，则在还原数据之前必须首先重新创建此服务。请务必创建具有相同配置的服务，例如，相同的分区方案，以便可以无缝地还原数据。一旦启动服务，就必须在此服务的每个分区上调用用于还原数据的 API（上面的 **OnDataLossAsync**）。实现此操作的一种方法是在每个分区上使用 **[FabricClient.TestManagementClient.StartPartitionDataLossAsync](https://msdn.microsoft.com/zh-cn/library/mt693569.aspx)**。

从这个角度来看，实现操作与上述情况相同。每个分区需要从外部存储中还原最新的相关备份。值得注意的一点是，分区 ID 现在可能已更改，因为运行时是动态创建分区 ID。因此，此服务需要还原相应的分区信息和服务名称来标识每个分区要还原的正确的最新备份。

>[AZURE.NOTE] 不建议在每个分区上使用 **FabricClient.ServiceManager.InvokeDataLossAsync** 来还原整个服务，因为这可能会损坏群集状态。

## 复制损坏的应用程序数据

如果新部署的应用程序升级有一个 bug，则可能会导致数据损坏。例如，应用程序升级可能使用无效的区号更新可靠字典中的每个电话号码记录。在此情况下，将复制无效的电话号码，因为 Service Fabric 并不知道要存储的数据的性质。

在检测到这样一个导致数据损坏的严重 bug 之后首先要做的是在应用程序级别冻结服务，并且如果可以，请升级到没有此 bug 的应用程序代码版本。但是，即使修复了服务代码，数据仍可能是损坏的，因此可能需要还原数据。在此情况下，还原最新备份可能不足以解决问题，因为此最新备份可能已损坏。因此，你需要查找在数据被损坏之前创建的最新备份。

如果你不确定哪些备份已损坏，那么你可以部署一个新的 Service Fabric 群集，然后还原受影响分区的备份，正如上面的“删除或丢失服务”情况一样。针对每个分区，从最新备份到最旧备份开始还原。当你找到一个未被损坏的备份时，移动或删除此分区的比此备份更新的所有备份。对每个分区重复此过程。此时，如果在生产群集中的分区上调用 **OnDataLossAsync**，在外部存储中找到的最新备份将是通过上述过程选取的备份。

现在，可使用“删除或丢失服务”部分中的步骤来将服务的状态还原到错误代码损坏此状态之前的状态。

请注意：

- 还原时，要还原的备份状态可能早于数据丢失前分区的状态。因此，你只能将还原作为最后的办法来恢复尽可能多的数据。

- 表示备份文件夹路径和备份文件夹内的文件路径的字符串可以大于 255 个字符，具体取决于 FabricDataRoot 路径和应用程序类型名称的长度。这会导致某些 .NET 方法（如 **Directory.Move**）引发 **PathTooLongException** 异常。一种解决方法是直接调用 kernel32 API，如 **CopyFile**。

## 备份和还原 Reliable Actors

Reliable Actors 的备份和还原以 Reliable Services 提供的备份和还原功能为基础。服务所有者应创建一个派生自 **ActorService**（这是托管执行组件的 Service Fabric 可靠服务）的自定义执行组件服务，然后像前面几节所述的 Reliable Services 一样进行备份/还原。因为会在每个分区上进行备份，所以该特定分区中所有执行组件的状态都会进行备份（还原也一样会在每个分区上进行）。


- 创建自定义执行组件服务时，必须在注册执行组件时注册自定义执行组件服务。请参阅 **ActorRuntime.RegistorActorAsync**。
- **KvsActorStateProvider** 当前仅支持完整备份。而且 **KvsActorStateProvider** 会忽略 **RestorePolicy.Safe** 选项。

>[AZURE.NOTE] 默认 ActorStateProvider（即 **KvsActorStateProvider**）**不会**自己清理备份文件夹（在通过 ICodePackageActivationContext.WorkDirectory 获取的应用程序工作文件夹下）。这可能会导致填满工作文件夹。将备份移动到外部存储之后，应在备份回调中显式清理备份文件夹。


## 测试备份和还原

请务必确保关键数据正在进行备份，并可进行还原。为此，可在 PowerShell 中调用会导致特定分区丢失数据的 **Invoke-ServiceFabricPartitionDataLoss** cmdlet，以测试服务的数据备份和还原功能是否按预期运行。此外，也可以通过编程方式调用数据丢失，并从该事件中还原。

>[AZURE.NOTE] 你可以在 Github 上找到 Web 引用应用中备份和还原功能的实现示例。有关更多详细信息，请查看 Inventory.Service 服务。

## 表象之下：有关备份和还原的更多详细信息

下面提供了有关备份和还原的更多详细信息。

### 备份
可靠性状态管理器具有在不阻止任何读取或写入操作的情况下创建一致的备份的功能。要实现此功能，它利用了检查点和日志持久性机制。可靠性状态管理器在特定时间点采用模糊（轻型）检查点来缓解来自事务日志的压力，并缩短恢复时间。调用 **BackupAsync** 时，可靠状态管理器指示所有可靠对象将其最新的检查点文件复制到本地备份文件夹。然后，可靠性状态管理器将复制所有日志记录（从“开始指针”开始到最新的日志记录）到备份文件夹中。由于所有日志记录直至最新日志记录都包含在备份中，并且可靠状态管理器保留了预写日志记录，因此，可靠状态管理器保证所有提交的事务（**CommitAsync** 已成功返回）都包含在备份中。

在调用 **BackupAsync** 之后提交的任何事务可能在备份中，也可能不在。一旦平台填充此本地备份文件夹（即由运行时完成本地备份），就将调用此服务的备份回调。此回调负责将此备份文件夹移至 Azure 存储空间等外部位置。

### 还原

可靠状态管理器具有使用 **RestoreAsync** API 从备份还原的功能。  
只能在 **OnDataLossAsync** 方法内部调用 **RestoreContext** 上的 **RestoreAsync** 方法。
由 **OnDataLossAsync** 返回的布尔值指示服务是否从外部源还原其状态。
如果 **OnDataLossAsync** 返回 true，则 Service Fabric 将使用此主副本重新生成所有其他副本。Service Fabric 可确保将接收 **OnDataLossAsync** 调用的副本先转换为主角色，但不会授予其读取状态或写入状态。
这意味着对于 StatefulService 实现程序，不会调用 **RunAsync**，直到 **OnDataLossAsync** 成功完成。
然后，将在新的主副本上调用 **OnDataLossAsync**。
将一次调用一回 API，如此循环，直到服务成功完成此 API（返回 true 或 false），并完成相关的重新配置。

**RestoreAsync** 首先删除调用它的主副本中的所有现有状态。  
之后，可靠状态管理器创建备份文件夹中存在的所有可靠对象。  
接下来，指示可靠对象从备份文件夹中的它们的检查点还原。  
最后，可靠性状态管理器从备份文件夹中的日志记录中恢复其自己的状态，并执行恢复。  
作为恢复过程的一部分，将对可靠对象重播从“起始点”开始的操作，该起始点已提交备份文件夹中的日志记录。  
此步骤可以确保恢复的状态是一致的。

<!---HONumber=Mooncake_0523_2016-->