<properties linkid="develop-net-tutorials-multi-tier-web-site-5-worker-role-b" pageTitle="Azure 云服务教程：辅助角色和 Azure 存储表、队列、Blob" metaKeywords="Azure tutorial, Azure storage tutorial, Azure multi-tier tutorial, Azure worker role tutorial, Azure blobs tutorial, Azure tables tutorial, Azure queues tutorial" description="了解如何使用 ASP.NET MVC 和 Azure 创建多层应用程序。该应用程序运行在云服务中，使用 Web 角色和辅助角色，并使用 Azure 存储表、队列和 Blob。" metaCanonical="" services="cloud-services,storage" documentationCenter=".NET" title="Azure 云服务教程：ASP.NET MVC Web 角色、辅助角色、Azure 存储表、队列和 Blob" authors="tdykstra,riande" solutions="" manager="wpickett" editor="mollybos" />

# 构建 Azure 电子邮件服务应用程序的辅助角色 B（电子邮件发送程序）- 第 5 部分（共 5 部分）。

这是分为五部分的一系列教程的第五个教程，这些教程讲述如何生成和部署 Azure 电子邮件服务示例应用程序。有关此应用程序和教程系列的信息，请参阅[本系列第一个教程][本系列第一个教程]。

在本教程中，你将学习：

-   如何向云服务项目添加辅助角色。
-   如何轮询队列和处理队列中的工作项。
-   如何使用 SendGrid 发送电子邮件。
-   如何通过覆盖 `OnStop` 方法处理计划的关闭操作。
-   如何通过确保没有发送重复的电子邮件来处理非计划的关闭操作。

## 本教程的章节

-   [将辅助角色 B 项目添加到解决方案][将辅助角色 B 项目添加到解决方案]
-   [添加对 Web 项目的引用][添加对 Web 项目的引用]
-   [向项目中添加存储客户端库 2.0 NuGet 包][向项目中添加存储客户端库 2.0 NuGet 包]
-   [添加对 SCL 1.7 程序集的引用][添加对 SCL 1.7 程序集的引用]
-   [将 SendGrid NuGet 包添加到项目][将 SendGrid NuGet 包添加到项目]
-   [添加项目设置][添加项目设置]
-   [添加辅助角色启动时运行的代码][添加辅助角色启动时运行的代码]
-   [测试辅助角色 B][测试辅助角色 B]
-   [后续步骤][后续步骤]

## <a name="addworkerrole"></a><span class="short-header">添加辅助角色 B</span>将辅助角色 B 项目添加到解决方案

1.  在解决方案资源管理器中右键单击云服务项目，然后选择**“新建辅助角色项目”**。

    ![“新建辅助角色项目”菜单][“新建辅助角色项目”菜单]

2.  在**“添加新角色项目”**对话框中，选择 **C#**，选择**“辅助角色”**，将项目命名为 WorkerRoleB，然后单击**“添加”**。

    ![“新建角色项目”对话框][“新建角色项目”对话框]

## <a name="addreference"></a><span class="short-header">添加引用</span>添加对 Web 项目的引用

你需要对 Web 项目的引用，因为实体类是在该项目中定义的。你需要在辅助角色 B 中使用实体类在应用程序所使用的 Azure 表中进行数据读写操作。

1.  右键单击 WorkerRoleB 项目，然后选择**“添加引用”**。

    ![在 WorkerRoleB 项目中添加引用][在 WorkerRoleB 项目中添加引用]

2.  在**引用管理器**中，将引用添加到 MvcWebRole 项目中（或 Web 应用程序项目中，如果你正在 Azure 网站中运行 Web UI）。

    ![将引用添加到 MvcWebRole][将引用添加到 MvcWebRole]

## <a name="sclpackage"></a><span class="short-header">添加 SCL 2.0 包</span>向项目中添加存储客户端库 2.0 NuGet 包

> [WACOM.NOTE] 使用 Visual Studio 2013 时，你可以跳过此部分，因为当前的 Azure 存储包安装在新辅助角色项目中。

添加项目时，它并不自动获取存储客户端库 NuGet 包的已更新版本。相反，它获得的是旧的 1.7 版本的包，因为那是项目模板中包含的内容。现在该解决方案有两个版本的 Azure 存储 NuGet 包：2.0 版在 MvcWebRole 和 WorkerRoleA 项目中，1.7 版在 WorkerRoleB 项目中。你需要卸载 1.7 版，然后在 WorkerRoleB 项目中安装 2.0 版。

1.  在“工具”菜单中，选择“库程序包管理器”，然后选择“管理解决方案的 NuGet 包”。

2.  在左窗格中选中**“安装的程序包”**后，向下滚动到 Azure 存储包。

    你将看到包被列出了两次，一次针对 1.7 版，一次针对 2.0 版。

3.  选择 1.7 版程序包，然后单击**“管理”**。

    此时会清除 MvcWebRole 和 WorkerRoleB 所对应的复选框，并选中 WorkerRoleB 的复选框。

4.  对于 WorkerRoleB，请清除该复选框，然后单击**“确定”**。

5.  当询问你是否是否想要卸载依赖的包时，请单击**“否”**。

    当卸载完成时，仅 2.0 版的包留在 NuGet 对话框中。

6.  针对 2.0 版的包单击**“管理”**。

    此时会选中 MvcWebRole 和 WorkerRoleA 所对应的复选框，并清除 WorkerRoleA 的复选框。

7.  请选中 WorkerRoleA 的复选框，然后单击**“确定”**。

## <a name="addref2"></a><span class="short-header">添加 SCL 1.7 引用</span>添加对 SCL 1.7 程序集的引用

> [WACOM.NOTE] 如果你已安装最新 SDK 并且使用的是 Visual Studio 2013，请跳过此部分

存储客户端库 (SCL) 2.0 版并没有进行诊断所需的所有内容，因此你必须添加对某个 1.7 版程序集的引用，正如你此前对其他两个项目所做的那样。

1.  右键单击 WorkerRoleB 项目，然后选择**“添加引用”**。

2.  单击对话框底部的**“浏览...”**按钮。

3.  导航到以下文件夹：

        C:\Program Files\Microsoft SDKs\Windows Azure\.NET SDK012-10\ref

4.  选择 *Microsoft.WindowsAzure.StorageClient.dll*，然后单击**“添加”**。

5.  在**“引用管理器”**对话框中，单击**“确定”**。

## <a name="addsendgrid"></a><span class="short-header">添加 SendGrid 包</span>将 SendGrid NuGet 包添加到项目

若要使用 SendGrid 发送电子邮件，你需要安装 SendGrid NuGet 包。

1.  在“解决方案资源管理器”中，右键单击 WorkerRoleB 项目，然后选择“管理 NuGet 包”。

    ![管理 NuGet 包][管理 NuGet 包]

2.  在**“管理 NuGet 包”**对话框中，选择**“联机”**选项卡，在搜索框中输入“sendgrid”，然后按 Enter。

3.  单击 SendGrid 包对应的“安装”。

    ![安装 Sendgrid 包][安装 Sendgrid 包]

4.  关闭该对话框。

## <a name="addsettings"></a><span class="short-header">添加项目设置</span>添加项目设置

像辅助角色 A 一样，辅助角色 B 需要用于表、队列和 Blob 的存储帐户凭据。此外，辅助角色需要具有凭据才能嵌入针对 SendGrid 服务的调用，以便发送电子邮件。为了构建取消订阅链接以包括所要发送的电子邮件，辅助角色需要知道应用程序的 URL。这些值存储在项目设置中。

对于存储帐户凭据，该过程与你在[第三个教程][第三个教程]中看到的相同。

1.  在**解决方案资源管理器**的云项目的“角色”下，右键单击**“WorkerRoleB”**，然后选择**“属性”**。

2.  选择“设置”选项卡。

3.  请确保在**“服务配置”**下拉列表中选择了“所有配置”。

4.  选择**“设置”**选项卡，然后单击**“添加设置”**。

5.  在“名称”列中输入“StorageConnectionString”。

6.  在“类型”下拉列表中选择“连接字符串”。

7.  单击行的右端的省略号 (**...**) 按钮，打开**“存储帐户连接字符串”**对话框。

8.  在**“创建存储连接字符串”**对话框中，单击**“你的订阅”**单选按钮。

9.  选择的**订阅**和**帐户名**与你为 Web 角色和辅助角色 A 选择的相同。

10. 请按照相同的过程来配置 **Microsoft.WindowsAzure.Plugins.Diagnostics.ConnectionString** 连接字符串的设置。

    接下来，请创建并配置仅由辅助角色 B 使用的三个新设置。

11. 在**“属性”**窗口的**“设置”**选项卡中，单击**“添加设置”**，然后添加三个类型为“字符串”的新设置：

    -   “名称”：SendGridUserName，**值**：你在[第二个教程][第二个教程]中构建的 SendGrid 用户名。

    -   “名称”：SendGridPassword，**值**：SendGrid 密码。

    -   “名称”：AzureMailServiceURL，**值**：在部署时应用程序会具有的基 URL，例如：http://sampleurl.cloudapp.net。

    ![WorkerRoleB 项目中的新设置][WorkerRoleB 项目中的新设置]

## <a name="addcode"></a>添加辅助角色启动时运行的代码

1.  在 WorkerRoleB 项目中，删除 WorkerRole.cs。

2.  右键单击 WorkerRoleB 项目，然后选择**“添加现有项”**。

    ![将现有项添加到辅助角色 B][将现有项添加到辅助角色 B]

3.  导航到下载了示例应用程序的文件夹，选择 WorkerRoleB 项目中的 WorkerRoleB.cs 文件，然后单击“添加”。

> [WACOM.NOTE] 对于装有最新 SDK 和最新 SendGrid NuGet 包的 Visual Studio 2013，请打开 *WorkerRoleB.cs* 并对代码进行以下更改：(1) 删除 `SendGridMail.Transport` 的 `using` 语句。(2) 将 `SendGrid.GenerateInstance` 的两个实例更改为 `SendGrid.GetInstance`。(3) 将 `REST.GetInstance` 的两个实例更改为 `Web.GetInstance`。

1.  打开 WorkerRoleB.cs 并检查代码。

    正如你在辅助角色 A 中所看到的那样，`OnStart` 方法初始化使用 Azure 存储实体所需的上下文类。它还可确保你在 `Run` 方法中需要的所有表、队列和 Blob 容器都存在。

    与辅助角色 A 相比，区别是在不存在的需要创建的资源中添加了 Blob 容器和订阅队列。你将使用 Blob 容器来获取包含 HTML 和纯文本电子邮件正文的文件。订阅队列用于发送订阅确认电子邮件。

        public override bool OnStart()
        {
            ServicePointManager.DefaultConnectionLimit = Environment.ProcessorCount;

            // Read storage account configuration settings
            ConfigureDiagnostics();
            Trace.TraceInformation("Initializing storage account in worker role B");
            var storageAccount = CloudStorageAccount.Parse(RoleEnvironment.GetConfigurationSettingValue("StorageConnectionString"));

            // Initialize queue storage 
            Trace.TraceInformation("Creating queue client.");
            CloudQueueClient queueClient = storageAccount.CreateCloudQueueClient();
            this.sendEmailQueue = queueClient.GetQueueReference("azuremailqueue");
            this.subscribeQueue = queueClient.GetQueueReference("azuremailsubscribequeue");

            // Initialize blob storage
            CloudBlobClient blobClient = storageAccount.CreateCloudBlobClient();
            this.blobContainer = blobClient.GetContainerReference("azuremailblobcontainer");

            // Initialize table storage
            var tableClient = storageAccount.CreateCloudTableClient();
            tableServiceContext = tableClient.GetDataServiceContext();

            Trace.TraceInformation("WorkerB: Creating blob container, queue, tables, if they don't exist.");
            this.blobContainer.CreateIfNotExists();
            this.sendEmailQueue.CreateIfNotExists();
            this.subscribeQueue.CreateIfNotExists();
            var messageTable = tableClient.GetTableReference("Message");
            messageTable.CreateIfNotExists();
            var mailingListTable = tableClient.GetTableReference("MailingList");
            mailingListTable.CreateIfNotExists();

            return base.OnStart();
        }

    `Run` 方法处理来自两个队列的工作项：用于发送到电子邮件列表（辅助角色 A 所创建的工作项）的邮件的队列，以及用于订阅确认电子邮件（MvcWebRole 项目中订阅 API 方法所创建的工作项）的队列。

        public override void Run()
        {
            CloudQueueMessage msg = null;

            Trace.TraceInformation("WorkerRoleB start of Run()");
            while (true)
            {
                try
                {
                    bool messageFound = false;

                    // If OnStop has been called, return to do a graceful shutdown.
                    if (onStopCalled == true)
                    {
                        Trace.TraceInformation("onStopCalled WorkerRoleB");
                        returnedFromRunMethod = true;
                        return;
                    }
                    // Retrieve and process a new message from the send-email-to-list queue.
                    msg = sendEmailQueue.GetMessage();
                    if (msg != null)
                    {
                        ProcessQueueMessage(msg);
                        messageFound = true;
                    }

                    // Retrieve and process a new message from the subscribe queue.
                    msg = subscribeQueue.GetMessage();
                    if (msg != null)
                    {
                        ProcessSubscribeQueueMessage(msg);
                        messageFound = true;
                    }

                    if (messageFound == false)
                    {
                        System.Threading.Thread.Sleep(1000 * 60);
                    }
                }
                catch (Exception ex)
                {
                    string err = ex.Message;
                    if (ex.InnerException != null)
                    {
                        err += " Inner Exception: " + ex.InnerException.Message;
                    }
                    if (msg != null)
                    {
                        err += " Last queue message retrieved: " + msg.AsString;
                    }
                    Trace.TraceError(err);
                    // Don't fill up Trace storage if we have a bug in either process loop.
                    System.Threading.Thread.Sleep(1000 * 60);
                }
            }
        }

    此代码运行会导致无限循环，直到辅助角色关闭为止。如果主队列中找到工作项，则代码会处理它，然后检查订阅队列。

                    // Retrieve and process a new message from the send-email-to-list queue.
                    msg = this.sendEmailQueue.GetMessage();
                    if (msg != null)
                    {
                        ProcessQueueMessage(msg);
                        messageFound = true;
                    }

                    // Retrieve and process a new message from the subscribe queue.
                    msg = this.subscribeQueue.GetMessage();
                    if (msg != null)
                    {
                        ProcessSubscribeQueueMessage(msg);
                        messageFound = true;
                    }

    如果没有任何项在任何一个队列中等待，代码将休眠 60 秒，然后再继续循环。

                    if (messageFound == false)
                    {
                        System.Threading.Thread.Sleep(1000 * 60);
                    }

    使用休眠时间的目的是尽量减少 Azure 存储事务成本，如[上一个教程][上一个教程]所述。

    通过 [GetMessage][GetMessage] 方法从队列中拉取队列项时，该队列项将在 30 秒的时间内变得对所有其他访问该队列的辅助角色和 Web 角色不可见。这就可以确保对于任何给定的队列消息，只有一个辅助角色实例会选取它进行处理。你可以显式设置这个*排他性租约* 时间（队列项不可见的时间），只需将[可见性超时][可见性超时]参数传递给 `GetMessage` 方法即可。如果辅助角色可能需要超过 30 秒的时间来处理队列消息，则应增加防止其他角色实例处理同一消息的排他性租约时间。

    另一方面，你不需要将排他性租约时间设置为过大的值。例如，如果排他性租约时间设置为 48 小时，并且一个辅助角色在使某个消息出队之后意外关闭，另一个辅助角色将不能在 48 小时内处理消息。排他性租约的最大值为 7 天。

    [GetMessages][GetMessages] 方法（注意名称末尾的“s”）可以用于在一次调用中从队列中拉取最多 32 条消息。每次队列访问都会产生较小的事务成本，不管是返回 32 条消息还是返回 0 条消息，事务成本都是一样的。下面的代码一次调用提取多达 32 条消息，然后再处理它们。

        foreach (CloudQueueMessage msg in sendEmailQueue.GetMessages(32))
        {
            ProcessQueueMessage(msg);
            messageFound = true;
        }

    在使用 `GetMessages` 删除多条消息时，请确保可见性超时为你的应用程序提供足够的时间来处理所有消息。可见性超时到期后，其他角色实例可以访问该消息，而一旦这样做，第一个实例在完成工作项的处理操作后将不能删除该消息。

    在主队列中找到工作项时，`Run` 方法会调用 `ProcessQueueMessage`：

        private void ProcessQueueMessage(CloudQueueMessage msg)
        {
            // Log and delete if this is a "poison" queue message (repeatedly processed
            // and always causes an error that prevents processing from completing).
            // Production applications should move the "poison" message to a "dead message"
            // queue for analysis rather than deleting the message.           
            if (msg.DequeueCount > 5)
            {
                Trace.TraceError("Deleting poison message:    message {0} Role Instance {1}.",
                    msg.ToString(), GetRoleInstance());
                sendEmailQueue.DeleteMessage(msg);
                return;
            }
            // Parse message retrieved from queue.
            // Example:  2012-01-01,0123456789email@domain.com,0
            var messageParts = msg.AsString.Split(new char[] { ',' });
            var partitionKey = messageParts[0];
            var rowKey = messageParts[1];
            var restartFlag = messageParts[2];
            Trace.TraceInformation("ProcessQueueMessage start:  partitionKey {0} rowKey {1} Role Instance {2}.",
                partitionKey, rowKey, GetRoleInstance());
            // If this is a restart, verify that the email hasn't already been sent.
            if (restartFlag == "1")
            {
                var retrieveOperationForRestart = TableOperation.Retrieve<SendEmail>(partitionKey, rowKey);
                var retrievedResultForRestart = messagearchiveTable.Execute(retrieveOperationForRestart);
                var messagearchiveRow = retrievedResultForRestart.Result as SendEmail;
                if (messagearchiveRow != null)
                {
                    // SendEmail row is in archive, so email is already sent. 
                    // If there's a SendEmail Row in message table, delete it,
                    // and delete the queue message.
                    Trace.TraceInformation("Email already sent: partitionKey=" + partitionKey + " rowKey= " + rowKey);
                    var deleteOperation = TableOperation.Delete(new SendEmail { PartitionKey = partitionKey, RowKey = rowKey, ETag = "*" });
                    try
                    {
                        messageTable.Execute(deleteOperation);
                    }
                    catch
                    {
                    }
                    sendEmailQueue.DeleteMessage(msg);
                    return;
                }
            }
                        // Get the row in the Message table that has data we need to send the email.
            var retrieveOperation = TableOperation.Retrieve<SendEmail>(partitionKey, rowKey);
            var retrievedResult = messageTable.Execute(retrieveOperation);
            var emailRowInMessageTable = retrievedResult.Result as SendEmail;
            if (emailRowInMessageTable == null)
            {
                Trace.TraceError("SendEmail row not found:  partitionKey {0} rowKey {1} Role Instance {2}.",
                    partitionKey, rowKey, GetRoleInstance());
                return;
            }
            // Derive blob names from the MessageRef.
            var htmlMessageBodyRef = emailRowInMessageTable.MessageRef + ".htm";
            var textMessageBodyRef = emailRowInMessageTable.MessageRef + ".txt";
            // If the email hasn't already been sent, send email and archive the table row.
            if (emailRowInMessageTable.EmailSent != true)
            {
                SendEmailToList(emailRowInMessageTable, htmlMessageBodyRef, textMessageBodyRef);

                var emailRowToDelete = new SendEmail { PartitionKey = partitionKey, RowKey = rowKey, ETag = "*" };
                emailRowInMessageTable.EmailSent = true;

                var upsertOperation = TableOperation.InsertOrReplace(emailRowInMessageTable);
                messagearchiveTable.Execute(upsertOperation);
                var deleteOperation = TableOperation.Delete(emailRowToDelete);
                messageTable.Execute(deleteOperation);
            }

            // Delete the queue message.
            sendEmailQueue.DeleteMessage(msg);

            Trace.TraceInformation("ProcessQueueMessage complete:  partitionKey {0} rowKey {1} Role Instance {2}.",
               partitionKey, rowKey, GetRoleInstance());
        }

    有害消息是指那些在进行处理时，会导致应用程序引发异常的消息。如果某条消息已从队列中拉取超过五次，我们就认为它无法进行处理，就会将其从队列中删除，这样我们就不会一直尝试对其进行处理。生产应用程序应考虑将有害消息移到“死消息”队列进行分析，而不是删除该消息。

    代码会将队列消息解析成检索 SendEmail 行所需的分区键和行键，以及一个重新启动标志。

            var messageParts = msg.AsString.Split(new char[] { ',' });
            var partitionKey = messageParts[0];
            var rowKey = messageParts[1];
            var restartFlag = messageParts[2];

    如果在意外关闭后重新启动了对此消息的处理，代码将检查 `messagearchive` 表来确定是否已发送过此电子邮件。如果此邮件已发送过，代码将删除 `SendEmail` 行（如果存在）并删除队列消息。

            if (restartFlag == "1")
            {
                var retrieveOperationForRestart = TableOperation.Retrieve<SendEmail>(partitionKey, rowKey);
                var retrievedResultForRestart = messagearchiveTable.Execute(retrieveOperationForRestart);
                var messagearchiveRow = retrievedResultForRestart.Result as SendEmail;
                if (messagearchiveRow != null)
                {
                    Trace.TraceInformation("Email already sent: partitionKey=" + partitionKey + " rowKey= " + rowKey);
                    var deleteOperation = TableOperation.Delete(new SendEmail { PartitionKey = partitionKey, RowKey = rowKey, ETag = "*" });
                    try
                    {
                        messageTable.Execute(deleteOperation);
                    }
                    catch
                    {
                    }
                    sendEmailQueue.DeleteMessage(msg);
                    return;
                }
            }

    接下来，我们从 `message` 表获取 `SendEmail` 行。该行具有发送电子邮件所需的所有信息，包含 HTML 和纯文本电子邮件正文的 Blob 除外。

            var retrieveOperation = TableOperation.Retrieve<SendEmail>(partitionKey, rowKey);
            var retrievedResult = messageTable.Execute(retrieveOperation);
            var emailRowInMessageTable = retrievedResult.Result as SendEmail;
            if (emailRowInMessageTable == null)
            {
                Trace.TraceError("SendEmail row not found:  partitionKey {0} rowKey {1} Role Instance {2}.",
                    partitionKey, rowKey, GetRoleInstance());
                return;
            }

    然后，该代码将发送电子邮件并存档 `SendEmail` 行。

            if (emailRowInMessageTable.EmailSent != true)
            {
                SendEmailToList(emailRowInMessageTable, htmlMessageBodyRef, textMessageBodyRef);

                var emailRowToDelete = new SendEmail { PartitionKey = partitionKey, RowKey = rowKey, ETag = "*" };
                emailRowInMessageTable.EmailSent = true;

                var upsertOperation = TableOperation.InsertOrReplace(emailRowInMessageTable);
                messagearchiveTable.Execute(upsertOperation);
                var deleteOperation = TableOperation.Delete(emailRowToDelete);
                messageTable.Execute(deleteOperation);
            }

    不能在一个事务中将行移到 messagearchive 表，因为这会影响多个表。

    最后，如果其他一切操作都成功，则会删除队列消息。

            sendEmailQueue.DeleteMessage(msg);

    使用 SendGrid 发送电子邮件的实际工作是通过 `SendEmailToList` 方法进行的。如果你想要使用 SendGrid 之外的其他服务，则只需更改此方法中的代码。

    **注意：**如果你在项目设置中的凭据无效，对 SendGrid 的调用将失败，但应用程序不会获得有关该故障的任何指示。如果你在生产应用程序中使用 SendGrid，请考虑为 Web API 设置不同的凭据，以免在管理员更改其 SendGrid 用户帐户密码的情况下引发无提示故障。有关详细信息，请参阅 [SendGrid MultiAuth - 多个帐户凭据][SendGrid MultiAuth - 多个帐户凭据]。你可以在 <https://sendgrid.com/credentials> 中设置凭据。

        private void SendEmailToList(string emailAddress, string fromEmailAddress, string subjectLine,
            string htmlMessageBodyRef, string textMessageBodyRef)
        {
            var email = SendGrid.GenerateInstance();
            email.From = new MailAddress(fromEmailAddress);
            email.AddTo(emailAddress);
            email.Html = GetBlobText(htmlMessageBodyRef);
            email.Text = GetBlobText(textMessageBodyRef);
            email.Subject = subjectLine;
            var credentials = new NetworkCredential(RoleEnvironment.GetConfigurationSettingValue("SendGridUserName"),
                RoleEnvironment.GetConfigurationSettingValue("SendGridPassword"));
            var transportREST = REST.GetInstance(credentials);
            transportREST.Deliver(email);
        }

        private string GetBlobText(string blogRef)
        {
            var blob = blobContainer.GetBlockBlobReference(blogRef);
            blob.FetchAttributes();
            var blobSize = blob.Properties.Length;
            using (var memoryStream = new MemoryStream((int)blobSize))
            {
                blob.DownloadToStream(memoryStream);
                return System.Text.Encoding.UTF8.GetString(memoryStream.ToArray());
            }
        }

    在 `GetBlobText` 方法中，出于性能原因，代码会获取 Blob 大小，然后使用该值来初始化 `MemoryStream` 对象。如果你未提供大小，`MemoryStream` 会分配 256 个字节，然后当下载超过该值时，它再分配 512 个字节，依此类推，每次分配的大小会加倍。对于大型 Blob 来说，这种分配方式效率很低，远不如在下载开始的时候就分配正确的大小。

    在订阅队列中找到工作项时，`Run` 方法会调用 `ProcessSubscribeQueueMessage`：

        private void ProcessSubscribeQueueMessage(CloudQueueMessage msg)
        {
            // Log and delete if this is a "poison" queue message (repeatedly processed
            // and always causes an error that prevents processing from completing).
            // Production applications should move the "poison" message to a "dead message"
            // queue for analysis rather than deleting the message.  
            if (msg.DequeueCount > 5)
            {
                Trace.TraceError("Deleting poison subscribe message:    message {0}.",
                    msg.AsString, GetRoleInstance());
                subscribeQueue.DeleteMessage(msg);
                return;
            }
            // Parse message retrieved from queue. Message consists of
            // subscriber GUID and list name.
            // Example:  57ab4c4b-d564-40e3-9a3f-81835b3e102e,contoso1
            var messageParts = msg.AsString.Split(new char[] { ',' });
            var subscriberGUID = messageParts[0];
            var listName = messageParts[1];
            Trace.TraceInformation("ProcessSubscribeQueueMessage start:    subscriber GUID {0} listName {1} Role Instance {2}.",
                subscriberGUID, listName, GetRoleInstance());
            // Get subscriber info. 
            string filter = TableQuery.CombineFilters(
                TableQuery.GenerateFilterCondition("PartitionKey", QueryComparisons.Equal, listName),
                TableOperators.And,
                TableQuery.GenerateFilterCondition("SubscriberGUID", QueryComparisons.Equal, subscriberGUID));
            var query = new TableQuery<Subscriber>().Where(filter);
            var subscriber = mailingListTable.ExecuteQuery(query).ToList().Single();
            // Get mailing list info.
            var retrieveOperation = TableOperation.Retrieve<MailingList>(subscriber.ListName, "mailinglist");
            var retrievedResult = mailingListTable.Execute(retrieveOperation);
            var mailingList = retrievedResult.Result as MailingList;

            SendSubscribeEmail(subscriberGUID, subscriber, mailingList);

            subscribeQueue.DeleteMessage(msg);

            Trace.TraceInformation("ProcessSubscribeQueueMessage complete: subscriber GUID {0} Role Instance {1}.",
                subscriberGUID, GetRoleInstance());
        }

    此方法执行以下任务：

    -   如果消息是“有害”消息，则记录并删除它。
    -   从队列消息中获取订户 GUID。
    -   使用 GUID 从 MailingList 表中获取订户信息。
    -   向新订户发送确认电子邮件。
    -   删除队列消息。

    就发送到列表的电子邮件来说，实际发送电子邮件是在单独的方法中进行的，这样可以轻松地根据需要转换电子邮件服务。

        private static void SendSubscribeEmail(string subscriberGUID, Subscriber subscriber, MailingList mailingList)
        {
            var email = SendGrid.GenerateInstance();
            email.From = new MailAddress(mailingList.FromEmailAddress);
            email.AddTo(subscriber.EmailAddress);
            string subscribeURL = RoleEnvironment.GetConfigurationSettingValue("AzureMailServiceURL") +
                "/subscribe?id=" + subscriberGUID + "&listName=" + subscriber.ListName;
            email.Html = String.Format("<p>Click the link below to subscribe to {0}. " +
                "If you don't confirm your subscription, you won't be subscribed to the list.</p>" +
                "<a href=\"{1}\">Confirm Subscription</a>", mailingList.Description, subscribeURL);
            email.Text = String.Format("Copy and paste the following URL into your browser in order to subscribe to {0}. " +
                "If you don't confirm your subscription, you won't be subscribed to the list.\n" +
                "{1}", mailingList.Description, subscribeURL);
            email.Subject = "Subscribe to " + mailingList.Description;
            var credentials = new NetworkCredential(RoleEnvironment.GetConfigurationSettingValue("SendGridUserName"), RoleEnvironment.GetConfigurationSettingValue("SendGridPassword"));
            var transportREST = REST.GetInstance(credentials);
            transportREST.Deliver(email);
        }

## <a name="testing"></a><span class="short-header">测试</span>测试辅助角色 B

1.  通过按 F5 运行应用程序。

2.  转到**“消息”**页后，可以看到一条你创建的用于测试辅助角色 A 的消息。大约一分钟后，刷新网页，你将看到行已从列表中消失，因为已存档。

3.  检查你希望在其中收到电子邮件的电子邮件收件箱。请注意，通过 SendGrid 发送电子邮件或将其传送到电子邮件客户端时，可能会存在延迟，因此可能需要等待一段时间才能看到电子邮件。你可能还需要检查垃圾邮件文件夹。

## <a name="nextsteps"></a><span class="short-header">后续步骤</span>后续步骤

你现在已从头开始生成了 Azure 电子邮件服务应用程序，你所拥有的项目与所下载的完整项目是相同的。若要部署到云中、在云中进行测试以及提升到生产环境，你可以使用与你在[第二个教程][第二个教程]中看到的过程一样的过程。如果你选择构建替代体系结构，请参阅 [Azure 网站入门教程][Azure 网站入门教程]，了解如何将 MVC 项目部署到 Azure 网站。

若要了解有关 Azure 存储空间的详细信息，请参阅以下资源：

-   [Microsoft Azure 存储空间基本知识][Microsoft Azure 存储空间基本知识]（Bruno Terkaly 的博客）

若要了解有关 Azure 表服务的详细信息，请参阅以下资源：

-   [Azure 表存储基本知识][Azure 表存储基本知识]（Bruno Terkaly 的博客）
-   [如何充分利用 Microsoft Azure 表][如何充分利用 Microsoft Azure 表]（Azure 存储团队博客）
-   [如何在 .NET 中使用表存储服务][如何在 .NET 中使用表存储服务]
-   [Microsoft Azure 存储客户端库 2.0 表深入探讨][Microsoft Azure 存储客户端库 2.0 表深入探讨]（Azure 存储团队博客）
-   [实际应用：为 Azure 表存储设计可扩展分区策略][实际应用：为 Azure 表存储设计可扩展分区策略]

若要了解有关 Azure 队列服务和 Azure Service Bus 队列的详细信息，请参阅以下资源：

-   [以队列为中心的工作模式（使用 Microsoft Azure 构建实际云应用程序)][以队列为中心的工作模式（使用 Microsoft Azure 构建实际云应用程序)]
-   [Azure 队列和 Azure Service Bus 队列 - 比较与对照][Azure 队列和 Azure Service Bus 队列 - 比较与对照]
-   [如何在 .NET 中使用队列存储服务][如何在 .NET 中使用队列存储服务]

若要了解有关 Azure Blob 服务的详细信息，请参阅以下资源：

-   [非结构化 Blob 存储（使用 Microsoft Azure 构建实际云应用程序）][非结构化 Blob 存储（使用 Microsoft Azure 构建实际云应用程序）]
-   [如何在 .NET 中使用 Azure Blob 存储服务][如何在 .NET 中使用 Azure Blob 存储服务]

若要了解有关自动缩放 Azure 云服务角色的详细信息，请参阅以下资源：

-   [如何使用自动缩放应用程序块][如何使用自动缩放应用程序块]
-   [自动缩放和 Azure][自动缩放和 Azure]
-   [使用 Azure 构建弹性自动缩放解决方案][使用 Azure 构建弹性自动缩放解决方案]（MSDN 第 9 频道视频）

## <a name="Acknowledgments"></a><span class="short-header">致谢</span>致谢

这些教程和示例应用程序是由 [Rick Anderson][Rick Anderson] 和 Tom Dykstra 编写的。我们真诚地感谢以下人员，感谢他们提供帮助：

-   Barry Dorrans (Twitter [@blowdart][@blowdart])
-   [Cory Fowler][Cory Fowler] (Twitter [@SyntaxC4][@SyntaxC4] )
-   [Joe Giardino][Joe Giardino]
-   Don Glover
-   Jai Haridas
-   [Scott Hunter][Scott Hunter] (Twitter:[@coolcsh][@coolcsh])
-   [Brian Swan][Brian Swan]
-   [Daniel Wang][Daniel Wang]
-   提供反馈的开发人员咨询委员会成员：
-   Damir Arh
-   Jean-Luc Boucho
-   Carlos dos Santos
-   Mike Douglas
-   Robert Fite
-   Gianni Rosa Gallina
-   Fabien Lavocat
-   Karl Ots
-   Carlos-Alejandro Perez
-   Sunao Tomita
-   Perez Jones Tsisah
-   Michiel van Otegem

  [本系列第一个教程]: /zh-cn/develop/net/tutorials/multi-tier-web-site/1-overview/
  [将辅助角色 B 项目添加到解决方案]: #addworkerrole
  [添加对 Web 项目的引用]: #addreference
  [向项目中添加存储客户端库 2.0 NuGet 包]: #sclpackage
  [添加对 SCL 1.7 程序集的引用]: #addref2
  [将 SendGrid NuGet 包添加到项目]: #addsendgrid
  [添加项目设置]: #addsettings
  [添加辅助角色启动时运行的代码]: #addcode
  [测试辅助角色 B]: #testing
  [后续步骤]: #nextsteps
  [“新建辅助角色项目”菜单]: ./media/cloud-services-dotnet-multi-tier-app-storage-1-worker-role-b/mtas-new-worker-role-project.png
  [“新建角色项目”对话框]: ./media/cloud-services-dotnet-multi-tier-app-storage-1-worker-role-b/mtas-add-new-role-project-dialog.png
  [在 WorkerRoleB 项目中添加引用]: ./media/cloud-services-dotnet-multi-tier-app-storage-1-worker-role-b/mtas-worker-b-add-reference-menu.png
  [将引用添加到 MvcWebRole]: ./media/cloud-services-dotnet-multi-tier-app-storage-1-worker-role-b/mtas-worker-b-reference-manager.png
  [管理 NuGet 包]: ./media/cloud-services-dotnet-multi-tier-app-storage-1-worker-role-b/mtas-worker-b-manage-nuget.png
  [安装 Sendgrid 包]: ./media/cloud-services-dotnet-multi-tier-app-storage-1-worker-role-b/mtas-worker-b-install-sendgrid.png
  [第三个教程]: /zh-cn/develop/net/tutorials/multi-tier-web-site/3-web-role/#configstorage
  [第二个教程]: /zh-cn/develop/net/tutorials/multi-tier-web-site/2-download-and-run/
  [WorkerRoleB 项目中的新设置]: ./media/cloud-services-dotnet-multi-tier-app-storage-1-worker-role-b/mtas-worker-b-settings.png
  [将现有项添加到辅助角色 B]: ./media/cloud-services-dotnet-multi-tier-app-storage-1-worker-role-b/mtas-worker-b-add-existing.png
  [上一个教程]: /zh-cn/develop/net/tutorials/multi-tier-web-site/4-worker-role-a/
  [GetMessage]: http://msdn.microsoft.com/zh-cn/library/windowsazure/ee741827.aspx
  [可见性超时]: http://msdn.microsoft.com/zh-cn/library/windowsazure/ee758454.aspx
  [GetMessages]: http://msdn.microsoft.com/zh-cn/library/windowsazure/microsoft.windowsazure.storageclient.cloudqueue.getmessages.aspx
  [SendGrid MultiAuth - 多个帐户凭据]: http://support.sendgrid.com/entries/21658978-sendgrid-multiauth-multiple-account-credentials
  [Azure 网站入门教程]: /zh-cn/develop/net/tutorials/get-started
  [Microsoft Azure 存储空间基本知识]: http://blogs.msdn.com/b/brunoterkaly/archive/2012/11/08/essential-knowledge-for-windows-azure-storage.aspx
  [Azure 表存储基本知识]: http://blogs.msdn.com/b/brunoterkaly/archive/2012/11/08/essential-knowledge-for-azure-table-storage.aspx
  [如何充分利用 Microsoft Azure 表]: http://blogs.msdn.com/b/windowsazurestorage/archive/2010/11/06/how-to-get-most-out-of-windows-azure-tables.aspx
  [如何在 .NET 中使用表存储服务]: http://www.windowsazure.com/zh-cn/develop/net/how-to-guides/table-services/
  [Microsoft Azure 存储客户端库 2.0 表深入探讨]: http://blogs.msdn.com/b/windowsazurestorage/archive/2012/11/06/windows-azure-storage-client-library-2-0-tables-deep-dive.aspx
  [实际应用：为 Azure 表存储设计可扩展分区策略]: http://msdn.microsoft.com/zh-cn/library/windowsazure/hh508997.aspx
  [以队列为中心的工作模式（使用 Microsoft Azure 构建实际云应用程序)]: http://www.asp.net/aspnet/overview/developing-apps-with-windows-azure/building-real-world-cloud-apps-with-windows-azure/queue-centric-work-pattern
  [Azure 队列和 Azure Service Bus 队列 - 比较与对照]: http://msdn.microsoft.com/zh-cn/library/windowsazure/hh767287.aspx
  [如何在 .NET 中使用队列存储服务]: /zh-cn/develop/net/how-to-guides/queue-service/
  [非结构化 Blob 存储（使用 Microsoft Azure 构建实际云应用程序）]: http://www.asp.net/aspnet/overview/developing-apps-with-windows-azure/building-real-world-cloud-apps-with-windows-azure/unstructured-blob-storage
  [如何在 .NET 中使用 Azure Blob 存储服务]: /zh-cn/develop/net/how-to-guides/blob-storage/
  [如何使用自动缩放应用程序块]: /zh-cn/develop/net/how-to-guides/autoscaling/
  [自动缩放和 Azure]: http://msdn.microsoft.com/zh-cn/library/hh680945(v=PandP.50).aspx
  [使用 Azure 构建弹性自动缩放解决方案]: http://channel9.msdn.com/Events/WindowsAzureConf/2012/B04
  [Rick Anderson]: http://blogs.msdn.com/b/rickandy/
  [@blowdart]: https://twitter.com/blowdart
  [Cory Fowler]: http://blog.syntaxc4.net/
  [@SyntaxC4]: https://twitter.com/SyntaxC4
  [Joe Giardino]: http://blogs.msdn.com/b/windowsazurestorage/
  [Scott Hunter]: http://blogs.msdn.com/b/scothu/
  [@coolcsh]: http://twitter.com/coolcsh
  [Brian Swan]: http://blogs.msdn.com/b/brian_swan/
  [Daniel Wang]: http://blogs.msdn.com/b/daniwang/
