---
title: Azure Cosmos DB 的用于 MongoDB 的 API 简介
description: 了解如何使用 Azure Cosmos DB 通过其用于 MongoDB 的 API 来存储和查询大量数据。
ms.service: cosmos-db
ms.subservice: cosmosdb-mongo
ms.topic: overview
origin.date: 05/20/2019
ms.date: 06/17/2019
author: rockboyfor
ms.author: v-yeche
ms.openlocfilehash: 325167507e7bb2957db78978757e841b4ac3c6bb
ms.sourcegitcommit: 153236e4ad63e57ab2ae6ff1d4ca8b83221e3a1c
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 06/18/2019
ms.locfileid: "67171362"
---
# <a name="azure-cosmos-dbs-api-for-mongodb"></a>Azure Cosmos DB 的用于 MongoDB 的 API

[Azure Cosmos DB](introduction.md) 是世纪互联针对任务关键型应用程序提供的多区域分配式多模型数据库服务。 Azure Cosmos DB 在中国各地提供[统包式多区域分发](distribute-data-globally.md)、[吞吐量和存储的弹性扩展](partition-data.md)、99% 的情况下低至个位数的毫秒级延迟以及得到保证的高可用性，所有这些均由[行业领先的 SLA](https://www.azure.cn/support/sla/cosmos-db/) 提供支持。 Azure Cosmos DB [自动为数据编制索引](https://www.vldb.org/pvldb/vol8/p1668-shukla.pdf)，不需要你管理架构和索引。 它采用多种模型，支持文档、键-值、图形和列式数据模型。 默认情况下，可以使用 SQL API 来与 Cosmos DB 交互。 此外，Cosmos DB 服务对 Cassandra、MongoDB、Gremlin 和 Azure 表存储等常见 NoSQL API 实现网络协议。 这样，你便可以使用熟悉的 NoSQL 客户端驱动程序和工具来与 Cosmos 数据库交互。

## <a name="wire-protocol-compatibility"></a>网络协议兼容性

Azure Cosmos DB 服务对 Cassandra、MongoDB、Gremlin 和 Azure 表存储等常见的 NoSQL 数据库实现网络协议。 它在 Cosmos DB 中直接有效地提供网络协议的本机实现，使 NoSQL 数据库的现有客户端 SDK、驱动程序和工具能够以透明方式与 Cosmos DB 交互。 Cosmos DB 不使用数据库的任何源代码来为任何 NoSQL 数据库提供网络兼容的 API。

默认情况下，Azure Cosmos DB 的用于 MongoDB 的 API 与 MongoDB 网络协议版本 3.2 兼容。 在网络协议版本 3.4 中添加的功能或查询运算符目前以预览版功能形式提供。 任何理解这些协议版本的 MongoDB 客户端驱动程序都应该可以通过本机方式连接到 Cosmos DB。

![Azure Cosmos DB 的用于 MongoDB 的 API](./media/mongodb-introduction/cosmosdb-mongodb.png) 

## <a name="key-benefits"></a>主要优点

作为一种完全托管的多区域分布式数据库即服务，Cosmos DB 的主要优势详见[此处](introduction.md)。 另外，Cosmos DB 可以通过本机方式实现常用 NoSQL API 的线路协议，因此具备以下优势：

* 在保留大部分应用程序逻辑的情况下，轻松地将应用程序迁移到 Cosmos DB。
* 使应用程序可以移植并且始终与云供应商无关。
* 针对 Cosmos DB 支持的常用 NoSQL API，获取行业领先且享有财务支持的 SLA。
* 根据需求弹性缩放为 Cosmos 数据库预配的吞吐量和存储，只为所需的吞吐量和存储付费。 这样可以显著节省成本。
* 通过多主数据库复制功能实现统包式多区域分布。

## <a name="cosmos-dbs-api-for-mongodb"></a>Cosmos DB 的用于 MongoDB 的 API

遵循快速入门创建 Cosmos 帐户，并迁移现有 MongoDB 应用程序以使用 Azure Cosmos DB，或者生成一个新的应用程序：

* [迁移现有的 MongoDB Node.js Web 应用](create-mongodb-nodejs.md)。
* [使用 Azure Cosmos DB 的用于 MongoDB 的 API 和 .NET SDK 生成 Web 应用](create-mongodb-dotnet.md)
* [使用 Azure Cosmos DB 的用于 MongoDB 的 API 和 Java SDK 生成控制台应用](create-mongodb-java.md)

## <a name="next-steps"></a>后续步骤

下面是一些可帮助入门的指南：

* 在[将 MongoDB 应用程序连接到 Azure Cosmos DB](connect-mongodb-account.md) 教程中了解如何获取帐户连接字符串信息。
* 在[将 Studio 3T 与 Azure Cosmos DB 配合使用](mongodb-mongochef.md)教程中了解如何在 Studio 3T 中创建 Cosmos 数据库与 MongoDB 应用之间的连接。
* 在[将 MongoDB 数据导入 Azure Cosmos DB](mongodb-migrate.md) 教程中了解如何将数据导入 Cosmos 数据库。
* 使用 [Robo 3T](mongodb-robomongo.md) 连接到 Cosmos 帐户。
* 了解如何[配置多区域分布式应用的读取首选项](../cosmos-db/tutorial-global-distribution-mongodb.md)。

<sup>注意：本文介绍了可与 MongoDB 数据库实现线路协议兼容的 Azure Cosmos DB 功能。Azure 不会运行 MongoDB 数据库来提供此服务。Azure Cosmos DB 并不隶属于 MongoDB, inc.</sup>

<!--Update_Description: update meta properties, wording update -->