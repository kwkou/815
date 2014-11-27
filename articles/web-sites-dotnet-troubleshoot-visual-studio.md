<properties title="在 Visual Studio 中排除 Azure 网站的故障" pageTitle="在 Visual Studio 中排除 Azure 网站的故障" metaKeywords="troubleshoot debug azure web site tracing logging" description="了解如何通过内置于 Visual Studio 2013 的远程调试、 跟踪和日志记录工具排除 Azure 网站的故障。" metaCanonical="" services="web-sites" documentationCenter=".NET" authors="tdykstra" solutions="" />

# 在 Visual Studio 中排除 Azure 网站的故障

在开发与测试 Web 应用程序期间，您可以[运行在调试模式下][运行在调试模式下]或使用 [IntelliTrace][IntelliTrace] 进行故障排除。可通过 IIS Express 本地运行或 Azure 网站远程运行调试模式。但是对于仅在生产环境中出现的错误，最佳调试方法可能是查看应用程序代码或 Web 服务器创建的日志。本教程将向您介绍如何使用 Visual Studio 工具，以通过远程运行调试模式或查看应用程序和 Web 服务器日志帮助调试运行于 Azure 网站的应用程序。

您将了解到以下内容：

-   Visual Studio 中提供的 Azure 站点管理功能。
-   如何使用 Visual Studio 远程视图在远程网站中进行快速更改。
-   如何在项目正运行于 Azure 网站时远程运行调试模式。
-   如何创建应用程序跟踪日志并在该应用程序正在创建日志时对其进行查看。
-   如何查看 Web 服务器日志，包括详细的错误消息和失败请求跟踪。
-   如何将诊断日志发送至 Azure 存储帐户并在其中进行查看。

### 教程章节

1.  [先决条件][先决条件]
2.  [站点配置和管理][站点配置和管理]
3.  [远程视图][远程视图]
4.  [远程调试][远程调试]
5.  [诊断日志概述][诊断日志概述]
6.  [创建并查看应用程序跟踪日志][创建并查看应用程序跟踪日志]
7.  [查看 Web 服务器日志][查看 Web 服务器日志]
8.  [查看详细的错误消息日志][查看详细的错误消息日志]
9.  [下载文件系统日志][下载文件系统日志]
10. [查看存储日志][查看存储日志]
11. [查看失败请求日志][查看失败请求日志]
12. [后续步骤][后续步骤]

## <a name="prerequisites"></a><span class="short-header">先决条件</span>

本教程适用于 [Azure 和 ASP.NET 入门][Azure 和 ASP.NET 入门]中设置的开发环境、Web 项目和 Azure 网站。在本教程中所示的代码示例适用于 C# MVC Web 应用程序，但对于 Visual Basic 和 Web 窗体应用程序，故障排除过程是一样的。

远程调试需要 Visual Studio 2013 或 Visual Studio 2012 Update 4。本教程中所示的其他功能也适用于 Visual Studio 2013 Express for Web 和 Visual Studio 2012 Express for Web。

流式日志功能仅适用于面向 .NET Framework 4 或更高版本的应用程序。

## <a name="sitemanagement"></a> 站点配置和管理

通过 Visual Studio，用户可以访问管理门户中提供的站点管理功能和配置设置的子集。本节将向您介绍具体提供的功能。

1.  如果您还未在 Visual Studio 中登录到 Azure，请单击“服务器资源管理器”中的“连接至 Azure”按钮。

    一种替代方法是安装可以访问您的帐户的管理证书。管理证书可以让服务器资源管理器访问其他 Azure 服务（SQL Database 和移动服务）。如果选择安装证书，请右键单击“服务器资源管理器”中的“Azure”，然后单击上下文菜单中的“管理订阅”。。在“管理 Azure 订阅”对话框中，单击“证书”选项卡，然后单击“导入”。按照说明为您的 Azure 帐户下载并导入一个订阅文件（也称为 *.publishsettings* 文件）。

    > [WACOM.NOTE]
    > 将此订阅文件下载并保存到源代码目录之外的文件夹中（例如，在 Downloads 文件夹中），然后在导入完成后将其删除。获得了此订阅文件访问权的恶意用户可以编辑、创建和删除您的 Azure 服务。

    有关从 Visual Studio 连接至 Azure 资源的更多信息，请参见[管理帐户、订阅和管理角色][管理帐户、订阅和管理角色]。

2.  在“服务资源管理器”中，展开“Azure”，然后展开“网站”。

3.  右键单击在 [Azure 和 ASP.NET 入门][Azure 和 ASP.NET 入门]中创建的网站的节点，然后单击“查看设置”。

    ![在服务器资源管理器中查看设置][在服务器资源管理器中查看设置]

    “Azure 网站”选项卡出现，您可在此处看到 Visual Studio 中提供的站点管理和配置任务。

    ![Azure 网站窗口][Azure 网站窗口]

    本教程将向您介绍如何使用日志记录和跟踪下拉列表。其中，还将用到远程调试，但会通过不同方式启用该调试。

    有关该窗口中“应用程序设置”和“连接字符串”方框的信息，请参见[Azure 网站：应用程序字符串和连接字符串的工作原理][Azure 网站：应用程序字符串和连接字符串的工作原理]。

    如果您想执行的站点管理任务无法在此窗口进行，可单击“完整的网站设置”打开一个到管理门户的浏览器窗口。有关更多信息，请参见[如何配置网站][如何配置网站]。

## <a name="remoteview"></a>远程视图

部署站点时，Web.config 文件中的 `customErrors` 标志通常设置为 `On` 或 `RemoteOnly`，这意味着如果出现问题您不会获得任何有帮助的错误消息。无论发生何种错误，您获得的都是类似如下所示的页面。

**'/' 应用程序中出现服务器错误：**
![毫无用处的错误页面][毫无用处的错误页面]

**发生错误：**
![毫无用处的错误页面][1]

**网站无法显示页面**
![毫无用处的错误页面][2]

找出错误原因的最简便方法通常是启用详细错误消息，如此，之前保留的屏幕快照中的第一个快照会告诉您如何做。而实现此目的需要在部署的 Web.config 文件中进行更改。您可以编辑项目中的 *Web.config* 文件，然后重新部署该项目，或创建 [Web.config 转换][Web.config 转换]并部署调试版本，但还有一个更快捷的方式：在“解决方案资源管理器”中，可以通过使用*远程视图*功能直接查看并编辑远程站点上的文件。

1.  在“服务器资源管理器”内，依次展开“Azure”、“网站”以及您打算部署到该网站的节点。

    该节点会为您提供到该网站的内容文件和日志文件的访问。

    ![文件和日志文件][文件和日志文件]

2.  展开“文件”节点，双击 *Web.config* 文件。

    ![打开 Web.config][打开 Web.config]

    Visual Studio 将从远程站点打开 Web.config 文件，并在标题栏中的文件名旁显示 [Remote]。

3.  将以下行添加至 `system.web` 元素：

    `<customErrors mode="off"></customErrors>`

    ![编辑 Web.config][编辑 Web.config]

4.  刷新显示无用错误消息的浏览器，现在，您将获得详细的错误消息，如下所示：

    ![详细的错误消息][详细的错误消息]

    （通过将以红色显示的行添加到 *Views\\Home\\Index.cshtml* 创建显示的错误。）

编辑 Web.config 文件示例向您演示了能够读取并编辑 Azure 网站上的文件使得故障排除变得更加简单，而这仅仅只是其中之一。

## <a name="remotedebug"></a>远程调试

如果详细的错误消息提供的信息量不够，而您无法本地重新创建该错误，则可以采用远程运行调试模式来进行故障排除.您可以设置断点、直接操作内存、逐行执行代码，甚至更改代码路径。

远程调试不适用于 Visual Studio Express 版。

本节将向您介绍如何使用[Azure 和 ASP.NET 入门][Azure 和 ASP.NET 入门] 中创建的项目进行远程调试。

1.  打开在 [Azure 和 ASP.NET 入门][Azure 和 ASP.NET 入门]中创建的 Web 项目。

2.  打开 *Controllers\\HomeController.cs*。

3.  删除 `About()` 方法并在其位置插入以下代码。

        public ActionResult About()
        {
            string currentTime = DateTime.Now.ToLongTimeString();
            ViewBag.Message = "The current time is " + currentTime;
            return View();
        }

4.  在 `ViewBag.Message` 行上[设置断点][运行在调试模式下]。

5.  在“解决方案资源管理器”中，右键单击该项目并单击“发布”。

6.  在“配置文件”下拉列表中，选择与 [Azure 和 ASP.NET 入门][Azure 和 ASP.NET 入门]中所用相同的配置文件。

7.  单击“设置”选项卡，将“配置”更改为“调试”，然后单击“发布”。

    ![在调试模式下发布][在调试模式下发布]

8.  部署结束后，浏览器将打开您的站点的 Azure URL，关闭浏览器。

9.  对于 Visual Studio 2013：在“服务器资源管理器”中依次展开“Azure”和“网站”，右键单击您的网站，然后单击“附加调试程序”。

    ![附加调试程序][附加调试程序]

    浏览器将自动打开您运行在 Azure 中的主页。您可能必须等待 20 秒左右以便 Azure 针对调试设置服务器。此延迟只在首次于网站上运行调试模式时出现。接下来的 48 小时内，再次启动调试将不会出现延迟。

10. 对于 Visual Studio 2012 Update 4：

    -   在 Azure 管理门户中，转至您的站点的“配置”选项卡，然后向下滚至“站点诊断”部分。

    -   将“远程调试”设置为“开启”，然后将“远程调试 Visual Studio 版本”设置为“2012”。。

    ![在管理门户中设置远程调试][在管理门户中设置远程调试]

    -   在 Visual Studio 的“调试”菜单中，单击“附加到进程”。

    -   在“限定符”框中，输入您的网站的 URL 中，无需前缀 `http://`。

    -   选择“显示来自所有用户的进程”。

    -   当系统提示输入凭据时，输入有权发布该网站的用户的名称和密码。若要获取这些凭据，请转至管理门户中网站的“仪表板”选项卡，单击“下载发布配置文件”。在文本编辑器中打开该文件，可在“userName=”和“userPWD=”第一次出现的地方找到用户名和密码。

    -   当进程出现在“可用进程”表中后，选择“w3wp.exe”然后单击“附加”。

    -   打开浏览器，输入您的站点 URL。

    您可能必须等待 20 秒左右以便 Azure 针对调试设置服务器。此延迟只在首次于网站上运行调试模式时出现。接下来的 48 小时内，再次启动调试将不会出现延迟。

11. 单击菜单中的“关于”。

    Visual Studio 将在断点处停止，代码在 Azure 中运行，而不是在您的本地计算机上。

12. 将鼠标悬停在 `currentTime` 变量上查看时间值。

    ![在运行于 Azure 中的调试模式下查看变量][在运行于 Azure 中的调试模式下查看变量]

    您看到时间是 Azure 服务器时间，可能与您本地计算机所处时区不同。

13. 为 `currentTime` 变量输入一个新值，如“Now running in Azure”。

14. 按 F5 继续运行。

    运行于 Azure 中的“关于”页面将显示您在 currentTime 变量中输入的新值。

    ![显示新值的关于页面][显示新值的关于页面]

### 有关远程调试的说明

-   不建议在生产环境中以调试模式运行。如果您的生产站点未进行扩展以容纳多个服务器实例，调试将占用唯一 Web 服务器实例的流量。如果您具有多个 web 服务器实例，在附加至调试程序时将获得一个随机实例，而您无法确保浏览器请求将前往该实例。此外，调试版本一般不会部署到生产环境，针对版本生成的编译器优化可能可以逐行显示源代码中出现的情况。至于如何解决生产环境中出现的问题，可利用的最佳资源是应用程序跟踪和 Web 服务器日志。

-   远程调试时避免长时间停止在断点处。Azure 会将停止时间超过几分钟的进程视为无反应进程而将其关闭。

-   进行调试的时候，服务器会向 Visual Studio 发送数据，这可能会影响到带宽费用。有关带宽费率的信息，请参见 [Azure 定价][Azure 定价]。

-   请确保 `compilation` 元素的 `debug` 属性（在 *Web.config* 文件中）设置为 true。在发布调试版本配置时，默认设置为 true。

        <system.web>
          <compilation debug="true" targetFramework="4.5" />
          <httpRuntime targetFramework="4.5" />
        </system.web>

-   如果发现调试程序并未针对希望调试的代码展开行动，您可能必须更改“仅我的代码”设置。有关更多信息，请参见[限制为仅逐行调试我的代码][限制为仅逐行调试我的代码]。

-   启用远程调试功能时，服务器上会出现一个计时器，48 小时后该功能将自动关闭。这一 48 小时的限制是出于安全性与性能的考虑。您可以根据需要轻松地多次重启该功能。我们建议，在不主动进行调试的时候，保持其处于禁用状态。

-   您可以手动将调试程序附加至任一进程。您不但可以调试网站进程 (w3wp.exe)，还可以调试其他进程，如 [WebJobs][WebJobs]。有关如何在 Visual Studio 中使用调试模式的信息，请参见[在 Visual Studio 中进行调试][在 Visual Studio 中进行调试]。

## <a name="logsoverview"></a><span class="short-header">日志概述</span>诊断日志概述

运行在 Azure 网站中的 ASP.NET 应用程序可以创建以下几类日志：

-   **应用程序跟踪日志**
    应用程序通过调用 [System.Diagnostics.Trace][System.Diagnostics.Trace] 类的方法创建这些日志。
-   **Web 服务器日志**
    Web 服务器为每一个对该站点的 HTTP 请求创建一个日志条目。
-   **详细的错误消息日志**
    Web 服务器为失败的 HTTP 请求（导致状态代码 400 或更大数字的请求）创建带有一些额外信息的 HTML 页面。
-   **失败请求跟踪日志**
    Web 服务器为失败的 HTTP 请求创建带有详细跟踪信息的 XML 文件。Web 服务器还可提供 XSL 文件以格式化浏览器中的 XML。

日志记录会影响站点性能，因此 Azure 允许您提供根据需要启用或禁用各种类型的日志。对于应用程序日志，可以指定只写入特定严重级别以上的日志。创建新的网站时，默认为禁用所有日志记录功能。

日志将写入网站文件系统中 *LogFiles* 文件夹内的文件。Web 服务器日志和应用程序日志也可写入 Azure 存储帐户。可在存储帐户中为日志留出大于文件系统预留量的空间。使用文件系统时，最多可存储 100 兆字节的日志。（文件系统日志仅适合短期保留。达到限制后，Azure 将删除旧日志文件以便为新日志腾出空间。）

## <a name="apptracelogs"></a><span class="short-header">应用程序日志</span>创建并查看应用程序跟踪日志

在本节中，您将执行以下任务：

-   将跟踪语句添加到在[之前教程][Azure 和 ASP.NET 入门]中创建的 Web 项目。
-   本地运行该项目时查看日志。
-   查看由运行于 Azure 中的应用程序生成的日志。

### 向应用程序添加跟踪语句

1.  打开 *Controllers\\HomeController.cs* 并使用以下代码替换文件内容，以便为 `System.Diagnostics` 添加 `Trace` 语句和 `using` 语句：

        using System;
        using System.Collections.Generic;
        using System.Configuration;
        using System.Diagnostics;
        using System.Linq;
        using System.Web;
        using System.Web.Configuration;
        using System.Web.Mvc;
        namespace MyExample.Controllers
        {
            public class HomeController : Controller
            {
                public ActionResult Index()
                {
                    Trace.WriteLine("Entering Index method");
                    ViewBag.Message = "Modify this template to jump-start your ASP.NET MVC application.";
                    Trace.TraceInformation("Displaying the Index page at " + DateTime.Now.ToLongTimeString());
                    Trace.WriteLine("Leaving Index method");
                    return View();
                }

                public ActionResult About()
                {
                    Trace.WriteLine("Entering About method");
                    ViewBag.Message = "Your app description page.";
                    Trace.TraceWarning("Transient error on the About page at " + DateTime.Now.ToShortTimeString());
                    Trace.WriteLine("Leaving About method");
                    return View();
                }

                public ActionResult Contact()
                {
                    Trace.WriteLine("Entering Contact method");
                    ViewBag.Message = "Your contact page.";
                    Trace.TraceError("Fatal error on the Contact page at " + DateTime.Now.ToLongTimeString());
                    Trace.WriteLine("Leaving Contact method");
                    return View();
                }
            }
        }

### 本地查看跟踪输出

1.  按 F5 以调试模式运行应用程序。

    默认跟踪侦听器将所有跟踪输出写入“输出”窗口，同时还有其他调试输出。 下图显示了来自添加至 `Index` 方法的跟踪语句的输出。

    ![在调试窗口中进行跟踪][在调试窗口中进行跟踪]

    以下步骤介绍了如何在网页中查看跟踪输出而无需在调试模式下进行编译。

2.  打开应用程序 Web.config 文件（位于项目文件夹中），将 `<system.diagnostics>` 元素添加到文件末尾的关闭 `</configuration>` 元素之前：

        <system.diagnostics>
            <trace>
              <listeners>
                <add name="WebPageTraceListener"
                    type="System.Web.WebPageTraceListener, 
                    System.Web, 
                    Version=4.0.0.0, 
                    Culture=neutral,
                    PublicKeyToken=b03f5f7f11d50a3a" />
              </listeners>
            </trace>
          </system.diagnostics>

    `WebPageTraceListener` 允许您通过浏览至 `/trace.axd` 查看跟踪输出。

3.  将[跟踪元素][跟踪元素]添加到 Web.config file 文件中的 `<system.web>` 下面，如下所示：

        <trace enabled="true" writeToDiagnosticsTrace="true" mostRecent="true" pageOutput="false" />

4.  按 Ctrl+F5 运行应用程序。

5.  在浏览器窗口的地址栏中，将 *trace.axd* 添加到 URL，然后按 Enter 键（URL 将类似于 http://localhost:53370/trace.axd）。

6.  在“应用程序跟踪”页面上，单击“查看细节”。

    ![trace.axd][trace.axd]

    “请求细节”页面显示，在“跟踪信息”部分您会看到添加至 `Index` 方法的跟踪语句的输出。

    ![trace.axd][3]

    默认情况下，`trace.axd` 仅供本地使用。如果希望可以从远程站点进行使用，可将 `localOnly="false"` 添加到 `trace` 元素（位于 *Web.config* 文件中），如下所示：

        <trace enabled="true" writeToDiagnosticsTrace="true" localOnly="false" mostRecent="true" pageOutput="false" />

    然而，出于安全考虑，一般不建议在生产站点中启用 `trace.axd`，在随后的内容中您将了解到一种在 Azure 网站中更为便捷地读取跟踪日志的方法。

### 在 Azure 中查看跟踪输出

1.  在“解决方案资源管理器”中，右键单击该 Web 项目并单击“发布”。

2.  在“发布 Web”对话框中，单击“发布”。。

    在 Visual Studio 发布您的更新后，其将打开一个到您主页的浏览器窗口（假设您没有清除“连接”选项卡上的“目的地 URL”）。

3.  在“服务器资源管理器”中，右键单击您的网站并选择“在输出窗口中查看流式传输日志”。

    ![上下文菜单中的查看流式传输日志][上下文菜单中的查看流式传输日志]

    “输出”窗口显示您已连接至日志流式传输服务，每过一分钟没有要显示的日志，就添加一个通知行。

    ![上下文菜单中的查看流式传输日志][4]

4.  在显示应用程序主页的浏览器窗口，单击“联系人”。

    几秒钟内，添加至 `Contact` 方法的错误级跟踪的输出显示在“输出”窗口中。

    ![输出窗口中的错误跟踪][输出窗口中的错误跟踪]

    Visual Studio 仅显示错误级跟踪，因为这是启用日志监视服务时的默认设置。创建新的 Azure 网站时，默认禁用所有日志记录，正如之前打开站点设置页面时所见：

    ![应用程序日志记录关闭][应用程序日志记录关闭]

    然而，如果选择了“在输出窗口中查看流式传输日志”，Visual Studio 会自动将“应用程序日志记录（文件系统）”更改为“错误”，这意味着将报告错误级日志。为了查看所有跟踪日志，可将该设置更改为“详细”。如果选择的严重级别低于错误，所有更高严重级别的日志也将得到报告。因此，如果选择“详细”，还可查看信息、警告以及错误日志。

5.  在“服务器资源管理器”中，右键单击网站，然后如之前所做单击“查看设置”。

6.  将“应用程序日志记录（文件系统）”更改为“详细”，然后单击“保存”。

    ![将跟踪级别设置为详细][将跟踪级别设置为详细]

7.  在现在显示的是“联系人”页面的浏览器窗口中，单击“主页”，然后依次单击“关于”和“联系人” 。

    几秒钟内,“输出”窗口将显示您的所有跟踪输出。

    ![详细的跟踪输出][详细的跟踪输出]

    在本节中，您通过使用 Azure 网站设置对日志记录进行了启用和禁用。您还可以通过修改 Web.config 文件启用和禁用跟踪侦听器。然而，修改 Web.config 文件会导致应用域回收，而通过网站启用日志记录则不会出此情况。如果问题重现需要花费较长时间或呈间歇性，回收应用域可能会“修正”该问题并强迫您一直等待直至其再次出现。在 Azure 中启用诊断功能不会如此，因此您可以立即开始捕获错误信息。

### 输出窗口特性

“输出”窗口的“Azure 日志”选项卡上有若干按钮和一个文本框：

![日志选项卡按钮][日志选项卡按钮]

这些控件执行以下功能：

-   清理“输出”窗口。
-   启用或禁用自动换行。
-   启动或停止监视日志。
-   指定要监视的日志。
-   下载日志。
-   基于搜索字符串或正则表达式筛选日志。
-   关闭“输出”窗口。

如果您输入一个搜索字符串或正则表达式，Visual Studio 将在客户端删选日志记录信息。这意味着您可以在日志显示在“输出”窗口后输入条件，并可更改筛选条件而不必重新生成日志。

## <a name="webserverlogs"></a><span class="short-header">Web 服务器日志</span>查看 Web 服务器日志

Web 服务器日志记录了站点上所有的 HTTP 活动。若要在“输出”窗口中查看这些日志，必须在站点上启用该日志并告知 Visual Studio 您希望对其进行监视。

1.  在通过“服务器资源管理器”打开的“Azure 网站配置”选项卡上，将“Web 服务器日志记录”的状态更改为“开启”，然后单击“保存”。

    ![启用 Web 服务器日志记录][启用 Web 服务器日志记录]

2.  在“输出”窗口中，单击“指定要监视的 Azure 日志”按钮。

    ![指定要监视的 Azure 日志。][指定要监视的 Azure 日志。]

3.  在“Azure 日志记录选项”对话框中，选择“Web 服务器日志”，然后单击“确定”。

    ![监视 Web 服务器日志][监视 Web 服务器日志]

4.  在显示网站的浏览器窗口，依次单击“主页”、“关于”以及“联系人”。

    应用程序日志通常第一个显示，其后是 Web 服务器日志。您可能必须得等待一会以便日志显示。

    ![输出窗口中的 Web 服务器日志][输出窗口中的 Web 服务器日志]

默认情况下，通过使用 Visual Studio 第一次启用 Web 服务器日志时，Azure 会将日志写入文件系统。或者，您可以使用管理门户指定将 Web 服务器日志写入存储帐户的 Blob 容器。有关更多信息，请参见[如何配置 Web 站点][如何配置网站]中的**站点诊断**一节。

如果使用管理门户将 Web 服务器日志记录写入 Azure 存储帐户，之后在 Visual Studio 中禁用日志记录，那么在 Visual Studio 中重新启用日志记录时，存储帐户设置将还原。

## <a name="detailederrorlogs"></a><span class="short-header">错误日志</span>查看详细的错误消息日志

详细的错误日志提供了有关导致错误响应代码（400或更高）的 HTTP 请求的一些额外信息。若要在“输出”窗口中查看这些日志，必须在站点上启用该日志并告知 Visual Studio 您希望对其进行监视。

1.  在通过“服务器资源管理器”打开的“Azure 网站配置”选项卡上，将“详细的错误消息”的状态更改为“开启”，然后单击“保存”。

    ![启用详细的错误消息][启用详细的错误消息]

2.  在“输出”窗口中，单击“指定要监视的 Azure 日志”按钮。

3.  在“Azure 日志记录选项”对话框中单击“所有日志”，然后单击“确定”。

    ![监视所有日志][监视所有日志]

4.  在浏览器的地址栏中，向 URL 添加一个额外字符以导致 404 错误（例如 `http://localhost:53370/Home/Contactx`），然后按 Enter。

    几秒后，详细的错误日志显示在 Visual Studio 的“输出”窗口中。

    ![输出窗口中详细的错误日志][输出窗口中详细的错误日志]

    Control + 单击该链接可在浏览器中看到排好格式的日志输出。

    ![浏览器窗口中详细的错误日志][浏览器窗口中详细的错误日志]

## <a name="downloadlogs"></a><span class="short-header">下载日志</span>下载文件系统日志

任何可在“输出”窗口中监视的日志都可作为 *.zip* 文件进行下载。

1.  在“输出”窗口中单击“下载流式传输日志”。

    ![日志选项卡按钮][5]

    “文件资源管理器”打开，其中显示 *Downloads* 文件夹中已下载文件处于选定状态。

    ![下载的文件][下载的文件]

2.  提取该 *.zip* 文件，您将看到以下文件夹结构：

    ![下载的文件][6]

    -   应用程序跟踪日志位于 *LogFiles\\Application* 文件夹的 *.txt* 文件中。
    -   Web 服务器位于 *LogFiles\\http\\RawLogs* 文件夹的 *.log* 文件中。可以使用诸如 [Log Parser][Log Parser] 之类的工具查看并处理这些文件。
    -   详细的错误消息日志位于 *LogFiles\\DetailedErrors* 文件夹的 *.html* 文件中。

    （*deployments* 文件夹内是由源代码管理发布创建的文件；其中没有任何有关 Visual Studio 发布的内容。*Git* 文件夹内是与源代码管理发布以及日志文件流式传输服务相关的跟踪。）

## <a name="storagelogs"></a><span class="short-header">存储日志</span>查看存储日志

应用程序跟踪日志也可发送至 Azure 存储帐户，可通过 Visual Studio 进行查看。若要如此，需要创建一个存储帐户，在管理门户中启用存储日志，然后在“Azure 网站”窗口的“日志”选项卡中对其进行查看。

您可以将日志发送到以下所有目的地或其中之一：

-   文件系统。
-   存储帐户表。
-   存储帐户 Blob。

您可以为每个目的地指定不同的严重级别。

表易于在线查看日志详细信息并且其支持流式传输；可在表中查询日志并在新日志创建的同时进行查看。Blob 易于下载文件中的日志并使用 HDInsight 对其进行分析，因为 HDInsight 知晓如何与 Blob 存储协同运作。有关更多信息请参见[数据存储选项（通过 Azure 构建实际云应用程序）][数据存储选项（通过 Azure 构建实际云应用程序）]中的 **Hadoop 和 MapReduce**。

当前您的文件系统日志设置为“详细”；从设置信息级日志到存储帐户表，以下将分别向您介绍。信息级意味着所有通过调用 `Trace.TraceInformation`、`Trace.TraceWarning` 以及 `Trace.TraceError` 创建的日志都将显示，但调用 `Trace.WriteLine` 创建的日志不包括在内。

与文件系统相比，存储帐户可为日志提供更大的存储空间以及更长的保留期。将应用程序跟踪日志发送到存储的另一个优势是您可以从每个日志获得一些无法从文件系统日志获得的其他信息。

1.  在“服务器资源管理器”中右键单击“在管理门户中打开”。

2.  在管理门户中，单击“存储”选项卡，然后单击“创建存储帐户”。

    ![创建存储帐户][创建存储帐户]

3.  为存储帐户输入唯一的 URL，然后单击“创建存储帐户”。

    ![输入一个 URL][输入一个 URL]

4.  在 Visual Studio 的“Azure 网站”窗口中，单击“日志”选项卡，然后单击“配置日志记录”。

    ![下载的文件][7]

    这将在管理门户中为您的网站打开“配置”选项卡。另一种访问此选项卡的方法是单击“网站”选项卡，单击您的网站，然后单击“配置”选项卡。

5.  在管理门户的“配置”选项卡中，向下滚动至应用程序诊断部分，然后将“应用程序日志记录（表存储）”的状态更改为“开启”。

6.  将“日志记录级别”更改为“信息”。

7.  单击“管理表存储”。

    ![单击管理表存储。][单击管理表存储。]

    在“针对应用程序诊断管理表存储”框中可选择您的存储帐户（如果您有多个此类帐户）。您可以创建一个新表或使用现有的表。

    ![管理表存储][管理表存储]

8.  在“对应用程序诊断管理表存储”框中，单击复选标记关闭该对话框。

9.  在管理门户的“配置”选项卡中，单击“保存”。

10. 在显示应用程序网站的浏览器窗口，依次单击“主页”、“关于”以及“联系人”。

    浏览这些网页而生成的日志记录信息将写入存储帐户。

11. 在 Visual Studio 中的“Azure 网站”窗口的“日志”选项卡上，单击“诊断摘要”下的“刷新”。

    ![单击刷新][单击刷新]

    “诊断摘要”部分默认显示过去 15 分钟的日志。可以更改此时间以查看更多日志。

    （如果收到“表未找到”错误，请验证是否在启用“应用程序日志记录（存储）”并随后单击“保存”之后已浏览至执行跟踪的页面。）

    ![存储日志][存储日志]

    请注意，在此视图中，可以看到每个日志的“进程 ID”和“线程 ID”，这在文件系统日志中是无法看到的。通过直接查看 Azure 存储表可看到额外字段。

12. 单击“查看所有应用程序日志”。

    跟踪日志表将显示在 Azure 存储表查看器中。

    （如果收到“序列未包含任何元素”错误，请打开“服务器资源管理器”，展开 Azure 节点下您的存储帐户节点，然后右键单击“表”并单击“刷新”。）

    ![服务器资源管理器中的跟踪表][服务器资源管理器中的跟踪表]

    ![表视图中的存储日志][表视图中的存储日志]

    该视图将显示任何其他视图都没提供的额外字段。该视图还支持您使用特殊的 Query Builder UI 构建查询以筛选日志。有关更多信息，请参见[通过服务器资源管理器浏览存储资源][通过服务器资源管理器浏览存储资源]中的“使用表资源 — 筛选实体”。

13. 要查看单个行的详细信息，右键单击其中一行，然后单击“编辑”。

    ![服务器资源管理器中的跟踪表][8]

## <a name="failedrequestlogs"></a><span class="short-header">失败请求日志</span>查看失败请求跟踪日志

在出现诸如 URL 重写或身份验证问题之类的情况下，需要详细了解 IIS 是如何处理 HTTP 请求时可求助于失败请求跟踪日志。

Azure 网站使用 IIS 7.0 及更高版本中提供的相同的失败请求跟踪功能。IIS 设置经过配置可记录指定错误，但您无法访问该设置。启用失败请求跟踪后，所有错误都将纳入捕获范围内。

使用 Visual Studio 可启用失败请求跟踪，但却无法在 Visual Studio 中对其进行查看。这些日志是 XML 文件。这些流式传输日志服务只监视认为在纯文本模式下可读的文件:*.txt*、*.html* 以及 *.log* 文件。

您可以通过 FTP 直接在浏览器中查看失败请求跟踪日志，或使用 FTP 工具将其下载到本地计算机后进行本地查看。在本节中，您将直接在浏览器中查看这些日志。

1.  在从“服务器资源管理器”打开的“Azure 网站”窗口的“配置”选项卡中，将“失败请求跟踪”的状态更改为“开启”，然后单击“保存”。

    ![启用失败请求跟踪][启用失败请求跟踪]

2.  在显示该网站的浏览器窗口的地址栏中，向 URL 添加一个额外字符，单击 Enter 将引发 404 错误。

    这将导致创建失败请求跟踪日志，以下步骤将向您介绍如何查看或下载该日志。

3.  在 Visual Studio 中，在“Azure 网站”窗口的“”选项卡中, 单击“在管理门户中打开”。

4.  在管理门户中，单击“仪表板”，然后单击“速览”部分中的“重置部署凭据”。

    ![在仪表板中重置 FTP 凭据链接][在仪表板中重置 FTP 凭据链接]

5.  输入新的用户名和密码。

    ![新的 FTP 用户名和密码][新的 FTP 用户名和密码]

6.  在管理门户“仪表板”选项卡中按 F5 刷新该页面，然后向下滚至可以看到“部署/FTP 用户”的地方。请注意，该用户名会将站点名称作为其前缀。**当您登录时，必须使用以站点名为前缀的完整用户名（如此处所示）。**

7.  在新的浏览器窗口，转至显示在网站管理门户页面“仪表板”选项卡中“FTP 主机名”下的 URL。“FTP 主机名”的位置靠近“速览”部分内的“部署/FTP 用户”。

8.  使用之前创建的 FTP 凭据（包括作为用户名前缀的站点名）登录。

    浏览器将显示该站点的根文件夹。

9.  打开 *LogFiles* 文件夹。

    ![打开 LogFiles 文件夹][打开 LogFiles 文件夹]

10. 打开名为 W3SVC 加数值的文件夹。

    ![打开 W3SVC 文件夹][打开 W3SVC 文件夹]

    该文件夹包含启用失败请求跟踪之后记录在案的任何错误的 XML 文件，以及一个可供浏览器格式化 XML 的 XSL 文件。

    ![W3SVC 文件夹][W3SVC 文件夹]

11. 单击希望为之查看跟踪信息的失败请求的 XML 文件。

    下图显示了示例错误的部分跟踪信息。

    ![浏览器中的失败请求跟踪][浏览器中的失败请求跟踪]

## <a name="nextsteps"></a><span class="short-header">后续步骤</span>后续步骤

以上部分介绍了如何通过 Visual Studio 轻松查看由 Azure 网站创建的日志。您可能希望更详细地了解对 Azure 网站进行故障排除、在 ASP.NET 应用程序中进行跟踪以及分析 Web 服务器日志。

### Azure 网站故障排除

有关排除 Azure 网站 (WAWS) 故障的更多信息，请参见以下资源：

-   [Azure 中的故障排除][Azure 中的故障排除]
     介绍基本知识的白皮书，其中对 WAWS 略有涉及。
-   [网站故障排除][网站故障排除]
     专注于 WAWS 的介绍性内容。
-   [针对 Azure 网站启用诊断日志记录][针对 Azure 网站启用诊断日志记录]
     与本教程所述内容基本一致，但着重于如何不借助于 Visual Studio 获取诊断日志。
-   [如何监视网站][如何监视网站]
     [配置诊断并下载日志][配置诊断并下载日志]部分提供了并未包含在故常排除文档内的重要信息。

若要针对特定故障排除问题寻求帮助，可在以下论坛之一开启话题讨论：

-   [ASP.NET 站点上的 Azure 论坛][ASP.NET 站点上的 Azure 论坛]。
-   [MSDN 上的 Azure 论坛][MSDN 上的 Azure 论坛]。
-   [StackOverflow.com][StackOverflow.com]。

### 在 Visual Studio 中进行调试

有关如何在 Visual Studio 中使用调试模式，请参见[在 Visual Studio 中进行调试][在 Visual Studio 中进行调试] MSDN 主题和[使用 Visual Studio 2010 进行调试的提示][使用 Visual Studio 2010 进行调试的提示]。

如果您的网站使用 Azure Web API 或移动服务后端并且需要进行调试，请参见[在 Visual Studio 中调试 .NET 后端][在 Visual Studio 中调试 .NET 后端]。

### 在 ASP.NET 应用程序中进行跟踪

Internet 上对于 ASP.NET 跟踪没有全面且最新的介绍。最佳做法是以针对 Web 窗体编写的老旧介绍性材料为主（因为 MVC 彼时并不存在），以专注于特定问题的新兴博客文章为辅。以下资源或许可以助您一臂之力：

-   [监视和遥测（通过 Azure 构建实际云应用程序）][监视和遥测（通过 Azure 构建实际云应用程序）]。
    电子书籍的章节就在 Azure 云应用程序中进行跟踪给出了建议。
-   [ASP.NET 跟踪][ASP.NET 跟踪]
     内容老旧，但不失为这一主题的优秀入门级资源。
-   [跟踪侦听器][跟踪侦听器]
     有关跟踪侦听器的信息，但未提及[WebPageTraceListener][WebPageTraceListener]。
-   [演练：集成 ASP.NET 跟踪与 System.Diagnostics 跟踪][演练：集成 ASP.NET 跟踪与 System.Diagnostics 跟踪]
     此工具已不时兴，但包括一些介绍性文章未涵盖的额外信息。
-   [在 ASP.NET MVC Razor 视图中进行跟踪][在 ASP.NET MVC Razor 视图中进行跟踪]
     除了在 Razor 视图中进行跟踪，文章还介绍了如何创建错误筛选器以便在 MVC 应用程序中记录所有未经处理的异常。有关如何在 Web 窗体应用程序中记录所有未经处理的异常，请参见 MSDN 上[错误处理程序的完整示例][错误处理程序的完整示例] 中的 Global.asax 示例。在 MVC 或 Web 窗体中，如果希望记录特定异常但想让默认框架对其进行处理，可捕获并重新引发异常，如下例所示：

        try
        {
           // Your code that might cause an exception to be thrown.
        }
        catch (Exception ex)
        {
            Trace.TraceError("Exception: " + ex.ToString());
            throw;
        } 

-   [从 Azure 命令行流式传输诊断跟踪日志记录（加上 Glimpse！）][从 Azure 命令行流式传输诊断跟踪日志记录（加上 Glimpse！）]
     如何使用命令行实现本教程中通过 Visual Studio 完成的任务。[Glimpse][Glimpse] 是一个用于调试 ASP.NET 应用程序的工具。
-   [使用 Azure 网站日志记录和诊断 — 与 David Ebbo 协作完成][使用 Azure 网站日志记录和诊断 — 与 David Ebbo 协作完成]和 [从 Azure 网站流式传输日志 — 与 David Ebbo 协作完成][从 Azure 网站流式传输日志 — 与 David Ebbo 协作完成]
     由 Scott Hanselman 和 David Ebbo 提供的视频。

对于错误日志记录，若不想编写自己的跟踪代码，可以使用开源日志记录框架，如 [ELMAH][ELMAH]。有关更多信息，请参见 [Scott Hanselman 有关 ELMAH 的博客文章][Scott Hanselman 有关 ELMAH 的博客文章]。

此外，请注意，如果希望从 Azure 获得流式传输日志，则不必使用 ASP.NET 或 System.Diagnostics 跟踪。Azure 网站流式传输日志服务会将其在 *LogFiles* 文件中找到的所有 *.txt*、*.html* 或 *.log* 文件进行流式传输。因此，您可以创建自己的日志记录系统以写入网站的文件系统，您的文件将自动进行流式传输和下载。您所要做的就是编写在 *d:\\home\\logfiles* 文件夹中创建文件的应用程序代码。

### 分析 Web 服务器日志

有关分析 Web 服务器日志的更多信息，请参见以下资源：

-   [LogParser][LogParser]
     用于查看 Web 服务器日志（*.log* 文件）中的数据的工具。
-   [使用 LogParser 解决 IIS 性能问题或应用程序错误][使用 LogParser 解决 IIS 性能问题或应用程序错误]
     可用于分析 Web 服务器日志的 Log Parser 工具介绍。
-   [Robert McMurray 有关 LogParser 使用的博客文章][Robert McMurray 有关 LogParser 使用的博客文章]
-   [IIS 7.0、IIS 7.5 以及 IIS 8.0 中的 HTTP 状态代码][IIS 7.0、IIS 7.5 以及 IIS 8.0 中的 HTTP 状态代码]

### 分析失败请求跟踪日志

Microsoft TechNet 网站包含的[使用失败请求跟踪][使用失败请求跟踪]部分对于了解如何使用这些日志非常有用。然而，该文档主要着重于在 IIS 中配置失败请求跟踪，并不适用于 Azure 网站。

### 调试云服务

如果您希望调试 Azure 云服务而不是网站，请参见[调试云服务][调试云服务]。

  [运行在调试模式下]: http://www.visualstudio.com/zh-cn/get-started/debug-your-app-vs.aspx
  [IntelliTrace]: http://msdn.microsoft.com/library/vstudio/dd264915.aspx
  [先决条件]: #prerequisites
  [站点配置和管理]: #sitemanagement
  [远程视图]: #remoteview
  [远程调试]: #remotedebug
  [诊断日志概述]: #logsoverview
  [创建并查看应用程序跟踪日志]: #apptracelogs
  [查看 Web 服务器日志]: #webserverlogs
  [查看详细的错误消息日志]: #detailederrorlogs
  [下载文件系统日志]: #downloadlogs
  [查看存储日志]: #storagelogs
  [查看失败请求日志]: #failedrequestlogs
  [后续步骤]: #nextsteps
  [Azure 和 ASP.NET 入门]: /zh-cn/develop/net/tutorials/get-started/
  [管理帐户、订阅和管理角色]: http://go.microsoft.com/fwlink/?LinkId=324796#BKMK_AccountVCert
  [在服务器资源管理器中查看设置]: ./media/web-sites-dotnet-troubleshoot-visual-studio/tws-viewsettings.png
  [Azure 网站窗口]: ./media/web-sites-dotnet-troubleshoot-visual-studio/tws-configtab.png
  [Azure 网站：应用程序字符串和连接字符串的工作原理]: http://blogs.msdn.com/b/windowsazure/archive/2013/07/17/windows-azure-web-sites-how-application-strings-and-connection-strings-work.aspx
  [如何配置网站]: /zh-cn/manage/services/web-sites/how-to-configure-websites/#howtochangeconfig
  [毫无用处的错误页面]: ./media/web-sites-dotnet-troubleshoot-visual-studio/genericerror.png
  [1]: ./media/web-sites-dotnet-troubleshoot-visual-studio/genericerror1.png
  [2]: ./media/web-sites-dotnet-troubleshoot-visual-studio/genericerror2.png
  [Web.config 转换]: http://www.asp.net/mvc/tutorials/deployment/visual-studio-web-deployment/web-config-transformations
  [文件和日志文件]: ./media/web-sites-dotnet-troubleshoot-visual-studio/fileandlogfiles.png
  [打开 Web.config]: ./media/web-sites-dotnet-troubleshoot-visual-studio/webconfig.png
  [编辑 Web.config]: ./media/web-sites-dotnet-troubleshoot-visual-studio/webconfigedit.png
  [详细的错误消息]: ./media/web-sites-dotnet-troubleshoot-visual-studio/detailederror.png
  [在调试模式下发布]: ./media/web-sites-dotnet-troubleshoot-visual-studio/tws-publishdebug.png
  [附加调试程序]: ./media/web-sites-dotnet-troubleshoot-visual-studio/tws-attachdebugger.png
  [在管理门户中设置远程调试]: ./media/web-sites-dotnet-troubleshoot-visual-studio/tws-debuginportal.png
  [在运行于 Azure 中的调试模式下查看变量]: ./media/web-sites-dotnet-troubleshoot-visual-studio/tws-debugviewinwa.png
  [显示新值的关于页面]: ./media/web-sites-dotnet-troubleshoot-visual-studio/tws-debugchangeinwa.png
  [Azure 定价]: /zh-cn/pricing/calculator/
  [限制为仅逐行调试我的代码]: http://msdn.microsoft.com/zh-cn/library/vstudio/y740d9d3.aspx#BKMK_Restrict_stepping_to_Just_My_Code
  [WebJobs]: http://www.asp.net/aspnet/overview/developing-apps-with-windows-azure/getting-started-with-windows-azure-webjobs
  [在 Visual Studio 中进行调试]: http://msdn.microsoft.com/zh-cn/library/vstudio/sc65sadd.aspx
  [System.Diagnostics.Trace]: http://msdn.microsoft.com/zh-cn/library/system.diagnostics.trace.aspx
  [在调试窗口中进行跟踪]: ./media/web-sites-dotnet-troubleshoot-visual-studio/tws-debugtracing.png
  [跟踪元素]: http://msdn.microsoft.com/zh-cn/library/vstudio/6915t83k(v=vs.100).aspx
  [trace.axd]: ./media/web-sites-dotnet-troubleshoot-visual-studio/tws-traceaxd1.png
  [3]: ./media/web-sites-dotnet-troubleshoot-visual-studio/tws-traceaxd2.png
  [上下文菜单中的查看流式传输日志]: ./media/web-sites-dotnet-troubleshoot-visual-studio/tws-viewlogsmenu.png
  [4]: ./media/web-sites-dotnet-troubleshoot-visual-studio/tws-nologsyet.png
  [输出窗口中的错误跟踪]: ./media/web-sites-dotnet-troubleshoot-visual-studio/tws-errortrace.png
  [应用程序日志记录关闭]: ./media/web-sites-dotnet-troubleshoot-visual-studio/tws-apploggingoff.png
  [将跟踪级别设置为详细]: ./media/web-sites-dotnet-troubleshoot-visual-studio/tws-applogverbose.png
  [详细的跟踪输出]: ./media/web-sites-dotnet-troubleshoot-visual-studio/tws-verbosetraces.png
  [日志选项卡按钮]: ./media/web-sites-dotnet-troubleshoot-visual-studio/tws-icons.png
  [启用 Web 服务器日志记录]: ./media/web-sites-dotnet-troubleshoot-visual-studio/tws-webserverloggingon.png
  [指定要监视的 Azure 日志。]: ./media/web-sites-dotnet-troubleshoot-visual-studio/tws-specifylogs.png
  [监视 Web 服务器日志]: ./media/web-sites-dotnet-troubleshoot-visual-studio/tws-monitorwslogson.png
  [输出窗口中的 Web 服务器日志]: ./media/web-sites-dotnet-troubleshoot-visual-studio/tws-wslogs.png
  [启用详细的错误消息]: ./media/web-sites-dotnet-troubleshoot-visual-studio/tws-detailedlogson.png
  [监视所有日志]: ./media/web-sites-dotnet-troubleshoot-visual-studio/tws-monitorall.png
  [输出窗口中详细的错误日志]: ./media/web-sites-dotnet-troubleshoot-visual-studio/tws-detailederrorlog.png
  [浏览器窗口中详细的错误日志]: ./media/web-sites-dotnet-troubleshoot-visual-studio/tws-detailederrorloginbrowser.png
  [5]: ./media/web-sites-dotnet-troubleshoot-visual-studio/tws-downloadicon.png
  [下载的文件]: ./media/web-sites-dotnet-troubleshoot-visual-studio/tws-downloadedfile.png
  [6]: ./media/web-sites-dotnet-troubleshoot-visual-studio/tws-logfilefolders.png
  [Log Parser]: http://www.microsoft.com/zh-cn/download/details.aspx?displaylang=en&id=24659
  [数据存储选项（通过 Azure 构建实际云应用程序）]: http://www.asp.net/aspnet/overview/developing-apps-with-windows-azure/building-real-world-cloud-apps-with-windows-azure/data-storage-options
  [创建存储帐户]: ./media/web-sites-dotnet-troubleshoot-visual-studio/tws-createstorage.png
  [输入一个 URL]: ./media/web-sites-dotnet-troubleshoot-visual-studio/tws-storageurl.png
  [7]: ./media/web-sites-dotnet-troubleshoot-visual-studio/tws-configlogging.png
  [单击管理表存储。]: ./media/web-sites-dotnet-troubleshoot-visual-studio/tws-stgsettingsmgmtportal.png
  [管理表存储]: ./media/web-sites-dotnet-troubleshoot-visual-studio/tws-choosestorageacct.png
  [单击刷新]: ./media/web-sites-dotnet-troubleshoot-visual-studio/tws-refreshstorage.png
  [存储日志]: ./media/web-sites-dotnet-troubleshoot-visual-studio/tws-storagelogs.png
  [服务器资源管理器中的跟踪表]: ./media/web-sites-dotnet-troubleshoot-visual-studio/tws-tracetableinse.png
  [表视图中的存储日志]: ./media/web-sites-dotnet-troubleshoot-visual-studio/tws-tracelogtableview.png
  [通过服务器资源管理器浏览存储资源]: http://msdn.microsoft.com/zh-cn/library/azure/ff683677.aspx
  [8]: ./media/web-sites-dotnet-troubleshoot-visual-studio/tws-tracetablerow.png
  [启用失败请求跟踪]: ./media/web-sites-dotnet-troubleshoot-visual-studio/tws-failedrequeston.png
  [在仪表板中重置 FTP 凭据链接]: ./media/web-sites-dotnet-troubleshoot-visual-studio/tws-resetftpcredentials.png
  [新的 FTP 用户名和密码]: ./media/web-sites-dotnet-troubleshoot-visual-studio/tws-enterftpcredentials.png
  [打开 LogFiles 文件夹]: ./media/web-sites-dotnet-troubleshoot-visual-studio/tws-logfilesfolder.png
  [打开 W3SVC 文件夹]: ./media/web-sites-dotnet-troubleshoot-visual-studio/tws-w3svcfolder.png
  [W3SVC 文件夹]: ./media/web-sites-dotnet-troubleshoot-visual-studio/tws-w3svcfoldercontents.png
  [浏览器中的失败请求跟踪]: ./media/web-sites-dotnet-troubleshoot-visual-studio/tws-failedrequestinbrowser.png
  [Azure 中的故障排除]: /zh-cn/develop/net/best-practices/troubleshooting/
  [网站故障排除]: /zh-cn/develop/net/best-practices/troubleshooting-web-sites/
  [针对 Azure 网站启用诊断日志记录]: /zh-cn/develop/net/common-tasks/diagnostics-logging-and-instrumentation/
  [如何监视网站]: /zh-cn/manage/services/web-sites/how-to-monitor-websites/
  [配置诊断并下载日志]: /zh-cn/manage/services/web-sites/how-to-monitor-websites/#howtoconfigdiagnostics
  [ASP.NET 站点上的 Azure 论坛]: http://forums.asp.net/1247.aspx/1?Azure+and+ASP+NET
  [MSDN 上的 Azure 论坛]: http://social.msdn.microsoft.com/Forums/zh-cn/home?forum=windowsazurezhchs
  [StackOverflow.com]: http://www.stackoverflow.com
  [使用 Visual Studio 2010 进行调试的提示]: http://weblogs.asp.net/scottgu/archive/2010/08/18/debugging-tips-with-visual-studio-2010.aspx
  [在 Visual Studio 中调试 .NET 后端]: http://blogs.msdn.com/b/azuremobile/archive/2014/03/14/debugging-net-backend-in-visual-studio.aspx
  [监视和遥测（通过 Azure 构建实际云应用程序）]: http://www.asp.net/aspnet/overview/developing-apps-with-windows-azure/building-real-world-cloud-apps-with-windows-azure/monitoring-and-telemetry
  [ASP.NET 跟踪]: http://msdn.microsoft.com/zh-cn/library/ms972204.aspx
  [跟踪侦听器]: http://msdn.microsoft.com/zh-cn/library/4y5y10s7.aspx
  [WebPageTraceListener]: http://msdn.microsoft.com/zh-cn/library/system.web.webpagetracelistener.aspx
  [演练：集成 ASP.NET 跟踪与 System.Diagnostics 跟踪]: http://msdn.microsoft.com/zh-cn/library/b0ectfxd.aspx
  [在 ASP.NET MVC Razor 视图中进行跟踪]: http://blogs.msdn.com/b/webdev/archive/2013/07/16/tracing-in-asp-net-mvc-razor-views.aspx
  [错误处理程序的完整示例]: http://msdn.microsoft.com/zh-cn/library/bb397417.aspx
  [从 Azure 命令行流式传输诊断跟踪日志记录（加上 Glimpse！）]: http://www.hanselman.com/blog/StreamingDiagnosticsTraceLoggingFromTheAzureCommandLinePlusGlimpse.aspx
  [Glimpse]: http://www.hanselman.com/blog/IfYoureNotUsingGlimpseWithASPNETForDebuggingAndProfilingYoureMissingOut.aspx
  [使用 Azure 网站日志记录和诊断 — 与 David Ebbo 协作完成]: http://azure.microsoft.com/zh-cn/documentation/videos/azure-web-site-logging-and-diagnostics/
  [从 Azure 网站流式传输日志 — 与 David Ebbo 协作完成]: http://azure.microsoft.com/zh-cn/documentation/videos/log-streaming-with-azure-web-sites/
  [ELMAH]: http://nuget.org/packages/elmah/
  [Scott Hanselman 有关 ELMAH 的博客文章]: http://www.hanselman.com/blog/NuGetPackageOfTheWeek7ELMAHErrorLoggingModulesAndHandlersWithSQLServerCompact.aspx
  [LogParser]: http://www.microsoft.com/zh-cn/download/details.aspx?id=24659
  [使用 LogParser 解决 IIS 性能问题或应用程序错误]: http://www.iis.net/learn/troubleshoot/performance-issues/troubleshooting-iis-performance-issues-or-application-errors-using-logparser
  [Robert McMurray 有关 LogParser 使用的博客文章]: http://blogs.msdn.com/b/robert_mcmurray/archive/tags/logparser/
  [IIS 7.0、IIS 7.5 以及 IIS 8.0 中的 HTTP 状态代码]: http://support.microsoft.com/kb/943891
  [使用失败请求跟踪]: http://www.iis.net/learn/troubleshoot/using-failed-request-tracing
  [调试云服务]: http://msdn.microsoft.com/zh-cn/library/azure/ee405479.aspx
