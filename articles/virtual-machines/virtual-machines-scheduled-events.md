<properties
    pageTitle="通过 Azure 元数据服务使用计划事件 | Azure"
    description="应对虚拟机上的有影响事件（在其发生之前）。"
    services="virtual-machines-windows, virtual-machines-linux, cloud-services"
    documentationcenter=""
    author="zivraf"
    manager="timlt"
    editor=""
    tags="" />
<tags
    ms.assetid="28d8e1f2-8e61-4fbe-bfe8-80a68443baba"
    ms.service="virtual-machines-windows"
    ms.devlang="na"
    ms.topic="article"
    ms.tgt_pltfrm="na"
    ms.workload="infrastructure-services"
    ms.date="12/10/2016"
    wacn.date="02/20/2017"
    ms.author="zivr" />  


# Azure 元数据服务 - 计划事件

Azure 元数据服务用于发现 Azure 中托管的虚拟机的相关信息。在公开的类别中，其中一个类别是“计划事件”，用于显示即将发生的事件（例如重启）的相关信息，使应用程序有所准备，限制中断情况的发生。该类别适用于所有 Azure 虚拟机类型，包括 PaaS 和 IaaS。该服务为虚拟机提供了执行预防性任务的时间，将事件的影响降到最低。例如，服务可能会清空会话、选举新领导，或者在观察到某个实例计划重启后复制数据，避免中断。


## 简介 - 为何使用计划事件？

使用计划事件可以了解（发现）那些可能会影响虚拟机可用性的即将发生的事件，从而采取主动操作，限制其对服务的影响。使用复制技术来维持状态的多实例工作负荷可能容易受多个实例中频繁发生的中断的影响。此类中断可能导致任务开销昂贵（例如，重新生成索引）甚至副本丢失。在很多其他情况下，使用正常关闭顺序可以提高服务的整体可用性。例如，完成（或取消）正在提交的事务、将其他任务重新分配给群集中的其他 VM（手动故障转移），从负载均衡器池中删除虚拟机。在有些情况下，将即将发生的事件通知管理员（或者仅记录此类事件）即可改进在云中托管的应用程序的可维护性。

在以下用例中，Azure 元数据服务会显示计划事件：
-   平台启动的“有影响”维护（例如，主机 OS 推出）
-   平台启动的“无影响”维护（例如，VM 就地迁移）
-   交互式调用（例如，用户重新启动或重新部署 VM）


## 计划事件 - 基础知识  

Azure 元数据服务会公开在 VM 中使用 REST 终结点运行虚拟机的相关信息。该信息通过不可路由的 IP 提供，因此不会在 VM 外部公开。

### 作用域 
计划事件显示给云服务中的所有虚拟机，或者显示给可用性集中的所有虚拟机。因此，应查看事件中的“资源”字段，确定哪些 VM 会受影响。

### 发现终结点
如果虚拟机是在虚拟网络 \(VNet\) 中创建的，可通过不可路由的 IP 169.254.169.254 使用元数据服务

如果虚拟机用于云服务 \(PaaS\)，可通过注册表发现元数据服务终结点。

    {HKEY_LOCAL_MACHINE\Software\Microsoft\Microsoft Azure\DeploymentManagement}

### 版本控制 
元数据服务按以下格式使用带版本的 API：http://{ip}/metadata/{version}/scheduledevents。建议让服务使用 http://{ip}/metadata/latest/scheduledevents 中提供的最新版本

### 使用标头
查询元数据服务时，必须提供以下标头：*Metadata: true*。

### 启用计划事件
第一次调用计划事件时，Azure 会在虚拟机上隐式启用此功能。因此，首次调用会出现响应延迟的现象，延迟时间最长可达一分钟。

## 使用 API

### 查询事件
进行以下调用即可查询计划事件

	curl -H Metadata:true http://169.254.169.254/metadata/latest/scheduledevents

响应包含计划事件的数组。数组为空意味着目前没有计划事件。如果有计划事件，响应会包含事件的数组：

	{
     "Events":[
          {
                "EventId":{eventID},
                "EventType":"Reboot" | "Redeploy" | "Pause",
                "ResourceType":"VirtualMachine",
                "Resources":[{resourceName}],
                "EventStatus":"Scheduled" | "Started",
                "NotBefore":{timeInUTC},              
         }
     ]
	}

EventType 可捕获对虚拟机的预期影响，其中：
- Pause：虚拟机计划暂停数秒。对内存、打开的文件或网络连接没有影响
- Reboot：虚拟机计划重启（内存将被擦除）。
- Redeploy：虚拟机计划移到另一节点（临时磁盘会丢失）。

对事件进行计划以后 \(Status = Scheduled\)，Azure 会共享在其后可以启动事件的时间（在“NotBefore”字段中指定）。

### 启动事件（加速）

了解即将发生的事件并完成正常关闭逻辑以后，即可进行 **POST** 调用，指示 Azure 加快操作速度（如果可能）
 
## PowerShell 示例 

以下示例读取元数据服务器中的计划事件并将其记录到 Application 事件日志中，然后进行确认。

    $localHostIP = "169.254.169.254"
    $ScheduledEventURI = "http://"+$localHostIP+"/metadata/latest/scheduledevents"

    # Call Azure Metadata Service - Scheduled Events 
    $scheduledEventsResponse =  Invoke-RestMethod -Headers @{"Metadata"="true"} -URI $ScheduledEventURI -Method get 

    if ($json.Events.Count -eq 0 )
    {
        Write-Output "++No scheduled events were found"
    }

    for ($eventIdx=0; $eventIdx -lt $scheduledEventsResponse.Events.Length ; $eventIdx++)
    {
        if ($scheduledEventsResponse.Events[$eventIdx].Resources[0].ToLower().substring(1) -eq $env:COMPUTERNAME.ToLower())
        {    
            # YOUR LOGIC HERE 
             pause "This Virtual Machine is scheduled for to "+ $scheduledEventsResponse.Events[$eventIdx].EventType

            # Acknoledge the event to expedite
            $jsonResp = "{""StartRequests"" : [{ ""EventId"": """+$scheduledEventsResponse.events[$eventIdx].EventId +"""}]}"
            $respbody = convertto-JSon $jsonResp
       
            Invoke-RestMethod -Uri $ScheduledEventURI  -Headers @{"Metadata"="true"} -Method POST -Body $jsonResp 
        }
    }


## C\# 示例 
以下代码演示客户端如何显示可与元数据服务通信的 API

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

不能使用以下数据结构分析计划事件

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

## Python 示例 

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


## 后续步骤 
[Azure 中虚拟机的计划内维护](/documentation/articles/virtual-machines-linux-planned-maintenance/)

<!---HONumber=Mooncake_0213_2017-->