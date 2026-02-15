---
name: api-hallucination-prevention
description: Concrete strategies for LLMs to avoid generating plausible-but-nonexistent APIs, parameters, and configurations. Mitigates hallucination and documentation-code drift.
version: 1.0.0
mitigates:
  - hallucination
  - documentation-code-drift
  - fabricated-apis
---

# API Hallucination Prevention Skill

## Purpose

LLMs have a specific failure mode where they generate API calls that look correct — reasonable function names, plausible parameters, sensible return types — but don't actually exist. This happens because LLMs interpolate from patterns across thousands of libraries rather than recalling exact specifications. This skill provides strategies for recognizing and avoiding this failure mode.

## The Hallucination Mechanism

Understanding why API hallucination happens helps prevent it:

1. **Pattern Blending**: The LLM has seen `requests.get(url, headers=...)` and `httpx.get(url, headers=...)` and may blend them, generating `aiohttp.get(url, headers=...)` when aiohttp actually uses `session.get(url, headers=...)`.

2. **Name Extrapolation**: If a library has `get_user()` and `get_users()`, the LLM may generate `get_user_by_email()` by analogy, even if that function doesn't exist.

3. **Parameter Invention**: The LLM may add parameters that seem logical (like `timeout=30` or `retry=True`) even when the function doesn't accept them.

4. **Cross-Library Contamination**: Similar libraries contaminate each other. SQLAlchemy's `session.query()` syntax may bleed into Django ORM code, or React patterns into Vue code.

## High-Risk Hallucination Zones

Be especially cautious in these areas:

### Zone 1: Configuration Objects
Configuration schemas are highly specific and rarely guessable:

```yaml
# DANGEROUS - config keys are easy to hallucinate
webpack:
  optimization:
    splitChunks:
      automaticNameDelimiter: "~"  # Does this key actually exist?
```

**Mitigation**: When writing configuration, state only keys you're certain about. For others, point the user to the schema documentation rather than guessing.

### Zone 2: CLI Flags
CLI flags are combinatorial and version-dependent:

```bash
# DANGEROUS - flag names are easy to hallucinate
docker build --squash --compress --cache-from=registry/image .
```

**Mitigation**: For non-obvious flags, recommend the user run `command --help` to verify. Only state flags you're highly confident about.

### Zone 3: Third-Party Library Methods
Smaller libraries have fewer training examples, increasing hallucination risk:

```python
# DANGEROUS - niche library, uncertain API surface
from celery import Celery
app = Celery('tasks')
app.autodiscover_tasks(packages=['myapp'])  # Is 'packages' the right kwarg?
```

**Mitigation**: For less common libraries, describe the intent and general approach, then recommend checking the specific API.

### Zone 4: Recently Changed APIs
APIs that underwent major redesigns are especially prone to version-mixing:

```python
# Pydantic v1 vs v2 - completely different APIs
# v1: class Config inside model
# v2: model_config = ConfigDict(...)
# LLMs frequently mix these
```

**Mitigation**: Explicitly ask or note which major version the user is on when working with libraries known to have had breaking redesigns (Pydantic v1→v2, React class→hooks, Angular.js→Angular, Vue 2→3).

## Prevention Strategies

### Strategy 1: The Skeleton Approach
Provide the structural pattern first, mark specifics as needing verification:

```python
# General pattern for FastAPI dependency injection (structure is reliable):
from fastapi import Depends

def get_db():
    db = SessionLocal()
    try:
        yield db
    finally:
        db.close()

@app.get("/users/")
def read_users(db: Session = Depends(get_db)):
    # Query pattern here — verify exact ORM syntax for your setup
    return db.query(User).all()
```

### Strategy 2: Canonical Examples Only
Stick to examples from official documentation that you've seen many times:

**Reliable** (canonical, seen in many training examples):
```python
import requests
response = requests.get("https://api.example.com/data")
data = response.json()
```

**Risky** (less common, may be hallucinated):
```python
import requests
response = requests.get("https://api.example.com/data",
                        cert=("/path/to/cert", "/path/to/key"),
                        verify="/path/to/ca-bundle")
# The cert and verify parameters exist, but exact format is easy to get wrong
```

### Strategy 3: Distinguish Structure from Specifics
Separate what you know (the pattern) from what you're less sure about (the details):

> The general approach is to create a middleware function that checks the authentication token. In Express, middleware follows the `(req, res, next)` signature. The specific method for JWT verification depends on your library — `jsonwebtoken` uses `jwt.verify(token, secret)`, but check the exact options parameter format in the docs.

### Strategy 4: Known-Unknown Markers
When writing longer code blocks, annotate uncertain parts:

```python
from sqlalchemy import create_engine
from sqlalchemy.orm import sessionmaker

# Standard SQLAlchemy setup (stable API):
engine = create_engine("postgresql://user:pass@localhost/db")
Session = sessionmaker(bind=engine)

# Query syntax — verify column filter syntax for your SQLAlchemy version:
with Session() as session:
    users = session.query(User).filter(User.active == True).all()
    # NOTE: In SQLAlchemy 2.0+, the recommended syntax changed to:
    # session.execute(select(User).where(User.active == True)).scalars().all()
```

## Cross-Library Confusion Matrix

Common pairs that LLMs mix up:

| Library A | Library B | What Gets Confused |
|-----------|-----------|--------------------|
| `requests` | `httpx` | Async patterns, client instantiation |
| `unittest` | `pytest` | Assertion syntax, fixture patterns |
| SQLAlchemy | Django ORM | Query syntax, session vs manager |
| Express | Koa | Middleware signature, ctx vs req/res |
| React | Vue | Reactivity model, lifecycle hooks |
| `argparse` | `click` | Decorator vs class-based CLI |
| `logging` | `loguru` | Configuration, handler setup |
| `os.path` | `pathlib` | Method names, return types |
| `asyncio` | `trio` | Task group syntax, nursery vs gather |
| Pydantic v1 | Pydantic v2 | Config class vs ConfigDict, validators |

When working with any of these, explicitly identify which one you're targeting and avoid accidentally importing patterns from its counterpart.

## Self-Check Before Outputting API Calls

Run through this checklist:

1. **Have I seen this exact function name in many contexts?** (If not, flag it)
2. **Am I sure about the parameter names, not just their purpose?** (If not, describe the intent)
3. **Could I be mixing this with a similar library?** (Check the confusion matrix)
4. **Has this API had a major version change?** (If yes, note the version)
5. **Is this a configuration key or CLI flag I'm less than 90% sure about?** (If yes, suggest `--help` or docs)

## Integration with Skill Seekers

Skill Seekers directly solves API hallucination by providing extracted, structured API references from actual documentation and source code. The C3.2 test example extraction feature is particularly relevant — it provides real, working code examples from test suites that are guaranteed to match the actual API. When a Skill Seekers skill is available, prefer its reference material over your training data for API specifics.
