## <a name="what-is"> </a>什么是队列存储？

Azure 队列存储是一项用于存储大量消息的服务，
用户可以通过经验证的呼叫，使用 HTTP 或 HTTPS 从
世界任何地方访问这些消息。一条队列消息的
大小最多可达 64 KB，一个队列中可以包含数百万条消息，最多可达
存储帐户的总容量限制。存储帐户可以容纳高达 200 TB 的 Blob、队列和表数据。请参阅 [Azure 存储的可伸缩性和性能目标](http://msdn.microsoft.com/zh-cn/library/dn249410.aspx)。

队列存储的常见用途包括：

-   <span>创建积压工作以进行异步处理</span>
-   将消息从 Azure Web 角色传递到 Azure
    辅助角色

## <a name="concepts"> </a>概念

队列服务包含以下组件：

![Queue1](./media/howto-queue-storage/queue1.png)


- **URL 格式：**可使用以下 URL 格式对队列进行寻址：   
	http://`<storage account>`.queue.core.chinacloudapi.cn/`<queue>` 
      
可使用以下 URL 访问示意图中的某个队列：  
	http://myaccount.queue.core.chinacloudapi.cn/imagesToDownload

-**Storage 帐户：**对 Azure 存储空间进行的所有访问都要通过存储帐户完成。请参阅 [Azure 存储的可伸缩性和性能目标](http://msdn.microsoft.com/zh-cn/library/dn249410.aspx)。

- **队列：**一个队列包含一组消息。所有消息都必须在队列中。

- **消息：**一条消息（不管采用何种格式）的最大大小为 64 KB。


<!--HONumber=41-->
