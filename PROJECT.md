# Project

## Goal

Preserve Jev Ultrafast as a readable, upstream-comparable browser-agent
baseline while evaluating whether it should become the browser-runtime
foundation for Karas-Service-Hub.

The immediate product question is not "how do we extend it?" but "how far does
the upstream architecture already solve the real browser tasks that motivated
our existing Browser Bridge?" Changes on this fork should therefore begin with
measured A/B experiments against real tasks before architecture is migrated.

## Scope

- The upstream Jev Ultrafast agent loop and Browser Harness integration.
- The `karas-dev` branch for local experiments and adaptations.
- Baseline and A/B testing against the existing Browser Bridge.
- Real-site compatibility experiments, starting with the Comm100 AI Agent Test
  flow used by the current browser-harness work.
- A thin Gateway/MCP integration only after the upstream baseline is understood.
- Office-safe structured browser control without screenshot-dependent agent
  behavior.

## Non-goals

- Replacing Browser Bridge before comparative evidence supports doing so.
- Rewriting the upstream loop before an unmodified baseline is recorded.
- Adding site-specific selectors, prepared plans, or hardcoded field values.
- Making screenshots part of the Office agent loop.
- Duplicating Karas-Service-Hub lifecycle, routing, or tunnel responsibilities
  inside this repository.

## Current Status

- Forked from `browser-use/jev-ultrafast`; upstream code is currently
  unchanged apart from Playbook project documentation.
- Development work is isolated on `karas-dev`; `main` remains the imported
  upstream baseline.
- Upstream commit at adoption: `1231850` ("docs: announce the Cloud waitlist
  below the README title").
- Baseline checks on the Office machine pass:
  - `uv run ruff check .`
  - `uv run python -m pytest` -> 31 passed
  - JavaScript syntax checks for `static/app.js` and `snapshot.js`
  - `uv build`
- The upstream agent has not yet been run against the Comm100 comparison task
  in this fork.
- No Jev-Ultrafast-specific Gateway provider or MCP surface has been added yet.

## Architecture

### Agent loop

`jev_ultrafast/agent.py` owns the bounded loop. Each cycle observes a page,
asks the policy for an operation/target decision, executes at most one mutation,
then observes again. A stale decision is discarded rather than replayed.

### Observation

`jev_ultrafast/snapshot.js` performs one browser-side DOM read that produces
visible page text, indexed controls/actions, current values/state, geometry,
scroll state, and freshness/guard information. The default model loop is
structured-state-driven rather than screenshot-driven.

### Policy

`jev_ultrafast/model.py` builds dynamic operation-specific target heads for
TypeSafe Jev. The model selects only from code-generated operations and observed
targets. For `TYPE_TEXT`, a separate small text model generates the field
value from the overall goal, selected field, visible page context, and recent
actions.

### Browser execution

`jev_ultrafast/browser.py` uses Browser Harness/CDP. Each `Agent` creates and
owns a background tab in the existing Chrome profile, attaches one CDP session,
rechecks freshness and geometry before input, and closes its owned target when
finished.

### Inspector

`jev_ultrafast/demo.py` and `jev_ultrafast/static/` provide the local
inspector. Screenshots/recordings are optional inspector/demo features and are
not required by the default model loop.

## Key Decisions

- **Keep upstream `main` as a comparison baseline.** Local product experiments
  belong on `karas-dev` so upstream changes and our adaptations remain easy to
  distinguish.
- **Measure before migrating.** The next step is an upstream-style Comm100 run,
  not an immediate Browser Bridge rewrite.
- **Preserve the small-loop design.** Jev chooses from observed operations and
  targets; workflow-specific plans and selectors do not belong in the runtime.
- **Keep screenshots out of the Office agent path.** Office browser work must
  remain based on DOM/accessibility/structured browser data. Optional upstream
  screenshot/demo features are not evidence that they should be exposed through
  the Office Gateway.
- **Reuse Karas-Service-Hub for remote exposure.** If this runtime is integrated,
  it should become another local Gateway provider rather than create a separate
  Secure Tunnel.

## Known Issues

- The upstream `Agent` creates its own background tab; it does not currently
  attach to an already-open user tab such as the existing Comm100 Overview tab.
- The project itself exposes no Jev-Ultrafast MCP/Gateway API. Browser Harness,
  installed as a dependency, has its own MCP tooling, but that is a lower-level
  browser surface rather than this agent loop.
- Upstream documents unsupported or incomplete cases including shadow roots,
  frames, canvas, uploads, pop-up tabs, nested scrolling, and arbitrary keyboard
  widgets.
- The DOM reader intentionally sends visible text rather than a full-page
  semantic snapshot. Whether that projection is sufficient for our real tasks
  remains to be measured.
- On the Office machine, Device Guard blocks the generated `pytest.exe`
  wrapper; `uv run python -m pytest` runs the same test suite successfully.

## Next

1. Run the closest possible **unmodified upstream baseline** on the Comm100
   task: open Test, generate a harmless message, send it, and decide whether the
   result is visibly complete.
2. Record steps, model calls, latency, token/cost data, visible page context,
   action path, and independently verified outcome.
3. Identify the minimum compatibility gap between the upstream owned-tab model
   and our existing authenticated Comm100 tab. Do not solve more than the
   baseline requires.
4. Compare the result with Browser Bridge before deciding whether to extend,
   wrap, or migrate either runtime.
5. Only then design a thin Karas Gateway/MCP adapter if Jev Ultrafast remains a
   useful candidate.

