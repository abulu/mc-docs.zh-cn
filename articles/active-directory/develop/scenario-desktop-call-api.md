---
title: 调用 Web API 的桌面应用（调用 Web API）- Microsoft 标识平台
description: 了解如何构建调用 Web API 的桌面应用（调用 Web API）
services: active-directory
documentationcenter: dev-center-name
author: jmprieur
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
ms.custom: aaddev
ms.collection: M365-identity-device-management
ms.openlocfilehash: 7d66f785a2448998fade918a8ee7b6b333fdf238
ms.sourcegitcommit: 9d5fd3184b6a47bf3b60ffdeeee22a08354ca6b1
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 06/21/2019
ms.locfileid: "67306025"
---
# <a name="desktop-app-that-calls-web-apis---call-a-web-api"></a>调用 Web API 的桌面应用 - 调用 Web API

现在你已有令牌，可以调用受保护的 Web API 了。

## <a name="calling-a-web-api-from-net"></a>通过 .NET 调用 Web API

[!INCLUDE [Call web API in .NET](../../../includes/active-directory-develop-scenarios-call-apis-dotnet.md)]

<!--
More includes will come later for Python and Java
-->

## <a name="calling-several-apis---incremental-consent"></a>调用若干 API - 增量许可

如果你需要为同一用户调用多个 API，则在获得了第一个 API 的令牌后，就可以直接调用 `AcquireTokenSilent`，在大多数情况下你将以无提示方式获取其他 API 的令牌。

```CSharp
var result = await app.AcquireTokenXX("scopeApi1")
                      .ExecuteAsync();

result = await app.AcquireTokenSilent("scopeApi2")
                  .ExecuteAsync();
```

需要交互的情况是：

- 用户已同意第一个 API，但现在需要同意更多范围（增量许可）
- 第一个 API 不需要多重身份验证，但下一个 API 需要。

```CSharp
var result = await app.AcquireTokenXX("scopeApi1")
                      .ExecuteAsync();

try
{
 result = await app.AcquireTokenSilent("scopeApi2")
                  .ExecuteAsync();
}
catch(MsalUiRequiredException ex)
{
 result = await app.AcquireTokenInteractive("scopeApi2")
                  .WithClaims(ex.Claims)
                  .ExecuteAsync();
}
```

## <a name="next-steps"></a>后续步骤

> [!div class="nextstepaction"]
> [移到生产环境](scenario-desktop-production.md)

