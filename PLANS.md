# 1. Product Requirements Document (PRD)

No active feature in PM Discovery.

# 2. Technical Implementation Plan

No active feature in EM Architecture.

# 3. Implementation Log & Test Results

No active feature in SWE Implementation.

# 4. Complete Features

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
