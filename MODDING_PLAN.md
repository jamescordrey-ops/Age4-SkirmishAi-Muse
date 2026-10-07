# AoE4 Modding Plan — Zycat AI Lite update (test mod), then our own AI mod

Written 2026-10-01 from live recon against Khan's PC. Verify again if the game patches.

## Phase 0 recon results (done)

**Game fingerprint (Khan's PC, via Steam API + SSH):**
- Age of Empires 4: Anniversary Edition, Steam app 1466860, buildid 24231237
- Game version 16.3.11308.0 — Season Thirteen era, patch from ~July 2026 (current as of Oct 2026)
- Content Editor (Beta) INSTALLED, Steam app 1846820, buildid 8076703
- Mods live at `C:\Users\Khan\OneDrive\Documents\My Games\Age of Empires IV\mods\`
  (extension\subscriptions\<mod.io id>\<hash>.sga; meta in `mods_meta_data.lua`)
- Khan has ~100 subscribed mods already, so the mod pipeline on his machine works.

**Target mod: Zycat AI Lite** (by Zycat, aka Zycat2021 on the forums)
- Type: tuning pack (+ likely a paired game mode; game-mode/tuning-pack bundles are the standard pattern per the aoemods wiki "Bundling Mods" page)
- What it does: parameter-only AI tuning. No custom scripting possible for AI: the `ai\personality*.scar` files are NOT accessible to modders. It tweaks the attribute-side parameters the AI reads (build-order adaptation, counter selection, military production, tower defense, etc., per the author's forum post).
- Age: forum thread ~Sept 2022, built for the patch-5.x era. Roughly 4 years / ~11 major versions stale. Old tuning packs are known to load silently without applying after patches break their attribute clones (Steam forum reports), so this needs a real re-cook, not just a re-upload.
- Khan does NOT have it subscribed currently. Exact mod.io URL/ID not captured (mod.io blocked automated browsing); find it in the in-game Mods browser by searching "Zycat".

**Toolchain:**
- Content Editor (Beta) — the official tool, already installed on Khan's PC. Tuning packs are built in its Attribute Editor: clone vanilla attribute files, edit values, Save and Build (F7). Cooked output is an .sga published to mod.io.
- Community tooling: AOEMods.Essence (C# library for .sga/.rgd Essence Engine files) — for reading/extracting/diffing attribute files outside the editor. The "Age of Empires 4 attribute dump" GitHub repo tracks patch-to-patch attribute changes.
- **aoe4-mcp** (https://github.com/heartmove/aoe4-mcp, Apache-2.0, cloned 2026-10-01 to `~/workspace/vendor/aoe4-mcp/`, installed and working): an MCP server that gives AI coding agents searchable AoE4 modding knowledge — official SCAR APIs, base data (units/techs/buildings), RDO game-mode configs, UI/XAML bindings, localization, generated-map scripts — plus project tools: `scan_project`, `explain_rdo_config`, `explain_rdo_options`, `prepare_scar_patch`, `check_code`, `draft_solution`. Directly useful for both phases: scanning Zycat's mod files, explaining its game-mode RDO entry, and drafting exact SCAR patches. **Gap:** the pre-built knowledge DB download (OneDrive link in its README) is dead, so official-API search is unavailable until we get a DB; project-analysis tools work without it. Fallback knowledge source: the game's own `scardocs/api` folder ships with the install on Khan's PC.

**Anti-cheat verdict: GO (with boundaries)**
- AoE4 has no kernel-level anti-cheat (no EAC, BattlEye, or Vanguard). It does have an internal integrity check: memory editors like Cheat Engine or WeMod running alongside the game cause crashes. Never use those.
- Mods are officially supported and sandboxed by design: tuning packs and game modes only work in custom games / single-player skirmish. Ranked matchmaking disables mods entirely, so there is no competitive-integrity risk and no ban vector from using a tuning pack in custom games.
- Rule: official mod surfaces only (Content Editor, .sga mods). Never touch the game process memory.

**Community route:**
- Knowledge base: the aoemods wiki (wiki.aoemods.com; GitHub mirror readable at github-wiki-see.page/m/aoemods/wiki). Key pages: "Bundling Mods: Game Mode, Tuning Pack, Map", "Population cap" (tuning-pack attribute workflow), "Tuning pack effect values".
- People: the AoE4 modders Discord (Zycat is active there). First move for Phase 1 should be asking Zycat for the original project source and permission to publish an update, with credit.

## Phase 1: update Zycat AI Lite to patch 16.3.11308 (the test mod)

Work split: Khan does in-game steps (subscribe, launch, test, publish click). I do analysis and the editor-side rebuild spec.

1. **Khan subscribes** to "Zycat AI Lite" in the in-game Mods browser. (This downloads the .sga to his mods folder. If it ships as a game-mode + tuning-pack pair, subscribe to both.)
2. **Smoke test the old mod as-is**: enable it, start a skirmish vs AI, play 10 minutes. Expected: it loads but the AI behaves like vanilla (stale attribute clones silently not applying). Screenshot the Mods screen showing it enabled. This establishes the baseline.
3. **Extract what the mod changes**: I pull the .sga from Khan's PC, extract the overridden attribute (.rgd) files with AOEMods.Essence, and diff each against the current vanilla equivalents. Output: a precise change list of (file, table, field, 2022 value, current vanilla value).
4. **Ask Zycat** (modders Discord) for the original Content Editor project + permission to publish a community update with credit. If he says yes, skip to step 6 with his source.
5. **Rebuild in Content Editor** (Khan's PC, he drives, I spec): create a new tuning pack mod, clone the same current-patch attribute files in the Attribute Editor, re-apply the change list from step 3, Save and Build (F7). If the mod has a game-mode half (.scar Lua win conditions), carry it over as-is; Scar API is stable across patches.
6. **Verify in game** (mod-verify protocol): Khan hosts a custom game with the rebuilt tuning pack, plays vs AI, captures: (a) Mods screen showing it enabled, (b) 10-min gameplay notes or screenshots showing AI behavior differences (military production, towers, counter units). I diagnose from that.
7. **Publish**: upload to mod.io from the editor as e.g. "Zycat AI Lite — Community Update for Patch 16.3" with credit to Zycat and a note on what changed. Khan presses publish.

**Known risks:**
- Some 2022 attribute fields may be renamed/removed in the current patch; the diff in step 3 will surface these, and each needs a judgment call (drop it, or map to the replacement field).
- AI behavior parameters may have moved between attribute files across patches; the attribute-dump repo helps map old paths to new ones.
- If Zycat's mod relied on fields that no longer exist, the "update" becomes a partial re-implementation. Still fine as a test mod.

## Phase 2: our own AoE4 AI mod from scratch (after Phase 1 works)

- Design doc first: what "better AI" means concretely (build-order adaptation, counter logic, production constancy, defense). Tuning packs can only move the parameters the AI reads, so scope the design to observable AI behavior levers, verified against the attribute files.
- Build in thin slices (one behavior lever per slice), each verified in-game via the mod-verify protocol before the next.
- Keep a MODLOG.md per mod-notes from day one.

## Quick reference

- Game: Steam 1466860, v16.3.11308.0 (Season Thirteen, ~July 2026)
- Editor: Steam 1846820 (Content Editor Beta)
- Mods dir: `C:\Users\Khan\OneDrive\Documents\My Games\Age of Empires IV\mods\`
- SSH: `khan@100.88.46.67` via `~/workspace/bin/ssh-proxy-connect.py`, key `~/.ssh/khan_pc`
- Wiki: aoemods wiki (mirror: github-wiki-see.page/m/aoemods/wiki)
- Essence file tools: AOEMods.Essence on GitHub
