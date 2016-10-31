<properties
	pageTitle="在登录页和访问面板页中添加公司品牌"
	description="了解如何在 Azure 登录页和访问面板页中添加公司品牌"
	services="active-directory"
	documentationCenter=""
	authors="curtand"
	manager="femila"
	editor=""/>  


<tags
	ms.service="active-directory"
	ms.workload="infrastructure-services"
	ms.tgt_pltfrm="na"
	ms.devlang="na"
	ms.topic="get-started-article"
	ms.date="09/30/2016"
	ms.author="curtand"
	wacn.date="10/31/2016"/>  


# 在登录页和访问面板页中添加公司品牌


许多公司想要在他们管理的所有网站和服务上应用一致的外观，以免用户感到困惑。Azure Active Directory 提供了此功能，它允许你使用自己的公司徽标和自定义颜色方案来自定义以下网页的外观：

- **登录页** - 登录到 Office 365 或其他使用 Azure AD 作为标识提供者的基于 Web 的应用程序时，将显示此页。在执行主领域发现期间或输入凭据时，你将与此页交互。主领域发现可让系统将联合用户重定向到其本地 STS（例如 AD FS）。

- **访问面板页** - 访问面板是一个基于 Web 的门户，可让你查看和启动 Azure AD 管理员授权你访问的基于云的应用程序。若要访问“访问面板”，请使用以下 URL：[https://myapps.microsoft.com](https://myapps.microsoft.com)。

本主题说明如何自定义登录页和访问面板页。

> [AZURE.NOTE]
><p>
><p>- 只有在升级到 Azure Active Directory 高级或基本版（或者成为 Office 365 用户）后，才可以使用公司品牌这一功能。有关详细信息，请参阅 [Azure Active Directory 版本](/documentation/articles/active-directory-editions/)。
><p>- 在中国，使用 Azure Active Directory 全球实例的客户可以使用 Azure Active Directory 高级和基本版。由中国的 21Vianet 运营的 Azure 服务目前不支持 Azure Active Directory高级和基本版。有关详细信息，请在 [Azure Active Directory 论坛](https://feedback.azure.com/forums/169401-azure-active-directory/)与我们联系。



## 自定义登录页

通常，如果你需要通过浏览器访问组织订阅的云应用和服务，可以使用登录页。

如果你已对登录页应用了更改，最多需要一小时更改才会出现。

仅当你使用租户特定的 URL（例如 https://outlook.com/**contoso**.com 或 https://mail.**contoso**.com）访问服务时，才出现带有品牌的登录页。

当你使用非租户特定的 URL（例如 https://mail.office365.com）访问服务时，将出现没有品牌的登录页。在此情况下，输入你的用户 ID 或选择用户磁贴之后，品牌就会出现。

> [AZURE.NOTE]
><p>
><p>- 在已配置品牌的 Azure 经典管理门户的“Active Directory”>“目录”>“域”部分中，域名必须显示为“活动”。
><p>- 登录页品牌不会带到 Microsoft 的使用者登录页。如果你使用个人 Microsoft 帐户登录，则可以看到 Azure AD 呈现的经过品牌打造的用户磁贴列表，但你组织的品牌将不会应用于 Microsoft 帐户登录页。


如果你要在此页上显示你公司的品牌、颜色和其他可自定义元素，请参阅下图以了解两种体验之间的差异。

以下屏幕截图显示自定义**之前**台式机上的 Office 365 登录页示例：

![自定义之前的 Office 365 登录页][1]

以下屏幕截图显示自定义**之后**台式机上的 Office 365 登录页示例：

![自定义之后的 Office 365 登录页][2]

以下屏幕截图显示自定义**之前**移动设备上的 Office 365 登录页示例：

![自定义之前的 Office 365 登录页][3]


以下屏幕截图显示自定义**之后**移动设备上的 Office 365 登录页示例：

![自定义之后的 Office 365 登录页][4]


当你调整浏览器窗口的大小时，通常会对大图（如前面显示的图）进行裁剪，使之适应不同的屏幕纵横比。考虑到这一点，你应尝试保留插图中的关键可视元素，使这些元素始终显示在左上角（从右到左显示的语言则为右上角）。这一点很重要，因为调整大小通常从右下角向顶部/左侧进行，或者从底部向顶部进行。

下图显示了朝左侧调整浏览器大小时插图的裁剪方式：

![][6]

这是朝顶部调整浏览器大小后该插图的显示方式：

![][7]

## 我可以在页上自定义哪些元素？

可以在登录页上自定义以下元素：

![][5]



| 页面元素 | 在页面上的位置 |
|:--            | ---                  |
|横幅徽标 | 显示在页面右上角。替换你要登录到的目标站点（例如Office 365 或 Azure）通常会显示的徽标。|
|大图/背景色 | 显示在页面左侧。替换你要登录到的目标站点显示的图像。在连接的带宽较低或屏幕较窄时，可能会显示背景色来代替大图。|
|使我保持登录状态 | 显示在“密码”文本框下面。 |
|登录页文本 | 当你使用工作或学校帐户登录前需要传达有用的信息时，此文本将显示在页脚的上方。例如，你可能想要提供技术支持的电话号码或法律声明。|


> [AZURE.NOTE]
所有元素都是可选的。例如，如果你指定了横幅徽标，但没有指定大图，则登录页将显示你的徽标和目标站点的插图（即 Office 365 加利福尼亚州高速公路图像）。


在登录页中，“使我保持登录状态”复选框允许用户在关闭和重新打开浏览器时保持登录状态。它不会影响会话生存期。可以隐藏 Azure Active Directory 登录页上的此复选框。

是否显示此复选框取决于“隐藏 KMSI”这一设置。

![][9]  



若要隐藏复选框，可将此设置配置为“隐藏”。

> [AZURE.NOTE] SharePoint Online 和 Office 2010 的某些功能取决于用户能否勾选此框。如果将此设置配置为“隐藏”，用户可能会在登录时看到其他意外的提示。




你还可以本地化此页上的所有元素。在配置“默认”的一组自定义元素后，便可以针对不同的区域设置配置其他版本。你还可以混搭各种元素。例如，你可以：

- 创建适用于所有区域性的“默认”大图，然后为英文和法文创建特定版本。当你将浏览器设置为这两种语言之一时，将出现特定的图像，对于其他所有语言，将出现默认插图。

- 为你的组织配置不同的徽标（例如日语或希伯来语版本）



## “访问面板”页自定义

“访问面板”页实质上是门户页，可让你快速访问管理员授权你访问的云应用。在此页上，你的应用显示为可单击的应用程序磁贴。


以下屏幕截图显示了自定义之后的访问面板页示例。

![][8]  


## 使用公司品牌配置你的目录

可以针对 Azure 经典管理门户中的每个目录配置一组默认的可自定义元素。保存默认设置后，管理员可以为不同的语言/区域设置添加每个元素的本地化版本。所有可自定义元素都是可选的。

例如，如果你配置了默认横幅徽标，但没有配置大图，则登录页将在右上角显示你的徽标。但是，会显示该站点的默认插图。

假设有以下配置：

- 默认的横幅徽标和英文登录页文本
- 特定于德语的登录页文本

如果你的语言首选项是德语，则会显示默认的横幅标志，但文本是德语。

虽然从技术上讲，你可以为 Azure AD 支持的每种语言都配置不同的设置，但由于维护和性能方面的原因，我们建议你尽量少使用不同的设置。

**若要将公司品牌添加到目录，请执行以下步骤：**

1. 以需要自定义的目录的管理员身份登录到 [Azure 经典管理门户](https://manage.windowsazure.cn)。
2. 选择要自定义的目录。
3. 在顶部菜单栏中，单击“配置”。
4. 单击“自定义品牌”。
4. 修改要自定义的元素。所有字段都是可选的。
5. 单击“保存”。

最长可能需要一个小时才能显示你针对登录页品牌所做的任何新更改。

**若要添加语言特定的公司品牌，请执行以下步骤：**

1. 以需要自定义的目录的管理员身份登录到 [Azure 经典管理门户](https://manage.windowsazure.cn)。
2. 选择要自定义的目录。
3. 在顶部菜单栏中，单击“配置”。
4. 单击“自定义品牌”。
2. 单击“为特定语言添加品牌”。
3. 选择要为其自定义徽标的语言，然后单击“下一步”。
3. 只编辑要为其配置特定于语言的覆盖的元素。所有字段都是可选的。如果将某一字段留空，则将改为显示自定义的默认值（如果未配置自定义的默认值，则显示 Microsoft 默认值）。
4. 单击“保存”。

**若要从目录中删除公司品牌，请执行以下步骤：**

1. 以需要自定义的目录的管理员身份登录到 [Azure 经典管理门户](https://manage.windowsazure.cn)。
2. 选择要自定义的目录。
3. 在顶部菜单栏中，单击“配置”。
4. 单击“自定义品牌”。
5. 在“自定义品牌”页上，选择“编辑现有品牌设置”，然后转到下一页。
3. 根据要删除哪些元素，执行以下一项或多项操作：

	a.在“横幅徽标”下面，选择“删除上载的徽标”。

    b.在“磁贴徽标”下面，选择“删除上载的徽标”。

    c.删除所有文本框中的文本。

    d.单击“下一步”。

    e.删除所有文本框中的文本。

4. 单击“保存”按钮以删除这些元素。
5. 如有必要，请再次单击“自定义品牌”，然后对需要删除的所有特定于语言的品牌重复这些步骤。当你单击“自定义品牌”并看到未配置任何现有设置的“自定义默认品牌”表单时，所有品牌设置均已删除。

## 测试和示例

我们建议你在生产环境中进行更改之前，先使用测试租户进行试验。

**若要验证是否已应用你的品牌，请执行以下操作：**

1. 打开 InPrivate 或 Incognito 浏览器会话。
2. 访问 https://outlook.com/contoso.com（将 contoso.com 替换为自定义的域）。

这一点同样适用于类似于 contoso.partner.onmschina.cn 的域。

为了帮助你创建有效的自定义设置，我们自定义了以下两个虚构的登录页：

- [http://aka.ms/aaddemo001](http://aka.ms/aaddemo001)
- [http://aka.ms/aaddemo002](http://aka.ms/aaddemo002)

若要测试特定于语言的设置，需要将你的 Web 浏览器中的默认语言首选项修改为你在自定义中设置的语言。在 Internet Explorer 的“Internet 选项”菜单中配置此项。

## 可自定义的元素

Azure AD 中的某些可自定义的元素具有多个用例。只需为每个目录配置公司徽标一次，然后即可在登录页和“访问面板”页上使用。某些可自定义的元素只特定于登录页。下表提供了不同的可自定义元素的详细信息。

名称 | 说明 | 约束 | 建议
	------------- | ------------- | ------------- | -------------
横幅徽标 | 横幅徽标将显示在登录页和“访问”面板上。 | <p>JPG 或 PNG</p><p>60x280 像素</p><p>10 KB</p> | <p>使用组织的完整徽标（包括象形图和标识）</p><p>使其高度保持在 30 像素以下，以免在移动设备上产生滚动条</p><p>使其大小保持在 4 KB 以下</p><p>使用透明的 PNG（不要想当然地认为登录页的背景始终为白色）</p>
磁贴徽标 | （目前未在登录页中使用）在将来，可能会使用此文本替换不同体验位置的通用“工作或学校帐户”象形图。 | <p>JPG 或 PNG</p><p>120x120 像素</p><p>10 KB</p> | <p>使其保持简单（没有小文本），因为可能会将此图像的大小调整至 50%
</p> |
登录页用户名标签 | （目前未在登录页中使用）在将来，可能会使用此文本替换不同体验位置的通用“工作或学校帐户”字符串。你可以将它设置为“Contoso 帐户”或“Contoso ID”等内容。 | <p>Unicode 文本，最多 50 个字符</p><p>仅纯文本（没有链接或 HTML 标记）</p> | <p>使其保持简单扼要</p><p>询问用户他们通常如何引用你为他们提供的工作或学校帐户。</p>
登录页文本 | 此“样本”文字显示在登录页表单下方，可用于告知其他说明或获取帮助和支持的位置。 | <p>Unicode 文本，最多 256 个字符</p><p>仅纯文本（没有链接或 HTML 标记）</p> | 使其保持在 250 个字符以下（大约 3 行文本）
登录页插图 | 此插图是显示在登录页表单左侧登录页上的大图像。 | <p>JPG 或 PNG</p><p>1420x1200</p><p>500 KB</p> | <p>1420x1200 像素</p><p>重要说明：使其保持尽可能小，最好在 200 KB 以下。如果此图像太大，则在未缓存该图像时，登录页的性能会受到影响</p><p>通常会裁剪此图像，使之适应不同的屏幕比率。请将主要视觉元素保留在左上角（RTL 语言为右上角），因为当浏览器窗口缩小时，调整大小将从底部/右下角向顶部/左上角进行。</p>
登录页背景色 | 登录页背景色在登录页表单左侧的区域中使用。 | 必须是十六进制格式的 RGB 颜色（示例：#FFFFFF） | <p>连接的带宽较低时，可能会显示背景色来代替大图</p><p>建议选取横幅徽标的主要颜色</p>


## 后续步骤

- [查看访问和使用情况报告](/documentation/articles/active-directory-view-access-usage-reports/)

<!--Image references-->

[1]: ./media/active-directory-add-company-branding/SignInPage_beforecustomization.png
[2]: ./media/active-directory-add-company-branding/SignInPage_aftercustomization.png
[3]: ./media/active-directory-add-company-branding/SignInPage_mobile_beforecustomization.png
[4]: ./media/active-directory-add-company-branding/SignInPage_mobile_aftercustomization.png
[5]: ./media/active-directory-add-company-branding/SignInPage_aftercustomization_elements.png
[6]: ./media/active-directory-add-company-branding/SignInPage_aftercustomization_croppedleft.png
[7]: ./media/active-directory-add-company-branding/SignInPage_aftercustomization_croppedtop.png
[8]: ./media/active-directory-add-company-branding/APBranding.png
[9]: ./media/active-directory-add-company-branding/hidekmsi.png

<!---HONumber=Mooncake_1024_2016-->
