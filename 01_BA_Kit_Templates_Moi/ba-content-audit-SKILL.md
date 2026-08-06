---
# MODIFICATION NOTE
# Source file: bakit/skills/ba-content-audit/SKILL.md
# Type: MODIFY (clone)
# Target destination: skills/ba-content-audit/SKILL.md
# Changes:
#   - Added --manifest argument for Incremental Audit mode
#   - Added Step 0: Incremental Audit (git diff + manifest comparison) before existing full audit
#   - Added Layer 3 (Content Correctness) checks per audit-guideline-post-update.md
# ---

name: ba-content-audit
description: Cross-artifact format, cross-reference consistency, and content correctness audit for BA-kit projects. Supports incremental audit (post-update) and full audit modes. Read-only — never mutates artifacts.
argument-hint: "--slug <slug> [--date <date>] [--manifest <path>]"
allowed-tools:
  - Read
  - Bash
  - Glob
  - Grep
  - AskUserQuestion
---

# BA Content Audit

Read-only cross-artifact audit for BA-kit lifecycle projects. Supports two modes:

- **Incremental mode** (`--manifest <path>`): audit only files listed in a Change Manifest, comparing plan (manifest) vs reality (git diff). Run this immediately after completing a Feature Plan update cycle.
- **Full mode** (no `--manifest`): scan all artifacts (intake → backbone → modules → compiled → shared), report format compliance and cross-reference integrity. Run before handoff/packaging.

Output goes to `plans/{slug}-{date}/shared/audit-report.md`.

## Invocation

```text
/ba-content-audit --slug warehouse-rfp
/ba-content-audit --slug warehouse-rfp --date 260611-0900
/ba-content-audit --slug warehouse-rfp --manifest plans/warehouse-rfp-260611-0900/shared/manifests/CM-260611-1030-spec-v2.md
```

## Arguments

| Argument | Required | Description |
|----------|----------|-------------|
| `--slug <slug>` | Yes | Project slug (e.g., `warehouse-rfp`) |
| `--date <date>` | No | Project date token (`YYMMDD-HHmm`). If omitted: glob `plans/{slug}-*`; single match → use; multiple matches → `AskUserQuestion`; zero → error |
| `--manifest <path>` | No | Path to a Change Manifest file. Triggers Incremental Audit mode. If omitted → Full Audit mode. |

## Execution Context

Read `core/contract.yaml` for path resolution and `core/contract-behavior.md` for shared behavior rules.

Read `references/audit-rules.md` for per-artifact-type format rules, severity classification, and cross-reference resolution logic.

Read `template/audit-report-template.md` for output format.

**Manifest directory note:** `shared/manifests/` must be gitignored — manifests are local-only and must not be committed to PR or main branch. If no manifest exists at the given path, error with: "Manifest not found at {path}. Check the path or run /ba-impact first to generate a manifest."

## Process

### Step 1: Resolve Project Path

1. Parse `--slug` (mandatory), optional `--date`, and optional `--manifest` from `$ARGUMENTS`.
2. Read `core/contract.yaml` → resolve `paths.project_root` with `{slug}` and `{date}`.
3. If `--date` provided → use exact path.
4. If `--date` omitted:
   - Glob `plans/{slug}-*` for matching directories.
   - Single match → auto-select.
   - Multiple matches → `AskUserQuestion` to let user pick.
   - Zero matches → error: "No project found for slug '{slug}'".
5. Verify project root exists on disk. If not → Blocking error, stop.
6. If `--manifest` provided → verify manifest file exists. If not → error (see note above). Detect base branch for git diff (default: `main`; fallback: `master`; if neither exists: ask BA).

### Step 2: Route to Audit Mode

- **`--manifest` provided** → Incremental Audit (Step 3a), then Full Audit (Step 3b).
- **No `--manifest`** → Full Audit only (Step 3b).

---

### Step 3a: Incremental Audit (only when --manifest provided)

**Purpose:** Compare what was *planned* (manifest) vs what *actually changed* (git diff). Catch drift, missing updates, and unplanned changes.

#### 3a.1 — Parse Change Manifest

Read the manifest file. Extract:
- List of planned files to change (from `## Detailed Changes` section)
- Per-file: section, action (ADD/UPDATE/DELETE/SKIP), source ref, tag (client-spec / BA-inferred)
- Feature Plan (from `## Feature Plan` section if present): expected US/UC files to create/modify

#### 3a.2 — Get Git Diff

Đọc `baseline_sha` từ frontmatter của manifest (field `baseline_sha` — SHA commit tại thời điểm manifest được tạo, trước khi apply bất kỳ thay đổi nào). Chạy:

```bash
git diff {baseline_sha}...HEAD -- plans/{slug}-{date}/ | head -500
```

Điều này so sánh đúng: trạng thái trước khi bắt đầu update cycle (lúc manifest ra đời) vs trạng thái hiện tại — không phụ thuộc vào `main` hay bất kỳ branch nào khác.

Nếu `baseline_sha` không có trong manifest → warn BA và fallback sang `git diff HEAD~1...HEAD` (chỉ commit gần nhất). Limit output 500 lines; nếu lớn hơn thì chạy per-file với limit 100 lines mỗi file.

#### 3a.3 — Layer 1: Completeness Check

Compare manifest plan vs git diff line by line:

| Check | Pass condition | Fail signal |
|-------|---------------|-------------|
| All planned files were touched | File appears in git diff | Planned file has no diff — missing update |
| No unplanned files changed | Every diffed file is in manifest | Unplanned file changed — flag for BA review |
| ID convention correct | New IDs match `US-{module}-{NNN}`, `UC-{module}-{slug}` format | ID drift |
| Backbone reflects all changes | backbone.md diff includes relevant feature/actor/rule updates | Backbone not updated |
| Compiled artifacts refreshed | `04_compiled/` has diff if source changed | Stale compiled output |

Report format for each issue:
```
[L1-COMPLETENESS] {severity}: {description}
  File: {path}
  Expected: {what manifest said}
  Actual: {what git diff shows}
  Fix: {exact action}
```

#### 3a.4 — Layer 2: Consistency Check

For each modified file pair (US↔UC, UC↔SRS, backbone↔FRD):

**2.1 Traceability chain** — verify forward and backward:
- Backbone feature ID → FRD feature → US `source_backbone_ids` → UC `feature_ref` → SRS FR → AC

**2.2 Cross-check pairs:**
- US `story_id` appears in UC `linked_stories` (or vice versa)
- UC `usecase_id` appears in SRS section referencing it
- Sequence diagram in UC matches description in Main Flow steps

**2.3 Terminology:**
- Actor names identical across backbone, FRD, US, UC
- Feature names consistent — no drift between "đăng nhập" vs "login" vs "sign in" within same project
- Field/entity names identical across UC steps, screen descriptions, and any data references

Report each inconsistency with source file + line reference.

#### 3a.5 — Layer 3: Content Correctness

**3.1 Acceptance Criteria quality:**
- Each AC/Main Flow step must be measurable — has a concrete observable outcome
- No vague steps like "hệ thống xử lý" without specifying what the result is
- No duplicate steps across different UCs for the same action

**3.2 Business Rule consistency:**
- Rules in backbone/FRD align with what UC error flows describe
- No rule appears differently across documents (e.g., lockout after 3 attempts in backbone but 5 in UC)

**3.3 Flow integrity:**
- No contradicting flows between features (e.g., Feature A enables something Feature B explicitly disables)
- Diagrams (Mermaid) match the text description of the same flow — actor sequence and step order must match

**3.4 Edge Case coverage:**
- Each UC with external interactions (API calls, DB writes, auth) has at least: 1 auth/session EC, 1 validation EC, 1 infra/unavailable EC
- EC IDs referenced in EF sections exist in the manifest or UC file

Report each correctness issue with severity: Blocking (contradicts a stated rule), Warning (measurability gap), Info (style/completeness suggestion).

#### 3a.6 — Incremental Audit Summary

Print summary before proceeding to full audit:
```
[Incremental Audit] Plans vs Reality
  Files planned: N | Files actually changed: M
  Unplanned changes: {list or "none"}
  Missing updates: {list or "none"}

  Layer 1 (Completeness): {X Blocking, Y Warning, Z Info}
  Layer 2 (Consistency): {X Blocking, Y Warning, Z Info}
  Layer 3 (Correctness): {X Blocking, Y Warning, Z Info}

Proceeding to full artifact audit...
```

---

### Step 3b: Full Audit Workflow

Route to existing audit workflows:

**Strategy selection** (both flows, `first-audit-workflow.md` Step 0):
- ≤60 files, ≤2 modules → single-agent sequential audit
- >60 files or >2 modules → multi-agent batched: orchestrator handles system files + index, spawns sub-agent per module (fresh context each), merges results

**First audit** (`references/first-audit-workflow.md`):
- Scan artifacts in lifecycle order
- Per-file: frontmatter check, mandatory sections check, cross-reference check
- Classify findings as Blocking / Warning / Info per `references/audit-rules.md`
- Write report per `template/audit-report-template.md`

**Re-audit** (`references/re-audit-workflow.md`):
- Load existing report (frontmatter + Grep for IDs only, not full read)
- Re-scan all artifacts (batched if large project)
- Compare old vs new findings using content-hash IDs
- Flag `[NEW]`, bump severity on persistent, move resolved to "Resolved" section
- Overwrite report preserving trace

### Step 4: Output

1. Write audit report to `paths.audit_report` path (resolved from contract.yaml).
   - If Incremental Audit ran: prepend `## Incremental Audit Results` section to report before full audit findings.
2. Print compact chat summary:
   - If incremental: Incremental findings summary first, then full audit summary
   - Blocking count + top 3-5 Blocking findings
   - Warning count + top 3-5 Warning findings
   - Info count
   - Report file path
   - "Fix commands: see `## Fix Commands` in report — copy-paste per finding."

## Boundaries

- **Read-only**: Never mutate any artifact file.
- **No external calls**: No MCP, no API, no network.
- **.pen files skipped**: Pencil MCP-managed, encrypted — skip without reading.
- **Empty modules**: Skip with Info finding.
- **Manifest files**: Read manifest for plan data only. Never write to manifest.
- **Output-limiting strategy**:
  - ID collection: Write to disk index via Bash (`grep -rE > .audit-id-index.txt`), zero context load. Use `files_with_matches` for discovery. Never load full ID map into context.
  - Cross-reference check: Per-file Grep for IDs, then query on-disk index (`grep -c "ID" .audit-id-index.txt`). One line result per check.
  - Orphan detection: Bash `sort | uniq -c | awk` on disk index, zero context.
  - Frontmatter check: Read first 40 lines only (`offset=0, limit=40`) — frontmatter always at top.
  - Template section check: Read TOC of template + artifact (first 80 lines) to get `##` headings; compare headings. Grep for missing sections if discrepancy flagged.
  - Wikilink/link collection: `files_with_matches` first, then content Grep only for files with links.
  - Full content read: Only when frontmatter/Grep/TOC indicate a finding requiring content-level verification. Even then, use `offset`+`limit` on files >200 lines.
  - Files >10KB: Never read fully. Always use offset+limit (<5KB per read) or Grep.
  - Never re-read same file section. Cross-reference findings to prior reads.
  - Report writing: Append findings per file incrementally. Initialize report with placeholder, append each file's findings as discovered, prepend frontmatter + summary at end. Never accumulate all findings in memory.
  - Re-audit existing report: Read frontmatter only (first 50 lines), Grep for finding IDs. Do NOT read full report.
  - Git diff: Limit to 500 lines total; fall back to per-file diff with 100-line limit.
  - Temp index cleanup: Remove `.audit-id-index.txt` after report finalized.
- **P2 artifacts deferred**: Tool-lanes, reverse, QC, memory-deep — not audited in v1.

## References

- `references/audit-rules.md` — Per-artifact-type format rules + cross-reference resolution
- `references/first-audit-workflow.md` — First-audit detailed steps
- `references/re-audit-workflow.md` — Re-audit detailed steps
- `template/audit-report-template.md` — Report output format
- `core/contract.yaml` — Path definitions
- `core/contract-behavior.md` — Shared behavior rules
- `ba-kit-proposal/02_BA_Absorption_Filter/change-manifest-template.md` — Manifest format reference
- `C:\Users\Ha Linh\Downloads\audit-guideline-post-update.md` — 3-layer audit guideline (Completeness, Consistency, Correctness)
