# Contributing

## Scope
This repository contains a RimWorld mod with XML defs and optional XML patch integrations.

## Folder layout
- `Defs/`: Shared defs loaded for all supported RimWorld versions.
- `1.4/`, `1.5/`, `1.6/`: Version-specific defs and patches.
- `CE/`: Combat Extended compatibility patches loaded only when CE is active.

## Texture contributions
- Keep paths consistent with existing `graphicData` and `wornGraphicPath` values.
- Provide both inventory/world and worn textures if applicable.

## Compatibility patches
- Prefer XML patches (`PatchOperation*`) over hard overrides.
- Guard optional integrations behind `PatchOperationFindMod` and safe `PatchOperationConditional` checks.
- Avoid patching hot paths with C# unless necessary.
