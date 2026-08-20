# The Proving Grounds — Self-Paced Security Training

**[Launch The Proving Grounds →](https://terrillpotts.github.io/proving-grounds/)**

![The Proving Grounds dashboard](screenshot.jpg)

An interactive, HTB/THM-style training platform organized into three top-level topic areas: **Cybersecurity**, **Cloud**, and **Networking**. **Cloud** is itself a group holding two independently-tracked platforms, **Azure** and **AWS**. XP, rank, and streak are tracked separately per track (Cybersecurity, Azure, AWS — Cloud itself has no progress of its own, it's just the landing page for picking a platform). Built as a single self-contained HTML file — no backend, no build step, no dependencies.

## What it is

**Cybersecurity**, **Azure**, and **AWS** are all fully built out; **Networking** is a placeholder track reserved for future content.

**Cybersecurity** is a CompTIA CySA+ (exam CS0-003) curriculum — lessons, hands-on terminal labs, and quizzes across all four exam domains, capped with a mixed final exam.

- **4 exam domains**, each with lessons, hands-on terminal-style labs, and a capstone investigation:
  - **D1 — Security Operations**: log/architecture analysis, indicators of compromise, threat intel & hunting, phishing/email security, SOC tooling & automation
  - **D2 — Vulnerability Management**: scanning & CVSS, prioritizing beyond CVSS (EPSS, exposure, zero-days), cloud/container vulnerabilities, web app vulnerability classes (XSS, injection, SSRF, RCE), scanning tools & frameworks
  - **D3 — Incident Response & Management**: attack frameworks (Kill Chain, Diamond Model, ATT&CK), the IR lifecycle, containment/eradication/recovery, digital forensics fundamentals
  - **D4 — Reporting & Communication**: audience-aware reporting, vulnerability & incident report structure, legal considerations, metrics that matter (KPIs/KRIs, MTTD/MTTR)
- **XP, rank, and streak progression** — 7,500 XP available across all domains and the final exam, with a rank ladder from Trainee Analyst to SOC Director
- **20-question final exam**, pooled from every domain's quiz bank and weighted to match the real CS0-003 blueprint (33/30/20/17% by domain), gated behind 100% domain completion. 80% to pass earns a permanent "CySA+ Certified" badge.
- **Content aligned to the official CompTIA CySA+ CS0-003 Exam Objectives (v6.0)** — every numbered sub-objective (1.1–4.2) is covered by at least one lesson or lab.

**Cloud → Azure** is a Microsoft Azure Fundamentals (exam AZ-900) curriculum, based on the official Microsoft Learn AZ-900 study guide — lessons, scenario-based terminal labs, and quizzes across all three exam domains, capped with a mixed final exam.

- **3 exam domains**, each with lessons, scenario-matching terminal labs, and a capstone:
  - **D1 — Cloud Concepts** (28%): cloud computing & the shared responsibility model, deployment models, consumption-based pricing & serverless, benefits of cloud services, IaaS/PaaS/SaaS
  - **D2 — Azure Architecture & Services** (38%): regions/availability zones/resource hierarchy, compute & virtual networking, storage services/tiers/redundancy, identity/access/security (Entra ID, RBAC, Zero Trust, Defender for Cloud)
  - **D3 — Azure Management & Governance** (34%): cost management, governance & compliance (Purview, Policy, resource locks), deployment tooling (CLI, Arc, IaC/ARM templates), monitoring (Advisor, Service Health, Azure Monitor)
- **XP, rank, and streak progression**, tracked independently from every other track — 3,450 XP across all domains plus the final exam, with its own rank ladder from Azure Trainee to Azure MVP
- **20-question final exam**, weighted 6/8/6 to match AZ-900's real domain weighting, gated behind 100% domain completion. 70% to pass (matching the real exam's passing score) earns a permanent "AZ-900 Certified" badge.

**Cloud → AWS** is an AWS Certified Cloud Practitioner (exam CLF-C02) curriculum — lessons, scenario-based terminal labs, and quizzes across all four exam domains, capped with a mixed final exam.

- **4 exam domains**, each with lessons, scenario-matching terminal labs, and a capstone:
  - **D1 — Cloud Concepts** (24%): the AWS Cloud & its value proposition, the six advantages of cloud computing, the AWS Well-Architected Framework's six pillars
  - **D2 — Security & Compliance** (30%): the AWS shared responsibility model, IAM (users/groups/roles/policies/MFA), governance & compliance concepts (Organizations, SCPs, KMS, Shield, WAF, Artifact)
  - **D3 — Cloud Technology & Services** (34%): global infrastructure (Regions/AZs/Edge Locations), deployment & operation methods (Console/CLI/SDKs/CloudFormation/Elastic Beanstalk), compute (EC2/Lambda/ECS/EKS/Fargate), storage/database/networking (S3/EBS/RDS/DynamoDB/VPC/Route 53/CloudFront)
  - **D4 — Billing, Pricing & Support** (12%): pricing models (On-Demand/Reserved/Spot/Free Tier), billing & cost tools (Cost Explorer/Budgets/Pricing Calculator), support plans & Trusted Advisor
- **XP, rank, and streak progression**, tracked independently from every other track — 3,800 XP across all domains plus the final exam, with its own rank ladder from AWS Trainee to AWS Hero
- **20-question final exam**, weighted 5/6/7/2 to match CLF-C02's real domain weighting, gated behind 100% domain completion. 70% to pass (matching the real exam's passing score) earns a permanent "AWS Cloud Practitioner Certified" badge.

## Try it

Open **[terrillpotts.github.io/proving-grounds](https://terrillpotts.github.io/proving-grounds/)** — progress auto-saves to your browser's local storage. Use the Export/Import buttons in the sidebar to carry progress between devices (there's no server-side account system, so this is manual).

## Architecture

Everything lives in one file, `index.html`, in a single `<script>` tag:

- `QUIZZES` / `LESSONS` / `LABS` — content banks, keyed by lesson/lab id, shared across all tracks
- `DOMAINS_CYBERSECURITY` / `DOMAINS_AZURE` / `DOMAINS_AWS` — each track's ordered list of domains and nodes (lessons/labs) with XP values and unlock sequencing
- `SECTIONS` — a tree of top-level tracks: `cybersecurity` and `networking` are leaves, `cloud` is a *group* holding `azure` and `aws` as its own leaf subsections. Every leaf carries its own domains, rank ladder, and exam config, swapped into a set of bare globals (`DOMAINS`, `RANKS`, `EXAM_POOL`, etc.) whenever the active leaf changes — see `DEVELOPMENT.md` for the full pattern, including how the group nesting works
- `EXAM_POOL_*` / `buildExam()` — each leaf track's final exam questions, drawn from the same `QUIZZES` data (nothing duplicated)
- `sectionStates` — per-track progress persistence via `localStorage`, plus JSON Export/Import covering all tracks at once
- a small hand-rolled router (`render()` / `view`) — no framework, no build step

## Running it locally

It's a static file — open `index.html` directly in a browser, or serve the folder with any static file server.

## License

No license file yet — all rights reserved by default. Open an issue if you'd like to reuse this for your own studying.
