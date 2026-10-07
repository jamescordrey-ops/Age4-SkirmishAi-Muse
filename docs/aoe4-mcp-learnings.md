# aoe4-mcp Learnings (incorporated 2026-10-07)

**Source:** https://github.com/heartmove/aoe4-mcp (MCP server for AoE4 modding, designed for Codex)
**Local copy:** ~/workspace/mods/aoe4/aoe4-mcp/

## What it is
A Model Context Protocol server that gives AI assistants structured access to AoE4 modding knowledge: official SCAR API, base game data, UI templates, localization, and project code. Built for Codex but the knowledge applies generally.

## Key Learnings for Our Project

### 1. RDO Files (NEW - we didn't know about these)
Game mode configuration files. They define the entry point for game modes.

**Location:** `scar/**/*.rdo` in a mod project

**Critical fields:**
- `m_scarWinConditionFile`: Entry SCAR file path (relative to `scar/`, no `.scar` suffix)
  - Example: if value is `my_mode/my_mode`, the file is `scar/my_mode/my_mode.scar`
- `m_scarMissionFile`: Mission/scenario entry SCAR (same format)
- `m_tuningModGUID`: Associated mod GUID (remove hyphens for loc keys)
- `m_key`: Stable config option key (read by SCAR/UI)
- `m_defaultValue`: Default option value
- `m_locStringKey`: Player-facing name (locdb entry id)
- `m_descriptionLocStringKey`: Player-facing description (locdb entry id)

**Why it matters:** This is how game modes tell the game which .scar to load. Our .sga needs the right RDO structure, not just the .scar file.

### 2. Official API Docs Location
The game itself ships API docs at: `<aoe4-install>/scardocs/api`

We should check if these exist on Khan's PC. They'd be the authoritative reference.

### 3. Official SCAR as Function Library
The game's .sga files contain official .scar code that can be imported and used as a function library. The aoe4-mcp indexes these for reference.

**Implication:** We can look at how the official AI .scar files use the API, then mirror those patterns.

### 4. Multiplayer Sync Rules
From agents.md:
- All gameplay code must be deterministic (no random without seed)
- Use `Network_RegisterEvent` and `Network_CallEvent` for multiplayer-shared state
- Pure UI updates can stay local

**Why it matters:** If we ever want our AI mod to work in multiplayer custom games, we need to follow these rules.

### 5. Locdb Key Format
- Mod loc keys: `$<guid-without-hyphens>:<entry_id>`
- Get the GUID from the mod's `.aoe4mod` file
- English CSV: `ID,Pipeline,PipelineStage,Notes,TranslationNotes,Tags,Text`
- Chinese CSV: `ID,SourceText,TranslatedText`
- Don't mix column formats

### 6. XAML Binding Syntax
When binding XAML properties: `{Binding [PropertyName]}` (with brackets), NOT `{Binding PropertyName}`

This is a gotcha that would cause silent failures.

### 7. Confidence Scoring
The MCP tracks whether an API is actually used in official SCAR code. APIs with no official usage are lower confidence.

**For us:** Prefer APIs we've seen in the official .scar files or lua-docs with usage examples.

## What We Can't Directly Use
- The MCP server itself (designed for Codex, not for Muse)
- The sanitized DB (requires download from OneDrive, may not be accessible)
- The Python tooling (we can read the code for reference, but it's built for the MCP protocol)

## What We Should Do
1. Check if `<aoe4>/scardocs/api` exists on Khan's PC
2. Extract official .scar files from game .sgas to study AI patterns
3. Learn the RDO format - we'll need it for proper game mode mods
4. Add RDO support to our .sga building workflow
5. Reference the `agents.md` workflow guidance when writing .scar code

## Attribution
If we use patterns or knowledge derived from aoe4-mcp in our mods, include in README:
```
Built with aoe4-mcp by heart&move
https://github.com/heartmove/aoe4-mcp
```
(License: Apache-2.0)
