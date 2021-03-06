---
title: "在 Azure Active Directory 应用程序库中列出你的应用程序 | Microsoft Docs"
description: "如何在 Azure Active Directory 应用库中列出支持单一登录的应用程序"
services: active-directory
documentationcenter: dev-center-name
author: bryanla
manager: mbaldwin
editor: 
ms.assetid: 820acdb7-d316-4c3b-8de9-79df48ba3b06
ms.service: active-directory
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: identity
ms.date: 01/09/2018
ms.author: bryanla
ms.custom: aaddev
ms.openlocfilehash: 502fb555bd3b381c9be0ff04e210cc07f9bf6cd8
ms.sourcegitcommit: c765cbd9c379ed00f1e2394374efa8e1915321b9
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 02/28/2018
---
# <a name="list-your-application-in-the-azure-active-directory-application-gallery"></a>在 Azure Active Directory 应用程序库中列出你的应用程序


##  <a name="what-is-the-azure-ad-application-gallery"></a>什么是 Azure AD 应用程序库？

Azure Active Directory (Azure AD) 是一种基于云的标识服务。 [Azure AD 应用程序库](https://azure.microsoft.com/marketplace/active-directory/all/)位于 Azure Marketplace 应用商店中，所有应用程序连接器都发布在其中以用于实现单一登录和用户预配。 使用 Azure AD 作为标识提供者的客户可以查找发布在这里的各种 SaaS 应用程序连接器。 IT 管理员从应用库中添加连接器，然后对连接器进行配置并将其用于单一登录和预配。 Azure AD 支持使用所有主要的联合身份验证协议（包括 SAML 2.0、OpenID Connect、OAuth 和 WS-Fed）进行单一登录。 

## <a name="what-are-the-benefits-of-listing-an-application-in-the-gallery"></a>将应用程序列在该库中有什么好处？

*  可以为客户提供尽可能最佳的单一登录体验。

*  简化并最小化了应用程序的配置。 

*  在库中快速搜索应用程序。

*  免费、基本和高级 Azure AD 客户都可以使用此集成。 

*  共同客户可以获得分步配置教程。 

*  使用 SCIM 的客户可以将预配用于同一应用。


##  <a name="prerequisites-implement-federation-protocol"></a>先决条件：实现联合身份验证协议

若要将某个应用程序列在 Azure AD 应用库中，首先需要实现 Azure AD 支持的以下联合身份验证协议之一。 请从这里阅读 Azure AD 应用程序库的条款和条件。 

*   **OpenID Connect**：在 Azure AD 中创建多租户应用程序并为应用程序实现 [Azure AD 许可框架](active-directory-integrating-applications.md#overview-of-the-consent-framework)。 将登录请求发送到公用终结点，以便任何客户都可以向应用程序提供许可。 你可以根据在令牌中收到的租户 ID 和用户 UPN 来控制用户访问。 若要将应用程序与 Azure AD 集成，请遵循[开发人员说明](active-directory-authentication-scenarios.md)。

*   **SAML 2.0** 或 **WS-Fed**：应用程序需要能够以 SP 或 IDP 模式执行 SAML/WS-Fed SSO 集成。 如果你的应用支持 SAML 2.0，则可以根据[用来添加自定义应用程序的说明](../active-directory-saas-custom-apps.md)将其直接与 Azure AD 租户集成。

*   **密码 SSO**：创建具有 HTML 登录页面的 Web 应用程序来配置[基于密码的单一登录](../active-directory-appssoaccess-whatis.md)。 基于密码的 SSO 也称为密码保管，可用于管理不支持标识联合的 Web 应用程序中的用户访问权限和密码。 如果有多个用户需要共享单个帐户（例如共享组织的社交媒体应用帐户），此功能也很有用。 

## <a name="submit-the-request-in-the-portal"></a>在门户中提交请求

在测试你的应用程序集成可以使用 Azure AD 正常工作后，在[应用程序网络门户](https://microsoft.sharepoint.com/teams/apponboarding/Apps)上提交你的访问请求。 如果你有 Office 365 帐户，请使用该帐户登录到此门户。 如果没有，请使用 Microsoft 帐户（例如 Outlook 或 Hotmail）进行登录。

在登录后，将显示以下页面。 在文本框中提供需要进行访问的业务理由，然后选择“请求访问”。 我们的团队将审核详细信息并相应地为你提供访问权限。 之后，你可以登录到该门户并为应用程序提交详细请求。

如果你有关于访问的任何问题，请联系 [Azure AD SSO 集成团队](<mailto:SaaSApplicationIntegrations@service.microsoft.com>)。

![SharePoint 门户上的访问请求](./media/active-directory-app-gallery-listing/accessrequest.png)

## <a name="timelines"></a>时间线
    
在库中列出 SAML 2.0 或 WS-Fed 应用程序这一过程的时间线是 7-10 个工作日。

   ![将 SAML 应用程序列到库中的时间线](./media/active-directory-app-gallery-listing/timeline.png)

在库中列出 OpenID Connect 应用程序这一过程的时间线是 2-5 个工作日。

   ![将 SAML 应用程序列到库中的时间线](./media/active-directory-app-gallery-listing/timeline2.png)

## <a name="escalations"></a>升级

若要进行升级，请向 [Azure AD SSO 集成团队](<mailto:SaaSApplicationIntegrations@service.microsoft.com>)发送电子邮件，我们会尽快进行回复。

