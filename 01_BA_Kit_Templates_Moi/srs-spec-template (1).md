---
type: srs-spec
module: "{module_slug}"
frd_ref: "frd/{module_slug}/frd.md"
fr_note: "Requirement code = UC ID, không dùng FR-id riêng"
created: "{YYYY-MM-DD}"
owner: "{@handle}"
source_backbone_ids: []
changelog:
  - {YYYY-MM-DD} | /srs | initial draft
---

# SRS Spec — {module_slug}

> Source file. Not a compiled deliverable. Edit here; compile to `srs.md` via `ba-start srs`.
> Bối cảnh nghiệp vụ: xem `frd_ref` ở trên — không diễn giải lại business context ở đây.

## Functional Requirements

> **Không có bảng riêng ở đây.** Danh sách tính năng + mapping tới UC đã có ở FRD §Feature List (`feature_ref`) và ở `usecase-item.md` frontmatter. Requirement code dùng để trace test case/code = **UC ID** trực tiếp (vd: `UC-checkout-apply-voucher`) — không tạo thêm `FR-id` riêng vì nó chỉ trỏ tới đúng 1 UC, gây trùng 2 tầng ID cho cùng 1 đối tượng.

## Non-Functional Requirements

> Nhà duy nhất của yêu cầu phi chức năng (số liệu, ngưỡng). FRD chỉ trỏ về đây, không lặp số liệu.

| ID | Category | Description | Threshold | Trace |
|---|---|---|---|---|
| NFR-{module}-001 | {performance/security/usability} | {requirement} | {threshold} | {backbone-id} |

## Business Rules

> Bảng scope/applies-to, không restate rule statement (rule text gốc chỉ tồn tại ở backbone).

| Rule Code | Applies To (screens/UCs) | Trace |
|---|---|---|
| CR-VAL-01 | {screens/UCs} | `02_backbone/common-rules.md#CR-VAL-01` |

## API / Integration Constraints

> Nhà duy nhất của chi tiết kỹ thuật tích hợp (protocol, payload, dependency kỹ thuật). FRD Integration Points chỉ nêu mục đích nghiệp vụ và trỏ về đây.

| Integration | Direction | Description |
|---|---|---|
| {service} | inbound/outbound | {constraint} |

## Data Constraints

> Nhà duy nhất của đặc tả dữ liệu (field-level). FRD không còn mục Data Requirements — mọi yêu cầu dữ liệu, kể cả ở mức khái niệm, khai báo tại đây.

| Entity | Field | Type | Constraint | Trace (ERD) |
|---|---|---|---|---|
| {entity} | {field} | {type/length} | {validation rule} | `srs/erd.md#{entity}` |

## Common Rules Reference

See `02_backbone/common-rules.md` for shared rule codes.

## Message List Reference

See `02_backbone/message-list.md` for shared message codes.
