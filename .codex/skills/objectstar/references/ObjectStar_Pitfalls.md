# ObjectStar Common Pitfalls

This document summarizes recurring problems in legacy ObjectStar (OSB) code that should be avoided or flagged during analysis or refactoring.

## 1. Overuse of ON ERROR
**Issue:** Catch-all `ON ERROR` blocks used to suppress or redirect flow, often without proper rollback or message logging.

**Impact:** Silences genuine failures, risks inconsistent data, causes debugging difficulties.

**Recommendation:** Handle specific exceptions (e.g., `GETFAIL`, `LOCKFAIL`) locally. Use `ON ERROR` only for unexpected/unrecoverable states.

---

## 2. Nested FORALL Loops on Large Tables
**Issue:** Loops within loops over non-indexed or large datasets.

**Impact:** Performance degradation, excessive locking, possible transaction overflow.

**Recommendation:** Restructure logic to avoid deep nested iteration. Use indexes and WHERE clauses. Pre-filter using session tables.

---

## 3. Missing COMMITs in Long Transactions
**Issue:** Batch jobs process thousands of rows without committing.

**Impact:** Intent list overflows (`COMMITLIMIT`), memory spikes, excessive rollback costs on failure.

**Recommendation:** Commit in chunks (e.g., every 1000 records). Catch and handle `COMMITLIMIT` exceptions if needed.

---

## 4. Hardcoded Constants and DSNs
**Issue:** Embedding file names, screen names, or magic values in rule logic.

**Impact:** Reduces portability, maintainability, and complicates migration.

**Recommendation:** Move such values to config/control tables. Use rule parameters where possible.

---

## 5. Implicit Global State
**Issue:** Shared scratch variables or screen tables used for data transfer without parameterization.

**Impact:** Hidden dependencies between rules, side effects, testing difficulties.

**Recommendation:** Pass values explicitly using rule parameters or session tables.

---

## 6. Reliance on Locking Side Effects
**Issue:** GET used for its side effect of acquiring a lock on a nonexistent key.

**Impact:** Behavior difficult to replicate in RDBMS; may lead to migration bugs.

**Recommendation:** Avoid using GET purely to force a lock. Use explicit lock flags or redesign logic.

---

## 7. Repeated Code Blocks
**Issue:** Business logic (e.g., tax calculation) duplicated in many rules.

**Impact:** Inconsistent behavior across the system. Hard to update.

**Recommendation:** Refactor to shared utility rules or libraries. Use CALLs to encapsulate logic.

