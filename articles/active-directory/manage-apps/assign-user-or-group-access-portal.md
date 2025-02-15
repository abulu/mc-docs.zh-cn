---
title: 在 Azure Active Directory 中向企业应用分配用户或组 | Microsoft Docs
description: 如何选择企业应用，在 Azure Active Directory 中向其分配用户或组
services: active-directory
author: msmimart
manager: CelesteDG
ms.service: active-directory
ms.subservice: app-mgmt
ms.workload: identity
ms.topic: conceptual
origin.date: 04/11/2019
ms.date: 07/04/2019
ms.author: v-junlch
ms.reviewer: luleon
ms.collection: M365-identity-device-management
ms.openlocfilehash: c854e8e013e5c40c585bb26af7e7023156833c3f
ms.sourcegitcommit: 5f85d6fe825db38579684ee1b621d19b22eeff57
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 07/05/2019
ms.locfileid: "67568605"
---
# <a name="assign-a-user-to-an-enterprise-app-in-azure-active-directory"></a>在 Azure Active Directory 中向企业应用分配用户
若要将用户分配到企业应用，必须具有适当的权限才能管理企业应用，并且必须是目录的全局管理员。

> [!NOTE]
> 有关本文中讨论的功能的许可要求，请参阅 [Azure Active Directory 定价页](https://www.azure.cn/pricing/details/active-directory)。

> [!NOTE]
> 对于 Microsoft 应用程序（例如 Office 365 应用），请使用 PowerShell 将用户分配到企业应用。


## <a name="assign-a-user-to-an-app---portal"></a>将用户分配到应用 - 门户
1. 使用目录全局管理员的帐户登录到 [Azure 门户](https://portal.azure.cn) 。
1. 选择“所有服务”  ，在文本框中输入 Azure Active Directory，并选择“Enter”  。
1. 选择“企业应用程序”。 
1. 在“企业应用程序 - 所有应用程序”  窗格上，你会看到你可以管理的应用的列表。 选择一个应用。
1. 在 ***appname*** 窗格（即标题中包含所选应用的名称的窗格）中，选择“用户和组”  。
1. 在“appname - 用户和组”窗格中，选择“添加用户”。  
1. 在“添加分配”窗格中选择“用户”   。

    ![将用户分配到应用](./media/assign-user-or-group-access-portal/assign-users.png)
1. 在“用户”窗格的列表中选择一个或多个用户，然后选择窗格底部的“选择”按钮。  
1. 在“添加分配”窗格中选择“角色”   。 然后，在“选择角色”窗格中选择一个需要应用到所选用户的角色，然后选择窗格底部的“确定”。  
1. 在“添加分配”窗格中，选择窗格底部的“分配”按钮。   已分配用户的权限将是该企业应用的选定角色所定义的权限。

## <a name="allow-all-users-to-access-an-app---portal"></a>允许所有用户访问某个应用 - 门户
允许所有用户访问某个应用程序：

1. 使用目录全局管理员的帐户登录到 [Azure 门户](https://portal.azure.cn) 。
1. 选择“所有服务”  ，在文本框中输入 Azure Active Directory，并选择“Enter”  。
1. 选择“企业应用程序”。 
1. 在“企业应用程序”  窗格中，选择“所有应用程序”  。 随后会列出你可以管理的应用。
1. 在“企业应用程序 - 所有应用程序”  窗格中，选择一个应用。
1. 在“appname”窗格上，选择“属性”  。
1. 在“appname - 属性”窗格上，将“需要进行用户分配?”设置设置为“否”    。 

## <a name="assign-a-user-to-an-app---powershell"></a>将用户分配到应用 - PowerShell

1. 以提升的权限打开 Windows PowerShell 命令提示符。

    >[!NOTE] 
    > 需要安装 AzureAD 模块（使用命令 `Install-Module -Name AzureAD`）。 出现安装 NuGet 模块或新的 Azure Active Directory V2 PowerShell 模块的提示时，请键入 Y，然后按 ENTER。

1. 运行 `Connect-AzureAD -AzureEnvironmentName AzureChinaCloud` 并使用全局管理员用户帐户登录。
1. 使用以下脚本将用户和角色分配到应用程序：

    ```powershell
    # Assign the values to the variables
    $username = "<You user's UPN>"
    $app_name = "<Your App's display name>"
    $app_role_name = "<App role display name>"
    
    # Get the user to assign, and the service principal for the app to assign to
    $user = Get-AzureADUser -ObjectId "$username"
    $sp = Get-AzureADServicePrincipal -Filter "displayName eq '$app_name'"
    $appRole = $sp.AppRoles | Where-Object { $_.DisplayName -eq $app_role_name }
    
    # Assign the user to the app role
    New-AzureADUserAppRoleAssignment -ObjectId $user.ObjectId -PrincipalId $user.ObjectId -ResourceId $sp.ObjectId -Id $appRole.Id
    ```     

有关如何将用户分配到应用程序角色的详细信息，请访问 [New-AzureADUserAppRoleAssignment](https://docs.microsoft.com/powershell/module/azuread/new-azureaduserapproleassignment?view=azureadps-2.0) 的文档

### <a name="example"></a>示例

此示例使用 PowerShell 将用户 Britta Simon 分配到 [Microsoft Workplace Analytics](https://products.office.com/business/workplace-analytics) 应用程序。

1. 在 PowerShell 中，将相应的值分配到变量 $username、$app_name 和 $app_role_name。 

    ```powershell
    # Assign the values to the variables
    $username = "britta.simon@contoso.com"
    $app_name = "Workplace Analytics"
    ```

1. 在此示例中，我们并不确切地知道要将哪个应用程序角色名称分配给 Britta Simon。 运行以下命令，使用用户 UPN 和服务主体显示名称获取用户 ($user) 和服务主体 ($sp)。

    ```powershell
    # Get the user to assign, and the service principal for the app to assign to
    $user = Get-AzureADUser -ObjectId "$username"
    $sp = Get-AzureADServicePrincipal -Filter "displayName eq '$app_name'"
    ```
        
1. 运行命令 `$sp.AppRoles`，显示可用于 Workplace Analytics 应用程序的角色。 在此示例中，我们要为 Britta Simon 分配“分析员”（访问权限受限）角色。
    
    ![Workplace Analytics 角色](./media/assign-user-or-group-access-portal/workplace-analytics-role.png)

1. 将角色名称分配到 `$app_role_name` 变量。
        
    ```powershell
    # Assign the values to the variables
    $app_role_name = "Analyst (Limited access)"
    $appRole = $sp.AppRoles | Where-Object { $_.DisplayName -eq $app_role_name }
    ```

1. 运行以下命令，将用户分配到应用角色：

    ```powershell
    # Assign the user to the app role
    New-AzureADUserAppRoleAssignment -ObjectId $user.ObjectId -PrincipalId $user.ObjectId -ResourceId $sp.ObjectId -Id $appRole.Id
    ```

## <a name="next-steps"></a>后续步骤
* [查看所有组](../fundamentals/active-directory-groups-view-azure-portal.md)
* [删除企业应用的用户分配](remove-user-or-group-access-portal.md)
* [Disable user sign-ins for an enterprise app](disable-user-sign-in-portal.md)
* [Change the name or logo of an enterprise app](change-name-or-logo-portal.md)

<!-- Update_Description: update metedata properties -->