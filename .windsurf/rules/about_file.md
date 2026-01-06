# ABOUT_FILE.md — RimWorld 1.6 About.xml Structure

This document defines the **required structure, rules, and best practices** for the `About/About.xml` file in a **RimWorld 1.6 XML-only mod**.

This file is intended for:
- Human authors
- AI agents generating or updating metadata
- Steam Workshop–bound mods

**Scope**
- RimWorld **1.6 only**
- No legacy or backward compatibility

---

## 1. Purpose of About.xml

`About.xml` is the **public-facing metadata** for a RimWorld mod. It is used by:
- The in-game mod list
- Steam Workshop pages
- Load order and dependency resolution

Errors or omissions here directly impact:
- Mod discoverability
- User trust
- Compatibility clarity

---

## 2. Required File Location

The file must exist at:

```
/Mods/<ModName>/About/About.xml
```

Rules:
- Filename must be exactly `About.xml`
- Case-sensitive on Linux

---

## 3. Required Structure (Minimal)

Every RimWorld 1.6 mod **must** include the following structure:

```xml
<ModMetaData>
  <name>My Mod Name</name>
  <author>AuthorName</author>
  <packageId>author.modname</packageId>
  <supportedVersions>
    <li>1.6</li>
  </supportedVersions>
  <description>
    <![CDATA[
    Short description of what the mod does.
    ]]>
  </description>
</ModMetaData>
```

---

## 4. Field Rules & Guidelines

### 4.1 `<name>`

- Human-readable mod name
- Should match Steam Workshop title
- Avoid version numbers in the name

---

### 4.2 `<author>`

- One or more author names
- Multiple authors may be comma-separated

---

### 4.3 `<packageId>` (Critical)

Rules:
- Must be **globally unique**
- Format: `author.modname`
- Lowercase only
- Never change after release

Example:
```
bradleyh.advancedhydroponics
```

Changing `packageId`:
- Breaks saves
- Breaks subscriptions
- Is effectively releasing a new mod

---

### 4.4 `<supportedVersions>`

Rules:
- Must include `<li>1.6</li>`
- Must NOT include older versions
- Do not list future versions unless tested

---

### 4.5 `<description>`

Rules:
- Use CDATA
- Plain text only (no BBCode)
- Honest, concise, non-marketing language

Recommended content:
- What the mod adds or changes
- Major gameplay impact
- Known limitations

Avoid:
- Load-order instructions unless required
- Claims of "100% compatible with everything"

---

## 5. Optional but Recommended Fields

### 5.1 `<modDependencies>`

Use only when **hard dependency exists**.

```xml
<modDependencies>
  <li>
    <packageId>author.dependency</packageId>
    <displayName>Dependency Mod</displayName>
  </li>
</modDependencies>
```

Rules:
- Dependency must be required to function
- Display name must match the dependency’s About.xml

---

### 5.2 `<loadAfter>` / `<loadBefore>`

Use sparingly.

Rules:
- Only specify when technically required
- Avoid Core unless strictly necessary

---

### 5.3 `<incompatibleWith>`

Use only for **known, verified conflicts**.

Rules:
- Never speculate
- Document conflict reason elsewhere if possible

---

## 6. Steam Workshop Alignment

Agents must ensure:
- `name` matches Workshop title
- Description aligns with Workshop page text
- Version support clearly states **RimWorld 1.6**

Do NOT:
- Advertise untested compatibility
- Use emojis or excessive formatting

---

## 7. Common Mistakes (Avoid)

- Missing `supportedVersions`
- Uppercase letters in `packageId`
- Changing `packageId` between releases
- Listing multiple RimWorld versions "just in case"
- Using About.xml as patch notes

---

## 8. AI Agent Rules

AI agents generating or editing `About.xml` must:

- Never invent authorship
- Never change `packageId` unless explicitly instructed
- Match mod scope honestly
- Prefer minimal, accurate descriptions

If information is missing:
- Ask before guessing

---

## 9. Lint Alignment

The following lint rules apply:

- **LINT-002** — RimWorld 1.6 only
- **LINT-050** — Core integrity

Additional About.xml–specific lint rules may be added later.

---

**End of ABOUT_FILE.md**

