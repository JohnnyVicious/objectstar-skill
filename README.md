<p align="center">
  <img src="https://img.shields.io/badge/Platform-TIBCO%20Objectstar-blue?style=for-the-badge" alt="Platform">
  <img src="https://img.shields.io/badge/AI%20Enabled-Claude%20%7C%20Codex%20%7C%20Copilot-purple?style=for-the-badge" alt="AI Enabled">
  <img src="https://img.shields.io/badge/License-Apache%202.0-green?style=for-the-badge" alt="License">
</p>

<h1 align="center">Objectstar AI Skills</h1>

<p align="center">
  <strong>Accelerating Legacy Mainframe Modernization with AI-Powered Code Intelligence</strong>
</p>

<p align="center">
  <em>Production-ready skills for Claude Code, OpenAI Codex, and GitHub Copilot CLI</em>
</p>

---

## Executive Summary

**The Challenge:** TIBCO Objectstar (Object Service Broker) powers critical enterprise mainframe applications across financial services, insurance, and government sectors. With extended support ending **March 30, 2027**, organizations face an urgent modernization imperative—but Objectstar's unique 4GL paradigm creates significant knowledge barriers for migration teams.

**The Solution:** This repository provides **AI-augmented code intelligence** that transforms how development teams analyze, understand, and migrate Objectstar applications. By equipping leading AI coding assistants with deep Objectstar expertise, we dramatically accelerate the modernization lifecycle.

### Business Impact

| Metric | Traditional Approach | AI-Augmented Approach |
|--------|---------------------|----------------------|
| Code Comprehension | Days per module | Minutes per module |
| Pattern Recognition | Manual, error-prone | Automated, consistent |
| Migration Planning | Weeks of analysis | Hours with AI assistance |
| Knowledge Transfer | Limited to SMEs | Democratized across teams |

---

## Key Capabilities

### Code Analysis & Comprehension
- **Condition Quadrant Parsing** — Automatically interpret Objectstar's unique Y/N matrix branching logic
- **Scope Chain Tracking** — Trace LOCAL variable visibility across descendant rule hierarchies
- **Exception Flow Mapping** — Document FORALL iteration patterns and handler coverage

### Refactoring Intelligence
- **Pattern Library** — 11 proven patterns for CRUD operations, transactions, screen handling, and batch processing
- **Anti-Pattern Detection** — Identify 5 critical anti-patterns that impact performance and maintainability
- **Browse Mode Optimization** — Recommend read-only access patterns for performance gains

### Migration Acceleration
- **Java Mapping Framework** — Complete element-by-element translation guidance
- **Data Type Conversion** — Semantic and syntax type mapping to modern equivalents
- **Strategy Playbooks** — Three migration approaches: automated tooling, rule-by-rule rewrite, Strangler Fig

---

## Platform Support

<table>
<tr>
<td align="center" width="33%">
<h3>Claude Code</h3>
<code>.claude/skills/ostar/</code>
<br><br>
<strong>Comprehensive skill</strong> with full syntax reference, pattern library, migration guides, and 100+ built-in function documentation
</td>
<td align="center" width="33%">
<h3>OpenAI Codex</h3>
<code>.codex/skills/objectstar/</code>
<br><br>
<strong>Optimized skill</strong> with syntax essentials, critical pitfalls, and migration strategy overview
</td>
<td align="center" width="33%">
<h3>GitHub Copilot</h3>
<em>Planned</em>
<br><br>
<strong>Coming soon</strong> — Copilot CLI integration for inline Objectstar assistance
</td>
</tr>
</table>

---

## Documentation Coverage

```
┌─────────────────────────────────────────────────────────────────┐
│                    OBJECTSTAR KNOWLEDGE BASE                     │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐          │
│  │   SYNTAX     │  │   PATTERNS   │  │  MIGRATION   │          │
│  │   Reference  │  │   Library    │  │   Guides     │          │
│  ├──────────────┤  ├──────────────┤  ├──────────────┤          │
│  │ • Data Types │  │ • CRUD Ops   │  │ • Java Maps  │          │
│  │ • Tables     │  │ • FORALL     │  │ • Type Conv  │          │
│  │ • Statements │  │ • Screens    │  │ • Strategies │          │
│  │ • Operators  │  │ • Batch Jobs │  │ • Exceptions │          │
│  │ • Exceptions │  │ • Anti-ptrns │  │ • FORALL→For │          │
│  └──────────────┘  └──────────────┘  └──────────────┘          │
│                                                                 │
│  ┌──────────────┐  ┌──────────────────────────────────┐        │
│  │   TOOLS      │  │         QUICK REFERENCE          │        │
│  │   Reference  │  │                                  │        │
│  ├──────────────┤  │  • Rule structure templates      │        │
│  │ • String Ops │  │  • Exception hierarchy tree      │        │
│  │ • Date/Time  │  │  • Reserved keywords list        │        │
│  │ • Math Funcs │  │  • Analysis checklists           │        │
│  │ • System     │  │  • Refactoring guidelines        │        │
│  └──────────────┘  └──────────────────────────────────┘        │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## The Objectstar Challenge

Objectstar applications present unique modernization challenges that require specialized knowledge:

| Objectstar Paradigm | Modern Equivalent | Complexity |
|--------------------|--------------------|------------|
| Condition Quadrants (Y/N matrix) | If/else, switch expressions | High — No direct mapping |
| LOCAL Variable Scope (descendant visibility) | Lexical scoping | High — Breaks encapsulation |
| Parameterized Tables | Filtered queries, composite keys | Medium — Semantic translation |
| TRANSFERCALL (no-return invocation) | Method calls with returns | High — Control flow redesign |
| FORALL with UNTIL exception | For-each loops | Medium — Pattern conversion |

**These skills encode the expertise needed to navigate these challenges systematically.**

---

## Quick Start

### Claude Code
```bash
# The skill auto-activates when working with .osr/.osb files
# or code containing Objectstar patterns

claude "Analyze this Objectstar rule and identify migration concerns"
```

### OpenAI Codex
```bash
# Skill activates on Objectstar-related queries
codex "Explain this FORALL pattern and suggest Java equivalent"
```

---

## Repository Structure

```
objectstar-skill/
├── .claude/
│   └── skills/
│       └── ostar/
│           ├── SKILL.md              # Skill definition & workflow
│           └── references/
│               ├── syntax.md         # Complete language reference
│               ├── patterns.md       # Code patterns & anti-patterns
│               ├── migration.md      # Java migration strategies
│               └── tools.md          # Built-in functions (100+)
│
├── .codex/
│   └── skills/
│       └── objectstar/
│           ├── SKILL.md              # Skill metadata
│           └── references/
│               ├── ObjectStar_Syntax.md
│               ├── ObjectStar_BuiltInTools.md
│               ├── ObjectStar_Pitfalls.md
│               └── ObjectStar_MigrationGuide.md
│
├── CLAUDE.md                         # Claude Code guidance
├── REVIEW_SUMMARY.md                 # Skills comparison & analysis
├── docs/
│   ├── SOURCES.md                    # External references & history
│   ├── TO_VERIFY.md                  # Items needing verification
│   └── 6.0-programming-in-rules/     # Official TIBCO documentation (markdown)
│       ├── 00-INDEX.md               # Document structure
│       ├── 02-rule-composition.md    # Rule components, condition quadrants
│       ├── 04-action-statements.md   # CALL, GET, FORALL, etc.
│       ├── 05-exception-handling.md  # ON/SIGNAL/UNTIL
│       └── ... (12 files total)
├── LICENSE                           # Apache 2.0
└── README.md
```

---

## Roadmap

- [x] Claude Code skill — Comprehensive implementation
- [x] OpenAI Codex skill — Optimized implementation
- [ ] GitHub Copilot CLI skill
- [ ] COBOL interop patterns (Objectstar ↔ COBOL bridges)

---

## Contributing

Contributions welcome. Please ensure changes to domain knowledge are reflected in both `.claude/` and `.codex/` skill directories to maintain cross-platform consistency.

---

## License

Apache 2.0 — See [LICENSE](LICENSE) for details.

---

<p align="center">
  <strong>Modernize with Confidence</strong><br>
  <em>AI-powered expertise for your Objectstar migration journey</em>
</p>
