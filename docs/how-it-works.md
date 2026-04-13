# How Serendipity Works

## The Core Mechanism

Serendipity is a Claude Code skill — not a separate application, not a background service, not a database. It's a set of instructions that modify how Claude handles research sessions.

### The Flow

```
1. ACTIVATE
   User calls /serendipity
   Skill reads ~/.serendipity/projects.md (your project registry)
   Skill reads ~/.serendipity/config.md (your preferences)
   Claude acknowledges activation and lists watched projects

2. INJECT
   When Claude dispatches any research agent (via the Agent tool),
   it appends peripheral insight capture instructions to the agent's prompt.
   These instructions include your project registry so the agent knows
   what topics matter to you beyond the current question.

3. RESEARCH (normal)
   The research agent does its job — answers the question you asked.
   At the end of its response, it includes a <serendipity> block
   with any peripheral findings tagged by project and confidence.

4. COLLECT (silent)
   Claude strips the <serendipity> blocks from displayed output.
   You see only the focused research answer.
   Peripheral findings are appended to ~/.serendipity/pending.jsonl
   (one JSON object per line) so they survive across sessions.

5. REVIEW (end of session, or later via /serendipity review)
   Claude reads pending.jsonl and presents a summary:
   "I have N pending insights. Want to review?"
   You review each one and decide: keep, skip, or done.
   Reviewing later — even days later, in a new session — works
   the same way because the pending file persists.

6. PERSIST (with permission)
   Kept insights are written to your configured location and the
   matching line is removed from pending.jsonl.
   Skipped insights are removed from pending.jsonl too.
   Anything you don't review (or stop reviewing with "done") stays
   in pending.jsonl and surfaces again next time.
   Nothing is ever saved to your configured location without your
   explicit approval.
```

### Why Prompt Injection?

Serendipity works by adding instructions to research agent prompts — not by running a separate analysis pipeline. This is intentional:

- **Token efficient:** The agent is already reading the source material. Asking it to notice peripheral findings adds ~300-700 tokens to the prompt vs. 10,000-50,000+ tokens for the research itself.
- **No extra API calls:** No post-processing step. No separate extraction model.
- **Quality:** The same model that understands the research context is the one identifying peripheral insights. It has the full context to judge relevance.

### What Gets Injected

When you dispatch a research agent with Serendipity active, the agent receives its normal prompt plus:

1. Your project registry (names, descriptions, topics of interest)
2. Instructions to maintain a separate list of peripheral findings
3. Rules: don't force findings, don't degrade main answer quality, Medium+ confidence only
4. The `<serendipity>` output format specification

The agent's primary job is unchanged. Peripheral insight capture is explicitly secondary.

## Limitations

- **Depends on model quality:** The relevance of captured insights depends on how well the model understands your projects and makes connections.
- **Capture is per-session, review is not:** Insights are captured during research sessions, but pending insights persist in `~/.serendipity/pending.jsonl` and can be reviewed later (even in a new session) via `/serendipity review`.
- **Claude Code only (v1):** Currently works as a Claude Code skill. Future versions may support other AI tools.
- **Not deterministic:** The same research may surface different peripheral findings on different runs.
