---
title: include 文件
description: include 文件
services: virtual-machines
author: rockboyfor
ms.service: virtual-machines
ms.topic: include
origin.date: 05/06/2019
ms.date: 07/01/2019
ms.author: v-yeche
ms.custom: include file
ms.openlocfilehash: d5519e631c0acdc45e3e734c6241ae4dbbd9c6bb
ms.sourcegitcommit: c61b10764d533c32d56bcfcb4286ed0fb2bdbfea
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 07/19/2019
ms.locfileid: "68332794"
---
## <a name="benefits-of-managed-disks"></a>托管磁盘的优势

接下来让我们看一下使用托管磁盘可以获得的一些好处。

### <a name="highly-durable-and-available"></a>高度持久和可用

托管磁盘具备 99.999% 的可用性。 托管磁盘实现这一点的方式是：提供三个包含数据的副本，确保高持久性。 如果其中一个或两个副本出现问题，剩下的副本能够确保数据的持久性和对故障的高耐受性。 此架构有助于 Azure 为基础结构即服务 (IaaS) 磁盘持续提供企业级的持久性，年化故障率为 0%，达到行业领先水平。

### <a name="simple-and-scalable-vm-deployment"></a>简单且可缩放的 VM 部署

托管磁盘支持在每个区域中的一个订阅中创建最多 50,000 个同一类型的 VM 磁盘  ，这样就可以在单个订阅中创建数以万计的 VM  。 通过允许使用某个市场映像在一个虚拟机规模集中创建多达 1,000 VM，此功能还可以进一步增加[虚拟机规模集](../articles/virtual-machine-scale-sets/virtual-machine-scale-sets-overview.md)的可伸缩性。

### <a name="integration-with-availability-sets"></a>集成可用性集

托管磁盘集成可用性集，可确保[可用性集中的 VM](../articles/virtual-machines/windows/manage-availability.md#use-managed-disks-for-vms-in-an-availability-set) 的磁盘彼此之间完全隔离以避免单点故障。 磁盘自动放置于不同的存储缩放单元（模块）。 如果某个模块因硬件或软件故障而失败，则只有其磁盘在该模块上的 VM 实例会失败。 例如，假定某个应用程序在 5 台 VM 上运行并且这些 VM 位于一个可用性集中。 这些 VM 的磁盘不会存储在同一个模块中，因此，如果一个模块失败，该应用程序的其他实例可以继续运行。

<!--Not Available on ### Integration with Availability Zones-->

### <a name="azure-backup-support"></a>Azure 备份支持

若要防范区域灾难，可以使用 [Azure 备份](../articles/backup/backup-overview.md)创建具有基于时间的备份和备份保留策略的备份作业。 这样就可以随意执行简单的 VM 还原。 目前，Azure 备份支持高达 4 TB (TiB) 的磁盘大小。 有关详细信息，请参阅[为具有托管磁盘的 VM 使用 Azure 备份](../articles/backup/backup-overview.md)。

<!--MOONCAKE: URL using-managed-disk-vms-with-azure-backup DIRECT TO /backup/backup-overview.md-->
<!--MOONCAKE: NOT EXITS ON #using-managed-disk-vms-with-azure-backup-->

### <a name="granular-access-control"></a>粒度访问控制

可以使用 [Azure 基于角色的访问控制 (RBAC)](../articles/role-based-access-control/overview.md) 将对托管磁盘的特定权限分配给一个或多个用户。 托管磁盘公开了各种操作，包括读取、写入（创建/更新）、删除，以及检索磁盘的[共享访问签名 (SAS) URI](../articles/storage/common/storage-dotnet-shared-access-signature-part-1.md)。 可以仅将某人员执行其工作所需的操作的访问权限授予该人员。 例如，如果不希望某人员将某个托管磁盘复制到存储帐户，则可以选择不授予对该托管磁盘的导出操作的访问权限。 类似地，如果不希望某人员使用 SAS URI 复制某个托管磁盘，则可以选择不授予对该托管磁盘的该权限。

## <a name="encryption"></a>Encryption

托管磁盘提供两种不同的加密。 第一种是存储服务加密 (SSE)，由存储服务执行。 第二种是 Azure 磁盘加密，可以在 VM 的 OS 和数据磁盘上启用。

### <a name="storage-service-encryption-sse"></a>存储服务加密 (SSE)

[Azure 存储服务加密](../articles/storage/common/storage-service-encryption.md)可提供静态加密和保护数据，使组织能够信守在安全性与符合性方面所做的承诺。 默认情况下，所有托管磁盘都启用了 SSE，所有可用托管磁盘的区域都有快照和映像。 有关详细信息请访问[托管磁盘常见问题解答页](../articles/virtual-machines/windows/faq-for-disks.md#managed-disks-and-storage-service-encryption)。

### <a name="azure-disk-encryption-ade"></a>Azure 磁盘加密 (ADE)

Azure 磁盘加密允许加密 IaaS 虚拟机使用的 OS 磁盘和数据磁盘。 此加密包括托管磁盘。 对于 Windows，驱动器是使用行业标准 BitLocker 加密技术加密的。 对于 Linux，磁盘是使用 DM-Crypt 技术加密的。 加密过程与 Azure Key Vault 集成，可让你控制和管理磁盘加密密钥。

<!--Not Available on  [Azure Disk Encryption for IaaS VMs](../articles/security/azure-security-disk-encryption-overview.md)-->

## <a name="disk-roles"></a>磁盘角色

在 Azure 中有三个主要磁盘角色：数据磁盘、OS 磁盘和临时磁盘。 这些角色将映射到附加到虚拟机的磁盘。

![操作中的磁盘角色](media/virtual-machines-managed-disks-overview/disk-types.png)

### <a name="data-disk"></a>数据磁盘

数据磁盘是附加到虚拟机的托管磁盘，用于存储应用程序数据或其他需要保留的数据。 数据磁盘注册为 SCSI 驱动器并且带有所选择的字母标记。 每个数据磁盘的最大容量为 32,767 gibibytes (GiB)。 虚拟机的大小决定了可附加的磁盘数目，以及可用来托管磁盘的存储类型。

### <a name="os-disk"></a>操作系统磁盘

每个虚拟机都附加了一个操作系统磁盘。 该 OS 磁盘有一个预先安装的 OS，是在创建 VM 时选择的。

此磁盘最大容量为 2,048 GiB。

### <a name="temporary-disk"></a>临时磁盘

每个 VM 包含一个不是托管磁盘的临时磁盘。 临时磁盘为应用程序和进程提供短期存储存储空间，仅用于存储页面或交换文件等数据。 在[维护事件](../articles/virtual-machines/windows/manage-availability.md?toc=%2fvirtual-machines%2fwindows%2ftoc.json#understand-vm-reboots---maintenance-vs-downtime)期间或[重新部署 VM](../articles/virtual-machines/troubleshooting/redeploy-to-new-node-windows.md?toc=%2Fazure%2Fvirtual-machines%2Fwindows%2Ftoc.json) 时，临时磁盘上的数据可能会丢失。 在 Azure Linux VM 上，临时磁盘默认为 /dev/sdb，而在 Windows VM 上，临时磁盘默认为 D:。 在 VM 成功标准重启期间，临时磁盘上的数据将保留。

<!--MOONCAKE: CORRECT ON D: disk for Temporary disk-->

## <a name="managed-disk-snapshots"></a>托管磁盘快照

托管磁盘快照是托管磁盘的只读完整副本，默认情况下它作为标准托管磁盘进行存储。 使用快照可以在任意时间点备份托管磁盘。 这些快照独立于源磁盘而存在，并可用来创建新的托管磁盘。 基于已使用大小对这些快照进行计费。 例如，如果创建预配容量为 64 GiB 且实际使用数据大小为 10 GiB 的托管磁盘的快照，则仅针对已用数据大小 10 GiB 对该快照计费。  

若要了解有关如何使用托管磁盘创建快照的详细信息，请查看下列资源：

* [在 Windows 中使用快照创建存储为托管磁盘的 VHD 的副本](../articles/virtual-machines/windows/snapshot-copy-managed-disk.md)
* [在 Linux 中使用快照创建存储为托管磁盘的 VHD 的副本](../articles/virtual-machines/linux/snapshot-copy-managed-disk.md)

### <a name="images"></a>映像

托管磁盘还支持创建托管自定义映像。 可以从存储帐户中的自定义 VHD 创建映像或者直接从通用化 (sysprepped) VM 创建映像。 此过程捕获单个映像。 该映像包含与 VM 关联的所有托管磁盘，包括 OS 磁盘和数据磁盘。 该托管自定义映像支持使用自定义映像创建数百台 VM，且不需要复制或管理任何存储帐户。

有关创建映像的信息，请查看以下文章：

* [如何在 Azure 中捕获通用 VM 的托管映像](../articles/virtual-machines/windows/capture-image-resource.md)
* [如何使用 Azure CLI 生成和捕获 Linux 虚拟机](../articles/virtual-machines/linux/capture-image.md)

#### <a name="images-versus-snapshots"></a>映像与快照

了解映像与快照之间的区别很重要。 使用托管磁盘，可以创建已解除分配的通用 VM 的映像。 此映像包括附加到该 VM 的所有磁盘。 可以使用此映像创建 VM，它包括所有磁盘。

快照是磁盘在创建快照那一刻的副本。 它仅应用于一个磁盘。 如果 VM 有一个磁盘（OS 磁盘），则可以为其创建快照或映像，并且可以通过该快照或映像创建 VM。

除了所包含的磁盘，快照无法感知任何其他磁盘。 因此，如果在要求对多个磁盘进行协调的方案（例如条带化方案）中使用，则会出现问题。 快照彼此之间将需要相互协调，而目前并不支持此功能。

<!--Pending on verify-->

## <a name="disk-allocation-and-performance"></a>磁盘分配和性能

下图描绘了如何使用三级预配系统为磁盘实时分配带宽和 IOPS：

![显示带宽和 IOPS 分配的三级预配系统](media/virtual-machines-managed-disks-overview/real-time-disk-allocation.png)

第一级预配设置单磁盘 IOPS 和带宽分配。  在第二级，计算服务器主机实施 SSD 预配，将其仅应用到存储在服务器的 SSD 上的数据，该 SSD 包括使用缓存的磁盘（ReadWrite 和 ReadOnly）以及本地和临时磁盘。 最后，VM 网络预配发生在第三级，它针对可供计算主机发送到 Azure 存储后端的任何 I/O。 使用此方案时，VM 的性能依赖于多种因素，从 VM 使用本地 SSD 的方式，到附加的磁盘数，再到所附加的磁盘的性能和缓存类型，不一而足。

对于这些限制，举例来说，Standard_DS1v1 VM 无法达到 P30 磁盘的 5,000 IOPS 的极限，不管它是否为缓存型，因为在 SSD 和网络级别存在限制：

![Standard_DS1v1 示例分配](media/virtual-machines-managed-disks-overview/example-vm-allocation.png)

Azure 将设置了优先级的网络通道用于磁盘流量，该流量获得的优先级高于其他低优先级网络流量。 这样有助于磁盘在发生网络争用的情况下保持其预期的性能。 类似地，Azure 存储在后台使用自动负载均衡来处理资源争用等问题。 Azure 存储会在你创建磁盘时分配必需的资源，并对资源应用主动和被动均衡以处理流量级别问题。 这进一步确保磁盘可以保持其预期的 IOPS 和吞吐量目标。 可以根据需要使用 VM 级别和磁盘级别的指标来跟踪性能并设置警报。

请参阅[高性能设计](../articles/virtual-machines/windows/premium-storage-performance.md)一文，了解优化 VM + 磁盘配置的最佳做法，以便实现所需的性能。

<!--Pending on verify-->

## <a name="next-steps"></a>后续步骤

在有关磁盘类型的文章中，详细了解 Azure 提供的各个磁盘类型以及哪个类型符合自己的需求，并了解其性能目标。

<!--Update_Description: wording update, update link-->