<properties
    pageTitle="在 Eclipse 中调试 Azure 应用程序"
    description="了解如何使用 Azure Toolkit for Eclipse 调试 Azure 应用程序。"
    services=""
    documentationCenter="java"
    authors="rmcmurray"
    manager="wpickett"
    editor=""/>

<tags
    ms.service="multiple"
    ms.date="05/04/2016" 
    wacn.date="06/13/2016"/>

<!-- Legacy MSDN URL = https://msdn.microsoft.com/library/azure/hh690949.aspx -->

# 在 Eclipse 中调试 Azure 应用程序 #

使用 Azure Toolkit for Eclipse，无论你的应用程序是在 Azure 中运行还是在计算模拟器中运行（如果你将 Windows 用作操作系统），都可进行调试。下图显示了用于启用远程调试的“调试”属性对话框：

![][ic719504]

本教程假设你已成功创建了一个应用程序，并且熟悉计算模拟器，知道如何部署到 Azure。

我们将使用[在 JSP 中使用 Azure 服务运行时库][]教程中的应用程序作为本主题的起点。在继续操作之前，请创建该应用程序（如果尚未这样做）。

## 在 Azure 中运行应用程序时调试应用程序 ##

>[AZURE.WARNING] 该工具包当前对远程 Java 调试的支持主要适用于在 Azure 计算模拟器中运行的部署。由于调试连接不安全，因此你不应在生产部署中启用远程调试。如果你需要调试 Azure 云中运行的应用程序，请使用过渡部署，但应该认识到，如果有人知道你云部署（即使是过渡部署）的 IP 地址，则可能会在未经授权的情况下访问你的调试会话。

1. 生成项目以在模拟器中进行测试：在 Eclipse 的项目资源管理器中，右键单击“MyAzureProject”，单击“属性”，单击“Azure”，然后将“生成目的”设置为“部署到云”。
1. 重新生成项目：在 Eclipse 菜单中，单击“项目”，然后单击“全部生成”。
1. 将应用程序部署到 Azure 中的过渡环境。
    >[AZURE.IMPORTANT] 如前所述，强烈建议你在大多数情况下都通过计算模拟器进行调试，仅当需要进一步调试时才在过渡环境中调试。不建议在生产环境中进行调试。
1. 准备好部署到 Azure 后，请从 [Azure 管理门户][]获取部署的 DNS 名称。过渡部署的 DNS 名称格式为 http://*&lt;guid&gt;*.cloudapp.net，其中，*&lt;guid&gt;* 是 Azure 分配的 GUID 值。
1. 在 Eclipse 的项目资源管理器中，右键单击“WorkerRole1”，单击“Azure”，然后单击“调试”。
1. 在“WorkerRole1 调试属性”对话框中：
    1. 选中“为此角色启用远程调试”。
    1. 对于“要使用的输入终结点”，请使用“调试(public:8090,private:8090)”。
    1. 确保已取消选中“以暂停模式启动 JVM，等待连接调试器”。
        >[AZURE.IMPORTANT] “以暂停模式启动 JVM，等待连接调试器”选项仅适用于在计算模拟器中进行的高级调试方案（而不适用于云部署）。如果使用了“以暂停模式启动 JVM，等待连接调试器”选项，它将会暂停服务器的启动进程，直到 Eclipse 调试器已连接到其 JVM。尽管你可以使用此选项来完成通过计算模拟器进行的调试会话，但不要使用它来完成云部署中进行的调试会话。服务器的初始化在 Azure 启动任务中发生，在完成启动任务之前，Azure 云不会使公共终结点可用。因此，如果在云部署中启用此选项，启动进程将不成功完成，因为它无法接收来自外部 Eclipse 客户端的连接。
    1. 单击“创建调试配置”。
1. 在“Azure 调试配置”对话框中：
    1. 对于“要调试的 Java 项目”，请选择“MyHelloWorld”项目。
    1. 对于“配置调试目标”，请选中“Azure 云(过渡)”。
    1. 确保已取消选中“Azure 计算模拟器”。
    1. 对于“主机”，请输入过渡部署的 DNS 名称，但不要输入前面的 **http://**。例如（使用你的 GUID 来代替此处所示的 GUID）：**4e616d65-6f6e-6d65-6973-526f62657274.cloudapp.net**
1. 单击“确定”关闭“Azure 调试配置”对话框。
1. 单击“确定”关闭“WorkerRole1 调试属性”对话框。
1. 如果尚未在 index.jsp 中设置断点，现在请设置它：
    1. 在 Eclipse 的项目资源管理器中，依次展开“MyHelloWorld”和“WebContent”，然后双击“index.jsp”。
    1. 在 index.jsp 中，右键单击 Java 代码左侧的蓝色条形，然后单击“切换断点”，如下图所示：
        ![][ic551537]
1. 在 Eclipse 的菜单中，单击“运行”，然后单击“调试配置”。
1. 在“调试配置”对话框的左窗格中，展开“远程 Java 应用程序”，选择“Azure 云(WorkerRole1)”，然后单击“调试”。
1. 在浏览器中运行你的过渡应用程序 **http://***&lt;guid&gt;***.cloudapp.net/MyHelloWorld**，并使用你的 DNS 名称中的 GUID 取代 *&lt;guid&gt;*。如果“确认角度切换”对话框有提示，请单击“是”。现在，你的调试会话应该执行到设置了断点的代码行。

>[AZURE.NOTE] 如果你正在尝试与包含多个运行中角色实例的部署建立远程调试连接，则目前你无法控制调试器最初要连接到的实例，因为 Azure 负载平衡器将随机选取一个实例。不过，一旦与该实例建立连接，你就可以继续调试同一个实例。另请注意，如果连接保持非活动状态 4 分钟以上（例如，你在某个断点处停止太长的时间），Azure 可能会关闭该连接。

## 调试多实例部署中的特定角色实例 ##

当你的部署在云中运行时，你很可能会在多个计算或角色实例中运行该部署。这样，你便可以利用 Azure 99.95% 的可用性保证，并扩大你的应用程序。

在这种情况下，你可能需要远程调试特定角色实例中的 Java 应用程序。但是，如果你只是为调试启用了常规输入终结点，则 Azure 负载平衡器基本上不允许你将调试器连接到特定的角色实例。它会将你连接到它随机选取的角色实例。

这是利用实例输入终结点简化特定角色实例调试的方案类型。

假设你打算运行部署的最多 5 个角色实例。请使用角色属性对话框中的“终结点”属性页创建实例输入终结点，并为其分配一个公用端口范围而不是一个端口号。例如，在“公用端口”输入框中指定 **81-85**。

在你使用此实例终结点部署应用程序后，Azure 将为每个角色实例分配此范围中的唯一端口号。然后，若要找出为哪个实例分配了哪个端口号，你可以使用工具包在部署中自动分配（例如，在网页页脚中返回值，使你可以在浏览时读取它）的 *InstanceEndpointName***\_PUBLICPORT** 环境变量（其中，*InstanceEndpointName* 是你在创建实例终结点时分配的名称）。

在了解为实例分配的公用端口号后，可以通过将它附加到服务主机名，在 Eclipse 中的调试配置中引用该端口号。这样，Eclipse 调试器便可以连接到该特定实例，而不是任何其他实例。

## 仅限 Windows：在计算模拟器中运行应用程序时调试应用程序 ##

>[AZURE.NOTE] Azure 模拟器仅在 Windows 上可用。如果你使用的是非 Windows 操作系统，请跳过本部分。

1. 生成项目以在模拟器中进行测试：在 Eclipse 的项目资源管理器中，右键单击“MyAzureProject”，单击“属性”，单击“Azure”，然后将“生成目的”设置为“在模拟器中测试”。
1. 重新生成项目：在 Eclipse 菜单中，单击“项目”，然后单击“全部生成”。
1. 在 Eclipse 的项目资源管理器中，右键单击“WorkerRole1”，单击“Azure”，然后单击“调试”。
1. 在“WorkerRole1 调试属性”对话框中：
    1. 选中“为此角色启用远程调试”。
    1. 对于“要使用的输入终结点”，请使用工具包自动生成的默认终结点，它列为“调试(public:8090,private:8090)”。
    1. 确保已取消选中“以暂停模式启动 JVM，等待连接调试器”选项。
        >[AZURE.IMPORTANT] “以暂停模式启动 JVM，等待连接调试器”选项仅适用于在计算模拟器中进行的高级调试方案（而不适用于云部署）。如果使用了“以暂停模式启动 JVM，等待连接调试器”选项，它将会暂停服务器的启动进程，直到 Eclipse 调试器已连接到其 JVM。尽管你可以使用此选项来完成通过计算模拟器进行的调试会话，但不要使用它来完成云部署中进行的调试会话。服务器的初始化在 Azure 启动任务中发生，在完成启动任务之前，Azure 云不会使公共终结点可用。因此，如果在云部署中启用此选项，启动进程将不成功完成，因为它无法接收来自外部 Eclipse 客户端的连接。
    1. 单击“创建调试配置”。
1. 在“Azure 调试配置”对话框中：
    1. 对于“要调试的 Java 项目”，请选择“MyHelloWorld”项目。
    1. 对于“配置调试目标”，请选中“Azure 计算模拟器”。
1. 单击“确定”关闭“Azure 调试配置”对话框。
1. 单击“确定”关闭“WorkerRole1 调试属性”对话框。
1. 在 Index.jsp 中设置一个断点：
    1. 在 Eclipse 的项目资源管理器中，依次展开“MyHelloWorld”和“WebContent”，然后双击“index.jsp”。
    1. 在 index.jsp 中，右键单击 Java 代码左侧的蓝色条形，然后单击“切换断点”，如下图所示：
        ![][ic551537]
       如果你在 Java 代码左侧的蓝色条形中看到了一个断点图标，则表示已设置断点。
1. 在计算模拟器中的 Azure 工具栏上单击“在 Azure 模拟器中运行”按钮，以启动应用程序。
1. 在 Eclipse 的菜单中，单击“运行”，然后单击“调试配置”。
1. 在“调试配置”对话框的左窗格中，展开“远程 Java 应用程序”，选择“Azure 模拟器(WorkerRole1)”，然后单击“调试”。
1. 在计算模拟器指出你的应用程序正在运行后，请在浏览器中运行 **http://localhost:8080/MyHelloWorld**。
    如果“确认角度切换”对话框有提示，请单击“是”。
    现在，你的调试会话应该执行到设置了断点的代码行。

本部分说明了如何在计算模拟器中进行调试；下一部分将说明如何调试已部署到 Azure 的应用程序。

## 调试说明 ##

* 调试后，可以将透视图从“调试”切换到“Java”，方法是单击 Eclipse 的菜单，单击“窗口”、“打开透视图”，然后选择你要使用的透视图。
* 若要在 GlassFish 中启用远程调试，请不使用 Azure Toolkit for Eclipse 的远程调试配置功能，而要手动配置 GlassFish。由于 GlassFish 处理环境变量中预定义的 Java 选项的方式问题，该工具包的远程调试配置功能不能在 GlassFish 上正常工作。如果已启用该工具包的远程调试配置功能，则它可能会阻止 GlassFish 启动。

## 另请参阅 ##

[适用于 Eclipse 的 Azure 工具包][]

[在 Eclipse 中为 Azure 创建 Hello World 应用程序][]

[安装 Azure Toolkit for Eclipse][]

有关将 Azure 与 Java 配合使用的详细信息，请参阅 [Azure Java 开发人员中心][]。

<!-- URL List -->

[Azure Java 开发人员中心]: /develop/java/
[Azure 管理门户]: http://manage.windowsazure.cn
[适用于 Eclipse 的 Azure 工具包]: /documentation/articles/azure-toolkit-for-eclipse/
[在 Eclipse 中为 Azure 创建 Hello World 应用程序]: /documentation/articles/azure-toolkit-for-eclipse-creating-a-hello-world-application/
[安装 Azure Toolkit for Eclipse]: /documentation/articles/azure-toolkit-for-eclipse-installation/
[在 JSP 中使用 Azure 服务运行时库]: /develop/java/

<!-- IMG List -->

[ic719504]: ./media/azure-toolkit-for-eclipse-debugging-azure-applications/ic719504.png
[ic551537]: ./media/azure-toolkit-for-eclipse-debugging-azure-applications/ic551537.png

<!---HONumber=Mooncake_0606_2016-->