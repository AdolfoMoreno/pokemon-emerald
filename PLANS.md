# 1. Product Requirements Document (PRD)

No active feature in PM Discovery.

# 2. Technical Implementation Plan

No active feature in EM Architecture.

# 3. Implementation Log & Test Results

No active feature in SWE Implementation.

# 4. Complete Features

## Feature: Bag only HMs

## Approved Branch

- Dedicated feature branch: `feature/hm-bag-use-removable-hms`

## Completion Date

- June 12, 2026

## Product Summary

- Added an opt-in settings menu entry labeled `Bag only HMs`.
- The setting offers `ON` and `OFF` values and defaults to `OFF`.
- With the setting `OFF`, current removable-HM baseline behavior is preserved and Bag-based HM field use is unavailable.
- With the setting `OFF`, HM Bag items use `TEACH` for the existing teaching behavior.
- With the setting `ON`, owned standard HM items in the Bag can be used for field moves without requiring a compatible Pokemon.
- With the setting `ON`, HM Bag items show `TEACH`, `GIVE`, `USE`, and `CANCEL` in a 2x2 context menu, with field `USE` below `TEACH`.
- Bag-based HM use follows the existing badge, map, field, and failure-message checks from normal field-move behavior.
- HM moves remain teachable to Pokemon in both setting states.
- Scope is limited to the standard HM moves supported by this project's HM table.
- Direct overworld prompts remain out of scope and are tracked under `# 5. Future Features`.

## Implementation Summary

- Stored the option in `SaveBlock2` as `optionsBagOnlyHMs` using unused options padding and initialized it to `FALSE` for new saves.
- Added `Bag only HMs` option-menu strings, declarations, task storage, row placement after `Buyable stones`, load/save behavior, and ON/OFF input/drawing.
- Added an HM-specific `TEACH` Bag action label so HM teaching no longer appears as generic `USE`.
- Added the `Bag only HMs = ON` HM context menu layout: `TEACH`, `GIVE`, `USE`, `CANCEL`.
- Added Bag-based standard HM field-use setup for HM01-HM08, guarded by `optionsBagOnlyHMs`.
- Reused existing party-menu field-move setup checks for badges, link/Union Room blocking, map/field validation, and failure messages.
- Used the first non-egg party Pokemon as the field-effect visual source without checking HM compatibility.
- Bypassed Surf's party-compatibility check only for Bag-based HM use while preserving the water-facing check.
- Forced Bag-based Cut to use the normal grass radius rather than expanding through `Hyper Cutter`.
- Added a Fly-map cancel guard so Bag-launched Fly returns to the field instead of the party menu.

## Test Summary

- `make -j4` after setting storage/options-menu/text changes: passed.
  - Memory output: EWRAM `249712 B / 256 KB`, IWRAM `30892 B / 32 KB`, ROM `14929764 B / 32 MB`.
- `make -j4` after HM Bag context-menu and Bag field-use changes: passed.
  - Memory output: EWRAM `249716 B / 256 KB`, IWRAM `30892 B / 32 KB`, ROM `14929764 B / 32 MB`.
- Final `make -j4`: passed.
  - Final memory output: EWRAM `249716 B / 256 KB`, IWRAM `30892 B / 32 KB`, ROM `14929764 B / 32 MB`.
- `git diff --check`: passed.
- Targeted reference scan confirmed `optionsBagOnlyHMs`, `Bag only HMs`, `TEACH`, HM-only context menus, default `OFF` initialization, Bag HM runtime guards, Fly cancel handling, and Cut normal-radius handling are present.
- User manually tested the feature and approved it.

## User Manual Testing Approval Note

- User confirmation: "tested and approved"

## Feature: Buyable stones

## Approved Branch

- Dedicated feature branch: `feature/lilycove-evolution-stones-vendor`

## Completion Date

- June 12, 2026

## Product Summary

- Added an opt-in settings menu entry labeled `Buyable stones`.
- The setting offers `ON` and `OFF` values and defaults to `OFF`.
- The feature applies only to the Lilycove Department Store 2F Right Clerk.
- With the setting `OFF`, the Right Clerk uses the vanilla inventory and dialogue.
- With the setting `ON`, the Right Clerk appends `ITEM_SUN_STONE`, `ITEM_MOON_STONE`, `ITEM_FIRE_STONE`, `ITEM_THUNDER_STONE`, `ITEM_WATER_STONE`, and `ITEM_LEAF_STONE` after the vanilla inventory in the approved order.
- Sun, Fire, Thunder, Water, and Leaf Stones keep their existing `2100` item prices.
- Moon Stone uses a shop-local `2100` price for this clerk without changing its global item price.
- No other shops, pickups, item descriptions, marts, or stone availability paths were changed.

## Implementation Summary

- Stored the option in `SaveBlock2` as `optionsBuyableStones` and initialized it to `FALSE` for new saves.
- Added the `Buyable stones` options-menu row after `Event Tickets` and before `Cancel`.
- Added option strings and load/save wiring through the existing options-menu flow.
- Added `IsBuyableStonesEnabled` for script gating.
- Added `SetLilycoveDeptStore2FRightClerkShopContext` so the shop can apply a local Moon Stone price override only for this clerk.
- Updated only `LilycoveCity_DepartmentStore_2F_EventScript_ClerkRight` to branch between the vanilla and expanded inventories.
- Added the expanded Right Clerk inventory with the approved stone order.
- Routed shop price display, affordability checks, quantity recalculation, and purchase totals through the shop-local price helper.

## Test Summary

- `make -j4` after setting storage/options-menu changes: passed.
- `make -j4` after shop script, script specials, and Moon Stone shop-price override: passed.
- Final `make -j4`: passed.
  - Final memory output: EWRAM `249712 B / 256 KB`, IWRAM `30892 B / 32 KB`, ROM `14929764 B / 32 MB`.
- `git diff --check`: passed.
- `git diff -- src/data/items.h`: no diff, confirming global item prices were not changed.
- Targeted reference scan confirmed `optionsBuyableStones`, the `Buyable stones` options-menu row, default `OFF` initialization, script gating, expanded Right Clerk inventory, approved stone order, and Moon Stone shop-local price helper are present.
- User manually tested the feature and approved it.

## User Manual Testing Approval Note

- User confirmation: "Tested and approved"

## Feature: Elite Four Ticket Rewards

## Approved Branch

- Dedicated feature branch: `feature/elite-four-ticket-rewards`

## Completion Date

- June 12, 2026

## Product Summary

- Added an opt-in settings menu entry labeled `Event Tickets`.
- The setting offers `ON` and `OFF` values and defaults to `OFF`.
- The reward is tied to Wallace's Hall of Fame flow only.
- With the setting `ON`, beating Wallace grants missing event ticket key items if all missing tickets fit in the bag.
- With the setting `OFF`, vanilla behavior is preserved and no tickets, reward flags, ship flags, or reward messages are produced.
- The feature adds only missing tickets and marks the reward claimed once all four target tickets are present.
- Bag capacity failure adds no tickets, shows the approved failure message, and leaves the reward unclaimed for a future Wallace win.
- Success and failure messages appear after the Hall of Fame League Champ card acknowledgement and before credits continue.

## Implementation Summary

- Stored the option in `SaveBlock2` as `optionsEventTickets` and initialized it to `FALSE` for new saves.
- Added the `Event Tickets` options-menu row after `Infinite TMs` and before `Cancel`.
- Added option strings and approved reward messages.
- Reused saved event flag `0x1AA` as `FLAG_RECEIVED_ELITE_FOUR_TICKET_REWARDS`.
- Added `MarkWallaceEventTicketRewardPending` and called it from the Wallace Hall of Fame script before `GameClear`.
- Consumed the Wallace-only pending marker before `TrySavingData(SAVE_HALL_OF_FAME)`, so successful item and flag changes are included in the automatic post-Wallace save.
- Added all-or-nothing missing-ticket capacity checks and added only missing `ITEM_AURORA_TICKET`, `ITEM_MYSTIC_TICKET`, `ITEM_EON_TICKET`, and `ITEM_OLD_SEA_MAP`.
- Set the matching ship destination flags on successful reward.

## Test Summary

- `make -j4` after setting storage/options-menu changes: passed.
- `make -j4` after Wallace reward hook changes: passed.
- Final `make -j4`: passed.
- Follow-up `make -j4` after manual-test timing fix: passed.
  - Final memory output: EWRAM `249708 B / 256 KB`, IWRAM `30892 B / 32 KB`, ROM `14929764 B / 32 MB`.
- `git diff --check`: passed.
- Targeted reference scan confirmed the setting storage/default/menu flow, Wallace-only marker, pre-save reward check, reward claim flag, reward item checks, and ship destination flags are present.
- User manually tested the corrected Wallace flow and approved it.

## User Manual Testing Approval Note

- User confirmation: "tested, working properly"

## Feature: TMs are not spent

## Approved Branch

- Dedicated feature branch: `feature/tms-are-not-spent`
- User reported this feature branch was merged to `master`.

## Completion Date

- June 11, 2026

## Product Summary

- Added an opt-in settings menu entry labeled `Infinite TMs`.
- The setting offers `ON` and `OFF` options matching the style of existing settings.
- The setting defaults to `OFF`.
- New saves keep the feature disabled by default.
- Existing vanilla saves that do not contain this setting load successfully and treat `Infinite TMs` as `OFF`.
- Once the player saves in this hack, the save data includes and persists the new `Infinite TMs` setting.
- When `Infinite TMs` is `OFF`, normal vanilla TM behavior is preserved and TMs are consumed after successful teaching.
- When `Infinite TMs` is `ON`, TMs used through the normal Bag TM-use flow are not consumed after successful teaching.
- HMs and non-TM items remain unchanged.
- Duplicate TM stacks remain unchanged when the setting is `ON`.
- Existing TM-use messages and friendship behavior remain unchanged.

## Implementation Summary

- Stored `Infinite TMs` in `SaveBlock2` using an existing spare options bit: `optionsInfiniteTMs`.
- Initialized `gSaveBlock2Ptr->optionsInfiniteTMs = FALSE` for new saves.
- Added `Infinite TMs` option-menu strings and declarations.
- Added `Infinite TMs` to the options menu after `Modern Exp Share` and before `Cancel`.
- Loaded the option value from `gSaveBlock2Ptr->optionsInfiniteTMs` and saved it back when leaving the menu.
- Added scrolling to the expanded options list so moving past the visible area scrolls the option rows while keeping the menu frame/background stationary.
- Updated `Modern Exp Share` and `Infinite TMs` to match the vanilla binary option layout, with `ON` on the left and `OFF` on the right.
- Resized the options window and frame to the seven-row viewport so option-window tile graphics no longer overlap the reserved frame tile range.
- Updated successful TM teaching so `RemoveBagItem(item, 1)` is skipped only when `optionsInfiniteTMs` is enabled.
- Kept the existing HM guard, friendship adjustment, and TM/HM teaching messages intact.

## Test Summary

- `make -j4` after save-storage/default change: passed.
- `make -j4` after option-menu/scroller/string changes: passed.
- `make -j4` after TM-consumption guard: passed.
- Final `make -j4`: passed.
  - Final memory output: EWRAM `249700 B / 256 KB`, IWRAM `30892 B / 32 KB`, ROM `14929764 B / 32 MB`.
- Follow-up `make -j4` after scroll viewport correction: passed.
- Follow-up `make -j4` after binary option layout correction: passed.
- Follow-up `make -j4` after options-window/frame resize: passed.
- `git diff --check`: passed.
- Targeted reference scan confirmed all `optionsInfiniteTMs` read/write/guard references are present.
- User manually tested the final result after the UI fixes and approved it.

## User Manual Testing Approval Note

- User confirmation: "Tested, approved"

# 5. Future Features

## Bag only HMs: Overworld Prompt Support

- Add Bag-only HM support to direct overworld prompts for `Surf`.
- Add Bag-only HM support to direct overworld prompts for `Cut`.
- Add Bag-only HM support to direct overworld prompts for `Rock Smash`.
- Add Bag-only HM support to direct overworld prompts for `Strength`.
- Add Bag-only HM support to direct overworld prompts for `Waterfall`.
- Add Bag-only HM support to direct overworld prompts for `Dive`.
- Future work must decide approved prompt text because current scripts buffer a party Pokemon nickname for messages like `{STR_VAR_1} used {STR_VAR_2}!`.
