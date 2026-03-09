# Model Router

**Cost-optimized AI model routing for OpenClaw agents.**

A practical skill for Claude-based agents to intelligently route tasks between Haiku (fast/cheap), Sonnet (balanced), and Opus (deep reasoning) models—saving 60-80% on API costs while maintaining quality.

## Problem

Most agent setups run everything on expensive models like Sonnet or Opus, wasting tokens on routine work:

- **Heartbeat checks** ❌ Opus ($75/M output)
- **File parsing** ❌ Sonnet ($15/M output)
- **Simple Q&A** ❌ Claude 4.5 ($120+/M output)
- **Sub-agent data fetching** ❌ Premium model

**Cost impact**: A typical agent using Opus/Sonnet for 100% of tasks spends ~$225/month, even though 80% of work is routine.

## Solution

Model Router applies a simple decision tree:

1. **Classify** the task (single-step? well-defined? novel?)
2. **Route** to the cheapest viable model (Fast → Balanced → Deep)
3. **Auto-escalate** if stuck, **auto-downgrade** when done

**Real-world savings**: Same 100K tokens/day → ~$19/month (80% routine on Haiku, 15% moderate on Sonnet, 5% complex on Opus).

## Features

✅ **Three-tier model hierarchy** — Not overly complex scoring, simple rules agents actually follow  
✅ **Per-agent defaults** — Design agents default to Fast; coding agents to Balanced  
✅ **Auto-escalation & downgrade** — Fail forward to a better model, auto-recover when task is done  
✅ **Sub-agent optimization** — Spawn helpers on Haiku by default, upgrade only when needed  
✅ **Optional silent mode** — Confirm once, then auto-route without asking  
✅ **Cost awareness** — Agent knows price ratios (1x / 3x / 5x) to inform decisions  
✅ **Human in the loop** — Suggests switches, doesn't force them  

## Installation

```bash
clawhub install acoemeta/model-router
```

## Quick Start

### Automatic Routing

Agent automatically classifies incoming tasks:

- **Simple queries** → Haiku ($1-$5/M output)
- **Writing/analysis** → Sonnet ($3-$15/M output)
- **Architecture/security** → Opus ($5-$25/M output)

### Manual Switching

```bash
/model anthropic/claude-haiku-4.5     # Cut costs now
/model anthropic/claude-sonnet-4.6    # Back to balanced
/model anthropic/claude-opus-4.6      # Heavy reasoning
```

### Silent Mode

```bash
/router silent
# Now agent auto-routes without confirmation
# Each response tagged: [🟢 Haiku] / [🟡 Sonnet] / [🔴 Opus]
```

## Model Tiers

| Tier | Model | Best For |
|------|-------|----------|
| 🟢 **Fast** | Haiku 4.5 | Lookups, formatting, translation, simple Q&A, heartbeat, cron, sub-tasks |
| 🟡 **Balanced** | Sonnet 4.6 | Writing, analysis, code, research, product thinking, interactive chat |
| 🔴 **Deep** | Opus 4.6 | Architecture, security review, complex debugging, high-stakes output |

## Cost Reference

| Tier | Input $/1M | Output $/1M | Relative Cost |
|------|-----------|------------|-----------------|
| Fast | $1.00 | $5.00 | **1x** |
| Balanced | $3.00 | $15.00 | 3x |
| Deep | $5.00 | $25.00 | 5x |

## Economics

**Assuming 100K tokens/day:**

| Strategy | Monthly Cost | Trade-off |
|----------|--------------|-----------|
| Pure Opus | ~$225 | Maximum capability, wasteful on routine |
| Pure Sonnet | ~$45 | Good baseline, still overspends on simple work |
| Pure Haiku | ~$6 | Very cheap, fails on complex tasks |
| **Model Router (80/15/5)** | **~$19** | ✅ Best of all worlds |

## Decision Rules

**Use Fast (Haiku) when:**
- Single-step, unambiguous task
- No judgment or synthesis required
- Cron / heartbeat / monitoring tasks
- Sub-agents doing fetch/parse/format work

**Use Balanced (Sonnet) when:**
- Multi-step but well-defined
- Writing, summarization, code generation
- Most interactive conversations (default)

**Use Deep (Opus) when:**
- Novel problem with no clear pattern
- Previous attempt on Sonnet failed
- High-stakes or irreversible actions
- Deep multi-step reasoning required

## Configuration

No extra config needed—install and it activates on every message. To customize models, edit the SKILL.md frontmatter or update your openclaw.json defaults.

## Per-Agent Defaults

| Agent | Name | Default | Role |
|-------|------|---------|------|
| main | Jonson | Balanced | Interactive dialogue, product decisions |
| dev | Evan | Balanced | Code quality needs stability |
| design | Ivy | Fast | Design descriptions, reference gathering |
| ops | Steven | Fast | Monitoring, reporting, formatting |

## Design Philosophy

1. **Keep it simple** — Three tiers, not ten. Easy for agents to understand.
2. **Cost transparency** — Explicit price ratios (1x/3x/5x) help agents make trade-offs.
3. **Human approval** — Suggests, doesn't auto-escalate silently. User stays in control.
4. **Progressive disclosure** — Starts Balanced (safe), escalates only when stuck.
5. **Sub-agent optimization** — Most cost savings come from routing helpers to Haiku.

## License

MIT
