---
title: 智能检测 - Azure Application Insights 中的异常卷异常上升 | Azure Docs
description: 使用 Azure Application Insights 监视应用程序异常，了解异常卷的异常模式。
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
ms.openlocfilehash: 133114842c3b8702a77e4e9dfb92913302b727aa
ms.sourcegitcommit: fd927ef42e8e7c5829d7c73dc9864e26f2a11aaa
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 07/04/2019
ms.locfileid: "67562670"
---
# <a name="abnormal-rise-in-exception-volume-preview"></a>异常卷的异常增加（预览）

Application Insights 自动分析应用程序中引发的异常，并对异常遥测中的异常模式发出警告。

此功能需要为应用[配置异常报告](/azure-monitor/app/asp-net-exceptions#set-up-exception-reporting)，除此之外，不需要其他特殊步骤。 在应用生成足够多的异常遥测数据后，此功能会激活。

## <a name="when-would-i-get-this-type-of-smart-detection-notification"></a>何时会收到此类型的智能检测通知？
对比前面七天计算的基线，如果应用显示某个特殊类型的异常数在一天内异常增加，则可能会收到此类型的通知。
机器学习算法广泛用于检测异常数的增加，同时考虑应用程序使用情况的自然增长。

## <a name="does-my-app-definitely-have-a-problem"></a>收到通知是否意味着我的应用肯定有问题？
不是，通知并不意味着应用肯定有问题。 虽然异常数大量增加通常指示应用程序出现问题，但这些异常也可能是良性的并得到应用程序的正确处理。

## <a name="how-do-i-fix-it"></a>如何解决问题？
通知包括诊断信息，以在诊断进程中提供支持：
1. **会审。** 通知会显示有多少用户或多少请求受到影响。 这可以帮助你对问题分配优先级。
2. **划分范围。** 该问题是影响所有流量，还是只影响某些操作？ 可以从通知中获取此信息。
3. **诊断。** 检测包括从中引发异常的方法以及异常类型的相关信息。 还可以使用链接到支持信息的相关项和报告，帮助进一步诊断问题。



