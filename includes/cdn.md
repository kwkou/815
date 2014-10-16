# 使用 Azure CDN

Azure 内容交付网络 (CDN) 为开发人员提供了一个全球解决方案，通过在美国、欧洲、亚洲、澳洲和南美洲的物理节点缓存 Blob 和计算实例的静态内容来交付高带宽内容。若要查看 CDN 节点位置的当前列表，请参阅 [Azure CDN 节点位置][Azure CDN 节点位置]。

此任务包括下列步骤：

-   [步骤 1：创建存储帐户][步骤 1：创建存储帐户]
-   [步骤 2：为存储帐户创建新的 CDN 终结点][步骤 2：为存储帐户创建新的 CDN 终结点]
-   [步骤 3：访问你的 CDN 内容][步骤 3：访问你的 CDN 内容]
-   [步骤 4：删除你的 CDN 内容][步骤 4：删除你的 CDN 内容]

使用 CDN 来缓存 Azure 数据的优点包括：

-   为远离内容源的最终用户提供更好的性能和用户体验，在使用的应用程序中需要很多“互联网往返”来加载内容
-   大规模分布，更好地处理瞬间的高负载，例如在产品发布活动开始时

现有 CDN 客户当前可在 [Azure 管理门户][Azure 管理门户]中使用 Azure CDN。CDN 是你的订阅的一项附加功能，具有单独的[计费计划][计费计划]。

<span id="Step1"></span> </a>

## 步骤 1：创建存储帐户

</p>
使用以下过程为 Azure 订阅创建新存储帐户。存储帐户提供对 Azure 存储服务的访问权限。存储帐户代表最高级别的命名空间，用于访问每个 Azure 存储服务组件：Blob 服务、队列服务和表服务。有关 Azure 存储服务的详细信息，请参阅[使用 Azure 存储服务]。

若要创建存储帐户，你必须是服务管理员或相关订阅的协同管理员。

<div class="dev-callout">
<strong>说明</strong>
<p>有关使用
Azure 服务管理 API 执行此操作的信息，请参阅<a href="http://msdn.microsoft.com/zh-cn/library/azure/hh264518.aspx">创建存储帐户</a>参考主题。</p>
</div>

**为 Azure 订阅创建存储帐户**

1.  登录到 [Azure 管理门户][Azure 管理门户]。
2.  在左下角单击“新建”，然后单击“存储”。
3.  单击“快速创建”。

    此时将显示“创建存储帐户”对话框。

    ![创建存储帐户][1]

4.  在“URL”字段中，键入子域名称。输入的名称可包含 3-24 个小写字母和数字。

    此值将成为用于对订阅的 Blob、队列或表资源进行寻址的 URI 中的主机名。若要对 Blob 服务中的容器资源进行寻址，你可以使用以下格式的 URI，其中 *\<StorageAccountLabel\>* 指你在“输入 URL”中键入的值：

    <http://>*\<StorageAcountLabel\>*.blob.core.chinacloudapi.cn/*\<mycontainer\>*

    **重要说明：**该 URL 标签构成了存储帐户 URI 的子域，在Azure 中的所有托管服务中必须是唯一的。

    此值还在门户中用作此存储帐户的名称，或在通过编程方式访问此帐户时使用。

5.  从“区域/地缘组”下拉列表中，选择存储帐户的地理位置。也可以使用地缘组。有关创建地缘组的说明，请参阅[如何在 Azure 中创建地缘组][如何在 Azure 中创建地缘组]。
6.  从“订阅”下拉列表中，选择将与存储帐户结合使用的订阅。
7.  单击**“创建存储帐户”**。创建存储帐户的过程可能需要几分钟时间完成。
8.  若要确认存储帐户已成功创建，请确认帐户显示在“存储”的所列项目中，而且其状态为“联机”。

<span id="Step2"></span> </a>

## 步骤 2：为存储帐户创建新的 CDN 终结点

</p>
启用对存储帐户或托管服务的 CDN 访问之后，所有公用对象都可以使用 CDN 边缘缓存。如果你修改了当前缓存在 CDN 中的对象，则新内容将无法通过 CDN 获取，直至 CDN 在缓存内容生存时间过期时刷新其内容。

**为存储帐户创建新的 CDN 终结点**

1.  在 [Azure 管理门户][Azure 管理门户]的导航窗格中单击“CDN”。

2.  在功能区上，单击“新建”。在“新建”对话框中，选择“应用程序服务”，然后选择“CDN”，再选择“快速创建”。

3.  在“原始域”下拉列表中，从可用存储帐户列表中选择你在上一部分中创建的存储帐户。

4.  单击“创建”按钮创建新的终结点。

5.  创建终结点之后，它将显示在订阅的终结点列表中。该列表视图显示用于访问缓存内容的 URL 以及原始域。

    原始域是 CDN 从其缓存内容的位置。原始域可以是存储帐户或云服务；在本例中，我们使用存储帐户。根据你指定的缓存控制设置或缓存网络的默认启发，存储内容被缓存到边缘服务器。有关详细信息，请参阅[如何管理 Blob 内容到期][如何管理 Blob 内容到期]。

    <div class="dev-callout">
<strong>说明</strong>
<p>为终结点创建的配置不会
即时可用；注册需要最多 60 分钟时间
通过 CDN 网络传播。试图立即
使用 CDN 域名的用户，将会收到状态代码 400
（&ldquo;错误的请求&rdquo;），直到内容可通过 CDN 获取。</p>
</div>

<span id="Step3"></span> </a>

## 步骤 3：访问 CDN 内容

</p>
若要访问 CDN 上的缓存内容，请使用门户中提供的 CDN URL。缓存 Blob 的地址将如下所示：

<http://>\<*CDNNamespace*\>.vo.msecnd.net/\<*myPublicContainer*\>/\<*BlobName*\>

<span id="Step4"></span> </a>

## 步骤 4：从 CDN 删除内容

</p>
如果不再希望在 Azure 内容交付网络 (CDN) 中缓存对象，你可以执行以下步骤之一：

-   对于 Azure Blob，你可从公共
    容器中删除 Blob。
-   你可将容器设为专用的，而不是公用的。有关详细信息，请参阅[限制对容器和 Blob 的访问][限制对容器和 Blob 的访问]。
-   你可以使用管理门户
    禁用或删除 CDN 终结点。
-   你可以修改托管服务，使其不再响应
    对象的请求。

已缓存在 CDN 中的对象将保留在缓存中，直至对象的生存时间到期。当生存时间到期时，CDN 将检查 CDN 终结点是否仍然有效，以及对象是否仍可匿名访问。如果无效，则无法再缓存该对象。

Azure CDN 当前没有可用的显式“清除”工具。

## 其他资源

-   [如何在 Azure 中创建地缘组][如何在 Azure 中创建地缘组]
-   [如何：管理 Azure 订阅的存储帐户][如何：管理 Azure 订阅的存储帐户]
-   [关于服务管理 API][关于服务管理 API]
-   [如何将 CDN 内容映射到自定义域][如何将 CDN 内容映射到自定义域]

  [Azure CDN 节点位置]: http://msdn.microsoft.com/zh-cn/library/azure/gg680302.aspx
  [步骤 1：创建存储帐户]: #Step1
  [步骤 2：为存储帐户创建新的 CDN 终结点]: #Step2
  [步骤 3：访问你的 CDN 内容]: #Step3
  [步骤 4：删除你的 CDN 内容]: #Step4
  [Azure 管理门户]: https://manage.windowsazure.cn/
  [计费计划]: /en-us/pricing/calculator/?scenario=full
  [创建存储帐户]: http://msdn.microsoft.com/zh-cn/library/azure/hh264518.aspx
  [1]: ./media/cdn/CDN_CreateNewStorageAcct.png
  [如何在 Azure 中创建地缘组]: http://msdn.microsoft.com/zh-cn/library/azure/hh531560.aspx
  [如何管理 Blob 内容到期]: http://msdn.microsoft.com/zh-cn/library/gg680306.aspx
  [限制对容器和 Blob 的访问]: http://msdn.microsoft.com/zh-cn/library/dd179354.aspx
  [如何：管理 Azure 订阅的存储帐户]: http://msdn.microsoft.com/zh-cn/library/azure/hh531567.aspx
  [关于服务管理 API]: http://msdn.microsoft.com/zh-cn/library/azure/ee460807.aspx
  [如何将 CDN 内容映射到自定义域]: http://msdn.microsoft.com/zh-cn/library/azure/gg680307.aspx
