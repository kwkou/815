<properties
    pageTitle="Azure 存储安全指南 | Azure"
    description="详细介绍保护 Azure 存储的多种方法，包括但不限于 RBAC、存储服务加密、客户端加密、SMB 3.0 和 Azure 磁盘加密。"
    services="storage"
    documentationcenter=".net"
    author="robinsh"
    manager="timlt"
    editor="tysonn" />
<tags
    ms.assetid="6f931d94-ef5a-44c6-b1d9-8a3c9c327fb2"
    ms.service="storage"
    ms.workload="storage"
    ms.tgt_pltfrm="na"
    ms.devlang="dotnet"
    ms.topic="article"
    ms.date="12/08/2016"
    wacn.date="01/06/2017"
    ms.author="robinsh" />

#Azure 存储安全指南

##概述

Azure 存储提供一套完善的安全功能，这些功能相辅相成，让开发人员能够生成安全的应用程序。存储帐户本身可以通过基于角色的访问控制和 Azure Active Directory 来保护。在应用程序和 Azure 之间传输数据时，可以使用[客户端加密](/documentation/articles/storage-client-side-encryption/)、HTTPS 或 SMB 3.0 来保护数据。使用[存储服务加密 (SSE)](/documentation/articles/storage-service-encryption/) 写入 Azure 存储时，可将数据设置为自动加密。可以使用 Azure 磁盘加密将虚拟机所用的 OS 和数据磁盘设置为进行加密。可以使用[共享访问签名](/documentation/articles/storage-dotnet-shared-access-signature-part-1/)来授予对 Azure 存储中数据对象的委派访问权限。

本文将概述其中可用于 Azure 存储的每项安全功能。提供了详述每项功能的文章的链接，让你能够轻松深入每个主题。

本文涵盖以下主题：

-   [管理平面安全性](#management-plane-security) - 保护存储帐户

    管理平面包含用于管理存储帐户的资源。此部分介绍 Azure Resource Manager 部署模型，以及如何使用基于角色的访问控制 (RBAC) 来控制访问存储帐户。此外，还将介绍如何管理存储帐户密钥以及重新生成此类密钥。

-   [数据平面安全](#data-plane-security) - 保护对数据的访问

    此部分介绍如何使用共享访问签名和存储访问策略，允许访问存储帐户中的实际数据对象，例如 Blob、文件、队列和表。将介绍服务级别 SAS 和帐户级别 SAS。此外，还将介绍如何限制访问特定的 IP 地址（或 IP 地址范围）、如何限制用于 HTTPS 的协议，以及如何在共享访问签名过期前撤销它。

-   [传输中加密](#encryption-in-transit)

    此部分介绍将数据传入和传出 Azure 存储时如何保护数据。将介绍 HTTPS 的建议用法，以及 SMB 3.0 对 Azure 文件共享使用的加密。同时还将介绍客户端加密，此法可用于在客户端应用程序中将数据传输到存储前加密数据，以及从存储传出数据后对其进行解密。

-   [静态加密](#encryption-at-rest)

    将介绍存储服务加密 (SSE) 以及如何为存储帐户启用它，从而在将块 Blob、页 Blob 和追加 Blob 写入 Azure 存储时自动对其加密。还将介绍如何使用 Azure 磁盘加密，并探究磁盘加密、SSE 与客户端加密之间的基本差异和情况。

-   使用[存储分析](#storage-analytics)审核 Azure 存储的访问

    此部分介绍如何在存储分析日志中查找某个请求的相关信息。将介绍实际的存储分析日志数据，并了解如何分辨请求是通过存储帐户密钥、共享访问签名还是以匿名方式发出的，以及该请求是成功还是失败。

-   [使用 CORS 启用基于浏览器的客户端](#Cross-Origin-Resource-Sharing-CORS)

    此部分介绍如何允许跨域资源共享 (CORS)。将介绍跨域访问，以及如何使用 Azure 存储内置的 CORS 功能来处理这种访问。

##<a id="management-plane-security"></a>管理平面安全性

管理平面包含影响存储帐户本身的操作。例如，可以创建或删除存储帐户、获取订阅中的存储帐户列表、检索存储帐户密钥，或重新生成存储帐户密钥。

创建新的存储帐户时，可以选择经典或 Resource Manager 部署模型。在 Azure 中创建资源的经典模型只允许以孤注一掷的方式访问订阅，然后访问存储帐户。

本指南着重于 Resource Manager 模型，即创建存储帐户的建议方法。使用 Resource Manager 存储帐户而不是提供整个订阅的访问权限，可以使用基于角色的访问控制 (RBAC) 以更高的限制级别来控制对管理平面的访问。

###如何使用基于角色的访问控制 (RBAC) 来保护存储帐户

下面介绍 RBAC 的定义及其用法。每个 Azure 订阅都有 Azure Active Directory。可以向该目录中的用户、组和应用程序授予访问权限，以便其管理使用 Resource Manager 部署模型的 Azure 订阅中的资源。这称为基于角色的访问控制 (RBAC)。若要管理此访问权限，可以使用 [Azure 门户](https://portal.azure.cn/)、[Azure CLI 工具](/documentation/articles/xplat-cli-install/)、[PowerShell](/documentation/articles/powershell-install-configure/) 或 [Azure 存储资源提供程序 REST API](https://msdn.microsoft.com/zh-cn/library/azure/mt163683.aspx)。

使用 Resource Manager 模型可以将存储帐户放在资源组中，并使用 Azure Active Directory 来控制对该特定存储帐户的管理平面的访问。例如，你可以授权特定用户访问存储帐户密钥，而其他用户可以查看有关存储帐户的信息，但无法访问存储帐户密钥。

####授予访问权限

通过将相应的 RBAC 角色分配给适当范围的用户、组和应用程序来授予访问权限。若要授予对整个订阅的访问权限，请在订阅级别分配角色。可以将权限授予资源组本身，以对该授予资源组中所有资源的访问权限。也可以将特定角色分配给特定资源，例如存储帐户。

关于使用 RBAC 访问 Azure 存储帐户的管理操作，需了解以下要点：

-   分配访问权限，就是将角色分配给你希望拥有访问权限的帐户。可控制对用于管理该存储帐户的操作的访问权限，但不能控制对帐户中数据对象的访问权限。例如，可以授予检索存储帐户属性（例如冗余）的权限，但不能授予检索 Blob 存储中的容器或容器中的数据的权限。

-   针对有权访问存储帐户中数据对象的用户，可为他们提供权限来读取存储帐户密钥，而该用户可以使用这些密钥来访问 Blob、队列、表和文件。

-   可将角色以分配给特定的用户帐户、用户组或应用程序。

-   每个角色都有“操作”和“非操作”列表。例如，“虚拟机参与者”角色具有“listKeys”操作，用于读取存储帐户密钥。“参与者”具有“非操作”，例如在 Active Directory 中更新用户的访问权限。

-   存储的角色包括（但不限于）：

	-	所有者 – 他们可以管理一切，包括访问权限。

    -	参与者 – 他们可以执行所有者可执行的所有操作，但分配访问权限除外。拥有此角色的用户可以查看和重新生成存储帐户密钥。他们可以使用存储帐户密钥来访问数据对象。

    -	读者 – 他们可以查看有关存储帐户的信息（机密除外）。例如，如果将存储帐户中拥有读者权限的角色分配给某个用户，该用户就可以查看存储帐户的属性，但无法对属性进行任何更改或查看存储帐户密钥。

    -	存储帐户参与者 - 他们可以管理存储帐户，既可读取订阅的资源组和资源，又可创建和管理订阅资源组部署。他们也可以访问存储帐户密钥，这又意味着他们可以访问数据平面。

    -	用户访问管理员 – 他们可以管理对存储帐户的用户访问。例如，他们可将“读者”权限授予特定用户。

    -	虚拟机参与者 – 他们可以管理虚拟机，但无法管理已连接的存储帐户。此角色可以列出存储帐户密钥，意味着分配此角色的用户可以更新数据平面。

		为了让用户能够创建虚拟机，他们必须能够在存储帐户中创建相应的 VHD 文件。为此，他们需要能够检索存储帐户密钥，并将它传递给创建 VM 的 API。因此，他们必须拥有此权限才能列出存储帐户密钥。

- 利用定义自定义角色的功能，可以从可对 Azure 资源执行的操作的列表中整理出一组操作。

- 必须先在 Azure Active Directory 中设置用户，才能为他们分配角色。

- 可以创建一份报告，描述哪个用户使用了 PowerShell 或 Azure CLI 在哪个范围为哪些对象授予/撤销哪种类型的访问权限。

####资源

-   [Azure Active Directory 基于角色的访问控制](/documentation/articles/role-based-access-control-configure/)

    此文解释了 Azure Active Directory 基于角色的访问控制及其工作原理。

-   [RBAC：内置角色](/documentation/articles/role-based-access-built-in-roles/)

    此文详细说明了 RBAC 中所有可用的内置角色。

-   [了解资源管理器部署和经典部署](/documentation/articles/resource-manager-deployment-model/)

    此文介绍了 Resource Manager 部署和经典部署模型，并说明使用 Resource Manager 和资源组的优点


-   [使用 REST API 管理基于角色的访问控制](/documentation/articles/role-based-access-control-manage-access-rest/)

	此文说明如何使用 REST API 来管理 RBAC。

-   [Azure 存储资源提供程序 REST API 参考](https://msdn.microsoft.com/zh-cn/library/azure/mt163683.aspx)

	有关使用 API 以编程方式管理存储帐户的参考信息。

-   [使用 Azure Resource Manager 进行身份验证的开发人员指南](http://www.dushyantgill.com/blog/2015/05/23/developers-guide-to-auth-with-azure-resource-manager-api/)

	此文说明如何使用 Resource Manager API 进行身份验证。

-   [Ignite 中提供的适用于 Azure 的基于角色的访问控制](https://channel9.msdn.com/events/Ignite/2015/BRK2707)

    这是第 9 频道中提供的 2015 MS Ignite 会议视频链接。此次研讨会讨论了 Azure 中的访问管理和报告功能，并探索使用 Azure Active Directory 安全访问 Azure 订阅的最佳实践。

###管理存储帐户密钥

存储帐户密钥是由 Azure 创建的 512 位字符串，配合存储帐户名称用于访问存储于存储帐户中的数据对象，例如 Blob、表中条目、队列消息，以及 Azure 文件共享中的文件。控制对存储帐户密钥的访问，将控制对该存储帐户的数据平面的访问。

每个存储帐户在 [Azure 门户](http://portal.azure.cn/)和 PowerShell cmdlet 中有两个称为“密钥 1”和“密钥 2”的密钥。可以使用多种方法之一手动重新生成这些密钥，包括（但不限于）使用 [Azure 门户](https://portal.azure.cn/)、PowerShell、Azure CLI，或以编程方式使用 .NET 存储客户端库或 Azure 存储服务 REST API。

有许多原因会导致重新生成存储帐户密钥。

-   可出于安全原因而定期重新生成密钥。

-   如果有人设法侵入应用程序并检索硬编码或存储在配置文件中的密钥，为其提供存储帐户的完整访问权限，则必须重新生成存储帐户密钥。

-   如果你的团队使用存储资源管理器应用程序来保留存储帐户密钥，则有团队成员离职时也需要重新生成密钥。有人离职后，应用程序仍将继续运行，使其能够访问你的存储帐户。这实际上是他们创建帐户级别共享访问签名的主要原因 – 可以改用帐户级别的 SAS，而不是将访问密钥存储在配置文件中。

####密钥重新生成计划

请勿无计划地重新生成正在使用的密钥。如果这样做，可能将切断对该存储帐户的所有访问权限，而这会造成严重中断。因此有两个密钥。一次只应重新生成一个密钥。

重新生成密钥之前，请务必列出依赖于存储帐户的所有应用程序，以及 Azure 中使用的所有其他服务。例如，如果 Azure 媒体服务依赖于存储帐户，则必须在重新生成密钥后将访问密钥与媒体服务重新同步。如果使用存储资源管理器之类的任何应用程序，也必须为这些应用程序提供新的密钥。请注意，如果 VM 的 VHD 文件存储在存储帐户中，它们将不会受到重新生成存储帐户密钥的影响。

可以在 Azure 门户中重新生成密钥。重新生成密钥之后，它们最多可能需要 10 分钟的时间跨存储服务进行同步。

准备就绪后，请遵循以下常规过程，其中详细描述了如何更改密钥。在本例中，假设你当前使用的是密钥 1，并想要更改所有项目以改用密钥 2。

1.  重新生成密钥 2 以确保密钥的安全。可以在 Azure 门户中执行此操作。

2.  在存储密钥所存储到的所有应用程序中，更改存储密钥以使用密钥 2 的新值。测试并发布应用程序。

3.  在所有应用程序和服务成功启动并运行之后，重新生成密钥 1。这可确保未明确提供新密钥的任何人都不再拥有存储帐户的访问权限。

如果你当前使用密钥 2，可以使用相同的过程，但反转密钥名称。

可以过几天后迁移，更改每个应用程序来使用新的密钥并进行发布。全部完成之后，应该返回并重新生成旧密钥，使其不再可用。

另一种做法是将存储帐户密钥作为机密放在 [Azure 密钥保管库](/home/features/key-vault/)中，并让应用程序从中检索该密钥。然后，重新生成密钥并更新 Azure 密钥保管库时，就不需要重新部署应用程序，因为它们将自动从 Azure 密钥保管库中选择新密钥。请注意，可以让应用程序每次在需要密钥时读取它，或者，可以将它缓存在内存中，如果使用密钥时失败，将再次从 Azure 密钥保管库中检索该密钥。

使用 Azure 密钥保管库还可以提高存储密钥的安全级别。如果使用此方法，永远都不需要将存储密钥硬编码于配置文件中，这样将删除某人不需特定权限即可访问密钥的途径。

使用 Azure 密钥保管库的另一个优点是，还可使用 Azure Active Directory 来控制对密钥的访问。这意味着，可以将访问权限授予少数必须从 Azure 密钥保管库检索密钥的应用程序，并了解其他应用程序在未特别授予它们权限的情况下无法访问密钥。

注意：建议同一时间在所有应用程序中只使用一个密钥。如果在某些地方使用密钥 1 并在其他地方使用密钥 2，将无法在没有部分应用程序失去访问的情况下轮转密钥。

####资源

-   [关于 Azure 存储帐户](/documentation/articles/storage-create-storage-account/#regenerate-storage-access-keys)

	此文提供存储帐户的概述，并介绍如何查看、复制和重新生成存储访问密钥。

-   [Azure 存储资源提供程序 REST API 参考](https://msdn.microsoft.com/zh-cn/library/mt163683.aspx)

	此文包含有关检索存储帐户密钥，以及使用 REST API 为 Azure 帐户重新生成存储帐户密钥的具体文章链接。注意：这适用于 Resource Manager 存储帐户。

-   [对存储帐户的操作](https://msdn.microsoft.com/zh-cn/library/ee460790.aspx)

    这篇存储服务管理器 REST API 参考文章包含有关使用 REST API 来检索和重新生成存储帐户密钥的具体文章链接。注意：此文适用于经典存储帐户。

-   [告别密钥管理 – 使用 Azure AD 管理对 Azure 存储空间数据的访问](http://www.dushyantgill.com/blog/2015/04/26/say-goodbye-to-key-management-manage-access-to-azure-storage-data-using-azure-ad/)

	本文介绍如何使用 Active Directory 来控制 Azure 密钥保管库中 Azure 存储密钥的访问。此外，说明如何使用 Azure 自动化作业每小时重新生成密钥。

##<a id="data-plane-security"></a>数据平面安全

数据平面安全是指用于保护存储在 Azure 存储的数据对象（Blob、队列、表和文件）的方法。我们已了解在传输数据期间加密数据和安全的方法，但该从何处着手来允许访问对象？

简单而言，有两种方法可用于控制对数据对象本身的访问。第一种是控制对存储帐户密钥的访问，第二种是使用共享访问签名来授予特定时间段对特定数据对象的访问权限。

要注意的一种例外情况是，可以通过相应设置保存 Blob 的容器的访问级别，来允许对 Blob 进行公共访问。如果将容器的访问权限设置为“Blob”或“容器”，则允许该容器中 Blob 的公共读取访问权限。这意味着 URL 指向该容器中 Blob 的任何人都可以在浏览器中打开它，而不需要使用共享访问签名或拥有存储帐户密钥。

###存储帐户密钥

存储帐户密钥是由 Azure 创建的 512 位字符串，配合存储帐户名称用于访问存储于存储帐户中的数据对象。

例如，可以读取 Blob、写入队列、创建表，以及修改文件。其中的许多操作可以通过 Azure 门户或使用多种存储资源管理器应用程序中的一种来执行。也可编写代码来使用 REST API 或某个存储客户端库来执行这些操作。

如[管理平面安全性](#management-plane-security)部分中所述，对经典存储帐户的存储密钥的访问权限可以通过提供对 Azure 订阅的完全访问权限来授予。使用 Azure Resource Manager 模型来访问存储帐户的存储密钥可以通过基于角色的访问控制 (RBAC) 来控制。

###如何使用共享访问签名和存储访问策略来委派对帐户中对象的访问权限

共享访问签名是一个字符串，包含可附加到 URI 的安全令牌，可用于委派存储对象的访问权限，以及指定访问的权限和日期/时间范围等限制。

可以授予对 Blob、容器、队列、文件和表的访问权限。使用表时，可以实际授予权限来访问表中某个范围的条目，方法是指定想要让用户有权访问的分区和行键范围。例如，如果已使用具有地理状态的分区键存储数据，则可为某人提供仅限加州数据的访问权限。

另举一例，可以为 Web 应用程序提供 SAS 令牌，使它能够将条目写入队列，并为辅助角色应用程序提供 SAS 令牌，以便从队列中获取和处理消息。或者，可以为某位客户提供 SAS 令牌，使他们能够将图片上传到 Blob 存储中的容器，并为 Web 应用程序提供权限来读取这些图片。在这两种情况下，都可实现关注点分离 – 每个应用程序只能获取执行其任务所需的访问权限。这是通过使用共享访问签名来实现的。

####使用共享访问签名的原因

为什么要使用 SAS 而不只是分发存储帐户密钥，哪一种方法更方便？ 分发存储帐户密钥就像是在存储王国中共享密钥。它将授予完全访问权限。其他人可以使用密钥，并将其整个音乐库上传到你的存储帐户。他们也许还能将你的文件替换为受病毒感染的版本，或窃取你的数据。不应草率地无限制分发对你存储帐户的访问权限。

如果使用共享访问签名，可以只为客户端提供有限时间内所需的权限。例如，如果有人将 Blob 上传到帐户，则可以授予他们刚好足够时间的写入权限来上传 Blob（当然，这取决于 Blob 的大小）。如果你改变想法，可以撤销该访问权限。

此外，可以指定将使用 SAS 所发出的请求限制为特定的 IP 地址或 Azure 外部的 IP 地址范围。还可要求使用特定协议（HTTPS 或 HTTP/HTTPS）来发出请求。这意味着，如果只想要允许 HTTPS 流量，可以将所需的协议设置为仅限 HTTPS，并阻止 HTTP 流量。

####共享访问签名的定义

共享访问签名是一组附加到指向资源的 URL 的查询参数

其中提供有关允许的访问权限的信息，以及准许该访问权限的时间长度。下面提供了一个示例：此 URI 将提供对 Blob 的读取权限，期限为五分钟。请注意，SAS 查询参数必须以 URL 编码，例如 %3A 表示冒号 (:)，%20 表示空格。

	http://mystorage.blob.core.chinacloudapi.cn/mycontainer/myblob.txt (URL to the blob)
	?sv=2015-04-05 (storage service version)
	&st=2015-12-10T22%3A18%3A26Z (start time, in UTC time and URL encoded)
	&se=2015-12-10T22%3A23%3A26Z (end time, in UTC time and URL encoded)
	&sr=b (resource is a blob)
	&sp=r (read access)
	&sip=168.1.5.60-168.1.5.70 (requests can only come from this range of IP addresses)
	&spr=https (only allow HTTPS requests)
	&sig=Z%2FRHIX5Xcg0Mq2rqI3OlWTjEg2tYkboXr1P9ZUXDtkk%3D (signature used for the authentication of the SAS)

####Azure 存储服务如何对共享访问签名进行身份验证

当存储服务收到请求时，它将获取输入查询参数，并使用与调用程序相同的方法来创建签名。然后比较这两个签名。如果它们相符，则存储服务可以检查存储服务版本以确保它有效、检查当前日期和时间是在指定时段内、确保请求的访问权限对应于发出的请求，等等。

以上述 URL 为例，如果 URL 指向文件而不是 Blob，此请求将会失败，因为它指定共享访问签名适用于 Blob。如果调用的 REST 命令是更新 Blob，则该命令将会失败，因为共享访问签名指定只准许读访问权限。

####共享访问签名的类型

-	服务级别 SAS 可用于访问存储帐户中的特定资源。其中的一些示例是检索容器中的 Blob 列表、下载 Blob、更新表中条目、将消息添加到队列，或将文件上传到文件共享。

-	帐户级别 SAS 可用于访问服务级别 SAS 可用的任何功能。此外，它可以为服务级别 SAS 不准许的资源提供选项，例如，能够创建容器、表、队列及文件共享。也可一次性指定对多个服务的访问权限。例如，可授权某人访问你存储帐户中的 Blob 和文件。

####创建 SAS URI

1.  可以按需创建临时 URI，每次都定义所有查询参数。

    这确实很有弹性，但如果有一组每次都相同的逻辑参数，则以使用存储访问策略为佳。

2.  可以针对整个容器、文件共享、表或队列创建存储访问策略。然后使用此策略作为所创建的 SAS URI 的基础。可以轻松撤销基于存储访问策略的权限。可以对每个容器、队列、表或文件共享最多定义 5 个策略。

    例如，如果要让许多人读取特定容器中的 Blob，则可以创建存储访问策略，表示“提供读访问权限”以及每次都相同的任何其他设置。然后，可以使用存储访问策略的设置，并指定过期日期/时间，来创建 SAS URI。这样做的优点是不需要每次指定所有查询参数。

####撤销

假设 SAS 已泄露，或者要基于公司安全或法规遵循要求更改 SAS。如何使用该 SAS 撤销对资源的访问权限？ 这取决于 SAS URI 的创建方式。

如果使用即席 URI，将有三个选项。可以颁发具有短期过期策略的 SAS 令牌，然后只需等待 SAS 过期。可以重命名或删除资源（假设令牌范围只限于单个对象）。可以更改存储帐户密钥。根据使用该存储帐户的服务数目，最后一个选项可能造成很大的影响，而且若无规划很可能出现意外行为。

如果使用派生自存储访问策略的 SAS，可以通过撤销存储访问策略来删除访问权限 – 只能在其过期后进行更改，或者完全删除它。这将立即生效，并使每个使用该存储访问策略创建的 SAS 失效。更新或删除存储访问策略可能将影响通过 SAS 访问该特定容器、文件共享、表或队列的用户，但如果将写入客户端，使得他们可在旧的 SAS 变成无效时请求一个新的 SAS，则这将可正常运行。

由于使用派生自存储访问策略的 SAS 可以立即撤销该 SAS，因此建议的最佳做法是尽可能使用存储访问策略。

####资源

有关使用共享访问签名和存储访问策略的详细信息和示例，请参阅以下文章：

-   下面是参考文章。

	-	[服务 SAS](https://msdn.microsoft.com/zh-cn/library/dn140256.aspx)

		此文提供有关使用服务级别 SAS 配合 Blob、队列、表范围和文件的示例。

	-	[构造服务 SAS](https://msdn.microsoft.com/zh-cn/library/dn140255.aspx)

	-	[构造帐户 SAS](https://msdn.microsoft.com/zh-cn/library/mt584140.aspx)

-   这些是使用 .NET 客户端库来创建共享访问签名和存储访问策略的教程。

    -	[使用共享访问签名 (SAS)](/documentation/articles/storage-dotnet-shared-access-signature-part-1/)
    -	[共享访问签名，第 2 部分：创建 SAS 并将 SAS 用于 Blob 服务](/documentation/articles/storage-dotnet-shared-access-signature-part-2/)

        此文包含 SAS 模型的说明、共享访问签名的示例，以及 SAS 用法最佳实践的建议。还介绍了如何撤销授予的权限。
	
-   按 IP 地址限制访问 (IP ACL)

    -	[什么是终结点访问控制列表 (ACL)？](/documentation/articles/virtual-networks-acl/)

    -	[构造服务 SAS](https://msdn.microsoft.com/zh-cn/library/azure/dn140255.aspx)

		这是适用于服务级别 SAS 的参考文章，其中包括执行 IP ACL 的示例。

	-	[构造帐户 SAS](https://msdn.microsoft.com/zh-cn/library/azure/mt584140.aspx)

    	这是适用于帐户级别 SAS 的参考文章，其中包括执行 IP ACL 的示例。

-   身份验证

	-    [Azure 存储服务的身份验证](https://msdn.microsoft.com/zh-cn/library/azure/dd179428.aspx)

-   共享访问签名入门教程

	-	[SAS 入门教程](https://github.com/Azure-Samples/storage-dotnet-sas-getting-started)

##<a id="encryption-in-transit"></a>传输中加密

###传输级加密 – 使用 HTTPS

为确保 Azure 存储数据安全，应采取另一个措施，即在客户端与 Azure 存储之间加密数据。第一条建议是始终使用 [HTTPS](https://en.wikipedia.org/wiki/HTTPS) 协议，这将确保通过公共 Internet 进行安全通信。

调用 REST API 或访问存储中的对象时，应该始终使用 HTTPS。此外，**共享访问签名**（可用于委派对 Azure 存储对象的访问权限）包含一个选项，用于指定在使用共享访问签名时只能使用 HTTPS 协议，以确保任何使用 SAS 令牌发出链接的人都使用正确的协议。

####资源

-   [在 Azure 应用服务中为应用启用 HTTPS](/documentation/articles/web-sites-configure-ssl-certificate/)

	此文说明如何为 Azure Web 应用启用 HTTPS。

###传输期间对 Azure 文件共享使用加密

使用 REST API 时，Azure 文件存储支持 HTTPS，但经常用作附加到 VM 的 SMB 文件共享。SMB 2.1 不支持加密，因此只允许在 Azure 中的相同区域内连接。但是，SMB 3.0 支持加密，并且可以配合 Windows Server 2012 R2、Windows 8、Windows 8.1 和 Windows 10 使用，允许跨区域访问，甚至桌面上的访问。

请注意，尽管 Azure 文件共享可以与 Unix 配合使用，但 Linux SMB 客户端尚不支持加密，因此只允许在 Azure 区域内访问。Linux 的加密支持已经在负责 SMB 功能的 Linux 开发人员的路线图上。当他们添加加密时，必须具有访问 Linux 上 Azure 文件共享的相同能力，就像对于 Windows 所做的一样。

####资源

-   [如何通过 Linux 使用 Azure 文件存储](/documentation/articles/storage-how-to-use-files-linux/)

    此文介绍如何在 Linux 系统上装载 Azure 文件共享，以及上传/下载文件。

-   [在 Windows 上开始使用 Azure 文件存储](/documentation/articles/storage-dotnet-how-to-use-files/)

	此文概述 Azure 文件共享，以及如何通过 PowerShell 与 .NET 来装载和使用这些文件共享。

-   [Azure 文件存储内部](https://azure.microsoft.com/blog/inside-azure-file-storage/)

    此文宣布了 Azure 文件存储的一般可用性，并提供有关 SMB 3.0 加密的技术详细信息。

###使用客户端加密来保护发送到存储的数据

另一个可帮助确保在客户端应用程序与存储之间传输时数据安全的选项是客户端加密。数据先经过加密，然后再传输到 Azure 存储。从 Azure 存储检索数据时，在客户端上收到数据之后会将其解密。即使数据在通过连接时已加密，但还是建议使用 HTTPS，因为它内置了数据完整性检查，有助于降低影响数据完整性的网络错误。

客户端加密也是一种可用于加密静态数据的方法，因为数据是以加密形式存储的。将在[静态加密](#encryption-at-rest)部分中详细介绍此功能。

##<a name="encryption-at-rest"></a>静态加密

有三项 Azure 功能可提供静态加密。Azure 磁盘加密可用于加密 IaaS 虚拟机中的 OS 和数据磁盘。其他两种（客户端加密和 SSE）都用于加密 Azure 存储中的数据。让我们了解其中每项功能，然后进行比较，以确定在哪种情况下使用哪项功能。

尽管可以使用客户端加密来加密传输中的数据（也将以其加密形式存储于存储中），你可能习惯在传输期间只使用 HTTPS，而且有一些方式可让数据在存储时自动加密。有两种做法可以执行此操作 - Azure 磁盘加密和 SSE。其中一种是用于直接加密 VM 使用的 OS 和数据磁盘上的数据，另一种用于加密写入 Azure Blob 存储的数据。

###存储服务加密 (SSE)

SSE 允许请求存储服务在将数据写入 Azure 存储时自动加密数据。读取 Azure 存储中的数据时，存储服务将在返回数据之前将其解密。这样，无需修改代码或将代码添加到任何应用程序，就可以保护数据。

这是应用到整个存储帐户的设置。可以通过更改设置的值来启用和禁用此功能。为此，可以使用 Azure 门户、PowerShell、Azure CLI、存储资源提供程序 REST API 或 .NET 存储客户端库。默认情况下，SSE 处于关闭状态。

目前，用于加密的密钥由 Azure 管理。我们最初将生成密钥，管理密钥的安全存储，并根据 Azure 内部策略的定义定期轮转密钥。将来，你将能够管理你自己的加密密钥，并可提供从 Azure 管理的密钥到客户管理的密钥的迁移路径。

此功能可用于使用 Resource Manager 部署模型创建的标准和高级存储帐户。SSE 仅适用于块 Blob、页 Blob 以及追加 Blob。其他类型的数据（包括表、队列和文件）将不会加密。

数据只有在启用 SSE 并将数据写入 Blob 存储时才会加密。启用或禁用 SSE 并不会影响现有的数据。换句话说，启用此加密时，不会返回并加密已存在的数据；也不会解密禁用 SSE 时已存在的数据。

如果想要通过经典存储帐户使用此功能，可以创建新的 Resource Manager 存储帐户，并使用 AzCopy 将数据复制到新帐户。

###客户端加密

在介绍传输中数据加密时，我们曾提到客户端加密。此功能可用于以编程方式加密客户端应用程序中的数据，然后通过连接发送数据以写入 Azure 存储，并在从 Azure 存储检索数据之后以编程方式解密数据。

这确实可提供传输中加密，但也会提供静态加密的功能。请注意，尽管将在传输过程中加密数据，仍建议使用 HTTPS 来充分利用内置的数据完整性检查，帮助降低影响数据完整性的网络错误。

例如，如果 Web 应用程序将存储 Blob 和检索 Blob，而你想让应用程序和数据尽可能保持安全，可使用此功能。在此情况下，请使用客户端加密。客户端与 Azure Blob 服务之间的流量包含加密的资源，并且没有人能够解释传输中的数据并将它重组到专用 Blob。

客户端加密内置于 Java 和 .NET 存储客户端库，这些库使用 Azure 密钥保管库 API 让实现变得很简单。加密和解密数据的程序将使用信封技术，并在每个存储对象中存储加密使用的元数据。例如，对于 Blob，会会其存储在 Blob 元数据中；对于队列，会它添加到每个队列消息。

对于加密本身，可以生成和管理自己的加密密钥。也可以使用 Azure 存储客户端库所生成的密钥，或者让 Azure 密钥保管库生成密钥。可以将加密密钥存储在本地密钥存储中，或将它们存储在 Azure 密钥保管库中。Azure 密钥保管库允许使用 Azure Active Directory 为特定用户授予 Azure 密钥保管库中密码的访问权限。这意味着，并非每个人都能读取 Azure 密钥保管库，以及检索用于进行客户端加密的密钥。

####资源

-   [在 Azure 存储中使用 Azure 密钥保管库加密和解密 Blob](/documentation/articles/storage-encrypt-decrypt-blobs-key-vault/)

    此文说明如何配合 Azure 密钥保管库使用客户端加密，包括如何使用 PowerShell 来创建 KEK 并将它存储在保管库中。

-   [Azure 存储的客户端加密和 Azure 密钥保管库](/documentation/articles/storage-client-side-encryption/)

    此文介绍客户端加密，并提供使用存储客户端库从四个存储服务加密和解密资源的示例。此外介绍了 Azure 密钥保管库。

###使用 Azure 磁盘加密来加密虚拟机所用的磁盘

Azure 磁盘加密是一项新功能，目前以预览版提供。此功能允许加密 IaaS 虚拟机使用的 OS 磁盘和数据磁盘。对于 Windows，驱动器是使用行业标准 BitLocker 加密技术加密的。对于 Linux，磁盘是使用 DM-Crypt 技术加密的。它与 Azure 密钥保管库集成，可用于控制和管理磁盘加密密钥。

Azure 磁盘加密解决方案支持以下三种客户加密方案：

-   在通过客户加密的 VHD 文件和客户提供的加密密钥（存储在 Azure 密钥保管库中）创建的新 IaaS VM 上启用加密。

-   在通过 Azure 应用商店创建的新 IaaS VM 上启用加密。

-   在 Azure 中已运行的现有 IaaS VM 上启用加密。

>[AZURE.NOTE] 对已在 Azure 中运行的 Linux VM，或从 Azure 应用商店中的映像新建的 Linux VM，当前不支持 OS 磁盘的加密。仅本地加密并上传到 Azure 的 VM 支持对 Linux VM 的 OS 卷加密。此限制仅适用于 OS 磁盘；支持对 Linux VM 的数据卷加密。

在 Azure 中启用时，该解决方案支持以下适用于公共预览版的 IaaS VM：

-   与 Azure 密钥保管库集成

-   标准 [A、D 系列 IaaS VM](/pricing/details/virtual-machines/)

-   在使用 [Azure 资源管理器](/documentation/articles/resource-group-overview/)模型创建的 IaaS VM 上启用加密


此功能可确保虚拟机磁盘上的所有数据在 Azure 存储中静态加密。

####资源

-   [适用于 Windows 和 Linux IaaS 虚拟机的 Azure 磁盘加密](https://gallery.technet.microsoft.com/Azure-Disk-Encryption-for-a0018eb0)

    此文介绍 Azure 磁盘加密预览版，并提供下载白皮书的链接。

###Azure 磁盘加密、SSE 和客户端加密的比较

####IaaS VM 及其 VHD 文件

对于 IaaS VM 使用的磁盘，建议使用 Azure 磁盘加密。可以启用 SSE 来加密在 Azure 存储中用于备份这些磁盘的 VHD 文件，但它只将加密新写入的数据。这意味着，如果创建 VM，然后对保存 VHD 文件的存储帐户启用 SSE，则只将加密更改，而不会加密原始 VHD 文件。

如果使用 Azure 应用商店创中的映像建 VM，Azure 将在 Azure 存储中对存储帐户执行映像的[浅层复制](https://en.wikipedia.org/wiki/Object_copying)，并且即使已启用 SSE，也不会将其加密。创建 VM 并启动更新映像后，SSE 将开始加密数据。因此，如果希望完全加密，最好对通过 Azure 应用商店中的映像创建的 VM 使用 Azure 磁盘加密。

如果通过本地将预先加密的 VM 带入 Azure 中，就能将加密密钥上传到 Azure 密钥保管库，并继续针对使用本地的 VM 使用加密。启用 Azure 磁盘加密即可处理此方案。

如果本地存在未加密 VHD，可将其作为自定义映像上传到库并从中预配 VM。如果使用 Resource Manager 模板执行此操作，可以要求它在启动 VM 时打开 Azure 磁盘加密。

添加数据磁盘并将其装载到 VM 时，可在该数据磁盘上开启 Azure 磁盘加密。它先在本地加密该数据磁盘，然后服务管理层将会对存储进行延迟写入，如此即可加密存储内容。

#### 客户端加密
客户端加密是加密数据的最安全方法，因为它将在传输前加密数据，并加密静态数据。但是，它需要向使用存储的应用程序添加代码，这可能不是理想行为。在这些情况下，可以针对传输中的数据使用 HTTPs，并使用 SSE 来加密静态数据。

通过客户端加密，可以加密表中条目、消息队列和 Blob。使用 SSE 只可以加密 Blob。如果需要将表和队列数据加密，应该使用客户端加密。

客户端加密完全通过应用程序来管理。这是最安全的方式，但要求以编程方式更改应用程序，并将密钥管理程序放在正确的位置。如果在传输期间想要额外的安全性，并且想要将存储的数据加密，可以使用此方法。

客户端加密将在客户端上生成更多负载，必须在缩放性计划中考虑到这一点，特别是要加密并传输大量数据时。

####存储服务加密 (SSE)

SSE 由 Azure 存储管理。使用 SSE 不是针对传输中数据安全性提供的，但它将在数据写入 Azure 存储时加密数据。使用此功能时不会对性能造成任何影响。

使用 SSE 只可以加密块 Blob、追加 Blob 和页 Blob。如果需要加密表数据或队列数据，应该考虑使用客户端加密。

如果使用 VHD 文件的存档或库作为创建新虚拟机的基础，可以创建新的存储帐户、启用 SSE，然后将 VHD 文件上传到该帐户。这些 VHD 文件将由 Azure 存储加密。

如果已针对 VM 中的磁盘启用 Azure 磁盘加密，并对保存 VHD 文件的存储帐户启用了 SSE，则它将正常运行；它会导致任何新写入的数据加密两次。

##<a id="storage-analytics"></a>存储分析

###使用存储分析来监视授权类型

对于每个存储帐户，可以启用 Azure 存储分析来执行日志记录和存储指标数据。若要检查存储帐户的性能指标，或者由于发生性能问题而需要排查存储帐户问题，这是一个绝佳工具。

可以在存储分析记录中看到的另一部分数据是其他人访问存储时使用的身份验证方法。例如，使用 Blob 存储，可以看到他们使用的是共享访问签名还是存储帐户密钥，或者访问的 Blob 是否为公共的。

如果要严密监视存储的访问，这可能非常有用。例如，在 Blob 存储中，可以将所有容器设置为专用，并通过应用程序实现 SAS 服务的使用。然后可以定期检查日志，了解是否使用存储帐户密钥访问了 Blob（这可能表示安全性遭到破坏），或者是否公开了不应公开的 Blob。

####日志的外观

通过 Azure 门户启用存储帐户指标和日志记录后，分析数据会开始快速累积。每个服务的日志记录与指标是分开的；只有在该存储帐户中有活动时才将写入日志，而根据设置指标的方式，将会每分钟、每小时或每天记录指标。

日志将存储在存储帐户中名为 $logs 的容器的块 Blob 中。启用存储分析后，将自动创建此容器。创建此容器之后，无法将它删除，但可以删除其内容。

在 $logs 容器下面，每个服务都有一个文件夹，另外还有对应于年/月/日/小时的子文件夹。在小时下面，日志只带有编号。下面是目录结构的外观：

![日志文件视图](./media/storage-security-guide/image1.png)

将记录对 Azure 存储的每个请求。下面是日志文件的快照，其中显示了前几个字段。

![日志文件快照](./media/storage-security-guide/image2.png)

如你所见，可以使用日志来跟踪对存储帐户的各种调用。

####所有这些字段的用途是什么？

在下面列出的资源中，有一篇文章提供了日志中许多字段的列表及其用途。下面是依次列出的字段列表：

![日志文件中字段的快照](./media/storage-security-guide/image3.png)

我们希望了解 GetBlob 的条目及其身份验证方法，因此需要查找操作类型“Get-Blob”的条目，并检查请求状态（第 4 列）和授权类型（第 8 列）。<sup></sup><sup></sup>

例如，在上述列表的前几列中，请求状态为“Success”且授权类型为“authenticated”。这意味着已使用存储帐户密钥验证请求。

####如何对我的 Blob 进行身份验证？

下面提供了我们感兴趣的三种用例。

1.  Blob 是公共的，可使用 URL 来访问（无需共享访问签名）。在本例中，请求状态为“AnonymousSuccess”且授权类型为“anonymous”。

    1\.0;2015-11-17T02:01:29.0488963Z;GetBlob;**AnonymousSuccess**;200;124;37;**anonymous**;;mystorage…

2.  Blob 是专用的且与共享访问签名配合使用。在本例中，请求状态为“SASSuccess”且授权类型为“sas”。

    1\.0;2015-11-16T18:30:05.6556115Z;GetBlob;**SASSuccess**;200;416;64;**sas**;;mystorage…

3.  Blob 是专用的，可使用存储密钥来访问。在本例中，请求状态为“**Success**”且授权类型为“**authenticated**”。

    1\.0;2015-11-16T18:32:24.3174537Z;GetBlob;**Success**;206;59;22;**authenticated**;mystorage…

可以使用 Microsoft Message Analyzer 来查看和分析这些日志。它包含搜索和筛选功能。例如，可能需要搜索 GetBlob 的实例，以查看其使用方式是否符合预期，例如为了确保其他人不会不恰当地访问你的存储帐户。

####资源

-   [存储分析](/documentation/articles/storage-analytics/)

	此文概述存储分析及其启用方法。

-   [存储分析日志格式](https://msdn.microsoft.com/zh-cn/library/azure/hh343259.aspx)

	此文介绍存储分析日志格式，并详细说明其中的可用字段，包括身份验证类型（指示请求使用的身份验证类型）。

-   [在 Azure 门户中监视存储帐户](/documentation/articles/storage-monitor-storage-account/)

	此文说明如何配置和监视存储帐户的指标与日志记录。

-   [使用 Azure 存储度量值和日志记录、AzCopy 及 Message Analyzer 进行点对点故障排除](/documentation/articles/storage-e2e-troubleshooting/)

	此文介绍如何使用存储分析进行故障排除，并说明如何使用 Microsoft Message Analyzer。

-   [Microsoft Message Analyzer 操作指南](https://technet.microsoft.com/zh-cn/library/jj649776.aspx)

	此文提供 Microsoft Message Analyzer 的参考信息，包括教程、快速入门和功能摘要的链接。

##<a id="Cross-Origin-Resource-Sharing-CORS"></a>跨源资源共享 (CORS)

###跨域访问资源

在某一个域中运行的 Web 浏览器对来自不同域的资源发出 HTTP 请求称为跨域 HTTP 请求。例如，contoso.com 中的 HTML 页面将对托管在 fabrikam.blob.core.chinacloudapi.cn 上的 jpeg 发出请求。出于安全原因，浏览器将限制从脚本（例如 JavaScript）中初始化的跨域 HTTP 请求。这意味着，当 contoso.com 的网页上有一些 JavaScript 代码请求 fabrikam.blob.core.chinacloudapi.cn 上的该 jpeg 时，浏览器将不允许该请求。

必须对 Azure 存储执行哪些操作？ 如果正在使用名为 Fabrikam 的存储帐户在 Blob 存储中存储 JSON 或 XML 数据文件等静态资产，则资产的域将是 fabrikam.blob.core.chinacloudapi.cn，contoso.com web 应用程序无法使用 JavaScript 来访问它们，因为域不相同。如果尝试调用某个 Azure 存储服务（例如表存储）以返回要通过 JavaScript 客户端处理的 JSON 数据，则这一点同样适用。

####可能的解决方案

解决此问题的方法之一是将类似于“storage.contoso.com”的自定义域分配给 fabrikam.blob.core.chinacloudapi.cn。问题在于，只能将该自定义域分配给一个存储帐户。如果资产存储在多个存储帐户中该怎么办？

解决此问题的另一种方法是让 Web 应用程序充当存储调用的代理。这意味着，如果要将文件上传到 Blob 存储，Web 应用程序可以在本地写入它，然后将它复制到 Blob 存储，或者将它全部读入内存，然后将它写入 Blob 存储。或者，可以编写专门的 Web 应用程序（例如 Web API），以在本地上传文件并将它们写入 Blob 存储。无论如何，都必须在确定可缩放性需求时考虑该功能。

####CORS 有何用途？

Azure 存储允许启用 CORS – 跨域资源共享。对于每个存储帐户，可以指定可访问该存储帐户中的资源的域。例如，在上述用例中，我们可以在 fabrikam.blob.core.chinacloudapi.cn 存储帐户中启用 CORS，并将它设置为允许访问 contoso.com。然后，Web 应用程序 contoso.com 就能直接访问 fabrikam.blob.core.chinacloudapi.cn 中的资源。

要注意的一点是，CORS 允许访问，但不提供所有对存储资源的非公共访问所需的身份验证。这意味着，如果它们是公共的，就只能访问 Blob，或者可以包含共享访问签名来提供相应的权限。表、队列和文件没有公共访问权限并需要 SAS。

默认情况下，对所有服务禁用了 CORS。可以使用 REST API 或存储客户端库调用某个方法来设置服务策略，以启用 CORS。执行该操作时，将在 XML 中包含 CORS 规则。以下示例将针对存储帐户的 Blob 服务使用“设置服务属性”操作来设置 CORS 规则。可以使用存储客户端库或 REST API 针对 Azure 存储执行该操作。

	<Cors>    
	    <CorsRule>
	        <AllowedOrigins>http://www.contoso.com, http://www.fabrikam.com</AllowedOrigins>
	        <AllowedMethods>PUT,GET</AllowedMethods>
	        <AllowedHeaders>x-ms-meta-data*,x-ms-meta-target*,x-ms-meta-abc</AllowedHeaders>
	        <ExposedHeaders>x-ms-meta-*</ExposedHeaders>
	        <MaxAgeInSeconds>200</MaxAgeInSeconds>
	    </CorsRule>
	<Cors>

下面是每一行的含义：

-   **AllowedOrigins** 指出哪些不匹配的域可以从存储服务请求并接收数据。这意味着，contoso.com 和 fabrikam.com 可以针对特定的存储帐户向 Blob 存储请求数据。也可以将此项设置为通配符 (*)，以允许所有域访问请求。

-   **AllowedMethods** 这是发出请求时可以使用的方法（HTTP 请求谓词）列表。在本示例中，只允许 PUT 和 GET。可以将此项设置为通配符 (*)，以允许使用所有方法。

-   **AllowedHeaders** 这是在发出请求时原始域可以指定的请求标头。在本示例中，允许所有以 x-ms-meta-data、x-ms-meta-target 和 x-ms-meta-abc 开头的元数据标头。通配符 (*) 表示允许任何以指定前缀开头的标头。

-   **ExposedHeaders** 告知浏览器应向请求颁发者公开的响应标头。在本示例中，将公开任何以“x-ms-meta-”开头的标头。

-   **MaxAgeInSeconds** 这是浏览器将缓存预检 OPTIONS 请求的最长时间。（有关预检请求的详细信息，请检查下面的第一篇文章。）

####资源

有关 CORS 及其启用方法的详细信息，请参阅以下资源。

-   [Azure.cn 上对 Azure 存储服务的跨域资源共享 (CORS) 支持](/documentation/articles/storage-cors-support/)

	此文概述 CORS，以及如何为不同的存储服务设置规则。

-   [MSDN 上对 Azure 存储服务的跨域资源共享 (CORS) 支持](https://msdn.microsoft.com/zh-cn/library/azure/dn535601.aspx)

	这是有关对 Azure 存储服务的 CORS 支持的参考文档。其中提供了适用于每个存储服务的文章链接，并提供示例演示，解释 CORS 文件中的每个元素。

-   [Azure 存储：CORS 简介](http://blogs.msdn.com/b/windowsazurestorage/archive/2014/02/03/windows-azure-storage-introducing-cors.aspx)

	这是宣布推出 CORS 并演示其用法的第一篇博客文章的链接。

##有关 Azure 存储安全性的常见问题

1.  **如果无法使用 HTTPS 协议，应如何验证传入或传出 Azure 存储的 Blob 的完整性？**

	如果出于任何原因需要使用 HTTP 而不是 HTTPS，并且正在使用块 Blob，可以使用 MD5 检查来帮助验证传输中 Blob 的完整性。这将有助于防止网络/传输层错误，但不一定可帮助防止中间攻击。

	如果可以使用提供传输级安全的 HTTPS，则使用 MD5 检查就很多余且不必要。
	
	有关详细信息，请查看 [Azure Blob MD5 Overview](http://blogs.msdn.com/b/windowsazurestorage/archive/2011/02/18/windows-azure-blob-md5-overview.aspx)（Azure Blob MD5 概述）。

2.  **美国政府实施的 FIPS 合规性要求是怎样的？**

	美国联邦信息处理标准 (FIPS) 定义了美国联邦政府计算机系统批准使用的加密算法，以保护敏感数据。如果在 Windows 服务器或桌面上启用 FIPS 模式，将告知 OS 仅应使用经 FIPS 验证的加密算法。如果某个应用程序使用不合规的算法，即表示该应用程序违规。使用 .NET Framework 4.5.2 或更高版本，应用程序可在计算机处于 FIPS 模式时自动切换加密算法来使用符合 FIPS 的算法。

	Microsoft 允许每个客户决定是否启用 FIPS 模式。我们相信，客户没有充分的理由违反政府法规，不按默认启用 FIPS 模式。

	**资源**

-	[为什么我们不再建议使用“FIPS 模式”](http://blogs.technet.com/b/secguide/archive/2014/04/07/why-we-re-not-recommending-fips-mode-anymore.aspx)

	此博客文章提供 FIPS 概述，并说明他们为什么默认不启用 FIPS 模式。

-   [FIPS 140 验证](https://technet.microsoft.com/zh-cn/library/cc750357.aspx)

	此文提供有关 Microsoft 产品和加密模块如何遵守美国联邦政府 FIPS 标准的信息。

-   [“系统加密：使用符合 FIPS 的算法进行加密、哈希处理和签名”安全设置在 Windows XP 和更高 Windows 版本中的效果](https://support.microsoft.com/zh-cn/kb/811833)

	此文介绍如何在较旧的 Windows 计算机中使用 FIPS 模式。

<!---HONumber=Mooncake_0103_2017-->
<!--Update_Description:add disk encryption related content-->