<properties linkid="develop-net-tutorials-multi-tier-web-site-4-worker-role-a" pageTitle="Azure 云服务教程：辅助角色和 Azure 存储表、队列、Blob" metaKeywords="Azure tutorial, Azure storage tutorial, Azure multi-tier tutorial, Azure worker role tutorial, Azure blobs tutorial, Azure tables tutorial, Azure queues tutorial" description="了解如何使用 ASP.NET MVC 和 Azure 创建多层应用程序。该应用程序运行在云服务中，使用 Web 角色和辅助角色，并使用 Azure 存储表、队列和 Blob。" metaCanonical="" services="cloud-services,storage" documentationCenter=".NET" title="Azure 云服务教程：ASP.NET MVC Web 角色、辅助角色、Azure 存储表、队列和 Blob" authors="tdykstra,riande" solutions="" manager="wpickett" editor="mollybos" />

# 构建 Azure 电子邮件服务应用程序的辅助角色 A（电子邮件计划程序）- 第 4 部分（共 5 部分）。

这是分为五部分的一系列教程的第四个教程，这些教程讲述如何生成和部署 Azure 电子邮件服务示例应用程序。有关此应用程序和教程系列的信息，请参阅[本系列第一个教程][本系列第一个教程]。

在本教程中，你将学习：

-   如何查询和更新 Azure 存储表。
-   如何将工作项添加到队列供另一个辅助角色进行处理。
-   如何通过覆盖 `OnStop` 方法处理计划的关闭操作。
-   如何通过确保没有电子邮件漏发且没有重复发送电子邮件来处理非计划的关闭操作。
-   如何通过 Azure 存储资源管理器来测试使用 Azure 存储表的辅助角色。

创建云服务项目时，你已经创建辅助角色 A 项目。因此，你现在只需对辅助角色执行编程操作，并将其配置为使用你的 Azure 存储帐户。

## 本教程的章节

-   [添加对 Web 项目的引用][添加对 Web 项目的引用]
-   [添加对 SCL 1.7 程序集的引用][添加对 SCL 1.7 程序集的引用]
-   [添加 SendEmail 模型][添加 SendEmail 模型]
-   [添加辅助角色启动时运行的代码][添加辅助角色启动时运行的代码]
-   [配置存储空间连接字符串][配置存储空间连接字符串]
-   [测试辅助角色 A][测试辅助角色 A]
-   [后续步骤][后续步骤]

## <a name="addref"></a><span class="short-header">添加项目引用</span>添加对 Web 项目的引用

你需要对 Web 项目的引用，因为实体类是在该项目中定义的。你需要在辅助角色 B 中使用相同的实体类在应用程序所使用的 Azure 表中进行数据读写操作。

**注意：**在生产应用程序中，你不需要通过辅助角色项目设置对 Web 项目的引用，因为这会导致引用一系列你不希望或不需要在辅助角色中使用的依赖性程序集。通常情况下，你会将共享模型类保存在类库项目中，让 Web 角色项目和辅助角色项目引用该类库项目。为了简化解决方案结构，本教程将模型类存储在 Web 项目中。

1.  右键单击 WorkerRoleA 项目，然后选择**“添加引用”**。

    ![在 WorkerRoleA 项目中添加引用][在 WorkerRoleA 项目中添加引用]

2.  在**引用管理器**中，将引用添加到 MvcWebRole 项目中（或 Web 应用程序项目中，如果你正在 Azure 网站中运行 Web UI），然后单击**“确定”**。

    ![将引用添加到 MvcWebRole][将引用添加到 MvcWebRole]

## <a name="addref2"></a><span class="short-header">添加 SCL 1.7 引用</span>添加对 SCL 1.7 程序集的引用

> [WACOM.NOTE] 如果你已安装 SDK 2.3 或更高版本，请跳过此步骤。

存储客户端库 (SCL) 2.0 版并没有进行诊断所需的所有内容，因此你必须添加对某个 1.7 版程序集的引用。如果你遵循了前面教程的步骤，你已经完成了此操作。在此处添加说明是考虑到你可能漏掉了该步骤。

1.  右键单击 WorkerRoleA 项目，然后选择**“添加引用”**。

2.  单击对话框底部的**“浏览...”**按钮。

3.  导航到以下文件夹：

        C:\Program Files\Microsoft SDKs\Windows Azure\.NET SDK012-10\ref

4.  选择 *Microsoft.WindowsAzure.StorageClient.dll*，然后单击**“添加”**。

5.  在**“引用管理器”**对话框中，单击**“确定”**。

## <a name="addmodel"></a><span class="short-header">添加 SendEmail 模型</span>添加 SendEmail 模型

辅助角色 A 在 `Message` 表中创建 `SendEmail` 行，辅助角色 B 则读取这些行，以便获取发送电子邮件所需的信息。下图显示了 `Message` 表中两个 `Message` 行和三个 `SendEmail` 行的部分属性。

![使用 sendmail 的邮件表][使用 sendmail 的邮件表]

`Message` 表中的这些行有多个用途：

-   提供辅助角色 B 发送单封电子邮件所需的所有信息。
-   这些行跟踪某封电子邮件是否已发送，这样是为了防止在辅助角色因故障而重新启动的情况下重复发送电子邮件。
-   这样辅助角色 A 就能够确定某个消息的所有电子邮件在何时发送完毕，以便将其状态标记为 `Complete`。

读写 `SendEmail` 行需要一个模型类。由于该模型类必须可供辅助角色 A 和 B 访问，而所有其他的模型类都是在 Web 项目中定义的，因此也应在 Web 项目中定义该模型类。

1.  在**解决方案资源管理器**中，右键单击 Web 项目中的 Models 文件夹，然后选择**“添加现有项”**。

    ![将现有项添加到 Web 项目中的 Models 文件夹][将现有项添加到 Web 项目中的 Models 文件夹]

2.  导航到下载了示例应用程序的文件夹，选择 Web 项目 Models 文件夹中的 *SendEmail.cs* 文件，然后单击“添加”。

3.  打开 *SendEmail.cs* 并检查代码。

        public class SendEmail : TableEntity
        {
            public long MessageRef { get; set; }
            public string EmailAddress { get; set; }
            public DateTime? ScheduledDate { get; set; }
            public String FromEmailAddress { get; set; }
            public string SubjectLine { get; set; }
            public bool? EmailSent { get; set; }
            public string SubscriberGUID { get; set; }
            public string ListName { get; set; }
        }

    此处的代码类似于其他模型类，只是没有包括 DataAnnotations 属性，因为没有与该模型关联的 UI -- 它不用在 MVC 控制器中。

## <a name="addcode"></a><span class="short-header">添加辅助角色代码</span>添加辅助角色启动时运行的代码

1.  在 WorkerRoleA 项目中，打开 *WorkerRole.cs* 并检查代码。

        public class WorkerRole : RoleEntryPoint
        {
            public override void Run()
            {
                // This is a sample worker implementation. Replace with your logic.
                Trace.WriteLine("WorkerRole1 entry point called", "Information");

                while (true)
                {
                    Thread.Sleep(10000);
                    Trace.WriteLine("Working", "Information");
                }
            }

            public override bool OnStart()
            {
                // Set the maximum number of concurrent connections 
                ServicePointManager.DefaultConnectionLimit = 12;

                // For information on handling configuration changes
                // see the MSDN topic at http://go.microsoft.com/fwlink/?LinkId=166357.

                return base.OnStart();
            }
        }

    这是辅助角色的默认模板代码。这里有一个 `OnStart` 方法，你可以将仅当辅助角色的实例启动时才会运行的初始化代码置于其中；此外还有一个 `Run` 方法，该方法在 `OnStart` 方法完成后调用。将此代码替换为你自己的初始化代码，然后运行代码。

2.  删除 *WorkerRole.cs*，右键单击 WorkerRoleA 项目，然后选择**“添加现有项”**。

    ![将现有项添加到辅助角色 A][将现有项添加到辅助角色 A]

3.  导航到下载了示例应用程序的文件夹，选择 WorkerRoleA 项目中的 *WorkerRoleA.cs* 文件，然后单击“添加”。

4.  打开 *WorkerRoleA.cs* 并检查代码。

    `OnStart` 方法用于初始化你在使用 Azure 存储实体时需要的上下文对象。它还可确保你要在 `Run` 方法中使用的所有表、队列和 Blob 容器都存在。执行这些任务的代码类似于你在前面看到的 MVC 控制器构造函数中的代码。你将配置此方法随后要用到的连接字符串。

        public override bool OnStart()
        {
            ServicePointManager.DefaultConnectionLimit = Environment.ProcessorCount;

            ConfigureDiagnostics();
            Trace.TraceInformation("Initializing storage account in WorkerA");
            var storageAccount = CloudStorageAccount.Parse(RoleEnvironment.GetConfigurationSettingValue("StorageConnectionString"));

            CloudQueueClient queueClient = storageAccount.CreateCloudQueueClient(); 
            sendEmailQueue = queueClient.GetQueueReference("azuremailqueue"); 
            var tableClient = storageAccount.CreateCloudTableClient(); 
            mailingListTable = tableClient.GetTableReference("mailinglist"); 
            messageTable = tableClient.GetTableReference("message"); 
            messagearchiveTable = tableClient.GetTableReference("messagearchive"); 

            // Create if not exists for queue, blob container, SentEmail table. 
            sendEmailQueue.CreateIfNotExists(); 
            messageTable.CreateIfNotExists(); 
            mailingListTable.CreateIfNotExists(); 
            messagearchiveTable.CreateIfNotExists(); 

            return base.OnStart();
        }

    你可能已在以前看到过有关如何使用 Azure 存储空间的文档，了解到循环中的初始化代码可以用来检查传输错误。而现在，这种方式已不再需要，因为 API 的内置重试机制会消除一过性的网络故障，最多可允许 3 次额外的尝试。

    `ConfigureDiagnostics` 方法由 `OnStart` 方法调用，可用于设置跟踪，让你能够查看 `Trace.Information` 和 `Trace.Error` 方法的输出。此方法在[第二个教程][第二个教程]中介绍。

    `OnStop` 方法将全局变量 `onStopCalled` 设置为 true，然后等待 `Run` 方法将全局变量 `returnedFromRunMethod` 设置为 true，此时表明它已准备好进行干净的关闭操作。

        public override void OnStop()
        {
            onStopCalled = true;
            while (returnedFromRunMethod == false)
            {
                System.Threading.Thread.Sleep(1000);
            }
        }

    当辅助角色因下述原因之一而关闭时，将调用 `OnStop` 方法：

    -   Azure 需要重新启动虚拟机（Web 角色或辅助角色实例）或托管虚拟机的物理机。
    -   你已经使用 Azure 管理门户上的“停止”按钮停止了云服务。
    -   你已将更新部署到云服务项目。

    `Run` 方法用于监视变量 `onStopCalled`，当该变量变为 `true` 时，它就会停止拉取要处理的新工作项。`OnStop` 和 `Run` 方法之间的这种协调方式使得辅助进程的关闭能够正常进行。

    Azure 会定期安装操作系统更新，以确保该平台既安全又可靠，并且性能优异。这些更新通常要求托管你的云服务的计算机关机并重新启动。有关详细信息，请参阅[重新启动角色实例进行 OS 升级][重新启动角色实例进行 OS 升级]。

    `Run` 方法可执行两种功能：

    -   扫描 `message` 表以查找计划在今天或更早时候发送的邮件（尚未为其创建队列工作项）。

    -   扫描 `message` 表以查找其状态指示已创建所有队列工作项但并没有发送所有电子邮件的邮件。如果它找到一个，则会扫描 `SendEmail` 行中是否存在该邮件以了解是否已发送所有电子邮件，如果已全部发送，则将状态更新为 `Completed` 并存档 `message` 行。

    该方法也查看全局变量 `onStopCalled`。当变量为 `true` 时，该方法会停止拉取要处理的新工作项，然后在已启动的任务完成时返回。

        public override void Run()
        {
            Trace.TraceInformation("WorkerRoleA entering Run()");
            while (true)
            {
                try
                {
                    var tomorrow = DateTime.Today.AddDays(1.0).ToString("yyyy-MM-dd");
                    // If OnStop has been called, return to do a graceful shutdown.
                    if (onStopCalled == true)
                    {
                        Trace.TraceInformation("onStopCalled WorkerRoleB");
                        returnedFromRunMethod = true;
                        return;
                    }
                    // Retrieve all messages that are scheduled for tomorrow or earlier
                    // and are in Pending or Queuing status.
                    string typeAndDateFilter = TableQuery.CombineFilters(
                        TableQuery.GenerateFilterCondition("RowKey", QueryComparisons.GreaterThan, "message"),
                        TableOperators.And,
                        TableQuery.GenerateFilterCondition("PartitionKey", QueryComparisons.LessThan, tomorrow));
                    var query = (new TableQuery<Message>().Where(typeAndDateFilter));
                    var messagesToProcess = messageTable.ExecuteQuery(query).ToList();
                    TableOperation replaceOperation;
                    // Process each message (queue emails to be sent).
                    foreach (Message messageToProcess in messagesToProcess)
                    {
                        string restartFlag = "0";
                        // If the message is already in Queuing status,
                        // set flag to indicate this is a restart.
                        if (messageToProcess.Status == "Queuing")
                        {
                            restartFlag = "1";
                        }

                        // If the message is in Pending status, change
                        // it to Queuing.
                        if (messageToProcess.Status == "Pending")
                        {
                            messageToProcess.Status = "Queuing";
                            replaceOperation = TableOperation.Replace(messageToProcess);
                            messageTable.Execute(replaceOperation);
                        }

                        // If the message is in Queuing status, 
                        // process it and change it to Processing status;
                        // otherwise it's already in processing status, and 
                        // in that case check if processing is complete.
                        if (messageToProcess.Status == "Queuing")
                        {
                            ProcessMessage(messageToProcess, restartFlag);

                            messageToProcess.Status = "Processing";
                            replaceOperation = TableOperation.Replace(messageToProcess);
                            messageTable.Execute(replaceOperation);
                        }
                        else
                        {
                            CheckAndArchiveIfComplete(messageToProcess);
                        }
                    }

                    // Sleep for one minute to minimize query costs. 
                    System.Threading.Thread.Sleep(1000 * 60);
                }
                catch (Exception ex)
                {
                    string err = ex.Message;
                    if (ex.InnerException != null)
                    {
                        err += " Inner Exception: " + ex.InnerException.Message;
                    }
                    Trace.TraceError(err);
                    // Don't fill up Trace storage if we have a bug in queue process loop.
                    System.Threading.Thread.Sleep(1000 * 60);
                }
            }
        }

    请注意，所有工作都是在 `while` 块的无限循环中完成的，而 `while` 块中的所有代码都是在 `try`-`catch` 块中包装的，目的是防止未经处理的异常。如果发生未经处理的异常，Azure 会引发 [UnhandledException][UnhandledException] 事件，辅助进程将终止，该角色进入脱机状态。Azure 将重新启动辅助角色，但这需要几分钟。`try` 块调用 `TraceError` 来记录错误，然后休眠 60 秒，这样如果错误持续存在，就不会重复发出太多次数的错误消息。在生产应用程序中，你可以在 `try` 块中向管理员发送电子邮件。

    `Run` 方法用于处理 `message` 表中其计划日期为明天以前的 `message` 行的查询：

                    // Retrieve all messages that are scheduled for tomorrow or earlier
                    // and are in Pending or Queuing status.
                    string typeAndDateFilter = TableQuery.CombineFilters(
                        TableQuery.GenerateFilterCondition("RowKey", QueryComparisons.GreaterThan, "message"),
                        TableOperators.And,
                        TableQuery.GenerateFilterCondition("PartitionKey", QueryComparisons.LessThan, tomorrow));
                    var query = (new TableQuery<Message>().Where(typeAndDateFilter));
                    var messagesToProcess = messageTable.ExecuteQuery(query).ToList();

    **注意：**在处理完邮件行后将其移到 `messagearchive` 表的一个好处是，这种查询只需指定 `PartitionKey` 和 `RowKey` 作为搜索条件。如果我们没有存档已处理的行，则该查询还必须指定一个非键字段（`Status`），而且必须搜索更多的行。表大小将会增加，查询将需要更长的时间，并且可能开始收到继续标记。

    如果邮件处于 `Pending` 状态，则处理尚未开始；如果邮件处于 `Queuing` 状态，则处理已在此前的某个时间开始，但在创建所有队列消息之前被中断。在这种情况下，则必须在需要发送某封电子邮件的辅助角色 B 中进行额外的检查，以确保该电子邮件此前尚未被发送。这种情况下需要使用 `restartFlag` 变量。

                        string restartFlag = "0";
                        if (messageToProcess.Status == "Queuing")
                        {
                            restartFlag = "1";
                        }

    接下来，代码将 `message` 行从 `Pending` 状态设置为 `Queuing` 状态。然后，对于上述行以及已处于 `Queuing` 状态的任何行，代码会调用 `ProcessMessage` 方法来创建用于为消息发送电子邮件的队列工作项。

                        if (messageToProcess.Status == "Pending")
                        {
                            messageToProcess.Status = "Queuing";
                            replaceOperation = TableOperation.Replace(messageToProcess);
                            messageTable.Execute(replaceOperation);
                        }

                        if (messageToProcess.Status == "Queuing")
                        {
                            ProcessMessage(messageToProcess, restartFlag);

                            messageToProcess.Status = "Processing";
                            replaceOperation = TableOperation.Replace(messageToProcess);
                            messageTable.Execute(replaceOperation);
                        }
                        else
                        {
                            CheckAndArchiveIfComplete(messageToProcess);
                        }

    处理完一个处于 `Queuing` 状态的邮件以后，代码将 `Message` 行的状态设置为 `Processing`。`message` 表中不处于 `Pending` 或 `Queuing` 状态的行已经处于 `Processing` 状态，对于这些行，代码会调用一个方法来检查该消息的所有电子邮件是否已发送。如果所有电子邮件已发送，则会存档 `message` 行。

    处理完查询所检索的所有记录后，代码会休眠一分钟。

                // Sleep for one minute to minimize query costs.
                System.Threading.Thread.Sleep(1000*60);

    每次 Azure 存储查询都有一个最低收费，即使该查询没有返回任何数据，因此持续反复扫描会给你增加不必要的 Azure 费用。撰写本教程时，每百万事务的费用是 $0.10（一个查询计为一个事务），因此可以将休眠时间设置为远远不到一分钟，而扫描表中是否存在要发送的邮件的费用仍然是最低收费。有关定价的详细信息，请参阅[第一个教程][本系列第一个教程]。

    **有关线程和最佳 CPU 利用率的说明：**`Run` 方法中存在两个任务（对电子邮件排队以及查看已完成的电子邮件），这两个任务在单个线程中按顺序运行。小型虚拟机 (VM) 有 1.75 GB 的 RAM 但只有一个 CPU，因此使用单个线程按顺序运行这些任务可能并没有问题。假设你的应用程序需要比小型 VM 所能提供的内存更多的内存才能高效运行。中型 VM 提供 3.5 GB 的 RAM 和 2 个 CPU，但此应用程序只会使用一个 CPU，因为它是单线程的。若要利用所有 Cpu，你需要为每个 CPU 创建一个工作线程。即便如此，单个 CPU 也并没有被单个线程充分利用。当某个线程进行网络或 I/O 调用时，该线程必须等待 I/O 或网络调用完成，而在等待期间，它不会做什么有用的工作。如果执行使用两个线程的 `Run` 方法，则当一个线程等待网络或 I/O 操作完成时，另一个线程可以做有用的工作。

    `ProcessMessage` 方法用于获取目标电子邮件列表的所有电子邮件地址，并为每个电子邮件地址创建一个队列工作项。在创建队列工作项时，它还会在 `Message` 表中创建 `SendEmail` 行。这些行向辅助角色 B 提供发送电子邮件所需的信息，并且包含一个用于跟踪每封电子邮件是否已发送的 `EmailSent` 属性。

        private void ProcessMessage(Message messageToProcess, string restartFlag)
        {
            // Get Mailing List info to get the "From" email address.
            var retrieveOperation = TableOperation.Retrieve<MailingList>(messageToProcess.ListName, "mailinglist");
            var retrievedResult = mailingListTable.Execute(retrieveOperation);
            var mailingList = retrievedResult.Result as MailingList;
            if (mailingList == null)
            {
                Trace.TraceError("Mailing list not found: " + messageToProcess.ListName + " for message: " + messageToProcess.MessageRef);
                return;
            }
            // Get email addresses for this Mailing List.
            string filter = TableQuery.CombineFilters(
               TableQuery.GenerateFilterCondition("PartitionKey", QueryComparisons.Equal, messageToProcess.ListName),
               TableOperators.And,
               TableQuery.GenerateFilterCondition("RowKey", QueryComparisons.NotEqual, "mailinglist"));
            var query = new TableQuery<Subscriber>().Where(filter);
            var subscribers = mailingListTable.ExecuteQuery(query).ToList();

            foreach (Subscriber subscriber in subscribers)
            {
                // Verify that the subscriber email address has been verified.
                if (subscriber.Verified == false)
                {
                    Trace.TraceInformation("Subscriber " + subscriber.EmailAddress + " not Verified, so not queuing ");
                    continue;
                }

                // Create a SendEmail entity for this email.              
                var sendEmailRow = new SendEmail
                {
                    PartitionKey = messageToProcess.PartitionKey,
                    RowKey = messageToProcess.MessageRef.ToString() + subscriber.EmailAddress,
                    EmailAddress = subscriber.EmailAddress,
                    EmailSent = false,
                    MessageRef = messageToProcess.MessageRef,
                    ScheduledDate = messageToProcess.ScheduledDate,
                    FromEmailAddress = mailingList.FromEmailAddress,
                    SubjectLine = messageToProcess.SubjectLine,
                    SubscriberGUID = subscriber.SubscriberGUID,
                    ListName = mailingList.ListName
                };

                // When we try to add the entity to the SendEmail table, 
                // an exception might happen if this worker role went 
                // down after processing some of the email addresses and then restarted.
                // In that case the row might already be present, so we do an Upsert operation.
                try
                {
                    var upsertOperation = TableOperation.InsertOrReplace(sendEmailRow);
                    messageTable.Execute(upsertOperation);
                }
                catch (Exception ex)
                {
                    string err = "Error creating SendEmail row:  " + ex.Message;
                    if (ex.InnerException != null)
                    {
                        err += " Inner Exception: " + ex.InnerException;
                    }
                    Trace.TraceError(err);
                }

                // Create the queue message.
                string queueMessageString =
                    sendEmailRow.PartitionKey + "," +
                    sendEmailRow.RowKey + "," +
                    restartFlag;
                var queueMessage = new CloudQueueMessage(queueMessageString);
                sendEmailQueue.AddMessage(queueMessage);
            }

            Trace.TraceInformation("ProcessMessage end PK: "
                + messageToProcess.PartitionKey);
        }

    代码首先从针对目标邮件列表的 `mailinglist` 表获取邮件列表行。该行有一个“发件人”电子邮件地址，需要提供给辅助角色 B 来发送电子邮件。

            // Get Mailing List info to get the "From" email address.
            var retrieveOperation = TableOperation.Retrieve<MailingList>(messageToProcess.ListName, "mailinglist");
            var retrievedResult = mailingListTable.Execute(retrieveOperation);
            var mailingList = retrievedResult.Result as MailingList;
            if (mailingList == null)
            {
                Trace.TraceError("Mailing list not found: " + messageToProcess.ListName + " for message: " + messageToProcess.MessageRef);
                return;
            }

    然后，它会查询 `mailinglist` 表中是否存在目标邮件列表的所有订户行。

            // Get email addresses for this Mailing List.
            string filter = TableQuery.CombineFilters(
               TableQuery.GenerateFilterCondition("PartitionKey", QueryComparisons.Equal, messageToProcess.ListName),
               TableOperators.And,
               TableQuery.GenerateFilterCondition("RowKey", QueryComparisons.NotEqual, "mailinglist"));
            var query = new TableQuery<Subscriber>().Where(filter);
            var subscribers = mailingListTable.ExecuteQuery(query).ToList();

    在处理查询结果的循环中，代码先查看是否已验证订户电子邮件地址，如果尚未验证，则不会将电子邮件排入队列中。

                // Verify that the subscriber email address has been verified.
                if (subscriber.Verified == false)
                {
                    Trace.TraceInformation("Subscriber " + subscriber.EmailAddress + " not Verified, so not queuing ");
                    continue;
                }

    接下来，代码在 `message` 表中创建 `SendEmail` 行。此行包含辅助角色 B 将要用来发送电子邮件的信息。该行在创建时，其 `EmailSent` 属性设置为 `false`。

                // Create a SendEmail entity for this email.              
                var sendEmailRow = new SendEmail
                {
                    PartitionKey = messageToProcess.PartitionKey,
                    RowKey = messageToProcess.MessageRef.ToString() + subscriber.EmailAddress,
                    EmailAddress = subscriber.EmailAddress,
                    EmailSent = false,
                    MessageRef = messageToProcess.MessageRef,
                    ScheduledDate = messageToProcess.ScheduledDate,
                    FromEmailAddress = mailingList.FromEmailAddress,
                    SubjectLine = messageToProcess.SubjectLine,
                    SubscriberGUID = subscriber.SubscriberGUID,
                    ListName = mailingList.ListName
                };
                try
                {
                    var upsertOperation = TableOperation.InsertOrReplace(sendEmailRow);
                    messageTable.Execute(upsertOperation);
                }
                catch (Exception ex)
                {
                    string err = "Error creating SendEmail row:  " + ex.Message;
                    if (ex.InnerException != null)
                    {
                        err += " Inner Exception: " + ex.InnerException;
                    }
                    Trace.TraceError(err);
                }

    代码使用“upsert”操作，因为如果辅助角色 A 在故障后重新启动，该行可能已存在。

    需要对每个电子邮件地址执行的最后一个任务是创建队列工作项，以便触发辅助角色 B 发送电子邮件。队列工作项包含刚创建的 `SendEmail` 行的分区键和行键的值，以及此前设置的重新启动标志。`SendEmail` 行包含辅助角色 B 发送电子邮件所需的所有信息。

                // Create the queue message.
                string queueMessageString =
                    sendEmailRow.PartitionKey + "," +
                    sendEmailRow.RowKey + "," +
                    restartFlag;
                var queueMessage = new CloudQueueMessage(queueMessageString);
                sendEmailQueue.AddMessage(queueMessage);

    `CheckAndUpdateStatusIfComplete` 方法用于检查处于“正在处理”状态的邮件，以了解是否已发送所有电子邮件。如果没有发现未发送的电子邮件，它会将行状态更新为 `Completed` 并存档该行。

        private void CheckAndArchiveIfComplete(Message messageToCheck)
        {
            // Get the list of emails to be sent for this message: all SendEmail rows
            // for this message.  
            string pkrkFilter = TableQuery.CombineFilters(
                TableQuery.GenerateFilterCondition("PartitionKey", QueryComparisons.Equal, messageToCheck.PartitionKey),
                TableOperators.And,
                TableQuery.GenerateFilterCondition("RowKey", QueryComparisons.LessThan, "message"));
            var query = new TableQuery<SendEmail>().Where(pkrkFilter);
            var emailToBeSent = messageTable.ExecuteQuery(query).FirstOrDefault();

            if (emailToBeSent != null)
            {
                return;
            }

            // All emails have been sent; copy the message row to the archive table.

            // Insert the message row in the messagearchive table
            var messageToDelete = new Message { PartitionKey = messageToCheck.PartitionKey, RowKey = messageToCheck.RowKey, ETag = "*" };
            messageToCheck.Status = "Complete";
            var insertOrReplaceOperation = TableOperation.InsertOrReplace(messageToCheck);
            messagearchiveTable.Execute(insertOrReplaceOperation);

            // Delete the message row from the message table.
            var deleteOperation = TableOperation.Delete(messageToDelete);
            messageTable.Execute(deleteOperation);
        }

## <a name="configure"></a><span class="short-header">配置存储</span>配置存储空间连接字符串

如果你在为 Web 角色配置存储帐户凭据的时候尚未为辅助角色 A 这样做，则请现在就这样做。

1.  在解决方案资源管理器中，右键单击 **AzureEmailService** 云项目的“角色”下的“WorkerRoleA”，然后选择**“属性”**。

2.  请确保在**“服务配置”**下拉列表中选择了“所有配置”。

3.  选择**“设置”**选项卡，然后单击**“添加设置”**。

4.  在“名称”列中输入 *StorageConnectionString*。

5.  在“类型”下拉列表中选择“连接字符串”。

6.  单击行右端的省略号 (**...**)，创建新的连接字符串。

7.  在**“存储帐户连接字符串”**对话框中，单击**“你的订阅”**。

8.  选择正确的**订阅**和**帐户名**，然后单击**“确定”**。

9.  设置诊断连接字符串。你可以将相同的存储帐户用于诊断连接字符串，但最好是将不同的存储帐户用于跟踪（诊断）信息。

## <a name="testing"></a><span class="short-header">测试</span>测试辅助角色 A

1.  通过按 F5 运行应用程序。

> [WACOM.NOTE] 使用 Visual Studio 2013 和最新 SDK 时，你可能会收到针对 `LogLevel` 引用的“引用不明确”错误。右键单击 `LogLevel`，单击“解析”，然后选择 `Microsoft.WindowsAzure.Diagnostics.LogLevel`。

1.  使用管理员 Web 页创建邮寄列表，并创建邮件列表的订户。将至少一个订户的 `Verified` 属性设置为 `true`，然后将电子邮件地址设置为你可以用来接收邮件的地址。

    除非你实现了辅助角色 B，否则不会发送电子邮件，不过你需要使用相同的测试数据来测试辅助角色 B。

2.  创建一封要发送到你所创建的邮件列表的邮件，将计划发送的日期设置为今天或过去的某个日期。

    ![挂起状态的新邮件][挂起状态的新邮件]

3.  在一分钟多一点的时候（由于在 Run 方法中设置了一分钟的休眠时间），刷新“邮件”网页，你会看到状态更改为“正在处理”。（你可能会先看到它更改为“正在排队”，但它很可能从“正在排队”快速转换为“正在处理”，导致你无法看到“正在排队”。）

    ![处于“正在处理”状态的新邮件][处于“正在处理”状态的新邮件]

4.  打开 Azure 存储资源管理器，然后选择你的测试用的存储帐户。

5.  在 Azure 存储资源管理器的**“存储类型”**下选择**“队列”**，然后选择 **azuremailqueue**。

    你会看到目标电子邮件列表中每个已验证订户都有一个队列消息。

    ![ASE 中的队列消息][ASE 中的队列消息]

6.  双击队列消息，然后在**“消息详细信息”**对话框中，选择**“消息”**选项卡。

    你会看到队列消息的内容：分区键（日期为 2012-12-14）、行键（MessageRef 值和电子邮件地址），以及重新启动标志，由逗号分隔。

    ![ASE 中的队列消息内容][ASE 中的队列消息内容]

7.  关闭**“消息详细信息”**对话框。

8.  在**“存储类型”**下，选择**“表”**，然后选择**“消息”**表。

9.  单击**“查询”**查看表中的所有行。

    你会看到计划要发送的邮件，在行键中有“Message”字样，后跟用于每个已验证订户的一行，行键中有电子邮件地址。

    ![ASE 中的消息表行][ASE 中的消息表行]

10. 双击行键中有一个电子邮件地址的行，以便查看辅助角色 A 所创建的 `SendEmail` 行的内容。

    ![消息表中的 SendEmail 行][消息表中的 SendEmail 行]

## <a name="nextsteps"></a><span class="short-header">后续步骤</span>后续步骤

你现在已构建辅助角色 A 并已验证它创建了辅助角色 B 发送电子邮件所需的队列消息和表行。在[下一教程][下一教程]中，你将构建和测试辅助角色 B。

有关如何使用 Azure 存储表、队列和 Blob 的其他资源的链接，请参阅[本系列最后一个教程][下一教程]结尾处的内容。

<div><a href="/zh-cn/develop/net/tutorials/multi-tier-web-site/5-worker-role-b/" class="site-arrowboxcta download-cta">教程 5</a></div>

  [本系列第一个教程]: /zh-cn/develop/net/tutorials/multi-tier-web-site/1-overview/
  [添加对 Web 项目的引用]: #addref
  [添加对 SCL 1.7 程序集的引用]: #addref2
  [添加 SendEmail 模型]: #addmodel
  [添加辅助角色启动时运行的代码]: #addcode
  [配置存储空间连接字符串]: #configure
  [测试辅助角色 A]: #testing
  [后续步骤]: #nextsteps
  [在 WorkerRoleA 项目中添加引用]: ./media/cloud-services-dotnet-multi-tier-app-storage-1-worker-role-a/mtas-worker-a-add-reference-menu.png
  [将引用添加到 MvcWebRole]: ./media/cloud-services-dotnet-multi-tier-app-storage-1-worker-role-a/mtas-worker-a-reference-manager.png
  [使用 sendmail 的邮件表]: ./media/cloud-services-dotnet-multi-tier-app-storage-1-worker-role-a/mtas-sendMailTbl.png
  [将现有项添加到 Web 项目中的 Models 文件夹]: ./media/cloud-services-dotnet-multi-tier-app-storage-1-worker-role-a/mtas-add-existing-for-sendemail-model.png
  [将现有项添加到辅助角色 A]: ./media/cloud-services-dotnet-multi-tier-app-storage-1-worker-role-a/mtas-worker-a-add-existing.png
  [第二个教程]: /zh-cn/develop/net/tutorials/multi-tier-web-site/2-download-and-run/
  [重新启动角色实例进行 OS 升级]: http://blogs.msdn.com/b/kwill/archive/2012/09/19/role-instance-restarts-due-to-os-upgrades.aspx
  [UnhandledException]: http://msdn.microsoft.com/zh-cn/library/system.appdomain.unhandledexception.aspx
  [挂起状态的新邮件]: ./media/cloud-services-dotnet-multi-tier-app-storage-1-worker-role-a/mtas-worker-a-test-pending.png
  [处于“正在处理”状态的新邮件]: ./media/cloud-services-dotnet-multi-tier-app-storage-1-worker-role-a/mtas-worker-a-test-processing.png
  [ASE 中的队列消息]: ./media/cloud-services-dotnet-multi-tier-app-storage-1-worker-role-a/mtas-worker-a-test-ase-queue.png
  [ASE 中的队列消息内容]: ./media/cloud-services-dotnet-multi-tier-app-storage-1-worker-role-a/mtas-worker-a-test-ase-queue-detail.png
  [ASE 中的消息表行]: ./media/cloud-services-dotnet-multi-tier-app-storage-1-worker-role-a/mtas-worker-a-test-ase-message-table.png
  [消息表中的 SendEmail 行]: ./media/cloud-services-dotnet-multi-tier-app-storage-1-worker-role-a/mtas-worker-a-test-ase-sendemail-row.png
  [下一教程]: /zh-cn/develop/net/tutorials/multi-tier-web-site/5-worker-role-b/
