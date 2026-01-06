# AGENTS.md — RimWorld 1.6 C# and XMLModding (Performance & Compatibility)


This document defines **agent behavior, rules, and best practices** for working on a **RimWorld 1.6 C# mod** (typically Harmony-based). It is intended for both human contributors and AI agents assisting with code generation, refactoring, debugging, or review.

**Primary goals**
- Performance: minimize per-tick/per-frame overhead, allocations, and reflection cost
- Compatibility: maximize coexistence with other mods and minimize patch conflicts
- Save safety: avoid breaking saves and handle upgrades/migrations cleanly

**Scope**
- RimWorld **1.6.x**
- **C#** and **Harmony** patching (XML defs and PatchOperations are also covered)
- Ignore legacy compatibility (≤1.5) unless explicitly asked

---

## 1. Modding Assumptions (RimWorld 1.6)

Agents must assume:
- RimWorld version **1.6.x**
- Game logic is primarily **single-threaded**; most Verse/Unity objects are not thread-safe
- Mods load in a dynamic environment with unknown load order and other mods applying patches

Agents must NOT:
- Hard-depend on private implementation details when a stable public API exists
- Assume other mods are absent
- Assume your patches run first/last unless you set explicit Harmony priority/ordering

---

## 2. Folder / Project Structure (Recommended)

Agents should prefer a structure compatible with typical RimWorld distribution:

```
/Mods/<ModName>/
 ├─ About/
 │   └─ About.xml
 ├─ Assemblies/
 │   └─ <ModName>.dll
 ├─ Defs/                       (optional)
 ├─ Patches/                    (optional)
 ├─ Languages/
 │   └─ English/
 │       └─ Keyed/
 ├─ Textures/
 ├─ Sounds/
 └─ Source/                     (optional, repo-only)
     ├─ <ModName>.csproj
     └─ ...
```

Rules:
- Prefer a single primary assembly unless there is a clear need for multiple
- Keep *runtime* files under the mod root and *build-only* artifacts under `Source/`
- If you ship `Source/`, ensure it is not required for the mod to run

---

## 3. Compatibility Rules (Strict)

### 3.1 “Do No Harm” Defaults

Agents must:
- Prefer **additive** behavior over replacing core/other-mod behavior wholesale
- Avoid patching hot methods broadly when a narrower hook exists
- Fail safely if an optional integration target is missing

Agents must NOT:
- Patch entire types or large call chains “just in case”
- Rely on brittle IL patterns without a maintenance plan
- Throw exceptions in patched methods (catch/guard instead)

### 3.2 Patch Surface Area

Rules:
- Patch the **smallest** method that achieves the effect
- Prefer `Postfix` over `Prefix` if you can simply observe/augment results
- If you must override behavior, use `Prefix` returning `false` only when necessary
- Prefer `Transpiler` only when there is no stable alternative

### 3.3 Harmony Identity and Ordering

Agents must:
- Use a unique Harmony ID (e.g. `com.<author>.<modname>`)
- Use explicit `HarmonyPriority` / ordering attributes only when justified

Agents should:
- Document why ordering is needed (e.g. “must run after X to read its modified result”)
- Treat ordering as a compatibility hazard: every order constraint increases conflict risk

---

## 4. Performance Rules (Strict)

### 4.1 Avoid Work in Hot Paths

Hot paths in RimWorld commonly include:
- Tick updates (`Tick`, `CompTick`, `CompTickRare`, `CompTickLong`)
- `Gizmo`/UI drawing (`DoWindowContents`, `OnGUI`, `GizmoOnGUI`)
- Frequent queries during pathing/combat/joy selection

Agents must:
- Avoid per-tick LINQ, closures, iterator allocations, and repeated `ToList()`/`ToArray()` in hot paths
- Avoid reflection in hot paths (cache delegates/MethodInfo once)
- Avoid repeated scanning of `DefDatabase<T>.AllDefs` at runtime; cache results

Agents should:
- Prefer event-like hooks over polling where possible
- Prefer `TickRare`/`TickLong` over `Tick` when behavior allows

### 4.2 Allocation Discipline (GC)

Rules:
- No allocations in per-tick loops unless proven negligible
- Do not allocate strings repeatedly for logging, labels, or UI tooltips
- Cache commonly used computed values (but avoid unbounded caches)

Recommended practices:
- Precompute static strings and `TaggedString` where appropriate
- Reuse lists via pooling patterns only when proven necessary (do not over-engineer)

### 4.3 Caching and Invalidation

Agents must:
- Keep caches scoped correctly:
  - `static` caches for def-level / global immutable data
  - `MapComponent` or per-map structures for map-scoped data
  - per-pawn caches only if lifecycle/invalidation is clear

Agents should:
- Invalidate caches when the underlying data can change (e.g. map removal, pawn death/despawn)
- Prefer “compute once at load” for data derived from defs

---

## 5. RimWorld Lifecycle / Initialization

Agents must:
- Avoid heavy work in static constructors that can delay startup
- Avoid touching `Current`/`Find` state before it exists

Agents should:
- Perform def-derived initialization at a safe time (after defs are loaded)
- Prefer delayed initialization for expensive scans

Notes:
- If you need to scan loaded types/defs for compatibility, do it once and cache the result
- Initialization order is a frequent compatibility pitfall; guard against null game state

---

## 6. Harmony Patching Guidelines (Deep Dive)

### 6.1 Target Selection

Agents must:
- Target stable public/protected methods when possible
- Avoid patching compiler-generated methods or lambdas unless unavoidable

Agents should:
- Use Harmony helpers (e.g. `AccessTools`) to locate members more robustly
- Prefer patching a single “decision point” rather than multiple downstream consequences

### 6.2 Prefix/Postfix Standards

Rules:
- `Prefix`: validate inputs, short-circuit safely, or adjust arguments with minimal side effects
- `Postfix`: adjust return values or append behavior; do not assume original method succeeded unless you can verify

### 6.3 Transpiler Standards

Agents must:
- Use transpilers only when necessary
- Keep transpilers small, well-structured, and resilient

Agents should:
- Match IL with conservative patterns (avoid relying on exact local indices when possible)
- Fail safely: if the expected pattern is not found, log once and leave the method unmodified

### 6.4 Patch Safety

Rules:
- Guard against nulls and missing state (maps can be null, pawns can be destroyed)
- Avoid throwing exceptions from patches
- Use `Log.ErrorOnce` / `Log.WarningOnce` for recurring issues

---

## 7. Optional Mod Integrations (Compatibility-First)

Agents must:
- Treat other mods as optional unless explicitly required
- Keep integration code isolated (separate classes/namespaces) so failures do not break base functionality

Agents should:
- Detect presence of other mods using RimWorld/Verse-provided mod listing facilities
- When integrating via reflection, cache `Type`, `MethodInfo`, and delegates once
- Prefer “best-effort” integration: if the other mod changes, degrade gracefully

Guideline:
- If an integration requires patch ordering, document why and what breaks without it

---

## 8. Save Compatibility & Versioning

### 8.1 General Rules

Agents must:
- Assume players will add/remove mods mid-save
- Keep `defName`s and database identifiers stable once released

Agents should:
- Store minimal necessary data in saves
- Prefer deriving data from defs at load time instead of persisting redundant fields

### 8.2 ExposeData Practices

Rules:
- Implement save data using standard `ExposeData` patterns (`Scribe_Values`, `Scribe_Collections`, etc.)
- Handle missing data safely (default values, null checks)

Migration guidance:
- When changing serialized structure, include version fields and migrate on load
- Treat old saves as first-class citizens; migration should be idempotent and safe

### 8.3 Removal / Missing Defs

Agents must:
- Handle cases where a saved reference points to a def that no longer exists

Agents should:
- Avoid storing direct references when an identifier can be stored more safely
- Validate and sanitize loaded data before using it in gameplay logic

---

## 9. Logging, Diagnostics, and User Experience

Agents must:
- Avoid log spam (it hurts performance and user trust)

Agents should:
- Use once-only logging for repeated failures
- Gate verbose diagnostics behind dev mode or a mod setting
- Make errors actionable: include what failed and what the mod will do instead

UI/UX rules:
- Do not block the UI thread with expensive computations during drawing
- Avoid building large strings every frame; cache UI text where possible

---

## 10. Threading and Asynchrony (Conservative)

Agents must:
- Assume Verse/Unity objects are not thread-safe
- Never mutate game state from background threads

Agents may:
- Offload pure computation to a background thread only if results are applied on the main thread safely

Agents should:
- Prefer avoiding threading entirely unless performance profiling proves it is necessary

---

## 11. API Stability & Defensive Coding

Agents must:
- Avoid depending on unstable internal behavior when a stable alternative exists

Agents should:
- Use feature detection (presence of type/method) instead of version checks
- Keep reflection wrappers narrow and cached
- Treat exceptions as bugs: fix root causes rather than suppressing broadly

---

## 12. Performance Review Checklist (Before Commit)

Agents should verify:
- [ ] No reflection or `DefDatabase` scans in hot paths
- [ ] No per-tick LINQ/allocations in frequently-called patches
- [ ] Caches have clear scope and invalidation strategy
- [ ] Logging is not spammy and is mostly once-only for recurring issues
- [ ] Transpilers fail safely if pattern changes

---

## 13. Compatibility Review Checklist (Before Commit)

Agents should verify:
- [ ] Patches are minimal and target narrow methods
- [ ] No hard dependency on other mods unless explicitly required
- [ ] Optional integrations degrade gracefully if missing or changed
- [ ] Save data changes include migrations and defaults
- [ ] Harmony ID is unique and patch ordering is justified

---

## 14. AI-Assisted Generation Rules (C#)

AI agents must:
- Ask for clarification when requirements are ambiguous
- Avoid inventing RimWorld/Verse APIs: verify names/usage before generating code
- Prefer simple, maintainable patches over clever/fragile solutions

AI agents should:
- Propose profiling steps when performance work is requested
- Provide a rollback plan for risky patches

---

## 15. Concrete Harmony Templates (Scaffolding Patterns)

This section provides recommended *patterns* for writing Harmony patches that are safe, compatible, and easy to maintain.

Agents must:
- Keep patches small and narrowly targeted
- Guard and fail safely (no exceptions leaving patched methods)
- Prefer readability over cleverness

### 15.1 Mod Initialization (Patch Bootstrap)

Preferred pattern:
- Patch once at startup
- Avoid heavy work in the bootstrap (patching only)

Template:
```csharp
using HarmonyLib;
using Verse;

[StaticConstructorOnStartup]
internal static class MyMod_HarmonyBootstrap
{
    private const string HarmonyId = "com.author.mymod";

    static MyMod_HarmonyBootstrap()
    {
        try
        {
            new Harmony(HarmonyId).PatchAll();
        }
        catch (System.Exception e)
        {
            Log.ErrorOnce($"[MyMod] Harmony bootstrap failed: {e}", 0x4B3A19D);
        }
    }
}
```

### 15.2 Prefix/Postfix Template (Safe Augmentation)

Template:
```csharp
using HarmonyLib;
using Verse;

[HarmonyPatch(typeof(TargetType), nameof(TargetType.TargetMethod))]
internal static class Patch_TargetType_TargetMethod
{
    [HarmonyPostfix]
    private static void Postfix(TargetType __instance)
    {
        if (__instance == null)
            return;

        try
        {
            // Read or augment results. Avoid allocations and scanning in hot paths.
        }
        catch (System.Exception e)
        {
            Log.ErrorOnce($"[MyMod] Patch failed: {e}", 0x2A9B7B61);
        }
    }
}
```

Guidelines:
- Use `Prefix` only when you must short-circuit or adjust arguments.
- If you use a `Prefix`, keep it side-effect minimal and return `true` unless you must skip vanilla.
- Do not capture closures or allocate per call in frequently-called methods.

### 15.3 Transpiler Template (Fail-Safe IL)

Rules:
- Use transpilers only when there is no stable alternative.
- If the IL pattern is not found, return the original instructions unchanged.

Template:
```csharp
using System.Collections.Generic;
using HarmonyLib;
using Verse;

[HarmonyPatch(typeof(TargetType), nameof(TargetType.TargetMethod))]
internal static class Patch_TargetType_TargetMethod_Transpiler
{
    [HarmonyTranspiler]
    private static IEnumerable<CodeInstruction> Transpiler(IEnumerable<CodeInstruction> instructions)
    {
        try
        {
            // Prefer conservative matching. If not found, do nothing.
            foreach (var ci in instructions)
                yield return ci;
        }
        catch (System.Exception e)
        {
            Log.ErrorOnce($"[MyMod] Transpiler failed; leaving method unmodified: {e}", 0x6E2D0F3C);
            foreach (var ci in instructions)
                yield return ci;
        }
    }
}
```

### 15.4 Cached Reflection (One-Time, Not Hot-Path)

Rules:
- If you must use reflection, resolve it once and cache the result.
- Never call `AccessTools`/reflection every tick.

Template:
```csharp
using HarmonyLib;
using System;
using System.Reflection;

internal static class MyMod_Reflection
{
    internal static readonly MethodInfo SomeMethod = AccessTools.Method(typeof(TargetType), "SomeMethod");

    internal static readonly Func<TargetType, int> SomeDelegate =
        SomeMethod != null ? AccessTools.MethodDelegate<Func<TargetType, int>>(SomeMethod) : null;
}
```

---

## 16. Profiling Workflow (Performance Investigation)

Agents should treat performance work as a loop:
- Measure
- Hypothesize
- Change a small thing
- Measure again

### 16.1 Baseline and Repro

Agents should:
- Establish a reproducible scenario (map size, pawn count, mods enabled)
- Record:
  - average TPS
  - GC behavior (spikes/frequency)
  - whether the issue is UI-only, tick-only, or both

### 16.2 RimWorld Dev Mode Tools

Agents should:
- Use Dev Mode profiling tools to identify hot methods before patching/rewriting code
- Prefer fixing the hottest call sites first (often repeated scans and allocations)

### 16.3 Unity Profiler (When Available)

Agents may:
- Use the Unity Profiler to validate CPU/GC impact of changes when the environment supports it

Agents must:
- Not rely on Unity-profiler-only conclusions; verify in normal gameplay conditions

### 16.4 Log-Based Timing (Low-Overhead Instrumentation)

Rules:
- Instrument only what you need.
- Do not spam logs; sample or gate behind dev mode/settings.

Template:
```csharp
using System.Diagnostics;
using Verse;

internal static class MyMod_Timing
{
    internal static void TimeOnce(string label, System.Action action, int errorOnceKey)
    {
        try
        {
            var sw = Stopwatch.StartNew();
            action();
            sw.Stop();
            Log.Message($"[MyMod] {label}: {sw.ElapsedMilliseconds} ms");
        }
        catch (System.Exception e)
        {
            Log.ErrorOnce($"[MyMod] Timing wrapper failed: {e}", errorOnceKey);
        }
    }
}
```

Guidelines:
- Prefer measuring startup/initialization paths outside of hot loops.
- If you must measure a hot path, sample every N calls/ticks and aggregate.

---

## 17. Settings and UI Performance

RimWorld UI code can be called frequently while windows are open. Settings windows in particular are easy places to accidentally allocate and generate GC.

Agents must:
- Avoid building new strings every frame in UI drawing
- Avoid repeated `.Translate()`/formatting work in drawing paths when the text is stable
- Avoid scanning defs in `DoSettingsWindowContents` / gizmo drawing

### 17.1 ModSettings and Serialization

Agents should:
- Keep settings data minimal and stable
- Use `ExposeData` with safe defaults
- Migrate carefully if changing serialized structure

### 17.2 Cached Labels, Tooltips, and Options

Guidelines:
- Precompute labels/tooltips that do not depend on live state.
- If a label depends on changing state, cache it and invalidate only when the state changes.

Examples of what to avoid in per-frame UI:
- `string` concatenation
- `.ToString()` on numbers every draw
- `.Translate()` called repeatedly for constant labels

### 17.3 Gizmos and Inspect Strings

Rules:
- Gizmo/inspect rendering is a hot path in some scenarios.
- Keep `GetGizmos()` and inspect-string generation allocation-light.

Guidelines:
- Yield existing cached gizmos when possible.
- Avoid creating new `Command_Action` / `FloatMenuOption` objects repeatedly unless necessary.

---

## 18. Additional RimWorld Modding Guidance

### 18.1 Multiplayer and Determinism (Even If You Don’t Support MP)

Agents should:
- Avoid non-deterministic behavior in shared logic (time-based randomness, non-seeded `Rand` usage, depending on real-world time)
- Keep logic stable across reloads and replays

Agents must:
- Treat threading/asynchrony as unsafe around Verse/Unity objects

### 18.2 Def Access and Lifecycle Pitfalls

Agents must:
- Avoid scanning `DefDatabase<T>.AllDefs` repeatedly at runtime

Agents should:
- Prefer `DefOf` for frequently used defs
- Avoid touching `Find`/`Current` game state too early during initialization
- Cache def-derived data after defs are loaded, and keep caches bounded

### 18.3 Patch Conflict Hygiene

Agents should:
- Use a consistent naming convention for patches (e.g. `Patch_<Type>_<Method>`)
- Keep ordering constraints minimal; if required, document why
- Log patch/transpiler failures once and continue safely (do not break gameplay)

### 18.4 Testing and Repro Discipline

Agents should:
- Maintain a minimal repro for bugs and performance issues
- Test with:
  - a clean mod list
  - a typical/heavy mod list (to surface conflicts)
- If you serialize data, test adding/removing the mod mid-save and loading older saves

### 18.5 UI and Graphics Performance Notes

Agents must:
- Avoid per-frame string building, repeated `.Translate()`, and repeated formatting work in draw paths

Agents should:
- Cache stable labels/tooltips and invalidate only when the underlying state changes
- Avoid creating temporary lists/collections during UI drawing

### 18.6 Save Migration Playbook

Agents should:
- Store a version field for serialized data and migrate idempotently on load
- Handle missing defs/references gracefully (defaults, null checks, sanitization)
- Avoid throwing exceptions from `ExposeData`; use once-only logging and recover

### 18.7 Performance Anti-Patterns (Callouts)

Avoid in hot paths (tick/UI):
- `DefDatabase<T>.AllDefs` scans
- Reflection (`AccessTools.*`, `GetMethod`, etc.)
- LINQ/allocations (closures, iterators)
- Rebuilding lists (`ToList()`, `ToArray()`) repeatedly
- Per-frame string concatenation/formatting

---

## 19. XML Modding (Defs & PatchOperations)

This section covers RimWorld 1.6 XML authoring practices (Defs, keyed translations, and patch operations). Even when the mod is primarily C#, XML content is typically part of the deliverable and a common compatibility surface.

### 19.1 Folder Structure (XML Content)

Agents should keep XML organized and predictable:

```
/Mods/<ModName>/
 ├─ Defs/                       (optional)
 │   ├─ ThingDefs/
 │   ├─ HediffDefs/
 │   ├─ RecipeDefs/
 │   ├─ PawnKindDefs/
 │   ├─ FactionDefs/
 │   ├─ BiomeDefs/
 │   └─ Misc/
 ├─ Patches/                    (optional)
 └─ Languages/
     └─ English/
         └─ Keyed/
```

Rules:
- Prefer one Def type per file when practical
- Split by feature, not by version
- Filenames should match the primary Def name when practical

### 19.2 XML Standards (Strict)

Agents must:
- Use consistent indentation (tabs or spaces) within a file; do not mix
- Keep XML human-readable and minimize unnecessary fields
- Prefer explicit values when they improve clarity

Agents must NOT:
- Invent undocumented nodes/attributes
- Copy Core XML blindly without pruning unused or irrelevant fields

### 19.3 Def Naming Conventions

All `defName`s must be:
- PascalCase
- Mod-prefixed to avoid collisions

Example:
```
MyMod_AdvancedHydroponicsBasin
```

Never:
- Reuse Core `defName`s
- Use generic names that are likely to collide

### 19.4 Localization Rules

All player-facing text must:
- Use keyed translations
- Avoid hardcoded strings in defs when a keyed entry is appropriate

Key naming examples:
```
MyMod_Item_Description
MyMod_Research_Title
```

Agents should:
- Keep descriptions concise and neutral

### 19.5 Patch Operations (Compatibility-First)

RimWorld 1.6 supports XML PatchOperations. Agents must use patching responsibly to minimize conflicts.

Agents must:
- Target specific XPath paths (avoid overly broad paths)
- Prefer additive patches over destructive ones
- Ensure failure mode is safe (missing nodes should not hard-error when avoidable)

Agents must NOT:
- Remove entire defs unless absolutely required
- Replace large chunks of Core defs without a clear compatibility plan

#### 19.5.1 Good vs Bad Examples

Good: add a stat base to a specific ThingDef
```xml
<Patch>
  <Operation Class="PatchOperationAdd">
    <xpath>/Defs/ThingDef[defName="Steel"]/statBases</xpath>
    <value>
      <Beauty>-2</Beauty>
    </value>
  </Operation>
</Patch>
```

Bad: overly broad XPath
```xml
<xpath>//ThingDef/statBases</xpath>
```

Bad: replace an entire def
```xml
<Operation Class="PatchOperationReplace">
  <xpath>/Defs/ThingDef[defName="Gun_AssaultRifle"]</xpath>
  <value>...</value>
</Operation>
```

### 19.6 Safe to Change vs High-Risk (Core XML)

Generally safer (with care):
- `statBases`
- `costList` and recipe ingredient amounts
- `workAmount` on `RecipeDef`
- `tradeTags`
- `description` and `label` (via localization)
- category/tag membership lists (e.g. `thingCategories`)

High-risk changes (avoid unless necessary):
- `thingClass`
- `comps` that affect ticking behavior
- `race` definitions
- `pawnGroupMakers`
- raid strategy configuration

---

## 20. Out of Scope

This document does **not** cover:
- Multiplayer compatibility guarantees (unless explicitly requested)
- RimWorld versions < 1.6

---

## 21. Afterword (Combined Document Validity)

This `agents.md` is the single combined guidance document for:
- C# modding (Harmony patching, performance, compatibility, save safety)
- XML modding (Defs, localization, patch operations)

The content is internally consistent under the stated scope (RimWorld **1.6.x**) and follows conservative, compatibility-first modding practices.

---

**End of AGENTS.md**

