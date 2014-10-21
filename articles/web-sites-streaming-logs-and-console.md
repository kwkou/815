<properties pageTitle="Streaming logs and console" description="Streaming logs and console overview" title="Streaming logs and console" authors="adamab" />

## 流式传输日志和控制台

### 流式传输日志

Microsoft Azure 预览版门户提供了集成的流式传输日志查看器，可用于实时查看来自你的网站的跟踪事件。

设置此功能需要几个简单的步骤：

-   在你的代码中编写跟踪
-   在 Azure 预览版门户中启用应用程序诊断
-   在网站叶片上单击流式传输日志部分

### 如何在你的代码中编写跟踪

在你的代码中编写跟踪是很容易的。在 C# 中，可以很简单地编写这样的代码：

    Trace.TraceInformation("My trace statement");

    Trace.TraceWarning("My warning statement");

    Trace.TraceError("My error statement");

Trace 类驻留在 System.Diagnostics 命名空间中。

在 node.js 应用中，你可以编写此代码以获得同样的结果：

    console.log("My trace statement").

### 如何启用和查看流式传输日志

诊断是基于每个网站来启用的。从[门户][门户]中，单击左菜单栏上的“浏览”按钮，然后单击“网站”以获取你所有网站的列表。

![][]

单击要配置的网站的名称。然后单击名为“诊断日志”的部分并将“应用程序日志记录(Filesystem)”开关设置为“开”。然后会显示“级别”选项，用于更改要捕获的跟踪的严重程度级别。如果你只是尝试熟悉该功能，此设置应设为“详细”，，因为这将确保你所有的跟踪语句都会被记录到日志中。

单击叶片顶部的“保存”，然后就可以开始查看日志了。

若要在门户内查看流式传输日志，请单击网站叶片上的“流式传输日志”部分。如果你的网站主动写入跟踪语句，则会以近乎实时的方式在生成的窗口中看到它们。

![][1]

## 控制台

Azure 预览版门户提供了对你的网站环境的控制台访问。你可以浏览网站的文件系统并运行 Powershell/cmd 脚本。在执行控制台命令时，对你绑定的是与运行中的网站代码相同的权限集。你将无法访问受保护的目录，也无法运行要求提升的权限的脚本。

若要使用控制台，请按上面部分的说明浏览到网站。单击“控制台”部分，控制台将会打开。

![][2]

若要熟悉控制台，请尝试这样一些基本命令：

    dir

    cd

<!-- Images. -->

  [门户]: https://portal.azure.com
  []: ./media/web-sites-streaming-logs-and-console/browse-sites.png
  [1]: ./media/web-sites-streaming-logs-and-console/streaming-logs.png
  [2]: ./media/web-sites-streaming-logs-and-console/console.png
