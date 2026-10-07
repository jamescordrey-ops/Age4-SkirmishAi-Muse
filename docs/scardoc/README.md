# ScarDoc Summary (for AoE4 AI modding)

**Primary:** AoE4 lua-docs at `~/workspace/mods/aoe4/aoemods-lua-docs/` (79 modules, 296 AI functions, AoE4-specific). Downloaded 2026-10-07 from https://github.com/aoemods/lua-docs.

**Fallback:** CoH2 ScarDoc at https://cm2.network/ScarDoc/function_list.htm (same Essence Engine API family, fewer AoE4-specific functions). Local copy: `~/workspace/mods/aoe4/scardoc/function_list.htm` (1.5MB, 12,123 lines, fetched 2026-10-07).

## What is ScarDoc?
The complete .scar scripting API reference for Relic's Essence Engine. .scar files are Lua scripts that control game logic, AI, missions, and UI.

## Categories (33 total, ~1,400 functions)

### Directly relevant to AI modding:
- **AIInterface** (36 functions) - THE AI control API. Budgets, build demands, difficulty, production limits, class preferences, attack orders, component toggles.
  - Key: AI_SetBuildTable, AI_SetDifficulty, AI_SetBudgetWeight, AI_EnableComponent, AI_ForceAttack, AI_SetProductionLimits
- **Player** (92 functions) - Player resources, tech, diplomacy
- **SGroup** (133 functions) - Squad group management (AI unit control)
- **Squad** (117 functions) - Individual squad orders and state
- **Entity** (113 functions) - Individual entity manipulation
- **Modifiers** (40 functions) - Apply/remove game modifiers
- **Tanks** (16 functions) - Vehicle-specific (less relevant for AoE4)

### Useful for testing/debugging:
- **Util** (46 functions) - Print, debug helpers
- **RuleSystem** (22 functions) - Event/rule triggers
- **Timer** (12 functions) - Timed events
- **Stats** (19 functions) - Game statistics

### For later (UI/custom game modes):
- **UI** (195 functions) - Custom interface elements
- **modalui** (28 functions) - Modal dialogs
- **World** (66 functions) - Map/world manipulation
- **Camera** (36 functions) - Camera control
- **Objectives** (30 functions) - Mission objectives

### Other:
Balance, battlesim, Blueprint, Command, Core, DesignerLib, EGroup, FOW, ID, Marker, NIS, Presentation, Proximity, Setup, Sound, Stinger, Various

## Discord advice (2026-10-07)
- Feed ScarDoc to the AI when working on tuning packs
- Not all variables load in tuning packs; test each section
- .scar files DO load in game mode (packaging is the issue, not the concept)
- Put raw XAML inside .scar for custom UI; game rejects standalone .xaml

## Next steps
1. Extract AIInterface function signatures with full docs for AI coding reference
2. Cross-reference with AoE4's actual .scar files to see which functions are used
3. Test which tuning pack sections actually load (one at a time, absurd values)
