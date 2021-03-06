---
title: "在 Azure 中创建运行 SQL&#92;IIS&#92;.NET 堆栈的 VM | Microsoft Docs"
description: "教程 - 在 Windows 虚拟机上安装 Azure SQL、IIS、.NET 堆栈。"
services: virtual-machines-windows
documentationcenter: virtual-machines
author: cynthn
manager: jeconnoc
editor: tysonn
tags: azure-resource-manager
ms.service: virtual-machines-windows
ms.devlang: na
ms.topic: tutorial
ms.tgt_pltfrm: vm-windows
ms.workload: infrastructure
ms.date: 02/27/2018
ms.author: cynthn
ms.custom: mvc
ms.openlocfilehash: ad84d6e8f74fa184ac2359ff7f08e6c8143d419a
ms.sourcegitcommit: 83ea7c4e12fc47b83978a1e9391f8bb808b41f97
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 02/28/2018
---
# <a name="install-a-sql92iis92net-stack-in-azure"></a>在 Azure 中安装 SQL&#92;IIS&#92;.NET 堆栈

在本教程中，我们将使用 Azure PowerShell 安装 SQL&#92;IIS&#92;.NET 堆栈。 此堆栈包含两个运行 Windows Server 2016 的 VM，一个带有 IIS 和 .NET，另一个带有 SQL Server。

> [!div class="checklist"]
> * 创建 VM 
> * 在 VM 上安装 IIS 和 .NET Core SDK
> * 创建运行 SQL Server 的 VM
> * 安装 SQL Server 扩展

[!INCLUDE [cloud-shell-powershell.md](../../../includes/cloud-shell-powershell.md)]

本教程需要 AzureRM.Compute 模块 4.3.1 或更高版本。 运行 `Get-Module -ListAvailable AzureRM.Compute` 即可查找版本。 如果需要升级，请参阅[安装 Azure PowerShell 模块](/powershell/azure/install-azurerm-ps)。

## <a name="create-a-iis-vm"></a>创建 IIS VM 

在此示例中，我们使用 [New-AzureRMVM](/powershell/module/azurerm.compute/new-azurermvm) cmdlet 在 PowerShell Cloud Shell 中快速创建 Windows Server 2016 VM，然后安装 IIS 和 .NET Framework。 IIS 和 SQL VM 共享资源组和虚拟网络，因此我们创建这些名称的变量。


```azurepowershell-interactive
$vmName = "IISVM"
$vNetName = "myIISSQLvNet"
$resourceGroup = "myIISSQLGroup"
New-AzureRmVm `
    -ResourceGroupName $resourceGroup `
    -Name $vmName `
    -Location "East US" `
    -VirtualNetworkName $vNetName `
    -SubnetName "myIISSubnet" `
    -SecurityGroupName "myNetworkSecurityGroup" `
    -AddressPrefix 192.168.0.0/16 `
    -PublicIpAddressName "myIISPublicIpAddress" `
    -OpenPorts 80,3389 
```

使用自定义脚本扩展安装 IIS 和 .NET framework。

```azurepowershell-interactive
Set-AzureRmVMExtension `
    -ResourceGroupName $resourceGroup `
    -ExtensionName IIS `
    -VMName $vmName `
    -Publisher Microsoft.Compute `
    -ExtensionType CustomScriptExtension `
    -TypeHandlerVersion 1.4 `
    -SettingString '{"commandToExecute":"powershell Add-WindowsFeature Web-Server,Web-Asp-Net45,NET-Framework-Features"}' `
    -Location EastUS
```

## <a name="create-another-subnet"></a>创建另一子网

为 SQL VM 创建第二个子网。 使用 [Get-AzureRmVirtualNetwork]{/powershell/module/azurerm.network/get-azurermvirtualnetwork} 获取 vNet。

```azurepowershell-interactive
$vNet = Get-AzureRmVirtualNetwork `
   -Name $vNetName `
   -ResourceGroupName $resourceGroup
```

使用 [Add-AzureRmVirtualNetworkSubnetConfig](/powershell/module/azurerm.network/add-azurermvirtualnetworksubnetconfig) 为子网创建配置。


```azurepowershell-interactive
Add-AzureRmVirtualNetworkSubnetConfig `
   -AddressPrefix 192.168.0.0/24 `
   -Name mySQLSubnet `
   -VirtualNetwork $vNet `
   -ServiceEndpoint Microsoft.Sql
```

通过 [Set-AzureRmVirtualNetwork](/powershell/module/azurerm.network/set-azurermvirtualnetwork) 使用新的子网信息更新 vNet
   
```azurepowershell-interactive   
$vNet | Set-AzureRmVirtualNetwork
```

## <a name="azure-sql-vm"></a>Azure SQL VM

使用 SQL Server 的预配置 Azure Marketplace 映像创建 SQL VM。 首先创建 VM，然后在 VM 上安装 SQL Server 扩展。 


```azurepowershell-interactive
New-AzureRmVm `
    -ResourceGroupName $resourceGroup `
    -Name "mySQLVM" `
    -ImageName "MicrosoftSQLServer:SQL2016SP1-WS2016:Enterprise:latest" `
    -Location eastus `
    -VirtualNetworkName $vNetName `
    -SubnetName "mySQLSubnet" `
    -SecurityGroupName "myNetworkSecurityGroup" `
    -PublicIpAddressName "mySQLPublicIpAddress" `
    -OpenPorts 3389,1401 
```

使用 [Set-AzureRmVMSqlServerExtension](/powershell/module/azurerm.compute/set-azurermvmsqlserverextension) 将 [SQL Server 扩展](/sql/virtual-machines-windows-sql-server-agent-extension.md)添加到 SQL VM。

```azurepowershell-interactive
Set-AzureRmVMSqlServerExtension `
   -ResourceGroupName $resourceGroup  `
   -VMName mySQLVM `
   -Name "SQLExtension" `
   -Location "EastUS"
```

## <a name="next-steps"></a>后续步骤

在本教程中，已使用 Azure PowerShell 安装 SQL&#92;IIS&#92;.NET 堆栈。 你已了解如何：

> [!div class="checklist"]
> * 创建 VM 
> * 在 VM 上安装 IIS 和 .NET Core SDK
> * 创建运行 SQL Server 的 VM
> * 安装 SQL Server 扩展

转到下一教程，了解如何使用 SSL 证书保护 IIS Web 服务器。

> [!div class="nextstepaction"]
> [使用 SSL 证书保护 IIS Web 服务器](tutorial-secure-web-server.md)

