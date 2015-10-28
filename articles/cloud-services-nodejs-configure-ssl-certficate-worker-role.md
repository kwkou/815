<properties 
	pageTitle="为云服务 (Node.js) 辅助角色配置 SSL" 
	description="在 Azure 中为 Node.js 云服务辅助角色配置 SSL" 
	services="cloud-services" 
	documentationCenter="nodejs" 
	authors="MikeWasson" 
	manager="wpickett" 
	editor=""/>

<tags 
	ms.service="cloud-services" 
	ms.date="02/25/2015" 
	wacn.date="10/03/2015"/>





# 在 Azure 辅助角色中为 Node.js 应用程序配置 SSL

安全套接字层 (SSL) 加密是用于保护通过 Internet 发送的数据的最常见方法。此常见任务讨论如何为辅助角色中以 Azure 云服务形式托管的 Node.js 应用程序指定 HTTPS 终结点。

> [AZURE.NOTE]本文中的步骤仅适用于辅助角色中以 Azure 云服务形式托管的 Node 应用程序。

此任务包括下列步骤：

-   [步骤 1：创建 Node.js 服务并将该服务发布到云]
-   [步骤 2：获取 SSL 证书]
-   [步骤 3：将应用程序修改为使用 SSL 证书]
-   [步骤 4：修改服务定义文件]
-   [步骤 5：使用 HTTPS 连接到角色实例]

## <a name="step1"> </a>步骤 1：创建 Node.js 服务并将该服务发布到云

你可以采用以下步骤通过 Azure PowerShell 创建一个简单的 Node.js“hello world”服务：

1. 在“开始”菜单或“开始”屏幕中，搜索 **Azure PowerShell**。最后，右键单击“Azure PowerShell”并选择“以管理员身份运行”。

	![Azure PowerShell 图标][powershell-menu]

	

2.  使用 **New-AzureServiceProject** cmdlet 创建新服务。

	![][1]

3.  使用 **Add-AzureNodeWorkerRole** cmdlet 向你的服务中添加辅助角色。

    ![][2]

4.  使用 **Publish-AzureServiceProject** cmdlet 将你的服务发布到云。

    ![][3]

	> [AZURE.NOTE]如果你以前未导入 Azure 订阅的发布设置，则在尝试发布时会收到错误。有关下载和导入订阅的发布设置的信息，请参阅[如何对 Node.js 使用 Azure PowerShell](/documentation/articles/install-configure-powershell#ImportPubSettings)

**Publish-AzureServiceProject** cmdlet 返回的“创建的网站 URL”值包含你的托管应用程序的完全限定域名。你将需要为此特定的完全限定域名获取一个 SSL 证书，并将其部署到 Azure。

## <a name="step2"> </a>步骤 2：获取 SSL 证书

若要为应用程序配置 SSL，你首先需要获取已由证书颁发机构 (CA)（出于此目的颁发证书的受信任的第三方）签署的 SSL 证书。如果您尚未获取 SSL 证书，将需要从销售 SSL 证书的公司购买一个 SSL 证书。

该证书必须满足 Azure 中的以下 SSL 证书要求：

-   证书必须包含私钥。
-   必须为密钥交换创建证书（**.pfx** 文件）。
-   证书的使用者名称必须与用于访问云服务的域匹配。由于您无法为 chinacloudapp.cn 域获取 SSL 证书，因此，该证书的使用者名称必须与用于访问应用程序的自定义域名匹配。例如，__mysecuresite.chinacloudapp.cn__。
-   该证书必须使用至少 2048 位加密。

包含证书的 **.pfx** 文件将添加到你的服务项目中，并在以下步骤中部署到 Azure。

## <a name="step3"> </a>步骤 3：将应用程序修改为使用 SSL 证书

将 Node.js 应用程序部署到辅助角色后，服务器证书和 SSL 连接将由 Node.exe 进行管理。若要处理 SSL 通信，必须使用“https”模块，而不是“http”。执行以下步骤将 SSL 证书添加到你的项目，然后将应用程序修改为使用该证书。

1.   将证书颁发机构 (CA) 提供给你的 **.pfx** 文件保存到包含你的应用程序的目录。例如，**c:\\node\\securesite\\workerrole1** 是包含本文中使用的应用程序的目录。

2.   使用 Notepad.exe 打开 **c:\\node\\securesite\\workerrole1\\server.js** 文件，并将文件内容替换为以下内容：

		var https = require('https');
		var fs = require('fs');

		var options = {
			pfx: fs.readFileSync('certificate.pfx'),
			passphrase: "password"
		};
		var port = process.env.PORT || 8000;
		https.createServer(options, function (req, res) {
 		    res.writeHead(200, { 'Content-Type': 'text/plain' });
		    res.end('Hello World\n');
		}).listen(port);

	> [AZURE.IMPORTANT]必须将“certificate.pfx”替换为证书的名称，并将“password”替换为证书文件的密码（如果有）。

3.   保存 **server.js** 文件。

对 **server.js** 文件进行修改会导致在将应用程序部署到 Azure 时，该程序会侦听端口 443（SSL 通信的标准端口）上的通信。该 **.pfx** 文件将用于实施此传输上的 SSL 通信。

## <a name="step4"> </a>步骤 4：修改服务定义文件

你的应用程序现在侦听端口 443 上的通信，所以你还必须修改服务定义，以允许通过此端口进行通信。

1.  在服务目录中，打开服务定义文件(**ServiceDefinition.csdef**)，更新 **Endpoints** 节中的 http **InputEndpoint** 元素，以允许通过端口 443 进行通信：

        <WorkerRole name="WorkerRole1" vmsize="Small">
        ...
            <Endpoints>
                <InputEndpoint name="HttpIn" protocol="tcp" 
                    port="443" />
            </Endpoints>
        ...
        </WorkerRole>

	进行此更改后，请保存 **ServiceDefinition.csdef** 文件。

2.  通过再次发布服务在云中刷新更新的配置。在 Azure PowerShell 提示符下，在服务目录中键入 **Publish-AzureServiceProject**。

## <a name="step5"> </a>步骤 5：使用 HTTPS 连接到角色实例

在 Azure 中启动并运行部署后，便可以使用 HTTPS 连接到该部署。

1.  在管理门户中，选择你的云服务，然后单击“仪表板”。

2. 向下滚动并单击显示为“站点 URL”的链接：

    ![站点 url][site-url]

	> [AZURE.IMPORTANT]如果门户中显示的站点 URL 未指定 HTTPS，则必须使用 HTTPS（而不是 HTTP）在浏览器中手动输入 URL。

3.  此时将打开新浏览器并显示你的网站。

    该浏览器会显示一个锁状图标，指示它使用的是 HTTPS 连接。它还指示已为您的应用程序正确配置 SSL。

    ![][8]

## 其他资源

[如何将证书与服务关联]

[在 Azure Web 角色中为 Node.js 应用程序配置 SSL]

[如何在 HTTPS 终结点上配置 SSL 证书]

  [步骤 1：创建 Node.js 服务并将该服务发布到云]: #step1
  [步骤 2：获取 SSL 证书]: #step2
  [步骤 3：将应用程序修改为使用 SSL 证书]: #step3
  [步骤 4：修改服务定义文件]: #step4
  [步骤 5：使用 HTTPS 连接到角色实例]: #step5
  [**Azure PowerShell**]: http://go.microsoft.com/?linkid=9790229&clcid=0x409
  
  
  
  
  [1]: ./media/cloud-services-nodejs-configure-ssl-certficate-worker-role/enable-ssl-01.png
  [2]: ./media/cloud-services-nodejs-configure-ssl-certficate-worker-role/enable-ssl-02-worker.png
  [3]: ./media/cloud-services-nodejs-configure-ssl-certficate-worker-role/enable-ssl-03-worker.png
  [Azure Management Portal]: http://manage.windowsazure.cn
  
  
  [如何将证书与服务关联]: http://msdn.microsoft.com/zh-cn/library/windowsazure/gg465718.aspx
  
  [site-url]: ./media/cloud-services-nodejs-configure-ssl-certficate-worker-role/site-url.png
  [8]: ./media/cloud-services-nodejs-configure-ssl-certficate-worker-role/enable-ssl-08.png
  [如何在 HTTPS 终结点上配置 SSL 证书]: http://msdn.microsoft.com/zh-cn/library/windowsazure/ff795779.aspx
  [powershell-menu]: ./media/cloud-services-nodejs-configure-ssl-certficate-worker-role/azure-powershell-start.png
  
  
  [在 Azure Web 角色中为 Node.js 应用程序配置 SSL]: /zh-cn/documentation/articles/cloud-services-configure-ssl-certificate/
  
 

<!---HONumber=71-->