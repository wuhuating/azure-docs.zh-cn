---
title: "Azure 文件同步代理发行说明 | Microsoft Docs"
description: "Azure 文件同步发行说明"
services: storage
documentationcenter: 
author: wmgries
manager: klaasl
editor: tamram
ms.assetid: 
ms.service: storage
ms.workload: storage
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: get-started-article
ms.date: 10/08/2017
ms.author: wgries
ms.openlocfilehash: 5f57edb33770acd7a97287d5cfd650b7fe8366f4
ms.sourcegitcommit: 782d5955e1bec50a17d9366a8e2bf583559dca9e
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 03/02/2018
---
# <a name="azure-file-sync-agent-release-notes"></a>Azure 文件同步代理发行说明
借助 Azure 文件同步（预览版），既可将组织的文件共享集中在 Azure 文件中，又不失本地文件服务器的灵活性、性能和兼容性。 它通过将 Windows Server 转换为 Azure 文件共享的快速缓存来实现这一点。 你可以使用 Windows Server 上的任意可用协议在本地访问数据（包括 SMB、NFS 和 FTPS），并且可以在世界各地获取所需的缓存数。

本文介绍 Azure 文件同步代理的支持版本的发行说明。

## <a name="supported-versions"></a>支持的版本
Azure 文件同步支持以下版本：

| 代理版本号 | 发行日期 | 支持截止至 |
|----------------------|--------------|------------------|
| 2.1.0.0 | 2018-02-28 | 当前版本 |
| 2.0.11.0 | 2018-02-08 | 当前版本 |
| 1.1.0.0 | 2017-09-26 | 2018-07-30 |

### <a name="azure-file-sync-agent-update-policy"></a>Azure 文件同步代理更新策略
[!INCLUDE [storage-sync-files-agent-update-policy](../../../includes/storage-sync-files-agent-update-policy.md)]

## <a name="agent-version-2100"></a>代理版本 2.1.0.0
以下发行说明适用于 2018 年 2 月 28 日发布的代理版本 2.1.0。 这些是附加到下面针对版本 2.0.11.0 的发行说明的

对此每月更新进行的特定更改包括：
- 对群集故障转移处理的改进。
- 对分层文件处理的改进，处理更可靠。
- 允许在添加到 2008R2 域环境的域控制器计算机上进行代理安装。
- 修复了在包含许多文件的服务器上生成诊断过多的问题。
- 改进了会话故障时的错误处理。
- 改进了出现文件传输问题时的错误处理。
- 将运行云分层（如果已在服务器终结点上启用）的默认时间间隔更改为一小时。 
- 临时阻止将 Azure 文件同步（存储同步服务）资源移到新的 Azure 订阅

## <a name="agent-version-20110"></a>代理版本 2.0.11.0
以下发行说明适用于 2018 年 2 月 9 日发布的代理版本 2.0.11.0。 

### <a name="agent-installation-and-server-configuration"></a>代理安装和服务器配置
若要详细了解如何使用 Windows Server 来安装和配置 Azure 文件同步代理，请参阅[规划 Azure 文件同步（预览版）部署](storage-sync-files-planning.md)和[如何部署 Azure 文件同步（预览版）](storage-sync-files-deployment-guide.md)。

- 代理安装包 (MSI) 必须使用提升的（管理员）权限进行安装。
- 在 Windows Server Core 或 Nano Server 部署选项上不受支持。
- 仅在 Windows Server 2016 和 2012 R2 上受支持。
- 代理需要至少 2 GB 的物理内存。

### <a name="interoperability"></a>互操作性
- 防病毒应用程序、备份应用程序和其他应用程序在访问分层文件时会导致不需要的撤回，除非这些应用程序遵从脱机属性，在读取这些文件的内容时跳过。 有关详细信息，请参阅[对 Azure 文件同步（预览版）进行故障排除](storage-sync-files-troubleshoot.md)。
- 此更新的新增功能是支持 DFS-R。  请参阅[规划指南](storage-sync-files-planning.md#distributed-file-system-dfs)以获取详细信息。
- 请勿使用文件服务器资源管理器（或其他）文件屏蔽：如果文件因文件屏蔽而被阻止，文件屏蔽可能导致无休止的同步故障。
- 复制已注册服务器（包括 VM 克隆）可能导致意外结果（具体说来，同步可能永远不会聚合）。
- 不支持在同一卷上进行重复数据删除和云分层操作。
 
### <a name="sync-limitations"></a>同步限制
以下各项不同步，但系统其余部分仍会继续正常运行：
- 长度超过 2048 个字符的路径
- 在大于 2K 的情况下安全描述符的 DACL 部分（只有在单个项上的访问控制条目大于某个数（大约为 40）的情况下，这才是一个问题）
- 安全描述符的 SACL 部分（用于审核）
- 扩展的属性
- 备用数据流
- 重分析点
- 硬链接
- 将更改从其他终结点同步到服务器文件时，不会保留压缩（如果在该文件上设置了压缩）
- 任何使用 EFS（或其他用户模式加密方式）加密的文件，此类文件会阻止我们的服务读取数据 
    
    > [!Note]  
    > Azure 文件同步始终加密传输中的数据，而在 Azure 中，数据可以进行静态加密。
 
### <a name="server-endpoints"></a>服务器终结点
- 服务器终结点只能在 NTFS 卷上创建。 Azure 文件同步目前不支持 ReFS、FAT、FAT32 等文件系统。
- 服务器终结点不能位于系统卷上（例如，不能接受 C:\MyFolder 作为路径，除非 C:\MyFolder 为装入点）。
- 故障转移群集仅适用于群集磁盘，不适用于群集共享卷 (CSV)。
- 服务器终结点不能嵌套，但可以彼此并行共存于同一卷上。
- 一次从服务器删除大量目录（超过 10,000 个）可能导致同步故障 - 请批量删除目录，每批少于 10,000 个。在确保删除操作同步成功以后，再删除下一批。
- 此版本增加了对同步根目录的支持，允许将其置于卷的根目录处。
- 请勿在服务器终结点中存储 OS 或应用程序分页文件。
- 此版本的变化是添加了新的事件，用于跟踪云分层 (EventID 9016)、同步上传进度 (EventID 9302) 以及未同步文件 (EventID 9900) 的总运行时。
- 此版本的变化是，快速 DR 命名空间同步性能大幅提高。
 
### <a name="cloud-tiering"></a>云分层
- 与前一版本不同的是，新文件会在 1 小时内根据分层策略设置进行分层（以前需要 32 小时）- 我们提供可以按需分层的 PowerShell cmdlet，方便你更有效地对分层进行评估，不需等待后台进程。
- 如果使用 Robocopy 将分层的文件复制到另一位置，则生成的文件不会分层，但脱机属性可能会进行设置，因为 Robocopy 会在复制操作中错误地包括该属性。
- 从 SMB 客户端查看文件属性时，脱机属性可能会显示为设置不正确，因为对文件元数据进行了 SMB 缓存。
- 与前一版本不同的是，文件现在是作为分层文件下载到服务器上的，但前提是文件是新的，或者已经是分层的文件。

## <a name="agent-version-1100"></a>代理版本 1.1.0.0
以下发行说明适用于 2017 年 9 月 9 日发布的代理版本 1.1.0.0。 此发行版标志着 Azure 文件同步的初始预览版！

### <a name="agent-installation-and-server-configuration"></a>代理安装和服务器配置
若要详细了解如何使用 Windows Server 来安装和配置 Azure 文件同步代理，请参阅[规划 Azure 文件同步（预览版）部署](storage-sync-files-planning.md)和[如何部署 Azure 文件同步（预览版）](storage-sync-files-deployment-guide.md)。

- 代理安装包 (MSI) 必须使用提升的（管理员）权限进行安装。
- 在 Windows Server Core 或 Nano Server 部署选项上不受支持。
- 仅在 Windows Server 2016 和 2012 R2 上受支持。
- 代理需要至少 2 GB 的物理内存。

### <a name="interoperability"></a>互操作性
- 防病毒应用程序、备份应用程序和其他应用程序在访问分层文件时会导致不需要的撤回，除非这些应用程序遵从脱机属性，在读取这些文件的内容时跳过。 有关详细信息，请参阅[对 Azure 文件同步（预览版）进行故障排除](storage-sync-files-troubleshoot.md)。
- 请勿使用文件服务器资源管理器（或其他）文件屏蔽：如果文件因文件屏蔽而被阻止，文件屏蔽可能导致无休止的同步故障。
- 复制已注册服务器（包括 VM 克隆）可能导致意外结果（具体说来，同步可能永远不会聚合）。
- 不支持在同一卷上进行重复数据删除和云分层操作。
 
### <a name="sync-limitations"></a>同步限制
以下各项不同步，但系统其余部分仍会继续正常运行：
- 长度超过 2048 个字符的路径
- 在大于 2K 的情况下安全描述符的 DACL 部分（只有在单个项上的访问控制条目大于某个数（大约为 40）的情况下，这才是一个问题）
- 安全描述符的 SACL 部分（用于审核）
- 扩展的属性
- 备用数据流
- 重分析点
- 硬链接
- 将更改从其他终结点同步到服务器文件时，不会保留压缩（如果在该文件上设置了压缩）
- 任何使用 EFS（或其他用户模式加密方式）加密的文件，此类文件会阻止我们的服务读取数据 
    
    > [!Note]  
    > Azure 文件同步始终加密传输中的数据，而在 Azure 中，数据可以进行静态加密。
 
### <a name="server-endpoints"></a>服务器终结点
- 服务器终结点只能在 NTFS 卷上创建。 Azure 文件同步目前不支持 ReFS、FAT、FAT32 等文件系统。
- 服务器终结点不能位于系统卷上（例如，不能接受 C:\MyFolder 作为路径，除非 C:\MyFolder 为装入点）。
- 故障转移群集仅适用于群集磁盘，不适用于群集共享卷 (CSV)。
- 服务器终结点不能嵌套，但可以彼此并行共存于同一卷上。
- 一次从服务器删除大量目录（超过 10,000 个）可能导致同步故障 - 请批量删除目录，每批少于 10,000 个。在确保删除操作同步成功以后，再删除下一批。
- 在卷的根目录不受支持。
- 请勿在服务器终结点中存储 OS 或应用程序分页文件。
 
### <a name="cloud-tiering"></a>云分层
- 为了确保文件能够正确撤回，系统不会在开始的某段时间（最多 32 小时）自动对新文件或已更改文件分层（包括在配置新服务器终结点后进行的首次分层）- 我们提供按需分层的 PowerShell cmdlet，可以更有效地对分层进行评估，不需等待后台进程。
- 如果使用 Robocopy 将分层的文件复制到另一位置，则生成的文件不会分层，但脱机属性可能会进行设置，因为 Robocopy 会在复制操作中错误地包括该属性。
- 从 SMB 客户端查看文件属性时，脱机属性可能会显示为设置不正确，因为对文件元数据进行了 SMB 缓存。
