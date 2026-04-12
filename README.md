# Serendipity

**You're throwing away 90% of what your AI research finds.**

Every time you run deep research with AI, the model traverses a massive information landscape — reading dozens of pages, evaluating hundreds of data points, making connections across domains. Then it filters everything down to a focused answer to your specific question. Everything else it found? Silently discarded. Tokens already spent, insights already paid for, thrown away.

Serendipity captures those peripheral insights and routes them to your active projects.

## How It Works

```
You ask: "Research the best TUI frameworks for Rust"
                    |
        AI researches deeply, reads 30+ sources
                    |
    +-----------+---+-----------+
    |                           |
Focused answer              Peripheral findings:
returned normally           - Found a Rust async pattern relevant
                              to your API Gateway project
                            - Spotted an observability library
                              relevant to your CLI Dashboard
                    |
        End of research session:
        "I captured 2 peripheral insights. Review? (y/n)"
                    |
        You review, keep what matters, skip the rest.
        Kept insights saved to your knowledge base.
```

**Token overhead: ~1-5%.** Serendipity recovers value from tokens already spent. It doesn't run background processing or extra API calls.

## Quick Start

### 1. Install

```bash
# In Claude Code
/plugin install serendipity
```

Or manually:
```bash
git clone https://github.com/ihsanmoha/serendipity.git
cp -r serendipity/skills/serendipity ~/.claude/skills/serendipity
```

### 2. Setup

```
/serendipity setup
```

You'll be asked for your active projects and where to save insights. Takes 30 seconds.

### 3. Use

```
/serendipity
```

Then run your research as normal. Serendipity activates silently, injects peripheral insight capture into your research agents, and prompts you to review findings when the research is done.

## Example

**Before Serendipity:**

You research "competitive landscape for developer CLI tools" and get a focused analysis. During that research, the AI found that three of the top tools use a specific plugin architecture that's directly relevant to the agent framework you're building separately. That finding is silently discarded.

**After Serendipity:**

Same research, same focused answer. But at the end:

```
---
Serendipity captured 3 peripheral insights this session:
  - 2 potentially relevant to "Agent Framework"
  - 1 general interest

Want to review them? (y/n)
---
```

You review:

```
[1/3] Plugin architecture pattern used by top CLI tools

Relevant to: Agent Framework
Why: Three leading CLI tools (Warp, Fig, Zed) use a 
     message-passing plugin architecture that mirrors 
     the agent-to-tool communication pattern you're designing
Source: https://example.com/cli-plugin-architectures
Confidence: High

Keep this insight? (y/n)
```

Kept insights are saved to your configured location (Obsidian, markdown files, or Claude Code memory).

## Configuration

### Project Registry (`~/.serendipity/projects.md`)

```markdown
# Active Projects

## API Gateway
Building a rate-limiting proxy in Rust.
Interested in: async patterns, load balancing, observability.

## Agent Framework
Designing a multi-agent orchestration library in Python.
Interested in: agent memory, tool use patterns, context management.

## General Interests
Open source growth, developer tooling trends, Rust ecosystem.
```

### Settings (`~/.serendipity/config.md`)

```markdown
# Serendipity Configuration

## Persistence
target: markdown          # markdown | obsidian | memory | custom
path: ~/.serendipity/insights/
structure: per-project    # per-project | single-file

## Filtering
min_confidence: Medium    # Low | Medium | High
```

### Persistence Options

| Target | Behavior |
|--------|----------|
| `markdown` | Writes to `~/.serendipity/insights/` (default) |
| `obsidian` | Writes to a `serendipity/` folder in your vault |
| `memory` | Saves as Claude Code memory files |
| `custom` | You specify any path |

## Commands

| Command | What it does |
|---------|-------------|
| `/serendipity` | Activate for current research session |
| `/serendipity setup` | First-time setup or reconfigure |
| `/serendipity review` | Review any pending insights |

## The Problem This Solves

AI research tools (Claude, ChatGPT, Perplexity, Gemini) are excellent at answering the question you asked. They're terrible at telling you what else they found along the way.

When you work on multiple projects, the cross-pollination of ideas between them is one of the highest-leverage activities you can do. But it requires you to manually ask "what else did you find?" after every research session — and even then, the AI only surfaces what it can remember from context.

Serendipity makes this automatic. You register your active projects once, and every research session becomes a source of potential insights for all of them.

**No databases. No embeddings. No vector stores. No background processing.** Just smart prompting that makes your existing research tokens work harder.

## How It Works (Technical)

Serendipity is a Claude Code skill — a markdown file that provides instructions to Claude. When activated:

1. **Reads your project registry** to know what topics matter to you
2. **Injects instructions** into research agent prompts: "While answering the main question, also note findings relevant to these other projects"
3. **Collects peripheral findings** from agent responses (you don't see these during focused work)
4. **Prompts for review** after research completes — you keep or skip each insight
5. **Persists** kept insights to your configured location

The research agent is already reading the content. Serendipity just asks it to notice more. That's why the token overhead is minimal.

## Contributing

Contributions welcome. The most impactful areas:

- **Prompt engineering:** Improve the quality/relevance of captured insights
- **Persistence integrations:** Add new targets (Notion, Logseq, etc.)
- **Testing:** Real-world usage reports — what worked, what was noise
- **Documentation:** Examples, guides, use cases

## License

MIT
