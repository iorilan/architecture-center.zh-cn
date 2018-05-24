---
title: 应用程序角色
description: 如何使用应用程序角色执行授权
author: MikeWasson
ms:date: 07/21/2017
pnp.series.title: Manage Identity in Multitenant Applications
pnp.series.prev: signup
pnp.series.next: authorize
ms.openlocfilehash: ec563936e5f00aba79d65844762feeed97ad547d
ms.sourcegitcommit: bb348bd3a8a4e27ef61e8eee74b54b07b65dbf98
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/21/2018
---
# <a name="application-roles"></a><span data-ttu-id="da8a3-103">应用程序角色</span><span class="sxs-lookup"><span data-stu-id="da8a3-103">Application roles</span></span>

<span data-ttu-id="da8a3-104">[![GitHub](../_images/github.png) 示例代码][sample application]</span><span class="sxs-lookup"><span data-stu-id="da8a3-104">[![GitHub](../_images/github.png) Sample code][sample application]</span></span>

<span data-ttu-id="da8a3-105">应用程序角色用于向用户分配权限。</span><span class="sxs-lookup"><span data-stu-id="da8a3-105">Application roles are used to assign permissions to users.</span></span> <span data-ttu-id="da8a3-106">例如，[Tailspin Surveys][Tailspin] 应用程序可定义以下角色：</span><span class="sxs-lookup"><span data-stu-id="da8a3-106">For example, the [Tailspin Surveys][Tailspin] application defines the following roles:</span></span>

* <span data-ttu-id="da8a3-107">管理员。</span><span class="sxs-lookup"><span data-stu-id="da8a3-107">Administrator.</span></span> <span data-ttu-id="da8a3-108">可在属于该租户的任何调查中执行所有 CRUD 操作。</span><span class="sxs-lookup"><span data-stu-id="da8a3-108">Can perform all CRUD operations on any survey that belongs to that tenant.</span></span>
* <span data-ttu-id="da8a3-109">创建者。</span><span class="sxs-lookup"><span data-stu-id="da8a3-109">Creator.</span></span> <span data-ttu-id="da8a3-110">可创建新调查。</span><span class="sxs-lookup"><span data-stu-id="da8a3-110">Can create new surveys.</span></span>
* <span data-ttu-id="da8a3-111">读者。</span><span class="sxs-lookup"><span data-stu-id="da8a3-111">Reader.</span></span> <span data-ttu-id="da8a3-112">可读取属于该租户的任何调查。</span><span class="sxs-lookup"><span data-stu-id="da8a3-112">Can read any surveys that belong to that tenant.</span></span>

<span data-ttu-id="da8a3-113">你会发现在[授权]过程中，角色最终会转换成权限。</span><span class="sxs-lookup"><span data-stu-id="da8a3-113">You can see that roles ultimately get translated into permissions, during [authorization].</span></span> <span data-ttu-id="da8a3-114">但是，首先要解决的问题是如何分配和管理角色。</span><span class="sxs-lookup"><span data-stu-id="da8a3-114">But the first question is how to assign and manage roles.</span></span> <span data-ttu-id="da8a3-115">我们标识了三个主要选项：</span><span class="sxs-lookup"><span data-stu-id="da8a3-115">We identified three main options:</span></span>

* [<span data-ttu-id="da8a3-116">Azure AD 应用角色</span><span class="sxs-lookup"><span data-stu-id="da8a3-116">Azure AD App Roles</span></span>](#roles-using-azure-ad-app-roles)
* [<span data-ttu-id="da8a3-117">Azure AD 安全组</span><span class="sxs-lookup"><span data-stu-id="da8a3-117">Azure AD security groups</span></span>](#roles-using-azure-ad-security-groups)
* <span data-ttu-id="da8a3-118">[应用程序角色管理员](#roles-using-an-application-role-manager)。</span><span class="sxs-lookup"><span data-stu-id="da8a3-118">[Application role manager](#roles-using-an-application-role-manager).</span></span>

## <a name="roles-using-azure-ad-app-roles"></a><span data-ttu-id="da8a3-119">使用 Azure AD 应用角色的角色</span><span class="sxs-lookup"><span data-stu-id="da8a3-119">Roles using Azure AD App Roles</span></span>
<span data-ttu-id="da8a3-120">这是我们在 Tailspin Surveys 应用中使用的方法。</span><span class="sxs-lookup"><span data-stu-id="da8a3-120">This is the approach that we used in the Tailspin Surveys app.</span></span>

<span data-ttu-id="da8a3-121">按照这种方法，SaaS 提供程序通过将应用程序角色添加到应用程序清单来定义该角色。</span><span class="sxs-lookup"><span data-stu-id="da8a3-121">In this approach, The SaaS provider defines the application roles by adding them to the application manifest.</span></span> <span data-ttu-id="da8a3-122">客户注册后，该客户的 AD 目录管理员会将用户分配给角色。</span><span class="sxs-lookup"><span data-stu-id="da8a3-122">After a customer signs up, an admin for the customer's AD directory assigns users to the roles.</span></span> <span data-ttu-id="da8a3-123">用户登录后，该用户的已分配角色将以声明方式发送过来。</span><span class="sxs-lookup"><span data-stu-id="da8a3-123">When a user signs in, the user's assigned roles are sent as claims.</span></span>

> [!NOTE]
> <span data-ttu-id="da8a3-124">如果该客户有 Azure AD Premium，管理员便可向角色分配一个安全组，该组内的成员可继承此应用角色。</span><span class="sxs-lookup"><span data-stu-id="da8a3-124">If the customer has Azure AD Premium, the admin can assign a security group to a role, and members of the group will inherit the app role.</span></span> <span data-ttu-id="da8a3-125">这种管理角色的方式比较方便，因为组所有者不一定得是 AD 管理员。</span><span class="sxs-lookup"><span data-stu-id="da8a3-125">This is a convenient way to manage roles, because the group owner doesn't need to be an AD admin.</span></span>
> 
> 

<span data-ttu-id="da8a3-126">此方法的优点：</span><span class="sxs-lookup"><span data-stu-id="da8a3-126">Advantages of this approach:</span></span>

* <span data-ttu-id="da8a3-127">编程模型简单。</span><span class="sxs-lookup"><span data-stu-id="da8a3-127">Simple programming model.</span></span>
* <span data-ttu-id="da8a3-128">角色特定于应用程序。</span><span class="sxs-lookup"><span data-stu-id="da8a3-128">Roles are specific to the application.</span></span> <span data-ttu-id="da8a3-129">一个应用程序的角色声明不会发送到另一个应用程序。</span><span class="sxs-lookup"><span data-stu-id="da8a3-129">The role claims for one application are not sent to another application.</span></span>
* <span data-ttu-id="da8a3-130">如果客户从其 AD 租户中删除此应用程序，则角色也会消失。</span><span class="sxs-lookup"><span data-stu-id="da8a3-130">If the customer removes the application from their AD tenant, the roles go away.</span></span>
* <span data-ttu-id="da8a3-131">应用程序不需要除读取用户配置文件之外的任何附加 Active Directory 权限。</span><span class="sxs-lookup"><span data-stu-id="da8a3-131">The application doesn't need any extra Active Directory permissions, other than reading the user's profile.</span></span>

<span data-ttu-id="da8a3-132">缺点：</span><span class="sxs-lookup"><span data-stu-id="da8a3-132">Drawbacks:</span></span>

* <span data-ttu-id="da8a3-133">如果客户没有 Azure AD Premium，则不能将安全组分配给角色。</span><span class="sxs-lookup"><span data-stu-id="da8a3-133">Customers without Azure AD Premium cannot assign security groups to roles.</span></span> <span data-ttu-id="da8a3-134">对于这类客户，所有的用户分配都必须由 AD 管理员来完成。</span><span class="sxs-lookup"><span data-stu-id="da8a3-134">For these customers, all user assignments must be done by an AD administrator.</span></span>
* <span data-ttu-id="da8a3-135">如果有从 Web 应用程序分离出来的后端 Web API，则 Web 应用的角色分配不适用于 Web API。</span><span class="sxs-lookup"><span data-stu-id="da8a3-135">If you have a backend web API, which is separate from the web app, then role assignments for the web app don't apply to the web API.</span></span> <span data-ttu-id="da8a3-136">有关该点详细讨论，请参阅[保护后端 Web API]。</span><span class="sxs-lookup"><span data-stu-id="da8a3-136">For more discussion of this point, see [Securing a backend web API].</span></span>

### <a name="implementation"></a><span data-ttu-id="da8a3-137">实现</span><span class="sxs-lookup"><span data-stu-id="da8a3-137">Implementation</span></span>
<span data-ttu-id="da8a3-138">**定义角色。**</span><span class="sxs-lookup"><span data-stu-id="da8a3-138">**Define the roles.**</span></span> <span data-ttu-id="da8a3-139">SaaS 提供程序可声明[应用程序清单]中的应用角色。</span><span class="sxs-lookup"><span data-stu-id="da8a3-139">The SaaS provider declares the app roles in the [application manifest].</span></span> <span data-ttu-id="da8a3-140">例如，以下是 Surveys 应用的清单条目：</span><span class="sxs-lookup"><span data-stu-id="da8a3-140">For example, here is the manifest entry for the Surveys app:</span></span>

```json
"appRoles": [
  {
    "allowedMemberTypes": [
      "User"
    ],
    "description": "Creators can create Surveys",
    "displayName": "SurveyCreator",
    "id": "1b4f816e-5eaf-48b9-8613-7923830595ad",
    "isEnabled": true,
    "value": "SurveyCreator"
  },
  {
    "allowedMemberTypes": [
      "User"
    ],
    "description": "Administrators can manage the Surveys in their tenant",
    "displayName": "SurveyAdmin",
    "id": "c20e145e-5459-4a6c-a074-b942bbd4cfe1",
    "isEnabled": true,
    "value": "SurveyAdmin"
  }
],
```

<span data-ttu-id="da8a3-141">`value` 属性出现在角色声明中。</span><span class="sxs-lookup"><span data-stu-id="da8a3-141">The `value`  property appears in the role claim.</span></span> <span data-ttu-id="da8a3-142">`id` 属性是定义角色的唯一标识符。</span><span class="sxs-lookup"><span data-stu-id="da8a3-142">The `id` property is the unique identifier for the defined role.</span></span> <span data-ttu-id="da8a3-143">始终为 `id` 生成新的 GUID 值。</span><span class="sxs-lookup"><span data-stu-id="da8a3-143">Always generate a new GUID value for `id`.</span></span>

<span data-ttu-id="da8a3-144">**分配用户**。</span><span class="sxs-lookup"><span data-stu-id="da8a3-144">**Assign users**.</span></span> <span data-ttu-id="da8a3-145">新客户注册时，应用程序在客户的 AD 租户中进行注册。</span><span class="sxs-lookup"><span data-stu-id="da8a3-145">When a new customer signs up, the application is registered in the customer's AD tenant.</span></span> <span data-ttu-id="da8a3-146">此时，该租户的 AD 管理员可以将用户分配到角色。</span><span class="sxs-lookup"><span data-stu-id="da8a3-146">At this point, an AD admin for that tenant can assign users to roles.</span></span>

> [!NOTE]
> <span data-ttu-id="da8a3-147">如前文所述，如果客户有 Azure AD Premium，则可将安全组分配给角色。</span><span class="sxs-lookup"><span data-stu-id="da8a3-147">As noted earlier, customers with Azure AD Premium can also assign security groups to roles.</span></span>
> 
> 

<span data-ttu-id="da8a3-148">在以下 Azure 门户屏幕截图中为 Survey 应用程序的用户和组。</span><span class="sxs-lookup"><span data-stu-id="da8a3-148">The following screenshot from the Azure portal shows users and groups for the Survey application.</span></span> <span data-ttu-id="da8a3-149">“管理员”和“创建者”是组，分别分配给 SurveyAdmin 和 SurveyCreator 角色。</span><span class="sxs-lookup"><span data-stu-id="da8a3-149">Admin and Creator are groups, assigned to SurveyAdmin and SurveyCreator roles respectively.</span></span> <span data-ttu-id="da8a3-150">Alice 是直接分配给 SurveyAdmin 角色的用户。</span><span class="sxs-lookup"><span data-stu-id="da8a3-150">Alice is a user who was assigned directly to the SurveyAdmin role.</span></span> <span data-ttu-id="da8a3-151">Bob 和 Charles 是还未直接分配给角色的用户。</span><span class="sxs-lookup"><span data-stu-id="da8a3-151">Bob and Charles are users that have not been directly assigned to a role.</span></span>

![用户和组](./images/running-the-app/users-and-groups.png)

<span data-ttu-id="da8a3-153">如下列屏幕截图所示，Charles 属于“管理员”组，因此他可继承 SurveyAdmin 角色。</span><span class="sxs-lookup"><span data-stu-id="da8a3-153">As shown in the following screenshot, Charles is part of the Admin group, so he inherits the SurveyAdmin role.</span></span> <span data-ttu-id="da8a3-154">而 Bob，尚未分配有角色。</span><span class="sxs-lookup"><span data-stu-id="da8a3-154">In the case of Bob, he has not been assigned a role yet.</span></span>

![“管理员”组成员](./images/running-the-app/admin-members.png)


> [!NOTE]
> <span data-ttu-id="da8a3-156">另一种方法是应用程序使用 Azure AD Graph API 以编程方式分配角色。</span><span class="sxs-lookup"><span data-stu-id="da8a3-156">An alternative approach is for the application to assign roles programmatically, using the Azure AD Graph API.</span></span> <span data-ttu-id="da8a3-157">但是，这需要应用程序获取客户 AD 目录的写入权限。</span><span class="sxs-lookup"><span data-stu-id="da8a3-157">However, this requires the application to obtain write permissions for the customer's AD directory.</span></span> <span data-ttu-id="da8a3-158">具有这些权限的应用程序可能会造成大量混乱 &mdash; 不过，客户相信此应用不会扰乱其目录。</span><span class="sxs-lookup"><span data-stu-id="da8a3-158">An application with those permissions could do a lot of mischief &mdash; the customer is trusting the app not to mess up their directory.</span></span> <span data-ttu-id="da8a3-159">很多客户可能不愿意授予该级别的访问权限。</span><span class="sxs-lookup"><span data-stu-id="da8a3-159">Many customers might be unwilling to grant this level of access.</span></span>
> 

<span data-ttu-id="da8a3-160">**获取角色声明**。</span><span class="sxs-lookup"><span data-stu-id="da8a3-160">**Get role claims**.</span></span> <span data-ttu-id="da8a3-161">用户登录时，应用程序会在类型为 `http://schemas.microsoft.com/ws/2008/06/identity/claims/role` 的声明中收到用户分配到的角色。</span><span class="sxs-lookup"><span data-stu-id="da8a3-161">When a user signs in, the application receives the user's assigned role(s) in a claim with type `http://schemas.microsoft.com/ws/2008/06/identity/claims/role`.</span></span>  

<span data-ttu-id="da8a3-162">用户可以有多个角色，也可以没有角色。</span><span class="sxs-lookup"><span data-stu-id="da8a3-162">A user can have multiple roles, or no role.</span></span> <span data-ttu-id="da8a3-163">不要在授权代码中假设用户只有一个角色声明。</span><span class="sxs-lookup"><span data-stu-id="da8a3-163">In your authorization code, don't assume the user has exactly one role claim.</span></span> <span data-ttu-id="da8a3-164">相反，应写入代码，以检查是否存在特定声明值：</span><span class="sxs-lookup"><span data-stu-id="da8a3-164">Instead, write code that checks whether a particular claim value is present:</span></span>

```csharp
if (context.User.HasClaim(ClaimTypes.Role, "Admin")) { ... }
```

## <a name="roles-using-azure-ad-security-groups"></a><span data-ttu-id="da8a3-165">使用 Azure AD 安全组的角色</span><span class="sxs-lookup"><span data-stu-id="da8a3-165">Roles using Azure AD security groups</span></span>
<span data-ttu-id="da8a3-166">在此方法中，角色表示为 AD 安全组。</span><span class="sxs-lookup"><span data-stu-id="da8a3-166">In this approach, roles are represented as AD security groups.</span></span> <span data-ttu-id="da8a3-167">应用程序根据用户的安全组用户成员身份向用户分配权限。</span><span class="sxs-lookup"><span data-stu-id="da8a3-167">The application assigns permissions to users based on their security group memberships.</span></span>

<span data-ttu-id="da8a3-168">优点：</span><span class="sxs-lookup"><span data-stu-id="da8a3-168">Advantages:</span></span>

* <span data-ttu-id="da8a3-169">对于没有 Azure AD Premium 客户，此方法可使客户利用安全组来管理角色分配。</span><span class="sxs-lookup"><span data-stu-id="da8a3-169">For customers who do not have Azure AD Premium, this approach enables the customer to use security groups to manage role assignments.</span></span>

<span data-ttu-id="da8a3-170">缺点：</span><span class="sxs-lookup"><span data-stu-id="da8a3-170">Disadvantages:</span></span>

* <span data-ttu-id="da8a3-171">复杂。</span><span class="sxs-lookup"><span data-stu-id="da8a3-171">Complexity.</span></span> <span data-ttu-id="da8a3-172">由于每个租户发送的组声明不同，因此应用程序必须为每个租户跟踪哪些安全组对应哪些应用程序角色。</span><span class="sxs-lookup"><span data-stu-id="da8a3-172">Because every tenant sends different group claims, the app must keep track of which security groups correspond to which application roles, for each tenant.</span></span>
* <span data-ttu-id="da8a3-173">如果客户从其 AD 租户中删除应用程序，安全组会保留在其 AD 目录中。</span><span class="sxs-lookup"><span data-stu-id="da8a3-173">If the customer removes the application from their AD tenant, the security groups are left in their AD directory.</span></span>

### <a name="implementation"></a><span data-ttu-id="da8a3-174">实现</span><span class="sxs-lookup"><span data-stu-id="da8a3-174">Implementation</span></span>
<span data-ttu-id="da8a3-175">在应用程序清单中，将 `groupMembershipClaims` 属性设置为“SecurityGroup”。</span><span class="sxs-lookup"><span data-stu-id="da8a3-175">In the application manifest, set the `groupMembershipClaims` property to "SecurityGroup".</span></span> <span data-ttu-id="da8a3-176">从 AAD 获取组成员资格声明需要此属性。</span><span class="sxs-lookup"><span data-stu-id="da8a3-176">This is needed to get group membership claims from AAD.</span></span>

```json
{
   // ...
   "groupMembershipClaims": "SecurityGroup",
}
```

<span data-ttu-id="da8a3-177">新客户注册时，应用程序会指导客户为应用程序所需的角色创建安全组。</span><span class="sxs-lookup"><span data-stu-id="da8a3-177">When a new customer signs up, the application instructs the customer to create security groups for the roles needed by the application.</span></span> <span data-ttu-id="da8a3-178">然后，客户需要将组对象 ID 输入到应用程序中。</span><span class="sxs-lookup"><span data-stu-id="da8a3-178">The customer then needs to enter the group object IDs into the application.</span></span> <span data-ttu-id="da8a3-179">应用程序会将其存储在一个表中，该表按租户将组 ID 映射到应用程序角色。</span><span class="sxs-lookup"><span data-stu-id="da8a3-179">The application stores these in a table that maps group IDs to application roles, per tenant.</span></span>

> [!NOTE]
> <span data-ttu-id="da8a3-180">或者，应用程序可使用 Azure AD Graph API 以编程方式创建组。</span><span class="sxs-lookup"><span data-stu-id="da8a3-180">Alternatively, the application could create the groups programmatically, using the Azure AD Graph API.</span></span>  <span data-ttu-id="da8a3-181">这样就不容易出错。</span><span class="sxs-lookup"><span data-stu-id="da8a3-181">This would be less error prone.</span></span> <span data-ttu-id="da8a3-182">但是，这需要应用程序获取客户 AD 目录的“读取和写入所有组”权限。</span><span class="sxs-lookup"><span data-stu-id="da8a3-182">However, it requires the application to obtain "read and write all groups" permissions for the customer's AD directory.</span></span> <span data-ttu-id="da8a3-183">很多客户可能不愿意授予该级别的访问权限。</span><span class="sxs-lookup"><span data-stu-id="da8a3-183">Many customers might be unwilling to grant this level of access.</span></span>
> 
> 

<span data-ttu-id="da8a3-184">用户登录时：</span><span class="sxs-lookup"><span data-stu-id="da8a3-184">When a user signs in:</span></span>

1. <span data-ttu-id="da8a3-185">应用程序以声明方式接收用户的组。</span><span class="sxs-lookup"><span data-stu-id="da8a3-185">The application receives the user's groups as claims.</span></span> <span data-ttu-id="da8a3-186">每个声明的值是组的对象 ID。</span><span class="sxs-lookup"><span data-stu-id="da8a3-186">The value of each claim is the object ID of a group.</span></span>
2. <span data-ttu-id="da8a3-187">Azure AD 会限制令牌中发送的组数。</span><span class="sxs-lookup"><span data-stu-id="da8a3-187">Azure AD limits the number of groups sent in the token.</span></span> <span data-ttu-id="da8a3-188">如果组数超过此限制，Azure AD 会发送特殊“超额”声明。</span><span class="sxs-lookup"><span data-stu-id="da8a3-188">If the number of groups exceeds this limit, Azure AD sends a special "overage" claim.</span></span> <span data-ttu-id="da8a3-189">如果该声明存在，应用程序必须查询 Azure AD Graph API，以获取该用户所属的所有组。</span><span class="sxs-lookup"><span data-stu-id="da8a3-189">If that claim is present, the application must query the Azure AD Graph API to get all of the groups to which that user belongs.</span></span> <span data-ttu-id="da8a3-190">有关详细信息，请参阅“组声明超额”一节中的[使用 AD 组在云应用程序中授权]。</span><span class="sxs-lookup"><span data-stu-id="da8a3-190">For details, see [Authorization in Cloud Applications using AD Groups], under the section titled "Groups claim overage".</span></span>
3. <span data-ttu-id="da8a3-191">应用程序在其数据库中查找对象 ID，以查找要分配给用户的相应应用程序角色。</span><span class="sxs-lookup"><span data-stu-id="da8a3-191">The application looks up the object IDs in its own database, to find the corresponding application roles to assign to the user.</span></span>
4. <span data-ttu-id="da8a3-192">应用程序将自定义声明值添加到表示应用程序角色的用户主体。</span><span class="sxs-lookup"><span data-stu-id="da8a3-192">The application adds a custom claim value to the user principal that expresses the application role.</span></span> <span data-ttu-id="da8a3-193">例如：`survey_role` = "SurveyAdmin"。</span><span class="sxs-lookup"><span data-stu-id="da8a3-193">For example: `survey_role` = "SurveyAdmin".</span></span>

<span data-ttu-id="da8a3-194">授权策略应使用自定义角色声明，而非组声明。</span><span class="sxs-lookup"><span data-stu-id="da8a3-194">Authorization policies should use the custom role claim, not the group claim.</span></span>

## <a name="roles-using-an-application-role-manager"></a><span data-ttu-id="da8a3-195">使用应用程序角色管理器的角色</span><span class="sxs-lookup"><span data-stu-id="da8a3-195">Roles using an application role manager</span></span>
<span data-ttu-id="da8a3-196">如果使用此方法，应用程序角色不会存储在 Azure AD 中。</span><span class="sxs-lookup"><span data-stu-id="da8a3-196">With this approach, application roles are not stored in Azure AD at all.</span></span> <span data-ttu-id="da8a3-197">相反，应用程序会在自身的数据库中存储每个用户的角色分配 &mdash; 例如，在 ASP.NET Identity 中使用 RoleManager 类。</span><span class="sxs-lookup"><span data-stu-id="da8a3-197">Instead, the application stores the role assignments for each user in its own DB &mdash; for example, using the **RoleManager** class in ASP.NET Identity.</span></span>

<span data-ttu-id="da8a3-198">优点：</span><span class="sxs-lookup"><span data-stu-id="da8a3-198">Advantages:</span></span>

* <span data-ttu-id="da8a3-199">应用可完全控制角色和用户分配。</span><span class="sxs-lookup"><span data-stu-id="da8a3-199">The app has full control over the roles and user assignments.</span></span>

<span data-ttu-id="da8a3-200">缺点：</span><span class="sxs-lookup"><span data-stu-id="da8a3-200">Drawbacks:</span></span>

* <span data-ttu-id="da8a3-201">更为复杂，维护更困难。</span><span class="sxs-lookup"><span data-stu-id="da8a3-201">More complex, harder to maintain.</span></span>
* <span data-ttu-id="da8a3-202">不能使用 AD 安全组管理角色分配。</span><span class="sxs-lookup"><span data-stu-id="da8a3-202">Cannot use AD security groups to manage role assignments.</span></span>
* <span data-ttu-id="da8a3-203">将用户信息存储在应用程序数据库中，若添加或删除用户，用户信息可能与租户的 AD 目录不同步。</span><span class="sxs-lookup"><span data-stu-id="da8a3-203">Stores user information in the application database, where it can get out of sync with the tenant's AD directory, as users are added or removed.</span></span>   


<span data-ttu-id="da8a3-204">[下一篇][授权]</span><span class="sxs-lookup"><span data-stu-id="da8a3-204">[**Next**][authorization]</span></span>

<!-- Links -->
[Tailspin]: tailspin.md

[授权]: authorize.md
[authorization]: authorize.md
[保护后端 Web API]: web-api.md
[Securing a backend web API]: web-api.md
[应用程序清单]: /azure/active-directory/active-directory-application-manifest/
[application manifest]: /azure/active-directory/active-directory-application-manifest/
[sample application]: https://github.com/mspnp/multitenant-saas-guidance
