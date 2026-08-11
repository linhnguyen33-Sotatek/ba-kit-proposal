
# BA Start Step - Backbone (MODIFIED)

<!--
MODIFICATION NOTE
Source file: bakit/skills/ba-start/steps/backbone.md
Type: MODIFY (clone)
Target destination: skills/ba-start/steps/backbone.md
Changes:
  - Thêm Sub-step 5.0: Common Rules Population Gate
    Sau khi write backbone, trước khi write common-rules, yêu cầu BA review và approve danh sách CR-* initial draft thay vì auto-populate.
    Lý do: Common Rules là quy tắc cross-feature — BA cần confirm để tránh CR-* bị gen sai scope (quá hẹp → thành AC của 1 feature, không phải shared rule).
    Chi tiết: xem "## Sub-step 5.0" bên dưới.
-->

This step requires:

- `core/contract.yaml`
- `core/contract-behavior.md`

## Memory Read Scope

- **Must read:** `core/contract.yaml`, `core/contract-behavior.md`, `paths.intake`
- **Must read when it exists:** `paths.plan`
- **Must read when optioning is completed:** the selected option file only
- **May read:** `paths.project_memory`, `paths.memory_index` (navigation only), `paths.memory_hot_vocabulary`, `paths.memory_hot_decisions`
- **Must NOT read:** `log.md`, `cold/`, `warm/` shards

## Governance Gate

Before mutating this artifact:
1. Always verify write authority for the target artifact and its owning memory shard.
2. For first-pass creation (when `paths.backbone` does not yet exist), skip only the impact-receipt requirement.
3. For reruns (artifact already exists): locate the active impact receipt at `paths.impact_receipt` (or the canonical receipt path for this slug/date). If no active receipt exists and `change_class` is not `wording-only`, emit `GOVERNANCE_BLOCK: impact_receipt missing or invalidated` and stop.
4. If either check fails: emit `GOVERNANCE_BLOCK: {reason}` and stop.
5. After mutation completes: offer to file the change into canonical memory using `templates/project-memory-fileback-record-template.md`.

Receipt reference: `templates/impact-receipt-template.md`

## Scope

Run Step 5 only.

## Prerequisites

- Resolve slug and date using `ba-kit resolve --slug <slug>`.
- **[CWD GATE]** After resolve, cd to the project root. All artifact paths in `contract.yaml` are relative to the project root.
- Require `paths.intake`.
- If intake is missing, print the exact missing path and stop.
- Read `paths.plan` when it exists.
- Run a narrow backbone preflight (xem file gốc `bakit/skills/ba-start/steps/backbone.md` §Prerequisites cho full gate logic).

## Output

- `paths.project_home`
- `paths.backbone`
- `paths.backbone_index`
- `paths.common_rules` — always; system-level shared rule registry
- `paths.message_list` — always; system-level shared message registry
- `paths.shared_rule_message_index` — always; compiled CR/MSG cross-reference index
- `paths.project_memory`
- `paths.design_doc` — when UI-backed scope exists
- `paths.shared_shell_contract` + `paths.shared_shell_index` — when UI-backed scope exists
- `paths.control_type_library` — when UI-backed scope exists AND Step 5.1a library gate passed

## Step 5 - Build the requirements backbone

*(Giữ nguyên logic gốc — xem `bakit/skills/ba-start/steps/backbone.md` §Step 5)*

---

## Sub-step 5.0 — Common Rules Population Gate [THÊM MỚI]

> **Vị trí trong flow:** Chạy SAU khi backbone.md đã được write, TRƯỚC khi write `paths.common_rules`.

**Vấn đề với flow hiện tại:**
Backbone step hiện tại tự động populate CR-* codes từ backbone scope mà không có BA review. Điều này dẫn đến:
- CR-* bị gen quá hẹp (chỉ apply 1 UC → thực chất là AC, không phải shared rule).
- CR-* bị gen trùng với AC đã có trong UC (vi phạm nguyên tắc §1.2.4: Common Rules ≠ AC).
- BA không có checkpoint để kiểm soát scope của CR-* trước khi module BAs bắt đầu reference.

**Flow mới:**

1. **Draft CR-* list:** Backbone phân tích scope từ intake và backbone vừa write → đề xuất danh sách CR-* draft gồm:
   - CR-VAL-* cho validation rules áp dụng ≥ 2 screens/UCs (email, password, required field chung).
   - CR-DIS-* cho display rules áp dụng ≥ 2 screens (pagination threshold, empty state, loading state).
   - CR-BEH-* cho behaviour rules áp dụng ≥ 2 screens (button disabled, form reset).

2. **Hiển thị draft cho BA review:**
   ```
   [backbone] Danh sách Common Rules đề xuất ({N} rules):

   | Code | Type | Rule Statement | Applies To (≥2 screens/UCs) | Phân loại |
   |---|---|---|---|---|
   | CR-VAL-01 | VAL | [statement] | [scope] | ✅ Shared rule |
   | CR-DIS-01 | DIS | [statement] | [scope] | ⚠️ Xem lại — chỉ thấy 1 screen |
   | CR-BEH-01 | BEH | [statement] | [scope] | ❌ Là AC — nên chuyển vào UC |

   ⚠️ Rule đánh dấu ❌ là AC (chỉ apply 1 feature) — không nên đưa vào Common Rules.
      Bỏ qua các rule này hay giữ lại?

   Approve list? (Y / chỉnh sửa / bỏ rule cụ thể):
   ```

3. **BA confirm:**
   - `Y` → write `paths.common_rules` với danh sách đã duyệt.
   - Chỉ định bỏ rule cụ thể (vd "bỏ CR-BEH-01") → loại khỏi list, write phần còn lại.
   - Chỉnh sửa free text → re-draft, show lại để confirm.

4. **Sau khi write:** Tiếp tục write `paths.message_list` và `paths.shared_rule_message_index` như flow gốc.

**Nguyên tắc phân loại CR-* vs AC:**
- **Common Rule (CR-*):** Áp dụng cho ≥ 2 screens hoặc UCs trong toàn dự án. Không phụ thuộc vào context của 1 feature cụ thể.
- **AC (trong UC):** Chỉ apply cho đúng 1 UC/feature. Nếu rule chỉ thấy ở 1 chỗ → đưa vào UC Main Flow hoặc Error Flow, không đưa vào common-rules.md.

---

*(Các step 5.1, 5.1a, 5.2 giữ nguyên logic gốc — xem `bakit/skills/ba-start/steps/backbone.md`)*
