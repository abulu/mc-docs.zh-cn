---
title: 设置 Azure Active Directory 组织的登录 - Azure Active Directory B2C | Microsoft Docs
description: 在 Azure Active Directory B2C 中设置登录特定 Azure Active Directory 组织。
services: active-directory-b2c
author: davidmu1
manager: celestedg
ms.service: active-directory
ms.workload: identity
ms.topic: conceptual
origin.date: 05/14/2018
ms.date: 06/05/2019
ms.author: v-junlch
ms.subservice: B2C
ms.openlocfilehash: a134be2398ab692c36bf2555746a80695ec1ff9f
ms.sourcegitcommit: 623e8f0d52c42d236ad2a0136d5aebd6528dbee3
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 06/19/2019
ms.locfileid: "67235942"
---
# <a name="set-up-sign-in-for-a-specific-azure-active-directory-organization-in-azure-active-directory-b2c"></a>在 Azure Active Directory B2C 中设置登录特定 Azure Active Directory 组织

>[!NOTE]
> 此功能目前以公共预览版提供。 请勿在生产环境中使用该功能。

若要将 Azure Active Directory (Azure AD) 用作 Azure AD B2C 中的[标识提供者](active-directory-b2c-reference-oauth-code.md)，需要创建一个表示它的应用程序。 本文介绍如何使用 Azure AD B2C 中的用户流为特定 Azure AD 组织中的用户启用登录。

## <a name="create-an-azure-ad-app"></a>创建 Azure AD 应用

若要让用户从特定的 Azure AD 组织登录，需要在与 Azure AD B2C 租户不同的组织 Azure AD 租户中注册一个应用程序。

1. 登录到 [Azure 门户](https://portal.azure.cn)。
2. 请确保使用的是包含 Azure AD 租户的目录。 选择顶部菜单中的“目录和订阅筛选器”，然后选择包含 Azure AD 租户的目录  。 此租户与 Azure AD B2C 租户不同。
3. 选择 Azure 门户左上角的“所有服务”，然后搜索并选择“应用注册”   。
4. 选择“新注册”。 
5. 输入应用程序的名称。 例如，`Azure AD B2C App`。
6. 对于此应用程序，接受选择“仅此组织目录中的帐户”  。
7. 对于“重定向 URI”  ，接受值 **Web**，并以全小写字母输入以下 URL，其中 `your-B2C-tenant-name` 将替换为 Azure AD B2C 租户的名称。 例如，`https://fabrikam.b2clogin.cn/fabrikam.partner.onmschina.cn/oauth2/authresp`：

    ```
    https://your--B2C-tenant-name.b2clogin.cn/your-B2C-tenant-name.partner.onmschina.cn/oauth2/authresp
    ```

    现在，所有 URL 都应使用 [b2clogin.cn](b2clogin.md)。

8. 单击“注册”  。 复制“应用程序(客户端) ID”供以后使用  。
9. 在应用程序菜单中依次选择“证书和机密”、“新建客户端机密”。  
10. 输入客户端密码的名称。 例如，`Azure AD B2C App Secret`。
11. 选择到期期限。 对于此应用程序，接受选项“1 年内”  。
12. 选择“添加”  ，然后复制显示的新客户端密码的值，以便以后使用。

## <a name="configure-azure-ad-as-an-identity-provider"></a>将 Azure AD 配置为标识提供者

1. 请确保使用的是包含 Azure AD B2C 租户的目录。 选择顶部菜单中的“目录和订阅筛选器”，然后选择包含 Azure AD B2C 租户的目录  。
2. 选择 Azure 门户左上角的“所有服务”，然后搜索并选择“Azure AD B2C”   。
3. 选择“标识提供者”  ，然后选择“添加”  。
4. 输入“名称”  。 例如，输入 `Contoso Azure AD`。
5. 选择“标识提供者类型”，选择“Open ID 连接(预览版)”并单击“确定”    。
6. 选择“设置此标识提供者” 
7. 对于“元数据 URL”  ，输入以下 URL，并将 `your-AD-tenant-domain` 替换为 Azure AD 租户的域名。 例如 `https://login.partner.microsoftonline.cn/contoso.partner.onmschina.cn/.well-known/openid-configuration`：

    ```
    https://login.partner.microsoftonline.cn/your-AD-tenant-domain/.well-known/openid-configuration
    ```

8. 对于“客户端 ID”  ，输入先前记录的应用程序 ID；对于“客户端密码”  ，输入先前记录的客户端密码。
9. （可选）输入  Domain_hint 的值。 例如，`ContosoAD`。 在请求中使用 domain_hint 引用此标识提供者时会用到该值  。 
10. 单击 **“确定”** 。
11. 选择“映射此标识提供者的声明”  并设置以下声明：
    
    - 在“用户 ID”中输入 `oid`  。
    - 在“显示名称”中输入 `name`  。
    - 在“名字”中输入 `given_name`  。
    - 在“姓氏”中输入 `family_name`  。
    - 在“电子邮件”中输入 `unique_name`  。

12. 单击“确定”，并单击“创建”保存配置。  

<!-- Update_Description: wording update -->