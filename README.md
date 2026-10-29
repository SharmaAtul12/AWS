# ☁️ My AWS Learning Notes

Simple, step-by-step notes for learning AWS, one service at a time.

Studying for the **AWS Certified Developer – Associate (DVA-C02)** exam, using the *Ultimate AWS Certified Developer Associate 2026 (DVA-C02)* course by **Stephane Maarek** on Udemy, backed by my own notes.

## 📒 Course material

- 📘 **[NOTES.pdf](./NOTES.pdf)** — course notes I'm learning from (added, work in progress)

## 🚀 Learning Journey

A running log of the journey — one entry per study day, so progress stays visible over time. Read it top to bottom like a story: each day flows into the next.

```text
┌─────────────────────────────┐
│   📅 Day 1 — 28 Oct 2026     │
│   Getting Started with AWS   │
└─────────────────────────────┘
               │
               ▼
┌─────────────────────────────┐
│   📅 Day 2 — 29 Oct 2026     │
│   IAM — Identity & Access    │
└─────────────────────────────┘
               │
               ▼
          🔜 Day 3 — EC2 (next)
```

### 📅 Day 1 — 28 October 2026 · Getting Started with AWS

The first day was all about setting the foundation before touching any real service.

- Kicked off the **Ultimate AWS Certified Developer Associate (DVA-C02)** course and set up my notes workflow.
- Learned **what AWS is** — a cloud provider offering on-demand, scalable servers and services — and why it dominates the market.
- Walked through the **AWS Global Infrastructure**: **Regions**, **Availability Zones (AZs)**, **Data Centers**, and **Edge Locations / Points of Presence**.
- Learned **how to choose a Region** — based on compliance/data governance, proximity to customers (latency), service availability, and pricing.
- Understood that each Region has **multiple AZs** (usually 3), each one being one or more isolated data centers connected by low-latency networking.
- Finished with the **Tour of the AWS Console**, learning the key difference between **Global services** (IAM, Route 53, CloudFront, WAF) and **Region-scoped services** (EC2, Elastic Beanstalk, Lambda).

📄 **Notes progress:** slides 1–20 (up to *Tour of the AWS Console*)
🎥 **Course progress:** opening section — *Getting Started with AWS*

                               ⬇️  *the foundation was set, so next came the first real service*

### 📅 Day 2 — 29 October 2026 · IAM (Identity & Access Management) ✅

Day 2 was the first full service — and a big one. **IAM completed**, both the concepts and hands-on practice. Full write-up lives in **[services/IAM.md](./services/IAM.md)**.

- Learned the **core idea of IAM**: who you are, what you can do, on which resources, and under what conditions — covering both **authentication** and **authorization**. Also that IAM is a **global** service.
- Studied the **identity building blocks**:
  - **Root user** — the all-powerful account owner, to be protected with MFA and used only when truly required.
  - **Users** — long-term identities for people, with passwords and/or access keys.
  - **Groups** — collections of users that share policies (groups contain users only).
  - **Roles** — identities that are **assumed temporarily** (trust policy + permission policy), ideal for EC2, Lambda, and cross-account access.
- Broke down **IAM policies** as JSON: `Version`, `Statement`, `Effect`, `Action`, `Resource`, `Condition` — plus the golden rule that **an explicit Deny always wins**.
- Dug into **Action vs Resource vs ARN vs Condition**, including that not every action supports resource-level permissions.
- Learned about **Service Control Policies (SCPs)** as account-level guardrails in AWS Organizations.
- Set up and practiced the **AWS CLI** on Windows (installed v2, ran `aws configure`, verified identity with `aws sts get-caller-identity`).
- **Real debugging lesson:** `aws iam list-users` worked, but `aws ec2 describe-instances` failed with `UnauthorizedOperation ... explicit deny in a service control policy` — proving the CLI was fine and the block came from an **SCP guardrail**, not a broken tool.
- Covered the **AWS SDK** (application code vs CLI vs Console), **IAM security tools** (Credential Report + Access Advisor), **password policy & MFA**, the **shared responsibility model**, and **least privilege** best practices.
- **Hands-on in the console:** created users `dev1` and `dev2`, explored Users / Groups / Roles / Policies / Permissions boundaries, and tried *copy permissions* vs *attach policies directly*.

📄 **Notes progress:** completed the full **AWS IAM** section (slides ~21–38)
🎥 **Course progress:** finished the *AWS Identity & Access Management (IAM)* section
📝 **Deliverable:** wrote a complete, beginner-friendly revision doc → **[services/IAM.md](./services/IAM.md)**

**Latest:** Day 2 (29 Oct 2026) — **IAM is complete** (concepts + CLI/SDK + hands-on). Up next: **Amazon EC2** basics on Day 3.

## 🗺️ Learning Path

```text
 IAM ──▶ EC2 ──▶ VPC ──▶ S3 ──▶ RDS ──▶ Lambda ──▶ CloudWatch / CloudTrail
```

| # | Service | What it is | Notes | Status |
|---|---------|-----------|-------|--------|
| 1 | **IAM** | Who can do what in AWS | [`services/IAM.md`](./services/IAM.md) | ✅ Completed (29 Oct) |
| 2 | **EC2** | Virtual servers | `EC2.md` | 🔜 |
| 3 | **VPC** | Your private network | `VPC.md` | 🔜 |
| 4 | **S3** | File storage | `S3.md` | 🔜 |
| 5 | **RDS** | Managed databases | `RDS.md` | 🔜 |
| 6 | **Lambda** | Run code without servers | `Lambda.md` | 🔜 |
| 7 | **CloudWatch / CloudTrail** | Monitoring and audit logs | `Monitoring.md` | 🔜 |

> Note: detailed IAM revision notes live in [`services/IAM.md`](./services/IAM.md), built from `NOTES.pdf` (pages 1–38) plus hands-on practice.

## 📖 How each file is organized

Every service file uses the same layout:

- 🆕 **What's New** — recent AWS changes, so old tutorials don't confuse you
- 🎯 **Why learn this?** — why the topic matters
- 📚 **The lesson** — simple explanation with flow diagrams
- ✅ **Check yourself** — quick questions before moving on
- ➡️ **Next up** — what comes next, and why
- 📝 **Glossary** — every term in plain words

## 💡 Golden rules (apply to every service)

1. Never use the **root user** for daily work. Turn on **MFA**.
2. Give only the access that is needed (**Least Privilege**).
3. Use **roles** and temporary credentials. Never put access keys in code.
4. **Stop or delete** practice resources and set a **budget alert**.
