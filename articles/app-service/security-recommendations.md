---
title: 适用于 Azure 应用服务的安全建议
description: 适用于 Azure 应用服务的安全建议。 实施这些建议将有助于你履行我们的共享职责模型中描述的安全职责，并改进 Web 应用解决方案的总体安全性。
services: app-service
author: barclayn
manager: barbkess
ms.service: app-service
ms.topic: conceptual
origin.date: 06/17/2019
ms.date: 07/15/2019
ms.author: v-biyu
ms.openlocfilehash: 5bcf4fd687bc8387f1e4ae977a25357d9998e303
ms.sourcegitcommit: a829f1191e40d8940a5bf6074392973128cfe3c0
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 07/04/2019
ms.locfileid: "67560490"
---
# <a name="security-recommendations-for-app-service"></a>适用于应用服务的安全建议

本文包含适用于 Azure 应用服务的安全建议。 实施这些建议将有助于你履行我们的共享职责模型中描述的安全职责，并改进 Web 应用解决方案的总体安全性。

## <a name="general"></a>常规

| 建议 | 注释 |
|-|-|----|
| 保持最新状态 | 使用最新版的受支持平台、编程语言、协议和框架。 |

## <a name="identity-and-access-management"></a>标识和访问管理

| 建议 | 注释 |
|-|----|
| 禁用匿名访问 | 除非需要支持匿名请求，否则请禁用匿名访问。 有关 Azure 应用服务身份验证选项的详细信息，请参阅 [Azure 应用服务中的身份验证和授权](overview-authentication-authorization.md)。|
| 需要身份验证 | 在可能情况下，请使用应用服务身份验证模块，而不是编写代码来处理身份验证和授权。 请参阅 [Azure 应用服务中的身份验证和授权](overview-authentication-authorization.md)。 |
| 使用经身份验证的访问权限保护后端资源 | 可以使用用户标识或应用程序标识向后端资源进行身份验证。 选择使用应用程序标识时，请使用[托管标识](overview-managed-identity.md)。
| 需要客户端证书身份验证 | 客户端证书身份验证只允许从那些可以使用你提供的证书进行身份验证的客户端进行连接，因此可以改进安全性。 |

## <a name="data-protection"></a>数据保护

| 建议 | 注释 |
|-|-|
| 将 HTTP 重定向到 HTTPS | 默认情况下，客户端可以使用 HTTP 或 HTTPS 连接到 Web 应用。 建议将 HTTP 重定向到 HTTPS，因为 HTTPS 使用 SSL/TLS 协议来提供既加密又经过身份验证的安全连接。 |
| 加密与 Azure 资源的通信 | 当应用连接到 Azure 资源（例如 [SQL 数据库](https://www.azure.cn/zh-cn/home/features/sql-database/)或 [Azure 存储](/storage/)）时，连接一直保持在 Azure 中。 由于连接经过 Azure 中的共享网络，因此应始终加密所有通信。 |
| 需要尽可能新的 TLS 版本 | 从 2018 年开始，新的 Azure 应用服务应用使用 TLS 1.2。 更新版的 TLS 包含针对旧协议版本的安全改进。 |
| 使用 FTPS | 应用服务支持使用 FTP 和 FTPS 来部署文件。 尽可能使用 FTPS 而不是 FTP。 如果未使用这两种协议或其中一种协议，则应[将其禁用](deploy-ftp.md#enforce-ftps)。 |
| 保护应用程序数据 | 请勿将应用程序密钥（例如数据库凭据、API 令牌或私钥）存储在代码或配置文件中。 广为接受的方法是使用所选语言的标准模式将它们作为[环境变量](https://wikipedia.org/wiki/Environment_variable)进行访问。 在 Azure 应用服务中，可以通过[应用设置](web-sites-configure.md)和[连接字符串](web-sites-configure.md)定义环境变量。 应用设置和连接字符串以加密方式存储在 Azure 中。 只有在应用启动并将应用设置注入应用的进程内存中之前，才会对应用设置进行解密。 加密密钥会定期轮换。 或者，可以将 Azure 应用服务应用与 [Azure Key Vault](/azure/key-vault/) 集成，以实现高级密钥管理。 通过[使用托管标识访问 Key Vault](../key-vault/tutorial-web-application-keyvault.md)，应用服务应用可以安全地访问所需的机密。 |

## <a name="networking"></a>网络

| 建议 | 注释 |
|-|-|
| 使用静态 IP 限制 | 使用 Windows 上的 Azure 应用服务，可定义允许访问应用的 IP 地址的列表。 允许列表可包括单个 IP 地址或由子网掩码定义的 IP 地址范围。 有关详细信息，请参阅 [Azure 应用服务静态 IP 限制](app-service-ip-restrictions.md)。  |
| 选择独立定价层 | 除了独立定价层，所有层都在 Azure 应用服务的共享网络基础结构上运行应用。
| 在访问本地资源时使用安全连接 | 可以使用[混合连接](app-service-hybrid-connections.md)或[虚拟网络集成](web-sites-integrate-with-vnet.md)连接到本地资源。 |
| 限制向入站网络流量公开 | 可以通过网络安全组限制网络访问并控制公开的终结点数。 |

## <a name="monitoring"></a>监视

| 建议 | 注释 |
|-|-|
|使用 Azure 安全中心标准层 | [Azure 安全中心](../security-center/security-center-app-services.md)以原生方式集成 Azure 应用服务。 它可以运行评估并提供安全建议。 |

## <a name="next-steps"></a>后续步骤

请咨询应用程序提供商，看是否有其他安全要求。
