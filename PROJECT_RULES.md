# PROJECT_RULES.md
These rules are MANDATORY for every session. If a request conflicts
with them, STOP and ask before proceeding.

## 1. Project Summary
- Game: 2D platformer roguelike in the spirit of Super Mario Bros 3
  (tight, momentum-based platforming + run-based item builds).
- Engine: Unity 2D (URP), C#, new Input System, Cinemachine.
- Design pillar: Collect enemy abilities by stomping them (an original take on copy-abilities).
- All characters, art, music, names and level layouts must be
  ORIGINAL. Never copy Nintendo assets, names or level layouts.

## 2. Session Protocol (follow every time)
1. Read PROJECT_RULES.md, DESIGN.md and PROGRESS.md before anything.
2. Work on ONE part/feature per session. Do not touch unrelated code.
3. Before writing code, state your approach and list the files you
   will create or change. Wait for approval on anything that adds a
   new system or changes architecture.
4. After finishing, update PROGRESS.md (see Section 8) and tell me
   exactly how to test the work in play mode.
5. Never mark a task complete unless it was actually tested or you
   say clearly that it is UNTESTED.
6. If something is unclear, ask ONE concise question rather than
   guessing on design decisions.
7. DESIGN.md is read-only unless I approve a change. Every approved
   change is logged in its Change Log with a YYYY-MM-DD HH:MM
   timestamp from the system clock.

## 3. Scope Lock (MVP)
- MVP = 1 character, 15-20 items, 3 enemy types, 20 chunks, 1 boss,
  a 5-level run with branching nodes (Normal, Shop, Rest, Treasure).
- Do NOT add content, features, or systems beyond the current part
  or beyond the MVP until I explicitly say the MVP gate has passed.
- No "bonus" features, extra polish, or speculative abstractions.

## 4. Key Technical Decisions (do not revisit without asking)
- Fixed timestep 60 Hz (Time.fixedDeltaTime = 1/60).
- Player is a CUSTOM KINEMATIC controller (Rigidbody2D Kinematic,
  BoxCast, resolve X axis then Y axis). Do not switch to dynamic
  physics.
- Tiles are 16x16 px, Pixels Per Unit = 16, 1 tile = 1 Unity unit.
- Levels are Tilemaps; interactive tiles are prefabs.
- Items, enemies, chunks, characters, worlds are ScriptableObjects
  (data-driven). Adding content should not require code changes.
- Systems communicate through events (EventBus), not direct
  cross-references.
- Player stat changes go through the StatModifier system, never by
  editing stats directly.
- Chunks are authored as plain-text grids (one character per tile) and imported into Unity by an editor tool; the validator works on the text grids using the base kit only (no items or abilities).

## 5. Architecture and Folder Layout
- Assets/Scripts/{Core,Player,Enemies,Items,Level,UI,Audio}
- Assets/Data/{Items,Enemies,Chunks,Characters,Worlds}
- Assets/{Prefabs,Scenes,Art/Placeholder}
- One responsibility per script. Prefer composition over inheritance
  (inheritance only for EnemyBase and BossBase).
- No new global singletons without approval. Existing allowed
  managers: GameStateManager, RunManager, AudioManager, EventBus.
- No magic numbers: every tuning value lives in a config
  ScriptableObject or a serialized field with a sensible default.
- Do not duplicate logic. Search for an existing system before
  writing a new one and reuse it.

## 6. Code Style
- C#, PascalCase for public members and types, _camelCase for
  private fields, camelCase for locals.
- Comment WHY, not WHAT. Keep methods short and named for intent.
- No dead code, no commented-out code, no leftover debug logs
  (use a DebugLog wrapper that can be toggled off).
- Unsubscribe from events in OnDisable/OnDestroy.
- Avoid per-frame allocations in Update/FixedUpdate (no LINQ, no new
  collections in hot paths).

## 7. Unity-Specific Rules
- NEVER hand-edit scene (.unity) or prefab (.prefab) YAML. Use the
  Unity MCP editor tools. Never delete or rename .meta files.
- Do not move, rename, or delete assets outside of Unity's editor
  tools (it breaks GUID references).
- After script changes, check the Unity console for errors and fix
  them before reporting completion.
- Input goes through the Input System; no direct Input.GetKey calls.
- Gameplay physics logic runs in FixedUpdate; visuals/UI in Update.

## 8. PROGRESS.md Update Protocol
At the end of EVERY session, update PROGRESS.md:
- Update Current Status and the MVP checklist.
- Move finished items to Completed.
- Add decisions to Decision Log and bugs to Known Issues.
- Add tuning changes to Tuning Log.
- Add a Session Log entry (what changed, files touched).
- Rewrite Next Steps and Notes For Next Session.
Keep PROGRESS.md under 300 lines; move old Session Log entries to
PROGRESS_ARCHIVE.md when it grows.
Timestamp rules:
- EVERY entry added to PROGRESS.md must be stamped
  YYYY-MM-DD HH:MM (my local time, 24-hour).
- Get the time by running a terminal command (PowerShell:
  Get-Date -Format "yyyy-MM-dd HH:mm" | Mac/Linux:
  date "+%Y-%m-%d %H:%M"). Never guess or invent a timestamp; if you
  cannot run the command, ask me for the current time.
- Never edit timestamps on existing entries. History is append-only.
- Session Log entries record both a START and an END timestamp.

## 9. Backups (this project does NOT use Git)
- Never run Git commands or create Git files (.git, .gitignore).
- I back up the project folder by hand. At the end of every
  completed part, remind me to make a backup (a zipped copy of
  Assets, Packages, ProjectSettings and the .md files in the
  project root (do NOT include Library, Temp, Logs, DebugLogs or
  UserSettings)).
- Before any large refactor, or any change touching more than 5
  files, tell me FIRST and wait for me to say "backup done".
- When I confirm a backup, record it in PROGRESS.md
  ("Last backup") with a timestamp.
- Never delete files or folders without asking. Prefer adding
  new files over rewriting large existing ones.

## 10. Testing Rules
- Core systems (StatModifier, Inventory, seeded generation, chunk
  validation, drop tables) get automated tests.
- Each part ends with a short manual test checklist for me.
- Never claim a fix works without stating how it was verified.

## 11. Do NOT
- Do not rewrite working code "to clean it up" unless asked.
- Do not change files outside the current task's scope.
- Do not silently change a Key Decision or tuning default.
- Do not invent features, items, or enemies I did not request.
- Do not skip ahead to future parts of the build plan.
