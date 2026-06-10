# Snyk → GitHub Advanced Security (Azure DevOps) — Discovery Tracker

Working doc for the GHAzDO evaluation. Each row maps back to the customer's question list so we
can track answers, open items, and who owns the follow-up. Update the **Status** and **Owner**
columns as we go.

- **Account:** _TODO_
- **Last updated:** 2026-06-09
- **Sources:** Microsoft Learn + GitHub docs (current through ADO Sprint 272 / 2026). Re-confirm
  anything marked *preview* before the next call.

**Status key:** ✅ Confirmed · 🟡 Confirmed, caveat · 🔴 Gap vs. Snyk · ⏳ Follow-up needed

---

## Quick status board

| # | Topic | Answer in short | Status | Owner |
|---|-------|-----------------|--------|-------|
| 1 | PR block on new High/Critical | Yes — `NewHighAndCritical` status check | 🟡 | |
| 2 | Pipeline/CI gating | Yes — fail build on threshold | ✅ | |
| 3 | Central pipeline templates | Yes — shared YAML template | ✅ | |
| 4 | Baselining migration | Scan default branch first, gate deltas only | ✅ | |
| 5 | Code scanning: default vs advanced | Advanced is the go-live path today | 🟡 | |
| 6 | Query suites | default / security-extended / security-and-quality | ✅ | |
| 7 | Dependency coverage + cross-branch | Cross-branch dismiss is automatic | 🟡 | |
| 8 | Org-wide CVE suppression | No one-click; API only | 🔴 | |
| 9 | License governance | Not supported | 🔴 | |
| 10 | Risk prioritization (reachability/fixability) | Severity only | 🔴 | |
| 11 | Zero-day exposure across repos | Security Overview + API/service hooks | 🟡 | |
| 12 | Secret + push protection independence | Yes, per repo/project/org | ✅ | |
| 13 | Self-hosted agent prereqs | CodeQL bundle, build runtimes, allowlist | ✅ | |
| 14 | Centralized reporting | Security Overview (+ Defender for Cloud) | ✅ | |
| 15 | JIT reporting access for leadership | No native JIT; group/RBAC scoped | 🟡 | |
| 16 | Permissions model | Contributor / Project Admin / Manage settings | ✅ | |
| 17 | Audit capabilities | Audit log w/ enablement + bypass events | 🟡 | |

---

## PR gating & policy

**Block PRs on new High/Critical only?** Yes. Two built-in status checks, applied as branch
policies:

- `AdvancedSecurity/NewHighAndCritical` — blocks only on *newly introduced* High/Critical. This is
  the direct equivalent of their current Snyk gate.
- `AdvancedSecurity/AllHighAndCritical` — blocks on *all* High/Critical, existing included.

Both look at dependency, code, and secret findings together.

Recommended approach today: `NewHighAndCritical` as a required branch policy. It's GA and stable.

Watch-outs to raise on the call:

- The new-only check needs a **build validation policy with the AS tasks scanning the PR branch**.
- PR annotations only work once the **default and target branches have been scanned** (you need a
  baseline before you can diff).
- The central alerts hub does **not** show PR-branch alerts — those live as inline PR annotations.

**What it takes to turn on annotations + enforcement:**

1. Pipeline includes the dependency and/or CodeQL tasks.
2. Build validation policy on the protected branch points at that pipeline.
3. Default + target branches already scanned once.
4. Add the AS status check as a branch policy (annotations inform; the status check is what blocks).

---

## Pipeline gating & centralized CI control

- **Beyond PR checks:** the CodeQL and dependency tasks run inside Azure Pipelines, so we can gate
  in CI directly — not only at the PR.
- **Fail the build on a threshold:** yes, same idea as their Snyk pipeline gate. The tasks can break
  the build on High/Critical.
- **Standardize across repos:** build one central YAML template (in a shared repo) that wraps
  init → build → analyze + dependency scan, and have each repo `extends` it. One place to maintain,
  consistent gating everywhere.

---

## Migration & baselining

- Scan the **default branch first** to set the baseline, then triage that backlog.
- Because `NewHighAndCritical` only gates the delta, existing debt won't block day-one PRs while new
  issues still get caught.
- Noise control: gate on new-only, dismiss baseline items with a reason (carries across branches),
  start on a lighter query suite, run full scans off the PR path. AS also auto-hides alerts from
  pipeline configs that haven't run in 90+ days.

---

## Code scanning setup

**Default vs advanced:**

- *Default setup* — turn on in settings, no pipeline edits, auto-detects languages, runs on a
  schedule, self-updates. Shared agent pool/schedule set at org level.
- *Advanced setup* — you add the CodeQL init/autobuild/analyze tasks to the pipeline. Full control:
  build modes, custom queries, compiled-language builds, PR-branch scanning.

> Flag for the call: **CodeQL default setup is still public preview and its rollout was paused
> (noted May 1, 2026).** For a production go-live now, **advanced (pipeline) setup is the GA path.**
> Re-check the preview status before we commit to anything.

**Query suites:**

- `default` — tightest, lowest noise.
- `security-extended` — more coverage, a few more false positives. Note GHAzDO advanced setup
  defaults to this one (GitHub.com defaults to `default`).
- `security-and-quality` — adds maintainability/quality rules; most findings, most noise.

Start on default/security-extended for clean gating; use security-and-quality for deep audits.

**When to go default → advanced:** compiled languages with custom builds, custom queries, monorepos,
PR-branch scanning, or when you need control over the agent pool and performance.

**Centralizing advanced config:** shared CodeQL config file + central pipeline template referenced by
every repo. Default setup already centralizes pool/schedule at the org level.

---

## Dependency scanning

- **Coverage:** runs wherever the task runs — default branch (one-click or task), PRs (via build
  validation), feature branches (if the pipeline runs there). PR diffing needs the default/target
  baseline first.
- **Ignore in one branch → other branches?** Yes, automatically. Dismissing an alert dismisses it
  across **all branches** in that repo; matching vulns elsewhere close too. Reversible.
- **Org-wide CVE suppression in one click?** No. Dismissal spans branches *within a repo*, not across
  repos or the whole org. Org-wide would need scripting against the Alerts API. **Gap vs. Snyk.**
- **License governance?** Not supported — dependency scanning is vulnerability-only. No OSS license
  policy/enforcement in the ADO product. **Gap vs. Snyk — call this out directly.**
- **Zero-day exposure across repos:** advisories come from the GitHub Advisory Database and raise
  alerts on rescan. Find org-wide impact through Security Overview (Risk tab, filter by tool/project/
  time), the Alerts API, and service hooks for new-alert events. No separate "bulletin" push.
- **Risk prioritization:** severity (CVSS), CVE/advisory metadata, and fix-version guidance are
  there. **Reachability analysis and a dedicated fixability/exploit score are not.** Prioritization is
  severity-driven. **Gap vs. Snyk.**

---

## Secrets scanning

- Secret scanning (repo scanning) and **push protection** are separate toggles, settable at repo,
  project, or org level — so yes, they can be configured independently.
- In the standalone licensing model, **GitHub Secret Protection** is its own SKU (push protection +
  secret alerts + Security Overview), separable from Code Security.

---

## Platform requirements (self-hosted agents)

- CodeQL bundle in the agent tool cache — either `enableAutomaticCodeQLInstall: true` on the CodeQL
  init task / the auto-install checkbox in classic, or install it manually.
- Build toolchains/runtimes for compiled languages (e.g., .NET, JDK) on the agent — CodeQL builds the
  code.
- Allowlist the governance URLs so dependency scanning can pull advisory data (e.g.
  `https://governance.dev.azure.com`, plus the advsec/vssps endpoints).
- Outbound internet for the CodeQL download if using auto-install.
- Supported repos: Azure Repos Git only.

---

## Visibility & reporting

- **Centralized view:** Security Overview — org-wide Risk and Coverage, new/fixed/dismissed columns,
  filter by project/tool/time, shareable filtered URLs, per-tool enablement across all repos.
- **Coverage / adoption / remediation:** Security Overview coverage + risk views, Alerts API for
  custom dashboards, service hooks to stream alert events out.
- **Defender for Cloud:** connect the ADO org for centralized posture management; AS findings show in
  the Defender portal and it also drives PR annotations. Good fit if the security team wants a
  multicloud posture view alongside ADO.
- **JIT access for leadership:** no purpose-built just-in-time reporting. Scope read access with ADO
  security groups or Defender for Cloud RBAC; true time-boxed JIT would lean on Entra PIM around those
  roles. Flag if hard JIT is a real requirement.

---

## Governance & operations

**Permissions (manage/dismiss alerts):**

- View repo alert summary → Contributor.
- Dismiss alerts → Project Administrator.
- Manage AS settings/permissions → Project Collection Admins or `Advanced Security: manage settings`.

Suggested split: devs = Contributor (view + triage in PR), AppSec = Project Admin (dismiss), platform/
security leads = manage settings.

**Audit:** the ADO audit log records AS enablement/disablement at repo/project/org level, CodeQL and
dependency default-setup changes, push-protection changes, and **push-protection bypass events** —
with actor + timestamp, streamable.
Caveats: auditing is public preview, needs an Entra ID-backed org, and default retention is 90 days
(stream/back up for longer).

---

## Demo plan (end-to-end)

1. Commit to a feature branch → pipeline runs CodeQL + dependency scan.
2. Open PR → inline annotations on new findings; `NewHighAndCritical` shows blocked.
3. Dev clicks "Show more details" → alert detail → remediation/fix guidance.
4. AppSec view: Advanced Security tab → triage, dismiss-with-reason (show cross-branch dismissal),
   link to a work item.
5. Pipeline gating: show a CI build failing on a High/Critical threshold, separate from the PR.
6. Push fix → rescan → check passes → merge.
7. Reporting: per-repo Advanced Security tab + org-wide Security Overview, with a shared filtered view.

---

## Fit against their five goals

| Their goal | GHAzDO | Notes |
|------------|--------|-------|
| PR gating on new High/Critical | Yes | `NewHighAndCritical` status check |
| Clean baselining | Yes | default-branch baseline, delta-only gating |
| Low developer friction | Yes | inline PR annotations + fix guidance |
| Centralized pipeline templates | Yes | shared YAML template + central CodeQL config |
| Centralized visibility/reporting | Yes | Security Overview (+ Defender for Cloud) |

## Open gaps to set expectations on (vs. Snyk)

- OSS **license governance** — not available.
- **Reachability** analysis — not available.
- Dedicated **fixability / exploit** prioritization — severity only.
- **Org-wide CVE suppression** — no one-click; API-scriptable.
- **CodeQL default setup** — preview/paused; go live on advanced pipeline setup.

---

## Follow-ups / parking lot

| Item | Owner | Due | Notes |
|------|-------|-----|-------|
| Confirm CodeQL default-setup preview status before rollout | | | Was paused 5/1/26 |
| Decide gate: NewHighAndCritical vs AllHighAndCritical | | | |
| Scope org-wide CVE suppression script (Alerts API) | | | Covers Snyk gap |
| Confirm whether license governance is a hard requirement | | | Snyk gap |
| Confirm JIT reporting requirement for leadership | | | |
| List self-hosted agent pools + allowlist owner | | | |
