<properties
    pageTitle="通过 Azure 元数据服务使用计划事件 | Azure"
    description="应对虚拟机上的有影响事件（在其发生之前）。"
    services="virtual-machines-windows, virtual-machines-linux, cloud-services"
    documentationcenter=""
    author="zivraf"
    manager="timlt"
    editor=""
    tags=""
    translationtype="Human Translation" />
<tags
    ms.assetid="28d8e1f2-8e61-4fbe-bfe8-80a68443baba"
    ms.service="virtual-machines-windows"
    ms.devlang="na"
    ms.topic="article"
    ms.tgt_pltfrm="na"
    ms.workload="infrastructure-services"
    ms.date="12/10/2016"
    wacn.date="04/24/2017"
    ms.author="zivr"
    ms.sourcegitcommit="a114d832e9c5320e9a109c9020fcaa2f2fdd43a9"
    ms.openlocfilehash="f1f6761cd213886d99550e6966b5c38300729003"
    ms.lasthandoff="04/14/2017" />

# <a name="azure-metadata-service---scheduled-events-preview"></a>Azure 元数据服务 - 计划事件（预览）

> [AZURE.NOTE] 
> 同意使用条款即可使用预览版。
>

计划事件是 Azure 元数据服务下面的子服务之一，显示有关即将发生的事件（例如，重新启动）的信息，使应用程序可以为其做准备并限制中断。 它可用于所有 Azure 虚拟机类型（包括 PaaS 和 IaaS）。 计划事件给虚拟机时间来执行预防性任务并最大限度地降低事件的影响。 

## <a name="introduction---why-scheduled-events"></a>简介 - 为什么要有计划事件？

使用计划事件，用户客户采取相应步骤来限制对服务的影响。 使用复制技术保持状态的多实例工作负荷可能易受到跨多个实例频繁发生的服务中断的影响。 此类中断可能导致任务开销昂贵（例如，重新生成索引）甚至副本丢失。 在很多其他情况下，使用正常关闭顺序可以提高服务的整体可用性。 例如，完成（或取消）正在提交的事务、将其他任务重新分配给群集中的其他 VM（手动故障转移），从负载均衡器池中删除虚拟机。 有时会通知管理员即将发生的事件，甚至仅仅记录此类事件也会帮助改进托管在云中的应用程序的可维护性。
Azure 元数据服务在以下用例中显示计划事件：
- 平台启动的维护（例如，主机 OS 部署）
- 用户启动的调用（例如，用户重启或重新部署 VM）

## <a name="scheduled-events---the-basics"></a>计划事件 - 基础知识  

Azure 元数据服务会公开在 VM 中使用 REST 终结点运行虚拟机的相关信息。 该信息通过不可路由的 IP 提供，因此不会在 VM 外部公开。

### <a name="scope"></a>作用域
计划事件将显示到云服务中的所有虚拟机或可用性集中的所有虚拟机上。 因此，应查看事件中的“资源”字段以确定哪些 VM 将会受到影响。 

### <a name="discover-the-endpoint"></a>发现终结点
当虚拟机是在虚拟网络 (VNet) 中创建时，可以从不可路由的 IP 169.254.169.254 使用元数据服务。否则，在适用于云服务和经典 VM 的默认情况下，需要通过其他逻辑来发现要使用的终结点。 请参阅此示例，了解如何[发现主机终结点] (https://github.com/azure-samples/virtual-machines-python-scheduled-events-discover-endpoint-for-non-vnet-vm)

### <a name="versioning"></a>版本控制 
元数据服务通过以下格式使用版本控制 API：http://{ip}/metadata/{version}/scheduledevents 建议你的服务使用以下网址提供的最新版本：http://{ip}/metadata/latest/scheduledevents

### <a name="using-headers"></a>使用标头
查询元数据服务时，必须提供以下标头： *Metadata: true*。 

### <a name="enable-scheduled-events"></a>启用计划事件
第一次调用计划事件时，Azure 会在虚拟机上隐式启用该功能。 因此，第一次调用时应该会延迟响应最多两分钟。

### <a name="testing-your-logic-with-user-initiated-operations"></a>通过用户启动的操作对逻辑进行测试
若要测试逻辑，可以使用 Azure 门户预览、API、CLI 或 PowerShell 启动生成计划事件的操作。 重新启动虚拟机会生成事件类型为“重新启动”的计划事件。 重新部署虚拟机会生成事件类型为“重新部署”的计划事件。
用户启动的操作在这两种情况下都需要更长的时间来完成，因为计划事件为应用程序留出更多的时间来正常关闭。 

## <a name="using-the-api"></a>使用 API

### <a name="query-for-events"></a>查询事件
进行以下调用即可查询计划事件

    curl -H Metadata:true http://169.254.169.254/metadata/latest/scheduledevents

响应包含计划事件的数组。 数组为空意味着目前没有计划事件。
如果有计划事件，响应会包含事件的数组： 

    {
     "DocumentIncarnation":{IncarnationID},
     "Events":[
          {
                "EventId":{eventID},
                "EventType":"Reboot" | "Redeploy" | "Freeze",
                "ResourceType":"VirtualMachine",
                "Resources":[{resourceName}],
                "EventStatus":"Scheduled" | "Started",
                "NotBefore":{timeInUTC},              
         }
     ]
    }

EventType 捕获对虚拟机的预期影响，其中：
- 冻结：虚拟机将计划暂停几秒。 对内存、打开文件或网络连接没有影响
- Reboot：虚拟机计划重启（内存将被擦除）。
- Redeploy：虚拟机计划移到另一节点（临时磁盘会丢失）。 

对事件进行计划以后 (Status = Scheduled)，Azure 会共享在其后可以启动事件的时间（在“NotBefore”字段中指定）。

### <a name="starting-an-event-expedite"></a>启动事件（加快）

了解即将发生的事件并完成正常关闭逻辑后，可以通过进行 **POST** 调用指示 Azure 更快地移动（如果可能） 

## <a name="powershell-sample"></a>PowerShell 示例 

以下示例将读取计划事件的元数据服务器并审核事件。

    # How to get scheduled events 
    function GetScheduledEvents($uri)
    {
        $scheduledEvents = Invoke-RestMethod -Headers @{"Metadata"="true"} -URI $uri -Method get
        $json = ConvertTo-Json $scheduledEvents
        Write-Host "Received following events: `n" $json
        return $scheduledEvents
    }

    # How to approve a scheduled event
    function ApproveScheduledEvent($eventId, $uri)
    {    
        # Create the Scheduled Events Approval Json
        $startRequests = [array]@{"EventId" = $eventId}
        $scheduledEventsApproval = @{"StartRequests" = $startRequests} 
        $approvalString = ConvertTo-Json $scheduledEventsApproval

        Write-Host "Approving with the following: `n" $approvalString

        # Post approval string to scheduled events endpoint
        Invoke-RestMethod -Uri $uri -Headers @{"Metadata"="true"} -Method POST -Body $approvalString
    }

    # Add logic relevant to your service here
    function HandleScheduledEvents($scheduledEvents)
    {

    }

    ######### Sample Scheduled Events Interaction #########

    # Set up the scheduled events uri for VNET enabled VM
    $localHostIP = "169.254.169.254"
    $scheduledEventURI = 'http://{0}/metadata/latest/scheduledevents' -f $localHostIP 

    # Get the document
    $scheduledEvents = GetScheduledEvents $scheduledEventURI

    # Handle events however is best for your service
    HandleScheduledEvents $scheduledEvents

    # Approve events when ready (optional)
    foreach($event in $scheduledEvents.Events)
    {
        Write-Host "Current Event: `n" $event
        $entry = Read-Host "`nApprove event? Y/N"
        if($entry -eq "Y" -or $entry -eq "y")
        {
        ApproveScheduledEvent $event.EventId $scheduledEventURI 
        }
    }

## <a name="c-sample"></a>C\# 示例 
以下示例是公开用于与元数据服务进行通信的 API 的客户端代码

       public class ScheduledEventsClient
        {
            private readonly string scheduledEventsEndpoint;
            private readonly string defaultIpAddress = "169.254.169.254"; 

            public ScheduledEventsClient()
            {
                scheduledEventsEndpoint = string.Format("http://{0}/metadata/latest/scheduledevents", defaultIpAddress);
            }
            /// Retrieve Scheduled Events 
            public string GetDocument()
            {
                Uri cloudControlUri = new Uri(scheduledEventsEndpoint);
                using (var webClient = new WebClient())
                {
                    webClient.Headers.Add("Metadata", "true");
                    return webClient.DownloadString(cloudControlUri);
                }   
            }

            /// Issues a post request to the scheduled events endpoint with the given json string
            public void PostResponse(string jsonPost)
            {
                using (var webClient = new WebClient())
                {
                    webClient.Headers.Add("Content-Type", "application/json");
                    webClient.UploadString(scheduledEventsEndpoint, jsonPost);
                }
            }
        }

无法使用以下数据结构分析计划事件 

        public class ScheduledEventsDocument
        {
            public List<CloudControlEvent> Events { get; set; }
        }

        public class CloudControlEvent
        {
            public string EventId { get; set; }
            public string EventStatus { get; set; }
            public string EventType { get; set; }
            public string ResourceType { get; set; }
            public List<string> Resources { get; set; }
            public DateTime NoteBefore { get; set; }
        }

        public class ScheduledEventsApproval
        {
            public List<StartRequest> StartRequests = new List<StartRequest>();
        }

        public class StartRequest
        {
            [JsonProperty("EventId")]
            private string eventId;

            public StartRequest(string eventId)
            {
                this.eventId = eventId;
            }
        }

一个使用客户端检索、处理和确认事件的示例程序：   

    public class Program
        {
        static ScheduledEventsClient client;
        static void Main(string[] args)
        {
            while (true)
            {
                client = new ScheduledEventsClient();
                string json = client.GetDocument();
                ScheduledEventsDocument scheduledEventsDocument = JsonConvert.DeserializeObject<ScheduledEventsDocument>(json);

                HandleEvents(scheduledEventsDocument.Events);

                // Wait for user response
                Console.WriteLine("Press Enter to approve executing events\n");
                Console.ReadLine();

                // Approve events
                ScheduledEventsApproval scheduledEventsApprovalDocument = new ScheduledEventsApproval();
                foreach (CloudControlEvent ccevent in scheduledEventsDocument.Events)
                {
                    scheduledEventsApprovalDocument.StartRequests.Add(new StartRequest(ccevent.EventId));
                }
                if (scheduledEventsApprovalDocument.StartRequests.Count > 0)
                {
                    // Serialize using Newtonsoft.Json
                    string approveEventsJsonDocument =
                        JsonConvert.SerializeObject(scheduledEventsApprovalDocument);

                    Console.WriteLine($"Approving events with json: {approveEventsJsonDocument}\n");
                    client.PostResponse(approveEventsJsonDocument);
                }

                Console.WriteLine("Complete. Press enter to repeat\n\n");
                Console.ReadLine();
                Console.Clear();
            }
        }

        private static void HandleEvents(List<CloudControlEvent> events)
        {
            // Add logic for handling events here
        }
    }

## <a name="python-sample"></a>Python 示例 

    #!/usr/bin/python

    import json
    import urllib2
    import socket
    import sys

    metadata_url="http://169.254.169.254/metadata/latest/scheduledevents"
    headers="{Metadata:true}"
    this_host=socket.gethostname()

    def get_scheduled_events():
       req=urllib2.Request(metadata_url)
       req.add_header('Metadata','true')
       resp=urllib2.urlopen(req)
       data=json.loads(resp.read())
       return data

    def handle_scheduled_events(data):
        for evt in data['Events']:
            eventid=evt['EventId']
            status=evt['EventStatus']
            resources=evt['Resources'][0]
            eventype=evt['EventType']
            restype=evt['ResourceType']
            notbefore=evt['NotBefore'].replace(" ","_")
            if this_host in evt['Resources'][0]:
                print "+ Scheduled Event. This host is scheduled for " + eventype + " not before " + notbefore
                print "++ Add you logic here"

    def main():
       data=get_scheduled_events()
       handle_scheduled_events(data)

    if __name__ == '__main__':
      main()
      sys.exit(0)


## <a name="next-steps"></a>后续步骤 
[Azure 中虚拟机的计划内维护](/documentation/articles/virtual-machines-linux-planned-maintenance/)
<!--Update_Description: wording update-->