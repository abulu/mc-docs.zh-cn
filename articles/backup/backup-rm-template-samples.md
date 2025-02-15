---
title: 用于 Azure 备份的 Azure 资源管理器模板
description: Azure 备份 PowerShell 示例
services: backup
author: lingliw
manager: digimobile
ms.service: backup
ms.topic: sample
ms.date: 01/31/2019
ms.author: v-lingwu
ms.custom: mvc
ms.openlocfilehash: 4a10626e9a2fa48d9d845e0cb4bba0f43a2ed5a2
ms.sourcegitcommit: 5191c30e72cbbfc65a27af7b6251f7e076ba9c88
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 07/05/2019
ms.locfileid: "67570472"
---
# <a name="azure-resource-manager-templates-for-azure-backup"></a>用于 Azure 备份的 Azure 资源管理器模板

下表包含可以与恢复服务保管库和 Azure 备份功能配合使用的 Azure 资源管理器模板的链接。

|   |   |
|---|---|
|**恢复服务保管库** | |
| [创建恢复服务保管库](https://github.com/Azure/azure-quickstart-templates/tree/master/101-recovery-services-vault-create)| 创建恢复服务保管库。 此保管库可以用于 Azure 备份和 Azure Site Recovery。 |
|**备份虚拟机**| |
| [备份资源管理器 VM](https://github.com/Azure/azure-quickstart-templates/tree/master/101-recovery-services-backup-vms) | 使用现有的恢复服务保管库和备份策略，备份同一资源组中的资源管理器虚拟机。|
| [将 IaaS VM 备份到恢复服务保管库](https://github.com/Azure/azure-quickstart-templates/tree/master/201-recovery-services-backup-classic-resource-manager-vms) | 用于备份经典虚拟机和资源管理器虚拟机的模板。 |
| [为 IaaS VM 创建每周备份策略](https://github.com/Azure/azure-quickstart-templates/tree/master/101-recovery-services-weekly-backup-policy-create) | 模板创建恢复服务保管库和每周备份策略，该策略用于备份经典虚拟机和资源管理器虚拟机。|
| [为 IaaS VM 创建每日备份策略](https://github.com/Azure/azure-quickstart-templates/tree/master/101-recovery-services-daily-backup-policy-create) | 模板创建恢复服务保管库和每日备份策略，该策略用于备份经典虚拟机和资源管理器虚拟机。|
| [部署启用了备份的 Windows Server VM](https://github.com/Azure/azure-quickstart-templates/tree/master/101-recovery-services-create-vm-and-configure-backup) | 模板创建启用默认备份策略的 Windows Server VM 和恢复服务保管库。|
|**监视备份作业** |  |
| [将 Azure Monitor 日志与 Azure Backup 配合使用](https://github.com/Azure/azure-quickstart-templates/tree/master/101-backup-oms-monitoring) | 模板部署用于 Azure 备份的 Azure Monitor 日志，后者用于监视备份和还原作业、备份警报以及在恢复服务保管库中使用的云存储。|  
|**在 Azure VM 中备份 SQL Server** |  |
| [在 Azure VM 中备份 SQL Server](https://github.com/Azure/azure-quickstart-templates/tree/master/101-recovery-services-vm-workload-backup) | 模板创建特定于恢复服务保管库和工作负荷的备份策略。 它使用 Azure 备份服务注册 VM 并在该 VM 上配置保护。 目前，它仅适用于 SQL 库映像。 |
|   |   |


<!-- Update_Description: update metedata properties -->