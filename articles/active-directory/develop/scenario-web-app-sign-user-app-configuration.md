---
title: 用于登录用户的 Web 应用（代码配置）- Microsoft 标识平台
description: 了解如何构建用于登录用户的 Web 应用（代码配置）
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
ms.openlocfilehash: a714f1c473d5b3cf0da46a0811a5c5dd6d33e42a
ms.sourcegitcommit: 9d5fd3184b6a47bf3b60ffdeeee22a08354ca6b1
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 06/21/2019
ms.locfileid: "67306000"
---
# <a name="web-app-that-signs-in-users---code-configuration"></a>用于登录用户的 Web 应用 - 代码配置

了解如何为用于登录用户的 Web 应用配置代码。

## <a name="libraries-used-to-protect-web-apps"></a>库用于保护 Web 应用

<!-- This section can be in an include for Web App and Web APIs -->
用于保护 Web 应用（和 Web API）的库为：

| 平台 | 库 | 说明 |
|----------|---------|-------------|
| ![.NET](./media/sample-v2-code/logo_net.png) | [适用于 .NET 的标识模型扩展](https://github.com/AzureAD/azure-activedirectory-identitymodel-extensions-for-dotnet/wiki) | 在由 ASP.NET 和 ASP.NET Core 直接使用的情况下，适用于 .NET 的 Microsoft 标识扩展提议了一组在 .NET Framework 和 .NET Core 上运行的 DLL。 在 ASP.NET/ASP.NET Core Web 应用中，可以使用 **TokenValidationParameters** 类控制令牌验证（尤其适用于某些 ISV 方案） |

## <a name="aspnet-core-configuration"></a>ASP.NET Core 配置

### <a name="application-configuration-files"></a>应用程序配置文件

在 ASP.NET Core 中，使用 Microsoft 标识平台登录用户的 Web 应用程序通过 `appsettings.json` 文件进行配置。 需填充的设置为：

- 云 `Instance`（如果需要应用在国家云中运行）
- `tenantId` 中的受众
- 应用程序的 `clientId`，从 Azure 门户中复制。

```JSon
{
  "AzureAd": {
    "Instance": "https://login.partner.microsoftonline.cn/",

    // Azure AD Audience among:
    // - the tenant Id as a a GUID obtained from the azure portal to sign-in users in your organization
    // - "organizations" to sign-in users in any work or school accounts
    // - "common" to sign-in users with any work and school account
    "TenantId": "[Enter the tenantId here]]",

    // Client Id (Application ID) obtained from the Azure portal
    "ClientId": "[Enter the Client Id]",
    "CallbackPath": "/signin-oidc",
    "SignedOutCallbackPath ": "/signout-callback-oidc"
  }
}
```

在 ASP.NET Core 中，有另一个文件包含应用程序的 URL (`applicationUrl`) 和 SSL 端口 (`sslPort`)，此外还有各种配置文件。

```JSon
{
  "iisSettings": {
    "windowsAuthentication": false,
    "anonymousAuthentication": true,
    "iisExpress": {
      "applicationUrl": "http://localhost:3110/",
      "sslPort": 44321
    }
  },
  "profiles": {
    "IIS Express": {
      "commandName": "IISExpress",
      "launchBrowser": true,
      "environmentVariables": {
        "ASPNETCORE_ENVIRONMENT": "Development"
      }
    },
    "webApp": {
      "commandName": "Project",
      "launchBrowser": true,
      "environmentVariables": {
        "ASPNETCORE_ENVIRONMENT": "Development"
      },
      "applicationUrl": "http://localhost:3110/"
    }
  }
}
```

在 Azure 门户中，需要在“身份验证”页中为应用程序注册的回复 URI  必须与这些 URL 匹配；也就是说，对于上面的两个配置文件，它们需要是 `https://localhost:44321/signin-oidc`（因为 applicationUrl 是 `http://localhost:3110`），但 `sslPort` 已指定 (44321)，且 `CallbackPath` 为 `/signin-oidc`（在 `appsettings.json` 中定义）。
  
注销 URI 将采用相同方式设置为 `https://localhost:44321/signout-callback-oidc`。

### <a name="initialization-code"></a>初始化代码

在 ASP.NET Core Web 应用（和 Web API）中，执行应用程序初始化的代码位于 `Startup.cs` 文件中。若要使用 Microsoft 标识平台（以前称为 Azure AD）v2.0 添加身份验证，需添加以下代码。 代码中的注释应该浅显易懂。

  > [!NOTE]
  > 如果通过 Visual Studio 或 `dotnet new mvc` 使用默认的 ASP.NET Core Web 项目来启动项目，则可默认使用 `AddAzureAD` 方法，因为相关的包会自动加载。 但是，如果从头生成一个项目并尝试使用以下代码，则建议将 NuGet 包“Microsoft.AspNetCore.Authentication.AzureAD.UI”  添加到项目，使 `AddAzureAD` 方法可用。
  
```CSharp
 services.AddAuthentication(AzureADDefaults.AuthenticationScheme)
         .AddAzureAD(options => configuration.Bind("AzureAd", options));

 services.Configure<OpenIdConnectOptions>(AzureADDefaults.OpenIdScheme, options =>
 {
  // The ASP.NET core templates are currently using Azure AD v1.0, and compute
  // the authority (as {Instance}/{TenantID}). We want to use the Microsoft Identity Platform v2.0 endpoint
  options.Authority = options.Authority + "/v2.0/";

  // If you want to restrict the users that can sign-in to specific organizations
  // Set the tenant value in the appsettings.json file to 'organizations', and add the
  // issuers you want to accept to options.TokenValidationParameters.ValidIssuers collection.
  // Otherwise validate the issuer
  options.TokenValidationParameters.IssuerValidator = AadIssuerValidator.ForAadInstance(options.Authority).ValidateAadIssuer;

  // Set the nameClaimType to be preferred_username.
  // This change is needed because certain token claims from Azure AD v1.0 endpoint
  // (on which the original .NET core template is based) are different in Azure AD v2.0 endpoint.
  // For more details see [ID Tokens](/active-directory/develop/id-tokens)
  // and [Access Tokens](/active-directory/develop/access-tokens)
  options.TokenValidationParameters.NameClaimType = "preferred_username";
  ...
```

## <a name="aspnet-configuration"></a>ASP.NET 配置

在 ASP.NET 中，应用程序通过 `Web.Config` 文件进行配置

```XML
<?xml version="1.0" encoding="utf-8"?>
<!--
  For more information on how to configure your ASP.NET application, please visit
  https://go.microsoft.com/fwlink/?LinkId=301880
  -->
<configuration>
  <appSettings>
    <add key="webpages:Version" value="3.0.0.0" />
    <add key="webpages:Enabled" value="false" />
    <add key="ClientValidationEnabled" value="true" />
    <add key="UnobtrusiveJavaScriptEnabled" value="true" />
    <add key="ida:ClientId" value="[Enter your client ID, as obtained from the app registration portal]" />
    <add key="ida:ClientSecret" value="[Enter your client secret, as obtained from the app registration portal]" />
    <add key="ida:AADInstance" value="https://login.partner.microsoftonline.cn/{0}{1}" />
    <add key="ida:RedirectUri" value="https://localhost:44326/" />
    <add key="vs:EnableBrowserLink" value="false" />
  </appSettings>
```

与在 ASP.NET Web 应用/Web API 中进行身份验证相关的代码位于 `App_Start/Startup.Auth.cs` 文件中。

```CSharp
 public void ConfigureAuth(IAppBuilder app)
 {
  app.SetDefaultSignInAsAuthenticationType(CookieAuthenticationDefaults.AuthenticationType);

  app.UseCookieAuthentication(new CookieAuthenticationOptions());

  app.UseOpenIdConnectAuthentication(
    new OpenIdConnectAuthenticationOptions
    {
     // The `Authority` represents the v2.0 endpoint - https://login.partner.microsoftonline.cn/common/v2.0
     // The `Scope` describes the initial permissions that your app will need.
     //  See https://docs.azure.cn/zh-cn/active-directory/develop/v2-permissions-and-consent
     ClientId = clientId,
     Authority = String.Format(CultureInfo.InvariantCulture, aadInstance, "common", "/v2.0"),
     RedirectUri = redirectUri,
     Scope = "openid profile",
     PostLogoutRedirectUri = redirectUri,
    });
 }
```

## <a name="next-steps"></a>后续步骤

> [!div class="nextstepaction"]
> [登录和注销](scenario-web-app-sign-user-sign-in.md)

