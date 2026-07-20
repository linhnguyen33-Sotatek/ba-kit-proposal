# Xương sống yêu cầu (Requirements Backbone)

**Dự án (Project):** [Tên dự án]
**Slug:** [initiative-slug]
**Ngày (Date):** [YYMMDD-HHmm]
**Chế độ triển khai (Engagement Mode):** [lite | hybrid | formal]
**Chủ sở hữu (Owner):** [BA owner]

## Tóm tắt phạm vi đã chốt

- Vấn đề kinh doanh:
- Kết quả mong muốn:
- Ngoài phạm vi:
- Các quyết định đã chốt:

## Mục tiêu kinh doanh và chỉ số thành công (Business Goals and Success Metrics)

| Goal ID | Mục tiêu | Chỉ số / Kết quả đo lường | Chủ sở hữu |
| --- | --- | --- | --- |
| BG-01 | [Mục tiêu] | [KPI / Outcome] | [Owner] |

## Nhóm người dùng và tác nhân (Users and Actors)

| Actor ID | Vai trò | Mục tiêu chính | Nguồn (Intake Ref) | Ghi chú |
| --- | --- | --- | --- | --- |
| ACT-01 | [Vai trò] | [Mục tiêu] | STK-01 | [Ghi chú] |

## Ma trận portal và sở hữu điều hướng (Portal Matrix)

Chốt ở cấp hệ thống trước khi viết screen spec theo module. Đây là nguồn chuẩn để module chỉ snapshot lại, không tự định nghĩa lại menu toàn cục.

| Portal ID | Portal Name | Target Actor / Access Scope | Owned Screen Families / Route Groups | Default Entry Context |
| --- | --- | --- | --- | --- |
| PORTAL-CSR | [CSR Portal] | [CSR / customer support / scoped access] | [Dashboard, Tickets, Customer Detail] | [Đăng nhập xong vào Dashboard] |

## Danh sách module (Module List)

> Module = đơn vị phân hệ để phân công và tác nghiệp song song. Mỗi module có thư mục riêng trong `03_modules/` với FRD, user stories, SRS, screen riêng. Module BA chỉ làm việc trên module được giao.
>
> **Module chốt trước, Feature được xác định sau và gắn vào Module** (xem bảng Feature Map bên dưới, cột `Linked Module`) — không liệt kê ngược Feature ID ở đây để tránh phải đồng bộ 2 chiều giữa 2 bảng.

| Module ID | Tên module | Mô tả | Module BA được giao | Portal áp dụng | Trạng thái |
| --- | --- | --- | --- | --- | --- |
| MOD-01 | [Tên module] | [Mô tả ngắn] | [@BA-handle hoặc TBD] | [PORTAL-CSR] | [recommended / in-progress / completed] |

## Bản đồ tính năng và phạm vi (Feature Map and Scope)

> Feature thuộc về đúng 1 Module (cột `Linked Module`). **ID sinh ra ở đây (`F-{NN}`) là ID xuyên suốt toàn bộ tài liệu**, không đổi tên ở tầng dưới: FRD Feature List **compile/filter** lại đúng các dòng thuộc module đó (không tự đặt ID mới) → UC `feature_ref` trỏ thẳng về `F-{NN}` này.
>
> Bảng này dừng ở mức capability/feature — **không viết AC, không breakdown story chi tiết** ở đây (Story Map đã bỏ: UC = feature, story sống trong UC khi module được triển khai, tránh phải đồng bộ 2 nơi).

| Feature ID | Tính năng / Capability | Mô tả | Ưu tiên | Nguồn (Intake Ref) | In Scope | Linked Module | Ghi chú |
| --- | --- | --- | --- | --- | --- | --- | --- |
| F-01 | [Tính năng] | [Mô tả] | [Must / Should / Could] | REQ-01 | [Yes / No] | MOD-01 | [Ghi chú] |

## Backbone yêu cầu phi chức năng (Non-Functional Backbone)

| NFR ID | Danh mục | Yêu cầu | Nguồn (Intake Ref) | Trigger / Gate |
| --- | --- | --- | --- | --- |
| NFR-01 | [Performance / Security / Availability] | [Yêu cầu] | CON-01 | [Khi nào cần đặc tả chi tiết] |

## UI và màn hình cần tài liệu (UI and Screen Coverage)

| Screen ID | Portal ID | Màn hình / Luồng | Mức độ phức tạp | Nguồn (Intake Ref) | Cần wireframe | Ghi chú |
| --- | --- | --- | --- | --- | --- | --- |
| SCR-01 | [PORTAL-CSR] | [Màn hình] | [Low / Medium / High] | SCRSRC-01 | [Yes / No / Critical-only] | [Ghi chú] |

## Hướng thiết kế UI cần chốt trước wireframe (UI Design Direction Before Wireframes)

- Có cần `designs/{slug}/DESIGN.md`: [Yes / No]
- Hướng tham chiếu ban đầu: [Tham chiếu DESIGN.md / brand / custom brief]
- Quyết định người dùng cần chốt trước Step 9: [Màu sắc, typography, density, component feel, responsive priority, navigation schema theo portal, active-menu rule, breadcrumb/back behavior, anti-patterns]

## Điều kiện tiến hành từng tài liệu

- FRD: [Required / Optional] + lý do
- User stories: [Required / Optional] + lý do
- SRS: [Required / Optional] + lý do
- Wireframes: [Required / Optional] + lý do
- Package HTML: [Required / Optional] + lý do

## Assumptions, Risks, Open Questions

### Assumptions

- 

### Risks

- 

### Open Questions

1. 

## Next Step

- [Bước tiếp theo được khuyến nghị theo mode]
