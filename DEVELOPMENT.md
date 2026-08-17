# Development notes

Maintainer notes for editing this project. See `README.md` for the public-facing project overview.

**Two published copies exist and both need updating on changes:**
- Claude Artifact: https://claude.ai/code/artifact/3831172f-1950-4713-80f9-6eb58ad0c3d0 (republish via the Artifact tool, passing that `url` so it updates in place instead of minting a new one)
- GitHub Pages: https://terrillpotts.github.io/cyber-range/ (served from `index.html` on the `main` branch of this repo — commit/push to update)

**Source of truth for editing:** `cysa-range.html` in this folder is the durable copy (any Claude session's scratchpad copy is temporary and disappears when that session ends). The version pushed to GitHub as `index.html` should always match it.

### Objectives alignment (added 2026-08-13)

Terrill supplied the official CompTIA CySA+ CS0-003 Exam Objectives PDF (Document Version 6.0) and asked that all content be rebased against it. A full gap analysis against every sub-bullet in objectives 1.1–4.2 turned up several sub-areas the original build never covered — most notably **2.4 (specific vulnerability/attack types: XSS, injection, SSRF, RCE, overflows, etc.) was entirely missing** despite being a real, testable chunk of the 30%-weight D2 domain. That gap plus named-tool recognition (Nessus, Burp Suite, Nmap, WHOIS, VirusTotal, etc.), OS/network architecture fundamentals (Registry, SASE, SDN, IAM types, PKI, DLP/PII/CHD), and the 2.5 control-type/attack-surface/secure-coding/SDLC taxonomy were addressed with new lessons+quizzes(+labs where hands-on practice fit). Smaller gaps (OSSTMM/OWASP Testing Guide, re-imaging, BC/DR, inhibitors to remediation, incident declaration/escalation, alert volume/SLOs/Top 10) were folded into existing lessons as targeted additions rather than new nodes. **If asked to extend this further, re-check the objectives PDF against `QUIZZES`/`LESSONS` keys first — don't assume the last audit is still complete after adding new content.**

## Architecture (all in one `<script>` tag, IIFE)

- `QUIZZES` — all quiz question banks, keyed by lesson.
- `LESSONS` — lesson HTML content, keyed by lesson.
- `LABS` — terminal lab file systems + objectives, keyed by lab.
- `DOMAINS` — the 4 domains, each with an ordered `nodes` array (lesson/lab refs, XP, unlock sequencing).
- `EXAM_POOL` / `EXAM_COMPOSITION` / `buildExam()` — final exam, built from `QUIZZES` (no duplicated content).
- `shuffleQuestion()` — reorders each question's options at render time (see "Known gotchas" below — this was a real bug, don't remove it).
- `state` / `STORAGE_KEY` (`range_cysa_progress_v1`) — progress persistence via `localStorage`, plus Export/Import as JSON (no cross-device sync capability exists for Claude artifacts, so this is the workaround).
- `render()` / `view` — simple hand-rolled router (`dashboard` / `domain` / `lesson` / `lab` / `exam`), no framework.

## How to keep developing this

1. Edit `cysa-range.html` directly (plain HTML/CSS/JS, no build step).
2. Test locally before publishing — **do not trust bracket-balance checks alone**, click through it in a real browser. A local static server works well:
   ```powershell
   # from this folder
   $listener = New-Object System.Net.HttpListener
   $listener.Prefixes.Add("http://localhost:8791/")
   $listener.Start()
   while ($true) {
     $context = $listener.GetContext()
     $path = $context.Request.Url.LocalPath.TrimStart('/')
     if ([string]::IsNullOrEmpty($path)) { $path = "cysa-range.html" }
     $full = Join-Path (Get-Location) $path
     if (Test-Path $full) {
       $bytes = [System.IO.File]::ReadAllBytes($full)
       $context.Response.ContentType = "text/html; charset=utf-8"
       $context.Response.OutputStream.Write($bytes,0,$bytes.Length)
     } else { $context.Response.StatusCode = 404 }
     $context.Response.OutputStream.Close()
   }
   ```
   Then open `http://localhost:8791/cysa-range.html`. (`file://` also works in a normal browser, just not inside Claude Code's browser-automation tool, which blocks local file navigation.)
3. Republish to the *same* Claude artifact URL by passing `url: "https://claude.ai/code/artifact/3831172f-1950-4713-80f9-6eb58ad0c3d0"` to the Artifact tool — otherwise a new conversation mints a brand-new URL.
4. Push the same updated file to GitHub as `index.html` on `main` (repo: `terrillpotts/cyber-range`) so the GitHub Pages copy matches.
5. After any content edit, re-copy this file back into the durable project folder (`C:\projects\cyber projects\cyber-range\cysa-range.html`) — the scratchpad copy a session works from is temporary.

## Known gotchas (found via testing, already fixed — don't reintroduce)

- **Answer-position bias:** when authoring quiz questions, the correct option kept landing in the same slot (a real bug found on 60/64 questions — all `correct:1`). Fixed via `shuffleQuestion()`, applied in both `mountQuiz()` and `buildExam()`. If you add new questions, you don't need to manually vary option order — the shuffle handles it — but don't remove the shuffle call.
- **Sidebar height:** `.sidebar` must stay `position:sticky; top:0; height:100vh; overflow-y:auto` (desktop) or it stretches to match the tallest page content (CSS Grid default) and the profile card/XP/reset button scroll off-screen on long pages like the exam review screen.
- **`view` must be declared before any code path that can call `render()`/`renderChrome()`** — `touchStreak()` fires at load and cascades into a render; if `var view` isn't initialized before that point (order matters even though `function` declarations hoist, `var` assignments don't), it throws and the whole page renders blank with no visible error unless you check the console.

## Persistence limitation

No cross-device sync capability exists for either published copy. Progress auto-saves to `localStorage` per browser/device (and separately, since the Claude artifact and GitHub Pages copy are different origins — progress does not carry over between them). Export/Import (JSON file) is the manual workaround for moving progress between devices — mention this if it comes up again, it's a platform limitation, not a bug to fix.
