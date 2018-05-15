---
title: 通过多区域 N 层应用程序确保高可用性
description: 如何在 Azure 上的多个区域中部署 VM 以实现高可用性和复原能力。
author: MikeWasson
ms.date: 05/03/2018
pnp.series.title: Windows VM workloads
pnp.series.prev: n-tier
ms.openlocfilehash: 48943094e7847e39b9fdc4c3f71e27f2e6e41293
ms.sourcegitcommit: a5e549c15a948f6fb5cec786dbddc8578af3be66
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/06/2018
---
# <a name="multi-region-n-tier-application-for-high-availability"></a><span data-ttu-id="71b8a-103">通过多区域 N 层应用程序确保高可用性</span><span class="sxs-lookup"><span data-stu-id="71b8a-103">Multi-region N-tier application for high availability</span></span>

<span data-ttu-id="71b8a-104">此参考体系结构展示了在多个 Azure 区域中运行 N 层应用程序以实现可用性和强健的灾难恢复基础结构的一组经过实践检验的做法。</span><span class="sxs-lookup"><span data-stu-id="71b8a-104">This reference architecture shows a set of proven practices for running an N-tier application in multiple Azure regions, in order to achieve availability and a robust disaster recovery infrastructure.</span></span> 

<span data-ttu-id="71b8a-105">[![0]][0]</span><span class="sxs-lookup"><span data-stu-id="71b8a-105">[![0]][0]</span></span> 

<span data-ttu-id="71b8a-106">下载此体系结构的 [Visio 文件][visio-download]。</span><span class="sxs-lookup"><span data-stu-id="71b8a-106">*Download a [Visio file][visio-download] of this architecture.*</span></span>

## <a name="architecture"></a><span data-ttu-id="71b8a-107">体系结构</span><span class="sxs-lookup"><span data-stu-id="71b8a-107">Architecture</span></span> 

<span data-ttu-id="71b8a-108">此体系结构基于[使用 SQL Server 的 N 层应用程序](n-tier-sql-server.md)中所示的体系结构。</span><span class="sxs-lookup"><span data-stu-id="71b8a-108">This architecture builds on the one shown in [N-tier application with SQL Server](n-tier-sql-server.md).</span></span> 

* <span data-ttu-id="71b8a-109">**主要和次要区域**。</span><span class="sxs-lookup"><span data-stu-id="71b8a-109">**Primary and secondary regions**.</span></span> <span data-ttu-id="71b8a-110">使用两个区域来实现更高的可用性。</span><span class="sxs-lookup"><span data-stu-id="71b8a-110">Use two regions to achieve higher availability.</span></span> <span data-ttu-id="71b8a-111">其中一个是主要区域。</span><span class="sxs-lookup"><span data-stu-id="71b8a-111">One is the primary region.</span></span> <span data-ttu-id="71b8a-112">另一个区域用于故障转移。</span><span class="sxs-lookup"><span data-stu-id="71b8a-112">The other region is for failover.</span></span>

* <span data-ttu-id="71b8a-113">**Azure 流量管理器**。</span><span class="sxs-lookup"><span data-stu-id="71b8a-113">**Azure Traffic Manager**.</span></span> <span data-ttu-id="71b8a-114">[流量管理器][traffic-manager]将传入请求路由到其中一个区域。</span><span class="sxs-lookup"><span data-stu-id="71b8a-114">[Traffic Manager][traffic-manager] routes incoming requests to one of the regions.</span></span> <span data-ttu-id="71b8a-115">在正常运行期间，它将请求路由到主要区域。</span><span class="sxs-lookup"><span data-stu-id="71b8a-115">During normal operations, it routes requests to the primary region.</span></span> <span data-ttu-id="71b8a-116">如果该区域变得不可用，则流量管理器将故障转移到次要区域。</span><span class="sxs-lookup"><span data-stu-id="71b8a-116">If that region becomes unavailable, Traffic Manager fails over to the secondary region.</span></span> <span data-ttu-id="71b8a-117">有关详细信息，请参阅[流量管理器配置](#traffic-manager-configuration)部分。</span><span class="sxs-lookup"><span data-stu-id="71b8a-117">For more information, see the section [Traffic Manager configuration](#traffic-manager-configuration).</span></span>

* <span data-ttu-id="71b8a-118">**资源组**。</span><span class="sxs-lookup"><span data-stu-id="71b8a-118">**Resource groups**.</span></span> <span data-ttu-id="71b8a-119">为主要区域、次要区域和流量管理器创建单独的[资源组][resource groups]。</span><span class="sxs-lookup"><span data-stu-id="71b8a-119">Create separate [resource groups][resource groups] for the primary region, the secondary region, and for Traffic Manager.</span></span> <span data-ttu-id="71b8a-120">这允许你将每个区域作为单个资源集合灵活进行管理。</span><span class="sxs-lookup"><span data-stu-id="71b8a-120">This gives you the flexibility to manage each region as a single collection of resources.</span></span> <span data-ttu-id="71b8a-121">例如，可以重新部署一个区域而无需关闭另一个区域。</span><span class="sxs-lookup"><span data-stu-id="71b8a-121">For example, you could redeploy one region, without taking down the other one.</span></span> <span data-ttu-id="71b8a-122">[链接资源组][resource-group-links]，以便可以运行查询来列出应用程序的所有资源。</span><span class="sxs-lookup"><span data-stu-id="71b8a-122">[Link the resource groups][resource-group-links], so that you can run a query to list all the resources for the application.</span></span>

* <span data-ttu-id="71b8a-123">**Vnet**。</span><span class="sxs-lookup"><span data-stu-id="71b8a-123">**VNets**.</span></span> <span data-ttu-id="71b8a-124">为每个区域创建一个单独的 VNet。</span><span class="sxs-lookup"><span data-stu-id="71b8a-124">Create a separate VNet for each region.</span></span> <span data-ttu-id="71b8a-125">请确保地址空间不重叠。</span><span class="sxs-lookup"><span data-stu-id="71b8a-125">Make sure the address spaces do not overlap.</span></span> 

* <span data-ttu-id="71b8a-126">**SQL Server Always On 可用性组**。</span><span class="sxs-lookup"><span data-stu-id="71b8a-126">**SQL Server Always On Availability Group**.</span></span> <span data-ttu-id="71b8a-127">如果使用的是 SQL Server，建议使用 [SQL Always On 可用性组][sql-always-on]以实现高可用性。</span><span class="sxs-lookup"><span data-stu-id="71b8a-127">If you are using SQL Server, we recommend [SQL Always On Availability Groups][sql-always-on] for high availability.</span></span> <span data-ttu-id="71b8a-128">创建同时包含两个区域中的 SQL Server 实例的单个可用性组。</span><span class="sxs-lookup"><span data-stu-id="71b8a-128">Create a single availability group that includes the SQL Server instances in both regions.</span></span> 

    > [!NOTE]
    > <span data-ttu-id="71b8a-129">另外请考虑使用 [Azure SQL 数据库][azure-sql-db]，它以云服务形式提供关系数据库。</span><span class="sxs-lookup"><span data-stu-id="71b8a-129">Also consider [Azure SQL Database][azure-sql-db], which provides a relational database as a cloud service.</span></span> <span data-ttu-id="71b8a-130">使用 SQL 数据库，不需要配置可用性组或管理故障转移。</span><span class="sxs-lookup"><span data-stu-id="71b8a-130">With SQL Database, you don't need to configure an availability group or manage failover.</span></span>  
    > 

* <span data-ttu-id="71b8a-131">**VPN 网关**。</span><span class="sxs-lookup"><span data-stu-id="71b8a-131">**VPN Gateways**.</span></span> <span data-ttu-id="71b8a-132">在每个 VNet 中创建一个 [VPN 网关][vpn-gateway]，并配置一个 [VNet 到 VNet 连接][vnet-to-vnet]以在两个 VNet 之间实现网络流量传输。</span><span class="sxs-lookup"><span data-stu-id="71b8a-132">Create a [VPN gateway][vpn-gateway] in each VNet, and configure a [VNet-to-VNet connection][vnet-to-vnet], to enable network traffic between the two VNets.</span></span> <span data-ttu-id="71b8a-133">对于 SQL Always On 可用性组，这是必需的。</span><span class="sxs-lookup"><span data-stu-id="71b8a-133">This is required for the SQL Always On Availability Group.</span></span>

## <a name="recommendations"></a><span data-ttu-id="71b8a-134">建议</span><span class="sxs-lookup"><span data-stu-id="71b8a-134">Recommendations</span></span>

<span data-ttu-id="71b8a-135">与部署到单个区域相比，多区域体系结构可以提供更高的可用性。</span><span class="sxs-lookup"><span data-stu-id="71b8a-135">A multi-region architecture can provide higher availability than deploying to a single region.</span></span> <span data-ttu-id="71b8a-136">如果区域性故障影响了主要区域，则可以使用[流量管理器][traffic-manager]故障转移到次要区域。</span><span class="sxs-lookup"><span data-stu-id="71b8a-136">If a regional outage affects the primary region, you can use [Traffic Manager][traffic-manager] to fail over to the secondary region.</span></span> <span data-ttu-id="71b8a-137">当应用程序的单个子系统出现故障时，此体系结构可能也比较有用。</span><span class="sxs-lookup"><span data-stu-id="71b8a-137">This architecture can also help if an individual subsystem of the application fails.</span></span>

<span data-ttu-id="71b8a-138">有多种常规方法可跨区域实现高可用性：</span><span class="sxs-lookup"><span data-stu-id="71b8a-138">There are several general approaches to achieving high availability across regions:</span></span> 

* <span data-ttu-id="71b8a-139">主动/被动（采用热备用模式）。</span><span class="sxs-lookup"><span data-stu-id="71b8a-139">Active/passive with hot standby.</span></span> <span data-ttu-id="71b8a-140">流量将前往一个区域，而另一个区域将以热备用模式等待。</span><span class="sxs-lookup"><span data-stu-id="71b8a-140">Traffic goes to one region, while the other waits on hot standby.</span></span> <span data-ttu-id="71b8a-141">“热备用模式”意味着次要区域中的 VM 已被分配并总是处于运行状态。</span><span class="sxs-lookup"><span data-stu-id="71b8a-141">Hot standby means the VMs in the secondary region are allocated and running at all times.</span></span>
* <span data-ttu-id="71b8a-142">主动/被动（采用冷备用模式）。</span><span class="sxs-lookup"><span data-stu-id="71b8a-142">Active/passive with cold standby.</span></span> <span data-ttu-id="71b8a-143">流量将前往一个区域，而另一个区域将以冷备用模式等待。</span><span class="sxs-lookup"><span data-stu-id="71b8a-143">Traffic goes to one region, while the other waits on cold standby.</span></span> <span data-ttu-id="71b8a-144">“冷备用模式”意味着次要区域中的 VM 不会被分配，直到故障转移需要它们。</span><span class="sxs-lookup"><span data-stu-id="71b8a-144">Cold standby means the VMs in the secondary region are not allocated until needed for failover.</span></span> <span data-ttu-id="71b8a-145">此方法的运行成本较低，但是当发生故障时通常需要花费更长时间才能联机。</span><span class="sxs-lookup"><span data-stu-id="71b8a-145">This approach costs less to run, but will generally take longer to come online during a failure.</span></span>
* <span data-ttu-id="71b8a-146">主动/主动。</span><span class="sxs-lookup"><span data-stu-id="71b8a-146">Active/active.</span></span> <span data-ttu-id="71b8a-147">两个区域都处于活动状态，并且会在它们之间对请求进行负载均衡。</span><span class="sxs-lookup"><span data-stu-id="71b8a-147">Both regions are active, and requests are load balanced between them.</span></span> <span data-ttu-id="71b8a-148">如果一个区域变得不可用，则不再使其参与轮换。</span><span class="sxs-lookup"><span data-stu-id="71b8a-148">If one region becomes unavailable, it is taken out of rotation.</span></span> 

<span data-ttu-id="71b8a-149">此参考体系结构侧重于“主动/被动（采用热备用模式）”，使用流量管理器进行故障转移。</span><span class="sxs-lookup"><span data-stu-id="71b8a-149">This reference architecture focuses on active/passive with hot standby, using Traffic Manager for failover.</span></span> <span data-ttu-id="71b8a-150">注意，可以为热备用模式部署少量 VM，然后根据需要横向扩展。</span><span class="sxs-lookup"><span data-stu-id="71b8a-150">Note that you could deploy a small number of VMs for hot standby and then scale out as needed.</span></span>

### <a name="regional-pairing"></a><span data-ttu-id="71b8a-151">区域配对</span><span class="sxs-lookup"><span data-stu-id="71b8a-151">Regional pairing</span></span>

<span data-ttu-id="71b8a-152">每个 Azure 区域都与同一地域内的另一个区域配对。</span><span class="sxs-lookup"><span data-stu-id="71b8a-152">Each Azure region is paired with another region within the same geography.</span></span> <span data-ttu-id="71b8a-153">通常，请选择同一区域对中的区域（例如“美国东部 2”和“美国中部”）。</span><span class="sxs-lookup"><span data-stu-id="71b8a-153">In general, choose regions from the same regional pair (for example, East US 2 and US Central).</span></span> <span data-ttu-id="71b8a-154">这样做的好处包括：</span><span class="sxs-lookup"><span data-stu-id="71b8a-154">Benefits of doing so include:</span></span>

* <span data-ttu-id="71b8a-155">如果发生大范围的故障，会优先恢复每个区域对中的至少一个区域。</span><span class="sxs-lookup"><span data-stu-id="71b8a-155">If there is a broad outage, recovery of at least one region out of every pair is prioritized.</span></span>
* <span data-ttu-id="71b8a-156">计划内 Azure 系统更新会按顺序提供给配对的区域，以尽可能减少停机时间。</span><span class="sxs-lookup"><span data-stu-id="71b8a-156">Planned Azure system updates are rolled out to paired regions sequentially, to minimize possible downtime.</span></span>
* <span data-ttu-id="71b8a-157">区域对位于同一地域内，以满足数据驻留要求。</span><span class="sxs-lookup"><span data-stu-id="71b8a-157">Pairs reside within the same geography, to meet data residency requirements.</span></span> 

<span data-ttu-id="71b8a-158">但请确保两个区域都支持应用程序所需的所有 Azure 服务（请参阅[每个区域的服务][services-by-region]）。</span><span class="sxs-lookup"><span data-stu-id="71b8a-158">However, make sure that both regions support all of the Azure services needed for your application (see [Services by region][services-by-region]).</span></span> <span data-ttu-id="71b8a-159">有关区域对的详细信息，请参阅[业务连续性和灾难恢复 (BCDR)：Azure 配对区域][regional-pairs]。</span><span class="sxs-lookup"><span data-stu-id="71b8a-159">For more information about regional pairs, see [Business continuity and disaster recovery (BCDR): Azure Paired Regions][regional-pairs].</span></span>

### <a name="traffic-manager-configuration"></a><span data-ttu-id="71b8a-160">流量管理器配置</span><span class="sxs-lookup"><span data-stu-id="71b8a-160">Traffic Manager configuration</span></span>

<span data-ttu-id="71b8a-161">配置流量管理器时，请考虑以下几点：</span><span class="sxs-lookup"><span data-stu-id="71b8a-161">Consider the following points when configuring Traffic Manager:</span></span>

* <span data-ttu-id="71b8a-162">**路由**。</span><span class="sxs-lookup"><span data-stu-id="71b8a-162">**Routing**.</span></span> <span data-ttu-id="71b8a-163">流量管理器支持多个[路由算法][tm-routing]。</span><span class="sxs-lookup"><span data-stu-id="71b8a-163">Traffic Manager supports several [routing algorithms][tm-routing].</span></span> <span data-ttu-id="71b8a-164">对于本文中所述的情况，请使用“优先级”路由（以前称为“故障转移”路由）。</span><span class="sxs-lookup"><span data-stu-id="71b8a-164">For the scenario described in this article, use *priority* routing (formerly called *failover* routing).</span></span> <span data-ttu-id="71b8a-165">使用此设置时，流量管理器将所有请求都发送到主要区域，除非主要区域变得无法访问。</span><span class="sxs-lookup"><span data-stu-id="71b8a-165">With this setting, Traffic Manager sends all requests to the primary region, unless the primary region becomes unreachable.</span></span> <span data-ttu-id="71b8a-166">那时，它将自动故障转移到次要区域。</span><span class="sxs-lookup"><span data-stu-id="71b8a-166">At that point, it automatically fails over to the secondary region.</span></span> <span data-ttu-id="71b8a-167">请参阅[配置故障转移路由方法][tm-configure-failover]。</span><span class="sxs-lookup"><span data-stu-id="71b8a-167">See [Configure Failover routing method][tm-configure-failover].</span></span>
* <span data-ttu-id="71b8a-168">**运行状况探测**。</span><span class="sxs-lookup"><span data-stu-id="71b8a-168">**Health probe**.</span></span> <span data-ttu-id="71b8a-169">流量管理器使用 HTTP（或 HTTPS）[探测][tm-monitoring]来监视每个区域的可用性。</span><span class="sxs-lookup"><span data-stu-id="71b8a-169">Traffic Manager uses an HTTP (or HTTPS) [probe][tm-monitoring] to monitor the availability of each region.</span></span> <span data-ttu-id="71b8a-170">探测检查指定 URL 路径的 HTTP 200 响应。</span><span class="sxs-lookup"><span data-stu-id="71b8a-170">The probe checks for an HTTP 200 response for a specified URL path.</span></span> <span data-ttu-id="71b8a-171">作为最佳做法，请创建一个用于报告应用程序整体运行状况的终结点，并使用此终结点进行运行状况探测。</span><span class="sxs-lookup"><span data-stu-id="71b8a-171">As a best practice, create an endpoint that reports the overall health of the application, and use this endpoint for the health probe.</span></span> <span data-ttu-id="71b8a-172">否则，探测可能会在应用程序的关键部分实际上已出现故障时报告终结点运行状况正常。</span><span class="sxs-lookup"><span data-stu-id="71b8a-172">Otherwise, the probe might report a healthy endpoint when critical parts of the application are actually failing.</span></span> <span data-ttu-id="71b8a-173">有关详细信息，请参阅[运行状况终结点监视模式][health-endpoint-monitoring-pattern]。</span><span class="sxs-lookup"><span data-stu-id="71b8a-173">For more information, see [Health Endpoint Monitoring Pattern][health-endpoint-monitoring-pattern].</span></span>   

<span data-ttu-id="71b8a-174">当流量管理器进行故障转移时，一段时间内客户端将无法访问应用程序。</span><span class="sxs-lookup"><span data-stu-id="71b8a-174">When Traffic Manager fails over there is a period of time when clients cannot reach the application.</span></span> <span data-ttu-id="71b8a-175">持续时间受以下因素影响：</span><span class="sxs-lookup"><span data-stu-id="71b8a-175">The duration is affected by the following factors:</span></span>

* <span data-ttu-id="71b8a-176">运行状况探测必须检测主要区域是否变得无法访问。</span><span class="sxs-lookup"><span data-stu-id="71b8a-176">The health probe must detect that the primary region has become unreachable.</span></span>
* <span data-ttu-id="71b8a-177">DNS 服务器必须更新 IP 地址的已缓存 DNS 记录，这取决于 DNS 生存时间 (TTL)。</span><span class="sxs-lookup"><span data-stu-id="71b8a-177">DNS servers must update the cached DNS records for the IP address, which depends on the DNS time-to-live (TTL).</span></span> <span data-ttu-id="71b8a-178">默认 TTL 为 300 秒（5 分钟），但可以在创建流量管理器配置文件时配置此值。</span><span class="sxs-lookup"><span data-stu-id="71b8a-178">The default TTL is 300 seconds (5 minutes), but you can configure this value when you create the Traffic Manager profile.</span></span>

<span data-ttu-id="71b8a-179">相关详细信息，请参阅[关于流量管理器监视][tm-monitoring]。</span><span class="sxs-lookup"><span data-stu-id="71b8a-179">For details, see [About Traffic Manager Monitoring][tm-monitoring].</span></span>

<span data-ttu-id="71b8a-180">如果流量管理器进行故障转移，我们建议执行手动故障回复，而不是实施自动故障回复。</span><span class="sxs-lookup"><span data-stu-id="71b8a-180">If Traffic Manager fails over, we recommend performing a manual failback rather than implementing an automatic failback.</span></span> <span data-ttu-id="71b8a-181">否则，可能会造成应用程序在区域之间来回转移的情况。</span><span class="sxs-lookup"><span data-stu-id="71b8a-181">Otherwise, you can create a situation where the application flips back and forth between regions.</span></span> <span data-ttu-id="71b8a-182">在进行故障回复之前，请验证是否所有应用程序子系统的运行状况都正常。</span><span class="sxs-lookup"><span data-stu-id="71b8a-182">Verify that all application subsystems are healthy before failing back.</span></span>

<span data-ttu-id="71b8a-183">请注意，默认情况下，流量管理器会自动进行故障回复。</span><span class="sxs-lookup"><span data-stu-id="71b8a-183">Note that Traffic Manager automatically fails back by default.</span></span> <span data-ttu-id="71b8a-184">若要禁止此操作，请在发生故障转移事件后手动降低主要区域的优先级。</span><span class="sxs-lookup"><span data-stu-id="71b8a-184">To prevent this, manually lower the priority of the primary region after a failover event.</span></span> <span data-ttu-id="71b8a-185">例如，假设主要区域的优先级为 1，次要区域的优先级为 2。</span><span class="sxs-lookup"><span data-stu-id="71b8a-185">For example, suppose the primary region is priority 1 and the secondary is priority 2.</span></span> <span data-ttu-id="71b8a-186">在故障转移后，请将主要区域的优先级设置为 3，以禁止自动故障回复。</span><span class="sxs-lookup"><span data-stu-id="71b8a-186">After a failover, set the primary region to priority 3, to prevent automatic failback.</span></span> <span data-ttu-id="71b8a-187">当准备好切换回来时，请将优先级更新为 1。</span><span class="sxs-lookup"><span data-stu-id="71b8a-187">When you are ready to switch back, update the priority to 1.</span></span>

<span data-ttu-id="71b8a-188">以下 [Azure CLI][install-azure-cli] 命令更新优先级：</span><span class="sxs-lookup"><span data-stu-id="71b8a-188">The following [Azure CLI][install-azure-cli] command updates the priority:</span></span>

```bat
azure network traffic-manager  endpoint set --resource-group <resource-group> --profile-name <profile>
    --name <traffic-manager-name> --type AzureEndpoints --priority 3
```    

<span data-ttu-id="71b8a-189">另一种方法是暂时禁用终结点，直到你准备好进行故障回复：</span><span class="sxs-lookup"><span data-stu-id="71b8a-189">Another approach is to temporarily disable the endpoint until you are ready to fail back:</span></span>

```bat
azure network traffic-manager  endpoint set --resource-group <resource-group> --profile-name <profile>
    --name <traffic-manager-name> --type AzureEndpoints --status Disabled
```

<span data-ttu-id="71b8a-190">可能需要重新部署某个区域中的资源，具体取决于故障转移原因。</span><span class="sxs-lookup"><span data-stu-id="71b8a-190">Depending on the cause of a failover, you might need to redeploy the resources within a region.</span></span> <span data-ttu-id="71b8a-191">在进行故障回复之前，请执行操作准备情况测试。</span><span class="sxs-lookup"><span data-stu-id="71b8a-191">Before failing back, perform an operational readiness test.</span></span> <span data-ttu-id="71b8a-192">测试应当验证的事项如下所示：</span><span class="sxs-lookup"><span data-stu-id="71b8a-192">The test should verify things like:</span></span>

* <span data-ttu-id="71b8a-193">VM 是否已正确配置。</span><span class="sxs-lookup"><span data-stu-id="71b8a-193">VMs are configured correctly.</span></span> <span data-ttu-id="71b8a-194">（所有必需的软件是否已安装、IIS 是否正在运行，等等。）</span><span class="sxs-lookup"><span data-stu-id="71b8a-194">(All required software is installed, IIS is running, and so on.)</span></span>
* <span data-ttu-id="71b8a-195">应用程序子系统运行状况是否正常。</span><span class="sxs-lookup"><span data-stu-id="71b8a-195">Application subsystems are healthy.</span></span> 
* <span data-ttu-id="71b8a-196">功能测试。</span><span class="sxs-lookup"><span data-stu-id="71b8a-196">Functional testing.</span></span> <span data-ttu-id="71b8a-197">（例如，是否可以从 Web 层访问数据库层。）</span><span class="sxs-lookup"><span data-stu-id="71b8a-197">(For example, the database tier is reachable from the web tier.)</span></span>

### <a name="configure-sql-server-always-on-availability-groups"></a><span data-ttu-id="71b8a-198">配置 SQL Server Always On 可用性组</span><span class="sxs-lookup"><span data-stu-id="71b8a-198">Configure SQL Server Always On Availability Groups</span></span>

<span data-ttu-id="71b8a-199">在 Windows Server 2016 之前，SQL Server Always On 可用性组需要一个域控制器，并且可用性组中的所有节点必须在同一 Active Directory (AD) 域中。</span><span class="sxs-lookup"><span data-stu-id="71b8a-199">Prior to Windows Server 2016, SQL Server Always On Availability Groups require a domain controller, and all nodes in the availability group must be in the same Active Directory (AD) domain.</span></span> 

<span data-ttu-id="71b8a-200">若要配置可用性组，请执行以下操作：</span><span class="sxs-lookup"><span data-stu-id="71b8a-200">To configure the availability group:</span></span>

* <span data-ttu-id="71b8a-201">至少在每个区域中放置两个域控制器。</span><span class="sxs-lookup"><span data-stu-id="71b8a-201">At a minimum, place two domain controllers in each region.</span></span>
* <span data-ttu-id="71b8a-202">为每个域控制器提供一个静态 IP 地址。</span><span class="sxs-lookup"><span data-stu-id="71b8a-202">Give each domain controller a static IP address.</span></span>
* <span data-ttu-id="71b8a-203">创建 VNet 到 VNet 连接，以允许在 Vnet 之间进行通信。</span><span class="sxs-lookup"><span data-stu-id="71b8a-203">Create a VNet-to-VNet connection to enable communication between the VNets.</span></span>
* <span data-ttu-id="71b8a-204">针对每个 VNet，将两个区域中的域控制器的 IP 地址添加到 DNS 服务器列表。</span><span class="sxs-lookup"><span data-stu-id="71b8a-204">For each VNet, add the IP addresses of the domain controllers (from both regions) to the DNS server list.</span></span> <span data-ttu-id="71b8a-205">可以使用以下 CLI 命令。</span><span class="sxs-lookup"><span data-stu-id="71b8a-205">You can use the following CLI command.</span></span> <span data-ttu-id="71b8a-206">有关详细信息，请参阅[管理虚拟网络 (VNet) 使用的 DNS 服务器][vnet-dns]。</span><span class="sxs-lookup"><span data-stu-id="71b8a-206">For more information, see [Manage DNS servers used by a virtual network (VNet)][vnet-dns].</span></span>

    ```bat
    azure network vnet set --resource-group dc01-rg --name dc01-vnet --dns-servers "10.0.0.4,10.0.0.6,172.16.0.4,172.16.0.6"
    ```

* <span data-ttu-id="71b8a-207">创建一个 [Windows Server 故障转移群集][wsfc] (WSFC) 群集，使其包括两个区域中的 SQL Server 实例。</span><span class="sxs-lookup"><span data-stu-id="71b8a-207">Create a [Windows Server Failover Clustering][wsfc] (WSFC) cluster that includes the SQL Server instances in both regions.</span></span> 
* <span data-ttu-id="71b8a-208">创建一个 SQL Server Always On 可用性组，使其包括主要区域和次要区域中的 SQL Server 实例。</span><span class="sxs-lookup"><span data-stu-id="71b8a-208">Create a SQL Server Always On Availability Group that includes the SQL Server instances in both the primary and secondary regions.</span></span> <span data-ttu-id="71b8a-209">有关步骤，请参阅[将 Always On 可用性组扩展到远程 Azure 数据中心 (PowerShell)](https://blogs.msdn.microsoft.com/sqlcat/2014/09/22/extending-alwayson-availability-group-to-remote-azure-datacenter-powershell/)。</span><span class="sxs-lookup"><span data-stu-id="71b8a-209">See [Extending Always On Availability Group to Remote Azure Datacenter (PowerShell)](https://blogs.msdn.microsoft.com/sqlcat/2014/09/22/extending-alwayson-availability-group-to-remote-azure-datacenter-powershell/) for the steps.</span></span>

  * <span data-ttu-id="71b8a-210">将主要副本放置在主要区域中。</span><span class="sxs-lookup"><span data-stu-id="71b8a-210">Put the primary replica in the primary region.</span></span>
  * <span data-ttu-id="71b8a-211">将一个或多个次要副本放置在主要区域中。</span><span class="sxs-lookup"><span data-stu-id="71b8a-211">Put one or more secondary replicas in the primary region.</span></span> <span data-ttu-id="71b8a-212">对它们进行配置以将同步提交与自动故障转移一起使用。</span><span class="sxs-lookup"><span data-stu-id="71b8a-212">Configure these to use synchronous commit with automatic failover.</span></span>
  * <span data-ttu-id="71b8a-213">将一个或多个次要副本放置在次要区域中。</span><span class="sxs-lookup"><span data-stu-id="71b8a-213">Put one or more secondary replicas in the secondary region.</span></span> <span data-ttu-id="71b8a-214">出于性能方面的原因，请对它们进行配置以使用*异步*提交。</span><span class="sxs-lookup"><span data-stu-id="71b8a-214">Configure these to use *asynchronous* commit, for performance reasons.</span></span> <span data-ttu-id="71b8a-215">（否则，所有 T-SQL 事务必须等待通过网络到次要区域的往返旅程。）</span><span class="sxs-lookup"><span data-stu-id="71b8a-215">(Otherwise, all T-SQL transactions have to wait on a round trip over the network to the secondary region.)</span></span>

    > [!NOTE]
    > <span data-ttu-id="71b8a-216">异步提交副本不支持自动故障转移。</span><span class="sxs-lookup"><span data-stu-id="71b8a-216">Asynchronous commit replicas do not support automatic failover.</span></span>
    >
    >

## <a name="availability-considerations"></a><span data-ttu-id="71b8a-217">可用性注意事项</span><span class="sxs-lookup"><span data-stu-id="71b8a-217">Availability considerations</span></span>

<span data-ttu-id="71b8a-218">使用复杂的 N 层应用时，可能不需要在次要区域中复制整个应用程序。</span><span class="sxs-lookup"><span data-stu-id="71b8a-218">With a complex N-tier app, you may not need to replicate the entire application in the secondary region.</span></span> <span data-ttu-id="71b8a-219">相反，可能只需复制支持业务连续性所需的关键子系统。</span><span class="sxs-lookup"><span data-stu-id="71b8a-219">Instead, you might just replicate a critical subsystem that is needed to support business continuity.</span></span>

<span data-ttu-id="71b8a-220">流量管理器是系统中的一个潜在故障点。</span><span class="sxs-lookup"><span data-stu-id="71b8a-220">Traffic Manager is a possible failure point in the system.</span></span> <span data-ttu-id="71b8a-221">如果流量管理器服务出现故障，则客户端在停机期间无法访问应用程序。</span><span class="sxs-lookup"><span data-stu-id="71b8a-221">If the Traffic Manager service fails, clients cannot access your application during the downtime.</span></span> <span data-ttu-id="71b8a-222">查看[流量管理器 SLA][tm-sla]，然后确定仅使用流量管理器是否能满足业务对高可用性的需求。</span><span class="sxs-lookup"><span data-stu-id="71b8a-222">Review the [Traffic Manager SLA][tm-sla], and determine whether using Traffic Manager alone meets your business requirements for high availability.</span></span> <span data-ttu-id="71b8a-223">如果不能，请考虑添加另一个流量管理解决方案作为故障回复机制。</span><span class="sxs-lookup"><span data-stu-id="71b8a-223">If not, consider adding another traffic management solution as a failback.</span></span> <span data-ttu-id="71b8a-224">如果 Azure 流量管理器服务出现故障，请将 DNS 中的 CNAME 记录更改为指向其他流量管理服务。</span><span class="sxs-lookup"><span data-stu-id="71b8a-224">If the Azure Traffic Manager service fails, change your CNAME records in DNS to point to the other traffic management service.</span></span> <span data-ttu-id="71b8a-225">（此步骤必须手动执行，并且在 DNS 更改被传播之前，应用程序将不可用。）</span><span class="sxs-lookup"><span data-stu-id="71b8a-225">(This step must be performed manually, and your application will be unavailable until the DNS changes are propagated.)</span></span>

<span data-ttu-id="71b8a-226">对于 SQL Server 群集，有两个故障转移方案需要考虑：</span><span class="sxs-lookup"><span data-stu-id="71b8a-226">For the SQL Server cluster, there are two failover scenarios to consider:</span></span>

- <span data-ttu-id="71b8a-227">主要区域中的所有 SQL Server 数据库副本都失败。</span><span class="sxs-lookup"><span data-stu-id="71b8a-227">All of the SQL Server database replicas in the primary region fail.</span></span> <span data-ttu-id="71b8a-228">例如，在发生区域性中断期间可能会出现此情况。</span><span class="sxs-lookup"><span data-stu-id="71b8a-228">For example, this could happen during a regional outage.</span></span> <span data-ttu-id="71b8a-229">在这种情况下，必须手动故障转移可用性组，尽管流量管理器在前端会自动进行故障转移。</span><span class="sxs-lookup"><span data-stu-id="71b8a-229">In that case, you must manually fail over the availability group, even though Traffic Manager automatically fails over on the front end.</span></span> <span data-ttu-id="71b8a-230">请按照[执行可用性组的强制手动故障转移 (SQL Server)](https://msdn.microsoft.com/library/ff877957.aspx) 一文中的步骤进行操作，该文章介绍了如何在 SQL Server 2016 中使用 SQL Server Management Studio、Transact-SQL 或 PowerShell 执行强制故障转移。</span><span class="sxs-lookup"><span data-stu-id="71b8a-230">Follow the steps in [Perform a Forced Manual Failover of a SQL Server Availability Group](https://msdn.microsoft.com/library/ff877957.aspx), which describes how to perform a forced failover by using SQL Server Management Studio, Transact-SQL, or PowerShell in SQL Server 2016.</span></span>

   > [!WARNING]
   > <span data-ttu-id="71b8a-231">使用强制故障转移时存在数据丢失风险。</span><span class="sxs-lookup"><span data-stu-id="71b8a-231">With forced failover, there is a risk of data loss.</span></span> <span data-ttu-id="71b8a-232">在主要区域恢复联机状态后，创建数据库快照并使用 [tablediff] 查明差异。</span><span class="sxs-lookup"><span data-stu-id="71b8a-232">Once the primary region is back online, take a snapshot of the database and use [tablediff] to find the differences.</span></span>
   >
   >
- <span data-ttu-id="71b8a-233">流量管理器故障转移到次要区域，但主要 SQL Server 数据库副本仍然可用。</span><span class="sxs-lookup"><span data-stu-id="71b8a-233">Traffic Manager fails over to the secondary region, but the primary SQL Server database replica is still available.</span></span> <span data-ttu-id="71b8a-234">例如，前端层可能会失败，但不会影响 SQL Server VM。</span><span class="sxs-lookup"><span data-stu-id="71b8a-234">For example, the front-end tier might fail, without affecting the SQL Server VMs.</span></span> <span data-ttu-id="71b8a-235">在这种情况下，Internet 流量将路由到次要区域中，并且该区域仍可以连接到主要副本。</span><span class="sxs-lookup"><span data-stu-id="71b8a-235">In that case, Internet traffic is routed to the secondary region, and that region can still connect to the primary replica.</span></span> <span data-ttu-id="71b8a-236">但是，延迟将有所增加，因为 SQL Server 连接是跨区域的。</span><span class="sxs-lookup"><span data-stu-id="71b8a-236">However, there will be increased latency, because the SQL Server connections are going across regions.</span></span> <span data-ttu-id="71b8a-237">在此情况下，应当执行手动故障转移，如下所述：</span><span class="sxs-lookup"><span data-stu-id="71b8a-237">In this situation, you should perform a manual failover as follows:</span></span>

   1. <span data-ttu-id="71b8a-238">暂时将次要区域中的 SQL Server 数据库副本切换为*同步*提交。</span><span class="sxs-lookup"><span data-stu-id="71b8a-238">Temporarily switch a SQL Server database replica in the secondary region to *synchronous* commit.</span></span> <span data-ttu-id="71b8a-239">这可确保在故障转移期间不会丢失数据。</span><span class="sxs-lookup"><span data-stu-id="71b8a-239">This ensures there won't be data loss during the failover.</span></span>
   2. <span data-ttu-id="71b8a-240">故障转移到该副本。</span><span class="sxs-lookup"><span data-stu-id="71b8a-240">Fail over to that replica.</span></span>
   3. <span data-ttu-id="71b8a-241">在故障回复到主要区域后，还原异步提交设置。</span><span class="sxs-lookup"><span data-stu-id="71b8a-241">When you fail back to the primary region, restore the asynchronous commit setting.</span></span>

## <a name="manageability-considerations"></a><span data-ttu-id="71b8a-242">可管理性注意事项</span><span class="sxs-lookup"><span data-stu-id="71b8a-242">Manageability considerations</span></span>

<span data-ttu-id="71b8a-243">更新部署时，请一次更新一个区域，以减少由于错误配置或应用程序中的错误而导致全局故障的可能性。</span><span class="sxs-lookup"><span data-stu-id="71b8a-243">When you update your deployment, update one region at a time to reduce the chance of a global failure from an incorrect configuration or an error in the application.</span></span>

<span data-ttu-id="71b8a-244">测试系统在发生故障时的复原能力。</span><span class="sxs-lookup"><span data-stu-id="71b8a-244">Test the resiliency of the system to failures.</span></span> <span data-ttu-id="71b8a-245">下面是要测试的一些常见故障方案：</span><span class="sxs-lookup"><span data-stu-id="71b8a-245">Here are some common failure scenarios to test:</span></span>

* <span data-ttu-id="71b8a-246">关闭 VM 实例。</span><span class="sxs-lookup"><span data-stu-id="71b8a-246">Shut down VM instances.</span></span>
* <span data-ttu-id="71b8a-247">对 CPU 和内存等资源进行压力测试。</span><span class="sxs-lookup"><span data-stu-id="71b8a-247">Pressure resources such as CPU and memory.</span></span>
* <span data-ttu-id="71b8a-248">断开网络连接/使网络传输出现延迟。</span><span class="sxs-lookup"><span data-stu-id="71b8a-248">Disconnect/delay network.</span></span>
* <span data-ttu-id="71b8a-249">使进程崩溃。</span><span class="sxs-lookup"><span data-stu-id="71b8a-249">Crash processes.</span></span>
* <span data-ttu-id="71b8a-250">使证书过期。</span><span class="sxs-lookup"><span data-stu-id="71b8a-250">Expire certificates.</span></span>
* <span data-ttu-id="71b8a-251">模拟硬件错误。</span><span class="sxs-lookup"><span data-stu-id="71b8a-251">Simulate hardware faults.</span></span>
* <span data-ttu-id="71b8a-252">关闭域控制器上的 DNS 服务。</span><span class="sxs-lookup"><span data-stu-id="71b8a-252">Shut down the DNS service on the domain controllers.</span></span>

<span data-ttu-id="71b8a-253">测量恢复时间，并验证它们是否满足你的业务要求。</span><span class="sxs-lookup"><span data-stu-id="71b8a-253">Measure the recovery times and verify they meet your business requirements.</span></span> <span data-ttu-id="71b8a-254">另外还测试故障模式的组合。</span><span class="sxs-lookup"><span data-stu-id="71b8a-254">Test combinations of failure modes, as well.</span></span>



<!-- Links -->
[hybrid-vpn]: ../hybrid-networking/vpn.md
[azure-dns]: /azure/dns/dns-overview
[azure-sla]: https://azure.microsoft.com/support/legal/sla/
[azure-sql-db]: https://azure.microsoft.com/documentation/services/sql-database/
[health-endpoint-monitoring-pattern]: https://msdn.microsoft.com/library/dn589789.aspx
[install-azure-cli]: /azure/xplat-cli-install
[regional-pairs]: /azure/best-practices-availability-paired-regions
[resource groups]: /azure/azure-resource-manager/resource-group-overview
[resource-group-links]: /azure/resource-group-link-resources
[resource-manager-overview]: /azure/azure-resource-manager/resource-group-overview
[services-by-region]: https://azure.microsoft.com/regions/#services
[sql-always-on]: https://msdn.microsoft.com/library/hh510230.aspx
[tablediff]: https://msdn.microsoft.com/library/ms162843.aspx
[tm-configure-failover]: /azure/traffic-manager/traffic-manager-configure-failover-routing-method
[tm-monitoring]: /azure/traffic-manager/traffic-manager-monitoring
[tm-routing]: /azure/traffic-manager/traffic-manager-routing-methods
[tm-sla]: https://azure.microsoft.com/support/legal/sla/traffic-manager/v1_0/
[traffic-manager]: https://azure.microsoft.com/services/traffic-manager/
[visio-download]: https://archcenter.blob.core.windows.net/cdn/vm-reference-architectures.vsdx
[vnet-dns]: /azure/virtual-network/virtual-networks-manage-dns-in-vnet
[vnet-to-vnet]: /azure/vpn-gateway/vpn-gateway-vnet-vnet-rm-ps
[vpn-gateway]: /azure/vpn-gateway/vpn-gateway-about-vpngateways
[wsfc]: https://msdn.microsoft.com/library/hh270278.aspx

[0]: ./images/multi-region-sql-server.png "Azure N 层应用程序的高可用性网络体系结构"
