---
title: include 文件
description: include 文件
services: functions
author: ggailey777
manager: jeconnoc
ms.service: azure-functions
ms.topic: include
origin.date: 03/14/2019
ms.date: 07/17/2019
ms.author: v-junlch
ms.custom: include file
ms.openlocfilehash: 1101ea9f818da61022e404e628d115f59f2c40da
ms.sourcegitcommit: c61b10764d533c32d56bcfcb4286ed0fb2bdbfea
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 07/19/2019
ms.locfileid: "68332797"
---
[Durable Functions](../articles/azure-functions/durable-functions-overview.md) 的配置设置。

```json
{
  "durableTask": {
    "hubName": "MyTaskHub",
    "controlQueueBatchSize": 32,
    "partitionCount": 4,
    "controlQueueVisibilityTimeout": "00:05:00",
    "workItemQueueVisibilityTimeout": "00:05:00",
    "maxConcurrentActivityFunctions": 10,
    "maxConcurrentOrchestratorFunctions": 10,
    "maxQueuePollingInterval": "00:00:30",
    "azureStorageConnectionStringName": "AzureWebJobsStorage",
    "trackingStoreConnectionStringName": "TrackingStorage",
    "trackingStoreNamePrefix": "DurableTask",
    "traceInputsAndOutputs": false]
  }
}
```

任务中心名称必须以字母开头且只能包含字母和数字。 如果未指定，则函数应用的默认任务中心名称是 **DurableFunctionsHub**。 有关详细信息，请参阅[任务中心](../articles/azure-functions/durable-functions-task-hubs.md)。

|属性  |默认 | 说明 |
|---------|---------|---------|
|hubName|DurableFunctionsHub|可以使用备用[任务中心](../articles/azure-functions/durable-functions-task-hubs.md)名称将多个 Durable Functions 应用程序彼此隔离，即使这些应用程序使用同一存储后端。|
|controlQueueBatchSize|32|要从控制队列中一次性拉取的消息数。|
|partitionCount |4|控制队列的分区计数。 可以是 1 到 16 之间的正整数。|
|controlQueueVisibilityTimeout |5 分钟|已取消排队的控制队列消息的可见性超时。|
|workItemQueueVisibilityTimeout |5 分钟|已取消排队的工作项队列消息的可见性超时。|
|maxConcurrentActivityFunctions |10 倍于当前计算机上的处理器数|可以在单个主机实例上并发处理的活动函数的最大数目。|
|maxConcurrentOrchestratorFunctions |10 倍于当前计算机上的处理器数|可以在单个主机实例上并发处理的业务流程协调程序函数的最大数目。|
|maxQueuePollingInterval|30 秒|最大的控制和工作项队列轮询时间间隔，采用 *hh:mm:ss* 格式。 值越高，可能导致的消息处理延迟也越高。 值越低，可能导致的存储成本会越高，因为存储事务数增高。|
|azureStorageConnectionStringName |AzureWebJobsStorage|应用设置的名称，其中的 Azure 存储连接字符串用于管理基础的 Azure 存储资源。|
|trackingStoreConnectionStringName||连接字符串的名称，用于“历史记录”和“实例”表。 如果未指定，则使用 `azureStorageConnectionStringName` 连接。|
|trackingStoreNamePrefix||指定 `trackingStoreConnectionStringName` 时用于“历史记录”和“实例”表的前缀。 如果未设置，则默认前缀值为 `DurableTask`。 如果 `trackingStoreConnectionStringName` 未指定，则“历史记录”和“实例”表会使用 `hubName` 值作为其前缀，`trackingStoreNamePrefix` 的任何设置都会被忽略。|
|traceInputsAndOutputs |false|一个指示是否跟踪函数调用的输入和输出的值。 跟踪函数执行事件时的默认行为是在函数调用的序列化输入和输出中包括字节数。 此行为提供的有关输入和输出情况的信息是最少的，不会导致日志膨胀，也不会无意中将敏感信息公开。 将此属性设置为 true 会导致默认函数日志记录将函数输入和输出的整个内容都记录下来。|

许多此类设置用于优化性能。 有关详细信息，请参阅[性能和缩放](../articles/azure-functions/durable-functions-perf-and-scale.md)。

