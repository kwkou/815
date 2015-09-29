<properties
   pageTitle="内容交付网络 (CDN) 指南 | Windows Azure"
   description="有关使用内容交付网络 (CDN) 传送 Azure 中托管的高带宽内容的指南。"
   services=""
   documentationCenter="na"
   authors="dragon119"
   manager="masimms"
   editor=""
   tags=""/>

<tags
   ms.service="best-practice"
   ms.date="04/28/2015"
   wacn.date="08/29/2015"/>

# 内容交付网络 (CDN) 指南

## 概述

Microsoft Azure 内容交付网络 (CDN) 为开发人员提供一个全局解决方案用于传递 Azure 中托管的高带宽内容。CDN 缓存公开从 Azure Blob 存储加载的可用对象，或位于战略性位置的应用程序文件夹的对象，以提供最大带宽向用户传送内容。它通常用于传送静态内容，例如图像、样式表、文档、文件、客户端脚本和 HTML 页面。

使用 CDN 的主优点是较低的延迟和更快将内容传送给用户，而与托管应用程序的数据中心的地理位置无关，并且可降低应用程序本身的负载，因为它能缓解访问和传送内容所需的处理。这种降低负载的优点可帮助提高应用程序的性能和可缩放性，以及减少达成特定级别的性能和可用性所需的处理资源，从而将托管成本降到最低。

如果 Azure CDN 不符合需要，你可以在应用程序中使用不是由 Azure 实施的其他内容交付网络系统。或者，你可以通过公开 Azure 存储空间或 Azure 计算实例中的静态内容，为使用其他提供商托管的应用程序使用 Azure CDN。![](./media/best-practices-cdn/CDN.png)

## 如何和为何使用 CDN

CDN 的典型用途包括：

- 传送客户端应用程序的静态资源，通常来自网站。这些内容可以是图像、样式表、文档、文件、客户端脚本、HTML 页面、HTML 片段，或服务器无需为每个请求修改的任何其他内容。应用程序可以在运行时创建项，并让 CDN 使用它们（例如，通过创建当前头条新闻列表），但不会为每个请求都这么做。

- 将公共静态内容和共享内容传送给设备，例如移动电话和平板电脑，其中应用程序本身是一个将 API 提供给客户端的 Web 服务。除了服务器无需为每个请求修改的其他内容外，CDN 可以传送静态数据集供客户端使用，（也许用于生成客户端 UI）。例如，它可用于传送 JSON 或 XML 文档。

- 让客户端使用仅包含公共静态内容的整个网站，而无需任何专用计算资源。

- 根据需要将视频文件流式传输到客户端。视频可从提供 CDN 连接的全球数据中心获得低延迟和可靠连接的好处。

- 一般而言，这可以改善用户体验，特别是与应用程序的数据中心位置相距遥远，从而发生较高延迟的用户。Web 应用程序中内容的总大小基本上都是静态的，使用 CDN 有助于保持性能和总体用户体验，同时无需将应用程序部署到多个数据中心。

- 应对为物联网 (IoT) 的移动设备和固定设备提供服务的应用程序日益增加的负载。如果需要处理广播消息和直接管理固件更新分发，则大量此类设备和装置可能很容易使应用程序无法应对。

- 处理高峰和偶发性请求而不需应用程序缩放，避免后续增加营运成本。例如，对于硬件设备（例如特定型号的路由器）或消费型设备（例如智能电视）将更新发布到操作系统时，如果数以百万的用户和设备在短期间内下载更新，则可能会根据需要生成庞大的高峰量。

- 下表显示了来自不同地理位置的第一个字节的时间中间值示例。目标 Web 角色部署到 Azure 美国西部。因为 CDN 很靠近 CDN 节点，因此较大的促升之间有更强的关联性。<!--有关 Azure CDN 节点位置的列表，请参阅 [Azure 内容交付网络 (CDN) 节点位置](http://msdn.microsoft.com/zh-cn/library/azure/gg680302.aspx)。-->

<table xmlns:xlink="http://www.w3.org/1999/xlink"><tr><th><a name="_MailEndCompose" href="#"><span /></a><br /></th><th><p>距第一字节的时间（源）</p></th><th><p>距第一字节的时间 (CDN)</p></th><th><p>% 更快的速度，适用于 CDN</p></th></tr><tr><td><p>* San Jose, CA</p></td><td><p>47.5</p></td><td><p>46.5</p></td><td><p>2 %</p></td></tr><tr><td><p>** Dulles, VA</p></td><td><p>109</p></td><td><p>40.5</p></td><td><p>169 %</p></td></tr><tr><td><p>Buenos Aires, AR</p></td><td><p>210</p></td><td><p>151</p></td><td><p>39 %</p></td></tr><tr><td><p>* London, UK</p></td><td><p>195</p></td><td><p>44</p></td><td><p>343 %</p></td></tr><tr><td><p>Shanghai, CN</p></td><td><p>242</p></td><td><p>206</p></td><td><p>17 %</p></td></tr><tr><td><p>* Singapore</p></td><td><p>214</p></td><td><p>74</p></td><td><p>189 %</p></td></tr><tr><td><p>* Tokyo, JP</p></td><td><p>163</p></td><td><p>48</p></td><td><p>240 %</p></td></tr><tr><td><p>Seoul, KR</p></td><td><p>190</p></td><td><p>190</p></td><td><p>0 %</p></td></tr></table>* 在同一城市中具有 Azure CDN 节点。  
* 在邻近城市中具有 Azure CDN 节点。


## 挑战  

计划使用 CDN 时必须考虑到几个难题：

- **部署** 必须确定从何处加载 CDN 内容（CDN 将从中提取内容的来源），以及是否需要将内容部署到多个存储系统（例如在 CDN 和替代位置中）。

- 应用程序部署机制必须考虑部署静态内容和资源，以及部署应用程序文件，例如 ASPX 页。例如，可能需要执行单独的步骤才能将内容加载到 Azure Blob 存储中。

- **版本控制和缓存控制** 必须考虑如何更新静态内容和部署新版本。CDN 当前不提供用于清理内容的机制，因此才有可用的新版本。这与管理客户端缓存（如在 Web 浏览器中）时遇到的难题类似。

- **测试** 在本地或在过渡环境中开发和测试应用程序时，很难对 CDN 设置执行本地测试。

- **SEO** 当使用图像和文档等内容时，它们都由不同的域提供，这会影响此内容的 SEO。

- **安全性** 许多 CDN 服务（例如 Azure CDN）当前不提供任何类型的内容访问控制。

- **复原能力** CDN 是应用程序潜在的单个故障点。它比 Blob 存储具有更低的可用性 SLA（可用于直接传送内容），因此可能需要考虑为重要内容实施故障回复机制。

- 客户端可从不允许访问 CDN 上资源的环境中连接。这可能是安全约束的环境，限制只能访问一组已知源，或防止从页来源以外的任何位置加载资源的环境。因此，需要实施故障回复。

- 应该实施一种机制来通过 CDN 监视内容可用性。

CDN 用处不大的应用场景包括：

- 内容具有低点击率，因此在生存时间的有效期间内可能偶尔访问，或只访问一次。第一次下载项时产生两笔交易费用（从来源到 CDN，然后从 CDN 到客户）。

- 数据是专用的，例如用于大型企业或供应链生态系统。


## 一般性指导原则和最佳实践

使用 CDN 是最小化应用程序负载和最大化可用性与性能的良好做法。你应该为应用程序使用的所有适当内容和资源考虑此方案。设计使用 CDN 的策略时，请注意以下几点：

- **来源** 通过 CDN 部署内容时，只需指定 CDN 服务用于访问和缓存内容的 HTTP（端口 80）终结点。+ 终结点可以指定 Azure Blob 存储容器，包括要通过 CDN 传送的静态内容。容器必须标记为公共。只能通过 CDN 获取公共容器中具有公共读取权限的 Blob。

- 终结点可在应用程序的某个计算层（例如 Web 角色或虚拟机）的根目录中指定名为 **cdn** 的文件夹。将在 CDN 上缓存资源请求的结果，包括动态资源，例如 ASPX 页。最小缓存期限为 300 秒。任何较短的期限会阻止将内容部署到 CDN（有关详细信息，请参阅“<a href="#cachecontrol" xmlns:dt="uuid:C2F41010-65B3-11d1-A29F-00AA00C14882" xmlns:xlink="http://www.w3.org/1999/xlink" xmlns:MSHelp="http://msdn.microsoft.com/mshelp">缓存控制</a>”部分）。

- 如果你使用 Azure 网站，可在创建 CDN 实例时选择网站，将终结点设置为该网站的根文件夹。可通过 CDN 获取网站的所有内容。

- 在大多数情况下，指向应用程序的某个计算层中文件夹内的 CDN 终结点将提供更大的弹性和控制度。例如，可让你更轻松地管理当前和将来的路由要求，以及动态生成静态内容（如图像缩图）。

- 从 ASPX 页等动态源传送内容时，可以使用查询字符串来区分缓存中的对象。但是，可以在指定 CDN 终结点时，通过管理门户中的设置来禁用这种行为。从 Blob 存储传送内容时，查询字符串被视为字符串文本，因此将有相同名称但不同查询字符串的两个项存储为 CDN 上的独立项。

- 可以对脚本等资源和其他内容使用 URL 重写，以避免将文件移到 CDN 原始文件夹。

- 使用 Azure 存储空间 Blob 保存 CDN 内容时，Blob 中资源的 URL 对容器和 Blob 名称区分大小写。

- 使用 Azure 网站时，可以在资源的链接中指定 CDN 实例的路径。例如，以下操作在通过 CDN 传送的网站的 **Images** 文件夹中指定图像文件：

  ```
  <img src="http://[your-cdn-instance].vo.msecnd.net/Images/image.jpg" />
  ```

- **部署** 如果未在应用程序部署包或过程中包含静态内容，该静态内容可能需要与应用程序分开设置和部署。考虑这如何影响版本控制方法，也就是用于管理应用程序组件和静态资源内容的方法。

- 考虑如何针对脚本与 CSS 文件处理绑定（将多个文件组合成一个文件）和缩减（删除不必要的字符，例如空格、换行符、注释和其他字符）。这些常用的方法可减少客户端的加载时间，并与通过 CDN 传送的内容兼容。有关详细信息，请参阅[捆绑和缩减](http://www.asp.net/mvc/tutorials/mvc-4/bundling-and-minification)。

- 如果需要将内容部署到其他位置，则这属于部署过程的附加步骤。如果应用程序更新 CDN 的内容（也许是按固定间隔或为了响应事件），则必须将更新的内容存储在任何其他位置以及 CDN 的终结点中。

- 无法为 Azure 过渡环境中部署的应用程序，或在本地 Azure 模拟器或 Visual Studio 中，设置 CDN 终结点。这会影响单元测试、功能测试，以及最终部署前测试。必须实施替代机制来支持这种做法。例如，可以使用临时自定义应用程序或公用程序将内容预先部署到 CDN，然后在缓存内容期间执行测试。或者，使用汇编指令或全局常量控制应用程序加载资源的位置。例如，在调试模式下运行时，可以从本地文件夹加载资源，例如客户端脚本绑定和其他内容，并在发布模式下运行时使用 CDN。

- CDN 不支持任何本机压缩功能。但是，它会传送已压缩的内容，例如 zip 和 gzip 文件。将应用程序文件夹用作 CDN 终结点时，服务器可能会按默认压缩一些内容，并且使用将内容直接传送到 Web 浏览器或其他类型的客户端时的同一方法。这依赖于从客户端发送的 **Accept-Encoding** 值。在 Azure 中，当 CPU 利用率低于 50% 时会按默认自动压缩内容。更改配置可能需要使用启动任务在 IIS 中打开动态输出压缩（如果使用云服务来托管应用程序）。有关详细信息，请参阅[通过 Web 角色使用 Azure CDN 来启用 gzip 压缩](http://blogs.msdn.com/b/avkashchauhan/archive/2012/03/05/enableing-gzip-compression-with-windows-azure-cdn-through-web-role.aspx)。

- **路由和版本控制** 你可能需要使用不同的 CDN 实例；例如，当部署新版的应用程序时，可能要使用不同的 CDN。如果将 Azure Blob 存储用作内容来源，可以只创建一个独立存储帐户或容器，并将 CDN 终结点指向它。如果将应用程序中的 **cdn** 根文件夹用作 CDN 终结点，则可以使用 URL 重写方法将请求定向到不同的文件夹。

- 不要使用查询字符串来表示 CDN 上资源链接中不同版本的应用程序，因为从 Azure Blob 存储提取内容时，查询字符串是资源名称（Blob 名称）的一部分。这还会影响客户端缓存资源的方式。

- 如果在 CDN 上缓存以前的资源，则在部署新版本的静态内容时更新应用程序可能是一个难题。有关详细信息，请参阅“<a href="#cachecontrol" xmlns:dt="uuid:C2F41010-65B3-11d1-A29F-00AA00C14882" xmlns:xlink="http://www.w3.org/1999/xlink" xmlns:MSHelp="http://msdn.microsoft.com/mshelp">缓存控制</a>”部分。

<a name="cachecontrol" href="#" xmlns:xlink="http://www.w3.org/1999/xlink"><span /></a>**缓存控制**+ 考虑如何管理缓存，以及在哪些应用程序层上管理。例如，在将文件夹用作 CDN 来源时，可以指定生成内容的页面可缓存性，以及特定文件夹中所有资源的内容过期时间。还可以为 CDN 以及使用标准 HTTP 标头的客户端指定缓存属性。尽管你可能已在服务器和客户端上管理缓存，但使用 CDN 可让你更了解内容的缓存方式和位置。

- 若要防止在 CDN 上使用对象，可以从来源中删除这些对象（Blob 容器或应用程序 **cdn** 根文件夹）、移除或删除 CDN 终结点，或者在使用 Blob 存储的情况下，将容器或 Blob 设为专用。但是，仅当项的生存时间 (TTL) 已过期时，才从 CDN 中将其删除。

- 如果未指定缓存过期期限（例如从 Blob 存储加载内容时），内容将在 CDN 中最长缓存 72 个小时。

- 在 Web 应用程序中，可以在 web.config 文件的 **system.webServer/staticContent** 节中，使用 **clientCache** 元素来配置所有内容的缓存和过期。你可以在任何文件夹中放置 web.config 文件，以便影响该文件夹中的文件和所有子文件夹中的文件。

- 如果使用动态方法（例如 ASP.NET）来创建 CDN 的内容，请确保在每个页面上指定 **Cache.SetExpires** 属性。CDN 不会缓存使用默认可缓存性设置（公共）的页面的输出。将缓存过期期限设置为适当值，以确保不会在极短时间内从应用程序丢弃和重新加载内容。

- **安全性** CDN 可以使用 CDN 提供的证书通过 HTTPS (SSL) 传送内容，但它还可以通过 HTTP 传送内容。无法阻止 HTTP 访问 CDN 中的项。你可能需要使用 HTTPS 来请求通过 HTTPS 加载的页面所显示的静态内容（例如购物车），以避免浏览器发出有关内容混合的警告。

- 许多 CDN 服务（例如 Azure CDN）当前不提供任何访问控制工具来安全访问内容。无法对 CDN 使用共享访问签名 (SAS)。

- 如果使用 CDN 传送客户端脚本，则当这些脚本使用 **XMLHttpRequest** 调用对数据、图像或字体等其他资源，或不同域中的字体发出 HTTP 请求时，可能会遇到问题。许多 Web 浏览器会阻止跨源资源共享 (CORS)，除非 Web 服务器已配置为设置相应的响应标头。有关 CORS 的详细信息，请参阅 <span class="highlight" xmlns:dt="uuid:C2F41010-65B3-11d1-A29F-00AA00C14882" xmlns:xlink="http://www.w3.org/1999/xlink" xmlns:MSHelp="http://msdn.microsoft.com/mshelp">API 安全注意事项</span>指南中的“威胁缓解”部分。可以将 CDN 配置为支持 CORS：+ 如果内容传送来源是 Azure Blob 存储，可以将 **CorsRule** 添加到服务属性。该规则可以指定 CORS 请求的允许来源、GET 等允许的方法，以及以秒为单位的规则最长期限（在加载原始内容后，客户端必须请求链接资源的时间段）。有关详细信息，请参阅 [Azure 存储空间服务的跨域资源共享 (CORS) 支持](http://msdn.microsoft.com/zh-cn/library/azure/dn535601.aspx)。

- 如果内容传送来源是应用程序中的文件夹（例如 **cdn** 根文件夹），你可以在应用程序配置文件中配置出站规则，以便在所有响应中设置 **Access-Control-Allow-Origin** 标头。有关使用重写规则的详细信息，请参阅 [URL 重写模块](http://www.iis.net/learn/extensions/url-rewrite-module)。请注意，使用 Azure 网站时不可以使用此方法。

- **自定义域**+ 大多数 CDN（包括 Azure CDN）允许你指定自定义域名，并使用它通过 CDN 访问资源。你还可以在 DNS 中使用 **CNAME** 记录来设置自定义子域名。使用这种方法可以提供另一个抽象与控制层。

- 如果你使用 **CNAME**，则无法（在编写本指南时）另外使用 SSL，因为 CDN 使用自身的单个 SSL 证书，而这与自定义域/子域名不匹配。

- **CDN 故障回复** 应该考虑应用程序如何处理 CDN 失败或暂时不可用的情况。如果 CDN 不可用，客户端应用程序可以使用执行前面请求期间的本地缓存资源的副本（位于客户端），或者可以使用代码来检测失败，而不是从来源请求资源（应用程序文件夹或保留资源的 Azure Blob 容器）。

- **SEO** 如果 SEO 是应用程序中重要的考虑因素，请确保：+ 在每个页面或资源中包含 **Rel** 规范标头。

- 使用 **CNAME** 子域记录，并使用此名称访问资源。

- 考虑 CDN 的 IP 地址所在的国家或地区，可能与应用程序本身所在的国家或地区不同而造成的影响。

- 使用 Azure Blob 存储作为来源时，针对 CDN 上的资源保持与应用程序文件夹中相同的文件结构。


- **监视和日志记录** 添加 CDN 作为应用程序监视策略的一部分，以检测并度量失败或发生的过度延迟现象。

- 为 CDN 启用日志记录，并在日常操作期间执行日志记录。

- **成本影响** 当 CDN 从应用程序加载数据时，你需要支付 CDN 和存储事务的出站数据传输费用。应该为内容设置可行的缓存过期期限以确保最新状态，但不要短到将内容从应用程序或 Blob 存储中重复地重新加载到 CDN。但是，使用过长的过期期限将更难从 CDN 删除项，因为必须等待其过期。

- 很少下载的项会产生两笔事务费用，而且不会大幅降低服务器负载。



## 示例代码
本部分包含一些代码示例以及有关使用 CDN 的技巧。


## URL 重写
以下内容摘自某个云服务托管应用程序根目录中的 Web.config 文件，其中演示了在使用 CDN 时如何执行 URL 重写。CDN 发出的、将要缓存的内容请求将会根据资源类型（例如脚本和图像）重定向到应用程序根目录中的特定文件夹。

```XML
<system.webServer>
  ...
  <rewrite>
    <rules>
      <rule name="VersionedResource" stopProcessing="false">
        <match url="(.*)_v(.*).(.*)" ignoreCase="true" />
        <action type="Rewrite" url="{R:1}.{R:3}" appendQueryString="true" />
      </rule>
      <rule name="CdnImages" stopProcessing="true">
        <match url="cdn/Images/(.*)" ignoreCase="true" />
        <action type="Rewrite" url="/Images/{R:1}" appendQueryString="true" />
      </rule>
      <rule name="CdnContent" stopProcessing="true">
        <match url="cdn/Content/(.*)" ignoreCase="true" />
        <action type="Rewrite" url="/Content/{R:1}" appendQueryString="true" />
      </rule>
      <rule name="CdnScript" stopProcessing="true">
        <match url="cdn/Scripts/(.*)" ignoreCase="true" />
        <action type="Rewrite" url="/Scripts/{R:1}" appendQueryString="true" />
      </rule>
      <rule name="CdnScriptBundles" stopProcessing="true">
        <match url="cdn/bundles/(.*)" ignoreCase="true" />
        <action type="Rewrite" url="/bundles/{R:1}" appendQueryString="true" />
      </rule>
    </rules>
  </rewrite>
  ...
</system.webServer>
```

添加的重写规则将执行以下重定向：

- 第一个规则可让你在资源的文件名中嵌入版本，然后将忽略该版本。例如，**Filename_v123.jpg** 重写为 **Filename.jpg**。

- 接下来，四个规则显示了当不想要将资源存储在 Web 角色的根目录中名为 **cdn** 的文件夹时，如何重定向请求。规则将 **cdn/Images**、**cdn/Content**、**cdn/Scripts** 和 **cdn/bundles** URL 映射到它们在 Web 角色中的相应根文件夹。使用 URL 重写需要对资源绑定做出一些更改。


## 绑定和缩减 ##

可以由 ASP.NET 处理绑定和缩减。在 MVC 项目中，可以在 **BundleConfig.cs** 中定义绑定。通过调用 **Script.Render** 方法（通常在代码的 view 类中），可以创建对缩减脚本绑定的引用。此引用包含内含一个基于绑定内容的查询字符串（包括哈希）。如果绑定内容发生更改，生成的哈希也会更改。  
默认情况下，Azure CDN 实例已禁用“查询字符串状态”设置。为了使 CDN 正确处理更新的脚本绑定，你必须为 CDN 实例启用“查询字符串状态”设置。请注意，在创建 CDN 并使设置更改生效之前，可能需要一个小时以上。


## 详细信息
+ [Azure CDN](/home/features/caching/)
+ [Azure 内容交付网络 (CDN) 概述](http://msdn.microsoft.com/zh-cn/library/azure/ff919703.aspx)
+ [Azure 内容交付网络最佳实践](http://azure.microsoft.com/blog/2011/03/18/best-practices-for-the-windows-azure-content-delivery-network/)

<!---HONumber=67-->