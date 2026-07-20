<purpose>
Orchestrate Phase 1 of the BA Absorption Filter pipeline when AI absorbs a new client specification
or external document.

This workflow applies Lớp 1 & Lớp 2 filters, performs direct logic analysis (Lớp 4, Steps 1-3),
and produces an Absorption Proposal for human review. Once approved, the proposal is passed
to `ba-impact` for Phase 2 (Impact Scan & Change Manifest generation).
</purpose>

<required_reading>
Read `~/.claude/ba-filter/ba-absorption-filter.md` (the ruleset) before any work.
Read `~/.claude/core/contract.yaml` and `~/.claude/core/contract-behavior.md` for paths and shared policy.
</required_reading>

<process>

<step name="validate">
If no source document is specified, ask:

"Which client specification or document should I absorb? Provide the file path or paste the content."

If the source is a file path, verify it exists. If it does not, stop and report.
</step>

<step name="resolve-project">
Detect BA-kit project context from CWD (matches `plans/{slug}-{date}` pattern).

1. Extract slug and date from the plan directory name.
2. If no project context is found, ask for `--slug` and `--date`.
3. Resolve module if the source document is module-specific.
</step>

<step name="read-filter">
Read `~/.claude/ba-filter/ba-absorption-filter.md` in full.
This is MANDATORY. Do not skip.
</step>

<step name="read-source">
Read the source document. For large documents:

1. If a `paths.source_summary` exists for this source, read that first.
2. Otherwise, read the full document but apply token discipline.
3. Identify sections relevant to BA scope vs sections to skip (Lớp 2 policy).
</step>

<step name="direct-analysis">
Apply Lớp 4 (Giai đoạn 1: Steps 1-2).

1. **Scope Identification**: Determine what UCs are new, what UCs have logic changes, and what UI rules change.
2. **Logic Analysis**: For each UC with a logic change:
   - Generate a Sequence Diagram (Happy Path + Alternative).
   - Perform Break Point Analysis.
   - Group into Edge Cases (EC-{MODULE}-{NNN}).
</step>

<step name="generate-proposal">
Apply Lớp 4 (Step 3).

Generate an **Absorption Proposal** and present it to the user:

```markdown
## Absorption Proposal

**Source**: {source document name}
**Type**: absorb
**Summary**: {1-2 sentence summary}

### ✅ In BA Scope — Cần xử lý:
1. UI rules updates: ...
2. New UCs: ...
3. Direct Logic Changes: 
   - UC-XXX: [mô tả]
   - Sequence Diagram: [mermaid code]
   - Edge Cases: [table with EC-IDs]

### ❌ Out of BA Scope — Skip:
{list of items dev tự define}
```
</step>

<step name="handoff">
Wait for user approval of the Absorption Proposal.

Once the BA approves the Proposal (including the Sequence and Edge Cases), STOP execution and instruct the user or the runtime to run `ba-impact`:

"Absorption Proposal đã được duyệt! Để quét ảnh hưởng lan truyền (cross-reference, shared rules) và sinh Change Manifest hoàn chỉnh, hãy chạy lệnh:
`/ba-impact --slug <slug> [đường_dẫn_tới_file_proposal_nếu_có]`"

Do NOT manually run the impact scan steps here. Let `ba-impact` handle Phase 2.
</step>

</process>

<stop_conditions>
- Source document not found → stop, report path.
- No project context resolvable → stop, ask for slug/date.
- Absorption Proposal rejected → stop, wait for revised scope.
</stop_conditions>
