# Zycat AI Lite — Tuning Decision Spec

Value-level blueprint of every tuning decision in Zycat's AI Lite mod,
isolated against 2022-era vanilla. This is the source document for the re-cook
onto the current patch.

- **Mod**: Zycat AI Lite (tuning pack), mod.io ID 55680
- **Tuning pack ID**: `189ed19cfaa04aaf8d53a8ecb42351d2`
- **Baseline**: 2022 vanilla attrib (github.com/aoemods/attrib @ b8cce48),
  matching the mod's era. 2022 vanilla difficulties were byte-identical except
  pbgid/profile_alias, so every value below that differs from vanilla is
  Zycat's decision.
- **Method**: official RGD decoder chokes on a UTF-16 string type in
  ai_settings, so a purpose-built parser (`/tmp/rgdfull.py`) read the binary
  directly using the key table stored in each file. All 20 changed ai_settings
  keys verified by absolute file offset.
- **Totals**: 77 files added (pure Zycat, no vanilla equivalent), 45 files
  modified (+1559 / -1878 / ~1300 changed leaves).

## 1. Difficulty wiring (ai_personality/default_skirmish)

Surgical repoint: each difficulty's economy `$PBGNAME` now points at the mod's
own economy groups (prefix `189ed19cfaa04aaf8d53a8ecb42351d2:`).

| Difficulty | Vanilla economy | Zycat economy group |
|---|---|---|
| Easy | default_economy_group | eg_easy |
| Standard | default_economy_group | eg_intermediate |
| Hard | default_economy_group | eg_hard |
| Hardest | default_economy_group | eg_hardest |

Economy group contents:

| Group | Points to |
|---|---|
| eg_easy | default_economy_free_easy |
| eg_intermediate | default_economy_free_medium |
| eg_hard | default_economy_free_optimized_hard |
| eg_hardest | default_economy_free_optimized_hard (same as Hard) |

Hardest gets no separate economy. Its edge over Hard comes entirely from
ai_settings (resource multipliers + starting bonuses). Matches Zycat's post:
"Hardest = Hard + 100 food / 150 wood / 100 gold start and 15% gather bonus."

## 2. ai_settings (all four difficulties changed)

Zycat's philosophy is visible in the graduation: the *smarts* (detection,
memory, pathing) are given to Standard and above, the *cheats* (multipliers,
starting resources) only to Hardest, and Easy is deliberately dumbed down.

| Key | Vanilla | Easy | Standard | Hard | Hardest |
|---|---|---|---|---|---|
| target_identification.max_dist_to_still_detect_in_fow | 10 | 10 | 64 | 64 | 64 |
| target_identification.targeting_memory_duration | 30 | 30 | 90 | 300 | 300 |
| target_identification.max_dist_to_still_detect_in_camo | 1 | 1 | 16 | 16 | 16 |
| target_identification.clump_detection_lead_movement_time | 3 | 3 | 3 | 5 | 5 |
| target_identification.enemy_clump_path_to_hq_settings.raw_distance_to_hq_max | 200 | 480 | 480 | 200 | 480 |
| tactics.vehicle_update_pos_delay_time | 5 | 5 | 1 | 1 | 1 |
| pathing.pathing_threat_memory_duration | 30 | 30 | 30 | 300 | 300 |
| pathing.pathing_detects_firing_arcs | true | **false** | true | true | true |
| pathing.pathing_detects_min_firing_range | true | **false** | true | true | true |
| pathing.pathing_detects_suppression | true | **false** | true | true | true |
| pathing.pathing_detects_squads_in_camoflauge | true | **false** | true | true | true |
| pathing.pathing_detects_opponents_fog_of_war | true | **false** | true | true | true |
| fallback.fallback_squad_health_ratio_end | 0.99 | 0.99 | **0.0** | 0.99 | 0.99 |
| fallback.fallback_capacity_ratio | 0.5 | 0.5 | **-1.0** | 0.5 | 0.5 |
| fallback.fallback_squad_health_ratio_start | 0.5 | 0.5 | **0.0** | 0.5 | 0.5 |
| skirmish_settings.combat_eval.desired_ratio | 0.75 | **0.6** | 0.75 | 0.85 | 0.85 |
| skirmish_settings.resource_multipliers.food | 1 | 1 | 1 | 1 | 1.15 |
| skirmish_settings.resource_multipliers.gold | 1 | 1 | 1 | 1 | 1.15 |
| skirmish_settings.resource_multipliers.wood | 1 | 1 | 1 | 1 | 1.15 |
| skirmish_settings.resource_multipliers.stone | 1 | 1 | 1 | 1 | 1.5 |
| starting_resource_bonuses.food | 0 | 0 | 0 | 0 | 100 |
| starting_resource_bonuses.gold | 0 | 0 | 0 | 0 | 100 |
| starting_resource_bonuses.wood | 0 (entry added) | 0 | 0 | 0 | 150 |

Reading the design:
- **Easy is nerfed**: desired_ratio 0.6 (more cowardly than vanilla 0.75),
  five pathing-awareness senses turned off. It reacts dumber, on purpose.
- **Standard gets the senses**: 64m fog-of-war detection, 16m camo
  detection, 1s vehicle updates, and squad fallback *disabled* (0.0/-1.0/0.0
  = never retreats). Punishing but fair.
- **Hard gets the memory**: 300s targeting and threat memory (vs 30),
  5s clump lead time, desired_ratio 0.85 (fights outnumbered less).
- **Hardest gets the cheat**: +15% food/gold/wood, +50% stone, +100f/+100g/+150w
  start. Note stone 1.5x was described in Zycat's post as Mongol-specific;
  in the file it is global to the difficulty.
- **Quirks** (flag for rebuild): Hard keeps vanilla raw_distance_to_hq_max
  200 while Easy/Standard/Hardest use 480. The wood starting-bonus entry did
  not exist in vanilla and was added.

## 3. Economy files and villager caps

The free_* economies are standalone streamlined files (3 production groups,
0 upgrades) vs default_economy's 75 production groups. Villager caps live in
`ai_economy_bag/squads/utility_entry/utility/max_num`:

| File | Villager cap |
|---|---|
| default_economy_free_easy | 42 |
| default_economy_free_medium | 69 |
| default_economy_free_optimized_hard | 96 |

Easy 42 / Intermediate 69 / Hard+Hardest 96. Zero-cheat: Hard's 96 comes with
no multipliers (see table above).

## 4. New AI abilities (10 files)

Each teaches the AI to use a game ability it previously ignored, with usage
priority/chance/filters. New files, copied verbatim in the rebuild.

| File | Game ability |
|---|---|
| races/chinese/imperial_palace_spy | imperial_spies |
| races/english/longbow_setup_camp | longbow_set_up_camp |
| races/english/longbow_volley | longbow_rate_of_fire_ability |
| races/fre/deploy_pavise | deploy_pavise_fre |
| races/hre/influence_auto_repair | influence_auto_repair_hre |
| races/mongol/khan_scouting_falcon | scout_falcon_sight_mon |
| races/rus/golden_gate_trading_buy_stone | buy_stone |
| races/rus/golden_gate_trading_sell_food | sell_food |
| races/rus/streltsy_double_time | streltsy_double_time_ability_rus |
| races/sultanate/forced_march | infantry_forced_march_sul |

Modified vanilla abilities (3):
- market_trading: can_do_chance 1 -> 0.01, priority 500 -> 5
  (AI barely touches the market; anti-cheese).
- khan_defensive_arrow: priority 500 -> 50 (used less).
- khan_maneuver_arrow: can_do_chance 1 -> 0, priority 500 -> 0 (disabled).

## 5. New AI entities (51 files)

Build-order and placement directives per civilization. New files, copied
verbatim in the rebuild.

- 7x age1_* : age-up landmark choices (chinese_barbican, chinese_imperial_academy,
  eng_abbey_hall, rus_kremlin, deer_stones, landmarks)
- 7x age2_* : age-2 landmark choices (chinese_imperial_palace, kingspalace,
  mon_kurultai, rus_high_trade, sul_house_of_learning, whitetower, landmarks)
- 3x age3_* : age-3 landmark choices (berkshire, rus_armory_landmark, landmarks)
- 8x tc_* : town center placement
- 8x farm_* : farm placement per civ (abb, chi, eng, fre, hre, mongol pasture, +2)
- 4x military_* : military building directives
- 3x mongol_*, 2x sultanate_*, 2x chinese_*, 2x abbasid_*
- 1x wonder, 1x outpost, 1x house, 1x granary, 1x imperial

Modified vanilla entity (1):
- farm.json: construction reworked (placement_function added,
  max_num_additional_builders 4 -> 2, near_to_structure refs removed).

## 6. New formation files (6)

- ai_formation_coordinator_attack_landmarks_enroute
- ai_formation_coordinator_attack_wolf (attacks wolves; boar/wolf play)
- ai_aggressive_path, ai_general_path, ai_naval_path
- scout_target_priority

## 7. Modified military tuning (41 files)

All surgical value changes; full leaf diffs saved at `/tmp/modified_diffs.json`
for mechanical application during the rebuild.

- **20x ai_formation_target_priority**: expanded unit-type score tables
  (added grenadier, trade_cart, cavalry_archer, spearman, villager, building
  with scores 6-9; e.g. archers now score villagers 8 and trade carts 9),
  target_damage_scale 1 -> 0.1 in places, new exclude_unit_types
  (building, ram). Net: the AI picks targets like a player.
- **12x ai_formation_coordinator**: harass retargeted (outpost not house,
  farm not resource_drop_off; max_units 8 -> 0; burn thresholds retuned),
  plus basic/defend_wall/attack_docks/attack_wonder/engage_path_blockers.
- **7x ai_formation_path**: scout combat_fitness_min 0.1 -> 0, unit-type
  score refs added; harass/monk/trader/villager/naval/campaign paths retuned.
- **1x ai_personality/default_skirmish**: the difficulty repoint (section 1).

## 8. default_economy overhaul (modified vanilla)

+1419 / -1878 / ~1300 changed leaves. The bulk is production_groups (75
groups, 875 changed leaves): unit blueprint reference updates across tiers
(e.g. spearman_2_eng replacing manatarms_1_eng, horseman/knight tier bumps
per civ), base_scale 600 -> 800, minimum_score_to_produce and max_current
retunes. Also squads (54), entities (122), expansion/dropoff/upgrade utilities.

**Open question for the rebuild**: nothing in Zycat's personality or economy
groups references default_economy; the difficulties resolve through the
eg_* -> free_* chain. The default_economy overhaul may be a fallback for the
vanilla personality path, or vestigial. The rebuild must decide whether to
carry it, and where its production-group updates belong on the current patch.

## 9. Rebuild notes

- The 77 added files have no vanilla counterpart: re-encode and ship verbatim
  (after confirming each path is still read on the current patch).
- The 45 modified files need their leaf diffs applied to *current-patch*
  vanilla files, not the 2022 originals, so 4 years of Relic AI fixes survive.
- ai_settings: apply the per-difficulty table from section 2 to current
  ai_settings files. The parser (`/tmp/rgdfull.py`) reads and writes keys by
  name; the official decoder still cannot round-trip these files (UTF-16
  type unhandled upstream).
- Known current-patch hazard: 74 of the mod's 180 paths are orphaned on
  16.3.11308 (difficulty chain points at files the game no longer reads).
  Mapping the eg_*/personality chain to current equivalents is the first
  rebuild step, before any values are applied.
- Permission: Zycat's source was never obtained. Khan to ask Zycat on the
  modders Discord for source + permission before publishing anything.
