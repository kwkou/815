
<properties 
	pageTitle="在 AD FS 中使用 Azure AD Connect Health | Windows Azure" 
	description="本页与 Azure AD Connect Health 相关，介绍如何监视本地 AD FS 基础结构。" 
	services="active-directory" 
	documentationCenter="" 
	authors="billmath" 
	manager="stevenpo" 
	editor="curtand"/>

<tags 
	ms.service="active-directory" 
	ms.date="08/14/2015" 
	wacn.date="11/02/2015"/>

# 在 AD FS 中使用 Azure AD Connect Health 
以下文档专门介绍如何使用 Azure AD Connect Health 来监视 AD FS 基础结构。

## AD FS 的警报
Azure AD Connect Health 警报部分将提供活动警报列表。每个警报均包含相关信息、解决方法步骤和相关文档的链接。选择活动或已解决的警报后，将看到一个新的边栏选项卡，其中将显示额外信息、可用于解决警报的方法步骤以及其他文档的链接。还可以查看过去已解决警报的相关历史数据。

选择警报后，将获取到额外信息、可用于解决警报的方法步骤以及其他文档的链接。

![Azure AD Connect Health 门户](./media/active-directory-aadconnect-health/alert2.png)



## AD FS 的使用情况分析
Azure AD Connect Health 使用情况分析可分析联合服务器的身份验证流量。选择使用情况分析框将会打开使用情况分析边栏选项卡，其中将显示度量值和分组。

>[AZURE.NOTE]若要将使用情况分析与 AD FS 结合使用，必须确保启用了 AD FS 审核。有关详细信息，请参阅[为 AD FS 启用审核](/documentation/articles/active-directory-aadconnect-health-operations#enable-auditing-for-ad-fs)。

![Azure AD Connect Health 门户](./media/active-directory-aadconnect-health/report1.png)

若要选择其他度量值、指定时间范围或更改分组，只需右键单击使用情况分析图表并选择“编辑图表”。然后就可以指定时间范围、更改或选择度量值以及更改分组。可以查看基于不同"度量值"的身份验证流量分布，并使用以下所述的相关"分组依据"参数对每个度量值进行分组。

| 度量值 | 分组依据 | 分组意味着什么，它为什么很有用？ |
| ------ | -------- | -------------------------------------------- |
| 请求总数：联合身份验证服务所处理的请求总数 | 全部 | 这将显示未分组的总请求数的计数。 |
| | 应用程序 | 此选项将基于目标信赖方对请求总数进行分组。此分组有助于了解具体某个应用程序正在接收多少百分比的总流量。 |
| | 服务器 | 此选项将基于处理请求的服务器对请求总数进行分组。此分组有助于了解总流量的负载分布。 |
| | 工作区加入 | 此选项将基于请求是否来自已加入工作区（已知）的设备，来对请求总数进行分组。此分组有助于了解是否使用标识基础结构未知的设备来访问资源。 |
| | 身份验证方法 | 此选项将基于用于身份验证的身份验证方法来对请求总数进行分组。此分组有助于了解用于身份验证的常见身份验证方法。可能的身份验证方法如下所示：<ol> <li>Windows 集成身份验证 (Windows)</li> <li>基于表单的身份验证（表单）</li> <li>SSO（单一登录）</li> <li>X509 证书身份验证（证书）</li> <br>请注意，如果联合服务器接收带有 SSO Cookie 的请求，则该请求会被计为 SSO（单一登录）。在这种情况下，如果此 cookie 有效，则不会要求用户提供凭据并该用户可无缝访问该应用程序。如果你有多个受联合服务器保护的信赖方，则这种情况很常见。 |
| | 网络位置 | 此选项将基于用户的网络位置对请求总数进行分组。该位置可以是 Intranet 或 Extranet。此分组有助于了解来自 Intranet 或 Extranet 的流量分别为多少百分比。 |
| 失败请求总数：联合身份验证服务处理的失败请求总数。<br>（此度量值仅在 Windows Server 2012 R2 的 AD FS 上可用）| 错误类型 | 这将基于预定义错误类型显示错误数。此分组有助于了解常见类型的错误。<ul><li>用户名或密码错误：因用户名或密码不正确而导致的错误。</li> <li>“Extranet 锁定”：收到无法访问 Extranet 的用户的请求而导致的失败</li><li>“过期密码”：用户使用已过期密码登录而导致的失败。</li><li>“禁用帐户”：用户使用已禁用帐户登录导致的失败。</li><li>“设备身份验证”：用户无法使用“设备身份验证”进行身份验证导致的失败。</li><li>“用户证书身份验证”：用户因证书无效而无法进行身份验证所导致的失败。</li><li>“MFA”：用户无法使用“多因素身份验证”进行身份验证导致的失败。</li><li>“其他凭据”：“颁发授权”：因授权失败而失败。</li><li>“颁发委派”：颁发委派错误导致的失败。</li><li>“令牌接受”：由于 ADFS 拒绝来自第三方标识提供者的令牌而导致的失败。</li><li>“协议”：因协议错误而失败。</li><li>“未知”：全部捕获。任何其他不属于定义类别的失败。</li> |
| | 服务器 | 这将基于服务器对错误进行分组。这有助于了解服务器间的错误分布。分布不均匀可能表示服务器处于错误状态。 |
| | 网络位置 | 这将基于请求的网络位置（Intranet 或 Extranet）对错误进行分组。这有助于了解要失败的请求类型。 |
| | 应用程序 | 这将基于目标应用程序（信赖方）对失败进行分组。这有助于了解查看到最多错误数量的目标应用程序。 |
| 用户计数：系统中处于活动状态的唯一用户数的平均数 | 全部 | 这将提供所选时间片内使用联合身份验证服务的用户数的平均计数。不对用户进行分组。<br>平均值将取决于所选的时间片。 |
| | 应用程序 | 这将基于目标应用程序（信赖方）对用户平均数进行分组。这有助于了解使用具体某个应用程序的用户数量。 |


## AD FS 的性能监视
Azure AD Connect Health 性能监视提供有关度量值的监视信息。选择“监视”框将打开提供度量值详细信息的边栏选项卡。


![Azure AD Connect Health 门户](./media/active-directory-aadconnect-health/perf1.png)


通过选择边栏选项卡顶部的“筛选器”选项，你可以按服务器进行筛选，以查看单个服务器的度量值。若要更改度量值，只需右键单击监视边栏选项卡下的监视图表，并选择“编辑图表”。然后在打开的新边栏选项卡中，你可以从下拉列表中选择其他度量值，并指定查看性能数据的时间范围。




## 相关链接

* [Azure AD Connect Health](/documentation/articles/active-directory-aadconnect-health)
* [适用于 AD FS 的 Azure AD Connect Health 代理安装](/documentation/articles/active-directory-aadconnect-health-agent-install-adfs)
* [Azure AD Connect Health 操作](/documentation/articles/active-directory-aadconnect-health-operations)
* [Azure AD Connect Health 常见问题](/documentation/articles/active-directory-aadconnect-health-faq)

<!---HONumber=76-->