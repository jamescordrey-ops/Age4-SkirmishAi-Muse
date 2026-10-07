# Phase 2 Test Mod — "Archers Only" AI Override

## What this is

A single modified Lua file that makes the AoE4 AI train **only archers**
(no other military units). This is a PASS/FAIL test of one question: **will
the game load a mod-supplied `ai/cardinal_scoring_functions.scar` instead of
the vanilla one inside Data.sga?**

- **Test file:** `files/ai_cardinal_scoring_functions.scar` (in this folder)
- **What changed:** 25 military-unit scoring functions had their bodies replaced
  with a single call to `Phase2_ArcherOnlyGate`, which returns 1000.0 for any
  production blueprint whose name contains "archer"/"longbow" and 0.0 for every
  other military unit. Villagers, buildings, techs, landmarks, economy, and
  tactics are **untouched**, so the AI still gathers, ages up, builds an
  archery range, and attacks — it just does it all with archers.
- **Vanilla backup:** `../hunt/ai_files/ai_cardinal_scoring_functions.scar`
  (untouched original, do not modify).

## Override mechanism under test (primary)

**Mod-archive path overlay.** The mod's cooked `.sga` will contain the file at
`ai/cardinal_scoring_functions.scar` — the same relative path it has inside
the game's `Data.sga`. The bet is that the engine's virtual file system checks
mod archives before base archives when resolving the personality's
`import('cardinal_scoring_functions.scar')`, so the modded version wins.

This is the simplest possible form of the open question from AI_REFERENCE.md
§7.1. If it works, every future AI tweak ships the same way. If it doesn't,
see Fallback below.

## Build steps (AOEMods.Essence sga-pack — verified working 2026-10-07)

> The Content Editor approach is deprecated: it produces .sga archives with
> file paths but no content. Use AOEMods.Essence `sga-pack` instead
> (round-trip verified byte-identical).

1. On the VM (or any machine with .NET 6): place the .scar at
   `input/ai/cardinal_scoring_functions.scar`
   (get the file from `~/workspace/mods/aoe4/phase2-test/files/`).
2. Run:
   ```
   dotnet AOEMods.Essence.CLI.dll sga-pack input output/phase2_test.sga phase2_test
   ```
3. Verify with `sga-unpack` that the .scar is inside at the right path.
4. Transfer `phase2_test.sga` to Khan's PC:
   `C:\Program Files (x86)\Steam\steamapps\common\Age of Empires IV\cardinal\archives\`
5. Add to `RelicGame.module` (back it up first):
   ```
   [data:common:12]
   required = 1
   archiveRoot = cardinal\archives
   archive.01 = phase2_test
   ```
6. Ensure spearman DLLs (`version.dll` + `version_orig.dll`) are in the game
   folder for the sig-check bypass. **Delete both before any online play.**

## Enable and verify loading

1. In-game: **Mods** browser → enable `Phase2 Archer Test`.
2. Host a **custom skirmish** (NOT ranked — mods are disabled there by design):
   - Map: a land map, e.g. **Dry Arabia** (no water — keeps the signal clean).
   - AI opponent: **1 AI, civ French** (standard archers, no civ-specific
     quirks), difficulty **Hard** (we want it building a real army).
   - Your civ: anything. Play normally or defensively.
3. See TEST_PROTOCOL.md for exactly what to do and what counts as PASS/FAIL.

## Fallback (if the primary mechanism fails)

**Symptom:** mod builds and enables fine, but the AI plays completely normally
(full mixed army).

That means the engine does **not** overlay `ai/`-pathed mod files. Next options,
in order:

1. **Verify the .sga first.** Send the cooked `.sga` back (or tell me its
   path) — I'll confirm the file is actually inside at the right path. A
   packaging mistake is the most boring explanation and the easiest to rule
   out.
2. **`lua_binding` repointing (tuning pack + script).** Instead of replacing
   the vanilla file, we ship a NEW function (`ScoringFunctions_Phase2Test`)
   and repoint the military `lua_binding` strings in `default_economy.xml`
   via a tuning-pack attribute override. This uses two officially-supported
   surfaces (tuning packs override attribs; the binding pattern is
   Relic-documented), but requires ~15 binding edits in the Attribute Editor
   and still needs a mod-loaded `.scar` defining the function — so it also
   depends on *some* form of script loading working.
3. **Research how the editor registers scripts.** There may be a manifest,
   `.rdo` entry, or AI-specific mod type that declares script overrides. The
   aoemods wiki and the Content Editor's own docs are the sources.

Do NOT repack or modify the game's own `Data.sga` — that risks integrity
checks. Mod overlays only.

## Path forward (researched 2026-10-04)

**Status:** The direct `ai\` overlay path is blocked at packaging. The Content
Editor's Archive burn rules have no include pattern for `ai\` (verified in
`_default.burnproj`: Lua burner handles `scar\**\*.scar`,
`scenarios\**\*.scar`, etc., but nothing under `ai\`). Hand-adding
`ai\**\*.scar` got the file into the `.sga` but the game rejected the archive:
`MOD -- Error loading mod pack: invalid file structure.` Burnproj reverted.
The `ai\` path is a dead end through the Content Editor.

**Key finding:** The AI's personality does `import('cardinal_scoring_functions.scar')`
with a BARE filename (no `ai/` prefix), yet the file lives at
`ai/cardinal_scoring_functions.scar` in Data.sga. The engine's `import()` therefore
resolves via VFS search across mounted archives, not by exact path. Other bare
imports (`tactics.scar`, `ai-view.scar`) resolve to engine archives, confirming
multi-archive search. This opens the door below.

### Recommended approach (ranked)

**1. `scar\` path overlay — same filename, cookable path (TRY FIRST)**
- Copy the already-built `files/ai_cardinal_scoring_functions.scar` to
  `assets\scar\cardinal_scoring_functions.scar` in the Content Editor project.
- Rebuild with the stock burnproj (no hacks — `scar\**\*.scar` is a natively
  supported pattern).
- Test in-game with the existing TEST_PROTOCOL.md (unchanged).
- *Why this might work:* if the VFS `import()` search checks mod archives
  before base archives and matches by filename (or if `scar/` is in the
  search path), our file wins over the vanilla `ai/` copy.
- *If PASS:* full AI Lua modding unlocked through `scar\` — ship any AI file
  this way. *If FAIL:* the VFS is strict path-aware; move to option 2.
- *Khan's work:* rebuild only (file placement can be done via SSH beforehand).

**2. `lua_binding` repointing via tuning pack**
- Tuning pack overrides `default_economy.xml`: change military production
  groups' `lua_binding` from `"ScoringFunctions_X"` to custom names
  (e.g. `"Phase2_ArchersOnly"`).
- Define the custom functions in a `scar\` file (e.g.
  `scar\phase2_ai.scar`), imported by the game-mode's winconditions scar so
  they're global before the AI scores production.
- *Requires (unproven):* AI and game-mode scripts share the Lua global
  namespace; `lua_binding` strings resolve via global lookup at scoring time.
- *More moving parts than option 1; try only if option 1 fails.*

**3. Debug the `ai\` cook failure**
- The file got into the `.sga` but the archive was structurally invalid.
  Unknown whether the `ai\` path itself, the hand-edited burn rule, or
  something about our specific 116KB file broke the packager.
- Could try: minimal test file at `ai\`, alternative burn rule syntax.
- *Deprioritized: fighting the tool. Only if options 1–2 fail.*

**4. Manual `.sga` construction (last resort)**
- Python `relic-tool-sga` has v10 read; `EssenceFSFactory` exposes `write`
  but v10 write is unproven, and the v11 writer plugin failed to install.
- Game archives are v10 but the editor cooks v11 mods; unknown if the game
  accepts v10 mod archives.
- *Highest risk, most work. Only if all else fails.*

### Honest unknowns
- Does `import()` match by filename across the VFS, or is it path-aware with
  a fixed search-path list? (Option 1 tests this directly.)
- Do mod archives take VFS priority for *script* files, or only for assets?
- Does the engine cache `lua_binding` function references at AI init, or
  resolve them per scoring call? (Matters for option 2's timing.)
- Why exactly did the `ai\` cook produce "invalid file structure"? (Matters
  only if we revisit option 3.)

## Test results (2026-10-04, evening session)

**Test A — `ai\` overlay via patched burnproj: FAILED (packaging)**
- Added `ai\**\*.scar` to the Lua burner includes in `_default.burnproj`.
- File made it into the `.sga` (verified via strings), but the game rejected
  the whole archive: `MOD -- Error loading mod pack: invalid file structure.`
  Mod did not appear in-game at all. Burnproj reverted afterward.

**Test B — `scar\` overlay via native cook path: FAILED (engine ignores it)**
- Moved modified file to `assets\scar\cardinal_scoring_functions.scar`.
- Cooked cleanly through the editor's native `scar\**\*.scar` rule; mod loaded
  fine in-game (`LoadWinCondition succeeded`, no errors, no Lua errors).
- In-game skirmish test: AI played normally (built knights etc.), our file was
  NOT used. The engine's `import('cardinal_scoring_functions.scar')` resolves
  to the vanilla `ai\` copy in Data.sga and does not check mod archives.
- **Verdict: the file-overlay path is dead.** No placement of a replacement
  `.scar` in a mod archive gets picked up by the AI.

**Remaining path:** `lua_binding` repointing (option 2 above). Research complete —
see "lua_binding repointing verdict" below.

## lua_binding repointing verdict (researched 2026-10-04)

### Verdict: NO-GO on custom Lua functions / GO on attribute-only production control

Two separate questions, two different answers:

**1. Repointing `lua_binding` to CUSTOM Lua functions: NO-GO.**

The `lua_binding` string (e.g. `"ScoringFunctions_Infantry"`) is resolved as a
global lookup in the AI's Lua state at production-evaluation time. For a custom
function name to resolve, it must exist as a global in that state. The AI's
state is populated solely by the personality file's `import()` calls, which
resolve exclusively to vanilla archives (proven by Test B: the engine ignored
our mod copy and loaded the vanilla `ai\` file). There is no viable injection
point for custom Lua:
- File overlay: dead (Test A + Test B).
- Game-mode scripts defining AI-visible globals: unproven, and scardocs shows
  explicit `lua_State*` parameters on several functions, suggesting separate
  Lua states for game-mode vs AI contexts.
- No community precedent exists (Steam forums: "AI files are not available
  for modification").

**2. Controlling AI production purely through tuning-pack attributes: GO.**

`default_economy.xml` (445 production groups) gives two powerful levers that
require zero Lua injection — just a tuning pack overriding the attribute file:

- **`minimum_score_to_produce` (Float, per production group):** the engine
  skips any group whose evaluated score is below this threshold. Set to
  999999 to disable a group entirely; set to 0/1 to leave it enabled. Vanilla
  uses 1 and 100, confirming it's a live value.
- **`lua_binding` repointed to a DIFFERENT VANILLA function:** e.g. point a
  production group at `ScoringFunctions_RangedInfantry` instead of its default.
  Every candidate function already exists in the AI's Lua state, so no
  injection needed. (Niche use; the threshold lever is the practical one.)

This is "expanded parameter tuning," not "custom AI programming": we control
WHAT the AI builds and in what quantities, but not the scoring algorithms,
tactics, or target selection (those remain locked in vanilla Lua). Still, it's
a large step beyond Zycat-style numeric tuning — it's direct control over the
AI's production choices.

### Recommended test (proves the mechanism)

**"No-army AI" via tuning pack:**
1. In the Content Editor, create a tuning pack that clones
   `attrib/ai/ai_economy/complete/default_economy.xml`.
2. In the clone, set `minimum_score_to_produce` to 999999 on every MILITARY
   production group (infantry, cavalry, siege, naval military, defensive
   structures — the exact group list to be provided from XML analysis).
   Leave economy/villager/building groups untouched.
3. Build, enable in a skirmish vs AI.
4. **PASS:** AI booms economy but fields no army (or only token units).
   **FAIL:** AI builds a normal army (thresholds ignored).

This is a cleaner signal than the archer-only attempt: binary, no need to
identify archer-specific groups, and it directly validates the lever we need
for all future tuning.

### What Khan builds vs what I prepare

- **I prepare:** the exact categorized list of production groups (military vs
  economy, from the 445 in `default_economy.xml`), which thresholds to change,
  and step-by-step Attribute Editor instructions.
- **Khan builds:** the tuning pack in the Content Editor (clone, edit, F7),
  enables it, runs the skirmish, reports what the AI did.

### Honest unknowns / risks

- **Biggest risk:** `minimum_score_to_produce` may not be respected at
  runtime (could be legacy/unused despite being set in vanilla). The test
  above is designed to answer exactly this.
- The Attribute Editor UI is unconfirmed to expose `minimum_score_to_produce`
  and `lua_binding` for editing (they're standard RGD nodes; Zycat's mod
  overrode ai attributes successfully, so likely fine).
- Categorizing all 445 production groups into military/economy needs care;
  some groups are mixed (e.g. groups containing both military and economic
  units across civs/ages).
- Even if this works, custom DECISION LOGIC (new scoring algorithms, custom
  tactics) remains out of reach without a Lua injection point.
