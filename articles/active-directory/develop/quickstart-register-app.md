---
title: 将应用注册到 Microsoft 标识平台 | Microsoft 标识平台
description: 了解如何添加应用程序并将其注册到 Microsoft 标识平台。
services: active-directory
documentationcenter: ''
author: rwike77
manager: CelesteDG
editor: ''
ms.service: active-directory
ms.subservice: develop
ms.devlang: na
ms.topic: quickstart
ms.tgt_pltfrm: na
ms.workload: identity
origin.date: 05/09/2019
ms.date: 06/25/2019
ms.author: v-junlch
ms.custom: aaddev
ms.reviewer: aragra, lenalepa, sureshja
ms.collection: M365-identity-device-management
ms.openlocfilehash: 466d7d421bf79cead0abc97f146cf08d4f2185d4
ms.sourcegitcommit: 5f85d6fe825db38579684ee1b621d19b22eeff57
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 07/05/2019
ms.locfileid: "67568740"
---
# <a name="quickstart-register-an-application-with-the-microsoft-identity-platform"></a>快速入门：将应用程序注册到 Microsoft 标识平台

企业开发人员和软件即服务 (SaaS) 提供商可以开发能够与 Microsoft 标识平台集成的商业云服务或业务线应用程序，以针对其服务提供安全的登录和授权。

本快速入门介绍了如何在 Azure 门户中使用**应用注册**体验来添加和注册应用程序，使应用能够与 Microsoft 标识平台集成。 若要详细了解新应用注册体验中的新功能和改进，请参阅[此博客文章](https://developer.microsoft.com/graph/blogs/new-app-registration/)。

## <a name="register-a-new-application-using-the-azure-portal"></a>使用 Azure 门户注册新应用程序

1. 使用工作或学校帐户登录到 [Azure 门户](https://portal.azure.cn)。
1. 如果你的帐户有权访问多个租户，请在右上角选择该帐户，并将门户会话设置为所需的 Azure AD 租户。
1. 在左侧导航窗格中，选择“Azure Active Directory”服务  ，然后选择“应用注册”>“新建注册”。 
1. 出现“注册应用程序”页后，请输入应用程序的注册信息： 

   - **名称**：输入一个会显示给应用用户的有意义的应用程序名称。
   - **支持的帐户类型** - 选择希望应用程序支持的具体帐户。

       | 支持的帐户类型 | 说明 |
       |-------------------------|-------------|
       | **仅此组织目录中的帐户** | 若要生成业务线 (LOB) 应用程序，请选择此选项。 如果不在目录中注册应用程序，则此选项不可用。<br><br>此选项映射到仅限 Azure AD 的单租户。<br><br>这是默认选项，除非你是在目录外部注册应用。 如果在目录外部注册应用，则默认设置为 Azure AD 多租户。 |
       | **任何组织目录中的帐户** | 若要以所有企业和教育客户为目标，请选择此选项。<br><br>此选项映射到仅限 Azure AD 的多租户。<br><br>如果已将应用注册为仅限 Azure AD 的单租户，则可通过“身份验证”边栏选项卡将其更新为 Azure AD 多租户，以及从多租户更新为单租户。  |

   - **重定向 URI (可选)** - 选择要生成的应用的类型：“Web”  或“公共客户端(移动和桌面)”  ，然后输入应用程序的重定向 URI (或回复 URL)。
       - 对于 Web 应用程序，请提供应用的基 URL。 例如，`http://localhost:31544` 可以是本地计算机上运行的 Web 应用的 URL。 用户将使用此 URL 登录到 Web 客户端应用程序。
       - 对于公共客户端应用程序，请提供 Azure AD 返回令牌响应时所用的 URI。 输入特定于应用程序的值，例如 `myapp://auth`。

     若要查看 Web 应用程序或本机应用程序的特定示例，请参阅[快速入门](/active-directory/develop/#quickstarts)。

1. 完成后，选择“注册”  。

    [![在 Azure 门户中注册新应用程序](./media/quickstart-add-azure-ad-app-preview/new-app-registration-expanded.png)](./media/quickstart-add-azure-ad-app-preview/new-app-registration-expanded.png#lightbox)

Azure AD 会将唯一的应用程序（客户端）ID 分配给应用，同时你会转到应用程序的“概览”页。  若要向应用程序添加其他功能，可以选择其他配置选项，包括品牌、证书和机密、API 权限，等等。

[![新注册应用的概览页](./media/quickstart-add-azure-ad-app-preview/new-app-overview-page-expanded.png)](./media/quickstart-add-azure-ad-app-preview/new-app-overview-page-expanded.png#lightbox)

## <a name="next-steps"></a>后续步骤

- 了解[权限和许可](v2-permissions-and-consent.md)。
- 若要在应用程序注册中启用其他配置功能（如凭据和权限），并使用户能够从其他租户登录，请参阅以下快速入门：
    - [将客户端应用程序配置为访问 Web API](quickstart-configure-app-access-web-apis.md)
    - [将应用程序配置为公开 Web API](quickstart-configure-app-expose-web-apis.md)
    - [修改应用程序支持的帐户](quickstart-modify-supported-accounts.md)
- 选择一个[快速入门](/active-directory/develop/#quickstarts)，了解如何快速生成应用并添加功能，例如获取令牌、刷新令牌、进行用户登录、显示某些用户信息，等等。
- 如需深入了解表示已注册应用程序和它们之间的关系的两个 Azure AD 对象，请参阅[应用程序对象和服务主体对象](app-objects-and-service-principals.md)。
- 如需深入了解开发应用时应使用的品牌准则，请参阅[应用程序的品牌准则](howto-add-branding-in-azure-ad-apps.md)。

<!-- Update_Description: wording update -->