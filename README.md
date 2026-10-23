# Flak Duster
Rimworld Flak Dusters mod Updated

Replaced flak textures with textures from [Retextured Apparel](https://steamcommunity.com/sharedfiles/filedetails/?id=2042613316)

Gives you the style of a duster and the protection of a flak jacket!

## Ideology Styles

The Flak Duster reuses duster styles from Ideology style categories. How many styles it gets depends on whether [XML Extensions](https://steamcommunity.com/sharedfiles/filedetails/?id=2574315206) is active:

- **With XML Extensions** (optional, recommended): the Flak Duster automatically gets **every** duster style from every loaded mod, including style packs that have no dedicated patch. Categories that already style the Flak Duster are skipped, so nothing is added twice.
- **Without XML Extensions**: built-in patches add the styles below.
  - Spikecore (Ideology)
  - ATH's style Gothic and Bloody Gothic
  - ATH's style Draconic
  - ATH's styles Norse

Each set of patches loads only when its condition is met (see `LoadFolders.xml`), so both never run at the same time. XML Extensions is never required.

> **Note:** XML Extensions currently lists support up to RimWorld 1.6. If it fails on 1.7, disable it: the built-in patches will take over.

## Combat Extended Compatibility

This mod is fully compatible with **Combat Extended 16.7+**.

### CE-Specific Stats
When CE is active, the Flak Duster receives:
- **Bulk**: 12 (inventory space when carried)
- **WornBulk**: 4 (inventory space when worn)
- **ArmorRating_Sharp**: 5mm RHA (light armor plating)
- **ArmorRating_Blunt**: 8mm RHA (padding for blunt protection)
- **ArmorRating_Heat**: 0.15

### Load Order
No special load order required. The mod automatically detects CE and loads appropriate patches.

### Vanilla Compatibility
When CE is not installed, the mod functions identically to vanilla with no errors or broken behavior.
