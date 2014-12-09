<properties umbracoNaviHide="0" pageTitle="将 Azure AD 与 Google Apps 集成" metaKeywords="Azure Google Apps Integration" description="了解有关将 Azure AD 与 Google Apps 集成的信息。" linkid="documentation-services-identity-windows-azure-ad-integration-with-google=apps" urlDisplayName="Azure AD Integration with Google Apps" headerExpose="" footerExpose="" disqusComments="0" editor="lisatoft" manager="terrylan" title="将 Azure AD 与 Google Apps 集成" authors="" />

# 将 Azure AD 与 Google Apps 集成

本教程的目的是介绍 Azure 与 Google Apps 的集成。在本教程中概述的方案假定您已具有以下各项：

-   一个有效的 Azure 订阅
-   Googe Apps 中的一个测试租户

如果您还没有 Google Apps 中的有效租户，例如，您可以在 Google Apps for Business 网站上进行注册以获取一个试用帐户。

在本教程中概述的方案由以下构建基块组成：

支持 Google Apps 的应用程序集成
配置单一登录
支持 Google Apps API 访问
添加自定义域
配置用户设置

# 支持 Google Apps 的应用程序集成

此部分的目的是概述如何支持 Google Apps 的应用程序集成。

## 若要支持 Google Apps 的应用程序集成，请执行以下步骤：

1.  在 Azure 管理门户中的左侧导航窗格中，单击“Active Directory”。
2.  从“目录”列表中，选择要为其启用目录集成的目录。
3.  若要打开应用程序视图，请在目录视图的顶部菜单中，单击“应用程序”。
4.  单击底部的“添加”以打开“添加应用程序”对话框。
5.  在“将应用与 Azure AD 集成”对话框中，单击“管理对应用程序的访问”。
6.  在“选择要管理的应用程序”页上，从应用程序列表中选择“Google Apps”。
7.  单击“完成”按钮以添加应用程序，然后关闭该对话框。

# 配置单一登录

此部分的目的是概述如何使用户使用基于 SAML 协议的联合身份验证通过他们在 Azure AD 中的帐户对 Google Apps 进行身份验证。

## 若要配置联合单一登录，请执行以下步骤：

1.  在 Azure 管理门户中，选择左侧导航窗格中的“Active Directory”以打开“Active Directory”对话框页。
2.  在目录列表中，选择您的目录以打开您的目录的配置页。
3.  从顶层菜单中选择“应用程序”。
4.  从应用程序列表中，选择“Google Apps”以打开 Google Apps 配置对话框。
5.  若要打开“配置单一登录”对话框，请单击“配置单一登录”。

    ![Google\_Tutorial\_01][Google\_Tutorial\_01]

6.  在“为此应用选择单一登录模式”对话框页上，选择“模式为用户使用他们在 Azure AD 中的帐户进行身份验证”，然后单击“下一步”按钮。

    ![Google\_Tutorial\_02][Google\_Tutorial\_02]

7.  在“配置应用 URL”对话框页上的“GOOGLE APPS 租户 URL”文本框中，键入 Google Apps 租户 URL，然后单击“下一步”按钮。

    ![Google\_Tutorial\_03][Google\_Tutorial\_03]

8.  在“在 Google Apps 中配置单一登录”对话框页上，执行以下步骤，然后单击“完成”按钮。

    -   单击“下载证书”，然后将证书另存为“c:\\googleapps.cer”。
    -   打开 Google Apps 登录页，然后登录

    ![Google\_Tutorial\_04][Google\_Tutorial\_04]

    -   在“管理员控制台”上，单击“安全”

    ![Google\_Tutorial\_05][Google\_Tutorial\_05]

    如果安全图标不可见，则应该单击页面底部的“更多控件”。

9.  在“安全”页上，单击“高级”设置。

    ![Google\_Tutorial\_06][Google\_Tutorial\_06]

10. 在该页面的“高级”设置部分中，选择“设置单一登录”。

    ![Google\_Tutorial\_07][Google\_Tutorial\_07]

11. 在“设置单一登录”页上，执行以下步骤：

    ![Google\_Tutorial\_08][Google\_Tutorial\_08]

        + Select <strong>Enable Single Sign-on</strong>.
        + On the <strong>Configure single sign-on at Google Apps</strong> page in the Azure AD Portal, copy the <strong>SINGLE SIGN-ON URL</strong>, and then paste it into the related textbox on the <strong>Security page</strong> in the Google Apps <strong>Admin console</strong>.
        + On the <strong>Configure single sign-on at Google Apps</strong> page in the Azure AD Portal, copy the <strong>Single sign-out service URL</strong>, and then paste it into the related textbox on the <strong>Security</strong> page in the Google Apps <strong>Admin console</strong>.
        + On the <strong>Configure single sign-on at Google Apps</strong> page in the Azure AD Portal, copy the <strong>Change password URL</strong>, and then paste it into the related textbox on the <strong>Security</strong> page in the Google Apps <strong>Admin console</strong>.
        + Click the <strong>Browse</strong> button to locate the <strong>Verification certificate</strong>, and then click Upload.
        + Click <strong>Save</strong> changes.

12. 在 Azure AD 门户中的“在 Google Apps 中配置单一登录”页上，单击“完成”按钮。

您现在可以转到[访问面板][访问面板]并测试对 Google Apps 的单一登录。

# 支持 Google Apps API 访问

在将 Azure Active Directory 与 Google Apps 集成以进行用户设置时，您必须支持 Google Apps 中的租户的 API 访问。

## 支持 Google Apps API 访问

1.  登录到您的 Google Apps 租户。
2.  在“管理员控制台”中，单击“安全”。

    ![Google\_Tutorial\_05][Google\_Tutorial\_05]

    </p>
    如果安全图标不可见，请单击管理员控制台底部的“更多控件”。

3.  在“安全”页上，单击“API 参考”以打开相关的配置对话框页。
4.  选择“支持 API 访问”。

    ![Google\_Tutorial\_09][Google\_Tutorial\_09]

    </p>

# 添加自定义域

使用 Google Apps 配置用户设置要求 Azure AD 域和 Google Apps 域具有相同的完全限定域名 (FQDN)。但是，例如，当您使用试用租户测试本教程中的方案时，租户的 FQDNS 通常不匹配。若要解决此问题，您可以在 Azure AD 和 Google Apps 中配置自定义域。
配置自定义域需要访问您的公共域的 DNS 区域文件。

![Google\_Tutorial\_10][Google\_Tutorial\_10]

## 若要在 Azure AD 中添加自定义域，请执行以下步骤：

1.  在 Azure 管理门户中，选择左侧导航窗格中的“Active Directory”以打开“Active Directory”对话框页。
2.  在目录列表中，选择您的目录以打开您的目录的配置页。
3.  从顶层菜单中选择“域”。
4.  若要打开“添加域名”文本框，请键入您的域名，然后单击“添加”。
5.  单击“下一步”以打开“验证您的域名”对话框页。

    ![Google\_Tutorial\_11][Google\_Tutorial\_11]

6.  选择“记录类型”，然后在您的 DNS 区域文件中注册所选的记录。

    ![Google\_Tutorial\_12][Google\_Tutorial\_12]

7.  通过使用“nslookup 命令”，您应该验证 DNS 记录是否已成功注册。

    ![Google\_Tutorial\_13][Google\_Tutorial\_13]

## 若要在 Google Apps 中添加自定义域，请执行以下步骤：

1.  登录到您的 Google Apps 租户。
2.  在“管理员控制台”中，单击“域”。

    ![Google\_Tutorial\_14][Google\_Tutorial\_14]

3.  单击“添加自定义域”。

    ![Google\_Tutorial\_15][Google\_Tutorial\_15]

4.  单击“使用您已经拥有的域”，然后单击“继续”。

    ![Google\_Tutorial\_16][Google\_Tutorial\_16]

5.  键入您的自定义域的名称，然后单击“继续”。

    ![Google\_Tutorial\_17][Google\_Tutorial\_17]

6.  完成这些步骤以验证域的所有权。

    如果您已配置联合单一登录，您必须更新您的联合单一登录配置中的 Google Apps 租户 URL。

# 配置用户设置

此部分的目的是概述如何对 Google Apps 支持 Active Directory 用户帐户的设置。

## 若要配置用户设置，请执行以下步骤：

1.  在 Azure 管理门户中，选择左侧导航窗格中的“Active Directory”以打开“Active Directory”对话框页。
2.  在目录列表中，选择您的目录以打开您的目录的配置页。
3.  从顶层菜单中选择“应用程序”。
    从应用程序列表中，选择“Google Apps”以打开“Google Apps”配置对话框。
4.  若要打开“配置帐户同步”对话框，请单击“配置帐户同步”。
5.  在“配置帐户同步”对话框页上，提供 Google Apps 域名、Google Apps 用户名和 Google Apps 密码，然后单击“下一步”按钮。

    ![Google\_Tutorial\_18][Google\_Tutorial\_18]

6.  在“确认”对话框页上，单击“完成”按钮以关闭“配置帐户同步”对话框。

您现在可以创建一个测试帐户、等待 10 分钟，然后验证该帐户是否已同步到 Google Apps。

另请参阅
[管理 Azure Active Directory 的应用程序访问增强功能的最佳实践
应用程序访问][管理 Azure Active Directory 的应用程序访问增强功能的最佳实践
应用程序访问]

  [Google\_Tutorial\_01]: ./media/integration-azure-google-apps/Google_Tutorial_01.png
  [Google\_Tutorial\_02]: ./media/integration-azure-google-apps/Google_Tutorial_02.png
  [Google\_Tutorial\_03]: ./media/integration-azure-google-apps/Google_Tutorial_03.png
  [Google\_Tutorial\_04]: ./media/integration-azure-google-apps/Google_Tutorial_04.png
  [Google\_Tutorial\_05]: ./media/integration-azure-google-apps/Google_Tutorial_05.png
  [Google\_Tutorial\_06]: ./media/integration-azure-google-apps/Google_Tutorial_06.png
  [Google\_Tutorial\_07]: ./media/integration-azure-google-apps/Google_Tutorial_07.png
  [Google\_Tutorial\_08]: ./media/integration-azure-google-apps/Google_Tutorial_08.png
  [访问面板]: https://login.microsoftonline.com/login.srf?wa=wsignin1.0&rpsnv=2&ct=1384289458&rver=6.1.6206.0&wp=MCMBI&wreply=https:%2F%2Faccount.activedirectory.windowsazure.com%2Flanding.aspx%3Ftarget%3D%252fapplications%252fdefault.aspx&lc=1033&id=500633
  [Google\_Tutorial\_09]: ./media/integration-azure-google-apps/Google_Tutorial_09.png
  [Google\_Tutorial\_10]: ./media/integration-azure-google-apps/Google_Tutorial_10.png
  [Google\_Tutorial\_11]: ./media/integration-azure-google-apps/Google_Tutorial_11.png
  [Google\_Tutorial\_12]: ./media/integration-azure-google-apps/Google_Tutorial_12.png
  [Google\_Tutorial\_13]: ./media/integration-azure-google-apps/Google_Tutorial_13.png
  [Google\_Tutorial\_14]: ./media/integration-azure-google-apps/Google_Tutorial_14.png
  [Google\_Tutorial\_15]: ./media/integration-azure-google-apps/Google_Tutorial_15.png
  [Google\_Tutorial\_16]: ./media/integration-azure-google-apps/Google_Tutorial_16.png
  [Google\_Tutorial\_17]: ./media/integration-azure-google-apps/Google_Tutorial_17.png
  [Google\_Tutorial\_18]: ./media/integration-azure-google-apps/Google_Tutorial_18.png
