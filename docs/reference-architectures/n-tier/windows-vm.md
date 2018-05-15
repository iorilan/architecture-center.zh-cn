---
title: 在 Azure 上运行 Windows VM
description: 如何在 Azure 上运行 Windows VM，并注意可伸缩性、弹性、可管理性和安全性。
author: telmosampaio
ms.date: 04/03/2018
ms.openlocfilehash: 748bcfcf599c7fa0e82691632d14e0f78fd78582
ms.sourcegitcommit: a5e549c15a948f6fb5cec786dbddc8578af3be66
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/06/2018
---
# <a name="run-a-windows-vm-on-azure"></a><span data-ttu-id="23acc-103">在 Azure 上运行 Windows VM</span><span class="sxs-lookup"><span data-stu-id="23acc-103">Run a Windows VM on Azure</span></span>

<span data-ttu-id="23acc-104">本文介绍一组经验证的做法，说明如何在 Azure 上运行 Windows 虚拟机 (VM)。</span><span class="sxs-lookup"><span data-stu-id="23acc-104">This article describes a set of proven practices for running a Windows virtual machine (VM) on Azure.</span></span> <span data-ttu-id="23acc-105">其中包括有关预配 VM 以及网络和存储组件的建议。</span><span class="sxs-lookup"><span data-stu-id="23acc-105">It includes recommendations for provisioning the VM along with networking and storage components.</span></span> [<span data-ttu-id="23acc-106">部署此解决方案。</span><span class="sxs-lookup"><span data-stu-id="23acc-106">**Deploy this solution.**</span></span>](#deploy-the-solution)

<span data-ttu-id="23acc-107">![[0]][0]</span><span class="sxs-lookup"><span data-stu-id="23acc-107">![[0]][0]</span></span>

## <a name="components"></a><span data-ttu-id="23acc-108">组件</span><span class="sxs-lookup"><span data-stu-id="23acc-108">Components</span></span>

<span data-ttu-id="23acc-109">除 VM 本身以外，预配 Azure VM 还需要其他一些组件，包括网络和存储资源。</span><span class="sxs-lookup"><span data-stu-id="23acc-109">Provisioning an Azure VM requires some additional components besides the VM itself, including networking and storage resources.</span></span>

* <span data-ttu-id="23acc-110">**资源组。**</span><span class="sxs-lookup"><span data-stu-id="23acc-110">**Resource group.**</span></span> <span data-ttu-id="23acc-111">[资源组][resource-manager-overview]是保存相关 Azure 资源的逻辑容器。</span><span class="sxs-lookup"><span data-stu-id="23acc-111">A [resource group][resource-manager-overview] is a logical container that holds related Azure resources.</span></span> <span data-ttu-id="23acc-112">一般情况下，可根据资源的生存期及其管理者将资源分组。</span><span class="sxs-lookup"><span data-stu-id="23acc-112">In general, group resources based on their lifetime and who will manage them.</span></span> 

* <span data-ttu-id="23acc-113">**VM**。</span><span class="sxs-lookup"><span data-stu-id="23acc-113">**VM**.</span></span> <span data-ttu-id="23acc-114">可以通过发布的映像列表或上传到 Azure Blob 存储的自定义托管映像或虚拟硬盘 (VHD) 文件来预配 VM。</span><span class="sxs-lookup"><span data-stu-id="23acc-114">You can provision a VM from a list of published images, or from a custom managed image or virtual hard disk (VHD) file uploaded to Azure Blob storage.</span></span>

* <span data-ttu-id="23acc-115">**托管磁盘**。</span><span class="sxs-lookup"><span data-stu-id="23acc-115">**Managed Disks**.</span></span> <span data-ttu-id="23acc-116">[Azure 托管磁盘][managed-disks]可代你处理存储，从而简化了磁盘管理。</span><span class="sxs-lookup"><span data-stu-id="23acc-116">[Azure Managed Disks][managed-disks] simplify disk management by handling the storage for you.</span></span> <span data-ttu-id="23acc-117">OS 磁盘是存储在 [Azure 存储][azure-storage]中的 VHD，因此即使主机关闭，OS 磁盘也仍然存在。</span><span class="sxs-lookup"><span data-stu-id="23acc-117">The OS disk is a VHD stored in [Azure Storage][azure-storage], so it persists even when the host machine is down.</span></span> <span data-ttu-id="23acc-118">我们还建议创建一个或多个[数据磁盘][data-disk]（用于保存应用程序数据的持久性 VHD）。</span><span class="sxs-lookup"><span data-stu-id="23acc-118">We also recommend creating one or more [data disks][data-disk], which are persistent VHDs used for application data.</span></span>

* <span data-ttu-id="23acc-119">**临时磁盘。**</span><span class="sxs-lookup"><span data-stu-id="23acc-119">**Temporary disk.**</span></span> <span data-ttu-id="23acc-120">使用临时磁盘（Windows 上的 `D:` 驱动器）创建 VM。</span><span class="sxs-lookup"><span data-stu-id="23acc-120">The VM is created with a temporary disk (the `D:` drive on Windows).</span></span> <span data-ttu-id="23acc-121">此磁盘存储在主机的物理驱动器上。</span><span class="sxs-lookup"><span data-stu-id="23acc-121">This disk is stored on a physical drive on the host machine.</span></span> <span data-ttu-id="23acc-122">它并非保存在 Azure 存储中，并且在重启期间以及发生其他 VM 生命周期事件期间可能会被删除。</span><span class="sxs-lookup"><span data-stu-id="23acc-122">It is *not* saved in Azure Storage and may be deleted during reboots and other VM lifecycle events.</span></span> <span data-ttu-id="23acc-123">只使用此磁盘存储临时数据，如页面文件或交换文件。</span><span class="sxs-lookup"><span data-stu-id="23acc-123">Use this disk only for temporary data, such as page or swap files.</span></span>

* <span data-ttu-id="23acc-124">**虚拟网络 (VNet)**。</span><span class="sxs-lookup"><span data-stu-id="23acc-124">**Virtual network (VNet).**</span></span> <span data-ttu-id="23acc-125">每个 Azure VM 都会部署到可细分为多个子网的 VNet 中。</span><span class="sxs-lookup"><span data-stu-id="23acc-125">Every Azure VM is deployed into a VNet that can be segmented into multiple subnets.</span></span>

* <span data-ttu-id="23acc-126">**网络接口 (NIC)**。</span><span class="sxs-lookup"><span data-stu-id="23acc-126">**Network interface (NIC)**.</span></span> <span data-ttu-id="23acc-127">NIC 使 VM 能够与虚拟网络进行通信。</span><span class="sxs-lookup"><span data-stu-id="23acc-127">The NIC enables the VM to communicate with the virtual network.</span></span>  

* <span data-ttu-id="23acc-128">**公共 IP 地址。**</span><span class="sxs-lookup"><span data-stu-id="23acc-128">**Public IP address.**</span></span> <span data-ttu-id="23acc-129">须使用公共 IP 地址才能与 VM 通信 &mdash; 例如，通过远程桌面 (RDP)。</span><span class="sxs-lookup"><span data-stu-id="23acc-129">A public IP address is needed to communicate with the VM &mdash; for example, via remote desktop (RDP).</span></span>  

* <span data-ttu-id="23acc-130">**Azure DNS**。</span><span class="sxs-lookup"><span data-stu-id="23acc-130">**Azure DNS**.</span></span> <span data-ttu-id="23acc-131">[Azure DNS][azure-dns] 是 DNS 域的托管服务，它使用 Microsoft Azure 基础结构提供名称解析。</span><span class="sxs-lookup"><span data-stu-id="23acc-131">[Azure DNS][azure-dns] is a hosting service for DNS domains, providing name resolution using Microsoft Azure infrastructure.</span></span> <span data-ttu-id="23acc-132">通过在 Azure 中托管域，可以使用与其他 Azure 服务相同的凭据、API、工具和计费来管理 DNS 记录。</span><span class="sxs-lookup"><span data-stu-id="23acc-132">By hosting your domains in Azure, you can manage your DNS records using the same credentials, APIs, tools, and billing as your other Azure services.</span></span>

* <span data-ttu-id="23acc-133">**网络安全组 (NSG)**。</span><span class="sxs-lookup"><span data-stu-id="23acc-133">**Network security group (NSG)**.</span></span> <span data-ttu-id="23acc-134">[网络安全组][nsg]用于允许或拒绝向 VM 传送网络流量。</span><span class="sxs-lookup"><span data-stu-id="23acc-134">[Network security groups][nsg] are used to allow or deny network traffic to VMs.</span></span> <span data-ttu-id="23acc-135">NSG 可与子网或单个 VM 实例相关联。</span><span class="sxs-lookup"><span data-stu-id="23acc-135">NSGs can be associated either with subnets or with individual VM instances.</span></span> 

* <span data-ttu-id="23acc-136">**诊断。**</span><span class="sxs-lookup"><span data-stu-id="23acc-136">**Diagnostics.**</span></span> <span data-ttu-id="23acc-137">诊断日志记录对于 VM 管理和故障排除至关重要。</span><span class="sxs-lookup"><span data-stu-id="23acc-137">Diagnostic logging is crucial for managing and troubleshooting the VM.</span></span>

## <a name="vm-recommendations"></a><span data-ttu-id="23acc-138">VM 建议</span><span class="sxs-lookup"><span data-stu-id="23acc-138">VM recommendations</span></span>

<span data-ttu-id="23acc-139">Azure 提供多种不同的虚拟机大小。</span><span class="sxs-lookup"><span data-stu-id="23acc-139">Azure offers many different virtual machine sizes.</span></span> <span data-ttu-id="23acc-140">有关详细信息，请参阅 [Azure 中虚拟机的大小][virtual-machine-sizes]。</span><span class="sxs-lookup"><span data-stu-id="23acc-140">For more information, see [Sizes for virtual machines in Azure][virtual-machine-sizes].</span></span> <span data-ttu-id="23acc-141">如果要将现有工作负荷转移到 Azure，开始时请先使用与本地服务器最匹配的 VM 大小。</span><span class="sxs-lookup"><span data-stu-id="23acc-141">If you are moving an existing workload to Azure, start with the VM size that's the closest match to your on-premises servers.</span></span> <span data-ttu-id="23acc-142">然后从 CPU、内存和每秒磁盘输入/输出操作次数 (IOPS) 等方面测量实际工作负荷的性能，并根据需要调整大小。</span><span class="sxs-lookup"><span data-stu-id="23acc-142">Then measure the performance of your actual workload with respect to CPU, memory, and disk input/output operations per second (IOPS), and adjust the size as needed.</span></span> <span data-ttu-id="23acc-143">如果 VM 需要多个 NIC，请注意每种 [VM 大小][vm-size-tables]都定义了最大 NIC 数量。</span><span class="sxs-lookup"><span data-stu-id="23acc-143">If you require multiple NICs for your VM, be aware that a maximum number of NICs is defined for each [VM size][vm-size-tables].</span></span>

<span data-ttu-id="23acc-144">通常选择离内部用户或客户最近的 Azure 区域。</span><span class="sxs-lookup"><span data-stu-id="23acc-144">Generally, choose an Azure region that is closest to your internal users or customers.</span></span> <span data-ttu-id="23acc-145">但是，并非所有 VM 大小都可在所有区域中使用。</span><span class="sxs-lookup"><span data-stu-id="23acc-145">However, not all VM sizes are available in all regions.</span></span> <span data-ttu-id="23acc-146">有关详细信息，请参阅[每个区域的服务][services-by-region]。</span><span class="sxs-lookup"><span data-stu-id="23acc-146">For more information, see [Services by region][services-by-region].</span></span> <span data-ttu-id="23acc-147">要获取特定区域中可用 VM 大小的列表，请从 Azure 命令行接口 (CLI) 运行以下命令：</span><span class="sxs-lookup"><span data-stu-id="23acc-147">For a list of the VM sizes available in a specific region, run the following command from the Azure command-line interface (CLI):</span></span>

```
az vm list-sizes --location <location>
```

<span data-ttu-id="23acc-148">要了解如何选择发布的 VM 映像，请参阅[查找 Windows VM 映像][select-vm-image]。</span><span class="sxs-lookup"><span data-stu-id="23acc-148">For information about choosing a published VM image, see [Find Windows VM images][select-vm-image].</span></span>

<span data-ttu-id="23acc-149">启用监视和诊断，包括基本运行状况指标、诊断基础结构日志和[启动诊断][boot-diagnostics]。</span><span class="sxs-lookup"><span data-stu-id="23acc-149">Enable monitoring and diagnostics, including basic health metrics, diagnostics infrastructure logs, and [boot diagnostics][boot-diagnostics].</span></span> <span data-ttu-id="23acc-150">如果 VM 陷入不可启动状态，启动诊断有助于诊断启动故障。</span><span class="sxs-lookup"><span data-stu-id="23acc-150">Boot diagnostics can help you diagnose boot failure if your VM gets into a non-bootable state.</span></span> <span data-ttu-id="23acc-151">有关详细信息，请参阅[启用监视和诊断][enable-monitoring]。</span><span class="sxs-lookup"><span data-stu-id="23acc-151">For more information, see [Enable monitoring and diagnostics][enable-monitoring].</span></span>  

## <a name="disk-and-storage-recommendations"></a><span data-ttu-id="23acc-152">磁盘和存储建议</span><span class="sxs-lookup"><span data-stu-id="23acc-152">Disk and storage recommendations</span></span>

<span data-ttu-id="23acc-153">为获得最佳磁盘 I/O 性能，建议使用[高级存储][premium-storage]，它在固态硬盘 (SSD) 上存储数据。</span><span class="sxs-lookup"><span data-stu-id="23acc-153">For best disk I/O performance, we recommend [Premium Storage][premium-storage], which stores data on solid-state drives (SSDs).</span></span> <span data-ttu-id="23acc-154">成本取决于预配磁盘的容量。</span><span class="sxs-lookup"><span data-stu-id="23acc-154">Cost is based on the capacity of the provisioned disk.</span></span> <span data-ttu-id="23acc-155">IOPS 和吞吐量（即数据传输速率）也取决于磁盘大小，因此在预配磁盘时，请全面考虑三个因素（容量、IOPS 和吞吐量）。</span><span class="sxs-lookup"><span data-stu-id="23acc-155">IOPS and throughput (that is, data transfer rate) also depend on disk size, so when you provision a disk, consider all three factors (capacity, IOPS, and throughput).</span></span> 

<span data-ttu-id="23acc-156">我们还建议使用[托管磁盘][managed-disks]。</span><span class="sxs-lookup"><span data-stu-id="23acc-156">We also recommend using [Managed Disks][managed-disks].</span></span> <span data-ttu-id="23acc-157">托管磁盘不需要存储帐户。</span><span class="sxs-lookup"><span data-stu-id="23acc-157">Managed disks do not require a storage account.</span></span> <span data-ttu-id="23acc-158">只需指定磁盘的大小和类型，就可以将它部署为高度可用的资源。</span><span class="sxs-lookup"><span data-stu-id="23acc-158">You simply specify the size and type of disk and it is deployed as a highly available resource.</span></span>

<span data-ttu-id="23acc-159">添加一个或多个数据磁盘。</span><span class="sxs-lookup"><span data-stu-id="23acc-159">Add one or more data disks.</span></span> <span data-ttu-id="23acc-160">刚创建的 VHD 尚未格式化，</span><span class="sxs-lookup"><span data-stu-id="23acc-160">When you create a VHD, it is unformatted.</span></span> <span data-ttu-id="23acc-161">登录到 VM 对磁盘进行格式化。</span><span class="sxs-lookup"><span data-stu-id="23acc-161">Log into the VM to format the disk.</span></span> <span data-ttu-id="23acc-162">如果可能，请将应用程序安装在数据磁盘上，而不是 OS 磁盘上。</span><span class="sxs-lookup"><span data-stu-id="23acc-162">When possible, install applications on a data disk, not the OS disk.</span></span> <span data-ttu-id="23acc-163">某些旧版应用程序可能需要将组件安装在 C: 驱动器上；在这种情况下，可使用 PowerShell [重设 OS 磁盘的大小][resize-os-disk]。</span><span class="sxs-lookup"><span data-stu-id="23acc-163">Some legacy applications might need to install components on the C: drive; in that case, you can [resize the OS disk][resize-os-disk] using PowerShell.</span></span>

<span data-ttu-id="23acc-164">创建用于保存诊断日志的存储帐户。</span><span class="sxs-lookup"><span data-stu-id="23acc-164">Create a storage account to hold diagnostic logs.</span></span> <span data-ttu-id="23acc-165">标准的本地冗余存储 (LRS) 帐户足以存储诊断日志。</span><span class="sxs-lookup"><span data-stu-id="23acc-165">A standard locally redundant storage (LRS) account is sufficient for diagnostic logs.</span></span>

> [!NOTE]
> <span data-ttu-id="23acc-166">如果不使用托管磁盘，请为每个 VM 创建单独的 Azure 存储帐户来存放虚拟硬盘 (VHD)，以避免达到存储帐户的 [(IOPS) 限制][vm-disk-limits]。</span><span class="sxs-lookup"><span data-stu-id="23acc-166">If you aren't using Managed Disks, create separate Azure storage accounts for each VM to hold the virtual hard disks (VHDs), in order to avoid hitting the [(IOPS) limits][vm-disk-limits] for storage accounts.</span></span> <span data-ttu-id="23acc-167">请注意存储帐户的总 I/O 限制。</span><span class="sxs-lookup"><span data-stu-id="23acc-167">Be aware of the total I/O limits of the storage account.</span></span> <span data-ttu-id="23acc-168">有关详细信息，请参阅[虚拟机磁盘限制][vm-disk-limits]。</span><span class="sxs-lookup"><span data-stu-id="23acc-168">For more information, see [virtual machine disk limits][vm-disk-limits].</span></span>


## <a name="network-recommendations"></a><span data-ttu-id="23acc-169">网络建议</span><span class="sxs-lookup"><span data-stu-id="23acc-169">Network recommendations</span></span>

<span data-ttu-id="23acc-170">公共 IP 地址可以是动态的或静态的。</span><span class="sxs-lookup"><span data-stu-id="23acc-170">The public IP address can be dynamic or static.</span></span> <span data-ttu-id="23acc-171">默认是动态的。</span><span class="sxs-lookup"><span data-stu-id="23acc-171">The default is dynamic.</span></span>

* <span data-ttu-id="23acc-172">如果需要不会更改的固定 IP 地址 &mdash; 例如，如果需要创建 DNS 'A' 记录或将 IP 地址添加到安全列表，请保留[静态 IP 地址][static-ip]。</span><span class="sxs-lookup"><span data-stu-id="23acc-172">Reserve a [static IP address][static-ip] if you need a fixed IP address that won't change &mdash; for example, if you need to create a DNS 'A' record or add the IP address to a safe list.</span></span>
* <span data-ttu-id="23acc-173">还可以为 IP 地址创建完全限定域名 (FQDN)。</span><span class="sxs-lookup"><span data-stu-id="23acc-173">You can also create a fully qualified domain name (FQDN) for the IP address.</span></span> <span data-ttu-id="23acc-174">然后，可在 DNS 中注册指向 FQDN 的 [CNAME 记录][cname-record]。</span><span class="sxs-lookup"><span data-stu-id="23acc-174">You can then register a [CNAME record][cname-record] in DNS that points to the FQDN.</span></span> <span data-ttu-id="23acc-175">有关详细信息，请参阅[在 Azure 门户中创建完全限定的域名][fqdn]。</span><span class="sxs-lookup"><span data-stu-id="23acc-175">For more information, see [Create a fully qualified domain name in the Azure portal][fqdn].</span></span>

<span data-ttu-id="23acc-176">所有 NSG 都包含一组[默认规则][nsg-default-rules]，其中包括阻止所有入站 Internet 流量的规则。</span><span class="sxs-lookup"><span data-stu-id="23acc-176">All NSGs contain a set of [default rules][nsg-default-rules], including a rule that blocks all inbound Internet traffic.</span></span> <span data-ttu-id="23acc-177">无法删除默认规则，但其他规则可以覆盖它们。</span><span class="sxs-lookup"><span data-stu-id="23acc-177">The default rules cannot be deleted, but other rules can override them.</span></span> <span data-ttu-id="23acc-178">要启用 Internet 流量，请创建允许特定端口的入站流量的规则 &mdash; 例如，将端口 80 用于 HTTP。</span><span class="sxs-lookup"><span data-stu-id="23acc-178">To enable Internet traffic, create rules that allow inbound traffic to specific ports &mdash; for example, port 80 for HTTP.</span></span>

<span data-ttu-id="23acc-179">若要启用 RDP，请添加允许 TCP 端口 3389 的入站流量的 NSG 规则。</span><span class="sxs-lookup"><span data-stu-id="23acc-179">To enable RDP, add an NSG rule that allows inbound traffic to TCP port 3389.</span></span>

## <a name="scalability-considerations"></a><span data-ttu-id="23acc-180">可伸缩性注意事项</span><span class="sxs-lookup"><span data-stu-id="23acc-180">Scalability considerations</span></span>

<span data-ttu-id="23acc-181">可以通过[更改 VM 大小][vm-resize]来扩展或缩小 VM。</span><span class="sxs-lookup"><span data-stu-id="23acc-181">You can scale a VM up or down by [changing the VM size][vm-resize].</span></span> <span data-ttu-id="23acc-182">要以水平方式横向扩展，请将两个或更多 VM 放在负载均衡器后面。</span><span class="sxs-lookup"><span data-stu-id="23acc-182">To scale out horizontally, put two or more VMs behind a load balancer.</span></span> <span data-ttu-id="23acc-183">有关详细信息，请参阅 [N 层参考体系结构](./n-tier-sql-server.md)。</span><span class="sxs-lookup"><span data-stu-id="23acc-183">For more information, see the [N-tier reference architecture](./n-tier-sql-server.md).</span></span>

## <a name="availability-considerations"></a><span data-ttu-id="23acc-184">可用性注意事项</span><span class="sxs-lookup"><span data-stu-id="23acc-184">Availability considerations</span></span>

<span data-ttu-id="23acc-185">为了提高可用性，请在可用性集中部署多个 VM。</span><span class="sxs-lookup"><span data-stu-id="23acc-185">For higher availability, deploy multiple VMs in an availability set.</span></span> <span data-ttu-id="23acc-186">这样还可提供更高的[服务级别协议 (SLA)][vm-sla]。</span><span class="sxs-lookup"><span data-stu-id="23acc-186">This also provides a higher [service level agreement (SLA)][vm-sla].</span></span>

<span data-ttu-id="23acc-187">VM 可能会受到[计划内维护][planned-maintenance]或[计划外维护][manage-vm-availability]的影响。</span><span class="sxs-lookup"><span data-stu-id="23acc-187">Your VM may be affected by [planned maintenance][planned-maintenance] or [unplanned maintenance][manage-vm-availability].</span></span> <span data-ttu-id="23acc-188">可以使用 [VM 重新启动日志][reboot-logs]来确定 VM 重新启动是否是由计划内维护导致的。</span><span class="sxs-lookup"><span data-stu-id="23acc-188">You can use [VM reboot logs][reboot-logs] to determine whether a VM reboot was caused by planned maintenance.</span></span>

<span data-ttu-id="23acc-189">若要防止在正常操作期间意外数据丢失（例如，由于用户错误），则还应使用 [Blob 快照][blob-snapshot]或其他工具实现时间点备份。</span><span class="sxs-lookup"><span data-stu-id="23acc-189">To protect against accidental data loss during normal operations (for example, because of user error), you should also implement point-in-time backups, using [blob snapshots][blob-snapshot] or another tool.</span></span>

## <a name="manageability-considerations"></a><span data-ttu-id="23acc-190">可管理性注意事项</span><span class="sxs-lookup"><span data-stu-id="23acc-190">Manageability considerations</span></span>

<span data-ttu-id="23acc-191">**资源组。**</span><span class="sxs-lookup"><span data-stu-id="23acc-191">**Resource groups.**</span></span> <span data-ttu-id="23acc-192">将共享相同生命周期、密切相关的资源放入同一[资源组][resource-manager-overview]。</span><span class="sxs-lookup"><span data-stu-id="23acc-192">Put closely associated resources that share the same lifecycle into the same [resource group][resource-manager-overview].</span></span> <span data-ttu-id="23acc-193">资源组可让你以组的形式部署和监视资源，并按资源组跟踪计费成本。</span><span class="sxs-lookup"><span data-stu-id="23acc-193">Resource groups allow you to deploy and monitor resources as a group and track billing costs by resource group.</span></span> <span data-ttu-id="23acc-194">还可以删除作为集的资源，这对于测试部署非常有用。</span><span class="sxs-lookup"><span data-stu-id="23acc-194">You can also delete resources as a set, which is very useful for test deployments.</span></span> <span data-ttu-id="23acc-195">指定有意义的资源名称，以便简化特定资源的查找并了解其角色。</span><span class="sxs-lookup"><span data-stu-id="23acc-195">Assign meaningful resource names to simplify locating a specific resource and understanding its role.</span></span> <span data-ttu-id="23acc-196">有关详细信息，请参阅 [Azure 资源的建议命名约定][naming-conventions]。</span><span class="sxs-lookup"><span data-stu-id="23acc-196">For more information, see [Recommended Naming Conventions for Azure Resources][naming-conventions].</span></span>

<span data-ttu-id="23acc-197">**停止 VM。**</span><span class="sxs-lookup"><span data-stu-id="23acc-197">**Stopping a VM.**</span></span> <span data-ttu-id="23acc-198">Azure 对“已停止”和“已解除分配”状态做了区分。</span><span class="sxs-lookup"><span data-stu-id="23acc-198">Azure makes a distinction between "stopped" and "deallocated" states.</span></span> <span data-ttu-id="23acc-199">当 VM 状态为已停止时（而不是当 VM 已解除分配时）将向你收费。</span><span class="sxs-lookup"><span data-stu-id="23acc-199">You are charged when the VM status is stopped, but not when the VM is deallocated.</span></span> <span data-ttu-id="23acc-200">在 Azure 门户中，“停止”按钮可解除分配 VM。</span><span class="sxs-lookup"><span data-stu-id="23acc-200">In the Azure portal, the **Stop** button deallocates the VM.</span></span> <span data-ttu-id="23acc-201">如果在已登录时通过 OS 关闭，VM 会停止，但不会解除分配，因此仍会产生费用。</span><span class="sxs-lookup"><span data-stu-id="23acc-201">If you shut down through the OS while logged in, the VM is stopped but **not** deallocated, so you will still be charged.</span></span>

<span data-ttu-id="23acc-202">**删除 VM。**</span><span class="sxs-lookup"><span data-stu-id="23acc-202">**Deleting a VM.**</span></span> <span data-ttu-id="23acc-203">如果删除 VM，则不会删除 VHD。</span><span class="sxs-lookup"><span data-stu-id="23acc-203">If you delete a VM, the VHDs are not deleted.</span></span> <span data-ttu-id="23acc-204">这意味着可以安全地删除 VM，而不会丢失数据。</span><span class="sxs-lookup"><span data-stu-id="23acc-204">That means you can safely delete the VM without losing data.</span></span> <span data-ttu-id="23acc-205">但是，仍将向你收取存储费用。</span><span class="sxs-lookup"><span data-stu-id="23acc-205">However, you will still be charged for storage.</span></span> <span data-ttu-id="23acc-206">若要删除 VHD，请从 [Blob 存储][blob-storage]中删除相应的文件。</span><span class="sxs-lookup"><span data-stu-id="23acc-206">To delete the VHD, delete the file from [Blob storage][blob-storage].</span></span> <span data-ttu-id="23acc-207">要防止意外删除，请使用[资源锁][resource-lock]锁定整个资源组或锁定单个资源（如 VM）。</span><span class="sxs-lookup"><span data-stu-id="23acc-207">To prevent accidental deletion, use a [resource lock][resource-lock] to lock the entire resource group or lock individual resources, such as a VM.</span></span>

## <a name="security-considerations"></a><span data-ttu-id="23acc-208">安全注意事项</span><span class="sxs-lookup"><span data-stu-id="23acc-208">Security considerations</span></span>

<span data-ttu-id="23acc-209">使用 [Azure 安全中心][security-center]可在一个中心视图中获得 Azure 资源的安全状态。</span><span class="sxs-lookup"><span data-stu-id="23acc-209">Use [Azure Security Center][security-center] to get a central view of the security state of your Azure resources.</span></span> <span data-ttu-id="23acc-210">安全中心监视潜在的安全问题，并全面描述了部署的安全运行状况。</span><span class="sxs-lookup"><span data-stu-id="23acc-210">Security Center monitors potential security issues and provides a comprehensive picture of the security health of your deployment.</span></span> <span data-ttu-id="23acc-211">安全中心针对每个 Azure 订阅进行配置。</span><span class="sxs-lookup"><span data-stu-id="23acc-211">Security Center is configured per Azure subscription.</span></span> <span data-ttu-id="23acc-212">启用安全数据收集，如 [Azure 安全中心快速入门指南][security-center-get-started]中所述。</span><span class="sxs-lookup"><span data-stu-id="23acc-212">Enable security data collection as described in the [Azure Security Center quick start guide][security-center-get-started].</span></span> <span data-ttu-id="23acc-213">启用数据收集后，安全中心会自动扫描该订阅下创建的所有 VM。</span><span class="sxs-lookup"><span data-stu-id="23acc-213">When data collection is enabled, Security Center automatically scans any VMs created under that subscription.</span></span>

<span data-ttu-id="23acc-214">**修补程序管理。**</span><span class="sxs-lookup"><span data-stu-id="23acc-214">**Patch management.**</span></span> <span data-ttu-id="23acc-215">如果启用，安全中心会检查是否缺少任何安全更新和关键更新。</span><span class="sxs-lookup"><span data-stu-id="23acc-215">If enabled, Security Center checks whether any security and critical updates are missing.</span></span> <span data-ttu-id="23acc-216">使用 VM 上的[组策略设置][group-policy]可实现自动系统更新。</span><span class="sxs-lookup"><span data-stu-id="23acc-216">Use [Group Policy settings][group-policy] on the VM to enable automatic system updates.</span></span>

<span data-ttu-id="23acc-217">**反恶意软件。**</span><span class="sxs-lookup"><span data-stu-id="23acc-217">**Antimalware.**</span></span> <span data-ttu-id="23acc-218">如果启用，安全中心会检查是否已安装反恶意软件。</span><span class="sxs-lookup"><span data-stu-id="23acc-218">If enabled, Security Center checks whether antimalware software is installed.</span></span> <span data-ttu-id="23acc-219">还可以使用安全中心从 Azure 门户中安装反恶意软件。</span><span class="sxs-lookup"><span data-stu-id="23acc-219">You can also use Security Center to install antimalware software from inside the Azure portal.</span></span>

<span data-ttu-id="23acc-220">**操作。**</span><span class="sxs-lookup"><span data-stu-id="23acc-220">**Operations.**</span></span> <span data-ttu-id="23acc-221">使用[基于角色的访问控制 (RBAC)][rbac] 来控制对部署的 Azure 资源的访问。</span><span class="sxs-lookup"><span data-stu-id="23acc-221">Use [role-based access control (RBAC)][rbac] to control access to the Azure resources that you deploy.</span></span> <span data-ttu-id="23acc-222">RBAC 允许将授权角色分配给开发运营团队的成员。</span><span class="sxs-lookup"><span data-stu-id="23acc-222">RBAC lets you assign authorization roles to members of your DevOps team.</span></span> <span data-ttu-id="23acc-223">例如，“读者”角色可以查看 Azure 资源，但不能创建、管理或删除这些资源。</span><span class="sxs-lookup"><span data-stu-id="23acc-223">For example, the Reader role can view Azure resources but not create, manage, or delete them.</span></span> <span data-ttu-id="23acc-224">某些角色特定于特定的 Azure 资源类型。</span><span class="sxs-lookup"><span data-stu-id="23acc-224">Some roles are specific to particular Azure resource types.</span></span> <span data-ttu-id="23acc-225">例如，“虚拟机参与者”角色可以执行重启或解除分配 VM、重置管理员密码、创建新的 VM 等操作。</span><span class="sxs-lookup"><span data-stu-id="23acc-225">For example, the Virtual Machine Contributor role can restart or deallocate a VM, reset the administrator password, create a new VM, and so on.</span></span> <span data-ttu-id="23acc-226">可能对此体系结构有用的其他[内置 RBAC 角色][rbac-roles]包括[开发测试实验室用户][rbac-devtest]和[网络参与者][rbac-network]。</span><span class="sxs-lookup"><span data-stu-id="23acc-226">Other [built-in RBAC roles][rbac-roles] that may be useful for this architecture include [DevTest Labs User][rbac-devtest] and [Network Contributor][rbac-network].</span></span> <span data-ttu-id="23acc-227">可将用户分配给多个角色，并且可以创建自定义角色以实现更细化的权限。</span><span class="sxs-lookup"><span data-stu-id="23acc-227">A user can be assigned to multiple roles, and you can create custom roles for even more fine-grained permissions.</span></span>

> [!NOTE]
> <span data-ttu-id="23acc-228">RBAC 不限制已登录到 VM 的用户可以执行的操作。</span><span class="sxs-lookup"><span data-stu-id="23acc-228">RBAC does not limit the actions that a user logged into a VM can perform.</span></span> <span data-ttu-id="23acc-229">这些权限由来宾 OS 上的帐户类型决定。</span><span class="sxs-lookup"><span data-stu-id="23acc-229">Those permissions are determined by the account type on the guest OS.</span></span>   

<span data-ttu-id="23acc-230">使用[审核日志][audit-logs]可查看预配操作和其他 VM 事件。</span><span class="sxs-lookup"><span data-stu-id="23acc-230">Use [audit logs][audit-logs] to see provisioning actions and other VM events.</span></span>

<span data-ttu-id="23acc-231">**数据加密。**</span><span class="sxs-lookup"><span data-stu-id="23acc-231">**Data encryption.**</span></span> <span data-ttu-id="23acc-232">如果需要加密 OS 磁盘和数据磁盘，请考虑使用 [Azure 磁盘加密][disk-encryption]。</span><span class="sxs-lookup"><span data-stu-id="23acc-232">Consider [Azure Disk Encryption][disk-encryption] if you need to encrypt the OS and data disks.</span></span> 

## <a name="deploy-the-solution"></a><span data-ttu-id="23acc-233">部署解决方案</span><span class="sxs-lookup"><span data-stu-id="23acc-233">Deploy the solution</span></span>

<span data-ttu-id="23acc-234">[GitHub][github-folder] 上提供了此体系结构的部署。</span><span class="sxs-lookup"><span data-stu-id="23acc-234">A deployment for this architecture is available on [GitHub][github-folder].</span></span> <span data-ttu-id="23acc-235">它将部署以下部分：</span><span class="sxs-lookup"><span data-stu-id="23acc-235">It deploys the following:</span></span>

  * <span data-ttu-id="23acc-236">一个虚拟网络，其中包含用于托管 VM 的名为 **web** 的单个子网。</span><span class="sxs-lookup"><span data-stu-id="23acc-236">A virtual network with a single subnet named **web** used to host the VM.</span></span>
  * <span data-ttu-id="23acc-237">一个 NSG，其中包含两个用于允许 RDP 和 HTTP 流量发送到 VM 的传入规则。</span><span class="sxs-lookup"><span data-stu-id="23acc-237">An NSG with two incoming rules to allow RDP and HTTP traffic to the VM.</span></span>
  * <span data-ttu-id="23acc-238">一个运行 Windows Server 2016 Datacenter Edition 最新版本的 VM。</span><span class="sxs-lookup"><span data-stu-id="23acc-238">A VM running the latest version of Windows Server 2016 Datacenter Edition.</span></span>
  * <span data-ttu-id="23acc-239">一个用于格式化两个数据磁盘的示例自定义脚本扩展，另一个用于部署 Internet Information Services (IIS) 的 PowerShell DSC 脚本。</span><span class="sxs-lookup"><span data-stu-id="23acc-239">A sample custom script extension that formats the two data disks, and a PowerShell DSC script that deploys Internet Information Services (IIS).</span></span>

### <a name="prerequisites"></a><span data-ttu-id="23acc-240">先决条件</span><span class="sxs-lookup"><span data-stu-id="23acc-240">Prerequisites</span></span>

1. <span data-ttu-id="23acc-241">克隆、下载[参考体系结构][ref-arch-repo] GitHub 存储库的 zip 文件或创建其分支。</span><span class="sxs-lookup"><span data-stu-id="23acc-241">Clone, fork, or download the zip file for the [reference architectures][ref-arch-repo] GitHub repository.</span></span>

2. <span data-ttu-id="23acc-242">确保在计算机上安装了 Azure CLI 2.0。</span><span class="sxs-lookup"><span data-stu-id="23acc-242">Make sure you have the Azure CLI 2.0 installed on your computer.</span></span> <span data-ttu-id="23acc-243">有关 CLI 安装说明，请参阅[安装 Azure CLI 2.0][azure-cli-2]。</span><span class="sxs-lookup"><span data-stu-id="23acc-243">For CLI installation instructions, see [Install Azure CLI 2.0][azure-cli-2].</span></span>

3. <span data-ttu-id="23acc-244">安装 [Azure 构建基块][azbb] npm 包。</span><span class="sxs-lookup"><span data-stu-id="23acc-244">Install the [Azure building blocks][azbb] npm package.</span></span>

4. <span data-ttu-id="23acc-245">在命令提示符、bash 提示符或 PowerShell 提示符下，输入以下命令登录到 Azure 帐户。</span><span class="sxs-lookup"><span data-stu-id="23acc-245">From a command prompt, bash prompt, or PowerShell prompt, enter the following command to log into your Azure account.</span></span>

  ```bash
  az login
  ```

### <a name="deploy-the-solution-using-azbb"></a><span data-ttu-id="23acc-246">使用 azbb 部署解决方案</span><span class="sxs-lookup"><span data-stu-id="23acc-246">Deploy the solution using azbb</span></span>

<span data-ttu-id="23acc-247">若要部署此参考体系结构，请执行以下步骤：</span><span class="sxs-lookup"><span data-stu-id="23acc-247">To deploy this reference architecture, follow these steps:</span></span>

1. <span data-ttu-id="23acc-248">导航到已在前面的先决条件步骤中下载的存储库的 `virtual-machines\single-vm\parameters\windows` 文件夹。</span><span class="sxs-lookup"><span data-stu-id="23acc-248">Navigate to the `virtual-machines\single-vm\parameters\windows` folder for the repository you downloaded in the prerequisites step above.</span></span>

2. <span data-ttu-id="23acc-249">打开 `single-vm-v2.json` 文件，输入括在引号中的用户名和密码，然后保存文件。</span><span class="sxs-lookup"><span data-stu-id="23acc-249">Open the `single-vm-v2.json` file and enter a username and password between the quotes, then save the file.</span></span>

  ```bash
  "adminUsername": "",
  "adminPassword": "",
  ```

3. <span data-ttu-id="23acc-250">运行 `azbb` 以部署示例 VM，如下所示。</span><span class="sxs-lookup"><span data-stu-id="23acc-250">Run `azbb` to deploy the sample VM as shown below.</span></span>

  ```bash
  azbb -s <subscription_id> -g <resource_group_name> -l <location> -p single-vm-v2.json --deploy
  ```

<span data-ttu-id="23acc-251">若要验证部署，请运行以下 Azure CLI 命令，找到 VM 的公共 IP 地址：</span><span class="sxs-lookup"><span data-stu-id="23acc-251">To verify the deployment, run the following Azure CLI command to find the public IP address of the VM:</span></span>

```bash
az vm show -n ra-single-windows-vm1 -g <resource-group-name> -d -o table
```

<span data-ttu-id="23acc-252">如果在 Web 浏览器中导航到此地址，应会看到默认的 IIS 主页。</span><span class="sxs-lookup"><span data-stu-id="23acc-252">If you navigate to this address in a web browser, you should see the default IIS homepage.</span></span>

<span data-ttu-id="23acc-253">有关自定义此部署的信息，请访问我们的 [GitHub 存储库][git]。</span><span class="sxs-lookup"><span data-stu-id="23acc-253">For information about customizing this deployment, visit our [GitHub repository][git].</span></span>

<!-- links -->
[audit-logs]: https://azure.microsoft.com/blog/analyze-azure-audit-logs-in-powerbi-more/
[availability-set]: /azure/virtual-machines/virtual-machines-windows-create-availability-set
[azbb]: https://github.com/mspnp/template-building-blocks/wiki/Install-Azure-Building-Blocks
[azbbv2]: https://github.com/mspnp/template-building-blocks
[azure-cli-2]: /cli/azure/install-azure-cli?view=azure-cli-latest
[azure-dns]: /azure/dns/dns-overview
[azure-storage]: /azure/storage/storage-introduction
[blob-snapshot]: /azure/storage/storage-blob-snapshots
[blob-storage]: /azure/storage/storage-introduction
[boot-diagnostics]: https://azure.microsoft.com/blog/boot-diagnostics-for-virtual-machines-v2/
[cname-record]: https://en.wikipedia.org/wiki/CNAME_record
[data-disk]: /azure/virtual-machines/virtual-machines-windows-about-disks-vhds
[disk-encryption]: /azure/security/azure-security-disk-encryption
[enable-monitoring]: /azure/monitoring-and-diagnostics/insights-how-to-use-diagnostics
[fqdn]: /azure/virtual-machines/virtual-machines-windows-portal-create-fqdn
[git]: https://github.com/mspnp/reference-architectures/tree/master/virtual-machines/single-vm
[github-folder]: https://github.com/mspnp/reference-architectures/tree/master/virtual-machines/single-vm
[group-policy]: https://docs.microsoft.com/previous-versions/windows/it-pro/windows-server-2012-R2-and-2012/dn595129(v=ws.11)
[log-collector]: https://azure.microsoft.com/blog/simplifying-virtual-machine-troubleshooting-using-azure-log-collector/
[manage-vm-availability]: /azure/virtual-machines/virtual-machines-windows-manage-availability
[managed-disks]: /azure/storage/storage-managed-disks-overview
[naming-conventions]: ../../best-practices/naming-conventions.md
[nsg]: /azure/virtual-network/virtual-networks-nsg
[nsg-default-rules]: /azure/virtual-network/virtual-networks-nsg#default-rules
[planned-maintenance]: /azure/virtual-machines/virtual-machines-windows-planned-maintenance
[premium-storage]: /azure/virtual-machines/windows/premium-storage
[premium-storage-supported]: /azure/virtual-machines/windows/premium-storage#supported-vms
[rbac]: /azure/active-directory/role-based-access-control-what-is
[rbac-roles]: /azure/active-directory/role-based-access-built-in-roles
[rbac-devtest]: /azure/active-directory/role-based-access-built-in-roles#devtest-labs-user
[rbac-network]: /azure/active-directory/role-based-access-built-in-roles#network-contributor
[reboot-logs]: https://azure.microsoft.com/blog/viewing-vm-reboot-logs/
[ref-arch-repo]: https://github.com/mspnp/reference-architectures
[resize-os-disk]: /azure/virtual-machines/virtual-machines-windows-expand-os-disk
[resource-lock]: /azure/resource-group-lock-resources
[resource-manager-overview]: /azure/azure-resource-manager/resource-group-overview
[security-center]: /azure/security-center/security-center-intro
[security-center-get-started]: /azure/security-center/security-center-get-started
[select-vm-image]: /azure/virtual-machines/virtual-machines-windows-cli-ps-findimage
[services-by-region]: https://azure.microsoft.com/regions/#services
[static-ip]: /azure/virtual-network/virtual-networks-reserved-public-ip
[virtual-machine-sizes]: /azure/virtual-machines/virtual-machines-windows-sizes
[visio-download]: https://archcenter.blob.core.windows.net/cdn/vm-reference-architectures.vsdx
[vm-disk-limits]: /azure/azure-subscription-service-limits#virtual-machine-disk-limits
[vm-resize]: /azure/virtual-machines/virtual-machines-linux-change-vm-size
[vm-size-tables]: /azure/virtual-machines/virtual-machines-windows-sizes
[vm-sla]: https://azure.microsoft.com/support/legal/sla/virtual-machines
[0]: ./images/single-vm-diagram.png "Azure 中的单一 Windows VM 体系结构"
