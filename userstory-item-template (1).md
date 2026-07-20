---
type: user-story
module: "{module_slug}"
story_id: "US-{module}-{NNN}"
slug: "{story-slug}"
actor: "{actor}"
priority: P0
status: draft
source_backbone_ids: []
linked_usecases: []
linked_screens: []
created: "{YYYY-MM-DD}"
owner: "{@handle}"
changelog:
  - {YYYY-MM-DD} | /stories | initial draft
---

# US-{module}-{NNN}: {story title}

> **Quy ước điều hướng:** dùng chung số thứ tự trong tên file với UC tương ứng để dễ tra cứu 2 chiều, vd: `us-01-wallet-detail.md` ↔ `uc-01-wallet-detail.md`. Trace đầy đủ nằm ở frontmatter (`linked_usecases`, `source_backbone_ids`) — không lặp lại ở body.
>
> **Thứ tự viết:** Story Statement viết TRƯỚC (đây là nguồn) — khi elaborate UC, copy nguyên văn statement này lên đầu file UC, không viết lại/diễn giải khác. Acceptance Criteria viết SAU, khi UC đã có Main/Alt/Error Flow — mỗi AC là bản rút gọn 1 dòng từ đúng 1 flow step bên UC, không tự viết mới. Khi flow ở UC thay đổi, cập nhật lại AC ở đây.

## Story Statement

As a **{actor}**, I want to **{goal}**, so that **{business value}**.

## Acceptance Criteria

- [ ] AC-{module}-001: {criterion — rút gọn từ UC Main Flow step tương ứng}
- [ ] AC-{module}-002: {criterion}

## Notes

{Any clarifying notes, constraints, or assumptions.}
