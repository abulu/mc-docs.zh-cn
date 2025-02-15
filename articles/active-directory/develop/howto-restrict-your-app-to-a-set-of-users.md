---
title: 如何将 Azure Active Directory 注册的应用限制为供一组用户使用
description: 了解如何将在 Azure AD 中注册的应用限制为仅供所选的一组用户访问。
services: active-directory
documentationcenter: ''
author: kalyankrishna1
manager: CelesteDG
editor: ''
ms.service: active-directory
ms.subservice: develop
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: conceptual
origin.date: 09/24/2018
ms.date: 06/24/2019
ms.author: v-junlch
ms.reviewer: ''
ms.custom: aaddev
ms.collection: M365-identity-device-management
ms.openlocfilehash: 49f0d634ece7d7a1f856c044cd3497eb0d4c0b77
ms.sourcegitcommit: 5f85d6fe825db38579684ee1b621d19b22eeff57
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 07/05/2019
ms.locfileid: "67568475"
---
# <a name="how-to-restrict-your-app-to-a-set-of-users"></a>如何：将应用限制为一组用户

默认情况下，在 Azure Active Directory (Azure AD) 租户中注册的应用程序可供租户的所有已成功进行身份验证的用户使用。

类似地，在使用[多租户](howto-convert-app-to-be-multi-tenant.md)应用时，如果此应用在 Azure AD 租户中预配，则该租户中的所有用户在相应的租户中成功进行身份验证以后，都能够访问此应用程序。

租户管理员和开发人员通常会要求一个应用只能供特定的一组用户使用。 开发人员可以使用基于角色的访问控制 (RBAC) 之类的常用授权模式来完成此操作，但这种方法要求开发人员完成大量的工作。

租户管理员和开发人员可以通过 Azure AD 将应用限制为仅供租户中特定的一组用户或安全组使用。

## <a name="supported-app-configurations"></a>支持的应用配置

将应用限制为仅供租户中特定的一组用户或安全组使用这一选项适用于以下类型的应用程序：

- 直接在 Azure AD 应用程序平台上生成且使用 OAuth 2.0/OpenID Connect 身份验证的应用程序（前提是用户或管理员已认可该应用程序）。

     > [!NOTE]
     > 此功能仅适用于 Web 应用/Web API 和企业应用程序。 注册为[本机](quickstart-v1-integrate-apps-with-azure-ad.md)的应用不能只限租户中的一组用户或安全组使用。

## <a name="update-the-app-to-enable-user-assignment"></a>更新应用，允许用户分配

1. 转到 [**Azure 门户**](https://portal.azure.cn/)，以“全局管理员”身份登录。 
1. 在顶栏中选择登录的帐户。 
1. 在“目录”下  ，选择要在其中注册应用的 Azure AD 租户。
1. 在左侧导航栏中，选择“Azure Active Directory”。  如果 Azure Active Directory 在导航窗格中不可用，则请执行以下步骤：

    1. 在左侧的主导航菜单顶部选择“所有服务”。 
    1. 在筛选器搜索框中键入“Azure Active Directory”  ，然后从结果中选择“Azure Active Directory”  项。

1. 使用 **Azure Active Directory** 左侧导航菜单，在“Azure Active Directory”窗格中选择“企业应用程序”   。
1. 选择“所有应用程序”  ，查看所有应用程序的列表。

     如果看不到希望其显示在这里的应用程序，请使用“所有应用程序”列表顶部的各种筛选器来限制此列表，或者在列表中向下滚动，以便找到应用程序。 

1. 从列表中选择要向其分配用户或安全组的应用程序。
1. 在应用程序的“概览”页中，从应用程序的左侧导航菜单中选择“属性”   。
1. 找到设置“需要进行用户分配?”，将其设置为“是”。   将此选项设置为“是”时，必须先将用户分配到此应用程序，然后用户才能访问它。 
1. 选择“保存”  以保存此配置更改。

## <a name="assign-users-and-groups-to-the-app"></a>将用户和组分配到应用

将应用配置为允许用户分配以后，即可继续操作，将用户和组分配到应用。

1. 在应用程序的左侧导航菜单中选择“用户和组”窗格  。
1. 在“用户和组”列表顶部选择“添加用户”按钮，以便打开“添加分配”窗格。   
1. 在“添加分配”  窗格中，选择“用户”  选择器。 

     将会显示用户和安全组的列表和一个文本框，后者用于搜索和查找特定用户或组。 此屏幕允许一次选择多个用户和组。

1. 选择好用户和组以后，按底部的“选择”按钮即可转到下一部分。 
1. 按底部的“分配”按钮即可完成将用户和组分配到应用的操作。  
1. 确认已添加的用户和组显示在更新的“用户和组”列表中。 


<!-- Update_Description: update metedata properties -->