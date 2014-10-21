<properties linkid="dev-nodejs-enablessl" urlDisplayName="Enable SSL" pageTitle="Configure SSL for a cloud service (Node.js) - Azure" metaKeywords="Node.js Azure SSL, Node.js Azure HTTPS" description="Learn how to specify an HTTPS endpoint for a Node.js web role and how to upload an SSL certificate to secure your application." metaCanonical="" services="cloud-services" documentationCenter="Node.js" title="Configuring SSL for a Node.js Application in an Azure Web Role" authors="larryfr" solutions="" manager="" editor="" />

# 在 Azure Web 角色中为 Node.js 应用程序配置 SSL

安全套接字层 (SSL) 加密是用于保护通过 Internet 发送的数据的最常见方法。此常见任务讨论如何为在 Web 角色中托管为 Azure 云服务的 Node.js 应用程序指定 HTTPS 终结点，以及如何上载 SSL 证书以保护你的应用程序。

<div class="dev-callout">说明
<p>本文中的步骤仅适用于在 Web 角色中托管为 Azure 云服务的 Node 应用程序；对于网站，请参阅<a href="../web-sites-configure-ssl-certificate/">为 Azure 网站配置 SSL 证书</a>。</p>
</div>

此任务包括下列步骤：

-   [步骤 1：创建 Node.js 服务并将该服务发布到云][步骤 1：创建 Node.js 服务并将该服务发布到云]
-   [步骤 2：获取 SSL 证书][步骤 2：获取 SSL 证书]
-   [步骤 3：导入 SSL 证书][步骤 3：导入 SSL 证书]
-   [步骤 4：修改服务定义和配置文件][步骤 4：修改服务定义和配置文件]
-   [步骤 5：使用 HTTPS 连接到角色实例][步骤 5：使用 HTTPS 连接到角色实例]

## <a name="step1"> </a>步骤 1：创建 Node.js 服务并将该服务发布到云

将 Node.js 应用程序部署到 Azure Web 角色后，服务器证书和 SSL 连接将由 Internet Information Services (IIS) 进行管理，以便能够像 http 服务一样编写 Node.js 服务。你可以采用以下步骤通过 Azure PowerShell 创建一个简单的 Node.js“hello world”服务：

1.  在“开始”菜单或“开始”屏幕中，搜索 **Azure PowerShell**。最后，右键单击“Azure PowerShell”并选择“以管理员身份运行”。

    ![Azure PowerShell 图标][Azure PowerShell 图标]

[WACOM.INCLUDE [install-dev-tools][install-dev-tools]]

1.  使用 **New-AzureServiceProject** cmdlet 创建新服务项目。

    ![][]

2.  使用 **Add-AzureNodeWebRole** cmdlet 向你的服务中添加 Web 角色：

    ![][1]

3.  使用 **Publish-AzureServiceProject** cmdlet 将你的服务发布到云。

    ![][2]

    <div class="dev-callout">
<strong>说明</strong>
<p>如果你以前未导入 Azure 订阅的发布设置，则在尝试发布时会收到错误。有关下载和导入订阅的发布设置的信息，请参阅<a href="https://www.windowsazure.com/zh-cn/develop/nodejs/how-to-guides/powershell-cmdlets/#ImportPubSettings">如何对 Node.js 使用 Azure PowerShell</a></p>
</div>

**Publish-AzureServiceProject** cmdlet 返回的“创建的网站 URL”值包含你的托管应用程序的完全限定域名。你将需要为此特定的完全限定域名获取一个 SSL 证书，并将其部署到 Azure。

## <a name="step2"> </a>步骤 2：获取 SSL 证书

若要为应用程序配置 SSL，你首先需要获取已由证书颁发机构 (CA)（出于此目的颁发证书的受信任的第三方）签署的 SSL 证书。如果你尚未获取 SSL 证书，则需要从销售 SSL 证书的公司购买一个 SSL 证书。

该证书必须满足 Azure 中的以下 SSL证书要求：

-   证书必须包含私钥。
-   必须为密钥交换创建证书（.pfx 文件）。
-   证书的使用者名称必须与用于访问
    云服务的域匹配。由于你无法为 cloudapp.net 域
    获取 SSL 证书，因此，该证书的使用者名称
    必须与用于访问应用程序的自定义域名匹配。例如 **mysecuresite.cloudapp.net**。
-   该证书必须使用至少 2048 位加密。

## <a name="step3"> </a>步骤 3：导入 SSL 证书

获取证书后，请将其安装到你的开发计算机上的证书存储中。根据你在下一步中进行的配置更改，会检索该证书并将其作为应用程序部署包的一部分上载到 Azure。

<div class="dev-callout">
<strong>说明</strong>
<p>此部分中使用的步骤基于 Windows 8 版本的证书导入向导。如果你使用的是以前版本的 Windows，则此处列出的步骤可能与向导中显示的顺序不一致。如果是这样，请在使用证书导入向导前充分阅读此部分，以便了解必须执行的全部操作。</p>
</div>

若要导入 SSL 证书，请执行以下步骤：

1.  使用 Windows 资源管理器导航到包含证书的 **.pfx** 文件所在的目录，然后双击证书。这将显示“证书导入向导”。

    ![证书向导][证书向导]

2.  在“存储位置”部分，选择“当前用户”，然后单击“下一步”。这会将证书安装在你的用户帐户的证书存储中。

3.  继续向导中的操作，接受默认值，直至转到“私钥保护”屏幕。你必须在这里输入证书的密码（如果有）。还必须选择“标志此密钥为可导出的”。最后，单击“下一步”(Next)。

    ![私钥保护][私钥保护]

4.  继续向导中的操作，接受默认值，直至证书已成功安装。

现在你必须修改服务定义，以引用已安装的证书。

## <a name="step4"> </a>步骤 4：修改服务定义和配置文件

必须将应用程序配置为引用此证书，并且必须添加 HTTPS 终结点。因此，需要更新服务定义和服务配置文件。

1.  在服务目录中，打开服务定义文件(ServiceDefinition.csdef)，在 **WebRole** 节中添加 **Certificates** 节，并包含有关证书的以下信息：

        <WebRole name="WebRole1" vmsize="ExtraSmall">
        ...
            <Certificates>
                <Certificate name="SampleCertificate" 
                    storeLocation="LocalMachine" storeName="My" />
            </Certificates>
        ...
        </WebRole>

    **Certificates** 节定义了我们的证书的名称、其位置及其所在存储的名称。由于我们将证书安装到了用户证书存储中，因此会使用值“My”。还可使用其他证书存储位置。有关详细信息，请参阅[如何将证书与服务关联][如何将证书与服务关联]。

2.  在你的服务定义文件中，更新 **Endpoints** 节中的 http **InputEndpoint** 元素以启用 HTTPS：

        <WebRole name="WebRole1" vmsize="Small">
        ...
            <Endpoints>
                <InputEndpoint name="Endpoint1" protocol="https" 
                    port="443" certificate="SampleCertificate" />
            </Endpoints>
        ...
        </WebRole>

    对服务定义文件进行的所有必需更改已完成，但你仍需要将证书信息添加到服务配置文件中。

3.  在服务配置文件（**ServiceConfiguration.Cloud.cscfg** 和 **ServiceConfiguration.Local.cscfg**）中，将证书添加到 **Role** 节中的空 **Certificates** 节，并将以下示例指纹值替换为你的证书指纹值：

        <Role name="WebRole1">
        ...
            <Certificates>
                <Certificate name="SampleCertificate" 
                    thumbprint="9427befa18ec6865a9ebdc79d4c38de50e6316ff" 
                    thumbprintAlgorithm="sha1" />
            </Certificates>
        ...
        </Role>

4.  通过再次发布服务刷新云中的服务配置。在 Azure PowerShell提示符下，在服务目录中键入 **Publish-AzureServiceProject**。

    作为发布过程的一部分，将从本地证书存储复制引用的证书，并将其包含在部署包中。

## <a name="step5"> </a>步骤 5：使用 HTTPS 连接到角色实例

在 Azure 中启动并运行部署后，可以使用 HTTPS 连接到该部署。

1.  在管理门户中，选择你的云服务，然后单击“仪表板”。

2.  向下滚动并单击显示为“站点 URL”的链接：

    ![站点 url][站点 url]

    <div class="dev-callout">
<strong>说明</strong>
<p>如果门户中显示的站点 URL 未指定 HTTPS，则必须使用 HTTPS（而不是 HTTP）在浏览器中手动输入 URL。</p>
</div>

3.  将打开新浏览器并显示你的网站。

    该浏览器会显示一个锁状图标，指示它使用的是 HTTPS 连接。它还指示已为你的应用程序正确配置 SSL。

    ![][3]

## 其他资源

[如何将证书与服务关联][如何将证书与服务关联]

[在 Azure 辅助角色中为 Node.js 应用程序配置 SSL][在 Azure 辅助角色中为 Node.js 应用程序配置 SSL]

[如何在 HTTPS 终结点上配置 SSL 证书][如何在 HTTPS 终结点上配置 SSL 证书]

  [为 Azure 网站配置 SSL 证书]: ../web-sites-configure-ssl-certificate/
  [步骤 1：创建 Node.js 服务并将该服务发布到云]: #step1
  [步骤 2：获取 SSL 证书]: #step2
  [步骤 3：导入 SSL 证书]: #step3
  [步骤 4：修改服务定义和配置文件]: #step4
  [步骤 5：使用 HTTPS 连接到角色实例]: #step5
  [Azure PowerShell 图标]: ./media/cloud-services-nodejs-configure-ssl-certificate/azure-powershell-start.png
  [install-dev-tools]: ../includes/install-dev-tools.md
  []: ./media/cloud-services-nodejs-configure-ssl-certificate/enable-ssl-01.png
  [1]: ./media/cloud-services-nodejs-configure-ssl-certificate/enable-ssl-02.png
  [2]: ./media/cloud-services-nodejs-configure-ssl-certificate/enable-ssl-03.png
  [如何对 Node.js 使用 Azure PowerShell]: https://www.windowsazure.com/zh-cn/develop/nodejs/how-to-guides/powershell-cmdlets/#ImportPubSettings
  [证书向导]: ./media/cloud-services-nodejs-configure-ssl-certificate/certificateimport.png
  [私钥保护]: ./media/cloud-services-nodejs-configure-ssl-certificate/exportable.png
  [如何将证书与服务关联]: http://msdn.microsoft.com/zh-cn/library/windowsazure/gg465718.aspx
  [站点 url]: ./media/cloud-services-nodejs-configure-ssl-certificate/site-url.png
  [3]: ./media/cloud-services-nodejs-configure-ssl-certificate/enable-ssl-08.png
  [在 Azure 辅助角色中为 Node.js 应用程序配置 SSL]: /zh-cn/develop/nodejs/common-tasks/enable-ssl-worker-role/
  [如何在 HTTPS 终结点上配置 SSL 证书]: http://msdn.microsoft.com/zh-cn/library/windowsazure/ff795779.aspx
