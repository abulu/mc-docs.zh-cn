---
title: 排查 Azure Monitor 指标图表问题
description: 排查创建、自定义或解释指标图表时出现的问题
author: vgorbenko
services: azure-monitor
ms.service: azure-monitor
ms.topic: conceptual
ms.date: 04/23/2019
ms.author: vitalyg
ms.subservice: metrics
ms.openlocfilehash: 21f9dd760f4690eab606e70946a39e6495dfef17
ms.sourcegitcommit: fd927ef42e8e7c5829d7c73dc9864e26f2a11aaa
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 07/04/2019
ms.locfileid: "67562866"
---
# <a name="troubleshooting-metrics-charts"></a>排查指标图表问题

在 Azure 指标资源管理器中创建、自定义或解释图表时如果遇到问题，请参考本文。 如果你不熟悉指标，请参阅[指标资源管理器入门](metrics-getting-started.md)和[指标资源管理器的高级功能](metrics-charts.md)。 还可以查看配置的指标图表[示例](metric-chart-samples.md)。

## <a name="cant-find-your-resource-to-select-it"></a>找不到要选择的资源

你已单击“选择资源”按钮，但资源选取器对话框中未显示你的资源。 

**解决方案：** 只有在选择订阅和资源组之后，指标资源管理器才会列出可用的资源。 如果看不到你的资源：

1. 确保已在“订阅”下拉列表中选择正确的订阅。  如果你的订阅未列出，请单击“目录 + 订阅设置”，并添加包含资源的订阅。 

1. 确保已选择正确的资源组。
    > [!WARNING]
    > 为提供最佳性能，当你首次打开指标资源管理器时，“资源组”下拉列表中不会包含预先选择的资源组。  必须至少选择一个组才能看到任何资源。

## <a name="chart-shows-no-data"></a>图表未显示任何数据

有时，在选择正确的资源和指标后，图表可能不会显示任何数据。 此行为可能是由以下多种原因造成的：

### <a name="microsoftinsights-resource-provider-isnt-registered-for-your-subscription"></a>未为你的订阅注册 Microsoft.Insights 资源提供程序

需要在订阅中注册 *Microsoft.Insights* 资源提供程序才能浏览指标。 在许多情况下，该资源提供程序会自动注册（即，配置警报规则、自定义任何资源的诊断设置或配置自动缩放规则之后）。 如果未注册 Microsoft.Insights 资源提供程序，必须遵循 [Azure 资源提供程序和类型](../../azure-resource-manager/resource-manager-supported-services.md)中所述的步骤手动将其注册。

**解决方案：** 打开“订阅”>“资源提供程序”选项卡，检查是否为订阅注册了 *Microsoft.Insights*。  

### <a name="you-dont-have-sufficient-access-rights-to-your-resource"></a>你对资源没有足够的访问权限

在 Azure 中，对指标的访问由[基于角色的访问控制 (RBAC)](../../role-based-access-control/overview.md) 控制。 只有[监视读取者](../../role-based-access-control/built-in-roles.md#monitoring-reader)、[监视参与者](../../role-based-access-control/built-in-roles.md#monitoring-contributor)或[参与者](../../role-based-access-control/built-in-roles.md#contributor)的成员才能浏览任何资源的指标。

**解决方案：** 请确保你对要在其中浏览指标的资源拥有足够的权限。

### <a name="your-resource-didnt-emit-metrics-during-the-selected-time-range"></a>资源在选定的时间范围内未发出指标

某些资源不会持续发出指标。 例如，Azure 不会收集已停止的虚拟机的指标。 其他资源可能只在发生某种情况时才发出指标。 例如，显示事务处理时间的指标至少需要有一个事务发生。 如果在选定的时间范围内未发生任何事务，图表自然是空的。 此外，尽管 Azure 中的大部分指标每隔一分钟收集一次，但有些指标的收集频率更低。 请参阅指标文档，了解有关你正在尝试浏览的指标的更多详细信息。

**解决方案：** 将图表时间更改为更大的范围。 一开始可以使用较大的时间粒度“过去 30 天”（或依赖于“自动时间粒度”选项）。

### <a name="you-picked-a-time-range-greater-than-30-days"></a>选取的时间范围超过 30 天

[Azure 中的大多数指标将存储 93 天](data-platform-metrics.md#retention-of-metrics)。 但是，在任何单个图表中，只能查询不超过 30 天的数据。 此限制不适用于[基于日志的指标](../app/pre-aggregated-metrics-log-metrics.md#log-based-metrics)。

**解决方案：** 如果你看到空白图表或者图表仅显示一部分指标数据，请确认时间选取器中的开始日期与结束日期之差是否不超过 30 天间隔。

### <a name="all-metric-values-were-outside-of-the-locked-y-axis-range"></a>所有指标值超过了已锁定的 y 轴范围

[锁定图表 y 轴的边界](metrics-charts.md#lock-boundaries-of-chart-y-axis)时，你可能无意中造成图表显示区域不显示图表线条。 例如，如果 y 轴已锁定为 0% 到 50% 的范围，而指标包含 100% 的常量值，则线条始终会显示在可视区域的外部，使图表看上去是空白的。

**解决方案：** 确认图表的 y 轴边界是否未锁定在指标值的范围以外。 如果 y 轴边界已锁定，你可以暂时将其重置，以确保指标值不超过图表范围。 不建议使用 **sum**、**min** 和 **max** 聚合以自动粒度锁定图表的 y 轴范围，因为在调整浏览器窗口大小或者更改不同的屏幕分辨率时，图表值会随着粒度而变化。 切换粒度可能会使图表显示区域保持空白。

### <a name="you-are-looking-at-a-guest-os-metric-but-didnt-enable-azure-diagnostic-extension"></a>你正在查看来宾 OS 指标，但未启用 Azure 诊断扩展

收集**来宾 OS** 指标需要配置 Azure 诊断扩展，或使用资源的“诊断设置”面板来启用该扩展。 

**解决方案：** 如果 Azure 诊断扩展已启用，但你仍然无法看到指标，请遵循 [Azure 诊断扩展故障排除指南](diagnostics-extension-troubleshooting.md#metric-data-doesnt-appear-in-the-azure-portal)中所述的步骤。 另请参阅[无法选取来宾 OS 命名空间和指标](metrics-troubleshoot.md#cannot-pick-guest-os-namespace-and-metrics)的故障排除步骤

## <a name="error-retrieving-data-message-on-dashboard"></a>仪表板上显示“检索数据时出错”消息

如果仪表板是使用后来已弃用并已从 Azure 中删除的某个指标创建的，则可能会发生此问题。 若要确认是否存在这种情况，请打开资源的“指标”选项卡，然后检查指标选取器中的可用指标。  如果该指标未显示，则表示它已从 Azure 中删除。 一般情况下，如果某个指标已弃用，会有一个更好的新指标可以提供有关资源运行状况的类似透视图。

**解决方案：** 在仪表板上选取图表的备选指标来更新出错的磁贴。 可以[查看 Azure 服务的可用指标列表](metrics-supported.md)。

## <a name="chart-shows-dashed-line"></a>图表显示虚线

Azure 指标图表使用虚线样式来指示两个已知时间粒度数据点之间存在缺失值（也称为“null 值”）。 例如，如果你在时间选择器中选择了“1 分钟”时间粒度，但指标是在 07:26、07:27、07:29 和 07:30 报告的（请注意第二和第三个数据点之间的分钟间隔），则 07:27 和 07:29 数据点之间会以虚线连接，所有其他数据点之间以实线连接。 当指标使用 **count** 和 **sum** 聚合时，虚线将下降到零。 对于 **avg**、**min** 或 **max** 聚合，虚线将连接两个最接近的已知数据点。 此外，当图表最右侧或最左侧缺少数据时，虚线将朝缺失数据点的方向延长。
  ![指标插图](./media/metrics-troubleshoot/missing-data-point-line-chart.png)

**解决方案：** 此行为是设计使然。 这样可以识别缺失的数据点。 折线图能够出色地可视化高密度指标的趋势，但对于包含稀疏值的指标，可能很难解释，尤其是必须将值与时间粒度关联时。 虚线可以方便阅读这些图表，但如果图表仍不清晰，请考虑使用不同的图表类型查看指标。 例如，对同一个指标使用散点图可以清晰地显示每个时间粒度，因为它只会在某个值存在时才可视化某个点，并在该值缺失时统一跳过数据点：![指标插图](./media/metrics-troubleshoot/missing-data-point-scatter-chart.png)

   > [!NOTE]
   > 如果你仍然偏向于使用折线图来查看指标，将鼠标移到图表上可在鼠标指针位置突出显示数据点，这可能有助于评估时间粒度。

## <a name="chart-shows-unexpected-drop-in-values"></a>图表显示值出现意外的下降

在许多情况下，指标值的明显下降是图表上显示的数据造成的一种错觉。 如果图表显示最近几分钟的数据，则总和或计数下降可能会给你造成误解，因为 Azure 此时尚未收到或处理最后的指标数据点。 指标处理延迟在几分钟范围内变化，具体取决于所用的服务。 对于显示最近 1 分钟或 5 分钟粒度时间范围的图表，过去几分钟的值的下降会更明显：![指标插图](./media/metrics-troubleshoot/drop-in-values.png)

**解决方案：** 此行为是设计使然。 我们相信，即使数据是部分性的或者不完整的，在收到数据后立即显示数据比较有利。   这样，就可以更快地做出重要结论，并立即开始调查。 例如，对于显示失败次数的指标，查看部分值 X 可以判断，在给定的分钟内至少发生了 X 次失败。 然后可以立即开始调查问题，而不是等到图表中显示此分钟内发生的确切失败次数，确切的数字可能不如立即调查那么重要。 收到整个数据集后，图表将会更新，但此时，它可能还会显示更近时间内发生的新的不完整数据点。

## <a name="cannot-pick-guest-os-namespace-and-metrics"></a>无法选取来宾 OS 命名空间和指标

虚拟机和虚拟机规模集有两种类别的指标：Azure 托管环境收集的“虚拟机主机”指标，以及虚拟机上运行的[监视代理](agents-overview.md)收集的“来宾 OS”指标。   启用 [Azure 诊断扩展](diagnostics-extension-overview.md)即可安装监视代理。

默认情况下，来宾 OS 指标存储在 Azure 存储帐户中，可通过资源的“诊断设置”选项卡选取这些指标。  如果未收集来宾 OS 指标或者指标资源管理器无法访问这些指标，则你只会看到“虚拟机主机”指标命名空间： 

![图表上的](./media/metrics-troubleshoot/cannot-pick-guest-os-namespace.png)

**解决方案：** 如果在指标资源管理器中未看到“来宾 OS”命名空间和指标： 

1. 确认 [Azure 诊断扩展](diagnostics-extension-overview.md)已启用并配置为收集指标。
    > [!WARNING]
    > 无法使用 [Log Analytics 代理](agents-overview.md#log-analytics-agent)（也称为 Microsoft Monitoring Agent 或“MMA”）将“来宾 OS”指标发送到存储帐户。 

1. 验证存储帐户是否不受防火墙的保护。

1. 如果未收集指标，请遵循 [Azure 诊断扩展故障排除指南](diagnostics-extension-troubleshooting.md#metric-data-doesnt-appear-in-the-azure-portal)。

## <a name="next-steps"></a>后续步骤

* [了解如何开始使用指标资源管理器](metrics-getting-started.md)
* [了解指标资源管理器的高级功能](metrics-charts.md)
* [查看 Azure 服务的可用指标列表](metrics-supported.md)
* [查看已配置的图表示例](metric-chart-samples.md)
