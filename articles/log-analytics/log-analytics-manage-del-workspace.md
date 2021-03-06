---
title: "删除 Azure Log Analytics 工作区 | Microsoft Docs"
description: "了解在个人订阅中创建 Log Analytics 工作区后如何删除它，以及如何重构工作区模型。"
services: log-analytics
documentationcenter: log-analytics
author: mgoedtel
manager: carmonm
editor: 
ms.assetid: 
ms.service: log-analytics
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 10/27/2017
ms.author: magoedte
ms.custom: mvc
ms.openlocfilehash: 4f455e26f078360f17f8118f8b5b1db5bd75d473
ms.sourcegitcommit: d87b039e13a5f8df1ee9d82a727e6bc04715c341
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 02/21/2018
---
# <a name="delete-an-azure-log-analytics-workspace-with-the-azure-portal"></a>使用 Azure 门户删除 Azure Log Analytics 工作区
本主题介绍如何使用 Azure 门户删除可能不再需要的 Log Analytics 工作区。 

## <a name="to-delete-a-workspace"></a>删除工作区 
删除 Log Analytics 工作区时，会在 30 天内从服务中删除与工作区相关的所有数据。  删除工作区时需谨慎，因为其中可能含有会对服务操作产生不利影响的重要数据和配置。  
 
如果是管理员并且存在多个关联到工作区的用户，则会断开这些用户和该工作区之间的关联。 如果这些用户与其他工作区关联，他们可以继续通过其他工作区使用 Log Analytics。 但是，如果他们未与其他工作区关联，则需要创建工作区才能使用 Log Analytics。 

1. 登录到 [Azure 门户](http://portal.azure.com)。 
2. 在 Azure 门户中，单击“所有服务”。 在资源列表中，键入“Log Analytics”。 开始键入时，会根据输入筛选该列表。 选择“Log Analytics”。
3. 在 Log Analytics 订阅窗格中，选择一个工作区，然后从中间窗格的顶端单击“删除”。<br><br> ![从工作区属性窗格中删除选项](media/log-analytics-manage-del-workspace/log-analytics-delete-workspace.png)<br>  
4. 显示询问是否确实要删除工作区的确认消息窗口时，单击“是”。<br><br> ![确认删除工作区](media/log-analytics-manage-del-workspace/log-analytics-delete-workspace-confirm.png)

