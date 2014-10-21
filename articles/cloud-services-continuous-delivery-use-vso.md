<properties linkid="dev-net-common-tasks-publishing-with-vso" urlDisplayName="Publishing with TFS" pageTitle="Continuous delivery with Visual Studio Online in Azure" metaKeywords="" description="Learn how to configure your Visual Studio Online team projects to automatically build and deploy to Azure web sites or cloud services." metaCanonical="" services="web-sites" documentationCenter=".NET" title="Continuous delivery to Azure using Visual Studio Online" authors="ghogen" solutions="" manager="" editor="" />

# 使用 Visual Studio Online 向 Azure 持续传送项目

Visual Studio Online（前称 Team Foundation Service）是 Microsoft 提供的常用 Team Foundation Server (TFS) 软件的云托管服务版本，该软件提供了可高度自定义的源代码和生成管理、敏捷开发和团队过程工作流、问题和工作项跟踪等功能。你可以将 Visual Studio Online 团队项目配置为自动生成并部署到 Azure 网站或云服务。有关如何使用本地 Team Foundation Server 设置持续生成和部署系统的信息，请参阅[在 Azure 中持续传送云服务][在 Azure 中持续传送云服务]。

本教程假设你已安装 Visual Studio 2013 和 Azure SDK。如果你尚未安装 Visual Studio 2013，请在 [www.visualstudio.com][www.visualstudio.com] 上选择“免费试用”链接以下载该软件。从[此处][此处]安装 Azure SDK。

若要使用 Visual Studio Online 将云服务设置为自动生成并部署到 Azure，请执行下列步骤：

-   [步骤 1：注册 Visual Studio Online。][步骤 1：注册 Visual Studio Online。]

-   [步骤 2：将项目签入到源代码管理。][步骤 2：将项目签入到源代码管理。]

-   [步骤 3：将项目连接到 Azure。][步骤 3：将项目连接到 Azure。]

-   [步骤 4：做出更改并触发重新生成和重新部署。][步骤 4：做出更改并触发重新生成和重新部署。]

-   [步骤 5：重新部署以前的生成（可选）][步骤 5：重新部署以前的生成（可选）]

-   [步骤 6：更改生产部署（仅限云服务）][步骤 6：更改生产部署（仅限云服务）]

## <a name="step1"></a><span class="short-header">注册 Visual Studio Online</span>步骤 1：注册 Visual Studio Online

1.  导航到 [][www.visualstudio.com]<http://www.visualstudio.com></a> 并创建一个 Visual Studio Online 帐户。单击“登录”链接。
    你将需要使用 Microsoft 帐户进行登录。如果这是你首次登录，系统会要求你提供有关你本人的一些信息，例如你的姓名和电子邮件地址。
    ![][]

2.  如果不是首次登录，在登录时你会看到此屏幕。请单击“立即创建免费帐户”链接。

    ![][1]

3.  为新项目创建帐户 URL。你的帐户将采用以下格式：https://\<accountname\>.visualstudio.com。

    ![][2]

4.  现在，你可以创建自己的第一个项目。输入项目名称和说明。选择要使用的版本控制系统。支持 Team Foundation 版本控制 (TFVC) 和 Git。可以在[使用版本控制][使用版本控制]中找到有关这些选项的详细信息。本演练假设你使用的是 TFVC。如果使用的是 Git，请参阅[使用 Visual Studio Online 和 Git 向 Azure 持续传送项目][使用 Visual Studio Online 和 Git 向 Azure 持续传送项目]。然后，请选择你的组织使用的过程模板，并选择“创建项目”按钮。有关过程模板的详细信息，请参阅[处理团队项目组件，选择过程模板][处理团队项目组件，选择过程模板]。

    ![][3]

5.  完成项目创建后，单击“使用 Visual Studio 打开以便连接”按钮以自动启动连接到团队项目的 Visual Studio。如果显示了任何安全对话框，请选择“允许”。

    ![][4]

## <a name="step2"> </a><span class="short-header">将项目签入到源代码管理。</span>步骤 2：将项目签入到源代码管理

1.  在 Visual Studio 中，打开你要部署的解决方案或者创建新的解决方案。你可以通过执行本演练中的步骤来部署网站或云服务（Azure 应用程序）。如果你想要创建新解决方案，请创建新的 Azure 云服务项目或新的 ASP.NET MVC 项目。确保项目以 .NET Framework 4 或 4.5 为目标。如果你要创建云服务项目，请添加 ASP.NET MVC Web 角色和辅助角色，并为 Web 角色选择 Internet 应用程序。出现提示时，请选择“Internet 应用程序”。如果你想要创建网站，请选择 ASP.NET Web 应用程序项目模板，然后选择 MVC。请参阅 [Azure 和 ASP.NET 入门][Azure 和 ASP.NET 入门]。

2.  打开解决方案的上下文菜单，然后选择“将解决方案添加到源代码管理”。

    ![][5]

3.  接受或更改默认值，然后选择“确定”按钮。一旦完成此过程，源代码管理图标就会显示在解决方案资源管理器中。

    ![][6]

4.  打开解决方案的快捷菜单，然后选择“签入”。

    ![][7]

5.  在团队资源管理器的“挂起的更改”区域中，为签入键入注释并选择“签入”按钮。

    ![][8]

记下在签入时用于包含或排除特定更改的选项。如果已排除所需的更改，请选择“全部包含”链接。

![][9]

## <a name="step3"> </a><span class="short-header">将项目连接到 Azure</span>步骤 3：将项目连接到 Azure

1.  此时你已拥有一个包含某些源代码的 VSO 团队项目，可以将该团队项目连接到 Azure。在 [Azure 门户][Azure 门户]中，选择云服务或网站，或通过选择左下角的 + 图标并依次选择“云服务”或“网站”和“快速创建”来创建新的云服务或网站。选择“使用 Visual Studio Online 设置发布”链接。

    ![][10]

2.  在向导中，在文本框中键入 Visual Studio Online 帐户的名称，然后单击“立即授权”链接。系统可能会要求你登录。

    ![][11]

3.  在 OAuth 弹出对话框中，选择“接受”以授予 Azure 在 VSO 中配置团队项目的权限。

    ![][12]

4.  授权成功后，将显示一个包含一系列 Visual Studio Online 团队项目的下拉列表。选择在之前的步骤中创建的团队项目的名称，然后选择向导的复选标记按钮。

    ![][13]

5.  链接你的项目后，你将看到一些有关将更改签入到 Visual Studio Online 团队项目中的说明。下次签入时，Visual Studio Online 将生成项目并将其部署到 Azure。通过单击“通过 Visual Studio 签入”链接并单击“启动 Visual Studio”链接（或位于门户屏幕底部具有等效功能的“Visual Studio”按钮）可立即尝试此操作。

    ![][14]

## <a name="step4"> </a><span class="short-header">触发重新生成</span>步骤 4：触发重新生成和重新部署项目

1.  在 Visual Studio 的团队资源管理器中，单击“源代码管理资源管理器”链接。

    ![][15]

2.  导航到解决方案文件并将其打开。

    ![][16]

3.  在解决方案资源管理器中，打开文件并进行更改。例如，更改 MVC Web 角色中的 Views\\Shared 文件夹下的文件 \_Layout.cshtml。

    ![][17]

4.  编辑网站的徽标，并按 Ctrl+S 来保存文件。

    ![][18]

5.  在团队资源管理器中，选择“挂起的更改”链接。

    ![][19]

6.  键入注释并选择“签入”按钮。

    ![][20]

7.  选择“主页”按钮可返回团队资源管理器主页。

    ![][21]

8.  选择“生成”链接可查看正在进行的生成。

    ![][22]

    团队资源管理器将显示已为你的签入触发生成。

    ![][23]

9.  双击正在进行的生成的名称可在生成进行中查看详细日志。

    ![][24]

10. 虽然生成正在进行中，但可查看使用向导将 TFS 链接到 Azure 时创建的生成定义。打开生成定义的快捷菜单，然后选择“编辑生成定义”。

    ![][25]

    在“触发器”选项卡中，你将看到生成定义已设置为默认情况下在每次签入时生成。

    ![][26]

    在“进程”选项卡中，你将看到部署环境已设置为云服务或网站的名称。如果你正在处理网站，你所看到的属性将不同于此处所示的属性。

    ![][27]

    如果你不想要使用默认值，请指定属性值。Azure 发布的属性显示在“部署”部分中。
    下表显示了“部署”部分中提供的属性：

    <table>

    <tr>
    <td>
    <b>属性</b>

    </td>
    <td>
    <b>默认值</b>

    </td>
    </tr>
    </p>
    > <tr>
    > <td>
     允许不受信任的证书
    
     </td>
    > <td>
     如果为 false，则 SSL 证书必须由根颁发机构签名。
    
     </td>
    > </tr>
    > <tr>
    > <td>
     允许升级
    
     </td>
    > <td>
     允许部署更新现有的部署，而不允许创建新部署。保留 IP 地址。
    
     </td>
    > </tr>
    > <tr>
    > <td>
     不要删除
    
     </td>
    > <td>
     如果为 true，则不会覆盖现有的无关部署（允许升级）。
    
     </td>
    > </tr>
    > <tr>
    > <td>
     部署设置的路径
    
     </td>
    > <td>
     网站的 .pubxml 文件的路径，相对于存储库的根文件夹。对于云服务将忽略该属性。
    
     </td>
    > </tr>
    > <tr>
    > <td>
     云服务名称
    
     </td>
    > <td>
     连接到的服务的名称
    
     </td>
    > </tr>
    > <tr>
    > <td>
     Sharepoint 部署环境
    
     </td>
    > <td>
     与服务名称相同
    
     </td>
    > </tr>
    > <tr>
    > <td>
     Windows Azure 部署环境
    
     </td>
    > <td>
     网站或云服务的名称
    
     </td>
    > </tr>
    > </table>
     如果存储帐户属性保留为空，Azure 将搜索该属性。如果存在与云服务同名的存储帐户，则将使用该帐户。否则，Azure 将使用其他存储帐户；或者如果没有存储帐户，Azure 将创建一个存储帐户。存储帐户在 Azure 中为存储文件和其他数据提供了一个位置。有关详细信息，请参阅[什么是存储帐户？][什么是存储帐户？]。

如果你在使用多个服务配置（.cscfg 文件），可以在“生成”，“高级”，“MSBuild 参数”设置中指定所需的服务配置。例如，若要使用 ServiceConfiguration.Test.cscfg，请设置 MSBuild 参数行选项 /p:TargetProfile=Test。

![][2]

1.  此时，你的生成应已成功完成。

    ![][28]

2.  如果双击生成名称，则 Visual Studio 将显示“生成摘要”，其中包括来自关联的单元测试项目的任何测试结果。

    ![][29]

3.  在 [Azure 门户][Azure 门户]中，可以在选定过渡环境后在“部署”选项卡上查看关联的部署。

    ![][30]

4.  浏览到你的站点的 URL。对于网站，只需单击命令栏上的“浏览”按钮。对于云服务，请在“仪表板”页上选择显示云服务过渡环境的“速览”部分中的 URL。默认情况下，来自云服务的持续集成的部署将发布到过渡环境。可以通过将“备用云服务环境”属性设置为“生产”来改变此情况。以下屏幕截图显示了站点 URL 在云服务仪表板页上的位置：

    ![][31]

    新的浏览器选项卡将打开以显示正在运行的网站。

    ![][32]

5.  对于云服务，如果对项目进行其他更改，则将触发更多生成，并且将累积多个部署。标记为“活动”的最新项目。

    ![][33]

## <a name="step5"> </a><span class="short-header">重新部署以前的生成</span>步骤 5：重新部署以前的生成

此步骤适用于云服务，并且是一个可选步骤。在管理门户中，选择以前的部署并单击“重新部署”按钮可使网站退回以前的签入。请注意，这将在 TFS 中触发新的生成，并在部署历史记录中创建新的条目。

![][34]

## <a name="step6"> </a><span class="short-header">更改生产部署</span>步骤 6：更改生产部署

此步骤仅适用于云服务而非网站。准备就绪后，可以在管理门户中选择“交换”按钮将过渡环境提升为生产环境。新部署的过渡环境将提升为生产环境，而之前的生产环境（如果有）将变成过渡环境。虽然生产环境和过渡环境的活动部署可能会不同，但最新生成的部署历史记录都是相同的，不管是哪一种环境。

![][35]

有关详细信息，请参阅 [Visual Studio Online][Visual Studio Online]。如果使用的是 Git，请参阅[在 Git 中共享代码][在 Git 中共享代码]和[从源代码管理发布到 Azure 网站][从源代码管理发布到 Azure 网站]。

  [在 Azure 中持续传送云服务]: ../cloud-services-dotnet-continuous-delivery
  [www.visualstudio.com]: http://www.visualstudio.com
  [此处]: http://go.microsoft.com/fwlink/?LinkId=239540
  [步骤 1：注册 Visual Studio Online。]: #step1
  [步骤 2：将项目签入到源代码管理。]: #step2
  [步骤 3：将项目连接到 Azure。]: #step3
  [步骤 4：做出更改并触发重新生成和重新部署。]: #step4
  [步骤 5：重新部署以前的生成（可选）]: #step5
  [步骤 6：更改生产部署（仅限云服务）]: #step6
  []: ./media/cloud-services-continuous-delivery-use-vso/tfs0.PNG
  [1]: ./media/cloud-services-continuous-delivery-use-vso/tfs36.PNG
  [https://\<accountname\>.visualstudio.com]: https://<accountname>.visualstudio.com
  [2]: ./media/cloud-services-continuous-delivery-use-vso/tfs37.PNG
  [使用版本控制]: http://go.microsoft.com/fwlink/?LinkId=324037
  [使用 Visual Studio Online 和 Git 向 Azure 持续传送项目]: http://go.microsoft.com/fwlink/?LinkId=397358
  [处理团队项目组件，选择过程模板]: http://go.microsoft.com/fwlink/?LinkId=324035
  [3]: ./media/cloud-services-continuous-delivery-use-vso/tfs1.png
  [4]: ./media/cloud-services-continuous-delivery-use-vso/tfs2.png
  [Azure 和 ASP.NET 入门]: http://www.windowsazure.com/zh-cn/documentation/articles/web-sites-dotnet-get-started/
  [5]: ./media/cloud-services-continuous-delivery-use-vso/tfs5.png
  [6]: ./media/cloud-services-continuous-delivery-use-vso/tfs6.png
  [7]: ./media/cloud-services-continuous-delivery-use-vso/tfs7.png
  [8]: ./media/cloud-services-continuous-delivery-use-vso/tfs8.png
  [9]: ./media/cloud-services-continuous-delivery-use-vso/tfs9.png
  [Azure 门户]: http://manage.windowsazure.cn
  [10]: ./media/cloud-services-continuous-delivery-use-vso/tfs10.png
  [11]: ./media/cloud-services-continuous-delivery-use-vso/tfs11.png
  [12]: ./media/cloud-services-continuous-delivery-use-vso/tfs12.png
  [13]: ./media/cloud-services-continuous-delivery-use-vso/tfs13.png
  [14]: ./media/cloud-services-continuous-delivery-use-vso/tfs14.png
  [15]: ./media/cloud-services-continuous-delivery-use-vso/tfs15.png
  [16]: ./media/cloud-services-continuous-delivery-use-vso/tfs16.png
  [17]: ./media/cloud-services-continuous-delivery-use-vso/tfs17.png
  [18]: ./media/cloud-services-continuous-delivery-use-vso/tfs18.png
  [19]: ./media/cloud-services-continuous-delivery-use-vso/tfs19.png
  [20]: ./media/cloud-services-continuous-delivery-use-vso/tfs20.png
  [21]: ./media/cloud-services-continuous-delivery-use-vso/tfs21.png
  [22]: ./media/cloud-services-continuous-delivery-use-vso/tfs22.png
  [23]: ./media/cloud-services-continuous-delivery-use-vso/tfs23.png
  [24]: ./media/cloud-services-continuous-delivery-use-vso/tfs24.png
  [25]: ./media/cloud-services-continuous-delivery-use-vso/tfs25.png
  [26]: ./media/cloud-services-continuous-delivery-use-vso/tfs26.png
  [27]: ./media/cloud-services-continuous-delivery-use-vso/tfs27.png
  [什么是存储帐户？]: http://www.windowsazure.cn/zh-cn/documentation/articles/storage-whatis-account
  [28]: ./media/cloud-services-continuous-delivery-use-vso/tfs28.png
  [29]: ./media/cloud-services-continuous-delivery-use-vso/tfs29.png
  [30]: ./media/cloud-services-continuous-delivery-use-vso/tfs30.png
  [31]: ./media/cloud-services-continuous-delivery-use-vso/tfs31.png
  [32]: ./media/cloud-services-continuous-delivery-use-vso/tfs32.png
  [33]: ./media/cloud-services-continuous-delivery-use-vso/tfs33.png
  [34]: ./media/cloud-services-continuous-delivery-use-vso/tfs34.png
  [35]: ./media/cloud-services-continuous-delivery-use-vso/tfs35.png
  [Visual Studio Online]: http://go.microsoft.com/fwlink/?LinkId=253861
  [在 Git 中共享代码]: http://www.visualstudio.com/get-started/share-your-code-in-git-vs.aspx
  [从源代码管理发布到 Azure 网站]: http://www.windowsazure.cn/zh-cn/documentation/articles/web-sites-publish-source-control
