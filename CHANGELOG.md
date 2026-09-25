# Changelog

Meaningful upstream and fork milestones are recorded here. Routine code changes
remain in Git history.

## 2026-09-24 - Karas fork adopted under the Project Playbook

### Why

The fork is being evaluated as a possible browser-agent runtime for
Karas-Service-Hub and needs durable local context without blurring upstream
behavior with experimental adaptations.

### What

- Created the `karas-dev` development branch while leaving `main` at the
  imported upstream baseline.
- Added the shared Project Playbook pointer to `AGENTS.md`.
- Added `PROJECT.md` to record the fork's current comparison goal, constraints,
  architecture, known gaps, and next experiment.
- Preserved the upstream README and implementation as the baseline.

### Result

The repository now has an explicit handoff contract for comparative work while
remaining easy to diff against upstream.

### Notes

At adoption, the Office baseline passed Ruff, 31 offline pytest tests, both
JavaScript syntax checks, and `uv build`. Device Guard required invoking
pytest as `uv run python -m pytest` rather than the generated wrapper.

## 2026-09-16 - Upstream optimized real-web runtime

### Why

The initial upstream prototype used many browser protocol calls and paid
latency for repeated accessibility-tree/DOM work.

### What

- Moved common page observation to one browser-side DOM snapshot.
- Kept code-owned node identity and target freshness checks.
- Combined Jev operation and operation-specific target heads in one decision
  request.
- Retained separate text generation for `TYPE_TEXT`.

### Result

The upstream performance report records a matched three-pair Google Flights
comparison where both arms passed 3/3 and the optimized median task time was
7.092 s versus 9.450 s for the frozen original runtime. The report explicitly
describes this as a small controlled comparison, not a broad reliability
benchmark.

