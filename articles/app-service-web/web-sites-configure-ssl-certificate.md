<!-- not suitable for Mooncake -->

<properties
	pageTitle="使用 HTTPS 保护应用的自定义域 | Azure"
	description="了解如何配置 SSL 证书绑定以在 Azure App Service 中保护应用的自定义域名。还将学习如何从多个工具处获取 SSL 证书。"
	services="app-service"
	documentationCenter=".net"
	authors="cephalin"
	manager="wpickett"
	editor="jimbe"
	tags="top-support-issue"/>

<tags
	ms.service="app-service"
	ms.workload="na"
	ms.tgt_pltfrm="na"
	ms.devlang="na"
	ms.topic="article"
	ms.date="08/08/2016"
	wacn.date=""
	ms.author="cephalin"/>

# 使用 HTTPS 保护应用的自定义域


> [AZURE.SELECTOR]
- [在 Azure 中购买 SSL 证书](/documentation/articles/web-sites-purchase-ssl-web-site/)
- [从其他位置使用 SSL 证书](/documentation/articles/web-sites-configure-ssl-certificate/)


本文演示如何为 Web 应用、移动应用后端或 [Azure App Service](/documentation/articles/app-service-value-prop-what-is/)（使用自定义域名）中的 API 应用启用 HTTPS。仅限服务器的身份验证。若需相互身份验证（包括客户端身份验证），请参阅[如何为应用服务配置 TLS 相互身份验证](/documentation/articles/app-service-web-configure-tls-mutual-auth/)。

若要使用 HTTPS 保护拥有自定义域名的应用，可为该域名添加证书。默认情况下，Azure 使用单个 SSL 证书保护 ***.chinacloudsites.cn** 通配符域的安全，因此客户端可能已在 **https://*&lt;appname>*.chinacloudsites.cn** 处访问了应用。但若想使用 **contoso.com**、**www.contoso.com** 和 ***.contoso.com** 等自定义域，默认证书不能提供保护。此外，与所有[ 通配符证书](https://casecurity.org/2014/02/26/pros-and-cons-of-single-domain-multi-domain-and-wildcard-certificates/)类似，默认证书的安全性没有使用证书的自定义域高。

>[AZURE.NOTE] 可随时在 [Azure 论坛](/support/forums/)获取 Azure 专家的帮助。有关更加个性化的支持，请转到 [Azure 支持](/support/contact/)，然后单击“获取支持”。

## <a name="bkmk_domainname"></a>所需条件
若要使用 HTTPS 保护自定义域名，需将自定义 SSL 证书绑定到 Azure 中的该自定义域。绑定自定义证书之前，需执行以下操作：

- **配置自定义域** - 应用服务只允许向应用中已配置的域名添加证书。相关说明，请参阅[将自定义域名映射到 Azure 应用](/documentation/articles/web-sites-custom-domain-name/)。
- **向上缩放到基本层或更高层** 较低定价层中的应用服务计划不支持自定义 SSL 证书。相关说明，请参阅[向上缩放 Azure 中的应用](/documentation/articles/web-sites-scale/)。
- **获取 SSL 证书** - 如果还没有该证书，需先从受信任的[证书颁发机构](http://zh.wikipedia.org/wiki/证书颁发机构) (CA) 获取。证书必须满足下列所有要求：

	- 由受信任的 CA（而非私人 CA 服务器）签名。
	- 包含私钥。
	- 创建用于交换密钥并导出到 .PFX 文件。
	- 至少使用 2048 位加密。
	- 其使用者名称应与需保护的自定义域相匹配。若要用一个证书保护多个域，需使用通配符名称（如 ***.contoso.com**）或指定 subjectAltName 值。
	- 与 CA 使用的所有**[中间证书](http://en.wikipedia.org/wiki/Intermediate_certificate_authorities)**合并。否则，在某些客户端上可能会遇到不可再现的互操作性问题。

		>[AZURE.NOTE] 若要获取满足所有要求的 SSL 证书，最简单的方法是 [直接在 Azure 门户预览版中购买](/documentation/articles/web-sites-purchase-ssl-web-site/)。本文演示了如何手动执行该操作，然后将它绑定到应用服务中的自定义域。<p>**椭圆曲线加密 (ECC) 证书**可用于应用服务，但文本不予讨论。按具体步骤结合 CA 来创建 ECC 证书。

## <a name="bkmk_getcert"></a>步骤 1。获取 SSL 证书

由于 CA 提供不同价位的多种 SSL 证书类型，首先应确定要购买哪种 SSL 证书。若要保护单个域名（**www.contoso.com**），只需基本的证书。若需保护多个域名（**contoso.com**、**www.contoso.com*** 和 ***mail.contoso.com**），则需要[通配符证书](http://en.wikipedia.org/wiki/Wildcard_certificate)或带[使用者备用名称](http://en.wikipedia.org/wiki/SubjectAltName) (`subjectAltName`) 的证书。

确定要购买的 SSL 证书后，向 CA 提交证书签名请求 (CSR)。如果收到 CA 返回的请求证书，则从证书中生成 .pfx 文件。可自选工具执行这些步骤。下面是常用工具的说明：

- [Certreq.exe 步骤](#bkmk_certreq) - 用于创建证书请求的 Windows 实用程序。自 Windows XP/Windows Server 2000 起，已随附在 Windows 中。
- [IIS 管理器步骤](#bkmk_iismgr) - 若已熟悉该工具，可选择使用。
- [OpenSSL 步骤](#bkmk_openssl) - [跨平台的开源工具](https://www.openssl.org)。可帮助用户从任何平台获取 SSL 证书。
- [使用 OpenSSL 的 subjectAltName 步骤](#bkmk_subjectaltname) - 获取 `subjectAltName` 证书的步骤。

购买证书前，如果想在应用服务中测试设置，可生成 [自签名证书](https://en.wikipedia.org/wiki/Self-signed_certificate)。本教程提供了两种生成该证书的方式：

- [自签名证书，Certreq.exe 步骤](#bkmk_sscertreq)
- [自签名证书，OpenSSL 步骤](#bkmk_ssopenssl)

### <a name="bkmk_certreq"></a>使用 Certreq.exe 获取证书

1. 创建文件（如 **myrequest.txt**），并向其复制以下文本，再将其保存在工作目录中。将 `<your-domain>` 占位符替换为应用的自定义域名。

		[NewRequest]
		Subject = "CN=<your-domain>"  ; E.g. "CN=www.contoso.com", or "CN=*.contoso.com" for a wildcard certificate
		Exportable = TRUE
		KeyLength = 2048              ; Required minimum is 2048
		KeySpec = 1
		KeyUsage = 0xA0
		MachineKeySet = True
		ProviderName = "Microsoft RSA SChannel Cryptographic Provider"
		ProviderType = 12
		HashAlgorithm = SHA256

		[EnhancedKeyUsageExtension]
		OID=1.3.6.1.5.5.7.3.1         ; Server Authentication

	若要深入了解 CSR 中的选项和其他可用选项，请参阅 [Certreq 参考文档](https://technet.microsoft.com/zh-cn/library/dn296456.aspx)。

4. 在命令提示符中，通过 `CD` 转到工作目录，再运行以下命令来创建 CSR：

		certreq -new myrequest.txt myrequest.csr

	现已在当前工作目录中创建 **myrequest.csr**。

5. 向 CA 提交 **myrequest.csr** 以获取 SSL 证书。上传文件，或将其内容从文本编辑器复制到 Web 窗体。

	有关 Microsoft 信任的 CA 列表，请参阅 [Microsoft 信任的根证书程序：参与者][cas] (Microsoft Trusted Root Certificate Program: Participants)。

6. CA 返回证书 ( CER) 文件后，即将其保存到工作目录中。然后，运行以下命令以完成挂起的 CSR。

		certreq -accept -user <certificate-name>.cer

	此命令将完成后的证书存储在 Windows 证书存储中。

6. 如果 CA 使用中间证书，请先安装再继续操作。这些证书通常从 CA 单独下载，会针对不同的 Web 服务器类型提供多种格式。为 Microsoft IIS 选择版本。

	下载证书后，在 Windows 资源管理器中右键单击每个证书，然后选择“安装证书”。使用“证书导入向导”中的默认值，然后继续选择“下一步”，直到完成导入。

7. 若要从证书存储中导出 SSL 证书，请按 `Win`+`R` 并运行 **certmgr.msc** 以启动证书管理器。选择“个人”>“证书”。“颁发给”列应会显示一个条目，内附自定义域名和“颁发者”列中生成证书时所用的 CA。

	![将证书管理器的图像插入此处][certmgr]

9. 右键单击该证书并选择“所有任务”>“导出”。在“证书导出向导”中，单击“下一步”，再选择“是, 导出私钥”，然后再次单击“下一步”。

	![导出私钥][certwiz1]

10. 选择“个人信息交换- PKCS #12”、“将所有证书包括在证书路径内(如可能)”和“导出所有扩展属性”。然后，单击“下一步”。

	![包括所有证书和扩展的属性][certwiz2]

11. 选择“密码”，然后输入并确认该密码。单击**“下一步”**。

	![指定密码][certwiz3]

12. 提供扩展名为 **.pfx** 的导出证书的路径和文件名。单击“下一步”完成操作。

	![提供文件路径][certwiz4]

现可将导出的 PFX 文件上传到应用服务。请参阅[步骤 2.上传和绑定自定义 SSL 证书](#bkmk_configuressl)。

### <a name="bkmk_iismgr"></a>使用 IIS 管理器获取证书

1. 使用 IIS 管理器生成要发送给 CA 的 CSR。有关生成 CSR 的详细信息，请参阅[申请 Internet 服务器证书 (IIS 7)][iiscsr]。

3. 向 CA 提交 CSR 以获取 SSL 证书。有关 Microsoft 信任的 CA 列表，请参阅 [Microsoft 信任的根证书程序：参与者][cas] (Microsoft Trusted Root Certificate Program: Participants)。


3. 使用 CA 发回的证书完成 CSR。有关完成 CSR 的详细信息，请参阅[安装 Internet 服务器证书 (IIS 7)][installcertiis]。

4. 如果 CA 使用中间证书，请先安装再继续操作。这些证书通常从 CA 单独下载，会针对不同的 Web 服务器类型提供多种格式。为 Microsoft IIS 选择版本。

	下载证书后，在 Windows 资源管理器中右键单击每个证书，然后选择“安装证书”。使用“证书导入向导”中的默认值，然后继续选择“下一步”，直到完成导入。

4. 从 IIS 管理器导出 SSL 证书。有关导出证书的详细信息，请参阅[导出服务器证书 (IIS 7)][exportcertiis] (Export a Server Certificate (IIS 7))。

	>[AZURE.IMPORTANT] 在“证书导出向导”中，确保选择“是, 导出私钥”
	>
	>![导出私钥][certwiz1]
	>
	> 还可选择“个人信息交换- PKCS #12”、“将所有证书包括在证书路径内(如可能)”和“导出所有扩展属性”。
	>
	>![包括所有证书和扩展的属性][certwiz2]

现可将导出的 PFX 文件上传到应用服务。请参阅[步骤 2.上传和绑定自定义 SSL 证书](#bkmk_configuressl)。

### <a name="bkmk_openssl"></a>使用 OpenSSL 获取证书

1. 在命令行终端，通过 `CD` 转到工作目录并运行以下命令来生成私钥和 CSR：

		openssl req -sha256 -new -nodes -keyout myserver.key -out server.csr -newkey rsa:2048

2. 在系统提示后，输入适当的信息。例如：

 		Country Name (2 letter code)
        State or Province Name (full name) []: Washington
        Locality Name (eg, city) []: Redmond
        Organization Name (eg, company) []: Microsoft
        Organizational Unit Name (eg, section) []: Azure
        Common Name (eg, YOUR name) []: www.microsoft.com
        Email Address []:

		Please enter the following 'extra' attributes to be sent with your certificate request

       	A challenge password []:

	完成后，工作目录中应有两个文件：**myserver.key** 和 **server.csr**。**Server.csr** 包含 CSR，稍后会需要 **myserver.key**。

3. 向 CA 提交 CSR 以获取 SSL 证书。有关 Microsoft 信任的 CA 列表，请参阅 [Microsoft 信任的根证书程序：参与者][cas] (Microsoft Trusted Root Certificate Program: Participants)。


4. CA 发回请求证书后，将其保存到工作目录中名为 **myserver.crt** 的文件下。如果 CA 提供的是文本格式，只需在文本编辑器中将内容复制到 **myserver.crt**，然后进行保存。该文件应如下所示：

		-----BEGIN CERTIFICATE-----
		MIIDJDCCAgwCCQCpCY4o1LBQuzANBgkqhkiG9w0BAQUFADBUMQswCQYDVQQGEwJV
		UzELMAkGA1UECBMCV0ExEDAOBgNVBAcTB1JlZG1vbmQxEDAOBgNVBAsTB0NvbnRv
		c28xFDASBgNVBAMTC2NvbnRvc28uY29tMB4XDTE0MDExNjE1MzIyM1oXDTE1MDEx
		NjE1MzIyM1owVDELMAkGA1UEBhMCVVMxCzAJBgNVBAgTAldBMRAwDgYDVQQHEwdS
		ZWRtb25kMRAwDgYDVQQLEwdDb250b3NvMRQwEgYDVQQDEwtjb250b3NvLmNvbTCC
		ASIwDQYJKoZIhvcNAQEBBQADggEPADCCAQoCggEBAN96hBX5EDgULtWkCRK7DMM3
		enae1LT9fXqGlbA7ScFvFivGvOLEqEPD//eLGsf15OYHFOQHK1hwgyfXa9sEDPMT
		3AsF3iWyF7FiEoR/qV6LdKjeQicJ2cXjGwf3G5vPoIaYifI5r0lhgOUqBxzaBDZ4
		xMgCh2yv7NavI17BHlWyQo90gS2X5glYGRhzY/fGp10BeUEgIs3Se0kQfBQOFUYb
		ktA6802lod5K0OxlQy4Oc8kfxTDf8AF2SPQ6BL7xxWrNl/Q2DuEEemjuMnLNxmeA
		Ik2+6Z6+WdvJoRxqHhleoL8ftOpWR20ToiZXCPo+fcmLod4ejsG5qjBlztVY4qsC
		AwEAATANBgkqhkiG9w0BAQUFAAOCAQEAVcM9AeeNFv2li69qBZLGDuK0NDHD3zhK
		Y0nDkqucgjE2QKUuvVSPodz8qwHnKoPwnSrTn8CRjW1gFq5qWEO50dGWgyLR8Wy1
		F69DYsEzodG+shv/G+vHJZg9QzutsJTB/Q8OoUCSnQS1PSPZP7RbvDV9b7Gx+gtg
		7kQ55j3A5vOrpI8N9CwdPuimtu6X8Ylw9ejWZsnyy0FMeOPpK3WTkDMxwwGxkU3Y
		lCRTzkv6vnHrlYQxyBLOSafCB1RWinN/slcWSLHADB6R+HeMiVKkFpooT+ghtii1
		A9PdUQIhK9bdaFicXPBYZ6AgNVuGtfwyuS5V6ucm7RE6+qf+QjXNFg==
		-----END CERTIFICATE-----

5. 在命令行终端中，运行以下命令从 **myserver.key** 和 **myserver.crt** 中导出 **myserver.pfx**:

		openssl pkcs12 -export -out myserver.pfx -inkey myserver.key -in myserver.crt

	系统提示后，定义密码以保护该 .pfx 文件。

	> [AZURE.NOTE] 如果 CA 使用中间证书，必须用 `-certfile` 参数包含它们。这些证书通常从 CA 单独下载，会针对不同的 Web 服务器类型提供多种格式。选择具有 `.pem` 扩展名的版本。<p>`openssl -export` 命令应与下例中类似，其创建包含 **intermediate-cets.pem** 文件内的中间证书的 .pfx 文件：<p>`openssl pkcs12 -chain -export -out myserver.pfx -inkey myserver.key -in myserver.crt -certfile intermediate-cets.pem`

现可将导出的 PFX 文件上传到应用服务。请参阅[步骤 2.上传和绑定自定义 SSL 证书](#bkmk_configuressl)。

### <a name="bkmk_subjectaltname"></a>使用 OpenSSL 获取 SubjectAltName 证书

1. 创建名为 **sancert.cnf** 的文件，向其复制以下文本，再将其保存到工作目录中：

		# -------------- BEGIN custom sancert.cnf -----
		HOME = .
		oid_section = new_oids
		[ new_oids ]
		[ req ]
		default_days = 730
		distinguished_name = req_distinguished_name
		encrypt_key = no
		string_mask = nombstr
		req_extensions = v3_req # Extensions to add to certificate request
		[ req_distinguished_name ]
		countryName = Country Name (2 letter code)
		countryName_default =
		stateOrProvinceName = State or Province Name (full name)
		stateOrProvinceName_default =
		localityName = Locality Name (eg, city)
		localityName_default =
		organizationalUnitName  = Organizational Unit Name (eg, section)
		organizationalUnitName_default  =
		commonName              = Your common name (eg, domain name)
		commonName_default      = www.mydomain.com
		commonName_max = 64
		[ v3_req ]
		subjectAltName=DNS:ftp.mydomain.com,DNS:blog.mydomain.com,DNS:*.mydomain.com
		# -------------- END custom sancert.cnf -----

	在以 `subjectAltName` 开头的行中，将值替换为要保护的所有域名（`commonName` 除外）。例如：

		subjectAltName=DNS:sales.contoso.com,DNS:support.contoso.com,DNS:fabrikam.com

	无需更改 `commonName` 在内的任何其他字段。系统将提示在后续几步中指定它们。

1. 在命令行终端，通过 `CD` 转到工作目录并运行以下命令：

		openssl req -sha256 -new -nodes -keyout myserver.key -out server.csr -newkey rsa:2048 -config sancert.cnf

2. 在系统提示后，输入适当的信息。例如：

 		Country Name (2 letter code) []: US
        State or Province Name (full name) []: Washington
        Locality Name (eg, city) []: Redmond
        Organizational Unit Name (eg, section) []: Azure
        Your common name (eg, domain name) []: www.microsoft.com

	完成后，工作目录中应有两个文件：**myserver.key** 和 **server.csr**。**Server.csr** 包含 CSR，稍后会需要 **myserver.key**。

3. 向 CA 提交 CSR 以获取 SSL 证书。有关 Microsoft 信任的 CA 列表，请参阅 [Microsoft 信任的根证书程序：参与者][cas] (Microsoft Trusted Root Certificate Program: Participants)。


4. CA 发回请求证书后，将其保存到名为 **myserver.crt** 的文件下。如果 CA 提供的是文本格式，只需在文本编辑器中将内容复制到 **myserver.crt**，然后进行保存。该文件应如下所示：

		-----BEGIN CERTIFICATE-----
		MIIDJDCCAgwCCQCpCY4o1LBQuzANBgkqhkiG9w0BAQUFADBUMQswCQYDVQQGEwJV
		UzELMAkGA1UECBMCV0ExEDAOBgNVBAcTB1JlZG1vbmQxEDAOBgNVBAsTB0NvbnRv
		c28xFDASBgNVBAMTC2NvbnRvc28uY29tMB4XDTE0MDExNjE1MzIyM1oXDTE1MDEx
		NjE1MzIyM1owVDELMAkGA1UEBhMCVVMxCzAJBgNVBAgTAldBMRAwDgYDVQQHEwdS
		ZWRtb25kMRAwDgYDVQQLEwdDb250b3NvMRQwEgYDVQQDEwtjb250b3NvLmNvbTCC
		ASIwDQYJKoZIhvcNAQEBBQADggEPADCCAQoCggEBAN96hBX5EDgULtWkCRK7DMM3
		enae1LT9fXqGlbA7ScFvFivGvOLEqEPD//eLGsf15OYHFOQHK1hwgyfXa9sEDPMT
		3AsF3iWyF7FiEoR/qV6LdKjeQicJ2cXjGwf3G5vPoIaYifI5r0lhgOUqBxzaBDZ4
		xMgCh2yv7NavI17BHlWyQo90gS2X5glYGRhzY/fGp10BeUEgIs3Se0kQfBQOFUYb
		ktA6802lod5K0OxlQy4Oc8kfxTDf8AF2SPQ6BL7xxWrNl/Q2DuEEemjuMnLNxmeA
		Ik2+6Z6+WdvJoRxqHhleoL8ftOpWR20ToiZXCPo+fcmLod4ejsG5qjBlztVY4qsC
		AwEAATANBgkqhkiG9w0BAQUFAAOCAQEAVcM9AeeNFv2li69qBZLGDuK0NDHD3zhK
		Y0nDkqucgjE2QKUuvVSPodz8qwHnKoPwnSrTn8CRjW1gFq5qWEO50dGWgyLR8Wy1
		F69DYsEzodG+shv/G+vHJZg9QzutsJTB/Q8OoUCSnQS1PSPZP7RbvDV9b7Gx+gtg
		7kQ55j3A5vOrpI8N9CwdPuimtu6X8Ylw9ejWZsnyy0FMeOPpK3WTkDMxwwGxkU3Y
		lCRTzkv6vnHrlYQxyBLOSafCB1RWinN/slcWSLHADB6R+HeMiVKkFpooT+ghtii1
		A9PdUQIhK9bdaFicXPBYZ6AgNVuGtfwyuS5V6ucm7RE6+qf+QjXNFg==
		-----END CERTIFICATE-----

5. 在命令行终端中，运行以下命令从 **myserver.key** 和 **myserver.crt** 中导出 **myserver.pfx**:

		openssl pkcs12 -export -out myserver.pfx -inkey myserver.key -in myserver.crt

	系统提示后，定义密码以保护该 .pfx 文件。

	> [AZURE.NOTE] 如果 CA 使用中间证书，必须用 `-certfile` 参数包含它们。这些证书通常从 CA 单独下载，会针对不同的 Web 服务器类型提供多种格式。选择具有 `.pem` 扩展名的版本。<p>`openssl -export` 命令应与下例中类似，其创建包含 **intermediate-cets.pem** 文件内的中间证书的 .pfx 文件：<p>`openssl pkcs12 -chain -export -out myserver.pfx -inkey myserver.key -in myserver.crt -certfile intermediate-cets.pem`

现可将导出的 PFX 文件上传到应用服务。请参阅[步骤 2.上传和绑定自定义 SSL 证书](#bkmk_configuressl)。

### <a name="bkmk_sscertreq"></a>使用 Certreq.exe生成自签名证书 ###

>[AZURE.IMPORTANT] 自签名证书仅用于测试目的。在访问受自签名证书保护的网站时，大多数浏览器会返回错误。某些浏览器甚至可能会拒绝导航到网站。

1. 创建文本文件（如 **mycert.txt**），向其复制以下文本并将其文件保存到工作目录中。将 `<your-domain>` 占位符替换为应用的自定义域名。

		[NewRequest]
		Subject = "CN=<your-domain>"  ; E.g. "CN=www.contoso.com", or "CN=*.contoso.com" for a wildcard certificate
		Exportable = TRUE
		KeyLength = 2048              ; KeyLength can be 2048, 4096, 8192, or 16384 (required minimum is 2048)
		KeySpec = 1
		KeyUsage = 0xA0
		MachineKeySet = True
		ProviderName = "Microsoft RSA SChannel Cryptographic Provider"
		ProviderType = 12
		HashAlgorithm = SHA256
		RequestType = Cert            ; Self-signed certificate
		ValidityPeriod = Years
		ValidityPeriodUnits = 1

		[EnhancedKeyUsageExtension]
		OID=1.3.6.1.5.5.7.3.1         ; Server Authentication

	参数 `RequestType = Cert` 很重要，可指定自签名证书。若要深入了解 CSR 中的选项和其他可用选项，请参阅 [](https://technet.microsoft.com/zh-cn/library/dn296456.aspx)。

4. 在命令提示符中，通过 `CD` 转到工作目录并运行以下命令：

		certreq -new mycert.txt mycert.crt
	
	新的自签名证书现已安装在证书存储中。

7. 若要导出证书存储中的证书，请按 `Win`+`R` 并运行 **certmgr.msc** 以启动证书管理器。选择“个人”>“证书”。“颁发给”列应会显示一个条目，内附自定义域名和“颁发者”列中生成证书时所用的 CA。

	![将证书管理器的图像插入此处][certmgr]

9. 右键单击该证书并选择“所有任务”>“导出”。在“证书导出向导”中，单击“下一步”，再选择“是, 导出私钥”，然后再次单击“下一步”。

	![导出私钥][certwiz1]

10. 选择“个人信息交换- PKCS #12”、“将所有证书包括在证书路径内(如可能)”和“导出所有扩展属性”。然后，单击“下一步”。

	![包括所有证书和扩展的属性][certwiz2]

11. 选择“密码”，然后输入并确认该密码。单击**“下一步”**。

	![指定密码][certwiz3]

12. 提供扩展名为 **.pfx** 的导出证书的路径和文件名。单击“下一步”完成操作。

	![提供文件路径][certwiz4]

现可将导出的 PFX 文件上传到应用服务。请参阅[步骤 2.上传和绑定自定义 SSL 证书](#bkmk_configuressl)。

### <a name="bkmk_ssopenssl"></a>使用 OpenSSL 生成自签名证书 ###

>[AZURE.IMPORTANT] 自签名证书仅用于测试目的。在访问受自签名证书保护的网站时，大多数浏览器会返回错误。某些浏览器甚至可能会拒绝导航到网站。

1. 创建一个名为 **serverauth.cnf** 的文本文件，然后向其复制以下内容，并将其保存到工作目录中：

        [ req ]
        default_bits           = 2048
        default_keyfile        = privkey.pem
        distinguished_name     = req_distinguished_name
        attributes             = req_attributes
        x509_extensions        = v3_ca

        [ req_distinguished_name ]
        countryName			= Country Name (2 letter code)
        countryName_min			= 2
        countryName_max			= 2
        stateOrProvinceName		= State or Province Name (full name)
        localityName			= Locality Name (eg, city)
        0.organizationName		= Organization Name (eg, company)
        organizationalUnitName		= Organizational Unit Name (eg, section)
        commonName			= Common Name (eg, your app's domain name)
        commonName_max			= 64
        emailAddress			= Email Address
        emailAddress_max		= 40

        [ req_attributes ]
        challengePassword		= A challenge password
        challengePassword_min		= 4
        challengePassword_max		= 20

        [ v3_ca ]
         subjectKeyIdentifier=hash
         authorityKeyIdentifier=keyid:always,issuer:always
         basicConstraints = CA:false
         keyUsage=nonRepudiation, digitalSignature, keyEncipherment
         extendedKeyUsage = serverAuth

2. 在命令行终端，通过 `CD` 转到工作目录并运行以下命令：

		openssl req -sha256 -x509 -nodes -days 365 -newkey rsa:2048 -keyout myserver.key -out myserver.crt -config serverauth.cnf

	此命令会基于 **serverauth.cnf** 中的设置创建两个文件：**myserver.crt**（自签名证书）和 **myserver.key**（私钥）。

3. 运行以下命令将证书导出到 .pfx 文件：

		openssl pkcs12 -export -out myserver.pfx -inkey myserver.key -in myserver.crt

	系统提示后，定义密码以保护该 .pfx 文件。

现可将导出的 PFX 文件上传到应用服务。请参阅[步骤 2.上传和绑定自定义 SSL 证书](#bkmk_configuressl)。

## <a name="bkmk_configuressl"></a>步骤 2。上传和绑定自定义 SSL 证书

继续操作之前，请查看[所需条件](#bkmk_domainname)部分并验证：

- 是否有映射到 Azure 应用的自定义域，
- 应用是否正在**基本**层或更高层上运行，
- 是否有 CA 的自定义域 SSL 证书。


1. 在浏览器中，打开 **[Azure 门户预览版](https://portal.azure.cn/)**。
2. 单击页面左侧的“应用服务”选项。
3. 单击要分配此证书的应用的名称。
4. 在“设置”中，单击“SSL 证书”
5. 单击“上载证书”
6. 选择[步骤 1](#bkmk_getcert) 中导出的 .pfx 文件并指定之前创建的密码。然后单击“上载”上载证书。随后可在“SSL 证书”边栏选项卡中看到上载的证书。
7. 在“ssl 绑定”部分中，单击“添加绑定”
8. 在“添加 SSL 绑定”边栏选项卡中，使用下拉列表选择要使用 SSL 保护的域名，然后选择要使用的证书。还可以选择是要使用**[服务器名称指示 (SNI)](http://en.wikipedia.org/wiki/Server_Name_Indication)** 还是基于 IP 的 SSL。

    ![插入 SSL 绑定的图像](./media/web-sites-configure-ssl-certificate/sslbindings.png)

       • 基于 IP 的 SSL 通过将服务器的专用公共 IP 地址映射到域名，将证书与域名相关联。这要求与您的服务相关联的每个域名（contoso.com、fabricam.com 等）都具有专用的 IP 地址。这是将 SSL 证书与某一 Web 服务器相关联的传统方法。• 基于 SNI 的 SSL 是对 SSL 和**[传输层安全性](http://zh.wikipedia.org/wiki/Transport_Layer_Security)** (TLS) 的扩展，它允许多个域共享相同的 IP 地址，并且对于每个域都有单独的安全证书。当前常用的大多数浏览器（包括 Internet Explorer、Chrome、Firefox 和 Opera）都支持 SNI，但是，较旧的浏览器可能不支持 SNI。有关 SNI 的详细信息，请参阅 Wikipedia 上的文章 **[Server Name Indication](http://en.wikipedia.org/wiki/Server_Name_Indication)**（服务器名称指示）。
     
9. 单击“添加绑定”保存更改并启用 SSL。

## 步骤 3.更改域名映射（仅限基于 IP 的 SSL）

如果使用 **SNI SSL** 绑定，请跳过此部分。多个 **SNI SSL** 绑定可在分配给应用的现有共享 IP 地址上协同工作。但是，如果创建**基于 IP 的 SSL** 绑定，应用服务会创建绑定专用的 IP 地址（因其需要）。只能创建一个专用 IP 地址，因此只能添加一个**基于 IP 的 SSL** 绑定。

因为此 IP 地址专用，下述情况将需要进一步配置应用：

- [使用 A 记录将自定义域映射](/documentation/articles/web-sites-custom-domain-name/#a)到 Azure 应用，且刚添加 **基于 IP 的 SSL** 绑定。此应用场景下，需按以下步骤重新映射现有的 A 记录以指向专用 IP 地址：

	1. 配置基于 IP 的 SSL 绑定后，将会向你的应用分配专用 IP 地址。可以在“自定义域”页面中应用设置的下面（紧靠在“主机名”部分的上方）找到此 IP 地址。此 IP 地址作为“外部 IP 地址”列出
    
	    ![虚拟 IP 地址](./media/web-sites-configure-ssl-certificate/virtual-ip-address.png)

	2. [将自定义域名的 A 记录重新映射到新的 IP 地址](/documentation/articles/web-sites-custom-domain-name/#a)。

- 应用中已有一个或多个 **SNI SSL** 绑定，且刚添加了**基于 IP 的 SSL** 绑定。绑定完成后，*&lt;appname>*.chinacloudsites.cn 域名指向新的 IP 地址。因此，任何[从自定义域映射](/documentation/articles/web-sites-custom-domain-name/#cname)到 *&lt;appname>* chinacloudsites.cn 的现有 CNAME（包括 **SNI SSL**保护的）也会收到新地址上的流量，其中该地址仅针对**基于 IP 的 SSL** 创建。此应用场景中，需按以下步骤将 **SNI SSL** 流量发回原始共享 IP 地址：

	1. 标识所有到带 **SNI SSL** 绑定的自定义域的 [CNAME 映射](/documentation/articles/web-sites-custom-domain-name/#cname)。

	2. 将每个 CNAME 记录重新映射到 **sni.**&lt;appname>.chinacloudsites.cn，而不是 &lt;appname>.chinacloudsites.cn。

## 步骤 4。测试自定义域的 HTTPS

接下来只需确保 HTTPS 适用于自定义域。在各种浏览器中，浏览到 `https://<your.custom.domain>` 以查看其是否适合应用。

- 如果应用显示证书验证错误，可能是因为使用自签名证书。

- 若非此情况，则可能在导出 .pfx 证书时遗漏了中间证书。返回到 [所需条件](#bkmk_domainname)，验证 CSR 是否满足应用服务的所有需求。

## <a name="bkmk_enforce"></a>在应用上强制实施 HTTPS

如果仍想允许到应用的 HTTP 访问，请跳过此步骤。应用服务 *不* 强制实施 HTTPS，因此访问者仍可使用 HTTP 访问应用。如果想要对应用强制实施 HTTPS，则可在应用的 `web.config` 文件中定义重写规则。无论应用的语言框架如何，每个应用服务应用都有此文件。

> [AZURE.NOTE] 存在语言特定的请求重定向。ASP.NET MVC 可使用 [RequireHttps](http://msdn.microsoft.com/zh-cn/library/system.web.mvc.requirehttpsattribute.aspx) 筛选器，而非 `web.config` 中的重写规则（请参阅[将安全的 ASP.NET MVC 5 应用部署到 Web 应用](/documentation/articles/web-sites-dotnet-deploy-aspnet-mvc-app-membership-oauth-sql-database/)）。

执行以下步骤:

1. 导航到应用的 Kudu 调试控制台。地址是 `https://<appname>.scm.chinacloudsites.cn/DebugConsole`。

2. 在调试控制台中，通过 CD 进入 `D:\home\site\wwwroot`。

3. 单击铅笔按钮打开 `web.config`。

	![](./media/web-sites-configure-ssl-certificate/openwebconfig.png)

	如果使用 Visual Studio 或 Git 部署应用，应用服务会在应用程序根目录中为 .NET、PHP、Node.js 或 Python 自动生成相应的 `web.config`。如果 `web.config` 不存在，则在基于 Web 的命令提示符处运行 `touch web.config`创建。或者可在本地项目中创建它并重新部署代码。

4. 若必需创建 `web.config`，请向其复制以下代码并进行保存。如果打开了现有的 web.config，只需将整个 `<rule>` 标记到 `web.config` 的 `configuration/system.webServer/rewrite/rules` 元素中。

		<?xml version="1.0" encoding="UTF-8"?>
		<configuration>
		  <system.webServer>
		    <rewrite>
		      <rules>
			    <!-- BEGIN rule TAG FOR HTTPS REDIRECT -->
		        <rule name="Force HTTPS" enabled="true">
		          <match url="(.*)" ignoreCase="false" />
		          <conditions>
		            <add input="{HTTPS}" pattern="off" />
		          </conditions>
		          <action type="Redirect" url="https://{HTTP_HOST}/{R:1}" appendQueryString="true" redirectType="Permanent" />
		        </rule>
				<!-- END rule TAG FOR HTTPS REDIRECT -->
		      </rules>
		    </rewrite>
          </system.webServer>
		</configuration>

	用户使用 HTTP 请求页面时，此规则都会将 HTTP 301（永久重定向）返回到 HTTPS 协议。它会从 http://contoso.com 重定向到 https://contoso.com。

	>[AZURE.IMPORTANT] 如果 `web.config` 中已有其他 `<rule>` 标记，则将复制的 `<rule>` 标记置于另一 `<rule>` 标记前。

4. 将文件保存在 Kudu 调试控制台。此操作立即生效并将所有请求重定向到 HTTPS。

有关 IIS URL 重写模块的详细信息，请参阅 [URL 重写](http://www.iis.net/downloads/microsoft/url-rewrite)文档。

## 更多资源 ##
- [Azure 信任中心](/support/trust-center/security/)
- [Azure 网站中解锁的设置选项](/blog/2014/01/28/more-to-explore-configuration-options-unlocked-in-windows-azure-web-sites/)
- [启用诊断日志记录](/documentation/articles/web-sites-enable-diagnostic-log/)
- [在 Azure App Service 中配置 Web Apps](/documentation/articles/web-sites-configure/)
- [Azure 管理门户](https://manage.windowsazure.cn)

[customdomain]: /documentation/articles/web-sites-custom-domain-name/
[iiscsr]: http://technet.microsoft.com/zh-cn/library/cc732906(WS.10).aspx
[cas]: http://social.technet.microsoft.com/wiki/contents/articles/31634.microsoft-trusted-root-certificate-program-participants-v-2016-april.aspx
[installcertiis]: http://technet.microsoft.com/zh-cn/library/cc771816(WS.10).aspx
[exportcertiis]: http://technet.microsoft.com/zh-cn/library/cc731386(WS.10).aspx
[openssl]: http://www.openssl.org/
[portal]: https://manage.windowsazure.cn/
[tls]: http://zh.wikipedia.org/wiki/Transport_Layer_Security
[staticip]: ./media/web-sites-configure-ssl-certificate/staticip.png
[website]: ./media/web-sites-configure-ssl-certificate/sslwebsite.png
[scale]: ./media/web-sites-configure-ssl-certificate/sslscale.png
[standard]: ./media/web-sites-configure-ssl-certificate/sslreserved.png
[pricing]: /pricing/overview/
[configure]: ./media/web-sites-configure-ssl-certificate/sslconfig.png
[uploadcert]: ./media/web-sites-configure-ssl-certificate/ssluploadcert.png
[uploadcertdlg]: ./media/web-sites-configure-ssl-certificate/ssluploaddlg.png
[sslbindings]: ./media/web-sites-configure-ssl-certificate/sslbindings.png
[sni]: http://en.wikipedia.org/wiki/Server_Name_Indication
[certmgr]: ./media/web-sites-configure-ssl-certificate/waws-certmgr.png
[certwiz1]: ./media/web-sites-configure-ssl-certificate/waws-certwiz1.png
[certwiz2]: ./media/web-sites-configure-ssl-certificate/waws-certwiz2.png
[certwiz3]: ./media/web-sites-configure-ssl-certificate/waws-certwiz3.png
[certwiz4]: ./media/web-sites-configure-ssl-certificate/waws-certwiz4.png

<!---HONumber=Mooncake_0926_2016-->