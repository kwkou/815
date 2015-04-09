<properties urlDisplayName="" pageTitle="共享访问签名：了解 SAS 模型 | Microsoft Azure" metaKeywords="Azure blob, Azure table, Azure queue, shared access signatures" description="了解有关使用共享访问签名委托对 blob、队列和表的访问" metaCanonical="" services="storage" documentationCenter="" title="Part 1: Understanding the SAS Model" solutions="" authors="tamram" manager="adinah" />




# 共享访问签名，第 1 部分：了解 SAS 模型

使用共享访问签名 (SAS) 是将对你存储帐户中 Blob、表和队列的受限访问权限授予其他客户端且不必公开帐户密钥的一种强大的方法。在有关共享访问签名的教程的第 1 部分中，我们将概要介绍 SAS 模型并复习 SAS 最佳实践。[本教程的第 2](/zh-cn/documentation/articles/storage-dotnet-shared-access-signature-part-2/) 部分将逐步引导你完成使用 Blob 服务创建共享访问签名的过程。

## 什么是共享访问签名？##

共享访问签名对存储帐户中的资源提供委托访问。这意味着你可为指定时间段和使用指定权限集向客户端授予针对 Blob、队列或表的受限权限，而不必共享你的帐户访问密钥。SAS 是在其查询参数中包含对存储资源进行验证了身份的访问所需的所有信息的 URI。若要使用 SAS 访问存储资源，客户端只需将 SAS 传入到相应的构造函数或方法。

## 何时应使用共享访问签名？##

需要将存储帐户中资源的访问权限提供给不能使用帐户密钥进行信任的客户端时，可以使用 SAS。你的存储帐户密钥包括主密钥和辅助密钥，这两种密钥都授予对你的帐户以及其中所有资源的管理访问权限。公开这两种帐户密钥的任何一种都会向可能的恶意或负面使用开放你的帐户。共享访问签名提供一种安全的方法，允许其他客户端根据你授予的权限读取、写入和删除你的存储帐户中的数据，而无需帐户密钥。

SAS 通常适用于用户需要在你的存储帐户中读取和写入其数据的服务情形。在存储帐户存储用户数据的情形中，有两种典型的设计模式：


1\. 客户端通过执行身份验证的前端代理服务上载和下载数据。此前端代理服务的优势在于允许验证业务规则，但对于大量数据或大量事务，创建可扩展以匹配需求的服务可能成本高昂或十分困难。

![sas-storage-fe-proxy-service][sas-storage-fe-proxy-service]

2\.	轻型服务按需对客户端进行身份验证，然后生成 SAS。在客户端接收 SAS 后，它们可以直接使用 SAS 定义的权限并且针对 SAS 允许的间隔访问存储帐户资源。SAS 减少了通过前端代理服务路由所有数据的需要。

![sas-storage-provider-service][sas-storage-provider-service]

许多实际服务可能会根据涉及的情形混合使用这两种方法，也就是说，通过前端代理对某些数据进行处理和验证，同时使用 SAS 直接保存和/或读取其他数据。

## 共享访问签名的工作方式 ##

共享访问签名是一种 URI，它指向存储资源并且包含一组特殊的查询参数，这些参数指示客户端如何访问资源。签名是其中一个参数，它是由 SAS 参数构造的并且使用帐户密钥进行签名。Azure 存储空间使用该签名对 SAS 进行身份验证。

共享访问签名具有定义它的以下约束，其中每个约束都表示为针对 URI 的参数：

- **存储资源。**你可以委托访问的存储资源包括容器、Blob、队列、表和表实体范围。
- **开始时间。**这是 SAS 生效的时间。共享访问签名的开始时间是可选的；如果省略，SAS 将立即生效。
- **到期时间。**这是之后 SAS 将不再有效的时间。最佳实践建议你为 SAS 指定到期时间或将其与存储访问策略相关联（请参阅下面的更多内容）。
- **权限。**对 SAS 指定的权限指示客户端可使用 SAS 对存储资源执行哪些操作。

这里是 SAS URI 的一个示例，它提供对某一 Blob 的读写权限。该表分解了 URI 的每个部分，以便理解它是如何影响 SAS 的：

https://myaccount.blob.core.chinacloudapi.cn/sascontainer/sasblob.txt?sv=2012-02-12&st=2013-04-29T22%3A18%3A26Z&se=2013-04-30T02%3A23%3A26Z&sr=b&sp=rw&sig=Z%2FRHIX5Xcg0Mq2rqI3OlWTjEg2tYkboXr1P9ZUXDtkk%3D

<table border="1" cellpadding="0" cellspacing="0">
    <tbody>
        <tr>
            <td valign="top" width="213">
                <p>
                    Blob URI
                </p>
            </td>
            <td valign="top" width="213">
                <p>
                    https://myaccount.blob.core.chinacloudapi.cn/sascontainer/sasblob.txt
                </p>
            </td>
            <td valign="top" width="213">
                <p>
                    Blob 的地址。请注意，强烈建议使用 HTTPS。
                </p>
            </td>
        </tr>
        <tr>
            <td valign="top" width="213">
                <p>
                    存储服务版本
                </p>
            </td>
            <td valign="top" width="213">
                <p>
                    sv=2012-02-12
                </p>
            </td>
            <td valign="top" width="213">
                <p>
                    对于存储服务版本 2012-02-12 和更高版本，此参数指示要使用的版本。
                </p>
            </td>
        </tr>
        <tr>
            <td valign="top" width="213">
                <p>
                    开始时间
                </p>
            </td>
            <td valign="top" width="213">
                <p>
                    st=2013-04-29T22%3A18%3A26Z
                </p>
            </td>
            <td valign="top" width="213">
                <p>
                    以 ISO 8061 格式指定。如果你想要 SAS 立即生效，则省略开始时间。
                </p>
            </td>
        </tr>
        <tr>
            <td valign="top" width="213">
                <p>
                    到期时间
                </p>
            </td>
            <td valign="top" width="213">
                <p>
                    se=2013-04-30T02%3A23%3A26Z
                </p>
            </td>
            <td valign="top" width="213">
                <p>
                    以 ISO 8061 格式指定。
                </p>
            </td>
        </tr>
        <tr>
            <td valign="top" width="213">
                <p>
                    资源
                </p>
            </td>
            <td valign="top" width="213">
                <p>
                    sr=b
                </p>
            </td>
            <td valign="top" width="213">
                <p>
                    资源是 Blob。
                </p>
            </td>
        </tr>
        <tr>
            <td valign="top" width="213">
                <p>
                    权限
                </p>
            </td>
            <td valign="top" width="213">
                <p>
                    sp=rw
                </p>
            </td>
            <td valign="top" width="213">
                <p>
                    SAS 授予的权限包括读取 (r) 和写入 (w)。
                </p>
            </td>
        </tr>
        <tr>
            <td valign="top" width="213">
                <p>
                    签名
                </p>
            </td>
            <td valign="top" width="213">
                <p>
                    sig=Z%2FRHIX5Xcg0Mq2rqI3OlWTjEg2tYkboXr1P9ZUXDtkk%3D
                </p>
            </td>
            <td valign="top" width="213">
                <p>
                    用于对 Blob 的访问权限进行身份验证。该签名是利用 SHA256 算法通过"字符串到签名"和密钥进行计算，
                    然后使用 Base64 编码进行编码的 HMAC。
                </p>
            </td>
        </tr>
    </tbody>
</table>


## 使用存储访问策略控制共享访问签名 ##

共享访问签名可以采取以下两种形式的一种：

- **临时 SAS：**在你创建一个临时 SAS 时，针对该 SAS 的开始时间、到期时间和权限全都在 SAS URI 上指定（在省略开始时间的情况下，也可以是暗示的）。可以在容器、blob、表或队列上创建此类型的 SAS。
- **带有存储访问策略的 SAS：**存储访问策略是针对资源容器（blob 容器、表或队列）定义的，并且可以用于管理针对一个或多个共享访问签名的约束。将某一 SAS 与存储访问策略相关联时，该 SAS 将继承为存储访问策略定义的约束 - 开始时间、到期时间和权限。

这两种形式之间的差异对于一个关键情形而言十分重要：吊销。一个 SAS 就是一个 URL，因此，获取该 SAS 的任何人都可以使用它，而与是谁请求它以便开始的无关。如果某一 SAS 是公开发布的，则世界上的任何人都可以使用它。在发生以下四种情况之一前分发的 SAS 是有效的：

1.	达到了对该 SAS 指定的到期时间。
2.	达到了对该 SAS 引用的存储访问策略指定的到期时间（如果引用某一存储访问策略并且该存储访问策略指定一个到期时间）。这可能是因为经过了该间隔而发生，或者是因为你修改了该存储访问策略以使到期时间已经是过去时间而发生（这是用于吊销该 SAS 的一种方法）。
3.	删除了该 SAS 引用的存储访问策略，这是用于吊销 SAS 的另一种方法。请注意，如果你使用完全相同的名称重新创建该存储访问策略，则根据与该存储访问策略相关联的权限，所有现有 SAS 标记都将再次有效（假定尚未经过该 SAS 的到期时间）。如果你想要吊销该 SAS，请确保使用不同时间（如果你使用将来的到期时间重新创建该访问策略）。
4.	将重新生成用于创建 SAS 的帐户密钥。请注意，这样做将导致使用该帐户密钥的所有应用程序组件身份验证失败，直到这些组件更新为使用其他有效帐户密钥或者重新生成的新帐户密钥。
 
## 使用共享访问签名的最佳实践 ##

当你在应用程序中使用共享访问签名时，需要知道以下两个可能的风险：

- 如果 SAS 泄露，则获取它的任何人都可以使用它，这可能会损害你的存储帐户。
- 如果提供给客户端应用程序的 SAS 到期并且应用程序无法从你的服务检索新 SAS，则可能会影响该应用程序的功能。  

下面这些针对使用共享访问签名的建议将帮助化解这些风险：

1. **始终使用 HTTPS** 创建 SAS 或分发 SAS。如果某一 SAS 通过 HTTP 传递并且被截取，则执行中间人攻击的攻击者将能够读取 SAS、然后使用它，就像目标用户可能具有一样，这可能会暴露敏感数据或者使恶意用户能够损坏数据。
2. **在可能的情况下引用存储访问策略。**存储访问策略为你提供了撤消权限而无需重新生成存储帐户密钥的选项。这些是很长时间 （或不限），并确保它将会定期更新，将其移动到未来的得更远设置过期期限。
3. **使用临时 SAS 的近期到期时间。**这样，即使某一 SAS 不知不觉泄漏，它也只是短时间内可用。这种做法在你不能引用存储访问策略的情况下特别重要。这种做法还有助于通过限制可用于上载到 blob 的时间，来限制可以写入到其中的数据量。
4. **如有必要，让客户端自动续订 SAS。**在预期的到期时间之前，客户端应续订 SAS，以便在提供该 SAS 的服务不可用的情况下能够有重试的时间。如果你的 SAS 旨在用于少量即时的短期操作，需要在给定的到期时间内完成，那么这可能就不是必需的，因为 SAS 不需要续订。但是，如果你有定期通过 SAS 发出请求的客户端，则过期可能性就会发挥作用。关键的考虑因素是，平衡 SAS 寿命较短（如上所述）的需求与确保客户端及早请求续订（以避免因 SAS 在成功续订前到期而中断）的需求。
5. **谨慎设置 SAS 开始时间。**如果将某一 SAS 的开始时间设置为**"现在"**，则可能会由于时钟偏差（由于计算机不同而存在的当前时间差异）而在前几分钟内间歇性地发生故障。一般情况下，请将开始时间设置为至少 15 分钟前，或者根本不设置它，这会使它在所有情况下都立即生效。同样的原则也适用于到期时间，请记住，对于任一方向上的任何请求，你可能会观察到最多 15 分钟的时钟偏差。请注意，对于使用早于 2012-02-12 的 REST 版本的客户端，不引用存储访问策略的 SAS 的最长持续时间为 1 小时，并且任何指定长于该时间的期限的策略均将失败。
6.	**具体对待要访问的资源。**典型的安全最佳实践是向用户提供所需的最小权限。如果用户仅需要对单个实体的读取访问权限，则为他们授予对该单个实体的读取访问权限，而不为他们授予对所有实体的读取/写入/删除访问权限。这也有助于减轻泄漏 SAS 的威胁，因为该 SAS 在攻击者的手中的威力将会较小。
7.	**了解你的帐户将由于任何用途而支出费用，包括在使用 SAS 的情况下。**如果你提供对 blob 的写入访问权限，则用户可以选择上载 200GB blob。如果你还向用户提供读取访问权限，则他们可能选择下载它 10 次，从而为你产生 2TB 的传出费用。同样，请提供有限的权限，以便帮助减轻恶意用户发起攻击的可能性。使用短期 SAS 可减少这种威胁（但请注意结束时间上的时钟偏差）。
8.	**验证使用 SAS 写入的数据。**客户端应用程序将数据写入你的存储帐户时，请切记该数据可能存在问题。如果应用程序需要该数据进行验证或经过授权后才可供使用，你应在写入数据后，在你的应用程序使用它之前执行此验证。这种做法还有助于防止正常获取 SAS 的用户或利用泄漏的 SAS 的用户将损坏的数据或恶意数据写入到你的帐户。
9. **不要总是使用 SAS。**有时与针对你存储帐户的特定操作相关联的风险超过了 SAS 的好处。对于此类操作，请创建中间层服务，以便在执行业务规则验证、身份验证和审核后写入到你的存储帐户。此外，有时以其他方式管理访问权限将更加简单。例如，如果想要使一个容器中的所有 blob 公开可读，则可以将该容器设为"公共"，而不是为每个客户端都提供一个 SAS 以便进行访问。
10.	**使用"存储分析"监视应用程序。**可以使用日志记录和度量来观察由于中断 SAS 提供程序服务或无意中删除存储访问策略而导致的任何身份验证失败高发情形。有关其他信息，请参阅 [Azure 存储空间团队博客](http://blogs.msdn.com/b/windowsazurestorage/archive/2011/08/03/windows-azure-storage-logging-using-logs-to-track-storage-requests.aspx)。

## 结论 ##

共享访问签名用于将存储帐户的受限权限提供给不应具有帐户密钥的客户端。因此，它们是安全模型的重要环节，适合使用 Azure 存储空间的任何应用程序。如果你按照本文中介绍的最佳实践执行，则可以使用 SAS 更灵活地访问你的存储帐户中的资源，且不会影响应用程序的安全性。

## 后续步骤 ##

[共享访问签名，第 2 部分：创建 SAS 并将其用于 Blob 服务](/zh-cn/documentation/articles/storage-dotnet-shared-access-signature-part-2/)

[管理对 Azure 存储资源的访问](http://msdn.microsoft.com/zh-cn/library/azure/ee393343.aspx)

[使用共享访问签名 (REST API) 委托访问权限](http://msdn.microsoft.com/zh-cn/library/azure/ee395415.aspx)

[介绍表和队列 SAS](http://blogs.msdn.com/b/windowsazurestorage/archive/2012/06/12/introducing-table-sas-shared-access-signature-queue-sas-and-update-to-blob-sas.aspx)
[sas-storage-fe-proxy-service]: ./media/storage-dotnet-shared-access-signature-part-1/sas-storage-fe-proxy-service.png
[sas-storage-provider-service]: ./media/storage-dotnet-shared-access-signature-part-1/sas-storage-provider-service.png


<!--HONumber=41-->
