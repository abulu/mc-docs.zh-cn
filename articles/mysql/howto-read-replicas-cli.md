---
title: 在 Azure Database for MySQL 中创建和管理只读副本
description: 本文介绍了如何使用 Azure CLI 在 Azure Database for MySQL 中设置和管理只读副本。
author: WenJason
ms.author: v-jay
ms.service: mysql
ms.topic: conceptual
origin.date: 05/28/2019
ms.date: 07/15/2019
ms.openlocfilehash: 3aa189a3d8764d0c4a6258f47211bdc615b1ea69
ms.sourcegitcommit: f4351979a313ac7b5700deab684d1153ae51d725
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 07/12/2019
ms.locfileid: "67845096"
---
# <a name="how-to-create-and-manage-read-replicas-in-azure-database-for-mysql-using-the-azure-cli"></a>如何使用 Azure CLI 在 Azure Database for MySQL 中创建和管理只读副本

> [!NOTE] 
> 将要查看的是 Azure Database for MySQL 的新服务。 若要查看经典 MySQL Database for Azure 的文档，请访问[此页](https://docs.azure.cn/zh-cn/mysql-database-on-azure/)。

在本文中，你将了解如何使用 Azure CLI 在与 Azure Database for MySQL 服务中的主服务器相同的 Azure 区域内创建和管理只读副本。

> [!NOTE]
> Azure CLI 尚不支持在与主服务器不同的区域中创建副本。 若要创建跨区域副本，请改用 [Azure 门户]( howto-read-replicas-portal.md)。

## <a name="prerequisites"></a>先决条件

- [安装 Azure CLI 2.0](/cli/install-azure-cli?view=azure-cli-latest)
- 将用作主服务器的 [Azure Database for MySQL 服务器](quickstart-create-mysql-server-database-using-azure-portal.md)。 

> [!IMPORTANT]
> 只读副本功能仅适用于“常规用途”或“内存优化”定价层中的 Azure Database for MySQL 服务器。 请确保主服务器位于其中一个定价层中。

## <a name="create-a-read-replica"></a>创建只读副本

可以使用以下命令创建只读副本服务器：

```azurecli
az mysql server replica create --name mydemoreplicaserver --source-server mydemoserver --resource-group myresourcegroup
```

`az mysql server replica create` 命令需要以下参数：

| 设置 | 示例值 | 说明  |
| --- | --- | --- |
| resource-group |  myresourcegroup |  要在其中创建副本服务器的资源组。  |
| name | mydemoreplicaserver | 所创建的新副本服务器的名称。 |
| source-server | mydemoserver | 要从中进行复制的现有主服务器的名称或 ID。 |

若要创建跨区域只读副本，请使用 `--location` 参数。 下面的 CLI 示例在中国北部创建副本。

```azurecli
az mysql server replica create --name mydemoreplicaserver --source-server mydemoserver --resource-group myresourcegroup --location chinanorth
```

> [!NOTE]
> 只读副本使用与主服务器相同的服务器配置创建。 副本服务器配置在创建后可以更改。 建议副本服务器的配置应保持在与主服务器相同或更大的值，以确保副本能够跟上主服务器。

## <a name="stop-replication-to-a-replica-server"></a>停止复制到副本服务器

> [!IMPORTANT]
> 停止复制到服务器操作不可逆。 一旦主服务器和副本服务器之间的复制停止，无法撤消。 然后，副本服务器将成为独立服务器，并且现在支持读取和写入。 此服务器不能再次成为副本服务器。

可以使用以下命令停止复制到只读副本服务器：

```azurecli
az mysql server replica stop --name mydemoreplicaserver --resource-group myresourcegroup
```

`az mysql server replica stop` 命令需要以下参数：

| 设置 | 示例值 | 说明  |
| --- | --- | --- |
| resource-group |  myresourcegroup |  副本服务器所在的资源组。  |
| name | mydemoreplicaserver | 要停止在其上进行复制的副本服务器的名称。 |

## <a name="delete-a-replica-server"></a>删除副本服务器

可以通过运行 **[az mysql server delete](/cli/mysql/server)** 命令删除只读副本服务器。

```azurecli
az mysql server delete --resource-group myresourcegroup --name mydemoreplicaserver
```

## <a name="delete-a-master-server"></a>删除主服务器

> [!IMPORTANT]
> 删除主服务器会停止复制到所有副本服务器，并删除主服务器本身。 副本服务器成为现在支持读取和写入的独立服务器。

若要删除主服务器，可以运行 **[az mysql server delete](/cli/mysql/server)** 命令。

```azurecli
az mysql server delete --resource-group myresourcegroup --name mydemoserver
```

## <a name="list-replicas-for-a-master-server"></a>列出主服务器的副本

若要查看给定的主服务器的所有副本，请运行以下命令： 

```azurecli
az mysql server replica list --server-name mydemoserver --resource-group myresourcegroup
```

`az mysql server replica list` 命令需要以下参数：

| 设置 | 示例值 | 说明  |
| --- | --- | --- |
| resource-group |  myresourcegroup |  要在其中创建副本服务器的资源组。  |
| server-name | mydemoserver | 主服务器的名称或 ID。 |

## <a name="next-steps"></a>后续步骤

- 详细了解[只读副本](concepts-read-replicas.md)
