---
name: version-aware-coding
description: Systematic approach for LLMs to generate code that accounts for version differences across frameworks and languages. Mitigates knowledge cutoff and stale pattern generation.
version: 1.0.0
mitigates:
  - knowledge-cutoff
  - stale-patterns
  - version-mismatch
---

# Version-Aware Code Generation Skill

## Purpose

LLMs are trained on code from many versions of frameworks and libraries simultaneously. Without explicit version awareness, they produce code that mixes patterns from different eras — using deprecated APIs alongside modern ones, or generating code for a version the user doesn't have. This skill provides a systematic approach to version-conscious code generation.

## When to Apply This Skill

Always apply when:
- Generating code for any framework or library (React, Django, Express, etc.)
- The user hasn't specified their version
- The user is asking about features that changed significantly across versions
- You're about to use an API you know has been deprecated or replaced

## Phase 1: Version Discovery

Before generating code, establish the version context. Use these strategies in order:

### Strategy A: Ask Directly (Preferred)
If the user hasn't specified versions:
> What version of React are you using? The approach differs significantly between React 18 and 19.

Only ask when the version materially affects your answer. Don't ask for `python --version` if the user wants a basic loop.

### Strategy B: Infer from Context Clues
Look for version signals in code the user has shared:

| Signal | Inference |
|--------|-----------|
| `createRoot()` | React 18+ |
| `ReactDOM.render()` | React 17 or earlier |
| `from django.urls import path` | Django 2.0+ |
| `from django.conf.urls import url` | Django 1.x (deprecated in 2.0, removed in 4.0) |
| `async/await` at top level | Node 14.8+ or modern browsers |
| `require()` | CommonJS (Node.js, any version) |
| `import` statements in `.js` | ESM (Node 12+ with config, or bundler) |
| `app.use(express.json())` | Express 4.16+ (built-in) |
| `app.use(bodyParser.json())` | Express < 4.16 (external middleware) |
| `pyproject.toml` | Python 3.6+ ecosystem |
| `setup.py` only | Older Python packaging |

### Strategy C: Default to Current Stable
When no version information is available, generate code for the current stable version but note your assumption:
> I'm writing this for React 19 (current stable). If you're on an earlier version, let me know — the approach may differ.

## Phase 2: Version-Appropriate Pattern Selection

### Pattern Deprecation Awareness

Maintain awareness of major deprecation milestones:

**React:**
- Class components → Function components + hooks (16.8+)
- `componentWillMount` etc. → `useEffect` (16.3+ deprecated, 18+ strict mode warns)
- `ReactDOM.render` → `createRoot` (18+)
- Client-only → Server Components (19+)

**Python:**
- `%` formatting → `str.format()` → f-strings (3.6+)
- `os.path` → `pathlib` (3.4+, increasingly preferred)
- `typing.Optional[X]` → `X | None` (3.10+)
- `asyncio.get_event_loop()` → `asyncio.run()` (3.7+)

**Node.js / JavaScript:**
- Callbacks → Promises → async/await
- `var` → `let`/`const` (ES6+)
- CommonJS `require` → ESM `import` (stable in Node 16+)
- `node-fetch` → built-in `fetch` (Node 18+)

**Django:**
- `url()` with regex → `path()` with converters (2.0+)
- Function-based views → Class-based views (progressive, both still valid)
- `django.conf.urls` → `django.urls` (2.0+)

### The "Don't Mix Eras" Rule

Never combine patterns from different version eras in the same code:

**Bad (mixed eras):**
```javascript
// React: mixing class component with hooks (impossible)
class MyComponent extends React.Component {
  const [state, setState] = useState(0); // hooks don't work in classes
}
```

**Bad (mixed eras, subtle):**
```python
# Python: mixing old and new patterns
from typing import Optional  # old style

def process(data: str | None):  # new style (3.10+)
    pass
```

**Good (consistent era):**
```python
# Python 3.10+ style throughout
def process(data: str | None) -> list[str]:
    return [item for item in data.split(",") if item]
```

## Phase 3: Migration Guidance

When a user's code uses deprecated patterns, offer migration paths:

### Gentle Migration (default)
Note the deprecation without insisting on immediate change:
> Your code uses `ReactDOM.render()`, which works but is considered legacy in React 18+. The modern equivalent is `createRoot()`. Want me to show the migration, or keep the current approach?

### Urgent Migration
Only push migration when the deprecated feature is removed or causes bugs:
> `django.conf.urls.url()` was removed in Django 4.0. Since you're on Django 4.1, this will cause an ImportError. Here's the updated version using `path()`.

## Phase 4: Version Annotations in Generated Code

When generating substantial code, annotate version assumptions:

```python
# Requires: Python 3.10+ (uses match statement and union types)
# Dependencies: FastAPI 0.100+, Pydantic v2

from fastapi import FastAPI

app = FastAPI()

@app.get("/items/{item_id}")
async def read_item(item_id: int) -> dict:
    match item_id:
        case 0:
            return {"name": "default"}
        case _:
            return {"name": f"item-{item_id}"}
```

## Quick Reference: Version-Sensitive Decision Points

| Decision | Ask/Check Version When... |
|----------|--------------------------|
| React component style | Always — hooks vs. classes is fundamental |
| Python type hints | 3.9 vs 3.10+ syntax differs significantly |
| Node.js imports | CJS vs ESM depends on version + config |
| CSS approach | Vanilla, modules, CSS-in-JS, or Tailwind |
| Testing framework | Jest vs Vitest, unittest vs pytest |
| Package manager | npm, yarn, pnpm, bun — affects lockfiles |
| Python packaging | setuptools vs pyproject.toml vs uv |

## Integration with Skill Seekers

Skill Seekers-generated skills solve the version problem at scale: they scrape **current** documentation for a specific framework version and embed it as reference material. When such a skill is loaded, version-aware coding becomes straightforward because the grounding data is current. This skill covers the cases where no such grounding data is available.
