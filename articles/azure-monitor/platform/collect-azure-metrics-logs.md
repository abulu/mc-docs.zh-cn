---
title: 将 Azure 服务日志和指标收集到 Log Analytics 工作区中 | Azure Docs
description: 在 Azure 资源上配置诊断以将日志和指标写入 Azure Monitor 中的 Log Analytics 工作区。
services: log-analytics
documentationcenter: ''
author: lingliw
manager: digimobile
editor: ''
ms.assetid: 84105740-3697-4109-bc59-2452c1131bfe
ms.service: log-analytics
ms.workload: na
ms.tgt_pltfrm: na
ms.topic: conceptual
ms.date: 6/4/2019
ms.author: v-lingwu
ms.openlocfilehash: 8869eb7b121cb5797d6064e680acf52c33817da6
ms.sourcegitcommit: fd927ef42e8e7c5829d7c73dc9864e26f2a11aaa
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 07/04/2019
ms.locfileid: "67562411"
---
# <a name="collect-azure-service-logs-and-metrics-into-log-analytics-workspace-in-azure-monitor"></a>将 Azure 服务日志和指标收集到 Azure Monitor 的 Log Analytics 工作区中

有四种不同方式可收集 Azure 服务的日志和指标：

1. 将 Azure 诊断定向到 Azure Monitor 的 Log Analytics 工作区（下表中的“诊断”） 
2. 从 Azure 诊断到 Azure 存储，再到 Azure Monitor 的 Log Analytics 工作区（下表中的“存储”） 
3. Azure 服务的连接器（下表中的*连接器*）
4. 使用脚本收集，然后将数据发布到 Azure Monitor 的 Log Analytics 工作区中（下表中的空白，用于未列出的服务）


| 服务                 | 资源类型                           | 日志        | 指标     | 解决方案 |
| --- | --- | --- | --- | --- |
| 应用程序网关    | Microsoft.Network/applicationGateways   | 诊断 | 诊断 | Azure 应用程序网关分析 |
| Application insights    |                                         | 连接器   | 连接器   | [Application Insights Connector](https://blogs.technet.microsoft.com/msoms/2016/09/26/application-insights-connector-in-oms/)（预览版） |
| 自动化帐户     | Microsoft.Automation/AutomationAccounts | 诊断 |             | [详细信息](../../automation/automation-manage-send-joblogs-log-analytics.md)|
| 批处理帐户          | Microsoft.Batch/batchAccounts           | 诊断 | 诊断 | |
| 经典云服务  |                                         | 存储     |             | [详细信息](azure-storage-iis-table.md) |
| 认知服务      | Microsoft.CognitiveServices/accounts    |             | 诊断 | |
| Data Lake Analytics     | Microsoft.DataLakeAnalytics/accounts    | 诊断 |             | |
| Data Lake Store         | Microsoft.DataLakeStore/accounts        | 诊断 |             | |
| 事件中心命名空间     | Microsoft.EventHub/namespaces           | 诊断 | 诊断 | |
| IoT 中心                | Microsoft.Devices/IotHubs               |             | 诊断 | |
| 密钥保管库               | Microsoft.KeyVault/vaults               | 诊断 |             | [密钥保管库分析](../insights/azure-key-vault.md) |
| 负载均衡器          | Microsoft.Network/loadBalancers         | 诊断 |             |  |
| Logic Apps              | Microsoft.Logic/workflows <br> Microsoft.Logic/integrationAccounts | 诊断 | 诊断 | |
| 网络安全组 | Microsoft.Network/networksecuritygroups | 诊断 |             | Azure 网络安全组分析 |
| 恢复保管库         | Microsoft.RecoveryServices/vaults       |             |             | [Azure 恢复服务分析（预览版）](https://github.com/krnese/AzureDeploy/blob/master/OMS/MSOMS/Solutions/recoveryservices/)|
| 搜索服务         | Microsoft.Search/searchServices         | 诊断 | 诊断 | |
| 服务总线命名空间   | Microsoft.ServiceBus/namespaces         | 诊断 | 诊断 | [Service Fabric 分析（预览版）](https://github.com/Azure/azure-quickstart-templates/tree/master/oms-servicebus-solution)|
| SQL (v12)               | Microsoft.Sql/servers/databases <br> Microsoft.Sql/servers/elasticPools |             | 诊断 | [Azure SQL Analytics（预览版）](../insights/azure-sql.md) |
| 存储                 |                                         |             | 脚本      | [Azure 存储分析（预览版）](https://github.com/Azure/azure-quickstart-templates/tree/master/oms-azure-storage-analytics-solution) |
| 虚拟机        | Microsoft.Compute/virtualMachines       | 分机   | 分机 <br> 诊断  | |
| 虚拟机规模集 | Microsoft.Compute/virtualMachines <br> Microsoft.Compute/virtualMachineScaleSets/virtualMachines |             | 诊断 | |
| Web 服务器场        | Microsoft.Web/serverfarms               |             | 诊断 | |
| 网站               | Microsoft.Web/sites <br> Microsoft.Web/sites/slots |             | 诊断 | [Azure Web 应用分析（预览版）](/azure-monitor/insights/azure-web-apps-analytics) |


> [!NOTE]
> 若要监视 Azure 虚拟机（Linux 和 Windows），建议安装 [Log Analytics VM 扩展](../../azure-monitor/learn/quick-collect-azurevm.md)。 该代理提供从虚拟机中收集到的见解。 对于虚拟机规模集，还可以使用扩展。
>
>

## <a name="azure-diagnostics-direct-to-log-analytics"></a>将 Azure 诊断定向到 Log Analytics
许多 Azure 资源都能将诊断日志和指标直接写入到 Azure Monitor 的 Log Analytics 工作区，这是收集数据进行分析的首选方法。 使用 Azure 诊断时，数据将立即写入到工作区，而无需先将数据写入到存储。

支持 [Azure Monitor](../../azure-monitor/overview.md) 的 Azure 资源可以直接向 Log Analytics 工作区发送其日志和指标。

> [!NOTE]
> 当前不支持通过诊断设置将多维指标发送到 Log Analytics 工作区。 多维指标将按平展后的单维指标导出，并跨维值聚合。
>
> 例如：  可以基于每个队列级别浏览和绘制事件中心上的“传入消息”指标。 但是，当通过诊断设置导出时，该指标将表示为事件中心的所有队列中的所有传入消息。
>
>

* 有关可用指标的详细信息，请参阅 [Azure 监视器支持的指标](../../azure-monitor/platform/metrics-supported.md)。
* 有关可用日志的详细信息，请参阅[诊断日志支持的服务和架构](diagnostic-logs-schema.md)。

### <a name="enable-diagnostics-with-powershell"></a>使用 PowerShell 启用诊断

[!INCLUDE [updated-for-az](../../../includes/updated-for-az.md)]

下面的 PowerShell 示例演示如何使用 [Set-AzDiagnosticSetting](https://docs.microsoft.com/powershell/module/Az.Monitor/Set-AzDiagnosticSetting) 对网络安全组启用诊断。 同一方法适用于所有受支持的资源 - 将 `$resourceId` 设置为要为其启用诊断的资源的资源 ID。

```powershell
$workspaceId = "/subscriptions/d2e37fee-1234-40b2-5678-0b2199de3b50/resourcegroups/oi-default-east-china/providers/microsoft.operationalinsights/workspaces/rollingbaskets"

$resourceId = "/SUBSCRIPTIONS/ec11ca60-1234-491e-5678-0ea07feae25c/RESOURCEGROUPS/DEMO/PROVIDERS/MICROSOFT.NETWORK/NETWORKSECURITYGROUPS/DEMO"

Set-AzDiagnosticSetting -ResourceId $ResourceId  -WorkspaceId $workspaceId -Enabled $true
```

### <a name="enable-diagnostics-with-resource-manager-templates"></a>使用 Resource Manager 模板启用诊断

要在资源创建时为其启用诊断，并将诊断发送到 Log Analytics 工作区，可以使用如下模板。 此示例适用于自动化帐户，但适用于所有受支持的资源类型。

```json
        {
            "type": "Microsoft.Automation/automationAccounts/providers/diagnosticSettings",
            "name": "[concat(parameters('omsAutomationAccountName'), '/', 'Microsoft.Insights/service')]",
            "apiVersion": "2015-07-01",
            "dependsOn": [
                "[concat('Microsoft.Automation/automationAccounts/', parameters('omsAutomationAccountName'))]",
                "[concat('Microsoft.OperationalInsights/workspaces/', parameters('omsWorkspaceName'))]"
            ],
            "properties": {
                "workspaceId": "[resourceId('Microsoft.OperationalInsights/workspaces', parameters('omsWorkspaceName'))]",
                "logs": [
                    {
                        "category": "JobLogs",
                        "enabled": true
                    },
                    {
                        "category": "JobStreams",
                        "enabled": true
                    }
                ]
            }
        }
```

[!INCLUDE [log-analytics-troubleshoot-azure-diagnostics](../../../includes/log-analytics-troubleshoot-azure-diagnostics.md)]

## <a name="azure-diagnostics-to-storage-then-to-log-analytics"></a>将 Azure 诊断定向到存储，然后再定向到 Log Analytics

要从某些资源中收集日志，可以将日志发送到 Azure 存储，再将 Log Analytics 工作区配置为从存储中读取日志。

Azure Monitor 可使用此方法从 Azure 存储收集以下资源和日志的诊断信息：

| Resource | 日志 |
| --- | --- |
| Service Fabric |ETWEvent <br> 操作事件 <br> 可靠角色事件 <br> 可靠服务事件 |
| 虚拟机 |Linux Syslog <br> Windows 事件 <br> IIS 日志 <br> Windows ETWEvent |
| Web 角色 <br> 辅助角色 |Linux Syslog <br> Windows 事件 <br> IIS 日志 <br> Windows ETWEvent |

> [!NOTE]
> 当用户向存储帐户发送诊断时，以及当 Log Analytics 工作区从存储帐户读取数据时，系统会针对存储和事务收取正常 Azure 数据费率。
>
>

若要详细了解 Azure Monitor 如何收集这些日志，请参阅[使用适用于 IIS 的 Blob 存储和适用于事件的表存储](azure-storage-iis-table.md)。

## <a name="connectors-for-azure-services"></a>Azure 服务的连接器

Application Insights 有连接器，它允许 Application Insights 收集要发送给 Log Analytics 工作区的数据。

详细了解 [Application Insights 连接器](https://blogs.technet.microsoft.com/msoms/2016/09/26/application-insights-connector-in-oms/)。

## <a name="scripts-to-collect-and-post-data-to-log-analytics-workspace"></a>使用脚本收集数据并将数据发布到 Log Analytics 工作区

对于未提供用于将日志和指标发送到 Log Analytics 工作区的直接方式的 Azure 服务，可以使用 Azure 自动化脚本来收集日志和指标。 然后，该脚本可以使用[数据收集器 API](../../azure-monitor/platform/data-collector-api.md) 将数据发送到工作区

Azure 模板库有[使用 Azure 自动化的示例](https://azure.microsoft.com/resources/templates/?term=OMS)，可从服务收集数据并将数据发送到 Azure Monitor。

## <a name="next-steps"></a>后续步骤

* [使用适用于 IIS 的 blob 存储和适用于事件的表存储](azure-storage-iis-table.md)，以便读取将诊断写入到表存储的 Azure 服务的日志或写入到 blob 存储的 IIS 日志。
* [启用解决方案](../../azure-monitor/insights/solutions.md)深入分析数据。
* [使用搜索查询](../../azure-monitor/log-query/log-query-overview.md)分析数据。




