---
title: 为混合环境启用 Azure Monitor（预览版） | Microsoft Docs
description: 本文介绍如何为包含一个或多个虚拟机的混合云环境启用用于 VM 的 Azure Monitor。
services: azure-monitor
documentationcenter: ''
author: lingliw
manager: digimobile
editor: ''
ms.assetid: ''
ms.service: azure-monitor
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 06/07/2019
ms.author: v-lingwu
ms.openlocfilehash: c5815ee4451122eba62a375bcd61ccbb811541e0
ms.sourcegitcommit: fd927ef42e8e7c5829d7c73dc9864e26f2a11aaa
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 07/04/2019
ms.locfileid: "67562954"
---
# <a name="enable-azure-monitor-for-vms-preview-for-a-hybrid-environment"></a>为混合环境启用用于 VM 的 Azure Monitor（预览版）

[!INCLUDE [updated-for-az](../../../includes/updated-for-az.md)]

本文介绍如何为数据中心或其他云环境中托管的虚拟机或物理计算机启用用于 VM 的 Azure Monitor（预览版）。 在此过程结束时，你将已成功地开始监视环境中的虚拟机，并了解这些虚拟机是否会遇到任何性能或可用性问题。 

在开始前，请务必查看[先决条件](vminsights-enable-overview.md)，并验证订阅和资源是否满足相关要求。

[!INCLUDE [log-analytics-agent-note](../../../includes/log-analytics-agent-note.md)]

>[!NOTE]
>用于 VM 的 Azure Monitor 映射依赖项代理本身不传输任何数据，它不需要对防火墙或端口做出任何更改。 映射数据始终由 Log Analytics 代理传输到 Azure Monitor 服务 - 要么采用直接传输的方式，要么通过 [Operations Management Suite 网关](../../azure-monitor/platform/gateway.md)进行传输（如果 IT 安全策略不允许网络中的计算机连接到 Internet）。

完成此任务的步骤概述如下：

1. 安装用于 Windows 或 Linux 的 Log Analytics 代理。 安装代理之前，请查看 [Log Analytics 代理概述](../platform/log-analytics-agent.md)一文，以了解系统先决条件和部署方法。

2. 下载并安装适用于 [Windows](https://aka.ms/dependencyagentwindows) 或 [Linux](https://aka.ms/dependencyagentlinux) 的用于 VM 的 Azure Monitor 映射依赖项代理。

3. 启用性能计数器收集。

4. 部署用于 VM 的 Azure Monitor。

## <a name="install-the-dependency-agent-on-windows"></a>在 Windows 上安装依赖项代理
可通过运行 `InstallDependencyAgent-Windows.exe` 在 Windows 计算机上手动安装 Dependency Agent。 如果在没有任何选项的情况下运行此可执行文件，它将启动一个安装向导，以交互方式指导用户安装代理。

>[!NOTE]
>需要*管理员*特权才能安装或卸载代理。

下表突出显示了通过命令行安装代理时支持的参数。

| 参数 | 说明 |
|:--|:--|
| /? | 返回命令行选项的列表。 |
| /S | 执行无需用户交互的无提示安装。 |

例如，若要使用 `/?` 参数运行安装程序，请输入 **InstallDependencyAgent-Windows.exe /?** 。

默认情况下，Windows Dependency Agent 的文件在 *C:\Program Files\Microsoft Dependency Agent* 中安装。 如果在安装完成后依赖项代理无法启动，请查看日志以获取详细的错误信息。 日志目录为 *%Programfiles%\Microsoft Dependency Agent\logs*。

## <a name="install-the-dependency-agent-on-linux"></a>在 Linux 上安装 Dependency Agent
通过 *InstallDependencyAgent-Linux64.bin*（具有自解压二进制文件的 Shell 脚本）在 Linux 服务器上安装 Dependency Agent。 可使用 `sh` 来运行该文件或将执行权限添加到文件本身。

>[!NOTE]
> 需要根目录访问才能安装或配置代理。
>

| 参数 | 说明 |
|:--|:--|
| -help | 获取命令行选项列表。 |
| -s | 执行无提示安装，无用户提示。 |
| --check | 检查权限和操作系统，但不安装代理。 |

例如，若要使用 `-help` 参数运行安装程序，请输入 **InstallDependencyAgent-Linux64.bin -help**。

通过运行命令 `sh InstallDependencyAgent-Linux64.bin` 来以 root 用户身份安装 Linux 依赖项代理。

如果 Dependency Agent 无法启动，请检查日志以获取详细的错误信息。 在 Linux 代理上，日志目录是 */var/opt/microsoft/dependency-agent/log*。

Dependency Agent 的文件放置在以下目录中：

| 文件 | 位置 |
|:--|:--|
| 核心文件 | /opt/microsoft/dependency-agent |
| 日志文件 | /var/opt/microsoft/dependency-agent/log |
| 配置文件 | /etc/opt/microsoft/dependency-agent/config |
| 服务可执行文件 | /opt/microsoft/dependency-agent/bin/microsoft-dependency-agent<br>/opt/microsoft/dependency-agent/bin/microsoft-dependency-agent-manager |
| 二进制存储文件 | /var/opt/microsoft/dependency-agent/storage |

## <a name="enable-performance-counters"></a>启用性能计数器
如果解决方案引用的 Log Analytics 工作区尚未配置为收集解决方案所需的性能计数器，则需要启用性能计数器。 为此，可以采用下面两种方式之一：
* 手动方式，如 [Log Analytics 中的 Windows 和 Linux 性能数据源](../../azure-monitor/platform/data-sources-performance-counters.md)所述
* 通过下载并运行可从 [Azure PowerShell 库](https://www.powershellgallery.com/packages/Enable-VMInsightsPerfCounters/1.1)获取的 PowerShell 脚本

## <a name="deploy-azure-monitor-for-vms"></a>部署用于 VM 的 Azure Monitor
此方法包含一个 JSON 模板，其中指定了用于在 Log Analytics 工作区中启用解决方案组件的配置。

如果不知道如何使用模板部署资源，请参阅以下内容：
* [使用 Resource Manager 模板和 Azure PowerShell 部署资源](../../azure-resource-manager/resource-group-template-deploy.md)
* [使用资源管理器模板和 Azure CLI 部署资源](../../azure-resource-manager/resource-group-template-deploy-cli.md)

若要使用 Azure CLI，首先需要在本地安装并使用 CLI。 必须运行 Azure CLI 2.0.27 版或更高版本。 若要确定版本，请运行 `az --version`。 若要安装或升级 Azure CLI，请参阅[安装 Azure CLI](/cli/install-azure-cli)。

### <a name="create-and-execute-a-template"></a>创建和执行模板

1. 将以下 JSON 语法复制并粘贴到该文件中：

    ```json
    {
        "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
        "contentVersion": "1.0.0.0",
        "parameters": {
            "WorkspaceName": {
                "type": "string"
            },
            "WorkspaceLocation": {
                "type": "string"
            }
        },
        "resources": [
            {
                "apiVersion": "2017-03-15-preview",
                "type": "Microsoft.OperationalInsights/workspaces",
                "name": "[parameters('WorkspaceName')]",
                "location": "[parameters('WorkspaceLocation')]",
                "resources": [
                    {
                        "apiVersion": "2015-11-01-preview",
                        "location": "[parameters('WorkspaceLocation')]",
                        "name": "[concat('ServiceMap', '(', parameters('WorkspaceName'),')')]",
                        "type": "Microsoft.OperationsManagement/solutions",
                        "dependsOn": [
                            "[concat('Microsoft.OperationalInsights/workspaces/', parameters('WorkspaceName'))]"
                        ],
                        "properties": {
                            "workspaceResourceId": "[resourceId('Microsoft.OperationalInsights/workspaces/', parameters('WorkspaceName'))]"
                        },

                        "plan": {
                            "name": "[concat('ServiceMap', '(', parameters('WorkspaceName'),')')]",
                            "publisher": "Microsoft",
                            "product": "[Concat('OMSGallery/', 'ServiceMap')]",
                            "promotionCode": ""
                        }
                    },
                    {
                        "apiVersion": "2015-11-01-preview",
                        "location": "[parameters('WorkspaceLocation')]",
                        "name": "[concat('InfrastructureInsights', '(', parameters('WorkspaceName'),')')]",
                        "type": "Microsoft.OperationsManagement/solutions",
                        "dependsOn": [
                            "[concat('Microsoft.OperationalInsights/workspaces/', parameters('WorkspaceName'))]"
                        ],
                        "properties": {
                            "workspaceResourceId": "[resourceId('Microsoft.OperationalInsights/workspaces/', parameters('WorkspaceName'))]"
                        },
                        "plan": {
                            "name": "[concat('InfrastructureInsights', '(', parameters('WorkspaceName'),')')]",
                            "publisher": "Microsoft",
                            "product": "[Concat('OMSGallery/', 'InfrastructureInsights')]",
                            "promotionCode": ""
                        }
                    }
                ]
            }
        ]
    }
    ```

1. 使用 *installsolutionsforvminsights.json* 文件名将此文件保存到某个本地文件夹中。

1. 捕获 *WorkspaceName*、*ResourceGroupName* 和 *WorkspaceLocation* 的值。 *WorkspaceName* 的值为 Log Analytics 工作区的名称。 *WorkspaceLocation* 的值是在其中定义工作区的区域。

1. 现在，可以使用以下 PowerShell 命令部署此模板：

    ```powershell
    New-AzResourceGroupDeployment -Name DeploySolutions -TemplateFile InstallSolutionsForVMInsights.json -ResourceGroupName ResourceGroupName> -WorkspaceName <WorkspaceName> -WorkspaceLocation <WorkspaceLocation - example: eastus>
    ```

    配置更改可能需要几分钟才能完成。 在完成后，系统会显示如下所示的消息，其中包括相应的结果：

    ```powershell
    provisioningState       : Succeeded
    ```
   启用监视后，可能需要约 10 分钟才能查看混合计算机的运行状况和指标。

## <a name="next-steps"></a>后续步骤

既然虚拟机已启用了监视，此信息在用于 VM 的 Azure Monitor 中可供分析。
 
- 若要了解如何使用运行状况功能，请参阅[查看用于 VM 的 Azure Monitor 的运行状况](vminsights-health.md)。
- 若要查看已发现的应用程序依赖项，请参阅[查看用于 VM 的 Azure Monitor 映射](vminsights-maps.md)。
- 若要通过 VM 的性能了解瓶颈和整体利用率，请参阅[查看 Azure VM 性能](vminsights-performance.md)。
- 若要查看已发现的应用程序依赖项，请参阅[查看用于 VM 的 Azure Monitor 映射](vminsights-maps.md)。