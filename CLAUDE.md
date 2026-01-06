# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository Purpose

This is a documentation-only repository providing AI assistant skills for analyzing, refactoring, and migrating TIBCO Objectstar (Object Service Broker) legacy mainframe applications. There is no executable code—only markdown documentation organized as skills for different AI assistants.

## Repository Structure

```
.claude/skills/ostar/     # Claude Code skill (primary)
  ├── SKILL.md            # Main skill definition and workflow
  ├── syntax.md           # Complete Objectstar syntax reference
  ├── patterns.md         # Code patterns and anti-patterns
  ├── migration.md        # Java migration strategies
  └── tools.md            # Built-in functions reference

.codex/skills/objectstar/ # OpenAI Codex skill variant
  ├── SKILL.md            # Codex skill metadata
  ├── object_star_syntax.md
  ├── object_star_pitfalls.md
  └── object_star_migration_guide.md
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

### Skill Activation Triggers

The Claude skill activates on:
- `.osr` or `.osb` file extensions
- Code containing Objectstar patterns: condition quadrants, FORALL loops, TRANSFERCALL, parameterized tables, LOCAL declarations
