---
title: Azure Application Insights 概览仪表板 | Azure Docs
description: 使用 Azure Application Insights 概览仪表板功能来监控应用程序。
services: application-insights
documentationcenter: ''
author: lingliw
manager: digimobile
ms.assetid: ea2a28ed-4cd9-4006-bd5a-d4c76f4ec20b
ms.service: application-insights
ms.workload: tbd
ms.tgt_pltfrm: ibiza
ms.topic: conceptual
ms.date: 6/4/2019
ms.author: v-lingwu
ms.openlocfilehash: 69a84a150b80961df717c3a0dfc9d21197233b24
ms.sourcegitcommit: fd927ef42e8e7c5829d7c73dc9864e26f2a11aaa
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 07/04/2019
ms.locfileid: "67562680"
---
# <a name="application-insights-overview-dashboard"></a>Application Insights 概述仪表板

Application Insights 一直都有一个总览窗格，可让用户快速、直接地评估应用程序的运行状况和性能。 新的概述仪表板提供了更快更灵活的体验。

## <a name="how-do-i-test-out-the-new-experience"></a>如何测试新体验？

现在默认情况下将启动新的概述仪表板：

![概览预览窗格](./media/overview-dashboard/overview.png)

## <a name="better-performance"></a>性能更好

时间范围选择已简化为简单的一键式界面。

![时间范围](./media/overview-dashboard/app-insights-overview-dashboard-03.png)

总体性能已大大提高。 只需单击一次即可访问常用功能，例如**搜索**和**分析**。 每个默认动态更新的 KPI 磁贴都可让你深入了解相应的 Application Insights 功能。 若要了解有关失败请求的详细信息，请在“调查”  标题下选择“失败”  ：

![失败数](./media/overview-dashboard/app-insights-overview-dashboard-04.png)

## <a name="application-dashboard"></a>应用程序仪表板

应用程序仪表板利用 Azure 内现有仪表板技术，为你的应用程序运行状况和性能提供了一个完全可自定义的单一窗格视图。

若要访问默认仪表板，请选择  左上角的“应用程序仪表板”。

![仪表板视图](./media/overview-dashboard/app-insights-overview-dashboard-05.png)

如果这是你第一次访问仪表板，它将启动默认视图：

![仪表板视图](./media/overview-dashboard/0001-dashboard.png)

如果你喜欢，可以保留默认视图。 或者，可以在仪表板中执行添加或删除操作，让仪表板契合团队的需求。

> [!NOTE]
> 所有有权访问 Application Insights 资源的用户共享相同的应用程序仪表板体验。 一个用户所做的更改将修改所有用户的视图。

若要导航回概览体验，只需选择：

![“概览”按钮](./media/overview-dashboard/app-insights-overview-dashboard-07.png)

## <a name="troubleshooting"></a>故障排除

如果选择“配置磁贴设置”  并将自定义时间范围设置为超过 31 天，则即使默认数据保留期为 90 天，仪表板也不会显示超过 31 天的数据。 目前没有针对此行为的解决方法。

## <a name="next-steps"></a>后续步骤

- [漏斗图](../../azure-monitor/app/usage-funnels.md)
- [保留](../../azure-monitor/app/usage-retention.md)
- [用户流](../../azure-monitor/app/usage-flows.md)




