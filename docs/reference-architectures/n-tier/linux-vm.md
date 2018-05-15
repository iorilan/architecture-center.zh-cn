---
title: 在 Azure 上运行 Linux VM
description: 如何在 Azure 上运行 Linux VM，并注意可伸缩性、弹性、可管理性和安全性。
author: telmosampaio
ms.date: 04/03/2018
ms.openlocfilehash: f29b7225c2e0edbb1569c9e3a55d112d12041af8
ms.sourcegitcommit: a5e549c15a948f6fb5cec786dbddc8578af3be66
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/06/2018
---
# <a name="run-a-linux-vm-on-azure"></a><span data-ttu-id="58e8f-103">在 Azure 上运行 Linux VM</span><span class="sxs-lookup"><span data-stu-id="58e8f-103">Run a Linux VM on Azure</span></span>

<span data-ttu-id="58e8f-104">本文介绍一组经验证的做法，说明如何在 Azure 上运行 Linux 虚拟机 (VM)。</span><span class="sxs-lookup"><span data-stu-id="58e8f-104">This article describes a set of proven practices for running a Linux virtual machine (VM) on Azure.</span></span> <span data-ttu-id="58e8f-105">其中包括有关预配 VM 以及网络和存储组件的建议。</span><span class="sxs-lookup"><span data-stu-id="58e8f-105">It includes recommendations for provisioning the VM along with networking and storage components.</span></span> [<span data-ttu-id="58e8f-106">部署此解决方案。</span><span class="sxs-lookup"><span data-stu-id="58e8f-106">**Deploy this solution.**</span></span>](#deploy-the-solution)

<span data-ttu-id="58e8f-107">![[0]][0]</span><span class="sxs-lookup"><span data-stu-id="58e8f-107">![[0]][0]</span></span>

## <a name="components"></a><span data-ttu-id="58e8f-108">组件</span><span class="sxs-lookup"><span data-stu-id="58e8f-108">Components</span></span>

<span data-ttu-id="58e8f-109">除 VM 本身以外，预配 Azure VM 还需要其他一些组件，包括网络和存储资源。</span><span class="sxs-lookup"><span data-stu-id="58e8f-109">Provisioning an Azure VM requires some additional components besides the VM itself, including networking and storage resources.</span></span>

* <span data-ttu-id="58e8f-110">**资源组。**</span><span class="sxs-lookup"><span data-stu-id="58e8f-110">**Resource group.**</span></span> <span data-ttu-id="58e8f-111">[资源组][resource-manager-overview]是保存相关 Azure 资源的逻辑容器。</span><span class="sxs-lookup"><span data-stu-id="58e8f-111">A [resource group][resource-manager-overview] is a logical container that holds related Azure resources.</span></span> <span data-ttu-id="58e8f-112">一般情况下，可根据资源的生存期及其管理者将资源分组。</span><span class="sxs-lookup"><span data-stu-id="58e8f-112">In general, group resources based on their lifetime and who will manage them.</span></span> 

* <span data-ttu-id="58e8f-113">**VM**。</span><span class="sxs-lookup"><span data-stu-id="58e8f-113">**VM**.</span></span> <span data-ttu-id="58e8f-114">可以通过发布的映像列表或上传到 Azure Blob 存储的自定义托管映像或虚拟硬盘 (VHD) 文件来预配 VM。</span><span class="sxs-lookup"><span data-stu-id="58e8f-114">You can provision a VM from a list of published images, or from a custom managed image or virtual hard disk (VHD) file uploaded to Azure Blob storage.</span></span> <span data-ttu-id="58e8f-115">Azure 支持运行大量常用的 Linux 分发，包括 CentOS、Debian、Red Hat Enterprise、Ubuntu 和 FreeBSD。</span><span class="sxs-lookup"><span data-stu-id="58e8f-115">Azure supports running various popular Linux distributions, including CentOS, Debian, Red Hat Enterprise, Ubuntu, and FreeBSD.</span></span> <span data-ttu-id="58e8f-116">有关详细信息，请参阅 [Azure 和 Linux][azure-linux]。</span><span class="sxs-lookup"><span data-stu-id="58e8f-116">For more information, see [Azure and Linux][azure-linux].</span></span>

* <span data-ttu-id="58e8f-117">**托管磁盘**。</span><span class="sxs-lookup"><span data-stu-id="58e8f-117">**Managed Disks**.</span></span> <span data-ttu-id="58e8f-118">[Azure 托管磁盘][managed-disks]可代你处理存储，从而简化了磁盘管理。</span><span class="sxs-lookup"><span data-stu-id="58e8f-118">[Azure Managed Disks][managed-disks] simplify disk management by handling the storage for you.</span></span> <span data-ttu-id="58e8f-119">OS 磁盘是存储在 [Azure 存储][azure-storage]中的 VHD，因此即使主机关闭，OS 磁盘也仍然存在。</span><span class="sxs-lookup"><span data-stu-id="58e8f-119">The OS disk is a VHD stored in [Azure Storage][azure-storage], so it persists even when the host machine is down.</span></span> <span data-ttu-id="58e8f-120">对于 Linux VM，OS 磁盘是 `/dev/sda1`。</span><span class="sxs-lookup"><span data-stu-id="58e8f-120">For Linux VMs, the OS disk is `/dev/sda1`.</span></span> <span data-ttu-id="58e8f-121">我们还建议创建一个或多个[数据磁盘][data-disk]（用于保存应用程序数据的持久性 VHD）。</span><span class="sxs-lookup"><span data-stu-id="58e8f-121">We also recommend creating one or more [data disks][data-disk], which are persistent VHDs used for application data.</span></span> 

* <span data-ttu-id="58e8f-122">**临时磁盘。**</span><span class="sxs-lookup"><span data-stu-id="58e8f-122">**Temporary disk.**</span></span> <span data-ttu-id="58e8f-123">使用临时磁盘创建 VM。</span><span class="sxs-lookup"><span data-stu-id="58e8f-123">The VM is created with a temporary disk.</span></span> <span data-ttu-id="58e8f-124">此磁盘存储在主机的物理驱动器上。</span><span class="sxs-lookup"><span data-stu-id="58e8f-124">This disk is stored on a physical drive on the host machine.</span></span> <span data-ttu-id="58e8f-125">它并非保存在 Azure 存储中，并且在重启期间以及发生其他 VM 生命周期事件期间可能会被删除。</span><span class="sxs-lookup"><span data-stu-id="58e8f-125">It is *not* saved in Azure Storage and may be deleted during reboots and other VM lifecycle events.</span></span> <span data-ttu-id="58e8f-126">只使用此磁盘存储临时数据，如页面文件或交换文件。</span><span class="sxs-lookup"><span data-stu-id="58e8f-126">Use this disk only for temporary data, such as page or swap files.</span></span> <span data-ttu-id="58e8f-127">对于 Linux VM，临时磁盘是 `/dev/sdb1`，并在 `/mnt/resource` 或 `/mnt` 中装入。</span><span class="sxs-lookup"><span data-stu-id="58e8f-127">For Linux VMs, the temporary disk is `/dev/sdb1` and is mounted at `/mnt/resource` or `/mnt`.</span></span>

* <span data-ttu-id="58e8f-128">**虚拟网络 (VNet)**。</span><span class="sxs-lookup"><span data-stu-id="58e8f-128">**Virtual network (VNet).**</span></span> <span data-ttu-id="58e8f-129">每个 Azure VM 都会部署到可细分为多个子网的 VNet 中。</span><span class="sxs-lookup"><span data-stu-id="58e8f-129">Every Azure VM is deployed into a VNet that can be segmented into multiple subnets.</span></span>

* <span data-ttu-id="58e8f-130">**网络接口 (NIC)**。</span><span class="sxs-lookup"><span data-stu-id="58e8f-130">**Network interface (NIC)**.</span></span> <span data-ttu-id="58e8f-131">NIC 使 VM 能够与虚拟网络进行通信。</span><span class="sxs-lookup"><span data-stu-id="58e8f-131">The NIC enables the VM to communicate with the virtual network.</span></span>

* <span data-ttu-id="58e8f-132">**公共 IP 地址。**</span><span class="sxs-lookup"><span data-stu-id="58e8f-132">**Public IP address.**</span></span> <span data-ttu-id="58e8f-133">需要使用公共 IP 地址与 VM 通信 &mdash; 例如，通过 SSH。</span><span class="sxs-lookup"><span data-stu-id="58e8f-133">A public IP address is needed to communicate with the VM &mdash; for example, via SSH.</span></span>

* <span data-ttu-id="58e8f-134">**Azure DNS**。</span><span class="sxs-lookup"><span data-stu-id="58e8f-134">**Azure DNS**.</span></span> <span data-ttu-id="58e8f-135">[Azure DNS][azure-dns] 是 DNS 域的托管服务，它使用 Microsoft Azure 基础结构提供名称解析。</span><span class="sxs-lookup"><span data-stu-id="58e8f-135">[Azure DNS][azure-dns] is a hosting service for DNS domains, providing name resolution using Microsoft Azure infrastructure.</span></span> <span data-ttu-id="58e8f-136">通过在 Azure 中托管域，可以使用与其他 Azure 服务相同的凭据、API、工具和计费来管理 DNS 记录。</span><span class="sxs-lookup"><span data-stu-id="58e8f-136">By hosting your domains in Azure, you can manage your DNS records using the same credentials, APIs, tools, and billing as your other Azure services.</span></span>

* <span data-ttu-id="58e8f-137">**网络安全组 (NSG)**。</span><span class="sxs-lookup"><span data-stu-id="58e8f-137">**Network security group (NSG)**.</span></span> <span data-ttu-id="58e8f-138">[网络安全组][nsg]用于允许或拒绝向 VM 传送网络流量。</span><span class="sxs-lookup"><span data-stu-id="58e8f-138">[Network security groups][nsg] are used to allow or deny network traffic to VMs.</span></span> <span data-ttu-id="58e8f-139">NSG 可与子网或单个 VM 实例相关联。</span><span class="sxs-lookup"><span data-stu-id="58e8f-139">NSGs can be associated either with subnets or with individual VM instances.</span></span>

* <span data-ttu-id="58e8f-140">**诊断。**</span><span class="sxs-lookup"><span data-stu-id="58e8f-140">**Diagnostics.**</span></span> <span data-ttu-id="58e8f-141">诊断日志记录对于 VM 管理和故障排除至关重要。</span><span class="sxs-lookup"><span data-stu-id="58e8f-141">Diagnostic logging is crucial for managing and troubleshooting the VM.</span></span>

## <a name="vm-recommendations"></a><span data-ttu-id="58e8f-142">VM 建议</span><span class="sxs-lookup"><span data-stu-id="58e8f-142">VM recommendations</span></span>

<span data-ttu-id="58e8f-143">Azure 提供多种不同的虚拟机大小。</span><span class="sxs-lookup"><span data-stu-id="58e8f-143">Azure offers many different virtual machine sizes.</span></span> <span data-ttu-id="58e8f-144">有关详细信息，请参阅 [Azure 中虚拟机的大小][virtual-machine-sizes]。</span><span class="sxs-lookup"><span data-stu-id="58e8f-144">For more information, see [Sizes for virtual machines in Azure][virtual-machine-sizes].</span></span> <span data-ttu-id="58e8f-145">如果要将现有工作负荷转移到 Azure，开始时请先使用与本地服务器最匹配的 VM 大小。</span><span class="sxs-lookup"><span data-stu-id="58e8f-145">If you are moving an existing workload to Azure, start with the VM size that's the closest match to your on-premises servers.</span></span> <span data-ttu-id="58e8f-146">然后从 CPU、内存和每秒磁盘输入/输出操作次数 (IOPS) 等方面测量实际工作负荷的性能，并根据需要调整大小。</span><span class="sxs-lookup"><span data-stu-id="58e8f-146">Then measure the performance of your actual workload with respect to CPU, memory, and disk input/output operations per second (IOPS), and adjust the size as needed.</span></span> <span data-ttu-id="58e8f-147">如果 VM 需要多个 NIC，请注意每种 [VM 大小][vm-size-tables]都定义了最大 NIC 数量。</span><span class="sxs-lookup"><span data-stu-id="58e8f-147">If you require multiple NICs for your VM, be aware that a maximum number of NICs is defined for each [VM size][vm-size-tables].</span></span>

<span data-ttu-id="58e8f-148">通常选择离内部用户或客户最近的 Azure 区域。</span><span class="sxs-lookup"><span data-stu-id="58e8f-148">Generally, choose an Azure region that is closest to your internal users or customers.</span></span> <span data-ttu-id="58e8f-149">但是，并非所有 VM 大小都可在所有区域中使用。</span><span class="sxs-lookup"><span data-stu-id="58e8f-149">However, not all VM sizes are available in all regions.</span></span> <span data-ttu-id="58e8f-150">有关详细信息，请参阅[每个区域的服务][services-by-region]。</span><span class="sxs-lookup"><span data-stu-id="58e8f-150">For more information, see [Services by region][services-by-region].</span></span> <span data-ttu-id="58e8f-151">要获取特定区域中可用 VM 大小的列表，请从 Azure 命令行接口 (CLI) 运行以下命令：</span><span class="sxs-lookup"><span data-stu-id="58e8f-151">For a list of the VM sizes available in a specific region, run the following command from the Azure command-line interface (CLI):</span></span>

```
az vm list-sizes --location <location>
```

<span data-ttu-id="58e8f-152">要了解如何选择发布的 VM 映像，请参阅[查找 Linux VM 映像][select-vm-image]。</span><span class="sxs-lookup"><span data-stu-id="58e8f-152">For information about choosing a published VM image, see [Find Linux VM images][select-vm-image].</span></span>

<span data-ttu-id="58e8f-153">启用监视和诊断，包括基本运行状况指标、诊断基础结构日志和[启动诊断][boot-diagnostics]。</span><span class="sxs-lookup"><span data-stu-id="58e8f-153">Enable monitoring and diagnostics, including basic health metrics, diagnostics infrastructure logs, and [boot diagnostics][boot-diagnostics].</span></span> <span data-ttu-id="58e8f-154">如果 VM 陷入不可启动状态，启动诊断有助于诊断启动故障。</span><span class="sxs-lookup"><span data-stu-id="58e8f-154">Boot diagnostics can help you diagnose boot failure if your VM gets into a non-bootable state.</span></span> <span data-ttu-id="58e8f-155">有关详细信息，请参阅[启用监视和诊断][enable-monitoring]。</span><span class="sxs-lookup"><span data-stu-id="58e8f-155">For more information, see [Enable monitoring and diagnostics][enable-monitoring].</span></span>  

## <a name="disk-and-storage-recommendations"></a><span data-ttu-id="58e8f-156">磁盘和存储建议</span><span class="sxs-lookup"><span data-stu-id="58e8f-156">Disk and storage recommendations</span></span>

<span data-ttu-id="58e8f-157">为获得最佳磁盘 I/O 性能，建议使用[高级存储][premium-storage]，它在固态硬盘 (SSD) 上存储数据。</span><span class="sxs-lookup"><span data-stu-id="58e8f-157">For best disk I/O performance, we recommend [Premium Storage][premium-storage], which stores data on solid-state drives (SSDs).</span></span> <span data-ttu-id="58e8f-158">成本取决于预配磁盘的容量。</span><span class="sxs-lookup"><span data-stu-id="58e8f-158">Cost is based on the capacity of the provisioned disk.</span></span> <span data-ttu-id="58e8f-159">IOPS 和吞吐量（即数据传输速率）也取决于磁盘大小，因此在预配磁盘时，请全面考虑三个因素（容量、IOPS 和吞吐量）。</span><span class="sxs-lookup"><span data-stu-id="58e8f-159">IOPS and throughput (that is, data transfer rate) also depend on disk size, so when you provision a disk, consider all three factors (capacity, IOPS, and throughput).</span></span> 

<span data-ttu-id="58e8f-160">我们还建议使用[托管磁盘][managed-disks]。</span><span class="sxs-lookup"><span data-stu-id="58e8f-160">We also recommend using [Managed Disks][managed-disks].</span></span> <span data-ttu-id="58e8f-161">托管磁盘不需要存储帐户。</span><span class="sxs-lookup"><span data-stu-id="58e8f-161">Managed disks do not require a storage account.</span></span> <span data-ttu-id="58e8f-162">只需指定磁盘的大小和类型，就可以将它部署为高度可用的资源。</span><span class="sxs-lookup"><span data-stu-id="58e8f-162">You simply specify the size and type of disk and it is deployed as a highly available resource.</span></span>

<span data-ttu-id="58e8f-163">添加一个或多个数据磁盘。</span><span class="sxs-lookup"><span data-stu-id="58e8f-163">Add one or more data disks.</span></span> <span data-ttu-id="58e8f-164">刚创建的 VHD 尚未格式化，</span><span class="sxs-lookup"><span data-stu-id="58e8f-164">When you create a VHD, it is unformatted.</span></span> <span data-ttu-id="58e8f-165">登录到 VM 对磁盘进行格式化。</span><span class="sxs-lookup"><span data-stu-id="58e8f-165">Log into the VM to format the disk.</span></span> <span data-ttu-id="58e8f-166">在 Linux shell 中，数据磁盘显示为 `/dev/sdc`、`/dev/sdd` 等。</span><span class="sxs-lookup"><span data-stu-id="58e8f-166">In the Linux shell, data disks are displayed as `/dev/sdc`, `/dev/sdd`, and so on.</span></span> <span data-ttu-id="58e8f-167">可以运行 `lsblk` 以列出块设备，包括磁盘。</span><span class="sxs-lookup"><span data-stu-id="58e8f-167">You can run `lsblk` to list the block devices, including the disks.</span></span> <span data-ttu-id="58e8f-168">要使用数据磁盘，请创建一个分区和文件系统，并装载磁盘。</span><span class="sxs-lookup"><span data-stu-id="58e8f-168">To use a data disk, create a partition and file system, and mount the disk.</span></span> <span data-ttu-id="58e8f-169">例如：</span><span class="sxs-lookup"><span data-stu-id="58e8f-169">For example:</span></span>

```bat
# Create a partition.
sudo fdisk /dev/sdc     # Enter 'n' to partition, 'w' to write the change.

# Create a file system.
sudo mkfs -t ext3 /dev/sdc1

# Mount the drive.
sudo mkdir /data1
sudo mount /dev/sdc1 /data1
```

<span data-ttu-id="58e8f-170">在添加数据磁盘时，将为磁盘分配逻辑单元号 (LUN) ID。</span><span class="sxs-lookup"><span data-stu-id="58e8f-170">When you add a data disk, a logical unit number (LUN) ID is assigned to the disk.</span></span> <span data-ttu-id="58e8f-171">或者，可以指定 LUN ID &mdash; 例如，若要更换磁盘并保留相同的 LUN ID，或者应用程序要查找特定 LUN ID。</span><span class="sxs-lookup"><span data-stu-id="58e8f-171">Optionally, you can specify the LUN ID &mdash; for example, if you're replacing a disk and want to retain the same LUN ID, or you have an application that looks for a specific LUN ID.</span></span> <span data-ttu-id="58e8f-172">但请记住，每个磁盘的 LUN ID 必须唯一。</span><span class="sxs-lookup"><span data-stu-id="58e8f-172">However, remember that LUN IDs must be unique for each disk.</span></span>

<span data-ttu-id="58e8f-173">可能会需要更改 I/O 计划程序，以便针对 SSD 的性能进行优化，因为使用高级存储帐户的 VM 的磁盘是 SSD。</span><span class="sxs-lookup"><span data-stu-id="58e8f-173">You may want to change the I/O scheduler to optimize for performance on SSDs because the disks for VMs with premium storage accounts are SSDs.</span></span> <span data-ttu-id="58e8f-174">常见的建议是对 SSD 使用 NOOP 计划程序，但应使用 [iostat] 等工具来监视工作负荷的磁盘 I/O 性能。</span><span class="sxs-lookup"><span data-stu-id="58e8f-174">A common recommendation is to use the NOOP scheduler for SSDs, but you should use a tool such as [iostat] to monitor disk I/O performance for your workload.</span></span>

<span data-ttu-id="58e8f-175">创建用于保存诊断日志的存储帐户。</span><span class="sxs-lookup"><span data-stu-id="58e8f-175">Create a storage account to hold diagnostic logs.</span></span> <span data-ttu-id="58e8f-176">标准的本地冗余存储 (LRS) 帐户足以存储诊断日志。</span><span class="sxs-lookup"><span data-stu-id="58e8f-176">A standard locally redundant storage (LRS) account is sufficient for diagnostic logs.</span></span>

> [!NOTE]
> <span data-ttu-id="58e8f-177">如果不使用托管磁盘，请为每个 VM 创建单独的 Azure 存储帐户来存放虚拟硬盘 (VHD)，以避免达到存储帐户的 [(IOPS) 限制][vm-disk-limits]。</span><span class="sxs-lookup"><span data-stu-id="58e8f-177">If you aren't using Managed Disks, create separate Azure storage accounts for each VM to hold the virtual hard disks (VHDs), in order to avoid hitting the [(IOPS) limits][vm-disk-limits] for storage accounts.</span></span> <span data-ttu-id="58e8f-178">请注意存储帐户的总 I/O 限制。</span><span class="sxs-lookup"><span data-stu-id="58e8f-178">Be aware of the total I/O limits of the storage account.</span></span> <span data-ttu-id="58e8f-179">有关详细信息，请参阅[虚拟机磁盘限制][vm-disk-limits]。</span><span class="sxs-lookup"><span data-stu-id="58e8f-179">For more information, see [virtual machine disk limits][vm-disk-limits].</span></span>

## <a name="network-recommendations"></a><span data-ttu-id="58e8f-180">网络建议</span><span class="sxs-lookup"><span data-stu-id="58e8f-180">Network recommendations</span></span>

<span data-ttu-id="58e8f-181">公共 IP 地址可以是动态的或静态的。</span><span class="sxs-lookup"><span data-stu-id="58e8f-181">The public IP address can be dynamic or static.</span></span> <span data-ttu-id="58e8f-182">默认是动态的。</span><span class="sxs-lookup"><span data-stu-id="58e8f-182">The default is dynamic.</span></span>

* <span data-ttu-id="58e8f-183">如果需要不会更改的固定 IP 地址（例如，如果需要在 DNS 中创建 A 记录，或者需要将 IP 地址添加到安全列表），请保留[静态 IP 地址][static-ip]。</span><span class="sxs-lookup"><span data-stu-id="58e8f-183">Reserve a [static IP address][static-ip] if you need a fixed IP address that won't change &mdash; for example, if you need to create an A record in DNS, or need the IP address to be added to a safe list.</span></span>
* <span data-ttu-id="58e8f-184">还可以为 IP 地址创建完全限定域名 (FQDN)。</span><span class="sxs-lookup"><span data-stu-id="58e8f-184">You can also create a fully qualified domain name (FQDN) for the IP address.</span></span> <span data-ttu-id="58e8f-185">然后，可在 DNS 中注册指向 FQDN 的 [CNAME 记录][cname-record]。</span><span class="sxs-lookup"><span data-stu-id="58e8f-185">You can then register a [CNAME record][cname-record] in DNS that points to the FQDN.</span></span> <span data-ttu-id="58e8f-186">有关详细信息，请参阅[在 Azure 门户中创建完全限定的域名][fqdn]。</span><span class="sxs-lookup"><span data-stu-id="58e8f-186">For more information, see [Create a fully qualified domain name in the Azure portal][fqdn].</span></span> <span data-ttu-id="58e8f-187">可以使用 [Azure DNS][azure-dns] 或其他 DNS 服务。</span><span class="sxs-lookup"><span data-stu-id="58e8f-187">You can use [Azure DNS][azure-dns] or another DNS service.</span></span>

<span data-ttu-id="58e8f-188">所有 NSG 都包含一组[默认规则][nsg-default-rules]，其中包括阻止所有入站 Internet 流量的规则。</span><span class="sxs-lookup"><span data-stu-id="58e8f-188">All NSGs contain a set of [default rules][nsg-default-rules], including a rule that blocks all inbound Internet traffic.</span></span> <span data-ttu-id="58e8f-189">无法删除默认规则，但其他规则可以覆盖它们。</span><span class="sxs-lookup"><span data-stu-id="58e8f-189">The default rules cannot be deleted, but other rules can override them.</span></span> <span data-ttu-id="58e8f-190">要启用 Internet 流量，请创建允许特定端口的入站流量的规则 &mdash; 例如，将端口 80 用于 HTTP。</span><span class="sxs-lookup"><span data-stu-id="58e8f-190">To enable Internet traffic, create rules that allow inbound traffic to specific ports &mdash; for example, port 80 for HTTP.</span></span>

<span data-ttu-id="58e8f-191">要启用 SSH，请添加允许 TCP 端口 22 的入站流量的 NSG 规则。</span><span class="sxs-lookup"><span data-stu-id="58e8f-191">To enable SSH, add an NSG rule that allows inbound traffic to TCP port 22.</span></span>

## <a name="scalability-considerations"></a><span data-ttu-id="58e8f-192">可伸缩性注意事项</span><span class="sxs-lookup"><span data-stu-id="58e8f-192">Scalability considerations</span></span>

<span data-ttu-id="58e8f-193">可以通过[更改 VM 大小][vm-resize]来扩展或缩小 VM。</span><span class="sxs-lookup"><span data-stu-id="58e8f-193">You can scale a VM up or down by [changing the VM size][vm-resize].</span></span> <span data-ttu-id="58e8f-194">要以水平方式横向扩展，请将两个或更多 VM 放在负载均衡器后面。</span><span class="sxs-lookup"><span data-stu-id="58e8f-194">To scale out horizontally, put two or more VMs behind a load balancer.</span></span> <span data-ttu-id="58e8f-195">有关详细信息，请参阅 [N 层参考体系结构](./n-tier-cassandra.md)。</span><span class="sxs-lookup"><span data-stu-id="58e8f-195">For more information, see the [N-tier reference architecture](./n-tier-cassandra.md).</span></span>

## <a name="availability-considerations"></a><span data-ttu-id="58e8f-196">可用性注意事项</span><span class="sxs-lookup"><span data-stu-id="58e8f-196">Availability considerations</span></span>

<span data-ttu-id="58e8f-197">为了提高可用性，请在可用性集中部署多个 VM。</span><span class="sxs-lookup"><span data-stu-id="58e8f-197">For higher availability, deploy multiple VMs in an availability set.</span></span> <span data-ttu-id="58e8f-198">这样还可提供更高的[服务级别协议 (SLA)][vm-sla]。</span><span class="sxs-lookup"><span data-stu-id="58e8f-198">This also provides a higher [service level agreement (SLA)][vm-sla].</span></span>

<span data-ttu-id="58e8f-199">VM 可能会受到[计划内维护][planned-maintenance]或[计划外维护][manage-vm-availability]的影响。</span><span class="sxs-lookup"><span data-stu-id="58e8f-199">Your VM may be affected by [planned maintenance][planned-maintenance] or [unplanned maintenance][manage-vm-availability].</span></span> <span data-ttu-id="58e8f-200">可以使用 [VM 重新启动日志][reboot-logs]来确定 VM 重新启动是否是由计划内维护导致的。</span><span class="sxs-lookup"><span data-stu-id="58e8f-200">You can use [VM reboot logs][reboot-logs] to determine whether a VM reboot was caused by planned maintenance.</span></span>

<span data-ttu-id="58e8f-201">若要防止在正常操作期间意外数据丢失（例如，由于用户错误），则还应使用 [Blob 快照][blob-snapshot]或其他工具实现时间点备份。</span><span class="sxs-lookup"><span data-stu-id="58e8f-201">To protect against accidental data loss during normal operations (for example, because of user error), you should also implement point-in-time backups, using [blob snapshots][blob-snapshot] or another tool.</span></span>

## <a name="manageability-considerations"></a><span data-ttu-id="58e8f-202">可管理性注意事项</span><span class="sxs-lookup"><span data-stu-id="58e8f-202">Manageability considerations</span></span>

<span data-ttu-id="58e8f-203">**资源组。**</span><span class="sxs-lookup"><span data-stu-id="58e8f-203">**Resource groups.**</span></span> <span data-ttu-id="58e8f-204">将共享相同生命周期、密切相关的资源放入同一[资源组][resource-manager-overview]。</span><span class="sxs-lookup"><span data-stu-id="58e8f-204">Put closely associated resources that share the same lifecycle into the same [resource group][resource-manager-overview].</span></span> <span data-ttu-id="58e8f-205">资源组可让你以组的形式部署和监视资源，并按资源组跟踪计费成本。</span><span class="sxs-lookup"><span data-stu-id="58e8f-205">Resource groups allow you to deploy and monitor resources as a group and track billing costs by resource group.</span></span> <span data-ttu-id="58e8f-206">还可以删除作为集的资源，这对于测试部署非常有用。</span><span class="sxs-lookup"><span data-stu-id="58e8f-206">You can also delete resources as a set, which is very useful for test deployments.</span></span> <span data-ttu-id="58e8f-207">指定有意义的资源名称，以便简化特定资源的查找并了解其角色。</span><span class="sxs-lookup"><span data-stu-id="58e8f-207">Assign meaningful resource names to simplify locating a specific resource and understanding its role.</span></span> <span data-ttu-id="58e8f-208">有关详细信息，请参阅 [Azure 资源的建议命名约定][naming-conventions]。</span><span class="sxs-lookup"><span data-stu-id="58e8f-208">For more information, see [Recommended naming conventions for Azure resources][naming-conventions].</span></span>

<span data-ttu-id="58e8f-209">**SSH**。</span><span class="sxs-lookup"><span data-stu-id="58e8f-209">**SSH**.</span></span> <span data-ttu-id="58e8f-210">在创建 Linux VM 之前，生成 2048 位 RSA 公共/专用密钥对。</span><span class="sxs-lookup"><span data-stu-id="58e8f-210">Before you create a Linux VM, generate a 2048-bit RSA public-private key pair.</span></span> <span data-ttu-id="58e8f-211">创建 VM 时，使用公钥文件。</span><span class="sxs-lookup"><span data-stu-id="58e8f-211">Use the public key file when you create the VM.</span></span> <span data-ttu-id="58e8f-212">有关详细信息，请参阅[如何在 Azure 中将 SSH 与 Linux 和 Mac 配合使用][ssh-linux]。</span><span class="sxs-lookup"><span data-stu-id="58e8f-212">For more information, see [How to Use SSH with Linux and Mac on Azure][ssh-linux].</span></span>

<span data-ttu-id="58e8f-213">**停止 VM。**</span><span class="sxs-lookup"><span data-stu-id="58e8f-213">**Stopping a VM.**</span></span> <span data-ttu-id="58e8f-214">Azure 对“已停止”和“已解除分配”状态做了区分。</span><span class="sxs-lookup"><span data-stu-id="58e8f-214">Azure makes a distinction between "stopped" and "deallocated" states.</span></span> <span data-ttu-id="58e8f-215">当 VM 状态为已停止时（而不是当 VM 已解除分配时）将向你收费。</span><span class="sxs-lookup"><span data-stu-id="58e8f-215">You are charged when the VM status is stopped, but not when the VM is deallocated.</span></span> <span data-ttu-id="58e8f-216">在 Azure 门户中，“停止”按钮可解除分配 VM。</span><span class="sxs-lookup"><span data-stu-id="58e8f-216">In the Azure portal, the **Stop** button deallocates the VM.</span></span> <span data-ttu-id="58e8f-217">如果在已登录时通过 OS 关闭，VM 会停止，但不会解除分配，因此仍会产生费用。</span><span class="sxs-lookup"><span data-stu-id="58e8f-217">If you shut down through the OS while logged in, the VM is stopped but **not** deallocated, so you will still be charged.</span></span>

<span data-ttu-id="58e8f-218">**删除 VM。**</span><span class="sxs-lookup"><span data-stu-id="58e8f-218">**Deleting a VM.**</span></span> <span data-ttu-id="58e8f-219">如果删除 VM，则不会删除 VHD。</span><span class="sxs-lookup"><span data-stu-id="58e8f-219">If you delete a VM, the VHDs are not deleted.</span></span> <span data-ttu-id="58e8f-220">这意味着可以安全地删除 VM，而不会丢失数据。</span><span class="sxs-lookup"><span data-stu-id="58e8f-220">That means you can safely delete the VM without losing data.</span></span> <span data-ttu-id="58e8f-221">但是，仍将向你收取存储费用。</span><span class="sxs-lookup"><span data-stu-id="58e8f-221">However, you will still be charged for storage.</span></span> <span data-ttu-id="58e8f-222">若要删除 VHD，请从 [Blob 存储][blob-storage]中删除相应的文件。</span><span class="sxs-lookup"><span data-stu-id="58e8f-222">To delete the VHD, delete the file from [Blob storage][blob-storage].</span></span> <span data-ttu-id="58e8f-223">要防止意外删除，请使用[资源锁][resource-lock]锁定整个资源组或锁定单个资源（如 VM）。</span><span class="sxs-lookup"><span data-stu-id="58e8f-223">To prevent accidental deletion, use a [resource lock][resource-lock] to lock the entire resource group or lock individual resources, such as a VM.</span></span>

## <a name="security-considerations"></a><span data-ttu-id="58e8f-224">安全注意事项</span><span class="sxs-lookup"><span data-stu-id="58e8f-224">Security considerations</span></span>

<span data-ttu-id="58e8f-225">使用 [Azure 安全中心][security-center]可在一个中心视图中获得 Azure 资源的安全状态。</span><span class="sxs-lookup"><span data-stu-id="58e8f-225">Use [Azure Security Center][security-center] to get a central view of the security state of your Azure resources.</span></span> <span data-ttu-id="58e8f-226">安全中心监视潜在的安全问题，并全面描述了部署的安全运行状况。</span><span class="sxs-lookup"><span data-stu-id="58e8f-226">Security Center monitors potential security issues and provides a comprehensive picture of the security health of your deployment.</span></span> <span data-ttu-id="58e8f-227">安全中心针对每个 Azure 订阅进行配置。</span><span class="sxs-lookup"><span data-stu-id="58e8f-227">Security Center is configured per Azure subscription.</span></span> <span data-ttu-id="58e8f-228">启用安全数据收集，如 [Azure 安全中心快速入门指南][security-center-get-started]中所述。</span><span class="sxs-lookup"><span data-stu-id="58e8f-228">Enable security data collection as described in the [Azure Security Center quick start guide][security-center-get-started].</span></span> <span data-ttu-id="58e8f-229">启用数据收集后，安全中心会自动扫描该订阅下创建的所有 VM。</span><span class="sxs-lookup"><span data-stu-id="58e8f-229">When data collection is enabled, Security Center automatically scans any VMs created under that subscription.</span></span>

<span data-ttu-id="58e8f-230">**修补程序管理。**</span><span class="sxs-lookup"><span data-stu-id="58e8f-230">**Patch management.**</span></span> <span data-ttu-id="58e8f-231">如果启用，安全中心会检查是否缺少任何安全更新和关键更新。</span><span class="sxs-lookup"><span data-stu-id="58e8f-231">If enabled, Security Center checks whether any security and critical updates are missing.</span></span> 

<span data-ttu-id="58e8f-232">**反恶意软件。**</span><span class="sxs-lookup"><span data-stu-id="58e8f-232">**Antimalware.**</span></span> <span data-ttu-id="58e8f-233">如果启用，安全中心会检查是否已安装反恶意软件。</span><span class="sxs-lookup"><span data-stu-id="58e8f-233">If enabled, Security Center checks whether antimalware software is installed.</span></span> <span data-ttu-id="58e8f-234">还可以使用安全中心从 Azure 门户中安装反恶意软件。</span><span class="sxs-lookup"><span data-stu-id="58e8f-234">You can also use Security Center to install antimalware software from inside the Azure portal.</span></span>

<span data-ttu-id="58e8f-235">**操作。**</span><span class="sxs-lookup"><span data-stu-id="58e8f-235">**Operations.**</span></span> <span data-ttu-id="58e8f-236">使用[基于角色的访问控制 (RBAC)][rbac] 来控制对部署的 Azure 资源的访问。</span><span class="sxs-lookup"><span data-stu-id="58e8f-236">Use [role-based access control (RBAC)][rbac] to control access to the Azure resources that you deploy.</span></span> <span data-ttu-id="58e8f-237">RBAC 允许将授权角色分配给开发运营团队的成员。</span><span class="sxs-lookup"><span data-stu-id="58e8f-237">RBAC lets you assign authorization roles to members of your DevOps team.</span></span> <span data-ttu-id="58e8f-238">例如，“读者”角色可以查看 Azure 资源，但不能创建、管理或删除这些资源。</span><span class="sxs-lookup"><span data-stu-id="58e8f-238">For example, the Reader role can view Azure resources but not create, manage, or delete them.</span></span> <span data-ttu-id="58e8f-239">某些角色特定于特定的 Azure 资源类型。</span><span class="sxs-lookup"><span data-stu-id="58e8f-239">Some roles are specific to particular Azure resource types.</span></span> <span data-ttu-id="58e8f-240">例如，“虚拟机参与者”角色可以执行重启或解除分配 VM、重置管理员密码、创建新的 VM 等操作。</span><span class="sxs-lookup"><span data-stu-id="58e8f-240">For example, the Virtual Machine Contributor role can restart or deallocate a VM, reset the administrator password, create a new VM, and so on.</span></span> <span data-ttu-id="58e8f-241">可能对此体系结构有用的其他[内置 RBAC 角色][rbac-roles]包括[开发测试实验室用户][rbac-devtest]和[网络参与者][rbac-network]。</span><span class="sxs-lookup"><span data-stu-id="58e8f-241">Other [built-in RBAC roles][rbac-roles] that may be useful for this architecture include [DevTest Labs User][rbac-devtest] and [Network Contributor][rbac-network].</span></span> <span data-ttu-id="58e8f-242">可将用户分配给多个角色，并且可以创建自定义角色以实现更细化的权限。</span><span class="sxs-lookup"><span data-stu-id="58e8f-242">A user can be assigned to multiple roles, and you can create custom roles for even more fine-grained permissions.</span></span>

> [!NOTE]
> <span data-ttu-id="58e8f-243">RBAC 不限制已登录到 VM 的用户可以执行的操作。</span><span class="sxs-lookup"><span data-stu-id="58e8f-243">RBAC does not limit the actions that a user logged into a VM can perform.</span></span> <span data-ttu-id="58e8f-244">这些权限由来宾 OS 上的帐户类型决定。</span><span class="sxs-lookup"><span data-stu-id="58e8f-244">Those permissions are determined by the account type on the guest OS.</span></span>   

<span data-ttu-id="58e8f-245">使用[审核日志][audit-logs]可查看预配操作和其他 VM 事件。</span><span class="sxs-lookup"><span data-stu-id="58e8f-245">Use [audit logs][audit-logs] to see provisioning actions and other VM events.</span></span>

<span data-ttu-id="58e8f-246">**数据加密。**</span><span class="sxs-lookup"><span data-stu-id="58e8f-246">**Data encryption.**</span></span> <span data-ttu-id="58e8f-247">如果需要加密 OS 磁盘和数据磁盘，请考虑使用 [Azure 磁盘加密][disk-encryption]。</span><span class="sxs-lookup"><span data-stu-id="58e8f-247">Consider [Azure Disk Encryption][disk-encryption] if you need to encrypt the OS and data disks.</span></span> 

## <a name="deploy-the-solution"></a><span data-ttu-id="58e8f-248">部署解决方案</span><span class="sxs-lookup"><span data-stu-id="58e8f-248">Deploy the solution</span></span>

<span data-ttu-id="58e8f-249">[GitHub][github-folder] 上提供了一个部署。</span><span class="sxs-lookup"><span data-stu-id="58e8f-249">A deployment is available on [GitHub][github-folder].</span></span> <span data-ttu-id="58e8f-250">它将部署以下部分：</span><span class="sxs-lookup"><span data-stu-id="58e8f-250">It deploys the following:</span></span>

  * <span data-ttu-id="58e8f-251">一个虚拟网络，其中包含用于托管 VM 的名为 **web** 的单个子网。</span><span class="sxs-lookup"><span data-stu-id="58e8f-251">A virtual network with a single subnet named **web** used to host the VM.</span></span>
  * <span data-ttu-id="58e8f-252">一个 NSG，其中包含两个用于允许 SSH 和 HTTP 流量发送到 VM 的传入规则。</span><span class="sxs-lookup"><span data-stu-id="58e8f-252">An NSG with two incoming rules to allow SSH and HTTP traffic to the VM.</span></span>
  * <span data-ttu-id="58e8f-253">一个运行 Ubuntu 16.04.3 LTS 最新版本的 VM。</span><span class="sxs-lookup"><span data-stu-id="58e8f-253">A VM running the latest version of Ubuntu 16.04.3 LTS.</span></span>
  * <span data-ttu-id="58e8f-254">一个示例自定义脚本扩展，它会格式化两个数据磁盘并将 Apache HTTP Server 部署到 Ubuntu VM。</span><span class="sxs-lookup"><span data-stu-id="58e8f-254">A sample custom script extension that formats the two data disks and deploys Apache HTTP Server to the Ubuntu VM.</span></span>

### <a name="prerequisites"></a><span data-ttu-id="58e8f-255">先决条件</span><span class="sxs-lookup"><span data-stu-id="58e8f-255">Prerequisites</span></span>

1. <span data-ttu-id="58e8f-256">克隆、下载[参考体系结构][ref-arch-repo] GitHub 存储库的 zip 文件或创建其分支。</span><span class="sxs-lookup"><span data-stu-id="58e8f-256">Clone, fork, or download the zip file for the [reference architectures][ref-arch-repo] GitHub repository.</span></span>

2. <span data-ttu-id="58e8f-257">确保在计算机上安装了 Azure CLI 2.0。</span><span class="sxs-lookup"><span data-stu-id="58e8f-257">Make sure you have the Azure CLI 2.0 installed on your computer.</span></span> <span data-ttu-id="58e8f-258">有关 CLI 安装说明，请参阅[安装 Azure CLI 2.0][azure-cli-2]。</span><span class="sxs-lookup"><span data-stu-id="58e8f-258">For CLI installation instructions, see [Install Azure CLI 2.0][azure-cli-2].</span></span>

3. <span data-ttu-id="58e8f-259">安装 [Azure 构建基块][azbb] npm 包。</span><span class="sxs-lookup"><span data-stu-id="58e8f-259">Install the [Azure building blocks][azbb] npm package.</span></span>

4. <span data-ttu-id="58e8f-260">在命令提示符、bash 提示符或 PowerShell 提示符下，输入以下命令登录到 Azure 帐户。</span><span class="sxs-lookup"><span data-stu-id="58e8f-260">From a command prompt, bash prompt, or PowerShell prompt, enter the following command to log into your Azure account.</span></span>

  ```bash
  az login
  ```

5. <span data-ttu-id="58e8f-261">创建 SSH 密钥对。</span><span class="sxs-lookup"><span data-stu-id="58e8f-261">Create an SSH key pair.</span></span> <span data-ttu-id="58e8f-262">有关详细信息，请参阅[如何创建和使用适用于 Azure 中 Linux VM 的 SSH 公钥和私钥对](/azure/virtual-machines/linux/mac-create-ssh-keys)。</span><span class="sxs-lookup"><span data-stu-id="58e8f-262">For more information, see [How to create and use an SSH public and private key pair for Linux VMs in Azure](/azure/virtual-machines/linux/mac-create-ssh-keys).</span></span>

### <a name="deploy-the-solution-using-azbb"></a><span data-ttu-id="58e8f-263">使用 azbb 部署解决方案</span><span class="sxs-lookup"><span data-stu-id="58e8f-263">Deploy the solution using azbb</span></span>

1. <span data-ttu-id="58e8f-264">导航到已在前面的先决条件步骤中下载的存储库的 `virtual-machines/single-vm/parameters/linux` 文件夹。</span><span class="sxs-lookup"><span data-stu-id="58e8f-264">Navigate to the `virtual-machines/single-vm/parameters/linux` folder for the repository you downloaded in the prerequisites step above.</span></span>

2. <span data-ttu-id="58e8f-265">打开 `single-vm-v2.json` 文件并输入括在引号中的用户名和 SSH 公钥，然后保存文件。</span><span class="sxs-lookup"><span data-stu-id="58e8f-265">Open the `single-vm-v2.json` file and enter a username and your SSH public key between the quotes, then save the file.</span></span>

  ```bash
  "adminUsername": "<your username>",
  "sshPublicKey": "ssh-rsa AAAAB3NzaC1...",
  ```

3. <span data-ttu-id="58e8f-266">运行 `azbb` 以部署示例 VM，如下所示。</span><span class="sxs-lookup"><span data-stu-id="58e8f-266">Run `azbb` to deploy the sample VM as shown below.</span></span>

  ```bash
  azbb -s <subscription_id> -g <resource_group_name> -l <location> -p single-vm-v2.json --deploy
  ```

<span data-ttu-id="58e8f-267">若要验证部署，请运行以下 Azure CLI 命令，找到 VM 的公共 IP 地址：</span><span class="sxs-lookup"><span data-stu-id="58e8f-267">To verify the deployment, run the following Azure CLI command to find the public IP address of the VM:</span></span>

```bash
az vm show -n ra-single-linux-vm1 -g <resource-group-name> -d -o table
```

<span data-ttu-id="58e8f-268">如果在 Web 浏览器中导航到此地址，应会看到默认的 Apache2 主页。</span><span class="sxs-lookup"><span data-stu-id="58e8f-268">If you navigate to this address in a web browser, you should see the default Apache2 homepage.</span></span>

<!-- links -->
[audit-logs]: https://azure.microsoft.com/blog/analyze-azure-audit-logs-in-powerbi-more/
[availability-set]: /azure/virtual-machines/virtual-machines-linux-manage-availability
[azbb]: https://github.com/mspnp/template-building-blocks/wiki/Install-Azure-Building-Blocks
[azbbv2]: https://github.com/mspnp/template-building-blocks
[azure-cli-2]: /cli/azure/install-azure-cli?view=azure-cli-latest
[azure-linux]: /azure/virtual-machines/virtual-machines-linux-azure-overview
[azure-storage]: /azure/storage/storage-introduction
[blob-snapshot]: /azure/storage/storage-blob-snapshots
[blob-storage]: /azure/storage/storage-introduction
[boot-diagnostics]: https://azure.microsoft.com/blog/boot-diagnostics-for-virtual-machines-v2/
[cname-record]: https://en.wikipedia.org/wiki/CNAME_record
[data-disk]: /azure/virtual-machines/virtual-machines-linux-about-disks-vhds
[disk-encryption]: /azure/security/azure-security-disk-encryption
[enable-monitoring]: /azure/monitoring-and-diagnostics/insights-how-to-use-diagnostics
[azure-dns]: /azure/dns/dns-overview
[fqdn]: /azure/virtual-machines/virtual-machines-linux-portal-create-fqdn
[git]: https://github.com/mspnp/reference-architectures/tree/master/virtual-machines/single-vm
[github-folder]: https://github.com/mspnp/reference-architectures/tree/master/virtual-machines/single-vm
[iostat]: https://en.wikipedia.org/wiki/Iostat
[manage-vm-availability]: /azure/virtual-machines/virtual-machines-linux-manage-availability
[managed-disks]: /azure/storage/storage-managed-disks-overview
[naming-conventions]: /azure/architecture/best-practices/naming-conventions.md
[nsg]: /azure/virtual-network/virtual-networks-nsg
[nsg-default-rules]: /azure/virtual-network/virtual-networks-nsg#default-rules
[planned-maintenance]: /azure/virtual-machines/virtual-machines-linux-planned-maintenance
[premium-storage]: /azure/virtual-machines/linux/premium-storage
[premium-storage-supported]: /azure/virtual-machines/linux/premium-storage#supported-vms
[rbac]: /azure/active-directory/role-based-access-control-what-is
[rbac-roles]: /azure/active-directory/role-based-access-built-in-roles
[rbac-devtest]: /azure/active-directory/role-based-access-built-in-roles#devtest-labs-user
[rbac-network]: /azure/active-directory/role-based-access-built-in-roles#network-contributor
[reboot-logs]: https://azure.microsoft.com/blog/viewing-vm-reboot-logs/
[ref-arch-repo]: https://github.com/mspnp/reference-architectures
[resource-lock]: /azure/resource-group-lock-resources
[resource-manager-overview]: /azure/azure-resource-manager/resource-group-overview
[security-center]: /azure/security-center/security-center-intro
[security-center-get-started]: /azure/security-center/security-center-get-started
[select-vm-image]: /azure/virtual-machines/virtual-machines-linux-cli-ps-findimage
[services-by-region]: https://azure.microsoft.com/regions/#services
[ssh-linux]: /azure/virtual-machines/virtual-machines-linux-mac-create-ssh-keys
[static-ip]: /azure/virtual-network/virtual-networks-reserved-public-ip
[virtual-machine-sizes]: /azure/virtual-machines/virtual-machines-linux-sizes
[visio-download]: https://archcenter.blob.core.windows.net/cdn/vm-reference-architectures.vsdx
[vm-disk-limits]: /azure/azure-subscription-service-limits#virtual-machine-disk-limits
[vm-resize]: /azure/virtual-machines/virtual-machines-linux-change-vm-size
[vm-size-tables]: /azure/virtual-machines/virtual-machines-linux-sizes
[vm-sla]: https://azure.microsoft.com/support/legal/sla/virtual-machines
[0]: ./images/single-vm-diagram.png "Azure 中的单个 Linux VM"
