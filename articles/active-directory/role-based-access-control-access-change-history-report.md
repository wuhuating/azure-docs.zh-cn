---
title: "创建访问权限更改历史记录报告 | Microsoft Docs"
description: "生成一个列出过去 90 天内使用基于角色的访问控制对 Azure 订阅的访问权限进行的所有更改的报告。"
services: active-directory
documentationcenter: 
author: kgremban
manager: femila
editor: 
ms.assetid: 2bc68595-145e-4de3-8b71-3a21890d13d9
ms.service: active-directory
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: identity
ms.date: 08/03/2016
ms.author: kgremban
translationtype: Human Translation
ms.sourcegitcommit: 2ea002938d69ad34aff421fa0eb753e449724a8f
ms.openlocfilehash: 44295ff647cbfd2d63ffe08d101da66b83a924f6


---
# <a name="create-an-access-change-history-report"></a>创建访问权限更改历史记录报告
无论何时授予或撤销订阅中的访问权限，更改都将记录在 Azure 事件中。 可创建访问权限更改历史记录报告来查看过去 90 天的所有更改。

## <a name="create-a-report-with-azure-powershell"></a>使用 Azure PowerShell 创建报告
若要在 PowerShell 中创建访问权限更改历史记录报告，请使用 `Get-AzureRMAuthorizationChangeLog` 命令。 有关此 cmdlet 的更多详细信息可在 [PowerShell 库](https://www.powershellgallery.com/packages/AzureRM.Storage/1.0.6/Content/ResourceManagerStartup.ps1)中找到。

调用此命令时，可指定你要列出的分配的属性，包括以下方面：

| 属性 | 说明 |
| --- | --- |
| **Action** |是授予还是撤销访问权限 |
| **Caller** |负责访问权限更改的所有者 |
| **日期** |访问权限更改的日期和时间 |
| **DirectoryName** |Azure Active Directory 目录 |
| **PrincipalName** |用户、组或应用程序的名称 |
| **PrincipalType** |分配是针对用户、组还是应用程序 |
| **RoleId** |已授予或已撤销角色的 GUID |
| **RoleName** |已授予或已撤销的角色 |
| **ScopeName** |订阅、资源组或资源的名称 |
| **ScopeType** |分配是在订阅、资源组中还是在资源范围内 |
| **SubscriptionId** |Azure 订阅的 GUID |
| **SubscriptionName** |Azure 订阅的名称 |

此示例命令列出过去 7 天内订阅中所有访问权限的更改：

```
Get-AzureRMAuthorizationChangeLog -StartTime ([DateTime]::Now - [TimeSpan]::FromDays(7)) | FT Caller,Action,RoleName,PrincipalType,PrincipalName,ScopeType,ScopeName
```

![PowerShell Get-AzureRMAuthorizationChangeLog - 屏幕快照](./media/role-based-access-control-configure/access-change-history.png)

## <a name="create-a-report-with-azure-cli"></a>使用 Azure CLI 创建报告
若要在 Azure 命令行界面 (CLI) 中创建访问权限更改历史记录报告，请使用 `azure role assignment changelog list` 命令。

## <a name="export-to-a-spreadsheet"></a>导出到电子表格
若要保存报告或处理数据，请将访问权限更改导出至 .csv 文件。 可在电子表格中查看该报告以进行审阅。

![更改日志被视为电子表格 - 屏幕快照](./media/role-based-access-control-configure/change-history-spreadsheet.png)

## <a name="see-also"></a>另请参阅
* 开始使用 [Azure 基于角色的访问控制](role-based-access-control-configure.md)
* 使用 [Azure RBAC 中的自定义角色](role-based-access-control-custom-roles.md)




<!--HONumber=Nov16_HO3-->

