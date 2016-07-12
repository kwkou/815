[Azure Web 应用](/documentation/services/web-sites/)支持从源代码控制和存储库工具（例如，Git 和 Mercurial）部署到 Web Apps。你可以使用这些工具维护 Web 应用的内容和代码，然后在需要时快速轻松地将更改推送到你的 Azure Web 应用。

在本文中，你将了解如何使用 Git 从本地计算机直接发布到 Web Apps（在 Azure 中，此发布方法称为**本地 Git**）。你还将了解如何启用从存储库站点（例如，GitHub 或 Mercurial）进行的部署。

> [AZURE.NOTE] 在使用[适用于 Mac 和 Linux 的 Azure 命令行工具](/documentation/articles/xplat-cli-install/)创建 Web 应用时，将自动执行本文中所述的许多 Git 命令。

## <a id="Step1"></a>步骤 1：安装 Git

安装 Git 所需的步骤因操作系统的不同而异。有关操作系统特定的分发和安装指南，请参阅[安装 Git]。

> [AZURE.NOTE] 在某些操作系统上，命令行和 GUI 版本的 Git 都可用。本文中提供的说明使用命令行版本。

## <a id="Step2"></a>步骤 2：创建本地存储库

执行下列任务可创建新的 Git 存储库。

1. 创建名为 MyGitRepository 的目录，用于包含 Git 存储库和 Web 应用文件。

2. 打开一个命令行工具，例如 **GitBash** (Windows) 或 **Bash** (Unix Shell)。在 OS X 系统上，可以通过 **Terminal** 应用程序访问命令行。

3. 在命令行中，更改为 MyGitRepository 目录。

		cd MyGitRepository

4. 使用以下命令可初始化新的 Git 存储库：

		git init

	这将返回一条消息，例如“已在 [路径] 中初始化空 Git 存储库”。

## <a id="Step3"></a>步骤 3：添加网页

Web Apps 支持用各种编程语言创建的应用程序。对于此示例，你将使用静态 .html 文件。

1. 通过使用文本编辑器，在 Git 存储库的根目录下（你先前创建的 MyGitRepository 目录）创建一个名为 **index.html** 的新文件。

2. 添加以下文本作为 index.html 文件的内容并保存该文件。

		Hello Git!

3. 在命令行中，验证当前位置是否在 Git 存储库的根目录下。然后使用以下命令将 **index.html** 文件添加到存储库：

		git add index.html 

	> [AZURE.NOTE] 你可以通过在命令后键入 -help 或 --help，查找有关任何 git 命令的帮助。例如，若要了解 add 命令的参数选项，请键入“git add -help”以获取命令行帮助，或键入“git add --help”以获取更加详细的帮助。

4. 接下来，使用以下命令将更改提交到存储库：

		git commit -m "Adding index.html to the repository"

	您应该会看到与下面类似的输出：

		[master (root-commit) 369a79c] Adding index.html to the repository
		 1 file changed, 1 insertion(+)
		 create mode 100644 index.html

## <a id="Step4"></a>启用 Web 应用存储库

执行以下步骤可为你的 Web 应用启用 Git 存储库。

1. 登录到 [Azure 经典管理门户]。

2. 单击“新建”按钮以创建将为其启用存储库的新 Web 应用。

2. 等待至 Web 应用创建过程在“Web 应用”视图中完成，然后选择该 Web 应用。

	![显示选定 Web 应用的图像][portal-select-website]

3. 选择“仪表板”选项卡。

4. 在“速览”部分中，选择“从源控件设置部署”。此时将显示以下“设置部署”对话框。

	![git-WhereIsYourSourceCode][git-WhereIsYourSourceCode]

4. 选择“本地 Git”，然后单击“下一页”箭头。

4. 如果这是你第一次在 Azure 中设置存储库，则需要为其创建登录凭据。你将使用它们从本地 Git 存储库登录到 Azure 的存储库和推送更改。

	![](./media/publishing-with-git/git_credentials.png)
	
5. 一小段延迟后，将显示一条指明存储库已就绪的消息。

	![git 说明][git-instructions]

## <a id="Step5"></a>部署项目

* [将本地文件推送到 Azure（本地 Git）](#Step6)
* [从存储库 Web 应用（例如，GitHub 或 Mercurial）上部署文件](#Step7)
* [从 GitHub 或 Mercurial 上部署 Visual Studio 解决方案](#Step75)

<a id="Step6"></a>使用以下步骤通过本地 Git 将 Web 应用发布到 Azure 中。


此时，经典管理门户将显示有关初始化本地存储库和添加文件的说明。你已经在本主题的前几个步骤中执行了此操作。但是，如果没有设置你的部署凭据，你必须转回至经典管理门户中的“仪表板”选项卡并单击“重置部署凭据”。

按照以下步骤，使用本地 Git 将你的 Web 应用发布到 Azure：

1. 通过使用命令行，确认当前位置是包含以前创建的 index.html 文件的本地 Git 存储库的根目录。

2. 复制 git remote add 命令，该命令已在经典管理门户所返回说明的步骤 3 中列出。它将类似于以下命令：

		git remote add azure https://username@needsmoregit.scm.chinacloudsites.cn:443/NeedsMoreGit.git

    > [AZURE.NOTE] **remote** 命令可将命名引用添加到远程存储库。在本示例中，它为 Web 应用的存储库创建名为“azure”的引用。

1. 从命令行中使用以下命令可将当前存储库内容从本地存储库推送到“azure”远程 Web 应用：

		git push azure master

	当你在经典管理门户中重置部署凭据时，系统将提示你输入以前创建的密码。输入该密码（请注意，在键入密码时，Gitbash 不会将星号回显到控制台）。您应该会看到与下面类似的输出：

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

	> [AZURE.NOTE] 为 Web 应用创建的存储库应推送请求，以便面向其存储库的 <strong>master</strong> 分支，后者将用作该 Web 应用的内容。

2. 在经典管理门户中，单击经典管理门户底部的“浏览”链接以验证是否已部署 **index.html**。这将显示一个包含“Hello Git!”的页面。

	![包含“Hello Git!”的网页][hello-git]

3. 通过使用文本编辑器，更改 **index.html** 文件以使其包含“Yay!”，然后保存该文件。

4. 从命令行使用以下命令可**添加**和**提交**更改，然后将更改**推送**到远程存储库：

		git add index.html
		git commit -m "Celebration"
		git push azure master

	完成 **push** 命令后，请刷新浏览器（你可能必须按 Ctrl+F5 才能正确刷新浏览器），你会发现该页面的内容此时将反映最新提交的更改。

### <a id="Step7"></a>从存储库站点（例如，GitHub 或 Mercurial）部署文件

通过使用本地 Git 将本地文件推送到 Azure，你可以手动将更新从本地项目推送到 Azure 中的 Web 应用，而在经典管理门户上单击“同步”之后，从 GitHub 或 Mercurial 部署会生成部署过程。

从 GitHub 部署文件要求你已经将本地项目发布到其中一项服务中。有关将项目发布到这些服务的详细信息，请参阅[创建存储库 (GitHub)] 或[快速启动 - Mercurial]。

1. 首先将你的 Web 应用文件放到将用于部署的选定存储库中。

2. 在 Web 应用的 Azure 经典管理门户中，转到“仪表板”选项卡。在“速览”部分中，选择“从源控件设置部署”。这将显示“设置部署”对话框，该对话框会询问“你的源代码在哪里?”。

2. 选择“外部存储库”。
	
3. 出现提示时，请输入存储库 URL 以及你需要部署的分支的名称（默认为 master）。

6. Azure 创建与所选存储库的关联，并从 master 分支拉入文件。在此过程完成后，“部署”页面上的“部署历史记录”将显示与下面类似的“活动部署”消息：

	![git-githubdeployed][git-githubdeployed]

7. 此时，已将你的项目从所选存储库部署到 Azure Web 应用。若要验证该站点是否处于活动状态，请单击经典管理门户底部的“浏览”链接。浏览器应导航到该 Web 应用。

8. 对你的项目进行更改，然后将更新推送到已与此 Web 应用关联的存储库。在经典管理门户上单击“同步”，完成推送到存储库后，你的 Web 应用应很快更新以反映更改。

### 从 GitHub 或 Mercurial 上部署 Visual Studio 解决方案

将 Visual Studio 解决方案推送到 Azure 中的 Web Apps 就像推送简单的 index.html 文件一样容易。Web Apps 部署过程简化了所有细节，包括还原 NuGet 依赖项和构建应用程序二进制文件。可以按照仅在你的 Git 存储库中维护代码的源控制最佳实践操作，并让 Web Apps 部署处理其余工作。

将 Visual Studio 解决方案推送到 Web Apps 的步骤与[上一部分](#Step7)中的步骤相同，前提是按以下方式配置你的解决方案和存储库：

-	在你的存储库根中，添加 `.gitignore` 文件，然后指定想要从你的存储库中排除的所有文件和文件夹，例如，`Obj`、`Bin` 和 `packages` 文件夹（有关格式信息，请参阅 [gitignore 文档](http://git-scm.com/docs/gitignore)）。例如：

		[Oo]bj/
		[Bb]in/
		*.user
		/TestResults
		*.vspscc
		*.vssscc
		*.suo
		*.cache
		*.csproj.user
		packages/*
		App_Data/
		/apps
		msbuild.log
		_app/
		nuget.exe

	>[AZURE.NOTE] 如果你使用 GitHub，可以选择在创建存储库时生成 Visual Studio 特定 .gitignore 文件，其中包括所有常见的临时文件和生成结果等。然后可以对其进行自定义以满足你的特定需求。

-	将整个解决方案的目录树添加到你的存储库中，其中 .sln 文件位于存储库根中。

按照说明设置存储库并将 Azure 中的 Web 应用配置为从其中一个联机 Git 存储库连续发布后，就可以在 Visual Studio 中从本地开发 ASP.NET 应用程序，并且只需将所做更改推送到联机 Git 存储库即可部署代码。

## <a id="Step8"></a>故障排除

以下是使用 Git 发布到 Azure 中的 Web 应用时遇到的常见错误或问题：

****

**症状**：无法访问“[siteURL]”：无法连接到 [scmAddress]

**原因**：如果 Web 应用无法启动和运行，则会出现此错误。

**解决方法**：在 Azure 经典管理门户中启动 Web 应用。在 Web 应用运行之前，Git 部署无法进行。


****

**症状**：无法解析主机“主机名”

**原因**：如果创建“azure”远程 Web 应用时输入的地址信息不正确，则会发生该错误。

**解决方法**：使用 `git remote -v` 命令列出所有远程 Web 应用以及关联的 URL。确认“azure”远程 Web 应用的 URL 正确。如果需要，请删除此远程 Web 应用并使用正确的 URL 重新创建它。

****

**症状**：无通用引用且未指定任何引用；不采取任何措施。或许你应指定一个分支，例如“master”。

**原因**：如果你在执行 Git 推送操作时未指定分支且未设置 Git 使用的 push.default 值，则会发生该错误。

**解决方法**：请再次执行推送操作，并指定 master 分支。例如：

	git push azure master

****

**症状**：src refspec [分支名] 不匹配任何内容。

**原因**：如果你尝试推送到“azure”远程 Web 应用上 master 分支之外的分支，则会发生该错误。

**解决方法**：请再次执行推送操作，并指定 master 分支。例如：

	git push azure master

****

**症状**：错误 - 已将更改提交到远程存储库，但未更新你的 Web 应用。

**原因**：如果你部署的是 Node.js 应用程序，其中包含用于指定其他必需模块的 package.json 文件，则会发生该错误。

**解决方法**：应在发生此错误前记录包含“npm ERR!”的其他消息，并可提供有关失败的其他上下文。以下是该错误的已知原因和相应的“npm ERR!”消息：

* **package.json 文件格式不正确**：npm ERR! 无法读取依赖项。

* **不具有 Windows 的二进制分发的本机模块**：

	* npm ERR! `cmd "/c" "node-gyp rebuild"` failed with 1

		或者

	* npm ERR! [modulename@version] preinstall: `make || gmake`


## 其他资源

* [如何使用适用于 Azure 的 PowerShell]
* [如何使用针对 Mac 和 Linux 的 Azure 命令行工具]
* [Git 文档]

[Azure Developer Center]: /documentation/
[Azure 经典管理门户]: https://manage.windowsazure.cn
[Git website]: http://git-scm.com
[安装 Git]: http://git-scm.com/book/zh/v2/%E8%B5%B7%E6%AD%A5-%E5%AE%89%E8%A3%85-Git
[如何使用适用于 Azure 的 PowerShell]: /documentation/articles/powershell-install-configure/
[如何使用针对 Mac 和 Linux 的 Azure 命令行工具]: /documentation/articles/xplat-cli-install/
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
[快速入门 - Mercurial]: http://mercurial.selenic.com/wiki/QuickStart
[使用 Dropbox 共享 Git 存储库]: https://gist.github.com/trey/2722927
[Continuous delivery to Azure using Visual Studio Team Services]: /documentation/articles/cloud-services-continuous-delivery-use-vso/

<!---HONumber=Mooncake_0328_2016-->