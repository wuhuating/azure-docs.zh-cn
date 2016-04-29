﻿<properties title="了解如何配置使用流量管理器的 Azure 网站，以使用借助 Moniker 注册的域名" pageTitle="使用流量管理器为 Azure 网站配置 Moniker 域名" metaKeywords="Windows Azure, Windows Azure  Websites, Moniker, Traffic Manager" description="了解如何配置  使用流量管理器的 Azure 网站，以使用借助 Moniker 注册的域名" services="web-sites" documentationCenter="" authors="larryfr,jroth" />

# 使用流量管理器为 Azure 网站配置自定义域名 (Moniker)

<div class="dev-center-tutorial-selector sublanding"><a href="/zh-cn/documentation/articles/web-sites-custom-domain-name" title="Custom Domain">自定义域</a><a href="/zh-cn/documentation/articles/web-sites-godaddy-custom-domain-name" title="GoDaddy">GoDaddy</a><a href="/zh-cn/documentation/articles/web-sites-network-solutions-custom-domain-name" title="Network Solutions">Network Solutions</a><a href="/zh-cn/documentation/articles/web-sites-registerdotcom-custom-domain-name" title="Register.com">Register.com</a><a href="/zh-cn/documentation/articles/web-sites-enom-custom-domain-name" title="Enom">Enom</a><a href="/zh-cn/documentation/articles/web-sites-moniker-custom-domain-name" title="Moniker" class="current">Moniker</a><a href="/zh-cn/documentation/articles/web-sites-dotster-custom-domain-name" title="Dotster">Dotster</a><a href="/zh-cn/documentation/articles/web-sites-domaindiscover-custom-domain-name" title="DomainDiscover">DomainDiscover</a><a href="/zh-cn/documentation/articles/web-sites-directnic-custom-domain-name" title="Directnic">Directnic</a></div>
<div class="dev-center-tutorial-subselector"><a href="/zh-cn/documentation/articles/web-sites-moniker-custom-domain-name/" title=" Websites"> 网站</a> | <a href="/zh-cn/documentation/articles/web-sites-moniker-traffic-manager-custom-domain-name/" title=" Website using Traffic Manager" class="current"> 使用流量管理器的网站</a></div>

[WACOM.INCLUDE [websites-cloud-services-css-guided-walkthrough](../includes/websites-cloud-services-css-guided-walkthrough.md)]

[WACOM.INCLUDE [intro](../includes/custom-dns-web-site-intro-traffic-manager.md)]

本文提供了如何将从[Moniker ](https://moniker.com)购买的自定义域名用于 Azure 网站的说明。

[WACOM.INCLUDE [tmwebsitefooter](../includes/custom-dns-web-site-traffic-manager-notes.md)]

[WACOM.INCLUDE [introfooter](../includes/custom-dns-web-site-intro-notes.md)]

本文内容：

-   [了解 DNS 记录](#understanding-records)
-   [将网站配置为标准模式](#bkmk_configsharedmode)
-   [为自定义域添加 DNS 记录](#bkmk_configurecname)
-   [为网站启用流量管理器](#enabledomain)

<h2><a name="understanding-records"></a>了解 DNS 记录</h2>

[WACOM.INCLUDE [understandingdns](../includes/custom-dns-web-site-understanding-dns-traffic-manager.md)]

<h2><a name="bkmk_configsharedmode"></a>将网站配置为标准模式</h2>

[WACOM.INCLUDE [modes](../includes/custom-dns-web-site-modes-traffic-manager.md)]

<a name="bkmk_configurecname"></a><h2>为自定义域添加 DNS 记录</h2>

要将自定义域与 Azure 网站关联，必须通过使用 Moniker 提供的工具在 DNS 表中为自定义域添加新条目。使用以下步骤找到用于 Moniker.com 的 DNS 工具

1. 使用你的帐户登录 Moniker.com，选择**我的域**，然后选择**管理模板**。

    ![My Domains page for Moniker](.\media\web-sites-moniker-custom-domain-name\Moniker_MyDomains.png)

2. 在**区域模板管理**页面上，选择**创建新模板**。

    ![Moniker Zone Template Management](.\media\web-sites-moniker-custom-domain-name\Moniker_ZoneManager.png)

3. 填写**模板名称**。 

5. 然后通过先选择**记录类型**来创建一个 DNS 记录。然后填写**主机名称**和**地址**。 

    ![Moniker Create Zone Template](.\media\web-sites-moniker-custom-domain-name\Moniker_CreateZoneTemplate_TM.png)

    * 添加 CNAME 记录时，必须将**主机名称**字段设置为您要使用的子域。例如，**www**。必须将**地址**字段设置为用于 Azure 网站的流量管理器配置文件的**.trafficmanager.cn**域名。例如，**contoso.trafficmanager.cn**。

	    > [AZURE.NOTE] 在将你的自定义域名与使用流量管理器进行负载平衡的网站相关联时，只能使用 CNAME 记录。
	 
7. 单击**添加**按钮以添加条目。

8. 添加完所有条目后，单击**保存**按钮。

5. 选择**域管理器**以返回到你的域列表。

6. 选中你的目标域的复选框，然后再次单击**管理模板**。

7. 找到在前面步骤中创建的新模板。然后单击**将所选域(1)置于此模板中**链接。

    ![Moniker Create Zone Template](.\media\web-sites-moniker-custom-domain-name\Moniker_ZoneAssignment.png)

<h2><a name="enabledomain"></a>启用流量管理器网站</h2>

[WACOM.INCLUDE [modes](../includes/custom-dns-web-site-enable-on-traffic-manager.md)]