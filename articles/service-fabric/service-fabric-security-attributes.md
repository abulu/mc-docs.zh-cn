---
title: Azure Service Fabric 的常见安全属性
description: 用于评估 Azure Service Fabric 的常见安全属性的清单
services: service-fabric
documentationcenter: ''
author: rockboyfor
manager: digimobile
ms.service: service-fabric
ms.topic: conceptual
origin.date: 04/16/2019
ms.date: 07/08/2019
ms.author: v-yeche
ms.openlocfilehash: c76912d0775a026d77416d68409be818174984e6
ms.sourcegitcommit: 8f49da0084910bc97e4590fc1a8fe48dd4028e34
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 07/12/2019
ms.locfileid: "67844948"
---
# <a name="security-attributes-for-azure-service-fabric"></a>Azure Service Fabric 的安全属性

本文记录了内置到 Azure Service Fabric 中的安全属性。 

[!INCLUDE [Security Attributes Header](../../includes/security-attributes-header.md)]

## <a name="preventative"></a>预防

| 安全属性 | Yes/No | 注释 |
|---|---|--|
| 静态加密：<ul><li>服务器端加密</li><li>使用客户托管密钥的服务器端加密</li><li>其他加密功能（例如客户端、始终加密等）</ul>| 是 | 客户拥有群集以及作为群集构建基础的虚拟机规模集。 可以在虚拟机规模集上启用 Azure 磁盘加密。 |
| 传输中加密：<ul><li>快速路由加密</li><li>VNet 中加密</li><li>VNet-VNet 加密</ul>| 是 |  |
| 加密密钥处理（CMK、BYOK 等）| 是 | 客户拥有群集以及作为群集构建基础的虚拟机规模集。 可以在虚拟机规模集上启用 Azure 磁盘加密。 |
| 列级加密（Azure 数据服务）| 不适用 |  |
| 加密的 API 调用| 是 | Service Fabric API 调用通过 Azure 资源管理器进行。 需要有效 JSON web 令牌 (JWT)。 |

## <a name="network-segmentation"></a>网络分段

| 安全属性 | Yes/No | 注释 |
|---|---|--|
| 服务终结点支持| 是 |  |
| VNet 注入支持| 是 |  |
| 网络隔离和防火墙支持| 是 | 使用网络安全组 (NSG)。 |
| 强制隧道支持| 是 | Azure 网络支持强制隧道。 |

## <a name="detection"></a>检测

| 安全属性 | Yes/No | 注释|
|---|---|--|
| Azure 监视支持（Log Analytics 等）| 是 | 使用 Azure 监视支持和第三方支持。 |

<!--Not Available on App insights-->

## <a name="identity-and-access-management"></a>标识和访问管理

| 安全属性 | Yes/No | 注释|
|---|---|--|
| 身份验证| 是 | 身份验证通过 Azure Active Directory 来进行。 |
| 授权| 是 | 通过 SFRP 进行针对调用的标识和访问管理 (IAM)。 直接对群集终结点进行的调用支持两个角色：用户和管理员。客户可以将 API 映射到任一角色。 |

## <a name="audit-trail"></a>审核线索

| 安全属性 | Yes/No | 注释|
|---|---|--|
| 控制和管理平面日志记录和审核| 是 | 所有控制平面操作都需经过审核和审批流程。 |
| 数据平面日志记录和审核| 不适用 | 客户拥有群集。  |

## <a name="configuration-management"></a>配置管理

| 安全属性 | Yes/No | 注释|
|---|---|--|
| 配置管理支持（配置的版本控制等）| 是 | |

<!--Update_Description: wording update -->
