<properties
    pageTitle="使用 Jenkins 针对 Azure Service Fabric Linux Java 应用程序进行持续生成和集成 | Azure"
    description="使用 Jenkins 针对 Linux Java 应用程序进行持续生成和集成"
    services="service-fabric"
    documentationcenter="java"
    author="sayantancs"
    manager="timlt"
    editor=""
    translationtype="Human Translation" />
<tags
    ms.assetid="02b51f11-5d78-4c54-bb68-8e128677783e"
    ms.service="service-fabric"
    ms.devlang="java"
    ms.topic="hero-article"
    ms.tgt_pltfrm="NA"
    ms.workload="NA"
    ms.date="02/27/2017"
    wacn.date="04/24/2017"
    ms.author="saysa"
    ms.sourcegitcommit="a114d832e9c5320e9a109c9020fcaa2f2fdd43a9"
    ms.openlocfilehash="335f41e2e4a40e64e87382ea338673de3ab79c27"
    ms.lasthandoff="04/14/2017" />

# <a name="build-and-deploy-your-linux-java-application-using-jenkins"></a>使用 Jenkins 生成和部署 Linux Java 应用程序
Jenkins 是常用的持续集成和部署工具。 本文档介绍如何使用 Jenkins 生成和部署 Service Fabric 应用程序。

## <a name="general-prerequisites"></a>常规先决条件
1. 在本地安装 git。 可以根据 OS 从[此处](https://git-scm.com/downloads)安装适当的 git 版本。 如果不熟悉 git，可以执行[文档](https://git-scm.com/docs)中提及的步骤，自行熟悉 git。
2. 准备好 Service Fabric Jenkins 插件。 从[此处](https://servicefabricdownloads.blob.core.windows.net/jenkins/serviceFabric.hpi)下载 Service Fabric Jenkins 插件。

## <a name="setting-up-jenkins-inside-a-service-fabric-cluster"></a>在 Service Fabric 群集中设置 Jenkins

### <a name="prerequisites"></a>先决条件
1. 准备好 Service Fabric Linux 群集。 通过 Azure 门户创建的 Service Fabric 群集已安装 Docker。 如果是在本地运行群集，请使用命令 ``docker info`` 查看 Docker 是否已安装。如果未安装，则请使用以下命令相应地进行安装：


		  sudo apt-get install wget
		  wget -qO- https://get.docker.io/ | sh
	
2. 执行以下步骤，在群集上部署 Service Fabric 容器应用程序：


		git clone https://github.com/Azure-Samples/service-fabric-java-getting-started.git -b JenkinsDocker
		cd service-fabric-java-getting-started/Services/JenkinsDocker/
		azure servicefabric cluster connect http://PublicIPorFQDN:19080   # Azure CLI cluster connect command
		bash Scripts/install.sh

这样就会在群集上安装 Jenkins 容器，并且可以使用 Service Fabric Explorer 进行监视。

### <a name="steps"></a>步骤
1. 从浏览器转到 URL `http://PublicIPorFQDN:8081`。 该 URL 会提供登录所需的初始管理员密码的路径。 可以继续使用 Jenkins 作为管理员用户，也可以在使用初始管理员帐户登录后创建和更改用户。

    > [AZURE.NOTE]
    > 需确保在创建群集时将端口 8081 指定为应用程序终结点端口
    >

2. 使用 `docker ps -a` 获取容器实例 ID。

3. 以 SSH 方式登录到容器，粘贴在 Jenkins 门户中显示的路径。 例如，如果在门户中显示的路径为 `PATH_TO_INITIAL_ADMIN_PASSWORD`，则可执行以下命令：


		  docker exec -t -i [first-four-digits-of-container-ID] /bin/bash   # This takes you inside Docker shell
		  cat PATH_TO_INITIAL_ADMIN_PASSWORD


4. 执行[此链接](https://help.github.com/articles/generating-a-new-ssh-key-and-adding-it-to-the-ssh-agent/)中提到的步骤，设置适用于 Jenkins 的 GitHub。

    * 根据 GitHub 中提供的说明生成 SSH 密钥，然后将 SSH 密钥添加到托管存储库的 GitHub 帐户。
    * 在 Jenkins Docker 外壳程序中（不是在主机上）运行上述链接中提到的命令
    * 若要从主机登录到 Jenkins 外科程序，请使用以下命令：


		      docker exec -t -i [first-four-digits-of-container-ID] /bin/bash


## <a name="setting-up-jenkins-outside-a-service-fabric-cluster"></a>在 Service Fabric 群集外设置 Jenkins

### <a name="prerequisites"></a>先决条件
需安装 Docker。 若要从终端安装 Docker，可以使用以下命令：


	  sudo apt-get install wget
	  wget -qO- https://get.docker.io/ | sh


现在，当用户在终端运行 ``docker info`` 时，就会在输出中看到 Docker 服务正在运行。

### <a name="steps"></a>步骤
  1. 拉取 Service Fabric Jenkins 容器映像：`docker pull servicefabric-microsoft.azurecr.io/jenkins:v1`

  2. 运行容器映像：`docker run -itd -p 8080:8080 servicefabric-microsoft.azurecr.io/jenkins:v1`

  3. 获取容器映像实例的 ID。 可以使用命令 `docker ps –a` 列出所有 Docker 容器

  4. 执行以下步骤，登录到 Jenkins 门户：

    * `docker exec [first-four-digits-of-container-ID] cat /var/jenkins_home/secrets/initialAdminPassword`
    如果容器 ID 为 2d24a73b5964，请使用 2d24。
    * 从门户 ``http://<HOST-IP>:8080`` 登录到 Jenkins 仪表板时，需要此密码
    * 首次登录以后，即可创建自己的用户帐户以备将来之用，也可继续使用管理员帐户。 一旦创建用户，接下来需使用这个用户。
  5. 执行[此链接](https://help.github.com/articles/generating-a-new-ssh-key-and-adding-it-to-the-ssh-agent/)中提到的步骤，设置适用于 Jenkins 的 GitHub。
      * 根据 GitHub 中提供的说明生成 SSH 密钥，然后将 SSH 密钥添加到托管存储库的 GitHub 帐户。
      * 在 Jenkins Docker 外壳程序中（不是在主机上）运行上述链接中提到的命令
      * 若要从主机登录到 Jenkins 外科程序，请使用以下命令：


	          docker exec -t -i [first-four-digits-of-container-ID] /bin/bash


    > [AZURE.NOTE]
    > 请确保托管 Jenkins 容器映像的群集/计算机使用面向公众的 IP，以便 Jenkins 实例接收来自 GitHub 的通知。
    >

## <a name="install-the-service-fabric-jenkins-plugin-from-portal"></a>从门户安装 Service Fabric Jenkins 插件

1. 转到 `http://PublicIPorFQDN:8081`

2. 在 Jenkins 仪表板中，选择“管理 Jenkins” -> “管理插件” -> “高级”。
可以在这里上载插件。 选择“选择文件”选项，然后选择 serviceFabric.hpi 文件（已在“先决条件”部分下载）。 一旦用户选择“上载”，Jenkins 就会自动安装该插件。 如果系统请求重新启动，请允许。

## <a name="creating-and-configuring-a-jenkins-job"></a>创建和配置 Jenkins 作业

1. 从仪表板创建**新项**

2. 输入项目名称（例如 **MyJob**），选择“自由样式项目”，然后单击“确定”

3. 然后转到作业页，单击“配置” -
  * 在常规部分的“Github 项目”下指定 GitHub 项目 URL，以便托管用于集成 Jenkins CI/CD 流（例如 ``https://github.com/sayantancs/SFJenkins``）的 Service Fabric Java 应用程序。
  * 在“源代码管理”部分，选择 **Git**。 指定存储库 URL，以便托管用于集成 Jenkins CI/CD 流（例如 ``https://github.com/sayantancs/SFJenkins.git``）的 Service Fabric Java 应用程序。 也可在此处指定要生成的分支，例如 ***/master**。

4. 执行以下步骤，以便配置 *GitHub*（即托管存储库的地方），使之能够与 Jenkins 通信：

  1. 转到 GitHub 存储库页。 转到“设置” -> “集成和服务”。

  2. 选择“添加服务”，键入 Jenkins，选择“Jenkins-Github 插件”。

  3. 输入 Jenkins Webhook URL（默认为 `http://<PublicIPorFQDN>:8081/github-webhook/`）。 单击“添加/更新服务”。

  4. 将向 Jenkins 实例发送一个测试事件。 用户会在 GitHub 中 Webhook 的旁边看到一个绿色复选标记，说明可以生成项目！

  5. 在“生成触发器”部分选择需要的生成选项。就此用例来说，我们计划在有内容推送到存储库时触发一个生成，因此相应的选项为“适用于 GITScm 轮询的 GitHub 挂钩触发器”（以前为“将更改推送到 GitHub 时生成”）

  6. 在“生成”部分：从“添加生成步骤”下拉列表中选择“调用 Gradle 脚本”选项。 在附带的小组件中，为应用程序指定“根生成脚本”的路径。 它将从指定的路径中选取 build.gradle，然后进行相应的操作。 如果创建名为 `MyActor` 的项目（使用 Eclipse 插件或 Yeoman 生成器），则根生成脚本应包含 ``${WORKSPACE}/MyActor``。 例如，此部分很可能如下所示：

      ![Service Fabric Jenkins 生成操作][build-step]

  7. 在“生成后操作”下拉列表中，选择“部署 Service Fabric 项目”。 此处需提供群集详细信息（在何处部署 Jenkins 编译的 Service Fabric 应用程序）以及其他应用程序详细信息（用于部署应用程序）。 可以使用以下屏幕截图作为参考：

      ![Service Fabric Jenkins 生成操作][post-build-step]

 > [AZURE.NOTE]
 > 如果使用 Service Fabric 部署 Jenkins 容器映像，则此处的群集可以与托管 Jenkins 容器应用程序的群集相同。
 >

### <a name="end-to-end-scenario"></a>端到端方案
现在已配置 GitHub 和 Jenkins，因此可以在存储库（例如 https://github.com/sayantancs/SFJenkins）的 ``MyActor`` 项目中进行一些示例性的更改，并将所做的更改推送到远程 ``master`` 分支（或任何已配置的可以使用的分支）。 此时会触发已配置的 Jenkins 作业 ``MyJob``。 其基本功能包括：从 GitHub 提取更改、生成这些更改并将应用程序部署到在生成后操作中指定的群集终结点。  

  <!-- Images -->
  [build-step]: ./media/service-fabric-cicd-your-linux-java-application-with-jenkins/build-step.png
  [post-build-step]: ./media/service-fabric-cicd-your-linux-java-application-with-jenkins/post-build-step.png