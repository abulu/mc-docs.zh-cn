---
title: 在 Azure Database for MySQL 中配置服务参数
description: 本文介绍如何使用 Azure CLI 命令行实用工具在 Azure Database for MySQL 中配置服务参数。
author: WenJason
ms.author: v-jay
ms.service: mysql
ms.devlang: azurecli
ms.topic: conceptual
origin.date: 07/18/2018
ms.date: 07/15/2019
ms.openlocfilehash: 6f8cea892a76a5600901d2aa47ea56df0670bb72
ms.sourcegitcommit: f4351979a313ac7b5700deab684d1153ae51d725
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 07/12/2019
ms.locfileid: "67845097"
---
# <a name="customize-server-configuration-parameters-by-using-azure-cli"></a>使用 Azure CLI 自定义服务器配置参数

> [!NOTE]
> 将要查看的是 Azure Database for MySQL 的新服务。 若要查看经典 MySQL Database for Azure 的文档，请访问[此页](https://docs.azure.cn/zh-cn/mysql-database-on-azure/)。

可以使用 Azure CLI、Azure 命令行实用工具来列出、显示和更新 Azure Database for MySQL 服务器的配置参数。 在服务器级别会公开引擎配置的一个子集，并可以进行修改。 

## <a name="prerequisites"></a>先决条件
若要逐步执行本操作方法指南，需要：
- [Azure Database for MySQL 服务器](quickstart-create-mysql-server-database-using-azure-cli.md)
- [Azure CLI](/cli/install-azure-cli) 命令行实用工具。

## <a name="list-server-configuration-parameters-for-azure-database-for-mysql-server"></a>列出 Azure Database for MySQL 服务器的服务器配置参数
若要列出服务器中的所有可修改参数及其值，请运行 [az mysql server configuration list](/cli/mysql/server/configuration#az-mysql-server-configuration-list) 命令。

可以列出资源组“myresourcegroup”  下服务器 **mydemoserver.mysql.database.chinacloudapi.cn** 的服务器配置参数。
```cli
az mysql server configuration list --resource-group myresourcegroup --server mydemoserver
```
有关每个列出参数的定义，请参阅[服务器系统变量](https://dev.mysql.com/doc/refman/5.7/en/server-system-variables.html)上的 MySQL 引用部分。

## <a name="show-server-configuration-parameter-details"></a>显示服务器配置参数详细信息
若要显示服务器的某个特定配置参数的详细信息，请运行 [az mysql server configuration show](/cli/mysql/server/configuration#az-mysql-server-configuration-show) 命令。

本示例显示了资源组“myresourcegroup”  下服务器 **mydemoserver.mysql.database.chinacloudapi.cn** 的服务器配置参数 **slow\_query\_log** 的详细信息。
```cli
az mysql server configuration show --name slow_query_log --resource-group myresourcegroup --server mydemoserver
```
## <a name="modify-a-server-configuration-parameter-value"></a>修改服务器配置参数值
此外，你还可以修改某个服务器配置参数的值，这会更新 MySQL 服务器引擎的基础配置值。 若要更新配置，请使用 [az mysql server configuration set](/cli/mysql/server/configuration#az-mysql-server-configuration-set) 命令。 

更新资源组“myresourcegroup”  下服务器 **mydemoserver.mysql.database.chinacloudapi.cn** 的服务器配置参数 **slow\_query\_log**。
```cli
az mysql server configuration set --name slow_query_log --resource-group myresourcegroup --server mydemoserver --value ON
```
若要重置配置参数的值，省去可选的 `--value` 参数，服务将应用默认值。 对于上述示例，它将如下所示：
```cli
az mysql server configuration set --name slow_query_log --resource-group myresourcegroup --server mydemoserver
```
此代码会将 slow\_query\_log  配置重置为默认值 OFF  。 

## <a name="next-steps"></a>后续步骤

- 如何配置 [Azure 门户中的服务器参数](howto-server-parameters.md)