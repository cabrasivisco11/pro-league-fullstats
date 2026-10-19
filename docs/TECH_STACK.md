# Technology and architecture — Pro League FullStats

This document describes the **observed role** of each technology in the FullStats edition. It does not distribute the mod's source code, scene, or game files.

| Technology or tool | Role in the project | Evidence boundary |
| --- | --- | --- |
| C# | Match rules, timing, pause state, scoring, and result presentation. | Local working-copy scripts were inspected but are not included here. |
| Unity / Unity UI | `Canvas`, `Text`, and `RectTransform` elements for the HUD clock, scoreboard, break display, and results. | Static inspection shows integration points; gameplay capture is needed to demonstrate appearance. |
| DynamicCSharp / Battlehub | Compilation of scene-embedded `CustomScript` components in the Mod Editor; scene and metadata serialization through Battlehub RTSaveLoad. | External compilation does not replace compilation or execution in the editor. |
| Game RPC interface | The host synchronizes configuration, time, points, statistics, and round transitions with clients. | The calls' presence does not by itself prove online synchronization quality. |
| Steam Workshop | Distribution of the published edition. | The [public item](https://steamcommunity.com/sharedfiles/filedetails/?id=3821094797) confirms its title, description, and credits, not performance outcomes. |
| Git, working copies, and backups | Version control for this documentation and recoverable states during mod investigation. | Mod backups remain outside this repository. |
| Codex | Assistance with analysis, code generation or adaptation, debugging, and documentation. | Credited explicitly; the work is not described as entirely hand-written. |

## Component flow

```text
Host selects match format
  └─ TimeManager_PL: configuration, clock, pause, transitions, RPCs
       ├─ RoundManager: current round, duration, wins
       ├─ PointsManager_PL: scores and pause/break guard
       ├─ ShowControls_PL: scoreboard, timer, break countdown
       └─ PLShowWinner: statistics, final result, tie decision
```

The round clock is separate from the game's native room timer. The edition sends the remaining round time to clients and displays it in the HUD. `PointsManager_PL` checks match, pause, and break state before accepting scoring events. At round end, statistics are captured before player data is reset so the between-round display retains scores and bonuses.

## Compilation and runtime constraints

The local record distinguishes two checks:

1. **External compilation:** C# compiled with `csc.exe` against installed *Drunken Wrestlers 2* DLLs. A historical check returned exit code 0 with two pre-existing warnings.
2. **Battlehub compilation:** Mod Editor logs recorded `compiled OK` for five scripts in an earlier check. A later FullStats AllModes experiment logged all six and `GameStarted()` without a C# error. That later log did not identify the selected scene, so it could not independently confirm the Title Fight UI or match flow.

In one compatibility pass, UI symbols including `TextAnchor`, `FontStyle.Bold`, and `HorizontalWrapMode` were avoided, as were direct `DW2.` namespace references. Long scripts also risked truncation when pasted into a `CustomScript` field, so experiments verified code embedded in a scene copy. A historical recompile produced `Type mismatch!` messages without a C# error. Those messages were recorded rather than treated as automatic proof of either failure or success in gameplay.

## Debugging method

1. Trace the room timer and round timer as separate systems.
2. Compare readable scripts with code embedded in the working scene.
3. Compile externally to catch syntax and unavailable references.
4. Open the scene in the Mod Editor, inspect logs, and check HUD and transitions in the game.
5. Test host and client behavior for scores, bonuses, timing, pauses, and round changes.
6. Preserve backups before changes and record which observations each test supports.

The [technical case study](CASE_STUDY.md) covers the problems, implementation, and validation limits in more detail.
