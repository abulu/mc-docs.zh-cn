---
title: 监视 Azure Site Recovery | Azure
description: 使用门户监视和排查 Azure Site Recovery 复制问题与操作
author: rockboyfor
manager: digimobile
ms.service: site-recovery
ms.topic: conceptual
origin.date: 03/18/2019
ms.date: 07/08/2019
ms.author: v-yeche
ms.openlocfilehash: 1d9171a7d5a7e4975880e3e12b8d099a6d7fbc7b
ms.sourcegitcommit: e575142416298f4d88e3d12cca58b03c80694a32
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 07/12/2019
ms.locfileid: "67861707"
---
# <a name="monitor-site-recovery"></a>监视 Site Recovery

本文介绍如何使用 Azure Site Recovery 中的内置监视功能进行监视和故障排除。 

## <a name="use-the-dashboard"></a>使用仪表板

1. 在保管库中，单击“概述”打开 Site Recovery 仪表板。  Site Recovery 和备份都有仪表板页，可在这些页之间切换。

    ![Site Recovery 仪表板](./media/site-recovery-monitor-and-troubleshoot/dashboard.png)

2. 该仪表板在单个位置合并了保管库的所有监视信息。 在该仪表板中，可以向下钻取到不同的区域。 

    ![Site Recovery 仪表板](./media/site-recovery-monitor-and-troubleshoot/site-recovery-overview-page.png)上获取。

3. 在“复制的项”中，单击“全部查看”可查看保管库中的所有服务器。  
4. 通过单击每个部分中的状态详细信息向下钻取。 在“基础结构”视图中，可按复制的计算机类型将监视信息排序。 

## <a name="monitor-replicated-items"></a>监视复制的项

“复制的项”部分显示保管库中已启用复制的所有计算机的运行状况。

**State** | **详细信息**
--- | ---
Healthy | 复制正常进行。 未检测到任何错误或警告症状。
警告 | 检测到一个或多个可能影响复制的警告症状。
关键 | 检测到一个或多个严重复制错误症状。<br/><br/> 这些错误症状通常指示复制处于停滞状态，或者复制进度跟不上数据更改速率。
不适用 | 目前预期服务器无法复制。 这可能包括已故障转移的计算机。

## <a name="monitor-test-failovers"></a>监视测试故障转移

可以查看保管库中计算机的测试故障转移状态。

- 我们建议每隔六个月在复制的计算机上至少运行测试故障转移一次。 这样，便可以在不中断生产环境的情况下，检查故障转移是否按预期工作。 
- 只有在成功完成故障转移以及故障转移后的清理过程之后，才将测试故障转移视为成功。

**State** | **详细信息**
--- | ---
建议的测试 | 自启用保护以来未进行测试故障转移的计算机。
已成功执行 | 已成功完成一次或多次测试故障转移的计算机。
不适用 | 目前不符合测试故障转移条件的计算机。 例如，已故障转移的计算机、正在进行初始复制/测试故障转移/故障转移的计算机。

## <a name="monitor-configuration-issues"></a>监视配置问题

“配置问题”部分显示可能影响到成功故障转移的问题列表。 

- 一个默认每隔 12 小时定期运行的验证程序操作将会检测配置问题（但不会检测软件更新可用性）。 单击“配置问题”部分标题旁边的刷新图标可以强制验证程序操作立即运行。 
- 单击相应的链接获取更多详细信息。 对于影响特定计算机的问题，请单击“目标配置”列中的“需要关注”。   详细信息包括补救措施的建议。

**State** | **详细信息**
--- | ---
缺少配置 | 缺少所需的设置，例如恢复网络或资源组。
缺少资源 | 指定的资源未找到，或者在订阅中不可用。 例如，已删除或迁移了资源。 受监视的资源包括目标资源组、目标 VNet/子网、日志/目标存储帐户、目标可用性集、目标 IP 地址。
订阅配额 |  将可用订阅资源配额的余量，与故障转移保管库中所有计算机所需的余量进行比较。<br/><br/> 如果资源不足，则报告不足的配额余量。<br/><br/> 配额是要监视的 VM 核心计数、VM 系列核心计数和网络接口卡 (NIC) 计数。
软件更新 | 新软件更新的可用性，以及有关即将过期的软件版本的信息。

## <a name="monitoring-errors"></a>监视错误 
“错误摘要”部分显示目前尚未解决的、可能影响保管库中服务器的复制的错误症状，以及受影响的计算机数目。 

- 该部分的开头显示影响本地基础结构组件的错误。 例如，未从本地配置服务器、VMM 服务器或 Hyper-V 主机上运行的 Azure Site Recovery 提供程序收到检测信号。
- 接下来显示影响已复制的服务器的复制错误症状。
- 表条目分别按错误严重性的降序以及受影响计算机数的降序排序。
- 参考受影响服务器数能够很好地了解单一根本问题是否影响了多台计算机。 例如，网络问题可能会影响复制到 Azure 的所有计算机。 
- 单个服务器上可能出现多个复制错误。 在这种情况下，每个错误症状会将该服务器计入到它所影响的服务器列表中。 解决问题后，复制参数将得到改善，而该错误将从计算机中清除。

## <a name="monitor-the-infrastructure"></a>监视基础架构。

“基础结构”视图显示参与复制的基础结构组件，以及服务器与 Azure 服务之间的连接运行状况。 

- 绿线表示连接正常。
- 带有叠加错误图标的红线指示存在一个或多个影响连接的错误症状。
-  将鼠标指针悬停在错误图标上会显示错误和受影响实体的数目。 单击图标会显示受影响实体的筛选列表。

    ![Site Recovery 基础结构视图（保管库）](./media/site-recovery-monitor-and-troubleshoot/site-recovery-vault-infra-view.png)

## <a name="tips-for-monitoring-the-infrastructure"></a>有关监视基础结构的提示

- 确保本地基础结构组件（配置服务器、进程服务器、VMM 服务器、Hyper-V 主机、VMware 计算机）运行最新版本的 Site Recovery 提供程序和/或代理。
- 若要使用基础结构视图的所有功能，应运行这些组件的[更新汇总 22](https://support.microsoft.com/help/4072852)。
- 若要使用基础结构视图，请选择适用于环境的复制方案。 可以在视图中向下钻取以查看更多详细信息。 下表显示了代表的方案。

    **方案** | **State**  | **视图可用？**
    --- |--- | ---
    **在本地站点之间复制** | 所有状态 | 否 
    **Azure 区域之间的 Azure VM 复制**  | 已启用复制/初始复制正在进行 | 是
    **Azure 区域之间的 Azure VM 复制** | 已故障转移/故障回复 | 否   
    **从 VMware 复制到 Azure** | 已启用复制/初始复制正在进行 | 是     
    **从 VMware 复制到 Azure** | 已故障转移/故障回复 | 否      
    **从 Hyper-V 复制到 Azure** | 已故障转移/故障回复 | 否

- 若要查看单个复制计算机的基础结构视图，请在保管库菜单中单击“复制的项”，然后选择一个服务器。   

### <a name="common-questions"></a>常见问题

**保管库基础结构视图中的虚拟机计数为何与“复制的项”中显示的总计数不同？**

保管库基础结构视图已根据复制方案划分了范围。 只有当前选定的复制方案中的计算机才包含在该视图中。 此外，我们只统计配置为复制到 Azure 的 VM。 已故障转移的计算机或者复制回到本地站点的计算机不会在该视图中统计。

**“概要”抽屈中显示的已复制项计数为何与仪表板上的已复制项总计数不同？**

只有已完成初始复制的计算机才会包含在“概要”抽屉显示的计数中。 “复制的项”中的总数包括保管库中的所有计算机，其中包括正在进行初始复制的计算机。

## <a name="monitor-recovery-plans"></a>监视恢复计划

在“恢复计划”部分，可以查看计划数目、创建新计划，以及修改现有计划。   

## <a name="monitor-jobs"></a>监视作业

“作业”部分反映 Site Recovery 操作的状态。 

- Azure Site Recovery 中的大多数操作以异步方式执行，将创建并使用一个跟踪作业来跟踪操作进度。 
- 作业对象包含跟踪操作状态和进度的全部所需信息。 

按如下所述监视作业：

1. 在仪表板中转到“作业”部分，可以看到过去 24 小时内已完成的、正在进行的或等待输入的作业的摘要。  可以单击任一状态获取相关作业的详细信息。
2. 单击“全部查看”可查看过去 24 小时内的所有作业。 

    > [!NOTE]
    > 还可以从保管库菜单 >“Site Recovery 作业”访问作业信息。  

2. “Site Recovery 作业”列表中显示了作业列表。  在顶部菜单中，可以获取特定作业的错误详细信息、根据特定的条件筛选作业列表，以及将选定作业的详细信息导出到 Excel。
3. 单击某个作业可深入查看更多信息。 

## <a name="monitor-virtual-machines"></a>监视虚拟机

在附加的仪表板中，可以在虚拟机页中监视计算机。 

1. 在保管库中，单击“复制的项”获取复制的计算机列表。   或者，可以单击仪表板页上的任何带范围快捷方式来查看受保护项的筛选列表。

    ![Site Recovery 中“复制的项”列表视图](./media/site-recovery-monitor-and-troubleshoot/site-recovery-virtual-machine-list-view.png)

2. 在“复制的项”页上，可以查看和筛选器信息。  在顶部的操作菜单中，可以针对特定的计算机执行操作，包括运行测试故障转移，或查看特定的错误。
3. 单击“列”可显示其他列，例如，显示 RPO、目标配置问题和复制错误。 
4. 单击“筛选器”可以根据复制运行状况或特定复制策略等特定参数来查看信息。 
5. 右键单击某个计算机可以启动操作，例如，执行测试故障转移，或查看与它关联的特定错误详细信息。
6. 单击某个计算机可以深入查看其更多详细信息。 详细信息包括：
    - **复制信息**：计算机的当前状态和运行状况。
    - **RPO**（恢复点目标）：虚拟机的当前 RPO，以及上次计算 RPO 的时间。
    - **恢复点**：计算机的最新可用恢复点。
    - **故障转移就绪性**：指示是否对该计算机运行了测试故障转移、计算机上运行的代理版本（适用于运行移动服务的计算机）和任何配置问题。
    - **错误**：列出当前在计算机上观察到的复制错误症状，以及可能的原因/措施。
    - **事件**：影响计算机的最近事件列表，按时间顺序列出。 错误详细信息显示当前可观测到的错误症状，而事件是影响了计算机的问题的历史记录。
    - **基础结构视图**：显示将计算机复制到 Azure 时方案的基础结构状态。

     ![Azure Site Recovery 中复制的项详细信息/概述](./media/site-recovery-monitor-and-troubleshoot/site-recovery-virtual-machine-details.png)

### <a name="common-questions"></a>常见问题

**RPO 与最新可用的恢复点有何不同？**

- Site Recovery 使用多步骤异步过程将计算机复制到 Azure。
- 在复制的倒数第二步，计算机上发生的最近更改将连同元数据一起复制到日志/缓存存储帐户。
- 这些更改将连同用于标识可恢复点的标记一起写入到目标区域中的存储帐户。
-  现在，Site Recovery 可以生成虚拟机的可恢复点。
- 此时，表示已符合 RPO，可将更改上传到存储帐户。 换而言之，此时的计算机 RPO 等于从对应于可恢复点的时间戳开始消逝的时间。
- 现在，Site Recovery 会从存储帐户中选取上传的数据，并将其应用到为计算机创建的副本磁盘。
- 然后，Site Recovery 生成一个恢复点，并使此恢复点可用于故障转移时的恢复操作。 因此，最新可用恢复点表示与已处理并已应用到副本磁盘的最新恢复点对应的时间戳。

> [!NOTE]
> 如果复制源计算机或本地基础结构服务器上的系统时间不正确，则会导致计算出的 RPO 值有偏差。 为准确报告 RPO，请确保所有服务器和计算机上的系统时钟准确。 

<!--Not Available on ## Subscribe to email notifications-->
<!-- Update_Description: update meta propreties -->