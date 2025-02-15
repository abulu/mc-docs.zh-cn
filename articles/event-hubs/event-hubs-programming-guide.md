---
title: 编程指南 - Azure 事件中心 | Azure Docs
description: 本文介绍如何使用 Azure .NET SDK 为 Azure 事件中心编写代码。
services: event-hubs
documentationcenter: na
author: ShubhaVijayasarathy
ms.service: event-hubs
ms.custom: seodec18
ms.topic: article
origin.date: 08/12/2018
ms.date: 07/15/2019
ms.author: v-biyu
ms.openlocfilehash: ccaa7929f61cdb1ed731ee11c8dc8d4db6030eb1
ms.sourcegitcommit: a829f1191e40d8940a5bf6074392973128cfe3c0
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 07/04/2019
ms.locfileid: "67560295"
---
# <a name="programming-guide-for-azure-event-hubs"></a>Azure 事件中心编程指南
本文介绍使用 Azure 事件中心编写代码时的一些常见情况。 内容假设你对事件中心已有初步的了解。 有关事件中心的概念概述，请参阅 [事件中心概述](event-hubs-what-is-event-hubs.md)。

## <a name="event-publishers"></a>事件发布者

使用 HTTP POST 或通过 AMQP 1.0 连接将事件发送到事件中心。 何时使用哪种方式的选择取决于要解决的特定方案。 AMQP 1.0 连接计量为服务总线中的中转连接计量，对于经常要以较高的消息量和较低的延迟传送消息的方案，适合选择此方式，因为它们提供持久的消息传递通道。

使用 .NET 托管 API 时，用于将数据发布到事件中心的主要构造是 [EventHubClient][] 和 [EventData][] 类。 [EventHubClient][] 提供 AMQP 信道，事件将通过该信道发送到事件中心。 [EventData][] 类表示一个事件，用于将消息发布到事件中心。 此类包括正文、一些元数据和有关事件的标头信息。 其他属性将在 [EventData][] 对象通过事件中心时添加到该对象。

## <a name="get-started"></a>入门
支持事件中心的 .NET 类在 [Microsoft.Azure.EventHubs](https://www.nuget.org/packages/Microsoft.Azure.EventHubs/) NuGet 包中提供。 可以使用 Visual Studio 解决方案资源管理器进行安装，也可使用 Visual Studio 中的[包管理器控制台](https://docs.nuget.org/docs/start-here/using-the-package-manager-console)。 为此，请在 [程序包管理器控制台](https://docs.nuget.org/docs/start-here/using-the-package-manager-console) 窗口中发出以下命令：

```shell
Install-Package Microsoft.Azure.EventHubs
```

## <a name="create-an-event-hub"></a>创建事件中心

可以通过 Azure 门户、Azure PowerShell 或 Azure CLI 来创建事件中心。 有关详细信息，请参阅[使用 Azure 门户创建事件中心命名空间和事件中心](event-hubs-create.md)。

## <a name="create-an-event-hubs-client"></a>创建事件中心客户端

与事件中心交互的主类是 [Microsoft.Azure.EventHubs.EventHubClient][EventHubClient]。 可以使用 [CreateFromConnectionString](https://docs.azure.cn/zh-cn/dotnet/api/microsoft.azure.eventhubs.eventhubclient.createfromconnectionstring?view=azure-dotnet) 方法实例化此类，如以下示例所示：

```csharp
private const string EventHubConnectionString = "Event Hubs namespace connection string";
private const string EventHubName = "event hub name";

var connectionStringBuilder = new EventHubsConnectionStringBuilder(EventHubConnectionString)
{
    EntityPath = EventHubName

};
eventHubClient = EventHubClient.CreateFromConnectionString(connectionStringBuilder.ToString());
```

## <a name="send-events-to-an-event-hub"></a>将事件发送到事件中心

请通过以下方式将事件发送到事件中心：创建一个 [EventHubClient][] 实例并通过 [SendAsync](https://docs.azure.cn/zh-cn/dotnet/api/microsoft.azure.eventhubs.eventhubclient.sendasync?view=azure-dotnet) 方法异步发送该实例。 此方法采用单个 [EventData][] 实例参数，并将其同步发送到事件中心。

## <a name="event-serialization"></a>事件序列化

[EventData][] 类有[两个重载构造函数](https://docs.azure.cn/zh-cn/dotnet/api/microsoft.azure.eventhubs.eventdata.-ctor?view=azure-dotnet)，这些构造函数采用各种参数、字节或字节数组来表示事件数据有效负载。 将 JSON 与 [EventData][] 类结合使用时，可以使用 **Encoding.UTF8.GetBytes()** 来检索 JSON 编码字符串的字节数组。 例如：

```csharp
for (var i = 0; i < numMessagesToSend; i++)
{
    var message = $"Message {i}";
    Console.WriteLine($"Sending message: {message}");
    await eventHubClient.SendAsync(new EventData(Encoding.UTF8.GetBytes(message)));
}
```

## <a name="partition-key"></a>分区键

> [!NOTE]
> 如果你不熟悉分区，请参阅[此文](event-hubs-features.md#partitions)。 

发送事件数据时，可指定一个在经哈希处理后生成分区分配的值。 请使用 [PartitionSender.PartitionID](https://docs.azure.cn/zh-cn/dotnet/api/microsoft.azure.eventhubs.partitionsender.partitionid?view=azure-dotnet) 属性指定分区。 但是，若要决定使用分区，则必须在可用性和一致性之间进行选择。 

### <a name="availability-considerations"></a>可用性注意事项

可选择使用分区键，并应仔细考虑是否使用分区键。 如果在发布事件时未指定分区键，则会使用循环分配。 在许多情况下，如果事件排序较为重要，使用分区键是一个不错的选择。 使用分区键时，这些分区需要单个节点上的可用性，并且可能会随时间推移发生故障；例如，在计算节点重启和修补时。 因此，如果设置了分区 ID，并且该分区由于某种原因变得不可用，则对该分区中的数据的访问尝试会失败。 如果高可用性是最重要的，请不要指定分区键；在这种情况下，将使用前述的轮循机制模型将事件发送到分区。 在这种情况下，需在可用性（无分区 ID）和一致性（将事件固定到分区 ID）之间做出明确选择。

另一个注意事项是处理事件处理中的延迟。 在某些情况下，丢弃数据并重试可能比尝试跟上处理要更好，后者可能会进而导致下游处理延迟。 例如，在拥有股票行情自动收录器的情况下，最好等待接收完整的最新数据，但在实时聊天或 VOIP 的情况下，则更希望能快速获得数据，即使数据不完整。

考虑到这些可用性需求，在这些情况下，可选择以下错误处理策略之一：

- 停止（在修复之前停止从事件中心读取）
- 丢弃（消息不重要，将其丢弃）
- 重试（根据需要重试消息）

有关详细信息和有关可用性和一致性之间的权衡的讨论，请参阅[事件中心中的可用性和一致性](event-hubs-availability-and-consistency.md)。 

## <a name="batch-event-send-operations"></a>批处理事件发送操作

分批发送事件可有助于提高吞吐量。 可以使用 [CreateBatch](https://docs.azure.cn/zh-cn/dotnet/api/microsoft.azure.eventhubs.eventhubclient.createbatch?view=azure-dotnet) API 创建一个批，以便稍后向其添加用于 [SendAsync](https://docs.azure.cn/zh-cn/dotnet/api/microsoft.azure.eventhubs.eventhubclient.sendasync?view=azure-dotnet) 调用的数据对象。

单个批不能超过事件的 256 KB 限制。 此外，批中的每个消息都要使用相同的发布者标识。 发送者负责确保批不超过最大事件大小。 如果超过该限制，则会生成客户端 **Send** 错误。 可以使用帮助器方法 [EventHubClient.CreateBatch](https://docs.azure.cn/zh-cn/dotnet/api/microsoft.azure.eventhubs.eventhubclient.createbatch?view=azure-dotnet) 来确保批不超过 256 KB。 从 [CreateBatch](https://docs.azure.cn/zh-cn/dotnet/api/microsoft.azure.eventhubs.eventhubclient.createbatch?view=azure-dotnet) API 获取空的 [EventDataBatch](https://docs.azure.cn/zh-cn/dotnet/api/microsoft.azure.eventhubs.eventdatabatch?view=azure-dotnet)，然后使用 [TryAdd](https://docs.azure.cn/zh-cn/dotnet/api/microsoft.azure.eventhubs.eventdatabatch.tryadd?view=azure-dotnet) 添加事件来构造批。 

## <a name="send-asynchronously-and-send-at-scale"></a>异步发送和按比例发送

请通过异步方式将事件发送到事件中心。 以异步方式发送可以增大客户端发送事件的速率。 [SendAsync](https://docs.azure.cn/zh-cn/dotnet/api/microsoft.azure.eventhubs.eventhubclient.sendasync?view=azure-dotnet) 返回 [Task](https://msdn.microsoft.com/library/system.threading.tasks.task.aspx) 对象。 可以在客户端上使用 [RetryPolicy](https://docs.azure.cn/zh-cn/dotnet/api/microsoft.servicebus.retrypolicy?view=azure-dotnet) 类来控制客户端重试选项。

## <a name="event-consumers"></a>事件使用者
[EventProcessorHost][] 类处理来自事件中心的数据。 在 .NET 平台上构建事件读取者时，应该使用此实现。 [EventProcessorHost][] 为事件处理器实现提供线程安全、多进程安全的运行时环境，该环境还能提供检查点和分区租用管理。

若要使用 [EventProcessorHost][] 类，可以实现 [IEventProcessor](https://docs.azure.cn/zh-cn/dotnet/api/microsoft.azure.eventhubs.processor.ieventprocessor?view=azure-dotnet)。 此接口包含四个方法：

* [OpenAsync](https://docs.azure.cn/zh-cn/dotnet/api/microsoft.azure.eventhubs.processor.ieventprocessor.openasync?view=azure-dotnet)
* [CloseAsync](https://docs.azure.cn/zh-cn/dotnet/api/microsoft.azure.eventhubs.processor.ieventprocessor.closeasync?view=azure-dotnet)
* [ProcessEventsAsync](https://docs.azure.cn/zh-cn/dotnet/api/microsoft.azure.eventhubs.processor.ieventprocessor.processeventsasync?view=azure-dotnet)
* [ProcessErrorAsync](https://docs.azure.cn/zh-cn/dotnet/api/microsoft.azure.eventhubs.processor.ieventprocessor.processerrorasync?view=azure-dotnet)

若要开始处理事件，请实例化 [EventProcessorHost][]，为事件中心提供适当的参数。 例如：

> [!NOTE]
> EventProcessorHost 及其相关类在 **Microsoft.Azure.EventHubs.Processor** 包中提供。 按照[此文](event-hubs-dotnet-framework-getstarted-receive-eph.md#add-the-event-hubs-nuget-package)中的说明或在[包管理器控制台](https://docs.nuget.org/docs/start-here/using-the-package-manager-console)窗口中发出以下命令，将包添加到 Visual Studio 项目中：`Install-Package Microsoft.Azure.EventHubs.Processor`。

```csharp
var eventProcessorHost = new EventProcessorHost(
        EventHubName,
        PartitionReceiver.DefaultConsumerGroupName,
        EventHubConnectionString,
        StorageConnectionString,
        StorageContainerName);
```

然后，调用 [RegisterEventProcessorAsync](https://docs.azure.cn/zh-cn/dotnet/api/microsoft.azure.eventhubs.processor.eventprocessorhost.registereventprocessorasync?view=azure-dotnet)，将 [IEventProcessor](https://docs.azure.cn/zh-cn/dotnet/api/microsoft.azure.eventhubs.processor.ieventprocessor?view=azure-dotnet) 实现注册到运行时：

```csharp
await eventProcessorHost.RegisterEventProcessorAsync<SimpleEventProcessor>();
```

此时，主机将尝试使用“贪婪”算法获取事件中心内每个分区上的租约。 这些租用只在指定的时间段内有效，之后必须续订。 当新节点（本例中的工作线程实例）进入联机状态时，它们将保留租约，以后每次尝试获取更多租约时，负载会在节点之间转移。

![事件处理程序主机](./media/event-hubs-programming-guide/IC759863.png)

经过一段时间后，就会建立平衡。 通过这种动态功能，可以向使用者应用基于 CPU 的自动缩放，以实现向上扩展和向下缩减。 由于事件中心没有直接的消息计数概念，平均 CPU 利用率通常是度量后端或使用者规模的最佳机制。 如果发布者开始发布的事件数超过了使用者可以处理的数量，可以使用使用者的 CPU 增大功能来实现工作线程实例数的自动缩放。

[EventProcessorHost][] 类还实现了基于 Azure 存储的检查点机制。 此机制按分区存储偏移量，每个使用者都能确定前一个使用者的最后一个检查点是什么。 分区通过租约在节点之间转移时，正是此同步机制在促进负载转移。

## <a name="publisher-revocation"></a>发布者吊销

除了 [EventProcessorHost][] 的高级运行时功能外，事件中心还支持吊销发布者，以阻止特定发布者向事件中心发送事件。 当发布者令牌已泄露，或者软件更新导致发布者行为不当时，这些功能很有用。 在这些情况下，可阻止发布者的标识（其 SAS 令牌的一部分）发布事件。

有关发布者吊销以及如何以发布者身份向事件中心发送事件的详细信息，请参阅 [事件中心大规模安全发布](https://code.msdn.microsoft.com/Service-Bus-Event-Hub-99ce67ab) 示例。

## <a name="next-steps"></a>后续步骤

若要了解有关事件中心方案的详细信息，请访问以下链接：

* [事件中心 API 概述](event-hubs-api-overview.md)
* [什么是事件中心](event-hubs-what-is-event-hubs.md)
* [事件中心中的可用性和一致性](event-hubs-availability-and-consistency.md)
* [事件处理程序主机 API 参考](https://docs.azure.cn/zh-cn/dotnet/api/microsoft.servicebus.messaging.eventprocessorhost?view=azure-dotnet)

[NamespaceManager]: https://docs.azure.cn/zh-cn/dotnet/api/microsoft.servicebus.namespacemanager?view=azure-dotnet
[EventHubClient]: https://docs.azure.cn/zh-cn/dotnet/api/microsoft.azure.eventhubs.eventhubclient?view=azure-dotnet
[EventData]: https://docs.azure.cn/zh-cn/dotnet/api/microsoft.azure.eventhubs.eventdata?view=azure-dotnet
[CreateEventHubIfNotExists]: https://docs.azure.cn/zh-cn/dotnet/api/microsoft.servicebus.namespacemanager.createeventhubifnotexists?view=azure-dotnet
[PartitionKey]: https://docs.azure.cn/zh-cn/dotnet/api/microsoft.servicebus.messaging.eventdata?view=azure-dotnet#Microsoft_ServiceBus_Messaging_EventData_PartitionKey
[EventProcessorHost]: https://docs.azure.cn/zh-cn/dotnet/api/microsoft.azure.eventhubs.processor

<!--Update_Description: update meta properties, wording update, update links -->