# Governance-Security-Hardening-AzureRBAC


# Lab 05 — Implementing Governance & Security Hardening




## Objective

In this lab, you will act as a **Cloud Administrator**. You will step out of the "Owner" role and learn how to restrict access for others. By the end, you will have:

- Created a user in **Microsoft Entra ID** and assigned limited permissions via **RBAC**
- Enforced rules with **Azure Policy** to block expensive VM deployments
- Set up a **Budget alert** to monitor spending



## Architecture

```
Admin (You)
    ├── Applies Policy ──────────────────┐
    └── Assigns RBAC ────────────────────┤
                                         ▼
                              Target Resource Group
                                         │
Junior User ──── Tries to Create VM ─── Blocked by Azure
```



## Prerequisites

- [ ] Active Azure Subscription
- [ ] Completed Week 5 Video Modules
- [ ] Ability to open an **Incognito / Private** browser window



## Lab Variables (Naming Convention)

Replace `[yourname]` with your first name, lowercase, no spaces.

| Resource | Name |
|---|---|
| Resource Group | `rg-lab05-gov-[yourname]` |
| Test User | `junior-dev-[yourname]` |
| Policy Assignment | `Restrict-VM-Sizes` |



## Phase 1 — Setup the Playground

1. Log in to the **Azure Portal** with your main admin account
2. In the search bar, go to **Resource Groups** → **+ Create**
3. Configure the resource group:
   - **Name:** `rg-lab05-gov-[yourname]`
   - **Region:** East US
4. Click **Review + create** → **Create**



## Phase 2 — RBAC (Role-Based Access Control)

We'll simulate hiring a junior developer who can only **view** resources — not modify them.

### Create the User

1. Search for **Microsoft Entra ID** in the portal search bar
2. In the left menu, click **Users** → **New user** → **Create new user**
3. Fill in the **Basics** tab:
   - **User principal name:** `junior-dev-[yourname]` *(the domain will auto-fill, e.g. `@yourtenant.onmicrosoft.com`)*
   - **Display name:** `Junior Developer`
   - **Password:** Uncheck *Auto-generate* and set a password you'll remember
4. Click **Review + create** → **Create**

### Assign the Role

1. Navigate to **Resource Groups** → `rg-lab05-gov-[yourname]`
2. In the left menu, click **Access control (IAM)**
3. Click **+ Add** → **Add role assignment**
4. **Role tab:** Search for `Reader`, select it → click **Next**
5. **Members tab:** Click **+ Select members** → search for `Junior Developer` → select them
6. Click **Review + assign** → **Review + assign**



## Phase 3 — Verify Access (The "Access Denied" Test)

1. Open a **New Incognito / Private** browser window
2. Go to [portal.azure.com](https://portal.azure.com)
3. Log in as `junior-dev-[yourname]@[yourdomain]`
4. Navigate to **Resource Groups** → `rg-lab05-gov-[yourname]`
5. Click **+ Create** inside the resource group
6. Search for **Storage Account** and attempt to create one

**Expected result:** Validation fails, the *Create* button is greyed out, or a red banner appears stating you do not have authorization.

> ✅ **Success!** You have successfully restricted this user. Close the Incognito window and return to your admin account.



## Phase 4 — Azure Policy (Preventing Expensive Mistakes)

Back as your admin account, you'll enforce a rule that prevents anyone — including yourself — from deploying expensive VM sizes.

1. Search for **Policy** in the portal search bar
2. Under **Authoring**, click **Assignments**
3. Click **Assign policy**

### Basics Tab

| Field | Value |
|---|---|
| Scope | Click `...` → select your subscription → select `rg-lab05-gov-[yourname]` as the Resource Group |
| Policy definition | Click `...` → search for `Allowed virtual machine size SKUs` → select it |
| Assignment name | `Restrict-VM-Sizes` |

### Parameters Tab

1. Uncheck **"Only show parameters that need input"**
2. **Allowed Size SKUs:** Select **only** `Standard_B1s` and `Standard_B1ms`

> Any attempt to deploy a D-series, G-series, or other expensive SKU will now be blocked within this resource group.

4. Click **Review + create** → **Create**

> ⏱ **Note:** Azure Policy can take **15–30 minutes** to take effect. If the test below doesn't block immediately, wait and try again.



## Phase 5 — Testing the Policy

1. Go to `rg-lab05-gov-[yourname]`
2. Click **Create** → **Virtual Machine**
3. Configure the **Basics** tab:
   - **Name:** `vm-policy-test`
   - **Image:** Ubuntu Server
   - **Size:** Change to `Standard_D2s_v3` *(or any size that is NOT B1s)*
4. Click **Review + create**

**Expected result:** A **Validation Failed** error appears. Click the error — it should cite **Policy check failed** and list `Restrict-VM-Sizes` as the reason.

> ✅ **Success!** The environment is now governed.



## Phase 6 — Cost Management (Budgets)

1. Go to `rg-lab05-gov-[yourname]`
2. In the left menu, find **Budgets** (under Cost Management) → click **+ Add**

### Create Budget

| Field | Value |
|---|---|
| Name | `Monthly-Lab-Budget` |
| Reset period | Billing month |
| Creation date | Today |
| Expiration date | 1 year from today |
| Amount | `$50` |

Click **Next**.

### Set Alert

| Field | Value |
|---|---|
| Type | Actual |
| % of budget | 80 |
| Alert recipients | Your personal email address |

Click **Create**.

> You'll receive an email alert when spending reaches 80% of your $50 monthly budget for this resource group.



## Troubleshooting

| Issue | Cause | Fix |
|---|---|---|
| Junior Dev can still create resources | Role may have been assigned at the wrong scope | Confirm the **Reader** role is assigned specifically to `rg-lab05-gov-[yourname]`, not the whole subscription. Test inside that exact resource group. |
| Policy didn't block the VM | Policy hasn't replicated yet | Wait 10–15 minutes and try again. Azure Policy is not instantaneous. |



## Clean Up Resources

1. Delete the resource group:
   - Go to **Resource Groups** → `rg-lab05-gov-[yourname]` → **Delete resource group**

2. Delete the test user:
   - Go to **Microsoft Entra ID** → **Users**
   - Find `junior-dev-[yourname]` → click **Delete**

> ⚠️ Always delete the test user. Leaving unused accounts in your directory is a security risk.



## Key Takeaways

- **RBAC** controls *who* can do what — assign the least privilege needed (Reader, not Owner)
- **Azure Policy** controls *what* can be deployed — enforced at the subscription or resource group scope
- **Budgets** don't stop spending, but they alert you before costs spiral
- Governance is a layered approach: identity + policy + cost controls working together
