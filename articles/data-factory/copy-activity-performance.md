---
title: Azure 数据工厂的复制活动性能和优化指南 | Microsoft Docs
description: 了解使用复制活动时影响 Azure 数据工厂中数据移动性能的关键因素。
services: data-factory
documentationcenter: ''
author: WenJason
manager: digimobile
ms.reviewer: douglasl
ms.service: data-factory
ms.workload: data-services
ms.tgt_pltfrm: na
ms.topic: conceptual
origin.date: 06/10/2019
ms.date: 07/08/2019
ms.author: v-jay
ms.openlocfilehash: 4a2dd06e05a68f4d7513ec4a5dd5254143d4a6e4
ms.sourcegitcommit: 5191c30e72cbbfc65a27af7b6251f7e076ba9c88
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 07/05/2019
ms.locfileid: "67569859"
---
# <a name="copy-activity-performance-and-tuning-guide"></a>复制活动性能和优化指南

Azure 数据工厂复制活动提供安全、可靠且高性能的一流数据加载解决方案。 它允许用户在各种云和本地数据存储中每天复制数十 TB 的数据。 速度超快的数据加载性能是确保用户能专注于核心“大数据”问题的关键：构建高级分析解决方案并从所有数据获得深入见解。

Azure 提供了一组企业级数据存储和数据仓库解决方案，并且复制活动提供了高度优化的数据加载体验，易于配置和设置。 使用单个复制活动，可完成：

* 以 **1.2 GBps** 的速度将数据加载到 **Azure SQL 数据仓库**。
* 以 **1.0 GBps** 的速度将数据加载到 **Azure Blob 存储**

本文介绍：

* 支持的源和接收器数据存储的[性能参考数字](#performance-reference)，可帮助用户规划项目；
* 可在不同情况下提高复制吞吐量的功能，包括[数据集成单元](#data-integration-units)、[并行复制](#parallel-copy)和[分步复制](#staged-copy)；
* 有关如何优化性能和调整影响复制性能的关键因素的[性能优化指南](#performance-tuning-steps)。

> [!NOTE]
> 如果对常规复制活动不熟悉，在阅读本文前请参阅[复制活动概述](copy-activity-overview.md)。
>

## <a name="performance-reference"></a>性能参考

作为参考，下表基于内部测试显示了**单次复制活动运行**中给定源和接收器对的复制吞吐量数目（**以 MBps 为单位**）。 为进行比较，它还演示了[数据集成单元](#data-integration-units)或[自承载 Integration Runtime 可伸缩性](concepts-integration-runtime.md#self-hosted-integration-runtime)的不同设置（多个节点）如何提升复制性能。

![性能矩阵](./media/copy-activity-performance/CopyPerfRef.png)

> [!IMPORTANT]
> 对 Azure Integration Runtime 执行复制活动时，允许的数据集成单元（以前称为数据移动单元）数量下限为两个。 如果未指定，请参阅[数据集成单元](#data-integration-units)中使用的默认数据集成单元数。

需要注意的要点：

* 使用以下公式计算吞吐量：[从源读取的数据大小]/[复制活动运行持续时间]。
* 表中的性能参考数字使用单次复制活动运行中的 [TPC-H](http://www.tpc.org/tpch/) 数据集测量得出。 基于文件的存储的测试文件是多个大小为 10 GB 的文件。
* 在 Azure 数据存储中，源和接收器位于同一 Azure 区域。
* 对于本地数据存储和云数据存储之间的混合复制，已在与数据存储分开并具有以下规格的计算机上运行每个自承载 Integration Runtime 节点。 单个活动运行时，复制操作仅使用测试计算机的一小部分 CPU、内存或网络带宽。
    <table>
    <tr>
        <td>CPU</td>
        <td>32 核 2.20 GHz Intel Xeon E5-2660 v2</td>
    </tr>
    <tr>
        <td>内存</td>
        <td>128 GB</td>
    </tr>
    <tr>
        <td>网络</td>
        <td>Internet 接口：10 Gbps；Intranet 接口：40 Gbps</td>
    </tr>
    </table>


> [!TIP]
> 通过使用更多数据集成单元 (DIU)，可以实现更高吞吐量。 请参阅[数据集成单元](#data-integration-units)部分，了解有关此功能和受支持方案的相关详细信息。 

## <a name="data-integration-units"></a>数据集成单元

**数据集成单元 (DIU)** （以前称为云数据移动单元或 DMU）是一种度量单位，代表单个单元在数据工厂中的能力（包含 CPU、内存、网络资源分配）。 **DIU 仅适用于 [Azure Integration Runtime](concepts-integration-runtime.md#azure-integration-runtime)** ，而不适用于[自承载 Integration Runtime](concepts-integration-runtime.md#self-hosted-integration-runtime)。

**为复制活动运行提供支持的数据集成单元数下限为两个。** 如果未指定，下表列出了不同复制方案中使用的默认 DIU 数目：

| 复制方案 | 服务决定的默认 DIU 数目 |
|:--- |:--- |
| 在基于文件的存储之间复制数据 | 4 到 32 个，具体取决于文件的数量和大小。 |
| 所有其他复制方案 | 4 |

若要替代此默认值，请如下所示指定 **dataIntegrationUnits** 属性的值。 **dataIntegrationUnits** 属性 **允许的值** **最大为 256** 。 复制操作在运行时使用的**实际 DIU 数**等于或小于配置的值，具体取决于数据模式。 有关为特定复制源和接收器配置更多单元时可能获得的性能增益级别的信息，请参阅[性能参考](#performance-reference)。

监视活动运行时，可以在复制活动输出中看到每次复制运行实际使用的数据集成单元数。 从[复制活动监视](copy-activity-overview.md#monitoring)中了解详细信息。

> [!NOTE]
> 当前仅当**将多个文件从 Azure 存储/Amazon S3/Google Cloud Storage/云 FTP/云 SFTP 复制到任何其他云数据存储**时，**大于 4** 的 DIU 设置才适用。
>

**示例：**

```json
"activities":[
    {
        "name": "Sample copy activity",
        "type": "Copy",
        "inputs": [...],
        "outputs": [...],
        "typeProperties": {
            "source": {
                "type": "BlobSource",
            },
            "sink": {
                "type": "BlobSink"
            },
            "dataIntegrationUnits": 32
        }
    }
]
```

### <a name="data-integration-units-billing-impact"></a>数据集成单元计费影响

请**务必**记住，会根据复制操作的总时间向你收费。 对数据移动计费的总持续时间是所有 DIU 的持续时间总和。 如果复制作业过去使用 2 个云单元花费 1 小时，现在使用 8 个云单元花费 15 分钟，则总费用几乎相同。

## <a name="parallel-copy"></a>并行复制

可使用 **parallelCopies** 属性指示要让“复制活动”使用的并行度。 可将此属性视为复制活动内，可从源并行读取或并行写入接收器数据存储的最大线程数。

对于每个复制活动运行，数据工厂确定用于将数据从源数据存储复制到目标数据存储的并行复制数。 它使用的默认并行复制数取决于使用的源和接收器类型：

| 复制方案 | 由服务确定的默认并行复制计数 |
| --- | --- |
| 在基于文件的存储之间复制数据 |取决于文件大小以及用于在两个云数据存储之间复制数据的数据集成单元 (DIU) 数，或自承载 Integration Runtime 计算机的物理配置。 |
| 将数据从任何源数据存储复制到 Azure 表存储 |4 |
| 所有其他复制方案 |1 |

> [!TIP]
> 在基于文件的存储之间复制数据时，默认行为（自动确定）通常会提供最佳吞吐量。 

要控制托管数据存储的计算机上的负载或优化复制性能，可选择替代默认值并为 parallelCopies 属性指定值  。 该值必须是大于或等于 1 的整数。 在运行时，为了获得最佳性能，复制活动使用小于或等于所设置的值。

```json
"activities":[
    {
        "name": "Sample copy activity",
        "type": "Copy",
        "inputs": [...],
        "outputs": [...],
        "typeProperties": {
            "source": {
                "type": "BlobSource",
            },
            "sink": {
                "type": "BlobSink"
            },
            "parallelCopies": 32
        }
    }
]
```

需要注意的要点：

* 在基于文件的存储之间复制数据时，**parallelCopies** 确定文件级别的并行度。 单个文件内的区块化会自动透明地在文件下进行，它旨在对给定源数据存储类型使用最佳区块大小，以并行和正交方式将数据加载到 parallelCopies。 数据移动服务在运行时用于复制操作的并行复制的实际数量不超过所拥有的文件数。 如果复制行为是 **mergeFile**，复制活动无法利用文件级别的并行度。
* 指定 **parallelCopies** 属性的值时，请考虑源数据存储和接收器数据存储上的负载会增加，如果使用自承载 Integration Runtime 为混合复制或其他复制的复制活动提供支持，其负载也会增加。 尤其在有多个活动或针对同一数据存储运行的相同活动有并发运行时，会发生这种情况。 如果注意到数据存储或自承载 Integration Runtime 负载过重，请降低 **parallelCopies**  值以减轻负载。
* 将数据从不基于文件的存储复制到基于文件的存储时，数据移动服务将忽略 **parallelCopies** 属性。 即使指定了并行性，在此情况下也不适用。
* **parallelCopies** 独立于 **dataIntegrationUnits**。 前者跨所有数据集成单元进行计数。

## <a name="staged-copy"></a>暂存复制

将数据从源数据存储复制到接收器数据存储时，可能会选择使用 Blob 存储作为过渡暂存存储。 暂存在以下情况下特别有用：

- **通过 PolyBase 从各种数据存储将数据引入 SQL 数据仓库**。 SQL 数据仓库使用 PolyBase 作为高吞吐量机制，将大量数据加载到 SQL 数据仓库中。 但是，源数据必须位于 Blob 存储中，并且它必须满足其他条件。 从 Blob 存储以外的数据存储加载数据时，可通过过渡暂存 Blob 存储激活数据复制。 在这种情况下，数据工厂会执行所需的数据转换，确保其满足 PolyBase 的要求。 然后，它使用 PolyBase 将数据高效加载到 SQL 数据仓库。 有关详细信息，请参阅[使用 PolyBase 将数据加载到 Azure SQL 数据仓库](connector-azure-sql-data-warehouse.md#use-polybase-to-load-data-into-azure-sql-data-warehouse)。
- **有时，通过速度慢的网络连接执行混合数据移动（即从本地数据存储复制到云数据存储）需要一段时间**。 为了提高性能，可使用暂存复制在本地压缩数据，缩短将数据移动到云中的暂存数据存储的时间，然后，可先在暂存存储中解压缩数据，再将它们加载到目标数据存储。
- **由于企业 IT 策略，不希望在防火墙中打开除端口 80 和端口 443 以外的端口**。 例如，将数据从本地数据存储复制到 Azure SQL 数据库接收器或 Azure SQL 数据仓库接收器时，需要对 Windows 防火墙和公司防火墙激活端口 1433 上的出站 TCP 通信。 在这种情况下，暂存复制可以利用自承载 Integration Runtime 首先在端口 443 上通过 HTTP 或 HTTPS 将数据复制到 Blob 存储暂存实例，然后将数据从 Blob 存储暂存加载到 SQL 数据库或 SQL 数据仓库。 在此流中，不需要启用端口 1433。

### <a name="how-staged-copy-works"></a>暂存复制的工作原理

激活暂存功能时，首先将数据从源数据存储复制到暂存 Blob 存储（自带）。 然后，将数据从暂存数据存储复制到接收器数据存储。 数据工厂自动管理两阶段流。 数据移动完成后，数据工厂还将清除暂存存储中的临时数据。

![暂存复制](media/copy-activity-performance/staged-copy.png)

使用暂存存储激活数据移动时，可指定是否要先压缩数据，再将数据从源数据存储移动到过渡数据存储或暂存数据存储，然后先解压缩数据，再将数据从过渡数据存储或暂存数据移动到接收器数据存储。

目前，不能使用暂存存储在两个本地数据存储之间复制数据。

### <a name="configuration"></a>配置

在复制活动中配置 **enableStaging** 设置，指定在将数据加载到目标数据存储之前是否要在 Blob 存储中暂存。 将 **enableStaging** 设置为 `TRUE` 时，指定下一个表中列出的其他属性。 如果未指定，则还需要创建 Azure 存储或存储共享访问签名链接服务供暂存用。

| 属性 | 说明 | 默认值 | 必选 |
| --- | --- | --- | --- |
| **enableStaging** |指定是否要通过过渡暂存存储复制数据。 |False |否 |
| **linkedServiceName** |指定 [AzureStorage](connector-azure-blob-storage.md#linked-service-properties) 链接服务的名称，这指用作过渡暂存存储的存储实例。 <br/><br/> 不能使用具有共享访问签名的存储通过 PolyBase 将数据加载到 SQL 数据仓库。 可在其他任何情况下使用它。 |不适用 |将 **enableStaging** 设置为 TRUE 时，则为是 |
| **路径** |指定要包含此暂存数据的 Blob 存储路径。 如果不提供路径，该服务将创建容器以存储临时数据。 <br/><br/> 只在使用具有共享访问签名的存储时，或者要求临时数据位于特定位置时才指定路径。 |不适用 |否 |
| **enableCompression** |指定是否应先压缩数据，再将数据复制到目标。 此设置可减少传输的数据量。 |False |否 |

>[!NOTE]
> 若使用暂存复制并启用压缩，则不支持对暂存 blob 链接服务的服务主体或 MSI 身份验证。

以下是具有上表所述属性的复制活动的示例定义：

```json
"activities":[
    {
        "name": "Sample copy activity",
        "type": "Copy",
        "inputs": [...],
        "outputs": [...],
        "typeProperties": {
            "source": {
                "type": "SqlSource",
            },
            "sink": {
                "type": "SqlSink"
            },
            "enableStaging": true,
            "stagingSettings": {
                "linkedServiceName": {
                    "referenceName": "MyStagingBlob",
                    "type": "LinkedServiceReference"
                },
                "path": "stagingcontainer/path",
                "enableCompression": true
            }
        }
    }
]
```

### <a name="staged-copy-billing-impact"></a>暂存复制计费影响

基于两个步骤进行计费：复制持续时间和复制类型。

* 在云复制期间（将数据从一个云数据存储复制到另一个云数据存储，两个阶段均由 Azure Integration Runtime 提供支持）使用暂存时，需要支付 [步骤 1 和步骤 2 的复制持续时间总和] x [云复制单元价格]。
* 在混合复制期间（将数据从本地数据存储复制到云数据存储，一个阶段由自承载 Integration Runtime 提供支持）使用暂存时，需要支付 [混合复制持续时间] x [混合复制单元价格] + [云复制持续时间] x [云复制单元价格]。

## <a name="performance-tuning-steps"></a>性能优化步骤

我们建议采用以下步骤，通过复制活动调整数据工厂服务的性能：

1. **建立基准**。 在开发阶段，通过对代表性数据示例使用复制活动来测试管道。 按照[复制活动监视](copy-activity-overview.md#monitoring)收集执行详细信息和性能特征。

2. **诊断和优化性能**。 如果观察到的性能不符合预期，则需要识别性能瓶颈。 然后，优化性能以消除或减少瓶颈的影响。 

    在某些情况下，当你在 ADF 中执行复制活动时，将直接在[复制活动监视](copy-activity-overview.md#monitor-visually)页上看到“性能调优技巧”  ，如以下示例所示。 它不仅告诉你针对给定复制运行所识别出的瓶颈，而且还指导你进行一些更改来提升复制吞吐量。 目前的性能优化提示提供如下建议：在将数据复制到 Azure SQL 数据仓库时使用 PolyBase；在数据存储端资源出现瓶颈时增加 Azure Cosmos DB RU 或 Azure SQL DB DTU；删除不必要的暂存副本等等。性能优化规则也将逐渐丰富。

    **示例：复制到 Azure SQL DB 时的性能优化提示**

    在此示例中，在复制运行期间，ADF 注意到接收器 Azure SQL DB 达到了很高的 DTU 利用率，这会减慢写入操作，因此，建议使用更多的 DTU 来增加 Azure SQL DB 层。 

    ![使用性能调优技巧进行复制监视](./media/copy-activity-overview/copy-monitoring-with-performance-tuning-tips.png)

    此外，以下是一些常见的注意事项。 本文不涵盖性能诊断的完整说明。

   * 性能功能：
     * [并行复制](#parallel-copy)
     * [数据集成单元](#data-integration-units)
     * [暂存复制](#staged-copy)
     * [自承载 Integration Runtime 可伸缩性](concepts-integration-runtime.md#self-hosted-integration-runtime)
   * [自承载 Integration Runtime](#considerations-for-self-hosted-integration-runtime)
   * [Source](#considerations-for-the-source)
   * [接收器](#considerations-for-the-sink)
   * [序列化和反序列化](#considerations-for-serialization-and-deserialization)
   * [压缩](#considerations-for-compression)
   * [列映射](#considerations-for-column-mapping)
   * [其他注意事项](#other-considerations)

3. **将配置扩展至整个数据集**。 对执行结果和性能满意时，可以扩展定义和管道以覆盖整个数据集。

## <a name="considerations-for-self-hosted-integration-runtime"></a>自承载 Integration Runtime 注意事项

如果在自承载 Integration Runtime 上执行复制活动，请注意以下事项：

**设置**：建议使用专用计算机托管 Integration Runtime。 请参阅[使用自承载 Integration Runtime 的注意事项](concepts-integration-runtime.md)。

**横向扩展**：具有一个或多个节点的单个逻辑自承载 Integration Runtime 可同时用于在同一时间运行的多个复制活动。 如果非常需要包含大量并发复制活动运行或需要复制大量数据的混合数据移动，请考虑[横向扩展自承载 Integration Runtime](create-self-hosted-integration-runtime.md#high-availability-and-scalability)，以便预配更多资源为复制提供支持。

## <a name="considerations-for-the-source"></a>有关源的注意事项

### <a name="general"></a>常规

确保基础数据存储未被在其上运行或针对其运行的其他工作负荷过渡占用。

有关 Azure 数据存储的信息，请参阅特定于数据存储的[监视和优化主题](#performance-reference)，帮助你了解数据存储性能特征、最大限度缩短响应时间以及最大限度提高吞吐量。

* 如果将数据**从 Blob 存储复制到 SQL 数据仓库**，请考虑使用 **PolyBase** 来提高性能。 有关详细信息，请参阅[使用 PolyBase 将数据加载到 Azure SQL 数据仓库](connector-azure-sql-data-warehouse.md#use-polybase-to-load-data-into-azure-sql-data-warehouse)。
* 如果将数据**从 HDFS 复制到 Azure Blob**，请考虑使用 **DistCp** 来提高性能。 请参阅[使用 DistCp 从 HDFS 复制数据](connector-hdfs.md#use-distcp-to-copy-data-from-hdfs)了解详细信息。
* 如果将数据**从 Redshift 复制到 Azure SQL 数据仓库/Azure BLob**，请考虑使用 **UNLOAD** 来提高性能。 请参阅[使用 UNLOAD 从 Amazon Redshift 复制数据](connector-amazon-redshift.md#use-unload-to-copy-data-from-amazon-redshift)了解详细信息。

### <a name="file-based-data-stores"></a>基于文件的数据存储

* **平均文件大小和文件计数**：复制活动一次传输一个文件的数据。 在移动相同数据量的情况下，如果数据由许多小文件而不是几个大文件组成，则由于每个文件都有启动阶段，总吞吐量会较低。 因此，尽可能将小文件合并为较大的文件，以获得更高的吞吐量。
* **文件格式和压缩**：有关提高性能的更多方法，请参阅[序列化和反序列化注意事项](#considerations-for-serialization-and-deserialization)和[压缩注意事项](#considerations-for-compression)部分。

### <a name="relational-data-stores"></a>关系数据存储

* **数据模式**：表架构会影响复制吞吐量。 复制相同数据量时，较大行大小可比较小行大小提供更佳的性能。 原因是数据库可以更有效地检索包含较少行的较少批量数据。
* **查询或存储过程**：优化在复制活动源中指定的查询或存储过程的逻辑，以更有效地提取数据。

## <a name="considerations-for-the-sink"></a>有关接收器的注意事项

### <a name="general"></a>常规

确保基础数据存储未被在其上运行或针对其运行的其他工作负荷过渡占用。

有关 Azure 数据存储的信息，请参阅特定于数据存储的[监视和优化主题](#performance-reference)。 这些主题可帮助用户了解数据存储性能特征、了解如何尽量缩短响应时间以及最大化吞吐量。

* 如果将数据**从任何数据存储复制到 Azure SQL 数据仓库**，请考虑使用 **PolyBase** 来提高性能。 有关详细信息，请参阅[使用 PolyBase 将数据加载到 Azure SQL 数据仓库](connector-azure-sql-data-warehouse.md#use-polybase-to-load-data-into-azure-sql-data-warehouse)。
* 如果将数据**从 HDFS 复制到 Azure Blob**，请考虑使用 **DistCp** 来提高性能。 请参阅[使用 DistCp 从 HDFS 复制数据](connector-hdfs.md#use-distcp-to-copy-data-from-hdfs)了解详细信息。
* 如果将数据**从 Redshift 复制到 Azure SQL 数据仓库/Azure BLob**，请考虑使用 **UNLOAD** 来提高性能。 请参阅[使用 UNLOAD 从 Amazon Redshift 复制数据](connector-amazon-redshift.md#use-unload-to-copy-data-from-amazon-redshift)了解详细信息。

### <a name="file-based-data-stores"></a>基于文件的数据存储

* **复制行为**：如果从基于文件的不同数据存储复制数据，则复制活动可通过 copyBehavior  属性提供三个选项。 它将保留层次结构、平展层次结构或合并文件。 保留或平展层次结构有少量的性能开销或没有性能开销，但合并文件会导致性能开销增加。
* **文件格式和压缩**：有关提高性能的更多方法，请参阅[序列化和反序列化注意事项](#considerations-for-serialization-and-deserialization)和[压缩注意事项](#considerations-for-compression)部分。

### <a name="relational-data-stores"></a>关系数据存储

* **复制行为和性能含义**：可以通过不同方式将数据写入 SQL 接收器，请参阅[将数据加载到 Azure SQL 数据库的最佳做法](connector-azure-sql-database.md#best-practice-for-loading-data-into-azure-sql-database)了解更多信息。

* **数据模式和批大小**：
  * 表架构会影响复制吞吐量。 复制相同数据量时，较大行大小会比较小行大小提供更好的性能，因为数据库可以更有效地提交较少的数据批次。
  * 复制活动以一系列批次插入数据。 可使用  **writeBatchSize** 属性设置批中的行数。 如果数据的行较小，可设置具有更高值的 **writeBatchSize** 属性，从更低的批开销和更高的吞吐量获益。 如果数据的行大小较大，请谨慎增加  **writeBatchSize**。 较高的值可能会导致复制失败（因为数据库负载过重）。

### <a name="nosql-stores"></a>NoSQL 存储

* 对于**表存储**：
  * **分区**：将数据写入交错分区会显着降低性能。 按分区键对源数据进行排序，以有效率地将数据插入各个分区，或调整逻辑将数据写入单个分区。

## <a name="considerations-for-serialization-and-deserialization"></a>序列化和反序列化注意事项

输入数据集或输出数据集是文件时，可能会发生序列化和反序列化。 有关复制活动支持的文件格式的详细信息，请参阅[支持的文件和压缩格式](supported-file-formats-and-compression-codecs.md)。

**复制行为**：

* 在基于文件的数据存储之间复制文件：
  * 输入和输出数据集具有相同的文件格式设置或没有文件格式设置时，数据移动服务执行**二进制复制**，而不进行任何序列化或反序列化。 与源和接收器文件格式设置彼此不同的情况相比，这提供的吞吐量更高。
  * 输入和输出数据集都为文本格式且只有编码类型不同时，数据移动服务仅进行编码转换。 它不进行任何序列化和反序列化，与二进制复制相比，这会产生一些性能开销。
  * 当输入和输出数据集具有不同的文件格式或不同的配置时（如分隔符），数据移动服务会反序列化源数据，以进行流式传输、转换，然后将其序列化为指示的输出格式。 与其他方案相比，此操作会产生更多的性能开销。
* 向/从不基于文件的数据存储复制文件时（例如，从基于文件的存储到关系存储），需要序列化或反序列化步骤。 此步骤将造成大量的性能开销。

**文件格式**：选择的文件格式可能会影响复制性能。 例如，Avro 是一种将元数据与数据一起存储的压缩二进制格式。 它在 Hadoop 生态系统中对处理和查询具有广泛的支持。 然而，Avro 进行序列化和反序列化的代价更高，这会导致比文本格式更低的复制吞吐量。 在选择整个处理流中使用的文件格式时，应考虑全面。 首先考虑数据存储的格式（源数据存储或从外部系统提取）；存储、分析处理和查询的最佳格式；应将数据导出到数据集市、供报表和可视化工具使用的格式。 有时，读取和写入性能不是最佳的文件格式可能对于整体分析处理来说却是不错的选择。

## <a name="considerations-for-compression"></a>压缩注意事项

输入或输出数据集是文件时，可以设置复制活动，使其在将数据写入目标时执行压缩或解压缩。 选择压缩时，请在输入/输出 (I/O) 和 CPU 之间进行权衡。 压缩数据会花费额外的计算资源。 但反过来减少了网络 I/O 和存储。 根据所用数据，可能会看到整体复制吞吐量的提升。

**编解码器**：每中压缩编解码器各有优点。 例如，虽然 bzip2 复制吞吐量最低，但使用 bzip2 可获得最佳的 Hive 查询性能，因为可将其拆分处理。 Gzip 是最平衡的选项，也是最常用的选项。 选择最适合端到端方案的编解码器。

**级别**：对于每个压缩编解码器，有以下两个选择：最快压缩和最佳压缩。 最快压缩选项可尽快压缩数据，不过无法以最佳方式压缩生成的文件。 最佳压缩选项花费更多时间进行压缩，产生最小量的数据。 可对这两个选项进行测试，确定可为方案提供最佳整体性能的选项。

**注意事项**：若要在本地存储和云之间复制大量数据，请考虑使用启用了压缩的[暂存复制](#staged-copy)。 当公司网络和 Azure 服务的带宽是限制因素，并希望输入数据集和输出数据集都处于未压缩形式时，使用过渡存储将非常有用。

## <a name="considerations-for-column-mapping"></a>列映射注意事项

可在复制活动中设置 **columnMappings** 属性，将全部或部分输入列映射到输出列。 数据移动服务从源读取数据后，它需要先对数据执行列映射，再将数据写入接收器。 这一额外处理会降低复制吞吐量。

如果源数据存储可查询，例如，如果存储是关系存储（如 SQL 数据库或 SQL Server），或者是 NoSQL 存储（如表存储或 Azure Cosmos DB），请考虑将列筛选和重排序逻辑推送到**查询**属性，而不使用列映射。 这样，当数据移动服务从源数据存储读取数据时会发生投影，使效率更高。

从[复制活动架构映射](copy-activity-schema-and-type-mapping.md)中了解详细信息。

## <a name="other-considerations"></a>其他注意事项

如果要复制的数据大小较大，可以调整业务逻辑来进一步对数据分区，并提高复制活动的运行频率，以减少每次复制活动运行的数据大小。

请谨慎对待需要数据工厂同时连接到同一数据存储的数据集数和复制活动数。 许多并发复制作业可能会限制数据存储，并导致性能下降，复制作业内部重试，甚至在某些情况下导致执行失败。

## <a name="sample-scenario-copy-from-an-on-premises-sql-server-to-blob-storage"></a>示例方案：从本地 SQL Server 复制到 Blob 存储

**场景**：构建管道，以 CSV 格式将数据从本地 SQL Server 复制到 Blob 存储。 要使复制作业更快，应将 CSV 文件压缩为 bzip2 格式。

**测试和分析**：复制活动的吞吐量小于 2 MBps，这比性能基准慢得多。

**性能分析和优化**：为排除性能问题，可查看数据的处理和移动方式。

1. **读取数据**：Integration Runtime 建立与 SQL Server 的连接并发送查询。 SQL Server 通过 Intranet 向 Integration Runtime 发送数据流，以此进行响应。
2. **序列化和压缩数据**：Integration Runtime 将数据流序列化为 CSV 格式，并将数据压缩为 bzip2 流。
3. **写入数据**：Integration Runtime 通过 Internet 将 bzip2 流上传到 Blob 存储。

如你所见，数据以流式处理顺序方式进行处理和移动：SQL Server > LAN> Integration Runtime > WAN > Blob 存储。 **整体性能受管道中最小吞吐量的限制**。

![数据流](./media/copy-activity-performance/case-study-pic-1.png)

以下的一个或多个因素可能会导致性能瓶颈：

* **源**：SQL Server 本身由于负载过重而吞吐量低。
* **自承载 Integration Runtime**：
  * **LAN**：Integration Runtime 的位置离 SQL Server 计算机很远，且带宽连接低。
  * **Integration Runtime**：Integration Runtime 已达到执行以下操作的负载限制：
    * **序列化**：将数据流序列化为 CSV 格式时吞吐量缓慢。
    * **压缩**：选择慢速压缩编解码器（例如，bzip2，其采用 Core i7，速度为 2.8 MBps）。
  * **WAN**：企业网络和 Azure 服务之间的带宽低（例如，T1 = 1,544 kbps；T2 = 6,312 kbps）。
* **接收器**：Blob 存储吞吐量低。 （这种情况不太可能发生，因为其 SLA 保证至少有 60 MBps。）

在这种情况下，bzip2 数据压缩可能会拖慢整个管道的速度。 切换到 gzip 压缩编解码器可能会缓解此瓶颈。

## <a name="reference"></a>参考

下面是有关一些受支持数据存储的性能监视和优化参考：

* Azure 存储（包括 Blob 存储和表存储）：[Azure 存储可伸缩性目标](../storage/common/storage-scalability-targets.md)和 [Azure存储性能和可伸缩性清单](../storage/common/storage-performance-checklist.md)
* Azure SQL 数据库：可[监视性能](../sql-database/sql-database-single-database-monitor.md)并检查数据库事务单位 (DTU) 百分比
* Azure SQL 数据仓库：其功能以数据仓库单位 (DWU) 衡量；请参阅[管理 Azure SQL 数据仓库中的计算能力（概述）](../sql-data-warehouse/sql-data-warehouse-manage-compute-overview.md)
* Azure Cosmos DB：[Azure Cosmos DB 中的性能级别](../cosmos-db/performance-levels.md)
* 本地 SQL Server：[监视和优化性能](https://msdn.microsoft.com/library/ms189081.aspx)
* 本地文件服务器：[文件服务器性能优化](https://msdn.microsoft.com/library/dn567661.aspx)

## <a name="next-steps"></a>后续步骤
请参阅其他复制活动文章：

- [复制活动概述](copy-activity-overview.md)
- [复制活动架构映射](copy-activity-schema-and-type-mapping.md)
- [复制活动容错](copy-activity-fault-tolerance.md)
