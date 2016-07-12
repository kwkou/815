<properties 
	pageTitle="使用 DASH.js 在 HTML5 应用程序中嵌入 MPEG-DASH 自适应流式处理视频" 
	description="本主题演示如何使用 DASH.js 在 HTML5 应用程序中嵌入 MPEG-DASH 自适应流式处理视频。" 
	authors="Juliako" 
	manager="dwrede" 
	editor="" 
	services="media-services" 
	documentationCenter=""/>

<tags
	ms.service="media-services"
 	ms.date="04/18/2016"  
	wacn.date="06/20/2016"/>


#使用 DASH.js 在 HTML5 应用程序中嵌入 MPEG-DASH 自适应流式处理视频

##概述

MPEG-DASH 是视频内容自适应流式处理的 ISO 标准，为那些希望传送高质量自适应视频流式处理输出的用户提供了显著的好处。使用 MPEG-DASH，当在网络阻塞时，视频流将自动降低到较低清晰度。这将减少在播放器下载下几秒钟要播放内容（又称缓冲）时观众看到“暂停”视频的可能性。当网络拥塞减少时，视频播放器将转而恢复到较高质量的流。这种适应所需带宽的能力也会导致视频开始的速度更快。这意味着可以在快速下载较低质量段播放最初的几秒钟，然后在已缓冲足够内容后提升到更高质量。

Dash.js 是用 JavaScript 编写的开源 MPEG-DASH 视频播放器。其目标是提供可以在需要视频播放的应用程序中自由重用的功能强大的跨平台播放器。它在支持 W3C 媒体源扩展 (MSE) 的任何浏览器（目前为 Chrome、Microsoft Edge 和 IE11，其他浏览器已指示有意支持 MSE）中提供 MPEG-DASH 播放。有关 DASH.js、js 的详细信息，请参阅 GitHub dash.js 存储库。


##创建基于浏览器的流式处理视频播放器

若要使用所需控件（如播放、暂停、后退等）创建显示视频播放器的简单网页，你将需要：

1. 创建一个 HTML 页
1. 添加视频标记
1. 添加 dash.js 播放器
1. 初始化播放器
1. 添加一些 CSS 样式
1. 在实现 MSE 的浏览器中查看结果

只需几行 JavaScript 代码，即可完成初始化播放器。使用 dash.js，将 MPEG-DASH 视频嵌入到基于浏览器的应用程序中确实就这么简单。

##创建 HTML 页

第一步是创建一个包含 <video> 元素的标准 HTML 页，将此文件保存为 basicPlayer.html，如以下示例所示：
	
	<!DOCTYPE html>
	<html>
	  <head><title>Adaptive Streaming in HTML5</title></head>
	  <body>
	    <h1>Adaptive Streaming with HTML5</h1>
	    <video id="videoplayer" controls></video>
	  </body>
	</html>

##添加 DASH.js 播放器

若要将 dash.js 引用实现添加到应用程序，需要从 1.0 版本的 dash.js 项目中找到 dash.all.js 文件。此文件应保存到你的应用程序的 JavaScript 文件夹中。此文件是一个易用文件，将所有必要的 dash.js 代码一起提取到单个文件中。如果你浏览 dash.js 存储库，你将找到各个文件、测试代码以及更多内容，但如果你只想使用 dash.js，那么 dash.all.js 文件就是你所需的文件。

若要将 dash.js 播放器添加到你的应用程序，请将脚本标记添加到 basicPlayer.html 的 head 部分中：

	<!-- DASH-AVC/265 reference implementation -->
	< script src="js/dash.all.js"></script>


接下来，创建一个函数以便在加载页面时初始化播放器。在加载 dash.all.js 的代码行后添加以下脚本：

	<script>
	// setup the video element and attach it to the Dash player
	function setupVideo() {
	  var url = "http://wams.edgesuite.net/media/MPTExpressionData02/BigBuckBunny_1080p24_IYUV_2ch.ism/manifest(format=mpd-time-csf)";
	  var context = new Dash.di.DashContext();
	  var player = new MediaPlayer(context);
	                  player.startup();
	                  player.attachView(document.querySelector("#videoplayer"));
	                  player.attachSource(url);
	}
	</script>

此函数首先创建一个 DashContext。此项用于为特定运行时环境配置应用程序。从技术角度看，它定义在构造应用程序时，依赖关系注入框架应使用的类。在大多数情况下，你将使用 Dash.di.DashContext。

接下来，实例化 dash.js 框架的主类 MediaPlayer。此类包含所需的核心方法（如播放和暂停）、管理与 video 元素的关系，还管理媒体演示描述 (MPD) 文件的解释，该文件说明了要播放的视频。

调用 MediaPlayer 类的 startup() 函数可确保播放器已准备好播放视频。除了其他用处以外，此函数还可确保已加载所有必需的类（如上下文所定义）。播放器准备就绪后，便可以使用 attachView() 函数将 video 元素附加到它。这使 MediaPlayer 可以将视频流注入到该元素，还可以根据需要控制播放。

将 MPD 文件的 URL 传递到 MediaPlayer，这样后者便了解有关它应播放的视频的信息。页面完全加载后，将需要执行刚创建的 setupVideo() 函数。可通过使用 body 元素的 onload 事件来执行此操作。将 <body> 元素更改为：

	<body onload="setupVideo()">

最后，使用 CSS 设置 video 元素的大小。在自适应流式处理环境中，这一点尤其重要，因为当播放适应不断变化的网络条件时，所播放的视频的大小可能会更改。在此简单演示中，直接通过将以下 CSS 添加到页面的 head 部分来强制将 video 元素设为可用浏览器窗口的 80%：
	
	<style>
	video {
	  width: 80%;
	  height: 80%;
	}
	</style>

##播放视频

若要播放视频，请将浏览器指向 basicPlayback.html 文件，并单击所显示的视频播放器上的“播放”。



##另请参阅

[开发视频播放器应用程序](/documentation/articles/media-services-develop-video-players/)

[GitHub dash.js 存储库](https://github.com/Dash-Industry-Forum/dash.js)

<!---HONumber=Mooncake_0613_2016-->