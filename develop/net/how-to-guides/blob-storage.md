<properties linkid="blob-storage" urlDisplayName="blob-storage" pageTitle="blob-storage" metaKeywords="blob-storage" description="blob-storage" metaCanonical="" services="" documentationCenter="develop"  title="中国 Windows Azure 应用程序开发人员说明" authors="" solutions="" manager="TK" editor="Haifeng Liu" />
<h1>如何在 .NET 中使用 Windows Azure Blob 存储服务</h1>
<div class="dev-center-tutorial-selector"><a href="/zh-cn/develop/net/how-to-guides/blob-storage-v17/" title="版本 1.7" class="current">版本 1.7</a> </div>
<p>本指南将演示如何使用 Windows Azure Blob 存储服务执行常见任务。相关示例用 C# 编写且使用 .NET API。所涉及的任务包括<strong>上载</strong>、<strong>列出</strong>、<strong>下载</strong>以及<strong>删除</strong> Blob。有关 Blob 的详细信息，请参阅<a href="#next-steps">后续步骤</a>一节。</p>
<h2>目录</h2>
<ul>
<li><a href="#what-is">什么是 Blob 存储</a></li>
<li><a href="#concepts">概念</a></li>
<li><a href="#create-account">创建 Windows Azure 存储帐户</a></li>
<li><a href="#setup-connection-string">设置存储连接字符串</a></li>
<li><a href="#configure-access">如何：以编程方式访问 Blob 存储</a></li>
<li><a href="#create-container">如何：创建容器</a></li>
<li><a href="#upload-blob">如何：将 Blob 上载到容器中</a></li>
<li><a href="#list-blob">如何：在容器中列出 Blob</a></li>
<li><a href="#download-blobs">如何：下载 Blob</a></li>
<li><a href="#delete-blobs">如何：删除 Blob</a></li>
<li><a href="#next-steps">后续步骤</a></li>
</ul>
<?UMBRACO_MACRO macroAlias="AzureChunkDisplayer" chunkpath="devcenter/shared" chunkname="howto-blob-storage" hide="0" />
<h2><a name="create-account"></a> <span class="short-header">创建帐户</span>创建 Windows Azure 存储帐户</h2>
<?UMBRACO_MACRO macroAlias="AzureChunkDisplayer" chunkpath="devcenter/shared" chunkname="create-storage-account" hide="0" />
<h2><a name="setup-connection-string"></a> <span class="short-header">设置连接字符串</span>设置存储连接字符串</h2>
<p>Windows Azure .NET 存储 API 支持使用存储连接字符串配置用于访问存储服务的终结点和凭据。您可以将存储连接字符串置于配置文件中，而不是在代码中对其进行硬编码：</p>
<ul>
<li>在使用 Windows Azure 云服务时，建议使用 Windows Azure 服务配置系统（<code>*.csdef</code> 和 <code>*.cscfg</code> 文件）存储连接字符串。</li>
<li>在使用 Windows Azure 网站或 Windows Azure 虚拟机时，建议使用 .NET 配置系统（如 <code>web.config</code> 文件）存储连接字符串。</li>
</ul>
<p>在上述两种情况下，您可以使用 <code>CloudConfigurationManager.GetSetting</code> 方法检索连接字符串，本指南稍后部分将对此进行介绍。</p>
<h3>在使用云服务时配置连接字符串</h3>
<p>该服务配置机制是 Windows Azure 云服务项目特有的，它使您能够从 Windows Azure 管理门户动态更改配置设置，而无需部署您的应用程序。</p>
<p>在 Windows Azure 服务配置中配置连接字符串：</p>
<ol>
<li>
<p>在 Visual Studio 解决方案资源管理器中，在您的 Windows Azure 部署项目的“角色”文件夹中，右键单击 Web 角色或辅助角色，然后单击“属性”。<br /><img src="http://wacnstorage.blob.core.chinacloudapi.cn/marketing-resource/media/devcenter/dotnet/blob5.png" alt="在 Visual Studio 中选择云服务角色的属性"/></p>
</li>
<li>
<p>单击“设置”选项卡并按“添加设置”按钮。<br /><img src="http://wacnstorage.blob.core.chinacloudapi.cn/marketing-resource/media/devcenter/dotnet/blob6.png" alt="在 Visual Studio 中添加云服务设置"/></p>
<p>新的 <strong>Setting1</strong> 条目稍后将显示在设置网格中。</p>
</li>
<li>
<p>在新的 <strong>Setting1</strong> 条目的“类型”下拉列表中，选择“连接字符串”。<br /><img src="http://wacnstorage.blob.core.chinacloudapi.cn/marketing-resource/media/devcenter/dotnet/blob7.png" alt="Blob7"/></p>
</li>
<li>
<p>单击 <strong>Setting1</strong> 条目最右侧的 <strong>...</strong> 按钮。将打开“存储帐户连接字符串”对话框。</p>
</li>
<li>
<p>选择是要定位到存储仿真程序（在本地计算机上模拟的 Windows Azure 存储），还是云中的实际存储帐户。本指南中的代码使用其中任一方式。如果您希望使用我们之前在 Windows Azure 中创建的存储帐户存储 Blob 数据，请输入从本教程前面的步骤中复制的“主访问密钥”值。<br /><img src="http://wacnstorage.blob.core.chinacloudapi.cn/marketing-resource/media/devcenter/dotnet/blob8.png" alt="Blob8"/></p>
</li>
<li>
<p>将条目“名称”从 <strong>Setting1</strong> 更改为更友好的名称，例如 <strong>StorageConnectionString</strong>。稍后将在本指南的代码中引用此连接字符串。<br /><img src="http://wacnstorage.blob.core.chinacloudapi.cn/marketing-resource/media/devcenter/dotnet/blob9.png" alt="Blob9"/></p>
</li>
</ol>
<h3>在使用网站或虚拟机时配置连接字符串</h3>
<p>在使用网站或虚拟机时，建议使用 .NET 配置系统（如 <code>web.config</code>）。您可以使用 <code>&lt;appSettings&gt;</code> 元素存储连接字符串：</p>
<pre class="prettyprint">&lt;configuration&gt;     &lt;appSettings&gt;         &lt;add key="StorageConnectionString"              value="BlobEndpoint=https:// [AccountKey].blob.core.chinacloudapi.cn/;QueueEndpoint=https:// [AccountKey].queue.core.chinacloudapi.cn/;TableEndpoint=https:// [AccountKey].table.core.chinacloudapi.cn/;AccountName=[AccountName];AccountKey=[AccountKey]" /&gt;     &lt;/appSettings&gt; &lt;/configuration&gt;</pre>
<p>阅读<a href="http://msdn.microsoft.com/zh-cn/library/windowsazure/ee758697.aspx">配置连接字符串</a>，了解有关存储连接字符串的详细信息。</p>
<p>您现在即可准备执行本指南中的操作任务。</p>
<h2><a name="configure-access"></a> <span class="short-header">以编程方式访问</span>如何：以编程方式访问 Blob 存储</h2>
<p>将以下代码命名空间声明添加到任何 C# 文件的顶部，您希望使用此类文件以编程方式访问 Windows Azure 存储：</p>
<pre class="prettyprint">using Microsoft.WindowsAzure; using Microsoft.WindowsAzure.StorageClient;</pre>
<p>您可以使用 <strong>CloudStorageAccount</strong> 类型和 <strong>CloudConfigurationManager</strong> 类型从 Windows Azure 服务配置中检索存储连接字符串和存储帐户信息：</p>
<pre class="prettyprint">CloudStorageAccount storageAccount = CloudStorageAccount.Parse(     CloudConfigurationManager.GetSetting("StorageConnectionString"));</pre>
<p>您可以使用 <strong>CloudBlobClient</strong> 类型检索表示存储在 Blob 存储服务中的容器和 Blob 的对象。以下代码使用我们在上面检索到的存储帐户对象创建 <strong>CloudBlobClient</strong> 对象：</p>
<pre class="prettyprint">CloudBlobClient blobClient = storageAccount.CreateCloudBlobClient();</pre>
<h2><a name="create-container"></a> <span class="short-header">创建容器</span>如何：创建容器</h2>
<p>所有存储 Blob 都驻留在一个容器中。您可以使用 <strong>CloudBlobClient</strong> 对象获取对要使用的容器的引用。如果该容器不存在，您可以创建它：</p>
<pre class="prettyprint">// Retrieve storage account from connection string CloudStorageAccount storageAccount = CloudStorageAccount.Parse(     CloudConfigurationManager.GetSetting("StorageConnectionString"));  // Create the blob client  CloudBlobClient blobClient = storageAccount.CreateCloudBlobClient();  // Retrieve a reference to a container  CloudBlobContainer container = blobClient.GetContainerReference("mycontainer");  // Create the container if it doesn't already exist container.CreateIfNotExist();</pre>
<p>默认情况下，新容器是专用容器，因此您必须指定存储访问密钥（已在上面完成此操作）才能从该容器下载 Blob。如果您要让容器中的文件可供所有人使用，则可以使用以下代码将容器设置为公共容器：</p>
<pre class="prettyprint">container.SetPermissions(     new BlobContainerPermissions { PublicAccess =      BlobContainerPublicAccessType.Blob });</pre>
<p>Internet 中的所有人都可以查看公共容器中的 Blob，但是，仅在您具有相应的访问密钥时，才能修改或删除它们。</p>
<h2><a name="upload-blob"></a> <span class="short-header">上载到容器</span>如何：将 Blob 上载到容器中</h2>
<p>若要将文件上载到 Blob，请获取容器引用，并使用它获取 Blob 引用。获取 Blob 引用后，可以通过对该 Blob 引用调用 <strong>UploadFromStream</strong> 方法，将任何数据流上载到该引用。此操作将创建 Blob（如果该 Blob 不存在），或者覆盖它（如果该 Blob 存在）。下面的代码示例演示了这一点，并假定已创建容器。</p>
<pre class="prettyprint">// Retrieve storage account from connection string CloudStorageAccount storageAccount = CloudStorageAccount.Parse(     CloudConfigurationManager.GetSetting("StorageConnectionString"));  // Create the blob client CloudBlobClient blobClient = storageAccount.CreateCloudBlobClient();  // Retrieve reference to a previously created container CloudBlobContainer container = blobClient.GetContainerReference("mycontainer");  // Retrieve reference to a blob named "myblob" CloudBlob blob = container.GetBlobReference("myblob");  // Create or overwrite the "myblob" blob with contents from a local file using (var fileStream = System.IO.File.OpenRead(@"path\myfile")) {     blob.UploadFromStream(fileStream); }</pre>
<p>其中，“path\myfile”需更换为有效的文件路径和名称，例如 “c:\sample.txt”。</p>
<h2><a name="list-blob"></a> <span class="short-header">在容器中列出 Blob</span>如何：在容器中列出 Blob</h2>
<p>要在容器中列出 Blob，请首先获取容器引用。然后，您可以使用容器的 <strong>ListBlobs</strong> 方法检索其中的 Blob。以下代码演示如何检索和输出容器中每个 Blob 的 <strong>Uri</strong>：</p>
<pre class="prettyprint">// Retrieve storage account from connection string CloudStorageAccount storageAccount = CloudStorageAccount.Parse(     CloudConfigurationManager.GetSetting("StorageConnectionString"));  // Create the blob client CloudBlobClient blobClient = storageAccount.CreateCloudBlobClient();  // Retrieve reference to a previously created container CloudBlobContainer container = blobClient.GetContainerReference("mycontainer");  // Loop over blobs within the container and output the URI to each of them foreach (var blobItem in container.ListBlobs()) {     Console.WriteLine(blobItem.Uri); }</pre>
<p>还可将 Blob 服务视为容器中的目录。这是为了让您能够以更类似于文件夹的结构来组织 Blob。例如，您可以创建一个名为“photos”的容器，您可以在其中上载名为“rootphoto1”、“2010/photo1”、“2010/photo2”和“2011/photo1”的 Blob。这实际上将在“photos”容器中创建目录“2010”和“2011”。当您对“photos”容器调用 <strong>ListBlobs</strong> 时，返回的集合将包含表示最高层中所含目录和 Blob 的 <strong>CloudBlobDirectory</strong> 和 <strong>CloudBlob</strong> 对象。在本例中，将返回目录“2010”和“2011”以及照片“rootphoto1”。或者，也可以通过将 <strong>UseFlatBlobListing</strong> 设置为 <strong>true</strong>，传入新的 <strong>BlobRequestOptions</strong> 类。这将导致返回每个 Blob，而无论目录如何。有关详细信息，请参阅 <a href="http://msdn.microsoft.com/zh-cn/library/windowsazure/ee772878.aspx">CloudBlobContainer.ListBlobs</a>。</p>
<h2><a name="download-blobs"></a> <span class="short-header">下载 Blob</span>如何：下载 Blob</h2>
<p>若要下载 Blob，请首先检索 Blob 引用。以下示例使用 <strong>DownloadToStream</strong> 方法将 Blob 内容传输到稍后可以保存到本地文件的流对象。您还可以调用 Blob 的 <strong>DownloadToFile</strong>、<strong>DownloadByteArray</strong> 或 <strong>DownloadText</strong> 方法。</p>
<pre class="prettyprint">// Retrieve storage account from connection string CloudStorageAccount storageAccount = CloudStorageAccount.Parse(     CloudConfigurationManager.GetSetting("StorageConnectionString"));  // Create the blob client CloudBlobClient blobClient = storageAccount.CreateCloudBlobClient();  // Retrieve reference to a previously created container CloudBlobContainer container = blobClient.GetContainerReference("mycontainer");  // Retrieve reference to a blob named "myblob" CloudBlob blob = container.GetBlobReference("myblob");  // Save blob contents to disk using (var fileStream = System.IO.File.OpenWrite(@"path\myfile")) {     blob.DownloadToStream(fileStream); }</pre>
<h2><a name="delete-blobs"></a> <span class="short-header">删除 Blob</span>如何：删除 Blob</h2>
<p>最后，若要删除 Blob，请获取 Blob 引用，然后对其调用 <strong>Delete</strong> 方法。</p>
<pre class="prettyprint">// Retrieve storage account from connection string CloudStorageAccount storageAccount = CloudStorageAccount.Parse(     CloudConfigurationManager.GetSetting("StorageConnectionString"));  // Create the blob client CloudBlobClient blobClient = storageAccount.CreateCloudBlobClient();  // Retrieve reference to a previously created container CloudBlobContainer container = blobClient.GetContainerReference("mycontainer");  // Retrieve reference to a blob named "myblob" CloudBlob blob = container.GetBlobReference("myblob");  // Delete the blob blob.Delete();</pre>
<h2><a name="next-steps"></a> <span class="short-header">后续步骤</span>后续步骤</h2>
<p>现在，您已了解有关 Blob 存储的基础知识，单击下面的链接可了解如何执行更复杂的存储任务。</p>
<ul>
<li>查看 Blob 服务参考文档，了解有关可用 API 的完整详情：
<ul>
<li><a href="http://msdn.microsoft.com/zh-cn/library/windowsazure/wl_svchosting_mref_reference_home">.NET 客户端库引用</a></li>
<li><a href="http://msdn.microsoft.com/zh-cn/library/windowsazure/dd179355">REST API 参考</a></li>
</ul>
</li>
<li>在以下位置了解有关可以使用 Windows Azure 存储执行的更高级任务：<a href="http://msdn.microsoft.com/zh-cn/library/windowsazure/gg433040.aspx">在 Windows Azure 中存储和访问数据</a>。</li>
<li>查看更多功能指南，以了解在 Windows Azure 中存储数据的其他方式。
<ul><li style="display:none">使用<a href="/zh-cn/develop/net/how-to-guides/table-services/">表存储</a>存储结构化数据。</li>
<li>使用 <a href="/zh-cn/develop/net/how-to-guides/sql-database/">SQL数据库</a> 存储关系数据。</li>
</ul>
</li>
</ul>
</div>]]></bodyText><umbracoNaviHide>1</umbracoNaviHide><pageTitle>Windows Azure 开发：如何使用 Blob 存储 (.NET)</pageTitle><localize>1</localize><localizePartial>0</localizePartial><sitemapHide>0</sitemapHide><linkid>dev-net-how-to-blob-storage</linkid><metaKeywords>开始使用 Azure Blob, Azure 非结构化数据, Azure 非结构化存储, Azure Blob, Azure Blob 存储, Azure Blob .NET, Azure Blob C#, Azure Blob C#</metaKeywords><metaDescription><![CDATA[了解如何使用 Windows Azure Blob 服务上载、下载、列出和删除 Blob 内容。这些示例用 C# 编写且使用 .NET API。]]></metaDescription><headerExpose></headerExpose><footerExpose></footerExpose><urlDisplayName>Blob 服务</urlDisplayName><disqusComments>1</disqusComments><metaCanonical></metaCanonical><isHeader>0</isHeader><pageTemplate>dynamic-leftnav</pageTemplate></TextpageLeftNav></localize>