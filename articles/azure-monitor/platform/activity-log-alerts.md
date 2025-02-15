---
title: Azure Monitor 中的活动日志警报
description: 当活动日志中出现某些事件时，通过 SMS、Webhook、短信、电子邮件等方式进行通知。
author: msvijayn
services: azure-monitor
ms.service: azure-monitor
ms.topic: conceptual
ms.date: 09/17/2018
ms.author: vinagara
ms.subservice: alerts
ms.openlocfilehash: a73e35cb31cb3272bb2ef940432be97cea7bd100
ms.sourcegitcommit: fd927ef42e8e7c5829d7c73dc9864e26f2a11aaa
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 07/04/2019
ms.locfileid: "67562471"
---
# <a name="alerts-on-activity-log"></a>根据活动日志发出警报 

## <a name="overview"></a>概述
活动日志警报是新发生的活动日志事件与警报中指定的条件匹配时激活的警报。 它们是 Azure 资源，因此，可通过使用 Azure 资源管理器模板进行创建。 此外，还可以在 Azure 门户中创建、更新或删除它们。 本文介绍活动日志警报背后的概念。 然后演示如何使用 Azure 门户设置有关活动日志事件的警报。 有关使用情况的详细信息，请参阅[创建和管理活动日志警报](../../azure-monitor/platform/alerts-activity-log.md)。

> [!NOTE]
> **无法**为活动日志的“警报”类别中的事件创建警报

通常，你会在以下情况下创建活动日志警报以接收通知：

* 对 Azure 订阅中的资源进行特定操作时，通常限于特定资源组或资源。 例如，可能会希望在删除 myProductionResourceGroup 中的任何虚拟机时接收通知。 或者，可能会希望在任何新角色分配到订阅中的用户时接收通知。
* 发生服务运行状况事件。 服务运行状况事件包括应用于订阅中资源的事件和维护事件的通知。

可以通过简单的类比来理解在活动日志上创建警报规则时可以基于的条件，那就是通过 [Azure 门户中的活动日志](../../azure-monitor/platform/activity-logs-overview.md)浏览或筛选事件。 在 Azure Monitor - 活动日志中，可以筛选或查找所需的事件，然后使用“添加活动日志警报”按钮创建警报  。

在上述任何情况下，活动日志警报只监视在其中创建该警报的订阅中的事件。

可以基于活动日志事件的 JSON 对象中的任何顶层属性配置活动日志警报。 有关详细信息，请参阅 [Azure 活动日志概述](./../../azure-monitor/platform/activity-logs-overview.md#categories-in-the-activity-log)。 若要了解有关服务运行状况事件的详细信息，请参阅[接收有关服务通知的活动日志警报](./../../azure-monitor/platform/alerts-activity-log-service-notifications.md)。 

活动日志警报有几个常见选项：

- **类别**：管理、服务运行状况、自动缩放、安全性、策略和建议。 
- **范围**：为其定义活动日志警报的单个资源或资源集。 可以在各个级别定义活动日志警报的范围：
    - 资源级别：例如，针对特定虚拟机
    - 资源组级别：例如，特定资源组中的所有虚拟机
    - 订阅级别：例如，某个订阅中的所有虚拟机（或）某个订阅中的所有资源
- **资源组**：默认情况下，警报规则保存在“范围”中定义的目标所在的同一资源组中。 用户也可以定义应存储警报规则的资源组。
- **资源类型**：资源管理器为警报的目标定义的命名空间。

- **操作名称**：资源管理器基于角色的访问控制操作名称。
- **级别**：事件的严重级别（详细、信息、警告、错误或严重）。
- **状态**：事件的状态，通常为“已启动”、“失败”或“成功”。
- **事件发起者**：也称为“调用方”。 电子邮件地址或执行操作的用户的 Azure Active Directory 标识符。

> [!NOTE]
> 在一个订阅中最多有 100 条警报规则可以针对以下任一范围活动创建：单个资源、资源组中的所有资源（或）整个订阅级别。

活动日志警报激活后会使用操作组生成操作或通知。 操作组是一组可重用的通知接收方，例如电子邮件地址、Webhook URL 或短信电话号码。 可以从多个警报中引用接收方，以集中和分组通知通道。 在定义活动日志警报时，有两个选项。 方法：

* 在活动日志警报中使用现有操作组。
* 创建新的操作组。

若要了解有关操作组的详细信息，请参阅[在 Azure 门户中创建和管理操作组](../../azure-monitor/platform/action-groups.md)。


## <a name="next-steps"></a>后续步骤
- 获取[警报概述](../../azure-monitor/platform/alerts-overview.md)。
- 了解如何[创建和修改活动日志警报](../../azure-monitor/platform/alerts-activity-log.md)。
- 查看[活动日志警报 webhook 架构](activity-log-alerts-webhook.md)。
- 了解[服务运行状况通知](../../azure-monitor/platform/service-notifications.md)。

