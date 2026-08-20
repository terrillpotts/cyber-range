# Development notes

Maintainer notes for editing this project. See `README.md` for the public-facing project overview.

**Two published copies exist and both need updating on changes:**
- Claude Artifact: https://claude.ai/code/artifact/3831172f-1950-4713-80f9-6eb58ad0c3d0 (republish via the Artifact tool, passing that `url` so it updates in place instead of minting a new one)
- GitHub Pages: https://terrillpotts.github.io/proving-grounds/ (served from `index.html` on the `main` branch of the `terrillpotts/proving-grounds` repo — commit/push to update)

**Source of truth for editing:** `proving-grounds.html` in this folder is the durable copy (any Claude session's scratchpad copy is temporary and disappears when that session ends). The version pushed to GitHub as `index.html` should always match it. (Renamed from `cysa-range.html` on 2026-08-18 when the platform was rebranded from "The Cyber Range" to "The Proving Grounds" — the repo was renamed from `cyber-range` to `proving-grounds` at the same time.)

### Objectives alignment (added 2026-08-13)

Terrill supplied the official CompTIA CySA+ CS0-003 Exam Objectives PDF (Document Version 6.0) and asked that all content be rebased against it. A full gap analysis against every sub-bullet in objectives 1.1–4.2 turned up several sub-areas the original build never covered — most notably **2.4 (specific vulnerability/attack types: XSS, injection, SSRF, RCE, overflows, etc.) was entirely missing** despite being a real, testable chunk of the 30%-weight D2 domain. That gap plus named-tool recognition (Nessus, Burp Suite, Nmap, WHOIS, VirusTotal, etc.), OS/network architecture fundamentals (Registry, SASE, SDN, IAM types, PKI, DLP/PII/CHD), and the 2.5 control-type/attack-surface/secure-coding/SDLC taxonomy were addressed with new lessons+quizzes(+labs where hands-on practice fit). Smaller gaps (OSSTMM/OWASP Testing Guide, re-imaging, BC/DR, inhibitors to remediation, incident declaration/escalation, alert volume/SLOs/Top 10) were folded into existing lessons as targeted additions rather than new nodes. **If asked to extend this further, re-check the objectives PDF against `QUIZZES`/`LESSONS` keys first — don't assume the last audit is still complete after adding new content.**

## Architecture (all in one `<script>` tag, IIFE)

- `QUIZZES` — all quiz question banks, keyed by lesson, shared across all sections.
- `LESSONS` — lesson HTML content, keyed by lesson, shared across all sections.
- `LABS` — terminal lab file systems + objectives, keyed by lab, shared across all sections.
- `DOMAINS_CYBERSECURITY` — the 4 CS0-003 domains, each with an ordered `nodes` array (lesson/lab refs, XP, unlock sequencing).
- `DOMAINS_CLOUD` — the 3 AZ-900 domains (Cloud Concepts / Azure Architecture & Services / Azure Management & Governance), same shape as `DOMAINS_CYBERSECURITY`. Added 2026-08-19 — see "Cloud (AZ-900) track" below.
- `SECTIONS` — top-level tracks above domains: `cybersecurity` and `cloud` (both live), `networking` (placeholder, `domains: []`). Each live section carries `domains`, `ranks`, `examPool`, `examComposition`, `examPassPct`, `examXP`, `examTitle`, `examBlueprint`, `examCertifiedLabel` — everything `setSection()` swaps into the bare globals below. Adding a 4th real section means populating a `DOMAINS_*` array in the same shape plus its own `RANKS_*`/`EXAM_*_*` set (see the Cloud block for the exact pattern) — nothing else about the section plumbing needs to change.
- `EXAM_POOL_CYBERSECURITY` / `EXAM_POOL_CLOUD` (+ `_COMPOSITION`, `_PASS_PCT`, `_XP`, `_TITLE`, `_BLUEPRINT`, `_CERTIFIED_LABEL` per section) / `buildExam()` — final exam, built from `QUIZZES` (no duplicated content). `buildExam()` itself is section-agnostic — it just reads whichever bare `EXAM_POOL`/`EXAM_COMPOSITION` are currently swapped in.
- `numberWord(n)` — spells out small integers (`3` → `"three"`) so dashboard/exam copy like "across three domains" stays grammatical regardless of how many domains a section has, instead of hardcoding a word.
- `shuffleQuestion()` — reorders each question's options at render time (see "Known gotchas" below — this was a real bug, don't remove it).
- **Reference-swap pattern:** `state`, `DOMAINS`, `RANKS`, `EXAM_POOL`, `EXAM_COMPOSITION`, `EXAM_PASS_PCT`, `EXAM_XP`, `EXAM_TITLE`, `EXAM_BLUEPRINT`, `EXAM_CERTIFIED_LABEL` are plain top-level `var`s that get *reassigned* by `setSection(id)` to point at whichever section is currently active (`state = sectionStates[id]`, `DOMAINS = sec.domains`, `RANKS = sec.ranks`, `EXAM_POOL = sec.examPool`, etc.). Every pre-existing function (`domainById`, `overallProgress`, `mountQuiz`, `renderLab`, `mountExam`, `getRank`, etc.) still reads the bare globals unmodified; they transparently operate on whatever section is active because the variable itself was swapped, not because those functions know about sections. **Don't "fix" this by threading a `sectionId` param through those functions — that's the whole point of the pattern.** The one deliberate exception is `renderHome()`, which shows *all* sections at once regardless of which is "current" — it calls `getRank(st.xp, sec.ranks)` and `sectionOverallProgress(sec)` to read each section's own state/ranks directly, bypassing the swapped globals (same reasoning as `sectionOverallProgress` already documented below).
- `sectionStates` / `STORAGE_KEY` (`range_cysa_progress_v1`) — one `{xp, completed, quizScores, labProgress, streak, lastVisit, examPassed, examAttempts, examBest}` object per section, persisted together under a single `localStorage` key. `normalizeSectionStates()` migrates the old pre-sections flat-object shape into `sections.cybersecurity` on load, so existing users don't lose progress. Export/Import now serialize/restore all sections at once, not just the active one.
- `render()` / `view` — hand-rolled router. `view.type` is `home` / `dashboard` / `domain` / `lesson` / `lab` / `exam`; every type except `home` also carries `sectionId`. `go(v)` calls `setSection(v.sectionId)` before switching `view`, whenever `v.sectionId` is present.

### Sections architecture (added 2026-08-18)

Three top-level tracks sit above domains: **Cybersecurity** (CS0-003) and **Cloud** (AZ-900) are both live; **Networking** is an empty placeholder — a real nav destination that renders a "coming soon" dashboard via the `if(!DOMAINS.length)` early-return in `renderDashboard()`. XP/rank/streak/exam progress are tracked independently per section via `sectionStates`, not globally. The Home page (`renderHome()`) lists all 3 sections with each one's own XP/rank/progress, pulled directly from `sectionStates[id]` (not the mutable `state` pointer, since Home must show all sections at once regardless of which is "current").

To add real content to Networking later: build a `DOMAINS_NETWORKING` array in the exact shape of `DOMAINS_CYBERSECURITY`/`DOMAINS_CLOUD`, plus its own `RANKS_NETWORKING` ladder and `EXAM_POOL_NETWORKING`/`EXAM_COMPOSITION_NETWORKING`/`EXAM_PASS_PCT_NETWORKING`/`EXAM_XP_NETWORKING`/`EXAM_TITLE_NETWORKING`/`EXAM_BLUEPRINT_NETWORKING`/`EXAM_CERTIFIED_LABEL_NETWORKING`, then point that section's fields at them in `SECTIONS`. Everything else (nav nesting, dashboard, domain/lesson/lab/exam rendering, XP awarding, progress tracking, exam copy) works with zero further changes — that's the payoff of the reference-swap pattern above, now proven twice (Cloud was the first real test of it beyond the original Cybersecurity-only content).

### Cloud (AZ-900) track (added 2026-08-19)

Terrill asked for a second full track under Cloud, based on the official Microsoft Learn AZ-900 study guide PDF (`Study guide for Exam AZ-900_ Microsoft Azure Fundamentals _ Microsoft Learn.pdf`), built the same way as the Cybersecurity/CS0-003 content — same lesson/quiz/lab/capstone shape, same XP/lock/unlock mechanics, same terminal engine (reused as-is for scenario-matching labs rather than literal CLI commands, since AZ-900 doesn't test hands-on CLI). This was also the first real population of the `cloud` section, which required generalizing several things that were previously Cybersecurity-only hardcodes:

- **Domains**: `DOMAINS_CLOUD` — D1 Cloud Concepts (28%, 5 nodes/950 XP), D2 Azure Architecture & Services (38%, 7 nodes/1400 XP), D3 Azure Management & Governance (34%, 6 nodes/1100 XP). Weights roughly match the real AZ-900 blueprint (25-30/35-40/30-35%).
- **Ranks**: `RANKS_CLOUD` — Cloud Trainee through Azure MVP, scaled to the smaller ~3,450 XP domain total (vs. Cybersecurity's ~6,750) rather than reusing `RANKS_CYBERSECURITY`'s SOC-themed names/thresholds.
- **Exam**: `EXAM_POOL_CLOUD`/`EXAM_COMPOSITION_CLOUD` — 20 questions, 6/8/6 split by domain. `EXAM_PASS_PCT_CLOUD = 70` (matches the real AZ-900 passing score of 700/1000) vs. Cybersecurity's 80%. `EXAM_XP_CLOUD = 600`. Title "AZ-900 Final Exam", certified badge "AZ-900 Certified".
- **Generalized UI text**: several strings were hardcoded to "CS0-003"/"CySA+ Certified"/"four domains"/"20 questions" across `renderChrome`, `renderDashboard`, `renderDomain`, `renderExam`, `mountExam` — these now read `EXAM_BLUEPRINT`/`EXAM_TITLE`/`EXAM_CERTIFIED_LABEL`/`numberWord(DOMAINS.length)`/the summed `EXAM_COMPOSITION` count instead, so the exact same render code produces correct copy for either section.
- **Real bug found and fixed during this build**: `LABS.*.title` fields are used in two places — raw HTML insertion in `renderLab()` (wants `&amp;` for a literal `&`) and `showToast('LAB COMPLETE', lab.title)` in `maybeCompleteLab()`, which runs the title through `escapeHtml()` (wants a plain `&`, since escaping already handles the entity). Every pre-existing Cybersecurity lab title happened to have no ampersand, so this dual-usage conflict was latent and untested. The 3 new Cloud lab titles that did contain `&` were authored with `&amp;` (copying the `LESSONS` title convention, which is *only* ever raw-inserted and has no such conflict) and surfaced as a literal `&amp;` in the "LAB COMPLETE" toast. Fixed by using a plain `&` in `LABS.*.title` specifically — browsers tolerate a bare `&` in raw HTML text content fine, so this satisfies both call sites. **If a new lab title needs an ampersand, use a plain `&`, not `&amp;`.**
- Confirmed via live click-through testing (completed all of D1 through the terminal including a multi-objective capstone, spot-checked D2/D3 lesson rendering, shortcut-completed the remaining nodes via direct `localStorage` write to reach 100%, then took and passed the real 20-question final exam end-to-end) — no console errors on either published copy afterward.

## How to keep developing this

1. Edit `proving-grounds.html` directly (plain HTML/CSS/JS, no build step).
2. Test locally before publishing — **do not trust bracket-balance checks alone**, click through it in a real browser. A local static server works well:
   ```powershell
   # from this folder
   $listener = New-Object System.Net.HttpListener
   $listener.Prefixes.Add("http://localhost:8791/")
   $listener.Start()
   while ($true) {
     $context = $listener.GetContext()
     $path = $context.Request.Url.LocalPath.TrimStart('/')
     if ([string]::IsNullOrEmpty($path)) { $path = "proving-grounds.html" }
     $full = Join-Path (Get-Location) $path
     if (Test-Path $full) {
       $bytes = [System.IO.File]::ReadAllBytes($full)
       $context.Response.ContentType = "text/html; charset=utf-8"
       $context.Response.OutputStream.Write($bytes,0,$bytes.Length)
     } else { $context.Response.StatusCode = 404 }
     $context.Response.OutputStream.Close()
   }
   ```
   Then open `http://localhost:8791/proving-grounds.html`. (`file://` also works in a normal browser, just not inside Claude Code's browser-automation tool, which blocks local file navigation.)
3. Republish to the *same* Claude artifact URL by passing `url: "https://claude.ai/code/artifact/3831172f-1950-4713-80f9-6eb58ad0c3d0"` to the Artifact tool — otherwise a new conversation mints a brand-new URL.
4. Push the same updated file to GitHub as `index.html` on `main` (repo: `terrillpotts/proving-grounds`) so the GitHub Pages copy matches.
5. After any content edit, re-copy this file back into the durable project folder (`C:\projects\cyber projects\proving-grounds\proving-grounds.html`) — the scratchpad copy a session works from is temporary.

## Known gotchas (found via testing, already fixed — don't reintroduce)

- **Answer-position bias:** when authoring quiz questions, the correct option kept landing in the same slot (a real bug found on 60/64 questions — all `correct:1`). Fixed via `shuffleQuestion()`, applied in both `mountQuiz()` and `buildExam()`. If you add new questions, you don't need to manually vary option order — the shuffle handles it — but don't remove the shuffle call.
- **Sidebar height:** `.sidebar` must stay `position:sticky; top:0; height:100vh; overflow-y:auto` (desktop) or it stretches to match the tallest page content (CSS Grid default) and the profile card/XP/reset button scroll off-screen on long pages like the exam review screen.
- **`view` must be declared before any code path that can call `render()`/`renderChrome()`** — `touchStreak()` fires at load and cascades into a render; if `var view` isn't initialized before that point (order matters even though `function` declarations hoist, `var` assignments don't), it throws and the whole page renders blank with no visible error unless you check the console.

## Persistence limitation

No cross-device sync capability exists for either published copy. Progress auto-saves to `localStorage` per browser/device (and separately, since the Claude artifact and GitHub Pages copy are different origins — progress does not carry over between them). Export/Import (JSON file) is the manual workaround for moving progress between devices — mention this if it comes up again, it's a platform limitation, not a bug to fix.
