<properties linkid="develop-media-services-tutorials-smooth-streaming-plugin-for-open-source-media-framework" urlDisplayName="Smooth Streaming Plugin" pageTitle="Smooth Streaming Plugin for the Open Source Media Framework" metaKeywords="" description="Learn how to use the Azure Media Services Smooth Streaming plugin for the Adobe Open Source Media Framework." metaCanonical="" services="media-services" documentationCenter="" title="How to Use the Microsoft Smooth Streaming Plugin for the Adobe Open Source Media Framework" authors="" solutions="" manager="" editor="" />

如何使用适用于 Adobe Open Source Media Framework 的 Microsoft 平滑流式处理插件
==============================================================================

概述
----

适用于 Open Source Media Framework 2.0 的 Microsoft 平滑流式处理插件 (SS for OSMF) 扩展了 OSMF 的默认功能，并在新的和现有的 OSMF 播放器中添加了 Microsoft 平滑流式处理内容播放功能。该插件还为 Strobe Media Playback (SMP) 添加了平滑流式处理播放功能。

SS for OSMF 包括两个版本的插件：

-   适用于 OSMF 的静态平滑流式处理插件 (.swc)

-   适用于 OSMF 的动态平滑流式处理插件 (.swf)

本文档假设读者具有 OSMF 和 OSMF 插件方面的一般实践知识。有关 OSMF 的详细信息，请参阅 [OSMF 官方网站](http://osmf.org/)上的文档。

### 适用于 OSMF 2.0 的平滑流式处理插件

该插件支持通过以下功能加载和播放按需平滑流式处理内容：

-   按需平滑流式处理播放（播放、暂停、定位、停止）
-   实时平滑流式处理播放（播放）
-   实时 DVR 功能（暂停、定位、DVR 播放、现场直播）
-   支持视频编解码器 - H.264
-   支持音频编解码器 - AAC
-   通过 OSMF 内置 API 切换多种音频语言
-   通过 OSMF 内置 API 选择最佳播放质量
-   通过 OSMF 字幕插件实现 Sidecar 隐藏字幕
-   AdobeÂ® FlashÂ® Player 10.2 或更高版本。
-   此版本仅支持 OSMF 2.0。

不支持以下功能：

-   VC-1 和 WMA 编解码器
-   内容保护 (PlayReady)
-   文本和稀疏曲目
-   特技（慢动作、快进和倒带）

下面是已知问题的列表：

-   播放包含 48KHz 音轨的平滑流式处理内容时会出现问题。Flash 运行时在呈现 48KHz 音频内容时会出现一个已知问题。由于此问题，使用 48KHz 设置编码的平滑流式处理内容可能无法正常播放。有关详细信息，请参阅：[使用 Flash Player](http://forums.adobe.com/message/4483498#4483498) 和 [Adobe Flash Player 11.3 - Bug 3210964](https://bugbase.adobe.com/index.cfm?event=bug&id=3210964)。
-   在一个页面上播放多种平滑流式处理内容可能会导致出现问题。这是 OSMF 存在的已知问题。
-   播放 Stage Video 内容可能会导致出现问题，在某些计算机上不显示视频。解决方法是禁用硬件加速或 Stage Video。

加载插件
--------

可静态（在编译时）或动态（在运行时）加载 OSMF 插件。适用于 OSMF 的平滑流式处理插件的下载内容包括动态和静态版本。

-   静态加载：静态加载需要使用一个静态库 (SWC) 文件。静态插件将添加为对项目的引用，并在编译时合并到最终输出文件中。

-   动态加载：动态加载需要使用一个预编译的 (SWF) 文件。动态插件在运行时中加载，不包含在项目输出中。（编译的输出）可以使用 HTTP 和 FILE 协议加载动态插件。

有关静态和动态加载的详细信息，请参阅官方的 [OSMF 插件页](http://osmf.org/dev/osmf/OtherPDFs/osmf_plugin_dev_guide.pdf)。

### SS for OSMF 静态加载

以下代码段演示如何静态加载 SS for OSMF 插件以及如何使用 OSMF MediaFactory 类播放基本视频。在包含 SS for OSMF 代码之前，请确保项目引用包含“MSAdaptiveStreamingPlugin-v1.0.3-osmf2.0.swc”静态插件。

``` {}
package 
{
    
    import com.microsoft.azure.media.AdaptiveStreamingPluginInfo;
    
    import flash.display.*;
    import org.osmf.media.*;
    import org.osmf.containers.MediaContainer;
    import org.osmf.events.MediaErrorEvent;
    import org.osmf.events.MediaFactoryEvent;
    import org.osmf.events.MediaPlayerStateChangeEvent;
    import org.osmf.layout.*;
    
    
    
    [SWF(width="1024", height="768", backgroundColor='#405050', frameRate="25")]
    public class TestPlayer extends Sprite
    {        
        public var _container:MediaContainer;
        public var _mediaFactory:DefaultMediaFactory;
        private var _mediaPlayerSprite:MediaPlayerSprite;
        

        public function TestPlayer( )
        {
            stage.quality = StageQuality.HIGH;

            initMediaPlayer();

        }
    
        private function initMediaPlayer():void
        {
        
            // Create the container (sprite) for managing display and layout
            _mediaPlayerSprite = new MediaPlayerSprite();    
            _mediaPlayerSprite.addEventListener(MediaErrorEvent.MEDIA_ERROR, onPlayerFailed);
            _mediaPlayerSprite.addEventListener(MediaPlayerStateChangeEvent.MEDIA_PLAYER_STATE_CHANGE, onPlayerStateChange);
            _mediaPlayerSprite.scaleMode = ScaleMode.NONE;
            _mediaPlayerSprite.width = stage.stageWidth;
            _mediaPlayerSprite.height = stage.stageHeight;
            //Adds the container to the stage
            addChild(_mediaPlayerSprite);
            
            // Create a mediafactory instance
            _mediaFactory = new DefaultMediaFactory();
            
            // Add the listeners for PLUGIN_LOADING
            _mediaFactory.addEventListener(MediaFactoryEvent.PLUGIN_LOAD,onPluginLoaded);
            _mediaFactory.addEventListener(MediaFactoryEvent.PLUGIN_LOAD_ERROR, onPluginLoadFailed );
            
            // Load the plugin class 
            loadAdaptiveStreamingPlugin( );  
            
        }
        
        private function loadAdaptiveStreamingPlugin( ):void
        {
            var pluginResource:MediaResourceBase;
            
            pluginResource = new PluginInfoResource(new AdaptiveStreamingPluginInfo( )); 
            _mediaFactory.loadPlugin( pluginResource ); 
        }
        
        private function onPluginLoaded( event:MediaFactoryEvent ):void
        {
            // The plugin is loaded successfully.
            // Your web server needs to host a valid crossdomain.xml file to allow plugin to download Smooth Streaming files.
        loadMediaSource("http://devplatem.vo.msecnd.net/Sintel/Sintel_H264.ism/manifest")
        
        }
        
        private function onPluginLoadFailed( event:MediaFactoryEvent ):void
        {
            // The plugin is failed to load ...
        }
        
        
        private function onPlayerStateChange(event:MediaPlayerStateChangeEvent) :void
        {
            var state:String;
            
            state =  event.state;
            
            switch (state)
            {
                case MediaPlayerState.LOADING: 
                    
                    // A new source is started to load.
                    
                    break;
                
                case  MediaPlayerState.READY :   
                    // Add code to deal with Player Ready when it is hit the first load after a source is loaded. 
                    
                    break;
                
                case MediaPlayerState.BUFFERING :
                    
                    break;
                
                case  MediaPlayerState.PAUSED :
                    break;      
                // other states ...          
            }
        }
        
        private function onPlayerFailed(event:MediaErrorEvent) :void
        {
            // Media Player is failed .           
        }
        
        private function loadMediaSource(sourceURL :String):void 
        {
            // Take an URL of SmoothStreamingSource's manifest and add it to the page.
            
            var resource:URLResource= new URLResource( sourceURL );
            
            var element:MediaElement = _mediaFactory.createMediaElement( resource );
            _mediaPlayerSprite.scaleMode = ScaleMode.LETTERBOX;
            _mediaPlayerSprite.width = stage.stageWidth;
            _mediaPlayerSprite.height = stage.stageHeight;
            
            // Add the media element
            _mediaPlayerSprite.media = element;
        }     
        
    }
}
```

### SS for OSMF 动态加载

以下代码段演示如何动态加载 SS for OSMF 插件以及如何使用 OSMF MediaFactory 类播放基本视频。在包含 SS for OSMF 代码之前，如果要使用 FILE 协议进行加载，请将“MSAdaptiveStreamingPlugin-v1.0.3-osmf2.0.swf”动态插件复制到项目文件夹；如果要进行 HTTP 加载，请将该插件复制到 Web 服务器下。不需要在项目引用中包含“MSAdaptiveStreamingPlugin-v1.0.3-osmf2.0.swc”。

``` {}
package 
{
    
    import flash.display.*;
    import org.osmf.media.*;
    import org.osmf.containers.MediaContainer;
    import org.osmf.events.MediaErrorEvent;
    import org.osmf.events.MediaFactoryEvent;
    import org.osmf.events.MediaPlayerStateChangeEvent;
    import org.osmf.layout.*;
    import flash.events.Event;
    import flash.system.Capabilities;

    
    //Sets the size of the SWF
    
    [SWF(width="1024", height="768", backgroundColor='#405050', frameRate="25")]
    public class TestPlayer extends Sprite
    {        
        public var _container:MediaContainer;
        public var _mediaFactory:DefaultMediaFactory;
        private var _mediaPlayerSprite:MediaPlayerSprite;
        
        
        public function TestPlayer( )
        {
            stage.quality = StageQuality.HIGH;
            initMediaPlayer();
        }
        
        private function initMediaPlayer():void
        {
            
            // Create the container (sprite) for managing display and layout
            _mediaPlayerSprite = new MediaPlayerSprite();    
            _mediaPlayerSprite.addEventListener(MediaErrorEvent.MEDIA_ERROR, onPlayerFailed);
            _mediaPlayerSprite.addEventListener(MediaPlayerStateChangeEvent.MEDIA_PLAYER_STATE_CHANGE, onPlayerStateChange);

            //Adds the container to the stage
            addChild(_mediaPlayerSprite);
            
            // Create a mediafactory instance
            _mediaFactory = new DefaultMediaFactory();
            
            // Add the listeners for PLUGIN_LOADING
            _mediaFactory.addEventListener(MediaFactoryEvent.PLUGIN_LOAD,onPluginLoaded);
            _mediaFactory.addEventListener(MediaFactoryEvent.PLUGIN_LOAD_ERROR, onPluginLoadFailed );
            
            // Load the plugin class 
            loadAdaptiveStreamingPlugin( );  
            
        }
        
        private function loadAdaptiveStreamingPlugin( ):void
        {
            var pluginResource:MediaResourceBase;
            var adaptiveStreamingPluginUrl:String;

            // Your dynamic plugin web server needs to host a valid crossdomain.xml file to allow loading plugins.

            adaptiveStreamingPluginUrl = "http://yourdomain/MSAdaptiveStreamingPlugin-v1.0.3-osmf2.0.swf";
            pluginResource = new URLResource(adaptiveStreamingPluginUrl);
            _mediaFactory.loadPlugin( pluginResource ); 

        }
        
        private function onPluginLoaded( event:MediaFactoryEvent ):void
        {
            // The plugin is loaded successfully.

            // Your web server needs to host a valid crossdomain.xml file to allow plugin to download Smooth Streaming files.

    loadMediaSource("http://devplatem.vo.msecnd.net/Sintel/Sintel_H264.ism/manifest")
        }
        
        private function onPluginLoadFailed( event:MediaFactoryEvent ):void
        {
            // The plugin is failed to load ...
        }
        
        
        private function onPlayerStateChange(event:MediaPlayerStateChangeEvent) :void
        {
            var state:String;
            
            state =  event.state;
            
            switch (state)
            {
                case MediaPlayerState.LOADING: 
                    
                    // A new source is started to load.
                    
                    break;
                
                case  MediaPlayerState.READY :   
                    // Add code to deal with Player Ready when it is hit the first load after a source is loaded. 
                    
                    break;
                
                case MediaPlayerState.BUFFERING :
                    
                    break;
                
                case  MediaPlayerState.PAUSED :
                    break;      
                // other states ...          
            }
        }
        
        private function onPlayerFailed(event:MediaErrorEvent) :void
        {
            // Media Player is failed .           
        }
        
        private function loadMediaSource(sourceURL :String):void 
        {
            // Take an URL of SmoothStreamingSource's manifest and add it to the page.
            
            var resource:URLResource= new URLResource( sourceURL );
            
            var element:MediaElement = _mediaFactory.createMediaElement( resource );
            _mediaPlayerSprite.scaleMode = ScaleMode.LETTERBOX;
            _mediaPlayerSprite.width = stage.stageWidth;
            _mediaPlayerSprite.height = stage.stageHeight;
            // Add the media element
            _mediaPlayerSprite.media = element;
        }     
        
    }
}
```

Strobe Media Playback 与 SS OSMF 动态插件
-----------------------------------------

适用于 OSMF 的平滑流式处理动态插件与 [Strobe Media Playback (SMP)](http://osmf.org/strobe_mediaplayback.html) 兼容。你可以使用 SS for OSMF 插件向 SMP 添加平滑流式处理内容播放功能。为此，请在进行 HTTP 加载时，使用以下步骤将“MSAdaptiveStreamingPlugin-v1.0.3-osmf2.0.swf”复制到 Web 服务器下：

1.  打开 [Strobe Media Playback 设置页](http://osmf.org/dev/2.0gm/setup.html)。
2.  将 src 设置为某个平滑流式处理源（例如 http://devplatem.vo.msecnd.net/Sintel/Sintel\_H264.ism/manifest）
3.  进行所需的配置更改，然后单击“Preview and Update”（预览并更新）。

    **注意**：你的内容 Web 服务器需要有效的 crossdomain.xml。

4.  使用常用的文本编辑器将该代码复制并粘贴到一个简单的 HTML 页，如以下示例所示：

        <html>
        <body>
        <object width="920" height="640"> 
        <param name="movie" value="http://osmf.org/dev/2.0gm/StrobeMediaPlayback.swf"></param>
        <param name="flashvars" value="src=http://devplatem.vo.msecnd.net/Sintel/Sintel_H264.ism/manifest &autoPlay=true"></param>
        <param name="allowFullScreen" value="true"></param>
        <param name="allowscriptaccess" value="always"></param>
        <param name="wmode" value="direct"></param>
        <embed src="http://osmf.org/dev/2.0gm/StrobeMediaPlayback.swf" 
            type="application/x-shockwave-flash" 
            allowscriptaccess="always" 
            allowfullscreen="true" 
            wmode="direct" 
            width="920" 
            height="640" 
            flashvars=" src=http://devplatem.vo.msecnd.net/Sintel/Sintel_H264.ism/manifest&autoPlay=true">
        </embed>
        </object>
        </body>
        </html>

5.  将平滑流式处理 OSMF 插件添加到 embed 代码中，然后保存。

		<html>
		<object width="920" height="640"> 
		<param name="movie" value="http://osmf.org/dev/2.0gm/StrobeMediaPlayback.swf"></param>
		<param name="flashvars" value="src=http://devplatem.vo.msecnd.net/Sintel/Sintel_H264.ism/manifest&autoPlay=true&plugin_AdaptiveStreamingPlugin=http://yourdomain/MSAdaptiveStreamingPlugin-v1.0.3-osmf2.0.swf&AdaptiveStreamingPlugin_retryLive=true&AdaptiveStreamingPlugin_retryInterval=10"></param>
		<param name="allowFullScreen" value="true"></param>
		<param name="allowscriptaccess" value="always"></param>
		<param name="wmode" value="direct"></param>
		<embed src="http://osmf.org/dev/2.0gm/StrobeMediaPlayback.swf" 
    		type="application/x-shockwave-flash" 
    		allowscriptaccess="always" 
    		allowfullscreen="true" 
    		wmode="direct" 
    		width="920" 
    		height="640" 
    		flashvars="src=http://devplatem.vo.msecnd.net/Sintel/Sintel_H264.ism/manifest&autoPlay=true&plugin_AdaptiveStreamingPlugin=http://yourdomain/MSAdaptiveStreamingPlugin-v1.0.3-osmf2.0.swf&AdaptiveStreamingPlugin_retryLive=true&AdaptiveStreamingPlugin_retryInterval=10">
		</embed>
		</object>
		</html>

6.  保存 HTML 页，然后发布到 Web 服务器。使用你最常用的、已启用 FlashÂ® Player 的 Internet 浏览器（Internet Explorer、Chrome、Firefox 等）浏览到已发布的网页。
7.  在 AdobeÂ® FlashÂ® Player 中欣赏平滑流式处理内容。

有关一般性 OSMF 开发的详细信息，请参阅官方的 [OSMF 开发页](http://osmf.org/resources.html)。

