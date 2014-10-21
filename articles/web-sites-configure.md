<properties linkid="manage-services-how-to-configure-websites" urlDisplayName="How to configure" pageTitle="How to configure web sites - Azure service management" metaKeywords="Azure websites, configuring Azure websites, Azure SQL database, Azure MySQL" description="Learn how to configure web sites in Azure, including how to configure a web site to use a SQL Database or MySQL database." metaCanonical="" services="web-sites" documentationCenter="" title="How to Configure Web Sites" authors="timamm" solutions="" manager="" editor="mollybos" />

# 如何配置网站

在 Azure 管理门户中，你可以更改用于网站的配置选项，而且可以将网站链接到其他 Azure 资源。例如，你可以将网站链接到 SQL Database 来提供其他功能。你也可以配置网站来使用全新或现有的 MySQL 数据库。

## 目录

-   [如何：更改网站的配置选项][如何：更改网站的配置选项]
-   [如何：将网站配置为使用 SQL Database][如何：将网站配置为使用 SQL Database]
-   [如何：将网站配置为使用 MySQL 数据库][如何：将网站配置为使用 MySQL 数据库]
-   [如何：配置自定义域名][如何：配置自定义域名]
-   [如何：将网站配置为使用 SSL][如何：将网站配置为使用 SSL]
-   [后续步骤][后续步骤]

## <a name="howtochangeconfig"></a>如何：更改网站的配置选项

<!-- HOW TO: CHANGE CONFIGURATION OPTIONS FOR A WEB SITE -->

执行下列步骤可更改网站的配置选项。

1.  在管理门户中，打开网站的管理页。
2.  单击“配置”选项卡，以打开“配置”管理页。
3.  根据需要为网站设置以下配置选项：
    -   **常规**
        -   **.NET Framework 版本** - 如果你的 Web 应用程序使用 .NET Framework，请设置该应用程序要求的 Framework 的版本。
        -   **PHP 版本** - 如果你的 Web 应用程序使用 PHP，请设置该 Web 应用程序要求的 PHP 的版本。
        -   **Java 版本** - 选择显示的 Java 版本以对 Web 应用程序启用此版本，或者选择**“关闭”**以禁用 Java。如果为你的 Web 应用程序启用 Java，利用“Web 容器”选项，你可以选择 Tomcat 版本或是 Jetty 版本。

            **注意**：出于技术原因，为你的网站启用 Java 会禁用 .NET、PHP 和 Python 选项。

        -   **Python 版本** - 显示 Python 版本（不可配置）。
        -   **托管管道模式** - 提供两个选项：“经典”和“集成”；“集成”是默认选项。只有当你有专门在旧版 IIS 上运行的旧网站时，才应该使用“经典”选项。
        -   **平台** - 对于“标准”模式下的网站，可以选择希望你的应用程序是在 32 位还是 64 位环境中运行。处于“免费”和“共享”模式下的网站始终在 32 位环境下运行。
        -   **Web 套接字** - 选择**“打开”**可允许网站使用实时请求模式应用程序（如聊天）。
        -   **始终打开** - 默认情况下，网站如果已处于空闲状态相当一段时间，则是未加载的状态。这样可以让系统节省资源。如果一个网站需要始终处于加载状态，你可以在标准模式下为该网站启用“始终打开”设置。如果禁用“始终打开”，持续的 Web 作业运行起来可能会不可靠，因此如果该网站上存在持续运行的 Web 作业，则应启用“始终打开”。
        -   **在 Visual Studio Online 中在线编辑** - 选择“打开”将启用通过 Visual Studio Online 进行实时代码编辑。在你保存此配置更改后，“仪表板”选项卡的“速览”部分将显示称作“在 Visual Studio Online 中在线编辑”的链接。单击该链接可以直接在线编辑你的网站。如果你需要进行身份验证，可以使用你的基本部署凭据。

            **注意**：如果你启用了“从源代码管理部署”，则部署可能会覆盖你在 Visual Studio Online 编辑器中进行的更改。因此，如果你想要直接使用 Visual Studio Online 编辑网站内容，最好不要使用“从源代码管理部署”功能。

    -   **证书** - 只是在“标准”模式下，你可以单击“上载”以上载自定义域的 SSL 证书。你上载的证书将在这里列出。支持通配符（“星号”）证书（带有星号的证书）。在你上载某一证书后，可以将其分配给你的订阅和区域中的任何网站。星号证书必须只上载一次，但可用于该证书有效的域内的任何站点。只有在对于给定证书在任何网站内都没有绑定处于活动状态的情况下，才能删除该证书。
        **注意：** 自定义域只在共享模式和标准模式下可用，而对自定义域的 SSL 支持只在标准模式下可用。有关为 Azure 上的自定义域配置 SSL 的信息，请参阅[为 Azure 网站启用 HTTPS][为 Azure 网站启用 HTTPS]。
    -   **域名** - 可在此处查看或添加网站的其他域名。可通过单击“管理域”添加自定义域。仅在“共享”和“标准”模式下提供自定义域。你可以在“缩放”管理页上更改网站模式。自定义域在“免费”模式下不可用。有关如何配置自定义域的详细信息，请参阅[为 Azure 网站配置自定义域名][为 Azure 网站配置自定义域名]。
    -   **SSL 绑定** - 仅在“标准”模式下提供与自定义域的 SSL 绑定。为特定域选择 SSL 模式（“SNI”、“IP”或“非 SSL”）。如果你选择 SNI 或 IP，则可以从你已上载的证书为域指定证书。有关为 Azure 上的自定义域配置 SSL 的信息，请参阅[为 Azure 网站启用 HTTPS][为 Azure 网站启用 HTTPS]。有关 SNI 的详细信息，请参阅[服务器名称指示][服务器名称指示]。
    -   **部署**- 使用以下设置可配置部署。
        -   **Git URL** - 如果你已在 Azure 网站上创建了 Git 存储库，则这就是其 URL - 你要向其推送内容的位置。
        -   **部署触发器 URL** - 可以在 GitHub、CodePlex、Bitbucket 或其他存储库上设置此 URL，将提交操作推送到存储库时就能触发部署。
        -   **要部署的分支** - 利用此选项，你可以在要向分支推送内容时指定要部署的分支。
    -   **应用程序诊断**- 设置选项以便从已使用跟踪检测了其代码的 Web 应用程序收集诊断跟踪。用于应用程序诊断的日志记录选项包括：
        -   **应用程序日志记录（文件系统）** - 选择“打开”将使应用程序日志写入网站的文件系统。启用后，文件系统日志记录将持续 12 小时期间。你可以从网站的 FTP 共享访问这些日志。可在“仪表板”上找到指向 FTP 共享的链接。在**“速览”**下，选择**“FTP 诊断日志”**或**“FTPS 诊断日志”**。
        -   **应用程序日志记录(存储)** - 选择“打开”以将应用程序日志写入 Azure 存储帐户。记录到存储帐户没有时间限制并且在你禁用前保持启用状态。默认情况下，日志存储在名为 WAWSAppLogTable 的表中。
        -   **日志记录级别** - 启用日志记录时，此选项指定要记录的信息量（“错误”、“警告”、“信息”或“详细”）。
        -   **诊断存储** - 单击“管理连接”打开带以下选项的“管理诊断存储”对话框，以便将日志保存到你的 Azure 存储帐户：
            -   **存储帐户名称** - 选择要将日志保存到其中的存储帐户。
            -   **存储访问密钥** - 显示所选存储帐户的密钥。如果你已经为存储帐户重新生成了密钥，则在此处手动键入新密钥，或者使用“同步”按钮之一。这些同步按钮仅在当前登录用户具有选定存储帐户的访问权限的情况下可用。
            -   **同步主密钥** - 检索 Azure 存储帐户的最新主密钥。
            -   **同步辅助密钥** - 检索 Azure 存储帐户的最新辅助密钥。
                **注意：**有关 Azure 存储访问密钥的详细信息，请参阅[如何：查看、复制和重新生成存储访问密钥][如何：查看、复制和重新生成存储访问密钥]。
    -   **网站诊断** - 设置用于收集网站诊断信息 的选项，包括：
        -   **Web 服务器日志记录** - 指定是否启用网站的 Web 服务器日志记录。Web 服务器日志以 W3C 扩展日志文件格式保存。可将日志保存到 Azure 存储空间或文件系统。
            **提示**：文件系统中日志存储的最大大小为 100 MB。如果你需要保留更多的历史记录，请使用 Azure 存储空间，它的存储容量更大。
            -   若要将 Web 服务器日志保存到 Azure 存储帐户，请选择“存储”，然后选择“管理存储”。在“管理用于站点诊断的存储”对话框中，使用“存储帐户”选项为将要存储这些日志的容器选择 Azure 存储帐户。使用“Azure Blob 容器”选项来选择将用来存储日志的容器，或选择“新建 Blob 容器”以启用“Blob 名称”框，在该框中可以指定新容器的名称。
                **注意**：如果你还没有存储帐户，请转到 Azure 门户的“存储”部分，在那里你可以单击“新建”来创建一个帐户。
            -   如果你选择“文件系统”，日志会保存到仪表板管理页面上“FTP 诊断日志”下列出的 FTP 站点。启用文件系统还将启用“配额”框，在该框中，你可为日志文件设置最大磁盘空间量。最小为 25MB，最大为 100MB。默认值为 35MB.。当配额用完时，最旧的文件会被最新的文件陆续覆盖。
            -   默认情况下，Azure 存储帐户中的 Web 服务器日志永远不会被删除。若要指定在其之后将自动删除日志的时段，请选择“设置保持”并且在“保留期”框中输入保留日志的天数。你也可以为文件系统存储使用“设置保持”选项。
        -   **“详细错误消息”**- 指定是否记录网站的详细错误消息。 如果启用此选项，则将详细的错误消息作为 .htm 文件保存到“仪表板”管理页上的“FTP 诊断日志”下方列出的 FTP 站点。 在连接到指定的 FTP 站之后，请导航到 /LogFiles/DetailedErrors/ 以检索包含详细错误信息的 .htm 文件。
        -   **“失败的请求跟踪”**- 指定是否启用失败的请求跟踪。如果启用， 失败的请求跟踪输出结果会写入到 XML 文件，并保存到仪表板管理页面上“FTP 诊断日志”下列出的 FTP 站点。 连接到指定的 FTP 站点后，请导航到 /LogFiles/W3SVC\#\#\#\#\#\#\#\#\#（其中 \#\#\#\#\#\#\#\#\# 代表网站的唯一标识符） 以检索包含失败的请求跟踪输出结果的 XML 文件。
            **重要**
            /LogFiles/W3SVC\#\#\#\#\#\#\#\#\#/ 文件夹包含一个 XSL 文件和一个或多个 XML 文件。一定要将该 XSL 文件下载到 XML 文件所在的同一目录，因为 该 XSL 文件提供在 Internet Explorer 中查看 XML 文件时对这些文件的内容进行格式设置和筛选的功能。
        -   **远程调试** - 将此选项设置为**“开”**，以在选择的 Visual Studio 2012 或 Visual Studio 2013 中启用远程调试。启用后，可以使用 Visual Studio 中的远程调试器来直接连接到你的 Azure 网站。
             **注意**：远程调试只允许持续 48 小时，并且对于超过 20 个字符的站点名称或用户名无效。
    -   **监视** - 对于“标准”模式下的网站，测试 HTTP 或 HTTPS 终结点的可用性。你可以从最多三个分布式地理位置测试终结点。如果 HTTP 响应代码大于或等于 400 或响应时间超过 30 秒，则监视测试失败。如果从所有指定的位置监视测试均成功，则终结点被视为可用。
    -   **开发人员分析** - 选择“外接程序”，以从列表中选择一个分析外接程序或转到 Azure 商店选择一个。选择“自定义”可从列表中选择 New Relic 之类的分析提供程序。如果你使用某一自定义提供程序，则必须在“提供程序密钥”框中输入许可证密钥。
         **注意**：有关 New Relic 与 Azure 网站配合使用的详细信息，请参阅 [Azure 网站上的 New Relic 应用程序性能管理][Azure 网站上的 New Relic 应用程序性能管理]。
    -   **应用程序设置** - 指定 Web 应用程序在启动时将加载的名称/值对。对于 .NET 网站，这些设置将 在运行时注入到 .NET 配置 AppSettings 中，替代现有的设置。对于 PHP 和 Node 网站，这些设置将会 作为运行时环境变量使用。
    -   **连接字符串** - 为链接的资源查看连接字符串。对于 .NET 网站，这些连接字符串将在运行时注入到 .NET 配置 connectionStrings 设置中，并且将重写其中的键等于链接的数据库名称的所有现有条目。对于 PHP 和 Node 网站，这些设置将作为运行时环境变量提供，并且用连接类型作为前缀。下面列出了环境变量前缀：
        -   SQL Server：SQLCONNSTR\_
        -   MySQL：MYSQLCONNSTR\_
        -   SQL Database：SQLAZURECONNSTR\_
        -   自定义：CUSTOMCONNSTR\_

        例如，如果 MySql 连接字符串被命名为 connectionstring1，则会通过环境变量 `MYSQLCONNSTR_connectionString1` 来访问它。
        **注意**：连接字符串在将数据库资源链接到网站时创建，在配置管理页上 查看时是只读的。
    -   **默认文档** - 如果网站默认文档尚未在此列表中，可在此添加。网站的默认 文档是当用户导航到网站且未指定网站上的特定页时显示的网页。因此，如果 网站为 http://contoso.com 且将默认文档设置为 default.htm，当用户将浏览器指向 http://www.contoso.com 时，将被 路由到 http://www.contoso.com/default.htm。如果你的网站包含列表中的多个文件，请通过更改列表中的文件顺序 确保你网站的默认文档位于列表顶部。
    -   **处理程序映射** - 添加自定义脚本处理器，以便处理针对特定文件扩展名的请求。在“扩展名”框中指定要处理的文件扩展名（例如 \*.php 或 handler.fcgi）。对与此模式匹配的文件的请求将由在“脚本处理器路径”框中指定的脚本处理器进行处理。脚本处理器需要绝对路径（可以使用路径 D:\\home\\site\\wwwroot 表示你的网站的根目录）。可在**“其他参数(可选)”**框中指定脚本处理器的可选命令行参数。
    -   **虚拟应用程序和目录** – 若要配置与你的网站关联的虚拟应用程序和目录，请指定每个虚拟目录及其对应于网站根目录的物理路径。或者，你也可以选中“应用程序”复选框，在网站配置中将一个虚拟目录标记为一个应用程序。

4.  在“配置”管理页的底部单击“保存”，以保存配置更改。

<!-- HOW TO: CONFIGURE A WEB SITE TO USE A SQL DATABASE -->

## <a name="howtoconfigSQL"></a>如何：将网站配置为使用 SQL Database

按照下列步骤操作可将网站链接到 SQL Database：

1.  在[管理门户][管理门户]中，选择“网站”,以显示当前登录的帐户创建的网站的列表。

2.  从网站列表中选择一个网站，以打开该网站的“管理”页。

3.  单击“链接的资源”选项卡，“链接的资源”页上会显示一条消息：“你没有链接的资源”。

4.  单击“链接资源”，以打开“链接资源”向导。

5.  单击“创建新资源”，以显示可连接到网站的资源类型的列表。

6.  单击“SQL 数据库”，以显示“链接数据库”向导。

7.  完成“链接数据库”向导的第 3 和第 4 页上必填的字段，然后单击第 4 页上的“完成”复选标记。

Azure 将使用指定的参数创建一个 SQL 数据库并将该数据库链接到网站。

<!-- HOW TO: CONFIGURE A WEB SITE TO USE A MYSQL DATABASE -->

## <a name="howtoconfigMySQL"></a>如何：将网站配置为使用 MySQL 数据库

若要配置网站使用 MySQL 数据库，请遵循与使用 SQL 数据库相同的步骤，但是在“链接资源”向导中，请选择“MySQL 数据库”，而不是“SQL 数据库”。

或者，你也可以选择使用“自定义创建”选项来创建网站。在“数据库”下拉列表中，选择“新建 MySQL 数据库”或“使用现有的 MySQL 数据库”。

## <a name="howtodomain"></a>如何：配置自定义域名

有关配置网站以使用自定义域名的信息，请参阅[为 Azure 网站配置自定义域名][1]。

## <a name="howtoconfigSSL"></a>如何：将网站配置为使用 SSL

有关为 Azure 上的自定义域配置 SSL 的信息，请参阅[为 Azure 网站启用 HTTPS][2]。

## <a name="next"></a>后续步骤

-   [如何缩放网站][如何缩放网站]

-   [如何监视网站][如何监视网站]

  [如何：更改网站的配置选项]: #howtochangeconfig
  [如何：将网站配置为使用 SQL Database]: #howtoconfigSQL
  [如何：将网站配置为使用 MySQL 数据库]: #howtoconfigMySQL
  [如何：配置自定义域名]: #howtodomain
  [如何：将网站配置为使用 SSL]: #howtoconfigSSL
  [后续步骤]: #next
  [为 Azure 网站启用 HTTPS]: http://www.windowsazure.com/en-us/documentation/articles/web-sites-configure-ssl-certificate/
  [为 Azure 网站配置自定义域名]: http://www.windowsazure.com/en-us/documentation/articles/web-sites-custom-domain-name/
  [服务器名称指示]: http://en.wikipedia.org/wiki/Server_Name_Indication
  [如何：查看、复制和重新生成存储访问密钥]: http://www.windowsazure.com/en-us/documentation/articles/storage-manage-storage-account/#regeneratestoragekeys
  [Azure 网站上的 New Relic 应用程序性能管理]: http://www.windowsazure.com/en-us/documentation/articles/store-new-relic-web-sites-dotnet-application-performance-management/
  [管理门户]: http://manage.windowsazure.cn
  [1]: http://www.windowsazure.cn/zh-cn/documentation/articles/web-sites-custom-domain-name/
  [2]: http://www.windowsazure.cn/zh-cn/documentation/articles/web-sites-configure-ssl-certificate/
  [如何缩放网站]: http://www.windowsazure.cn/zh-cn/documentation/articles/web-sites-scale/
  [如何监视网站]: http://www.windowsazure.cn/zh-cn/documentation/articles/web-sites-monitor/
