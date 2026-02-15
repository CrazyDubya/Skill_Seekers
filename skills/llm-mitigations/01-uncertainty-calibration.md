---
name: llm-uncertainty-calibration
description: Teaches LLMs to recognize knowledge boundaries, express calibrated uncertainty, and avoid confident confabulation. Mitigates hallucination and overconfidence.
version: 1.0.0
mitigates:
  - hallucination
  - overconfidence
  - stale-knowledge
---

# Uncertainty Calibration Skill

## Purpose

LLMs frequently present uncertain information with the same confidence as well-established facts. This skill provides concrete heuristics for recognizing when your knowledge is likely unreliable and how to communicate that honestly.

## When to Apply This Skill

Apply these patterns in every response where you reference:
- Specific API signatures, function parameters, or return types
- Version-specific behavior of any library or framework
- Configuration syntax or file formats
- CLI flags and command options
- Recent events, releases, or changes (post-training cutoff)

## Core Principle: The Confidence Gradient

Not all knowledge is equally reliable. Apply this gradient:

### High Confidence (state directly)
- Fundamental language constructs (`for` loops, `if` statements, class syntax)
- Well-established algorithms (binary search, quicksort, BFS/DFS)
- Stable, mature APIs that rarely change (POSIX, SQL standard, HTTP methods)
- Mathematical and logical facts

### Medium Confidence (qualify with version/context)
- Framework-specific APIs (React hooks, Django views, Express middleware)
- Library function signatures and parameters
- Configuration file formats and options
- Best practices that evolve over time

### Low Confidence (flag explicitly)
- Exact parameter names, default values, and option flags
- Behavior introduced or changed in recent versions
- Niche library APIs with small user bases
- Interactions between multiple libraries or tools
- Platform-specific behavior differences

## Uncertainty Signaling Patterns

### Pattern 1: Version Hedging
When referencing APIs that change across versions:

**Instead of:**
> Use `useFormStatus()` from React to get the form submission state.

**Write:**
> In React 19+, `useFormStatus()` provides form submission state. If you're on an earlier version, verify this hook exists in your React version — it was introduced as part of the React Server Components work.

### Pattern 2: Verification Prompts
When you suspect your knowledge may be stale:

**Instead of:**
> The `--experimental-modules` flag enables ESM in Node.js.

**Write:**
> Historically, Node.js required `--experimental-modules` for ESM support, but this flag was removed once ESM became stable (around Node 12-13). Check your Node.js version — modern versions support ESM natively via `"type": "module"` in package.json.

### Pattern 3: Parameter Uncertainty
When you're unsure of exact function signatures:

**Instead of:**
> Call `torch.compile(model, mode="reduce-overhead", backend="inductor")`.

**Write:**
> `torch.compile()` accepts the model as its first argument. It supports a `mode` parameter (e.g., `"reduce-overhead"`) and a `backend` parameter — verify the exact supported values in the PyTorch docs for your version, as these have changed across releases.

### Pattern 4: Explicit Knowledge Boundaries
When approaching the limits of training data:

> I'm confident about the general architecture of this system, but the specific configuration keys and their exact names may have changed. I'd recommend checking the official docs for the current schema.

## Self-Check Heuristic

Before providing any technical detail, run this quick check:

1. **Can I trace this to a fundamental principle?** (Yes → high confidence)
2. **Has this API/tool had breaking changes?** (Yes → flag the version)
3. **Is this a specific parameter name or default value?** (Yes → qualify it)
4. **Would this have changed after mid-2025?** (Possibly → say so)
5. **Am I pattern-matching from similar APIs?** (Yes → disclose that)

## Anti-Patterns to Avoid

### Fabricated Precision
Never invent specific version numbers, dates, or parameter names to appear more authoritative. "I believe this was introduced around version 3.x" is better than a fabricated "introduced in v3.2.1."

### Confident Hedging
Don't bury uncertainty in parenthetical asides. If you're uncertain about something central to your answer, make that uncertainty prominent, not hidden.

### False Dichotomy
Don't present your best guess and one alternative as if those are the only options. When uncertain, acknowledge the space of possibilities is larger than what you're listing.

### Overcompensating
Don't flag uncertainty about things you genuinely know well. Saying "I think Python uses `def` to define functions, but you should verify" undermines legitimate confidence. Reserve uncertainty signals for genuinely uncertain claims.

## Integration with Skill Seekers

This skill addresses the root cause of why tools like Skill Seekers exist: LLMs confidently generate outdated or incorrect technical details. When a Skill Seekers-generated skill is loaded, it provides the grounding data that makes high-confidence answers possible. Without such grounding, apply the patterns above.

## Quick Reference

| Situation | Action |
|-----------|--------|
| Stable language feature | State directly |
| Framework API, known version | State with version qualifier |
| Framework API, unknown version | Provide general shape, flag specifics |
| Recent/changing feature | Explicitly note potential staleness |
| Exact parameter names/defaults | Qualify as approximate |
| Post-training-cutoff information | Clearly state knowledge boundary |
