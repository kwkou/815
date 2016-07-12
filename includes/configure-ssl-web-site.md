
本文介绍如何在 Azure 中为 Web 应用配置 HTTPS。其中不涉及客户端证书身份验证；有关该方面的信息，请参阅[如何为 Web 应用配置 TLS 相互身份验证](/documentation/articles/app-service-web-configure-tls-mutual-auth/)。

默认情况下，Azure 已使用 *.chinacloudsites.cn 域的通配符证书为你的应用启用 HTTP。如果不打算配置自定义域，可以直接利用默认的 HTTPS 证书。但是，就像[所有通配符域](https://casecurity.org/2014/02/26/pros-and-cons-of-single-domain-multi-domain-and-wildcard-certificates/)一样，这并不如将自定义域与自己的证书搭配使用那么安全。

本文档的其余部分提供了有关为自定义域启用 HTTPS 的详细信息，例如 **contoso.com**、**www.contoso.com** 或 **\*.contoso.com**

##<a name="bkmk_domainname"></a> 为自定义域启用 SSL

若要为自定义域启用 HTTPS（例如 **contoso.com**），必须先[在 Azure Web 应用中配置自定义域名](/documentation/articles/web-sites-custom-domain-name/)。然后，执行下列操作：

1. [获取 SSL 证书](#bkmk_getcert)
2. [配置标准定价层](#bkmk_standardmode)
2. [在应用中配置 SSL](#bkmk_configuressl)
3. [对应用强制实施 SSL](#bkmk_enforce)（可选）

如果在本文中有任何需要协助的地方，你可以联系 [MSDN Azure 和 CSDN Azure](/support/forums/) 上的 Azure 专家。或者，你也可以提出 Azure 支持事件。请转到 [Azure 支持站点](/support/contact/)并单击“获取支持”。

##<a name="bkmk_getcert"></a> 1\.获取 SSL 证书

请求 SSL 证书之前，必须先确定将受证书保护的域名。这将确定必须获取的证书类型。如果你只需保护单个域名（例如 **contoso.com** 或 **www.contoso.com**）的安全，基本证书就足够了。如果你需要保护多个域名（例如 **contoso.com**、**www.contoso.com** 和 **mail.contoso.com**）的安全，则可以获取[通配符证书](http://en.wikipedia.org/wiki/Wildcard_certificate)或带有[使用者备用名称](http://en.wikipedia.org/wiki/SubjectAltName) (subjectAltName) 的证书。

配合 Azure 使用的 SSL 证书必须由[证书颁发机构](http://zh.wikipedia.org/wiki/证书颁发机构) (CA) 签名。如果你尚未获取 SSL 证书，将需要从颁发 SSL 证书的公司购买一个 SSL 证书。

该证书必须满足 Azure 中的以下 SSL 证书要求：

* 证书必须包含私钥。
* 必须为密钥交换创建证书，并且该证书可导出到个人信息交换 (.pfx) 文件。
* 证书的使用者名称必须与用于访问应用的域匹配。如果您需要使用此证书为多个域提供服务，则您将需要使用通配符值或指定 subjectAltName 值，如前面所述。
* 该证书应使用至少 2048 位加密。
* 从专用 CA 服务器颁发的证书不受 Azure Web 应用支持。

若要获取可用于 Azure Web 应用的 SSL 证书，你可以将证书签名请求 (CSR) 提交到证书颁发机构，然后基于收到的证书生成 .pfx 文件。可以使用所选的工具来执行此操作。以下是获取证书的一些常见方式：

- [使用 Certreq.exe 获取证书](#bkmk_certreq)
- [使用 IIS 管理器获取证书](#bkmk_iismgr)
- [使用 OpenSSL 获取证书](#bkmk_openssl)
- [使用 OpenSSL 获取 SubjectAltName 证书](#bkmk_subjectaltname)
- [生成自签名证书（仅用于测试）](#bkmk_selfsigned)

> [AZURE.NOTE]在遵循步骤操作时，系统将提示你输入**公用名**，例如 `www.contoso.com`。对于通配符证书，此值应为 \*.domainname（例如，\*.contoso.com）。如果需要同时支持通配符名称（如 *.contoso.com）和根域名（如 contoso.com），可以使用通配符 subjectAltName 证书。
><p>Azure Web 应用支持椭圆曲线加密 (ECC) 证书；不过，它们相对较新，你应该在具体步骤中使用 CA 来创建 CSR。

你可能还需要获取**[中间证书](http://en.wikipedia.org/wiki/Intermediate_certificate_authorities)**（也称为链证书），前提是这些证书由你的 CA 使用。与“非链式证书”相比，使用中间证书被认为更安全，因此 CA 通常使用这些证书。中间证书通常作为从 CA 网站的单独下载提供。本文中的步骤提供了有关如何确保任何中间证书与上载到应用的证书合并的步骤。

> [AZURE.NOTE]
> 如果你的 CA 使用中间证书，每个中间证书则必须和为你的域颁发的证书一起安装。无法安装任何中间证书可能会导致很难为某些客户端重现互操作性问题。

###<a name="bkmk_certreq"></a> 使用 Certreq.exe 获取证书（仅限 Windows）

Certreq.exe 是用于创建证书请求的 Windows 实用程序。它已成为自 Windows XP/Windows Server 2000 之后基本 Windows 安装的一部分，因此应该在最新的 Windows 系统上提供它。使用以下步骤获取使用 certreq.exe 的 SSL 证书。

1. 打开“记事本”并创建包含以下内容的新文档。将 Subject 一行中的 **mysite.com** 替换为应用的自定义域名。例如，Subject =“CN=www.contoso.com”。

		[NewRequest]
		Subject = "CN=mysite.com"
		Exportable = TRUE
		KeyLength = 2048
		KeySpec = 1
		KeyUsage = 0xA0
		MachineKeySet = True
		ProviderName = "Microsoft RSA SChannel Cryptographic Provider"
		ProviderType = 12

		[EnhancedKeyUsageExtension]
		OID=1.3.6.1.5.5.7.3.1

	有关上述指定的选项以及其他可用选项的详细信息，请参阅 [Certreq 参考文档](http://technet.microsoft.com/zh-cn/library/cc725793.aspx)。

2. 将该文本文件另存为 **myrequest.txt**。

3. 在“开始”屏幕或“开始”菜单中，运行 **cmd.exe**。

4. 从命令提示符处，使用以下命令创建证书请求文件：

		certreq -new \path\to\myrequest.txt \path\to\create\myrequest.csr

	指定在步骤 1 中创建的 **myrequest.txt** 文件的路径，以及要在创建 **myrequest.csr** 文件时使用的路径。

5. 向证书颁发机构提交 **myrequest.csr** 以获取 SSL 证书。这可能涉及上载文件，或在记事本中打开文件并将内容直接粘贴到 Web 表单中。

6. 证书颁发机构向您提供证书 (.CER) 文件后，请将此文件保存到用来生成请求的计算机，然后使用以下命令接受该请求并完成证书生成过程。

		certreq -accept -user mycert.cer

	在这种情况下，从证书颁发机构收到的 **mycert.cer** 证书将用于完成证书的签名。将不创建任何文件；相反，该证书将存储在 Windows 证书存储中。

6. 如果您的 CA 使用中间证书，则必须先安装这些证书，才能在后续步骤中导出证书。通常这些证书作为来自您的 CA 的单独下载提供，并且针对不同的 Web 服务器类型以几种格式提供。选择为 Microsoft IIS 提供的版本。

	下载证书后，在资源管理器中右键单击它，然后选择“安装证书”。使用“证书导入向导”中的默认值，然后继续选择“下一步”，直到完成导入。

7. 若要从证书存储中导出证书，请从“开始”屏幕或“开始”菜单运行 **certmgr.msc**。当“证书管理器”出现时，展开“个人”文件夹，然后选择“证书”。在“颁发给”字段中，查找具有你为其请求了一个证书的自定义域名的条目。在“颁发者”字段中，它应列出用于此证书的证书颁发机构。

	![将证书管理器的图像插入此处][certmgr]

9. 右键单击该证书并选择“所有任务”，然后选择“导出”。在“证书导出向导”中，单击“下一步”，然后选择“是，导出私钥”。单击**“下一步”**。

	![导出私钥][certwiz1]

10. 选择“个人信息交换 - PKCS #12”、“将所有证书包括在证书链中”和“导出所有扩展属性”。单击**“下一步”**。

	![包括所有证书和扩展的属性][certwiz2]

11. 选择“密码”，然后输入并确认该密码。单击**“下一步”**。

	![指定密码][certwiz3]

12. 提供将包含已导出证书的路径和文件名。文件名应使用 **.pfx** 扩展名。单击“下一步”以完成此过程。

	![提供文件路径][certwiz4]

你现在可以将导出的 PFX 文件上载到 Azure 中的应用。

###<a name="bkmk_openssl"></a> 使用 OpenSSL 获取证书

1. 通过从命令行、bash 或终端会话使用以下语句，生成私钥和证书签名请求：

		openssl req -new -nodes -keyout myserver.key -out server.csr -newkey rsa:2048

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

	在此过程完成后，你应该有两个文件：**myserver.key** 和 **server.csr**。**server.csr** 包含证书签名请求。

3. 向证书颁发机构提交 CSR 以获取 SSL 证书。

4. 在你从 CA 获取了某一证书后，将其保存到名为 **myserver.crt** 的文件中。如果你的 CA 以文本格式提供了证书，只需将证书文本粘贴到 **myserver.crt** 文件中。在文本编辑器中查看时，该文件内容应该类似于以下内容：

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

	保存文件。

5. 从命令行、Bash 或终端会话中，使用以下命令将 **myserver.key** 和 **myserver.crt** 转换为 **myserver.pfx**，这是 Azure Web 应用要求的格式：

		openssl pkcs12 -export -out myserver.pfx -inkey myserver.key -in myserver.crt

	在系统提示后，输入一个密码以便保护该 .pfx 文件。

	> [AZURE.NOTE] 如果您的 CA 使用中间证书，则必须先安装这些证书，才能在下一步中导出证书。通常这些证书作为来自您的 CA 的单独下载提供，并且针对不同的 Web 服务器类型以几种格式提供。选择作为 PEM 文件（.pem 文件扩展名）提供的版本。
	> <p>以下命令演示如何创建包含中间证书的 .pfx 文件，这些中间证书包含在 **intermediate-cets.pem** 文件中：
	> <p>`openssl pkcs12 -chain -export -out myserver.pfx -inkey myserver.key -in myserver.crt -certfile intermediate-cets.pem`

	运行此命令后，你应该具有适合与 Azure Web 应用配合使用的 **myserver.pfx** 文件。

###<a name="bkmk_iismgr"></a> 使用 IIS 管理器获取证书

如果你熟悉 IIS 管理器，则可以用它来生成可与 Azure Web 应用配合使用的证书。

1. 使用 IIS 管理器生成要发送给证书颁发机构的 CSR。有关生成 CSR 的详细信息，请参阅[申请 Internet 服务器证书 (IIS 7)][iiscsr]。

2. 向证书颁发机构提交 CSR 以获取 SSL 证书。

3. 使用证书颁发机构供应商提供的证书完成 CSR。有关完成 CSR 的详细信息，请参阅[安装 Internet 服务器证书 (IIS 7)][installcertiis]。

4. 如果您的 CA 使用中间证书，则必须先安装这些证书，才能在下一步中导出证书。通常这些证书作为来自您的 CA 的单独下载提供，并且针对不同的 Web 服务器类型以几种格式提供。选择为 Microsoft IIS 提供的版本。

	下载证书后，在资源管理器中右键单击它，然后选择“安装证书”。使用“证书导入向导”中的默认值，然后继续选择“下一步”，直到完成导入。

4. 从 IIS 管理器导出证书。有关导出证书的详细信息，请参阅[导出服务器证书 (IIS 7)][exportcertiis]。在后面的步骤中将使用导出的文件，以便上载到 Azure 来用于你的应用。

	> [AZURE.NOTE] 在导出过程中，请务必选择选项“是，导出私钥”。<strong></strong>这将在导出的证书中包括私钥。
	><p> 在导出过程中，请务必选择选项“在证书路径中包括所有证书”和“导出所有扩展属性”。这将在导出的证书中包括任何中间证书。

###<a name="bkmk_subjectaltname"></a> 使用 OpenSSL 获取 SubjectAltName 证书

OpenSSL 可用于创建使用 SubjectAltName 扩展以使单个证书支持多个域名的证书请求，但它需要一个配置文件。以下步骤将演示如何创建配置文件，然后使用它来请求证书。

1. 创建一个名为 __sancert.cnf__ 的新文件，并且使用以下代码作为该文件的内容：

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

	请注意以“subjectAltName”开头的行。将当前列出的域名替换为要支持的域名（包含公用名）。例如：

		subjectAltName=DNS:sales.contoso.com,DNS:support.contoso.com,DNS:fabrikam.com

	您不需要更改 commonName\_default 字段，因为系统将会在以下一个步骤中提示您输入您的公用名。

2. 保存 __sancert.cnf__ 文件。

1. 通过使用 sancert.cnf 配置文件生成私钥和证书签名请求。在 bash 或终端会话中，使用以下命令：

		openssl req -new -nodes -keyout myserver.key -out server.csr -newkey rsa:2048 -config sancert.cnf

2. 在系统提示后，输入适当的信息。例如：

 		Country Name (2 letter code) []: US
        State or Province Name (full name) []: Washington
        Locality Name (eg, city) []: Redmond
        Organizational Unit Name (eg, section) []: Azure
        Your common name (eg, domain name) []: www.microsoft.com


	在此过程完成后，你应该有两个文件：**myserver.key** 和 **server.csr**。**server.csr** 包含证书签名请求。

3. 向证书颁发机构提交 CSR 以获取 SSL 证书。

4. 在你从 CA 获取了某一证书后，将其保存到名为 **myserver.crt** 的文件中。如果你的 CA 以文本格式提供了证书，只需将证书文本粘贴到 **myserver.crt** 文件中。在文本编辑器中查看时，该文件内容应该类似于以下内容：

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

	保存文件。

5. 从命令行、Bash 或终端会话中，使用以下命令将 **myserver.key** 和 **myserver.crt** 转换为 **myserver.pfx**，这是 Azure Web 应用要求的格式：

		openssl pkcs12 -export -out myserver.pfx -inkey myserver.key -in myserver.crt

	在系统提示后，输入一个密码以便保护该 .pfx 文件。

	> [AZURE.NOTE] 如果您的 CA 使用中间证书，则必须先安装这些证书，才能在下一步中导出证书。通常这些证书作为来自您的 CA 的单独下载提供，并且针对不同的 Web 服务器类型以几种格式提供。选择作为 PEM 文件（.pem 文件扩展名）提供的版本。
	> <p>以下命令演示如何创建包含中间证书的 .pfx 文件，这些中间证书包含在 **intermediate-cets.pem** 文件中：
	> <p>`openssl pkcs12 -chain -export -out myserver.pfx -inkey myserver.key -in myserver.crt -certfile intermediate-cets.pem`

	运行此命令后，你应该具有适合与 Azure Web 应用配合使用的 **myserver.pfx** 文件。

###<a name="bkmk_selfsigned"></a> 生成自签名证书（仅用于测试）

在某些情况下，您可能想要获取证书以便进行测试，并且将从受信任的 CA 购买证书延迟到生产开始之时。自签名证书可满足此需求。自签名证书是您创建的证书，就像您是证书颁发机构一样进行签名。尽管可以使用此证书来保护某一应用，但在访问该应用时大多数浏览器都将返回错误，因为该证书不是由受信任的 CA 签名的。某些浏览器甚至可能会拒绝允许你查看该应用。

- [使用 makecert 生成自签名证书](#bkmk_ssmakecert)
- [使用 OpenSSL 生成自签名证书](#bkmk_ssopenssl)

####<a name="bkmk_ssmakecert"></a> 使用 makecert 生成自签名证书 ####

您可以通过执行以下步骤从 Windows 系统中删除 Visual Studio 已安装的测试证书：

1. 在“开始”菜单或“开始”屏幕中，搜索“开发人员命令提示符”。最后，右键单击“开发人员命令提示符”并选择“以管理员身份运行”。

	如果你收到“用户帐户控制”对话框，请选择“是”以便继续。

2. 从开发人员命令提示符处，使用以下命令创建新的自签名证书。必须使用应用的 DNS 替代 **serverdnsname**。

		makecert -r -pe -b 01/01/2013 -e 01/01/2014 -eku 1.3.6.1.5.5.7.3.1 -ss My -n CN=serverdnsname -sky exchange -sp "Microsoft RSA SChannel Cryptographic Provider" -sy 12 -len 2048

	此命令将创建日期介于 01/01/2013 和 01/01/2014 之间的适当的证书，并且将在 CurrentUser 证书存储中存储该位置。

3. 在“开始”菜单或“开始”屏幕中，搜索“Windows PowerShell”并启动此应用程序。

4. 从 Windows PowerShell 提示处，使用以下命令导出以前创建的证书：

		$mypwd = ConvertTo-SecureString -String "password" -Force -AsPlainText
		get-childitem cert:\currentuser\my -dnsname serverdnsname | export-pfxcertificate -filepath file-to-export-to.pfx -password $mypwd

	这会将指定的密码作为安全字符串存储于 $mypwd 中，然后通过使用 **dnsname** 参数指定的 DNS 名称查找证书，并且导出到 **filepath** 参数指定的文件。包含密码的安全字符串用于保护导出的文件。

####<a name="bkmk_ssopenssl"></a>使用 OpenSSL 生成自签名证书 ####

1. 创建一个名为 **serverauth.cnf** 的新文档，并且使用以下代码作为该文件的内容：

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

	这将指定生成可由 Azure Web 应用使用的 SSL 证书所需的配置设置。

2. 通过从命令行、bash 或终端会话使用以下语句，生成新的自签名证书：

		openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout myserver.key -out myserver.crt -config serverauth.cnf

	这将使用在 **serverauth.cnf** 文件中指定的配置设置创建一个新证书。

3. 若要将该证书导出到可上载到 Azure Web 应用中的应用的 .PFX 文件，请使用以下命令：

		openssl pkcs12 -export -out myserver.pfx -inkey myserver.key -in myserver.crt

	在系统提示后，输入一个密码以便保护该 .pfx 文件。

	可使用此命令生成的 **myserver.pfx** 来出于测试目的保护你的应用。

##<a name="bkmk_standardmode"></a> 2\.配置标准定价层

为自定义域启用 HTTPS 只适用于 Azure Web 应用中的**标准**层。请按照以下步骤将你的 App Service 计划切换到**标准**层。

> [AZURE.NOTE] 将应用从**免费**层切换到**标准**层之前，你应该删除订阅已有的支出上限，否则如果你在计费周期结束之前达到你的上限，可能会出现你的应用变得不可用的风险。有关共享和**标准**层的详细信息，请参阅[定价详细信息][pricing]。

1. 在浏览器中，打开[经典管理门户][portal]。

2. 在“Web Apps”选项卡中，单击你的网站的名称。

	![选择网站][website]

3. 单击“缩放”选项卡。

	![“缩放”选项卡][scale]

4. 在“常规”部分中，通过单击“标准”设置应用服务计划定价层。

	![选定的标准模式][standard]

5. 单击“保存”。在系统提示后，单击“是”。
	> [AZURE.NOTE]如果出现“为 Web 应用‘&lt;应用名称&gt;’配置缩放失败”错误，你可以使用详细信息按钮来了解详细信息。可能会出现“可用的标准实例服务器不足，无法满足此请求。”错误。如果收到此错误，请联系 [Azure 支持人员](/support/contact/)。

##<a name="bkmk_configuressl"></a> 3\.在应用中配置 SSL

在执行本部分中的这些步骤之前，必须将某个自定义域名与你的应用相关联。有关详细信息，请参阅[为 Web 应用配置自定义域名][customdomain]。

1.	在你的浏览器中，打开 [Azure 经典管理门户][portal]。

2. 在“Web Apps”选项卡中，单击站点名称，然后选择“配置”选项卡。

	![“配置”选项卡][configure]

3. 在“证书”部分中，单击“上载证书”

	![上载证书][uploadcert]

4. 通过使用“上载证书”对话框，选择以前使用 IIS 管理器或 OpenSSL 创建的 .pfx 证书文件。如果有，指定用于保护 .pfx 文件的密码。最后，单击**复选标记**以上载证书。
	
	![“上载证书”对话框][uploadcertdlg]

5. 在“配置”选项卡的“ssl 绑定”部分中，使用下拉菜单选择要使用 SSL 保护的域名，然后选择要使用的证书。你还可以选择是使用[服务器名称指示][sni] (SNI) 还是使用基于 IP 的 SSL。

	![SSL 绑定][sslbindings]
	
	* 基于 IP 的 SSL 通过将服务器的专用公共 IP 地址映射到域名，将证书与域名相关联。这要求与您的服务相关联的每个域名（contoso.com、fabricam.com 等）都具有专用的 IP 地址。这是将 SSL 证书与某一 Web 服务器相关联的传统方法。

	* 基于 SNI 的 SSL 是对 SSL 和[传输层安全性][tls] (TLS) 的扩展，它允许多个域共享相同的 IP 地址，并且对于每个域都有单独的安全证书。当前常用的大多数浏览器（包括 Internet Explorer、Chrome、Firefox 和 Opera）都支持 SNI，但是，较旧的浏览器可能不支持 SNI。有关 SNI 的详细信息，请参阅 Wikipedia 上的文章[服务器名称指示][sni]。

6. 单击“保存”以保存更改和启用 SSL。

> [AZURE.NOTE] 如果你选择“基于 IP 的 SSL”，并且你的自定义域使用 A 记录进行配置，则必须执行以下附加步骤：
><p> 1. 配置基于 IP 的 SSL 绑定后，将会向你的应用分配专用 IP 地址。你可以在应用的“仪表板”页上的“速览”部分中找到此 IP 地址。它将会作为“虚拟 IP 地址”列出：  
><p>     ![虚拟 IP 地址](./media/configure-ssl-web-site/staticip.png)
><p>     请注意，此 IP 地址将与以前用于配置您的域的 A 记录的虚拟 IP 地址不同。如果将您配置为使用基于 SNI 的 SSL，或未将您配置为使用 SSL，则不会为此条目列出任何地址。
><p> 2. 通过使用您的域名注册机构所提供的工具，修改您的自定义域名的 A 记录以指向上一步中的 IP 地址。

此时，你应该能够使用 `HTTPS://` 而不是 `HTTP://` 访问你的应用，以便验证证书是否已正确配置。

##<a name="bkmk_enforce"></a> 4\.对应用强制实施 HTTPS

Azure Web 应用*不* 强制实施 HTTPS。访问者可能仍使用 HTTP 访问应用，这可能危及应用的安全。如果想为应用强制实施 HTTPS，可以使用 **URL 重写**模块。URL 重写模块包含在 Azure Web 应用中，你能够用它定义将请求传递给应用程序之前应用于传入请求的规则。**该模块可用于以 Azure 支持的任何编程语言编写的应用程序。**

> [AZURE.NOTE] .NET MVC 应用程序应使用 [RequireHttps](http://msdn.microsoft.com/zh-cn/library/system.web.mvc.requirehttpsattribute.aspx) 筛选器而不是 URL 重写。有关使用 RequireHttps 的详细信息，请参阅[将安全的 ASP.NET MVC 5 应用部署到 Web 应用](/documentation/articles/web-sites-dotnet-deploy-aspnet-mvc-app-membership-oauth-sql-database/)。
> <p>有关使用其他编程语言和框架以编程方式重定向请求的信息，请参阅这些技术的文档。

URL 重写规则在存储于应用程序根目录中的 **web.config** 文件内定义。以下示例包含可强制所有传入流量使用 HTTPS 的基本 URL 重写规则。

<a name="example"></a>**URL 重写示例 Web.Config**

	<?xml version="1.0" encoding="UTF-8"?>
	<configuration>
	  <system.webServer>
	    <rewrite>
	      <rules>
	        <rule name="Force HTTPS" enabled="true">
	          <match url="(.*)" ignoreCase="false" />
	          <conditions>
	            <add input="{HTTPS}" pattern="off" />
	          </conditions>
	          <action type="Redirect" url="https://{HTTP_HOST}/{R:1}" appendQueryString="true" redirectType="Permanent" />
	        </rule>
	      </rules>
	    </rewrite>
	  </system.webServer>
	</configuration>

此规则的工作方式是当用户使用 HTTP 请求某个页面时，返回 HTTP 状态码 301（永久重定向）。301 将请求重定向到访问者请求的同一个 URL，但使用 HTTPS 来替换请求的 HTTP 部分。例如，HTTP://contoso.com 将重定向到 HTTPS://contoso.com。

> [AZURE.NOTE] 如果应用程序是用 **Node.js**、**PHP**、**Python Django** 或 **Java** 编写的，则它可能不包含 web.config 文件。但是，托管在 Azure Web 应用中时，**Node.js**、**Python Django** 和 **Java** 确实都会使用 web.config，而 Azure 在部署期间自动创建此文件，因此你永远看不到此文件。如果在应用程序中添加一个 web.config 文件，则它将覆盖 Azure 自动生成的文件。

###.NET

对于 .NET 应用程序，请修改应用程序的 web.config 文件，并将[示例](#example)中的 **&lt;rewrite>** 节添加到 **&lt;system.WebServer>** 节。

如果 web.config 文件已经包含 **&lt;rewrite>** 节，请将[示例](#example)中的 **&lt;rule>** 添加为 **&lt;rules>** 节中的第一个条目。

###PHP

对于 PHP 应用程序，只需将[示例](#example)另存为应用程序根目录中的 web.config 文件，然后将应用程序重新部署到应用即可。

###Node.js、Python Django 和 Java

如果 Node.js、Python Django 和 Java 应用程序未提供 web.config 文件，则此文件将自动创建，但因为此文件是在部署过程中创建的，因此它只存在于服务器上。自动生成的文件包含告知 Azure 如何托管应用程序的设置。

若要从应用检索和修改自动生成的文件，请遵循以下步骤。

1. 使用 FTP 下载文件（请参阅[通过 FTP 上载/下载文件和收集诊断日志](http://blogs.msdn.com/b/avkashchauhan/archive/2012/06/19/windows-azure-website-uploading-downloading-files-over-ftp-and-collecting-diagnostics-logs.aspx)）。

2. 将文件添加到应用程序的根目录。

3. 使用以下信息添加重写规则。

	* **Node.js 和 Python Django**

		为 Node.js 和 Python Django 应用程序生成的 web.config 文件已包含 **&lt;rewrite>** 节，其中包含应用正常运行所需的 **&lt;rule>** 条目。若要强制应用使用 HTTPS，请将示例中的 **&lt;rule>** 添加为 **&lt;rules>** 节中的第一个条目。这会在保留其他规则不变的情况下强制 HTTPS。

	* **Java**

		在使用 Apache Tomcat 的 Java 应用程序中，其 web.config 文件不包含 **&lt;rewrite>** 节，因此必须将示例中的 **&lt;rewrite>** 节添加到 **&lt;system.webServer>** 节。

4. 把它放回文件夹 /site/wwwroot

部署包含重写规则以实施 HTTPS 的 web.config 后，该文件应立即生效并将所有请求重定向到 HTTPS。

有关 IIS URL 重写模块的详细信息，请参阅 [URL 重写](http://www.iis.net/downloads/microsoft/url-rewrite)文档。

## 更多资源 ##
- [Azure 信任中心](/support/trust-center/security/)
- [Azure 网站中解锁的设置选项](https://azure.microsoft.com/zh-cn/blog/2014/01/28/more-to-explore-configuration-options-unlocked-in-windows-azure-web-sites/)
- [启用诊断日志记录](/documentation/articles/web-sites-enable-diagnostic-log/)
- [在 Azure 中配置 Web 应用](/documentation/articles/web-sites-configure/)
- [Azure 经典管理门户](https://manage.windowsazure.cn)

[customdomain]: /documentation/articles/web-sites-custom-domain-name/
[iiscsr]: http://technet.microsoft.com/zh-cn/library/cc732906(WS.10).aspx
[cas]: http://go.microsoft.com/fwlink/?LinkID=269988
[installcertiis]: http://technet.microsoft.com/zh-cn/library/cc771816(WS.10).aspx
[exportcertiis]: http://technet.microsoft.com/zh-cn/library/cc731386(WS.10).aspx
[openssl]: http://www.openssl.org/
[portal]: https://manage.windowsazure.cn/
[tls]: http://zh.wikipedia.org/wiki/Transport_Layer_Security
[staticip]: ./media/configure-ssl-web-site/staticip.png
[website]: ./media/configure-ssl-web-site/sslwebsite.png
[scale]: ./media/configure-ssl-web-site/sslscale.png
[standard]: ./media/configure-ssl-web-site/sslreserved.png
[pricing]: /pricing/overview/
[configure]: ./media/configure-ssl-web-site/sslconfig.png
[uploadcert]: ./media/configure-ssl-web-site/ssluploadcert.png
[uploadcertdlg]: ./media/configure-ssl-web-site/ssluploaddlg.png
[sslbindings]: ./media/configure-ssl-web-site/sslbindings.png
[sni]: http://en.wikipedia.org/wiki/Server_Name_Indication
[certmgr]: ./media/configure-ssl-web-site/waws-certmgr.png
[certwiz1]: ./media/configure-ssl-web-site/waws-certwiz1.png
[certwiz2]: ./media/configure-ssl-web-site/waws-certwiz2.png
[certwiz3]: ./media/configure-ssl-web-site/waws-certwiz3.png
[certwiz4]: ./media/configure-ssl-web-site/waws-certwiz4.png

<!---HONumber=Mooncake_0328_2016-->