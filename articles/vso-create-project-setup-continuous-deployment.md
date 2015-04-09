<properties linkid="create-vso-project-setup-continuous-deployment" urlDisplayName="How to create a VSO project and setup Continuous Deployment" pageTitle="How to create a Visual Studio Online team project and setup Continuous Deployment - Windows Azure" metaKeywords="Visual Studio Online create team project, continuous deployment to Azure" description="Learn how to create a Visual Studio Online team project and configure it for continuous deployment to Windows Azure." metaCanonical="" services="cloud-services, visual-studio-online" documentationCenter="" title="How to Create and Deploy a Cloud Service" authors="jimlamb" solutions="" writer="jimlamb" manager="" editor=""  />

# 创建 Visual Studio Online 项目并设置对 Windows Azure 的连续部署

[WACOM.INCLUDE [免责声明][免责声明]]

利用 Windows Azure 管理门户，你可以在 Visual Studio Online 上创建团队项目，并将你的 Web 应用程序配置为可连续部署到网站。

## 目录

-   [如何创建团队项目][如何创建团队项目]
-   [如何新建 Web 应用程序并将其添加至 Git 版本控制][如何新建 Web 应用程序并将其添加至 Git 版本控制]
-   [如何设置连续部署][如何设置连续部署]

## <a name="create_team_project"></a>如何创建团队项目

1.  登录到管理门户。
2.  单击左下角的“新建”。
3.  单击“团队项目”。
4.  为团队项目提供一个名称。请注意，团队项目一旦创建，就不能再更改其名称。
5.  选择要用于项目的版本控制的类型。你可以选择 Git（分布式版本控制系统）或 Team Foundation 版本控制（集中式版本控制系统）。不确定要用哪种系统？在[此处][此处]了解更多信息。
6.  选择过程模板。有关过程模板的比较，请参阅[处理团队项目组件][处理团队项目组件]。
7.  选择要使用的 Visual Studio Online 帐户，以便创建此团队项目、添加用户并监视资源使用情况。
8.  选中“添加到开始板”框，这样你的新团队项目将自动出现在开始板上。
9.  单击“创建”。

## <a name="create_web_app"></a>如何新建 Web 应用程序并将其添加至 Git 版本控制

1.  从开始板上单击你的新团队项目。
2.  在“代码”小视窗的“存储库”部分中，单击根据团队项目命名的 Git 存储库。
3.  在存储库/分支叶片中，单击 **Visual Studio** 叶片级命令，以在 Visual Studio 中打开你的新存储库。你的 Web 浏览器可能会提示你授权启动 Visual Studio。
4.  在 Visual Studio 的团队资源管理器工具窗口中，单击“克隆此存储库”以在本地磁盘上设置新存储库的本地克隆。
5.  在团队资源管理器主页的“解决方案”部分，单击“新建...”以在刚克隆的存储库中创建新项目。
6.  在“新建项目”对话框中，展开 Visual Basic 或 Visual C# 节点（具体取决于你的编程语言喜好），然后选择 **Web**。
7.  单击可用项目模板列表中的“ASP.NET Web 应用程序”并为你的 Web 应用程序输入名称。
8.  单击“确定”。
9.  切换到团队资源管理器工具窗口，导航到“更改”页面，输入提交消息。
10. 单击“提交”按钮上的下拉箭头并单击“提交并同步”，以提交更改并将该提交的更改推送到你之前克隆的远程存储库。

## <a name="continuous_deployment"></a>如何设置连续部署

1.  登录到管理门户。
2.  从开始板中单击你之前创建的团队项目。
3.  在“生成”小视窗中，单击**“设置连续部署”**部分。
4.  选择要将你的 Web 应用程序部署到其中的网站。
5.  选择你的源代码所在的存储库（目前，你在团队项目中应该只有一个存储库）。
6.  选择要生成的分支。Visual Studio Online 将在此分支上最近提交的内容中扫描单独的一个 Visual Studio 解决方案文件 (\*.sln)。如果找到，它将配置新生成定义（名称以“\_CD”结尾）以生成该解决方案。如果找不到解决方案文件，或者，如果找到了多个文件，它仍将配置新生成定义，但它会被禁用，你需要在 Visual Studio 中编辑它以指定要生成的解决方案。
7.  单击**“创建”**。
8.  Visual Studio Online 将创建新生成定义以生成和部署 Web 应用程序项目并尝试让该定义的新生成排队。

### 验证你的部署是否已成功完成

1.  从开始板中单击你之前创建的团队项目。
2.  在“生成”小视窗中，单击“最新生成”部分，以打开生成叶片。
3.  在该生成叶片中，单击“部署”部分中的第一项，以打开关联的网站。
4.  在该网站叶片上，单击“浏览”叶片级命令以浏览该网站并验证你的 Web 应用程序的部署。

  [免责声明]: ../includes/disclaimer.md
  [如何创建团队项目]: #create_team_project
  [如何新建 Web 应用程序并将其添加至 Git 版本控制]: #create_web_app
  [如何设置连续部署]: #continuous_deployment
  [此处]: http://msdn.microsoft.com/zh-cn/library/ms181368.aspx
  [处理团队项目组件]: http://msdn.microsoft.com/zh-cn/library/ms400752.aspx
