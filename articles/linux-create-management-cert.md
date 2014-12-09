<properties linkid="manage-linux-common-tasks-manage-certs" urlDisplayName="Manage certificates" pageTitle="在 Azure 中为 Linux 虚拟机管理证书" metaKeywords="Azure management certs, uploading management certs, Azure Service Management API" description="了解如何在 Azure 中为 Linux 创建和上载管理证书。如果您使用服务管理 API，则证书是必需的。" metaCanonical="" services="virtual-machines" documentationCenter="" title="在 Azure 中为 Linux 创建管理证书" authors="kathydav" solutions="" manager="jeffreyg" editor="tysonn" />

# 在 Azure 中为 Linux 创建管理证书

在您要使用服务管理 API 与 Azure 映像平台进行交互时，需要管理证书。

<http://msdn.microsoft.com/zh-cn/library/azure/gg981929.aspx> 中已包含有关如何创建和管理这些证书的文档。您也可以使用 OpenSSL 创建管理证书。有关更多信息，请参见 [OpenSSL][OpenSSL]。但是，此文档主要关注如何使用可能并非所有 Linux 用户均可访问的 Silverlight 门户。它介绍如何能够获取对这些证书的访问权限，将这些证书与我们的各种工具、不同合作伙伴集成以及独立使用这些证书，直到此功能添加到 Azure 管理门户中。

## 目录

-   [从 publishsettings 文件获取管理证书][从 publishsettings 文件获取管理证书]
-   [使用 Azure 管理门户安装管理证书][使用 Azure 管理门户安装管理证书]

## <span id="publishsettings"></span></a>如何：创建并上载管理证书

我们为您提供一种针对 Azure 创建管理证书的简便方式，即访问：<https://manage.windowsazure.cn/publishsettings/Index?client=vs&SchemaVersion=1.0>

此网站将要求您使用您的门户凭据进行登录，然后将为您生成管理证书，此证书与将要求您下载的 publishsettings 文件中您的 subscriptionID 打包在一起。

![linuxcredentials][linuxcredentials]

请确保将此文件保存到安全位置，因为您将无法恢复它并将需要生成新的管理证书。（可在系统中使用的证书总数是有限制的。请参阅此网站上的相应部分以确认此限制。）然后，您可以通过多种方式使用此证书：

### 在 Visual Studio 中

![VSpublish][VSpublish]

### 在 Linux Azure 命令行中

您可以通过运行 Azure 帐户导入命令导入该证书以供使用：

![cmdlinepublish][cmdlinepublish]

对于任何其他合作伙伴或者需要该工具的软件，您将需要从文件本身提取管理证书并对其进行 Base 64 解码。ScaleXtreme 和 SUSE Studio 等一些合作伙伴可直接在其当前窗体中使用此文件。

为了提取管理证书，您需要遵循此过程。

您需要从此文件中提取 ManagementCertificate 之后引号之间的 base 64 编码内容。

    ?xml version="1.0" encoding="utf-8"?>
    <PublishData>
      <PublishProfile
        PublishMethod="AzureServiceManagementAPI"
        Url="https://management.core.windows.net/"
        ManagementCertificate="xxxxx">
        <Subscription
          Id="8a4a0a51-728e-482e-8daa-c477f03c541d"
          Name="hjereztest" />
      </PublishProfile>
    </PublishData>

您需要将此内容复制到文件，然后在 Linux 中使用可供使用的 base 64 解码器对其进行解码：

    Base64 -d [encodedfile] > [decodedfile].pfx

这将为您提供一个可使用 openssl 转换为其他格式或者根据需要提取私钥的 pfx 文件：

    openssl.exe pkcs12 -in publicAndprivate.pfx -nocerts -out privateKey.pem 

在 Windows 中，您可以使用 powershell 或免费的 Windows base 64 解码器（例如 [http://www.fourmilab.ch/webtools/base64/base64.zip][http://www.fourmilab.ch/webtools/base64/base64.zip]）解码和提取 PFX 文件，方法是运行命令

    base64 -d key.txt ->key.pfx

#### 证书生成限制

只能使用此工具在平台中最多创建 10 个证书。
遗憾的是，您无法在生成密钥后恢复您生成的密钥，因为我们不会在系统中的任何位置保存这些私钥。
如果您达到系统中证书的上限，则需要通过 Azure 论坛与支持人员联系，以便清除您的证书或使用可呈现旧版 Silverlight 门户的浏览器来执行这些任务。

#### 私钥泄露

如果您的私钥可随时被泄露，您将需要使用支持 Silverlight 的浏览器访问旧版门户并删除文件上相应的管理证书，或者通过论坛与支持人员联系。
生成一个新证书并不够，因为全部 10 个证书都有效并且已泄露的旧密钥将仍然能够访问该网站。

## <span id="management"></span></a>使用 Azure 管理门户安装管理证书

可通过多种方法创建管理证书。有关创建证书的详细信息，请参阅[为 Azure 创建管理证书（可能为英文页面）][为 Azure 创建管理证书（可能为英文页面）]。在创建证书后，将其添加到 Azure 中您的订阅。

1.  登录到 Azure 管理门户。

2.  在导航窗格中，单击“设置”。

3.  在“管理证书”下，单击“上载管理证书”。

4.  在“上载管理证书”中，浏览到证书文件，然后单击“确定”。

### 获取证书指纹和订阅 ID

您需要已添加的管理证书的指纹，并且需要订阅 ID 才能将 .vhd 文件上载到 Azure。

1.  从管理门户中，单击“设置”。

2.  在“管理证书”下，单击您的证书，然后通过将指纹复制并粘贴到稍后可以对其进行检索的位置，从“属性”窗格中记录指纹。

您还需要您的订阅的 ID 以上载 .vhd 文件。

1.  从管理门户中，单击“所有项目”。

2.  在中间窗格中的“订阅”下，复制订阅并将其粘贴到稍后可以对其进行检索的位置。

### 如果您生成了您自己的密钥，请向工具提供此信息

#### 对于 CSUpload

1.  以管理员身份打开“Azure SDK 命令提示符”窗口。
2.  通过使用以下命令并将“Subscriptionid”和“CertThumbprint”替换为您先前获取的值，设置连接字符串：

        csupload Set-Connection "SubscriptionID=<Subscriptionid>;CertificateThumbprint=<Thumbprint>;ServiceManagementEndpoint=https://management.core.windows.net"

#### 对于 Linux Azure 命令行工具

您将需要通过命令对您使用 openssl 创建的 PFX 文件进行 base 64 编码：

        Base64 [file] > [econded file]

然后，您将需要将您的订阅 ID 和 base64 编码的 pfx 合并到具有以下结构的单个文件中：

        ?xml version="1.0" encoding="utf-8"?>
        <PublishData>
          <PublishProfile
            PublishMethod="AzureServiceManagementAPI"
            Url="https://management.core.windows.net/"
            ManagementCertificate="xxxxx">
            <Subscription
              Id="8a4a0a51-728e-482e-8daa-c477f03c541d"
              Name="hjereztest" />
          </PublishProfile>
        </PublishData>
        

其中 xxxxx 是[已编码文件]的内容，您将使用这些内容通过以下命令向 Linux Azure 命令行工具提供详细信息：
Azure 帐户导入（文件）

  [OpenSSL]: http://openssl.org/
  [从 publishsettings 文件获取管理证书]: #createcert
  [使用 Azure 管理门户安装管理证书]: #management
  [linuxcredentials]: ./media/linux-create-management-cert/linuxcredentials.png
  [VSpublish]: ./media/linux-create-management-cert/VSpublish.png
  [cmdlinepublish]: ./media/linux-create-management-cert/cmdlinepublish.png
  [http://www.fourmilab.ch/webtools/base64/base64.zip]: 
  [为 Azure 创建管理证书（可能为英文页面）]: http://msdn.microsoft.com/zh-cn/library/azure/gg551722.aspx
