---
name: serendipity
description: "Use when running research that may surface peripheral insights relevant to other projects. Captures 'research exhaust' — insights the AI encounters during deep research but would normally discard — and routes them to your active projects. Invoke before research sessions or pass 'setup' to configure, 'review' to see captured insights."
user-invocable: true
argument-hint: "[setup|review] or invoke before research to activate"
---

# Serendipity

Capture peripheral insights from AI research sessions. When you research topic X, the AI traverses a wide information landscape but filters everything down to a focused answer — silently discarding findings relevant to your other projects. Serendipity recovers those insights at near-zero token cost.

## Activation

When this skill is invoked:

1. **If argument is `setup`** — jump to the Setup Flow section below
2. **If argument is `review`** — jump to End-of-Session Review section below
3. **Otherwise** — this is a research activation. Follow the Research Activation flow.

## Research Activation

### Step 1: Check for Project Registry

Check if `~/.serendipity/projects.md` exists by reading it.

- **If it exists:** Read it and proceed to Step 2.
- **If it does not exist:** Tell the user: "Serendipity needs a project registry to know what insights to watch for. Let me set that up first." Then follow the Setup Flow below. After setup completes, return here and proceed to Step 2.

### Step 2: Read Configuration

Check if `~/.serendipity/config.md` exists.

- **If it exists:** Read it and note the persistence target, path, structure, and min_confidence settings.
- **If it does not exist:** Use defaults: persistence target = markdown, path = `~/.serendipity/insights/`, structure = per-project, min_confidence = Medium.

### Step 3: Acknowledge Activation

Tell the user:

```
Serendipity active. Watching for peripheral insights relevant to:
- [list project names from registry]

Proceed with your research — I'll capture tangential findings silently 
and ask you about them when the research is done.
```

### Step 4: Research Agent Injection

**This is the critical mechanism.** For EVERY research agent you dispatch during this session (via the Agent tool), you MUST append the following to the agent's prompt. Replace `{PROJECTS_CONTENT}` with the full text of the user's `~/.serendipity/projects.md` file:

```
---

PERIPHERAL INSIGHT CAPTURE (Serendipity)

While completing your main task, maintain a separate mental list of 
peripheral findings — things you encounter during your research that 
are NOT directly relevant to the question you were asked, but ARE 
potentially relevant to one of the user's registered projects or 
interests listed below.

USER'S ACTIVE PROJECTS AND INTERESTS:
{PROJECTS_CONTENT}

RULES:
- Your PRIMARY job is answering the main question. Do not let insight 
  capture degrade the quality of your main response.
- Only note findings you are Medium or High confidence are genuinely 
  relevant to a registered project. Do not force findings.
- Each finding should be something specific and actionable — not vague 
  observations like "this area is evolving quickly."
- Include source URLs when available.

At the END of your response (after your complete main answer), include 
any peripheral findings in this exact format:

<serendipity>
- finding: [one sentence describing what you found]
  relevant_to: [project name from registry, or "General Interests"]
  why: [one sentence explaining why this is relevant to that project]
  source: [URL or "from research context"]
  confidence: [High or Medium]
</serendipity>

If you found no peripheral insights worth noting, include an empty block:
<serendipity></serendipity>
```

### Step 5: Collect Insights During Session

As research agents return their responses:

1. Look for `<serendipity>` blocks in each agent's response
2. Parse the findings from each block
3. Store them internally — do NOT show them to the user yet
4. Present only the main research answer to the user
5. If duplicates appear across agents (same finding, same project), keep only one

### Step 6: Trigger Review

When all research for the current task is complete (all agents have returned and you've delivered the final answer to the user), proceed to End-of-Session Review.

If the user continues with non-research work after the research phase, that's fine — present the review at a natural break point. Do not interrupt focused implementation or planning work.

---

## End-of-Session Review

If no peripheral insights were captured during this session, say:

```
Serendipity: No peripheral insights captured this session.
```

If insights were captured, present them:

```
---
Serendipity captured [N] peripheral insights this session:
  - [count] potentially relevant to "[Project Name]"
  - [count] potentially relevant to "[Other Project]"
  - [count] general interest

Want to review them? (y/n)
---
```

**If the user says no:** Say "Got it — insights discarded." Do not persist anything.

**If the user says yes:** Present each insight one at a time in this format:

```
[1/N] [Title derived from finding]

Relevant to: [project name]
Why: [explanation of relevance]
Source: [URL if available]
Confidence: [High/Medium]

Keep this insight? (y/n)
```

For each insight the user wants to keep, persist it following the Persistence section below.

After reviewing all insights, summarize:

```
Serendipity: Kept [X] of [N] insights. 
[Written to configured location if any were kept.]
```

---

## Persistence

When the user keeps an insight, write it to the configured persistence target.

### Markdown (default)

**If structure is `per-project`:** Write to `{path}/{project-name-slugified}.md`. Append to the file if it exists, create it if not. Use this format:

```markdown
## [Finding title]
- **Date:** [YYYY-MM-DD]
- **Source session:** [Brief description of what research was being done]
- **Why relevant:** [The "why" from the finding]
- **Source:** [URL if available]
- **Confidence:** [High/Medium]
- **Status:** Unreviewed

---
```

**If structure is `single-file`:** Append to `{path}/serendipity-insights.md` using the same format.

### Obsidian

Same as markdown, but write to the configured Obsidian vault path. Files go in a `serendipity/` subfolder within the vault.

### Claude Code Memory

Write each insight as a memory file to the current project's memory directory. Use this format:

```markdown
---
name: [Finding title]
description: [One-line summary — relevant to project name]
type: reference
---

[Full insight content with source, why, and context]
```

Also add a pointer line to the project's MEMORY.md index.

---

## Setup Flow

Guide the user through first-time setup:

### Step 1: Create Directory

Create `~/.serendipity/` directory if it doesn't exist.

### Step 2: Project Registry

Ask the user:

```
What projects or topics are you currently working on? For each one, 
give me:
- A name
- One sentence describing what it is
- Key topics you're interested in for that project

Example:
  "API Gateway — Building a rate-limiting proxy in Rust. 
   Interested in: async patterns, load balancing, observability."

List as many as you want. You can also add "General Interests" for 
broad topics that don't belong to a specific project.
```

Take the user's response and write `~/.serendipity/projects.md`:

```markdown
# Active Projects

## [Project Name]
[User's description]
Interested in: [user's topics]

## [Next Project]
...

## General Interests
[User's general interests, or a sensible default based on their projects]
```

### Step 3: Configuration

Ask the user:

```
Where should kept insights be saved?

1. Plain markdown files (default) — ~/.serendipity/insights/
2. Obsidian vault — specify your vault path
3. Claude Code memory — saved to project memory files
4. Custom path — you specify

Pick 1-4 (or just press enter for default):
```

Write `~/.serendipity/config.md` based on their choice:

```markdown
# Serendipity Configuration

## Persistence
target: [markdown|obsidian|memory|custom]
path: [chosen path]
structure: per-project

## Filtering
min_confidence: Medium
```

### Step 4: Confirm

```
Serendipity configured:
- Watching [N] projects: [list names]
- Saving insights to: [path]

You're all set. Run /serendipity before a research session to activate, 
or /serendipity review to check captured insights.
```

---

## Managing Your Registry

To add a project: Edit `~/.serendipity/projects.md` directly, or tell the assistant "add [project name] to my serendipity registry" and it will append it.

To remove a project: Edit the file directly or ask the assistant to remove it.

To update interests: Same — edit directly or ask.

The registry should stay concise. If it grows beyond ~500 tokens (roughly 8-10 projects with descriptions), consider archiving completed projects to keep injection costs low.

---

## Key Principles

1. **Never interrupt focused work.** Insights are collected silently and presented only at natural break points.
2. **Never persist without permission.** Every kept insight requires explicit user approval.
3. **Never degrade research quality.** The main research answer is always the priority. Peripheral insight capture is secondary.
4. **Minimal token overhead.** The injection adds ~300-700 tokens per research agent — negligible compared to typical research costs of 10K-50K+ tokens.
5. **Plain text everywhere.** No databases, no embeddings, no external services. Just markdown files.
