# Changelog

## 1.2
- Added optional XML Extensions auto-patcher (`StylesXmlExtensions/`) that gives the Flak Duster every duster style from every loaded mod.
- Moved per-mod style patches to `StylesManual/`; they now load only when XML Extensions is not active.

## 1.1
- Added RimWorld 1.7 support.
- Consolidated shared defs and patches into `Common/`, loaded by all versions via `LoadFolders.xml`.
- Added `Legacy/` patch restoring `ArmorRating_Sharp` 0.40 on 1.4/1.5 to preserve per-version parity with the vanilla flak jacket.
## 1.0
- Consolidated `Apparel_FlakDuster` defs to inherit from shared `Apparel_FlakDusterBase`.
- Normalized style integration patches to use `PatchOperationSequence` consistently.
