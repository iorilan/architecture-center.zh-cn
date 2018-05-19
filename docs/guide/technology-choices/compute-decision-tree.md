---
title: Azure 计算服务的决策树
description: 用于选择计算服务的流程图
author: MikeWasson
ms.date: 04/21/2018
ms.openlocfilehash: 3dcfbd156d4fced863a56bcc8bb74483aa665f9f
ms.sourcegitcommit: 7ced70ebc11aa0df0dc0104092d3cc6ad5c28bd6
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/11/2018
---
# <a name="decision-tree-for-azure-compute-services"></a><span data-ttu-id="15afe-103">Azure 计算服务的决策树</span><span class="sxs-lookup"><span data-stu-id="15afe-103">Decision tree for Azure compute services</span></span>

<span data-ttu-id="15afe-104">Azure 提供多种方式来托管应用程序代码。</span><span class="sxs-lookup"><span data-stu-id="15afe-104">Azure offers a number of ways to host your application code.</span></span> <span data-ttu-id="15afe-105">术语“计算”指的是计算资源（应用程序在这些资源上运行）的承载模型。</span><span class="sxs-lookup"><span data-stu-id="15afe-105">The term *compute* refers to the hosting model for the computing resources that your application runs on.</span></span> <span data-ttu-id="15afe-106">以下流程图将会帮助你选择应用程序的计算服务。</span><span class="sxs-lookup"><span data-stu-id="15afe-106">The following flowchart will help you to choose a compute service for your application.</span></span> <span data-ttu-id="15afe-107">该流程图将引导你创建一组关键决策条件用于访问建议。</span><span class="sxs-lookup"><span data-stu-id="15afe-107">The flowchart guides you through a set of key decision criteria to reach a recommendation.</span></span> 

<span data-ttu-id="15afe-108">**请将此流程图视为起点。**</span><span class="sxs-lookup"><span data-stu-id="15afe-108">**Treat this flowchart as a starting point.**</span></span> <span data-ttu-id="15afe-109">每个应用程序有独特的要求，因此请将该建议作为起点。</span><span class="sxs-lookup"><span data-stu-id="15afe-109">Every application has unique requirements, so use the recommendation as a starting point.</span></span> <span data-ttu-id="15afe-110">然后执行更详细的评估，例如查看：</span><span class="sxs-lookup"><span data-stu-id="15afe-110">Then perform a more detailed evaluation, looking at aspects such as:</span></span>
 
- <span data-ttu-id="15afe-111">特征集</span><span class="sxs-lookup"><span data-stu-id="15afe-111">Feature set</span></span>
- [<span data-ttu-id="15afe-112">服务限制</span><span class="sxs-lookup"><span data-stu-id="15afe-112">Service limits</span></span>](/azure/azure-subscription-service-limits)
- [<span data-ttu-id="15afe-113">成本</span><span class="sxs-lookup"><span data-stu-id="15afe-113">Cost</span></span>](https://azure.microsoft.com/pricing/)
- [<span data-ttu-id="15afe-114">SLA</span><span class="sxs-lookup"><span data-stu-id="15afe-114">SLA</span></span>](https://azure.microsoft.com/support/legal/sla/)
- [<span data-ttu-id="15afe-115">区域可用性</span><span class="sxs-lookup"><span data-stu-id="15afe-115">Regional availability</span></span>](https://azure.microsoft.com/global-infrastructure/services/)
- <span data-ttu-id="15afe-116">开发人员生态系统和团队技能</span><span class="sxs-lookup"><span data-stu-id="15afe-116">Developer ecosystem and team skills</span></span>
- [<span data-ttu-id="15afe-117">计算比较表</span><span class="sxs-lookup"><span data-stu-id="15afe-117">Compute comparison tables</span></span>](./compute-comparison.md)

<span data-ttu-id="15afe-118">如果应用程序包括多个工作负荷，请单独评估每个工作负荷。</span><span class="sxs-lookup"><span data-stu-id="15afe-118">If your application consists of multiple workloads, evaluate each workload separately.</span></span> <span data-ttu-id="15afe-119">完整的解决方案可能会合并两个或更多个计算服务。</span><span class="sxs-lookup"><span data-stu-id="15afe-119">A complete solution may incorporate two or more compute services.</span></span>

![](../images/compute-decision-tree.svg)

