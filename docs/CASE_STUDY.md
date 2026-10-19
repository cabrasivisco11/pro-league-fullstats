# Technical case study — Pro League Rounds FullStats Edition

## Objective and scope

The FullStats edition extends **Nixagorn's original Pro League Rounds** for *Drunken Wrestlers 2*. The goal was to present consistent round timing, a readable HUD timer, detailed match statistics, and clear results across match modes. The work involved script integration, Unity UI, debugging, validation, and Steam Workshop release. Codex assisted with analysis, code generation or adaptation, and documentation.

The published item describes three formats: Unranked (three 3-minute rounds), Ranked (three 5-minute rounds), and Title Fight (five 5-minute rounds), with 30-second breaks. When a match remains tied, the host can choose a 1-minute extra round or declare a draw.

## Problems investigated

1. **Two clocks, one match.** The game has a native room timer; the mod also needs a timer for each round. An early report described a reset between rounds, but that behavior was **not reproduced during the initial investigation**. Scene lifecycle and the native timer were hypotheses, not confirmed causes.
2. **State shared by timing and scoring.** The pause state must stop the round clock and prevent scoring. Breaks must also block scoring, independently of what the HUD displays.
3. **Results lost during a break.** In a FullStats experiment, the complete statistics panel disappeared before the break ended while a simplified summary remained. The local investigation found two contributing mechanisms: `SetIntermission` disabled the original result text, and the original results routine cleared it about 25 seconds after displaying it.
4. **HUD compatibility.** The custom MM:SS clock needed a stable position and readable digits without disrupting the game's native HUD.
5. **Tie resolution.** An early version of the original mod examined locally did not explicitly resolve a final tie by total points. That was a code-review finding, not a measured match outcome. Nixagorn's original pause fix and tie-break improvements remain credited; this edition documents their integration with FullStats.
6. **Different compilation environments.** An external C# compiler can catch syntax and reference issues, but it cannot establish that DynamicCSharp/Battlehub will compile a scene or that the game will behave correctly.

## Architecture and implementation

```text
Host selects a mode → countdown → round timer → round results
                                           ├→ break → next round
                                           └→ tie decision → extra round or draw
```

The main responsibilities observed in the local working copy were:

| Component | Responsibility |
| --- | --- |
| `TimeManager_PL` | Match configuration, clock, pause state, transitions, and RPC synchronization. |
| `RoundManager` | Round duration, current round, wins, and match state. |
| `PointsManager_PL` | Points and bonuses; rejects scoring while paused or between rounds. |
| `ShowControls_PL` | On-screen controls, scoreboard, and per-round clock. |
| `PLShowWinner` | Round statistics, final results, and tie decision display. |

The host sends remaining round time to clients, and Unity UI displays it on the HUD. `PointsManager_PL` checks `MatchPaused` and `IsIntermission` before accepting points. During the FullStats break experiment, names, round scores, wins, and bonus text were captured before player data was reset; an RPC sent that snapshot to clients. The original result text was kept visible for the intended part of the break and cleared before the next round.

The FullStats panel work was first documented for the three-round modes and then extended to Title Fight in a later AllModes experiment. The later log alone did not establish that the five-round mode looked correct or completed correctly in gameplay.

## Debugging and compatibility work

**Timer and UI.** The investigation traced when the room timer and round timer start, pause, synchronize, and end. HUD work examined the game's existing `UnityEngine.UI.Text` component and layout behavior. External compilation was paired with in-editor and in-game checks because static code inspection could not prove the final appearance.

**DynamicCSharp/Battlehub.** The scene stores code in `CustomScript` components. One historical compatibility pass avoided direct `DW2.` namespace references and UI symbols that were problematic in that environment, including `TextAnchor`, `FontStyle.Bold`, and `HorizontalWrapMode`. Long scripts also created a risk of truncation when pasted into an editor field, so experiments compared readable scripts with the code embedded in a working scene copy.

**Logs.** A historical pre-publication check compiled six scripts externally with `csc.exe` against installed game DLLs, returning exit code 0 with two pre-existing warnings. The Battlehub log showed `compiled OK` for five named components; `PointsManager_PL` was embedded and passed external compilation but had no individual log confirmation in that check. A requested recompile also logged two `Type mismatch!` messages without a C# compiler error. A later AllModes experiment logged six scripts as `compiled OK` and `GameStarted()` without a C# error, but its log did not name the selected scene. These are specific observations, not proof that every published behavior passed.

**Version safety.** Working copies and backups preserved previous scene states. Script text was compared with the scene's embedded `CustomScript` fields, and Battlehub metadata was checked in the local process. No scene, script, or game asset is redistributed in this repository.

## Validation and outcome

The process separated four questions: does the C# compile externally; does Battlehub compile the scene; does the UI and match flow work in the game; and do host and client receive consistent state? Passing one stage did not imply passing the next.

The project owner reported that the latest version [published on Steam Workshop](https://steamcommunity.com/sharedfiles/filedetails/?id=3821094797) worked in their tests. This repository does not include the final session's raw logs, screenshots, or test matrix. It therefore makes no quantitative claims about test coverage, performance, users, or downloads. The [demo plan](DEMO_SCRIPT.md) lists the real gameplay evidence still to capture for this repository.

| Evidence source | What it supports | Limit |
| --- | --- | --- |
| Local working-copy scripts and notes | Component responsibilities and implemented control paths. | Reading code is not a gameplay test. |
| Historical compiler and Battlehub logs | Specific successful compilations and recorded warnings. | They do not independently validate the final release or online play. |
| Project owner's test report | The latest published version worked in their tests. | No final-session log or complete test matrix is included here. |
| [Steam Workshop item](https://steamcommunity.com/sharedfiles/filedetails/?id=3821094797) | Public title, feature description, publication, and credits. | A listing is not an independent technical test. |

## Lessons

- Keep a reported bug, a suspected cause, and a reproduced failure distinct.
- Treat timing, scoring, HUD, and transitions as one shared-state flow.
- Use external compilation for fast feedback, then verify inside the game's script runtime and in actual matches.
- Preserve recoverable working states and document what each test did and did not establish.
- Make authorship and third-party rights explicit when extending an existing mod.

## Credits and rights

The original mod, original pause fix, and tie-break improvements are credited to **Nixagorn** on the Workshop item. The published FullStats edition credits **PREGUIÇA** for the custom HUD timer, statistics, visual adjustments, and integration. **Codex assisted** with analysis, generated or adapted code, debugging, and documentation. This case study does not claim sole or entirely manual authorship and does not grant a license to third-party code or assets.
