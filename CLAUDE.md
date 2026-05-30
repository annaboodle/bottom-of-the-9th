# Project Notes for Claude/Codex

This project is a Wherigo cartridge built with the `wheriflo` toolkit. The current cartridge is `bottom-of-the-9th/BottomOfThe9Th.lua`.

## Wheriflo version and local services

- Current pinned version as of 2026-05-29: `wheriflo==2.1.5`.

### Files that pin the wheriflo version — update ALL of these on every bump

The pin is duplicated across multiple files. When bumping `wheriflo`, search-and-replace the version in every entry below; do not assume one file is canonical. (Past bumps have left `.claude/launch.json` stuck on `0.6.4` and `README.md` stuck on `0.4.2`.)

- `.mcp.json` — project MCP config (the wheriflo MCP server used by Claude Code in this repo)
- `.codex/config.toml` — Codex MCP config (must match `.mcp.json`)
- `.claude/launch.json` — both launch configs (web player + copy editor)
- `CLAUDE.md` — the "Current pinned version as of …" line above, plus the inline version mention in the watch-mode bullet, plus this section's date
- `README.md` — the three `wheriflo==VERSION` snippets in the Building section

Sanity-check the bump:

```bash
grep -rn "wheriflo==" . --include="*.json" --include="*.toml" --include="*.md" 2>/dev/null | grep -v node_modules | grep -v ".git/"
```

All hits should report the same version (excluding historical issue-writeup `.md` files like `wheriflo-*-issue.md` which intentionally pin to the version where the bug was observed).

If `uvx` reports "no version of wheriflo==X.Y.Z" right after a fresh TestPyPI release, retry with `--refresh` — the index cache lags behind the JSON metadata for a few minutes after publication.


- Always start wheriflo services in watch mode so browser tools update after cartridge edits. As of `wheriflo==2.1.5`:
  - `wheriflo play` supports `--watch` (opt in).
  - `wheriflo edit` watches **by default** (use `--no-watch` to opt out). Issue #26 was implemented in 1.0.5 — both servers detect external `.lua` mtime changes within ~3s and bump `/api/poll` generation in lockstep, so a live `edit` + `play --watch` pair stay synchronized.
- Check the latest TestPyPI version with:

```bash
curl -s https://test.pypi.org/pypi/wheriflo/json
```

- Start the web player:

```bash
uvx --index-url https://test.pypi.org/simple/ --extra-index-url https://pypi.org/simple/ --from wheriflo==VERSION wheriflo play --watch bottom-of-the-9th/BottomOfThe9Th.lua
```

- Start the copy editor:

```bash
uvx --index-url https://test.pypi.org/simple/ --extra-index-url https://pypi.org/simple/ --from wheriflo==VERSION wheriflo edit --port 8001 bottom-of-the-9th/BottomOfThe9Th.lua
```

- Player URL: `http://localhost:8000/`
- Copy editor URL: `http://localhost:8001/`
- If a port is already in use, inspect with `lsof -iTCP:8000 -sTCP:LISTEN -P` or `lsof -iTCP:8001 -sTCP:LISTEN -P`, then stop the old PID before restarting.

## GitHub issue workflow

- `gh` is installed at `/opt/homebrew/bin/gh`; it may not be on PATH in sandboxed shells.
- Use the absolute path when needed:

```bash
/opt/homebrew/bin/gh issue view ISSUE --repo harrisonhoffman1/wheriflo --comments
```

- If commenting on a closed issue with unresolved behavior, reopen it.
- Continually update this `CLAUDE.md` file during future sessions when new project workflow lessons, wheriflo findings, GitHub issue statuses, local setup details, or recurring user preferences come up. Treat it as the living handoff for the next assistant, not a static note.
- When adding follow-up comments to closed issues, run:

```bash
/opt/homebrew/bin/gh issue reopen ISSUE --repo harrisonhoffman1/wheriflo --comment "Reopening because ..."
```

- Prefer adding focused comments to existing issues when the behavior is already covered. Create a new issue when the request has a distinct implementation/product shape.
- The wheriflo maintainer has access to `https://github.com/annaboodle/bottom-of-the-9th/` only as a repro/test cartridge. When mentioning that repo in issues, explicitly say not to make changes to the cartridge repo; fixes should be made in `wheriflo`.

## Wheriflo copy editor findings from this session

Relevant wheriflo issues created or updated:

- `#3` - Can't see selection options for questions. **Still broken in `wheriflo==1.0.5`** (verified 2026-05-25): `inputSelectPlayer.Choices = playerNames` (runtime-built array) is not statically inferred. Comment posted noting this is likely the same root cause as the remaining piece of #17 — both are dynamic data flow that the parser doesn't follow back to source.
- `#17` - Not all copy appearing in editor. **Partially fixed in `wheriflo==1.0.5`** (verified 2026-05-25): `pick({...})` variant pools now extract fully (big win — the "You pull into second standing up..." regression is resolved). Remaining bug: dynamic concatenations of the form `someVar .. [[longstring]]` produce entries with empty `text`/`raw` and `full_expr` truncated at `[[`. Four examples in our cartridge: `inputSwing.Text`, `inputLeadSize.Text`, two assignments to `inputRunnerPlay.Text`. Also flagged `variant_title: "Text"` mislabel on variant sets.
  - **Added 2026-05-26 comment:** user could not find the first-base safe variant `"Safe at first! You pull up on the bag, chest heaving..."` in the editor UI. In `wheriflo==1.2.1`, `/api/entries` does include it as a `variant_set` under `First Base -> OnEnter` at line 2984, so this is likely a UI discoverability/visibility issue rather than a pure parser miss.
  - **Added 2026-05-27 comment:** completion-code win `MessageBox` around `BottomOfThe9Th.lua:2416` is missing in `wheriflo==1.4.1`. The editor API has zero hits for exact substrings like `As the celebrations continue`, `Heads up`, `Catching it`, `post-game party`, `full code above`, and `TEST-CODE`. This is a mixed long-string + function call + runtime variable concatenation (`getSkipperName()`, `code`, `shortCode`) and is especially important because it is the final completion-code screen.
- `#18` - Dynamic string UI. Reopened because dynamic chips/expressions still need better presentation and inspection.
- `#19` - Dynamic copy variant explorer. Reopened again on 2026-05-22 because in `wheriflo 1.0.0` variant editors are full width but still look like raw/code textareas instead of the polished message editor.
- `#20` - Structured editor for trivia/question sets. **CLOSED — fully fixed in `wheriflo==1.0.5`** (verified + confirming comment posted 2026-05-25). `/api/entries` returns each trivia as a `trivia_group` with question + all 4 choices + marked answer.
- `#21` - Cartridge flow map with jump-to-section testing.
- `#22` - Duplicate sidebar section names scroll to the first matching section.
- `#23` - Order copy editor story entries by gameplay flow. **CLOSED — fully fixed in `wheriflo==1.0.5`** (verified + confirming comment posted 2026-05-25). Field priority is honored (Zone Name `field_priority: 0` before Description `field_priority: 1`); OnStart has `gameplay_order: 0` so it lands at the top of Story; sidebar now groups by category (Cartridge / Locations / Items / Questions & Choices) as a UX bonus.
- `#24` - Sidebar subitem order does not match main section order.
- `#25` - Clickable variable inspector for dynamic copy tokens. **`possible_values` shipped in `wheriflo==1.2.1`** (verified 2026-05-26) — `/api/variable-info` now returns possible_values + derivation + value_type with the badges (`DERIVED VALUE` / `RANDOMIZED` / `FINITE VALUES` / `COMPUTED TEXT`) all surfacing in the popover. `short` chip cleanly shows the 7 Mariner names. Follow-up comment posted 2026-05-26 covering two remaining areas: **(A)** visual hierarchy — possible values are item 5 of 6 in the popover, should be the headline with source/derivation/function collapsed into a "Dev info" disclosure; **(B)** four data-quality gaps in `possible_values` extraction: `pick({...})` variants not enumerated (`situationFlavor`, `batterFlavor`), composed templates show only static prefix (`countText` reports just "Count:"), long-string finite values render as empty bar (`tutorialText`), unreachable initial values pollute the list (`baseLabel` shows empty + FIRST/SECOND/THIRD when the empty is dead-on-arrival). Same content hierarchy should apply to the chip-click → modal once that lands (see #28 cross-reference).
- `#26` - Copy editor watch/live reload support. **Implemented in `wheriflo==1.0.5`** (verified 2026-05-25). Watch is on by default for `wheriflo edit`; concurrent `edit` + `play --watch` validated end-to-end (both servers' `/api/poll` generation increments in lockstep on external file touches). Left a follow-up comment asking for clarification on `--no-watch` semantics — at the API layer the generation counter still ticks; the flag likely controls the browser-side auto-refresh JS rather than server state.
- `#28` - **Filed 2026-05-26**: umbrella feature request for full-screen rendered message previews + variables side panel. The argument: the editor is fragment-oriented (one Lua expression = one entry) but the editor's job is screen-oriented (one Wherigo screen = one unit of copy to review). Headline anchor is the lead-size prompt screen (`inputLeadSize.Text` at lines 1210-1216) — one Wherigo screen currently scattered across 2 unrelated editor sections with 4 body strings dropped on the floor and the 3 button labels invisible. Depends upstream on #17 (parser long-string truncation) + #3 (choice-array inference); reuses #18 (chip primitive) + #25 (popover/modal). **Refined 2026-05-26 via follow-up comment**: sidebar gets a toggleable Screens / Variables mode (Screens is primary navigation, Variables is reference/audit); variable editing happens in a modal opened from either chip-click in the preview OR from Variables sidebar — same modal both ways. Replaces the original "inline variables side panel" editing concept.
- `#29` - **Filed 2026-05-26**: bug — copy editor `function_context` attribution ignores anonymous-function containers (zone OnEnter, callbacks). Discrete parser issue. Headline repro: 4 sibling variant blocks inside `zoneSecondBase.OnEnter` (anonymous function at line 2956) attributed to two different first-base named functions (`awardFirstBaseItem`, `startFirstBaseTrivia`) — even worse, inconsistent across siblings. Secondary repro on the lead-size case (line 1210 attributed to `offerMooseMagic` instead of `showPitchPrompt`). Fallback to last-seen named-function declaration is the root cause; fix should recognize anon-function assignments to table fields as new context.
- `#30` - **Filed 2026-05-26**: player bug — opening Map and clicking **Go Here** triggers the zone message/input panel behind the Leaflet map instead of above it. Looks like a stacking-context / z-index issue between the map container and the modal overlay.
- `#31` - **Filed 2026-05-27**: player/runtime bug — when a zone becomes active while the player is already inside it, the app can show "inside zone" but not fire `OnEnter` until the player leaves and re-enters. Repros: restart while standing in the dugout; start cartridge while already standing in the tunnel/alley. Requested either auto-triggering `OnEnter` on activation if already inside, or an official documented helper/pattern.
  - **Live iOS follow-up 2026-05-28:** still reproducible when Home Plate activates while the player is already inside it. The first cartridge workaround checked containment before activation; that failed on iOS. Updated helper now activates the zone first, then checks `zone:Contains(...)` and manually calls `OnEnter` if already inside. Comment posted on #31.
  - **Live iOS validation 2026-05-28:** the revised activate-first helper worked in the field. When Home Plate became active while the player was already inside/near the zone, the normal Home Plate flow triggered without requiring the player to leave and re-enter.
- `#32` - **Filed 2026-05-27**: builder-ergonomics feature request — surface the `Player.CompletionCode` pattern via three complementary shapes: (1) lint warning when a cartridge sets neither `cart.Complete = true` nor references `Player.CompletionCode`, (2) docs section explaining the canonical pattern + the per-player-per-download nature of codes + the wherigo.com upload-and-redownload testing flow, (3) starter template with a stub `showCompletion()` function. Filed after our cartridge nearly shipped with an `XXX` placeholder in the win MessageBox because the trap is non-obvious.
- `#33` - **Filed 2026-05-27**: player simulator feature request — map/location list should allow inspecting active location descriptions without teleporting. Real Wherigo lets players tap a location they are not at to see its title/description/distance; wheriflo currently only exposes a **Go Here** teleport action.
- `#34` - **Filed 2026-05-27**: item command simulator/client parity. Original report: simulator item detail did not render `ZCommand` action buttons. In `wheriflo==2.1.0` the simulator renders and fires the Golden Trident command, but a live iOS Wherigo test showed `cmdHoistTrident.OnClick` did not work. **Confirmed fixed in our cartridge on iOS 2026-05-28** by switching to the canonical named command shape (`item.Commands.cmdHoistTrident` + `function itemTrident:OncmdHoistTrident(target)`). Follow-up comment posted asking wheriflo to model/support/lint toward this real-client-compatible pattern.
- `#35` - **Filed 2026-05-27**: lint/build should flag `ZCommand.Enabled = true` because it makes official Groundspeak compile fail with HTTP 500 even though local build/validate pass.
- `#36` - **Filed 2026-05-29 / fixed in `wheriflo==2.1.3`, reopened 2026-05-29 for iOS matching docs**: player simulator bug — free-text `ZInput` (`InputType = "Text"`) originally displayed the prompt but rendered no text field or submit button. `2.1.3` added `/api/answer_text`, a web-player text field + Submit button, Enter-key submit, and fallback text input when choices are empty. **Live iOS finding 2026-05-29:** the entered text can be routed correctly only when compared directly inside `OnGetInput`; passing `answer` into helper/debug functions made it display as `nil` or produced bogus values like `LUACALL UPPER`. `Wherigo.NoCaseEquals`, `string.upper`, and `string.find` were all unsafe in our tests. Production answer gates should use literal `answer == "..."` comparisons inline in the handler.
- `#37` - **Filed 2026-05-29 / docs landed in `wheriflo==2.1.4`**: docs/design-guide request — recommend optional early on-location verification prompts for geocache-style cartridges that should discourage simulator-only play. Pattern: ask one friendly, diegetic question about a stable physical detail near the starting location; normalize answers; provide retries.
- `#38` - **Filed 2026-05-29**: docs/troubleshooting request — `uvx ... wheriflo compile` can fail in sandboxed agent environments when `uv` cannot access its default cache under `~/.cache/uv` (`Operation not permitted`). Build/validate may pass and the official compile can still succeed when rerun outside the sandbox, so document a local cache workaround such as `UV_CACHE_DIR=.uv-cache` or `/tmp/uv-cache` and make clear this is not necessarily a cartridge/Groundspeak failure.

Compile finding from 2026-05-27:

- Official Groundspeak compile has previously failed with HTTP 500 when a standalone `ZCommand` set `Enabled = true`; if a compile fails after command changes, retry without `Enabled` before changing behavior. For item commands, prefer the canonical builder/Urwigo-style shape: `item.Commands = { cmdName = Wherigo.ZCommand{...} }`, set `Text`, `CmdWith = false`, `EmptyTargetListText`, `Custom = true`, and `WorksWithAll = true`, then put behavior in `function item:OncmdName(target) ... end`. Do not rely on `cmd.OnClick` for real iOS/Garmin parity.

Specific observations:

- Prompt text and available choices should be shown together. Examples: `inputSwing.Text` with `inputSwing.Choices`, `inputLeadSize.Text` with `inputLeadSize.Choices`, `inputRestart.Text` with its multi-line choices.
- Dynamic selection prompts should also be grouped with options when inferable. Current example: `inputSelectPlayer.Text = text` plus `inputSelectPlayer.Choices = playerNames`; the editor currently shows only `Choose your Mariner:` as local text and separates player option descriptions elsewhere.
- Confirmed missing extraction in `wheriflo 1.0.1`: `You pull into second standing up. The outfielder gets the ball back in, but too late to make a play. Your teammate did the job.` does not appear in `/api/entries`.
- Trivia tables should be shown as structured units: `q`, all `choices`, and `answer`, with the correct answer visibly marked.
- Dynamic strings need full-message previews with protected variable chips, not broken separate cards.
- Clicking a variable chip like `short`, `strikes`, or `currentBatter` should ideally show source, meaning, and possible values if inferable.
- `pick({...})` groups should be represented as expandable variants, with all possible text variants visible and editable.
- Variant text fields should use the same polished, full-width editor style as regular message cards.
- Large sections such as `Story -> inputWinChoice` need subgrouping by source context/function, such as `handleWalk`, `handleBatterStrikeout`, `resolveBIPOut`, etc.
- Sidebar subitems must match the order of the main content.
- Duplicate sidebar labels need unique internal anchors so each row scrolls to its own section.
- Story copy should be navigable in likely gameplay order, not only source order. `OnStart` should appear near the beginning, not buried at the bottom.
- Within object sections, identity fields should come first: `ZONE NAME`, `ITEM NAME`, `CHARACTER NAME`, `TASK NAME`, then initial description, then runtime description updates.
- The copy editor should ideally have `--watch` or equivalent live reload behavior so external Lua edits refresh the editor and copy-editor saves reliably trigger the watched player.

## Cartridge-specific reminders

- Main file: `bottom-of-the-9th/BottomOfThe9Th.lua`.
- Current local services may leave state files such as `bottom-of-the-9th/.wheriflo-edits.json`; do not delete them unless the user asks.
- The working tree may contain user edits from the copy editor. Do not revert unrelated changes.
- Build/compile workflow from `SESSION-HANDOFF.md` still applies after cartridge source changes:

```bash
wheriflo build -> wheriflo compile -> wheriflo play
```

- The copy editor is for text review/editing; if it changes cartridge source, verify before overwriting or reverting anything.
- Apostrophes are okay inside Lua strings, especially double-quoted strings. The apostrophe restriction only applies to Lua comments in this project, so do not over-sanitize natural copy contractions.
- Do not put final cache coordinates in public docs, especially `README.md`. The coordinates can live in cartridge source and player-facing victory/item copy where gameplay reveals them, but not in documentation that someone can read before playing.
- For anti-simulator protection, prefer an early on-location verification question that can only be answered by physically visiting the starting area. Keep it diegetic and low-friction; this cartridge uses the hot dog vendor asking what is written on the metal disk at the player's feet before ballpark selection.
- For free-text `ZInput` answers on iOS, use direct literal comparisons inside the `OnGetInput` handler. Do not pass `answer` into helper functions for matching/debugging, and do not use `string.upper`, `string.lower`, `string.find`, `string.gsub`, `type`, `string.len`, or `Wherigo.NoCaseEquals` for production routing unless re-validated on real iOS. Live test evidence from 2026-05-29: a diagnostic helper displayed `raw answer: nil` for all attempts, but inline exact routing still correctly rejected `banana` and accepted `park dept` / `PARK DEPT`. For case-insensitive-ish acceptance, enumerate exact variants inline:
  `if answer == "park dept" or answer == "PARK DEPT" or answer == "Park Dept" then ... end`.
- When enabling a destination zone after a message/input callback, use `activateZoneOrEnterIfInside(...)` instead of directly setting `Active`/`Visible`. This preserves the normal arrival flow when the player is already standing inside the zone at the moment it becomes active. The helper intentionally activates the zone before checking containment because the iOS app may not report `zone:Contains(...)` accurately for inactive zones. This activate-first pattern was validated in a real iOS field test on 2026-05-28.

### Field selection flow

- The cartridge starts at `The Tunnel`, a shared spot between two physical baseball diamonds.
- The player chooses the field before gameplay: **Kingdome = north field**, **T-Mobile Park = south field**. This mapping reflects where the real stadiums were.
- The selected field re-anchors Dugout, Home Plate, First Base, Second Base, and Third Base via `setFieldCoordinates(field)`.
- Field choice is session-locked. `resetGameState()` should not clear `selectedField`; restarts and play-again flows keep the same field and send the player back to that field's dugout.
- Start copy should remain venue-agnostic until the tunnel choice. Win copy should respect `selectedField` so Kingdome runs do not mention T-Mobile Park.

### End-of-game flow

- After a win, `inputWinChoice` offers three branches: **Run it back** (same field, fresh lineup), **Switch ballparks** (flips `selectedField` + calls `setFieldCoordinates`, then restarts at the other field), and **Call it a night** (ends the session cleanly).
- "Call it a night" calls `deactivateAllBaseZones()` so the player can walk around the park without re-triggering the dugout. This replaces the old pattern of leaving `zoneDugout.Active = true` post-game, which caused accidental restarts.
- The replay affordance after "Call it a night" is the **Golden Trident** (`itemTrident`). It's granted on "Call it a night" (`Visible = true` + `MoveTo(Player)`) and shows a `Hoist the Golden Trident` command in the player's inventory.
- Trident lifecycle mirrors Moose Magic / Pine Tar: **granted** when earned, **consumed on use** (`Visible = false` inside `itemTrident:OncmdHoistTrident(...)`), **re-granted** on the next qualifying "Call it a night" win.
- Hoisting opens `inputHoistChoice` with two options: **Run it back** or **Switch ballparks**. No "End" option here — the player chose to re-engage by tapping the trident, so the path is "pick a field and restart."
- Loss flow (`inputRestart`) was intentionally not extended with switch-ballparks or trident; it still has its original three branches (`Try again same player` / `Return to player selection` / `End the cartridge`). Future revisit if desired.
- **Live iOS finding from 2026-05-28:** item command events are validated on the actual Wherigo iOS app when they use canonical builder-style wiring. `cmd.OnClick` did not fire on iOS, even though it worked in the wheriflo simulator. The working shape is a named command table entry (`item.Commands.cmdHoistTrident = Wherigo.ZCommand{...}`) plus an item method handler (`function itemTrident:OncmdHoistTrident(target) ... end`). iOS displayed the custom `Hoist the Golden Trident` button and ran the restart flow successfully. Use this pattern for all future actionable inventory items.
- Passive inventory items currently leave `Commands` unset instead of assigning `Commands = {}`. Live iOS testing on 2026-05-28 confirmed this did not hide the generic `Start` button on non-action items; the button appears to be official iOS app chrome/fallback behavior. Leave as-is for now because it is a minor UX oddity and not game-breaking.

## Baserunning out chance (formerly "timed run")

The cartridge has a baserunning out system that gives running between bases a low chance of failing — e.g., thrown out at first on an infield single, thrown out at home on a sac fly. The current implementation is in `isRunnerSafe(destination)` / `getRunnerOutBaseRate(reason, dest)` / `getRunnerOutMsg(reason, dest)`.

### Why it is NOT actually timed

Original intent: measure how long the player takes to walk between zone OnEnter events, fail them if too slow. This is impossible — the Wherigo sandbox blocks `os.time()` and no other time primitive is exposed to cartridge code. There is no way to measure elapsed real time from inside a cartridge.

### What was tried and did not work

First implementation used a stat-based dice surrogate, named `checkTimedRun()`:

```lua
score = timedRunDifficulty + runnerSpeed + floor(runnerIQ/2) + math.random(1, 6)
-- safe if score >= 5
```

The math is too lenient. Even the slowest player (Edgar / Naylor: speed 2, IQ 5) scores `0 + 2 + 2 + (1-6) = 5 to 10` — never failing the >= 5 threshold. The 4 failure messages never fired during real play. User IRL-tested on a baseball diamond (deliberately slow runs, deliberately fast runs) and was always safe — confirming the system was effectively a no-op.

### Current implementation (replacement)

`isRunnerSafe(destination)` rolls a per-scenario base out percent (looked up via `getRunnerOutBaseRate(advanceReason, destination)`), then adjusts by the runner's speed and IQ. Walks always return safe (skip the roll). big_hit to first always returns safe (the ball is in the gap, you are guaranteed at first).

Adjustment formula:

```lua
outChance = baseRate + (3 - runnerSpeed) * 2 + (4 - runnerIQ)
-- clamped to [1, 30]
```

Per-scenario base rates: see `getRunnerOutBaseRate` (typical range 4 to 15 percent). Per-scenario failure narrations: see `getRunnerOutMsg`, 18 unique variants across 8 scenario x destination combos plus 4 defensive per-base fallbacks.

### If revisiting

If a future Wherigo or wheriflo version exposes a time primitive (or a callback timing API), consider:
- Measuring zone OnEnter timestamp deltas to detect "too slow" baserunning.
- Forcing a short timed prompt during the run ("answer in N seconds") as a stand-in for real-time pressure.
- GPS-position-based speed estimation if Wherigo exposes raw player coordinates with timestamps.

Until then, the per-scenario probability approach is the cleanest stand-in. The base rates and stat adjustments in `getRunnerOutBaseRate` are tunable — bump them up or down by single digits to make baserunning feel riskier or safer.
