---
name: context-window-optimization
description: Strategies for LLMs and users to maximize the value extracted from limited context windows. Mitigates context overflow, information loss, and attention degradation.
version: 1.0.0
mitigates:
  - context-window-limits
  - information-loss
  - attention-degradation
---

# Context Window Optimization Skill

## Purpose

LLMs have finite context windows. Even with large windows (100K-200K tokens), attention quality degrades as context grows — information in the middle gets less weight than information at the beginning or end (the "lost in the middle" effect). This skill provides strategies for both LLMs and users to maximize the value of limited context.

## The Context Window Problem

Context windows create three distinct problems:

1. **Hard Limit**: Beyond the window size, information is simply unavailable
2. **Attention Degradation**: Even within the window, information retrieval quality drops as more context is loaded
3. **Noise Dilution**: Irrelevant context actively degrades the quality of responses to relevant queries

## Strategy 1: Hierarchical Information Architecture

Structure information in layers, from most to least essential:

### Layer 1: Active Working Set (always in context)
- The specific file being discussed
- Immediately relevant function signatures
- Current error messages or test output
- The user's stated goal

### Layer 2: Reference Material (load on demand)
- Related module interfaces
- Configuration schemas
- Test examples for the current feature
- Type definitions

### Layer 3: Background Knowledge (summarize, don't embed)
- Full project architecture
- Historical design decisions
- Tangentially related modules
- Comprehensive API references

### Application
When building skills or providing context to an LLM:

**Inefficient:**
```
Here's the entire React documentation (50,000 tokens)...
Now, how do I use useEffect?
```

**Efficient:**
```
Here's the useEffect reference page (500 tokens)...
Here's a summary of related hooks: useState, useCallback, useMemo (200 tokens)...
How do I use useEffect to fetch data on mount?
```

## Strategy 2: Progressive Disclosure Pattern

For LLMs generating responses, apply progressive disclosure:

### Step 1: Answer the Direct Question
Provide the core answer first, concisely:
> Use `useEffect` with an empty dependency array to run code once on mount.

### Step 2: Show the Essential Code
One focused example:
```javascript
useEffect(() => {
  fetchData();
}, []);
```

### Step 3: Add Context Only If Relevant
Only expand if the user's question warrants it:
> For cleanup (e.g., aborting the fetch on unmount), return a cleanup function from the effect.

### What NOT to Do
Don't preemptively dump all related information:
- Don't explain the entire hook lifecycle
- Don't compare all hook types
- Don't provide 5 different examples when 1 suffices
- Don't add caveats about edge cases unless asked

## Strategy 3: Chunked Problem-Solving

For complex tasks that exceed practical context limits, decompose explicitly:

### Decomposition Pattern
```
Task: Refactor authentication system from JWT to session-based

Chunk 1: Analyze current JWT implementation (read auth middleware + routes)
Chunk 2: Design session schema and storage (database migration)
Chunk 3: Implement session middleware (replace JWT verification)
Chunk 4: Update routes to use sessions (swap decorators/middleware)
Chunk 5: Update tests (mock sessions instead of tokens)
Chunk 6: Migration strategy (handle existing JWTs during transition)
```

Each chunk can be addressed in a focused context window without needing all other chunks loaded simultaneously.

### Cross-Chunk References
When working on Chunk 3 but referencing Chunk 1:

**Don't**: Load all of Chunk 1's code into context
**Do**: Summarize the relevant interface:
> The current JWT middleware exports `authenticateToken(req, res, next)` which sets `req.user` from the decoded token. The session middleware needs to maintain this same interface.

## Strategy 4: Context-Efficient Skill Design

When creating skills (relevant to Skill Seekers users):

### Router Skills
For large frameworks, create a router skill that directs to sub-skills:

```markdown
# React Router Skill

When the user asks about:
- Component lifecycle → Load `react-hooks.md`
- State management → Load `react-state.md`
- Routing → Load `react-router.md`
- Server components → Load `react-server.md`
- Testing → Load `react-testing.md`
```

This keeps the active context small while providing access to comprehensive knowledge.

### Reference File Sizing
Optimal reference file sizes for LLM consumption:

| File Size | Use Case | Context Impact |
|-----------|----------|----------------|
| < 2K tokens | Quick reference cards | Minimal — can always be loaded |
| 2K-10K tokens | Single-topic deep dives | Moderate — load one at a time |
| 10K-50K tokens | Comprehensive module docs | Heavy — load only when specifically needed |
| > 50K tokens | Split into sub-files | Too large for single-context use |

### Frontloading Critical Information
Place the most important content first in any document:

```markdown
# React useEffect

## Quick Answer
`useEffect(callback, deps)` runs side effects after render.
Empty deps `[]` = run once. No deps = run every render.

## Common Patterns
[3-5 most common patterns]

## Detailed Reference
[Full API documentation]

## Edge Cases
[Less common scenarios]
```

This ensures that even if the LLM's attention degrades toward the end of the document, the essential information is processed with full attention.

## Strategy 5: Active Forgetting

Explicitly identify when context can be released:

### During Multi-Step Tasks
After completing a step, summarize its output and release the details:

```
Step 1 output (summary): Created User model with fields: id, email, name,
created_at. Migration file: 001_create_users.py. ✓ Complete.

[Full Step 1 context no longer needed — summary above is sufficient]

Step 2 (active): Implement user registration endpoint...
```

### During Code Review
Review files sequentially rather than loading all at once:

```
File 1/5: auth.py — Reviewed ✓ (3 issues found: missing input validation
on line 42, SQL injection risk on line 67, unused import on line 3)

File 2/5: routes.py — Reviewing now...
[Only routes.py needs to be in active context]
```

## Anti-Patterns

### Context Hoarding
Loading "everything that might be relevant" into context. This is the LLM equivalent of keeping 50 browser tabs open.

### Repeated Full Dumps
Asking the LLM to re-read entire files it has already analyzed. Instead, reference specific sections or summarize prior analysis.

### Monolithic Skills
Creating a single 100K-token skill file. This will overwhelm context and degrade response quality. Split into focused sub-skills with a router.

## Quick Reference

| Problem | Solution |
|---------|----------|
| Too much context loaded | Use hierarchical layers, load on demand |
| Degraded quality mid-document | Frontload critical information |
| Complex multi-step task | Decompose into chunks, summarize between steps |
| Large skill file | Split into router + sub-skills |
| Need full project context | Summarize architecture, detail only active file |
| Repeated context loading | Summarize and release completed work |

## Integration with Skill Seekers

Skill Seekers addresses context optimization through several features:
- **Router Skills** (`generate_router.py`): Automatically splits large documentation into sub-skills with intelligent routing
- **Smart Categorization** (`doc_scraper.py`): Organizes content by topic so only relevant categories need loading
- **Reference File Architecture**: Separates detailed references from the main SKILL.md, enabling on-demand loading
- **Token Efficiency**: The `AI_SKILL_STANDARDS.md` explicitly designs for token efficiency in skill format
