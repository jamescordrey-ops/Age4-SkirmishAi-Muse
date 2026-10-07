# AoE4 AI Modding

Reverse-engineering and modding Age of Empires 4 AI behavior. Goal: full control over AI logic via .scar files.

## Status

- **Spearman DLL inject:** WORKS. Byte-pattern memory patches bypass signature checks for local testing.
- **Phase 2 (.scar override):** TESTING - .sga packing solved with AOEMods.Essence, in-game test pending.
- **Zycat AI Lite re-cook:** Paused. Tuning pack update for current patch.

## Project Structure

```
├── MODLOG.md              # Chronological dev log
├── MODDING_PLAN.md        # Full modding plan and recon
├── zycat/
│   └── SPEC.md            # Zycat AI Lite re-cook specification
└── phase2-test/
    └── MOD_SPEC.md        # Phase 2 archers-only test spec
```

## Key Findings

### DLL Inject (Working)
Built from Rust source. Sideloaded as `version.dll` + `version_orig.dll` in the AoE4 folder. Both patches succeed:
- SIGCHECK (17 bytes)
- AEGISDFHCHECK (12 bytes)

**Safety:** Breaks online sign-in. Delete both DLLs before any online play.

### .sga Building (Solved 2026-10-07)
Packaging is solved with **AOEMods.Essence** `sga-pack` (verified: round-trip pack/unpack is byte-identical, unlike the Content Editor which produced archives with file paths but no content).

- Tool: `AOEMods.Essence.CLI.dll sga-pack <input-dir> <output.sga> <archive-name>` (needs .NET 6)
- Test .sga (`khan_test.sga`) built with marker `print("KHAN_SGA_TEST_SUCCESS")`, placed in `cardinal\archives` + RelicGame.module entry on Khan's PC
- Spearman DLL (`version.dll` + `version_orig.dll`) restored to game folder for sig-check bypass
- **Awaiting in-game verification:** launch, start skirmish, check warnings.log for the marker

Earlier dead ends (for the record):
1. Content Editor burn rules exclude `ai\**\*.scar` by default
2. relic-tool-sga doesn't support SGA v11
3. Content Editor creates file entries without content for filesystem-added files

## Safety Rules

- DLL breaks online sign-in. Always delete before playing online.
- Never run Cheat Engine/WeMod alongside the game.
- Mods are for custom games only, disabled in ranked.
- Verify game files via Steam after removing DLLs.

## License

Private research project. Not for distribution.
