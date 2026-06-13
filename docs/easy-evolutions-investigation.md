# Easy Evolutions Lead Investigation

## Status

- Phase: Lead Investigation
- Feature branch: `feature/no-trade-level-stone-evolutions`
- Base branch: `master`
- PRD source: `PLANS.md`, `# 1. Product Requirements Document (PRD)`
- Investigation output path approved by user: `docs/easy-evolutions-investigation.md`
- Source code changes made during investigation: none

## Product Context Received From PM

- Feature label is `Easy Evolutions`.
- Feature is opt-in, defaults `OFF`, and must be toggled from the Options menu.
- `OFF` behavior must match real original vanilla Emerald evolution behavior.
- Existing project evolution changes must be folded into the feature and only apply when `Easy Evolutions` is `ON`.
- Existing project evolution changes should be preserved in the `ON` behavior unless an approved split-evolution decision changes them.
- Single-path level evolutions stay level evolutions.
- Single-path Evolution Stone evolutions stay Evolution Stone evolutions.
- Single-path trade or non-stone special evolutions become level evolutions when `ON`.
- Converted second-stage/final single-path evolutions use level `20`.
- Converted third-stage/final single-path evolutions use prior evolution level plus `5`.
- Split evolution families are case-by-case and must be brought back to the user before EM architecture finalizes behavior.
- Scope is existing Emerald National Dex evolution data only.
- No stone availability, shops, routes, rewards, item prices, item descriptions, or item distribution changes are in scope.

## Current Behavior

### Evolution Data

- `src/data/pokemon/evolution.h` defines `const struct Evolution gEvolutionTable[NUM_SPECIES][EVOS_PER_MON]`.
- `include/pokemon.h` defines `struct Evolution` as `{ u16 method; u16 param; u16 targetSpecies; }`.
- `include/constants/pokemon.h` defines the available methods:
  - `EVO_FRIENDSHIP`, `EVO_FRIENDSHIP_DAY`, `EVO_FRIENDSHIP_NIGHT`
  - `EVO_LEVEL`
  - `EVO_TRADE`, `EVO_TRADE_ITEM`
  - `EVO_ITEM`
  - `EVO_LEVEL_ATK_GT_DEF`, `EVO_LEVEL_ATK_EQ_DEF`, `EVO_LEVEL_ATK_LT_DEF`
  - `EVO_LEVEL_SILCOON`, `EVO_LEVEL_CASCOON`
  - `EVO_LEVEL_NINJASK`, `EVO_LEVEL_SHEDINJA`
  - `EVO_BEAUTY`
- `EVOS_PER_MON` is `5`; Eevee already uses all five slots.

### Runtime Evolution Checks

- `src/pokemon.c:GetEvolutionTargetSpecies` is the central runtime selector.
- Before checking methods it blocks evolution for Everstone holders unless the caller is only checking item usability with `EVO_MODE_ITEM_CHECK`.
- `EVO_MODE_NORMAL` checks friendship, level, attack/defense level splits, Wurmple personality splits, Ninjask, and beauty.
- `EVO_MODE_TRADE` checks `EVO_TRADE` and `EVO_TRADE_ITEM`; when a trade-item evolution succeeds it clears the held item.
- `EVO_MODE_ITEM_USE` and `EVO_MODE_ITEM_CHECK` check `EVO_ITEM` against the used item.
- Normal-mode matching does not break after the first match, so later matching entries can override earlier matching entries in the same species row.
- Item-mode matching breaks after the first matching `EVO_ITEM`.

### Evolution Callers

- Post-battle level-up evolution calls `GetEvolutionTargetSpecies(..., EVO_MODE_NORMAL, ...)` in `src/battle_main.c:TryEvolvePokemon`.
- Rare Candy and party-menu level-up flow call `GetEvolutionTargetSpecies(..., EVO_MODE_NORMAL, ITEM_NONE)` in `src/party_menu.c:PartyMenuTryEvolution`.
- Evolution Stone party-menu usability calls `GetEvolutionTargetSpecies(..., EVO_MODE_ITEM_CHECK, item)` in `src/party_menu.c:DisplayPartyPokemonDataForMoveTutorOrEvolutionItem`.
- Actual Evolution Stone use calls `GetEvolutionTargetSpecies(..., EVO_MODE_ITEM_USE, item)` through `src/pokemon.c` item effect handling.
- In-game and link trade evolution flows call `GetEvolutionTargetSpecies(..., EVO_MODE_TRADE, ITEM_NONE)` in `src/trade.c`.
- Daycare egg species resolution scans `gEvolutionTable` backwards by target species in `src/daycare.c:GetEggSpecies`.
- Shedinja creation depends on Nincada's first two table slots in `src/evolution_scene.c:CreateShedinja`.

## Existing Settings Pattern

- Current feature options live in the `SaveBlock2` bitfield in `include/global.h`.
- Existing added bits include:
  - `optionsInfiniteTMs`
  - `optionsEventTickets`
  - `optionsBuyableStones`
  - `optionsBagOnlyHMs`
- `padding2:15` remains after the existing added options, so there is bitfield room for another one-bit setting.
- New-game defaults are initialized in `src/new_game.c:SetDefaultOptions`.
- `src/option_menu.c` mirrors saved options into task data, processes ON/OFF input, saves back on exit, and draws choices in the scrolling Options menu.
- `NUM_TASK_DATA` is `16`; current option-menu task data uses indices through `data[12]`.
- Current Options menu visible count is `7`, and the menu already scrolls for the added rows.
- Existing option text is declared in `include/strings.h` and defined in `src/strings.c`.

## Branch And History Review

### Approved Branches

- Current feature branch: `feature/no-trade-level-stone-evolutions`
- Base branch: `master`
- Prior evolution branch reviewed: `pokemon-evolutions`

### Prior Evolution Implementation

- Commit reviewed: `cf95333f1 Changed some evolutions to not allow trades`
- That commit changed only `src/data/pokemon/evolution.h`.
- It replaced seven vanilla trade or trade-item evolutions with level evolutions.
- The `pokemon-evolutions` branch has been merged into the current line; `master..pokemon-evolutions` has no remaining diff for the evolution files.

### Prior Non-Vanilla Evolution Changes Now Present

These are current project behavior and must move behind `Easy Evolutions = ON` per the PRD:

| Species row | Vanilla behavior | Current project behavior |
| --- | --- | --- |
| `SPECIES_KADABRA` | `EVO_TRADE` to `SPECIES_ALAKAZAM` | `EVO_LEVEL, 35` to `SPECIES_ALAKAZAM` |
| `SPECIES_MACHOKE` | `EVO_TRADE` to `SPECIES_MACHAMP` | `EVO_LEVEL, 35` to `SPECIES_MACHAMP` |
| `SPECIES_GRAVELER` | `EVO_TRADE` to `SPECIES_GOLEM` | `EVO_LEVEL, 35` to `SPECIES_GOLEM` |
| `SPECIES_HAUNTER` | `EVO_TRADE` to `SPECIES_GENGAR` | `EVO_LEVEL, 35` to `SPECIES_GENGAR` |
| `SPECIES_SEADRA` | `EVO_TRADE_ITEM, ITEM_DRAGON_SCALE` to `SPECIES_KINGDRA` | `EVO_LEVEL, 40` to `SPECIES_KINGDRA` |
| `SPECIES_SCYTHER` | `EVO_TRADE_ITEM, ITEM_METAL_COAT` to `SPECIES_SCIZOR` | `EVO_LEVEL, 35` to `SPECIES_SCIZOR` |
| `SPECIES_PORYGON` | `EVO_TRADE_ITEM, ITEM_UP_GRADE` to `SPECIES_PORYGON2` | `EVO_LEVEL, 30` to `SPECIES_PORYGON2` |

## Current Split Evolution Families

These rows have two or more possible target forms and require user approval before EM architecture finalizes behavior:

| Family row | Current methods | Notes for user approval |
| --- | --- | --- |
| `SPECIES_GLOOM` | Leaf Stone to Vileplume; Sun Stone to Bellossom | Already stone-only, but still split and case-by-case by PRD. |
| `SPECIES_POLIWHIRL` | Water Stone to Poliwrath; trade with King's Rock to Politoed | Politoed currently uses a trade-item split branch. |
| `SPECIES_SLOWPOKE` | Level 37 to Slowbro; trade with King's Rock to Slowking | Converting Slowking to a level method could conflict with Slowbro unless levels/order are chosen carefully. |
| `SPECIES_EEVEE` | Thunder/Water/Fire Stone to Jolteon/Vaporeon/Flareon; friendship day/night to Espeon/Umbreon | Eevee uses all five available evolution slots. |
| `SPECIES_TYROGUE` | Level 20 with attack/defense comparison to Hitmonchan/Hitmonlee/Hitmontop | Already level-triggered, but stat-gated and split. |
| `SPECIES_WURMPLE` | Level 7 personality branch to Silcoon/Cascoon | Already level-triggered, but hidden personality-gated and split. |
| `SPECIES_NINCADA` | Level 20 Ninjask/Shedinja special handling | Shedinja creation depends on the first two table slots. |
| `SPECIES_CLAMPERL` | Trade with Deep Sea Tooth to Huntail; trade with Deep Sea Scale to Gorebyss | Both branches currently require trade items. |

Ralts/Kirlia and Snorunt are not split in the current Gen III National Dex evolution table:

- `SPECIES_RALTS` evolves by level 20 to Kirlia.
- `SPECIES_KIRLIA` evolves by level 30 to Gardevoir.
- `SPECIES_SNORUNT` evolves by level 42 to Glalie.

## Single-Path Conversions Implied By The PRD

The prior non-vanilla changes above are already approved to be preserved for `ON` behavior. The remaining current single-path trade or special methods are:

| Species row | Current method | PRD-implied `ON` direction |
| --- | --- | --- |
| `SPECIES_ONIX` | `EVO_TRADE_ITEM, ITEM_METAL_COAT` to Steelix | Convert to level-only, likely level 20 by the second-stage/final rule. |
| `SPECIES_GOLBAT` | Friendship to Crobat | Convert to level-only, likely level 27 because Zubat evolves to Golbat at 22. |
| `SPECIES_CHANSEY` | Friendship to Blissey | Convert to level-only, likely level 20. |
| `SPECIES_PICHU` | Friendship to Pikachu | Convert to level-only, likely level 20. |
| `SPECIES_CLEFFA` | Friendship to Clefairy | Convert to level-only, likely level 20. |
| `SPECIES_IGGLYBUFF` | Friendship to Jigglypuff | Convert to level-only, likely level 20. |
| `SPECIES_TOGEPI` | Friendship to Togetic | Convert to level-only, likely level 20. |
| `SPECIES_FEEBAS` | Beauty 170 to Milotic | Convert to level-only, likely level 20. |
| `SPECIES_AZURILL` | Friendship to Marill | Convert to level-only, likely level 20. |

These target levels are derived from the approved PM rules. EM should still verify them against the final approved split decisions and implementation approach before writing the technical plan.

## Anticipated Problems And Regression Risks

1. A table-only edit is not sufficient because `OFF` must behave like original vanilla Emerald. The current `gEvolutionTable` already contains non-vanilla trade replacements, so implementation needs a guard strategy that can return vanilla behavior while `OFF` and Easy behavior while `ON`.
2. Trade-item evolution mode clears held items when it succeeds. If `Easy Evolutions = ON` suppresses trade-based evolution, the implementation must avoid clearing held items as a side effect.
3. Split families can be sensitive to entry order because normal-mode evolution checks do not break on the first match.
4. Eevee already uses all five evolution slots, so adding extra entries is not available without changing `EVOS_PER_MON`, which would affect table size and every scan over the table.
5. Nincada/Shedinja is fragile because `CreateShedinja` checks `gEvolutionTable[preEvoSpecies][0].method == EVO_LEVEL_NINJASK` and reads `[1].targetSpecies`.
6. Daycare egg species scans target species through `gEvolutionTable`. Most Easy Evolution changes keep target species unchanged, but any architecture using alternate tables should ensure egg species resolution remains stable.
7. Existing saves and default values need careful review. New games are handled by `SetDefaultOptions`, but EM should decide whether old save data needs defensive normalization so `Easy Evolutions` is reliably `OFF`.
8. Options menu task data still has room, but adding another row requires updating every existing menu enum/name/input/draw/save path consistently.
9. Rare Candy and normal level-up should work automatically if converted methods use the normal evolution path and `param <= level`, but automated testing should cover both.
10. Actual link/in-game trade behavior must be explicitly considered for `ON`: the PRD says no trades should be required, but EM should confirm whether trade-triggered evolution should be suppressed while `ON` or simply made unnecessary by level/stone alternatives.
11. If an alternate evolution table is introduced, all extern declarations and const qualifiers around `gEvolutionTable` should be reviewed. `src/evolution_scene.c` currently declares it without `const`.

## Open Questions Blocking EM Architecture

1. User approval is still needed for every split evolution family:
   - Gloom
   - Poliwhirl
   - Slowpoke
   - Eevee
   - Tyrogue
   - Wurmple
   - Nincada
   - Clamperl
2. EM must decide, with user approval, how to represent vanilla-vs-Easy data without leaking Easy behavior into `OFF`.
3. EM must decide, with user approval, whether `Easy Evolutions = ON` should suppress trade-triggered evolutions entirely or allow trades as an alternate trigger where the data still supports them.
4. EM must decide, with user approval, whether existing saves need explicit option sanitization/migration beyond the new-game default.

## Recommended Areas For EM Review

- `src/data/pokemon/evolution.h`: source evolution data and prior non-vanilla changes.
- `src/pokemon.c:GetEvolutionTargetSpecies`: central guard point for normal, trade, and item evolution modes.
- `src/battle_main.c:TryEvolvePokemon`: post-battle level-up evolution caller.
- `src/party_menu.c:PartyMenuTryEvolution`: Rare Candy and party level-up evolution caller.
- `src/party_menu.c:DisplayPartyPokemonDataForMoveTutorOrEvolutionItem`: Evolution Stone usability preview.
- `src/pokemon.c` item effect handling for Evolution Stone use.
- `src/trade.c`: in-game and link trade evolution callers.
- `src/evolution_scene.c:CreateShedinja`: Nincada/Shedinja special case.
- `src/daycare.c:GetEggSpecies`: evolution table reverse scan for eggs.
- `include/global.h`, `src/new_game.c`, `src/option_menu.c`, `include/strings.h`, `src/strings.c`: settings storage, default value, Options menu, and text integration.
- `src/save.c`: save-block size assertions.

## Suggested Split Decision Worksheet For User Approval

These are not approved decisions. They are the split families that need explicit user answers before EM can finalize architecture.

| Family | Current behavior | Decision needed |
| --- | --- | --- |
| Gloom | Vileplume by Leaf Stone; Bellossom by Sun Stone | Keep both stones unchanged, or change either branch? |
| Poliwhirl | Poliwrath by Water Stone; Politoed by King's Rock trade | Choose Politoed's `Easy Evolutions` method. |
| Slowpoke | Slowbro at level 37; Slowking by King's Rock trade | Choose Slowking's `Easy Evolutions` method without making Slowbro unreachable. |
| Eevee | Three stone forms; Espeon/Umbreon by friendship day/night | Choose Espeon and Umbreon's `Easy Evolutions` methods. |
| Tyrogue | Level 20 with attack/defense branch | Keep stat-based level split, or choose simpler level/stone methods. |
| Wurmple | Level 7 personality branch | Keep personality-based level split, or choose simpler level/stone methods. |
| Nincada | Level 20 to Ninjask plus Shedinja special | Keep current special behavior, or choose a different approved behavior. |
| Clamperl | Huntail/Gorebyss by trade items | Choose Huntail and Gorebyss `Easy Evolutions` methods. |
