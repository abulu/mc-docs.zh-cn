---
title: Azure 数据工厂中的数据集 | Microsoft Docs
description: 了解数据工厂中的数据集。 数据集表示输入/输出数据。
services: data-factory
documentationcenter: ''
author: WenJason
manager: digimobile
ms.reviewer: douglasl
ms.service: data-factory
ms.workload: data-services
ms.tgt_pltfrm: na
ms.topic: conceptual
origin.date: 04/25/2018
ms.date: 07/08/2019
ms.author: v-jay
ms.openlocfilehash: fa9a9fc3943426dfbe647d61ec4ded7f289f3559
ms.sourcegitcommit: 5191c30e72cbbfc65a27af7b6251f7e076ba9c88
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 07/05/2019
ms.locfileid: "67569891"
---
# <a name="datasets-in-azure-data-factory"></a>Azure 数据工厂中的数据集

本文介绍了数据集的涵义，采用 JSON 格式定义数据集的方式以及数据集在 Azure 数据工厂管道中的用法。

如果对数据工厂不熟悉，请参阅 [Azure 数据工厂简介](introduction.md)了解相关概述。

## <a name="overview"></a>概述
数据工厂可以包含一个或多个数据管道。 “管道”  是共同执行一项任务的活动  的逻辑分组。 管道中的活动定义对数据执行的操作。 现在，数据集这一名称的意义已经变为看待数据的一种方式，就是以输入和输出的形式指向或引用活动中要使用的数据   。 数据集可识别不同数据存储（如表、文件、文件夹和文档）中的数据。 例如，Azure Blob 数据集可在 Blob 存储中指定供活动读取数据的 Blob 容器和文件夹。

创建数据集之前，必须创建[链接服务](concepts-linked-services.md)，将数据存储链接到数据工厂  。 链接的服务类似于连接字符串，它定义数据工厂连接到外部资源时所需的连接信息。 不妨这样考虑：数据集代表链接的数据存储中的数据结构，而链接服务则定义到数据源的连接。 例如，Azure 存储链接服务可将存储帐户链接到数据工厂。 Azure Blob 数据集表示 blob 容器以及包含要处理的输入 blob 的 Azure 存储帐户的文件夹。

下面是一个示例方案。 要将数据从 Blob 存储复制到 SQL 数据库，请创建以下两个链接服务：Azure 存储和 Azure SQL 数据库。 然后创建两个数据集：Azure Blob 数据集（即 Azure 存储链接服务）和 Azure SQL 表数据集（即 Azure SQL 数据库链接服务）。 Azure 存储和 Azure SQL 数据库链接服务分别包含数据工厂在运行时用于连接到 Azure 存储和 Azure SQL 数据库的连接字符串。 Azure Blob 数据集指定 blob 容器和 blob 文件夹，该文件夹包含 Blob 存储中的输入 blob。 Azure SQL 表数据集指定要向其复制数据的 SQL 数据库中的 SQL 表。

下图显示了数据工厂中管道、活动、数据集和链接服务之间的关系：

![管道、活动、数据集和链接服务之间的关系](media/concepts-datasets-linked-services/relationship-between-data-factory-entities.png)


## <a name="dataset-json"></a>数据集 JSON
数据工厂中的数据集采用以下 JSON 格式定义：

```json
{
    "name": "<name of dataset>",
    "properties": {
        "type": "<type of dataset: AzureBlob, AzureSql etc...>",
        "linkedServiceName": {
                "referenceName": "<name of linked service>",
                "type": "LinkedServiceReference",
        },
        "structure": [
            {
                "name": "<Name of the column>",
                "type": "<Name of the type>"
            }
        ],
        "typeProperties": {
            "<type specific property>": "<value>",
            "<type specific property 2>": "<value 2>",
        }
    }
}
```
下表描述了上述 JSON 中的属性：

属性 | 说明 | 必须 |
-------- | ----------- | -------- |
name | 数据集名称。 请参阅 [Azure 数据工厂 - 命名规则](naming-rules.md)。 |  是 |
type | 数据集的类型。 指定数据工厂支持的类型之一（例如：AzureBlob、AzureSqlTable）。 <br/><br/>有关详细信息，请参阅[数据集类型](#dataset-type)。 | 是 |
structure | 数据集的架构。 有关详细信息，请参阅[数据集架构](#dataset-structure)。 | 否 |
typeProperties | 每种类型的类型属性各不相同（例如：Azure Blob、Azure SQL 表）。 若要详细了解受支持的类型及其属性，请参阅[数据集类型](#dataset-type)。 | 是 |

## <a name="dataset-example"></a>数据集示例
在以下示例中，数据集表示 SQL 数据库中名为 MyTable 的表。

```json
{
    "name": "DatasetSample",
    "properties": {
        "type": "AzureSqlTable",
        "linkedServiceName": {
                "referenceName": "MyAzureSqlLinkedService",
                "type": "LinkedServiceReference",
        },
        "typeProperties":
        {
            "tableName": "MyTable"
        },
    }
}

```
请注意以下几点：

- type 设置为 AzureSqlTable。
- tableName 类型属性（特定于 AzureSqlTable 类型）设置为 MyTable。
- linkedServiceName 引用 AzureSqlDatabase 类型的链接服务，该类型在下一 JSON 片段中定义。

## <a name="dataset-type"></a>数据集类型
数据集的类型很多，具体取决于使用的数据存储。 请参阅下表，获取数据工厂支持的数据存储的列表。 单击数据存储，了解如何创建链接服务和该数据存储的数据集。

[!INCLUDE [data-factory-v2-supported-data-stores](../../includes/data-factory-v2-supported-data-stores-dataflow.md)]

在上一节中的示例中，数据集的类型设置为 AzureSqlTable  。 同样，对于 Azure Blob 数据集，数据集的类型设置为 AzureBlob  ，如以下 JSON 中所示：

```json
{
    "name": "AzureBlobInput",
    "properties": {
        "type": "AzureBlob",
        "linkedServiceName": {
                "referenceName": "MyAzureStorageLinkedService",
                "type": "LinkedServiceReference",
        },

        "typeProperties": {
            "fileName": "input.log",
            "folderPath": "adfgetstarted/inputdata",
            "format": {
                "type": "TextFormat",
                "columnDelimiter": ","
            }
        }
    }
}
```

## <a name="dataset-structure"></a>数据集结构
使用结构部分提供用于隐藏类型以及将列从源映射到目标的类型信息。

结构中的每个列都包含以下属性：

属性 | 说明 | 必须
-------- | ----------- | --------
name | 列的名称。 | 是
type | 列的数据类型。 数据工厂支持以下临时数据类型作为允许的值：**Int16、Int32、Int64、Single、Double、Decimal、Byte[]、Boolean、String、Guid、Datetime、Datetimeoffset 和 Timespan** | 否
culture | 类型为 .NET 类型 `Datetime` 或 `Datetimeoffset` 时要使用的基于 .NET 的区域性。 默认为 `en-us`。 | 否
format | 类型为 .NET 类型 `Datetime` 或 `Datetimeoffset` 时要使用的格式字符串。 请参阅[自定义日期和时间格式字符串](https://docs.microsoft.com/dotnet/standard/base-types/custom-date-and-time-format-strings)，了解如何设置日期时间格式。 | 否

### <a name="example"></a>示例
在下面的示例中，假设源 Blob 数据采用 CSV 格式，并且包含三列： userid、name 和 lastlogindate。 它们的类型分别为 Int64、String 和 Datetime，并采用使用星期几的缩写法语名称的自定义日期时间格式。

将按如下所示定义 Blob 数据集结构并指定列的类型定义：

```json
"structure":
[
    { "name": "userid", "type": "Int64"},
    { "name": "name", "type": "String"},
    { "name": "lastlogindate", "type": "Datetime", "culture": "fr-fr", "format": "ddd-MM-YYYY"}
]
```

### <a name="guidance"></a>指南

若要了解何时加入结构信息以及在结构  部分包含哪些信息，请参阅以下指南。 详细了解数据工厂如何通过[架构和类型映射](copy-activity-schema-and-type-mapping.md)将源数据映射到接收器，以及何时指定结构信息。

- **对于强架构数据源**，仅当要将源列映射到接收器列且其名称不同时，才指定“结构”部分。 此类结构化的数据源将存储数据架构和类型信息，以及数据本身。 结构化的数据源的示例包括 SQL Server、Oracle 和 Azure SQL 数据库。<br/><br/>由于类型信息已可用于结构化数据源，因此包含结构部分时不应包含类型信息。
- **对于无/弱架构数据源（例如 blob 存储中的文本文件）** ，当数据集是复制活动的输入且应将源数据集的数据类型转换为接收器的本机类型时，请加入结构。 另外，当需要将源列映射到接收器列时，请加入结构。

## <a name="create-datasets"></a>创建数据集
可以使用以下任一工具或 SDK 创建数据集：[.NET API](quickstart-create-data-factory-dot-net.md)、[PowerShell](quickstart-create-data-factory-powershell.md)、[REST API](quickstart-create-data-factory-rest-api.md)、Azure 资源管理器模板和 Azure 门户

## <a name="next-steps"></a>后续步骤
请参阅以下教程，了解使用下列某个工具或 SDK 创建管道和数据集的分步说明。

- [快速入门：使用 .NET 创建数据工厂](quickstart-create-data-factory-dot-net.md)
- [快速入门：使用 PowerShell 创建数据工厂](quickstart-create-data-factory-powershell.md)
- [快速入门：使用 REST API 创建数据工厂](quickstart-create-data-factory-rest-api.md)
- [快速入门：使用 Azure 门户创建数据工厂](quickstart-create-data-factory-portal.md)
