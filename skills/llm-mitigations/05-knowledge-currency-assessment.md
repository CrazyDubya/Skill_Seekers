---
name: knowledge-currency-assessment
description: Framework for LLMs to assess whether their knowledge about specific technologies is likely current or stale, and to act accordingly. Mitigates knowledge cutoff, documentation drift, and stale recommendations.
version: 1.0.0
mitigates:
  - knowledge-cutoff
  - documentation-drift
  - stale-recommendations
  - no-persistent-memory
---

# Knowledge Currency Assessment Skill

## Purpose

Not all LLM knowledge goes stale at the same rate. The Linux kernel's system call interface changes slowly; JavaScript framework APIs change rapidly. This skill provides a framework for assessing how likely it is that your knowledge about a specific technology is still current, and what to do when it probably isn't.

## The Staleness Spectrum

Technologies fall on a spectrum from highly stable to rapidly changing:

### Tier 1: Glacial Change (Trust your knowledge)
Knowledge rarely goes stale. Safe to state confidently.

- **SQL standard syntax** — SELECT, JOIN, WHERE haven't changed meaningfully in decades
- **HTTP protocol** — Methods, status codes, header semantics are standardized
- **POSIX APIs** — `open()`, `read()`, `write()` are essentially permanent
- **Core language syntax** — Python's `for`/`if`/`def`, JavaScript's core operators
- **Fundamental data structures** — Hash maps, linked lists, trees, graphs
- **Established algorithms** — Sorting, searching, graph traversal
- **Regular expressions** — Core syntax is standardized (PCRE, POSIX)
- **Git fundamentals** — `add`, `commit`, `push`, `branch`, `merge`

### Tier 2: Slow Evolution (Mostly trustworthy, note version)
Changes happen but are well-documented and gradual.

- **Major languages** (Python, Java, Go, Rust) — New features added yearly but backward compatible
- **Databases** (PostgreSQL, MySQL, SQLite) — Stable core, new features added conservatively
- **Linux/Unix tools** — `grep`, `sed`, `awk`, `curl` change slowly
- **Standard libraries** — Python stdlib, Java stdlib, Go stdlib
- **Build tools** — Make, CMake (core features stable, extensions evolve)
- **Mature frameworks** — Django, Rails, Spring (breaking changes are rare and well-announced)

### Tier 3: Active Evolution (Qualify with version)
APIs and best practices shift meaningfully across versions.

- **Web frameworks** — React, Vue, Angular, Next.js, Nuxt
- **Package managers** — npm, yarn, pnpm, bun (rapidly differentiating)
- **Cloud services** — AWS, GCP, Azure (new services/changes monthly)
- **DevOps tools** — Kubernetes, Docker, Terraform (frequent releases)
- **Python ecosystem** — FastAPI, Pydantic, SQLAlchemy (active development)
- **Testing frameworks** — Vitest gaining on Jest, pytest plugins evolving
- **Build/bundle tools** — Vite, esbuild, Turbopack (active competition)

### Tier 4: Rapid Flux (Assume potentially stale)
Changes so fast that training data is almost certainly behind.

- **AI/ML libraries** — PyTorch, TensorFlow, transformers, LangChain
- **LLM tooling** — LangChain, LlamaIndex, CrewAI, AutoGen (weekly breaking changes)
- **AI APIs** — OpenAI, Anthropic, Google AI (model names, parameters, pricing change frequently)
- **Bleeding-edge frameworks** — Whatever is trending on Hacker News this month
- **Cryptocurrency/Web3** — Protocol changes, new chains, shifting standards
- **Newly released tools** — Anything less than 6 months old at the time of training

## Assessment Protocol

When asked about a technology, run this assessment:

### Step 1: Place on the Spectrum
Which tier does this technology fall into?

### Step 2: Estimate Drift Window
How long has it been since your training data was current?

| Tier | Acceptable Drift | Action |
|------|-------------------|--------|
| Tier 1 | 5+ years | State confidently |
| Tier 2 | 1-2 years | State with version note |
| Tier 3 | 3-6 months | Qualify heavily, recommend verification |
| Tier 4 | 1-2 weeks | Provide general patterns only, recommend checking docs |

### Step 3: Apply Appropriate Response Pattern

**Tier 1 Response** (confident):
> SQL `LEFT JOIN` returns all rows from the left table and matching rows from the right, with NULL for non-matching right columns.

**Tier 2 Response** (version-noted):
> In Python 3.10+, you can use `match`/`case` for structural pattern matching. In earlier versions, use `if`/`elif` chains or dictionary dispatch.

**Tier 3 Response** (qualified):
> Next.js uses the App Router (introduced in 13.4) with file-based routing in the `app/` directory. The exact configuration options and conventions may have evolved since my training data — check the current Next.js docs for the latest patterns, especially around Server Components and caching behavior.

**Tier 4 Response** (general patterns only):
> LangChain's API changes frequently. The general pattern is to create a chain that combines an LLM, a prompt template, and an output parser. However, the exact class names, import paths, and method signatures are likely different from what I last saw. I'd recommend checking the current LangChain documentation for the specific API.

## Drift Indicators

Watch for signals that your knowledge may be outdated:

### User-Side Signals
- User mentions a version number you don't recognize
- User's code uses import paths you haven't seen
- User references a feature you're unfamiliar with
- User's error messages reference unknown APIs or deprecation warnings

### Self-Assessment Signals
- You're generating code and feel uncertain about parameter names
- You know the library has undergone a major rewrite
- You're interpolating from similar libraries rather than recalling specifics
- The topic involves a tool that was very new at your knowledge cutoff

### Environmental Signals
- The user's package.json or requirements.txt shows versions beyond your knowledge
- Error messages suggest API changes
- The user corrects you on an API detail

## Recovery Patterns

When you determine your knowledge is likely stale:

### Pattern A: Provide the Stable Core
Focus on the aspects that haven't changed:

> I know Django's ORM still uses model classes with field definitions, and `objects.filter()` for queries. The specific new features in Django 5.x are beyond my training data, but the core query API is stable.

### Pattern B: Describe Intent, Not Implementation
Tell the user what to search for:

> You'll want a streaming response handler for the OpenAI API. The exact method name and parameter format may have changed — look for "streaming" in the current OpenAI Python SDK documentation.

### Pattern C: Offer Multiple Plausible Approaches
When you're unsure which version of an API is current:

> For Pydantic model validation, the approach depends on your version:
> - **Pydantic v1**: Use `@validator` decorators and `class Config`
> - **Pydantic v2**: Use `@field_validator` decorators and `model_config = ConfigDict(...)`
>
> Check your installed version with `pip show pydantic`.

### Pattern D: Leverage User's Environment
Ask the user to help bridge the knowledge gap:

> I'm not sure about the exact current API for this library. Could you share the output of `pip show <library>` or the relevant section of your package lockfile? That will help me give you version-appropriate guidance.

## Framework Volatility Index

A reference for common technologies and their change rate:

| Technology | Volatility | Last Major Disruption | Notes |
|------------|-----------|----------------------|-------|
| Python (language) | Low | 3.10 (match/case) | Backward compatible additions |
| JavaScript (language) | Low | ES6/2015 | Yearly incremental additions |
| React | Medium-High | 18 (concurrent), 19 (server components) | Fundamental paradigm shifts |
| Vue | Medium | 3.0 (composition API) | Migration tools provided |
| Django | Low-Medium | 4.0 (removed deprecated code) | Long deprecation cycles |
| FastAPI | Medium | Pydantic v2 migration | Tied to Pydantic's changes |
| Node.js | Medium | ESM migration, fetch built-in | Gradual modernization |
| TypeScript | Medium | 5.0 (decorators) | Generally additive |
| Kubernetes | High | Every minor version adds/changes APIs | Alpha/beta API churn |
| Terraform | High | Provider version dependencies | State format changes |
| PyTorch | High | 2.0 (compile), frequent API changes | Research-driven evolution |
| LangChain | Very High | Multiple breaking rewrites | Assume stale |

## Integration with Skill Seekers

This skill addresses the foundational problem that Skill Seekers was built to solve: LLMs cannot self-update their knowledge. Skill Seekers provides the mechanism to keep LLM knowledge current by:

- **Scraping current documentation** — Replaces stale training data with fresh content
- **Conflict detection** — Identifies where documentation and code have drifted apart
- **Always-current updates** — Re-running a scrape takes minutes, not months of retraining
- **Version-specific skills** — Each skill is generated from a specific version's documentation

When a Skill Seekers skill is available for a Tier 3 or Tier 4 technology, trust the skill's reference material over your training data. That's exactly what it's for.
