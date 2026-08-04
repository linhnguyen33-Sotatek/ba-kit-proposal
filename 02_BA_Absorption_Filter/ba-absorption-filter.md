# BA Absorption Filter — Ruleset

> **Purpose**: Bộ quy tắc trung gian giữa AI và BA docs. AI PHẢI đọc file này trước khi absorb bất kỳ spec nào từ khách.
> **Version**: 2.1.0
> **Created**: 2026-07-17
> **Owner**: @linh.nguyen

---

## Lớp 1 — Scope Boundary (Giới hạn của AI)

### AI CHỈ ĐƯỢC cập nhật nội dung dựa trên:

1. **Client spec** — tài liệu chính thức từ khách (ví dụ: `admin_ui_spec_*.md`, `ib_spec_*.md`, `pf_spec_*.md`, `auth_spec_*.md`, `WL_SPEC_*.md`)
2. **BA define** — yêu cầu do BA tự define qua manual commit hoặc impact analysis (changelog tag: `manual`, `/ba-impact`, `/ba-do`)
3. **OQ confirmed** — thông tin đã được xác nhận qua Open Questions (OQ-*-NNN status = resolved)

### AI KHÔNG ĐƯỢC:

- Tự suy luận technical implementation detail ngoài những gì source cung cấp
- Bịa database schema, API endpoint, column type khi client spec không mention
- Tự thêm columns, data types, constraints, indexes khi không có source
- Tự fetch thông tin từ codebase để điền vào BA docs (trừ khi BA yêu cầu reverse mode)
- Giả định behavior kỹ thuật mà không có căn cứ từ spec

### Ngoại lệ:

- **Business Rule logic**: BA có quyền define BR mới dựa trên phân tích luồng (Lớp 4), nhưng phải ghi rõ source trong changelog
- **UI behavior**: BA có quyền define UI behavior khi spec khách đã define screen nhưng chưa define interaction detail
- **Edge Case**: BA có quyền define EC từ sequence analysis (Lớp 4 Step 2)

---

## Lớp 2 — Technical Content Policy (Chính sách nội dung kỹ thuật)

### Nguyên tắc chung

Khi client spec **CÓ** nội dung kỹ thuật → ghi đúng như spec, kèm source traceability.
Khi client spec **KHÔNG CÓ** → ghi `—` (dash). Dev tự define. KHÔNG bịa, KHÔNG đính chú `[TBD]`.

### Bảng policy chi tiết

| Loại nội dung | Client spec CÓ | Client spec KHÔNG CÓ |
|---|---|---|
| D1/Database table schema | ✅ Ghi đúng như spec | `—` (dev tự define) |
| API endpoint path | ✅ Ghi đúng | `—` |
| Column name + type | ✅ Ghi đúng | `—` |
| Constraint / Index | ✅ Ghi đúng | BA define được — ghi source trong changelog |
| Error code (MSG-ERR-*) | ✅ Ghi đúng | BA define được — ghi source trong changelog |
| HTTP status code | ✅ Ghi đúng | `—` |
| Cron schedule / interval | ✅ Ghi đúng | `—` |
| Business Rule logic (BR-*) | ✅ Ghi đúng + source tag | BA define được — ghi source trong changelog |
| UI behavior / interaction | ✅ Ghi đúng | BA define được — ghi source trong changelog |
| Sequence / process flow | ✅ Ghi đúng | BA define theo Lớp 4 flow |
| Validation rule (VR-*) | ✅ Ghi đúng | BA define được nếu đủ context từ UI spec |
| State machine transition | ✅ Ghi đúng | BA define được nếu logic rõ ràng |

### Về tag `[BA-inferred]`

- Tag `[BA-inferred]` **CHỈ xuất hiện trong Absorption Proposal / Change Manifest**
- **KHÔNG BAO GIỜ** ghi tag này vào docs output (FRD, SRS, UC, ASCII screen)

---

## Lớp 3 — Change Manifest (File thay đổi tập trung)

### Khi nào cần Change Manifest?

| Trigger | Cần Manifest? | Lý do |
|---|---|---|
| Absorb spec mới từ khách | ✅ BẮT BUỘC | `ba-impact` sinh ra Manifest từ Proposal đã duyệt |
| Impact từ requirement change | ✅ BẮT BUỘC | Cần trace ảnh hưởng lan truyền |
| SRS authoring (UC mới) | ✅ BẮT BUỘC | Cần confirm UC scope + EC trước khi viết |
| Fix typo / update date / format fix | ❌ Auto-apply | Minor change, không ảnh hưởng logic |

### Format

Sử dụng template tại `~/.claude/ba-filter/change-manifest-template.md`.

---

## Lớp 4 — BA Requirement Translation Guideline (v2.1)

### Mục đích
Phân chia trách nhiệm: 
- `ba-absorb` phân tích logic cốt lõi (Sequence, Edge Case) cho requirement trực tiếp.
- `ba-impact` quét ảnh hưởng lan truyền (cross-reference) và đóng gói Change Manifest.

---

### GIAI ĐOẠN 1: `ba-absorb` (Phân tích cốt lõi)

Khi nhận spec mới, luồng `ba-absorb` chạy các bước sau:

#### Step 1 — Lọc Tech & Xác định Scope
Đọc spec, áp dụng Lớp 2. Summary thành Scope:
- Thuộc BA scope (UI rules, UC mới, UC đổi logic/UI).
- Ngoài BA scope (skip data, db, internal api).

#### Step 2 — Phân tích Luồng & Cạnh (Sequence & Edge Cases)
Với mỗi UC **trực tiếp** bị thay đổi logic:
1. **Sequence Diagram**: Vẽ luồng chạy (Happy Path + Alternative).
2. **Break Point Analysis**: Áp dụng `seq-to-ec-template.md`, tìm điểm gãy.
3. **Edge Case Grouping**: Gộp các điểm gãy thành bảng EC chuẩn (`EC-{MOD}-{NNN}`).

#### Step 3 — Xuất Absorption Proposal
Đóng gói kết quả Step 1 & 2 thành **Absorption Proposal**:
- Scope
- Sequence Diagrams (cho các luồng bị đổi)
- Edge Cases table
- UI/Msg updates

→ Đưa cho BA human review, sửa đổi và Approve.

---

### GIAI ĐOẠN 2: `ba-impact` (Quét lan truyền & Compile Manifest)

Sau khi BA Approve Proposal, chuyển cho `ba-impact` chạy tiếp:

#### Step 4 — Impact Scan (Quét ảnh hưởng chéo)
Từ danh sách UC trực tiếp trong Proposal, `ba-impact` quét:
1. UC nào consume output từ UC này?
2. Shared rules (BR-*, CR-*) nào bị đụng?
3. Có vỡ luồng state machine không?

#### Step 5 — Compile Change Manifest
Ghép kết quả quét ảnh hưởng chéo (Step 4) với Proposal đã duyệt (gồm Seq & EC) vào **Change Manifest** chuẩn (Lớp 3).

→ Đưa cho BA human review lần cuối. BA Approve thì apply changes.

---

## Bảng mapping Edge Case (Sử dụng cho Lớp 4 Step 2)

Sử dụng template chi tiết tại `~/.claude/ba-filter/seq-to-ec-template.md`.

**EC Naming Convention:**
- Format: `EC-{MODULE}-{NNN}`
- `{MODULE}`: slug module (ADM, POOL, EX, IB, OPR...)
- `{NNN}`: 3-digit sequential number
- **Flow Ref** column: ghi rõ EC thuộc luồng nào (UC-ID + section)

---

## Changelog

| Date | Author | Change |
|---|---|---|
| 2026-07-17 | @linh.nguyen | v2.1 — Tách biệt rõ 2 giai đoạn ba-absorb và ba-impact |
| 2026-07-17 | @linh.nguyen | Initial version — 4 lớp kiểm soát |
