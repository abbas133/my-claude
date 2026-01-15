# Claude Code Mastery Guide
## From Basics to Building Your Agent Team

---

# Part 1: Foundation - Understanding the System

## 1.1 The Context Window Mental Model

Think of Claude's context window as RAM, not storage:

```
┌─────────────────────────────────────────────────┐
│              200K TOKEN CONTEXT WINDOW          │
├─────────────────────────────────────────────────┤
│ System Prompt (Claude's instructions) ~10K      │
│ CLAUDE.md (YOUR instructions)         ~???      │ ← You control this
│ Skill Metadata (names + descriptions) ~500      │
│ Agent Metadata (names + descriptions) ~300      │
│ Conversation History                  ~???      │ ← Grows over time
│ Files Claude Reads                    ~???      │ ← On demand
├─────────────────────────────────────────────────┤
│ WORKING MEMORY (for reasoning)        ~???      │ ← What's left
└─────────────────────────────────────────────────┘
```

**Your 1545-line CLAUDE.md was using ~15-20K tokens** before Claude even started working. That's 10% of the window gone immediately.

**Rule of Thumb:** CLAUDE.md should be <200 lines. Everything else goes in skills or reference files.

## 1.2 What Gets Loaded When

| Content | When Loaded | Impact |
|---------|-------------|--------|
| CLAUDE.md | Always (every message) | HIGH - keep tiny |
| Skill metadata (name + description) | Always | LOW - just names |
| Skill body (SKILL.md content) | When skill triggers | MEDIUM |
| Skill references | When Claude reads them | ON-DEMAND |
| Agent metadata | Always | LOW - just names |
| Agent body | When agent invoked | In agent's context |
| Files | When Claude reads | ON-DEMAND |

## 1.3 The Three-File Rule for CLAUDE.md

Your CLAUDE.md should answer only these questions:

1. **What is this project?** (1-2 sentences)
2. **What's the structure?** (basic tree, <20 lines)
3. **What conventions MUST be followed?** (bullet points)
4. **What commands are used?** (4-5 common commands)
5. **Where to find more info?** (point to other files)

Everything else → Skills or Reference files

---

# Part 2: Skills - Teaching Claude Domain Expertise

## 2.1 How Skills Work

```
User: "Fix the overflow in the cart screen"

Claude's Brain:
1. Read skill metadata (always in context)
   - "dart-flutter-expert: Flutter development..."
   - "ui-ux-expert: UI/UX design..."
   
2. Match request → skill description
   - "overflow" + "cart screen" matches "ui-ux-expert"
   
3. Load SKILL.md body into context
   - Now Claude has the detailed instructions
   
4. Execute with skill knowledge
```

## 2.2 Skill Structure

```
.claude/skills/
└── my-skill/
    ├── SKILL.md           # Required: frontmatter + instructions
    ├── scripts/           # Optional: executable code
    ├── references/        # Optional: detailed docs (loaded on demand)
    └── assets/            # Optional: templates, images
```

## 2.3 Writing Effective Skill Descriptions

The description is the **trigger**. Claude uses it to decide when to use the skill.

**Bad:**
```yaml
description: Flutter development skill
```

**Good:**
```yaml
description: Expert Flutter/Dart guidance. Use when working with Flutter projects, implementing BLoC/Riverpod patterns, writing Dart code, optimizing performance, or following Clean Architecture.
```

Include:
- What the skill does
- Specific trigger keywords
- When to use it

## 2.4 Progressive Disclosure Pattern

Don't put everything in SKILL.md. Use references:

```markdown
# SKILL.md

## Quick Reference
[Essential info that's always needed]

## Detailed Documentation
For advanced patterns, see `references/advanced-patterns.md`
For API details, see `references/api-reference.md`
```

Claude loads references only when needed, saving context.

---

# Part 3: Agents - Your Specialized Team

## 3.1 How Agents Differ from Skills

| Aspect | Skills | Agents |
|--------|--------|--------|
| Purpose | Add knowledge | Delegate tasks |
| Context | Shared with main | **Isolated** |
| Execution | Claude uses info | Agent works independently |
| Use Case | "How do I..." | "Do this for me..." |

**Key Insight:** Agents have their own context window. When you delegate to an agent, the heavy work happens in the agent's context, not the orchestrator's.

## 3.2 Agent Configuration

```yaml
---
name: agent-name              # Required
description: When to use...   # Required - Claude uses this to route
model: sonnet                 # Optional: haiku, sonnet, opus
tools:                        # Optional: restrict what agent can do
  - Read
  - Write
  - Grep
  - Bash
---

# Instructions for the agent
```

## 3.3 How Claude Routes to Agents

Claude decides based on the **description**. You don't need special syntax.

**This works:**
```
"Fix the overflow issue in the cart screen"
→ Claude sees "overflow" + "UI issue"
→ Matches ui-fixer description
→ Delegates automatically
```

**You can be explicit:**
```
"Use ui-fixer to fix the overflow in cart screen"
```

**What DOESN'T work:**
```
"@ui-fixer.md fix this"  ← Wrong! @ is for file references
```

## 3.4 The Orchestrator Pattern

Your main Claude session is the "orchestrator." It should:
- Understand the high-level goal
- Break down into subtasks
- Delegate to specialists
- Synthesize results

```
┌─────────────────────────────────────┐
│         ORCHESTRATOR (You + Claude) │
│  - Understands goal                 │
│  - Maintains global state           │
│  - Routes to specialists            │
└──────────────┬──────────────────────┘
               │ delegates
    ┌──────────┴──────────┐
    │                     │
┌───▼───────┐       ┌─────▼────────┐
│ ui-fixer  │       │ accessibility│
│           │       │ -validator   │
└───────────┘       └──────────────┘
(own context)        (own context)
```

## 3.5 Agent Chaining

Agents can delegate to other agents:

```yaml
# accessibility-validator.md
---
name: accessibility-validator
description: Audits for WCAG compliance. Delegates fixes to ui-fixer.
---

When issues found, delegate to ui-fixer:
"Fix accessibility issue: [details]"
```

---

# Part 4: Context Management Strategies

## 4.1 Pre-emptive Compaction

Don't wait for auto-compact. Compact at natural breakpoints:

```
# After completing a feature
/compact "Preserve: completed auth feature, current focus on cart"

# Before starting complex task
/compact "Summarize progress, preserve architecture decisions"
```

## 4.2 File-Based State

Store important state in files, not conversation:

```markdown
# .claude/state/current-task.md

## Current Focus
Implementing cart checkout flow

## Decisions Made
- Using Stripe for payments
- Optimistic UI updates

## Next Steps
1. Add payment form
2. Handle errors
3. Success confirmation
```

Then reference: "Read current-task.md and continue"

## 4.3 Session Decomposition

Break large tasks into discrete sessions:

```
Session 1: Planning
→ Output: plan.md

Session 2: Implementation
/clear
→ Read plan.md, implement
→ Output: code changes

Session 3: Testing
/clear
→ Read plan.md, test
→ Output: test results
```

## 4.4 Check Context Usage

```
/context
```

If >60%, consider:
- Manual /compact
- Delegating to subagent
- Starting fresh session with file-based state

---

# Part 5: Your Optimized Setup

## 5.1 What Changed

| Before | After | Why |
|--------|-------|-----|
| 1545-line CLAUDE.md | ~80-line CLAUDE.md | 90% less context waste |
| Detailed project docs in CLAUDE.md | Point to external files | Load on demand |
| Large agent files | Condensed agents | Faster loading |
| Large skill files | Condensed skills | Essential info only |

## 5.2 File Structure

```
your-project/
├── CLAUDE.md                              # Tiny! Just essentials
├── .claude/
│   ├── agents/
│   │   ├── ui-fixer.md                   # UI bug fixes
│   │   └── accessibility-validator.md    # A11y audits
│   ├── skills/
│   │   ├── dart-flutter-expert/
│   │   │   └── SKILL.md
│   │   └── ui-ux-expert/
│   │       └── SKILL.md
│   └── settings.local.json
├── Eisaal Foundation - Architecture.md    # Detailed docs (external)
├── Eisaal Foundation - Requirements.md    # Detailed docs (external)
└── Eisaal Foundation - Theme Configs.md   # Detailed docs (external)
```

## 5.3 How to Use

**For UI fixes:**
```
"The cart screen has overflow on small devices"
→ Claude routes to ui-fixer automatically
```

**For accessibility:**
```
"Audit the checkout page for accessibility"
→ Claude routes to accessibility-validator
→ Validator delegates fixes to ui-fixer
```

**For general Flutter work:**
```
"Help me implement the payment form"
→ Claude loads dart-flutter-expert skill automatically
```

---

# Part 6: Building More Agents

## 6.1 Agent Template

```yaml
---
name: agent-name
description: Clear description with trigger keywords. Use when [specific scenarios].
model: sonnet
tools:
  - Read
  - Write
  - Grep
  - Glob
  - Bash
---

# Agent Name

You are [role]. You [primary responsibility].

## Protocol
1. [Step 1]
2. [Step 2]
3. [Step 3]

## Quick Reference
[Tables, code snippets for common tasks]

## Report Format
[How to report back to orchestrator]

## Constraints
- DO NOT [thing to avoid]
- ONLY [scope limitation]
```

## 6.2 Useful Agents to Create

| Agent | Purpose |
|-------|---------|
| code-reviewer | Review PRs, check quality |
| test-generator | Write unit/widget tests |
| performance-auditor | Find performance issues |
| api-designer | Design REST/GraphQL APIs |
| documentation-writer | Generate docs |

## 6.3 When NOT to Use Agents

- Simple questions (just ask Claude directly)
- Tasks requiring heavy conversation (agents are one-shot)
- Tasks needing your continuous input

---

# Part 7: Troubleshooting

## 7.1 "Agent not being used"

**Cause:** Description doesn't match your request

**Fix:** Update description with more trigger keywords

## 7.2 "Skill not loading"

**Causes:**
1. Wrong folder structure (needs SKILL.md inside folder)
2. Description doesn't match request
3. YAML frontmatter syntax error

**Check:**
```bash
ls -la .claude/skills/
cat .claude/skills/my-skill/SKILL.md | head -10
```

## 7.3 "Context filling up too fast"

**Causes:**
1. CLAUDE.md too large
2. Too many files being read
3. Long conversation without compaction

**Fixes:**
1. Reduce CLAUDE.md to <200 lines
2. Use /compact at breakpoints
3. Delegate heavy tasks to agents
4. Use file-based state

## 7.4 "Agent uses too much context"

**Cause:** Agent file too detailed

**Fix:** Keep agent files <150 lines. Move detailed references to skill files.

---

# Part 8: Practice Exercises

## Exercise 1: Reduce Your CLAUDE.md
1. Open your current CLAUDE.md
2. For each section, ask: "Is this needed for EVERY task?"
3. Move detailed info to external files
4. Target: <200 lines

## Exercise 2: Create a New Agent
1. Identify a repetitive task you do
2. Write an agent following the template
3. Test by describing the task naturally
4. Iterate on description if not triggering

## Exercise 3: Monitor Context
1. Start a session
2. Run `/context` after each major action
3. Notice what consumes context
4. Practice compacting at 60%

## Exercise 4: Agent Chaining
1. Give accessibility-validator a file to audit
2. Watch it delegate to ui-fixer
3. Verify the fix was applied

---

# Quick Reference Card

## Commands
```
/context    # Check context usage
/compact    # Compress conversation
/clear      # Start fresh
/agents     # List available agents
/skills     # List available skills
```

## File Locations
```
CLAUDE.md           # Project root
.claude/agents/     # Custom agents
.claude/skills/     # Custom skills
```

## Context Budget
- CLAUDE.md: <200 lines (~2K tokens)
- Skills: <100 lines each
- Agents: <150 lines each
- Compact at: 60% usage

## Delegation
Just describe the task naturally. Claude routes based on descriptions.
