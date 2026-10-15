# 🔐 AWS IAM — Identity and Access Management

> **My AWS learning notes — Service #1: IAM**
> Simple notes, step by step. Read from top to bottom, one section at a time.

---

## 📖 How to use these notes

Every section follows the same pattern, so you always know where you are:

| Part of a section | What it gives you |
|-------------------|-------------------|
| 🎯 **Why learn this?** | The reason this topic matters |
| 📚 **The lesson** | The main idea, explained simply, with diagrams |
| 💡 **Tips / ⚠️ Warnings** | Small things that save you from big mistakes |
| ✅ **Check yourself** | 2–3 quick questions. If you can answer them, move on |
| ➡️ **Next up** | What you'll learn next, and why it comes next |

---

## 🗺️ Learning Roadmap

This is the path these notes follow. Each step builds on the one before it.

```text
 PART 1: BASICS               PART 2: PERMISSIONS          PART 3: HANDS-ON
 ┌──────────────────┐        ┌──────────────────┐        ┌──────────────────┐
 │ 1. What is IAM?  │        │ 4. Policies      │        │ 6. Create dev1   │
 │ 2. Root User     │ ─────▶ │ 5. How login     │ ─────▶ │    (a real user) │
 │ 3. Building      │        │    works (MFA,   │        │ 7. Roles for EC2 │
 │    Blocks        │        │    keys, etc.)   │        │    (machines)    │
 └──────────────────┘        └──────────────────┘        └────────┬─────────┘
                                                                  │
          ┌───────────────────────────────────────────────────────┘
          ▼
 PART 4: REAL WORLD           PART 5: WRAP-UP
 ┌──────────────────┐        ┌──────────────────────────┐
 │ 8. How companies │        │ 10. Best practices       │
 │    use IAM       │ ─────▶ │ 11. Common mistakes      │
 │ 9. Watching &    │        │ 12. The complete flow    │
 │    cleaning up   │        │ 13. What to learn next   │
 └──────────────────┘        │ 14. Glossary             │
                             └──────────────────────────┘
```

---

## 📑 Table of Contents

- [🆕 What's New in AWS IAM (2024–2026)](#-whats-new-in-aws-iam-20242026)
- **Part 1 — Basics**
  - [1. What is IAM?](#1-what-is-iam)
  - [2. The Root User](#2-the-root-user)
  - [3. The 4 Building Blocks](#3-the-4-building-blocks)
- **Part 2 — Permissions**
  - [4. IAM Policies (the rulebook)](#4-iam-policies-the-rulebook)
  - [5. How Login Works (Authentication)](#5-how-login-works-authentication)
- **Part 3 — Hands-on**
  - [6. Hands-on: Create `dev1` and give it EC2 access](#6-hands-on-create-dev1-and-give-it-ec2-access)
  - [7. IAM Roles for EC2 (giving a machine permissions)](#7-iam-roles-for-ec2-giving-a-machine-permissions)
- **Part 4 — Real World**
  - [8. How Real Companies Use IAM](#8-how-real-companies-use-iam)
  - [9. Watching and Cleaning Up Access](#9-watching-and-cleaning-up-access)
- **Part 5 — Wrap-up**
  - [10. Best Practices Checklist](#10-best-practices-checklist)
  - [11. Common Beginner Mistakes](#11-common-beginner-mistakes)
  - [12. Conclusion: The Complete IAM Flow](#12-conclusion-the-complete-iam-flow)
  - [13. What to Learn Next](#13-what-to-learn-next)
  - [14. Glossary](#14-glossary)
  - [📎 Sources](#-sources)

---

## 🆕 What's New in AWS IAM (2024–2026)

AWS changes often. Tutorials and videos can be a year or two old, so your screen may look different. Here are the recent changes that matter for a beginner. Each one is explained again inside the section where it belongs.

| When | What changed | What it means for you | Section |
|------|--------------|-----------------------|---------|
| 2024 → 2025 | **MFA is required for root users** in all account types | You must set up MFA on your root login | [2](#2-the-root-user), [5](#5-how-login-works-authentication) |
| 2024 | **Passkeys** can be used as MFA | Log in with your fingerprint / face / device PIN | [5](#5-how-login-works-authentication) |
| 2024 | **IMDSv2 is the default** on new EC2 setups | Safer way for EC2 to get role credentials | [7](#7-iam-roles-for-ec2-giving-a-machine-permissions) |
| Nov 2024 | **Centralized root access** for member accounts | Companies can remove root passwords from member accounts | [8](#8-how-real-companies-use-iam) |
| Nov 2024 | **Resource Control Policies (RCPs)** | A new kind of company-wide guardrail, for resources | [8](#8-how-real-companies-use-iam) |
| Sep 2025 | **SCPs support the full IAM policy language** | Company guardrails can now be much more precise | [8](#8-how-real-companies-use-iam) |
| Nov 2025 | **`aws login` CLI command** | Use the CLI with your console login, no access keys needed | [5](#5-how-login-works-authentication) |
| Nov 2025 | Free MFA security key program **ended** | Existing keys still work. No new free keys. | [5](#5-how-login-works-authentication) |
| Aug 2026 | **IAM role manager** | AWS can create service roles for you automatically | [7](#7-iam-roles-for-ec2-giving-a-machine-permissions) |
| Sep 2026 | **New AWS experience** (projects + team members) | On new accounts, you **can't** create IAM users with console access. You invite team members instead. | [6](#6-hands-on-create-dev1-and-give-it-ec2-access) |

> 💡 **If your AWS screen doesn't match the tutorial:** check whether your account is on the **new AWS experience**. It hides some classic IAM options on purpose. See [Section 6.4](#64--the-new-aws-ui-why-the-password-option-is-missing).

---

# PART 1 — BASICS

---

## 1. What is IAM?

### 🎯 Why learn this?
IAM is the **front door** of AWS. Every click in the console, every CLI command, and every app that talks to AWS goes through IAM first. If you understand IAM, every other AWS service becomes easier, because they all use it.

### 📚 The lesson

**IAM = Identity and Access Management.**

It answers two questions every time someone (or something) tries to do anything in AWS:

```text
         Someone tries to do something in AWS
                         │
                         ▼
        ┌──────────────────────────────────┐
        │  1. WHO are you?                 │  ← Identity
        │     (prove it: password, MFA…)   │     "Authentication"
        └────────────────┬─────────────────┘
                         ▼
        ┌──────────────────────────────────┐
        │  2. WHAT are you allowed to do?  │  ← Access Management
        │     (check the rulebook)         │     "Authorization"
        └────────────────┬─────────────────┘
                         ▼
               ✅ Allowed   or   ❌ Denied
```

**Simple picture:** IAM is a **security guard with a rulebook** at the entrance of a building.
- The guard checks your **ID card** → *who are you?*
- Then checks the **rulebook** → *which rooms can you enter?*

### Key facts
- IAM is **free**. You never pay for users, groups, roles, or policies.
- IAM is **global**. It is not tied to one region (like Mumbai or Virginia). A user you create works everywhere.
- IAM controls access to **all 200+ AWS services**.

### ✅ Check yourself
1. What are the two questions IAM always asks?
2. Does IAM cost money?
3. If you create a user while the console is set to the Mumbai region, can that user work in the Virginia region?

### ➡️ Next up
Every AWS account starts with one very powerful login: the **Root User**. Before you create anything else, you need to know why that login is dangerous.

---

## 2. The Root User

### 🎯 Why learn this?
The root user is the **master key** to your whole AWS account. Most serious AWS security problems start with a root login that was misused or leaked. Learning to protect it is your first real security task.

### 📚 The lesson

When you create a new AWS account, the **email and password you signed up with** become the **Root User**.

The Root User (the account creator):
- 🛡️ Has **complete, unlimited access** to the whole AWS account.
- 🔲 Can use **all 200+ AWS services**.
- 💳 Can change billing, close the account, and delete anything.
- 🚫 **Cannot be limited** by normal IAM policies inside the account.

### The big question

```text
                 In every company, there are multiple teams
                                    │
      ┌──────────────────┬──────────┴───────┬──────────────────┐
      ▼                  ▼                  ▼                  ▼
  Development         Testing             Cloud              DevOps
     Team              Team               Team                Team

          Can all teams log in using the ROOT credentials
                   and do whatever they want?
                                    │
                                    ▼
                                 ❌ NO
```

### Why not?

If every team shared the root login:
- 💥 Anyone could break or delete **everything**, by accident or on purpose.
- 🕵️ You could never tell **who did what**. Everyone looks like "root".
- 🔓 One leaked password = **the whole account is lost**.

### The right way ✅

```text
  Root User  ──▶  Lock it away + turn on MFA
                  (use only for rare, special tasks)

  Everyone else ──▶  Their OWN identity (user or role)
                     with ONLY the permissions they need
```

This idea, *"give only the access that is needed, nothing more"*, is called the **Principle of Least Privilege**. It is the most important rule in all of AWS security. You'll see it again in every section.

### Tasks only the root user should do

Use root only for these rare jobs:
- Change account settings (account name, root email, root password).
- Change or close **billing / payment** settings, or close the account.
- Fix IAM permissions if every admin got locked out.
- Change your AWS Support plan.
- A few other account-level tasks (for example, signing up as a seller on AWS Marketplace).

For **everything else**, use an IAM user or role.

### ⚠️ Recent changes
- **MFA is now required for root users.** AWS rolled this out for all account types. You'll be asked to set it up. ([AWS announcement](https://aws.amazon.com/about-aws/whats-new/2025/06/aws-iam-mfa-root-users-across-all-account-types/))
- **Companies can now remove root passwords completely** from their member accounts (called *centralized root access*). More on this in [Section 8](#8-how-real-companies-use-iam).

### ✅ Check yourself
1. Who is the root user of an AWS account?
2. Name two reasons teams should not share the root login.
3. What does "Least Privilege" mean, in your own words?

### ➡️ Next up
If not root, then *what* should people log in with? You need to learn the **4 building blocks** of IAM: users, groups, roles, and policies.

---

## 3. The 4 Building Blocks

### 🎯 Why learn this?
Everything in IAM is built from just **four pieces**. Once you know them, every IAM screen in the console will make sense.

### 📚 The lesson

| Building block | What it is | Simple example |
|----------------|-----------|----------------|
| 👤 **User** | One person or one app, with its own long-term login details. | One employee's ID card. |
| 👥 **Group** | A collection of users. Give permissions to the group, and every user in it gets them. | A department, like "Developers". Everyone gets the same keys. |
| 🎭 **Role** | An identity that someone or something *borrows* for a while. It has no password. It gives **temporary** credentials. | A visitor badge you borrow for a task, then return. |
| 📜 **Policy** | A JSON document that lists what is **allowed** or **denied**. | The rulebook that says which doors each ID card opens. |

### How they connect

```text
                    📜 POLICY (the rulebook)
                    "Allow EC2" / "Deny delete"
                             │
               attached to one of these…
          ┌──────────────────┼──────────────────┐
          ▼                  ▼                  ▼
      👤 USER            👥 GROUP            🎭 ROLE
     (one person)     (many users)     (borrowed, temporary)
                           │
                    ┌──────┼──────┐
                    ▼      ▼      ▼
                  dev1   dev2   dev3   ← all get the group's policy
```

> 💡 **Remember:** users, groups, and roles are just **identities**. On their own, they can do **nothing**. The **policy** is what gives them power.

### User or Role? How to choose

```text
          Who needs access?
                │
     ┌──────────┴───────────┐
     ▼                      ▼
  A person               A machine or app
     │                   (EC2, Lambda, …)
     │                          │
     ▼                          ▼
 Is the company using     ✅ Use a ROLE
 SSO (Identity Center)?      (never store keys
     │                        in a machine)
 ┌───┴────┐
 ▼        ▼
Yes       No (small setup / learning)
 │        │
 ▼        ▼
✅ Role   IAM User is OK,
via SSO   but keep it least-privilege + MFA
```

**Why AWS prefers roles:** roles use **temporary credentials** that expire on their own. If someone steals them, they stop working soon. A user's password or access key works until someone manually changes it.

### Groups: a quick tip
Don't attach policies to users one by one. Put users in groups, and attach policies to the group.

```text
 ❌ Hard to manage                  ✅ Easy to manage
 dev1 ── Policy A                   Group "Developers" ── Policy A
 dev2 ── Policy A                        ├── dev1
 dev3 ── Policy A                        ├── dev2
 (change it 3 times)                     └── dev3
                                    (change it once)
```

### ✅ Check yourself
1. Which building block actually gives permissions?
2. Why is a role safer than an access key stored on a server?
3. You hire 10 new developers. Do you attach a policy to each one, or use a group?

### ➡️ Next up
Policies are the heart of IAM. Next you'll learn how to **read and write a policy**, and how AWS decides "allow" or "deny".

---

# PART 2 — PERMISSIONS

---

## 4. IAM Policies (the rulebook)

### 🎯 Why learn this?
Policies are **how** access actually works in AWS. Every "Access Denied" error you'll ever see comes from a policy. If you can read a policy, you can fix those errors yourself.

### 📚 The lesson

### 4.1 — What is a policy?

A **policy** is a document, written in **JSON**, that lists **permissions**: what is **allowed** and what is **denied**.

A policy does nothing by itself. It only works after you **attach** it to a user, group, or role. After that, every time that identity tries to do something, AWS checks its policies first.

```text
   dev1 clicks "Launch EC2 instance"
                 │
                 ▼
   AWS looks at the policies attached to dev1
                 │
       ┌─────────┴──────────┐
       ▼                    ▼
  Policy says ALLOW    No policy allows it
  ec2:RunInstances     (or one says DENY)
       │                    │
       ▼                    ▼
  ✅ Instance launches  ❌ "You are not authorized
                            to perform this operation"
```

### 4.2 — Why do policies matter?

Policies let you:
- 🔒 **Follow Least Privilege** — give only the exact actions needed.
- 🎯 **Be precise** — control down to one action on one resource.
- ♻️ **Reuse rules** — write once, attach to many users or groups.
- 📝 **Keep a clear record** — permissions are written down and can be reviewed.

### 4.3 — Real use cases

| You want to… | What the policy does |
|--------------|----------------------|
| Let the dev team manage EC2, but nothing else | Allow `ec2:*`. Everything else stays denied. |
| Give an app read-only access to one S3 bucket | Allow `s3:GetObject` on that one bucket only |
| Stop anyone from deleting production databases | `Deny` on `rds:DeleteDBInstance` |
| Allow access only when the person used MFA | Add a `Condition` that checks for MFA |
| Allow work only in one region (like Mumbai) | Add a `Condition` on the region |

### 4.4 — The parts of a policy

When you create a new policy, the **AWS Policy Editor** starts you with this template:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "Statement1",
      "Effect": "Allow",
      "Action": [],
      "Resource": []
    }
  ]
}
```

Read it like a sentence:

```text
  "Effect"   +   "Action"          +   "Resource"           +  "Condition"
  ALLOW          to READ FILES         in MY-BUCKET             only IF MFA is used
  (or DENY)      (s3:GetObject)        (arn:aws:s3:::…)         (optional)
```

| Field | Meaning | Example |
|-------|---------|---------|
| **Version** | Version of the policy language. Always use `2012-10-17`. It's not a date you change. | `"2012-10-17"` |
| **Statement** | The list of rules. One policy can hold many rules. | `[ {…}, {…} ]` |
| **Sid** | Optional name for a rule, so humans know what it's for. | `"AllowS3Read"` |
| **Effect** | `Allow` or `Deny`. | `"Allow"` |
| **Action** | What can be done, written as `service:Action`. | `"s3:GetObject"`, `"ec2:*"` |
| **Resource** | Which thing(s) it applies to, written as an **ARN**. `"*"` means everything. | `"arn:aws:s3:::my-bucket/*"` |
| **Condition** | Optional "only if…" rules. | Only if MFA is on. Only from one IP. |

> ⚠️ **Watch the stars:**
> - `"ec2:*"` = every action **in EC2**.
> - `"*"` in Action = every action **in every service**. That's admin-level power. Use it very carefully.

> 💡 **What's an ARN?** An **Amazon Resource Name** is the full "address" of something in AWS.
> Example: `arn:aws:s3:::my-example-bucket` is the address of an S3 bucket.

### 4.5 — A real example: read-only access to one S3 bucket

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "AllowReadOnlyMyBucket",
      "Effect": "Allow",
      "Action": ["s3:GetObject", "s3:ListBucket"],
      "Resource": [
        "arn:aws:s3:::my-example-bucket",
        "arn:aws:s3:::my-example-bucket/*"
      ]
    }
  ]
}
```

**In plain words:** "Allow this identity to **list** the bucket `my-example-bucket` and **read** files inside it. Nothing else."

> 💡 Why two resources? `ListBucket` works on the **bucket itself** (`…bucket`). `GetObject` works on the **files inside it** (`…bucket/*`).

### 4.6 — A Deny example: protect production

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "NeverDeleteDatabases",
      "Effect": "Deny",
      "Action": "rds:DeleteDBInstance",
      "Resource": "*"
    }
  ]
}
```

**In plain words:** "No matter what other policies say, this identity can **never delete** a database."

### 4.7 — The Policy Editor: Visual vs JSON

When you create a policy in the console (**IAM → Policies → Create policy**), there's a toggle in the top-right:

```text
 ┌─────────────────────────────────────────────────────────────┐
 │ Step 1: Specify permissions          [ Visual | JSON ]      │
 │                                                             │
 │  Visual mode:                   JSON mode:                  │
 │  1. Choose a service  (EC2)     Type or paste the policy    │
 │  2. Choose actions    (Start)   document directly.          │
 │  3. Choose resources  (All)                                 │
 │  4. Add conditions    (opt.)                                │
 │         │                              │                    │
 │         └──────── same JSON result ────┘                    │
 ├─────────────────────────────────────────────────────────────┤
 │ Step 2: Review and create  → give it a name → Create        │
 └─────────────────────────────────────────────────────────────┘
```

- **Visual** → point and click. AWS writes the JSON for you. Best when starting out.
- **JSON** → type it yourself. Faster once you know the parts.

> 💡 **Learning trick:** build a policy in **Visual**, then switch to **JSON** to see what AWS wrote. It's a fast way to learn the syntax.

### 4.8 — Types of policies

**By who manages them:**

| Type | Who makes it | When to use it |
|------|-------------|----------------|
| **AWS managed** | AWS (e.g., `AmazonEC2FullAccess`, `ReadOnlyAccess`) | Quick start, common jobs |
| **Customer managed** | You | Your own rules that you reuse on many identities |
| **Inline** | You, inside **one** user/group/role only | Rare one-off cases. It's deleted with that identity. |

**By what they're attached to:**

| Type | Attached to | Example |
|------|-------------|---------|
| **Identity-based** | A user, group, or role | "dev1 can use EC2" |
| **Resource-based** | A resource, like an S3 bucket | "This bucket allows reads from account X" |
| **Permissions boundary** | A user or role, as a **maximum limit** | "dev1 can *never* go beyond EC2 + S3, even if given more" |

> 💡 A **permissions boundary** doesn't give any permissions. It's a **ceiling**. The user gets only what is allowed by **both** their policies **and** the boundary.

### 4.9 — How AWS decides: Allow or Deny?

This is the most important rule in this section:

```text
                   A request comes in
                           │
                           ▼
          ┌───────────────────────────────────┐
          │ Is there an explicit DENY          │── Yes ──▶ ❌ DENIED
          │ in ANY policy that applies?        │          (Deny always wins)
          └──────────────────┬────────────────┘
                             │ No
                             ▼
          ┌───────────────────────────────────┐
          │ Is there an explicit ALLOW         │── No ───▶ ❌ DENIED
          │ for this action + resource?        │          (default = deny)
          └──────────────────┬────────────────┘
                             │ Yes
                             ▼
                         ✅ ALLOWED
```

Remember it as:

> **Explicit Deny  >  Explicit Allow  >  Default Deny**
>
> 1. By default, **everything is denied**.
> 2. An **Allow** opens a door.
> 3. A **Deny** closes it, and **no Allow can open it again**.

> 💡 In companies there are extra layers (SCPs, RCPs, permissions boundaries). Each layer can only **limit** access. None of them **add** access. You'll see the full version in [Section 8.4](#84--company-wide-guardrails-scps-and-rcps).

### ✅ Check yourself
1. What are the 4 main fields inside a policy statement?
2. One policy says Allow `s3:*`, another says Deny `s3:DeleteObject`. Can the user delete a file?
3. What does `"Resource": "*"` mean?

### ➡️ Next up
Policies answer *"what can you do?"*. But first, AWS has to know *"who are you?"*. Next: the different ways to **log in** to AWS.

---

## 5. How Login Works (Authentication)

### 🎯 Why learn this?
There are several ways to prove who you are to AWS. Some are safe. Some are risky if handled badly (like access keys). Choosing the right one is half of AWS security.

### 📚 The lesson

### 5.1 — The ways to log in

```text
                    How are you using AWS?
                             │
         ┌───────────────────┼─────────────────────┐
         ▼                   ▼                     ▼
   🖥️ Web Console       ⌨️ CLI / SDK / code     🤖 A machine (EC2, Lambda)
         │                   │                     │
         ▼                   ▼                     ▼
   Password + MFA       Best: aws login or      IAM Role
   (or SSO login)       SSO (temporary)          (temporary, automatic)
                        Old way: access keys
                        (long-term, risky)
```

| Method | Used for | Expires? | Safety |
|--------|----------|----------|--------|
| **Password** | Console (website) login | No (until changed) | 🟡 OK with MFA |
| **MFA** | A second proof, on top of the password | Code changes every ~30 sec | 🟢 Strong |
| **Access keys** (Key ID + Secret) | CLI / SDK / code | ❌ No, they last forever | 🔴 Risky if leaked |
| **`aws login`** (new) | CLI / SDK, using your console login | ✅ Yes, temporary | 🟢 Strong |
| **SSO / Identity Center** | Console + CLI in companies | ✅ Yes, temporary | 🟢 Strongest for people |
| **IAM Role** | Machines and apps | ✅ Yes, auto-refreshed | 🟢 Strongest for machines |

### 5.2 — MFA (Multi-Factor Authentication)

MFA means you need **two things** to log in:

```text
   Something you KNOW          +        Something you HAVE
   ┌──────────────────┐                ┌──────────────────────┐
   │ 🔑 Password      │                │ 📱 Phone app code    │
   └──────────────────┘                │ 🔐 Security key      │
                                       │ 👆 Passkey (finger / │
                                       │    face / device PIN)│
                                       └──────────────────────┘
                  Both needed ──▶ ✅ Logged in
```

Even if someone steals your password, they **still can't log in** without your phone or key.

**Latest MFA updates:**
- **MFA is mandatory for root users** across all account types. ([AWS](https://aws.amazon.com/about-aws/whats-new/2025/06/aws-iam-mfa-root-users-across-all-account-types/))
- **Passkeys** are supported. You can log in with your fingerprint, face, or device PIN. They are very hard to phish. ([AWS blog](https://aws.amazon.com/blogs/security/passkeys-enhance-security-and-usability-as-aws-expands-mfa-requirements/))
- You can add **up to 8 MFA devices** per user. Add a backup so you don't get locked out if you lose your phone. ([AWS docs](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_credentials_mfa.html))
- AWS's **free MFA security key program ended on Nov 6, 2025**. Existing keys still work. ([AWS blog](https://aws.amazon.com/blogs/security/you-can-now-assign-multiple-mfa-devices-in-iam/))

### 5.3 — Access keys: handle with care

Access keys are a pair: an **Access Key ID** (like a username) and a **Secret Access Key** (like a password). Code and the CLI use them.

> ⚠️ **The biggest beginner mistake in AWS:** putting access keys in code and pushing it to GitHub. Bots scan GitHub all the time, and leaked keys can be abused within minutes (for example, to run expensive servers on your bill).
>
> **Rules:**
> - Never put keys in code or in a Git repo.
> - Never create access keys for the root user.
> - Prefer `aws login`, SSO, or roles instead.
> - If a key leaks: **deactivate it right away**, then delete it and check CloudTrail.

### 5.4 — 🆕 `aws login`: CLI without access keys

Since **November 2025**, the AWS CLI has a command that uses your **normal console login** instead of access keys:

```bash
aws login
```

```text
  You type: aws login
        │
        ▼
  Browser opens ──▶ you sign in (password + MFA)
        │
        ▼
  CLI gets TEMPORARY credentials
  (auto-refreshed for up to 12 hours)
        │
        ▼
  aws s3 ls, aws ec2 describe-instances … ✅
  (no access keys saved on your laptop)
```

Things to know:
- Needs **AWS CLI version 2.32.0 or newer**.
- Works with root, IAM users, and federated logins.
- An IAM user needs the managed policy **`SignInLocalDevelopmentAccess`** to use it.
- If your company uses IAM Identity Center, use `aws configure sso` / `aws sso login` instead.

([AWS CLI docs](https://docs.aws.amazon.com/cli/latest/userguide/cli-configure-sign-in.html))

### ✅ Check yourself
1. What are the "two things" in MFA?
2. Why are access keys more risky than `aws login`?
3. A friend pushed their access keys to GitHub. What should they do first?

### ➡️ Next up
Enough theory. Now let's **build something**: create a real user called `dev1`, give it EC2 permissions, and log in with it.

---

# PART 3 — HANDS-ON

---

## 6. Hands-on: Create `dev1` and give it EC2 access

### 🎯 Why learn this?
This is where theory becomes real. You'll see **authentication** (logging in) and **authorization** (permissions) as two separate steps, and you'll learn why the new AWS screen may look different from the tutorial.

### 📚 The lesson

**Goal:**

```text
 Create dev1 ──▶ Give EC2 permissions ──▶ Let dev1 log in ──▶ dev1 uses EC2 ✅
```

### 6.1 — Step 1: Create the IAM user

Console: **IAM → Users → Create user** → name it `dev1`.

```text
IAM
 └── Users
      └── dev1          ← an IAM user (an identity)
```

### 6.2 — Step 2: Give `dev1` EC2 permissions

Attach the AWS managed policy **`AmazonEC2FullAccess`**:

```text
dev1
 │
 └── 📜 AmazonEC2FullAccess
          │
          └── EC2 ✅
```

> 💡 **Better habit:** create a group called `Developers`, attach the policy to the **group**, and add `dev1` to it. Same result, much easier later.

> ⚠️ `AmazonEC2FullAccess` is fine for learning. In a real job, you'd give a smaller policy (Least Privilege).

Now `dev1` **has permission** to use EC2, but **can't log in to the console yet**.

### 6.3 — Step 3: Turn on console access (the classic way)

In the **classic** IAM console, you can do this in two places:

```text
 Option A: while creating the user
   Create user → ☑ "Provide user access to the AWS Management Console"
              → set a password

 Option B: after creating the user
   IAM → Users → dev1 → Security credentials
       → Console sign-in → Enable console access → set a password
```

### 6.4 — The new AWS UI: why the password option is missing

You may not see any console password option. Your **Security credentials** tab may only show:

```text
Security credentials
 ├── Access keys
 ├── API keys
 ├── SSH keys
 ├── Git credentials
 └── other programmatic credentials
     ❌ no "Console sign-in / password" section
```

**Why?** In **September 2026**, AWS launched a **new AWS experience** for new accounts. It is built around **projects** and **team members**. AWS's own docs say that in this new experience, **you cannot create IAM users with console access**. Instead, you **invite team members**. ([AWS docs](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_users_create.html))

```text
     Classic AWS account                      New AWS experience
 ┌──────────────────────────┐           ┌──────────────────────────────┐
 │ IAM User dev1            │           │ AWS Settings (settings.aws.com)│
 │  + console password      │           │  → Team                       │
 │  + policies              │           │  → Invite new team member     │
 │                          │           │  → choose project(s)          │
 │ Logs in with:            │           │                               │
 │ account ID + user name   │           │ Team member logs in with:     │
 │ + password               │           │ their own AWS Builder ID      │
 └──────────────────────────┘           └──────────────────────────────┘
```

**How team members work in the new experience** ([AWS docs](https://docs.aws.amazon.com/accounts/latest/reference/invite-team-members.html)):
1. Open **AWS Settings** → **Team** → **Invite new team member**.
2. Enter their email and pick the project(s) to share.
3. They have **14 days** to accept. They sign in with an **AWS Builder ID**.
4. Team members get **admin access to the project's AWS services**, but they **can't** touch billing, spend limits, root actions, or invite others.

> ⚠️ In the new experience you **can't pick which services** a team member can use. If you want fine control (for example "EC2 only"), you need to **activate advanced features**. That gives you full control of AWS Organizations, IAM Identity Center, SCPs, and RCPs. ([AWS docs](https://docs.aws.amazon.com/accounts/latest/reference/activate-advanced-features.html))

### 6.5 — Step 3 (other way): create the password with the CLI

Behind the scenes, an IAM user's console password lives in something called a **Login Profile**. In a classic account, if you can't find the button, you can create the login profile with the CLI:

```bash
aws iam create-login-profile \
  --user-name dev1 \
  --password "YourStrongPasswordHere" \
  --password-reset-required
```

- `--password-reset-required` forces `dev1` to choose a **new password** on first login. This is a good habit, because then only `dev1` knows the password.
- For quick practice you can use `--no-password-reset-required` instead.

> ⚠️ If your account is on the **new AWS experience**, AWS says console-access IAM users aren't supported there. Use **Invite team members** (6.4) instead, or activate advanced features.

> 💡 Never paste real passwords into notes or shared files. `YourStrongPasswordHere` is just a placeholder.

### 6.6 — Step 4: `dev1` signs in

IAM users **don't** log in on the normal root email page. They use the **account sign-in page**:

```text
https://<your-account-id>.signin.aws.amazon.com/console
```

They need **3 things**:
1. **Account ID** (12 digits) or account alias
2. **User name** → `dev1`
3. **Password** (from the login profile)

> 💡 You can find the exact sign-in URL in **IAM → Dashboard**. You can also set an **account alias** (like `mycompany-learning`) so the URL is easier to remember.

### 6.7 — The complete `dev1` flow

```text
┌─────────────────────────┐
│       AWS Account       │
└────────────┬────────────┘
             ▼
┌─────────────────────────┐
│      IAM User: dev1     │
└────────────┬────────────┘
      ┌──────┴───────┐
      ▼              ▼
┌────────────┐ ┌──────────────────────┐
│   Login    │ │      IAM Policy      │
│  Profile   │ │ AmazonEC2FullAccess  │
│ (Password) │ │                      │
└─────┬──────┘ └──────────┬───────────┘
      │  WHO are you?     │  WHAT can you do?
      ▼                   │
┌───────────────┐         │
│ AWS Management│         │
│    Console    │         │
└───────┬───────┘         │
        └────────┬────────┘
                 ▼
        ┌─────────────────┐
        │ IAM checks the  │
        │ policies        │
        └────────┬────────┘
                 ▼
        ┌─────────────────┐
        │  Amazon EC2 ✅  │
        └─────────────────┘
```

### 6.8 — The key lesson: two separate things

| Question | Name | Handled by |
|----------|------|-----------|
| **"Who are you?"** | Authentication | **Login Profile** (password) + MFA |
| **"What can you do?"** | Authorization | **IAM Policy** (`AmazonEC2FullAccess`) |

> **Login Profile = can log in.**
> **IAM Policy = can do things.**

They don't depend on each other:

| Has login profile? | Has policy? | Result |
|:------------------:|:-----------:|--------|
| ❌ | ✅ | Can't open the console. (Could still use the CLI with keys.) |
| ✅ | ❌ | Can log in, but sees "Access Denied" everywhere. |
| ✅ | ✅ | ✅ Logs in **and** uses EC2. |

**In our case:** `dev1` + `AmazonEC2FullAccess` was already correct. The only missing piece was the **login profile (password)**.

### ✅ Check yourself
1. What is a Login Profile?
2. `dev1` can log in but sees "Access Denied" on EC2. What is missing?
3. Why might your AWS console not show a console password option at all?

### ➡️ Next up
`dev1` is a **person**. But servers and apps need AWS access too. Should you give a server a password or access key? **No.** Next: **IAM roles for EC2**.

---

## 7. IAM Roles for EC2 (giving a machine permissions)

### 🎯 Why learn this?
Almost every real app on AWS runs on a server that needs to talk to other AWS services (S3, databases, logs). Doing this the wrong way is one of the most common ways companies leak access. Doing it the right way is simple, once you see it.

### 📚 The lesson

### 7.1 — The problem

An **EC2 instance** (a virtual server) runs your app. The app needs to read files from **S3**. How does the server prove who it is?

### 7.2 — The wrong way ❌

```text
┌─────────────────────────────┐
│        EC2 Instance         │
│                             │
│  config.txt:                │
│   ACCESS_KEY=AKIA....       │  ← stored on the server
│   SECRET_KEY=wJal....       │
└─────────────────────────────┘
   😱 Anyone who gets into the server can copy the keys.
   😱 The keys never expire.
   😱 Changing them means updating every server by hand.
```

### 7.3 — The right way ✅: attach an IAM Role

```text
┌──────────────────────────────┐
│         EC2 Instance         │
│                              │
│   Attached IAM Role:         │
│   "EC2-S3-ReadOnly-Role"     │
│          │                   │
│          └── 📜 Policy:      │
│              AmazonS3ReadOnly│
│                              │
│   (no keys stored anywhere)  │
└───────────────┬──────────────┘
                │ uses the role's TEMPORARY credentials
                ▼
        ┌───────────────┐
        │   Amazon S3   │  ← read access ✅
        └───────────────┘
```

### 7.4 — How it works behind the scenes

```text
 ① You create a ROLE  ──▶  ② You attach a POLICY to the role
                                   (e.g., AmazonS3ReadOnlyAccess)
                                          │
                                          ▼
 ③ You attach the role to the EC2 instance
    (at launch: "Advanced details → IAM instance profile",
     or later:  Actions → Security → Modify IAM role)
                                          │
                                          ▼
 ④ AWS creates TEMPORARY credentials for the role and puts them
    in the Instance Metadata Service (IMDS) at 169.254.169.254
    (only code running ON that machine can reach this address)
                                          │
                                          ▼
 ⑤ The AWS CLI / SDK on the instance finds them AUTOMATICALLY
    → your code has ZERO keys in it
                                          │
                                          ▼
 ⑥ AWS refreshes the credentials before they expire ♻️
```

### 7.5 — Trust policy: "who can use this role?"

Every role has **two** policies:

```text
              🎭 IAM ROLE: EC2-S3-ReadOnly-Role
     ┌──────────────────────┬──────────────────────────┐
     │  TRUST POLICY        │  PERMISSIONS POLICY      │
     │  "WHO can wear       │  "WHAT can the wearer    │
     │   this badge?"       │   do?"                   │
     │                      │                          │
     │  → EC2 service       │  → Read from S3          │
     └──────────────────────┴──────────────────────────┘
```

When you create a role for EC2 in the console (**IAM → Roles → Create role → AWS service → EC2**), AWS writes the trust policy for you. It looks like this:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": { "Service": "ec2.amazonaws.com" },
      "Action": "sts:AssumeRole"
    }
  ]
}
```

**In plain words:** "The EC2 service is allowed to **assume** (wear) this role."

> 💡 **Instance Profile:** EC2 can't hold a role directly. It needs a small "wrapper" called an **Instance Profile**. When you create the role in the console, AWS makes the instance profile for you, with the same name. So in the EC2 launch screen you just pick the role by name.

### 7.6 — 🆕 IMDSv2: the safer metadata service

There are two versions of the metadata service:

| | IMDSv1 (old) | IMDSv2 (new, default) |
|---|---|---|
| How it works | Simple request, no token | Must get a **session token** first |
| Safety | Weaker. Some web attacks can trick the server into leaking credentials. | Much harder to trick |
| Today | Being phased out | **Default** on new instance types and console Quick Starts. Amazon Linux 2023 turns off IMDSv1 by default. |

> 💡 In the EC2 launch screen, under **Advanced details → Metadata version**, keep **"V2 only (token required)"**. ([AWS docs](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/configuring-IMDS-new-instances.html))

### 7.7 — 🆕 IAM role manager (August 2026)

Creating roles by hand for every service is extra work. AWS now has **IAM role manager**, an account setting that **creates the right role for you** inside some service consoles. ([AWS docs](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_roles_create_role-manager.html))

```text
  You create a Lambda function in the console
                 │
       role manager is ON?
         ┌───────┴────────┐
         ▼                ▼
        Yes               No
         │                │
         ▼                ▼
  AWS creates the       You create or pick
  role for you from     a role yourself
  a "role template"     (the classic way)
```

Things to know:
- Works in consoles like **Lambda, RDS, CloudFormation, CloudWatch, EKS, Secrets Manager, Step Functions, Glue, Backup, EventBridge, Elastic Beanstalk**, and SageMaker Unified Studio. The list keeps growing.
- Turn it on/off in **IAM → Account settings → Role manager**.
- Roles it creates are **normal roles**. You can view, edit, or delete them. CloudTrail logs their creation.
- For "open-ended" work (like running your own code), the templates can give **broad access**. Review and shrink them to follow Least Privilege.
- On **new AWS experience** accounts it's **on by default**, and you can't turn it off until you activate advanced features.

### 7.8 — Tags vs Roles (don't mix them up)

On the EC2 launch screen, **Tags** and **IAM role** are two **different** sections:

| | 🏷️ Tags | 🎭 IAM Role (Instance Profile) |
|---|---|---|
| What it is | `Key = Value` labels | An identity with permissions |
| Example | `Name = web-server`, `Env = dev` | `EC2-S3-ReadOnly-Role` |
| Used for | Organizing, searching, billing reports | Letting the server use AWS services |
| Gives permissions? | ❌ **No** | ✅ **Yes** |
| Where on launch screen | "Name and tags" | "Advanced details → IAM instance profile" |

> 💡 Advanced: policies **can** check tags in a `Condition` (for example, "devs can stop only instances tagged `Env = dev`"). But a tag by itself never gives permission.

### 7.9 — Why roles are better

- 🔒 **No secrets stored** on the machine. Nothing to leak.
- ⏱️ **Temporary credentials**. A stolen copy stops working soon.
- 🔄 **Easy to change**. Edit the role's policy and every instance gets the change right away.
- 🎯 **Least Privilege**. Give the server only what its app needs.

### ✅ Check yourself
1. Where does an EC2 instance get its role's credentials from?
2. What's the difference between a role's trust policy and its permissions policy?
3. Can a tag like `Env = prod` give a server permission to read S3?

### ➡️ Next up
You now know how **one person** and **one server** get access in **one account**. But real companies have hundreds of people and many accounts. Next: **how companies use IAM in the real world**.

---

# PART 4 — REAL WORLD

---

## 8. How Real Companies Use IAM

### 🎯 Why learn this?
In a real job you'll almost never create IAM users by hand. Companies use **AWS Organizations**, **many accounts**, **SSO**, and **company-wide guardrails**. Knowing this picture makes you job-ready, and it explains why AWS keeps pushing roles and SSO.

### 📚 The lesson

### 8.1 — One account isn't enough

```text
┌──────────────────────────────────────┐
│            One AWS Account            │
│                                       │
│   🖥️ Servers     🗄️ Databases         │
│   👤 Users       ⚙️ Settings          │
│                                       │
│   😬 Testing and Production mixed     │
└──────────────────┬───────────────────┘
                   ▼
           AWS Organizations
```

With one account, a mistake while testing could break production. So companies use **AWS Organizations**: a service to manage **many AWS accounts** from one place, with one bill and shared rules.

### 8.2 — Many accounts, one organization

```text
                        AWS Organizations
                     (Management account at top)
                                │
      ┌───────────┬─────────────┼───────────┬────────────┐
      ▼           ▼             ▼           ▼            ▼
 Development   Testing     Production   Security     Staging
   Account     Account       Account     Account     Account
```

Accounts are often grouped into folders called **OUs (Organizational Units)**, like "Workloads", "Security", "Sandbox". Rules can be applied to a whole OU at once.

**Why separate accounts?**
- 🧱 **Limits damage** — a bug or hack in Development can't reach Production.
- 💰 **Clear billing** — see exactly what each environment costs.
- 🔐 **Strong walls** — each account is fully separate by default.

### 8.3 — 🆕 Centralized root access

Every account in an organization has its **own root user**. That's a lot of master keys to protect.

Since **November 2024**, companies can **remove the root password, access keys, and MFA** from member accounts completely, from one central place. When a root-only task is needed, the security team gets **short-term root access** for just that task. ([AWS docs](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_root-enable-root-access.html))

```text
 Before:  50 accounts = 50 root passwords to protect 😰
 After:   0 root passwords in member accounts
          → short-term root access only when needed ✅
```

### 8.4 — Company-wide guardrails: SCPs and RCPs

**SCP = Service Control Policy.** A rule set at the **organization or OU level** that defines the **maximum** permissions for every account under it.

```text
                    SCP (Service Control Policies)
   ┌─────────────────────┬────────────────────┬────────────────────┐
   │ 📄 Define what      │ 👤 Even if you're  │ 🔒 the SCP can     │
   │ actions are NOT     │ Admin in the       │ still block the    │
   │ allowed             │ Development acct   │ action             │
   └─────────────────────┴────────────────────┴────────────────────┘

   Example: only allow the Mumbai region (ap-south-1)
     🌍 Any other region     →  ❌ Blocked
     🕌 ap-south-1 (Mumbai)  →  ✅ Allowed
```

Example SCP that blocks every region except Mumbai:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "DenyOutsideMumbai",
      "Effect": "Deny",
      "NotAction": ["iam:*", "organizations:*", "sts:*", "support:*"],
      "Resource": "*",
      "Condition": {
        "StringNotEquals": { "aws:RequestedRegion": "ap-south-1" }
      }
    }
  ]
}
```

> 💡 Why the `NotAction` list? Global services like IAM aren't tied to Mumbai, so we leave them out. Otherwise nobody could manage users anymore. Real SCPs usually list a few more global services. Always test SCPs on a test OU first.

**The key idea:**

> **An SCP limits even an Admin.** If the SCP says "Mumbai only", an account admin with `AdministratorAccess` **still can't** launch a server in Virginia. Nothing inside the account can get past an SCP.

**Think of it like this:**

```text
   🏛️ SCP          = the country's CONSTITUTION  (set by the organization)
   📜 IAM policy   = local city LAWS              (set inside each account)

   A city law can never break the constitution.
```

**Important SCP facts:**
- SCPs **never give** permissions. They only set limits. You still need IAM policies to allow things.
- SCPs **don't apply to the management account** (the top account). Keep that account for organization management only.
- 🆕 Since **September 2025**, SCPs support the **full IAM policy language** (conditions, specific resource ARNs, and more), so guardrails can be much more precise. ([AWS blog](https://aws.amazon.com/blogs/security/unlock-new-possibilities-aws-organizations-service-control-policy-now-supports-full-iam-language/))

**🆕 RCP = Resource Control Policy** (since November 2024):

| | SCP | RCP |
|---|---|---|
| Limits… | What **identities** (users/roles) in your accounts can do | What can be done **to your resources** (like S3 buckets), by **anyone** |
| Example | "Our people can't use regions outside Mumbai" | "Nobody outside our company can ever read our S3 buckets" |
| Gives permissions? | ❌ No | ❌ No |

([AWS docs](https://docs.aws.amazon.com/organizations/latest/userguide/orgs_manage_policies_rcps.html))

**The full company-level decision flow** (all layers must agree):

```text
                     A request comes in
                             │
                             ▼
         Explicit DENY anywhere? ──────── Yes ──▶ ❌ DENIED
                             │ No
                             ▼
         Does the SCP allow it? ───────── No ───▶ ❌ DENIED
                             │ Yes
                             ▼
         Does the RCP allow it? ───────── No ───▶ ❌ DENIED
         (if the target resource is covered)
                             │ Yes
                             ▼
         Inside the permissions boundary? ─ No ─▶ ❌ DENIED
         (if one is set)
                             │ Yes
                             ▼
         Does an identity or resource ─── No ───▶ ❌ DENIED
         policy ALLOW it?                          (default deny)
                             │ Yes
                             ▼
                        ✅ ALLOWED
```

> 💡 This is a simplified view. The main point: every layer can only **remove** access. Only identity or resource policies **give** access.

### 8.5 — Nobody creates 500 IAM users by hand

With 5 accounts and hundreds of employees, making an IAM user for each person in each account would be a mess. And when someone leaves, you'd have to delete them everywhere.

Companies already have a **company login system**, the same one used for email and chat:

```text
        Okta          Azure AD (Entra ID)     Google Workspace
          ╲                  │                      ╱
           ╲                 │                     ╱
            ─────────────────┼────────────────────
                             ▼
                ┌──────────────────────────┐
                │  AWS IAM Identity Center │
                │       (AWS SSO)          │
                └──────────────────────────┘
```

**AWS IAM Identity Center** (its old name was *AWS SSO*) **connects** to the company's login system. AWS doesn't keep a second copy of users. One company login now controls AWS access too. This is called **federation**.

> 💡 Microsoft renamed **Azure AD** to **Microsoft Entra ID**. Same product, new name.

### 8.6 — One login, many accounts

```text
                                       ┌── 📦 Development Account
                                       ├── 📋 Testing Account
 👤 Employee ──▶ AWS IAM Identity  ────┼── 🗄️ Production Account
                  Center (SSO)         ├── 🛡️ Security Account
                                       └── 📦 Staging Account
```

The employee logs in **once** and sees a list (the **AWS access portal**) of every account they're allowed to use.

> 💡 This is the company version of what we did for `dev1`. Instead of a password per account, it's **one identity → many accounts**, managed in one place.

### 8.7 — Different roles in different accounts

Logging in is just **authentication**. What you can do in each account comes from **roles**.

In Identity Center you create **permission sets** (for example "Developer" or "ReadOnly"). When you assign a permission set to a person for an account, Identity Center **creates a matching IAM role** in that account.

```text
 👤 Developer ──▶ AWS IAM Identity Center
                         │
             ┌───────────┴────────────┐
             ▼                        ▼
   📦 Development Account     🗄️ Production Account
   ┌─────────────────────┐   ┌──────────────────────┐
   │ 🔑 Developer Role   │   │ 👁️ ReadOnly Role     │
   │ Build / Deploy /    │   │ View logs:     ✅    │
   │ Modify              │   │ Delete server: ❌    │
   └─────────────────────┘   └──────────────────────┘
```

**Same person, different power in each place.** That's Least Privilege at company scale.

Inside an account, the person may have one or more roles to choose from:

```text
 📦 Development Account
   ┌───────────────────────────────────────────────┐
   │ 🔑 Developer Role  → build and deploy  ✅ (assigned)
   │ 🔒 ReadOnly Role   → view only             (not assigned)
   │ 🔒 Admin Role      → full access           (not assigned)
   └───────────────────────────────────────────────┘
```

| Role | Can do |
|------|--------|
| **Admin** | Everything in that account |
| **Developer** | Build, deploy, change app resources |
| **ReadOnly** | Look only. No changes. |

### 8.8 — What the employee actually sees

```text
 👤 Employee
     │
     ▼
 ┌──────────────────────────┐
 │ 📧 name@company.com      │
 │ 🔒 ••••••••••••          │
 └────────────┬─────────────┘
              ▼
 📱 MFA: enter code 123 456
              │
              ▼
 Company login (Okta / Google Workspace / Entra ID) ✅
              │
              ▼
 AWS access portal → pick account → pick role → AWS Console
```

Same as logging in to company email. No separate AWS password. MFA is enforced in one place.

For the CLI, developers run:

```bash
aws configure sso      # one-time setup
aws sso login          # each day: opens the browser, logs in, gets temporary credentials
```

### 8.9 — Offboarding: when someone leaves

```text
 👤 Employee 1 ──▶ ┌────────────────────────────┐
                   │      Company Directory     │       ┌──────────────┐
                   │ Employee 1   [ Active   ]  │ ────▶ │ aws ☁️        │
                   │ Employee 2   [ Inactive ]  │       │ Access       │
                   │ Employee 3   [ Inactive ]  │       │ Granted ✅   │
                   └────────────────────────────┘       └──────────────┘
                     Inactive people get NO access ❌
```

When someone leaves, IT turns off their account **once**, in the company directory. Their access to **every AWS account** stops right away.

> Compare: with manual IAM users, if someone forgets to delete `dev1` in even one account, an ex-employee still has a way in. Central identity removes that risk.

### 8.10 — Everything is recorded

```text
 👥 Security Team ──monitors──▶ ┌──────────────────────────────┐
                                │ 📋 Audit Log (CloudTrail)     │
                                │ • Who logged in?              │
                                │ • Which role did they use?    │
                                │ • What actions did they do?   │
                                │ • Which API calls were made?  │
                                └──────────────┬───────────────┘
                                               │ records
                                               ▼
                                      🔒 Production Account
```

**AWS CloudTrail** records every action. Companies usually send all CloudTrail logs to the separate **Security account**, so nobody in other accounts can delete them. This is needed for audits (SOC 2, ISO 27001, HIPAA) and for investigating problems.

### 8.11 — Apps use roles too

The same idea from [Section 7](#7-iam-roles-for-ec2-giving-a-machine-permissions), in a real production app:

```text
┌──────────────────────────────┐
│        🖥️ EC2 Instance        │
│  ┌────────────────────────┐  │         ┌─────────────────┐
│  │  ⏻ Spring Boot App     │──┼───✅───▶│  🗄️ RDS Database │
│  └────────────────────────┘  │         └─────────────────┘
│         🎭 IAM Role          │
└──────────────────────────────┘
```

The Spring Boot app **doesn't store a database password** in its code. It uses its **IAM role** to either:
- connect to RDS using **IAM database authentication** (a short-lived login token), or
- fetch the database password from **AWS Secrets Manager** at startup.

Same rule as always: **no stored secrets, temporary credentials, only the access needed.**

### 8.12 — The full company picture

```text
  Company Directory (Okta / Entra ID / Google Workspace)
                     │  employee logs in once, with MFA
                     ▼
         AWS IAM Identity Center (SSO)
                     │  permission sets → roles in each account
    ┌────────────────┼──────────────┬───────────────┬──────────────┐
    ▼                ▼              ▼               ▼              ▼
 Dev Acct        Test Acct      Prod Acct      Security Acct   Staging Acct
 Developer Role                 ReadOnly Role  (logs stored)
 (build/deploy)                 (view only)
    │                               │
    ▼                               ▼
 EC2 + Spring Boot ──role──▶ RDS   (apps use roles, no stored secrets)

 ┌────────────────────────────────────────────────────────────────┐
 │ All accounts live inside:   AWS Organizations                  │
 │ Root passwords:             removed (centralized root access)  │
 │ Guardrails everywhere:      SCPs + RCPs (e.g., Mumbai only)    │
 │ Every action recorded by:   CloudTrail → Security Team         │
 └────────────────────────────────────────────────────────────────┘
```

**In one line:** one trusted login, many accounts, the right role in each account, company-wide guardrails on top, and a full record underneath.

### ✅ Check yourself
1. Why do companies use many AWS accounts instead of one?
2. An admin in the Dev account tries to launch a server in Virginia, but the SCP says "Mumbai only". What happens?
3. Someone leaves the company. With Identity Center, how many places does IT need to turn off their access?

### ➡️ Next up
Giving access is only half the job. Over time, people collect permissions they no longer use. Next: how to **watch and clean up** access.

---

## 9. Watching and Cleaning Up Access

### 🎯 Why learn this?
Least Privilege isn't "set once and forget". Permissions pile up. Keys get old. People change teams. AWS gives you free tools to find and remove access that nobody uses.

### 📚 The lesson

```text
     GIVE access  ──▶  WATCH it  ──▶  CLEAN it up  ──┐
         ▲                                            │
         └───────────────── repeat ───────────────────┘
```

| Tool | What it tells you | Where to find it |
|------|-------------------|------------------|
| **CloudTrail** | Who did what, when, from where | CloudTrail console → Event history |
| **IAM Access Analyzer — external access** | Which of your resources (S3, KMS, roles…) can be reached from **outside** your account or company | IAM → Access Analyzer |
| **IAM Access Analyzer — internal access** | Who **inside** your company can reach your important resources | IAM → Access Analyzer |
| **IAM Access Analyzer — unused access** | Roles, keys, passwords, and permissions that **haven't been used** | IAM → Access Analyzer |
| **Last accessed info** | When a user or role last used each service | IAM → Users/Roles → *Last Accessed* tab |
| **Credential report** | A CSV of all users: password age, MFA on/off, key age | IAM → Credential report |
| **Policy validation** | Warnings and errors while you write a policy | Shown inside the Policy Editor |

> 💡 For unused permissions, Access Analyzer can **suggest a smaller policy** based on what was actually used. It also gives quick links to delete unused roles, keys, and passwords. ([AWS docs](https://docs.aws.amazon.com/IAM/latest/UserGuide/access-analyzer-findings.html))

> ⚠️ External access analysis is free. The **unused access** analyzer is a **paid** feature (charged per role/user checked each month). Check the pricing before turning it on in a personal account.

> 🆕 **IAM Policy Autopilot** (2026) can help create least-privilege policies by looking at your app's code. It's even available as a **Kiro Power**. ([AWS](https://aws.amazon.com/about-aws/whats-new/2026/02/aws-iam-policy-autopilot-kiro-power/))

### A simple monthly clean-up routine

```text
 □ Open the credential report → anyone without MFA? Fix it.
 □ Any access keys older than 90 days? Rotate them, or delete them.
 □ Any users/roles not used in 90 days? Disable, then delete them.
 □ Check "Last accessed" → remove services nobody uses.
 □ Check Access Analyzer → anything shared outside by mistake?
```

### ✅ Check yourself
1. Which tool tells you who deleted a server last Tuesday?
2. Which tool finds S3 buckets that people outside your account can reach?
3. What should you do with an access key that hasn't been used in 6 months?

### ➡️ Next up
You've now seen the whole IAM story. Next, let's turn it all into a **checklist** you can follow every time.

---

# PART 5 — WRAP-UP

---

## 10. Best Practices Checklist

### 🎯 Why learn this?
This is the short version of the whole document. Come back to this list whenever you set up a new AWS account.

### 📚 The checklist

**🔑 Root user**
- [ ] Turn on **MFA** for the root user (passkey or authenticator app). Add a backup MFA device.
- [ ] **Never** create access keys for root.
- [ ] Use root only for the rare root-only tasks.

**👤 People**
- [ ] Give everyone their **own** login. Never share logins.
- [ ] In companies, use **IAM Identity Center (SSO)** instead of IAM users.
- [ ] Turn on **MFA** for every human login.
- [ ] Put users in **groups**. Attach policies to groups.
- [ ] Set a strong **password policy** (IAM → Account settings).

**📜 Permissions**
- [ ] Follow **Least Privilege**. Start small and add only what's needed.
- [ ] Use AWS managed policies to start, then move to smaller **customer managed** policies.
- [ ] Use **Deny** rules to protect critical things (like production databases).
- [ ] Use **permissions boundaries** if you let others create roles.

**🤖 Machines and code**
- [ ] Use **IAM roles** for EC2, Lambda, and other services. **Never** store keys on servers.
- [ ] Keep **IMDSv2** required on EC2.
- [ ] Use **`aws login`** or **`aws sso login`** for the CLI instead of long-term access keys.
- [ ] **Never** put keys in code or Git.
- [ ] If role manager creates roles for you, **review** them and shrink broad permissions.

**🏢 Companies (many accounts)**
- [ ] Use **AWS Organizations** with separate accounts for Dev / Test / Prod / Security.
- [ ] Use **SCPs** (and **RCPs**) for guardrails no account admin can get around.
- [ ] Turn on **centralized root access** to remove member-account root passwords.
- [ ] Send all **CloudTrail** logs to a separate Security account.

**🧹 Ongoing**
- [ ] Rotate or remove old access keys.
- [ ] Delete unused users, roles, and permissions.
- [ ] Review **IAM Access Analyzer** findings regularly.

### ➡️ Next up
Knowing the right way is good. Knowing the **common wrong ways** helps even more.

---

## 11. Common Beginner Mistakes

### 🎯 Why learn this?
Almost everyone makes these mistakes when starting AWS. Seeing them now means you won't.

| ❌ Mistake | 😬 What can go wrong | ✅ Do this instead |
|-----------|---------------------|-------------------|
| Using root for daily work | One slip can delete everything. No record of who did what. | Create an admin user/role and use that. |
| Pushing access keys to GitHub | Bots find them in minutes. Surprise bill. | Use `aws login` or roles. Add `.aws/` and `.env` to `.gitignore`. |
| Giving everyone `AdministratorAccess` "to save time" | Anyone can break anything. | Give the smallest policy that works. |
| Storing keys inside EC2 | Hack the server = steal the keys. | Attach an IAM role. |
| Thinking tags give permissions | Server gets "Access Denied". | Attach an IAM role (Instance Profile). |
| Forgetting MFA | Stolen password = full access. | Turn on MFA (passkeys are easiest). |
| Sharing one IAM user among a team | No idea who did what. Can't remove one person. | One identity per person. |
| Never cleaning up old users and keys | Old, forgotten access becomes a back door. | Monthly review with the credential report. |
| Following an old tutorial exactly | Buttons are missing or moved. | Check the [What's New](#-whats-new-in-aws-iam-20242026) table. Your account may be on the new AWS experience. |

### ➡️ Next up
Time to put everything together into **one complete flow**.

---

## 12. Conclusion: The Complete IAM Flow

### 🎯 Why learn this?
This section connects every part of the document into **one story**: from your first login, to `dev1`, to servers, all the way to how a big company runs IAM.

### 📚 The complete flow

```text
 ════════════════════════════════════════════════════════════════════
  STAGE 1 — FOUNDATIONS                              (Sections 1–3)
 ════════════════════════════════════════════════════════════════════
   You create an AWS account
          │
          ▼
   👑 Root User exists (unlimited power)
          │
          ▼
   🔒 Turn on MFA for root → lock it away
      (use only for billing / account-level emergencies)
          │
          ▼
   🧱 Learn the 4 blocks: User · Group · Role · Policy

 ════════════════════════════════════════════════════════════════════
  STAGE 2 — PERMISSIONS & LOGIN                      (Sections 4–5)
 ════════════════════════════════════════════════════════════════════
   📜 Policies (JSON): Effect + Action + Resource (+ Condition)
          │
          ▼
   ⚖️ Rule: Explicit Deny > Explicit Allow > Default Deny
          │
          ▼
   🔑 Login: password + MFA · aws login · SSO · roles
      (avoid long-term access keys)

 ════════════════════════════════════════════════════════════════════
  STAGE 3 — A PERSON GETS ACCESS (dev1)                  (Section 6)
 ════════════════════════════════════════════════════════════════════
   👤 Create IAM user "dev1"
          │
          ├──▶ 🔐 Login Profile (password)   → WHO are you?   (Authentication)
          └──▶ 📜 AmazonEC2FullAccess        → WHAT can you do? (Authorization)
          │
          ▼
   dev1 signs in at <account-id>.signin.aws.amazon.com
          │
          ▼
   IAM checks policies ──▶ EC2 ✅

   (New AWS experience? → Invite a team member in AWS Settings instead.)

 ════════════════════════════════════════════════════════════════════
  STAGE 4 — A MACHINE GETS ACCESS (EC2 role)             (Section 7)
 ════════════════════════════════════════════════════════════════════
   🖥️ Launch EC2
          │
          ▼
   🎭 Attach IAM Role (through an Instance Profile)
          │
          ▼
   ⏱️ Temporary credentials from IMDSv2 (auto-refreshed)
          │
          ▼
   App talks to S3 / RDS — no keys stored anywhere ✅

 ════════════════════════════════════════════════════════════════════
  STAGE 5 — THE COMPANY GROWS                        (Sections 8–9)
 ════════════════════════════════════════════════════════════════════
   🏢 One account ──▶ many accounts in AWS Organizations
          │             (Dev · Test · Prod · Security · Staging)
          ▼
   🗝️ Centralized root access → no root passwords in member accounts
          │
          ▼
   🏛️ SCPs + RCPs → guardrails even admins can't get past
          │
          ▼
   🌐 Company login (Okta / Entra ID / Google) → IAM Identity Center
          │
          ▼
   🎭 Right role per account (Developer in Dev, ReadOnly in Prod)
          │
          ▼
   🚪 Employee leaves → turn off once → access gone everywhere
          │
          ▼
   📋 CloudTrail records everything → Security Team reviews
          │
          ▼
   🧹 Access Analyzer finds unused access → clean up → repeat
```

### The one idea behind everything

Every topic in this document is really **the same pattern**, just at bigger and bigger sizes:

```text
   ┌──────────┐    ┌──────────────┐    ┌──────────────┐    ┌──────────┐    ┌──────────┐
   │ 1. WHO   │ ─▶ │ 2. PROVE IT  │ ─▶ │ 3. WHAT CAN  │ ─▶ │ 4. ONLY  │ ─▶ │ 5. WRITE │
   │ is       │    │ (Authenti-   │    │ THEY DO?     │    │ WHAT'S   │    │ IT ALL   │
   │ asking?  │    │  cation)     │    │ (Authori-    │    │ NEEDED   │    │ DOWN     │
   │          │    │              │    │  zation)     │    │ (Least   │    │ (Cloud-  │
   │          │    │              │    │              │    │ Privilege│    │  Trail)  │
   └──────────┘    └──────────────┘    └──────────────┘    └──────────┘    └──────────┘
```

| Size | What "Identity" is | How they prove it | What gives permission |
|------|-------------------|-------------------|-----------------------|
| 🧑 One learner | IAM user `dev1` | Password + MFA | IAM policy |
| 🖥️ One server | IAM role on EC2 | Temporary credentials (IMDSv2) | Role's policy |
| 🏢 A whole company | Employee in the company directory | SSO + MFA | Permission set → role per account, limited by SCPs/RCPs |

> **The tools get bigger, but the idea never changes:**
> **know who's asking, give them only what they need, and record everything.**
>
> That's IAM, and it's the base of security in all of AWS.

---

## 13. What to Learn Next

### 🎯 Why this order?
Every AWS service uses IAM. The next services are picked so that each one **uses what you just learned** and prepares you for the next.

```text
   ✅ IAM  (you are here)
     │   "Who can do what"
     ▼
   🖥️ EC2 — virtual servers
     │   You'll use: IAM roles on instances, IMDSv2, key pairs,
     │   security groups
     ▼
   🌐 VPC — your private network
     │   Where EC2 lives. Subnets, route tables, security groups.
     ▼
   🪣 S3 — file storage
     │   You'll use: bucket policies (resource-based policies!),
     │   Block Public Access, the S3 policy example from Section 4.5
     ▼
   🗄️ RDS — managed databases
     │   You'll use: IAM database authentication, Secrets Manager,
     │   the Spring Boot example from Section 8.11
     ▼
   ⚡ Lambda — run code without servers
     │   You'll use: execution roles (role manager can create them)
     ▼
   📊 CloudWatch + CloudTrail — watching everything
         You'll use: the audit ideas from Sections 8.10 and 9
```

### Practice before moving on (hands-on tasks)
- [ ] Turn on MFA for your root user (try a passkey).
- [ ] Create a group `Developers` with `AmazonEC2ReadOnlyAccess`, and put `dev1` in it.
- [ ] Log in as `dev1` and try to **launch** an instance. You should see "Access Denied". Then explain why.
- [ ] Write a custom policy in the **Visual** editor, then switch to **JSON** and read it.
- [ ] Create a role for EC2 with `AmazonS3ReadOnlyAccess`. Attach it to an instance and run `aws s3 ls` from inside it.
- [ ] Install AWS CLI v2.32+ and try `aws login`.
- [ ] Open the **credential report** and read it.

> 💡 **Free Tier tip:** stop or terminate EC2 instances when you're done practicing, and set up a **budget alert** in Billing so you never get a surprise bill.

---

## 14. Glossary

| Term | Simple meaning |
|------|----------------|
| **IAM** | Identity and Access Management. Controls who can do what in AWS. |
| **Authentication** | Proving **who you are** (logging in). |
| **Authorization** | Deciding **what you're allowed to do** (permissions). |
| **Root User** | The all-powerful login created with the account. |
| **IAM User** | One identity (person or app) with long-term login details. |
| **IAM Group** | A collection of users that share the same policies. |
| **IAM Role** | An identity that is borrowed for a while, with temporary credentials. |
| **Policy** | A JSON document that allows or denies actions. |
| **Statement** | One rule inside a policy. |
| **Effect / Action / Resource / Condition** | The parts of a rule: allow or deny / what / on which thing / only if. |
| **ARN** | Amazon Resource Name. The full "address" of something in AWS. |
| **Least Privilege** | Give only the permissions that are needed. Nothing more. |
| **Explicit Deny** | A `Deny` rule. Always wins over any `Allow`. |
| **Implicit (Default) Deny** | If nothing allows it, it's denied. |
| **AWS Managed Policy** | A ready-made policy from AWS (e.g., `AmazonEC2FullAccess`). |
| **Customer Managed Policy** | A policy you create and can reuse. |
| **Inline Policy** | A policy built into one single user, group, or role. |
| **Resource-based Policy** | A policy attached to a resource (e.g., an S3 bucket policy). |
| **Permissions Boundary** | A maximum limit on what a user or role can ever do. |
| **Login Profile** | The console password of an IAM user. |
| **MFA** | Multi-Factor Authentication. Password + a second proof. |
| **Passkey** | Log in with fingerprint, face, or device PIN. Very hard to phish. |
| **Access Keys** | Long-term Key ID + Secret used by the CLI or code. Handle with care. |
| **`aws login`** | CLI command (2025) that uses your console login to get temporary credentials. |
| **Temporary Credentials** | Short-lived keys that expire on their own (used by roles and SSO). |
| **Trust Policy** | The part of a role that says **who** can use (assume) it. |
| **AssumeRole** | The action of "putting on" a role to get its permissions. |
| **Instance Profile** | A wrapper that lets an EC2 instance use an IAM role. |
| **IMDS / IMDSv2** | Instance Metadata Service at `169.254.169.254`. Where EC2 gets its role credentials. v2 is the safer, token-based version. |
| **IAM Role Manager** | 2026 feature that creates service roles for you automatically. |
| **New AWS Experience** | 2026 simplified AWS setup, built on projects and team members. |
| **AWS Builder ID** | A personal AWS identity. Team members use it in the new AWS experience. |
| **AWS Organizations** | Manage many AWS accounts from one place. |
| **Management Account** | The top account of an organization. |
| **OU (Organizational Unit)** | A folder of accounts inside an organization. |
| **SCP (Service Control Policy)** | Organization guardrail that limits what identities can do, even admins. |
| **RCP (Resource Control Policy)** | Organization guardrail that limits what can be done to your resources. |
| **Centralized Root Access** | Removing root passwords from member accounts and using short-term root only when needed. |
| **IAM Identity Center (AWS SSO)** | Connects a company login to AWS. One login for many accounts. |
| **Permission Set** | A template in Identity Center that becomes a role in each assigned account. |
| **Federation** | Logging in to AWS with an outside login system (Okta, Entra ID, Google). |
| **Identity Provider (IdP)** | The outside login system (Okta, Entra ID, Google Workspace). |
| **CloudTrail** | Records every action in your AWS account. |
| **IAM Access Analyzer** | Finds outside access, internal access, and unused access. |
| **Credential Report** | A CSV list of all users with their password, MFA, and key status. |

---

## 📎 Sources

Recent updates in these notes come from official AWS pages:

- [MFA required for root users across all account types](https://aws.amazon.com/about-aws/whats-new/2025/06/aws-iam-mfa-root-users-across-all-account-types/)
- [Passkeys and expanded MFA requirements](https://aws.amazon.com/blogs/security/passkeys-enhance-security-and-usability-as-aws-expands-mfa-requirements/)
- [MFA in IAM (up to 8 devices)](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_credentials_mfa.html)
- [Multiple MFA devices / free security key program ended](https://aws.amazon.com/blogs/security/you-can-now-assign-multiple-mfa-devices-in-iam/)
- [Create an IAM user (new AWS experience note)](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_users_create.html)
- [Invite team members in AWS Settings](https://docs.aws.amazon.com/accounts/latest/reference/invite-team-members.html)
- [Activate advanced AWS features](https://docs.aws.amazon.com/accounts/latest/reference/activate-advanced-features.html)
- [New AWS experience announcement (Sep 2026)](https://aws.amazon.com/about-aws/whats-new/2026/09/New-AWS-Builder-Experience/)
- [IAM role manager](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_roles_create_role-manager.html)
- [`aws login` — CLI with console credentials](https://docs.aws.amazon.com/cli/latest/userguide/cli-configure-sign-in.html)
- [IMDSv2 for new instances](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/configuring-IMDS-new-instances.html)
- [Centralize root access for member accounts](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_root-enable-root-access.html)
- [SCPs now support the full IAM language](https://aws.amazon.com/blogs/security/unlock-new-possibilities-aws-organizations-service-control-policy-now-supports-full-iam-language/)
- [Resource control policies (RCPs)](https://docs.aws.amazon.com/organizations/latest/userguide/orgs_manage_policies_rcps.html)
- [IAM Access Analyzer findings](https://docs.aws.amazon.com/IAM/latest/UserGuide/access-analyzer-findings.html)
- [IAM Policy Autopilot as a Kiro Power](https://aws.amazon.com/about-aws/whats-new/2026/02/aws-iam-policy-autopilot-kiro-power/)

> *Content from AWS sources was rephrased for compliance with licensing restrictions.*
