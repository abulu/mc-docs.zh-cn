---
title: Microsoft 标识平台身份验证库 | Microsoft Docs
description: Microsoft 标识平台终结点的兼容客户端库和服务器中间件库，以及相关的库、源代码和示例链接。
services: active-directory
documentationcenter: ''
author: negoe
manager: CelesteDG
editor: ''
ms.assetid: 19cec615-e51f-4141-9f8c-aaf38ff9f746
ms.service: active-directory
ms.subservice: develop
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: identity
origin.date: 05/07/2019
ms.date: 07/01/2019
ms.author: v-junlch
ms.reviewer: jmprieur, saeeda
ms.custom: aaddev
ms.collection: M365-identity-device-management
ms.openlocfilehash: 096ff138d238764e3686d7274684ae01a51ea9d4
ms.sourcegitcommit: 5f85d6fe825db38579684ee1b621d19b22eeff57
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 07/05/2019
ms.locfileid: "67568729"
---
# <a name="microsoft-identity-platform-authentication-libraries"></a>Microsoft 标识平台身份验证库

[Microsoft 标识平台终结点](azure-ad-endpoint-comparison.md)支持行业标准协议 OAuth 2.0 和 OpenID Connect 1.0。 Microsoft 身份验证库 (MSAL) 设计为适用于 Microsoft 标识平台终结点。 还可以使用支持 OAuth 2.0 和 OpenID Connect 1.0 的开放源代码库。

建议使用协议领域的专家根据安全开发生命周期 (SDL) 方法（例如 [Microsoft 遵循的方法][Microsoft-SDL]）编写的库。 如果决定手动编写协议代码，应遵循 Microsoft 的 SDL 等方法并密切注意每个协议的标准规范中的安全注意事项。

> [!NOTE]
> 正在寻找 Azure AD 身份验证库 (ADAL)？ 请查看 [ADAL 库指南](active-directory-authentication-libraries.md)。

## <a name="types-of-libraries"></a>库的类型

Microsoft 标识平台终结点使用两种类型的库：

* **客户端库**：本机客户端和服务器使用客户端库获取用于调用某个资源（例如 Microsoft Graph）的访问令牌。
* **服务器中间件库**：Web 应用使用服务器中间件库进行用户登录。 Web API 使用服务器中间件库验证本机客户端或其他服务器发送的令牌。

## <a name="library-support"></a>库支持

库的支持类型有两种：

* **Microsoft 支持**：Microsoft 为这些库提供修补程序，并对这些库进行 SDL 审慎调查。
* **兼容**：Microsoft 已在基本方案中测试这些库并确认它们适用于 Microsoft 标识平台终结点。 Microsoft 不提供这些库的修复程序，且尚未审查这些库。 问题和功能请求应重定向到库的开源项目。

有关适用于 Microsoft 标识平台终结点的库列表，请参阅本文的后续部分。

## <a name="microsoft-supported-client-libraries"></a>Microsoft 支持的客户端库

客户端身份验证库用于获取令牌以调用受保护的 Web API

| 平台 | 库 | 下载 | 源代码 | 示例 | 参考 | 概念文档 | 路线图 |
| --- | --- | --- | --- | --- | --- | --- | ---|
| ![Javascript](./media/sample-v2-code/logo_js.png) | MSAL.js  | [NPM](https://www.npmjs.com/package/msal) |[GitHub](https://github.com/AzureAD/microsoft-authentication-library-for-js/blob/dev/lib/msal-angularjs/README.md) |  [单页应用](https://github.com/Azure-Samples/active-directory-javascript-singlepageapp-dotnet-webapi-v2) | [引用](https://htmlpreview.github.io/?https://raw.githubusercontent.com/AzureAD/microsoft-authentication-library-for-js/blob/dev/lib/msal-core/docs/classes/_useragentapplication_.useragentapplication.html) | [概念文档](msal-overview.md)| [路线图](https://github.com/AzureAD/microsoft-authentication-library-for-js/wiki#roadmap)
|![Angular JS](./media/sample-v2-code/logo_angular.png) | MSAL Angular JS | [NPM](https://www.npmjs.com/package/@azure/msal-angularjs) | [GitHub](https://github.com/AzureAD/microsoft-authentication-library-for-js/blob/dev/lib/msal-angularjs/README.md) |  |  | |
![Angular](./media/sample-v2-code/logo_angular.png) | MSAL Angular（预览） | [NPM](https://www.npmjs.com/package/@azure/msal-angular) |[GitHub](https://github.com/AzureAD/microsoft-authentication-library-for-js/blob/dev/lib/msal-angular/README.md) | | | |
| ![.NET framework](./media/sample-v2-code/logo_NET.png) ![UWP](./media/sample-v2-code/logo_windows.png) ![Xamarin](./media/sample-v2-code/logo_xamarin.png) | MSAL .NET  |[NuGet](https://www.nuget.org/packages/Microsoft.Identity.Client) |[GitHub](https://github.com/AzureAD/microsoft-authentication-library-for-dotnet) | [桌面应用](tutorial-v2-windows-desktop.md) | [MSAL.NET](https://docs.azure.cn/zh-cn/dotnet/api/microsoft.identity.client?view=azure-dotnet-preview) |[wiki](https://github.com/AzureAD/microsoft-authentication-library-for-dotnet/wiki#conceptual-documentation) | [路线图](https://github.com/AzureAD/microsoft-authentication-library-for-dotnet/wiki#roadmap)
| ![Python](./media/sample-v2-code/logo_python.png) | MSAL Python（预览版） | [PyPI](https://pypi.org/project/msal) | [GitHub](https://github.com/AzureAD/microsoft-authentication-library-for-python) | [示例](https://github.com/AzureAD/microsoft-authentication-library-for-python/tree/dev/sample) | [ReadTheDocs](https://msal-python.rtfd.io/) | [wiki](https://github.com/AzureAD/microsoft-authentication-library-for-python/wiki) | [路线图](https://github.com/AzureAD/microsoft-authentication-library-for-python/wiki/Roadmap)
| ![Java](./media/sample-v2-code/logo_java.png) | MSAL Java（预览版） | [Maven](https://mvnrepository.com/artifact/com.microsoft.azure/msal4j) | [GitHub](https://github.com/AzureAD/microsoft-authentication-library-for-java) | [示例](https://github.com/AzureAD/microsoft-authentication-library-for-java/tree/dev/src/samples) | | | [路线图](https://github.com/AzureAD/microsoft-authentication-library-for-java/wiki)
| ![iOS / Objective C 或 swift](./media/sample-v2-code/logo_iOS.png) | MSAL obj_c（预览） | [GitHub](https://github.com/AzureAD/microsoft-authentication-library-for-objc) |[GitHub](https://github.com/AzureAD/microsoft-authentication-library-for-objc) | [iOS 应用](https://github.com/Azure-Samples/active-directory-msal-ios-swift) |  |
|![Android / Java](./media/sample-v2-code/logo_Android.png) | MSAL（预览） | [中央存储库](https://repo1.maven.org/maven2/com/microsoft/identity/client/msal/) |[GitHub](https://github.com/AzureAD/microsoft-authentication-library-for-android) | [Android 应用](quickstart-v2-android.md) | [JavaDocs](https://javadoc.io/doc/com.microsoft.identity.client/msal) | | |

## <a name="microsoft-supported-server-middleware-libraries"></a>Microsoft 支持的服务器中间件库

中间件库用于保护 Web 应用程序和 Web API。 对于使用 ASP.NET 或 ASP.NET Core 编写的 Web 应用或 Web API，ASP.NET/ASP.NET Core 使用中间件库

| 平台 | 库 | 下载 | 源代码 | 示例 | 参考
| --- | --- | --- | --- | --- | --- |
| ![.NET](./media/sample-v2-code/logo_NET.png) ![.NET Core](./media/sample-v2-code/logo_NETcore.png) | ASP.NET 安全性 |[NuGet](https://www.nuget.org/packages/Microsoft.AspNet.Mvc/) |[GitHub](https://github.com/aspnet/AspNetCore) |[MVC 应用](quickstart-v2-aspnet-webapp.md) |[ASP.NET API 参考](https://docs.azure.cn/zh-cn/dotnet/api/overview?view=aspnetcore-2.0) |
| ![.NET](./media/sample-v2-code/logo_NET.png)| 适用于 .NET 的 IdentityModel 扩展| |[GitHub](https://github.com/AzureAD/azure-activedirectory-identitymodel-extensions-for-dotnet) | [MVC 应用](quickstart-v2-aspnet-webapp.md) |[引用](https://docs.azure.cn/zh-cn/dotnet/api/overview/activedirectory/client?view=azure-dotnet) |
| ![Node.js](./media/sample-v2-code/logo_nodejs.png) | Azure AD Passport |[NPM](https://www.npmjs.com/package/passport-azure-ad) |[GitHub](https://github.com/AzureAD/passport-azure-ad) | [Web 应用](https://github.com/AzureADQuickStarts/AppModelv2-WebApp-OpenIDConnect-nodejs) | |

## <a name="compatible-client-libraries"></a>兼容的客户端库

| 平台 | 库名称 | 测试的版本 | 源代码 | 示例 |
|:---:|:---:|:---:|:---:|:---:|
|![Javascript](./media/sample-v2-code/logo_js.png)|[Hello.js](https://adodson.com/hello.js/) | 1.13.5 |[Hello.js](https://github.com/MrSwitch/hello.js) |[SPA](https://github.com/Azure-Samples/active-directory-javascript-graphapi-web-v2) |
| ![Java](./media/sample-v2-code/logo_java.png) | [Scribe Java](https://github.com/scribejava/scribejava) | [版本 3.2.0](https://github.com/scribejava/scribejava/releases/tag/scribejava-3.2.0) | [ScribeJava](https://github.com/scribejava/scribejava/) | |
| ![Java](./media/sample-v2-code/logo_java.png) | [Gluu OpenID Connect 库](https://github.com/GluuFederation/oxAuth) | [版本 3.0.2](https://github.com/GluuFederation/oxAuth/releases/tag/3.0.2) | [Gluu OpenID Connect 库](https://github.com/GluuFederation/oxAuth) | |
| ![Python](./media/sample-v2-code/logo_python.png) | [Requests-OAuthlib](https://github.com/requests/requests-oauthlib) | [版本 1.2.0](https://github.com/requests/requests-oauthlib/releases/tag/v1.2.0) | [Requests-OAuthlib](https://github.com/requests/requests-oauthlib) | |
| ![Node.js](./media/sample-v2-code/logo_nodejs.png) | [openid-client](https://github.com/panva/node-openid-client) | [版本 2.4.5](https://github.com/panva/node-openid-client/releases/tag/v2.4.5) | [openid-client](https://github.com/panva/node-openid-client) | |
| ![PHP](./media/sample-v2-code/logo_php.png) | [The PHP League oauth2-client](https://github.com/thephpleague/oauth2-client) | [版本 1.4.2](https://github.com/thephpleague/oauth2-client/releases/tag/1.4.2) | [oauth2-client](https://github.com/thephpleague/oauth2-client/) | |
| ![Ruby](./media/sample-v2-code/logo_ruby.png) |[OmniAuth](https://github.com/omniauth/omniauth/wiki) |omniauth:1.3.1<br />omniauth-oauth2:1.4.0 |[OmniAuth](https://github.com/omniauth/omniauth)<br />[OmniAuth OAuth2](https://github.com/intridea/omniauth-oauth2) |  |
| ![iOS](./media/sample-v2-code/logo_iOS.png) ![Android](./media/sample-v2-code/logo_Android.png) | [React Native 应用身份验证](https://github.com/FormidableLabs/react-native-app-auth) | [版本 4.2.0](https://github.com/FormidableLabs/react-native-app-auth/releases/tag/v4.2.0) | [React Native 应用身份验证](https://github.com/FormidableLabs/react-native-app-auth) | |

对于任何符合标准的库，都可以使用 Microsoft 标识平台终结点，因此了解去哪里寻求支持非常重要。

* 有关库代码中的问题和新功能请求，请联系库所有者。
* 有关服务端协议实现中的问题和新功能请求，请联系 Microsoft。
* 有关要在协议中看到的其他功能，请[提出功能请求](https://feedback.azure.com/forums/169401-azure-active-directory)。

## <a name="related-content"></a>相关内容

有关 Microsoft 标识平台终结点的详细信息，请参阅 [Microsoft 标识平台概述][AAD-App-Model-V2-Overview]。

<!--Image references-->

<!--Reference style links -->
[AAD-App-Model-V2-Overview]: v2-overview.md
[ClientLib-NET-Lib]: https://www.nuget.org/packages/Microsoft.Identity.Client
[ClientLib-NET-Repo]: https://github.com/AzureAD/microsoft-authentication-library-for-dotnet
[ClientLib-Node-Lib]: https://www.npmjs.com/package/passport-azure-ad
[ClientLib-Node-Repo]: https://github.com/AzureAD/passport-azure-ad
[ClientLib-Node-Sample]:/
[ClientLib-Iosmac-Lib]:/
[ClientLib-Iosmac-Repo]:/
[ClientLib-Iosmac-Sample]:/
[ClientLib-Android-Lib]:/
[ClientLib-Android-Repo]:/
[ClientLib-Android-Sample]:/
[ClientLib-Js-Lib]:/
[ClientLib-Js-Repo]:/
[ClientLib-Js-Sample]:/

[Microsoft-SDL]: https://www.microsoft.com/sdl/default.aspx
[ServerLib-Net4-Owin-Oidc-Lib]: https://www.nuget.org/packages/Microsoft.Owin.Security.OpenIdConnect/
[ServerLib-Net4-Owin-Oidc-Repo]: https://katanaproject.codeplex.com/
[ServerLib-Net4-Owin-Oauth-Lib]: https://www.nuget.org/packages/Microsoft.Owin.Security.OAuth/
[ServerLib-Net4-Owin-Oauth-Repo]: https://katanaproject.codeplex.com/
[ServerLib-Net-Jwt-Lib]: https://www.nuget.org/packages/System.IdentityModel.Tokens.Jwt
[ServerLib-Net-Jwt-Repo]: https://github.com/AzureAD/azure-activedirectory-identitymodel-extensions-for-dotnet
[ServerLib-Net-Jwt-Sample]:/
[ServerLib-NetCore-Owin-Oidc-Lib]: https://www.nuget.org/packages/Microsoft.AspNetCore.Authentication.OpenIdConnect/
[ServerLib-NetCore-Owin-Oidc-Repo]: https://github.com/aspnet/Security
[ServerLib-NetCore-Owin-Oidc-Sample]: https://github.com/Azure-Samples/active-directory-dotnet-webapp-openidconnect-aspnetcore-v2
[ServerLib-NetCore-Owin-Oauth-Lib]: https://www.nuget.org/packages/Microsoft.AspNetCore.Authentication.OAuth/
[ServerLib-NetCore-Owin-Oauth-Repo]: https://github.com/aspnet/Security
[ServerLib-NetCore-Owin-Oauth-Sample]:/
[ServerLib-Node-Lib]: https://www.npmjs.com/package/passport-azure-ad
[ServerLib-Node-Repo]: https://github.com/AzureAD/passport-azure-ad/

<!-- Update_Description: wording update -->