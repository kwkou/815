# 从源控件发布到 Azure 网站

Azure 网站支持从源代码控件和存储库工具（例如，BitBucket、CodePlex、Dropbox、Git、GitHub、Mercurial 和 TFS）进行连续部署。可以使用这些工具维护您网站的内容和代码，然后在需要时快速轻松地将更改推送到您的站点。

在本文中，您将了解如何使用 Git 从本地计算机直接发布到 Azure 网站（在 Azure 中，此发布方法称为**"本地 Git"**）。您还将了解如何启用从存储库网站（例如，BitBucket、CodePlex、DropBox、GitHub 或 Mercurial）进行的连续部署。有关使用 TFS 进行连续部署的信息，请参阅[使用 Visual Studio Online 向 Azure 连续传送项目]。

> [chinacloudsites.cn] 在使用<a href="/zh-cn/develop/nodejs/how-to-guides/command-line-tools/">针对 Mac 和 Linux 的 Azure 命令行工具</a>创建网站时，将自动执行本文中所述的许多 Git 命令。

此任务包括下列步骤：

* [安装 Git](#Step1)
* [创建本地存储库](#Step2)
* [添加网页](#Step3)
* [启用网站存储库](#Step4)
* [部署项目](#Step5)
	* [将本地文件推送到 Azure（本地 Git）](#Step6)
	* [从存储库网站（例如，BitBucket、CodePlex、Dropbox、GitHub 或 Mercurial）部署文件](#Step7)
* [故障排除](#Step8)

<h2><a id="Step2"></a>安装 Git</h2>

安装 Git 需要执行的步骤根据操作系统的不同而异。有关操作系统特定的分发和安装指南，请参阅[安装 Git]。

> [WACOM.NOTE] 在某些操作系统上，命令行和 GUI 版本的 Git 都可用。本文中提供的说明使用命令行版本。

<h2><a id="Step2"></a>创建本地存储库</h2>

执行下列任务可创建新的 Git 存储库。

1. 创建名为 MyGitRepository 的目录，用于包含 Git 存储库和网站文件。

2. 打开一个命令行，例如 **GitBash** (Windows) 或 **Bash** (Unix Shell)。在 OS X 系统上，可以通过 **Terminal** 应用程序访问命令行。

3. 在命令行中，更改为 MyGitRepository 目录。

		cd MyGitRepository

4. 使用以下命令可初始化新的 Git 存储库：

		git init

	这将返回一条消息，例如"已在 [路径]* 中初始化空 Git 存储库"***。

<h2><a id="Step3"></a>添加网页</h2>

Azure 网站支持用各种编程语言创建的应用程序。对于此示例，您将使用静态 .html 文件。有关将用其他编程语言创建的网站发布到 Azure 的信息，请参阅 [Azure 开发人员中心]。

1. 通过使用文本编辑器，在 Git 存储库的根目录下（您先前创建的 MyGitRepository 目录）创建一个名为 **index.html** 的新文件。

2. 添加以下文本作为 index.html 文件的内容并保存该文件。

		Hello Git!

3. 在命令行中，验证当前位置是否在 Git 存储库的根目录下。然后使用以下命令将 **index.html** 文件添加到存储库：

		git add index.html 

	> [WACOM.NOTE] 您可以通过在命令后键入 -help 或 --help，查找有关任何 git 命令的帮助。例如，若要了解 add 命令的参数选项，请键入"git add -help"以获取命令行帮助，或键入"git add --help"以获取更加详细的帮助。

4. 接下来，使用以下命令将更改提交到存储库：

		git commit -m "Adding index.html to the repository"

	您应该会看到与下面类似的输出：

		[master (root-commit) 369a79c] Adding index.html to the repository
		 1 file changed, 1 insertion(+)
		 create mode 100644 index.html

<h2><a id="Step4"></a>启用网站存储库</h2>

执行以下步骤可使用 Azure 门户为您的网站启用 Git 存储库：

1. 登录到 [Azure 门户]。

2. 在页面的左侧，选择"网站"，然后选择要为其启用存储库的网站。

	![An image displaying a selected web site][portal-select-website]

3. 选择"仪表板"选项卡。

4. 在"速览"部分中，选择"从源控件设置部署"。此时将显示以下"设置部署"对话框。

	![git-WhereIsYourSourceCode][git-WhereIsYourSourceCode]

4. 选择"本地 Git"，然后单击"下一页"箭头。

	
5. 一小段延迟后，将显示一条指明存储库已就绪的消息。 

	![git-instructions][git-instructions]

<h2><a id="Step5"></a>部署项目</h2>

<h3><a id="Step6"></a>将本地文件推送到 Azure（本地 Git）</h3>

此时，门户将显示有关初始化本地存储库和添加文件的说明。您已经在本主题的前几个步骤中执行了此操作。但是，如果您尚未设置部署凭据，则必须按照说明中的步骤 3 操作。您将进入标记为"重置部署凭据"的链接。

按照以下步骤，使用本地 Git 将您的网站发布到 Azure：

1. 通过使用命令行，确认当前位置是包含以前创建的 index.html 文件的本地 Git 存储库的根目录。

2. 复制 git remote add 命令，该命令已在门户所返回说明的步骤 3 中列出。它将类似于以下命令：

		git remote add azure https://username@needsmoregit.scm.chinacloudsites.cn:443/NeedsMoreGit.git

    > [WACOM.NOTE] **remote** 命令可将命名引用添加到远程存储库。在本例中，它为 Azure 网站存储库创建名为  'azure' 的引用。

1. 从命令行中使用以下命令可将当前存储库内容从本地存储库推送到  'azure' 远程网站：

		git push azure master

	当您在门户中重置部署凭据时，系统将提示您输入以前创建的密码。输入该密码（请注意，在键入密码时，Gitbash 不会将星号回显到控制台）。您应该会看到与下面类似的输出：

		Counting objects: 6, done.
		Compressing objects: 100% (2/2), done.
		Writing objects: 100% (6/6), 486 bytes, done.
		Total 6 (delta 0), reused 0 (delta 0)
		remote: New deployment received.
		remote: Updating branch 'master'.
		remote: Preparing deployment for commit id '369a79c929'.
		remote: Preparing files for deployment.
		remote: Deployment successful.
		To https://username@needsmoregit.scm.chinacloudsites.cn:443/NeedsMoreGit.git
		* [new branch]		master -> master

	> [WACOM.NOTE] 为 Azure 网站创建的存储库应推送请求以便面向其存储库的 <strong>master</strong> 分支，后者将用作该网站的内容。

2. 在门户中，单击门户底部的"浏览"链接以验证是否已部署 **index.html**。这将显示一个包含"Hello Git!"的页面。

	![A webpage containing 'Hello Git!'][hello-git]

3. 通过使用文本编辑器，更改 **index.html** 文件以使其包含"Yay!"，然后保存该文件。

4. 从命令行使用以下命令可**添加**和**提交**更改，然后将更改**推送**到远程存储库：

		git add index.html
		git commit -m "Celebration"
		git push azure master

	完成 **push** 命令后，请刷新浏览器（您可能必须按 Ctrl+F5 才能正确刷新浏览器），您会发现该页面的内容此时将反映最新提交的更改。

	![A webpage containing 'Yay!'][yay]

<h3><a id="Step7"></a>从存储库网站（例如，BitBucket、CodePlex、Dropbox、GitHub 或 Mercurial）部署文件</h3>

通过使用"本地 Git"将本地文件推送到 Azure，可以手动将更新从本地项目推送到您的 Azure 网站，而从 BitBucket、CodePlex、Dropbox、GitHub 或 Mercurial 进行部署会生成一个连续部署过程，在此过程中，Azure 会从项目中拉入最新的更新。

这两种方法都会将项目部署到 Azure 网站，如果您有多个人员在处理同一个项目并希望确保始终发布最新版本（不管是谁执行了最新更新），则连续部署会很有用。此外，如果您将上述工具之一用作应用程序的中央存储库，则连续部署也很有用。

从 GitHub、CodePlex 或 BitBucket 部署文件需要您已将本地项目发布到这些服务之一。有关将项目发布到这些服务的详细信息，请参阅[创建存储库 (GitHub)]、[使用 Git 和 CodePlex]、[创建存储库 (BitBucket)]、[使用 Dropbox 共享 Git 存储库]或[快速入门- Mercurial]。

1. 首先，将您的网站文件放到将用于连续部署的选定存储库中。

2. 在网站的 Azure 门户中，转到"仪表板"选项卡。在"速览"部分中，选择"从源控件设置部署"。这将显示"设置部署"对话框，该对话框会询问"您的源代码在哪里?"。 

2. 选择要用于连续部署的源控件方法。
	
3. 在系统提示时，为选定服务输入凭据。

4. 在授权 Azure 访问您的帐户后，系统将提示您提供存储库列表。 

	![git-ChooseARepositoryToDeploy][git-ChooseARepositoryToDeploy]
  
5. 选择要与 Azure 网站关联的存储库。单击复选标记以继续。

	> [WACOM.NOTE] 在使用 GitHub 或 BitBucket 启用连续部署时，将显示公用项目和专用项目。

6. Azure 创建与所选存储库的关联，并从 master 分支拉入文件。在此过程完成后，"部署"页面上的"部署历史记录"将显示与下面类似的"活动部署"消息：

	![git-githubdeployed][git-githubdeployed]

7. 此时，已将您的项目从所选存储库部署到 Azure 网站。若要验证该站点是否处于活动状态，请单击门户底部的"浏览"链接。浏览器应导航到该网站。

8. 若要验证连续部署是否正在进行，请更改您的项目，然后将所做的更新推送到已与此网站关联的存储库。推送到存储库完成后，您的网站应很快更新以反映更改。可以在网站的"部署"页面上验证是否已拉入更新。

	![git-GitHubDeployed-Updated][git-GitHubDeployed-Updated]


<h4>连续部署的工作方式</h4>
连续部署通过提供在网站的"配置"选项卡的"部署"部分中找到的"部署触发器 URL"来工作。

![git-DeploymentTrigger][git-DeploymentTrigger]

在对存储库进行更新时，会将 POST 请求发送到此 URL，这将告知您的 Azure 网站已更新存储库。此时，将检索更新并将其部署到您的网站。

<h4>指定要使用的分支</h4>

启用连续部署时，将默认为存储库的 **master** 分支。若要使用其他分支，请执行下列步骤：

1. 在门户中，选择您的网站，然后选择"配置"。

2. 在此页面的"部署"部分中，在"要部署的分支"字段中输入要使用的分支，然后按 Enter。最后，单击"保存"。

	Azure 应立即开始基于对新分支所做的更改进行更新。

<h4>禁用连续部署</h4>

可从 Azure"仪表板"禁用连续部署。在"速览"部分下，选择用于断开与所使用存储库的连接的选项：

![git-DisconnectFromGitHub][git-DisconnectFromGitHub]	

在显示确认消息时回答"是"后，若要从其他源设置发布，您可以返回到"速览"并单击"从源控件设置部署"。

<h2><a id="Step8"></a>故障排除</h2>

以下是使用 Git 发布到 Azure 网站时遇到的常见错误或问题：

****

**症状**：无法访问"[siteURL]"：无法连接到 [scmAddress]

**原因**：如果网站无法正常工作，则可能发生此错误。

**解决方法**：在 Azure 门户中启动网站。在网站运行之前，Git 部署无法进行。 


****

**症状**：无法解析主机 'hostname'

**原因**：如果创建  'azure' 远程网站时输入的地址信息不正确，则会发生该错误。

**解决方法**：使用  `git remote -v` 命令列出所有远程网站以及关联的 URL。确认  'azure' 远程网站的 URL 正确。如果需要，请删除此远程网站并使用正确的 URL 重新创建它。

****

**症状**：无通用引用且未指定任何引用；不采取任何措施。或许您应指定一个分支，例如  'master'。

**原因**：如果您在执行 Git 推送操作时未指定分支且未设置 Git 使用的 push.default 值，则会发生该错误。

**解决方法**：请再次执行推送操作，并指定 master 分支。例如：

	git push azure master

****

**症状**：src refspec [分支名] 不匹配任何内容。

**原因**：如果您尝试推送到  'azure' 远程网站上 master 分支之外的分支，则会发生该错误。

**解决方法**：请再次执行推送操作，并指定 master 分支。例如：

	git push azure master

****

**症状**：错误 - 已将更改提交到远程存储库，但未更新您的网站。

**原因**：如果您部署的是 Node.js 应用程序，其中包含用于指定其他必需模块的 package.json 文件，则会发生该错误。

**解决方法**：应在发生此错误前记录包含"npm ERR!"的其他消息，并可提供有关失败的其他上下文。以下是该错误的已知原因和相应的"npm ERR!"消息：

* **package.json 文件不正确**：npm ERR!无法读取依赖项。

* **不具有 Windows 的二进制分发的本机模块**：

	* npm ERR! \`cmd "/c" "node-gyp rebuild"\` failed with 1

		或者

	* npm ERR! [modulename@version] preinstall: \`make || gmake\`


## 其他资源

* [如何使用适用于 Azure 的 PowerShell]
* [如何使用针对 Mac 和 Linux 的 Azure 命令行工具]
* [Git 文档]

[Azure 开发人员中心]: http://www.windowsazure.cn/zh-cn/documentation/
[Azure 门户]: http://manage.windowsazure.cn
[Git 网站]: http://git-scm.com
[安装 Git]: http://git-scm.com/book/en/Getting-Started-Installing-Git
[如何使用适用于 Azure 的 PowerShell]: http://www.windowsazure.cn/zh-cn/develop/nodejs/how-to-guides/powershell-cmdlets/
[如何使用针对 Mac 和 Linux 的 Azure 命令行工具]: /zh-cn/develop/nodejs/how-to-guides/command-line-tools/
[Git 文档]: http://git-scm.com/documentation

[portal-select-website]: ./media/publishing-with-git/git-select-website.png
[git-WhereIsYourSourceCode]: ./media/publishing-with-git/git-WhereIsYourSourceCode.png
[git-instructions]: ./media/publishing-with-git/git-instructions.png
[portal-deployment-credentials]: ./media/publishing-with-git/git-deployment-credentials.png

[git-ChooseARepositoryToDeploy]: ./media/publishing-with-git/git-ChooseARepositoryToDeploy.png
[hello-git]: ./media/publishing-with-git/git-hello-git.png
[yay]: ./media/publishing-with-git/git-yay.png
[git-githubdeployed]: ./media/publishing-with-git/git-GitHubDeployed.png
[git-GitHubDeployed-Updated]: ./media/publishing-with-git/git-GitHubDeployed-Updated.png
[git-DisconnectFromGitHub]: ./media/publishing-with-git/git-DisconnectFromGitHub.png
[git-DeploymentTrigger]: ./media/publishing-with-git/git-DeploymentTrigger.png
[创建存储库 (GitHub)]: https://help.github.com/articles/create-a-repo
[使用 Git 和 CodePlex]: http://codeplex.codeplex.com/wikipage?title=Using%20Git%20with%20CodePlex&referringTitle=Source%20control%20clients&ProjectName=codeplex
[创建存储库 (BitBucket)]: https://confluence.atlassian.com/display/BITBUCKET/Create+an+Account+and+a+Git+Repo
[快速入门- Mercurial]: http://mercurial.selenic.com/wiki/QuickStart
[使用 Dropbox 共享 Git 存储库]: https://gist.github.com/trey/2722927
[使用 Visual Studio Online 向 Azure 连续传送项目]: http://azure.microsoft.com/zh-cn/documentation/articles/cloud-services-continuous-delivery-use-vso/
<!--HONumber=41-->
