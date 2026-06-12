# Bag Capacity Investigation

## Feature

- Feature branch: `feature/expand-bag-capacity`
- Base branch: `master`
- Workflow scope: investigation-only
- Product goal: identify current Bag pocket limits, current implementation constraints, save compatibility risks, and safe expansion ranges for the current hack.
- User-approved exception: a possible later implementation will not require an options menu entry.

## Current Bag Pocket Limits

Current Bag pocket slot capacities are defined in `include/constants/global.h`:

| Pocket | Constant | Current slots | Useful pressure from current item table |
| --- | --- | ---: | --- |
| Items | `BAG_ITEMS_COUNT` | 30 | High. `src/data/items.h` currently assigns about 207 item-table entries to `POCKET_ITEMS`, including dummy/unused entries. |
| Key Items | `BAG_KEYITEMS_COUNT` | 30 | High. `src/data/items.h` currently assigns 57 entries to `POCKET_KEY_ITEMS`. |
| Poke Balls | `BAG_POKEBALLS_COUNT` | 16 | Low today. The item table currently assigns 12 entries to `POCKET_POKE_BALLS`. |
| TMs/HMs | `BAG_TMHM_COUNT` | 64 | Low today. The item table currently assigns 58 entries to `POCKET_TM_HM`. |
| Berries | `BAG_BERRIES_COUNT` | 46 | Low today. The item table currently assigns 43 entries to `POCKET_BERRIES`. |

The practical pressure in the current hack is regular Items and Key Items. Balls, TMs/HMs, and Berries already have enough slots for all currently defined items in their pockets, unless future custom items add more distinct entries.

## Current Behavior

- `struct ItemSlot` is two `u16` fields, `itemId` and encrypted `quantity`, so each Bag slot costs 4 bytes in save data and RAM.
- `struct BagPocket` stores a pointer to a slot array plus an `u8 capacity`.
- `SetBagItemsPointers()` in `src/item.c` wires the five runtime `gBagPockets[]` entries directly to arrays inside `gSaveBlock1Ptr`.
- Bag item quantities are encrypted with `gSaveBlock2Ptr->encryptionKey`; changing capacity must preserve quantity encryption behavior.
- `AddBagItem`, `CheckBagHasSpace`, `CheckBagHasItem`, `RemoveBagItem`, `ClearBag`, sorting, compaction, and total quantity counting mostly iterate through `gBagPockets[pocket].capacity`.
- TM/HM and Berry pockets are special-cased to prevent duplicate item slots. Normal Items, Key Items, and Balls can use multiple slots for the same item when quantity exceeds one slot's stack capacity.
- Stack capacity is separate from slot capacity:
  - Non-Berry Bag stacks use `MAX_BAG_ITEM_CAPACITY` = 99.
  - Berry stacks use `MAX_BERRY_CAPACITY` = 999.
  - PC item stacks use `MAX_PC_ITEM_CAPACITY` = 999 and are out of scope.
- Battle Pyramid Bag storage is separate and uses `PYRAMID_BAG_ITEMS_COUNT` = 10. This investigation treats it as out of scope unless a later implementation explicitly chooses to change it.
- PC item storage is separate and uses `PC_ITEMS_COUNT` = 50. It is out of scope.

## Relevant Files, Functions, and Tables

- `include/constants/global.h`
  - Owns `BAG_ITEMS_COUNT`, `BAG_KEYITEMS_COUNT`, `BAG_POKEBALLS_COUNT`, `BAG_TMHM_COUNT`, and `BAG_BERRIES_COUNT`.
- `include/constants/item.h`
  - Defines both item-table pocket IDs (`POCKET_ITEMS`, `POCKET_POKE_BALLS`, etc.) and runtime Bag pocket indexes (`ITEMS_POCKET`, `BALLS_POCKET`, etc.).
- `include/global.h`
  - Defines `struct ItemSlot`.
  - Defines `struct SaveBlock1`, where the five Bag pocket arrays are stored at offsets around `0x560` through `0x790`.
  - Current `SaveBlock1` size is documented as `0x3D88`.
- `include/item.h`
  - Defines `struct BagPocket` with an `u8 capacity`.
- `src/item.c`
  - Defines `gBagPockets`.
  - `SetBagItemsPointers()` maps pocket pointers and capacities.
  - `AddBagItem`, `CheckBagHasSpace`, `CheckBagHasItem`, `RemoveBagItem`, `ClearBag`, `CompactItemsInBagPocket`, `SortBerriesOrTMHMs`, and `CountTotalItemQuantityInBag` are the core capacity users.
- `src/item_menu.c`
  - `MAX_POCKET_ITEMS` is calculated from the largest Bag pocket constant plus one Cancel row.
  - `LoadBagItemListBuffers`, `UpdatePocketItemList`, and list/cursor functions use the runtime pocket capacities.
  - Wally tutorial temporarily copies only Items and Poke Balls pockets.
- `include/item_menu.h`
  - `struct BagMenu` stores `numItemStacks[POCKETS_COUNT]` and `numShownItems[POCKETS_COUNT]` as `u8`.
  - `GetItemListPosition()` returns `u8`.
- `src/load_save.c`
  - `struct LoadedSaveData` mirrors Bag pockets during link/Union Room save handling.
  - `LoadPlayerBag()` and `SavePlayerBag()` copy every Bag pocket using the same `BAG_*_COUNT` constants.
  - `SetSaveBlocksPointers()` calls `SetBagItemsPointers()`.
- `src/save.c` and `include/save.h`
  - `SaveBlock1` is stored across four 3968-byte save sectors.
  - `STATIC_ASSERT(sizeof(struct SaveBlock1) <= 3968 * 4)` enforces the hard save-space ceiling.
- `src/rom_header_gf.c`
  - Exports `saveBlock1Size` and all five Bag count values in the GF ROM header.
- `src/hall_of_fame.c`
  - Current hack's Elite Four ticket reward counts free Key Item slots through `gBagPockets[KEYITEMS_POCKET].capacity`.
- `src/data/items.h`
  - Owns item-to-pocket assignments used to estimate current useful pressure by pocket.
- `src/berry_tag_screen.c`
  - Berry Tag browsing uses Bag Berry pocket positions and should be rechecked if Berry capacity changes.

## Save Data and Memory Constraints

### SaveBlock1 hard limit

- `SECTOR_DATA_SIZE` is 3968 bytes.
- SaveBlock1 has four sectors, so hard capacity is `3968 * 4 = 15872` bytes (`0x3E00`).
- Current `SaveBlock1` size is `0x3D88 = 15752` bytes.
- Current remaining SaveBlock1 headroom is `0x78 = 120` bytes.
- Each added Bag slot costs 4 bytes in `SaveBlock1`.
- Therefore, a direct in-place expansion of the existing Bag arrays can add at most 30 total slots across all five Bag pockets before the `SaveBlock1FreeSpace` static assertion fails.

### Static EWRAM impact

The current build on `feature/expand-bag-capacity` passes:

- EWRAM: 249716 B / 256 KiB
- IWRAM: 30892 B / 32 KiB
- ROM: 14929764 B / 32 MiB

For a direct constant-based expansion, each added Bag slot increases static EWRAM by about 8 bytes:

- 4 bytes in `gSaveblock1` / `SaveBlock1ASLR`.
- 4 bytes in `gLoadedSaveData`.

Static EWRAM is not the main limiting factor for a small expansion. SaveBlock1 sector headroom is the main hard limit.

### Heap/runtime impact

- `AddBagItem()` allocates a temporary copy of the target pocket with `itemPocket->capacity * sizeof(struct ItemSlot)`, so each added slot in that pocket costs another 4 temporary heap bytes during item addition.
- Bag menu list buffers are heap-allocated:
  - `ListBuffer1` uses one `struct ListMenuItem` per `MAX_POCKET_ITEMS` entry.
  - `ListBuffer2` uses one item-name buffer per `MAX_POCKET_ITEMS` entry.
  - Current `MAX_POCKET_ITEMS` is based on the largest pocket, `BAG_TMHM_COUNT` = 64, plus Cancel.
  - If no pocket grows beyond 64 slots, these buffers do not grow.
  - If the largest pocket grows beyond 64, each additional maximum-pocket slot adds about 32 heap bytes for Bag list buffers.
- Wally tutorial allocates a temporary copy of Items and Poke Balls only, so expanding those pockets increases that temporary heap allocation by 4 bytes per added slot in those two pockets.
- `MoveSaveBlocks_ResetHeap()` uses heap copies of save blocks; direct SaveBlock1 growth also reduces temporary heap space there by 4 bytes per added slot.

## UI Behavior and Redesign Risk

- The Bag UI shows up to 8 item rows at once (`MAX_ITEMS_SHOWN` = 8), with scrolling for longer pockets.
- No UI redesign appears necessary for modest expansion because the list is already scroll-based.
- `MAX_POCKET_ITEMS` already derives from the largest pocket constant, so list buffers grow automatically with constants.
- `gBagPosition.cursorPosition[]` and `scrollPosition[]` are `u16`, which is comfortable for modest expansion.
- Several list counts and helpers remain `u8`:
  - `struct BagPocket.capacity`
  - `gBagMenu->numItemStacks[]`
  - `gBagMenu->numShownItems[]`
  - `GetItemListPosition()` return type
  - many item loops in `src/item.c`
- Because the Bag normally adds a Cancel row, a practical type ceiling is well below 255 visible entries unless those APIs are widened. This is not a blocker for the save-limited small expansions identified here, but it matters for any future large-capacity design.
- Berry Tag screen navigation should be manually verified if Berry slots change, because it reads adjacent Berry pocket positions directly.

## Similar Implementations and History Reviewed

### Current feature branch versus base

- `feature/expand-bag-capacity` currently has no committed source-code delta from `master`.
- The only current working-tree planning change before this report was `PLANS.md`.
- No Bag capacity constants have been changed in this workflow.

### Recent custom feature history

Reviewed recent branch/merge history touching Bag-adjacent behavior:

- `4e9b1d81b` / branch `feature/hm-bag-use-removable-hms`: added HM Bag usability paths in `src/item_menu.c` and related field-move handling. It did not change Bag capacities, but it increases the importance of regression testing TM/HM pocket context menus if capacities change.
- `d6d330e24` / branch `feature/tms-are-not-spent`: changed TM consumption behavior in `src/party_menu.c` and added option storage. It did not change TM/HM slot capacity.
- `1f72ac81b` / branch `feature/elite-four-ticket-rewards`: added event-ticket rewards and a free Key Item slot counter in `src/hall_of_fame.c`. It correctly reads `gBagPockets[KEYITEMS_POCKET].capacity`, so it should benefit automatically from a capacity increase.
- `e006cf625` / branch `feature/lilycove-evolution-stones-vendor`: changed shop behavior and item purchase pricing logic. It does not change capacity constants, but shop purchase flows use shared `CheckBagHasSpace` / `AddBagItem` behavior.

No prior local branch found here appears to have implemented expanded regular Bag capacities.

## Safe Expansion Ranges

These ranges assume a direct implementation that increases the existing `BAG_*_COUNT` constants and keeps the same SaveBlock1 layout style.

### Hard maximum

- Maximum total added slots across all Bag pockets: 30.
- Example hard-limit distributions:
  - Items +30, all other pockets unchanged.
  - Key Items +30, all other pockets unchanged.
  - Items +15 and Key Items +15.
  - All five pockets +6 each.
- All five pockets +6 exactly consumes the current 120-byte SaveBlock1 headroom and leaves no room for future SaveBlock1 growth.
- All five pockets +7 each exceeds SaveBlock1 capacity and should fail to build unless other SaveBlock1 data is moved/reduced.

### Conservative range

- Recommended direct-expansion planning range: 20 to 25 total added slots across all pockets.
- Example conservative distributions:
  - Items +10 and Key Items +10.
  - Items +15 and Key Items +10.
  - All five pockets +5 each.
- Because current Balls, TMs/HMs, and Berries already cover all currently defined entries, the most efficient use of save space is probably Items and Key Items rather than equal expansion across all pockets.

### Existing-save-compatible range

- There is no safe direct in-place range for existing saves without a migration strategy.
- Even adding one slot before later SaveBlock1 fields changes offsets for later save data.
- Existing saves would be at risk unless implementation either:
  - performs explicit old-layout-to-new-layout migration before gameplay uses shifted fields, or
  - stores extra slots somewhere that does not move existing SaveBlock1 offsets, such as an unused region, and adapts Bag logic to read/write both base and extension storage.
- `unused_3598[0x180]` provides 384 unused bytes in SaveBlock1, but it is far after the current Bag arrays. It could store extension slots without shifting later fields if the implementation treats extra slots as extension storage rather than simply enlarging the original arrays. That path is more complex than changing constants.

## Anticipated Implementation Problems

- Save compatibility is the main risk. The existing loader copies sectors into the current struct layout. A naive array-size increase shifts later fields such as Pokeblocks, Dex flags, variables, TV data, mail, Wonder Card data, and more.
- Save migration needs a reliable old-save detection marker/version. The current PRD explicitly requires save compatibility risk analysis, but no migration design has been approved.
- Direct constant changes may be easy for new games but risky for existing saves.
- `struct BagPocket.capacity` and several Bag list counters are `u8`; they are fine for small expansions but not for large inventory redesigns.
- Equal expansion across all pockets spends scarce SaveBlock1 space on pockets that do not need it today.
- The GF ROM header publishes Bag counts and save block size. Any capacity change should be tested for compatibility with tooling or link features that may read those fields.
- The link/Union Room Bag backup path in `src/load_save.c` must stay synchronized with any capacity design.
- Wally tutorial copies only Items and Poke Balls; if either pocket grows, this code likely still works through `sizeof`, but it should be built and manually tested.
- Berry Tag screen navigation and Berry Blender/Berry Crush entry flows should be manually tested if Berry capacity changes.
- Any implementation that uses extension storage instead of contiguous arrays would require broader changes to Add/Remove/Check/Sort/UI list assembly logic.

## Open Questions for EM Review

- Should a later implementation prioritize existing-save compatibility, or is a new-save-only capacity change acceptable?
- If existing saves must remain compatible, should the architecture use migration of shifted SaveBlock1 data or extension slots stored in an unused SaveBlock1 region?
- Should capacity increases target only pressured pockets, especially Items and Key Items, or should every Bag pocket receive a small increase despite low current pressure in Balls/TMs/Berries?
- How much SaveBlock1 headroom should be preserved for future features?
- Should `struct BagPocket.capacity`, `GetItemListPosition()`, and Bag count fields remain `u8` because expansion is small, or should EM widen them proactively?
- Should the Battle Pyramid Bag remain explicitly out of scope in a later implementation?
- Should the GF ROM header count changes be considered compatibility-sensitive for this hack's target tooling/link use?

## Recommended EM Review Areas

- Choose a save compatibility policy before any source edit.
- Decide whether to spend the 120-byte SaveBlock1 tail headroom directly or preserve it by using extension storage in `unused_3598`.
- Prefer targeted expansion of Items and Key Items unless the product goal changes to future-proof all pockets for new item additions.
- Keep any direct constant-based expansion to 20 to 25 total slots if future SaveBlock1 margin matters.
- If all pockets must grow equally, `+5 per pocket` is the conservative direct range; `+6 per pocket` is the absolute direct maximum and leaves no SaveBlock1 margin.
- Require automated build verification plus manual tests for Bag scrolling, item pickup/shop purchase/full-bag messages, Key Item reward flow, TM/HM pocket behavior, Berry Tag browsing if Berries change, Wally tutorial, and save/load.
