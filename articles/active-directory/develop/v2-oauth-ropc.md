---
title: 使用 Microsoft 标识平台通过资源所有者密码凭据 (ROPC) 授予让用户登录 | Azure
description: 支持使用资源所有者密码凭据授予的无浏览器身份验证流。
services: active-directory
documentationcenter: ''
author: rwike77
manager: CelesteDG
editor: ''
ms.service: active-directory
ms.subservice: develop
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: conceptual
origin.date: 04/20/2019
ms.date: 07/01/2019
ms.author: v-junlch
ms.reviewer: hirsin
ms.custom: aaddev
ms.collection: M365-identity-device-management
ms.openlocfilehash: e0b84656b4f41f95c2355788156f94c67bf93551
ms.sourcegitcommit: 5f85d6fe825db38579684ee1b621d19b22eeff57
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 07/05/2019
ms.locfileid: "67568678"
---
# <a name="microsoft-identity-platform-and-the-oauth-20-resource-owner-password-credential"></a>Microsoft 标识平台和 OAuth 2.0 资源所有者密码凭据

Microsoft 标识平台支持[资源所有者密码凭据 (ROPC) 授予](https://tools.ietf.org/html/rfc6749#section-4.3)，后者允许应用程序通过直接处理用户的密码来登录用户。 ROPC 流需要的信任和用户公开程度很高，只有不能使用其他更安全的流时，才应使用此流。

> [!IMPORTANT]
>
> * Microsoft 标识平台终结点仅支持将 ROPC 用于 Azure AD 租户。 这意味着，必须使用特定于租户的终结点 (`https://login.partner.microsoftonline.cn/{TenantId_or_Name}`) 或 `organizations` 终结点。
> * 没有密码的帐户不能通过 ROPC 登录。 对于这种情况，建议改用适合应用的其他流。
> * 如果用户需使用多重身份验证 (MFA) 来登录应用程序，则系统会改为阻止用户。

## <a name="protocol-diagram"></a>协议图

下图显示了 ROPC 流。

![ROPC 流](./media/v2-oauth2-ropc/v2-oauth-ropc.svg)

## <a name="authorization-request"></a>授权请求

ROPC 流是单一请求&mdash;它将客户端标识和用户的凭据发送到 IDP，然后接收返回的令牌。 在这样做之前，客户端必须请求用户的电子邮件地址 (UPN) 和密码。 在成功进行请求之后，客户端应立即以安全方式释放内存中的用户凭据， 而不得保存这些凭据。

> [!TIP]
> 尝试在 Postman 中执行此请求！
> [![在 Postman 中运行](./media/v2-oauth2-auth-code-flow/runInPostman.png)](https://app.getpostman.com/run-collection/f77994d794bab767596d)


```
// Line breaks and spaces are for legibility only.

POST {tenant}/oauth2/v2.0/token
Host: login.partner.microsoftonline.cn
Content-Type: application/x-www-form-urlencoded

client_id=6731de76-14a6-49ae-97bc-6eba6914391e
&scope=user.read%20openid%20profile%20offline_access
&username=MyUsername@myTenant.com
&password=SuperS3cret
&grant_type=password
```

| 参数 | 条件 | 说明 |
| --- | --- | --- |
| `tenant` | 必须 | 一个目录租户，用户需登录到其中。 此参数可采用 GUID 或友好名称格式。 此参数不能设置为 `common` 或 `consumers`，但可以设置为 `organizations`。 |
| `grant_type` | 必须 | 必须设置为 `password`。 |
| `username` | 必须 | 用户的电子邮件地址。 |
| `password` | 必须 | 用户的密码。 |
| `scope` | 建议 | 以空格分隔的[范围](v2-permissions-and-consent.md)或权限的列表，这是应用需要的。 在交互式流中，管理员或用户必须提前同意这些范围。 |

### <a name="successful-authentication-response"></a>成功的身份验证响应

以下示例显示了一个成功的令牌响应：

```json
{
    "token_type": "Bearer",
    "scope": "User.Read profile openid email",
    "expires_in": 3599,
    "access_token": "eyJ0eXAiOiJKV1QiLCJhbGciOiJSUzI1NiIsIng1dCI6Ik5HVEZ2ZEstZnl0aEV1Q...",
    "refresh_token": "AwABAAAAvPM1KaPlrEqdFSBzjqfTGAMxZGUTdM0t4B4...",
    "id_token": "eyJ0eXAiOiJKV1QiLCJhbGciOiJub25lIn0.eyJhdWQiOiIyZDRkMTFhMi1mODE0LTQ2YTctOD..."
}
```

| 参数 | 格式 | 说明 |
| --------- | ------ | ----------- |
| `token_type` | String | 始终设置为 `Bearer`。 |
| `scope` | 空格分隔的字符串 | 如果返回了访问令牌，则此参数会列出该访问令牌的有效范围。 |
| `expires_in`| int | 包含的访问令牌的有效时间，以秒为单位。 |
| `access_token`| 不透明字符串 | 针对请求的[范围](v2-permissions-and-consent.md)颁发。 |
| `id_token` | JWT | 如果原始 `scope` 参数包含 `openid` 范围，则颁发。 |
| `refresh_token` | 不透明字符串 | 如果原始 `scope` 参数包含 `offline_access`，则颁发。 |

可以运行 [OAuth 代码流文档](v2-oauth2-auth-code-flow.md#refresh-the-access-token)中描述的同一个流，使用刷新令牌来获取新的访问令牌和刷新令牌。

### <a name="error-response"></a>错误响应

如果用户未提供正确的用户名或密码，或者客户端未收到请求的许可，则身份验证会失败。

| 错误 | 说明 | 客户端操作 |
|------ | ----------- | -------------|
| `invalid_grant` | 身份验证失败 | 凭据不正确，或者客户端没有所请求范围的许可。 如果没有授予范围，则会返回 `consent_required` 错误。 如果发生这种情况，客户端应通过 Webview 或浏览器向用户发送交互式提示。 |
| `invalid_request` | 请求的构造方式不正确 | 授予类型在 `/common` 或 `/consumers` 身份验证上下文中不受支持。  请改用 `/organizations`。 |
| `invalid_client` | 应用未正确设置 | 如果未在[应用程序清单](reference-app-manifest.md)中将 `allowPublicClient` 属性设置为 true，则可能发生这种情况。 之所以需要 `allowPublicClient` 属性，是因为 ROPC 授予没有重定向 URI。 在设置此属性之前，Azure AD 不能确定应用是公共客户端应用程序还是机密客户端应用程序。 ROPC 只能用于公共客户端应用。 |

## <a name="learn-more"></a>了解详细信息

* 请通过[示例控制台应用程序](https://github.com/azure-samples/active-directory-dotnetcore-console-up-v2)自行试用 ROPC。
* 若要确定是否应使用 v2.0 终结点，请阅读 [Microsoft 标识平台限制](azure-ad-endpoint-comparison.md)。

<!-- Update_Description: update metedata properties -->