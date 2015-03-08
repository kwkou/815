**如果这些条件都成立**，Azure 将确定您的应用程序使用 Python：

- 根文件夹中的 requirements.txt 文件
- 根文件夹中的任何 .py 文件或指定 python 的 runtime.txt

如果是这种情况，它将使用特定于 Python 的部署脚本，此脚本执行文件的标准同步以及其他 Python 操作，例如：

- 自动管理虚拟环境
- 使用 pip 来安装 requirements.txt 中列出的软件包
- 根据所选的 Python 版本创建相应的 web.config。
- 收集 Django 应用程序的静态文件

您可以控制默认部署步骤的某些方面，而无需自定义脚本。

如果您想要跳过所有特定于 Python 的部署步骤，可以创建此空文件：

    \.skipPythonDeployment

如果您想要跳过为 Django 应用程序收集静态文件的操作：

    \.skipDjango 

为了更大程度控制部署，可以通过创建以下文件来覆盖默认部署脚本：

    \.deployment
    \deploy.cmd

您可以使用 [Azure 命令行界面][]来创建这些文件。从项目文件夹使用以下命令：

    azure site deploymentscript --python

这些文件不存在时，Azure 创建一个临时部署脚本然后运行此脚本。它等同于使用以上命令创建的脚本。

[Azure 命令行界面]: /zh-cn/downloads/
