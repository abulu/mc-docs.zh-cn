---
title: Azure 中的警报和通知监视概述
description: Azure 中的警报概述。 警报、经典警报、警报界面。
author: lingliw
services: monitoring
ms.service: azure-monitor
ms.topic: conceptual
ms.date: 6/4/2019
ms.author: v-lingwu
ms.subservice: alerts
ms.openlocfilehash: da2b21103e302efc08c7d6103bfa3116f0e03c4c
ms.sourcegitcommit: fd927ef42e8e7c5829d7c73dc9864e26f2a11aaa
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 07/04/2019
ms.locfileid: "67562430"
---
# <a name="overview-of-alerts-in-21vianet-azure"></a>世纪互联 Azure 中的警报概述 

本文介绍什么是警报及其优点，以及如何开始使用警报。  


## <a name="what-are-alerts-in-21vianet-azure"></a>什么是世纪互联 Azure 中的警报？
在监视数据中发现重要情况时，警报会以主动的方式通知你。 有了警报，你就可以在系统的用户注意到问题之前确定和解决这些问题。 

本文讨论 Azure Monitor 中的统一警报体验，其现在包括由 Log Analytics 和 Application Insights 管理的警报。 [以前的警报体验](alerts-classic.overview.md)和警报类型称为“经典警报”。  单击警报页顶部的“查看经典警报”即可查看这个旧的体验和旧的警报类型。  

## <a name="overview"></a>概述

下图表示了警报流。 

![警报流](media/alerts-overview/Azure-Monitor-Alerts.svg)

警报规则独立于警报，也独立于警报触发时采取的操作。 

**警报规则** - 警报规则捕获警报的目标和条件。 警报规则可以是“已启用”或“已禁用”状态。 警报只有在启用后才会触发。 

警报规则的关键属性包括：

**目标资源** - 定义适用于警报的范围和信号。 目标可以是任何 Azure 资源。 示例目标：虚拟机、存储帐户、虚拟机规模集、Log Analytics 工作区或 Application Insights 资源。 对于某些资源（例如虚拟机）来说，可以将多个资源指定为警报规则的目标。

**信号** - 信号由目标资源发出，可以有多种类型。 指标、活动日志、Application Insights 和日志。

**条件** - 条件是应用于目标资源的信号和逻辑的组合。 示例: 
   - CPU 百分比 > 70%
   - 服务器响应时间 > 4 毫秒 
   - 日志查询的结果计数 > 100

**警报名称** - 用户配置的警报规则的具体名称

**警报说明** - 用户配置的警报规则的说明

**严重性** - 警报规则中指定的条件符合后确定的警报严重性。 严重性的范围为 0 到 4。

**操作** - 触发警报时执行的特定操作。 有关详细信息，请参阅[操作组](../../azure-monitor/platform/action-groups.md)。

## <a name="what-you-can-alert-on"></a>可以报警的内容

可以按照“监视数据源”中的说明，针对指标和日志发出警报。 这些检查包括但不限于：
- 指标值
- 日志搜索查询
- 活动日志事件
- 基础 Azure 平台的运行状况
- 网址可用性测试

以前，Azure Monitor 指标、Application Insights、Log Analytics 和服务运行状况都有单独的警报功能。 随着时间推移，Azure 改进并组合了用户界面和不同的警报方法。 此整合仍在进行中。 因此，仍有一些警报功能未出现在新的警报系统中。  

| **监视器源** | **信号类型**  | **说明** | 
|-------------|----------------|-------------|
| 服务运行状况 | 活动日志  | 不支持。 请参阅[创建有关服务通知的活动日志警报](../../azure-monitor/platform/alerts-activity-log-service-notifications.md)。  |
| Application Insights | Web 可用性测试 | 不支持。 请参阅 [Web 测试警报](../../azure-monitor/app/monitor-web-app-availability.md)。 适用于任何经检测可将数据发送到 Application Insights 的网站。 网站的可用性或响应度低于预期时，就会收到通知。 |

## <a name="manage-alerts"></a>管理警报
可以设置警报状态来指定它在解决过程中所处的阶段。 符合警报规则中指定的条件以后，就会创建或触发警报，其状态为“新”。  可以在确认警报和关闭警报时更改状态。 所有状态更改都存储在警报历史记录中。

支持以下警报状态。

| 状态 | 说明 |
|:---|:---|
| 新建 | 只是检测到了问题，但尚未审查问题。 |
| 已确认 | 管理员已审查警报，并已开始进行处理。 |
| 已关闭 | 问题已解决。 关闭某个警报后，可通过将其更改为另一种状态来重新打开它。 |

**警报状态**不同于且独立于**监视条件**。 警报状态是由用户设置的。 监视条件是由系统设置的。 当警报触发后，警报的监视条件设置为“已触发”  。 当导致警报触发的基础条件解除后，监视条件会设置为“已解决”。  在用户更改警报状态之前，警报状态不会改变。 了解[如何更改警报和智能组的状态](https://aka.ms/managing-alert-smart-group-states)。

## <a name="smart-groups"></a>智能组 
智能组为预览版。 

智能组是根据机器学习算法对警报进行的聚合。这些算法有助于降低警报噪音，对故障排除也有帮助。 [详细了解智能组](https://aka.ms/smart-groups)和[如何管理智能组](https://aka.ms/managing-smart-groups)。


## <a name="alerts-experience"></a>警报体验 
默认的“警报”页提供特定时间范围内创建的警报的摘要。 该页显示每种严重性的警报总数，列中会标识处于每种状态的、具有每种严重性的警报总数。 选择任一严重性可打开按该严重性筛选的“[所有警报](#all-alerts-page)”页。

或者，可以[使用 REST API 以编程方式枚举在订阅上生成的警报实例](#manage-your-alert-instances-programmatically)。

它不显示或跟踪旧的[经典警报](#classic-alerts)。 可以通过更改订阅或筛选器参数来更新页面。 

![“警报”页](media/alerts-overview/alerts-page.png)

可以通过选择页面顶部的下拉菜单中的值，来对此视图进行筛选。

| 列 | 说明 |
|:---|:---|
| 订阅 | 选择要查看其警报的 Azure 订阅。 （可选）可以选择你的所有订阅。 视图中仅包含你在所选订阅中有权访问的警报。 |
| 资源组 | 选择单个资源组。 只有包含选定资源组中的目标的警报才会包含在视图中。 |
| 时间范围 | 只有在选定时间范围内触发的警报才会包含在该视图中。 支持的值为过去 1 小时、过去 24 小时、过去 7 天和过去 30 天。 |

选择“警报”页面顶部的以下值打开另一个页面。

| Value | 说明 |
|:---|:---|
| 警报总数 | 符合选定条件的警报总数。 选择此值会打开未经筛选的“所有警报”视图。 |
| 智能组 | 从符合选定条件的警报创建的智能组总数。 选择此值会在“所有警报”视图中打开智能组列表。
| 警报规则总数 | 选定订阅和资源组中的警报规则总数。 选择此值会打开根据选定订阅和资源组筛选的“规则”视图。


## <a name="manage-alert-rules"></a>管理警报规则
单击“管理警报规则”  即可显示“规则”页  。 “规则”是用于跨 Azure 订阅管理所有警报规则的一个地方  。 此页列出所有警报规则，这些规则可以根据目标资源、资源组、规则名称或状态排序。 还可以在此页中编辑、启用或禁用警报规则。  

 ![警报规则](./media/alerts-overview/alerts-preview-rules.png)


## <a name="create-an-alert-rule"></a>创建警报规则
可以通过一致的方式创作警报，而不考虑监视服务或信号类型。 单个页面中提供了所有触发的警报和相关详细信息。
 
使用以下三个步骤创作新警报规则：
1. 选取警报的目标。 
1. 从目标的可用信号中选择信号。 
1. 指定要应用到信号中数据的逻辑。 
 
这个创作过程经过了简化，用户在选择 Azure 资源之前，不再需要知道受支持的监视源或信号。 可用信号列表会根据你选择的目标资源自动筛选。 另外，还将根据该目标引导你自动定义警报规则的逻辑。  

可以在[使用 Azure Monitor 创建、查看和管理警报](../../azure-monitor/platform/alerts-metric.md)中详细了解如何创建警报规则。

警报可在多个 Azure 监视服务中使用。 有关如何以及何时使用其中每种服务的信息，请参阅[监视 Azure 应用程序和资源](../../azure-monitor/overview.md)。 


## <a name="all-alerts-page"></a>“所有警报”页 
单击“总警报数”即可查看“所有警报”页。 在这里，可以查看在选定时间范围内创建的警报列表。 可以查看各个警报的列表，或包含这些警报的智能组列表。 选择页面顶部的标题可在视图之间进行切换。

![“所有警报”页](media/alerts-overview/all-alerts-page.png)

可以通过选择页面顶部的下拉菜单中的以下值，可以对该视图进行筛选。

| 列 | 说明 |
|:---|:---|
| 订阅 | 选择要查看其警报的 Azure 订阅。 （可选）可以选择你的所有订阅。 视图中仅包含你在所选订阅中有权访问的警报。 |
| 资源组 | 选择单个资源组。 只有包含选定资源组中的目标的警报才会包含在视图中。 |
| 资源类型 | 选择一个或多个资源类型。 只有包含选定类型中的目标的警报才会包含在视图中。 仅在指定资源组后，才显示此列。 |
| Resource | 选择资源。 只有包含该资源（作为目标）的警报才会包含在视图中。 仅在指定资源类型后，才显示此列。 |
| 严重性 | 选择警报严重性，或选择“所有”以包含所有严重性的警报  。 |
| 监视条件 | 选择监视条件，或选择“所有”以包括所有条件的警报  。 |
| 警报状态 | 选择警报状态，或选择“所有”以包含所有状态的警报  。 |
| 监视服务 | 选择一个服务，或选择“所有”以包含所有服务  。 只会包含使用该服务（作为目标）的规则创建的警报。 |
| 时间范围 | 只有在选定时间范围内触发的警报才会包含在该视图中。 支持的值为过去 1 小时、过去 24 小时、过去 7 天和过去 30 天。 |

选择页面顶部的“列”可以选择要显示的列  。 

## <a name="alert-details-page"></a>“警报详细信息”页
选择某个警报时，会显示“警报详细信息”页。 该页提供警报详细信息，并可在其中更改警报的状态。

![警报详细信息](media/alerts-overview/alert-detail2.png)

“警报详细信息”页包括以下部分。

| 部分 | 说明 |
|:---|:---|
| 摘要 | 显示警报的属性和其他重要信息。 |
| 历史记录 | 列出警报执行的每个操作，以及对警报进行的任何更改。 目前仅限状态更改。 |
| 诊断 | 有关包含警报的智能组的信息。 “警报计数”表示包含在智能组中的警报数量  。 包括同一智能组中在过去 30 天内创建的其他警报，无论警报列表页面中的时间筛选器是什么。 选择某个警报以查看其详细信息。 |

## <a name="manage-your-alert-instances-programmatically"></a>以编程方式管理警报实例

在许多情况下，你可能希望以编程方式查询针对订阅生成的警报。 这可以是在 Azure 门户之外创建自定义视图，也可以是分析警报以确定模式和趋势。

可以使用[警报管理 REST API](https://aka.ms/alert-management-api) 或[用于警报的 Azure Resource Graph REST API](https://docs.microsoft.com/rest/api/azureresourcegraph/resources/resources) 查询针对订阅生成的警报。

[用于警报的 Azure Resource Graph REST API](https://docs.microsoft.com/rest/api/azureresourcegraph/resources/resources) 允许你大规模地查询警报实例。 对于必须管理跨多个订阅生成的警报的情况，建议使用此 API。 

以下对 API 的示例请求返回一个订阅中的警报计数：

```json
{
  "subscriptions": [
    <subscriptionId>
  ],
  "query": "where type =~ 'Microsoft.AlertsManagement/alerts' | summarize count()",
  "options": {
            "dataset":"alerts"
  }
}
```
可以查询警报的[“基本”](/azure-monitor/platform/alerts-common-schema-definitions#essentials-fields)字段。

[警报管理 REST API](https://aka.ms/alert-management-api) 可用于获取有关特定警报的详细信息，包括其[“警报上下文”](/azure-monitor/platform/alerts-common-schema-definitions#alert-context-fields)字段。

## <a name="classic-alerts"></a>经典警报 

2018 年 6 月之前的 Azure Monitor 指标和活动日志警报功能称为“警报（经典）”。 

有关详细信息，请参阅[经典警报](./../../azure-monitor/platform/alerts-classic.overview.md)


## <a name="next-steps"></a>后续步骤

- [详细了解智能组](https://aka.ms/smart-groups)
- [了解操作组](../../azure-monitor/platform/action-groups.md)
- [在 Azure 中管理警报实例](https://aka.ms/managing-alert-instances)
- [Managing Smart Groups](https://aka.ms/managing-smart-groups)（管理智能组）





