<properties 
	pageTitle="轮询长时运行的操作" 
	description="本主题展示了如何轮询长时运行的操作。" 
	services="media-services" 
	documentationCenter="" 
	authors="juliako" 
	manager="dwrede" 
	editor=""/>

<tags
	ms.service="media-services"
 	ms.date="04/18/2016" 
	wacn.date="06/20/2016"/>


#使用 Azure 媒体服务传送实时流

##概述

Azure 媒体服务提供了相应的 API 用来请求媒体服务启动操作（例如创建、启动、停止或删除频道）。这些操作是长时运行的。

媒体服务 .NET SDK 提供了用来发送请求并等待操作完成的 API（在内部，这些 API 以特定的时间间隔轮询操作进度）。例如，当调用 channel.Start() 时，该方法将在频道启动后返回。你还可以使用异步版本：await channel.StartAsync()（有关基于任务的异步模式的信息，请参阅 [TAP](https://msdn.microsoft.com/zh-cn/library/hh873175(v=vs.110).aspx))。发送操作请求并且在操作完成之前一直轮询操作状态的 API 称作“轮询方法”。建议为富客户端应用程序和/或有状态服务使用这些方法（特别是异步版本）。

某些情况下，应用程序不能等待长时运行的 http 请求并且希望手动轮询操作进度。一个典型的示例是与无状态 web 服务进行交互的浏览器：当浏览器请求创建频道时，web 服务会启动一个长时运行的操作并将操作 ID 返回到浏览器。然后，浏览器可以根据该 ID 询问 web 服务来获取操作状态。媒体服务 .NET SDK 提供了非常适用于此情况的 API。这些 API 称为“非轮询方法”。
"非轮询方法"具有以下命名模式：Send*OperationName*Operation（例如，SendCreateOperation）。Send*OperationName*Operation 方法返回 **IOperation** 对象；返回的对象包含可以用来跟踪操作的信息。Send*OperationName*OperationAsync 方法将返回 **Task<IOperation>**。

当前，以下类支持非轮询方法：**Channel**、**StreamingEndpoint** 和 **Program**。

若要轮询操作状态，请对 **OperationBaseCollection** 类使用 **GetOperation** 方法。使用以下时间间隔来检查操作状态：对于 **Channel** 和 **StreamingEndpoint** 操作，使用 30 秒；对于 **Program** 操作，使用 10 秒。


##示例

以下示例定义了一个名为 **ChannelOperations** 的类。可以将该类定义用作你的 web 服务类定义的起点。为简单起见，以下示例使用了方法的非异步版本。

示例还展示了客户端可以如何使用该类。

###ChannelOperations 类定义

	/// <summary> 
	/// The ChannelOperations class only implements 
	/// the Channel’s creation operation. 
	/// </summary> 
	public class ChannelOperations
	{
	    // Read values from the App.config file.
	    private static readonly string _mediaServicesAccountName =
	        ConfigurationManager.AppSettings["MediaServicesAccountName"];
	    private static readonly string _mediaServicesAccountKey =
	        ConfigurationManager.AppSettings["MediaServicesAccountKey"];
	
		private static readonly String _defaultScope = "urn:WindowsAzureMediaServices";

		// Azure China uses a different API server and a different ACS Base Address from the Global.
		private static readonly String _chinaApiServerUrl = "https://wamsshaclus001rest-hs.chinacloudapp.cn/API/";
		private static readonly String _chinaAcsBaseAddressUrl = "https://wamsprodglobal001acs.accesscontrol.chinacloudapi.cn";

	    // Field for service context.
	    private static CloudMediaContext _context = null;
	    private static MediaServicesCredentials _cachedCredentials = null;
	
	    public ChannelOperations()
	    {
	            _cachedCredentials = new MediaServicesCredentials(_mediaServicesAccountName,
	                _mediaServicesAccountKey, _defaultScope, _chinaAcsBaseAddressUrl);
	
				// Create the API server Uri
				_apiServer = new Uri(_chinaApiServerUrl);

	            _context = new CloudMediaContext(_apiServer, _cachedCredentials);    }
	
	    /// <summary>  
	    /// Initiates the creation of a new channel.  
	    /// </summary>  
	    /// <param name="channelName">Name to be given to the new channel</param>  
	    /// <returns>  
	    /// Operation Id for the long running operation being executed by Media Services. 
	    /// Use this operation Id to poll for the channel creation status. 
	    /// </returns> 
	    public string StartChannelCreation(string channelName)
	    {
	        var operation = _context.Channels.SendCreateOperation(
	            new ChannelCreationOptions
	            {
	                Name = channelName,
	                Input = CreateChannelInput(),
	                Preview = CreateChannelPreview(),
	                Output = CreateChannelOutput()
	            });
	
	        return operation.Id;
	    }
	
	    /// <summary> 
	    /// Checks if the operation has been completed. 
	    /// If the operation succeeded, the created channel Id is returned in the out parameter.
	    /// </summary> 
	    /// <param name="operationId">The operation Id.</param> 
	    /// <param name="channel">
	    /// If the operation succeeded, 
	    /// the created channel Id is returned in the out parameter.</param>
	    /// <returns>Returns false if the operation is still in progress; otherwise, true.</returns> 
	    public bool IsCompleted(string operationId, out string channelId)
	    {
	        IOperation operation = _context.Operations.GetOperation(operationId);
	        bool completed = false;
	
	        channelId = null;
	
	        switch (operation.State)
	        {
	            case OperationState.Failed:
	                // Handle the failure. 
	                // For example, throw an exception. 
					// Use the following information in the exception: operationId, operation.ErrorMessage.
	                break;
	            case OperationState.Succeeded:
	                completed = true;
	                channelId = operation.TargetEntityId;
	                break;
	            case OperationState.InProgress:
	                completed = false;
	                break;
	        }
	        return completed;
	    }
	
	
	    private static ChannelInput CreateChannelInput()
	    {
	        return new ChannelInput
	        {
	            StreamingProtocol = StreamingProtocol.RTMP,
	            AccessControl = new ChannelAccessControl
	            {
	                IPAllowList = new List<IPRange>
	                {
	                    new IPRange
	                    {
	                        Name = "TestChannelInput001",
	                        Address = IPAddress.Parse("0.0.0.0"),
	                        SubnetPrefixLength = 0
	                    }
	                }
	            }
	        };
	    }
	
	    private static ChannelPreview CreateChannelPreview()
	    {
	        return new ChannelPreview
	        {
	            AccessControl = new ChannelAccessControl
	            {
	                IPAllowList = new List<IPRange>
	                {
	                    new IPRange
	                    {
	                        Name = "TestChannelPreview001",
	                        Address = IPAddress.Parse("0.0.0.0"),
	                        SubnetPrefixLength = 0
	                    }
	                }
	            }
	        };
	    }
	
	    private static ChannelOutput CreateChannelOutput()
	    {
	        return new ChannelOutput
	        {
	            Hls = new ChannelOutputHls { FragmentsPerSegment = 1 }
	        };
	    }
	}

###客户端代码

	ChannelOperations channelOperations = new ChannelOperations();
	string opId = channelOperations.StartChannelCreation("MyChannel001");
	
	string channelId = null;
	bool isCompleted = false;
	
	while (isCompleted == false)
	{
	    System.Threading.Thread.Sleep(TimeSpan.FromSeconds(30));
	    isCompleted = channelOperations.IsCompleted(opId, out channelId);
	}
	
	// If we got here, we should have the newly created channel id.
	Console.WriteLine(channelId);
 



<!---HONumber=Mooncake_0613_2016-->