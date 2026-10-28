# ☁️ My AWS Learning Notes

Simple, step-by-step notes for learning AWS, one service at a time.

Studying for the **AWS Certified Developer – Associate (DVA-C02)** exam, using the *Ultimate AWS Certified Developer Associate 2026 (DVA-C02)* course by **Stephane Maarek** on Udemy, backed by my own notes.

## 📒 Course material

- 📘 **[NOTES.pdf](./NOTES.pdf)** — course notes I'm learning from (added, work in progress)

## 🚀 Learning Journey

A running log of the journey — one entry per study day, so progress stays visible over time.

| Day | Date | What I covered | Notes progress | Course progress |
|-----|------|----------------|----------------|-----------------|
| 1 | 28 Oct 2026 | Got started: intro and the **Tour of the AWS Console** | Completed slides 1–20 (up to *Tour of the AWS Console*) | Watched the opening videos of the Udemy course |

**Latest:** Day 1 (28 Oct 2026) — finished the first 20 slides of the notes and the early course videos. Up next: picking up from the slide after the AWS Console tour.

## 🗺️ Learning Path

```text
 IAM ──▶ EC2 ──▶ VPC ──▶ S3 ──▶ RDS ──▶ Lambda ──▶ CloudWatch / CloudTrail
```

| # | Service | What it is | Notes | Status |
|---|---------|-----------|-------|--------|
| 1 | **IAM** | Who can do what in AWS | _notes in NOTES.pdf_ | 📖 Studying |
| 2 | **EC2** | Virtual servers | `EC2.md` | 🔜 |
| 3 | **VPC** | Your private network | `VPC.md` | 🔜 |
| 4 | **S3** | File storage | `S3.md` | 🔜 |
| 5 | **RDS** | Managed databases | `RDS.md` | 🔜 |
| 6 | **Lambda** | Run code without servers | `Lambda.md` | 🔜 |
| 7 | **CloudWatch / CloudTrail** | Monitoring and audit logs | `Monitoring.md` | 🔜 |

> Note: the standalone `IAM.md` file was removed — IAM notes now live in `NOTES.pdf`.

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
