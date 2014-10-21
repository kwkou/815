<properties linkid="dev-net-commons-tasks-enable-ssl" urlDisplayName="Enable SSL" pageTitle="Configure SSL for a cloud service - Azure" metaKeywords="Azure SSL, Azure HTTPS, Azure SSL, Azure HTTPS, .NET Azure SSL, .NET Azure HTTPS, C# Azure SSL, C# Azure HTTPS, VB Azure SSL, VB Azure HTTPS" description="Learn how to specify an HTTPS endpoint for a web role and how to upload an SSL certificate to secure your application." metaCanonical="" services="cloud-services" documentationCenter=".NET" title="Configuring SSL for an application in Azure" authors="" solutions="" manager="jeffreyg" editor="mollybos" />

# 在 Azure 中为应用程序配置 SSL

安全套接字层 (SSL) 加密是用于保护通过 Internet 发送的数据的最常见方法。此常见任务讨论了如何为 Web 角色指定 HTTPS 终结点以及如何上载 SSL 证书来保护你的应用程序。

<div class="dev-callout">说明
<p>本任务中的过程适用于 Azure 云服务；对于网站，请参阅<a href="../web-sites-configure-ssl-certificate/">为 Azure 网站配置 SSL 证书</a>。</p>
</div>

此任务包括下列步骤：

-   [步骤 1：获取 SSL 证书][步骤 1：获取 SSL 证书]
-   [步骤 2：修改服务定义和配置文件][步骤 2：修改服务定义和配置文件]
-   [步骤 3：上载部署包和证书][步骤 3：上载部署包和证书]
-   [步骤 4：使用 HTTPS 连接到角色实例][步骤 4：使用 HTTPS 连接到角色实例]

此任务将使用生产部署；本主题的末尾提供了有关如何使用过渡部署的信息。

## <a name="step1"> </a><span class="short-header">获取 SSL 证书</span>步骤 1：获取 SSL 证书

若要为应用程序配置 SSL，你首先需要获取已由证书颁发机构 (CA)（出于此目的颁发证书的受信任的第三方）签署的 SSL 证书。如果你尚未获取 SSL 证书，则需要从销售 SSL 证书的公司购买一个 SSL 证书。

该证书必须满足 Azure 中的以下 SSL 证书要求：

-   证书必须包含私钥。
-   必须为密钥交换创建证书，并且该证书可导出到个人信息交换 (.pfx) 文件。
-   证书的使用者名称必须与用于访问云服务的域匹配。你无法从证书颁发机构 (CA) 处获取针对 cloudapp.net 域的 SSL 证书。你必须获取在访问服务时要使用的自定义域名。在从 CA 处请求证书时，该证书的使用者名称必须与用于访问应用程序的自定义域名匹配。例如，如果自定义域名为 **contoso.com**，则可以从 CA 处请求针对 ***.contoso.com** 或 **www.contoso.com** 的证书。
-   该证书必须使用至少 2048 位加密。

你可以创建并使用测试用的自签名证书。自签名证书不通过 CA 进行身份验证并可使用 cloudapp.net 域作为网站 URL。例如，以下任务使用其常见名 (CN) 为 **sslexample.cloudapp.net** 的自签名证书。有关如何使用 IIS 管理器创建自签名证书的详细信息，请参见[如何为角色创建证书][如何为角色创建证书]。

接下来，你必须在服务定义和服务配置文件中包含有关此证书的信息。

## <a name="step2"> </a><span class="short-header">修改服务定义/配置文件步骤</span>步骤 2：修改服务定义和配置文件

必须将应用程序配置为使用此证书，并且必须添加 HTTPS 终结点。因此，需要更新服务定义和服务配置文件。

1.  在你的开发环境中，打开服务定义文件 (CSDEF)，在 **WebRole** 节中添加 **Certificates** 节，并包含以下证书相关信息：

        <WebRole name="CertificateTesting" vmsize="Small">
        ...
            <Certificates>
                <Certificate name="SampleCertificate" 
                             storeLocation="LocalMachine" 
                             storeName="CA" />
            </Certificates>
        ...
        </WebRole>

    **Certificates** 节定义了我们的证书的名称、其位置及其所在存储的名称。我们已选择将此证书存储到 CA（证书颁发机构）存储中，但你也可以选择其他选项。有关详细信息，请参阅[如何将证书与服务关联][如何将证书与服务关联]。

2.  在你的服务定义文件中，在 **Endpoints** 节中添加 **InputEndpoint** 元素以启用 HTTPS：

        <WebRole name="CertificateTesting" vmsize="Small">
        ...
            <Endpoints>
                <InputEndpoint name="HttpsIn" protocol="https" port="443" 
                    certificate="SampleCertificate" />
            </Endpoints>
        ...
        </WebRole>

3.  在你的服务定义文件中，在 **Sites** 节中添加 **Binding** 元素。这将添加 HTTPS 绑定以将终结点映射到你的网站：

        <WebRole name="CertificateTesting" vmsize="Small">
        ...
            <Sites>
                <Site name="Web">
                    <Bindings>
                        <Binding name="HttpsIn" endpointName="HttpsIn" />
                    </Bindings>
                </Site>
            </Sites>
        ...
        </WebRole>

    对服务定义文件进行的所有必需更改已完成，但你仍需要将证书信息添加到服务配置文件中。

4.  在服务配置文件 (CSCFG) ServiceConfiguration.Cloud.cscfg 中，在 **Role** 节中添加 **Certificates** 节，并将下面显示的示例指纹值替换为证书的指纹值：

        <Role name="Deployment">
        ...
            <Certificates>
                <Certificate name="SampleCertificate" 
                    thumbprint="9427befa18ec6865a9ebdc79d4c38de50e6316ff" 
                    thumbprintAlgorithm="sha1" />
            </Certificates>
        ...
        </Role>

（上面的示例使用 **sha1** 作为指纹算法。请为证书的指纹算法指定适当的值。）

现在已更新服务定义和服务配置文件，请打包你的部署以上载到 Azure。如果你使用的是 **cspack**，请确保不要使用**/generateConfigurationFile** 标记，因为这会覆盖刚刚插入的证书信息。

## <a name="step3"> </a><span class="short-header">上载到 Azure</span>步骤 3：上载部署包和证书

已将部署包更新为使用此证书，并且已添加 HTTPS 终结点。现在你可以使用管理门户将包和证书上载到 Azure。

1.  登录到 [Azure 管理门户][Azure 管理门户]。
2.  依次单击“新建”、“云服务”和“自定义创建”。
3.  在“创建云服务”对话框中，输入 URL、区域/地缘组和订阅的值。确保选中“立即部署云服务包”，并单击“下一步”按钮。
4.  在“发布云服务”对话框中，输入云服务的所需信息，为环境选择“生产”，并确保选中“立即添加证书”。（如果你的任何角色包含单个实例，请确保选中“即使一个或多个角色包含单个实例也进行部署”。）

    ![发布云服务][发布云服务]

5.  单击“下一步”按钮。
6.  在“添加证书”对话框中，输入 SSL 证书 .pfx 文件的位置和该证书的密码，然后单击“附加证书”。

    ![添加证书][添加证书]

7.  确保你的证书已在“附加的证书”部分中列出。

    ![附加的证书][附加的证书]

8.  单击“完成”按钮以创建云服务。当部署达到“就绪”状态时，你可以继续执行后续步骤。

## <a name="step4"> </a><span class="short-header">使用 HTTPS 进行连接</span>步骤 4：使用 HTTPS 连接到角色实例

在 Azure 中启动并运行部署后，可以使用 HTTPS 连接到该部署。

1.  在管理门户中，选择你的部署，然后单击“站点 URL”下的链接。

    ![确定网站 URL][确定网站 URL]

2.  在 Web 浏览器中，将链接修改为使用 **https** 而不是 **http**，然后访问该页。

    **注意：**如果你使用的是自签名证书，则当你浏览到与自签名证书关联的 HTTPS 终结点时，浏览器中将显示一个证书错误。使用由受信任的证书颁发机构签名的证书可避免此问题；同时，你可以忽略此错误。（另一个选项是将自签名证书添加到用户的受信任证书颁发机构证书存储中。）

    ![SSL 示例网站][SSL 示例网站]

若要对过渡部署而非生产部署使用 SSL，你首先需要确定用于过渡部署的 URL。将云服务部署到过渡环境，而不包括证书或任何证书信息。部署后，你可以确定基于 GUID 的 URL，此 URL 将在管理门户的“网站 URL”字段中列出。使用等效于基于 GUID 的 URL（例如，**32818777-6e77-4ced-a8fc-57609d404462.cloudapp.net**）的公用名 (CN) 创建一个证书，再使用管理门户将该证书添加到过渡云服务，将该证书的信息添加到你的 CSDEF 和 CSCFG 文件，重新打包你的应用程序，然后将过渡部署更新为使用新的包和 CSCFG 文件。

## <a name="additional_resources"> </a><span class="short-header">其他资源</span>其他资源

-   [如何将证书与服务关联][如何将证书与服务关联]

-   [如何在 HTTPS 终结点上配置 SSL 证书][如何在 HTTPS 终结点上配置 SSL 证书]

  [为 Azure 网站配置 SSL 证书]: ../web-sites-configure-ssl-certificate/
  [步骤 1：获取 SSL 证书]: #step1
  [步骤 2：修改服务定义和配置文件]: #step2
  [步骤 3：上载部署包和证书]: #step3
  [步骤 4：使用 HTTPS 连接到角色实例]: #step4
  [如何为角色创建证书]: http://msdn.microsoft.com/zh-cn/library/windowsazure/gg432987.aspx
  [如何将证书与服务关联]: http://msdn.microsoft.com/zh-cn/library/windowsazure/gg465718.aspx
  [Azure 管理门户]: http://manage.windowsazure.cn
  [发布云服务]: ./media/cloud-services-dotnet-configure-ssl-certificate/CreateCloudService.png
  [添加证书]: ./media/cloud-services-dotnet-configure-ssl-certificate/AddCertificate.png
  [附加的证书]: ./media/cloud-services-dotnet-configure-ssl-certificate/AddCertificateComplete.png
  [确定网站 URL]: ./media/cloud-services-dotnet-configure-ssl-certificate/CopyURL.png
  [SSL 示例网站]: ./media/cloud-services-dotnet-configure-ssl-certificate/SSLCloudService.png
  [如何在 HTTPS 终结点上配置 SSL 证书]: http://msdn.microsoft.com/zh-cn/library/windowsazure/ff795779.aspx
