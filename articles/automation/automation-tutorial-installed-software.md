---
title: "使用 Azure 自动化发现在计算机上安装的软件 | Microsoft Docs"
description: "使用清单发现在环境中的计算机上安装的软件。"
services: automation
keywords: "清单, 自动化, 更改, 跟踪"
author: jennyhunter-msft
ms.author: jehunte
ms.date: 02/28/2018
ms.topic: tutorial
ms.service: automation
ms.custom: mvc
manager: carmonm
ms.openlocfilehash: 97cd2c91ca2c70b044518c43d49356918202d5ff
ms.sourcegitcommit: 83ea7c4e12fc47b83978a1e9391f8bb808b41f97
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 02/28/2018
---
# <a name="discover-what-software-is-installed-on-your-azure-and-non-azure-machines"></a>发现在 Azure 计算机和非 Azure 计算机上安装的软件

本教程介绍如何发现在环境中安装的软件。 可以收集和查看清单，了解计算机上的软件、文件、Linux 守护程序、Windows 服务和 Windows 注册表项。 跟踪计算机的配置有助于查明环境中的操作问题，更好地了解计算机的状态。

本教程介绍如何执行下列操作：

> [!div class="checklist"]
> * 载入适用于更改跟踪和清单的 VM
> * 查看已安装软件
> * 在清单日志中搜索已安装软件

## <a name="prerequisites"></a>先决条件

要完成本教程，需要：

* Azure 订阅。 如果还没有帐户，可以[激活 MSDN 订户权益](https://azure.microsoft.com/pricing/member-offers/msdn-benefits-details/)或注册[免费帐户](https://azure.microsoft.com/free/?WT.mc_id=A261C142F)。
* [自动化帐户](automation-offering-get-started.md)，用于保存观察程序、操作 Runbook 和观察程序任务。
* 要载入的[虚拟机](../virtual-machines/windows/quick-create-portal.md)。

## <a name="log-in-to-azure"></a>登录 Azure

通过 http://portal.azure.com 登录到 Azure 门户。

## <a name="enable-change-tracking-and-inventory"></a>启用更改跟踪和清单

就本教程来说，首先需为 VM 启用更改跟踪和清单。 如果以前已为 VM 启用**更改跟踪**解决方案，则此步骤不是必需的。

1. 在左侧菜单上选择“虚拟机”，然后从列表中选择一个 VM。
2. 在左侧菜单的“操作”部分单击“清单”。 此时会打开“启用更改跟踪和清单”页。

![清单载入配置横幅](./media/automation-tutorial-installed-software/enableinventory.png)

若要启用此解决方案，请配置要使用的位置、Log Analytics 工作区和自动化帐户，然后单击“启用”。 如果这些字段灰显，则意味着已为 VM 启用其他自动化解决方案，因此必须使用同一工作区和自动化帐户。

[Log Analytics](../log-analytics/log-analytics-overview.md?toc=%2fazure%2fautomation%2ftoc.json) 工作区用于收集由功能和服务（如清单）生成的数据。
工作区提供了一个位置来查看和分析来自多个数据源的数据。

启用解决方案最多可能需要 15 分钟。 在此期间，不应关闭浏览器窗口。
启用该解决方案后，VM 中有关已安装软件和更改的信息会流向 Log Analytics。
这些数据需花费 30 分钟到 6 小时的时间才能用于分析。

## <a name="view-installed-software"></a>查看已安装软件

启用更改跟踪和清单解决方案以后，即可在“清单”页查看结果。

在 VM 中的“操作”下选择“清单”。

在“清单”页上单击“软件”选项卡。

在“软件”选项卡上有一个表，列出了已发现的软件。 软件按软件名称和版本分组。

可在表中查看每个软件记录的高级详细信息。 这些详细信息包括：软件名称、版本、发布者、上次刷新时间（由组中的计算机报告的最近刷新时间）、计算机数（装有该软件的计算机的计数）。

![软件清单](./media/automation-tutorial-installed-software/inventory-software.png)

单击一行即可查看软件记录的属性以及装有该软件的计算机的名称。

若要查找特定的软件或软件组，可以在软件列表正上方的文本框中搜索。
可以使用筛选器根据软件名称、版本或发布者进行搜索。

例如，搜索“Contoso”会返回名称、发布者或版本中包含“Contoso”的所有软件。

## <a name="search-inventory-logs-for-installed-software"></a>在清单日志中搜索已安装软件

清单生成发送到 Log Analytics 的日志数据。 若要通过运行查询来搜索日志，请选择“清单”窗口顶部的“Log Analytics”。

清单数据存储在 **ConfigurationData** 类型下。
以下 Log Analytics 查询示例返回包含“Microsoft”的发布者，以及每个发布者的软件记录（按 SoftwareName 和计算机分组）数。

```
ConfigurationData
| summarize arg_max(TimeGenerated, *) by SoftwareName, Computer
| where ConfigDataType == "Software"
| search Publisher:"Microsoft"
| summarize count() by Publisher
```

若要详细了解如何在 Log Analytics 中运行和搜索日志文件，请参阅 [Azure Log Analytics](https://docs.loganalytics.io/index)。

### <a name="single-machine-inventory"></a>单个计算机的清单

若要查看单个计算机的软件清单，可以访问 Azure VM 资源页中的“清单”，也可以使用 Log Analytics 筛选出相应的计算机。 以下 Log Analytics 查询示例返回名为 ContosoVM 的计算机的软件列表。

```
ConfigurationData
| where ConfigDataType == "Software" 
| summarize arg_max(TimeGenerated, *) by SoftwareName, CurrentVersion
| where Computer =="ContosoVM"
| render table
```

## <a name="next-steps"></a>后续步骤

本教程介绍了如何查看软件清单，例如，如何：

> [!div class="checklist"]
> * 载入适用于更改跟踪和清单的 VM
> * 查看已安装软件
> * 在清单日志中搜索已安装软件

继续阅读更改跟踪和清单解决方案的概述可以了解其详细信息。

> [!div class="nextstepaction"]
> [变更管理和清单解决方案](automation-change-tracking.md)