<properties 
	pageTitle="在客户端上插入广告" 
	description="本主题说明如何在客户端上插入广告。" 
	services="media-services" 
	documentationCenter="" 
	authors="juliako" 
	manager="dwrede" 
	editor=""/>

<tags
	ms.service="media-services"
	ms.date="04/18/2016"
	wacn.date="06/27/2016"/>


#在客户端上插入广告

此主题涵盖有关如何在客户端上插入多种类型的广告的信息。

有关在实时流式处理视频中隐藏式字幕和广告支持的详细信息，请参阅[支持的隐藏式字幕和广告插入标准](/documentation/articles/media-services-live-streaming-with-onprem-encoders/#cc_and_ads)。

>[AZURE.NOTE] Azure Media Player 目前不支持广告。

##<a id="insert_ads_into_media"></a>向媒体中插入广告

Azure 媒体服务通过“Windows 媒体平台：播放器框架”提供广告插入支持。附带广告支持的播放器框架在 Windows 8、Silverlight、Windows Phone 8 和 iOS 设备上均可用。每个播放器框架均包含显示如何实现播放器应用程序的示例代码。可插入 media:list 中的广告有三种。

- **线性** - 暂停主视频的全帧广告。
- **非线性** - 播放主视频时显示的叠加式广告，通常为放置在播放器内的一个徽标或其他静态图像。
- **伴随** - 在播放器之外显示的广告。

广告可置于主视频时间线中的任何一个时间点。你必须告知播放器何时播放广告以及播放哪些广告。完成该操作需使用一组标准的基于 XML 的文件：视频广告服务模板 (VAST)、数字视频多广告播放列表 (VMAP)、媒体抽象排序模板 (MAST) 和数字视频播放器广告接口定义 (VPAID)。VAST 文件用于指定要显示哪些广告。VMAP 文件用于指定何时播放各种广告并且包含 VAST XML。MAST 文件是对包含 VAST XML 的广告进行排序的另一种方法。VPAID 文件用于定义视频播放器与广告或广告服务器之间的接口。

每个播放器框架的工作方式不同，且都将被各自的主题所涵盖。本主题将介绍用来插入广告的基本机制。视频播放器应用程序从广告服务器请求广告。广告服务器可以通过多种方式进行响应：

- 返回一个 VAST 文件
- 返回一个 VMAP 文件 （使用嵌入的 VAST)
- 返回一个 MAST 文件 （使用嵌入的 VAST)
- 返回一个含有 VPAID 广告的 VAST 文件

 
###使用视频广告服务模板 (VAST) 文件

VAST 文件指定要显示的广告。以下 XML 是线性广告 VAST 文件的示例：
	
	<VAST version="2.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:noNamespaceSchemaLocation="oxml.xsd">
	  <Ad id="115571748">
	    <InLine>
	      <AdSystem version="2.0 alpha">Atlas</AdSystem>
	      <AdTitle>Unknown</AdTitle>
	      <Description>Unknown</Description>
	      <Survey></Survey>
	      <Error></Error>
	      <Impression id="Atlas"><![CDATA[http://www.myserver.com/tracking-resource]]></Impression>
	      <Creatives>
	        <Creative id="video" sequence="0" AdID="">
	          <Linear>
	            <Duration>00:00:32</Duration>
	            <TrackingEvents>
	              <Tracking event="start"><![CDATA[http://www.myserver.com/start-tracking-resource]]></Tracking>
	              <Tracking event="midpoint"><![CDATA[http://www.myserver.com/midpoint-tracking-resource]]></Tracking>
	              <Tracking event="complete"><![CDATA http://www.myserver.com/complete-tracking-resource]]></Tracking>
	              <Tracking event="expand"><![CDATA[http://www.myserver.com/expand-tracking-resource]]></Tracking>
	            </TrackingEvents>
	            <VideoClicks>
	              <ClickThrough id="Atlas Redirect"><![CDATA[http://www.myserver.com/click-resource]]></ClickThrough>
	              <ClickTracking id="Spare"></ClickTracking>
	            </VideoClicks>
	            <MediaFiles>
	              <MediaFile apiFramework="Windows Media" id="windows_progressive_200" maintainAspectRatio="true" scaleable="true"  delivery="progressive" bitrate="200" width="400" height="300" type="video/x-ms-wmv">
	                <![CDATA[http://www.myserver.com/media/myad_200_4x3.wmv]]>
	              </MediaFile>
	              <MediaFile apiFramework="Windows Media" id="windows_progressive_300" maintainAspectRatio="true" scaleable="true"  delivery="progressive" bitrate="300" width="400" height="300" type="video/x-ms-wmv">
	                <![CDATA[http://www.myserver.com/media/myad_300_4x3.wmv]]>
	              </MediaFile>
	            </MediaFiles>
	          </Linear>
	        </Creative>
	      </Creatives>
	      <Extensions>
	        <Extension type="Atlas">
	        </Extension>
	      </Extensions>
	    </InLine>
	  </Ad>
	</VAST>
	
通过 **<Linear>** 元素描述线性广告。它指定广告的持续时间、跟踪事件、点击行为、点击跟踪和许多 **<MediaFile>** 元素。在 **<TrackingEvents>** 元素内指定跟踪事件，并允许广告服务器跟踪在观看广告时发生的各种事件。在这种情况下将跟踪开始、中点、完成和展开事件。于显示广告时发生开始事件。已观看至少 50% 的广告时间线时发生中点事件。广告结束时发生完成事件。用户将视频播放器展开为全屏时发生展开事件。“点击”是使用 **<ClickThrough>** 元素在 **<VideoClicks>** 元素内指定的，指定用户点击广告时要显示的资源的 URI。“点击跟踪”是在 **<ClickTracking>** 元素中指定的，也在 **<VideoClicks>** 元素中指定，指定用户点击广告时要请求的播放器的跟踪资源。**<MediaFile>** 元素指定有关广告特定编码的信息。如果存在多个 **<MediaFile>** 元素，则视频播放器可以选择适用于该平台的最佳编码。

可以按指定顺序显示线性广告。若要执行此操作，请向 VAST 文件添加其他 <Ad> 元素，并使用 sequence 属性指定顺序。以下示例对此进行了说明：
	
	<VAST version="2.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:noNamespaceSchemaLocation="oxml.xsd">
	  <Ad id="1" sequence="0">
	    <InLine>
	      <AdSystem version="2.0 alpha">Atlas</AdSystem>
	      <AdTitle>Unknown</AdTitle>
	      <Description>Unknown</Description>
	      <Survey></Survey>
	      <Error></Error>
	      <Impression id="Atlas"><![CDATA[http://myserver.com/Impression/Ad1trackingResouce]]></Impression>
	      <Creatives>
	        <Creative id="video" sequence="0" AdID="">
	          <Linear>
	            <Duration>00:00:32</Duration>
	            <MediaFiles>
	              <!-- ... -->
	            </MediaFiles>
	          </Linear>
	        </Creative>
	      </Creatives>
	    </InLine>
	  </Ad>
	  <Ad id="2" sequence="1">
	    <InLine>
	      <AdSystem version="2.0 alpha">Atlas</AdSystem>
	      <AdTitle>Unknown</AdTitle>
	      <Description>Unknown</Description>
	      <Survey></Survey>
	      <Error></Error>
	      <Impression id="Atlas"><![CDATA[http://myserver.com/Impression/Ad2trackingResouce]]></Impression>
	      <Creatives>
	        <Creative id="video" sequence="0" AdID="">
	          <Linear>
	            <Duration>00:00:30</Duration>
	            <MediaFiles>
	              <!-- ... -->
	            </MediaFiles>
	          </Linear>
	        </Creative>
	      </Creatives>
	    </InLine>
	  </Ad>
	</VAST>
	
非线性广告也是在 <Creative> 元素中指定的。以下示例演示描述非线性广告的 <Creative> 元素。

	<Creative id="video" sequence="1" AdID="">
	  <NonLinearAds>
	    <NonLinear width="216" height="121" minSuggestedDuration="00:00:15">
	      <StaticResource creativeType="image/png"><![CDATA[http://myserver/images/image.png]]></StaticResource>
	      <StaticResource creativeType="image/jpg"><![CDATA[http://myserver/images/image.jpg]]></StaticResource>
	    </NonLinear>
	    <TrackingEvents>
	         <Tracking event="acceptInvitation"><![CDATA[http://myserver/tracking/trackingID]></Tracking>
	         <Tracking event="collapse"><![CDATA[http://myserver/tracking/trackingID2]]></Tracking>
	     </TrackingEvents>
	   </NonLinearAds>
	</Creative>

 
**<NonLinearAds>** 元素可以包含一个或多个 **<NonLinear>** 元素，其中每一个均可描述一个非线性广告。**<NonLinear>** 元素指定非线性广告的资源。资源可以是 **<StaticResouce>**、**<IFrameResource>** 或 **<HTMLResouce>**。**<StaticResource>** 描述非 HTML 资源，并定义指定资源显示方式的 creativeType 属性：

Image/gif、image/jpeg、image/png - 资源显示在 HTML **<img>** 标记中。

Application/x-javascript - 资源显示在 HTML <**脚本**> 标记中。

Application/x-shockwave-flash – 资源显示在 Flash Player 中。

**<IFrameResource>** 描述可以在 IFrame 中显示的 HTML 资源。**<HTMLResource>** 描述可以插入网页中的一段 HTML 代码。**<TrackingEvents>** 指定跟踪事件以及事件发生时要请求的 URI。在此示例中，跟踪了 acceptInvitation 事件和折叠事件。有关 **<NonLinearAds>** 元素及其子元素的详细信息，请参阅 IAB.NET/VAST。请注意，**<TrackingEvents>** 元素位于 **<NonLinearAds>** 元素内，而不是 **<NonLinear>** 元素内。

在 <CompanionAds> 元素内定义伴随广告。<CompanionAds> 元素可以包含一个或多个 <Companion> 元素。每个 <Companion> 元素均描述一个伴随广告，并且可以包含以与非线性广告相同的方式定义的 <StaticResource><IFrameResource> 或 <HTMLResource>。VAST 文件可以包含多个伴随广告，播放器应用程序可以选择最适合显示的广告。有关 VAST 的详细信息，请参阅 [VAST 3.0](http://www.iab.net/media/file/VASTv3.0.pdf)。

###使用数字视频多广告播放列表 (VMAP) 文件

VMAP 文件使你能够指定发生广告中断的时间、每次中断的时长、可在中断期间显示的广告数以及可在中断期间显示的广告类型。以下是定义单次广告中断的示例 VMAP 文件：
	
	<vmap:VMAP xmlns:vmap="http://www.iab.net/vmap-1.0" version="1.0">
	  <vmap:AdBreak breakType="linear" breakId="mypre" timeOffset="start">
	    <vmap:AdSource allowMultipleAds="true" followRedirects="true" id="1">
	      <vmap:VASTData>
	        <VAST version="2.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:noNamespaceSchemaLocation="oxml.xsd">
	          <Ad id="115571748">
	            <InLine>
	              <AdSystem version="2.0 alpha">Atlas</AdSystem>
	              <AdTitle>Unknown</AdTitle>
	              <Description>Unknown</Description>
	              <Survey></Survey>
	              <Error></Error>
	              <Impression id="Atlas"><![CDATA[http://view.atdmt.com/000/sview/115571748/direct;ai.201582527;vt.2/01/634364885739970673]]></Impression>
	              <Creatives>
	                <Creative id="video" sequence="0" AdID="">
	                  <Linear>
	                    <Duration>00:00:32</Duration>
	                    <MediaFiles>
	                      <MediaFile apiFramework="Windows Media" id="windows_progressive_200" maintainAspectRatio="true" scaleable="true"  delivery="progressive" bitrate="200" width="400" height="300" type="video/x-ms-wmv">
	                        <![CDATA[http://smf.blob.core.chinacloudapi.cn/samples/ads/media/XBOX_HD_DEMO_700_1_000_200_4x3.wmv]]>
	                      </MediaFile>
	                      <MediaFile apiFramework="Windows Media" id="windows_progressive_300" maintainAspectRatio="true" scaleable="true"  delivery="progressive" bitrate="300" width="400" height="300" type="video/x-ms-wmv">
	                        <![CDATA[http://smf.blob.core.chinacloudapi.cn/samples/ads/media/XBOX_HD_DEMO_700_2_000_300_4x3.wmv]]>
	                      </MediaFile>
	                      <MediaFile apiFramework="Windows Media" id="windows_progressive_500" maintainAspectRatio="true" scaleable="true"  delivery="progressive" bitrate="500" width="400" height="300" type="video/x-ms-wmv">
	                        <![CDATA[http://smf.blob.core.chinacloudapi.cn/samples/ads/media/XBOX_HD_DEMO_700_1_000_500_4x3.wmv]]>
	                      </MediaFile>
	                      <MediaFile apiFramework="Windows Media" id="windows_progressive_700" maintainAspectRatio="true" scaleable="true" delivery="progressive" bitrate="700" width="400" height="300" type="video/x-ms-wmv">
	                        <![CDATA[http://smf.blob.core.chinacloudapi.cn/samples/ads/media/XBOX_HD_DEMO_700_2_000_700_4x3.wmv]]>
	                      </MediaFile>
	                    </MediaFiles>
	                  </Linear>
	                </Creative>
	              </Creatives>
	            </InLine>
	          </Ad>
	        </VAST>
	      </vmap:VASTData>
	    </vmap:AdSource>
	    <vmap:TrackingEvents>
	      <vmap:Tracking event="breakStart">
	        http://MyServer.com/breakstart.gif
	      </vmap:Tracking>
	    </vmap:TrackingEvents>
	  </vmap:AdBreak>
	</vmap:VMAP>
	 
VMAP 文件以 <VMAP> 元素开头，该元素包含一个或多个 <AdBreak> 元素，每一个均定义一个广告中断。每一个广告中断均指定一个中断类型、中断 ID 和时间偏移量。BreakType 属性指定可在中断期间播放的广告类型：线性广告、非线性广告或显示广告。显示广告映射到 VAST 伴随广告。可以在逗号（不含空格）分隔的列表中指定多个广告类型。BreakID 是广告的可选标识符。TimeOffset 指定显示广告的时间。可以通过以下方式之一进行指定：

1. 时间 - 采用 hh:mm:ss 或 hh:mm:ss.mmm 格式，其中 .mmm 为毫秒。此属性的值指定从视频时间线开始到广告中断开始的一段时间。
1. 百分比 - 采用 n% 格式，其中 n 为播放广告前要播放的视频时间线所占的百分比
1. 开始/结束 - 指定应在播放视频之前或之后显示广告
1. 位置 - 指定广告中断的计时未知时（例如，实时流式处理过程中）的广告中断顺序。采用 #n 格式指定每次广告中断的顺序，其中 n 为整数 1 或更大整数。1 表示应在第一个机会时播放广告，2 表示应在第二个机会时播放广告，以此类推。

在 <**AdBreak**> 元素中可以有一个 <**AdSource**> 元素。<**AdSource**> 元素包含以下属性：

1. Id - 指定广告源的标识符
1. allowMultipleAds – 一个布尔值，指定是否可以在广告中断期间显示多个广告
1. followRedirects – 一个可选布尔值，指定视频播放器是否应遵循广告响应中的重定向

<**AdSource**> 元素为播放器提供线内广告响应或对广告响应的引用。它可包含以下元素之一：

- <VASTAdData> 表示 VMAP 文件中嵌入了一个 VAST 广告响应
- <AdTagURI> 引用另一个系统中的广告响应的 URI
- <CustomAdData> 表示非 VAST 响应的任意字符串

在此示例中，线内广告响应是使用包含 VAST 广告响应的 <VASTAdData> 元素指定的。有关其他元素的详细信息，请参阅 [VMAP](http://www.iab.net/guidelines/508676/digitalvideo/vsuite/vmap)。

<**AdBreak**> 元素还可以包含一个 <**TrackingEvents**> 元素。<**TrackingEvents**> 元素允许跟踪广告中断的开始或结束，或广告中断期间是否发生了错误。<**TrackingEvents**> 元素包含一个或多个 <**Tracking**> 元素，其中每一个均指定一个跟踪事件和一个跟踪 URI。可能的跟踪事件是：

1. breakStart - 跟踪广告中断的开始
1. breakEnd - 跟踪广告中断的完成
1. error - 跟踪广告中断期间发生的错误

以下示例演示指定跟踪事件的 VMAP 文件

	<vmap:VMAP xmlns:vmap="http://www.iab.net/vmap-1.0" version="1.0">
	  <vmap:AdBreak breakType="linear" breakId="mypre" timeOffset="start">
	    <vmap:AdSource allowMultipleAds="true" followRedirects="true" id="1">
	      <vmap:VASTData>
	        <!--Inline VAST -->
	      </vmap:VASTData>
	    </vmap:AdSource>
	    <vmap:TrackingEvents>
	      <vmap:Tracking event="breakStart">
	        http://MyServer.com/breakstart.gif
	      </vmap:Tracking>
	      <vmap:Tracking event="breakend">
	        http://MyServer.com/breakend.gif
	      </vmap:Tracking>
	      <vmap:Tracking event="error">
	        http://MyServer.com/error.gif
	      </vmap:Tracking>
	    </vmap:TrackingEvents>
	  </vmap:AdBreak>
	</vmap:VMAP>

有关 <**TrackingEvents**> 元素及其子元素的详细信息，请参阅 http://iab.org/VMAP.pdf

###使用媒体摘要排序模板 (MAST) 文件

MAST 文件允许指定定义何时显示广告的触发器。以下是一个示例 MAST 文件，它包含前置式广告、中置式广告和后置式广告的触发器。

	<MAST xsi:schemaLocation="http://openvideoplayer.sf.net/mast http://openvideoplayer.sf.net/mast/mast.xsd" xmlns="http://openvideoplayer.sf.net/mast" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance">
	  <triggers>
	    <trigger id="preroll" description="preroll every item"  >
	      <startConditions>
	        <condition type="event" name="OnItemStart" />
	      </startConditions>
	      <sources>
	        <source uri="http://smf.blob.core.chinacloudapi.cn/samples/win8/ads/vast_linear.xml" format="vast">
	          <sources />
	        </source>
	      </sources>
	    </trigger>
	
	    <trigger id="midroll" description="midroll at 15 sec."  >
	      <startConditions>
	        <condition type="property" name="Position" value="00:00:15.0" operator="GEQ" />
	      </startConditions>
	      <endConditions>
	        <condition type="event" name="OnItemEnd"/>
	        <!--This 'resets' the trigger for the next clip-->
	      </endConditions>
	      <sources>
	        <source uri="http://smf.blob.core.chinacloudapi.cn/samples/win8/ads/vast_linear.xml" format="vast">
	          <sources />
	        </source>
	      </sources>
	    </trigger>
	
	    <trigger id="postroll" description="postroll"  >
	      <startConditions>
	        <condition type="event" name="OnItemEnd"/>
	      </startConditions>
	      <sources>
	        <source uri="http://smf.blob.core.chinacloudapi.cn/samples/win8/ads/vast_linear.xml" format="vast">
	          <sources />
	        </source>
	      </sources>
	    </trigger>
	  </triggers>
	</MAST>

 

MAST 文件以 **<MAST>** 元素开头，该元素包含一个 **<triggers>** 元素。<triggers> 元素包含一个或多个 **<trigger>** 定义应何时播放广告的元素。

**<trigger>** 元素包含一个 **<startConditions>** 元素，后者指定应开始播放广告的时间。**<startConditions>** 元素包含一个或多个 <condition> 元素。当每个 <condition> 评估结果为 true 时，将启动或撤销一个触发器，具体取决于 <condition> 是否分别包含在 **< startConditions**> 或 **<endConditions>** 元素中。有多个 <condition> 元素时，将它们视为隐式 OR，任何评估结果为 true 的条件均将导致触发器启动。<condition> 元素可以嵌套。预设了子 <condition> 元素时，将它们视为隐式 AND；若要启动触发器，则所有条件的评估结果必须都为 true。<condition> 元素包含定义条件的以下属性：

1. **type** – 指定的条件、事件或属性的类型
1. **name** – 要在评估过程中使用的属性或事件的名称
1. **value** – 将针对其进行评估的属性的值
1. **operator** – 要在评估期间使用的操作符：EQ（等于）、NEQ（不等于）、GTR（大于）、GEQ（大于或等于）、LT（小于）、LEQ（小于或等于）、MOD（取模）

**<endConditions>** 也包含 <condition> 元素。当某个条件的评估结果为 true 时，重置触发器。<trigger> 元素也包含 <sources> 元素，后者包含一个或多个 <source> 元素。<source> 元素定义广告响应的 URI 和广告响应的类型。在此示例中为 VAST 响应给定了一个 URI。


	<trigger id="postroll" description="postroll"  >
      <startConditions>
        <condition/>
      </startConditions>
      <sources>
        <source uri="http://smf.blob.core.chinacloudapi.cn/samples/win8/ads/vast_linear.xml" format="vast">
          <sources />
        </source>
      </sources>
    </trigger>
 

###使用视频播放器广告接口定义 (VPAID)

VPAID 是用于使可执行广告单元能够与视频播放器进行通信的 API。这可实现高度交互的广告体验。用户可以与广告交互；广告可以对查看者采取的操作做出响应。例如，广告可能会显示允许用户查看详细信息或更长时间版广告的按钮。视频播放器必须支持 VPAID API 且可执行广告必须实现该 API。当播放机从广告服务器请求广告时，服务器可能使用包含 VPAID 广告的 VAST 响应进行响应。

在必须于运行时环境（例如 Adobe Flash™ 或可以在 Web 浏览器中执行的 JavaScript）中执行的代码中创建可执行广告。当广告服务器返回包含 VPAID 广告的 VAST 响应时，<MediaFile> 元素中 apiFramework 属性的值必须为“VPAID”。此属性指定所含广告为 VPAID 可执行广告。类型属性必须设置为可执行文件（如“application/x-shockwave-flash”或“application/x-javascript”）的 MIME 类型。以下 XML 代码片段演示包含 VPAID 可执行广告的 VAST 响应中的 <MediaFile> 元素。

	
	<MediaFiles>
	   <MediaFile id="1" delivery="progressive" type=”application/x-shockwaveflash”
	              width=”640” height=”480” apiFramework=”VPAID”>
	       <!-- CDATA wrapped URI to executable ad -->
	   </MediaFile>
	</MediaFiles>
 

可以使用 VAST 响应中 <Linear> 或 <NonLinear> 元素内的 <AdParameters> 元素来初始化可执行广告。有关 <AdParameters> 元素的详细信息，请参阅 [VAST 3.0](http://www.iab.net/media/file/VASTv3.0.pdf)。有关 VPAID API 的详细信息，请参阅 [VPAID 2.0](http://www.iab.net/media/file/VPAID_2.0_Final_04-10-2012.pdf)。

##实现带有支持广告的 Windows 或 Windows Phone 8 播放器

Microsoft Media Platform：适用于 Windows 8 和 Windows Phone 8 的播放器框架包含示例应用程序集合，这些示例应用程序向你展示如何使用该框架实现视频播放器应用程序。可以从[适用于 Windows 8 和 Windows Phone 8 的播放器框架](https://playerframework.codeplex.com)下载播放器框架和示例。

打开 Microsoft.PlayerFramework.Xaml.Samples 解决方案时，将看到项目中的多个文件夹。Advertising 文件夹包含与创建具有广告支持的视频播放器相关的示例代码。Advertising 文件夹中有多个 XAML/cs 文件，每一个均显示如何以不同的方式插入广告。以下列表对每个文件进行描述：

- AdPodPage.xaml 演示如何显示广告荚。
- AdSchedulingPage.xaml 演示如何安排广告。
- FreeWheelPage.xaml 演示如何使用 FreeWheel 插件来安排广告。
- MastPage.xaml 演示如何使用 MAST 文件来安排广告。
- ProgrammaticAdPage.xaml 演示如何以编程方式将广告安排到视频中。
- ScheduleClipPage.xaml 演示如何在无 VAST 文件的情况下安排广告。
- VastLinearCompanionPage.xaml 演示如何插入线性广告和伴随广告。
- VastNonLinearPage.xaml 演示如何插入非线性广告。
- VmapPage.xaml 演示如何使用 VMAP 文件来指定广告。

每一个示例均使用由播放器框架定义的 MediaPlayer 类。大多数示例都使用为不同广告响应格式添加支持的插件。ProgrammaticAdPage 示例以编程方式与 MediaPlayer 实例交互。

###AdPodPage 示例

此示例使用 AdSchedulerPlugin 来定义何时显示广告。在此示例中，安排于 5 秒后播放中置式广告。广告荚（按顺序播放的一组广告）是在从广告服务器返回的 VAST 文件中指定的。在 <RemoteAdSource> 元素中指定 VAST 文件的 URI。

	<mmppf:MediaPlayer x:Name="player" Source="http://smf.blob.core.chinacloudapi.cn/samples/videos/bigbuck.mp4">
	
	    <mmppf:MediaPlayer.Plugins>
	        <ads:AdSchedulerPlugin>
	            <ads:AdSchedulerPlugin.Advertisements>
	
	                <ads:MidrollAdvertisement Time="00:00:05">
	                    <ads:MidrollAdvertisement.Source>
	                        <ads:RemoteAdSource Uri="http://smf.blob.core.chinacloudapi.cn/samples/win8/ads/vast_adpod.xml" Type="vast"/>
	                    </ads:MidrollAdvertisement.Source>
	                </ads:MidrollAdvertisement>
	
	            </ads:AdSchedulerPlugin.Advertisements>
	        </ads:AdSchedulerPlugin>
	        <ads:AdHandlerPlugin/>
	    </mmppf:MediaPlayer.Plugins>
	</mmppf:MediaPlayer>

有关 AdSchedulerPlugin 的详细信息，请参阅 [Windows 8 和 Windows Phone 8 上的播放器框架中的广告](http://playerframework.codeplex.com/wikipage?title=Advertising&referringTitle=Windows%208%20Player%20Documentation)

###AdSchedulingPage

此示例还使用 AdSchedulerPlugin。它会安排三种广告：前置式广告、中置式广告和后置式广告。在 <RemoteAdSource> 元素中指定每种广告的 VAST 文件的 URI。
	
	<mmppf:MediaPlayer x:Name="player" Source="http://smf.blob.core.chinacloudapi.cn/samples/videos/bigbuck.mp4">
	            <mmppf:MediaPlayer.Plugins>
	                <ads:AdSchedulerPlugin>
	                    <ads:AdSchedulerPlugin.Advertisements>
	
	                        <ads:PrerollAdvertisement>
	                            <ads:PrerollAdvertisement.Source>
	                                <ads:RemoteAdSource Uri="http://smf.blob.core.chinacloudapi.cn/samples/win8/ads/vast_linear.xml" Type="vast"/>
	                            </ads:PrerollAdvertisement.Source>
	                        </ads:PrerollAdvertisement>
	
	                        <ads:MidrollAdvertisement Time="00:00:15">
	                            <ads:MidrollAdvertisement.Source>
	                                <ads:RemoteAdSource Uri="http://smf.blob.core.chinacloudapi.cn/samples/win8/ads/vast_linear.xml" Type="vast"/>
	                            </ads:MidrollAdvertisement.Source>
	                        </ads:MidrollAdvertisement>
	
	                        <ads:PostrollAdvertisement>
	                            <ads:PostrollAdvertisement.Source>
	                                <ads:RemoteAdSource Uri="http://smf.blob.core.chinacloudapi.cn/samples/win8/ads/vast_linear.xml" Type="vast"/>
	                            </ads:PostrollAdvertisement.Source>
	                        </ads:PostrollAdvertisement>
	
	                    </ads:AdSchedulerPlugin.Advertisements>
	                </ads:AdSchedulerPlugin>
	                <ads:AdHandlerPlugin/>
	            </mmppf:MediaPlayer.Plugins>
	        </mmppf:MediaPlayer>


###FreeWheelPage

此示例使用 FreeWheelPlugin，它指定一个指定 URL 的源属性，该 URL 指向一个 SmartXML 文件，该文件指定广告内容和广告安排信息。
	
	<mmppf:MediaPlayer x:Name="player" Source="http://smf.blob.core.chinacloudapi.cn/samples/videos/bigbuck.mp4">
	            <mmppf:MediaPlayer.Plugins>
	                <ads:FreeWheelPlugin Source="http://smf.blob.core.chinacloudapi.cn/samples/win8/ads/freewheel.xml"/>
	                <ads:AdHandlerPlugin/>
	            </mmppf:MediaPlayer.Plugins>
	        </mmppf:MediaPlayer>

###MastPage

此示例使用允许使用 MAST 文件的 MastSchedulerPlugin。源属性指定 MAST 文件的位置。
	
	<mmppf:MediaPlayer x:Name="player" Source="http://smf.blob.core.chinacloudapi.cn/samples/videos/bigbuck.mp4">
	            <mmppf:MediaPlayer.Plugins>
	                <ads:MastSchedulerPlugin Source="http://smf.blob.core.chinacloudapi.cn/samples/win8/ads/mast.xml" />
	                <ads:AdHandlerPlugin/>
	            </mmppf:MediaPlayer.Plugins>
	        </mmppf:MediaPlayer>

###ProgrammaticAdPage

此示例以编程方式与 MediaPlayer 交互。ProgrammaticAdPage.xaml 文件实例化 MediaPlayer：

	<mmppf:MediaPlayer x:Name="player" Source="http://smf.blob.core.chinacloudapi.cn/samples/videos/bigbuck.mp4"/>

ProgrammaticAdPage.xaml.cs 文件创建 AdHandlerPlugin，添加 TimelineMarker 以指定何时显示广告，然后为 MarkerReached 事件添加处理程序，该程序加载指定指向 VAST 文件的 URI 的 RemoteAdSource，然后播放广告。
	
	public sealed partial class ProgrammaticAdPage : Microsoft.PlayerFramework.Samples.Common.LayoutAwarePage
	    {
	        AdHandlerPlugin adHandler;
	
	        public ProgrammaticAdPage()
	        {
	            this.InitializeComponent();
	            adHandler = new AdHandlerPlugin();
	            player.Plugins.Add(new AdHandlerPlugin());
	            player.Markers.Add(new TimelineMarker() { Time = TimeSpan.FromSeconds(5), Type = "myAd" });
	            player.MarkerReached += pf_MarkerReached;
	        }
	
	        async void pf_MarkerReached(object sender, TimelineMarkerRoutedEventArgs e)
	        {
	            if (e.Marker.Type == "myAd")
	            {
	                var adSource = new RemoteAdSource() { Type = VastAdPayloadHandler.AdType, Uri = new Uri("http://smf.blob.core.chinacloudapi.cn/samples/win8/ads/vast_linear.xml") };
	                //var adSource = new AdSource() { Type = DocumentAdPayloadHandler.AdType, Payload = SampleAdDocument };
	                var progress = new Progress<AdStatus>();
	                try
	                {
	                    await player.PlayAd(adSource, progress, CancellationToken.None);
	                }
	                catch { /* ignore */ }
	            }
	        }

###ScheduleClipPage

此示例使用 AdSchedulerPlugin 通过指定包含广告的 .wmv 文件安排中置式广告。
	
	<mmppf:MediaPlayer x:Name="player" Source="http://smf.chinacloudapp.cn/html5/media/bigbuck.mp4">
	            <mmppf:MediaPlayer.Plugins>
	                <ads:AdSchedulerPlugin>
	                    <ads:AdSchedulerPlugin.Advertisements>
	
	                        <ads:MidrollAdvertisement Time="00:00:05">
	                            <ads:MidrollAdvertisement.Source>
	                                <ads:AdSource Type="clip">
	                                    <ads:AdSource.Payload>
	                                        <ads:ClipAdPayload MediaSource="http://smf.blob.core.chinacloudapi.cn/samples/ads/media/XBOX_HD_DEMO_700_2_000_700_4x3.wmv" MimeType="video/x-ms-wmv" />
	                                    </ads:AdSource.Payload>
	                                </ads:AdSource>
	                            </ads:MidrollAdvertisement.Source>
	                        </ads:MidrollAdvertisement>
	
	                    </ads:AdSchedulerPlugin.Advertisements>
	                </ads:AdSchedulerPlugin>
	                <ads:AdHandlerPlugin/>
	            </mmppf:MediaPlayer.Plugins>
	        </mmppf:MediaPlayer>

###VastLinearCompanionPage

此示例阐释如何使用 AdSchedulerPlugin 来安排中置式线性广告和伴随广告。<RemoteAdSource> 元素指定 VAST 文件的位置。
	
	<mmppf:MediaPlayer Grid.Row="1"  x:Name="player" Source="http://smf.blob.core.chinacloudapi.cn/samples/videos/bigbuck.mp4">
	            <mmppf:MediaPlayer.Plugins>
	                <ads:AdSchedulerPlugin>
	                    <ads:AdSchedulerPlugin.Advertisements>
	
	                        <ads:MidrollAdvertisement Time="00:00:05">
	                            <ads:MidrollAdvertisement.Source>
	                                <ads:RemoteAdSource Uri="http://smf.blob.core.chinacloudapi.cn/samples/win8/ads/vast_linear_companions.xml" Type="vast"/>
	                            </ads:MidrollAdvertisement.Source>
	                        </ads:MidrollAdvertisement>
	
	                    </ads:AdSchedulerPlugin.Advertisements>
	                </ads:AdSchedulerPlugin>
	                <ads:AdHandlerPlugin/>
	            </mmppf:MediaPlayer.Plugins>
	        </mmppf:MediaPlayer>

###VastLinearNonLinearPage

此示例使用 AdSchedulerPlugin 来安排线性广告和非线性广告。使用 <RemoteAdSource> 元素来指定 VAST 文件位置。
	
	<mmppf:MediaPlayer x:Name="player" Source="http://smf.blob.core.chinacloudapi.cn/samples/videos/bigbuck.mp4">
	            <mmppf:MediaPlayer.Plugins>
	                <ads:AdSchedulerPlugin>
	                    <ads:AdSchedulerPlugin.Advertisements>
	
	                        <ads:MidrollAdvertisement Time="00:00:05">
	                            <ads:MidrollAdvertisement.Source>
	                                <ads:RemoteAdSource Uri="http://smf.blob.core.chinacloudapi.cn/samples/win8/ads/vast_linear_nonlinear.xml" Type="vast"/>
	                            </ads:MidrollAdvertisement.Source>
	                        </ads:MidrollAdvertisement>
	                        
	                    </ads:AdSchedulerPlugin.Advertisements>
	                </ads:AdSchedulerPlugin>
	                <ads:AdHandlerPlugin/>
	            </mmppf:MediaPlayer.Plugins>
	        </mmppf:MediaPlayer>

###VMAPPage

此示例使用 VmapSchedulerPlugin，以便使用 VMAP 文件安排广告。在 <VmapSchedulerPlugin> 元素的源属性中指定指向 VMAP 文件的 URI。
	
	<mmppf:MediaPlayer x:Name="player" Source="http://smf.blob.core.chinacloudapi.cn/samples/videos/bigbuck.mp4">
	            <mmppf:MediaPlayer.Plugins>
	                <ads:VmapSchedulerPlugin Source="http://smf.blob.core.chinacloudapi.cn/samples/win8/ads/vmap.xml"/>
	                <ads:AdHandlerPlugin/>
	            </mmppf:MediaPlayer.Plugins>
	        </mmppf:MediaPlayer>

##实现带有广告支持的 iOS 视频播放器


Microsoft Media Platform：适用于 iOS 的播放器框架包含示例应用程序集合，这些示例应用程序向你展示如何使用该框架实现视频播放器应用程序。可以从 [Azure 媒体播放器框架](https://github.com/Azure/azure-media-player-framework)下载播放器框架和示例。GitHub 页面具有指向 Wiki（含有关播放器框架的其他信息）的链接和播放器示例简介：[Azure 媒体播放器 Wiki](https://github.com/Azure/azure-media-player-framework/wiki/How-to-use-Azure-media-player-framework)。


###使用 VMAP 安排广告

以下示例演示如何使用 VMAP 文件安排广告。

	// How to schedule an Ad using VMAP.
	//First download the VMAP manifest
	
	if (![framework.adResolver downloadManifest:&manifest withURL:[NSURL URLWithString:@"http://portalvhdsq3m25bf47d15c.blob.core.chinacloudapi.cn/vast/PlayerTestVMAP.xml"]])
	        {
	            [self logFrameworkError];
	        }
	        else
	        {
	            // Schedule a list of ads using the downloaded VMAP manifest
	            if (![framework scheduleVMAPWithManifest:manifest])
	            {
	                [self logFrameworkError];
	            }          
	        }

###使用 VAST 安排广告

以下示例演示如何安排后期绑定 VAST 广告。
	
	//Example:3 How to schedule a late binding VAST ad.
	// set the start time for the ad
    adLinearTime.startTime = 13;
    adLinearTime.duration = 0;
	// Specify the URI of the VAST file
    NSString *vastAd1=@"http://portalvhdsq3m25bf47d15c.blob.core.chinacloudapi.cn/vast/PlayerTestVAST.xml";
	// Create an AdInfo object
	 AdInfo *vastAdInfo1 = [[[AdInfo alloc] init] autorelease];
	// set URL to VAST file
	vastAdInfo1.clipURL = [NSURL URLWithString:vastAd1];
	// set running time of ad
    vastAdInfo1.mediaTime = [[[MediaTime alloc] init] autorelease];
    vastAdInfo1.mediaTime.clipBeginMediaTime = 0;
    vastAdInfo1.mediaTime.clipEndMediaTime = 10;
    vastAdInfo1.policy = @"policy for late binding VAST";
	// specify ad type
    vastAdInfo1.type = AdType_Midroll;
    vastAdInfo1.appendTo=-1;
    adIndex = 0;
	// schedule ad
    if (![framework scheduleClip:vastAdInfo1 atTime:adLinearTime forType:PlaylistEntryType_VAST andGetClipId:&adIndex])
    {
        [self logFrameworkError];
    }
         

以下示例演示如何安排早期绑定 VAST 广告。

    //Example:4 Schedule an early binding VAST ad
    //Download the VAST file
    if (![framework.adResolver downloadManifest:&manifest withURL:[NSURL URLWithString:@"http://portalvhdsq3m25bf47d15c.blob.core.chinacloudapi.cn/vast/PlayerTestVAST.xml"]])
    {
        [self logFrameworkError];
    }
    else
    {
        adLinearTime.startTime = 7;
        adLinearTime.duration = 0;
        
		// Create AdInfo instance
	    AdInfo *vastAdInfo2 = [[[AdInfo alloc] init] autorelease];
	    vastAdInfo2.mediaTime = [[[MediaTime alloc] init] autorelease];
	    vastAdInfo2.policy = @"policy for early binding VAST";
		// specify ad type
	    vastAdInfo2.type = AdType_Midroll;
	    vastAdInfo2.appendTo=-1;
		// schedule ad
	    if (![framework scheduleVASTClip:vastAdInfo2 withManifest:manifest atTime:adLinearTime andGetClipId:&adIndex])
	    {
	        [self logFrameworkError];
	    }
	}

以下示例演示如何使用粗剪编辑 (RCE) 插入广告

	//Example:1 How to use RCE.
	// specify manifest for ad content
	NSString *secondContent=@"http://wamsblureg001orig-hs.chinacloudapp.cn/6651424c-a9d1-419b-895c-6993f0f48a26/The%20making%20of%20Microsoft%20Surface-m3u8-aapl.ism/Manifest(format=m3u8-aapl)";
	
	// specify ad length
    mediaTime.currentPlaybackPosition = 0;
    mediaTime.clipBeginMediaTime = 0;
    mediaTime.clipEndMediaTime = 80;
	// append ad content
    if (![framework appendContentClip:[NSURL URLWithString:secondContent] withMediaTime:mediaTime andGetClipId:&clipId])
    {
        [self logFrameworkError];
    }

以下示例演示如何安排广告荚。

	//Example:5 Schedule an ad Pod.
	// Set start time for ad
	adLinearTime.startTime = 23;
	adLinearTime.duration = 0;
	
	// Specify URL to content
	NSString *adpodSt1=@"https://portalvhdsq3m25bf47d15c.blob.core.chinacloudapi.cn/asset-e47b43fd-05dc-4587-ac87-5916439ad07f/Windows%208_%20Cliffjumpers.mp4?st=2012-11-28T16%3A31%3A57Z&se=2014-11-28T16%3A31%3A57Z&sr=c&si=2a6dbb1e-f906-4187-a3d3-7e517192cbd0&sig=qrXYZBekqlbbYKqwovxzaVZNLv9cgyINgMazSCbdrfU%3D";
	// Create an AdInfo instance
	AdInfo *adpodInfo1 = [[[AdInfo alloc] init] autorelease];
	// set URI to ad content
	adpodInfo1.clipURL = [NSURL URLWithString:adpodSt1];
	// Set ad running time
	adpodInfo1.mediaTime = [[[MediaTime alloc] init] autorelease];
	adpodInfo1.mediaTime.clipBeginMediaTime = 0;
	adpodInfo1.mediaTime.clipEndMediaTime = 17;
	adpodInfo1.policy = @"policy for ad pod 1";
	// Set ad type
	adpodInfo1.type = AdType_Midroll;
	adpodInfo1.appendTo=-1;
	adIndex = 0;
	// Schedule ad
	if (![framework scheduleClip:adpodInfo1 atTime:adLinearTime forType:PlaylistEntryType_Media andGetClipId:&adIndex])
	{
	    [self logFrameworkError];
	}

以下示例演示如何安排非粘性中置式广告。不管查看器执行了什么查找，非粘性广告均仅播放一次。
	
	//Example:6 Schedule a single non sticky mid roll Ad
	// specify URL to content
	NSString *oneTimeAd=@"http://wamsblureg001orig-hs.chinacloudapp.cn/5389c0c5-340f-48d7-90bc-0aab664e5f02/Windows%208_%20You%20and%20Me%20Together-m3u8-aapl.ism/Manifest(format=m3u8-aapl)";
	
	// create an AdInfo instance
	AdInfo *oneTimeInfo = [[[AdInfo alloc] init] autorelease];
	// set URL of ad
	oneTimeInfo.clipURL = [NSURL URLWithString:oneTimeAd];
	oneTimeInfo.mediaTime = [[[MediaTime alloc] init] autorelease];
	oneTimeInfo.mediaTime.clipBeginMediaTime = 0;
	oneTimeInfo.mediaTime.clipEndMediaTime = -1;
	oneTimeInfo.policy = @"policy for one-time ad";
	// set ad start time
	adLinearTime.startTime = 43;
	adLinearTime.duration = 0;
	// set ad type
	oneTimeInfo.type = AdType_Midroll;
	// non sticky ad
	oneTimeInfo.deleteAfterPlayed = YES;
	// schedule ad
	if (![framework scheduleClip:oneTimeInfo atTime:adLinearTime forType:PlaylistEntryType_Media andGetClipId:&adIndex])
	{
	    [self logFrameworkError];
	}

以下示例演示如何安排粘性中置式广告。每当到达时间线上指定时间点时就会显示粘性广告。

	//Example:7 Schedule a single sticky mid roll Ad
	NSString *stickyAd=@"http://wamsblureg001orig-hs.chinacloudapp.cn/2e4e7d1f-b72a-4994-a406-810c796fc4fc/The%20Surface%20Movement-m3u8-aapl.ism/Manifest(format=m3u8-aapl)";
	// create AdInfo instance
	AdInfo *stickyAdInfo = [[[AdInfo alloc] init] autorelease];
	// set URI to ad
	stickyAdInfo.clipURL = [NSURL URLWithString:stickyAd];
	stickyAdInfo.mediaTime = [[[MediaTime alloc] init] autorelease];
	stickyAdInfo.mediaTime.clipBeginMediaTime = 0;
	stickyAdInfo.mediaTime.clipEndMediaTime = 15;
	stickyAdInfo.policy = @"policy for sticky mid-roll ad";
	// set ad start time
	adLinearTime.startTime = 64;
	adLinearTime.duration = 0;
	// set ad type
	stickyAdInfo.type = AdType_Midroll;
	stickyAdInfo.deleteAfterPlayed = NO;
	// schedule ad
	if (![framework scheduleClip:stickyAdInfo atTime:adLinearTime forType:PlaylistEntryType_Media andGetClipId:&adIndex])
	{
	    [self logFrameworkError];
	}


以下示例演示如何安排后置式广告。

	//Example:8 Schedule Post Roll Ad
	NSString *postAdURLString=@"http://wamsblureg001orig-hs.chinacloudapp.cn/aa152d7f-3c54-487b-ba07-a58e0e33280b/wp-m3u8-aapl.ism/Manifest(format=m3u8-aapl)";
	// create AdInfo instance
	AdInfo *postAdInfo = [[[AdInfo alloc] init] autorelease];
	postAdInfo.clipURL = [NSURL URLWithString:postAdURLString];
	postAdInfo.mediaTime = [[[MediaTime alloc] init] autorelease];
	postAdInfo.mediaTime.clipBeginMediaTime = 0;
	// set ad length
	postAdInfo.mediaTime.clipEndMediaTime = 45;
	postAdInfo.policy = @"policy for post-roll ad";
	// set ad type
	postAdInfo.type = AdType_Postroll;
	adLinearTime.duration = 0;
	if (![framework scheduleClip:postAdInfo atTime:adLinearTime forType:PlaylistEntryType_Media andGetClipId:&adIndex])
	{
	    [self logFrameworkError];
	}

以下示例演示如何安排前置式广告。
	
	//Example:9 Schedule Pre Roll Ad
	NSString *adURLString = @"http://wamsblureg001orig-hs.chinacloudapp.cn/2e4e7d1f-b72a-4994-a406-810c796fc4fc/The%20Surface%20Movement-m3u8-aapl.ism/Manifest(format=m3u8-aapl)";
	AdInfo *adInfo = [[[AdInfo alloc] init] autorelease];
	adInfo.clipURL = [NSURL URLWithString:adURLString];
	adInfo.mediaTime = [[[MediaTime alloc] init] autorelease];
	adInfo.mediaTime.currentPlaybackPosition = 0;
	adInfo.mediaTime.clipBeginMediaTime = 40; //You could play a portion of an Ad. Yeh!
	adInfo.mediaTime.clipEndMediaTime = 59;
	adInfo.policy = @"policy for pre-roll ad";
	adInfo.appendTo = -1;
	adInfo.type = AdType_Preroll;
	adLinearTime.duration = 0;
	// schedule ad
	if (![framework scheduleClip:adInfo atTime:adLinearTime forType:PlaylistEntryType_Media andGetClipId:&adIndex])
	{
	    [self logFrameworkError];
	}

以下示例演示如何安排中置叠加式广告。
	
	// Example10: Schedule a Mid Roll overlay Ad
	NSString *adURLString = @"https://portalvhdsq3m25bf47d15c.blob.core.chinacloudapi.cn/asset-e47b43fd-05dc-4587-ac87-5916439ad07f/Windows%208_%20Cliffjumpers.mp4?st=2012-11-28T16%3A31%3A57Z&se=2014-11-28T16%3A31%3A57Z&sr=c&si=2a6dbb1e-f906-4187-a3d3-7e517192cbd0&sig=qrXYZBekqlbbYKqwovxzaVZNLv9cgyINgMazSCbdrfU%3D";
	//Create AdInfo instance
	AdInfo *adInfo = [[[AdInfo alloc] init] autorelease];
	adInfo.clipURL = [NSURL URLWithString:adURLString];
	adInfo.mediaTime = [[[MediaTime alloc] init] autorelease];
	adInfo.mediaTime.currentPlaybackPosition = 0;
	adInfo.mediaTime.clipBeginMediaTime = 0;
	// specify ad length
	adInfo.mediaTime.clipEndMediaTime = 20;
	adInfo.policy = @"policy for mid-roll overlay ad";
	adInfo.appendTo = -1;
	// specify ad type
	adInfo.type = AdType_Midroll;
	// specify ad start time & duration
	adLinearTime.startTime = 300;
	adLinearTime.duration = 20;
	// schedule ad            if (![framework scheduleClip:adInfo atTime:adLinearTime forType:PlaylistEntryType_Media andGetClipId:&adIndex])
	{
	    [self logFrameworkError];
	}


 
##另请参阅

[开发视频播放器应用程序](/documentation/articles/media-services-develop-video-players/)

<!---HONumber=Mooncake_0620_2016-->