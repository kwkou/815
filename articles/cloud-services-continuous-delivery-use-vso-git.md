<properties linkid="dev-net-common-tasks-publishing-with-vso" urlDisplayName="Publishing with TFS" pageTitle="使用 Visual Studio Online 在 Azure 中持续交付" metaKeywords="" description="了解如何将 Visual Studio Online 团队项目配置为自动生成并部署到 Azure 网站或云服务。" metaCanonical="" services="web-sites" documentationCenter=".NET" title="使用 Visual Studio Online 和 Git 向 Azure 持续传送项目" authors="ghogen" solutions="" manager="" editor="" />

# 使用 Visual Studio Online 和 Git 向 Azure 持续传送项目

可以使用 Visual Studio Online 团队项目为您的源代码托管 Git 存储库，并在将提交推送到存储库时自动生成并部署到 Azure 网站或云服务。

您将需要安装 Visual Studio 2013 和 Azure SDK。如果你尚未安装 Visual Studio 2013，请在 [www.visualstudio.com][www.visualstudio.com] 上选择“免费试用”链接以下载该软件。从[此处][此处]安装 Azure SDK。

若要使用 Visual Studio Online 将云服务设置为自动生成并部署到 Azure，请执行下列步骤：

-   [步骤 1：注册 Visual Studio online 并创建一个 Git 存储库。][步骤 1：注册 Visual Studio online 并创建一个 Git 存储库。]

-   [步骤 2：创建一个项目并将其推送到 Git 存储库。][步骤 2：创建一个项目并将其推送到 Git 存储库。]

-   [步骤 3：将项目连接到 Azure。][步骤 3：将项目连接到 Azure。]

-   [步骤 4：做出更改并触发重新生成和重新部署。][步骤 4：做出更改并触发重新生成和重新部署。]

-   [步骤 5：重新部署以前的生成（可选）][步骤 5：重新部署以前的生成（可选）]

-   [步骤 6：更改生产部署][步骤 6：更改生产部署]

-   [步骤 7：从工作分支进行部署][步骤 7：从工作分支进行部署]

## <a name="step1"></a><span class="short-header">步骤 1：注册 Visual Studio online 并创建一个 Git 存储库。</span>第 1 步：注册 Visual Studio online 并创建一个 Git 存储库

1.  如果还没有 Visual Studio Online 帐户，请按照[此处][1]的说明操作。

2.  为新项目创建帐户 URL，使用以下格式：https://\<accountname\>.visualstudio.com.
    ![][0]

3.  现在，你可以创建自己的第一个项目。输入项目名称和说明。选择 Git 作为源代码控制系统。然后选择您的组织使用的过程模板，并选择**创建项目**按钮。有关过程模板的详细信息，请参阅[处理团队项目组件，选择过程模板][处理团队项目组件，选择过程模板]。
    ![][2]

4.  选择**在 Visual Studio 中打开以连接**按钮，自动启动连接到您的团队项目的 Visual Studio。如果显示了任何安全对话框，请选择**允许**。
    ![][3]

5.  在团队资源管理器中，选择**克隆此存储库**链接。
    ![][4]

6.  指定本地副本的位置，然后选择**克隆**按钮。

## <a name="step2"> </a><span class="short-header">创建一个项目并将其提交到存储库。</span>第 2 步：创建一个项目并将其提交到存储库

1.  在团队资源管理器的“解决方案”部分，选择新的链接在本地存储库中创建一个新项目。
    ![][5]

2.  可以按照本演练中的步骤部署网站或云服务（Azure 应用程序）。
    创建新的 Windows Azure 云服务项目，
    或新的 ASP.NET MVC 项目。确保该项目针对.NET Framework 4 或 4.5，如果要创建一个云服务项目，请添加 ASP.NET MVC web 角色和辅助角色。
    如果要创建一个网站，请选择 ASP.NET Web 应用程序项目模板，然后选择 MVC。请参阅 [Azure 和 ASP.NET 入门][Azure 和 ASP.NET 入门]。

3.  打开解决方案的快捷菜单，然后选择**提交**。
    ![][6]

4.  如果这是您首次在 Visual Studio Online 中使用 Git，则需要提供一些信息，以在 Git 中对自己进行标识。在团队资源管理器的**挂起更改**区域中，输入您的用户名和电子邮件地址。键入用于提交的注释，然后选择**提交**。
    ![][7]

5.  记下在签入时用于包含或排除特定更改的选项。如果已排除所需的更改，请选择**全部包含**链接。

6.  您现在已提交存储库的本地副本中所做的更改。接下来，与该服务器同步这些更改。选择**同步**链接。

## <a name="step3"> </a><span class="short-header">将项目连接到 Azure</span>步骤 3：将项目连接到 Azure

1.  现在，已在 Visual Studio Online 的 Git 存储库中包含某些源代码，可以将您的 git 存储库连接到 Azure。在 [Azure 门户][Azure 门户]中，选择云服务或网站，或通过选择左下角的 + 图标并依次选择**云服务**或**网站**和**快速创建**来创建新的云服务或网站。<br.>
    ![][8]

2.  对于云服务，选择**使用 Visual Studio Online 设置发布**链接。对于网站，选择**从源代码管理设置部署**链接。
    ![][9]

3.  在向导中，在文本框中键入 Visual Studio Online 帐户的名称，然后选择**“立即授权”**链接。系统可能会要求你登录。
    ![][10]

4.  在 OAuth 弹出对话框中，选择**“接受”**以授予 Azure 在 Visual Studio Online 中配置团队项目的权限。
    ![][11]

5.  授权成功后，将显示一个包含一系列 Visual Studio Online 团队项目的下拉列表。选择在之前的步骤中创建的团队项目的名称，然后选择向导的复选标记按钮。
    ![][12]

下次您将提交推送到存储库时，Visual Studio Online 将生成并将您的项目部署到 Azure。

## <a name="step4"> </a><span class="short-header">触发重新生成</span>步骤 4：触发重新生成和重新部署项目

1.  在 Visual Studio 中，打开文件并进行更改。例如，更改 MVC Web 角色中的 Views\\Shared 文件夹下的文件 \_Layout.cshtml。
    ![][13]

2.  编辑该站点的页脚文本并保存该文件。
    ![][14]

3.  在解决方案资源管理器中，打开解决方案节点、项目节点或您已更改文件的快捷菜单，并选择**提交**。

4.  键入一条注释，然后选择**提交**。
    ![][7]

5.  选择**同步**链接。
    ![][15]

6.  选择**推送**链接，将您的提交推送到 Visual Studio Online 中的存储库。（您还可以使用**同步**按钮将您的提交复制到存储库。差异在于**同步**还从存储库中提取最新的更改。）
    ![][16]

7.  选择“主页”按钮可返回团队资源管理器主页。
    ![][17]

8.  选择**“生成”**链接可查看正在进行的生成。
    ![][18]
    团队资源管理器将显示已为你的签入触发生成。
    ![][19]

9.  要查看进行生成时的详细日志，请双击正在进行的生成的名称。

10. 虽然生成正在进行中，但可查看使用向导链接到 Azure 时创建的生成定义。打开生成定义的快捷菜单，然后选择**编辑生成定义**。
    ![][20]
    在**触发器**选项卡中，您将看到默认设置为在每次签入时的生成定义。（对于云服务，Visual Studio Online 生成并自动将主分支部署到过渡环境。您仍必须执行一个手动步骤，以部署到实时网站。对于不具有过渡环境的网站，它将主分支直接部署到实时站点。
    ![][21]
    在**进程**选项卡中，可以看到部署环境设置为您的云服务或网站的名称。
    ![][22]
    如果您希望并非默认的不同值，请指定属性的值。Azure 发布的属性在“部署”部分中，并且您可能还需要设置 MSBuild 参数。例如，在云服务项目中，指定"云"以外的服务配置，将 MSbuild 参数设置为 /p:TargetProfile =*YourProfile*，其中 *YourProfile* 将一个服务配置文件匹配如 ServiceConfiguration.*YourProfile*.cscfg. 这样的名称。
    下表显示“部署”部分中的可用属性：

    <table>

    <tr>
    <td>
    **属性**

    </td>
    <td>
    **默认值**

    </td>
    </tr>
    </p>
    > <tr>
    > <td>
    > 允许不受信任的证书
    >
    > </td>
    > <td>
    > 如果为 false，则 SSL 证书必须由根颁发机构签名。
    >
    > </td>
    > </tr>
    > <tr>
    > <td>
    > 允许升级
    >
    > </td>
    > <td>
    > 允许部署更新现有的部署，而不允许创建新部署。保留 IP 地址。
    >
    > </td>
    > </tr>
    > <tr>
    > <td>
    > 不要删除
    >
    > </td>
    > <td>
    > 如果为 true，则不会覆盖现有的无关部署（允许升级）。
    >
    > </td>
    > </tr>
    > <tr>
    > <td>
    > 部署设置的路径
    >
    > </td>
    > <td>
    > 网站的 .pubxml 文件的路径，相对于存储库的根文件夹。对于云服务将忽略该属性。
    >
    > </td>
    > </tr>
    > <tr>
    > <td>
    > 云服务名称
    >
    > </td>
    > <td>
    > 连接到的服务的名称
    >
    > </td>
    > </tr>
    > <tr>
    > <td>
    > Sharepoint 部署环境
    >
    > </td>
    > <td>
    > 与服务名称相同
    >
    > </td>
    > </tr>
    > <tr>
    > <td>
    > Windows Azure 部署环境
    >
    > </td>
    > <td>
    > 网站或云服务的名称
    >
    > </td>
    > </tr>
    > </table>
    > 对于云服务，如果[存储帐户][存储帐户]属性保留为空，Azure 将搜索该属性。如果存在与
    > 云服务同名的存储帐户，则将使用该帐户。否则，Azure 将使用其他存储帐户；
    > 或者如果没有存储帐户，Azure 将创建一个存储帐户。即使您的服务不使用存储，云服务也需要存储帐户来存储诊断数据。存储帐户在 Azure 中为文件和其他数据提供了一个位置。

11. 此时，你的生成应已成功完成。
    ![][23]

12. 如果双击生成名称，则 Visual Studio 将显示“生成摘要”，其中包括来自关联的单元测试项目的任何测试结果。
    ![][24]

13. 在 [Azure 门户][Azure 门户]中，可以在选定过渡环境后在“部署”选项卡上查看关联的部署。
    ![][25]

14. 浏览到你的站点的 URL。对于网站，只需选择门户中的**浏览**按钮。对于云服务，选择**“仪表板”**页面的**“速览”**部分中的 URL，其显示过渡环境。默认情况下，来自云服务的持续集成的部署将发布到过渡环境。可以通过将“备用云服务环境”属性设置为“生产”来改变此情况。这里显示了站点 URL 在云服务仪表板页上的位置：
    ![][26]
    新的浏览器选项卡将打开以显示正在运行的网站。
    ![][27]

15. 如果对项目进行其他更改，则将触发更多生成，并且将累积多个部署。标记为“活动”的最新项目。
    ![][28]

## <a name="step5"> </a><span class="short-header">重新部署以前的生成</span>步骤 5：重新部署以前的生成

此步骤是可选的。在管理门户中，选择以前的部署并单击**“重新部署”**可使网站退回以前的签入。请注意，这将在 TFS 中触发新的生成，并在部署历史记录中创建新的条目。
![][29]

## <a name="step6"> </a><span class="short-header">更改生产部署</span>步骤 6：更改生产部署

准备就绪后，可以在管理门户中选择**交换**将过渡环境提升为生产环境。新部署的过渡环境将提升为生产环境，而之前的生产环境（如果有）将变成过渡环境。虽然生产环境和过渡环境的活动部署可能会不同，但最新生成的部署历史记录都是相同的，不管是哪一种环境。
![][30]

## <a name="step7"> </a><span class="short-header">从工作分支部署</span>第 6 步：从工作分支部署。

使用 Git 时，您通常在工作分支中进行更改，当您的开发达到完成状态时，将集成到主分支。在一个项目的开发阶段，您需要生成并将工作分支部署到 Azure。

1.  在团队资源管理器中选择**主页**按钮，然后选择**分支**按钮。
    ![][31]

2.  选择**新分支**链接。
    ![][32]

3.  输入分支的名称，如 "working"，然后选择**创建分支**。这将创建一个新的本地分支。
    ![][33]

4.  发布该分支。选择**取消发布分支**中的分支名称，然后选择**发布**。
    ![][34]

5.  默认情况下，只有更改主分支才会触发一次持续生成。要为工作分支设置持续构建，在团队资源管理器中选择生成页，然后选择**编辑生成定义**。

6.  打开**源代码设置**选项卡。在**监视持续集成和生成的分支**下面，选择**单击此处添加新行**。
    ![][35]

7.  指定您创建的分支，例如 refs/heads/working。
    ![][36]

8.  在代码中进行更改，打开已更改的文件的快捷菜单，然后选择**提交**。
    ![][7]

9.  选择**未同步提交**链接，然后选择**同步**按钮或**推送**链接，将更改复制到 Visual Studio Online 中的工作分支的副本。
    ![][15]

10. 导航到**生成**视图并查找刚刚为工作分支触发的生成。

有关详细信息，请参阅 [Visual Studio Online][Visual Studio Online]。有关使用 Git 和 Visual Studio Online 的其他提示，请参阅[在 Git 中共享您的代码][在 Git 中共享您的代码]，有关使用不受 Visual Studio Online 管理可发布到 Azure 的 Git 存储库的信息，请参阅[从到 Azure 网站的源代码控制发布][从到 Azure 网站的源代码控制发布]。

  [www.visualstudio.com]: http://www.visualstudio.com
  [此处]: http://go.microsoft.com/fwlink/?LinkId=239540
  [步骤 1：注册 Visual Studio online 并创建一个 Git 存储库。]: #step1
  [步骤 2：创建一个项目并将其推送到 Git 存储库。]: #step2
  [步骤 3：将项目连接到 Azure。]: #step3
  [步骤 4：做出更改并触发重新生成和重新部署。]: #step4
  [步骤 5：重新部署以前的生成（可选）]: #step5
  [步骤 6：更改生产部署]: #step6
  [步骤 7：从工作分支进行部署]: #step7
  [1]: http://go.microsoft.com/fwlink/?LinkId=397665
  [0]: ./media/cloud-services-continuous-delivery-use-vso-git/CreateANewAccount.PNG
  [处理团队项目组件，选择过程模板]: http://go.microsoft.com/fwlink/?LinkId=324035
  [2]: ./media/cloud-services-continuous-delivery-use-vso-git/CreateTeamProjectInGit.PNG
  [3]: ./media/cloud-services-continuous-delivery-use-vso/tfs2.png
  [4]: ./media/cloud-services-continuous-delivery-use-vso-git/CloneThisRepository.PNG
  [5]: ./media/cloud-services-continuous-delivery-use-vso-git/CreateNewSolutionInClonedRepo.PNG
  [Azure 和 ASP.NET 入门]: http://www.windowsazure.cn/zh-cn/documentation/articles/web-sites-dotnet-get-started/
  [6]: ./media/cloud-services-continuous-delivery-use-vso-git/CommitMenuItem.PNG
  [7]: ./media/cloud-services-continuous-delivery-use-vso-git/CommitAChange2.PNG
  [Azure 门户]: http://manage.windowsazure.cn
  [8]: ./media/cloud-services-continuous-delivery-use-vso-git/CreateCloudService.PNG
  [9]: ./media/cloud-services-continuous-delivery-use-vso-git/SetUpPublishingWithVSO.PNG
  [10]: ./media/cloud-services-continuous-delivery-use-vso-git/AuthorizeConnection.PNG
  [11]: ./media/cloud-services-continuous-delivery-use-vso-git/ConnectionRequest.PNG
  [12]: ./media/cloud-services-continuous-delivery-use-vso-git/ChooseARepo3.PNG
  [13]: ./media/cloud-services-continuous-delivery-use-vso/tfs17.png
  [14]: ./media/cloud-services-continuous-delivery-use-vso-git/MakeACodeChange.PNG
  [15]: ./media/cloud-services-continuous-delivery-use-vso-git/SyncChanges2.PNG
  [16]: ./media/cloud-services-continuous-delivery-use-vso-git/PushCurrentBranch.PNG
  [17]: ./media/cloud-services-continuous-delivery-use-vso-git/TeamExplorerHome.png
  [18]: ./media/cloud-services-continuous-delivery-use-vso-git/TeamExplorerBuilds.PNG
  [19]: ./media/cloud-services-continuous-delivery-use-vso-git/BuildInQueue.png
  [20]: ./media/cloud-services-continuous-delivery-use-vso/tfs25.png
  [21]: ./media/cloud-services-continuous-delivery-use-vso/tfs26.png
  [22]: ./media/cloud-services-continuous-delivery-use-vso-git/ProcessTab.PNG
  [存储帐户]: http://www.windowsazure.cn/zh-cn/documentation/articles/storage-whatis-account
  [23]: ./media/cloud-services-continuous-delivery-use-vso/tfs28.png
  [24]: ./media/cloud-services-continuous-delivery-use-vso/tfs29.png
  [25]: ./media/cloud-services-continuous-delivery-use-vso/tfs30.png
  [26]: ./media/cloud-services-continuous-delivery-use-vso/tfs31.png
  [27]: ./media/cloud-services-continuous-delivery-use-vso/tfs32.png
  [28]: ./media/cloud-services-continuous-delivery-use-vso/tfs33.png
  [29]: ./media/cloud-services-continuous-delivery-use-vso/tfs34.png
  [30]: ./media/cloud-services-continuous-delivery-use-vso/tfs35.png
  [31]: ./media/cloud-services-continuous-delivery-use-vso-git/BranchesInTeamExplorer.PNG
  [32]: ./media/cloud-services-continuous-delivery-use-vso-git/NewBranch.PNG
  [33]: ./media/cloud-services-continuous-delivery-use-vso-git/CreateBranch.PNG
  [34]: ./media/cloud-services-continuous-delivery-use-vso-git/PublishBranch.PNG
  [35]: ./media/cloud-services-continuous-delivery-use-vso-git/SourceSettingsPage.PNG
  [36]: ./media/cloud-services-continuous-delivery-use-vso-git/IncludeWorkingBranch.PNG
  [Visual Studio Online]: http://go.microsoft.com/fwlink/?LinkId=253861
  [在 Git 中共享您的代码]: http://www.visualstudio.com/get-started/share-your-code-in-git-vs.aspx
  [从到 Azure 网站的源代码控制发布]: http://www.windowsazure.cn/zh-cn/documentation/articles/web-sites-publish-source-control
