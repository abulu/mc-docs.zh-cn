---
title: 按管理员任务委托最小特权角色 - Azure Active Directory | Microsoft Docs
description: 在 Azure Active Directory 中为标识任务委托角色
services: active-directory
documentationcenter: ''
author: curtand
manager: mtillman
editor: ''
ms.service: active-directory
ms.workload: identity
ms.subservice: users-groups-roles
ms.topic: article
origin.date: 05/31/2019
ms.date: 07/04/2019
ms.author: v-junlch
ms.reviewer: vincesm
ms.custom: it-pro
ms.collection: M365-identity-device-management
ms.openlocfilehash: a9b9805f74eb9094d55e65328c828aa22476c903
ms.sourcegitcommit: 5f85d6fe825db38579684ee1b621d19b22eeff57
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 07/05/2019
ms.locfileid: "67568747"
---
# <a name="administrator-roles-by-admin-task-in-azure-active-directory"></a>在 Azure Active Directory 中按管理员任务委托管理员角色

本文介绍了通过在 Azure Active Directory (Azure AD) 中分配最小特权角色来限制用户管理员权限所需的信息。 你将能查找按功能区域整理的管理员任务、执行每项任务所需的最小特权角色，以及可以执行任务的其他非全局管理员角色。

## <a name="company-properties"></a>公司属性

任务 | 最小特权角色 | 其他角色
---- | --------------------- | ----------------
配置公司属性 | 全局管理员角色 | 

## <a name="connect"></a>连接

任务 | 最小特权角色 | 其他角色
---- | --------------------- | ----------------
读取所有配置 | 全局管理员角色 | 

## <a name="custom-domain-names"></a>自定义域名

任务 | 最小特权角色 | 其他角色
---- | --------------------- | ----------------
管理域 | 全局管理员角色 | 
读取所有配置 | 目录读者 | 默认用户角色（[请参阅文档](/active-directory/fundamentals/users-default-permissions)）

## <a name="enterprise-applications"></a>企业应用程序

任务 | 最小特权角色 | 其他角色
---- | --------------------- | ----------------
同意任何委托的权限 | 云应用程序管理员 | 应用程序管理员
同意应用程序权限（不包括 Microsoft Graph 或 Azure AD Graph） | 云应用程序管理员 | 应用程序管理员
同意 Microsoft Graph 或 Azure AD Graph 的应用程序权限 | 全局管理员角色 | 
同意应用程序访问自己的数据 | 默认用户角色（[请参阅文档](/active-directory/fundamentals/users-default-permissions)） | 
创建企业应用程序 | 云应用程序管理员 | 应用程序管理员
管理应用程序代理 | 应用程序管理员 | 
管理用户设置 | 全局管理员角色 | 
读取组或应用的访问评审 | 安全读取者 | 安全管理员、用户管理员
读取所有配置 | 默认用户角色（[请参阅文档](/active-directory/fundamentals/users-default-permissions)） | 
更新企业应用程序分配 | 企业应用程序所有者（[请参阅文档](/active-directory/fundamentals/users-default-permissions)） | 云应用程序管理员、应用程序管理员
更新企业应用程序所有者 | 企业应用程序所有者（[请参阅文档](/active-directory/fundamentals/users-default-permissions)） | 云应用程序管理员、应用程序管理员
更新企业应用程序属性 | 企业应用程序所有者（[请参阅文档](/active-directory/fundamentals/users-default-permissions)） | 云应用程序管理员、应用程序管理员
更新企业应用程序预配 | 企业应用程序所有者（[请参阅文档](/active-directory/fundamentals/users-default-permissions)） | 云应用程序管理员、应用程序管理员
更新企业应用程序自助服务 | 企业应用程序所有者（[请参阅文档](/active-directory/fundamentals/users-default-permissions)） | 云应用程序管理员、应用程序管理员
更新单一登录属性 | 企业应用程序所有者（[请参阅文档](/active-directory/fundamentals/users-default-permissions)） | 云应用程序管理员、应用程序管理员

## <a name="groups"></a>组

任务 | 最小特权角色 | 其他角色
---- | --------------------- | ----------------
分配许可证 | 用户管理员 | 
创建组 | 用户管理员 | 
创建、更新或删除组或应用的访问评审 | 用户管理员 | 
管理组到期时间 | 用户管理员 | 
管理组设置 | 全局管理员角色 | 
读取所有配置（隐藏成员身份除外） | 目录读者 | 默认用户角色（[请参阅文档](/active-directory/fundamentals/users-default-permissions)）
读取隐藏成员身份 | 组成员 | 组所有者、密码管理员、Exchange 管理员、SharePoint 管理员、Teams 管理员、用户管理员
读取具有隐藏成员身份的组的成员身份 | 支持管理员 | 用户管理员、Teams 管理员
撤销许可证 | 许可证管理员 | 用户管理员
更新组成员身份 | 组所有者（[请参阅文档](/active-directory/fundamentals/users-default-permissions)） | 用户管理员
更新组所有者 | 组所有者（[请参阅文档](/active-directory/fundamentals/users-default-permissions)） | 用户管理员
更新组属性 | 组所有者（[请参阅文档](/active-directory/fundamentals/users-default-permissions)） | 用户管理员

## <a name="licenses"></a>许可证

任务 | 最小特权角色 | 其他角色
---- | --------------------- | ----------------
分配许可证 | 许可证管理员 | 用户管理员
读取所有配置 | 目录读者 | 默认用户角色（[请参阅文档](/active-directory/fundamentals/users-default-permissions)）
撤销许可证 | 许可证管理员 | 用户管理员
试用或购买订阅 | 计费管理员 | 


## <a name="monitoring---audit-logs"></a>监视 - 审核日志

任务 | 最小特权角色 | 其他角色
---- | --------------------- | ----------------
读取审核日志 | 报告读者 | 安全读取者、安全管理员

## <a name="monitoring---sign-ins"></a>监视 - 登录

任务 | 最小特权角色 | 其他角色
---- | --------------------- | ----------------
读取登录日志 | 报告读者 | 安全读取者、安全管理员

## <a name="multi-factor-authentication"></a>多重身份验证

任务 | 最小特权角色 | 其他角色
---- | --------------------- | ----------------
删除选定用户生成的所有现有应用密码 | 全局管理员角色 | 
禁用 MFA | 全局管理员角色 | 
启用 MFA | 全局管理员角色 | 
管理 MFA 服务设置 | 全局管理员角色 | 
要求选定的用户再次提供联系方法 | 身份验证管理员 | 
在所有记住的设备上还原多重身份验证  | 身份验证管理员 | 

## <a name="organizational-relationships"></a>组织关系

任务 | 最小特权角色 | 其他角色
---- | --------------------- | ----------------
管理标识提供者 | 全局管理员角色 | 
管理设置 | 全局管理员角色 | 
管理使用条款 | 全局管理员角色 | 
读取所有配置 | 全局管理员角色 | 

## <a name="password-reset"></a>密码重置

任务 | 最小特权角色 | 其他角色
---- | --------------------- | ----------------
配置身份验证方法 | 全局管理员角色 |
配置自定义 | 全局管理员角色 |
配置通知 | 全局管理员角色 |
配置本地集成 | 全局管理员角色 |
配置密码重置属性 | 用户管理员 | 全局管理员角色
配置注册 | 全局管理员角色 |
读取所有配置 | 安全管理员 | 用户管理员 |

## <a name="roles-and-administrators"></a>角色和管理员

任务 | 最小特权角色 | 其他角色
---- | --------------------- | ----------------
管理角色分配 | 特权角色管理员 | 
读取 Azure AD 角色的访问评审  | 安全读取者 | 安全管理员、特权角色管理员
读取所有配置 | 默认用户角色（[请参阅文档](/active-directory/fundamentals/users-default-permissions)） | 

## <a name="security---authentication-methods"></a>安全性 - 身份验证方法

任务 | 最小特权角色 | 其他角色
---- | --------------------- | ----------------
配置身份验证方法 | 全局管理员角色 | 
读取所有配置 | 全局管理员角色 | 

## <a name="security---identity-security-score"></a>安全性 - 标识安全分数

任务 | 最小特权角色 | 其他角色 | 
---- | --------------------- | ----------------
读取所有配置 | 安全读取者 | 安全管理员
读取安全分数 | 安全读取者 | 安全管理员
更新事件状态 | 安全管理员 | 

## <a name="users"></a>用户

任务 | 最小特权角色 | 其他角色
---- | --------------------- | ----------------
将用户添加到目录角色 | 特权角色管理员 | 
将用户添加到组 | 用户管理员 | 
分配许可证 | 许可证管理员 | 用户管理员
创建来宾用户 | 来宾邀请者 | 用户管理员
创建用户 | 用户管理员 | 
删除用户 | 用户管理员 | 
使受限管理员的刷新令牌失效（请参阅文档） | 用户管理员 | 
使非管理员的刷新令牌失效（请参阅文档） | 密码管理员 | 用户管理员
使特权管理员的刷新令牌失效（请参阅文档） | 全局管理员角色 | 
读取基本配置 | 默认用户角色（[请参阅文档](/active-directory/fundamentals/users-default-permissions)） | 
重置受限管理员的密码（请参阅文档） | 用户管理员 | 
重置非管理员的密码（请参阅文档） | 密码管理员 | 用户管理员
重置特权管理员的密码 | 全局管理员角色 | 
撤销许可证 | 许可证管理员 | 用户管理员
更新除用户主体名称之外的所有属性 | 用户管理员 | 
更新受限管理员的用户主体名称（请参阅文档） | 用户管理员 | 
更新特权管理员的用户主体名称属性（请参阅文档） | 全局管理员角色 | 
更新用户设置 | 全局管理员角色 | 


## <a name="support"></a>支持

任务 | 最小特权角色 | 其他角色
---- | --------------------- | ----------------
提交支持票证 | 服务管理员 | 应用程序管理员、Azure 信息保护管理员、计费管理员、云应用程序管理员、符合性管理员、Dynamics 365 管理员、桌面分析管理员、Exchange 管理员、密码管理员、Intune 管理员、Skype for Business 管理员、Power BI 管理员、特权身份验证管理员、SharePoint 管理员、Teams 通信管理员、Teams 管理员、用户管理员、工作区分析管理员

## <a name="next-steps"></a>后续步骤

* [如何分配或删除 Azure AD 管理员角色](directory-manage-roles-portal.md)
* [Azure AD 管理员角色参考](directory-assign-admin-roles.md)

<!-- Update_Description: wording update -->