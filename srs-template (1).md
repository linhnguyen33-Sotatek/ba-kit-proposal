# Đặc tả yêu cầu phần mềm (Software Requirements Specification)

> **Tài liệu tổng hợp:** Bản SRS này được compile tự động từ các file nguồn chuẩn. Không chỉnh sửa trực tiếp. Nguồn: `srs/spec.md`, `usecases/uc-*.md`, `ascii-screen/*.md`, `srs/flows.md`, `srs/states.md`, `srs/erd.md`.
> Đây là **file tổng hợp** — được phép restate đầy đủ nội dung (không chỉ reference) vì mục đích của file này là điểm đọc duy nhất cho dev/QC.

## Tóm tắt dành cho BA và stakeholder

**Dự án (Project):** [Tên dự án]
**Module:** [Tên module]
**Ngày compile (Compiled):** [YYYY-MM-DD]
**Tham chiếu FRD:** [link tới `frd/{module_slug}/frd.md`] — đọc để nắm bối cảnh nghiệp vụ, không restate lại Overview ở đây.

## Mục đích và phạm vi (Purpose and Scope)

[Mục đích và phạm vi của module — điền từ backbone]

## Yêu cầu chức năng & Use Cases (Functional Requirements)

> Compiled từ `usecases/uc-*.md` + `userstories/us-*.md`. Chỉ **1 bảng duy nhất** — không tách riêng "Functional Requirements" vs "Use Case Specifications" vì cả hai vốn cùng trỏ tới cùng 1 UC. Requirement code = **UC ID**, không dùng `FR-id` riêng. Cột `US Ref` trỏ tới file User Story (Story Statement gốc + Acceptance Criteria checklist rút gọn) — dùng cho QC/PO/PM confirm pass/fail nhanh mà không cần đọc full flow.

| Mã UC | Feature Ref (FRD) | US Ref | Tên UC | Story Statement | Tác nhân chính | Trigger | Điều kiện tiên quyết | Hậu điều kiện | Ưu tiên | Trạng thái |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| UC-{module}-{slug} | F-01 | US-{module}-{NNN} | [Tên] | As a {actor}, I want {goal}, so that {value} | [Tác nhân] | [Trigger] | [ĐK] | [HK] | [P0/P1/P2] | [draft] |

> Main/Alternate/Error Flow, Diagram, và Cross-Function Impact chi tiết của từng UC: xem file gốc `usecases/uc-{slug}.md` (không copy toàn bộ flow vào bảng compiled này để tránh file quá dài — link trực tiếp tới file UC).

## Mô tả màn hình (Screen Descriptions)

Mô tả chi tiết từng màn hình được compile từ `ascii-screen/*.md`. Mỗi màn hình bao gồm: fields, actions, control states, message placement, và ASCII wireframe.

## Yêu cầu phi chức năng (Non-Functional Requirements)

| Mã (ID) | Danh mục (Category) | Yêu cầu (Requirement) | Mục tiêu (Target) |
| --- | --- | --- | --- |
| NFR-{module}-001 | [Hiệu năng/Bảo mật/Usability] | [Yêu cầu] | [Mục tiêu] |

## Tham chiếu quy tắc/thông điệp dùng chung

Các rule và message dùng chung được compile từ backbone. Module artifact tham chiếu bằng code (`CR-*`, `MSG-*`).

### Common Rules (từ `02_backbone/common-rules.md`)

| Code | Type | Rule Statement | Applies To | Edge Cases |
| --- | --- | --- | --- | --- |
| CR-VAL-01 | VAL | [Quy tắc] | [Phạm vi] | [Edge case] |

### Message List (từ `02_backbone/message-list.md`)

| Code | Type | Canonical Text | Surface | Applies To |
| --- | --- | --- | --- | --- |
| MSG-ERR-01 | ERR | [Nội dung] | [inline/toast/banner] | [Phạm vi] |

### Control Type Library (từ `02_backbone/control-type-library.md`)

Các control type dùng trong module. Xem file đầy đủ để biết default behaviour và edge cases.

| Control Type | Mô tả | Interactive |
| --- | --- | --- |
| `text_input` | Text Input | Yes |
| `button` | Button (primary/secondary/danger/ghost/icon) | Yes |

## Sơ đồ luồng dữ liệu (Data Flow Diagrams)

> **Scope:** Luồng DỮ LIỆU giữa các hệ thống/module, không mô tả thứ tự hành động hay actor (khác với FRD Workflows và UC Sequence Diagram — xem quy ước 3 tầng trong `frd-template.md` và `usecase-item-template.md`).

Được compile từ `srs/flows.md`.

## Sơ đồ thực thể quan hệ (Entity Relationship Diagram)

Được compile từ `srs/erd.md`.
