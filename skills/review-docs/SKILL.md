---
name: review-docs
description: Multi-persona document review and improvement. Spawns parallel review agents (quality, cross-reference, persona-based) to evaluate research documents, then presents findings and optionally applies fixes. Use when user says 'review docs', 'evaluate this doc', 'review research', or 'critique this'.
---

# Document Review & Improvement

Evaluate research documents through multiple expert perspectives, then improve them based on findings.

## When to Activate

- Reviewing research documents for quality, accuracy, consistency
- Getting multi-perspective feedback on long-form writing
- Finding contradictions, gaps, or weak arguments across a document set
- Improving documents based on structured critique

## Usage

```
/review-docs <path>                          # Review file or folder, all personas, report-then-fix
/review-docs <path> --auto-fix               # Apply high-priority fixes automatically
/review-docs <path> --personas "adversarial, academic"  # Subset of personas
/review-docs <path> --quality-only           # Skip personas, just quality + cross-ref
```

## Arguments

| Arg | Default | Description |
|-----|---------|-------------|
| `path` | (required) | File or folder to review |
| `--auto-fix` | off | Apply high-priority fixes without asking |
| `--personas` | all | Comma-separated list of persona names to use |
| `--quality-only` | off | Run only quality and cross-reference review (no personas) |
| `--skip-cross-ref` | off | Skip cross-reference review (useful for single-doc review) |

## Workflow

### Step 1: Scope Discovery

1. If `path` is a file, review that single document
2. If `path` is a folder, discover all `.md` files and present the list
3. Ask user to confirm scope (or let them exclude files)

### Step 2: Load Personas

Load persona definitions from `.claude/skills/review-docs/personas/`. Each persona file defines:
- **Role**: Who they are
- **Lens**: What they evaluate through
- **Criteria**: Specific things they look for
- **Output format**: Grade + ranked issues + improvements

Default: load ALL personas. If `--personas` is specified, filter to those names only.

### Step 3: Spawn Review Agents (Parallel)

Launch these agents in parallel using the Agent tool:

#### Quality Reviewer Agent
```
Prompt: Read all documents in scope. Evaluate each for:
- Factual accuracy (flag claims that seem wrong or unsourced)
- Logical coherence (arguments that don't follow)
- Completeness (sections that feel thin or unfinished)
- Structural integrity (headers, flow, navigation)

Output format:
- Per-doc rating: Strong / Adequate / Weak
- Issues ranked: High / Medium / Low priority
- Each issue: what's wrong, where (file + line), suggested fix
```

#### Cross-Reference Reviewer Agent (unless --skip-cross-ref)
```
Prompt: Read all documents in scope. Check for:
- Data contradictions (same fact cited differently across docs)
- Broken or incorrect navigation links
- Missing cross-references (doc A discusses topic covered in doc B but doesn't reference it)
- Terminology inconsistencies
- Timeline/date conflicts

Output format:
- Table of contradictions (location, values, which is correct)
- Table of broken links (location, error, correction)
- Missing cross-references (source doc, target doc, topic)
```

#### Persona Reviewer Agents (one per persona)
```
Prompt: You are a {role}. Read all documents in scope through the lens of {lens}.

Evaluate against these criteria:
{criteria from persona file}

Output format:
1. Overall grade (A+ through F)
2. Top 3 strengths
3. Issues ranked by severity (with specific file + line references)
4. Specific improvement suggestions (concrete edits, not vague advice)
5. What's missing that should be there
```

#### Fact Checker Agent (if fact-checker persona is included)
```
Prompt: You are a fact-checker. Read documents in scope and extract the 10-15 most important verifiable factual claims per document. For each claim, use the Perplexity search script to verify:

```bash
python scripts/perplexity_search.py --ask "Is it true that [claim]? Provide sources."
```

Categorize each claim as Verified, Unverified, Disputed, or Incorrect. Focus on statistics, dates, institutional details, and cause-effect assertions. Skip well-known facts.

Output format: tables of verified, disputed/incorrect, and unverified claims with file:line references and sources.
```

Note: The fact-checker persona requires the Perplexity API key (`PERPLEXITY_API_KEY`) in the environment or `~/.claude/.env`. If the key is not available, skip this persona and note it in the review output.

### Step 4: Consolidate & Present

Collect all agent results. Present to user as:

```markdown
## Review Summary

**Scope:** {N} documents, {total lines} lines
**Reviewers:** Quality + Cross-ref + {N} personas

### Quality
{per-doc ratings}

### Issues by Priority
**High:** {list with file:line references}
**Medium:** {list}
**Low:** {list}

### Persona Grades
| Persona | Grade | Key Finding |
|---------|-------|-------------|
| ... | ... | ... |

### Cross-Reference Issues
{contradictions, broken links, missing refs}
```

### Step 5: Fix (report-then-fix or auto-fix)

**Default (report-then-fix):**
- Present the consolidated report
- Ask: "Apply high-priority fixes? (y/n/select)"
- If yes: apply fixes using Edit tool, working through high → medium → low
- After each priority tier, ask whether to continue to next tier

**With --auto-fix:**
- Apply all high-priority fixes automatically
- Present medium and low for user decision

**Fix rules:**
- Never remove content without replacement — rewrite, don't delete
- Preserve the author's voice and style
- For contradictions: pick the most recent/authoritative value and make consistent
- For missing cross-references: add them inline where natural
- For broken links: fix the link, don't remove it
- For factual issues: flag for user if the correct value isn't clear

### Step 6: Humanizer Post-Fix Pass

After fixes are applied (Step 5), run a humanizer sweep on all edited sections. The review fixes themselves can introduce AI writing patterns — this pass catches them.

**Process:**
1. Collect the list of files modified during Step 5
2. For each modified file, spawn an Agent with the full humanizer skill prompt (loaded from `.claude/skills/review-docs/humanizer-prompt.md`)
3. The agent reads each modified file and flags any AI writing patterns introduced by the fixes
4. Present findings to user: quote the problematic text, name the pattern, and provide a rewrite

**Prompt for post-fix humanizer agent:**
```
You are a writing editor. Read the following file and identify any passages that exhibit
AI writing patterns. Focus especially on text that was recently edited (it may have
different voice or style from surrounding text).

Load the full pattern reference from:
.claude/skills/review-docs/humanizer-prompt.md

For each issue found:
- Quote the passage with file:line reference
- Name the pattern (e.g., "significance inflation", "copula avoidance")
- Provide a concrete rewrite
- Rate severity: High / Medium / Low

Output only issues found. If the text reads naturally, say "No AI patterns detected."
```

**Auto-fix behavior:**
- With `--auto-fix`: apply high-severity humanizer fixes automatically, present medium/low for review
- Default: present all findings and ask before applying

**Skip conditions:**
- If `--quality-only` is set, skip this step (no personas ran, so no persona-influenced fixes to check)
- If no fixes were applied in Step 5, skip this step

## Persona Library

Personas are defined in `.claude/skills/review-docs/personas/*.md`. Each file is one persona.

### Built-in Personas

| Persona | File | Best For |
|---------|------|----------|
| Retail Investor | `retail-investor.md` | Actionability, jargon, practical guidance |
| Adversarial Critic | `adversarial-critic.md` | Weak arguments, bias, "so what" test |
| Academic Economist | `academic-economist.md` | Rigor, missing literature, methodology |
| Economic Historian | `economic-historian.md` | Historical accuracy, anachronisms, context |
| Geopolitical Strategist | `geopolitical-strategist.md` | Coverage gaps, scenarios, policy realism |
| Financial Advisor | `financial-advisor.md` | Fiduciary defensibility, practical tools |
| Fact Checker | `fact-checker.md` | Verifiable claims, statistics, historical accuracy |
| Central Banker | `central-banker.md` | Operational accuracy, Fed mechanics, policy implementation |
| Humanizer | `humanizer.md` | AI writing patterns, natural voice, soulless prose |
| Data Visualization Specialist | `data-visualization-specialist.md` | Visual aids, chart recommendations, flow diagrams |

### Creating Custom Personas

Copy any existing persona file and modify. Required fields:
- `name`: Short identifier (used in --personas flag)
- `role`: One-line description of who this person is
- `lens`: What perspective they bring
- `criteria`: Numbered list of specific things to evaluate
- `output_notes`: Any special instructions for this persona's output

### Example: Adding a "Software Engineer" persona
```markdown
---
name: software-engineer
role: Senior software engineer reviewing technical documentation
lens: Implementation feasibility and technical accuracy
---

## Evaluation Criteria

1. Are code examples correct and runnable?
2. Are technical claims accurate?
3. Are there missing edge cases or error scenarios?
4. Is the documentation sufficient to implement from?
5. Are there unstated assumptions about the reader's environment?

## Output Notes

- Flag any code that won't compile/run as-is
- Note missing dependency or version information
- Grade implementation readiness, not prose quality
```

## Tips

- For a single doc, use `--skip-cross-ref` since there's nothing to cross-reference
- Run `/review-docs` early and iterate — don't wait until docs are "done"
- The adversarial critic is the most valuable persona for catching blind spots
- Custom personas take 2 minutes to create and permanently expand review coverage
- The fact-checker persona requires `PERPLEXITY_API_KEY` -- set it in `~/.claude/.env` or your environment
