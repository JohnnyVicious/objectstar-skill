# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository Purpose

This is a documentation-only repository providing AI assistant skills for analyzing, refactoring, and migrating TIBCO Objectstar (Object Service Broker) legacy mainframe applications. There is no executable code—only markdown documentation organized as skills for different AI assistants.

## Repository Structure

```
.claude/skills/ostar/           # Claude Code skill: reviewing-objectstar
  ├── SKILL.md                  # Main skill definition and workflow
  └── references/
      ├── syntax.md             # Complete Objectstar syntax reference
      ├── patterns.md           # Code patterns and anti-patterns
      ├── migration.md          # Java migration strategies
      └── tools.md              # Built-in functions reference

.codex/skills/objectstar/       # OpenAI Codex skill: analyzing-objectstar
  ├── SKILL.md                  # Codex skill metadata
  └── references/
      ├── ObjectStar_Syntax.md
      ├── ObjectStar_Pitfalls.md
      └── ObjectStar_MigrationGuide.md

REVIEW_SUMMARY.md               # Skills comparison, issues, and fix status
docs/                           # Reference documentation and external sources
```

## Development Guidelines

### Parallel Skill Maintenance

When updating domain knowledge, changes should be reflected in **both** `.claude/` and `.codex/` directories to maintain consistency across AI platforms. The Claude skill is more comprehensive; the Codex skill is a condensed reference.

### Objectstar Language Context

Objectstar is a 4GL mainframe platform with unique characteristics documented in the skills:
- **Condition quadrants** (Y/N matrix logic) instead of IF/THEN/ELSE
- **Rules** instead of programs/functions
- **Tables** for everything (screens, data, sessions)
- **LOCAL variables** with non-standard scoping (visible in descendant rules)
- **No traditional loops**—uses FORALL with UNTIL exception patterns

Extended support ends March 30, 2027, making migration documentation critical.

### Skill Naming Convention

Skills follow Anthropic's gerund naming convention:
- Claude skill: `reviewing-objectstar`
- Codex skill: `analyzing-objectstar`

### Skill Activation Triggers

The Claude skill activates on:
- `.osr` or `.osb` file extensions
- Code containing Objectstar patterns: condition quadrants, FORALL loops, TRANSFERCALL, parameterized tables, LOCAL declarations

## Lessons Learned

Insights from previous sessions that may help future work:

### Always Verify Domain Claims

When one skill document contradicts another, verify against official sources before assuming which is correct. Example: The Codex skill originally showed `IF/ENDIF` syntax, but [TIBCO's official documentation](https://docs.tibco.com/pub/object_service_broker/5.2.0_august_2010/html/tib_osb_quick_reference/qref.htm) confirmed Objectstar has **no IF/THEN/ELSE** — it uses condition quadrants exclusively.

### Anthropic Skill Authoring Guidelines

Key requirements from [Anthropic's best practices](https://platform.claude.com/docs/en/agents-and-tools/agent-skills/best-practices):
- **Naming**: Use gerund form (verb + -ing), lowercase with hyphens
- **Description**: Write in third person, be specific about when to use
- **Structure**: SKILL.md under 500 lines, reference files one level deep
- **File paths**: Use forward slashes, keep references in `references/` subdirectory

### Use Conventional Commits

This repository uses [Conventional Commits](https://www.conventionalcommits.org/en/v1.0.0/). Format:

```
<type>(scope): short description

Optional longer description.
```

Common types: `fix`, `feat`, `docs`, `refactor`, `chore`

Example:
```
fix(skills): rename skills to follow gerund naming convention
```

### Track Review Status

The `REVIEW_SUMMARY.md` file tracks:
- Comparison between Claude and Codex skills
- Issues found vs Anthropic guidelines
- Fix status (completed vs open)
- Remaining recommendations

Consult this file before making changes to understand what's been reviewed and what still needs work.

## External References

### TIBCO Official Documentation

These are the authoritative sources for Objectstar/OSB language details:

| Resource | URL | Notes |
|----------|-----|-------|
| **Quick Reference** | [tib_osb_quick_reference](https://docs.tibco.com/pub/object_service_broker/5.2.0_august_2010/html/tib_osb_quick_reference/qref.htm) | Best for verifying keywords, operators, statements |
| **Programming in Rules** | [tib_osb_processing.pdf](https://docs.tibco.com/pub/object_service_broker/6.0.0_july_2012/doc/pdf/tib_osb_processing.pdf) | Comprehensive rules language guide (large PDF) |
| **Getting Started** | [tib_osb_getting_started.pdf](https://docs.tibco.com/pub/object_service_broker/5.2.0_august_2010/pdf/tib_osb_getting_started.pdf) | Introduction and tutorials |
| **Managing Data** | [tib_osb_mdata.pdf](https://docs.tibco.com/pub/object_service_broker/6.0.0_july_2012/doc/pdf/tib_osb_mdata.pdf) | Table types, data operations |
| **Product Documentation** | [TIBCO OSB 6.0.0](https://docs.tibco.com/products/tibco-object-service-broker) | Full documentation index |
| **End of Support Notice** | [Support Article](https://support.tibco.com/s/article/Tibco-KnowledgeArticle-Article-44626) | Extended support ends March 30, 2027 |

### Migration Resources

| Resource | URL | Notes |
|----------|-----|-------|
| **FreeSoft ObjectStar Conversion** | [freesoftus.com](https://freesoftus.com/services/application-code-conversion/objectstar-conversion/) | Migration tooling vendor, discusses LOCAL typing issues |
| **FreeSoft ObjectStar Overview** | [objectstar page](https://freesoftus.com/services/application-code-conversion/objectstar/) | General migration considerations |

### Anthropic Skill Development

| Resource | URL | Notes |
|----------|-----|-------|
| **Skill Authoring Best Practices** | [platform.claude.com](https://platform.claude.com/docs/en/agents-and-tools/agent-skills/best-practices) | Naming, structure, progressive disclosure |
| **Claude Code Skills Docs** | [code.claude.com](https://code.claude.com/docs/en/skills) | How skills work in Claude Code |
| **Skills GitHub Repo** | [github.com/anthropics/skills](https://github.com/anthropics/skills) | Official skill examples |

### Verification Notes

- **Confirmed**: Objectstar has no IF/THEN/ELSE keywords (verified via Quick Reference)
- **Confirmed**: Conditions evaluate to **Y or N** (verified via [TIBCO Glossary](https://docs.tibco.com/pub/object_service_broker/5.2.0_august_2010/html/tib_osb_getting_started/glossary.htm))
- **Confirmed**: Action Sequence Numbers determine execution order (verified via TIBCO Glossary)
- **Confirmed**: Uses FORALL with UNTIL exception patterns for iteration
- **Confirmed**: Extended support ends March 30, 2027
- **Partially verified**: The Y/N condition quadrant format — core concepts (Y/N logic, action sequence numbers) are official; the exact visual formatting (pipe separators, dashed lines, column layout) is reconstructed

See `docs/TO_VERIFY.md` for full verification details and sources.
