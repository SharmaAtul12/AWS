# AWS IAM — Revision Notes

> AWS Certified Developer Associate (DVA-C02)
> Beginner-friendly revision document. Concepts explained and related to the current AWS experience.

---

## Table of Contents

1. [IAM — Core Concept](#1-iam--core-concept)
2. [Root User](#2-root-user)
3. [IAM Users](#3-iam-users)
4. [IAM Groups](#4-iam-groups)
5. [IAM Roles](#5-iam-roles)
6. [User vs Group vs Role](#6-user-vs-group-vs-role)
7. [IAM Policies](#7-iam-policies)
8. [Action](#8-action)
9. [Resource](#9-resource)
10. [ARN](#10-arn)
11. [Condition](#11-condition)
12. [Policy Evaluation](#12-policy-evaluation)
13. [Service Control Policies (SCP)](#13-service-control-policies-scp)
14. [IAM Policy vs SCP](#14-iam-policy-vs-scp)
15. [IAM Console Practical Work](#15-iam-console-practical-work)
16. [IAM Roles Console](#16-iam-roles-console)
17. [AWS CLI](#17-aws-cli)
18. [AWS CLI Installation on Windows](#18-aws-cli-installation-on-windows)
19. [AWS CLI Configuration](#19-aws-cli-configuration)
20. [AWS STS and Identity Verification](#20-aws-sts-and-identity-verification)
21. [Practical CLI Debugging Lesson](#21-practical-cli-debugging-lesson)
22. [AWS SDK](#22-aws-sdk)
23. [IAM Security Tools](#23-iam-security-tools)
24. [Least Privilege](#24-least-privilege)
25. [Password Policy & MFA](#25-password-policy--mfa)
26. [Shared Responsibility Model for IAM](#26-shared-responsibility-model-for-iam)
27. [IAM Security Best Practices](#27-iam-security-best-practices)
28. [Important Flow Diagrams](#28-important-flow-diagrams)
29. [IAM Mental Models](#29-iam-mental-models)

---

## 1. IAM — Core Concept

**IAM = Identity and Access Management.**

IAM is the AWS service that decides **who can do what** in your AWS account. It answers four questions:

> - **Who are you?** (identity)
> - **What are you allowed to do?** (actions)
> - **What AWS resources can you access?** (resources)
> - **Under what conditions?** (conditions)

IAM controls two things:

- **Authentication** — proving who you are (login, access keys, assumed role).
- **Authorization** — deciding what you're allowed to do once identified.

### IAM is a global service

IAM is **not tied to a single AWS Region**. A user, group, role, or policy you create exists across the whole account. This is different from a service like **EC2**, where resources (instances, volumes) live inside a specific Region.

```text
Region-scoped (e.g. EC2, Lambda)   →  resource lives in us-east-1, eu-west-1, ...
Global (e.g. IAM, Route 53, CloudFront)  →  not bound to one Region
```

---

## 2. Root User

The **root user** is the original identity created when the AWS account is first made. It is the account owner.

- It has **extremely powerful, unrestricted permissions**.
- It should **NOT** be used for normal, day-to-day AWS work.
- Use it **only** for the few tasks that genuinely require root/account-level access (e.g. changing the account's support plan, closing the account, some billing settings).
- **Enable MFA** on the root user.
- **Protect root credentials carefully** — treat them as the keys to the entire account.

Best practice (also stated in the course notes): the root account is created by default and shouldn't be used or shared — use it for account setup, then lock it away.

### Mental model

```text
AWS ACCOUNT
│
├── ROOT USER
│   └── Account owner / extremely powerful
│
└── IAM
    ├── Users
    ├── Groups
    └── Roles
```

---

## 3. IAM Users

> An **IAM User** represents a long-term identity in an AWS account — usually a real person (or sometimes an application).

```text
dev1 → represents a developer/person
dev2 → represents another developer/person
```

Key points:

- IAM users can be given **permissions**.
- Permissions can come **through groups** or as **directly attached policies**.
- IAM users can have **long-term credentials**:
  - **Access keys** (for CLI/SDK/API).
  - A **console password** (for the AWS Management Console).
- **Access keys are secrets** — treat them exactly like passwords.
  - Access Key ID ≈ username
  - Secret Access Key ≈ password
- **Never commit access keys to GitHub** (or any source control).
- In real-world architecture, **prefer temporary credentials / roles** over long-lived access keys where possible.

> IAM Users are still important for certification learning, even though AWS increasingly recommends **temporary credentials** and **IAM Identity Center** (formerly AWS SSO) for human access in production.

---

## 4. IAM Groups

> A **group** is a collection of IAM users.

```text
Developers
│
├── dev1
├── dev2
└── dev3
```

If a policy is attached to the `Developers` group, every user in that group **inherits** those permissions.

Why groups are useful:

- Avoid repeating the same policy for every user.
- Easier, centralized permission management.
- A user can belong to **multiple groups**.

Rules worth remembering (from the course):

- Groups **only contain users** — a group cannot contain another group.
- A user does **not** have to belong to a group, and a user can belong to **several** groups.

### Mental model

```text
USER → belongs to → GROUP → receives → POLICY → gives → PERMISSIONS
```

---

## 5. IAM Roles

Roles are one of the **most important developer concepts** in AWS — study this carefully.

> An **IAM Role** is an identity that can be **assumed temporarily**. It has no long-term password or access keys. When assumed, it hands out **temporary credentials**.

A role has **two sides**:

### Trust Policy

> **WHO or WHAT** is allowed to assume this role?

### Permission Policy

> **WHAT** can the role do once it has been assumed?

### Flow

```text
WHO / WHAT
    ↓
TRUST POLICY
    ↓
CAN ASSUME ROLE?
    ↓
IAM ROLE
    ↓
PERMISSION POLICY
    ↓
TEMPORARY CREDENTIALS
    ↓
AWS RESOURCES
```

### Example

```text
EC2 Instance
      ↓
assumes
      ↓
EC2-S3-Access-Role
      ↓
gets temporary credentials
      ↓
can access S3
```

### Common trust relationships

- **AWS service → role** (e.g. an EC2 instance, a Lambda function, CloudFormation assuming a role).
- **Another AWS account → role** (cross-account access).
- **Custom trust policy** — you manually define the trusted principal.

Common roles you'll see on the exam: **EC2 Instance Roles, Lambda Function Roles, Roles for CloudFormation.**

### Why roles beat embedded access keys

Running an app on an EC2 instance or Lambda? **Attach a role** instead of hard-coding access keys. The role delivers **short-lived, automatically rotated** credentials, so there are no long-lived secrets to leak, store, or accidentally commit.

---

## 6. User vs Group vs Role

| Concept | Meaning | Example |
|---|---|---|
| **User** | Identity representing a person/application | `dev1` |
| **Group** | Collection of users | `Developers` |
| **Role** | Identity that can be assumed | `EC2-S3-Access-Role` |
| **Policy** | Permission rules | Allow `s3:GetObject` |

### Memory trick

```text
User   = Who you are
Group  = Which team you belong to
Policy = What you're allowed to do
Role   = Identity you can temporarily assume
```

---

## 7. IAM Policies

> A **policy** is a **JSON document** that defines permissions.

```text
IAM POLICY
│
├── Effect
├── Action
├── Resource
└── Condition
```

Two top-level wrapper elements:

- **Version** — the policy language version. Always include `"2012-10-17"`.
- **Statement** — one or more permission statements (required). Each statement can also have an optional **Sid** (statement identifier) and, for resource/trust policies, a **Principal** (the account/user/role the statement applies to).

### Example policy (from the course notes)

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": "ec2:Describe*",
      "Resource": "*"
    },
    {
      "Effect": "Allow",
      "Action": "elasticloadbalancing:Describe*",
      "Resource": "*"
    },
    {
      "Effect": "Allow",
      "Action": [
        "cloudwatch:ListMetrics",
        "cloudwatch:GetMetricStatistics",
        "cloudwatch:Describe*"
      ],
      "Resource": "*"
    }
  ]
}
```

### Effect

Two possible values:

```text
Allow
Deny
```

Important rule:

> **Explicit Deny always overrides Allow.**

```text
IAM Policy → Allow EC2
SCP        → Deny EC2
                 ↓
             FINAL = DENY
```

---

## 8. Action

> **Action = What operation can the identity perform?**

Each AWS service exposes its own set of API actions, written as `service:Operation`:

```text
ec2:DescribeInstances
ec2:StartInstances
s3:GetObject
iam:CreateUser
iam:GetUser
```

### Read vs write actions (practical IAM exercise)

Read-only actions give information but do **not** let you change anything:

- **`iam:GetUser`** — retrieve information about an IAM user.
- **`iam:GetUserPolicy`** — retrieve information about an **inline policy** attached to an IAM user.

These are **separate** from create/modify actions:

```text
iam:GetUser
        ↓
READ user information

iam:CreateUser
        ↓
CREATE user
```

Granting `iam:GetUser` does **not** grant `iam:CreateUser`. Each action is a distinct permission.

---

## 9. Resource

> **Resource = Which AWS resource does the action apply to?**

This trips up a lot of beginners, so take it slowly. In the console you usually choose between **All** and **Specific**.

### All

Conceptually equal to:

```json
"Resource": "*"
```

> Apply this permission to **all** resources that the selected action supports.

### Specific

> Apply this permission **only** to the resources you list.

Usually specified with an **ARN**:

```text
arn:aws:iam::ACCOUNT_ID:user/dev1
```

### Mental model

```text
ACTION
  ↓
What can I do?

RESOURCE
  ↓
On what can I do it?
```

### Analogy

```text
"READ files"
       ↓
Action

"READ only these 3 files"
       ↓
Resource restriction
```

### Important nuance

> **Not every AWS action supports resource-level permissions.**

Some actions must use:

```text
Resource: "*"
```

The **AWS Service Authorization Reference** (and the visual policy editor in the console) tells you whether a given action supports resource-level permissions, which condition keys it supports, etc.

---

## 10. ARN

> **ARN = Amazon Resource Name.** It uniquely identifies a single AWS resource.

```text
arn:aws:iam::123456789012:user/dev1
```

### Breakdown

```text
arn
 ↓
aws            (partition)
 ↓
service        (iam, s3, ec2, ...)
 ↓
account        (the 12-digit account id)
 ↓
resource       (user/dev1, bucket, instance id, ...)
```

The general shape is:

```text
arn:partition:service:region:account-id:resource
```

### Examples

```text
IAM user :  arn:aws:iam::123456789012:user/dev1
S3 bucket:  arn:aws:s3:::my-bucket
S3 object:  arn:aws:s3:::my-bucket/reports/2026.csv
EC2 inst :  arn:aws:ec2:us-east-1:123456789012:instance/i-0abc123
```

> Note: IAM and S3 ARNs usually leave the **region** field blank (they're global / region-agnostic), which is why you see `:::` in them.

---

## 11. Condition

> **Condition = Under what circumstances should this permission apply?**

```text
Allow an action
ONLY when MFA is present
```

Conditions let you narrow a permission based on request context, for example:

- **MFA** present or not
- **Source IP address**
- **Time** of day / date range
- **Request attributes** (tags, encryption settings, etc.)
- Other **AWS condition keys**

Example condition requiring MFA:

```json
"Condition": {
  "Bool": { "aws:MultiFactorAuthPresent": "true" }
}
```

Keep it simple: a condition is just an extra "**only if...**" attached to a permission.

---

## 12. Policy Evaluation

The basic rule for deciding whether a request is allowed:

```text
Explicit DENY
     ↓
always wins
```

### Simplified flow

```text
Request
  ↓
Authentication
  ↓
Policies evaluated
  ↓
Explicit Deny?
  ├── YES → DENY ❌
  └── NO
       ↓
   Allow exists?
       ├── YES → ALLOW ✅
       └── NO  → DENY ❌
```

Two things to remember:

- Permissions can come from **multiple policies** (group policies, directly attached policies, etc.) — they **add up**.
- The **default** is deny. If nothing explicitly allows an action, it is denied (**implicit deny**). An **explicit deny** anywhere overrides any allow.

---

## 13. Service Control Policies (SCP)

This was an important **hands-on lesson** from the CLI exercise.

> **SCP = Service Control Policy.** SCPs belong to **AWS Organizations** and act as **permission guardrails** for accounts and organizational units (OUs).

Key idea:

- An SCP **does not grant** permissions by itself.
- It defines the **maximum** set of actions an account is **allowed** to use, and can **explicitly deny** actions across the whole account — regardless of what IAM policies say.

### Example

```text
IAM Policy
    ↓
Allow ec2:DescribeInstances
    ↓
SCP
    ↓
Explicit Deny
    ↓
FINAL RESULT = DENIED
```

### The real error encountered

```text
UnauthorizedOperation

You are not authorized to perform:
ec2:DescribeInstances

with an explicit deny in a service control policy
```

Lesson learned:

> Even when an IAM user has an IAM policy that **allows** an action, an **explicit SCP Deny** can block it.

So permissions aren't just:

```text
IAM User → IAM Policy → AWS Resource
```

They're really:

```text
User / Role
     ↓
IAM permissions
     ↓
AWS Organizations SCP guardrails
     ↓
AWS Resource
```

---

## 14. IAM Policy vs SCP

| IAM Policy | SCP |
|---|---|
| Grants / controls permissions for identities | Sets **maximum** permission boundaries for accounts/OUs |
| Attached to users / groups / roles | Managed through **AWS Organizations** |
| Can **Allow** or **Deny** | Commonly used as **guardrail restrictions** |
| Works at the identity / resource permission level | Restricts what an **account** can ultimately do |
| Explicit Deny wins | Explicit Deny can block an IAM Allow |

In one line:

> **IAM policy = what an identity is granted. SCP = the ceiling an entire account cannot exceed.**

---

## 15. IAM Console Practical Work

What I actually practiced in the IAM Console.

I created an IAM user:

```text
dev1
```

And worked with another user:

```text
dev2
```

I explored these sections of the IAM Console:

- Users
- Groups
- Roles
- Policies
- Permissions
- Permissions boundaries

### Set permissions — the options

**Copy permissions**
> Copies permissions from an existing user, including that user's **group memberships** and attached/inline **policies**. Handy for onboarding a new user identical to an existing one.

**Attach policies directly**
> Attaches a managed policy straight to the user. Works, but AWS generally recommends **group-based management** when multiple users share the same permissions (easier to maintain).

### Permissions boundary

> A **permissions boundary** defines the **maximum** permissions an identity can ever receive.

Important distinction:

```text
Permissions Boundary ≠ permission grant
```

A boundary does **not** give any permissions — it only caps the maximum. The effective permissions are the **intersection** of the boundary and the attached policies. For this learning exercise it can normally be **left unset** unless specifically required.

---

## 16. IAM Roles Console

The current role-creation flow in the AWS Console:

```text
Step 1 → Select trusted entity
Step 2 → Add permissions
Step 3 → Name / Review / Create
```

**Trusted entity types** include:

- **AWS service** (e.g. EC2, Lambda) — the most common choice for a service to act on your behalf.
- **AWS account** — another account (cross-account access).
- **Custom trust policy** — manually define the trusted principal.

Conceptually, creating a role combines:

```text
Trust Policy
+
Permission Policy
=
IAM Role
```

> **Service-linked roles:** AWS automatically creates certain roles for specific services (named like `AWSServiceRoleForXxx`). Don't casually delete these — the service relies on them to function.

---

## 17. AWS CLI

> **AWS CLI = AWS Command Line Interface.**

It lets you interact with AWS services from the terminal instead of clicking through the console. It talks directly to the public AWS service APIs, and it's great for scripting and automation. (It's open source, and is actually built on top of the AWS SDK for Python.)

### Examples

```bash
aws iam list-users
```

```bash
aws ec2 describe-instances --max-items 1
```

```bash
aws s3 ls
```

### General structure

```text
aws
 ↓
service
 ↓
command
 ↓
options
```

### Walkthrough

```text
aws ec2 describe-instances --max-items 1
```

means:

```text
aws
 ↓
EC2 service
 ↓
describe-instances operation
 ↓
return at most 1 result
```

---

## 18. AWS CLI Installation on Windows

I'm on Windows, so the recommended beginner path is **AWS CLI v2** via the **MSI installer**.

```text
AWS CLI v2
      ↓
Windows MSI installer
      ↓
Install
      ↓
Open a NEW CMD / PowerShell / Git Bash
      ↓
aws --version
```

Notes:

- Open a **new** terminal after installing so the updated `PATH` is picked up.
- If you have administrator access, the MSI installer can install **for all users**.
- `msiexec` is just an **alternative** command-line way to run the same MSI — don't overfocus on it; the double-click installer is fine.

### Verify

```bash
aws --version
```

---

## 19. AWS CLI Configuration

After installing, configure credentials so the CLI can authenticate to AWS:

```bash
aws configure
```

It prompts for:

```text
AWS Access Key ID
AWS Secret Access Key
Default region          (e.g. us-east-1)
Output format           (json / text / table)
```

These credentials identify the IAM user (or other credential source) the CLI will act as.

### Security rules

```text
❌ Never commit credentials to GitHub
❌ Never paste secret keys into source code
❌ Never share secret keys
```

> Prefer **temporary credentials / roles** where possible (e.g. an instance role on EC2, or short-lived credentials via IAM Identity Center) instead of long-lived access keys.

---

## 20. AWS STS and Identity Verification

A very useful "who am I?" command:

```bash
aws sts get-caller-identity
```

> It tells you **which AWS identity the CLI is currently using** (Account ID, User/Role ARN, and User ID).

```text
Terminal
   ↓
AWS CLI
   ↓
Credentials
   ↓
STS            (Security Token Service)
   ↓
Current AWS identity
```

This is one of the **first commands to run** when debugging AWS credential issues — it confirms the CLI is authenticated and shows exactly which identity you're acting as.

---

## 21. Practical CLI Debugging Lesson

A concrete debugging experience worth remembering.

I configured the AWS CLI with credentials for:

```text
dev1
```

Then this worked successfully:

```bash
aws iam list-users
```

Which proved:

```text
AWS CLI installation → ✅
Credentials → ✅
Authentication → ✅
IAM permission for that operation → ✅
```

But this failed:

```bash
aws ec2 describe-instances --max-items 1
```

with:

```text
UnauthorizedOperation
... explicit deny in a service control policy
```

### Conclusion

> The AWS CLI itself was **NOT** broken.

The request **reached AWS successfully** (so install + auth were fine). Authorization failed because an **Organizations SCP explicitly denied** the EC2 action. The problem was a permission guardrail, not the tool.

### Debugging flow

```text
Command fails
     ↓
Is AWS CLI installed?        → aws --version
     ↓
Is authentication working?
     ↓
aws sts get-caller-identity  → Which identity am I?
     ↓
Does IAM policy allow the action?
     ↓
Is there an explicit Deny?
     ↓
SCP / permissions boundary / resource policy / etc.
```

---

## 22. AWS SDK

> **AWS SDK = Software Development Kit.** It lets your **application code** interact with AWS services programmatically.

Three ways to reach AWS, compared:

```text
AWS Console
→ GUI (click in the browser)

AWS CLI
→ Terminal commands

AWS SDK
→ Application code (libraries embedded in your app)
```

The SDK is language-specific (JavaScript, Python, Java, Go, .NET, Ruby, PHP, C++, plus mobile and IoT SDKs). Fun fact from the course: the **AWS CLI is built on the AWS SDK for Python (boto)**.

### Conceptual flow

```text
My Node.js Application
        ↓
AWS SDK
        ↓
AWS API
        ↓
S3 / DynamoDB / EC2 / Lambda / etc.
```

### Example — modern AWS SDK for JavaScript (v3)

```javascript
import { S3Client, ListBucketsCommand } from "@aws-sdk/client-s3";

const client = new S3Client({});

const result = await client.send(
  new ListBucketsCommand({})
);

console.log(result.Buckets);
```

### Key takeaways

- **CLI** is convenient for humans / DevOps / scripts.
- **SDK** is convenient when **application code** needs AWS.
- Both ultimately call the **same AWS APIs**.
- Permissions still come from **IAM credentials / roles** either way.

```text
AWS CLI ──────┐
              ├──→ AWS APIs → IAM authorization → AWS service
AWS SDK ──────┘
```

---

## 23. IAM Security Tools

Two important IAM auditing tools.

### IAM Credential Report (account-level)

> The **Credential Report** is an account-level report listing all IAM users and the status of their credentials.

It helps audit things like:

- Whether users have console passwords
- Password age
- Access key information (existence, age, last used)
- MFA-related credential info
- Credential activity / status

```text
IAM Credential Report
        ↓
Account-wide credential audit
```

Useful for periodic **security reviews**.

### IAM Access Advisor (user-level)

> **Access Advisor** shows the service permissions granted to a principal **and when those services were last accessed**.

```text
User/Role
   ↓
Which AWS services have actually been used?
   ↓
Last accessed information
```

Example:

```text
User has permissions for:
EC2
S3
DynamoDB
Lambda
RDS

Access Advisor shows:
EC2 → used recently
S3  → used recently
RDS → never used
```

This supports **least-privilege cleanup** — revoke permissions for services that are never used.

### Distinction to remember

```text
Credential Report
→ Credential / security status (account-level)

Access Advisor
→ Service access / last-accessed info (user-level)
```

---

## 24. Least Privilege

One of the **most important IAM principles**:

> Give an identity **only** the permissions it actually needs — no more.

```text
Bad:
Give everyone AdministratorAccess

Better:
Developer
 ↓
Only required services/actions
 ↓
Only required resources
```

This ties directly to the policy building blocks:

```text
Action    → only the operations needed
Resource  → only the resources needed
Condition → only under the needed circumstances
```

Example:

```text
Allow:   s3:GetObject
On:      a specific bucket/path
Only when: specific conditions are satisfied
```

---

## 25. Password Policy & MFA

*(Supplement from course pages 1–38 — strengthens the security story above.)*

### IAM Password Policy

Stronger passwords = higher account security. In IAM you can configure a password policy:

- Minimum password length
- Require character types (uppercase, lowercase, numbers, non-alphanumeric)
- Allow users to change their own passwords
- Require password rotation after some time (expiration)
- Prevent password re-use

### Multi-Factor Authentication (MFA)

> **MFA = something you know (password) + something you own (security device).**

Main benefit:

> If a password is stolen or hacked, the account is **still not compromised** — the attacker lacks the second factor.

Protect **both** the root account and IAM users with MFA.

**MFA device options in AWS:**

- **Virtual MFA device** — Google Authenticator, Authy (phone apps).
- **Universal 2nd Factor (U2F) security key** — e.g. YubiKey.
- **Hardware key fob MFA device** (incl. a GovCloud variant).

### How users can access AWS (3 options)

```text
1. AWS Management Console   → protected by password + MFA
2. AWS CLI                  → protected by access keys
3. AWS SDK (code)           → protected by access keys
```

---

## 26. Shared Responsibility Model for IAM

Who's responsible for what:

| **AWS is responsible for** | **You are responsible for** |
|---|---|
| Infrastructure (global network security) | Managing Users, Groups, Roles, Policies |
| Configuration and vulnerability analysis of the IAM service | Enabling MFA on all accounts |
| Compliance validation | Rotating keys often |
| | Using IAM tools to apply appropriate permissions |
| | Analyzing access patterns & reviewing permissions |

> AWS secures the IAM service itself; **you** are responsible for configuring it safely.

---

## 27. IAM Security Best Practices

- Don't use **root** for daily work (use it only for account setup / required tasks).
- One physical user = one IAM user. **Never share** IAM users or access keys.
- Enable **MFA** everywhere (root + IAM users).
- Follow **least privilege**.
- Prefer **roles and temporary credentials** where possible.
- Avoid long-lived **access keys** when unnecessary; **rotate** keys often.
- **Never commit credentials** to source control.
- Use **groups** for common user permissions.
- **Review unused permissions** regularly.
- Use the **IAM Credential Report** for credential auditing.
- Use **IAM Access Advisor** to find unnecessary service access.
- Understand **explicit Deny** (it always wins).
- Understand **SCPs** (account-level guardrails).
- Create and enforce a strong **password policy**.
- Protect access keys like passwords.

---

## 28. Important Flow Diagrams

### IAM overview

```text
AWS ACCOUNT
│
├── ROOT USER
│
└── IAM
    ├── USERS
    ├── GROUPS
    ├── ROLES
    └── POLICIES
```

### User permissions

```text
USER
 ↓
GROUP
 ↓
POLICY
 ↓
PERMISSIONS
 ↓
AWS RESOURCE
```

### Role

```text
WHO / WHAT
 ↓
TRUST POLICY
 ↓
ASSUME ROLE
 ↓
IAM ROLE
 ↓
PERMISSION POLICY
 ↓
TEMPORARY CREDENTIALS
 ↓
AWS RESOURCE
```

### Policy

```text
POLICY
│
├── EFFECT → Allow / Deny
├── ACTION → What?
├── RESOURCE → On what?
└── CONDITION → Under what circumstances?
```

### CLI / SDK

```text
AWS CLI ──────┐
              ├──→ AWS APIs → IAM → AWS Services
AWS SDK ──────┘
```

### Authorization

```text
Request
  ↓
IAM permissions
  ↓
Explicit Deny?
  ├── YES → ❌ DENY
  └── NO
       ↓
     Allow?
       ├── YES → ✅ ALLOW
       └── NO  → ❌ DENY
```

---

## 29. IAM Mental Models

### Core vocabulary

```text
User      = Who you are
Group     = Who you are grouped with
Role      = Identity you temporarily assume
Policy    = What you can do
Action    = What operation
Resource  = On what resource
Condition = Under what circumstances
SCP       = Organization-level guardrail
```

### Authentication vs Authorization

```text
Authentication
→ Who are you?

Authorization
→ What are you allowed to do?
```

### Quick permission math

```text
ALLOW + ALLOW          = ALLOW
ALLOW + no permission  = DENY   (implicit deny)
ALLOW + explicit DENY  = DENY   (explicit deny always wins)
```

> ⚠️ This last line is a **simplified learning model**. AWS's full policy evaluation considers multiple policy types and contexts (identity-based policies, resource-based policies, permissions boundaries, SCPs, session policies). But for revision, "**explicit deny always wins, and no allow means deny**" captures the essence.

---

### Quick revision summary

- **Users** → mapped to a physical person, can have a console password.
- **Groups** → contain users only.
- **Policies** → JSON documents that define permissions.
- **Roles** → for EC2 instances / AWS services / cross-account (temporary credentials).
- **Security** → MFA + password policy.
- **AWS CLI** → manage AWS from the command line.
- **AWS SDK** → manage AWS from application code.
- **Access Keys** → used by CLI/SDK (keep secret).
- **Audit** → IAM Credential Report + IAM Access Advisor.
