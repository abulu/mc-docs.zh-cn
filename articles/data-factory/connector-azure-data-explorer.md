---
title: 使用 Azure 数据工厂向/从 Azure 数据资源管理器复制数据
description: 了解如何通过在 Azure 数据工厂管道中使用复制活动来向/从 Azure 数据资源管理器复制数据。
services: data-factory
documentationcenter: ''
author: WenJason
manager: digimobile
ms.reviewer: douglasl
ms.service: data-factory
ms.workload: data-services
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: conceptual
origin.date: 04/16/2019
ms.date: 07/08/2019
ms.author: v-jay
ms.openlocfilehash: 220fa60ab90309de6db6097571dab6a4535fb9bb
ms.sourcegitcommit: 5191c30e72cbbfc65a27af7b6251f7e076ba9c88
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 07/05/2019
ms.locfileid: "67571496"
---
# <a name="copy-data-to-or-from-azure-data-explorer-using-azure-data-factory"></a>使用 Azure 数据工厂向/从 Azure 数据资源管理器复制数据

本文概述了如何在 Azure 数据工厂中使用复制活动向/从 [Azure 数据资源管理器](../data-explorer/data-explorer-overview.md)复制数据。 它是基于概述复制活动总体的[复制活动概述](copy-activity-overview.md)一文。

## <a name="supported-capabilities"></a>支持的功能

可以将数据从任何受支持的源数据存储复制到 Azure 数据资源管理器。 可以将数据从 Azure 数据资源管理器复制到任何受支持的接收器数据存储。 有关复制活动支持作为源或接收器的数据存储列表，请参阅[支持的数据存储](copy-activity-overview.md)表。

>[!NOTE]
>自版本 3.14 起，支持使用自承载集成运行时在 Azure 数据资源管理器与本地数据存储之间复制数据。

Azure 数据资源管理器连接器可用来执行以下操作：

* 将 Azure Active Directory (Azure AD) 应用程序令牌身份验证与**服务主体**配合使用来复制数据。
* 作为源时，请使用 KQL (Kusto) 查询来检索数据。
* 作为接收器时，请将数据追加到目标表。

## <a name="getting-started"></a>入门

[!INCLUDE [data-factory-v2-connector-get-started](../../includes/data-factory-v2-connector-get-started.md)]

对于特定于 Azure 数据资源管理器连接器的数据工厂实体，以下部分提供了有关用于定义这些实体的属性的详细信息。

## <a name="linked-service-properties"></a>链接服务属性

Azure 数据资源管理器连接器使用服务主体身份验证。 按照以下步骤获取服务主体并授予权限：

1. 遵循[将应用程序注册到 Azure AD 租户](../storage/common/storage-auth-aad-app.md#register-your-application-with-an-azure-ad-tenant)，在 Azure Active Directory (Azure AD) 中注册一个应用程序实体。 记下下面的值，这些值用于定义链接服务：

    - 应用程序 ID
    - 应用程序密钥
    - 租户 ID

2. 在 Azure 数据资源管理器中授予服务主体适当的权限。 请参阅[管理 Azure 数据资源管理器数据库权限](../data-explorer/manage-database-permissions.md)，获取有关角色和权限的详细信息以及有关管理权限的演练。 一般情况下，你需要

    - **作为源**，至少向数据库授予“数据库查看器”  角色。
    - **作为接收器**，至少向数据库授予“数据库引入器”  角色。

>[!NOTE]
>使用 ADF UI 进行创作时，列出链接服务上的数据库或列出数据集上的表的操作可能需要向服务主体授予更高的特权权限。 或者，可以选择手动输入数据库名称和表名称。 只要向服务主体授予读/写数据的适当权限，复制活动执行就可以正常工作。

Azure 数据资源管理器链接服务支持以下属性：

| 属性 | 说明 | 必选 |
|:--- |:--- |:--- |
| type | **type** 属性必须设置为 **AzureDataExplorer** | 是 |
| endpoint | Azure 数据资源管理器群集的终结点 URL，格式为 `https://<clusterName>.<regionName>.kusto.chinacloudapi.cn`。 | 是 |
| database | 数据库的名称。 | 是 |
| tenant | 指定应用程序的租户信息（域名或租户 ID）。 在 [Kusto 连接字符串](https://docs.microsoft.com/azure/kusto/api/connection-strings/kusto#application-authentication-properties)中，这通常称为“机构/授权 ID”  。 可以将鼠标指针悬停在 Azure 门户右上角来检索它。 | 是 |
| servicePrincipalId | 指定应用程序的客户端 ID。 在 [Kusto 连接字符串](https://docs.microsoft.com/azure/kusto/api/connection-strings/kusto#application-authentication-properties)中，这通常称为“AAD 应用程序客户端 ID”  。 | 是 |
| servicePrincipalKey | 指定应用程序的密钥。 在 [Kusto 连接字符串](https://docs.microsoft.com/azure/kusto/api/connection-strings/kusto#application-authentication-properties)中，这通常称为“AAD 应用程序密钥”  。 将此字段标记为 **SecureString** 以安全地将其存储在数据工厂中或[引用存储在 Azure Key Vault 中的机密](store-credentials-in-key-vault.md)。 | 是 |

**链接服务属性示例：**

```json
{
    "name": "AzureDataExplorerLinkedService",
    "properties": {
        "type": "AzureDataExplorer",
        "typeProperties": {
            "endpoint": "https://<clusterName>.<regionName>.kusto.chinacloudapi.cn",
            "database": "<database name>",
            "tenant": "<tenant name/id e.g. microsoft.partner.onmschina.cn>",
            "servicePrincipalId": "<service principal id>",
            "servicePrincipalKey": {
                "type": "SecureString",
                "value": "<service principal key>"
            }
        }
    }
}
```

## <a name="dataset-properties"></a>数据集属性

有关可用于定义数据集的各部分和属性的完整列表，请参阅[数据集](concepts-datasets-linked-services.md)一文。 本部分提供了 Azure 数据资源管理器数据集支持的属性列表。

若要将数据复制到 Azure 数据资源管理器，请将数据集的 type 属性设置为 **AzureDataExplorerTable**。

支持以下属性：

| 属性 | 说明 | 必选 |
|:--- |:--- |:--- |
| type | **type** 属性必须设置为 **AzureDataExplorerTable** | 是 |
| 表 | 链接服务引用的表的名称。 | 对于接收器为必需的，对于源不是必需的 |

**数据集属性示例**

```json
{
   "name": "AzureDataExplorerDataset",
    "properties": {
        "type": "AzureDataExplorerTable",
        "linkedServiceName": {
            "referenceName": "<Azure Data Explorer linked service name>",
            "type": "LinkedServiceReference"
        },
        "typeProperties": {
            "table": "<table name>"
        }
    }
}
```

## <a name="copy-activity-properties"></a>复制活动属性

有关可用于定义活动的各部分和属性的完整列表，请参阅[管道](concepts-pipelines-activities.md)一文。 本部分提供了 Azure 数据资源管理器源和接收器支持的属性列表。

### <a name="azure-data-explorer-as-source"></a>Azure 数据资源管理器作为源

若要从 Azure 数据资源管理器复制数据，请将复制活动源中的 **type** 属性设置为 **AzureDataExplorerSource**。 复制活动源  部分支持以下属性：

| 属性 | 说明 | 必选 |
|:--- |:--- |:--- |
| type | 复制活动源的 **type** 属性必须设置为：**AzureDataExplorerSource** | 是 |
| 查询 | 以 [KQL 格式](https://docs.microsoft.com/azure/kusto/query/)指定的只读请求。 使用自定义 KQL 查询作为参考。 | 是 |
| queryTimeout | 查询请求超时前的等待时间。默认值是 10 分钟 (00:10:00)；允许的最大值是 1 小时 (01:00:00)。 | 否 |

>[!NOTE]
>默认情况下，Azure 数据资源管理器源的大小限制为 500,000 条记录或 64 MB。 若要检索所有记录而不截断，可以在查询的开头指定 `set notruncation;`。 有关详细信息，请参阅[查询限制](https://docs.microsoft.com/azure/kusto/concepts/querylimits)。

**示例：**

```json
"activities":[
    {
        "name": "CopyFromAzureDataExplorer",
        "type": "Copy",
        "typeProperties": {
            "source": {
                "type": "AzureDataExplorerSource",
                "query": "TestTable1 | take 10",
                "queryTimeout": "00:10:00"
            },
            "sink": {
                "type": "<sink type>"
            }
        },
        "inputs": [
            {
                "referenceName": "<Azure Data Explorer input dataset name>",
                "type": "DatasetReference"
            }
        ],
        "outputs": [
            {
                "referenceName": "<output dataset name>",
                "type": "DatasetReference"
            }
        ]
    }
]
```

### <a name="azure-data-explorer-as-sink"></a>Azure 数据资源管理器作为接收器

若要将数据复制到 Azure 数据资源管理器，请将复制活动接收器中的 type 属性设置为 **AzureDataExplorerSink**。 复制活动接收器部分中支持以下属性  ：

| 属性 | 说明 | 必选 |
|:--- |:--- |:--- |
| type | 复制活动接收器的 **type** 属性必须设置为：**AzureDataExplorerSink** | 是 |
| ingestionMappingName | 基于 Kusto 表预先创建的 **[映射](https://docs.microsoft.com/azure/kusto/management/mappings#csv-mapping)** 的名称。 若要将源中的列映射到 Azure 数据资源管理器（适用于 **[所有支持的源存储/格式](copy-activity-overview.md#supported-data-stores-and-formats)** ，包括 CSV/JSON/Avro 格式等），可以使用复制活动[列映射](copy-activity-schema-and-type-mapping.md)（按名称隐式映射或按配置显式映射）和/或 Azure 数据资源管理器映射。 | 否 |

**示例：**

```json
"activities":[
    {
        "name": "CopyToAzureDataExplorer",
        "type": "Copy",
        "typeProperties": {
            "source": {
                "type": "<source type>"
            },
            "sink": {
                "type": "AzureDataExplorerSink",
                "ingestionMappingName": "<optional Azure Data Explorer mapping name>"
            }
        },
        "inputs": [
            {
                "referenceName": "<input dataset name>",
                "type": "DatasetReference"
            }
        ],
        "outputs": [
            {
                "referenceName": "<Azure Data Explorer output dataset name>",
                "type": "DatasetReference"
            }
        ]
    }
]
```

## <a name="next-steps"></a>后续步骤

* 有关 Azure 数据工厂中复制活动支持作为源和接收器的数据存储的列表，请参阅[支持的数据存储](copy-activity-overview.md#supported-data-stores-and-formats)。
