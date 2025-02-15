---
title: Azure Monitor 中的统一警报和监视替换经典警报和监视
description: 关于停用之前在“警报(经典)”下的 Azure 门户中显示的经典监视服务和功能的概述。 经典警报和监视包括以下内容：针对 Azure 资源的经典指标警报、针对 Application Insights 的经典指标警报、针对 Application Insights 的经典 Web 测试警报、针对 Application Insights 的经典自定义指标警报和针对 Application Insights SmartDetection v1 的经典警报
author: lingliw
services: azure-monitor
ms.service: azure-monitor
ms.topic: conceptual
ms.date: 6/4/2019
ms.author: v-lingwu
ms.subservice: alerts
ms.openlocfilehash: fafec2de1e849f098b4ee2c39d92c8782097bf21
ms.sourcegitcommit: fd927ef42e8e7c5829d7c73dc9864e26f2a11aaa
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 07/04/2019
ms.locfileid: "67562328"
---
# <a name="unified-alerting--monitoring-in-azure-monitor-replaces-classic-alerting--monitoring"></a>Azure Monitor 中的统一警报和监视替换经典警报和监视

Azure Monitor 现已成为统一的完整堆栈监视服务，它现在支持跨资源使用“一个指标”和“一个警报”；如需更多信息，请参阅[关于新 Azure Monitor 的博客文章](https://azure.microsoft.com/blog/new-full-stack-monitoring-capabilities-in-azure-monitor/)。新构建的 Azure 监视和警报平台更为快速、更为智能并且可以扩展 - 与不断增长的云计算扩展齐头并进，并与 Azure 智能云理念保持一致。 

新的 Azure 监视和警报平台建成后，我们**将于 2019 年 8 月在 Azure 公有云中弃用**“经典”监视和警报平台（托管于 Azure 警报的“查看经典警报”部分内）  。

> [!NOTE]
> 由于迁移工具的延迟推出，经典警报迁移的停用日期已从原来宣布的 2019 年 6 月 30 日[推迟至 2019 年 8 月 31 日](https://www.azure.cn/zh-cn/what-is-new/)。

 ![Azure 门户中的经典警报](media/monitoring-classic-retirement/monitor-alert-screen2.png) 

我们鼓励你开始在新平台中重新创建警报。 对于有大量警报的客户，我们努力提供自动化方式，将现有经典警报移到新的警报系统，而无需中断或增加成本。

> [!IMPORTANT]
> 基于活动日志创建的经典警报规则不会被弃用或迁移。 可以从新的 Azure Monitor -“警报”按现样访问和使用基于活动日志创建的所有经典警报规则。 有关详细信息，请参阅[使用 Azure Monitor 创建、查看和管理活动日志警报](../../azure-monitor/platform/alerts-activity-log.md)。 类似地，可以从新的“服务运行状况”部分按现样访问和使用基于服务运行状况的警报。 有关详细信息，请参阅[基于服务运行状况通知的警报](../../azure-monitor/platform/alerts-activity-log-service-notifications.md)。

## <a name="unified-metrics-and-alerts-in-application-insights"></a>Application Insights 中的统一指标和警报

Azure Monitor 的新指标平台现将支持来自 Application Insights 的监视。 此项移动意味着 Application Insights 将挂接到操作组，允许除上一封电子邮件和 webhook 调用之外的更多选项。 警报现可触发语音呼叫、Azure Functions、逻辑应用、短信以及 ServiceNow 和自动化 Runbook 等 ITSM 工具。 凭借准实时的监视和警报，新平台允许 Application Insights 用户利用相同的技术在其他 Azure 资源中进行监视，并为 Azure 产品的监视提供支持。

新的适用于 Application Insights 的统一监视和警报将包含：

- **Application Insights 平台指标** – 提供 Application Insights 产品中常用的预建指标。 有关详细信息，请参阅这篇有关如何使用[新 Azure Monitor 上的 Application Insights 平台指标](../../azure-monitor/app/pre-aggregated-metrics-log-metrics.md#pre-aggregated-metrics)的文章。
- **Application Insights 可用性和 Web 测试** - 提供可评估 Web 应用或服务器的响应能力和可用性的功能。 有关详细信息，请参阅这篇有关如何使用[新 Azure Monitor 上 Application Insights 的可用性测试和警报](../../azure-monitor/app/monitor-web-app-availability.md)的文章。
- **Application Insights 自定义指标** – 使你能够定义和发出自己的监视和警报指标。 有关详细信息，请参阅这篇有关如何使用[新 Azure Monitor 上 Application Insights 的自定义指标](../../azure-monitor/app/pre-aggregated-metrics-log-metrics.md#custom-metrics-dimensions-and-pre-aggregation)的文章。
- **Application Insights 故障异常（智能检测的一部分）** - 如果 Web 应用的失败 HTTP 请求速率或依赖项调用速率出现异常上升，Application Insights 会准实时地自动通知你。 我们即将发布作为新 Azure Monitor 的一部分的 Application Insights 故障异常（智能检测的一部分），并将更新这篇包含下一次迭代链接的文档，因为它将在接下来几个月内推出。

## <a name="unified-metrics-and-alerts-for-other-azure-resources"></a>其他 Azure 资源的统一指标和警报

从 2018 年 3 月开始，已提供 Azure 资源的新一代警报和多维度监视。 现在，新指标平台和警报更快速，具有准实时的功能。 更重要的是，新指标平台警报能够提供更多粒度，因为新平台包括维度选项，使你能够切片和筛选到特定值组合、条件或操作。 与新 Azure Monitor 中的所有警报一样，新指标警报通过使用 ActionGroup 增加了可扩展性 - 使证书能够扩展到电子邮件或 Webhook 之外，到达短信、语音、Azure Function、自动化 Runbook 等。 有关详细信息，请参阅[使用 Azure Monitor 创建、查看和管理指标警报](../../azure-monitor/platform/alerts-metric.md)。
Azure 资源的新指标按以下形式提供：

- **Azure Monitor 标准平台指标** – 提供来自各种 Azure 服务和产品的常用预填充指标。 有关详细信息，请参阅这篇有关 [Azure Monitor 上支持的指标](../../azure-monitor/platform/alerts-metric-near-real-time.md#metrics-and-dimensions-supported)和 [Azure Monitor 上支持的指标警报](../../azure-monitor/platform/alerts-metric-overview.md#supported-resource-types-for-metric-alerts)文章。
- **Azure Monitor 自定义指标** – 提供来自用户驱动源（包括 Azure 诊断代理）的指标。 有关详细信息，请参阅这篇有关 [Azure Monitor 中的自定义指标](../../azure-monitor/platform/metrics-custom-overview.md)的文章。 使用自定义指标，还可以发布 [Microsoft Azure 诊断代理](../../azure-monitor/platform/collect-custom-metrics-guestos-resource-manager-vm.md)和 [InfluxData Telegraf 代理](../../azure-monitor/platform/collect-custom-metrics-linux-telegraf.md)收集的指标。

## <a name="retirement-of-classic-monitoring-and-alerting-platform"></a>经典监视和警报平台的停用

如前文所述，鉴于 Azure 门户的[“警报(经典)”部分](../../azure-monitor/platform/alerts-classic.overview.md)中当前可用的经典监视和警报平台已由新系统代替，经典平台将于接下来数月内停用。
旧的经典监视和警报将于 2019 年 8 月 31 日停用；包括关闭相关 API、Azure 门户界面以及其中的服务。 具体而言，将弃用以下功能：

- 当前可通过 Azure 门户的[警报(经典)部分](../../azure-monitor/platform/alerts-classic.overview.md)使用 Azure 资源的旧（经典）指标和警报；可作为 [microsoft.insights/alertrules](https://docs.microsoft.com/rest/api/monitor/alertrules) 资源访问
- 当前可通过 Azure 门户的[警报(经典)部分](../../azure-monitor/platform/alerts-classic.overview.md)使用 Application Insights 的旧（经典）平台和自定义指标以及相关警报；可作为 [microsoft.insights/alertrules](https://docs.microsoft.com/rest/api/monitor/alertrules) 资源访问
- 旧（经典）故障异常警报当前在 Azure 门户中作为 [Application Insights 内的智能检测](../../azure-monitor/app/proactive-diagnostics.md)提供；其中配置的警报显示在 Azure 门户的[警报(经典)部分](../../azure-monitor/platform/alerts-classic.overview.md)

所有经典监视和警报系统，包括相应的 [API](https://msdn.microsoft.com/library/azure/dn931945.aspx)、[PowerShell](../../azure-monitor/platform/alerts-classic-portal.md)、[CLI](../../azure-monitor/platform/alerts-classic-portal.md)、[Azure 门户页](../../azure-monitor/platform/alerts-classic-portal.md)和[资源模板](../../azure-monitor/platform/alerts-enable-template.md)在 2019 年 8 月结束之前都可继续使用。 

2019 年 8 月底，在 Azure Monitor 中：

- 经典监视和警报服务将停用，并且不再可用于创建新的警报规则。
- 在 2019 年 8 月之后，继续存在于“警报”（经典）中的任何警报规则将继续执行并引发通知，但是不可修改。
- 从 2019 年 9 月开始，经典监视和警报中可迁移的警报规则将由 Microsoft 在几周内分阶段自动移到新的 Azure 监视平台中的等效项。 该过程无缝进行且没有任何停机时间，并且客户在监视覆盖范围内将没有任何损失。
- 迁移到新的警报平台的警报规则将提供与之前相同的监视覆盖范围，但将触发具有新的有效负载的通知。 与经典警报规则相关联的任何电子邮件地址、Webhook 终结点或逻辑应用链接在迁移时都将被转移，但行为可能不正确，因为警报有效负载在新平台中将有所不同。
- 一些[无法自动迁移且需要用户手动操作的经典警报规则](alerts-understand-migration.md#classic-alert-rules-that-will-not-be-migrated)将继续运行到 2020 年 6 月。

> [!IMPORTANT]
> 世纪互联 Azure Monitor 已经分阶段推出了[工具，可以很快将](alerts-using-migration-tool.md)其经典警报规则自动迁移到新平台。 从 2019 年 7 月开始，将强制针对仍然存在且可迁移的所有经典警报规则运行该工具。 在迁移经典警报规则后，客户将需要确保对使用经典警报规则有效负载的自动化进行修改以处理来自 [Application Insights 中的统一指标和警报](#unified-metrics-and-alerts-in-application-insights)或[其他 Azure 资源的统一指标和警报](#unified-metrics-and-alerts-for-other-azure-resources)的新有效负载。 有关更多详细信息，请参阅[准备经典警报规则迁移](alerts-prepare-migration.md)

我们正在推出迁移工具，允许你自行将将 Azure 门户的[“警报(经典)”部分](../../azure-monitor/platform/alerts-classic.overview.md)中的警报迁移到新的 Azure 警报。 迁移到新 Azure Monitor 的警报(经典)中配置的所有规则都将继续免费，不收取任何费用。 已迁移的经典警报规则也不会对通过电子邮件、Webhook 或逻辑应用推送通知收取费用。 但是，使用新通知或操作类型（例如短信、语音呼叫、ITSM 迁移等）将收取费用，无论是添加到已迁移的警报还是添加到新警报中。 有关详细信息，请参阅 [Azure Monitor 定价](https://www.azure.cn/pricing/details/monitor/)。

此外，以下各项将根据 [Azure Monitor 定价](https://www.azure.cn/pricing/details/monitor/)的范围收取费用：

- 新 Azure Monitor 平台上除免费单位数之外创建的任何新警报（非迁移）规则
- Azure Monitor 内除免费单位数之外引入和保留的任何数据
- Application Insights 执行的任何多测试 Web 测试
- Azure Monitor 内除免费单位数之外存储的任何自定义指标

本文将不断更新有关新 Azure 监视和警报功能的链接和详细信息，以及工具的可用性，以帮助用户采用新的 Azure Monitor 平台。


## <a name="next-steps"></a>后续步骤

* 了解[新的统一 Azure Monitor](../../azure-monitor/overview.md)。
* 了解新的 [Azure 警报](../../azure-monitor/platform/alerts-overview.md)。
