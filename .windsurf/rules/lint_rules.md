# LINT_RULES.md — RimWorld 1.6 XML + C# Mods

This document defines **mandatory lint rules** for RimWorld **1.6 mods** covering both **XML** and **C# (Harmony)**.

These rules are intended for:
- Automated tooling (CI, pre-commit hooks)
- AI agent validation
- Human review consistency

A lint failure should be treated as a **blocking error** unless explicitly waived.

---

## 1. Global Rules

### LINT-001 — Unique DefNames
- ❌ Duplicate `defName` values
- ✅ Every DefName must be unique and mod-prefixed

---

### LINT-002 — RimWorld 1.6 Only
- ❌ Version checks for ≤1.5
- ❌ Legacy fallback XML
- ✅ Assume Core 1.6 schema

---

## 2. XML Structure Rules

### LINT-010 — Valid XML
- ❌ Malformed XML
- ❌ Empty Def nodes
- ✅ Human-readable formatting

---

### LINT-011 — One Primary Def Per File
- ❌ Multiple unrelated Defs in one file
- ✅ Split by Def type or feature

---

## 3. Patch Rules

### LINT-020 — Explicit XPath Targets
- ❌ XPath starting with `//Defs`
- ❌ Broad selectors without `defName`
- ✅ Target specific DefNames

---

### LINT-021 — No Full Def Replacement
- ❌ `PatchOperationReplace` on entire Def nodes
- ✅ Additive or targeted patches only

---

### LINT-022 — Safe Failure Mode
- ❌ Patches that hard-error if path missing
- ✅ Patches that fail gracefully

---

## 4. Localization Rules

### LINT-030 — No Hardcoded Strings
- ❌ Player-facing text in XML Defs
- ✅ Use keyed localization

---

### LINT-031 — English Required
- ❌ Missing English keys
- ✅ Complete English localization

---

## 5. Balance Rules

### LINT-040 — No Strictly Better Items
- ❌ Items outperform Core equivalents without cost
- ✅ Clear trade-offs

---

### LINT-041 — Extreme Values Justified
- ❌ Outlier stats without explanation
- ✅ Vanilla-based comparisons

---

## 6. Safety Rules

### LINT-050 — Core Integrity
- ❌ Editing Core files directly
- ❌ Patching forbidden Core systems

Never Touch:
- GameRules
- ScenarioDefs
- Core research tree

---

## 7. AI-Specific Rules

### LINT-060 — No Invented Fields
- ❌ Undocumented or speculative XML fields
- ✅ Fields verified in Core 1.6

---

### LINT-061 — Explain Decisions
- ❌ Silent balance or behavior changes
- ✅ Clear reasoning provided

---

### LINT-070 — XPath must not start with "/Defs/"
- ❌ XPath starting with "/Defs"
- ✅ Target specific DefNames

---

## 8. C# / Harmony Rules

### LINT-100 — No Exceptions Escaping Patches
- ❌ Exceptions thrown out of Harmony `Prefix`/`Postfix`/`Transpiler`
- ✅ Guard nulls and use `try/catch` inside patches; recover safely

---

### LINT-101 — Patch the Smallest Surface Area
- ❌ Patching broad/hot methods when a narrower hook exists
- ✅ Patch the smallest stable method that achieves the effect

---

### LINT-102 — Avoid Transpilers Unless Necessary
- ❌ Transpilers used when a Prefix/Postfix or smaller hook would work
- ✅ Transpilers only when no stable alternative exists

---

### LINT-103 — Transpilers Must Fail Safely
- ❌ Transpiler that assumes an IL pattern always matches
- ✅ If pattern not found, leave method unmodified and log once

---

### LINT-110 — No Reflection in Hot Paths
- ❌ `AccessTools.*`, `Type.GetType`, `GetMethod`, `Invoke`, etc. inside tick/UI hot paths
- ✅ Resolve reflection once and cache `MethodInfo`/delegates

---

### LINT-111 — No Def Scans in Hot Paths
- ❌ Scanning `DefDatabase<T>.AllDefs` in tick/UI hot paths
- ✅ Cache def-derived results outside hot paths

---

### LINT-112 — No LINQ/Closures in Hot Paths
- ❌ LINQ, iterators, or closure allocations in `Tick`/`CompTick*`/UI draw paths
- ✅ Use allocation-light loops and reuse cached collections when justified

---

### LINT-120 — No Log Spam
- ❌ Logging every tick / every draw / in tight loops
- ✅ Use `Log.ErrorOnce` / `Log.WarningOnce`; gate diagnostics behind dev mode/settings

---

### LINT-121 — Avoid Per-Frame String Building in UI
- ❌ Repeated string concatenation/formatting/translation each draw for stable text
- ✅ Cache stable labels/tooltips; invalidate only when inputs change

---

### LINT-130 — Save Data Must Be Versioned When Structure Changes
- ❌ Changing serialized structure without a migration path
- ✅ Add version fields and perform idempotent migrations on load

---

### LINT-131 — ExposeData Must Be Safe
- ❌ Throwing exceptions during `ExposeData`
- ✅ Use defaults/null guards; log once and sanitize

---

### LINT-140 — Optional Integrations Must Degrade Gracefully
- ❌ Hard dependency on other mods without explicit requirement
- ✅ Feature-detect and fail softly if integration targets are missing/changed

---

### LINT-141 — Cache Integration Reflection
- ❌ Re-resolving integration types/methods repeatedly at runtime
- ✅ Resolve once and cache; avoid reflection in hot paths

---

### LINT-150 — No Unsafe Threading
- ❌ Mutating Verse/Unity game state from background threads
- ✅ Background work (if any) must be pure computation; apply results on the main thread safely

**End of LINT_RULES.md**

