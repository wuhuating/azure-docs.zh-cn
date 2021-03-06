---
title: "Azure 资源管理器模板示例 | Microsoft Docs"
description: "Azure 资源管理器模板示例 - 应用服务"
services: app-service
documentationcenter: app-service
author: tfitzmac
manager: timlt
editor: ggailey777
tags: azure-service-management
ms.service: app-service
ms.devlang: na
ms.topic: sample
ms.tgt_pltfrm: na
ms.workload: app-service
ms.date: 02/26/2018
ms.author: tomfitz
ms.custom: mvc
ms.openlocfilehash: e28a27b9a00164fcbba6eb5d3e5435548486ffa4
ms.sourcegitcommit: 088a8788d69a63a8e1333ad272d4a299cb19316e
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 02/27/2018
---
# <a name="azure-resource-manager-templates-for-azure-web-apps"></a>用于 Azure Web 应用的 Azure 资源管理器模板

下表包含 Azure 资源管理器模板链接。 有关如何在创建 Web 应用模板时避免常见错误的建议，请参阅[使用 Azure 资源管理器模板部署 Web 应用的指南](web-sites-rm-template-guidance.md)。

| | |
|-|-|
|**部署 Web 应用**||
| [链接到 GitHub 存储库的 Web 应用](https://github.com/Azure/azure-quickstart-templates/tree/master/201-web-app-github-deploy)| 部署从 GitHub 拉取代码的 Azure Web 应用。 |
| [使用自定义部署槽的 Web 应用](https://github.com/Azure/azure-quickstart-templates/tree/master/101-webapp-custom-deployment-slots)| 使用自定义部署槽/环境部署 Azure Web 应用。 |
|**配置 Web 应用**||
| [来自 Key Vault 的 Web 应用证书](https://github.com/Azure/azure-quickstart-templates/tree/master/201-web-app-certificate-from-key-vault)| 部署来自 Key Vault 机密的 Azure Web 应用证书并将其用于 SSL 绑定。 |
| [使用自定义域的 Web 应用](https://github.com/Azure/azure-quickstart-templates/tree/master/201-web-app-custom-domain)| 使用自定义主机名部署 Azure Web 应用。 |
| [使用自定义域和 SSL 的 Web 应用](https://github.com/Azure/azure-quickstart-templates/tree/master/201-web-app-custom-domain-and-ssl)| 使用自定义主机名部署 Azure Web 应用，并从 Key Vault 获取 Web 应用证书用于 SSL 绑定。 |
| [使用 GoLang 扩展的 Web 应用](https://github.com/Azure/azure-quickstart-templates/tree/master/101-webapp-with-golang)| 使用 Golang 站点扩展部署 Azure Web 应用，以便运行在 Azure 的 Golang 上开发的 Web 应用程序。 |
| [使用 Java 8 和 Tomcat 8 的 Web 应用](https://github.com/Azure/azure-quickstart-templates/tree/master/201-web-app-java-tomcat)| 部署启用了 Java 8 和 Tomcat 8 的 Azure Web 应用，以便在 Azure 中运行 Java 应用程序。 |
|**Linux Web 应用**||
| [Linux 上的使用 MySQL 的 Web 应用](https://github.com/Azure/azure-quickstart-templates/tree/master/101-webapp-linux-managed-mysql) | 在 Linux 上部署使用 Azure Database for MySQL 的 Azure Web 应用。 |
| [Linux 上的使用 PostgreSQL 的 Web 应用](https://github.com/Azure/azure-quickstart-templates/tree/master/101-webapp-linux-managed-postgresql) | 在 Linux 上部署使用 Azure Database for PostgreSQL 的 Azure Web 应用。 |
|**使用连接的资源的 Web 应用**||
| [使用 MySQL 的 Web 应用](https://github.com/Azure/azure-quickstart-templates/tree/master/101-webapp-managed-mysql)| 在 Windows 上部署使用 Azure Database for MySQL 的 Azure Web 应用。 |
| [使用 PostgreSQL 的 Web 应用](https://github.com/Azure/azure-quickstart-templates/tree/master/101-webapp-managed-postgresql)| 在 Windows 上部署使用 Azure Database for PostgreSQL 的 Azure Web 应用。 |
| [使用 SQL 数据库的 Web 应用](https://github.com/Azure/azure-quickstart-templates/tree/master/201-web-app-sql-database)| 在“基本”服务级别部署 Azure Web 应用和 SQL 数据库。 |
| [使用 Blob 存储连接的 Web 应用](https://github.com/Azure/azure-quickstart-templates/tree/master/201-web-app-blob-connection)| 使用 Blob 存储连接字符串部署 Azure Web 应用，以便使用来自 Web 应用的 Azure Blob 存储。 |
| [使用 Redis 缓存的 Web 应用](https://github.com/Azure/azure-quickstart-templates/tree/master/201-web-app-with-redis-cache)| 使用 Redis 缓存部署 Azure Web 应用。 |
|**应用服务环境**||
| [创建应用服务环境 v2](https://github.com/Azure/azure-quickstart-templates/tree/master/201-web-app-asev2-create) | 在虚拟网络中创建应用服务环境 v2。 |
| [Create an App Service Environment v2 with an ILB Address](https://github.com/Azure/azure-quickstart-templates/tree/master/201-web-app-asev2-ilb-create/)（使用 ILB 地址创建应用服务环境 v2） | 使用专用的内部负载均衡器地址在虚拟网络中创建应用服务环境 v2。 |
| [Configure the Default SSL Certificate for an ILB ASE or an ILB ASE v2](https://github.com/Azure/azure-quickstart-templates/tree/master/201-web-app-ase-ilb-configure-default-ssl)（为 ILB ASE 或 ILB ASE v2 配置默认的 SSL 证书） | 为 ILB 应用服务环境或 ILB 应用服务环境 v2 配置默认的 SSL 证书 |
| | |
