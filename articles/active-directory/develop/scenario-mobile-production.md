---
title: 调用 Web API 的移动应用（移到生产环境）- Microsoft 标识平台
description: 了解如何构建调用 Web API 的移动应用（移到生产环境）
services: active-directory
documentationcenter: dev-center-name
author: danieldobalian
manager: CelesteDG
ms.service: active-directory
ms.subservice: develop
ms.devlang: na
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.workload: identity
origin.date: 05/07/2019
ms.date: 06/20/2019
ms.author: v-junlch
ms.reviwer: brandwe
ms.custom: aaddev
ms.collection: M365-identity-device-management
ms.openlocfilehash: 893545c67e82328294c28af298e7dae6b3d2dc15
ms.sourcegitcommit: 9d5fd3184b6a47bf3b60ffdeeee22a08354ca6b1
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 06/21/2019
ms.locfileid: "67305906"
---
# <a name="mobile-app-that-calls-web-apis---move-to-production"></a>调用 Web API 的移动应用 - 移到生产环境

本文详细介绍了如何在将应用移到生产环境之前提高应用的质量和可靠性。

## <a name="handling-errors-in-mobile-applications"></a>在移动应用程序中处理错误

此时，应用中可能会出现许多错误情况。 要处理的主要情况是无提示失败和回退到交互。 生产时应考虑的其他情况包括无网络情况、服务中断、管理员同意的要求以及其他特定于场景的情况。

每个 MSAL 库都有说明如何处理这些情况的示例代码和 wiki 内容：

- [MSAL Android Wiki](https://github.com/AzureAD/microsoft-authentication-library-for-android)
- [MSAL iOS Wiki](https://github.com/AzureAD/microsoft-authentication-library-for-objc/wiki)
- [MSAL.NET Wiki](https://github.com/AzureAD/microsoft-authentication-library-for-dotnet/wiki)

## <a name="mitigating-and-investigating-issues"></a>缓解和调查问题

若要诊断应用中的问题，最好先收集数据。 有关可以收集的数据类型的信息，请参阅 MSAL 平台 wiki。

- 用户在遇到问题时可能会寻求帮助。 最佳做法是捕获和临时存储日志，并提供用户可以将日志上传到的位置。 MSAL 提供日志记录扩展来捕获有关身份验证的详细信息。
- 如果该扩展可用，请通过 MSAL 启用遥测，以收集有关用户如何登录应用的数据。

## <a name="next-steps"></a>后续步骤

[!INCLUDE [Move to production common steps](../../../includes/active-directory-develop-scenarios-production.md)]

