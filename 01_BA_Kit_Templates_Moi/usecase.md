# BA Start Step - Use Case (ADD NEW)

<!--
MODIFICATION NOTE
Source file: N/A — file mới hoàn toàn
Type: ADD NEW
Target destination: skills/ba-start/steps/usecase.md
Reference: skills/ba-start/steps/stories.md (pattern), ba-kit-proposal/01_BA_Kit_Templates_Moi/usecase-item-template.md, ba-kit-proposal/02_BA_Absorption_Filter/seq-to-ec-template.md
-->

This step requires:

- `core/contract.yaml`
- `core/contract-behavior.md`

## Memory Read Scope

- **Must read:** `core/contract.yaml`, `core/contract-behavior.md`, `paths.backbone_index`
- **Must read:** `paths.userstories_index` — để lấy context từ US đã gen ở step trước
- **May read:** targeted `paths.backbone` sections, `paths.frd` (when exists), `paths.project_memory`
- **Must NOT read:** `log.md`, `cold/`, `warm/` shards, unrelated module shards

## Backbone Authority Gate

Before writing any UC file, validate backbone alignment:
- UC actor, goal, and feature must trace to backbone module scope.
- If alignment fails: emit `BACKBONE_ALIGNMENT_FAIL: usecase_scope` and stop.
- Recovery: run `ba-start impact --slug <slug>` or refresh backbone, then rerun.

## Governance Gate

Before mutating this artifact:
1. **Skip this gate for first-pass creation** (when `paths.usecases_root` does not yet exist).
2. For reruns (artifacts already exist): verify write authority and locate the active impact receipt at `paths.impact_receipt`. If no active receipt exists and `change_class` is not `wording-only`, emit `GOVERNANCE_BLOCK: impact_receipt missing or invalidated` and stop.
3. After mutation completes: offer to file the change into canonical memory using `templates/project-memory-fileback-record-template.md`.

## Scope

Run usecase step only. Produce one UC file per feature intent.

## Prerequisites

- Resolve slug, date, and module using `ba-kit resolve --slug <slug> [--module <module>]`.
- Require `paths.backbone` and `paths.userstories_index`.
- If User Story index is missing → stop and ask to run stories step first.
- Run a narrow usecase preflight:
  - If `ba-kit guardrail --command usecase --slug <slug> --date <date> --module <module>` returns `status=block`, surface the block message and stop.
  - read `paths.backbone_index` first
  - read targeted backbone sections relevant to this feature
  - read the specific US file(s) linked to this UC for context
  - do not scan unrelated module folders

## Output

- `paths.usecases_index` — folder index navigator
- `paths.usecase_item` — one file per UC (`usecases/uc-{slug}.md`)

---

## Step UC — Produce Use Cases

Generate detailed Use Cases from the backbone feature map, user stories, and FRD.

Use templates:
- `01_BA_Kit_Templates_Moi/usecase-item-template.md` (or `templates/usecase-item-template.md` when installed) for each UC file
- `templates/usecases-index-template.md` for the index (create if not exists)

### Sub-step UC-1 — Validate Input

Trước khi draft bất kỳ UC nào, kiểm tra đã có đủ thông tin để điền đầy đủ template chưa.

**Checklist thông tin cần thiết:**

| Yếu tố | Câu hỏi kiểm tra | Nguồn ưu tiên |
|--------|-----------------|---------------|
| Actor chính | Ai là người trigger UC này? (cụ thể, không chung chung) | Backbone actor registry |
| Actor phụ | Có hệ thống/actor nào khác tham gia không? | FRD integration points |
| Preconditions | Điều kiện nào phải đúng trước khi UC bắt đầu? | Backbone / US ACs |
| Trigger | Sự kiện gì khởi động UC này? (user action / system event / scheduled) | US story statement |
| Main Flow | Đủ bước để mô tả toàn bộ tương tác actor↔system từ trigger đến postcondition | FRD workflows + US ACs |
| Alternate Flows | Có ít nhất 1 alternative path không? | FRD business rules |
| Error/Exception Flows | Điều gì có thể sai? Hệ thống phản ứng thế nào? | FRD business rules |
| Postconditions | Definition of Done — state sau khi UC thành công là gì? | US benefit |

Với mỗi yếu tố còn thiếu → hỏi BA phần đó. Không đoán, không tự suy diễn từ tên feature.

Hỏi từng UC một. Format câu hỏi:
```
Để gen UC [{uc-slug}], em cần thêm thông tin:
1. {câu hỏi cụ thể về yếu tố thiếu}
2. {câu hỏi cụ thể...}
```

Chỉ chuyển sang Sub-step UC-2 khi đã đủ toàn bộ thông tin.

### Sub-step UC-2 — Draft Sequence + Edge Cases

Khi đã đủ thông tin:

**Bước 2a — Vẽ Mermaid Sequence Diagram:**

Vẽ main flow theo format trong `seq-to-ec-template.md §1`:
- Main flow: tất cả bước tương tác actor↔system theo thứ tự
- Alternate flows: các nhánh rẽ chính (không cần liệt kê hết mọi error ở đây)

**Bước 2b — Break Point Analysis:**

Theo format `seq-to-ec-template.md §2`, phân tích mỗi interaction line trong diagram:
- Có thể gãy ở đâu?
- Root cause category là gì?
- User-facing error message là gì?
- Severity?

**Bước 2c — Group thành Edge Cases:**

Nhóm break points có cùng root cause + cùng error message → 1 EC.
Đặt tên theo format `EC-{MODULE}-{NNN}`.

**Bước 2d — Show draft cho BA:**

```
[Draft] UC: {uc-slug}

Story Statement:
  As a {actor}, I want to {goal}, so that {benefit}.

Main Flow ({N} bước):
  1. {step}
  2. {step}
  ...

Alternate Flows: {N} flow(s)
  AF-01: {tên}

Error Flows: {N} flow(s)
  EF-01: {tên}

Edge Cases phát hiện: {N}
  {EC-ID}: {tên} — {severity}
  ...

Sequence Diagram:
[mermaid block]

Confirm draft? (Y / sửa: {mô tả thay đổi})
```

### Sub-step UC-3 — Confirm Loop

Nếu BA phản hồi `sửa: ...` → cập nhật draft theo feedback và show lại. Không giới hạn số vòng — loop cho đến khi BA confirm Y.

### Sub-step UC-4 — Write + Self-Validate

Sau khi BA confirm Y:

**Bước 4a — Write file:**

Write `usecases/uc-{slug}.md` theo `usecase-item-template.md`. Điền đầy đủ tất cả sections:
- Frontmatter: `usecase_id`, `slug`, `actor`, `priority`, `feature_ref`, `source_backbone_ids`, `linked_screens` (để trống nếu chưa có), `created`, `owner`, `changelog`
- Story Statement
- Actors (primary + secondary)
- Preconditions
- Trigger
- Main Flow (các bước = ACs trực tiếp)
- Alternate Flows (AF-NN)
- Error/Exception Flows (EF-NN) — mỗi EF reference EC tương ứng
- Postconditions
- Cross-Function Impact (Within Module + Across Modules)
- Diagram (Mermaid sequence diagram đã confirm)
- Open Questions (nếu còn)

**Bước 4b — Self-validate:**

Ngay sau khi write, tự kiểm tra:

```
[Self-validate] uc-{slug}:
✅ / ❌ Frontmatter đầy đủ (usecase_id, actor, feature_ref, source_backbone_ids)
✅ / ❌ Story Statement có đủ As a / I want / so that
✅ / ❌ Main Flow đủ bước để mô tả toàn bộ tương tác từ trigger đến postcondition, actor và system action phân biệt rõ
✅ / ❌ Ít nhất 1 AF và 1 EF
✅ / ❌ Postconditions định nghĩa DoD rõ ràng
✅ / ❌ Cross-Function Impact đã điền (hoặc ghi "None" có chủ ý)
✅ / ❌ Sequence diagram có đủ actor + system participants
✅ / ❌ Edge Cases từ Break Point Analysis đã map vào EF sections

Kết quả: PASS / WARN [{list section có vấn đề}] / FAIL [{lý do}]
```

- **PASS**: tiếp tục.
- **WARN**: báo BA, hỏi có muốn fix ngay không — nếu Y thì fix, nếu N thì note vào Open Questions.
- **FAIL**: dừng, fix trước khi tiếp tục.

**Bước 4c — Update index:**

Sau khi write UC file, cập nhật hoặc tạo `paths.usecases_index`. Chạy:

```bash
ba-kit validate-index --index-key usecases_index --slug <slug> --date <date> --module <module> --writeback
```

- PASS hoặc WARN: tiếp tục.
- FAIL: dừng, fix index.

### Sub-step UC-5 — Handoff Prompt

Sau khi UC file đã write và validate xong:

**Nếu còn features trong Feature Plan:**
```
✅ Use Case [{uc-id}] đã hoàn thành: usecases/uc-{slug}.md

Feature [{current}] xong. Tiếp tục với Feature [{next}]?
Lệnh tiếp theo: /ba-start stories --slug {slug} --module {module}
(Y/n)
```

**Nếu đây là feature cuối cùng:**
```
✅ Use Case [{uc-id}] đã hoàn thành: usecases/uc-{slug}.md

Tất cả {N} features trong Feature Plan đã hoàn thành.

BA có muốn chạy audit để review toàn bộ thay đổi không?
Lệnh: /ba-content-audit --slug {slug} --manifest plans/{slug}-{date}/shared/manifests/CM-{date}-{slug}.md
(Y/n)
```

Nếu Y → in lệnh và dừng. Không tự động gọi audit step.

**[CWD RESTORE — MANDATORY]** After usecase generation completes, restore shell CWD to project root. Run `ba-kit resolve --slug <slug> --date <date> [--module <module>]` and `cd` to the `Project root:` path. Do NOT leave the shell stranded in `usecases/`.

---

## Generation Rules

- Derive one UC per coherent actor↔system interaction goal.
- UC actor must trace to backbone actor registry.
- UC `feature_ref` must map to a backbone Feature ID (`F-{NN}`).
- Main Flow steps serve as Acceptance Criteria — do not create a separate AC checklist.
- Edge Cases derived from `seq-to-ec-template.md` break point analysis must map into EF sections.
- `linked_screens` populated by SRS step — leave empty here.
- No bundled behaviors in a single UC — one interaction goal per UC.
