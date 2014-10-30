<properties title="灵活扩展安全配置" pageTitle="灵活扩展安全配置" description="使用 Azure SQL Database 灵活扩展的拆分/合并服务安全" metaKeywords="Elastic Scale Security Configurations, Azure SQL Database sharding, elastic scale " services="sql-database" documentationCenter="" manager="jhubbard" authors="sidneyh@microsoft.com"/>

<tags ms.service="sql-database" ms.workload="sql-database" ms.tgt_pltfrm="na" ms.devlang="na" ms.topic="article" ms.date="10/02/2014" ms.author="sidneyh"></tags>

# 灵活扩展安全配置

Microsoft Azure SQL Database 灵活扩展包括自托管服务。此分发包括服务配置文件，该文件包含必须配置的安全相关设置。

1.  [配置证书][配置证书]
2.  [允许的 IP 地址][允许的 IP 地址]
3.  [防止拒绝服务][防止拒绝服务]
4.  [其他安全注意事项][其他安全注意事项]

## <a name="configuring-certificates"></a>配置证书

通过两种方式配置证书。

1.  [配置 SSL 证书][配置 SSL 证书]
2.  [配置客户端证书][配置客户端证书]

## <a name="obtain-certificates"></a>获取证书

可从公共证书颁发机构 (CA) 或 [Windows 证书服务][Windows 证书服务]中获取证书。这些方法是获取证书的首选方法。

如果这些选项不可用，您可以生成“自签名证书”。

## <a name="tools"></a>用于生成证书的工具

-   [makecert.exe][makecert.exe]
-   [pvk2pfx.exe][pvk2pfx.exe]

### 运行工具

-   有关适用于 Visual Studio 的开发人员命令提示符，请参阅 [Visual Studio 命令提示符][Visual Studio 命令提示符]

    如果已安装该工具，请转到：

        %ProgramFiles(x86)%\Windows Kits\x.y\bin\x86 

-   从以下位置获取 WDK：[Windows 8.1：下载工具包和工具][Windows 8.1：下载工具包和工具]

## <a name="to-configure-ssl-cert"></a>配置 SSL 证书

若要对通信进行加密并对服务器进行身份验证，需要使用 SSL 证书。从下面的三个方案中选择最适合的方案，然后执行其所有步骤：

### 创建新的自签名证书

1.  [创建自签名证书][创建自签名证书]
2.  [为自签名 SSL 证书创建 PFX 文件][为自签名 SSL 证书创建 PFX 文件]
3.  [将 SSL 证书上载到云服务][将 SSL 证书上载到云服务]
4.  [在服务配置文件中更新 SSL 证书][在服务配置文件中更新 SSL 证书]
5.  [导入 SSL 证书颁发机构][导入 SSL 证书颁发机构]

#### 使用证书存储中的现有证书

1.  [从证书存储中导出 SSL 证书][从证书存储中导出 SSL 证书]
2.  [将 SSL 证书上载到云服务][将 SSL 证书上载到云服务]
3.  [在服务配置文件中更新 SSL 证书][在服务配置文件中更新 SSL 证书]

#### 在 PFX 文件中使用现有证书

1.  [将 SSL 证书上载到云服务][将 SSL 证书上载到云服务]
2.  [在服务配置文件中更新 SSL 证书][在服务配置文件中更新 SSL 证书]

## <a name="configuring-client-certs"></a>配置客户端证书

若要对服务请求进行身份验证，需要使用客户端证书。从下面的三个方案中选择最适合的方案，然后执行其所有步骤：

### 禁用客户端证书

1.  [禁用基于客户端证书的身份验证][禁用基于客户端证书的身份验证]

### 颁发新的自签名客户端证书

1.  [创建自签名证书颁发机构][创建自签名证书颁发机构]
2.  [将 CA 证书上载到云服务][将 CA 证书上载到云服务]
3.  [在服务配置文件中更新 CA 证书][在服务配置文件中更新 CA 证书]
4.  [颁发客户端证书][颁发客户端证书]
5.  [为客户端证书创建 PFX 文件][为客户端证书创建 PFX 文件]
6.  [导入客户端证书][导入客户端证书]
7.  [复制客户端证书指纹][复制客户端证书指纹]
8.  [在服务配置文件中配置允许的客户端][在服务配置文件中配置允许的客户端]

### 使用现有客户端证书

1.  [查找 CA 公钥][查找 CA 公钥]
2.  [将 CA 证书上载到云服务][将 CA 证书上载到云服务]
3.  [在服务配置文件中更新 CA 证书][在服务配置文件中更新 CA 证书]
4.  [复制客户端证书指纹][复制客户端证书指纹]
5.  [在服务配置文件中配置允许的客户端][在服务配置文件中配置允许的客户端]
6.  [配置客户端证书吊销检查][配置客户端证书吊销检查]

## <a name="allowed-ip-addresses"></a>允许的 IP 地址

可将对服务终结点的访问限制为特定范围的 IP 地址。

## 默认配置

默认配置拒绝对 HTTP 终结点的所有访问。这是建议设置。因为对这些终结点的请求可能带有敏感信息（如数据库凭据）。
默认配置允许对 HTTPS 终结点的所有访问。可能会进一步限制此设置。

### 更改配置

在“服务配置文件”的 **<endpointacls>** 部分中配置应用的访问控制组规则和终结点。

    <EndpointAcls>
      <EndpointAcl role="SplitMergeWeb" endPoint="HttpIn" accessControl="DenyAll" />
      <EndpointAcl role="SplitMergeWeb" endPoint="HttpsIn" accessControl="AllowAll" />
    </EndpointAcls>

在服务配置文件的 <accesscontrol name> 部分中配置访问控制组中的规则。

在网络访问控制列表文档中介绍了格式。
例如，若要仅允许范围 100.100.0.0 到 100.100.255.255 中的 IP 访问 HTTPS 终结点，规则应如下所示：

    <AccessControl name="Retricted">
      <Rule action="permit" description="Some" order="1" remoteSubnet="100.100.0.0/16"/>
      <Rule action="deny" description="None" order="2" remoteSubnet="0.0.0.0/0" />
    </AccessControl>
    <EndpointAcls>
    <EndpointAcl role="SplitMergeWeb" endPoint="HttpsIn" accessControl="Restricted" />

## <a name="denial-of-service-prevention"></a>防止拒绝服务

可使用两种受支持的不同机制检测和防止拒绝服务攻击：

-   限制每台远程主机的并发请求数（默认为禁用）
-   限制每台远程主机的访问率（默认为启用）

这些机制还基于在 IIS 的动态 IP 安全中记录的功能。更改此配置时，请注意以下因素：

-   通过远程主机信息的代理和网络地址转换设备的行为
-   考虑对 Web 角色中任何资源的每个请求（例如，加载脚本、图像等）

## 限制并发访问数

配置此行为的设置如下：

    <Setting name="DynamicIpRestrictionDenyByConcurrentRequests" value="false" />
    <Setting name="DynamicIpRestrictionMaxConcurrentRequests" value="20" />

将 DynamicIpRestrictionDenyByConcurrentRequests 更改为 true 以启用此保护。

## 限制访问率

配置此行为的设置如下：

    <Setting name="DynamicIpRestrictionDenyByRequestRate" value="true" />
    <Setting name="DynamicIpRestrictionMaxRequests" value="100" />
    <Setting name="DynamicIpRestrictionRequestIntervalInMilliseconds" value="2000" />

## 配置对拒绝请求的响应

以下设置将配置对拒绝请求的响应：

    <Setting name="DynamicIpRestrictionDenyAction" value="AbortRequest" />

有关其他受支持的值，请参考 IIS 中动态 IP 安全文档。

## 用于配置服务证书的操作

本主题仅供参考。请遵循以下部分中概括的配置步骤：

-   配置 SSL 证书
-   配置客户端证书

## <a name="create-self-signed-cert"></a>创建自签名证书

执行：

    makecert ^
      -n "CN=myservice.cloudapp.net" ^
      -e MM/DD/YYYY ^
      -r -cy end -sky exchange -eku "1.3.6.1.5.5.7.3.1" ^
      -a sha1 -len 2048 ^
      -sv MySSL.pvk MySSL.cer

自定义：

-   -n，带有服务 URL。通配符 ("CN=\*.cloudapp.net") 和替代名称 ("CN=myservice1.cloudapp.net, CN=myservice2.cloudapp.net") 均受支持。
-   -e，带有证书过期日期
    创建强密码并在提示时指定它。

## <a name="create-pfx-for-self-signed-cert"></a>为自签名 SSL 证书创建 PFX 文件

执行：

        pvk2pfx -pvk MySSL.pvk -spc MySSL.cer

输入密码，然后使用以下选项导出证书：

-   是，导出私钥
-   导出所有扩展属性

## <a name="export-ssl-from-store"></a>从证书存储中导出 SSL 证书

-   查找证书
-   依次单击“操作”-\>“所有任务”-\>“导出…”
-   使用以下选项将证书导出到 .PFX 文件中：

    -   是，导出私钥
    -   包括证书路径中的所有证书（如果可能）
        \*导出所有扩展属性

## <a name="upload-ssl"></a>将 SSL 证书上载到云服务

使用带有 SSL 密钥对的现有或生成的 .PFX 文件上载证书：

-   输入用于保护私钥信息的密码

## <a name="update-ssl-in-csfg"></a>在服务配置文件中更新 SSL 证书

在服务配置文件中，使用已上载到云服务的证书指纹更新以下设置的指纹值：

    <Certificate name="SSL" thumbprint="" thumbprintAlgorithm="sha1" />

## <a name="import-ssl-ca"></a>导入 SSL 证书颁发机构

在将与该服务通信的所有帐户/计算机中，按照以下步骤进行操作：

-   在 Windows 资源管理器中，双击 .CER 文件
-   在“证书”对话框中，单击“安装证书…”
-   将证书导入到“受信任的根证书颁发机构”存储中

## <a name="turn-off-client-cert"></a>禁用基于客户端证书的身份验证

仅支持基于客户端证书的身份验证，禁用它即可公开访问服务终结点，除非使用了其他机制（例如 Microsoft Azure 虚拟网络）。

在服务配置文件中，将这些设置更改为 false 以关闭该功能：

    <Setting name="SetupWebAppForClientCertificates" value="false" />
    <Setting name="SetupWebserverForClientCertificates" value="false" />

然后，复制与 CA 证书设置中 SSL 证书相同的指纹：

    <Certificate name="CA" thumbprint="" thumbprintAlgorithm="sha1" />

## <a name="create-self-signed-ca"></a>创建自签名证书颁发机构

执行以下步骤来创建自签名证书，以充当证书颁发机构：

    makecert ^
    -n "CN=MyCA" ^
    -e MM/DD/YYYY ^
     -r -cy authority -h 1 ^
     -a sha1 -len 2048 ^
      -sr localmachine -ss my ^
      MyCA.cer

对其进行自定义

-   -e，带有证书到期日期

## <a name="find-ca-public-key"></a>查找 CA 公钥

所有客户端证书都必须由服务信任的证书颁发机构颁发。为了将证书上载到云服务，需要查找颁发了客户端证书（将用于身份验证）的证书颁发机构提供的公钥。

如果具有公钥的文件不可用，则将其从证书存储中导出：

-   查找证书

    -   搜索同一证书颁发机构颁发的客户端证书
-   双击该证书
-   在“证书”对话框中选择“证书路径”选项卡
-   双击路径中的 CA 条目
-   记下证书属性
-   关闭“证书”对话框
-   查找证书

    -   搜索前面记下的 CA
-   依次单击“操作”-\>“所有任务”-\>“导出…”
-   使用以下选项将证书导出到 .CER 中：

    -   否，不导出私钥
    -   包括证书路径中的所有证书（如果可能）
    -   导出所有扩展属性

## <a name="upload-ca-cert"></a>将 CA 证书上载到云服务

使用带有 CA 公钥的现有或生成的 .CER 文件上载证书。

## <a name="update-ca-in-csft"></a>在服务配置文件中更新 CA 证书

在服务配置文件中，使用已上载到云服务的证书指纹更新以下设置的指纹值：

    <Certificate name="CA" thumbprint="" thumbprintAlgorithm="sha1" />

使用同一指纹更新以下设置的值：

    <Setting name="AdditionalTrustedRootCertificationAuthorities" value="" />

## <a name="issue-client-certs"></a>颁发客户端证书

授予了访问服务权限的每个用户都应具有一个颁发的客户端证书供其独占使用，并且应选择自己的强密码来包含其私钥。

必须在生成和存储了自签名 CA 证书的同一计算机上执行以下步骤：

    makecert ^
      -n "CN=My ID" ^
      -e MM/DD/YYYY ^
      -cy end -sky exchange -eku "1.3.6.1.5.5.7.3.2" ^
      -a sha1 -len 2048 ^
      -in "MyCA" -ir localmachine -is my ^
      -sv MyID.pvk MyID.cer

自定义：

-   -n，带有将使用此证书进行身份验证的客户端的 ID
-   -e，带有证书过期日期
-   MyID.pvk 和 MyID.cer，带有用于此客户端证书的唯一文件名

此命令将提示创建密码，然后使用一次该密码。使用强密码。

## <a name="create-pfx-files"></a>为客户端证书创建 PFX 文件

针对每个生成的客户端证书，执行：

    pvk2pfx -pvk MyID.pvk -spc MyID.cer

自定义：

    •    MyID.pvk and MyID.cer with the filename for the client certificate

输入密码，然后使用以下选项导出证书：

-   是，导出私钥
-   导出所有扩展属性
-   将向其颁发此证书的单个用户应选择导出密码

## <a name="import-client-cert"></a>导入客户端证书

为其颁发了客户端证书的每个用户都应将密钥对导入到将用于与服务通信的计算机中：

-   在 Windows 资源管理器中，双击 .PFX 文件
-   至少使用以下选项将证书导入到个人存储中：

    -   包括选中的所有扩展属性

## <a name=copy-client-cert"></a>复制客户端证书指纹</h2> 

<p>为其颁发了客户端证书的每个用户都必须遵循以下步骤，才能获取将添加到服务配置文件的证书的指纹：</p> 

<ul> 
<li>运行 certmgr.exe</li> <li>选择“个人”选项卡</li> 
<li>双击要用于身份验证的客户端证书</li> <li>在打开的“证书”对话框中，选择“详细信息”选项卡</li> 
<li>确保“显示”可显示全部内容</li> 
<li>选择列表中名为“Thumbprint”的字段</li> 
<li>复制指纹的值<br />** 删除第一个数字前不可见的 Unicode 字符<br />** 删除所有空格</li> 
</ul> 

<h2 id="configure-allowed-clients-in-the-service-configuration-file"><a name="configure-allowed-client"></a>在服务配置文件中配置允许的客户端</h2> 

<p>在服务配置文件中，使用以逗号分隔的客户端证书（允许访问服务）的指纹列表更新以下设置的值：</p> 

	<Setting name="AllowedClientCertificateThumbprints" value="" />

<h2 id="configure-client-certificate-revocation-check"><a name="configure-client-revocation"></a>配置客户端证书吊销检查</h2> 

<p>默认设置不会通过证书颁发机构检查客户端证书吊销状态。若要启用检查，请在颁发了客户端证书的证书颁发机构支持此类检查时，使用在 X509RevocationMode 枚举中定义的值之一更改以下设置：</p> 

	Setting name="ClientCertificateRevocationCheck" value="NoCheck" />

<h2 id="common-certificate-operations">常见证书操作</h2> 

<p>• 配置 SSL 证书<br />
• 配置客户端证书</p> 

<h2 id="find-certificate">查找证书</h2> 

<p>按照以下步骤进行操作：</p> 

<ol> <li>运行 mmc.exe。</li> 
<li>“文件”->“添加/删除管理单元…”</li> 
<li>选择证书</li> 
<li>单击“添加”</li> 
<li>选择证书存储位置</li> 
<li>单击“完成”</li> 
<li>单击“确定”</li> 
<li>展开证书</li> 
<li>展开证书存储</li> 
<li>展开证书子节点</li> 
<li>在右侧的列表中选择某个证书</li> 
</ol> 

<h2 id="export-certificate">导出证书</h2> 

<p>在证书导出向导中：</p> 
<ol> 
<li>单击“下一步”</li> 
<li>选择“是，导出私钥”</li> 
<li>单击“下一步”</li> 
<li>选择所需的输出文件格式</li> 
<li>检查所需选项</li> 
<li>检查密码</li> 
<li>输入强密码并进行确认</li> 
<li>单击“下一步”</li> 
<li>在证书的存储位置键入或浏览文件名（使用 .PFX 扩展名）</li> 
<li>单击“下一步”</li> 
<li>单击“完成”</li> 
<li>单击“确定”</li> 
</ol> 

<h2 id="import-certificate">导入证书</h2> 

<p>在证书导入向导中：</p> 

<ol> 
<li>
<p>选择存储位置。</p> 
<ol> 
<li>当前用户（如果在当前用户下运行的进程将访问该服务）</li> 
<li>本地计算机（如果此计算机中的其他进程将访问该服务）</li> 
</ol>
</li> <li>单击“下一步”</li> 
<li>如果要从文件中导入，请确认文件路径</li> 
<li><p>如果要从 .PFX 文件中导入：</p> 
<ol> 
<li>输入用于保护私钥的密码</li> 
<li>选择导入选项</li> 
</ol>
</li> 
<li>选择“将证书放入以下存储”</li> 
<li>单击“浏览”</li> 
<li>选择所需存储</li> 
<li><p>单击“完成”</p> 
<ol> 
<li>如果已选中“受信任的根证书颁发机构”存储，请单击“是”</li> 
</ol>
</li> 
<li>
<p>在所有对话窗口上单击“确定”</p>
</li> 
</ol> 
<h2 id="upload-certificate"><a name="upload-certificate"></a>上载证书</h2> 
<p>在 <a href="http://manage.windowsazure.com/">Azure 管理门户中</a></p>
<ol> 
<li>选择云服务</li> 
<li>选择云服务</li> 
<li>单击顶部菜单上的“证书”</li> 
<li>单击底部栏中的“上载”</li> 
<li>选择证书文件</li> 
<li>如果是 .PFX 文件，则输入私钥密码</li> 
<li>完成操作后，从列表中的新条目复制证书指纹</li> 
</ol> 

<h1 id="other-security-considerations"><a name="other-security"></a>其他安全注意事项</h1> 

<p>使用 HTTPS 终结点时，本文档中介绍的 SSL 设置将对服务及其客户端之间的通信进行加密。这一点很重要，因为该通信中包含了数据库访问凭据以及其他可能的敏感信息。但是，请注意，该服务会将内部状态（包括凭据）保留在其内部表中，该表位于您在 Microsoft Azure 订阅中为元数据存储提供的 Microsoft Azure SQL Database 中。在服务配置文件（.CSCFG 文件）中，该数据库已定义为以下设置的一部分：</p> 
	<Setting name="ElasticScaleMetadata" value="Server=…" />

<p>不会对存储在此数据库中的数据进行加密。若要避免从服务请求中公开凭据或其他敏感信息，请保护此数据库并总是以安全方式访问它。此外，由于服务部署的 Web 角色和辅助角色都可以访问元数据库，因此请同样确保这两个角色是最新且安全的。</p>

[AZURE.INCLUDE [elastic-scale-include](../includes/elastic-scale-include.md)]

  [配置证书]: #configuring-certificates
  [允许的 IP 地址]: #allowed-ip-addresses
  [防止拒绝服务]: #denial-of-service-prevention
  [其他安全注意事项]: #other-security
  [配置 SSL 证书]: #to-configure-ssl-cert
  [配置客户端证书]: #configuring-client-certs
  [Windows 证书服务]: http://msdn.microsoft.com/en-us/library/windows/desktop/aa376539.aspx
  [makecert.exe]: http://msdn.microsoft.com/en-us/library/bfsktky3.aspx
  [pvk2pfx.exe]: http://msdn.microsoft.com/en-us/library/windows/hardware/ff550672.aspx
  [Visual Studio 命令提示符]: http://msdn.microsoft.com/en-us/library/ms229859.aspx
  [Windows 8.1：下载工具包和工具]: http://msdn.microsoft.com/en-US/windows/hardware/gg454513#drivers
  [创建自签名证书]: #create-self-signed-cert
  [为自签名 SSL 证书创建 PFX 文件]: #create-pfx-for-self-signed-cert
  [将 SSL 证书上载到云服务]: #upload-ssl
  [在服务配置文件中更新 SSL 证书]: #update-ssl-in-csfg
  [导入 SSL 证书颁发机构]:  "import-ssl-ca"
  [从证书存储中导出 SSL 证书]: #export-ssl-from-store
  [禁用基于客户端证书的身份验证]: #turn-off-client-cert
  [创建自签名证书颁发机构]: #create-self-signed-ca
  [将 CA 证书上载到云服务]: #upload-ca-cert
  [在服务配置文件中更新 CA 证书]: #update-ca-in-csft
  [颁发客户端证书]: #issue-client-certs
  [为客户端证书创建 PFX 文件]: #create-pfx-files
  [导入客户端证书]: #import-client-cert
  [复制客户端证书指纹]: #copy-client-cert
  [在服务配置文件中配置允许的客户端]: #configure-allowed-client
  [查找 CA 公钥]: #find-ca-public-key
  [配置客户端证书吊销检查]: #configure-client-revocation
