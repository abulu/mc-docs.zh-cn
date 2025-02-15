---
title: 在 Azure 中使用 DPM/Azure 备份服务器保护 SharePoint 场
description: 本文概述如何在 Azure 中使用 DPM/Azure 备份服务器保护 SharePoint 场
services: backup
author: lingliw
manager: digimobile
ms.service: backup
ms.topic: conceptual
origin.date: 10/18/2018
ms.date: 11/26/2018
ms.author: v-lingwu
ms.openlocfilehash: 42c1f839785bbc41518d29d57b76e41138011737
ms.sourcegitcommit: 68f7c41974143a8f7bd9b7a54acf41c09893e587
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 07/19/2019
ms.locfileid: "68332031"
---
# <a name="back-up-a-sharepoint-farm-to-azure"></a>将 SharePoint 场备份到 Azure
使用 System Center Data Protection Manager (DPM) 将 SharePoint 场备份到 Azure，其方法与备份其他数据源极为类似。 Azure 备份提供灵活的备份计划来创建每日、每周、每月或每年备份点，并提供适用于各种备份点的保留策略选项。 利用 DPM，不仅可以存储本地磁盘副本以实现快速的恢复时间目标 (RTO)，还可以将副本存储到 Azure 进行经济高效的长期保留。

## <a name="sharepoint-supported-versions-and-related-protection-scenarios"></a>SharePoint 支持的版本与相关保护方案
DPM 的 Microsoft Azure 备份支持以下方案：

| 工作负载 | 版本 | Sharepoint 部署 | DPM 部署类型 | DPM - System Center 2012 R2 | 保护和恢复 |
| --- | --- | --- | --- | --- | --- |
| SharePoint |SharePoint 2013、SharePoint 2010、SharePoint 2007、SharePoint 3.0 |作为物理服务器或 Hyper-V/VMware 虚拟机部署的 SharePoint <br> -------------- <br> SQL AlwaysOn |物理服务器或本地 Hyper-V 虚拟机 |支持从更新汇总 5 备份到 Azure |保护 SharePoint 场恢复选项：从磁盘恢复点恢复场、数据库、文件或列表项。  从 Azure 恢复点恢复场和数据库。 |

## <a name="before-you-start"></a>开始之前
在将 SharePoint 场备份到 Azure 之前，需要确保满足几个条件。

### <a name="prerequisites"></a>先决条件
在继续之前，请确保符合使用 Azure 备份保护工作负荷的所有[先决条件](backup-azure-dpm-introduction.md#prerequisites-and-limitations) 。 先决条件包括如下任务：创建备份保管库、下载保管库凭据、安装 Azure 备份代理，以及在保管库中注册 DPM/Azure 备份服务器。

### <a name="dpm-agent"></a>DPM 代理
必须在运行 SharePoint Server 或 SQL Server 的服务器以及属于 SharePoint 场的任何其他服务器上安装 DPM 代理。 有关如何设置保护代理的详细信息，请参阅[设置保护代理](https://technet.microsoft.com/library/hh758034\(v=sc.12\).aspx)。  唯一的例外是，只能在单个 Web 前端 (WFE) 服务器上安装代理。 DPM 只需将一台 WFE 服务器上的代理作为保护的入口点。

### <a name="sharepoint-farm"></a>SharePoint 场
针对场中的每 1000 万个项，必须有至少 2 GB 的卷空间用于放置 DPM 文件夹。 此空间对目录生成是必要的。 为了使 DPM 恢复特定项（网站集合、站点、列表、文档库、文件夹、单个文档与列表项），目录生成将创建一个包含在每个内容数据库中的 URL 列表。 可以在 DPM 管理员控制台的“恢复”任务区域中，查看“可恢复项”窗格中的 URL 列表  。

### <a name="sql-server"></a>SQL Server
DPM 以 LocalSystem 帐户的形式运行。 若要备份 SQL Server 数据库，DPM 需要对运行 SQL Server 的服务器的帐户具有 sysadmin 特权。 备份之前，先在运行 SQL Server 的服务器上将 NT AUTHORITY\SYSTEM 设置为 *sysadmin*。

如果 SharePoint 场有使用 SQL Server 别名配置的 SQL Server 数据库，请在 DPM 要保护的前端 Web 服务器上安装 SQL Server 客户端组件。

### <a name="sharepoint-server"></a>SharePoint Server
尽管性能取决于许多因素，例如 SharePoint 场的大小，但一般做法是使用一台 DPM 服务器来保护 25 TB 的 SharePoint 场。

### <a name="dpm-update-rollup-5"></a>DPM 更新汇总 5
要开始在 Azure 上保护 SharePoint 场，需要安装 DPM 更新汇总 5 或更高版本。 如果使用 SQL AlwaysOn 配置 SharePoint 场，更新汇总 5 将提供在 Azure 上保护该场的功能。
有关详细信息，请参阅介绍 [DPM 更新汇总 5](https://blogs.technet.com/b/dpm/archive/2015/02/11/update-rollup-5-for-system-center-2012-r2-data-protection-manager-is-now-available.aspx) 的博客文章

### <a name="whats-not-supported"></a>不支持的功能
* 保护 SharePoint 场的 DPM 不会保护搜索索引或应用程序服务数据库。 需要单独为这些数据库配置保护。
* DPM 不提供横向扩展文件服务器 (SOFS) 共享托管的 SharePoint SQL Server 数据库备份。

## <a name="configure-sharepoint-protection"></a>配置 SharePoint 保护
必须使用 ConfigureSharePoint.exe 配置 SharePoint VSS 编写器服务（WSS 编写器服务），才能使用 DPM 保护 SharePoint  。

可以在前端 Web 服务器的 [DPM 安装路径]\bin 文件夹中找到 ConfigureSharePoint.exe  。 此工具可将 SharePoint 场的凭据提供给保护代理。 应在单个 WFE 服务器上运行该工具。 如果有多个 WFE 服务器，在配置保护组时，请只选择其中一个。

### <a name="to-configure-the-sharepoint-vss-writer-service"></a>配置 SharePoint VSS 写入器服务
1. 在 WFE 服务器上的命令提示符下，切换到 [DPM 安装路径]\bin\
2. 输入 ConfigureSharePoint-EnableSharePointProtection。
3. 输入场管理员凭据。 此帐户应是 WFE 服务器上本地管理员组的成员。 如果场管理员不是本地管理员，请在 WFE 服务器上授予以下权限：
   * 授予 WSS_Admin_WPG 组对 DPM 文件夹 (%Program Files%\Microsoft Data Protection Manager\DPM) 的完全控制权。
   * 授予 WSS_Admin_WPG 组对 DPM 注册表项 (HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Microsoft Data Protection Manager) 的读取访问权限。

> [!NOTE]
> 每当 SharePoint 场管理员凭据发生更改时，都需要重新运行 ConfigureSharePoint.exe。
> 
> 

## <a name="back-up-a-sharepoint-farm-by-using-dpm"></a>使用 DPM 备份 SharePoint 场
如前所述配置 DPM 和 SharePoint 场之后，可以使用 DPM 保护 SharePoint。

### <a name="to-protect-a-sharepoint-farm"></a>保护 SharePoint 场
1. 在 DPM 管理员控制台的“保护”选项卡中，单击“新建”   。
    ![新建保护选项卡](./media/backup-azure-backup-sharepoint/dpm-new-protection-tab.png)

2. 在“创建新保护组”向导的“选择保护组类型”页上，选择“服务器”，并单击“下一步”     。

    ![选择保护组类型](./media/backup-azure-backup-sharepoint/select-protection-group-type.png)

3. 在“选择组成员”屏幕上，选中要保护的 SharePoint 服务器对应的复选框，并单击“下一步”   。

    ![选择组成员](./media/backup-azure-backup-sharepoint/select-group-members2.png)

   > [!NOTE]
   > 在已安装 DPM 代理的情况下，可以在向导中看到该服务器。 DPM 还会显示其结构。 由于已运行 ConfigureSharePoint.exe，DPM 将与 SharePoint VSS 写入器服务及其对应的 SQL Server 数据库通信，并识别 SharePoint 场结构、关联的内容数据库和任何对应项。
   > 
   > 
4. 在“**选择数据保护方法**”页上，输入“**保护组**”的名称，并选择偏好的“*保护方法*”。 单击“下一步”。 

    ![选择数据保护方法](./media/backup-azure-backup-sharepoint/select-data-protection-method1.png)

   > [!NOTE]
   > 磁盘保护方法有助于实现短暂的恢复时间目标。 相比于磁带，Azure 是经济高效的长期保护目标。 有关详细信息，请参阅[使用 Azure 备份来取代磁带基础结构](/backup/backup-azure-backup-cloud-as-tape/)
   > 
   > 
5. 在“**指定短期目标**”页上，选择偏好的“**保留范围**”，并指定备份时间。

    ![指定短期目标](./media/backup-azure-backup-sharepoint/specify-short-term-goals2.png)

   > [!NOTE]
   > 由于恢复大多数针对少于五天的数据进行，因此我们在此示例中选择了在磁盘上保留五天，并确保在非生产时段进行备份。
   > 
   > 
6. 复查为保护组分配的存储池磁盘空间，并单击“下一步”  。

7. 对于每个保护组，DPM 将分配磁盘空间用于存储和管理副本。 此时，DPM 必须创建选定数据的副本。 选择创建副本的方法和时间，并单击“**下一步**”。

    ![选择副本创建方法](./media/backup-azure-backup-sharepoint/choose-replica-creation-method.png)

   > [!NOTE]
   > 若要确保不会影响网络流量，请选择生产时段之外的时间。
   > 
   > 
8. DPM 可对副本执行一致性检查，以确保数据完整性。 有两个可用的选项。 可以定义运行一致性检查的计划，或在副本变得不一致时，让 DPM 自动运行一致性检查。 选择用户偏好的选项，并单击“下一步”  。

    ![一致性检查](./media/backup-azure-backup-sharepoint/consistency-check.png)

9. 在“指定联机保护数据”页上，选择要保护的 SharePoint 场，然后单击“下一步”   。

    ![DPM SharePoint 保护 1](./media/backup-azure-backup-sharepoint/select-online-protection1.png)

10. 在“指定联机备份计划”页上，选择偏好的计划，并单击“下一步”   。

    ![Online_backup_schedule](./media/backup-azure-backup-sharepoint/specify-online-backup-schedule.png)

    > [!NOTE]
    > DPM 每天在不同的时间最多以 Azure 为目标执行两次备份。 Azure 备份还可以使用 [Azure 备份网络限制](//backup/backup-configure-vault#enable-network-throttling)，来控制高峰期和非高峰期用于备份的 WAN 带宽量。
    > 
    > 
11. 根据选择的备份计划，在“**指定联机保留策略**”页上，选择每日、每周、每月和每年备份点的保留策略。

    ![Online_retention_policy](./media/backup-azure-backup-sharepoint/specify-online-retention.png)

    > [!NOTE]
    > DPM 使用 grandfather-father-son 保留方案，可让你为不同的备份点选择不同的保留策略。
    > 
    > 
12. 类似于磁盘，需要在 Azure 中创建初始引用点副本。 选择在 Azure 中创建初始备份副本的偏好选项，并单击“下一步”  。

    ![Online_replica](./media/backup-azure-backup-sharepoint/online-replication.png)

13. 在“摘要”页上复查选择的设置，并单击“创建组”   。 创建保护组之后，会看到成功消息。

    ![摘要](./media/backup-azure-backup-sharepoint/summary.png)

## <a name="restore-a-sharepoint-item-from-disk-by-using-dpm"></a>使用 DPM 从磁盘还原 SharePoint 项
在以下示例中，“恢复 SharePoint 项”被意外删除，需要恢复  。
![DPM SharePoint 保护 4](./media/backup-azure-backup-sharepoint/dpm-sharepoint-protection5.png)

1. 打开“**DPM 管理员控制台**”。 DPM 保护的所有 SharePoint 场都在“**保护**”选项卡中显示。

    ![DPM SharePoint 保护 3](./media/backup-azure-backup-sharepoint/dpm-sharepoint-protection4.png)

2. 若要开始恢复该项，请选择“恢复”选项卡  。

    ![DPM SharePoint 保护 5](./media/backup-azure-backup-sharepoint/dpm-sharepoint-protection6.png)

3. 可以通过在恢复点范围内执行基于通配符的搜索，在 SharePoint 中搜索“恢复 SharePoint 项”  。

    ![DPM SharePoint 保护 6](./media/backup-azure-backup-sharepoint/dpm-sharepoint-protection7.png)

4. 从搜索结果中选择相应的恢复点，单击右键该项，并选择“**恢复**”。

5. 还可以浏览各个恢复点，并选择要恢复的数据库或项。 选择“**日期 > 恢复时间**”，并选择正确的“**数据库 > SharePoint 场 > 恢复点 > 项**”。

    ![DPM SharePoint 保护 7](./media/backup-azure-backup-sharepoint/dpm-sharepoint-protection8.png)

6. 右键单击该项，并选择“**恢复**”打开“**恢复向导**”。 单击“下一步”。 

    ![复查恢复选择](./media/backup-azure-backup-sharepoint/review-recovery-selection.png)

7. 选择用户要执行的恢复类型，并单击“下一步”  。

    ![恢复类型](./media/backup-azure-backup-sharepoint/select-recovery-type.png)

   > [!NOTE]
   > 示例中所选的“恢复到原始”会将该项恢复到原始 SharePoint 站点  。
   > 
   > 
8. 选择要使用的“恢复过程”  。

   * 如果 SharePoint 场未更改，并且与正在还原的恢复点相同，请选择“不使用恢复场进行恢复”  。
   * 如果 SharePoint 场自创建恢复点后已更改，请选择“使用恢复场进行恢复”  。

     ![恢复过程](./media/backup-azure-backup-sharepoint/recovery-process.png)

9. 提供暂时恢复数据库的暂存 SQL Server 实例位置，并在要恢复该项的 DPM 服务器和运行 SharePoint 的服务器上提供暂存文件共享。

    ![暂存位置 1](./media/backup-azure-backup-sharepoint/staging-location1.png)

    DPM 将托管 SharePoint 项的内容数据库附加到临时 SQL Server 实例。 DPM 服务器将从内容数据库恢复该项，并将它放在 DPM 服务器上的暂存文件位置。 现在，需要将 DPM 服务器上位于暂存位置的已恢复项导出到 SharePoint 场上的暂存位置。

    ![暂存位置 2](./media/backup-azure-backup-sharepoint/staging-location2.png)

10. 选择“指定恢复选项”，并将安全设置应用到 SharePoint 场，或应用恢复点的安全设置  。 单击“下一步”。 

    ![恢复选项](./media/backup-azure-backup-sharepoint/recovery-options.png)

    > [!NOTE]
    > 可以选择限制网络带宽使用率。 这可以在生产时段最大程度地降低对生产服务器的影响。
    > 
    > 
11. 复查摘要信息，并单击“恢复”开始恢复文件  。

    ![恢复摘要](./media/backup-azure-backup-sharepoint/recovery-summary.png)

12. 现在，在“DPM 管理员控制台”中选择“监视”选项卡，查看恢复的“状态”    。

    ![恢复状态](./media/backup-azure-backup-sharepoint/recovery-monitoring.png)

    > [!NOTE]
    > 文件现已还原。 可以刷新 SharePoint 站点来检查已还原的文件。
    > 
    > 

## <a name="restore-a-sharepoint-database-from-azure-by-using-dpm"></a>使用 DPM 从 Azure 还原 SharePoint 数据库

1. 若要恢复 SharePoint 内容数据库，请浏览各个恢复点（如上所示），并选择要还原的恢复点。

    ![DPM SharePoint 保护 8](./media/backup-azure-backup-sharepoint/dpm-sharepoint-protection9.png)

2. 双击 SharePoint 恢复点以显示可用的 SharePoint 目录信息。

   > [!NOTE]
   > 由于 SharePoint 场在 Azure 中受长期保留保护，因此 DPM 服务器上没有可用的目录信息（元数据）。 这样，每当需要恢复时间点 SharePoint 内容数据库时，都需要重新编录 SharePoint 场。
   > 
   > 
3. 单击“重新编目”  。

    ![DPM SharePoint 保护 10](./media/backup-azure-backup-sharepoint/dpm-sharepoint-protection12.png)

    此时显示“**云重新编录**”状态窗口。

    ![DPM SharePoint 保护 11](./media/backup-azure-backup-sharepoint/dpm-sharepoint-protection13.png)

    完成编录后，状态更改为“成功”  。 单击“**关闭**”。

    ![DPM SharePoint 保护 12](./media/backup-azure-backup-sharepoint/dpm-sharepoint-protection14.png)

4. 单击 DPM“恢复”选项卡中显示的 SharePoint 对象，获取内容数据库结构  。 右键单击相应的项，并单击“**恢复**”。

    ![DPM SharePoint 保护 13](./media/backup-azure-backup-sharepoint/dpm-sharepoint-protection15.png)
5. 此时，请按照本文前面介绍的恢复步骤，从磁盘恢复 Sharepoint 内容数据库。

## <a name="next-steps"></a>后续步骤
- 查看 [System Center 2012 - Data Protection Manager 发行说明](https://technet.microsoft.com/zh-cn/library/jj860415.aspx)
- 查看 [System Center 2012 SP1 中的 Data Protection Manager 发行说明](https://technet.microsoft.com/zh-cn/library/jj860394.aspx)

<!-- Update_Description: update metedata properties -->