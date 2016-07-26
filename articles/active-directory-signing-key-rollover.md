<properties
	pageTitle="Azure AD 中的签名密钥滚动更新 | Azure"
	description="本文介绍 Azure Active Directory 的签名密钥滚动更新最佳实践"
	services="active-directory"
	documentationCenter=".net"
	authors="priyamohanram"
	manager="mbaldwin"
	editor=""/>

<tags
	ms.service="active-directory"
	ms.date="05/31/2016"
	wacn.date="07/26/2016"/>

# Azure Active Directory 中的签名密钥滚动更新

本主题介绍你需要了解的有关 Azure Active Directory (Azure AD) 中用来为安全令牌签名的公钥的信息。必须注意的是，这些密钥每 6 周按计划滚动更新一次。在紧急情况下，密钥可以远远不到 6 周就更改一次。所有使用 Azure AD 的应用程序应该都能以编程方式处理密钥滚动更新过程。请继续阅读以了解密钥的工作原理，以及如何更新应用程序以处理密钥滚动更新。

## Azure AD 中的签名密钥概述

Azure AD 使用基于行业标准构建的公钥加密，在它自己和使用它的应用程序之间建立信任关系。具体而言，它的工作方式如下：Azure AD 使用包含公钥和私钥对的签名密钥。当用户登录到使用 Azure AD 进行身份验证的应用程序时，Azure AD 将创建一个包含用户相关信息的安全令牌。此令牌由 Azure AD 使用其私钥进行签名，然后会发送回应用程序。若要验证该令牌是否有效且确实来自 Azure AD，应用程序必须使用由 Azure AD 公开，包含在租户的联合元数据文档中的公钥来验证令牌的签名。此公钥 – 以及衍生它的签名密钥 – 是由 Azure AD 中所有租户使用的同一个密钥。

为安全起见，Azure AD 的公钥每隔 6 周滚动更新一次。在紧急情况下，该密钥可以远远不到 6 周就滚动更新一次。任何与 Azure AD 集成的应用程序都应准备好处理密钥滚动更新事件，而不管滚动更新可能发生的频率是多少。根据你创建应用程序的时间以及生成应用程序时使用的身份验证库，你的应用程序不一定包含处理密钥滚动更新所必需的逻辑。如果你的应用程序在未包含该逻辑的情况下尝试使用过期的公钥来验证令牌上的签名，则登录请求将失败。

因为密钥可能在任何时间滚动更新，联合元数据文档中始终有多个有效公钥可用。应用程序应准备使用该文档中指定的任何密钥，因为可能很快会对一个密钥进行滚动更新，而另一个密钥可能会取而代之，依此类推。建议让应用程序将这些密钥缓存到数据库或配置文件中，以提高在登录过程中与 Azure AD 的通信效率并使用不同密钥来快速验证令牌。

## 如何使用密钥滚动更新逻辑更新应用程序

应用程序如何处理密钥滚动更新取决于各种变量，例如所使用的标识框架、框架版本或应用程序类型。下面各部分将说明如何更新最常见的应用程序类型和配置。你也可以按照步骤进行操作以确保该逻辑正常工作。

### 使用 Visual Studio 2013 创建的 Web 应用程序

如果你的应用程序是使用 Visual Studio 2013 中的 Web 应用程序模板生成的，并且你从“更改身份验证”菜单中选择了“组织帐户”，则应用程序已包含处理密钥滚动更新所必需的逻辑。此逻辑将组织的唯一标识符和签名密钥信息存储到与项目关联的两个数据库表中。你可以在项目的 Web.config 文件中找到数据库的连接字符串。

如果你已手动将身份验证添加到解决方案，则应用程序未包含必要的密钥滚动更新逻辑。你需要自己编写该逻辑，或者执行手动检索最新密钥并更新应用程序中的步骤。

以下步骤将帮助你验证该逻辑是否在应用程序中正常工作。

1. 在 Visual Studio 2013 中，打开解决方案，然后单击右侧窗口上的“服务器资源管理器”选项卡。
2. 依次展开“数据连接”、DefaultConnection 和“表”。找到 **IssuingAuthorityKeys** 表，右键单击它，然后单击“显示表数据”。
3. 在 **IssuingAuthorityKeys** 表中将至少有一行与密钥的指纹值相对应。删除该表中的所有行。
4. 右键单击“Tenants”表，然后单击“显示表数据”。
5. 在 **Tenants** 表中，至少有一行与唯一的目录租户标识符相对应。删除该表中的所有行。如果未同时删除 **IssuingAuthorityKeys** 和 **Tenants** 表中的行，则运行时你将收到错误。
6. 构建并运行应用程序。登录到你的帐户后，可以停止应用程序。
7. 返回到“服务器资源管理器”，查看 **IssuingAuthorityKeys** 和 **Tenants** 表中的值。你会注意到，已自动使用联合元数据文档中的相应信息对这两个表进行重新填充。

### 使用 Visual Studio 2012 创建的 Web 应用程序

如果你的应用程序是在 Visual Studio 2012 中生成的，则你可能已使用标识和访问工具配置了应用程序。你还可能会使用[验证颁发者名称注册表 (VINR)](https://msdn.microsoft.com/library/dn205067.aspx)。VINR 负责维护受信任标识提供程序 (Azure AD) 以及验证它们颁发令牌时所使用密钥的相关信息。使用 VINR 还可轻松地自动更新存储在 Web.config 文件中的密钥信息，具体方法是：下载与你的目录关联的最新联合元数据文档，使用最新文档检查配置是否过期，然后根据需要更新应用程序以使用新密钥。

如果你是使用 Microsoft 提供的代码示例或演练文档创建的应用程序，则密钥滚动更新逻辑已包含在你的项目中。你会注意到下面的代码已存在于你的项目中。如果应用程序尚未包含该逻辑，请按照下面的步骤添加该逻辑，并验证该逻辑是否正常工作。

1. 在“解决方案资源管理器”中，添加对相应项目的 **System.IdentityModel** 程序集的引用。
2. 打开 **Global.asax.cs** 文件并添加以下 using 指令：
		
		using System.Configuration;
		using System.IdentityModel.Tokens;

3. 在 **Global.asax.cs** 文件中添加以下方法：

		protected void RefreshValidationSettings()
		{
		    string configPath = AppDomain.CurrentDomain.BaseDirectory + "\" + "Web.config";
		    string metadataAddress =
		                  ConfigurationManager.AppSettings["ida:FederationMetadataLocation"];
		    ValidatingIssuerNameRegistry.WriteToConfig(metadataAddress, configPath);
		}

4. 在 **Global.asax.cs** 中的 **Application\_Start()** 方法内调用 **RefreshValidationSettings()** 方法，如下所示：
		
		protected void Application_Start()
		{
		    AreaRegistration.RegisterAllAreas();
		    ...
		    RefreshValidationSettings();
		}


执行这些步骤后，系统将使用联合元数据文档中的最新信息（包括最新密钥）更新应用程序的 Web.config。每次在 IIS 中回收应用程序池时，都会进行此更新；默认情况下，IIS 设置为每 29 个小时回收一次应用程序。

遵循以下步骤验证密钥滚动更新逻辑是否正常工作。

1. 验证你的应用程序正在使用上面的代码后，打开 **Web.config** 文件并导航到 **\<issuerNameRegistry\>** 块中，特别是要找到以下几行：

		<issuerNameRegistry type="System.IdentityModel.Tokens.ValidatingIssuerNameRegistry, System.IdentityModel.Tokens.ValidatingIssuerNameRegistry">
		        <authority name="https://sts.windows.net/ec4187af-07da-4f01-b18f-64c2f5abecea/">
		          <keys>
		            <add thumbprint="3A38FA984E8560F19AADC9F86FE9594BB6AD049B" />
		          </keys>

2. 在 **\<add thumbprint=””\>** 设置中，通过将任一字符替换为不同的字符来更改指纹值。保存 **Web.config** 文件。

3. 生成然后运行应用程序。如果你能完成登录过程，则应用程序将通过从你的目录的联合元数据文档下载所需的信息来成功地更新密钥。如果你在登录时遇到问题，请通过阅读 [Adding Sign-On to Your Web Application Using Azure AD（使用 Azure AD 将登录名添加到 Web 应用程序中）](https://github.com/Azure-Samples/active-directory-dotnet-webapp-openidconnect)主题，或下载并检查以下代码示例来确保你应用程序中的更改是正确的：[Multi-Tenant Cloud Application for Azure Active Directory（用于 Azure Active Directory 的多租户云应用程序）](https://code.msdn.microsoft.com/multi-tenant-cloud-8015b84b)。


### 使用 Visual Studio 2008 或 2010 以及适用于 .NET 3.5 的 Windows Identity Foundation (WIF) v1.0 创建的 Web 应用程序

如果你在 WIF v1.0 中构建应用程序，则系统未提供相应的机制来自动刷新应用程序的配置以使用新密钥。更新密钥的最简单方法是使用 WIF SDK 中包含的 FedUtil 工具，该工具可以检索最新的元数据文档并更新你的配置。下面提供了相关说明。或者，也可以执行以下操作之一：

- 按照下面的“手动检索最新密钥并更新应用程序”部分中的说明操作，并编写逻辑以通过编程方式执行这些步骤。
- 将应用程序更新到 .NET 4.5，该版本包括位于系统命名空间中的 WIF 的最新版本。然后，你可以使用[验证颁发者名称注册表 (VINR)](https://msdn.microsoft.com/library/dn205067.aspx) 来执行应用程序配置的自动更新。


1. 请确认你已在开发计算机上为 Visual Studio 2008 或 2010 安装了 WIF v1.0 SDK。如果尚未安装，可以[从此处下载](https://www.microsoft.com/zh-cn/download/details.aspx?id=4451)。
2. 在 Visual Studio 中打开解决方案，然后右键单击相应的项目并选择“更新联合元数据”。如果此选项不可用，则表示 FedUtil 和/或 WIF v1.0 SDK 尚未安装。
3. 系统提示时，请选择“更新”以开始更新联合元数据。如果你有权访问托管应用程序的服务器环境，则可以选择使用 FedUtil 的[自动元数据更新计划程序](https://msdn.microsoft.com/library/ee517272.aspx)。
4. 单击“完成”以完成更新过程。

### 使用 JSON Web 令牌 (JWT) 的 Web API

如果你的应用程序在为请求授权时使用 Azure AD 所颁发的 JWT 令牌来调用 Web API，则验证 JWT 令牌的方式与验证登录请求的方式相同：使用 Azure AD 的公钥来验证签名。Web API 应用程序需要准备好处理密钥滚动更新，因为这些应用程序最终使用同一 X509 证书来为令牌签名。

如果你在 Visual Studio 2013 中使用 Web API 模板创建了 Web API 应用程序，然后从“更改身份验证”菜单中选择了“组织帐户”，则应用程序中已包含必需的逻辑。如果你已手动配置身份验证，请参阅下面的说明，了解如何配置 Web API 来自动更新其密钥信息。

以下代码片段演示如何从联合元数据文档获取最新密钥，然后使用 [JWT 令牌处理程序](https://msdn.microsoft.com/library/dn205065.aspx)来验证令牌。该代码片段假设你使用自己的缓存机制来持久保存密钥（以便验证将来从 Azure AD 获取的令牌），无论是将它保存在数据库中、配置文件中，还是保存在其他位置。
		
		
		using System;
		using System.Collections.Generic;
		using System.Linq;
		using System.Text;
		using System.Threading.Tasks;
		using System.IdentityModel.Tokens;
		using System.Configuration;
		using System.Security.Cryptography.X509Certificates;
		using System.Xml;
		using System.IdentityModel.Metadata;
		using System.ServiceModel.Security;
		using System.Threading;
		
		namespace JWTValidation
		{
		    public class JWTValidator
		    {
		        private string MetadataAddress = "[Your Federation Metadata document address goes here]";
		
		        // Validates the JWT Token that's part of the Authorization header in an HTTP request.
		        public void ValidateJwtToken(string token)
		        {
		            JwtSecurityTokenHandler tokenHandler = new JwtSecurityTokenHandler()
		            {
		                // Do not disable for production code
		                CertificateValidator = X509CertificateValidator.None
		            };
		
		            TokenValidationParameters validationParams = new TokenValidationParameters()
		            {
		                AllowedAudience = "[Your App ID URI goes here, as registered in the Azure Classic Portal]",
		                ValidIssuer = "[The issuer for the token goes here, such as https://sts.windows.net/68b98905-130e-4d7c-b6e1-a158a9ed8449/]",
		                SigningTokens = GetSigningCertificates(MetadataAddress)
		
		                // Cache the signing tokens by your desired mechanism
		            };
		
		            Thread.CurrentPrincipal = tokenHandler.ValidateToken(token, validationParams);
		        }
		
		        // Returns a list of certificates from the specified metadata document.
		        public List<X509SecurityToken> GetSigningCertificates(string metadataAddress)
		        {
		            List<X509SecurityToken> tokens = new List<X509SecurityToken>();
		
		            if (metadataAddress == null)
		            {
		                throw new ArgumentNullException(metadataAddress);
		            }
		
		            using (XmlReader metadataReader = XmlReader.Create(metadataAddress))
		            {
		                MetadataSerializer serializer = new MetadataSerializer()
		                {
		                    // Do not disable for production code
		                    CertificateValidationMode = X509CertificateValidationMode.None
		                };
		
		                EntityDescriptor metadata = serializer.ReadMetadata(metadataReader) as EntityDescriptor;
		
		                if (metadata != null)
		                {
		                    SecurityTokenServiceDescriptor stsd = metadata.RoleDescriptors.OfType<SecurityTokenServiceDescriptor>().First();
		
		                    if (stsd != null)
		                    {
		                        IEnumerable<X509RawDataKeyIdentifierClause> x509DataClauses = stsd.Keys.Where(key => key.KeyInfo != null && (key.Use == KeyType.Signing || key.Use == KeyType.Unspecified)).
		                                                             Select(key => key.KeyInfo.OfType<X509RawDataKeyIdentifierClause>().First());
		
		                        tokens.AddRange(x509DataClauses.Select(token => new X509SecurityToken(new X509Certificate2(token.GetX509RawData()))));
		                    }
		                    else
		                    {
		                        throw new InvalidOperationException("There is no RoleDescriptor of type SecurityTokenServiceType in the metadata");
		                    }
		                }
		                else
		                {
		                    throw new Exception("Invalid Federation Metadata document");
		                }
		            }
		            return tokens;
		        }
		    }
		}


### 手动检索最新密钥并更新应用程序

如果你的应用程序类型或平台目前不支持自动更新密钥的机制，你可以执行以下步骤：

1. 在 Web 浏览器中，转到 https://manage.windowsazure.com， 登录到你的帐户，然后单击左侧菜单中的 Active Directory 图标。
2. 单击注册应用程序的目录，然后单击命令栏上的“查看终结点”链接。
3. 从单一登录和目录终结点的列表中，复制“联合元数据文档”链接。
4. 在浏览器中打开新的选项卡，并转到你刚复制的 URL。你将看到联合元数据 XML 文档的内容。有关此文档的详细信息，请参阅“Federation Metadata”（联合元数据）主题。
5. 为了更新应用程序以使用新密钥，请找到每个 **\<RoleDescriptor\>** 块，然后复制每个块的 **\<X509Certificate\>** 元素的值。例如：
		
		<RoleDescriptor xmlns:fed="http://docs.oasis-open.org/wsfed/federation/200706" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" protocolSupportEnumeration="http://docs.oasis-open.org/wsfed/federation/200706" xsi:type="fed:SecurityTokenServiceType">
		      <KeyDescriptor use="signing">
		            <KeyInfo xmlns="http://www.w3.org/2000/09/xmldsig#">
		                <X509Data>
		                    <X509Certificate>MIIDPjC…BcXWLAIarZ</X509Certificate>

6. 在复制 **\<X509Certificate\>** 元素的值之后，打开纯文本编辑器并粘贴该值。请务必删除任何尾随空格，然后使用 **.cer** 扩展名保存该文件。

现已创建用作 Azure AD 公钥的 X509 证书。使用此证书的详细信息（如指纹和过期日期），可以通过手动或编程方式检查应用程序当前使用的证书和指纹是否有效。

<!---HONumber=AcomDC_0718_2016-->