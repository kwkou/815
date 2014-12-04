<properties linkid="configure-hyper-v-recovery-vault" urlDisplayName="configure-hyper-v-recovery-vault" pageTitle="配置 Hyper-V 恢复管理器以保护 VMM 云中的虚拟机" metaKeywords="hyper-v recovery, VMM, clouds, disaster recovery" description="Azure Hyper-V 恢复管理器通过协调位于 System Center 2012 VMM 私有云中的虚拟机的复制和恢复，来帮助您保护应用程序和服务。" metaCanonical="" umbracoNaviHide="0" disqusComments="1" title="配置 Azure Hyper-V 恢复管理器" editor="jimbe" manager="cfreeman" authors="" />

# 配置 Azure Hyper-V 恢复管理器

<div class="dev-callout"> 

<P>对于位于 System Center 2012 Service Pack 1 (SP1) 或 System Center 2012 R2 中的虚拟机管理器 (VMM) 服务器上的私有云中的 Hyper-V 虚拟机，Azure Hyper-V 恢复管理器负责协调和管理其保护事宜。HYPER-V 恢复管理器安排将这些虚拟机从一台本地 Hyper-V 主机服务器故障转移到另一台。Azure 中的 HYPER-V 恢复管理器保管库用于存储您的保护配置。</P>

<h2><a id="about"></a>关于本教程</h2>
<P>本教程提供了 Hyper-V 恢复管理器部署的快速演练。有关详细信息，请阅读以下内容：</P>
<UL>
<LI><a href="http://go.microsoft.com/fwlink/?LinkId=321294">Hyper-V 恢复管理器的规划指南</a> - 提供有关在开始完整部署之前您应该完成的规划步骤的详细信息。</LI>
<LI><a href="http://go.microsoft.com/fwlink/?LinkId=321295">Hyper-V 恢复管理器的部署指南</a> - 提供完整部署的分步说明。</LI>
<LI>如果您在本教程期间遇到问题，请查看 <a href="http://go.microsoft.com/fwlink/?LinkId=389879">Hyper-V 恢复管理器：常见错误情况和解决方法</a>或者在 <a href="http://social.msdn.microsoft.com/Forums/windowsazure/zh-cn/home?forum=hypervrecovmgr">Azure 恢复服务论坛</a>上发布您的问题</LI>
</UL>
</div>

## <span id="before"></span></a>开始之前

<div class="dev-callout"> 
<P>在开始此教程之前，请验证先决条件。</P>

<h3><a id="HVRMPrereq"></a>Hyper-V 恢复管理器先决条件</h3>

<UL>
<LI>**Azure 帐户** - 您将需要一个已启用 Azure 恢复服务功能的 Azure 帐户。如果您没有帐户或未注册该功能，请参阅 <a href="http://azure.microsoft.com/zh-cn/pricing/free-trial/?WT.mc_id=AB7B32386">Azure 免费试用版</a>和 <a href="http://azure.microsoft.com/zh-cn/pricing/details/recovery-manager/">Hyper-V 恢复管理器定价详细信息</a>。</LI>
<LI>**证书 (.cer)** - 若要在 Hyper-V 恢复管理器保管库中注册 VMM 服务器，您需要将包含公钥的管理证书 (.cer) 上载到该保管库。注意以下各项：<UL>
<LI>为了学习教程，您可以使用通过 Makecert.exe 工具创建的自签名证书。对于完整部署，您可以使用任何符合&ldquo;规划指南&rdquo;中的<a href=" http://msdn.microsoft.com/library/azure/dn469078.aspx">先决条件和支持</a>中所述的证书先决条件的有效 SSL 证书。</LI>
<LI>在任何时候，每个保管库都只有一个与其相关联的 .cer 证书。您可以根据需要上载新证书以覆盖现有证书。</LI>
</UL></LI>
<LI>**证书（.pfx 文件）** - .cer 证书必须导出为 .pfx 文件（包含私钥）。您将在包含要保护的虚拟机的每台 VMM 服务器上导入此文件。然后，在部署期间，如果您在每台 VMM 服务器上安装 Hyper-V 恢复管理器提供程序代理，则选择 .pfx 文件以向保管库注册 VMM 服务器。</LI>

</UL>

<h3><a id="VMMPrereq"></a>VMM 先决条件</h3>

<UL>
<LI>至少有一台在 System Center 2012 SP1 或 System Center 2012 R2 上运行的 VMM 服务器。</LI>
<LI>VMM 私有云。如果您要运行一台 VMM 服务器，它将需要配置两个云。对于两个或多个 VMM 服务器，您将需要至少一个您想要保护的源 VMM 服务器上的云，以及一个将用于恢复的目标 VMM 服务器上的云。您要保护的主要云必须包含以下内容：<UL>
<LI>一个或多个 VMM 主机组</LI>
<LI>每个主机组中的一个或多个 Hyper-V 主机服务器。</LI>
        </UL></LI>
<LI>如果您要使虚拟机在故障转移后连接到 VM 网络，则在 Hyper-V 恢复管理器中配置网络映射。在开始部署之前，请验证以下内容：<UL>
<LI>源 VMM 服务器上的云中的虚拟机已连接到 VM 网络。该 VM 网络应该链接到与该云相关联的逻辑网络。</LI>
<LI>目标 VMM 服务器上的目标云有相应的 VM 网络。该 VM 网络应该链接到与目标云相关联的相应逻辑网络。</LI>
    </UL></LI>
    
<P>若要了解有关网络映射的详细信息，请阅读&ldquo;规划指南&rdquo;中的<a href="http://go.microsoft.com/fwlink/?LinkId=324817">准备网络映射</a>。</P>
</UL>


<h2><a id="tutorial"></a>教程步骤</h2> 

在验证完先决条件后，请执行以下操作：
<UL>
<LI><a href="#createcert">获取并配置证书</a> - 获取一个 .cer 证书、将其导出为 .pfx 文件，并将 .pfx 文件导入到 VMM 服务器。</LI>
<LI><a href="#vault">步骤 1：创建保管库</a> - 在 Azure 中创建一个 Hyper-V 恢复管理器保管库。</LI>
<LI><a href="#upload">步骤 2：上载证书</a> - 将管理证书上载到保管库。</LI>
<LI><a href="#download">步骤 3：下载并安装提供程序</a> - 在要保护的 VMM 服务器上下载并安装 Hyper-V 恢复管理器提供程序。然后，向保管库注册 VMM 服务器。</LI>
<LI><a href="#clouds">步骤 4：配置云保护</a> - 为 VMM 云配置保护设置。</LI>
<LI><a href="#networks">步骤 5：配置网络映射</a> - 将 VM 网络从源 VMM 服务器映射到目标 VMM 服务器。</LI>
<LI><a href="#virtualmachines">步骤 6：为虚拟机启用保护</a> - 为位于 VMM 云中的 Hyper-V 虚拟机启用保护。</LI>
<LI><a href="#recovery plans">步骤 7：配置并运行恢复计划</a> - 创建和自定义将虚拟机组合在一起以进行故障转移的恢复计划。然后，运行该恢复计划。</LI>
<LI><a href="#jobs">步骤 8：监视</a> - 使用&ldquo;作业&rdquo;和&ldquo;仪表板&rdquo;选项卡监视设置、状态和进度。</LI>
</UL>



<a name="createcert"></a> <h2>获取和配置证书</h2>

按如下所示获取和配置证书：
<OL>
<LI><a href="#obtaincert">获取用于本演练的证书</a> - 使用 MakeCert 工具获取证书，或使用符合<a href="http://go.microsoft.com/fwlink/?LinkId=321294">证书要求</a>的现有证书。</LI>
<LI><a href="#exportcert">以 .pfx 格式导出证书</a> - 在证书所在或创建证书的服务器上，将 .cer 文件导出为 .pfx 文件（包含私钥）。 </LI>
<LI><a href="#importcert">将 .pfx 证书导入到 VMM 服务器</a> - 在完成 .pfx 文件导出后，将其导入到您要向保管库注册的每台 VMM 服务器上的个人证书存储中。</LI>
</OL>


<h3><a id="obtaincert"></a>获取自签名证书 (.cer)</h3>
<P>使用 MakeCert 工具创建符合所有证书要求的 .cer x.509 证书，如下所示：</P>
<ol>
<LI>
在您要运行 MakeCert 的计算机上，下载 <a href="http://msdn.microsoft.com/zh-cn/windows/desktop/aa904949">Windows SDK</a> 的最新版本。请注意，makecert.exe 命令是核心 Windows 软件开发工具包的一部分，因此您无需下载和安装整个 SDK。</LI>
<LI>在&ldquo;指定位置&rdquo;页上，选择&ldquo;将适用于 Windows 8.1 的 Windows 软件开发工具包安装到此计算机中&rdquo;。</LI>
<LI>在&ldquo;加入客户体验改善计划 (CEIP)&rdquo;页上，选择您的首选设置。</LI>
<LI>在&ldquo;许可协议&rdquo;页上，单击&ldquo;接受&rdquo;以接受条款。</LI>
<LI>在&ldquo;选择要安装的功能&rdquo;页上，清除&ldquo;Windows 软件开发工具包&rdquo;之外的所有选项。</LI>
<LI>在安装完成后，验证 makecert.exe 是否出现在文件夹 C:\ProgramFiles (x86)\Windows Kits\<i>WindowsVersion</i>\bin\x64 中。</LI>
<LI>使用管理员权限打开命令提示符 (cmd.exe) 并导航到 makecert.exe 文件夹。</LI> 
<LI>&gt;运行以下命令以创建自签名证书。将 CertificateName 替换为您希望用于该证书的名称，并在 -e 后指定证书的实际到期日期：</LI>
<code>
makecert.exe -r -pe -n CN=CertificateName -ss my -sr localmachine -eku 1.3.6.1.5.5.7.3.2 -len 2048 -e 01/01/2016 CertificateName.cer</code>
</ol>
<P>将在同一个文件夹中创建并存储该证书。您可能希望将它移动到一个更容易访问的位置，以便在下一步中将其导出。</P>

<h3><a id="exportcert"></a>以 .pfx 格式导出证书</h3>
<P>完成此过程中的步骤，以便以 .pfx 格式导出 .cer 文件。</P>
<ol>
<li>从&ldquo;开始&rdquo;屏幕中，键入&ldquo;mmc.exe&rdquo;以启动 Microsoft 管理控制台 (MMC)。</li>
<li>在&ldquo;文件&rdquo;菜单上，单击&ldquo;添加/删除管理单元&rdquo;。此时将显示&ldquo;添加或删除管理单元&rdquo;对话框。</li>
<li>在&ldquo;可用管理单元&rdquo;中，单击&ldquo;证书&rdquo;，然后单击&ldquo;添加&rdquo;。</li>
<li>选择&ldquo;计算机帐户&rdquo;，然后单击&ldquo;下一步&rdquo;。</li>
<li>选择&ldquo;本地计算机&rdquo;，然后单击&ldquo;完成&rdquo;。</li>
<li>在 MMC 的控制台树中，展开&ldquo;证书&rdquo;，然后展开&ldquo;个人&rdquo;。</li>
<li>在详细信息窗格中，单击您要管理的证书。</li>
<li>在&ldquo;操作&rdquo;菜单上，指向&ldquo;所有任务&rdquo;，然后单击&ldquo;导出&rdquo;。此时将显示&ldquo;证书导出向导&rdquo;。单击&ldquo;下一步&rdquo;。</li>
<li>在&ldquo;导出私钥&rdquo;页上，单击&ldquo;是&rdquo;以导出私钥。单击&ldquo;下一步&rdquo;。请注意，这只在您要在安装后将私钥导出到其他服务器时才是必需的。</li>
<li>在&ldquo;导出文件格式&rdquo;页上，选择&ldquo;个人信息交换 - PKCS #12 (.PFX)&rdquo;。单击&ldquo;下一步&rdquo;。</li>
<li>在&ldquo;密码&rdquo;页上，键入并确认用于加密私钥的密码。单击&ldquo;下一步&rdquo;。</li>
<li>按照向导上的页面以 .pfx 格式导出证书。</li>
</ol>


<h3><a id="importcert"></a>将 .pfx 证书导入到 VMM 服务器</h3>
<p>在导出服务器后，将其复制到每台要在保管库中注册的 VMM 服务器，然后导入它，如下所示。请注意，如果您在 VMM 服务器上运行 MakeCert.exe，则您无需在该服务器上导入证书。</p>
 
<ol>
<li>将私钥 (.pfx) 证书文件复制到本地服务器上的某个位置。</li>
<li>在证书 MMC 管理单元中，选择&ldquo;计算机帐户&rdquo;，然后单击&ldquo;下一步&rdquo;。</li>
<li>选择本地计算机，然后单击&ldquo;完成&rdquo;。在&ldquo;添加/删除管理单元&rdquo;对话框中，单击&ldquo;确定&rdquo;。 </li>
<li>在 MMC 中，展开&ldquo;证书&rdquo;、右键单击&ldquo;个人&rdquo;、指向&ldquo;所有任务&rdquo;，然后单击&ldquo;导入&rdquo;以启动证书导入向导。</li>
<li>在&ldquo;证书导入向导&rdquo;欢迎页上，单击&ldquo;下一步&rdquo;。</li>
<li>在&ldquo;要导入的文件&rdquo;页上，单击&ldquo;浏览&rdquo;并找到包含 .pfx 证书文件的文件夹，该文件包含要导入的证书。选择合适的文件，然后单击&ldquo;打开&rdquo;。</li>
<li>在&ldquo;密码&rdquo;页上的&ldquo;密码&rdquo;框中，键入您在前面步骤中指定的私钥文件的密码，然后单击&ldquo;下一步&rdquo;。</li>
<li>在&ldquo;证书存储&rdquo;页上，选择&ldquo;将所有证书放在以下存储中&rdquo;、单击&ldquo;浏览&rdquo;、选择&ldquo;个人&rdquo;存储、单击&ldquo;确定&rdquo;，然后单击&ldquo;下一步&rdquo;。</li>
<li>在&ldquo;完成证书导入向导&rdquo;页上，单击&ldquo;完成&rdquo;。</li>
</ol>

在完成这些步骤后，您将可以选择可供上载到保管库的 .cer 证书，也可以在提供程序安装期间注册 VMM 服务器时选择 .pfx 证书。
</div>

<a name="vault"></a>

## 步骤 1：创建保管库

</p>
1.  登录到[管理门户][管理门户]。

2.  单击“数据服务”，然后单击“恢复服务”。

    ![预览计划][预览计划]

3.  单击“恢复服务”、单击“新建”、指向“Hyper-V 恢复管理器保管库”，然后单击“快速创建”。

    ![新保管库][新保管库]

4.  在“名称”中，输入一个友好名称以标识此保管库。

5.  在“区域”中，为保管库选择地理区域。可用的地理区域包括亚太地区东南部、欧洲北部和美国东部。\*\*\*\*

6.  在“订阅”中，键入订阅详细信息。

7.  单击“创建保管库”。

在门户底部的通知中查看保管库状态。一条消息将确认已成功创建保管库。它将在“恢复服务”页上列为“联机”。

<a name="upload"></a>

## 步骤 2：上载证书 (.cer)

</p>
1.  在“恢复服务”页上，打开所需的保管库。
2.  单击“快速启动”图标以打开“快速启动”页。

    ![“快速启动”图标][“快速启动”图标]

3.  单击“管理证书”。

    ![快速启动][快速启动]

4.  在“管理证书”对话框中，单击“浏览文件”以找到要上载到保管库的 .cer 文件。

    ![管理证书][管理证书]

<a name="download"></a>

## 步骤 3：下载并安装提供程序

在每台要在保管库中注册的 VMM 服务器上安装 Hyper-V 恢复管理器提供程序。提供程序安装文件的最新版本存储在 Azure 下载中心中。当您在 VMM 服务器上运行该文件时，将安装提供程序，并且将向保管库注册 VMM 服务器。

</p>
1.  在“快速启动”页上，单击“下载提供程序”以获取提供程序安装 .exe 文件。在 VMM 服务器上运行此文件以开始提供程序安装。

    ![下载代理][下载代理]

2.  按照下列步骤操作完成提供程序安装。

    ![安装完成][安装完成]

3.  完成提供程序安装后，按照向导步骤向保管库注册 VMM 服务器。
4.  在“Internet 连接”页上，指定在 VMM 服务器上运行的提供程序如何连接到 Internet。提供程序可使用服务器上的默认 Internet 连接设置，或单击“使用代理服务器进行 Internet 请求”以使用自定义设置。

    ![Internet 设置][Internet 设置]

    如果您要在本演练期间使用自定义设置，请阅读部署指南的[步骤 2：安装提供程序并注册 VMM 服务器][步骤 2：安装提供程序并注册 VMM 服务器]中的信息。

5.  在“证书注册”页上，选择与您上载到保管库的 .cer 相对应的 .pfx 文件。

    ![证书注册][证书注册]

    ![证书保管库][证书保管库]

6.  在“VMM 服务器”页上，为 VMM 服务器指定友好名称。此名称用于在 Hyper-V 恢复管理器控制台中标识该服务器。
7.  选择“将云数据与保管库同步”以将位于 VMM 服务器上的所有私有云上的数据与 Hyper-V 恢复管理器保管库同步。此操作只需在每台服务器上发生一次。如果您不希望同步所有云，则可以单独发布每个云以同步它，然后再配置云保护设置。
8.  单击“注册”完成此过程。

    ![Internet 设置][1]

在此阶段，Hyper-V 恢复管理器将检索到来自 VMM 服务器的元数据，以安排故障转移和恢复。在服务器成功注册后，它的友好名称将显示在保管库中“服务器”页的“资源”选项卡上。

## <span id="clouds"></span></a>步骤 4：配置云保护设置

在注册 VMM 服务器后，您可以配置云保护设置。如果您在 VMM 服务器上安装提供程序时未启用选项“将云数据与保管库同步”，您需要将云从 VMM 控制台发布到 Hyper-V 恢复管理器。在 Hyper-V 恢复管理器检测到云后，您可以配置保护设置。

### <span id="publishclouds"></span></a>发布云

1.  在 VMM 控制台中，打开“VM 和服务”工作区。
2.  在“VM 和服务”窗格中，打开要发布的云。
3.  在云属性的“常规”页上，若要发布云，请选择“将关于此云的配置数据发送到 Azure Hyper-V 恢复管理器”。在发布云后，它将显示在保管库中。

    ![云][云]

    ![已发布的云][已发布的云]

### <span id="configureclouds"></span></a>配置云

要配置云以进行保护，请执行以下操作：

1.  在“快速启动”页上，单击“配置保护设置”。
2.  在“受保护的项”选项卡上，选择要配置的云并转到“配置”选项卡。

    ![云配置][云配置]

3.  在“目标位置”中，指定管理要用于恢复的云的 VMM 服务器。
4.  在“目标云”中，指定要用于源云中故障转移虚拟机的目标云。
5.  在“复制频率”中，保留默认设置。此值指定数据应在源和目标位置之间同步的频率。仅当 Hyper-V 主机运行 Windows Server 2012 R2 时它才相关。对于其他服务器，使用的默认设置为 5 分钟。
6.  在“其他恢复点”中，保留默认设置。此值指定您是否要创建其他恢复点。当默认值为零时，副本主机服务器上只存储主虚拟机的最新恢复点。
7.  在“应用程序一致的快照频率”中，保留默认设置。此值指定创建快照的频率。快照使用卷影复制服务 (VSS) 来确保应用程序在拍摄快照时处于一致状态。如果您确实希望为教程演练设置此值，请确保将它设置为小于您配置的其他恢复点的数量。
8.  在“数据传输压缩”中，指定是否应压缩所传输的复制数据。
9.  在“身份验证”中，指定如何在主服务器和恢复 Hyper-V 主机服务器之间对通信进行身份验证。除非您配置了有效的 Kerberos 环境，否则我们建议您选择 HTTPS 以用于本演练。选择 HTTPS 后，主机服务器将使用服务器证书对每个其他服务器进行身份验证，并且通信将加密。HYPER-V 恢复管理器将自动配置要用于 HTTPS 身份验证的证书。不需要手动配置。请注意，此设置仅适用于在 Windows Server 2012 R2 上运行的 Hyper-V 主机服务器。
10. 在“端口”中，保留默认设置。此值将设置源和目标 Hyper-V 主机计算机将侦听复制通信的端口号。
11. 在“初始复制设置”中，指定将如何处理从源位置到目标位置的初始数据复制过程，然后开始常规复制。

    -   **通过网络** — 通过网络复制数据可能非常耗时并占用大量资源。如果云包含的虚拟机所具有的虚拟硬盘相对较小，并且主要 VMM 服务器通过较宽的带宽连接到辅助 VMM 服务器，则我们建议您使用此选项。您可以指定应立即开始复制，或者选择一个复制时间。如果您使用网络复制，我们建议您将其安排在非高峰时段进行。
    -   **脱机** — 这种方法指定将使用外部媒体执行初始复制。如果您要避免网络性能下降或在地理上处于远程位置，则这种方法很有用。要使用这种方法，请在源云中指定导出位置，并在目标云中指定导入位置。当您为虚拟机启用保护时，虚拟硬盘将复制到指定的导出位置。您将其发送到目标站点，并将其复制到导入位置。系统将导入的信息复制到副本虚拟机。有关脱机复制先决条件的完整列表，请参阅“部署指南”中的[步骤 3：为 VMM 云配置保护设置][步骤 3：为 VMM 云配置保护设置]。

#### <span id="cloudsettingd"></span></a>配置保护后的设置

在配置云后，将配置已在源和目标云中配置的所有群集和主机服务器以进行复制。尤其是，配置以下各项：

-   Hyper-V 副本使用的防火墙规则，以便打开复制流量端口。
-   安装复制所需的证书。
-   配置 Hyper-V 副本设置。

可在“配置”选项卡上修改云设置。注意：

-   我们建议您选择可满足您要保护的虚拟机的恢复要求的目标云。
-   云只能属于单个云对 — 要么作为主云，要么作为目标云。
-   在保存云配置之后，将在“作业”选项卡上创建一个作业并监视此作业。在保存配置后，若要修改目标位置或目标云，您必须删除云配置，然后重新配置该云。

## <span id="networks"></span></a>步骤 5：配置网络映射

在“网络”选项卡上，配置源和目标 VMM 服务器上的 VM 网络之间的映射。网络映射确保在故障转移后，副本虚拟机将连接到相应的网络，并确保将副本虚拟机放置在可访问 VM 网络的主机上。验证“规划指南”的[先决条件和支持][先决条件和支持]中的 VMM 网络要求。我们建议您按如下所示映射网络：

-   映射配置为保护的每个云所使用的网络。
-   在启用虚拟机进行保护之前，请先执行网络映射，因为在放置副本虚拟机的过程中要使用映射。如果您在启用保护后配置映射，则映射会正常进行，但您可能需要迁移某些副本虚拟机。

要映射网络，请执行以下操作：

1.  在“快速启动”页上，单击“配置网络映射”。
2.  在“源位置”中，选择源 VMM 服务器。
3.  在“目标位置”中，选择目标 VMM 服务器。如果您要使用单个 VMM 服务器部署 Hyper-V 恢复管理器，则源和目标网络将相同。
    将显示源 VM 网络及其相关联的目标 VM 网络的列表。对于未映射的网络，将显示空值。
4.  选择源和目标列表中的未映射项，然后单击“映射”。服务将检测目标服务器上的 VM 网络并显示它们。

    ![管理证书][2]

5.  在“选择目标网络”页上，选择要在目标 VMM 服务器上使用的目标 VM 网络。
    ![目标网络][目标网络]

6.  单击源和目标网络名称旁边的信息图标，以查看每个网络的子网和类型。

    当您选择目标网络时，将显示使用源网络的受保护云，还显示与云关联的目标 VM 网络的可用性。我们建议您选择可用于进行保护的所有云的目标网络。

7.  单击复选标记以完成映射过程。一个作业将启动以跟踪映射进度。这可以在“作业”选项卡上查看。

    这会将与源 VM 网络相连的虚拟机相对应的任何现有副本虚拟机连接到目标 VM 网络。这也会将在与源 VM 网络相连的虚拟机上启用复制之后创建的新副本虚拟机连接到目标 VM 网络。

### <span id="modifynetworks"></span></a>修改网络映射

可以在“网络”选项卡上修改或删除网络映射。注意以下各项：

-   如果您取消一个所选的映射，则将删除此映射，并且副本虚拟机将从映射时所连接的网络断开。
-   当您修改所选映射时，更改将会更新，副本虚拟机将连接到所提供的新网络设置。

</p>
## <span id="virtualmachines"></span></a>步骤 6：为虚拟机启用保护

在正确配置服务器、云和网络后，您可以在云中为虚拟机启用保护。通过右键单击要保护的每个虚拟机并选择“启用恢复”，在 VMM 控制台中启用保护。

启用保护后，您可以在云的虚拟机列表中查看该虚拟机。您可以在“作业”选项卡中查看启用保护操作的进度。

![虚拟机][已发布的云]

当您启用虚拟机保护时，将创建三个作业。将运行“启用保护”作业。然后，在初始复制完成后，运行另外两个“完成保护”作业。仅当这三个作业成功完成后，虚拟机才会准备好进行故障转移。

## <span id="recoveryplans"></span></a>步骤 7：配置和运行恢复计划

恢复计划将虚拟机集合成组，以便可以将它们作为单个单元进行处理，并指定各个组应进行故障转移的顺序。除了定义虚拟机的组，您还可以自定义恢复计划以运行自动脚本，或等待确认手动操作。若要创建恢复计划，请执行以下操作：

1.  在“快速启动”页上，单击“创建恢复计划”。
2.  在“指定恢复页名称和目标”页上，键入名称以及源和目标 VMM 服务器。如果您要在单台 VMM 服务器上部署 Hyper-V 恢复管理器，则源和目标服务器将相同。源 VMM 服务器必须包含启用了保护的虚拟机。

    ![创建恢复计划][创建恢复计划]

3.  在“选择虚拟机”页上，选择要添加到恢复计划中的虚拟机。这些虚拟机将添加到恢复计划的默认组（组 1）中。
4.  单击复选标记可创建恢复计划。可在“恢复计划”选项卡上删除您创建的恢复计划。

    ![恢复计划 VM][恢复计划 VM]

在创建恢复计划后，您可以执行以下操作：

-   自定义计划。您可以向恢复计划中添加新组，也可以向组添加虚拟机。您可以以脚本形式定义自定义操作，并定义在指定的组前后要完成的手动操作。组名称是递增的。在默认恢复计划组 1 之后，您可以创建组 2，然后组 3，依次类推。虚拟机根据组顺序进行故障转移。有关自定义恢复计划的详细信息，请参阅“部署指南”中的[步骤 6：创建并自定义恢复计划][步骤 6：创建并自定义恢复计划]。
-   运行测试故障转移。

### <span id="run"></span></a>运行恢复计划

恢复计划可以作为主动测试或计划故障的转移的一部分运行，也可以在未计划的故障转移期间运行。本演练介绍如何运行测试故障转移。有关其他类型的故障转移的信息，请阅读[操作和监视指南][操作和监视指南]。

### <span id="protect"></span></a>运行计划。

运行测试故障转移以在测试环境中验证恢复计划。测试故障转移用于验证您的恢复计划和虚拟机故障转移测试是否按预期运行。测试故障转移在隔离的网络中模拟您的故障转移和恢复机制。

##### <span id="networkrecommendations"></span></a>开始之前

当触发测试故障转移时，您需要指定在故障转移后，测试虚拟机应如何连接到网络。

-   如果您要使用现有网络，我们建议您创建一个未在生产中使用的单独逻辑网络以实现此目的。
-   如果您选择该选项自动创建测试 VM 网络，则将在测试故障转移完成后，自动清理临时网络和测试虚拟机。
-   如果您使用基于虚拟 LAN (VLAN) 的逻辑网络，请确保隔离您添加到逻辑网络中的网站。
-   如果您使用基于 Windows 网络虚拟化的逻辑网络，Hyper-V 恢复管理器将自动创建隔离的 VM 网络。

</p>
##### <span id="runtest"></span></a>运行测试故障转移

按如下方式运行恢复计划的测试故障转移：

<ol>
<li>
在“恢复计划”选项卡上，选择所需的恢复计划。

</li>
<li>
若要启动故障转移，请单击“测试故障转移”按钮。

</li>
<li>
在“确认测试故障转移”页上，指定在测试故障转移后，虚拟机应如何连接到网络，如下所示：

</li>
-   “无”- 选择此设置以指定在测试故障转移中不应使用 VM 网络。如果您要测试各台虚拟机而不是网络配置，请使用此选项。它还提供测试故障转移功能如何工作的快速概览。测试虚拟机在故障转移后将不会连接到网络。
-   “使用现有”- 如果您已经创建并隔离了要用于测试故障转移的 VM 网络，则选择此选项。在故障转移后，测试故障转移中使用的所有测试虚拟机都将连接到在“VM 网络”中指定的网络。
-   “自动创建”- 选择此设置，以指定 Hyper-V 恢复管理器应基于您在“逻辑网络”中指定的设置自动创建 VM 网络及其相关的网站。如果恢复计划使用多个 VM 网络，则使用此选项。在使用 Windows 网络虚拟化网络的情况下，此选项可用于通过与副本虚拟机的网络相同的设置（子网和 IP 地址池）自动创建 VM 网络。在测试故障转移完成后，将自动清理这些 VM 网络。

</ol>
测试故障转移完成后，执行一下操作：

1.  验证虚拟机成功启动。
2.  在验证虚拟机成功启动后，完成测试故障转移以清理隔离的环境。如果您选择自动创建 VM 网络，则清理过程将删除所有测试虚拟机和测试网络。
3.  单击“说明”以记录并保存与测试故障转移相关联的任何观测结果。
4.  当为恢复计划启动测试故障转移时，故障转移过程将显示在恢复计划的详细信息页上。您可以看到各测试故障转移步骤及其状态，并查看或创建与测试故障转移关联的说明。此外，可以通过在“作业”选项卡上查看故障转移作业来查看故障转移作业的详细信息。详细信息将显示与故障转移关联的每个作业、其状态、启动作业的时间及其持续时间。您可以将故障转移列表中的作业导出到 Excel 电子表格中。

注意以下各项：

-   在“确认测试故障转移”页上，会显示将在其上创建测试虚拟机的 VMM 服务器的详细信息。
-   您可以在“作业”选项卡上跟踪测试故障转移作业的进度。

## <span id="jobsdashboard"></span></a>步骤 8：监视

### <span id="jobsdashboard"></span></a>使用“作业”选项卡和“仪表板”

#### <span id="jobs"></span></a>作业

“作业”选项卡显示由 Hyper-V 恢复管理器保管库执行的主要任务，包括为云配置保护、为虚拟机启用和禁用保护、运行故障转移（计划、非计划或测试）、提交非计划的故障转移以及配置反向复制。

在“作业”选项卡上，您可以执行以下操作：

-   **运行作业查询** — 您可以运行查询来检索与指定的条件相匹配的作业。您可以使用以下参数筛选作业：

-   查找在特定 VMM 服务器上运行的作业。
-   查找在云、虚拟机、网络、云中的恢复计划或所有这些之上执行的作业。
-   标识在特定日期和时间范围内发生的作业。

注意，查询最多可以返回 200 个作业，因此，我们建议您缩小查询参数范围以使返回的作业数小于最大值。

-   **获取作业详细信息** — 您可以单击“作业”列表中的某个作业以获取更多详细信息。详细信息包括作业名称、作业状态、开始时间以及持续时间。在“作业详细信息”选项卡上，您可以将作业详细信息导出到 Excel 电子表格，或者尝试重新启动失败、跳过或取消的作业。
-   **导出作业** - 您可以将作业查询的结果导出到 Excel 电子表格。
-   **重新启动** - 您可以重新启动失败的作业以尝试再次运行它们。
-   **错误详细信息** - 对于失败的作业，您可以单击“错误详细信息”以获得此作业的错误列表。系统将显示每个错误的可能原因和建议操作。可以将错误说明复制到剪贴板上以进行故障排除。

> “作业”选项卡提供以下信息：
>
> -   **名称** - 作业名称
> -   **项** - 云的名称、恢复计划、VMM 服务器或运行此作业的虚拟机。
> -   **状态** - 作业状态：已完成、已取消、失败或其他
> -   **类型** - 作业类型
> -   **启动时间** - 启动作业的时间
> -   **任务时间** - 作业持续时间
>
> </p>

![“作业”选项卡][“作业”选项卡]

#### <span id="protect"></span></a>运行作业查询

您可以运行查询来检索与指定的条件相匹配的作业，包括：

-   在特定 VMM 服务器上运行的作业。
-   在特定云、虚拟机、网络、恢复计划或所有这些之上执行的作业。
-   在特定时间范围内发生的作业。

</p>
若要运行作业查询，请执行以下操作：

1.  打开“作业”选项卡
2.  在“服务器”中，指定运行了此作业的 VMM 服务器
3.  在“类型”中，指定此作业是在云、虚拟机、网络中还是在云中的恢复计划上执行的。
4.  在“状态”中，指定作业状态。
5.  在“持续时间”中，指定作业发生的时间范围。

</p>
#### <span id="dashboard"></span></a>仪表板

“仪表板”选项卡提供有关贵组织中 Hyper-V 恢复管理器使用情况的快速概览。它提供了一个集中化的门户，让您可以查看受 Hyper-V 恢复管理器保管库保护的虚拟机。“仪表板”提供了以下功能：

在“仪表板”选项卡上，您可以执行以下任务：

-   **下载提供程序** - 下载 Hyper-V 恢复管理器提供程序以便安装到 VMM 服务器上。
-   **管理证书** - 修改与保管库相关联的证书的设置。
-   **删除** - 删除保管库。
-   **重新同步虚拟机** - 如果 Hyper-V 恢复管理器检测到任何主虚拟机和恢复虚拟机未按预期同步，则您可以查看相关虚拟机的列表，并从“仪表板”中选择您要尝试重新同步的任何虚拟机。

“仪表板”选项卡提供了以下信息：

-   </strong>速览<strong> — 显示有关恢复服务和 Hyper-V 恢复管理器保管库的关键配置信息。您可以了解保管库是否联机、向保管库分配了哪个证书、证书何时到期以及服务的订阅详细信息。
-   </strong>最近的作业<strong> - 显示过去 24 小时内成功或失败的作业，或者正在进行或等待操作的作业。
-   </strong>问题\*\* - 仪表板显示有关 VMM 服务器连接问题、云配置设置问题或虚拟机复制同步问题的信息。您可以获得有关问题的更多详细信息、查看与此问题相关联的作业或尝试重新同步虚拟机。

</p>
![仪表板][仪表板]

## <span id="next"></span></a>后续步骤

-   若要在完全生产环境中规划和部署 Hyper-V 恢复管理器，请参阅 [Hyper-V 恢复管理器的规划指南][Hyper-V 恢复管理器的规划指南]和 [Hyper-V 恢复管理器的部署指南][Hyper-V 恢复管理器的部署指南]。
-   有关问题，请访问 [Azure 恢复服务论坛][3]。

  [Hyper-V 恢复管理器的规划指南]: http://go.microsoft.com/fwlink/?LinkId=321294
  [Hyper-V 恢复管理器的部署指南]: http://go.microsoft.com/fwlink/?LinkId=321295
  [Azure 恢复服务论坛]: http://social.msdn.microsoft.com/Forums/windowsazure/zh-cn/home?forum=hypervrecovmgr
  [先决条件和支持]: http://msdn.microsoft.com/library/azure/dn469078.aspx
  [管理门户]: https://manage.windowsazure.cn
  [预览计划]: ./media/hyper-v-recovery-manager-configure-vault/RS_PreviewProgram.png
  [新保管库]: ./media/hyper-v-recovery-manager-configure-vault/RS_hvvault.png
  [“快速启动”图标]: ./media/hyper-v-recovery-manager-configure-vault/RS_QuickStartIcon.png
  [快速启动]: ./media/hyper-v-recovery-manager-configure-vault/RS_QuickStart.png
  [管理证书]: ./media/hyper-v-recovery-manager-configure-vault/RS_ManageCert.png
  [下载代理]: ./media/hyper-v-recovery-manager-configure-vault/RS_installwiz.png
  [安装完成]: ./media/hyper-v-recovery-manager-configure-vault/RS_SetupComplete.png
  [Internet 设置]: ./media/hyper-v-recovery-manager-configure-vault/RS_ProviderProxy.png
  [步骤 2：安装提供程序并注册 VMM 服务器]: http://go.microsoft.com/fwlink/?LinkId=378266
  [证书注册]: ./media/hyper-v-recovery-manager-configure-vault/RS_CertReg1.png
  [证书保管库]: ./media/hyper-v-recovery-manager-configure-vault/RS_CertReg2.png
  [1]: ./media/hyper-v-recovery-manager-configure-vault/RS_PublishCloudSetup.png
  [云]: ./media/hyper-v-recovery-manager-configure-vault/RS_PublishCloud.png
  [已发布的云]: ./media/hyper-v-recovery-manager-configure-vault/RS_Clouds.png
  [云配置]: ./media/hyper-v-recovery-manager-configure-vault/RS_CloudConfig.png
  [步骤 3：为 VMM 云配置保护设置]: http://go.microsoft.com/fwlink/?LinkId=323469
  [2]: ./media/hyper-v-recovery-manager-configure-vault/RS_networks.png
  [目标网络]: ./media/hyper-v-recovery-manager-configure-vault/RS_TargetNetwork.png
  [创建恢复计划]: ./media/hyper-v-recovery-manager-configure-vault/RS_RecoveryPlan1.png
  [恢复计划 VM]: ./media/hyper-v-recovery-manager-configure-vault/RS_RecoveryPlan2.png
  [步骤 6：创建并自定义恢复计划]: http://go.microsoft.com/fwlink/?LinkId=378271
  [操作和监视指南]: http://msdn.microsoft.com/library/azure/dn440569.aspx
  [“作业”选项卡]: ./media/hyper-v-recovery-manager-configure-vault/RS_Jobs.png
  [仪表板]: ./media/hyper-v-recovery-manager-configure-vault/RS_Dashboard.png
  [3]: http://go.microsoft.com/fwlink/?LinkId=313628
