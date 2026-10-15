# ☁️ My AWS Learning Notes

Simple, step-by-step notes for learning AWS, one service at a time.

## 🗺️ Learning Path

```text
 ✅ IAM ──▶ EC2 ──▶ VPC ──▶ S3 ──▶ RDS ──▶ Lambda ──▶ CloudWatch / CloudTrail
 (done)
```

| # | Service | What it is | Notes | Status |
|---|---------|-----------|-------|--------|
| 1 | **IAM** | Who can do what in AWS | [IAM.md](./IAM.md) | ✅ Done |
| 2 | **EC2** | Virtual servers | `EC2.md` | ⏳ Next |
| 3 | **VPC** | Your private network | `VPC.md` | 🔜 |
| 4 | **S3** | File storage | `S3.md` | 🔜 |
| 5 | **RDS** | Managed databases | `RDS.md` | 🔜 |
| 6 | **Lambda** | Run code without servers | `Lambda.md` | 🔜 |
| 7 | **CloudWatch / CloudTrail** | Monitoring and audit logs | `Monitoring.md` | 🔜 |

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
