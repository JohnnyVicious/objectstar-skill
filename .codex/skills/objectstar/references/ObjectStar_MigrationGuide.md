# ObjectStar Migration Guide

Guidance for migrating TIBCO ObjectStar applications to modern platforms such as Java, C#, or REXX.

## 1. Migration Preparation
- Inventory all ObjectStar rules, screens, and tables from the MetaStore
- Classify rules: OTP screen logic vs. batch job logic
- Extract dependencies (CALL, EXECUTE, TRANSFERCALL chains)
- Document exception flows and transaction boundaries

## 2. Target Platform Mapping
| ObjectStar Concept | Target Equivalent | Notes |
|--------------------|-------------------|-------|
| RULE               | Function / Method | Include clear parameterization |
| GET                | SQL SELECT        | Ensure transaction context is preserved |
| FORALL             | Loop with query   | Prefer indexed iteration |
| EXCEPTION BLOCK    | try/catch         | Map each ON clause to catch block |
| SCREEN TABLES      | DTO / ViewModel   | Model form data explicitly |
| SESSION TABLES     | Temp tables / in-memory collections | Scope to transaction |
| TRANSFERCALL       | Controller redirect / method exit | Ensure call chain integrity |

## 3. Migration Strategy
- **Phase 1:** Refactor ObjectStar code for clarity and modularity (before porting)
- **Phase 2:** Migrate shared business logic first (pure computation rules)
- **Phase 3:** Migrate batch logic (replace FORALL with DB cursors or streaming APIs)
- **Phase 4:** Reimplement screens in UI layer (web, terminal, etc.)
- **Phase 5:** Map transactions and error handling carefully

## 4. Challenges to Expect
- Typeless variables in ObjectStar require type inference in target language
- Implicit intent list behavior must be modeled explicitly
- Dynamic table references (e.g., `TABLES.NAME = "XYZ"; GET TABLES.NAME`) may need refactoring
- Parameterized tables may require separate schemas or tenant-aware logic

## 5. Tooling Suggestions
- Build a parser or use regex to extract rule names, calls, and table access
- Use code classification tags (e.g., `@batch`, `@screen`, `@utility`) to sort rule types
- Maintain an exception mapping dictionary (ObjectStar → target exceptions)

## 6. Verification
- Define behavioral tests for existing rules before migration
- Use log comparison or output parity (e.g., spool files, reports)
- Ensure transactional semantics (commit/rollback) are preserved

## 7. Recommended Refactorings Before Migration
- Eliminate catch-all `ON ERROR` blocks
- Break up large monolithic rules
- Replace screen navigation logic (`DISPLAY`, `TRANSFERCALL`) with explicit state transitions
- Remove or isolate direct screen references in shared logic rules
- Externalize hardcoded values to config tables or parameters

