# The Proving Grounds — Self-Paced Security Training

**[Launch The Proving Grounds →](https://terrillpotts.github.io/proving-grounds/)**

![The Proving Grounds dashboard](screenshot.jpg)

An interactive, HTB/THM-style training platform organized into three independently-tracked topic areas: **Cybersecurity**, **Cloud**, and **Networking**. XP, rank, and streak are tracked separately per track. Built as a single self-contained HTML file — no backend, no build step, no dependencies.

## What it is

**Cybersecurity** and **Cloud** are both fully built out; **Networking** is a placeholder track reserved for future content.

**Cybersecurity** is a CompTIA CySA+ (exam CS0-003) curriculum — lessons, hands-on terminal labs, and quizzes across all four exam domains, capped with a mixed final exam.

- **4 exam domains**, each with lessons, hands-on terminal-style labs, and a capstone investigation:
  - **D1 — Security Operations**: log/architecture analysis, indicators of compromise, threat intel & hunting, phishing/email security, SOC tooling & automation
  - **D2 — Vulnerability Management**: scanning & CVSS, prioritizing beyond CVSS (EPSS, exposure, zero-days), cloud/container vulnerabilities, web app vulnerability classes (XSS, injection, SSRF, RCE), scanning tools & frameworks
  - **D3 — Incident Response & Management**: attack frameworks (Kill Chain, Diamond Model, ATT&CK), the IR lifecycle, containment/eradication/recovery, digital forensics fundamentals
  - **D4 — Reporting & Communication**: audience-aware reporting, vulnerability & incident report structure, legal considerations, metrics that matter (KPIs/KRIs, MTTD/MTTR)
- **XP, rank, and streak progression** — 7,500 XP available across all domains and the final exam, with a rank ladder from Trainee Analyst to SOC Director
- **20-question final exam**, pooled from every domain's quiz bank and weighted to match the real CS0-003 blueprint (33/30/20/17% by domain), gated behind 100% domain completion. 80% to pass earns a permanent "CySA+ Certified" badge.
- **Content aligned to the official CompTIA CySA+ CS0-003 Exam Objectives (v6.0)** — every numbered sub-objective (1.1–4.2) is covered by at least one lesson or lab.

**Cloud** is a Microsoft Azure Fundamentals (exam AZ-900) curriculum, based on the official Microsoft Learn AZ-900 study guide — lessons, scenario-based terminal labs, and quizzes across all three exam domains, capped with a mixed final exam.

- **3 exam domains**, each with lessons, scenario-matching terminal labs, and a capstone:
  - **D1 — Cloud Concepts** (28%): cloud computing & the shared responsibility model, deployment models, consumption-based pricing & serverless, benefits of cloud services, IaaS/PaaS/SaaS
  - **D2 — Azure Architecture & Services** (38%): regions/availability zones/resource hierarchy, compute & virtual networking, storage services/tiers/redundancy, identity/access/security (Entra ID, RBAC, Zero Trust, Defender for Cloud)
  - **D3 — Azure Management & Governance** (34%): cost management, governance & compliance (Purview, Policy, resource locks), deployment tooling (CLI, Arc, IaC/ARM templates), monitoring (Advisor, Service Health, Azure Monitor)
- **XP, rank, and streak progression**, tracked independently from Cybersecurity — 3,450 XP across all domains plus the final exam, with its own rank ladder from Cloud Trainee to Azure MVP
- **20-question final exam**, weighted 6/8/6 to match AZ-900's real domain weighting, gated behind 100% domain completion. 70% to pass (matching the real exam's passing score) earns a permanent "AZ-900 Certified" badge.

## Try it

Open **[terrillpotts.github.io/proving-grounds](https://terrillpotts.github.io/proving-grounds/)** — progress auto-saves to your browser's local storage. Use the Export/Import buttons in the sidebar to carry progress between devices (there's no server-side account system, so this is manual).

## Architecture

Everything lives in one file, `index.html`, in a single `<script>` tag:

- `QUIZZES` / `LESSONS` / `LABS` — content banks, keyed by lesson/lab id, shared across all tracks
- `DOMAINS_CYBERSECURITY` / `DOMAINS_CLOUD` — each track's ordered list of domains and nodes (lessons/labs) with XP values and unlock sequencing
- `SECTIONS` — the top-level tracks (Cybersecurity, Cloud, Networking); each carries its own domains, rank ladder, and exam config, swapped into a set of bare globals (`DOMAINS`, `RANKS`, `EXAM_POOL`, etc.) whenever the active track changes — see `DEVELOPMENT.md` for the full pattern
- `EXAM_POOL_*` / `buildExam()` — each track's final exam questions, drawn from the same `QUIZZES` data (nothing duplicated)
- `sectionStates` — per-track progress persistence via `localStorage`, plus JSON Export/Import covering all tracks at once
- a small hand-rolled router (`render()` / `view`) — no framework, no build step

## Running it locally

It's a static file — open `index.html` directly in a browser, or serve the folder with any static file server.

## License

No license file yet — all rights reserved by default. Open an issue if you'd like to reuse this for your own studying.
