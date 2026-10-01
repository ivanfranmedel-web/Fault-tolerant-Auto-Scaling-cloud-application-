# AWS Cloud Security Hardening

**End-to-end cloud security case study:** baseline deployment, vulnerability analysis, and defense-in-depth hardening, built on AWS across three iterations.

Live project build-out of a three-tier web application, re-examined through a security engineering lens: attack surface mapping, misconfiguration triage, IAM least-privilege design, and network segmentation. Full write-up in [`docs/Cloud-Security-Hardening-Report.pdf`](docs/Cloud-Security-Hardening-Report.pdf).

---

## Overview

This repository documents a cloud security hardening project completed as part of my Cloud Computing coursework at TU Dublin. Rather than just standing up infrastructure, the goal across all three iterations was to **treat every default AWS setting as a potential vulnerability until proven otherwise**, then systematically close the gaps.

The target system: a three-tier student records application (EC2 to RDS) deployed across two Availability Zones in `eu-west-1`.

**Stack:** Amazon EC2 - VPC - RDS (MySQL) - Application Load Balancer - Auto Scaling - IAM - AWS Secrets Manager - Amazon CloudWatch

---

## The Three Iterations

### 🔹 Iteration 1 - Baseline Architecture & Attack Surface Mapping
Stood up a minimal EC2 instance inside a custom VPC to establish a working baseline, then catalogued the default attack surface before any hardening was applied.

- Built a custom **VPC** (public subnet, Internet Gateway, route table) rather than relying on the AWS default VPC
- Deployed a single **EC2** instance with **no IAM role attached**, flagged as a baseline risk (no scoped permissions, broad implicit trust)
- Security group shipped with **port 22 (SSH) and port 80 (HTTP) open to `0.0.0.0/0`**, documented as the starting attack surface
- Confirmed the instance was **directly internet-facing**, with no intermediary layer (load balancer, WAF, bastion) between the public internet and the application

**Outcome:** a working but unhardened system - the "before" state used as the baseline for every later comparison.

### 🔹 Iteration 2 - Vulnerability Analysis & Risk Research
Decoupled the database from the application server and, in doing so, surfaced a real misconfiguration, reviewed here against relevant **OWASP Top 10 (2021)** categories.

- Migrated the database to a **private-subnet RDS (MySQL)** instance with no route to the Internet Gateway, removing the database from the public attack surface entirely
- Introduced **AWS Secrets Manager** to hold database credentials instead of storing them in application code or environment files
- **Root-caused a live security misconfiguration:** the Secrets Manager secret was found holding a literal placeholder value instead of a real endpoint, silently breaking the credential chain, mapped to **OWASP A05:2021 - Security Misconfiguration** (stale/placeholder config left in a trust-sensitive store)
- Locked the new RDS security group down to **accept inbound traffic only from the application tier's security group** on port 3306, not from any CIDR range

**Outcome:** a documented, reproducible misconfiguration found and fixed through systematic elimination, not luck.

### 🔹 Iteration 3 - Hardening with Defense-in-Depth Controls
Closed the remaining gaps from Iterations 1 and 2 using layered security controls, mapped to **OWASP A01:2021 - Broken Access Control** and standard cloud hardening practice.

- **IAM least-privilege role** attached to EC2 so the application retrieves credentials via a scoped role instead of long-lived static secrets (addresses the implicit-trust gap from Iteration 1)
- **Network segmentation via Application Load Balancer:** EC2 security group rewritten to accept inbound traffic **only from the ALB's security group**, direct internet access to application instances is no longer possible, eliminating the original Iteration 1 attack surface
- **RDS deletion protection + automated backups** (7-day retention), data-integrity and availability controls against accidental or malicious deletion
- **Amazon CloudWatch monitoring dashboard**, detective control covering CPU utilization, request count, and healthy host count in real time
- **Auto Scaling Group (2-4 instances)**, validated with a load test to confirm the system absorbs traffic spikes and scales back down, availability hardening against resource-exhaustion-style conditions

**Outcome:** the Iteration 1 attack surface (open SSH/HTTP to the world, no IAM role, no segmentation) is fully closed, with monitoring in place to detect drift going forward.

---

## Before / After: Security Posture

| Control Area | Iteration 1 (Baseline) | Iteration 3 (Hardened) |
|---|---|---|
| **EC2 exposure** | Public, direct inbound on 22/80 from anywhere | Reachable only via ALB; direct access blocked |
| **Credential handling** | No IAM role attached | Scoped IAM role + Secrets Manager |
| **Database exposure** | N/A (co-located with app) | Private subnet, no internet route |
| **Database resilience** | N/A | Automated backups + deletion protection |
| **Observability** | None | CloudWatch dashboard (CPU / requests / healthy hosts) |
| **Availability** | Single instance, single point of failure | Multi-AZ, Auto Scaling, load-tested |

---

## Core Competencies

- **Cloud Security Architecture** - VPC segmentation, public/private subnet design, defense-in-depth layering
- **IAM & Least Privilege** - scoped instance roles over static credentials
- **Secrets Management** - AWS Secrets Manager integration and credential lifecycle
- **Network Security Controls** - security group design, ALB-only ingress enforcement
- **Vulnerability Triage** - root-cause analysis of a live misconfiguration under time pressure
- **OWASP Top 10 Risk Mapping** - translating infrastructure findings into standard risk categories
- **Monitoring & Detection** - CloudWatch-based observability for drift and anomaly detection
- **Resilience Engineering** - Auto Scaling, automated backups, deletion protection

---

## Project Structure

```
AWS-Cloud-Security-Hardening/
├── README.md
└── docs/
    └── Cloud-Security-Hardening-Report.pdf   # full write-up, all 3 iterations
```

---

## About

**Ivan Medel** - Third-year BSc (Hons) Computing student, TU Dublin (Tallaght), focused on cloud security and AppSec.
📍 Based near Citywest, Dublin, open to a 6-month paid internship.

[LinkedIn](https://linkedin.com/in/medel-ivan-francis) · [GitHub](https://github.com/ivanfranmedel-web)
