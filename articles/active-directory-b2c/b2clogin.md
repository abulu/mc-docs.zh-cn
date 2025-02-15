---
title: 将重定向 URL 设置为 b2clogin.cn - Azure Active Directory B2C | Microsoft Docs
description: 了解如何在 Azure Active Directory B2C 的重定向 URL 中使用 b2clogin.cn。
services: active-directory-b2c
author: davidmu1
manager: daveba
ms.service: active-directory
ms.workload: identity
ms.topic: conceptual
origin.date: 01/28/2019
ms.date: 04/03/2019
ms.author: v-junlch
ms.subservice: B2C
ms.openlocfilehash: 4dc3adafd429405c756fe3cd5f2570eb6283315e
ms.sourcegitcommit: 623e8f0d52c42d236ad2a0136d5aebd6528dbee3
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 06/19/2019
ms.locfileid: "67235957"
---
# <a name="set-redirect-urls-to-b2clogincn-for-azure-active-directory-b2c"></a>将 Azure Active Directory B2C 的重定向 URL 设置为 b2clogin.cn

在你的 Azure Active Directory (Azure AD) B2C 应用程序中设置用于注册和登录的标识提供者时，需要指定一个重定向URL。 过去使用的是 login.partner.microsoftonline.cn，现在应当使用 b2clogin.cn。

使用 b2clogin.cn 可以提供更多优势，例如：

- Microsoft 服务在 cookie 标头中使用的空间就会减少。
- URL 不再包括对 Microsoft 的引用。 例如，`https://your-tenant-name.b2clogin.cn/tenant-id/oauth2/authresp`。

使用 b2clogin.cn 时，考虑可能需要更改的这些设置：

- 将标识提供者应用程序中的重定向 URL 设置为使用 b2clogin.cn。 
- 将 Azure AD B2C 应用程序设置为将 b2clogin.cn 用于用户流引用和令牌终结点。 
- 如果使用的是 MSAL，则需要将 **ValidateAuthority** 属性设置为 `false`。
- 请确保更改用户界面自定义的 CORS 设置中定义的任何“允许的源”  。  

## <a name="change-redirect-urls"></a>更改重定向 URL

若要使用 b2clogin.cn，请在标识提供者应用程序的设置中查找受信任 URL 的列表，并将其更改为重定向回 Azure AD B2C。  目前，你可能已将其设置为重定向回某个 login.partner.microsoftonline.cn 站点。 

你需要更改重定向 URL 以便 `your-tenant-name.b2clogin.cn` 经过授权。 请务必将 `your-tenant-name` 替换为你的 Azure AD B2C 租户的名称，并且，如果 URL 中存在 `/te`，请将其删除。 对于每个标识提供者，此 URL 略有不同，因此请检查相应的页面来获取确切的 URL。

可以通过以下文章查找标识提供者的设置信息：

- [LinkedIn](active-directory-b2c-setup-li-app.md)
- [GitHub](active-directory-b2c-setup-github-app.md)
- [微博](active-directory-b2c-setup-weibo-app.md)
- [QQ](active-directory-b2c-setup-qq-app.md)
- [微信](active-directory-b2c-setup-wechat-app.md)
- [Azure AD](active-directory-b2c-setup-oidc-azure-active-directory.md)
- [自定义 OIDC](active-directory-b2c-setup-oidc-idp.md)

## <a name="update-your-application"></a>更新应用程序

Azure AD B2C 应用程序可能在多个地方引用了 `login.partner.microsoftonline.cn`，例如用户流引用和令牌终结点。  请确保授权终结点、令牌终结点和证书颁发者都已更新为使用 `your-tenant-name.b2clogin.cn`。  

## <a name="set-the-validateauthority-property"></a>设置 ValidateAuthority 属性

如果使用的是 MSAL，请将 **ValidateAuthority** 属性设置为 `false`。 当 **ValidateAuthority** 设置为 `false` 时，允许重定向到 b2clogin.cn。 

以下示例展示了如何设置此属性：

在[适用于 .Net 的 MSAL](https://github.com/AzureAD/microsoft-authentication-library-for-dotnet) 中：

```CSharp
 ConfidentialClientApplication client = new ConfidentialClientApplication(...); // can also be PublicClientApplication
 client.ValidateAuthority = false;
```

在[适用于 JavaScript 的 MSAL](https://github.com/AzureAD/microsoft-authentication-library-for-js) 中：

```Javascript
this.clientApplication = new UserAgentApplication(
  env.auth.clientId,
  env.auth.loginAuthority,
  this.authCallback.bind(this),
  {
    validateAuthority: false
  }
);
```

