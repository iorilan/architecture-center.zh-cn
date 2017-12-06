---
title: "补偿事务"
description: "撤销一系列会共同定义最终一致操作的工作。"
keywords: "设计模式"
author: dragon119
ms.date: 06/23/2017
pnp.series.title: Cloud Design Patterns
pnp.pattern.categories: resiliency
ms.openlocfilehash: f8337717c4afd6b558f0da8e1ded3a8071340db7
ms.sourcegitcommit: b0482d49aab0526be386837702e7724c61232c60
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 11/14/2017
---
# <a name="compensating-transaction-pattern"></a><span data-ttu-id="6b8da-104">补偿事务模式</span><span class="sxs-lookup"><span data-stu-id="6b8da-104">Compensating Transaction pattern</span></span>

[!INCLUDE [header](../_includes/header.md)]

<span data-ttu-id="6b8da-105">撤消一系列步骤执行的工作，当一个或多个步骤失败时，这些步骤会共同定义最终一致的操作。</span><span class="sxs-lookup"><span data-stu-id="6b8da-105">Undo the work performed by a series of steps, which together define an eventually consistent operation, if one or more of the steps fail.</span></span> <span data-ttu-id="6b8da-106">遵循最终一致性模型的操作常见于实现复杂业务流程和工作流的云托管应用程序中。</span><span class="sxs-lookup"><span data-stu-id="6b8da-106">Operations that follow the eventual consistency model are commonly found in cloud-hosted applications that implement complex business processes and workflows.</span></span>

## <a name="context-and-problem"></a><span data-ttu-id="6b8da-107">上下文和问题</span><span class="sxs-lookup"><span data-stu-id="6b8da-107">Context and problem</span></span>

<span data-ttu-id="6b8da-108">云中运行的应用程序经常修改数据。</span><span class="sxs-lookup"><span data-stu-id="6b8da-108">Applications running in the cloud frequently modify data.</span></span> <span data-ttu-id="6b8da-109">该数据可能在不同的地理位置中的多个数据源间传播。</span><span class="sxs-lookup"><span data-stu-id="6b8da-109">This data might be spread across various data sources held in different geographic locations.</span></span> <span data-ttu-id="6b8da-110">为避免争用和提高分布式环境中的性能，应用程序不应提供事务强一致性。</span><span class="sxs-lookup"><span data-stu-id="6b8da-110">To avoid contention and improve performance in a distributed environment, an application shouldn't try to provide strong transactional consistency.</span></span> <span data-ttu-id="6b8da-111">相反，应用程序应实现最终一致性。</span><span class="sxs-lookup"><span data-stu-id="6b8da-111">Rather, the application should implement eventual consistency.</span></span> <span data-ttu-id="6b8da-112">在此模型中，典型的业务操作包含一系列单独的步骤。</span><span class="sxs-lookup"><span data-stu-id="6b8da-112">In this model, a typical business operation consists of a series of separate steps.</span></span> <span data-ttu-id="6b8da-113">虽然在执行这些步骤的期间，系统状态的总体视图可能会不一致，但操作完成且所有的步骤执行后，系统会再次变得一致。</span><span class="sxs-lookup"><span data-stu-id="6b8da-113">While these steps are being performed, the overall view of the system state might be inconsistent, but when the operation has completed and all of the steps have been executed the system should become consistent again.</span></span>

> <span data-ttu-id="6b8da-114">[Data Consistency Primer](https://msdn.microsoft.com/library/dn589800.aspx)（数据一致性入门）提供了分布式事务缩放性不佳的原因以及有关最终一致性模型原则的信息。</span><span class="sxs-lookup"><span data-stu-id="6b8da-114">The [Data Consistency Primer](https://msdn.microsoft.com/library/dn589800.aspx) provides information about why distributed transactions don't scale well, and the principles of the eventual consistency model.</span></span>

<span data-ttu-id="6b8da-115">最终一致性模型中的一个难题是如何处理失败步骤。</span><span class="sxs-lookup"><span data-stu-id="6b8da-115">A challenge in the eventual consistency model is how to handle a step that has failed.</span></span> <span data-ttu-id="6b8da-116">这种情况下，可能需要撤销此操作中先前步骤已完成的所有工作。</span><span class="sxs-lookup"><span data-stu-id="6b8da-116">In this case it might be necessary to undo all of the work completed by the previous steps in the operation.</span></span> <span data-ttu-id="6b8da-117">然而，数据不能回滚，因为应用程序的其他并发实例可能已更改了数据。</span><span class="sxs-lookup"><span data-stu-id="6b8da-117">However, the data can't simply be rolled back because other concurrent instances of the application might have changed it.</span></span> <span data-ttu-id="6b8da-118">即使在数据并未被并发实例更改的情况下，撤销步骤也可能不仅仅是还原原始状态。</span><span class="sxs-lookup"><span data-stu-id="6b8da-118">Even in cases where the data hasn't been changed by a concurrent instance, undoing a step might not simply be a matter of restoring the original state.</span></span> <span data-ttu-id="6b8da-119">可能需要应用多种业务特定规则（参阅示例部分中所述的旅行网站）。</span><span class="sxs-lookup"><span data-stu-id="6b8da-119">It might be necessary to apply various business-specific rules (see the travel website described in the Example section).</span></span>

<span data-ttu-id="6b8da-120">如果实现最终一致性的操作跨多个异类数据存储，则撤销操作中的步骤需要依次访问每个数据存储。</span><span class="sxs-lookup"><span data-stu-id="6b8da-120">If an operation that implements eventual consistency spans several heterogeneous data stores, undoing the steps in the operation will require visiting each data store in turn.</span></span> <span data-ttu-id="6b8da-121">必须以可靠方式撤销每个数据存储中执行的工作，以此防止系统保持不一致。</span><span class="sxs-lookup"><span data-stu-id="6b8da-121">The work performed in every data store must be undone reliably to prevent the system from remaining inconsistent.</span></span>

<span data-ttu-id="6b8da-122">并非所有受实现最终一致性操作影响的数据都可能保留在数据库中。</span><span class="sxs-lookup"><span data-stu-id="6b8da-122">Not all data affected by an operation that implements eventual consistency might be held in a database.</span></span> <span data-ttu-id="6b8da-123">在面向服务的体系结构 (SOA) 环境中，操作可能会调用服务中的操作，并导致该服务的状态发生更改。</span><span class="sxs-lookup"><span data-stu-id="6b8da-123">In a service oriented architecture (SOA) environment an operation could invoke an action in a service, and cause a change in the state held by that service.</span></span> <span data-ttu-id="6b8da-124">若要撤消该操作，则也必须撤消该状态更改。</span><span class="sxs-lookup"><span data-stu-id="6b8da-124">To undo the operation, this state change must also be undone.</span></span> <span data-ttu-id="6b8da-125">这可能涉及到再次调用服务和再执行一个撤销第一次操作效果的操作。</span><span class="sxs-lookup"><span data-stu-id="6b8da-125">This can involve invoking the service again and performing another action that reverses the effects of the first.</span></span>

## <a name="solution"></a><span data-ttu-id="6b8da-126">解决方案</span><span class="sxs-lookup"><span data-stu-id="6b8da-126">Solution</span></span>

<span data-ttu-id="6b8da-127">解决方案是实现补偿事务。</span><span class="sxs-lookup"><span data-stu-id="6b8da-127">The solution is to implement a compensating transaction.</span></span> <span data-ttu-id="6b8da-128">补偿事务中的步骤必须撤销原始操作中步骤的效果。</span><span class="sxs-lookup"><span data-stu-id="6b8da-128">The steps in a compensating transaction must undo the effects of the steps in the original operation.</span></span> <span data-ttu-id="6b8da-129">补偿事务可能无法仅将当前状态替换为此操作开始时系统所处的状态，因为此方法可能会覆盖由应用程序其他并发实例所作的更改。</span><span class="sxs-lookup"><span data-stu-id="6b8da-129">A compensating transaction might not be able to simply replace the current state with the state the system was in at the start of the operation because this approach could overwrite changes made by other concurrent instances of an application.</span></span> <span data-ttu-id="6b8da-130">相反，必须是一个智能过程，该智能过程会考虑到并发实例执行的所有工作。</span><span class="sxs-lookup"><span data-stu-id="6b8da-130">Instead, it must be an intelligent process that takes into account any work done by concurrent instances.</span></span> <span data-ttu-id="6b8da-131">该过程通常特定于应用程序，由原始操作执行的工作的性质驱动。</span><span class="sxs-lookup"><span data-stu-id="6b8da-131">This process will usually be application specific, driven by the nature of the work performed by the original operation.</span></span>

<span data-ttu-id="6b8da-132">常见方法是使用工作流来实现需要补偿的最终一致操作。</span><span class="sxs-lookup"><span data-stu-id="6b8da-132">A common approach is to use a workflow to implement an eventually consistent operation that requires compensation.</span></span> <span data-ttu-id="6b8da-133">原始操作进行期间，系统会记录每个步骤以及如何可撤销相应步骤执行的操作的相关信息。</span><span class="sxs-lookup"><span data-stu-id="6b8da-133">As the original operation proceeds, the system records information about each step and how the work performed by that step can be undone.</span></span> <span data-ttu-id="6b8da-134">如果操作在任何时刻失败，则工作流会回退已完成的步骤，并撤销每个步骤。</span><span class="sxs-lookup"><span data-stu-id="6b8da-134">If the operation fails at any point, the workflow rewinds back through the steps it's completed and performs the work that reverses each step.</span></span> <span data-ttu-id="6b8da-135">请注意，补偿事务可能并非必须按照原始操作的逆序来撤销工作，其可能会并行执行部分撤销步骤。</span><span class="sxs-lookup"><span data-stu-id="6b8da-135">Note that a compensating transaction might not have to undo the work in the exact reverse order of the original operation, and it might be possible to perform some of the undo steps in parallel.</span></span>

> <span data-ttu-id="6b8da-136">这种方法类似于 [Clemens Vasters 博客](http://vasters.com/clemensv/2012/09/01/Sagas.aspx)中所述的 Sagas 策略。</span><span class="sxs-lookup"><span data-stu-id="6b8da-136">This approach is similar to the Sagas strategy discussed in [Clemens Vasters’ blog](http://vasters.com/clemensv/2012/09/01/Sagas.aspx).</span></span>

<span data-ttu-id="6b8da-137">补偿事务也是最终一致操作，并且也可能会失败。</span><span class="sxs-lookup"><span data-stu-id="6b8da-137">A compensating transaction is also an eventually consistent operation and it could also fail.</span></span> <span data-ttu-id="6b8da-138">系统应能够在失败时恢复补偿事务，然后继续。</span><span class="sxs-lookup"><span data-stu-id="6b8da-138">The system should be able to resume the compensating transaction at the point of failure and continue.</span></span> <span data-ttu-id="6b8da-139">可能需要重复已失败的步骤，因此补偿事务中的步骤应定义为幂等命令。</span><span class="sxs-lookup"><span data-stu-id="6b8da-139">It might be necessary to repeat a step that's failed, so the steps in a compensating transaction should be defined as idempotent commands.</span></span> <span data-ttu-id="6b8da-140">有关详细信息，请参阅 Jonathan Oliver 博客中的 [Idempotency Patterns](http://blog.jonathanoliver.com/2010/04/idempotency-patterns/)（幂等模式）。</span><span class="sxs-lookup"><span data-stu-id="6b8da-140">For more information, see [Idempotency Patterns](http://blog.jonathanoliver.com/2010/04/idempotency-patterns/) on Jonathan Oliver’s blog.</span></span>

<span data-ttu-id="6b8da-141">某些情况下，除非手动干预，否则可能无法恢复已失败的步骤。</span><span class="sxs-lookup"><span data-stu-id="6b8da-141">In some cases it might not be possible to recover from a step that has failed except through manual intervention.</span></span> <span data-ttu-id="6b8da-142">这类情况下，系统应会发出警报，并会提供尽可能多的有关失败原因的信息。</span><span class="sxs-lookup"><span data-stu-id="6b8da-142">In these situations the system should raise an alert and provide as much information as possible about the reason for the failure.</span></span>

## <a name="issues-and-considerations"></a><span data-ttu-id="6b8da-143">问题和注意事项</span><span class="sxs-lookup"><span data-stu-id="6b8da-143">Issues and considerations</span></span>

<span data-ttu-id="6b8da-144">在决定如何实现此模式时，请考虑以下几点：</span><span class="sxs-lookup"><span data-stu-id="6b8da-144">Consider the following points when deciding how to implement this pattern:</span></span>

<span data-ttu-id="6b8da-145">确定实现最终一致性的操作中的步骤何时失败并非易事。</span><span class="sxs-lookup"><span data-stu-id="6b8da-145">It might not be easy to determine when a step in an operation that implements eventual consistency has failed.</span></span> <span data-ttu-id="6b8da-146">步骤可能不会立即失败，相反，其可能会阻止。</span><span class="sxs-lookup"><span data-stu-id="6b8da-146">A step might not fail immediately, but instead could block.</span></span> <span data-ttu-id="6b8da-147">可能需要实现某种形式的超时机制。</span><span class="sxs-lookup"><span data-stu-id="6b8da-147">It might be necessary to implement some form of time-out mechanism.</span></span>

<span data-ttu-id="6b8da-148">—归纳补偿逻辑并不可行。</span><span class="sxs-lookup"><span data-stu-id="6b8da-148">-Compensation logic isn't easily generalized.</span></span> <span data-ttu-id="6b8da-149">补偿事务特定于应用程序。</span><span class="sxs-lookup"><span data-stu-id="6b8da-149">A compensating transaction is application specific.</span></span> <span data-ttu-id="6b8da-150">它依赖于具有足够信息的应用程序，从而能够撤消失败的操作中每个步骤的效果。</span><span class="sxs-lookup"><span data-stu-id="6b8da-150">It relies on the application having sufficient information to be able to undo the effects of each step in a failed operation.</span></span>

<span data-ttu-id="6b8da-151">应将补偿事务中的步骤定义为幂等命令。</span><span class="sxs-lookup"><span data-stu-id="6b8da-151">You should define the steps in a compensating transaction as idempotent commands.</span></span> <span data-ttu-id="6b8da-152">这样，补偿事务自身失败时可重复步骤。</span><span class="sxs-lookup"><span data-stu-id="6b8da-152">This enables the steps to be repeated if the compensating transaction itself fails.</span></span>

<span data-ttu-id="6b8da-153">处理该原始操作中步骤的基础结构和补偿事务必须具有复原性。</span><span class="sxs-lookup"><span data-stu-id="6b8da-153">The infrastructure that handles the steps in the original operation, and the compensating transaction, must be resilient.</span></span> <span data-ttu-id="6b8da-154">其不得丢失补偿失败步骤所需信息，且必须能够可靠地监视补偿逻辑的进程。</span><span class="sxs-lookup"><span data-stu-id="6b8da-154">It must not lose the information required to compensate for a failing step, and it must be able to reliably monitor the progress of the compensation logic.</span></span>

<span data-ttu-id="6b8da-155">补偿事务不一定会将系统中的数据返回到原始操作开始时其所处的状态。</span><span class="sxs-lookup"><span data-stu-id="6b8da-155">A compensating transaction doesn't necessarily return the data in the system to the state it was in at the start of the original operation.</span></span> <span data-ttu-id="6b8da-156">相反，它补偿操作失败前由已成功完成的步骤所执行的工作。</span><span class="sxs-lookup"><span data-stu-id="6b8da-156">Instead, it compensates for the work performed by the steps that completed successfully before the operation failed.</span></span>

<span data-ttu-id="6b8da-157">补偿事务中步骤的顺序不一定与原始操作中步骤的顺序完全相反。</span><span class="sxs-lookup"><span data-stu-id="6b8da-157">The order of the steps in the compensating transaction doesn't necessarily have to be the exact opposite of the steps in the original operation.</span></span> <span data-ttu-id="6b8da-158">例如，一个数据存储可能比另一个数据存储对不一致性更加敏感，因而补偿事务中撤销对此存储的更改的步骤应该会首先发生。</span><span class="sxs-lookup"><span data-stu-id="6b8da-158">For example, one data store might be more sensitive to inconsistencies than another, and so the steps in the compensating transaction that undo the changes to this store should occur first.</span></span>

<span data-ttu-id="6b8da-159">对完成操作所需的每个资源采用短期的基于超时的锁并预先获取这些资源，这样有助于增加总体活动成功的可能性。</span><span class="sxs-lookup"><span data-stu-id="6b8da-159">Placing a short-term timeout-based lock on each resource that's required to complete an operation, and obtaining these resources in advance, can help increase the likelihood that the overall activity will succeed.</span></span> <span data-ttu-id="6b8da-160">仅在获取所有资源后才应执行工作。</span><span class="sxs-lookup"><span data-stu-id="6b8da-160">The work should be performed only after all the resources have been acquired.</span></span> <span data-ttu-id="6b8da-161">锁过期之前必须完成所有操作。</span><span class="sxs-lookup"><span data-stu-id="6b8da-161">All actions must be finalized before the locks expire.</span></span>

<span data-ttu-id="6b8da-162">考虑使用包容性更强的重试逻辑来尽可能避免会触发补偿事务的失败。</span><span class="sxs-lookup"><span data-stu-id="6b8da-162">Consider using retry logic that is more forgiving than usual to minimize failures that trigger a compensating transaction.</span></span> <span data-ttu-id="6b8da-163">如果实现最终一致性操作中的步骤失败，请尝试会失败处理为暂时异常，然后重复相应步骤。</span><span class="sxs-lookup"><span data-stu-id="6b8da-163">If a step in an operation that implements eventual consistency fails, try handling the failure as a transient exception and repeat the step.</span></span> <span data-ttu-id="6b8da-164">仅在步骤出现反复失败或不可恢复性失败时停止操作并启动补偿事务。</span><span class="sxs-lookup"><span data-stu-id="6b8da-164">Only stop the operation and initiate a compensating transaction if a step fails repeatedly or irrecoverably.</span></span>

> <span data-ttu-id="6b8da-165">实现补偿事务的许多难题与实现最终一致性中的难题相同。</span><span class="sxs-lookup"><span data-stu-id="6b8da-165">Many of the challenges of implementing a compensating transaction are the same as those with implementing eventual consistency.</span></span> <span data-ttu-id="6b8da-166">有关详细信息，请参阅 [Data Consistency Primer](https://msdn.microsoft.com/library/dn589800.aspx)（数据一致性入门）中的“实现最终一致性注意事项”部分。</span><span class="sxs-lookup"><span data-stu-id="6b8da-166">See the section Considerations for Implementing Eventual Consistency in the [Data Consistency Primer](https://msdn.microsoft.com/library/dn589800.aspx) for more information.</span></span>

## <a name="when-to-use-this-pattern"></a><span data-ttu-id="6b8da-167">何时使用此模式</span><span class="sxs-lookup"><span data-stu-id="6b8da-167">When to use this pattern</span></span>

<span data-ttu-id="6b8da-168">仅对失败时必须撤销的操作使用此模式。</span><span class="sxs-lookup"><span data-stu-id="6b8da-168">Use this pattern only for operations that must be undone if they fail.</span></span> <span data-ttu-id="6b8da-169">如果可能，请设计相关解决方案来避免需要补偿事务所带来的麻烦。</span><span class="sxs-lookup"><span data-stu-id="6b8da-169">If possible, design solutions to avoid the complexity of requiring compensating transactions.</span></span>

## <a name="example"></a><span data-ttu-id="6b8da-170">示例</span><span class="sxs-lookup"><span data-stu-id="6b8da-170">Example</span></span>

<span data-ttu-id="6b8da-171">旅行网站让客户订购旅行路线。</span><span class="sxs-lookup"><span data-stu-id="6b8da-171">A travel website lets customers book itineraries.</span></span> <span data-ttu-id="6b8da-172">单个路线可能包含一系列航班和酒店。</span><span class="sxs-lookup"><span data-stu-id="6b8da-172">A single itinerary might comprise a series of flights and hotels.</span></span> <span data-ttu-id="6b8da-173">一位先从西雅图到伦敦再到巴黎的客户在创建路线时，可执行以下步骤：</span><span class="sxs-lookup"><span data-stu-id="6b8da-173">A customer traveling from Seattle to London and then on to Paris could perform the following steps when creating an itinerary:</span></span>

1. <span data-ttu-id="6b8da-174">订购从西雅图到伦敦的 F1 航班机票。</span><span class="sxs-lookup"><span data-stu-id="6b8da-174">Book a seat on flight F1 from Seattle to London.</span></span>
2. <span data-ttu-id="6b8da-175">订购从伦敦到巴黎的 F2 航班机票。</span><span class="sxs-lookup"><span data-stu-id="6b8da-175">Book a seat on flight F2 from London to Paris.</span></span>
3. <span data-ttu-id="6b8da-176">订购从巴黎到西雅图的 F3 航班机票。</span><span class="sxs-lookup"><span data-stu-id="6b8da-176">Book a seat on flight F3 from Paris to Seattle.</span></span>
4. <span data-ttu-id="6b8da-177">预定伦敦 H1 酒店房间。</span><span class="sxs-lookup"><span data-stu-id="6b8da-177">Reserve a room at hotel H1 in London.</span></span>
5. <span data-ttu-id="6b8da-178">预定巴黎 H2 酒店房间。</span><span class="sxs-lookup"><span data-stu-id="6b8da-178">Reserve a room at hotel H2 in Paris.</span></span>

<span data-ttu-id="6b8da-179">虽然每个步骤为单独操作，但这些步骤会构成一个最终一致操作。</span><span class="sxs-lookup"><span data-stu-id="6b8da-179">These steps constitute an eventually consistent operation, although each step is a separate action.</span></span> <span data-ttu-id="6b8da-180">因此，除执行这些步骤外，系统也必须记录撤销每个步骤所需的对立操作，以处理客户取消路线的情况。</span><span class="sxs-lookup"><span data-stu-id="6b8da-180">Therefore, as well as performing these steps, the system must also record the counter operations necessary to undo each step in case the customer decides to cancel the itinerary.</span></span> <span data-ttu-id="6b8da-181">之后，执行对立操作所需的步骤可作为补偿事务运行。</span><span class="sxs-lookup"><span data-stu-id="6b8da-181">The steps necessary to perform the counter operations can then run as a compensating transaction.</span></span>

<span data-ttu-id="6b8da-182">请注意，补偿事务中的步骤与原始步骤的顺序可能会不正好相反，且补偿事务中每个步骤中的逻辑必须要考虑到特定于业务的规则。</span><span class="sxs-lookup"><span data-stu-id="6b8da-182">Notice that the steps in the compensating transaction might not be the exact opposite of the original steps, and the logic in each step in the compensating transaction must take into account any business-specific rules.</span></span> <span data-ttu-id="6b8da-183">例如，如果取消航班机票订购，客户可能不会享有已支付金额的全额退款。</span><span class="sxs-lookup"><span data-stu-id="6b8da-183">For example, unbooking a seat on a flight might not entitle the customer to a complete refund of any money paid.</span></span> <span data-ttu-id="6b8da-184">该图介绍了生成补偿事务来撤销长时间运行的事务，从而来订购旅行路线。</span><span class="sxs-lookup"><span data-stu-id="6b8da-184">The figure illustrates generating a compensating transaction to undo a long-running transaction to book a travel itinerary.</span></span>

![生成补偿事务来撤销长时间运行的事务，从而来订购旅行路线](./_images/compensating-transaction-diagram.png)


> <span data-ttu-id="6b8da-186">补偿事务中的步骤可能会并行执行，具体取决于每个步骤的补偿逻辑的设计方式。</span><span class="sxs-lookup"><span data-stu-id="6b8da-186">It might be possible for the steps in the compensating transaction to be performed in parallel, depending on how you've designed the compensating logic for each step.</span></span>

<span data-ttu-id="6b8da-187">在许多业务解决方案中，单个步骤失败并不始终要求通过使用补偿事务回滚系统。</span><span class="sxs-lookup"><span data-stu-id="6b8da-187">In many business solutions, failure of a single step doesn't always necessitate rolling the system back by using a compensating transaction.</span></span> <span data-ttu-id="6b8da-188">例如，如果&mdash;在旅行网站上预订航班 F1、F2 和 F3 后&mdash;，客户无法预订 H1 酒店的房间，则首选方案是为客户提供该市另一家酒店的房间，而不是取消航班。</span><span class="sxs-lookup"><span data-stu-id="6b8da-188">For example, if&mdash;after having booked flights F1, F2, and F3 in the travel website scenario&mdash;the customer is unable to reserve a room at hotel H1, it's preferable to offer the customer a room at a different hotel in the same city rather than canceling the flights.</span></span> <span data-ttu-id="6b8da-189">客户仍可决定取消（这种情况下，会运行补偿事务并撤销 F1、F2 和 F3 航班预订），但应由客户而不是系统作出此决定。</span><span class="sxs-lookup"><span data-stu-id="6b8da-189">The customer can still decide to cancel (in which case the compensating transaction runs and undoes the bookings made on flights F1, F2, and F3), but this decision should be made by the customer rather than by the system.</span></span>

## <a name="related-patterns-and-guidance"></a><span data-ttu-id="6b8da-190">相关模式和指南</span><span class="sxs-lookup"><span data-stu-id="6b8da-190">Related patterns and guidance</span></span>

<span data-ttu-id="6b8da-191">实现此模式时可能，可能也会与以下模式和指南相关：</span><span class="sxs-lookup"><span data-stu-id="6b8da-191">The following patterns and guidance might also be relevant when implementing this pattern:</span></span>

- <span data-ttu-id="6b8da-192">[Data consistency primer](https://msdn.microsoft.com/library/dn589800.aspx)（数据一致性入门）。</span><span class="sxs-lookup"><span data-stu-id="6b8da-192">[Data Consistency Primer](https://msdn.microsoft.com/library/dn589800.aspx).</span></span> <span data-ttu-id="6b8da-193">补偿事务模式通常用于撤消实现最终一致性模型的操作。</span><span class="sxs-lookup"><span data-stu-id="6b8da-193">The Compensating Transaction pattern is often used to undo operations that implement the eventual consistency model.</span></span> <span data-ttu-id="6b8da-194">该入门指导提供了有关最终一致性优点和不足的信息。</span><span class="sxs-lookup"><span data-stu-id="6b8da-194">This primer provides information on the benefits and tradeoffs of eventual consistency.</span></span>

- <span data-ttu-id="6b8da-195">[计划程序代理监督模式](scheduler-agent-supervisor.md)。</span><span class="sxs-lookup"><span data-stu-id="6b8da-195">[Scheduler-Agent-Supervisor Pattern](scheduler-agent-supervisor.md).</span></span> <span data-ttu-id="6b8da-196">介绍如何实现弹性系统，这些弹性系统执行使用分布式服务和资源的业务操作。</span><span class="sxs-lookup"><span data-stu-id="6b8da-196">Describes how to implement resilient systems that perform business operations that use distributed services and resources.</span></span> <span data-ttu-id="6b8da-197">有时，可能需要使用补偿事务来撤销操作执行的工作。</span><span class="sxs-lookup"><span data-stu-id="6b8da-197">Sometimes, it might be necessary to undo the work performed by an operation by using a compensating transaction.</span></span>

- <span data-ttu-id="6b8da-198">[重试模式](./retry.md)。</span><span class="sxs-lookup"><span data-stu-id="6b8da-198">[Retry Pattern](./retry.md).</span></span> <span data-ttu-id="6b8da-199">补偿事务执行成本比较高，因此可按照重试模式，通过实现有效的重试失败策略来尽可能少地使用补偿事务。</span><span class="sxs-lookup"><span data-stu-id="6b8da-199">Compensating transactions can be expensive to perform, and it might be possible to minimize their use by implementing an effective policy of retrying failing operations by following the Retry pattern.</span></span>