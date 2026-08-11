---
# MODIFICATION NOTE
# Source file: bakit/skills/ba-impact/SKILL.md
# Type: MODIFY (clone)
# Target destination: skills/ba-impact/SKILL.md
# Changes: Added Phase 0 (pre-processing pipeline) and Feature Plan section at end of process
# ---

name: ba-impact
description: Analyze requirement changes against the current BA artifact set and recommend the next exact commands without mutating artifacts.
argument-hint: "[--slug <slug>] [change statement|file]"
allowed-tools:
  - Read
  - Bash
  - AskUserQuestion
---

# BA Impact

Use this command when a requirement, rule, actor, scope item, or screen behavior changes in an existing BA project and you want impact analysis before editing anything.

## Invocation

```text
/ba-impact --slug warehouse-rfp Export CSV phai co audit log
/ba-impact --slug warehouse-rfp docs/changes/new-rule.md
/ba-impact Khong co nhom admin user
```

<execution_context>
Read `~/.claude/core/workflows/impact.md`, `~/.claude/core/contract.yaml`, `~/.claude/core/contract-behavior.md`.
If missing, fall back to reading from the BA-kit repo root at `core/workflows/impact.md`, etc.
</execution_context>

<context>
$ARGUMENTS
</context>

<process>

## Phase 0 — Pre-Processing (chạy TRƯỚC impact workflow)

### Bước 0.1 — Technical Filter

Scan toàn bộ input (change statement hoặc file được truyền vào) trước khi phân tích.

Tìm kiếm và flag những nội dung kỹ thuật không thuộc BA scope:
- Tên cột DB, schema table, kiểu dữ liệu (varchar, int, uuid...)
- API endpoint cụ thể (/api/v1/..., REST path)
- Chiến lược auth kỹ thuật (JWT rotation, session storage, hashing algorithm)
- Framework/SDK/library name (Spring Boot, React, Axios...)
- Payload structure, JSON schema chi tiết
- Tên function/service/class cụ thể

Với mỗi mục bị flag: ghi vào block `[TECHNICAL-NOTE]` tách biệt, KHÔNG đưa vào FRD/Backbone.

Báo cáo ngắn gọn cho BA:
```
[Technical Filter] Đã tách N mục kỹ thuật sang phần ghi chú — không đưa vào FRD/Backbone:
- [danh sách mục kỹ thuật]

Tiếp tục phân tích phần business requirement còn lại.
```

Nếu không có mục kỹ thuật nào → skip báo cáo, tiếp tục ngay.

### Bước 0.2 — Feature Count

Đọc input đã lọc + backbone hiện tại (FRD là optional — đọc nếu tồn tại). Từ backbone, trace ra tất cả file liên quan: FRD, US, UC đang tồn tại. **Không đọc `intake.md`.**

Phân loại thay đổi:

**Feature mới hoàn toàn:** Chưa có trong backbone, cần tạo mới actor/UC/US
**Feature cần update:** Đã có trong backbone, cần sửa rule/flow/actor hiện tại

Xuất danh sách để BA xác nhận:
```
[Feature Count]
Feature mới: N
  - [tên feature] — [mô tả 1 dòng]

Feature cần update: M
  - [tên feature] — [thay đổi gì]

Xác nhận danh sách này đúng chưa? (Y / sửa: [feedback])
```

Chờ BA confirm trước khi tiếp tục. Nếu BA sửa → cập nhật danh sách và confirm lại.

### Bước 0.3 — Input Clarity Check

Với từng feature trong danh sách đã confirm, kiểm tra input có đủ 3 yếu tố:

- **Actor:** Ai thực hiện hành động này? (cụ thể — không chấp nhận "người dùng" chung chung nếu hệ thống có nhiều actor)
- **UI Coverage:** Có màn hình/flow UI không? Nếu có, đã mô tả đủ trạng thái và hành động chưa?
- **Business Rules:** Điều kiện trigger là gì? Constraint nào? Edge case nào?

Với mỗi yếu tố còn thiếu → hỏi BA cụ thể phần đó, không đoán, không tự điền.

Hỏi từng feature một (không hỏi batch). Chỉ chuyển sang feature tiếp theo khi feature hiện tại đã đủ thông tin.

### Bước 0.4 — Contradiction Check

So sánh proposed changes với rules hiện tại trong backbone và FRD:

1. Đọc backbone trước (FRD optional), trace ra các file liên quan (US, UC) từ backbone. **Không đọc `intake.md`.** Chỉ đọc sections liên quan đến features đã identify — không load toàn bộ artifact set
2. Với mỗi rule/constraint mới → kiểm tra có mâu thuẫn với rule hiện tại không

Nếu phát hiện contradiction:
```
[Contradiction Found]
Feature: [tên]
Mâu thuẫn:
  - Rule mới: [nội dung]
  - Rule hiện tại (nguồn: [file §section]): [nội dung]

Chốt rule nào? (A: giữ mới / B: giữ cũ / C: merge — mô tả cụ thể)
```

Loop cho đến khi BA resolve tất cả contradictions. Ghi lại kết quả resolve để đưa vào Change Manifest.

Nếu không có contradiction → ghi `[Contradiction Check] Không phát hiện mâu thuẫn.`

### Bước 0.5 — Change Manifest + Feature Plan

Tạo 1 file duy nhất gồm 2 phần, lưu tại `plans/{slug}-{date}/shared/manifests/CM-{YYMMDD}-{HHMM}-{source-slug}.md`.

Trước khi tạo file, chạy `git rev-parse HEAD` để lấy SHA commit hiện tại và ghi vào frontmatter field `baseline_sha`. SHA này là trạng thái repo trước khi apply bất kỳ thay đổi nào — dùng làm baseline cho incremental audit sau này.

**Phần 1 — Change Manifest** (theo template `ba-kit-proposal/02_BA_Absorption_Filter/change-manifest-template.md`):
- File nào cần thay đổi (backbone, frd — không phải US/UC)
- Section nào trong file đó
- Loại thay đổi: ADD / UPDATE / DELETE
- Source từ input nào
- Tag: client-spec hay BA-inferred

Không đi sâu vào logic/flow của từng feature ở phần này.

**Phần 2 — Feature Plan** (section `## Feature Plan` ở cuối file):
Dựa vào danh sách features từ Bước 0.2, liệt kê US/UC cần tạo/sửa cho từng feature:

```
## Feature Plan

Feature 1: [tên feature]
  Loại: [Mới / Update]
  User Story cần tạo: us-[slug]        (nếu mới)
  User Story cần modify: us-[slug]     (nếu update)
  UC mới cần tạo: uc-[slug]            (nếu mới)
  UC cần modify: uc-[slug]             (nếu update)
  Mô tả: [1 dòng tóm tắt]

Feature 2: ...
```

**Lưu ý gitignore:** Thư mục `shared/manifests/` PHẢI được thêm vào `.gitignore`. Không commit manifest lên PR hoặc main — chỉ lưu local.

Hiển thị toàn bộ manifest + feature plan cho BA review:
```
[Change Manifest + Feature Plan] Đã tạo: plans/{slug}-{date}/shared/manifests/CM-{date}-{slug}.md

Backbone/FRD sẽ thay đổi: N files
  [danh sách file và sections]

Feature Plan: M features
  [danh sách feature]

Review và xác nhận? (Y / sửa: [feedback])
```

**Sau khi BA approve:**
Agent tự chia task và thực hiện lần lượt các thay đổi trong manifest — cập nhật backbone trước, FRD sau. Không hỏi thêm BA trừ khi phát hiện thiếu thông tin trong quá trình update. Sau khi update xong backbone/FRD, **tự động bắt đầu viết User Story cho Feature 1** (gọi `steps/stories.md`) mà không cần BA gõ lệnh thêm.

---

## Phase 1 — Update Backbone + FRD

Sau khi BA approve manifest, agent thực hiện lần lượt các thay đổi được liệt kê trong Change Manifest:
1. Cập nhật backbone (rules, actors, feature map) trước
2. Cập nhật FRD (business logic, workflows) sau

Không hỏi thêm BA trừ khi phát hiện thiếu thông tin trong quá trình update. Không tạo US/UC mới ở phase này.

Sau khi update xong backbone + FRD, tự động bắt đầu `steps/stories.md` cho Feature 1 trong Feature Plan — không cần BA gõ lệnh thêm.

---

## Phase 2 — Per Feature: Stories → Use Cases

Thực hiện lần lượt từng feature trong Feature Plan:

1. **`steps/stories.md`** — Info-sufficiency gate → draft US → BA confirm → write
2. **`steps/usecase.md`** — Validate input → draft sequence+EC → BA confirm → write+self-validate
3. Sau khi xong UC → chuyển sang feature tiếp theo
4. Sau khi xong feature cuối → hỏi BA có muốn chạy audit không:

```
✅ Tất cả {N} features đã hoàn thành.
BA có muốn chạy audit toàn bộ thay đổi không?
Lệnh: /ba-content-audit --slug {slug} --manifest plans/{slug}-{date}/shared/manifests/CM-{date}-{slug}.md
(Y/n)
```

</process>
