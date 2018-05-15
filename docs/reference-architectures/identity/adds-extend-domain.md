---
title: 将 Active Directory 域服务 (AD DS) 扩展到 Azure
description: 将本地 Active Directory 域扩展到 Azure
author: telmosampaio
ms.date: 05/02/2018
pnp.series.title: Identity management
pnp.series.prev: azure-ad
pnp.series.next: adds-forest
ms.openlocfilehash: 763fffd321a1b50a562ef462dab59aafae717908
ms.sourcegitcommit: 0de300b6570e9990e5c25efc060946cb9d079954
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/03/2018
---
# <a name="extend-active-directory-domain-services-ad-ds-to-azure"></a><span data-ttu-id="4c870-103">将 Active Directory 域服务 (AD DS) 扩展到 Azure</span><span class="sxs-lookup"><span data-stu-id="4c870-103">Extend Active Directory Domain Services (AD DS) to Azure</span></span>

<span data-ttu-id="4c870-104">此参考体系结构展示了如何使用 Active Directory 域服务 (AD DS) 将 Active Directory 环境扩展到 Azure 以提供分布式身份验证服务。</span><span class="sxs-lookup"><span data-stu-id="4c870-104">This reference architecture shows how to extend your Active Directory environment to Azure to provide distributed authentication services using Active Directory Domain Services (AD DS).</span></span> [<span data-ttu-id="4c870-105">**部署此解决方案**。</span><span class="sxs-lookup"><span data-stu-id="4c870-105">**Deploy this solution**.</span></span>](#deploy-the-solution)

<span data-ttu-id="4c870-106">[![0]][0]</span><span class="sxs-lookup"><span data-stu-id="4c870-106">[![0]][0]</span></span> 

<span data-ttu-id="4c870-107">下载此体系结构的 [Visio 文件][visio-download]。</span><span class="sxs-lookup"><span data-stu-id="4c870-107">*Download a [Visio file][visio-download] of this architecture.*</span></span>

<span data-ttu-id="4c870-108">AD DS 用来对安全域中包括的用户、计算机、应用程序或其他标识进行身份验证。</span><span class="sxs-lookup"><span data-stu-id="4c870-108">AD DS is used to authenticate user, computer, application, or other identities that are included in a security domain.</span></span> <span data-ttu-id="4c870-109">它可以托管在本地，但是如果你的应用程序部分托管在本地部分托管在 Azure 中，则将此功能复制到 Azure 中可能更为高效。</span><span class="sxs-lookup"><span data-stu-id="4c870-109">It can be hosted on-premises, but if your application is hosted partly on-premises and partly in Azure, it may be more efficient to replicate this functionality in Azure.</span></span> <span data-ttu-id="4c870-110">这可以降低由于将身份验证和本地授权请求从云发送回在本地运行的 AD DS 而导致的延迟。</span><span class="sxs-lookup"><span data-stu-id="4c870-110">This can reduce the latency caused by sending authentication and local authorization requests from the cloud back to AD DS running on-premises.</span></span> 

<span data-ttu-id="4c870-111">如果本地网络和 Azure 虚拟网络通过 VPN 或 ExpressRoute 连接进行连接，通常会使用此体系结构。</span><span class="sxs-lookup"><span data-stu-id="4c870-111">This architecture is commonly used when the on-premises network and the Azure virtual network are connected by a VPN or ExpressRoute connection.</span></span> <span data-ttu-id="4c870-112">此体系结构还支持双向复制，这意味着既可以在本地又可以在云中进行更改，并且两个源都将保持一致。</span><span class="sxs-lookup"><span data-stu-id="4c870-112">This architecture also supports bidirectional replication, meaning changes can be made either on-premises or in the cloud, and both sources will be kept consistent.</span></span> <span data-ttu-id="4c870-113">此体系结构的典型用途包括功能分布在本地和 Azure 中的混合应用程序以及使用 Active Directory 执行身份验证的应用程序和服务。</span><span class="sxs-lookup"><span data-stu-id="4c870-113">Typical uses for this architecture include hybrid applications in which functionality is distributed between on-premises and Azure, and applications and services that perform authentication using Active Directory.</span></span>

<span data-ttu-id="4c870-114">有关其他注意事项，请参阅[选择用于将本地 Active Directory 与 Azure 相集成的解决方案][considerations]。</span><span class="sxs-lookup"><span data-stu-id="4c870-114">For additional considerations, see [Choose a solution for integrating on-premises Active Directory with Azure][considerations].</span></span> 

## <a name="architecture"></a><span data-ttu-id="4c870-115">体系结构</span><span class="sxs-lookup"><span data-stu-id="4c870-115">Architecture</span></span> 

<span data-ttu-id="4c870-116">此体系结构扩展了 [Azure 与 Internet 之间的外围网络][implementing-a-secure-hybrid-network-architecture-with-internet-access]中显示的体系结构。</span><span class="sxs-lookup"><span data-stu-id="4c870-116">This architecture extends the architecture shown in [DMZ between Azure and the Internet][implementing-a-secure-hybrid-network-architecture-with-internet-access].</span></span> <span data-ttu-id="4c870-117">它包括以下组件。</span><span class="sxs-lookup"><span data-stu-id="4c870-117">It has the following components.</span></span>

* <span data-ttu-id="4c870-118">**本地网络**。</span><span class="sxs-lookup"><span data-stu-id="4c870-118">**On-premises network**.</span></span> <span data-ttu-id="4c870-119">本地网络包括可以对位于本地的组件执行身份验证和授权的本地 Active Directory 服务器。</span><span class="sxs-lookup"><span data-stu-id="4c870-119">The on-premises network includes local Active Directory servers that can perform authentication and authorization for components located on-premises.</span></span>
* <span data-ttu-id="4c870-120">**Active Directory 服务器**。</span><span class="sxs-lookup"><span data-stu-id="4c870-120">**Active Directory servers**.</span></span> <span data-ttu-id="4c870-121">它们是域控制器，用于实现在云中作为 VM 运行的目录服务 (AD DS)。</span><span class="sxs-lookup"><span data-stu-id="4c870-121">These are domain controllers implementing directory services (AD DS) running as VMs in the cloud.</span></span> <span data-ttu-id="4c870-122">这些服务器可以为在 Azure 虚拟网络中运行的组件提供身份验证。</span><span class="sxs-lookup"><span data-stu-id="4c870-122">These servers can provide authentication of components running in your Azure virtual network.</span></span>
* <span data-ttu-id="4c870-123">**Active Directory 子网**。</span><span class="sxs-lookup"><span data-stu-id="4c870-123">**Active Directory subnet**.</span></span> <span data-ttu-id="4c870-124">AD DS 服务器托管在一个单独的子网中。</span><span class="sxs-lookup"><span data-stu-id="4c870-124">The AD DS servers are hosted in a separate subnet.</span></span> <span data-ttu-id="4c870-125">网络安全组 (NSG) 规则对 AD DS 服务器进行保护，并提供防火墙以阻止来自意外来源的流量。</span><span class="sxs-lookup"><span data-stu-id="4c870-125">Network security group (NSG) rules protect the AD DS servers and provide a firewall against traffic from unexpected sources.</span></span>
* <span data-ttu-id="4c870-126">**Azure 网关和 Active Directory 同步**。</span><span class="sxs-lookup"><span data-stu-id="4c870-126">**Azure Gateway and Active Directory synchronization**.</span></span> <span data-ttu-id="4c870-127">Azure 网关在本地网络与 Azure VNet 之间提供连接。</span><span class="sxs-lookup"><span data-stu-id="4c870-127">The Azure gateway provides a connection between the on-premises network and the Azure VNet.</span></span> <span data-ttu-id="4c870-128">这可以是 [VPN 连接][azure-vpn-gateway]，也可以是 [Azure ExpressRoute][azure-expressroute]。</span><span class="sxs-lookup"><span data-stu-id="4c870-128">This can be a [VPN connection][azure-vpn-gateway] or [Azure ExpressRoute][azure-expressroute].</span></span> <span data-ttu-id="4c870-129">云中的 Active Directory 服务器与本地 Active Directory 服务器之间的所有同步请求都通过该网关。</span><span class="sxs-lookup"><span data-stu-id="4c870-129">All synchronization requests between the Active Directory servers in the cloud and on-premises pass through the gateway.</span></span> <span data-ttu-id="4c870-130">用户定义的路由 (UDR) 处理传递到 Azure 的本地流量的路由。</span><span class="sxs-lookup"><span data-stu-id="4c870-130">User-defined routes (UDRs) handle routing for on-premises traffic that passes to Azure.</span></span> <span data-ttu-id="4c870-131">传入到 Active Directory 服务器或从中传出的流量不通过此方案中使用的网络虚拟设备 (NVA)。</span><span class="sxs-lookup"><span data-stu-id="4c870-131">Traffic to and from the Active Directory servers does not pass through the network virtual appliances (NVAs) used in this scenario.</span></span>

<span data-ttu-id="4c870-132">有关配置 UDR 和 NVA 的详细信息，请参阅[在 Azure 中实现安全的混合网络体系结构][implementing-a-secure-hybrid-network-architecture]。</span><span class="sxs-lookup"><span data-stu-id="4c870-132">For more information about configuring UDRs and the NVAs, see [Implementing a secure hybrid network architecture in Azure][implementing-a-secure-hybrid-network-architecture].</span></span> 

## <a name="recommendations"></a><span data-ttu-id="4c870-133">建议</span><span class="sxs-lookup"><span data-stu-id="4c870-133">Recommendations</span></span>

<span data-ttu-id="4c870-134">以下建议适用于大多数方案。</span><span class="sxs-lookup"><span data-stu-id="4c870-134">The following recommendations apply for most scenarios.</span></span> <span data-ttu-id="4c870-135">除非有优先于这些建议的特定要求，否则请遵循这些建议。</span><span class="sxs-lookup"><span data-stu-id="4c870-135">Follow these recommendations unless you have a specific requirement that overrides them.</span></span> 

### <a name="vm-recommendations"></a><span data-ttu-id="4c870-136">VM 建议</span><span class="sxs-lookup"><span data-stu-id="4c870-136">VM recommendations</span></span>

<span data-ttu-id="4c870-137">根据预计的身份验证请求量决定 [VM 大小][vm-windows-sizes]。</span><span class="sxs-lookup"><span data-stu-id="4c870-137">Determine your [VM size][vm-windows-sizes] requirements based on the expected volume of authentication requests.</span></span> <span data-ttu-id="4c870-138">使用在本地托管着 AD DS 的计算机的规范作为起点，并使其与 Azure VM 大小相匹配。</span><span class="sxs-lookup"><span data-stu-id="4c870-138">Use the specifications of the machines hosting AD DS on premises as a starting point, and match them with the Azure VM sizes.</span></span> <span data-ttu-id="4c870-139">在部署后，监视利用率并根据 VM 上的实际负载纵向扩展或收缩。</span><span class="sxs-lookup"><span data-stu-id="4c870-139">Once deployed, monitor utilization and scale up or down based on the actual load on the VMs.</span></span> <span data-ttu-id="4c870-140">有关确定 AD DS 域控制器大小的详细信息，请参阅 [Capacity Planning for Active Directory Domain Services][capacity-planning-for-adds]（Active Directory 域服务的容量规划）。</span><span class="sxs-lookup"><span data-stu-id="4c870-140">For more information about sizing AD DS domain controllers, see [Capacity Planning for Active Directory Domain Services][capacity-planning-for-adds].</span></span>

<span data-ttu-id="4c870-141">创建一个单独的虚拟数据磁盘，用以存储 Active Directory 的数据库、日志和 SYSVOL。</span><span class="sxs-lookup"><span data-stu-id="4c870-141">Create a separate virtual data disk for storing the database, logs, and SYSVOL for Active Directory.</span></span> <span data-ttu-id="4c870-142">不要将这些项与操作系统存储在同一磁盘上。</span><span class="sxs-lookup"><span data-stu-id="4c870-142">Do not store these items on the same disk as the operating system.</span></span> <span data-ttu-id="4c870-143">注意，默认情况下，附加到 VM 的数据磁盘使用直写式缓存。</span><span class="sxs-lookup"><span data-stu-id="4c870-143">Note that by default, data disks that are attached to a VM use write-through caching.</span></span> <span data-ttu-id="4c870-144">但是，这种形式的缓存可能会与 AD DS 的要求发生冲突。</span><span class="sxs-lookup"><span data-stu-id="4c870-144">However, this form of caching can conflict with the requirements of AD DS.</span></span> <span data-ttu-id="4c870-145">因此，请将数据磁盘上的“主机缓存首选项”设置设为“无”。</span><span class="sxs-lookup"><span data-stu-id="4c870-145">For this reason, set the *Host Cache Preference* setting on the data disk to *None*.</span></span> <span data-ttu-id="4c870-146">有关详细信息，请参阅[放置 Windows Server AD DS 数据库和 SYSVOL][adds-data-disks]。</span><span class="sxs-lookup"><span data-stu-id="4c870-146">For more information, see [Placement of the Windows Server AD DS database and SYSVOL][adds-data-disks].</span></span>

<span data-ttu-id="4c870-147">部署至少两个运行 AD DS 作为域控制器的 VM，并将它们添加到[可用性集][availability-set]中。</span><span class="sxs-lookup"><span data-stu-id="4c870-147">Deploy at least two VMs running AD DS as domain controllers and add them to an [availability set][availability-set].</span></span>

### <a name="networking-recommendations"></a><span data-ttu-id="4c870-148">网络建议</span><span class="sxs-lookup"><span data-stu-id="4c870-148">Networking recommendations</span></span>

<span data-ttu-id="4c870-149">为每个 AD DS 服务器的 VM 网络接口 (NIC) 配置一个静态专用 IP 地址，以提供完整的域名服务 (DNS) 支持。</span><span class="sxs-lookup"><span data-stu-id="4c870-149">Configure the VM network interface (NIC) for each AD DS server with a static private IP address for full domain name service (DNS) support.</span></span> <span data-ttu-id="4c870-150">有关详细信息，请参阅[如何在 Azure 门户中设置静态专用 IP 地址][set-a-static-ip-address]。</span><span class="sxs-lookup"><span data-stu-id="4c870-150">For more information, see [How to set a static private IP address in the Azure portal][set-a-static-ip-address].</span></span>

> [!NOTE]
> <span data-ttu-id="4c870-151">不要为任何 AD DS 的 VM NIC 配置公共 IP 地址。</span><span class="sxs-lookup"><span data-stu-id="4c870-151">Do not configure the VM NIC for any AD DS with a public IP address.</span></span> <span data-ttu-id="4c870-152">有关更多详细信息，请参阅[安全注意事项][security-considerations]。</span><span class="sxs-lookup"><span data-stu-id="4c870-152">See [Security considerations][security-considerations] for more details.</span></span>
> 
> 

<span data-ttu-id="4c870-153">Active Directory 子网 NSG 要求规则允许来自本地的传入流量。</span><span class="sxs-lookup"><span data-stu-id="4c870-153">The Active Directory subnet NSG requires rules to permit incoming traffic from on-premises.</span></span> <span data-ttu-id="4c870-154">有关 AD DS 使用的端口的详细信息，请参阅 [Active Directory and Active Directory Domain Services Port Requirements][ad-ds-ports]（Active Directory 和 Active Directory 域服务端口要求）。</span><span class="sxs-lookup"><span data-stu-id="4c870-154">For detailed information on the ports used by AD DS, see [Active Directory and Active Directory Domain Services Port Requirements][ad-ds-ports].</span></span> <span data-ttu-id="4c870-155">此外，请确保 UDR 表不通过此体系结构中使用的 NVA 来路由 AD DS 流量。</span><span class="sxs-lookup"><span data-stu-id="4c870-155">Also, ensure the UDR tables do not route AD DS traffic through the NVAs used in this architecture.</span></span> 

### <a name="active-directory-site"></a><span data-ttu-id="4c870-156">Active Directory 站点</span><span class="sxs-lookup"><span data-stu-id="4c870-156">Active Directory site</span></span>

<span data-ttu-id="4c870-157">在 AD DS 中，站点表示一个物理位置、网络或表示设备集合。</span><span class="sxs-lookup"><span data-stu-id="4c870-157">In AD DS, a site represents a physical location, network, or collection of devices.</span></span> <span data-ttu-id="4c870-158">AD DS 站点用来管理 AD DS 数据库复制，它们将位置彼此靠近且由高速网络连接的 AD DS 对象分组到一起。</span><span class="sxs-lookup"><span data-stu-id="4c870-158">AD DS sites are used to manage AD DS database replication by grouping together AD DS objects that are located close to one another and are connected by a high speed network.</span></span> <span data-ttu-id="4c870-159">AD DS 包括了相应的逻辑来选择用于在站点之间复制 AD DS 数据库的最佳策略。</span><span class="sxs-lookup"><span data-stu-id="4c870-159">AD DS includes logic to select the best strategy for replacating the AD DS database between sites.</span></span>

<span data-ttu-id="4c870-160">建议你创建一个 AD DS 站点，并使其包括在 Azure 中为你的应用程序定义的子网。</span><span class="sxs-lookup"><span data-stu-id="4c870-160">We recommend that you create an AD DS site including the subnets defined for your application in Azure.</span></span> <span data-ttu-id="4c870-161">然后，配置本地 AD DS 站点之间的站点链接，AD DS 将自动执行最高效的数据库复制。</span><span class="sxs-lookup"><span data-stu-id="4c870-161">Then, configure a site link between your on-premises AD DS sites, and AD DS will automatically perform the most efficient database replication possible.</span></span> <span data-ttu-id="4c870-162">注意，除了初始配置之外，此数据库复制还要求进行少量其他配置。</span><span class="sxs-lookup"><span data-stu-id="4c870-162">Note that this database replication requires little beyond the initial configuration.</span></span>

### <a name="active-directory-operations-masters"></a><span data-ttu-id="4c870-163">Active Directory 操作主机</span><span class="sxs-lookup"><span data-stu-id="4c870-163">Active Directory operations masters</span></span>

<span data-ttu-id="4c870-164">可以向 AD DS 域控制器分配操作主机角色以支持在复制的 AD DS 数据库实例之间执行一致性检查。</span><span class="sxs-lookup"><span data-stu-id="4c870-164">The operations masters role can be assigned to AD DS domain controllers to support consistency checking between instances of replicated AD DS databases.</span></span> <span data-ttu-id="4c870-165">有五个操作主机角色：架构主机、域命名主机、相对标识符主机、主域控制器主机模拟器和基础结构主机。</span><span class="sxs-lookup"><span data-stu-id="4c870-165">There are five operations master roles: schema master, domain naming master, relative identifier master, primary domain controller master emulator, and infrastructure master.</span></span> <span data-ttu-id="4c870-166">有关这些角色的详细信息，请参阅 [What are Operations Masters?][ad-ds-operations-masters]（什么是操作主机？）。</span><span class="sxs-lookup"><span data-stu-id="4c870-166">For more information about these roles, see [What are Operations Masters?][ad-ds-operations-masters].</span></span>

<span data-ttu-id="4c870-167">建议不要向在 Azure 中部署的域控制器分配操作主机角色。</span><span class="sxs-lookup"><span data-stu-id="4c870-167">We recommend you do not assign operations masters roles to the domain controllers deployed in Azure.</span></span>

### <a name="monitoring"></a><span data-ttu-id="4c870-168">监视</span><span class="sxs-lookup"><span data-stu-id="4c870-168">Monitoring</span></span>

<span data-ttu-id="4c870-169">监视域控制器 VM 以及 AD DS 服务的资源并创建一个计划来快速更正任何问题。</span><span class="sxs-lookup"><span data-stu-id="4c870-169">Monitor the resources of the domain controller VMs as well as the AD DS Services and create a plan to quickly correct any problems.</span></span> <span data-ttu-id="4c870-170">有关详细信息，请参阅 [Monitoring Active Directory][monitoring_ad]（监视 Active Directory）。</span><span class="sxs-lookup"><span data-stu-id="4c870-170">For more information, see [Monitoring Active Directory][monitoring_ad].</span></span> <span data-ttu-id="4c870-171">可以在监视服务器上（请参阅体系结构图）安装 [Microsoft Systems Center][microsoft_systems_center] 之类的工具来帮助执行这些任务。</span><span class="sxs-lookup"><span data-stu-id="4c870-171">You can also install tools such as [Microsoft Systems Center][microsoft_systems_center] on the monitoring server (see the architecture diagram) to help perform these tasks.</span></span>  

## <a name="scalability-considerations"></a><span data-ttu-id="4c870-172">可伸缩性注意事项</span><span class="sxs-lookup"><span data-stu-id="4c870-172">Scalability considerations</span></span>

<span data-ttu-id="4c870-173">AD DS 是为实现可伸缩性而设计的。</span><span class="sxs-lookup"><span data-stu-id="4c870-173">AD DS is designed for scalability.</span></span> <span data-ttu-id="4c870-174">你不需要配置负载均衡器或流量控制器来将请求定向到 AD DS 域控制器。</span><span class="sxs-lookup"><span data-stu-id="4c870-174">You don't need to configure a load balancer or traffic controller to direct requests to AD DS domain controllers.</span></span> <span data-ttu-id="4c870-175">唯一的可伸缩性注意事项是为运行 AD DS 的 VM 配置满足你的网络负载要求的合适大小，监视 VM 上的负载并根据需要纵向扩展或收缩。</span><span class="sxs-lookup"><span data-stu-id="4c870-175">The only scalability consideration is to configure the VMs running AD DS with the correct size for your network load requirements, monitor the load on the VMs, and scale up or down as necessary.</span></span>

## <a name="availability-considerations"></a><span data-ttu-id="4c870-176">可用性注意事项</span><span class="sxs-lookup"><span data-stu-id="4c870-176">Availability considerations</span></span>

<span data-ttu-id="4c870-177">将运行 AD DS 的 VM 部署到一个[可用性集][availability-set]中。</span><span class="sxs-lookup"><span data-stu-id="4c870-177">Deploy the VMs running AD DS into an [availability set][availability-set].</span></span> <span data-ttu-id="4c870-178">另外，请考虑根据需要向至少一台（可能更多）服务器分配[备用操作主机][standby-operations-masters]角色。</span><span class="sxs-lookup"><span data-stu-id="4c870-178">Also, consider assigning the role of [standby operations master][standby-operations-masters] to at least one server, and possibly more depending on your requirements.</span></span> <span data-ttu-id="4c870-179">备用操作主机是操作主机的主动副本，在故障转移期间可以用来替代主操作主机服务器。</span><span class="sxs-lookup"><span data-stu-id="4c870-179">A standby operations master is an active copy of the operations master that can be used in place of the primary operations masters server during fail over.</span></span>

## <a name="manageability-considerations"></a><span data-ttu-id="4c870-180">可管理性注意事项</span><span class="sxs-lookup"><span data-stu-id="4c870-180">Manageability considerations</span></span>

<span data-ttu-id="4c870-181">执行定期 AD DS 备份。</span><span class="sxs-lookup"><span data-stu-id="4c870-181">Perform regular AD DS backups.</span></span> <span data-ttu-id="4c870-182">不要只是简单地复制域控制器的 VHD 文件，而是要执行定期备份，因为在复制 VHD 上的 AD DS 数据库文件时，该文件可能会未处于一致状态，导致无法重新启动数据库。</span><span class="sxs-lookup"><span data-stu-id="4c870-182">Don't simply copy the VHD files of domain controllers instead of performing regular backups, because the AD DS database file on the VHD may not be in a consistent state when it's copied, making it impossible to restart the database.</span></span>

<span data-ttu-id="4c870-183">不要使用 Azure 门户关闭域控制器 VM。</span><span class="sxs-lookup"><span data-stu-id="4c870-183">Do not shut down a domain controller VM using Azure portal.</span></span> <span data-ttu-id="4c870-184">相反，请从来宾操作系统进行关闭和重新启动。</span><span class="sxs-lookup"><span data-stu-id="4c870-184">Instead, shut down and restart from the guest operating system.</span></span> <span data-ttu-id="4c870-185">通过门户进行关闭会导致 VM 被解除分配，这会重置 Active Directory 存储库的 `VM-GenerationID` 和 `invocationID`。</span><span class="sxs-lookup"><span data-stu-id="4c870-185">Shutting down through the portal causes the VM to be deallocated, which resets both the `VM-GenerationID` and the `invocationID` of the Active Directory repository.</span></span> <span data-ttu-id="4c870-186">这将放弃 AD DS 相对标识符 (RID) 池并将 SYSVOL 标记为非权威的，并且可能需要配置域控制器。</span><span class="sxs-lookup"><span data-stu-id="4c870-186">This discards the AD DS relative identifier (RID) pool and marks SYSVOL as nonauthoritative, and may require reconfiguration of the domain controller.</span></span>

## <a name="security-considerations"></a><span data-ttu-id="4c870-187">安全注意事项</span><span class="sxs-lookup"><span data-stu-id="4c870-187">Security considerations</span></span>

<span data-ttu-id="4c870-188">AD DS 服务器提供身份验证服务并且是引入注目的攻击目标。</span><span class="sxs-lookup"><span data-stu-id="4c870-188">AD DS servers provide authentication services and are an attractive target for attacks.</span></span> <span data-ttu-id="4c870-189">若要保护它们，请通过以下方法阻止直接 Internet 连接：将 AD DS 服务器放置在一个单独的子网中并使用 NSG 作为防火墙。</span><span class="sxs-lookup"><span data-stu-id="4c870-189">To secure them, prevent direct Internet connectivity by placing the AD DS servers in a separate subnet with an NSG acting as a firewall.</span></span> <span data-ttu-id="4c870-190">关闭 AD DS 服务器上的所有端口，但身份验证、授权和服务器同步所需的那些端口除外。</span><span class="sxs-lookup"><span data-stu-id="4c870-190">Close all ports on the AD DS servers except those necessary for authentication, authorization, and server synchronization.</span></span> <span data-ttu-id="4c870-191">有关详细信息，请参阅 [Active Directory and Active Directory Domain Services Port Requirements][ad-ds-ports]（Active Directory 和 Active Directory 域服务端口要求）。</span><span class="sxs-lookup"><span data-stu-id="4c870-191">For more information, see [Active Directory and Active Directory Domain Services Port Requirements][ad-ds-ports].</span></span>

<span data-ttu-id="4c870-192">考虑通过一对子网和 NVA 在服务器周围实现一个额外的安全外围，如[在 Azure 中实现具有 Internet 访问的安全的混合网络体系结构][implementing-a-secure-hybrid-network-architecture-with-internet-access]中所述。</span><span class="sxs-lookup"><span data-stu-id="4c870-192">Consider implementing an additional security perimeter around servers with a pair of subnets and NVAs, as described in [Implementing a secure hybrid network architecture with Internet access in Azure][implementing-a-secure-hybrid-network-architecture-with-internet-access].</span></span>

<span data-ttu-id="4c870-193">使用 BitLocker 或 Azure 磁盘加密对托管着 AD DS 数据库的磁盘进行加密。</span><span class="sxs-lookup"><span data-stu-id="4c870-193">Use either BitLocker or Azure disk encryption to encrypt the disk hosting the AD DS database.</span></span>

## <a name="deploy-the-solution"></a><span data-ttu-id="4c870-194">部署解决方案</span><span class="sxs-lookup"><span data-stu-id="4c870-194">Deploy the solution</span></span>

<span data-ttu-id="4c870-195">[GitHub][github] 上提供了此体系结构的部署。</span><span class="sxs-lookup"><span data-stu-id="4c870-195">A deployment for this architecture is available on [GitHub][github].</span></span> <span data-ttu-id="4c870-196">请注意，整个部署最长可能需要花费两个小时，包括创建 VPN 网关和运行配置 AD DS 的脚本。</span><span class="sxs-lookup"><span data-stu-id="4c870-196">Note that the entire deployment can take up to two hours, which includes creating the VPN gateway and running the scripts that configure AD DS.</span></span>

### <a name="prerequisites"></a><span data-ttu-id="4c870-197">先决条件</span><span class="sxs-lookup"><span data-stu-id="4c870-197">Prerequisites</span></span>

1. <span data-ttu-id="4c870-198">克隆、下载[参考体系结构][github] GitHub 存储库的 zip 文件或创建其分支。</span><span class="sxs-lookup"><span data-stu-id="4c870-198">Clone, fork, or download the zip file for the [reference architectures][github] GitHub repository.</span></span>

2. <span data-ttu-id="4c870-199">安装 [Azure CLI 2.0][azure-cli-2]。</span><span class="sxs-lookup"><span data-stu-id="4c870-199">Install [Azure CLI 2.0][azure-cli-2].</span></span>

3. <span data-ttu-id="4c870-200">安装 [Azure 构建基块][azbb] npm 包。</span><span class="sxs-lookup"><span data-stu-id="4c870-200">Install the [Azure building blocks][azbb] npm package.</span></span>

4. <span data-ttu-id="4c870-201">在命令提示符、bash 提示符或 PowerShell 提示符下使用以下命令登录到 Azure 帐户。</span><span class="sxs-lookup"><span data-stu-id="4c870-201">From a command prompt, bash prompt, or PowerShell prompt, log into your Azure account by using the command below.</span></span>

   ```bash
   az login
   ```

### <a name="deploy-the-simulated-on-premises-datacenter"></a><span data-ttu-id="4c870-202">部署模拟的本地数据中心</span><span class="sxs-lookup"><span data-stu-id="4c870-202">Deploy the simulated on-premises datacenter</span></span>

1. <span data-ttu-id="4c870-203">导航到 GitHub 存储库的 `identity/adds-extend-domain` 文件夹。</span><span class="sxs-lookup"><span data-stu-id="4c870-203">Navigate to the `identity/adds-extend-domain` folder of the GitHub repository.</span></span>

2. <span data-ttu-id="4c870-204">打开 `onprem.json` 文件。</span><span class="sxs-lookup"><span data-stu-id="4c870-204">Open the `onprem.json` file.</span></span> <span data-ttu-id="4c870-205">搜索 `adminPassword` 和 `Password` 的实例并添加密码值。</span><span class="sxs-lookup"><span data-stu-id="4c870-205">Search for instances of `adminPassword` and `Password` and add values for the passwords.</span></span>

3. <span data-ttu-id="4c870-206">运行以下命令，并等待部署完成：</span><span class="sxs-lookup"><span data-stu-id="4c870-206">Run the following command and wait for the deployment to finish:</span></span>

    ```bash
    azbb -s <subscription_id> -g <resource group> -l <location> -p onprem.json --deploy
    ```

### <a name="deploy-the-azure-vnet"></a><span data-ttu-id="4c870-207">部署 Azure VNet</span><span class="sxs-lookup"><span data-stu-id="4c870-207">Deploy the Azure VNet</span></span>

1. <span data-ttu-id="4c870-208">打开 `azure.json` 文件。</span><span class="sxs-lookup"><span data-stu-id="4c870-208">Open the `azure.json` file.</span></span>  <span data-ttu-id="4c870-209">搜索 `adminPassword` 和 `Password` 的实例并添加密码值。</span><span class="sxs-lookup"><span data-stu-id="4c870-209">Search for instances of `adminPassword` and `Password` and add values for the passwords.</span></span> 

2. <span data-ttu-id="4c870-210">在同一文件中，搜索 `sharedKey` 的实例并输入 VPN 连接的共享密钥。</span><span class="sxs-lookup"><span data-stu-id="4c870-210">In the same file, search for instances of `sharedKey` and enter shared keys for the VPN connection.</span></span> 

    ```bash
    "sharedKey": "",
    ```

3. <span data-ttu-id="4c870-211">运行以下命令并等待部署完成。</span><span class="sxs-lookup"><span data-stu-id="4c870-211">Run the following command and wait for the deployment to finish.</span></span>

    ```bash
    azbb -s <subscription_id> -g <resource group> -l <location> -p onoprem.json --deploy
    ```

   <span data-ttu-id="4c870-212">部署到本地 VNet 所在的同一个资源组。</span><span class="sxs-lookup"><span data-stu-id="4c870-212">Deploy to the same resource group as the on-premises VNet.</span></span>

### <a name="test-connectivity-with-the-azure-vnet"></a><span data-ttu-id="4c870-213">测试与 Azure VNet 的连接</span><span class="sxs-lookup"><span data-stu-id="4c870-213">Test connectivity with the Azure VNet</span></span>

<span data-ttu-id="4c870-214">部署完成后，可以测试从模拟本地环境到 Azure VNet 的连接。</span><span class="sxs-lookup"><span data-stu-id="4c870-214">After deployment completes, you can test conectivity from the simulated on-premises environment to the Azure VNet.</span></span>

1. <span data-ttu-id="4c870-215">使用 Azure 门户导航到已创建的资源组。</span><span class="sxs-lookup"><span data-stu-id="4c870-215">Use the Azure portal, navigate to the resource group that you created.</span></span>

2. <span data-ttu-id="4c870-216">找到名为 `ra-onpremise-mgmt-vm1` 的 VM。</span><span class="sxs-lookup"><span data-stu-id="4c870-216">Find the VM named `ra-onpremise-mgmt-vm1`.</span></span>

3. <span data-ttu-id="4c870-217">单击 `Connect` 来与 VM 建立远程桌面会话。</span><span class="sxs-lookup"><span data-stu-id="4c870-217">Click `Connect` to open a remote desktop session to the VM.</span></span> <span data-ttu-id="4c870-218">用户名为 `contoso\testuser`，密码为 `onprem.json` 参数文件中指定的密码。</span><span class="sxs-lookup"><span data-stu-id="4c870-218">The username is `contoso\testuser`, and the password is the one that you specified in the `onprem.json` parameter file.</span></span>

4. <span data-ttu-id="4c870-219">在远程桌面会话中，与 10.0.4.4（名为 `adds-vm1` 的 VM 的 IP 地址）建立另一个远程桌面会话。</span><span class="sxs-lookup"><span data-stu-id="4c870-219">From inside your remote desktop session, open another remote desktop session to 10.0.4.4, which is the IP address of the VM named `adds-vm1`.</span></span> <span data-ttu-id="4c870-220">用户名为 `contoso\testuser`，密码为 `azure.json` 参数文件中指定的密码。</span><span class="sxs-lookup"><span data-stu-id="4c870-220">The username is `contoso\testuser`, and the password is the one that you specified in the `azure.json` parameter file.</span></span>

5. <span data-ttu-id="4c870-221">在 `adds-vm1` 的远程桌面会话中，转到“服务器管理器”并单击“添加要管理的其他服务器”。</span><span class="sxs-lookup"><span data-stu-id="4c870-221">From inside the remote desktop session for `adds-vm1`, go to **Server Manager** and click **Add other servers to manage.**</span></span> 

6. <span data-ttu-id="4c870-222">在“Active Directory”选项卡中，单击“立即查找”。</span><span class="sxs-lookup"><span data-stu-id="4c870-222">In the **Active Directory** tab, click **Find now**.</span></span> <span data-ttu-id="4c870-223">应会看到 AD、AD DS 和 Web VM 的列表。</span><span class="sxs-lookup"><span data-stu-id="4c870-223">You should see a list of the AD, AD DS, and Web VMs.</span></span>

   ![](./images/add-servers-dialog.png)

## <a name="next-steps"></a><span data-ttu-id="4c870-224">后续步骤</span><span class="sxs-lookup"><span data-stu-id="4c870-224">Next steps</span></span>

* <span data-ttu-id="4c870-225">了解在 Azure 中[创建 AD DS 资源林][adds-resource-forest]的最佳做法。</span><span class="sxs-lookup"><span data-stu-id="4c870-225">Learn the best practices for [creating an AD DS resource forest][adds-resource-forest] in Azure.</span></span>
* <span data-ttu-id="4c870-226">了解在 Azure 中[创建 Active Directory 联合身份验证服务 (AD FS) 基础结构][adfs]的最佳做法。</span><span class="sxs-lookup"><span data-stu-id="4c870-226">Learn the best practices for [creating an Active Directory Federation Services (AD FS) infrastructure][adfs] in Azure.</span></span>

<!-- links -->

[adds-resource-forest]: adds-forest.md
[adfs]: adfs.md
[azure-cli-2]: /azure/install-azure-cli
[azbb]: https://github.com/mspnp/template-building-blocks/wiki/Install-Azure-Building-Blocks
[implementing-a-secure-hybrid-network-architecture]: ../dmz/secure-vnet-hybrid.md
[implementing-a-secure-hybrid-network-architecture-with-internet-access]: ../dmz/secure-vnet-dmz.md

[adds-data-disks]: https://msdn.microsoft.com/library/azure/jj156090.aspx#BKMK_PlaceDB
[ad-ds-operations-masters]: https://technet.microsoft.com/library/cc779716(v=ws.10).aspx
[ad-ds-ports]: https://technet.microsoft.com/library/dd772723(v=ws.11).aspx
[availability-set]: /azure/virtual-machines/virtual-machines-windows-create-availability-set
[azure-expressroute]: /azure/expressroute/expressroute-introduction
[azure-vpn-gateway]: /azure/vpn-gateway/vpn-gateway-about-vpngateways
[capacity-planning-for-adds]: http://social.technet.microsoft.com/wiki/contents/articles/14355.capacity-planning-for-active-directory-domain-services.aspx
[considerations]: ./considerations.md
[GitHub]: https://github.com/mspnp/reference-architectures/tree/master/identity/adds-extend-domain
[microsoft_systems_center]: https://www.microsoft.com/server-cloud/products/system-center-2016/
[monitoring_ad]: https://msdn.microsoft.com/library/bb727046.aspx
[security-considerations]: #security-considerations
[set-a-static-ip-address]: /azure/virtual-network/virtual-networks-static-private-ip-arm-pportal
[standby-operations-masters]: https://technet.microsoft.com/library/cc794737(v=ws.10).aspx
[visio-download]: https://archcenter.blob.core.windows.net/cdn/identity-architectures.vsdx
[vm-windows-sizes]: /azure/virtual-machines/virtual-machines-windows-sizes

[0]: ./images/adds-extend-domain.png "使用 Active Directory 保护混合网络体系结构的安全"
