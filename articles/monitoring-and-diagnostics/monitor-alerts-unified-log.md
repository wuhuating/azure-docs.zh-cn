---
title: "Azure Monitor - 警报（预览版）中的日志警报 | Microsoft Docs"
description: "当满足指定的复杂查询条件时，Azure 警报（预览版）会触发电子邮件、通知、调用网站 URL (Webhook) 或自动化。"
author: msvijayn
manager: kmadnani1
editor: 
services: monitoring-and-diagnostics
documentationcenter: monitoring-and-diagnostics
ms.assetid: f7457655-ced6-4102-a9dd-7ddf2265c0e2
ms.service: monitoring-and-diagnostics
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 02/02/2018
ms.author: vinagara
ms.openlocfilehash: 438776e7f0885dbdb0d66ccdd18d854e14beb299
ms.sourcegitcommit: fbba5027fa76674b64294f47baef85b669de04b7
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 02/24/2018
---
# <a name="log-alerts-in-azure-monitor---alerts-preview"></a>Azure Monitor - 警报（预览版）中的日志警报
本文详细说明了 Analytics 中的警报规则在 Azure 警报（预览版）中如何工作，并介绍了不同类型的日志警报规则之间的差异。

目前，Azure 警报（预览版）支持与来自 [Azure Log Analytics](../log-analytics/log-analytics-tutorial-viewdata.md) 和 [Application Insights](../application-insights/app-insights-cloudservices.md#view-azure-diagnostic-events) 的查询相关的日志警报。

> [!WARNING]

> 目前，Azure 警报（预览版）中的日志警报不支持跨工作区或跨应用查询。

另外，用户可以在 Azure 中精选的 Analytics 平台中完善其查询，然后*通过保存查询将它们导出以在警报（预览版）中使用*。 要遵循的步骤如下：
- 对于 Application Insights：转到 Analytics 门户，验证查询及其结果。 然后，以唯一名称将其保存到“共享查询”。
- 对于 Log Analytics：转到“日志搜索”，验证查询及其结果。 然后，以唯一名称将其保存到任何类别。

[在警报（预览版）中创建日志警报](monitor-alerts-unified-usage.md)时，可以看到保存的查询作为信号类型“日志(已保存的查询)”列出；如以下示例中所示：![导入到警报的已保存查询](./media/monitor-alerts-unified/AlertsPreviewResourceSelectionLog-new.png)

> [!NOTE]
> 使用“日志(已保存的查询)”会导致导入到警报中。 因此，之后在 Analytics 中所做的任何更改都不会反映在已保存的警报规则中，反之亦然。

## <a name="log-alert-rules"></a>日志警报规则

警报是由 Azure 警报（预览版）通过定期自动运行日志查询来创建的。  如果日志查询的结果符合特定条件，则会创建警报记录。 然后，该规则可使用[操作组](monitoring-action-groups.md)自动运行一个或多个操作以主动通知你存在警报或调用另一个进程，例如，使用[基于 json 的 webhook](monitor-alerts-unified-log-webhook.md) 将数据发送到外部应用程序。 不同类型的警报规则使用不同的逻辑来执行此分析。

预警规则通过以下详细信息定义：

- **日志查询**。  这是每次触发预警规则时都会运行的查询。  此查询返回的记录用于确定是否创建警报。
- **时间窗口**。  指定查询的时间范围。  查询仅返回在当前时间的这个范围内创建的记录。  时间窗口可以是介于 5 分钟和 1440 分钟（或 24 小时）之间的任何值。 例如，如果时间范围设置为 60 分钟，且在下午 1:15 运行查询，则仅返回中午 12:15 和下午 1:15 之间创建的记录。
- **频率**。  指定应运行查询的频率。 可以是介于 5 分钟到 24 小时之间的任何值。 应等于或小于时间范围。  如果该值大于时间范围，则会有记录缺失的风险。<br>例如，假设时间范围为 30 分钟，频率为 60 分钟。  如果查询在下午 1:00 运行，则会返回中午 12:30 和下午 1:00 之间的记录。  下次运行查询的时间是下午 2:00，会返回下午 1:30 到 2:00 之间的记录。  在下午 1:00 和 1:30 之间创建的任何记录不会获得评估。
- **阈值**。  对日志搜索的结果进行评估，确定是否应创建警报。  不同类型的警报规则的阈值不同。

Log Analytics 中的预警规则均为以下两种类型之一。  这些类型中的每一种都在随后的相应部分进行了详细介绍。

- **[结果数](#number-of-results-alert-rules)**。 当日志搜索返回的记录数超出指定数目时，将创建单个警报。
- **[指标度量值](#metric-measurement-alert-rules)**。  为日志搜索结果中其值超出指定阈值的每个对象创建警报。

警报规则类型之间的差异如下所示。

- **“结果数”警报规则始终创建单个警报，而“指标度量”预警规则将为超出阈值的每个对象创建一个警报。
- “结果数”预警规则会在超出阈值一次时创建一个警报。 当阈值在特定的时间间隔内超出特定的次数时，“指标度量”警报规则即可创建一个警报。

## <a name="number-of-results-alert-rules"></a>“结果数”警报规则
当搜索查询返回的记录数超出指定的阈值时，“结果数”警报规则将创建一个警报。 此类预警规则适用于处理 Windows 事件日志、Syslog、WebApp Response 和自定义日志等事件。  生成特定错误事件时，或在特定时间范围内生成多个错误事件时，就可能需要创建警报。

**阈值**：**“结果数”警报规则的阈值要么超出某个特定值，要么低于该值。  如果日志搜索返回的记录数与此条件匹配，则创建警报。

若要针对单个事件发出警报，请将结果数设置为大于 0 并检查自上次运行查询起创建的某事件的出现次数。 某些应用程序可能会记录不一定引发警报的偶然错误。  例如，应用程序可能会重试导致错误事件的进程，而下一次就会成功。  在这种情况下，可能不想创建警报，除非在特定时间范围内创建多个事件。  

某些情况下，可能需要在没有事件的情况下创建警报。  例如，进程可能记录常规事件以指明其运行正常。  如果它不在特定时间范围内记录某个事件，则应创建警报。  在这种情况下，应将阈值设置为**小于 1**。

### <a name="example"></a>示例
假设你希望知道你的基于 web 的应用何时向用户返回代码为 500 的响应，即内部服务器错误。 可以创建一个警报规则，详情如下：  
**查询：** requests | where resultCode == "500"<br>
**时间窗口：**30 分钟<br>
**警报频率：**五分钟<br>
**阈值：**大于 0<br>

然后，警报将每隔 5 分钟运行一次查询，在 30 分钟的数据中查找结果代码为 500 的任何记录。 如果一个这样的记录都找不到，则它会引发警报并触发所配置的操作。

## <a name="metric-measurement-alert-rules"></a>指标度量警报规则

“指标度量”警报规则为查询中其值超出指定阈值的每个对象创建一个警报。  这些规则具有下述不同于“结果数”警报规则的差异。

**聚合函数**：确定要执行的计算以及可能要聚合的数字字段。  例如，**count()** 返回查询中的记录数，**avg(CounterValue)** 返回 CounterValue 字段在特定时间间隔内的平均值。

> [!NOTE]

> 查询中的聚合函数必须名为：AggregatedValue 并提供数值。 


**分组字段**：将为此字段的每个实例创建包含聚合值的记录，并可为每个实例生成警报。  例如，如果需要为每台计算机生成一个警报，则可使用“按计算机”。   

> [!NOTE]

> 对于基于 Application Insights 的指标度量警报规则，可以指定对数据分组的字段。 若要执行此操作，请使用规则定义中的“聚合基于”选项。   

**间隔**：定义一个时间间隔，在该间隔内对数据进行聚合。  例如，如果指定“五分钟”，则会在为警报指定的时间范围内，为分组字段（按 5 分钟间隔进行聚合）的每个实例创建一个记录。
> [!NOTE]
> 必须在查询中使用 Bin 函数。 另外，如果通过使用 Bin 函数为时间窗口生成了不均匀的时间间隔 - 则警报将改用 bin_at function 来确保存在一个固定点

**阈值**：“指标度量”警报规则的阈值通过一个聚合值和一个违规次数来定义。  如果日志搜索中的某数据点超出该值，则被视为违规。  如果结果中某对象的违规次数超出指定值，则会针对该对象创建警报。

#### <a name="example"></a>示例
考虑一下这样一种情形：如果任何计算机的处理器利用率在 30 分钟内超出 90% 三次，则需发出警报。  可以创建一个警报规则，详情如下：  

**查询：**Perf | where ObjectName == "Processor" and CounterName == "% Processor Time" | summarize AggregatedValue = avg(CounterValue) by bin(TimeGenerated, 5 m), Computer<br>
**时间窗口：**30 分钟<br>
**警报频率：**五分钟<br>
**聚合值：**大于 90<br>
**触发警报的标准：**总违规次数大于 5 次<br>

查询将按 5 分钟的时间间隔为每台计算机创建一个平均值。  对于在前 30 分钟 内收集的数据，此查询将每隔 5 分钟运行一次。  下面显示的示例数据是针对三台计算机的。

![示例查询结果](../log-analytics/media/log-analytics-alerts/metrics-measurement-sample-graph.png)

在此示例中，将为 srv02 和 srv03 创建单独的警报，因为二者都在指定的时间范围内超出 90% 的阈值 3 次。  如果将“触发警报的标准:”更改为“连续”，则只会为 srv03 创建警报，因为对于连续三个示例，它都超出了阈值。


## <a name="next-steps"></a>后续步骤
* 了解[用于日志警报的 Webhook 操作](monitor-alerts-unified-log-webhook.md)
* [获取 Azure 警报（预览版）的概述](monitoring-overview-unified-alerts.md)
* 了解如何[使用 Azure 警报（预览版）](monitor-alerts-unified-usage.md)
* 详细了解 [Application Insights](../application-insights/app-insights-analytics.md)
* 详细了解 [Log Analytics](../log-analytics/log-analytics-overview.md)。    
