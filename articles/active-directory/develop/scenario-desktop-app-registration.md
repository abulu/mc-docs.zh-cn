---
title: 调用 Web API 的桌面应用（应用注册）- Microsoft 标识平台
description: 了解如何构建调用 Web API 的桌面应用（应用注册）
services: active-directory
documentationcenter: dev-center-name
author: jmprieur
manager: CelesteDG
editor: ''
ms.assetid: 820acdb7-d316-4c3b-8de9-79df48ba3b06
ms.service: active-directory
ms.subservice: develop
ms.devlang: na
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.workload: identity
origin.date: 04/18/2019
ms.date: 06/20/2019
ms.author: v-junlch
ms.custom: aaddev
ms.collection: M365-identity-device-management
ms.openlocfilehash: 312d4d0f2ecd2d42c094e38d15a64072b0d31964
ms.sourcegitcommit: 9d5fd3184b6a47bf3b60ffdeeee22a08354ca6b1
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 06/21/2019
ms.locfileid: "67305862"
---
# <a name="desktop-app-that-calls-web-apis---app-registration"></a>调用 Web API 的桌面应用 - 应用注册

本文包含适用于桌面应用程序的应用注册详细信息。

## <a name="supported-accounts-types"></a>支持的帐户类型

在桌面应用程序中支持的帐户类型取决于你想要启用的体验，因此也取决于你想要使用的流。

### <a name="audience-for-interactive-token-acquisition"></a>交互式令牌获取的受众

如果桌面应用程序使用交互式身份验证，则可通过任何[帐户类型](quickstart-register-app.md#register-a-new-application-using-the-azure-portal)登录用户

### <a name="audience-for-desktop-app-silent-flows"></a>桌面应用无提示流的受众

- 若要使用集成 Windows 身份验证或用户名/密码，应用程序需在你自己的租户（LOB 开发人员）或 Azure Active Directory 组织（ISV 方案）中登录用户。 
- 如果使用传入 B2C 颁发机构和策略的社交标识来登录用户，则只能使用交互式和用户-密码身份验证。

## <a name="redirect-uris"></a>重定向 URI

同样，可以在桌面应用程序中使用的重定向 URI 将取决于要使用的流。

- 如果使用交互式身份验证，则需使用 `https://login.partner.microsoftonline.cn/common/oauth2/nativeclient`。 单击应用程序的“身份验证”部分中的相应 URL 即可实现此配置 
  
  > [!IMPORTANT]
  > 目前，在默认情况下，MSAL.NET 会在 Windows 上运行的桌面应用程序中使用另一重定向 URI (`urn:ietf:wg:oauth:2.0:oob`)。 我们在将来需要更改此默认设置，因此建议你使用 `https://login.partner.microsoftonline.cn/common/oauth2/nativeclient`

- 如果应用仅使用集成 Windows 身份验证、用户名/密码或设备代码流，则不需为应用程序注册重定向 URI。 事实上，这些流会往返 Microsoft 标识平台 v2.0 终结点，因此不会在任何特定 URI 上回调你的应用程序。 为了将它们与也没有重定向 URI 的机密客户端应用程序流（在守护程序应用程序中使用的客户端凭据流）区分，需宣称你的应用程序是公共客户端应用程序。 实现此配置的方法是：转到应用程序的“身份验证”部分，  然后在“高级设置”子部分针对“将应用程序视为公共客户端”问题（位于“默认客户端类型”段落）选择“是”    

  ![允许公共客户端](./media/scenarios/default-client-type.png)

## <a name="api-permissions"></a>API 权限

桌面应用程序代表已登录用户调用 API。 它们需要请求委托的权限。 它们不能请求应用程序权限（这些只能在[守护程序应用程序](scenario-daemon-overview.md)中处理）

## <a name="next-steps"></a>后续步骤

> [!div class="nextstepaction"]
> [桌面应用 - 应用配置](scenario-desktop-app-configuration.md)

