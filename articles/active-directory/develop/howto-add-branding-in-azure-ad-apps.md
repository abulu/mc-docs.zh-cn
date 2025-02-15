---
title: 适用于应用程序的品牌准则 | Microsoft Docs
description: 介绍面向开发人员的 Azure Active Directory 资源的综合性指南
services: active-directory
documentationcenter: dev-center-name
author: rwike77
manager: CelesteDG
editor: ''
ms.assetid: 72f4e464-1352-4a49-a18f-c37f58e7d5c4
ms.service: active-directory
ms.subservice: develop
ms.devlang: na
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.workload: identity
origin.date: 09/24/2018
ms.date: 06/24/2019
ms.author: v-junlch
ms.reviewer: arielgo
ms.custom: aaddev, signin_art
ms.collection: M365-identity-device-management
ms.openlocfilehash: c4755c84f3656a73fe8fcb8bdfc54f31c04b7c92
ms.sourcegitcommit: 5f85d6fe825db38579684ee1b621d19b22eeff57
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 07/05/2019
ms.locfileid: "67568495"
---
# <a name="branding-guidelines-for-applications"></a>适用于应用程序的品牌准则

使用 Azure Active Directory (Azure AD) 开发应用程序时，如果客户想要使用工作或学校帐户（在 Azure AD 中托管）进行注册和登录到应用程序，你需要指导客户进行相关操作。

在本文中，你将：

- 了解 Microsoft 管理的两种用户帐户以及如何在应用程序中引用 Azure AD 帐户
- 了解添加要在应用中使用的 Microsoft 徽标所需执行的操作
- 下载官方的**登录**或**使用 Microsoft 登录**图像以在应用中使用
- 了解品牌和导航的注意事项


## <a name="referring-to-azure-ad-accounts-in-your-application"></a>在应用程序中引用 Azure AD 帐户

Microsoft 不会向最终用户显示 Azure 或 Active Directory 品牌名称，你也应该如此。

- 在用户登录后，应尽量使用组织的名称和徽标。 这比使用“你的组织”等通用词语要好。
- 如果用户未登录，将他们的帐户称为“工作或学校帐户”，并使用 Microsoft 徽标来表明这些帐户由 Microsoft 管理。 请勿使用“企业帐户”、“业务帐户”或“公司帐户”等词语，这会给用户造成混淆。

## <a name="user-account-pictogram"></a>用户帐户象形图

在先前版本的准则中，我们建议使用“蓝色徽章”象形图。 根据用户和开发人员的反馈，我们现在建议改用 Microsoft 徽标。 Microsoft 徽标帮助用户了解，他们可以重用其用于 Office 365 或其他 Microsoft 业务服务的帐户来登录应用。

## <a name="signing-up-and-signing-in-with-azure-ad"></a>使用 Azure AD 注册和登录

应用程序可以为注册和登录提供不同的路径，以下部分提供了这两种应用场景的可视指南。

**如果应用支持最终用户注册（例如免费试用或免费增值模式）** ：可以显示“登录”  按钮，让用户使用其工作帐户访问应用。 当用户首次访问你的应用程序时，Azure AD 会显示许可提示。

**如果应用需要只有管理员才能授予的权限，或者需要组织许可：** 请将管理员请求与用户登录区别开来。 “获取此应用”按钮  会将管理员重定向到登录页，然后要求他们代表其组织中的用户授权同意，这还有一个额外的好处，那就是在你的应用中抑制最终用户许可提示。

## <a name="visual-guidance-for-app-acquisition"></a>有关获取应用程序的可视指南

“获取应用程序”链接必须将用户重定向到 Azure AD 的访问权限授予（授权）页，以方便组织的管理员对应用程序进行授权，使其有权访问 Microsoft 托管的组织数据。 有关如何请求访问权限的详细信息，请参阅[将应用程序与 Azure Active Directory 集成](quickstart-v1-integrate-apps-with-azure-ad.md) 一文。

管理员许可应用后，可以选择将应用添加到其用户的 Office 365 应用启动器体验（可从 waffle 访问）。 如果想要广告此功能，可以使用类似于“将此应用程序添加到组织”词语，并显示类似于以下示例的按钮：

![应用程序类型和方案](./media/howto-add-branding-in-azure-ad-apps/add-to-my-org.png)

但是，我们建议编写说明性的文本而不要依赖于按钮。 例如：

> *如果已使用了 Office 365 或 Microsoft 的其他业务服务，则可以授予 <your_app_name> 对组织数据的访问权限。这样，用户便可以使用其现有工作帐户访问 <your_app_name>。*

若要下载官方 Microsoft 徽标以在应用中使用，请右键单击要使用的徽标，然后将其保存到计算机。

| 资产                                | PNG 格式 | SVG 格式 |
| ------------------------------------ | ---------- | ---------- |
| Microsoft 徽标  | ![Microsoft 徽标 PNG](./media/howto-add-branding-in-azure-ad-apps/ms-symbollockup_mssymbol_19.png) | ![Microsoft 徽标 SVG](./media/howto-add-branding-in-azure-ad-apps/ms-symbollockup_mssymbol_19.svg) |

## <a name="visual-guidance-for-sign-in"></a>有关登录的可视指南

应用应显示登录按钮，用于将用户重定向到与用来与 Azure AD 集成的协议对应的登录终结点。 以下部分详细描述了该按钮的外观。

### <a name="pictogram-and-sign-in-with-microsoft"></a>图标和“通过 Microsoft 登录”

Microsoft 徽标和“通过 Microsoft 登录”词语的关联可唯一地将 Azure AD 与应用支持的其他标识提供者区别开来。 如果没有足够的空间来容纳“通过 Microsoft 登录”，则可以将其缩短为“登录”。 可以对该按钮使用浅色或深色方案。

下图显示在使用应用资产时，Microsoft 建议的红线。 红线适用于“Microsoft 登录”或更短的“登录”版本。

![Microsoft 登录红线](./media/howto-add-branding-in-azure-ad-apps/sign-in-with-microsoft-redlines.png)

若要下载官方图像以在应用中使用，请右键单击要使用的图像，然后将其保存到计算机。

| 资产                                | PNG 格式 | SVG 格式 |
| ------------------------------------ | ---------- | ---------- |
| Microsoft 登录（深色主题）  | ![登录按钮深色主题 PNG](./media/howto-add-branding-in-azure-ad-apps/ms-symbollockup_signin_dark.png) | ![Microsoft 登录按钮深色主题 SVG](./media/howto-add-branding-in-azure-ad-apps/ms-symbollockup_signin_dark.svg) |
| Microsoft 登录（浅色主题） | ![登录按钮浅色主题 PNG](./media/howto-add-branding-in-azure-ad-apps/ms-symbollockup_signin_light.png) | ![Microsoft 登录按钮浅色主题 SVG](./media/howto-add-branding-in-azure-ad-apps/ms-symbollockup_signin_light.svg) |
| 登录（深色主题）                 | ![登录短按钮深色主题 PNG](./media/howto-add-branding-in-azure-ad-apps/ms-symbollockup_signin_dark_short.png) | ![登录短按钮深色主题 SVG](./media/howto-add-branding-in-azure-ad-apps/ms-symbollockup_signin_dark_short.svg) |
| 登录（浅色主题）                | ![登录短按钮浅色主题 PNG](./media/howto-add-branding-in-azure-ad-apps/ms-symbollockup_signin_light_short.png) | ![登录短按钮浅色主题 SVG](./media/howto-add-branding-in-azure-ad-apps/ms-symbollockup_signin_light_short.svg) |


## <a name="branding-dos-and-donts"></a>品牌注意事项

**务必**将“工作或学校帐户”与“通过 Microsoft 登录”按钮结合使用来提供附加说明，以便帮助最终用户识别他们是否可以使用该应用。 **请勿** 使用“企业帐户”、“业务帐户”或“公司帐户”等其他词语。

**请勿**使用“Office 365 ID”或“Azure ID”。 Office 365 也是 Microsoft 的消费型产品名称，它不使用 Azure AD 进行身份验证。

**不要** 更改 Microsoft 徽标。

**请勿**向最终用户显示 Azure 或 Active Directory 品牌。 但是，可以对开发人员、IT 专业人员和管理员使用这些词语。

## <a name="navigation-dos-and-donts"></a>导航注意事项

**要** 提供让用户注销以及切换到其他用户帐户的方法。 

<!-- Update_Description: update metedata properties -->