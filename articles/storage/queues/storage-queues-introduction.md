---
title: Azure 队列简介 - Azure 存储
description: Azure 队列简介
services: storage
author: WenJason
ms.service: storage
ms.topic: overview
origin.date: 06/07/2019
ms.date: 07/15/2019
ms.author: v-jay
ms.reviewer: cbrooks
ms.subservice: queues
ms.openlocfilehash: 167ffc22dfe2a2f88615f47fc01d006848ff0bf3
ms.sourcegitcommit: 80336a53411d5fce4c25e291e6634fa6bd72695e
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 07/12/2019
ms.locfileid: "67844521"
---
# <a name="what-are-azure-queues"></a>什么是 Azure 队列？

Azure 队列存储是一个可存储大量消息的服务。 可以使用 HTTP 或 HTTPS 通过经验证的调用从世界任何位置访问消息。 队列消息大小最大可为 64 KB。 一个队列可以包含数百万条消息，直至达到存储帐户的总容量限值。

## <a name="common-uses"></a>常见用途

队列存储的常见用途包括：

* 创建积压工作以进行异步处理
* 将消息从 Azure Web 角色传递到 Azure 辅助角色

## <a name="queue-service-concepts"></a>队列服务概念

队列服务包含以下组件：

![队列概念](./media/storage-queues-introduction/queue1.png)

* **URL 格式：** 可使用以下 URL 格式对队列进行寻址：

    `https://<storage account>.queue.core.chinacloudapi.cn/<queue>` 
  
    可使用以下 URL 访问示意图中的某个队列：  
  
    `https://myaccount.queue.core.chinacloudapi.cn/images-to-download`

* **存储帐户：** 对 Azure 存储进行的所有访问都要通过存储帐户完成。 有关存储帐户容量的详细信息，请参阅 [Azure 存储可伸缩性和性能目标](../common/storage-scalability-targets.md?toc=%2fstorage%2fqueues%2ftoc.json) 。

* **队列：** 一个队列包含一组消息。 队列名称**必须**全部小写。 有关命名队列的详细信息，请参阅 [命名队列和元数据](https://msdn.microsoft.com/library/azure/dd179349.aspx)。

* **消息：** 一条消息（不管采用何种格式）的最大大小为 64 KB。 在 2017-07-29 以前的版本中，允许的最大生存时间为 7 天。 在 2017-07-29 或更高版本中，最大生存时间可以是任何正数，或者是 -1（表示消息不会过期）。 如果省略此参数，则默认的生存时间为 7 天。

## <a name="next-steps"></a>后续步骤

* [创建存储帐户](../storage-create-storage-account.md?toc=%2fstorage%2fqueues%2ftoc.json)
* [使用 .NET 的队列入门](storage-dotnet-how-to-use-queues.md)