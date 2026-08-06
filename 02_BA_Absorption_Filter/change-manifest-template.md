# Change Manifest Template

> Copy template này khi tạo Change Manifest mới.
> Lưu tại: `plans/{slug}-{date}/shared/manifests/CM-{YYMMDD}-{HHMM}-{source-slug}.md`

---

```markdown
---
type: change-manifest
status: pending-review
created: YYYY-MM-DD
author: "@hien.duong"
source: "[tên file spec / OQ-ID / BA instruction]"
trigger: "[absorb | impact | manual]"
baseline_sha: "[git commit SHA tại thời điểm manifest được tạo — dùng cho incremental audit]"
---

# Change Manifest — [Source Name] → [Target Module(s)]

## Meta

| Field | Value |
|---|---|
| Date | YYYY-MM-DD HH:MM |
| Source | [tên đầy đủ file spec / OQ resolution / BA instruction] |
| Trigger | [absorb \| impact \| manual] |
| Baseline SHA | [git commit SHA — `git rev-parse HEAD` tại lúc tạo manifest] |
| Total files affected | N |
| New UCs | UC-XXX, UC-YYY |
| Updated UCs | UC-ZZZ |
| Indirectly affected UCs | UC-AAA |

## Scope Summary

### ✅ In BA Scope
1. [Mô tả item 1]
2. [Mô tả item 2]

### ❌ Out of BA Scope (Skip)
- [Item skip 1 — dev tự define]
- [Item skip 2 — dev tự define]

---

## Detailed Changes

### File: [relative path to file]

| Section | Action | Before | After | Source | Tag |
|---|---|---|---|---|---|
| §X [section name] | ADD / UPDATE / DELETE / SKIP | [old content or —] | [new content or —] | [spec ref] | client-spec / BA-inferred |

> **Action legend:**
> - `ADD` — thêm mới (row, section, rule)
> - `UPDATE` — sửa nội dung existing
> - `DELETE` — xóa
> - `SKIP` — có trong spec nhưng ngoài BA scope, ghi `—`
>
> **Tag legend:**
> - `client-spec` — nội dung đến từ spec khách
> - `BA-inferred` — BA/AI suy luận (CHỈ hiện ở manifest, KHÔNG vào docs)

### File: [next file...]

| Section | Action | Before | After | Source | Tag |
|---|---|---|---|---|---|
| ... | ... | ... | ... | ... | ... |

---

## Edge Cases (nếu có UC mới/update)

| EC-ID | Name | Flow Ref | User Sees | System Does | Recovery |
|---|---|---|---|---|---|
| EC-{MOD}-{NNN} | [tên EC] | [UC-ID §section] | [error message / UI state] | [system action] | [recovery path] |

---

## Cross-Function Impact

| Downstream UC | Module | Data / State Affected | Impact Description |
|---|---|---|---|
| [UC-ID] | [module] | [data/state items] | [mô tả ảnh hưởng] |

---

## Review Decision

> BA Lead fill phần này sau khi review:

| Decision | Notes |
|---|---|
| [ ] Approved — proceed with all changes | |
| [ ] Approved with modifications — see notes | |
| [ ] Rejected — reason: | |

**Reviewer**: _______________
**Date**: _______________
```
