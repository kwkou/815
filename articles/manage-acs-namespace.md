<properties linkid="manage-services-manage-acs" urlDisplayName="Manage ACS" pageTitle="Access Control Service - Azure service management" metaKeywords="" description="Learn how to manage your Azure Access Control Service (ACS) using certificates and keys." metaCanonical="" services="active-directory" documentationCenter="" title="Managing Your ACS Namespace" authors="" solutions="" manager="" editor="" />

# 管理 ACS 命名空间

本主题概述了为确保使用 Azure Acccess Control 服务 (ACS) 的应用程序不间断地持续正常工作，而建议你定期执行的管理任务。这些管理任务如下所示：

1.  重要说明：跟踪由 ACS 命名空间、信赖方应用程序、服务标识、标识提供程序和 ACS 管理服务帐户使用的证书、密钥和密码的到期情况并执行其滚动更新。有关详细信息，请参阅下面的“证书和密钥管理指南”。

2.  查看你的标识提供程序、服务标识、规则和门户管理员，并删除过期的项目。

有关 ACS 的详细信息，请参阅 [Access Control 服务 2.0][Access Control 服务 2.0]。

## 证书和密钥管理指南

为了安全起见，请确保 ACS 中使用的证书和密钥会过期。跟踪过期日期很重要，以便能够续订这些证书和密钥。

用于滚动更新令牌签名（对称密钥或 X.509 证书）或令牌解密证书的高级步骤为：

1.  在 ACS 中，在将过期的现有证书或密钥旁边将新的证书或密钥配置为“辅助”密钥。
2.  告知使用服务的合作伙伴，他们需要在某个截止日期之前更新其相应的密钥。
3.  合作伙伴应为其信赖方或标识提供程序更新相应的证书或密钥。例如，为包含新的令牌签名验证证书的 ACS 命名空间导入更新的 WS 联合身份验证元数据，或手动在应用程序配置中配置对称密钥。
4.  更新所有应用程序后（或截止日期已过后），在 ACS 配置中将新的证书或密钥标记为主要。
5.  在合理的宽限期过后，将从 ACS 配置中删除旧证书或密钥。

用于滚动更新令牌加密证书的高级步骤为：

1.  你（或你的合作伙伴）在信赖方应用程序中更新用于令牌解密的相应的证书或密钥。
2.  在 ACS 中，在将过期的现有证书旁边配置新的加密证书。
3.  删除旧的加密证书。

用于滚动更新服务标识或管理服务密钥的高级步骤为：

1.  在 ACS 中，在将过期的现有证书或密钥旁边配置新的证书或密钥。
2.  你（或你的合作伙伴）在客户端应用程序中更新用于令牌请求的相应的证书或密钥。
3.  更新所有客户端后（或合理的宽限期过后），将删除旧的证书或密钥。

当证书或密钥过期后，ACS 将无法发出阻止信赖方正常运营的令牌。ACS 将忽略过期的证书和密钥，这实际上会导致出现异常，就好像起初未配置证书或密钥一样。在下面部分中，你将了解有关 ACS 托管的每个证书和密钥的信息，如何续订证书和密钥以及如何识别证书和密钥是否已过期且需要续订。

-   使用 ACS 管理门户中的“证书和密钥”部分以管理与服务命名空间和信赖方应用程序相关的证书和密钥。有关这些凭据类型的详细信息，请参阅[证书和密钥][证书和密钥]。
-   使用 ACS 管理门户中的“服务标识”部分以管理与服务标识相关的凭据（证书、密钥或密码）。有关服务标识的详细信息，请参阅[服务标识][服务标识]。
-   使用 ACS 管理门户中的“管理服务”部分以管理与 ACS 管理服务帐户相关的凭据（凭据、密钥或密码）。有关 ACS 管理服务的详细信息，请参阅 [ACS 管理服务][ACS 管理服务]。

有一些证书和密钥类型在 ACS 管理门户中不可见。具体而言，对于 WS 联合身份验证标识提供程序（如 AD FS）而言，你必须主动检查标识提供程序使用的证书的有效性。目前，通过 WS 联合身份验证标识提供程序的元数据提供的证书在 ACS 管理门户上不可见。若要验证证书的有效性，你必须使用管理服务检查 [IdentityProviderKey][IdentityProviderKey] 的 StartDate 和 EndDate 属性的有效日期和过期日期。当证书或密钥因到期而失效时，ACS 将开始引发异常，即特定于证书或密钥的 [ACS 错误代码][ACS 错误代码]。有关特定的错误代码，请参阅以下部分。

你可以使用 [ACS 管理服务][ACS 管理服务]以编程方式更新证书和密钥。请考虑查看可作为 [代码示例：管理服务][代码示例：管理服务]的一部分下载的 KeyManagement 代码示例。

## 可用证书和密钥

以下列表显示了 ACS 中使用的可用证书和密钥，并且必须跟踪其到期日期：

-   令牌签名证书
-   令牌签名密钥
-   令牌加密证书
-   令牌解密证书
-   服务标识凭据
-   ACS 管理服务帐户凭据
-   标识提供程序签名和加密证书

本主题的其余部分详细介绍了每个证书和密钥。

## 令牌签名证书

ACS 会对自己颁发的所有安全令牌进行签名。在你生成使用 ACS 颁发的 SAML 令牌的应用程序时，将使用 X.509 证书进行签名。

你可以通过 ACS 管理门户的“证书和密钥”部分管理令牌签名证书。

**管理令牌签名证书**

1.  打开 Internet 浏览器并访问 [Azure 管理门户] (<http://manage.windowsazure.cn>)。

2.  使用 Windows Live ID 登录到该网站。如果你没有 Windows Live ID，则请单击“注册”以自行创建该 ID。

3.  使用你的 Windows Live ID 登录后，你将重定向到管理门户页。在该页的左下方，单击“Service Bus 和 Access Control”。

    ![][0]

4.  若要启动 ACS 管理门户，请单击左侧树中的“Access Control”，选择要配置的 ACS 服务命名空间，然后从该页顶部的工具栏中单击“Access Control 服务”按钮。

    ![][1]

    此时，你的屏幕应与下图类似：

    ![][2]

5.  在“服务设置”部分的下方，单击左侧树中的“证书和密钥”。

    ![][3]

    此时，你的屏幕应与下图类似：

    ![][4]

6.  在“令牌签名”部分的下方，使用“添加”按钮以在将过期的现有证书旁边将 ACS 中的新证书配置为“辅助”密钥。

7.  告知使用服务的合作伙伴，他们需要在某个截止日期之前更新其相应的密钥。

8.  合作伙伴应为其信赖方或标识提供程序更新相应的证书。例如，为包含新的令牌签名验证证书的 ACS 命名空间导入更新的 WS 联合身份验证元数据，或手动在应用程序配置中配置对称密钥。

9.  更新所有应用程序后（或截止日期已过后），在 ACS 配置中将新的证书标记为主要。

10. 在合理的宽限期过后，使用“证书和密钥”页的“令牌签名”部分下方的“删除”按钮可从 ACS 配置中删除旧证书。

有关详细信息，请参阅[证书和密钥][证书和密钥]。

在签名证书过期后，如果你尝试请求令牌，将收到以下错误：


<table><tr><td><b>错误代码</b></td>
<td><b>消息</b></td>
<td><b>修复消息所需的操作</b></td>
</tr>
<tr>
<td>ACS50004</td>
<td>未配置主要 X.509 签名证书。SAML 需要签名证书。</td>
<td>如果所选的信赖方将 SAML 用作令牌类型，请确保为该信赖方或服务命名空间配置有效的 X.509 证书。证书必须设置为主证书且必须在其有效期内。</td></tr>
</table> 

## 令牌签名密钥

ACS 会对自己颁发的所有安全令牌进行签名。在你生成使用 ACS 颁发的 SWT 令牌的应用程序时，将使用 256 位对称签名密钥。

你可以通过 ACS 管理门户的“证书和密钥”部分管理令牌签名密钥。

**管理令牌签名密钥**

1.  打开 Internet 浏览器并访问 [Azure 管理门户] (<http://manage.windowsazure.cn>)。

2.  使用 Windows Live ID 登录到该网站。如果你没有 Windows Live ID，则请单击“注册”以自行创建该 ID。

3.  使用你的 Windows Live ID 登录后，你将重定向到管理门户页。在该页的左下方，单击“Service Bus 和 Access Control”。

    ![][0]

4.  若要启动 ACS 管理门户，请单击左侧树中的“Access Control”，选择要配置的 ACS 服务命名空间，然后从该页顶部的工具栏中单击“Access Control 服务”按钮。

    ![][1]

    此时，你的屏幕应与下图类似：

    ![][2]

5.  在“服务设置”部分的下方，单击左侧树中的“证书和密钥”。

    ![][3]

    此时，你的屏幕应与下图类似：

    ![][4]

6.  在“令牌签名”部分的下方，使用“添加”按钮以在将过期的现有密钥旁边将 ACS 中的新密钥配置为“辅助”密钥。

7.  告知使用服务的合作伙伴，他们需要在某个截止日期之前更新其相应的密钥。

8.  合作伙伴应为其信赖方或标识提供程序更新相应的密钥。例如，为包含新的令牌签名验证证书的 ACS 命名空间导入更新的 WS 联合身份验证元数据，或手动在应用程序配置中配置对称密钥。

9.  更新所有应用程序后（或截止日期已过后），在 ACS 配置中将新的密钥标记为主要。

10. 在合理的宽限期过后，使用“证书和密钥”页的“令牌签名”部分下方的“删除”按钮可从 ACS 配置中删除旧密钥。

有关详细信息，请参阅[证书和密钥][证书和密钥]。

在签名密钥过期后，如果你尝试请求令牌，将收到以下错误：

<table><tr><td><b>错误代码</b></td>
<td><b>消息</b></td>
<td><b>修复消息所需的操作</b></td>
</tr>
<tr>
<td>ACS50003</td>
<td>未配置主要对称签名密钥。SWT 需要对称签名密钥。</td>
<td>如果所选的信赖方将 SWT 用作其令牌类型，请确保为该信赖方或服务命名空间配置对称密钥，并确保该密钥被设置为主要且在其有效期内。</td></tr>
</table> 

## 令牌加密证书

如果信赖方应用程序是通过 WS-Trust 协议使用所有权确认令牌的 Web 服务，则令牌加密是必需的，在其他情况下，令牌加密是可选的。

你可以通过 ACS 管理门户的“证书和密钥”部分管理令牌加密证书。

**管理令牌加密证书**

1.  打开 Internet 浏览器并访问 [Azure 管理门户] (<http://manage.windowsazure.cn>)。

2.  使用 Windows Live ID 登录到该网站。如果你没有 Windows Live ID，则请单击“注册”以自行创建该 ID。

3.  使用你的 Windows Live ID 登录后，你将重定向到管理门户页。在该页的左下方，单击“Service Bus 和 Access Control”。

    ![][0]

4.  若要启动 ACS 管理门户，请单击左侧树中的“Access Control”，选择要配置的 ACS 服务命名空间，然后从该页顶部的工具栏中单击“Access Control 服务”按钮。

    ![][1]

    此时，你的屏幕应与下图类似：

    ![][2]

5.  在“服务设置”部分的下方，单击左侧树中的“证书和密钥”。

    ![][3]

    此时，你的屏幕应与下图类似：

    ![][5]

6.  你（或你的合作伙伴）在信赖方应用程序中更新用于令牌解密的相应的证书或密钥。
7.  使用“添加”按钮以在 ACS 中将到期的现有证书的旁边配置新的加密证书。
8.  使用“删除”以删除旧的加密证书。

有关详细信息，请参阅[证书和密钥][证书和密钥]。

在加密证书过期后，如果你尝试请求令牌，将收到以下错误：


<table><tr><td><b>错误代码</b></td>
<td><b>消息</b></td>
<td><b>修复消息所需的操作</b></td>
</tr>
<tr>
<td>ACS50005</td>
<td>需要令牌加密，但未为信赖方配置加密证书。</td>
<td>为选定的信赖方禁用令牌加密，或上载要用于令牌加密的 X.509 证书。</td></tr>
</table> 

## 令牌解密证书

ACS 可以接受来自 WS 联合身份验证标识提供程序（例如，AD FS 2.0）的加密令牌。将使用 ACS 中托管的 X.509 证书进行解密。

你可以通过 ACS 管理门户的“证书和密钥”部分管理令牌解密证书。

**管理令牌解密证书**

1.  打开 Internet 浏览器并访问 [Azure 管理门户] (<http://manage.windowsazure.cn>)。

2.  使用 Windows Live ID 登录到该网站。如果你没有 Windows Live ID，则请单击“注册”以自行创建该 ID。

3.  使用你的 Windows Live ID 登录后，你将重定向到管理门户页。在该页的左下方，单击“Service Bus 和 Access Control”。

    ![][0]

4.  若要启动 ACS 管理门户，请单击左侧树中的“Access Control”，选择要配置的 ACS 服务命名空间，然后从该页顶部的工具栏中单击“Access Control 服务”按钮。

    ![][1]

    此时，你的屏幕应与下图类似：

    ![][2]

5.  在“服务设置”部分的下方，单击左侧树中的“证书和密钥”。

    ![][3]

    此时，你的屏幕应与下图类似：

    ![][6]

6.  在“令牌解密”部分的下方，使用“添加”按钮以在将过期的现有证书旁边将 ACS 中的新证书配置为“辅助”密钥。

7.  告知使用服务的合作伙伴，他们需要在某个截止日期之前更新其相应的密钥。

8.  合作伙伴应为其信赖方或标识提供程序更新相应的证书。例如，为包含新的令牌签名验证证书的 ACS 命名空间导入更新的 WS 联合身份验证元数据，或手动在应用程序配置中配置对称密钥。

9.  更新所有应用程序后（或截止日期已过后），在 ACS 配置中将新的证书标记为主要。

10. 在合理的宽限期过后，使用“证书和密钥”页的“令牌签名”部分下方的“删除”按钮可从 ACS 配置中删除旧证书。

有关详细信息，请参阅[证书和密钥][证书和密钥]。

在解密证书过期后，如果你尝试请求令牌，将收到以下错误：


<table><tr><td><b>错误代码</b></td>
<td><b>消息</b></td>
</tr>
<tr>
<td>ACS10001</td>
<td>处理 SOAP 标头时出错。</td>
</tr>
<tr><td>ACS20001</td>
<td>处理 WS 联合身份验证登录响应时出错。</td></tr>
</table> 

## 服务标识凭据

服务标识是为 ACS 命名空间全局配置的凭据，这些凭据允许应用程序或客户端使用 ACS 直接进行身份验证并接收令牌。有三类可与 ACS 服务标识关联的凭据，即对称密钥、密码和 X.509 证书。

你可以通过 ACS 管理门户的服务标识页管理服务标识凭据。

**管理服务标识凭据**

1.  打开 Internet 浏览器并访问 [Azure 管理门户] (<http://manage.windowsazure.cn>)。

2.  使用 Windows Live ID 登录到该网站。如果你没有 Windows Live ID，则请单击“注册”以自行创建该 ID。

3.  使用你的 Windows Live ID 登录后，你将重定向到管理门户页。在该页的左下方，单击“Service Bus 和 Access Control”。

    ![][0]

4.  若要启动 ACS 管理门户，请单击左侧树中的“Access Control”，选择要配置的 ACS 服务命名空间，然后从该页顶部的工具栏中单击“Access Control 服务”按钮。

    ![][1]

    此时，你的屏幕应与下图类似：

    ![][2]

5.  在“服务设置”部分的下方，单击左侧树中的“服务标识”。

    ![][7]

6.  单击要编辑的服务标识。

    ![][8]

7.  在“凭据”部分，使用“添加”按钮以在 ACS 中将到期的现有证书或密钥旁边配置新的证书或密钥。

8.  你（或你的合作伙伴）在客户端应用程序中更新用于令牌请求的相应的证书或密钥。

9.  更新所有客户端后（或合理的宽限期过后），使用“删除”按钮以删除旧的证书或密钥。

有关详细信息，请参阅[服务标识][服务标识]。

以下是在凭据过期时 ACS 将引发的异常：

<table>
<colgroup>
<col width="25%" />
<col width="25%" />
<col width="25%" />
<col width="25%" />
</colgroup>
<tbody>
<tr class="odd">
<td align="left"><strong>凭据&gt;</strong></td>
<td align="left"><strong>错误代码</strong></td>
<td align="left"><strong>消息</strong></td>
<td align="left"><strong>修复消息所需的操作</strong></td>
</tr>
<tr class="even">
<td align="left">对称密钥、密码</td>
<td align="left">ACS50006</td>
<td align="left">签名验证失败。（消息中可能包含了更多详细信息。）</td>
<td align="left"></td>
</tr>
<tr class="odd">
<td align="left">X.509 证书</td>
<td align="left">ACS50016</td>
<td align="left">使用者为“&lt;证书使用者名称&gt;”、指纹为“&lt;证书指纹&gt;”的 X509 证书与任何配置的证书均不匹配。</td>
<td align="left">确保请求的证书已上载到 ACS。</td>
</tr>
</tbody>
</table>

若要验证和更新对称密钥或密码的过期日期，或要将新证书作为服务标识凭据上载，请按照[如何：添加具有 X.509 证书、密码或对称密钥的服务标识][如何：添加具有 X.509 证书、密码或对称密钥的服务标识]中概述的说明进行操作。“编辑服务标识”页中可用的服务标识凭据的列表。

## 管理服务凭据

ACS 管理服务是 ACS 的一个关键组件，它允许你以编程方式在 ACS 命名空间中管理和配置设置。ACS 管理服务帐户可以与三种凭据类型相关联：对称密钥、密码和 X.509 证书。

你可以通过 ACS 管理门户的“管理服务”页管理管理服务凭据。

**管理 ACS 管理服务凭据**

1.  打开 Internet 浏览器并访问 [Azure 管理门户] (<http://manage.windowsazure.cn>)。

2.  使用 Windows Live ID 登录到该网站。如果你没有 Windows Live ID，则请单击“注册”以自行创建该 ID。

3.  使用你的 Windows Live ID 登录后，你将重定向到管理门户页。在该页的左下方，单击“Service Bus 和 Access Control”。

    ![][0]

4.  若要启动 ACS 管理门户，请单击左侧树中的“Access Control”，选择要配置的 ACS 服务命名空间，然后从该页顶部的工具栏中单击“Access Control 服务”按钮。

    ![][1]

    此时，你的屏幕应与下图类似：

    ![][2]

5.  在“管理”部分的下方，单击左侧树中的“管理服务”。

    ![][9]

6.  单击管理服务帐户。

    ![][10]

7.  在“凭据”部分，使用“添加”按钮以在 ACS 中将到期的现有证书或密钥旁边配置新的证书或密钥。

8.  你（或你的合作伙伴）在客户端应用程序中更新用于令牌请求的相应的证书或密钥。

9.  更新所有客户端后（或合理的宽限期过后），使用“删除”按钮以删除旧的证书或密钥。

有关详细信息，请参阅 [ACS 管理服务][ACS 管理服务]。

如果这些凭据过期，ACS 将引发下列异常：

<table>
<colgroup>
<col width="25%" />
<col width="25%" />
<col width="25%" />
<col width="25%" />
</colgroup>
<tbody>
<tr class="odd">
<td align="left"><strong>凭据&gt;</strong></td>
<td align="left"><strong>错误代码</strong></td>
<td align="left"><strong>消息</strong></td>
<td align="left"><strong>修复消息所需的操作</strong></td>
</tr>
<tr class="even">
<td align="left">对称密钥、密码</td>
<td align="left">ACS50006</td>
<td align="left">签名验证失败。（消息中可能包含了更多详细信息。）</td>
<td align="left"></td>
</tr>
<tr class="odd">
<td align="left">X.509 证书</td>
<td align="left">ACS50016</td>
<td align="left">使用者为“&lt;证书使用者名称&gt;”、指纹为“&lt;证书指纹&gt;”的 X509 证书与任何配置的证书均不匹配。</td>
<td align="left">确保请求的证书已上载到 ACS。</td>
</tr>
</tbody>
</table>

ACS 管理门户中的“编辑管理服务帐户”页上提供了 ACS 管理服务帐户凭据的列表。

## WS 联合身份验证标识提供程序证书

WS 联合身份验证标识提供程序证书可通过其元数据获得。在配置 WS 联合身份验证标识提供程序（例如 AD FS）时，将使用通过 URL 或以文件形式提供的 WS 联合身份验证元数据来配置 WS 联合身份验证签名证书，请参阅 [WS 联合身份验证标识提供程序][WS 联合身份验证标识提供程序]和[如何：将 AD FS 2.0 配置为标识提供程序][如何：将 AD FS 2.0 配置为标识提供程序]以了解更多信息。在 ACS 中配置 WS 联合身份验证标识提供程序后，可使用 ACS 管理服务来查询它的证书有效性。请注意，对于每次通过 ACS 管理门户或 ACS 管理服务进行的元数据连续上载，都将替换密钥。

以下是在证书过期时 ACS 将引发的异常：


<table><tr><td><b>错误代码</b></td>
<td><b>消息</b></td>
</tr>
<tr>
<td>ACS10001</td>
<td>处理 SOAP 标头时出错。</td>
</tr>
<tr><td>ACS20001</td>
<td>处理 WS 联合身份验证登录响应时出错。 </td></tr>
<tr><td>ACS50006</td><td>签名验证失败。（消息中可能包含了更多详细信息。）</td></tr>
</table> 

  [Access Control 服务 2.0]: http://msdn.microsoft.com/zh-cn/library/gg429786.aspx
  [证书和密钥]: http://msdn.microsoft.com/zh-cn/library/gg185932.aspx
  [服务标识]: http://msdn.microsoft.com/zh-cn/library/gg185945.aspx
  [ACS 管理服务]: http://msdn.microsoft.com/zh-cn/library/gg185972.aspx
  [IdentityProviderKey]: http://msdn.microsoft.com/zh-cn/library/hh124084.aspx
  [ACS 错误代码]: http://msdn.microsoft.com/zh-cn/library/gg185949.aspx
  [代码示例：管理服务]: http://msdn.microsoft.com/zh-cn/library/gg185970.aspx
  [0]: ./media/manage-acs-namespace/ACS1.png
  [1]: ./media/manage-acs-namespace/ACS2.png
  [2]: ./media/manage-acs-namespace/ACS3.png
  [3]: ./media/manage-acs-namespace/ACS4.png
  [4]: ./media/manage-acs-namespace/ACS5.png
  [5]: ./media/manage-acs-namespace/ACS7.png
  [6]: ./media/manage-acs-namespace/ACS9.png
  [7]: ./media/manage-acs-namespace/ACS11.png
  [8]: ./media/manage-acs-namespace/ACS112.png
  [如何：添加具有 X.509 证书、密码或对称密钥的服务标识]: http://msdn.microsoft.com/zh-cn/library/gg185924.aspx
  [9]: ./media/manage-acs-namespace/ACS14.png
  [10]: ./media/manage-acs-namespace/ACS15.png
  [WS 联合身份验证标识提供程序]: http://msdn.microsoft.com/zh-cn/library/gg185933.aspx
  [如何：将 AD FS 2.0 配置为标识提供程序]: http://msdn.microsoft.com/zh-cn/library/gg185961.aspx
