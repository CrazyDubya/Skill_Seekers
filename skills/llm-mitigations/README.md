# LLM Weakness Mitigation Skills

Five skills designed to compensate for the core weaknesses that Skill Seekers identifies in LLMs.

## Weaknesses Identified

| # | Weakness | How Skill Seekers Exposes It |
|---|----------|------------------------------|
| 1 | **Knowledge Cutoff** | LLMs generate outdated patterns (e.g., React class components); the tool exists to provide "always current" docs |
| 2 | **Context Window Limits** | Router skills exist specifically to avoid context overflow; token efficiency is a design constraint |
| 3 | **Documentation-Code Drift** | Conflict detection finds 4 types of discrepancies between what docs say and what code does |
| 4 | **Generic Domain Knowledge** | Without structured skills, LLMs give "generic, outdated answers" instead of framework-specific guidance |
| 5 | **API Hallucination** | LLMs confidently generate plausible-but-nonexistent APIs; structured reference files force grounding |
| 6 | **No Persistent Memory** | Context is lost between sessions; skills provide persistent, loadable knowledge |
| 7 | **Data Quality Sensitivity** | "70% of RAG time is preprocessing"; smart categorization and AI enhancement address noisy input |

## The Five Mitigation Skills

| Skill | File | Mitigates |
|-------|------|-----------|
| **Uncertainty Calibration** | `01-uncertainty-calibration.md` | Hallucination, overconfidence, stale knowledge |
| **Version-Aware Coding** | `02-version-aware-coding.md` | Knowledge cutoff, stale patterns, version mismatch |
| **API Hallucination Prevention** | `03-api-hallucination-prevention.md` | Hallucination, documentation-code drift, fabricated APIs |
| **Context Window Optimization** | `04-context-window-optimization.md` | Context overflow, information loss, attention degradation |
| **Knowledge Currency Assessment** | `05-knowledge-currency-assessment.md` | Knowledge cutoff, documentation drift, stale recommendations |

## Usage

Load any of these files as project context or skills in your LLM of choice:

- **Claude**: Add to a Claude Project as a knowledge file
- **Cursor/Windsurf**: Place in project root or reference in settings
- **Claude Code**: Include in CLAUDE.md or reference from it
- **ChatGPT**: Upload as a file to a GPT or conversation

## Relationship to Skill Seekers

These skills address the same problems from the opposite direction:

- **Skill Seekers** provides **external grounding data** (fresh docs, verified APIs, structured references) to prevent LLM failures
- **These mitigation skills** provide **internal behavioral patterns** that help LLMs fail more gracefully when grounding data isn't available

They are complementary. An LLM with both a Skill Seekers-generated React skill *and* the uncertainty calibration skill will produce better results than either alone — it has current data to draw from, and it knows how to handle the boundaries of that data.
