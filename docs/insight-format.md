# Insight Format Specification

This document defines the format used by Serendipity for capturing and persisting peripheral insights. This format is designed to be human-readable, tool-agnostic, and simple enough that other tools could adopt it.

## Agent Output Format

Research agents include peripheral findings in a `<serendipity>` block at the end of their response:

```xml
<serendipity>
- finding: Ratatui's immediate-mode rendering reduces frame allocation overhead by 40% compared to retained-mode TUI frameworks
  relevant_to: CLI Dashboard Tool
  why: Your dashboard renders high-frequency data updates — immediate-mode would eliminate the retained widget tree overhead you're hitting
  source: https://ratatui.rs/concepts/rendering/
  confidence: High

- finding: The OpenTelemetry Collector has a tail-sampling processor that makes probabilistic sampling decisions at the trace level, not span level
  relevant_to: General Interests
  why: Relevant to observability patterns — most teams sample wrong by deciding per-span
  source: https://opentelemetry.io/docs/collector/
  confidence: Medium
</serendipity>
```

### Fields

| Field | Required | Description |
|-------|----------|-------------|
| `finding` | Yes | One sentence describing what was found. Should be specific and actionable. |
| `relevant_to` | Yes | Project name from the user's registry, or "General Interests" |
| `why` | Yes | One sentence explaining why this finding matters for that project |
| `source` | Yes | URL if available, otherwise "from research context" |
| `confidence` | Yes | `High` or `Medium`. Low-confidence findings should not be included. |

### Rules

- Findings must be specific. "This area is evolving quickly" is not a finding.
- Each finding should relate to a registered project or general interest.
- The agent should not force findings. An empty block is valid and expected.
- The main research answer always comes first. The `<serendipity>` block is appended after.

## Persisted Insight Format

When a user keeps an insight, it is written in this markdown format:

```markdown
## Ratatui's immediate-mode rendering reduces frame allocation overhead by 40%
- **Date:** 2026-04-12
- **Source session:** Research on TUI framework comparison for Rust
- **Why relevant:** Your dashboard renders high-frequency data updates — immediate-mode would eliminate the retained widget tree overhead
- **Source:** https://ratatui.rs/concepts/rendering/
- **Confidence:** High
- **Status:** Unreviewed

---
```

### Fields

| Field | Description |
|-------|-------------|
| **Title** | H2 heading derived from the finding text |
| **Date** | ISO date when the insight was captured |
| **Source session** | Brief description of what research produced this insight |
| **Why relevant** | Explanation of relevance to the tagged project |
| **Source** | URL or "from research context" |
| **Confidence** | High or Medium |
| **Status** | `Unreviewed` (default), can be manually changed to `Reviewed`, `Actioned`, `Dismissed` |

### File Organization

**Per-project (default):** One file per project, named by slugified project name. Example: `cli-dashboard-tool.md` contains all insights tagged to that project.

**Single-file:** All insights in one file: `serendipity-insights.md`, grouped by project with H1 headings.

## Empty Response

When no peripheral insights are found, the agent returns:

```xml
<serendipity></serendipity>
```

This signals that the agent followed the instructions but found nothing worth flagging. No review prompt is shown to the user.
