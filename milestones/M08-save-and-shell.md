# M08 · Save/Load & The Game Shell

**Days 57–63 · 5–11 Nov 2026 · project: `Hearthfall`**

> A game you can't quit and come back to isn't a game, it's a demo. This week you put a shell around Hearthfall: serialization, save slots, scene flow, a main menu, and a pause screen. Then, on Monday and Tuesday, you write the ending gate and the three epilogues — the thing every system since M03 has been quietly building toward.
>
> The ending gate is about forty lines of pure C#. That's the payoff of keeping the rules in Core.

**You end holding:** a game you can quit and come back to, with a menu, a pause screen, and three endings wired to the ledgers.

---

## Day 57 — Serializing Core state, and versioning it before you need to
**Thu 5 Nov · 60 min**

**Objective:** `GameState` round-trips to JSON and back with nothing lost — including the conscience ledger and pending consequences — and the file says which schema version it is.

**Why:** Save serialization is where architecture decisions get invoiced. Yours is good, so today is mostly mechanical — provided you get versioning in before there are saves in the wild.

### Concepts (10 min)
- **Core serializes itself.** `Hearthfall.Core` has no `using UnityEngine`, so `JsonUtility` is off the table — and it couldn't handle your data anyway: no dictionaries, no polymorphism, no nullables, no properties.
- **`System.Text.Json` vs Newtonsoft.** STJ is fast, modern, and strict; polymorphism needs a converter or a type discriminator. Newtonsoft (`com.unity.nuget.newtonsoft-json`) handles polymorphism with `TypeNameHandling` and is battle-tested in Unity. Pick one, write it in `CONVENTIONS.md`, don't revisit it. Recommended: Newtonsoft for pragmatism, with **explicit discriminators**, not `TypeNameHandling.All` — that setting is a deserialization vulnerability and it bakes assembly names into your save files.
- **Version field first, always.** `"schemaVersion": 1` at the root. Day one. The migration you'll need is on Day 68 when you rename a flag and every existing save breaks.
- **A migration is a function `JObject → JObject`**, applied in sequence from the file's version to the current one. Boring, small, and it's why your saves survive the rest of the curriculum.
- **The trap:** serializing derived state. If `ConsequenceEngine` caches a computed list, serialize the inputs and rebuild the cache on load. Two sources of truth in a save file is a bug factory.
- **Polymorphic collections are your hard case:** `PendingConsequence`, `Condition`, `Effect`. Those are the ones the tests must prove.

### Build (40 min)
1. Add your chosen JSON package. Newtonsoft: **Window > Package Manager > + > Add package by name** — `com.unity.nuget.newtonsoft-json` (verify this in Unity 6). Reference it from the `Hearthfall.Core` asmdef.
2. `Core/Persistence/SaveData.cs` — a DTO wrapping `SchemaVersion`, `SavedAtUtc`, a display label, and the `GameState`.
3. `Core/Persistence/SaveSerializer.cs` — `string Serialize(GameState)` and `GameState Deserialize(string)`. No file I/O in here; that's Day 58's job and it keeps this testable.
4. Add discriminators to your polymorphic types. A `$kind` string property on the base and a converter that switches on it.
5. `Core/Persistence/Migrations/` — an `ISaveMigration` with `FromVersion` and `Apply`, plus a runner that chains them. Write zero migrations. Just build the road.
6. Serialize the conscience ledger explicitly: `ChoiceId`, `Weight`, `Description`, `AtSeconds`. Every field, no exceptions — Day 62 quotes `Description` verbatim.
7. Test: build a `GameState` with two pending consequences, four conscience points, coin with `LifetimeEarned`, an active quest, and set flags. Save, load, assert deep equality field by field.
8. Test: a payload with `schemaVersion: 0` throws a clear, named exception rather than a `NullReferenceException` three frames later.

### Acceptance criteria
- [ ] Core serializes without referencing UnityEngine
- [ ] `schemaVersion` present in every saved file
- [ ] Migration chain exists and runs, even with zero migrations registered
- [ ] Pending consequences round-trip with their concrete types intact
- [ ] Conscience ledger round-trips with descriptions verbatim
- [ ] A deep-equality save/load test passes in EditMode

### Failure modes
- **Polymorphic list deserializes as base type** → missing discriminator or unregistered converter. Assert on concrete type in the test, not just count.
- **Private fields silently dropped** → STJ ignores fields and private setters by default. Newtonsoft is more forgiving. Either way, test the whole object.
- **Dictionary keys mangled** → non-string keys need a converter. Consider whether that dictionary should be a list of records.
- **Tests pass, the real save doesn't** → your test `GameState` is simpler than a played one. Build the test state with `PlaythroughDriver` instead of by hand.

**Stretch:** A round-trip fuzz test — run `PlaythroughDriver` through three different branches, save/load each, assert the post-load state produces identical subsequent behaviour.

**Commit:** `feat(core): versioned gamestate serialization`

---

## Day 58 — Save slots, autosave, load, and what belongs in a save
**Fri 6 Nov · 60 min**

**Objective:** Three manual slots plus an autosave, written to disk, listed with timestamps, loadable — and unbreakable by a corrupt file.

**Why:** Everyone builds save/load. Almost nobody builds it so it survives a crash mid-write. That's an hour of work and it's the difference between a hobby project and a product.

### Concepts (10 min)
- **`Application.persistentDataPath`** is the only writable path you can rely on across platforms. On Windows it's under `AppData/LocalLow/<Company>/<Product>`. Never write next to the executable.
- **What belongs in a save:** Core state. Flags, quest log, conscience ledger, coin, pending consequences, current scene id, player position. That's it.
- **What doesn't:** anything you can rebuild. Enemy positions, particle state, UI state, camera state. If it's regenerable from Core plus scene data, it's not save data — it's a bigger file and a worse bug.
- **Atomic writes.** Write to `slot1.tmp`, flush, close, then `File.Replace` or delete-and-move onto `slot1.json`. A crash mid-write then destroys a temp file instead of a ten-hour playthrough.
- **Never crash on load.** Corrupt file, truncated file, file from a future version, file from a different game — all four must produce a readable message and a working menu, not a stack trace.
- **The trap:** autosaving on a timer. Autosave on *events* — scene transition, quest completion, major choice resolved. A timed autosave will eventually fire mid-dialogue and save you into a state you can't cleanly resume.

### Build (40 min)
1. `Unity/Persistence/SaveFileStore.cs` — the only class in the project that touches the filesystem. Read, write, delete, enumerate. It calls Day 57's serializer.
2. Slot naming: `save_1.json`, `save_2.json`, `save_3.json`, `save_auto.json` under `Path.Combine(Application.persistentDataPath, "saves")`.
3. Implement the atomic write. Temp file, then replace.
4. `SaveSlotInfo` — slot index, timestamp, schema version, a label like "Vaskirk · Day 3", and `IsCorrupt`. Enumerating slots must never throw; a slot that fails to parse comes back flagged.
5. Autosave triggers: scene transition, quest completed, a Three Ledgers choice resolved. Explicit calls from Core event handlers, not a coroutine.
6. Wire an in-game quick save/load to F5/F9 through the Input System for your own testing. Temporary; keep or cut on Day 60.
7. Corruption test: save, truncate the file to half its bytes, load. Then overwrite it with `{}`. Then with `not json at all`. All three must be handled.
8. Verify the folder in Explorer. Open a save in an editor and read it. You should be able to see your own conscience ledger in plain text.

### Acceptance criteria
- [ ] Saves land in `persistentDataPath`, never the project folder
- [ ] Three manual slots plus autosave, each with timestamp and label
- [ ] Writes are atomic
- [ ] Autosave fires on events, not on a timer
- [ ] Three corruption cases load without throwing
- [ ] Loading a save restores flags, quests, ledger, and coin correctly

### Failure modes
- **Save works in the Editor, not in a build** → you used a project-relative path. `persistentDataPath` only.
- **Load leaves the old state around** → you mutated `GameState` instead of replacing it and re-wiring subscribers. Rebuild through `GameRoot`.
- **Autosave during dialogue produces an unresumable state** → gate autosave on "not mid-conversation", or serialize the dialogue cursor too. Pick one now.
- **Slot list throws on a bad file** → catch per slot, not around the loop.

**Stretch:** Keep a rolling backup — before overwriting `save_1.json`, move the old one to `save_1.bak`. Ten lines, and it has saved real shipped games.

**Commit:** `feat: save slots with atomic writes and corruption handling`

---

## Day 59 — Scene flow and a game manager that survives scene loads
**Sat 7 Nov · 60 min**

**Objective:** Move between Hearthfall, the Wealdrun, and Vaskirk without losing state, with a loading screen instead of a frozen frame.

**Why:** Scene transitions are where beginners' state management dies. Yours won't, because Core doesn't live in a scene — but you still have to prove it.

### Concepts (10 min)
- **Two approaches to persistence.** `DontDestroyOnLoad` marks an object immortal; a **bootstrap scene** stays loaded and you load gameplay scenes *additively* on top. Use the bootstrap scene.
- **Why bootstrap wins:** deterministic startup order, no "which scene did I press Play in" problem, no duplicate-manager bug when you return to the menu, and objects that are visible in a scene you can inspect rather than floating in a `DontDestroyOnLoad` pseudo-scene.
- **`GameRoot` lives in the bootstrap scene.** It constructs Core once, on `Awake`, and outlives every gameplay scene. Still not a singleton — the scene owns it and hands references down.
- **Async loading:** `SceneManager.LoadSceneAsync(name, LoadSceneMode.Additive)` returns an `AsyncOperation` you can await for a progress bar. `allowSceneActivation = false` lets you hold at 90% until you're ready — note that it caps at 0.9, which trips everyone once.
- **Unloading matters.** Load additive, then `UnloadSceneAsync` the old one, then optionally `Resources.UnloadUnusedAssets()`. Forgetting the unload is a slow memory leak you'll notice in week three.
- **Position restore is view concern, not Core rule.** Core stores "scene id and coordinates". The scene loader places the player. Core doesn't know what a Transform is and it stays that way.

### Build (40 min)
1. Create `Scenes/Bootstrap.unity` containing only `GameRoot`, the persistent canvas, and the audio listener. Set it as the first scene in **File > Build Profiles** (Unity 6 renamed Build Settings — verify this).
2. `Unity/App/SceneFlow.cs` — `LoadGameplayScene(string sceneId)`: show loader, load additive, unload previous, place player, hide loader.
3. A `SceneLoader` canvas in Bootstrap: a fade, a label, a progress bar driven by `AsyncOperation.progress` (remember the 0.9 cap).
4. `SpawnPoint` component with an id. Core stores the spawn id or coordinates; the loader finds the match and positions the player.
5. Handle "load a save from the menu": `GameRoot` rebuilds Core from `SaveData`, then `SceneFlow` loads the saved scene and positions the player.
6. Handle "return to main menu": unload gameplay scenes, tear down Core cleanly, unsubscribe everything. Leaked event subscriptions across a menu round-trip are the classic bug here.
7. Add a minimum loader display time of ~0.5s. An instant flash reads as a glitch.
8. Test the full loop three times: play, save, menu, load, play. Watch the Console for duplicate-subscription warnings.

### Acceptance criteria
- [ ] A bootstrap scene loads first and persists
- [ ] Gameplay scenes load additively and old ones unload
- [ ] `GameRoot` and Core survive every transition, with no duplicates
- [ ] Loading screen shows real progress and a minimum duration
- [ ] Loading a save puts the player in the right scene at the right position
- [ ] Menu → game → menu → game leaves no leaked subscriptions

### Failure modes
- **Two `GameRoot`s** → you both marked it `DontDestroyOnLoad` and put it in bootstrap. Pick one; pick bootstrap.
- **Progress bar sticks at 90%** → that's the documented cap with `allowSceneActivation = false`. Remap 0–0.9 to 0–1.
- **Events fire twice after returning to the menu** → old subscribers weren't disposed. Give `GameRoot` an explicit teardown and call it.
- **Player spawns at origin** → spawn id mismatch, or you positioned before the scene finished activating.

**Stretch:** A `Scene` enum or id constants file generated from your scene list, so a typo is a compile error instead of a black screen.

**Commit:** `feat: bootstrap scene flow with async loading`

---

## Day 60 — Main menu, pause, settings: the shell that makes it a product
**Sun 8 Nov · 60 min**

**Objective:** A main menu, a pause menu, and a settings screen that persists — the frame that makes Hearthfall feel like software someone shipped.

**Why:** The shell is the first thing anyone sees on itch.io, and it's ten days away. It's also low-risk work you can do while tired, which is why it's on a Sunday.

### Concepts (10 min)
- **Settings are not save data.** Settings persist per machine, across playthroughs, and survive deleting all saves. `PlayerPrefs` is genuinely fine for this — it's small, keyed, and platform-appropriate.
- **`Time.timeScale = 0` pauses physics, animation, and `Update` deltas — not `Update` itself.** Anything using `Time.deltaTime` stops; anything using `Time.unscaledDeltaTime` keeps going. Your pause UI must use unscaled.
- **The gotcha with teeth:** `yield return new WaitForSeconds(x)` never completes while paused. Your typewriter effect from M04 will hang forever. `WaitForSecondsRealtime` is the fix — go find every coroutine that should run while paused.
- **Continue must be disabled with no save**, not hidden and not broken. Grey it out and it reads as "you haven't played yet" rather than "this button is broken".
- **Every screen needs an obvious way out**, and Esc should always do the least destructive thing available.
- **The trap:** applying settings only on close. Apply live — text speed and volume should change as the slider moves, or the player can't tell what they're setting.

### Build (40 min)
1. `Scenes/MainMenu.unity`, loaded additively over Bootstrap. Title, then New Game · Continue · Settings · Quit.
2. Continue: query `SaveFileStore` for the most recent valid slot. None or all corrupt → disabled.
3. New Game: if any save exists, confirm. Then construct fresh Core through `GameRoot` and load Act I.
4. Pause menu: Esc during gameplay. Resume · Save · Load · Settings · Quit to Menu. Set `Time.timeScale = 0`, switch the input context to `UI`, and restore both on resume.
5. `Unity/Settings/GameSettings.cs` — text speed (Day 33), master/music/SFX volume, marker mode (Day 46), resolution, fullscreen. Load from `PlayerPrefs` on boot, save on change.
6. Wire volumes to an AudioMixer with exposed parameters. Remember mixer volume is decibels: `Mathf.Log10(v) * 20`, and guard `v = 0`.
7. Resolution via `Screen.SetResolution` and `Screen.fullScreenMode`, populated from `Screen.resolutions`. **Verify this** — Unity 6 changed some fullscreen mode behaviour.
8. Audit every coroutine for `WaitForSeconds` and convert the ones that must run while paused. Then pause mid-typewriter and confirm it still finishes.

### Acceptance criteria
- [ ] Main menu with all four options, Continue correctly disabled with no save
- [ ] Esc pauses, pause menu works, resume restores timescale and input context
- [ ] Settings persist across a full application restart
- [ ] Settings apply live, not on close
- [ ] No coroutine hangs while paused
- [ ] Quit to menu tears down cleanly and New Game after it works

### Failure modes
- **Timescale stays 0 after resume** → an early return or an exception skipped the restore. Restore in a `finally`, or on menu close rather than on button click.
- **UI unresponsive while paused** → animated UI or an input action map driven by scaled time.
- **Volume slider feels broken at the low end** → you set the mixer linearly. Use the log conversion.
- **Settings reset every launch** → `PlayerPrefs.Save()` never called, or you're reading before the load runs.

**Stretch:** A key rebinding screen. The Input System has `PerformInteractiveRebinding` built in, so it's less work than it sounds and it's a real accessibility win.

**Commit:** `feat: main menu, pause, and persistent settings`

---

## Day 61 — The ending gate: reading Coin, Conscience, and Roots at the final door
**Mon 9 Nov · 60 min**

**Objective:** One pure function that takes a `GameState` and returns an `EndingId` — plus tests proving every ending is reachable and no run falls through the cracks.

**Why:** Sixty days of architecture exist so that today is small. The whole thematic payload of Hearthfall passes through about forty lines of C# with no Unity in sight.

### Concepts (10 min)
- **`EndingSelector` is a pure function.** `GameState` in, `EndingId` out. No I/O, no events, no side effects, no randomness. This makes the most important decision in your game trivially testable.
- **Three inputs, one gate.** `Coin.LifetimeEarned` — what you *took*, not what you kept. `Conscience` — the ledger's weighted sum, computed here and nowhere else. `Roots` — `FatherHealth`, `ValleyLevy`, `HarvestSold`, `EnidsLastLetterTone`.
- **Thresholds are data, not literals.** `EndingThresholds` as a record you can tune, override in tests, and adjust after playtesting without touching logic. You will change these numbers four times.
- **The final choice is not symmetric.** A and C both end on a choice node — stay or go. **B has no choice node.** The gate resolves B *before* the final scene is authored, and the final scene simply isn't offered. That asymmetry is the entire design.
- **Selection must be total.** Every possible state maps to exactly one ending. Order your checks so B is tested first (it's the most specific), then A, then C as the default. C being the fallback is correct — it's meant to be the common ending.
- **Still no meter.** The player never sees Coin thresholds, conscience weights, or which ending they're tracking toward. Not in the journal, not in the debug build's HUD, nowhere the player can reach.

### Build (40 min)
1. `Core/Endings/EndingId.cs` — `SetTheSwordDown`, `TheTithe`, `ThePeasantKnight`.
2. `Core/Endings/EndingThresholds.cs` — a record with the coin and conscience boundaries, plus a `Default` static. Comment each number with the *design intent*, not the value.
3. `Core/Endings/EndingSelector.cs` — `static EndingId Select(GameState state, EndingThresholds t)`. Compute the conscience sum from the ledger inline; do not cache it on `GameState`.
4. B first: conscience below floor **and** lifetime coin above the high bar **and** at least two of the Three Ledgers taken. A second: conscience above ceiling and coin above the "enough to clear the debt" line. C otherwise.
5. `Core/Endings/EndingGate.cs` — resolves the ending, records it into `GameState`, and exposes `OffersFinalChoice` (false for B). The final scene reads this and branches.
6. Author the final-choice dialogue node for A and C: stay, or go. It's one node with two options and it decides nothing except which epilogue plays — the ending was already chosen.
7. Tests, one per ending, driven by `PlaythroughDriver`: a clean run, a take-all-three run, and a middling run. Assert the `EndingId` **and** `OffersFinalChoice`.
8. A totality test: generate a grid of coin/conscience/flag combinations and assert every one returns a valid ending and never throws.

### Acceptance criteria
- [ ] `EndingSelector.Select` is pure — no fields, no I/O, no Unity
- [ ] Thresholds live in a tunable data record
- [ ] All three endings reachable and covered by a playthrough test
- [ ] Selection is total across a generated combination grid
- [ ] Ending B reports `OffersFinalChoice == false`
- [ ] Nothing in the UI exposes any ledger value to the player

### Failure modes
- **Every test run returns C** → your thresholds are wrong, not your logic. Log the actual computed values in the test and calibrate against a real playthrough.
- **B unreachable** → the coin bar is above what a full-corruption run actually earns. Have `PlaythroughDriver` print `LifetimeEarned` for the take-all-three run and set the bar from that.
- **Ending changes after you look at it** → something in the gate mutated state. It's a query. Make the method static and the argument read-only if you have to.
- **Debug window leaks the ending** → fine in the Editor window, never in a player-facing build.

**Stretch:** Add the current ending projection to your Day 40 State Inspector, Editor-only. Enormously useful for the next two weeks of tuning, and it must never ship.

**Commit:** `feat(core): ending selection gate`

---

## Day 62 — Writing the three epilogues, and making B land without a score screen
**Tue 10 Nov · 60 min**

**Objective:** Three playable epilogues, one of which quotes the player's own conscience ledger back at him, and an "endings seen" record in the journal.

**Why:** This is the day the game means something. Everything else is machinery for this.

### Concepts (10 min)
- **Epilogues are content, and you already have the system.** They're dialogue sequences with conditions. No new tech today except one piece: text that interpolates from the ledger.
- **Ending B's power is specificity.** Not "you were ruthless" — *"you named Tam Ferrier in the third week of Lent."* Pull the real `ConsciencePoint.Description` strings, in chronological order by `AtSeconds`, and print them as a list. That is the score screen, except it's in his own handwriting and it isn't scored.
- **Direction for B: subtract.** No music sting. No score screen. No villain reveal. No "The End" card that congratulates. A cold, procedural read of the ledger, a long pause, and the final image. Silence is the loudest tool you own and this is the one place to use it.
- **A and C need warmth and restlessness respectively.** A: he is going to be poor and tired forever and it was the right trade — write it plainly, don't oversell it. C: the money in Enid's hands, one meal, the road in the morning. Bittersweet, not tragic.
- **Author the descriptions carefully.** Every `AddConsciencePoint` call across the whole game now doubles as epilogue text. Go back and rewrite the lazy ones today, while it's cheap.
- **The trap:** explaining the ending. If the epilogue tells the player what it meant, you've written a lecture. Show the levy figure, the empty field, the last letter. Let him do the arithmetic.

### Build (40 min)
1. `Content/Endings/ending_a.json`, `ending_b.json`, `ending_c.json` — your existing dialogue format, launched by `EndingGate`.
2. A `ConscienceLedgerRecital` effect type: emits the ledger's descriptions, ordered by `AtSeconds`, one per beat, with a deliberate pause between each. Used only by B.
3. Audit every `ConsciencePoint.Description` in your content. Rewrite each as a plain declarative sentence in past tense — it must read well standing alone in a list.
4. Write B's final image. This was your M08 story homework; if it isn't decided, decide it in the first ten minutes. Whatever it is, it should be an image of the valley or the house, not of him.
5. Write A and C. A: the sword above the hearth, the roof, the debt cleared. C: the coin in Enid's hands, one meal, gone by morning.
6. Presentation: fade to black between beats, unscaled timing, no skip on the first pass. Disable the pause menu during epilogues.
7. `EndingsSeen` in `GameState` — a set of `EndingId`. Recorded on completion, shown as a third journal tab, undiscovered ones as locked slots with no hints.
8. **Play B end to end.** Do not skip it because you wrote it. If it doesn't land on you, it won't land on anyone; fix it now.

### Acceptance criteria
- [ ] All three epilogues playable through the ending gate
- [ ] B recites real conscience-ledger descriptions in chronological order
- [ ] B has no score screen, no music sting, and no final choice
- [ ] Every conscience description reads as a standalone sentence
- [ ] `EndingsSeen` persists in the save and shows in the journal
- [ ] You played B and it landed

### Failure modes
- **B's recital reads as a bug list** → your descriptions are system messages, not prose. Rewrite them; it's the fix and it's fast.
- **The ledger is empty in B** → conscience points weren't recorded on the corrupt path. Check with the State Inspector mid-run, not after.
- **Fades hang** → `WaitForSeconds` again. Unscaled everywhere in the epilogue.
- **Ending feels rushed** → it's too fast. Double every pause and play it again. Almost always the right call.

**Stretch:** One Kokoro-voiced line at the end of B, flat and unemotional. One line, not a scene. If it's the wrong side of melodramatic, cut it without regret.

**Commit:** `content: three endings with conscience recital`

---

## Day 63 — BUFFER
**Wed 11 Nov**

- **Catch up.** Days 57–62 are dense and two of them are content days.
- **Tune the ending thresholds.** Play a take-all-three run and a clean run and set the numbers from real values, not guesses.
- **Story homework:** **Ending B's final image** was due by M08. If Day 62 shipped a placeholder, fix it today — it's the last frame of your game.
- **Polish the shell.** The main menu is your itch.io screenshot in seven days.
- **Rest.** Ship week starts tomorrow.

### Milestone review

Run `/review`. Ask specifically whether any ending logic lives outside `EndingSelector` — a UI check like "if coin > 500 show the other button" anywhere in `Hearthfall.Unity` means the gate is no longer the single source of truth, and it will drift.

### Where you are

**Hearthfall is a game you can quit and come back to.** It has a menu, a pause screen, saves that survive a crash mid-write, scene flow that doesn't leak, and three endings selected by a forty-line pure function that a test suite proves is total.

You have written an ending that refuses to congratulate the player, and it works because you kept a ledger of descriptions instead of an integer. That decision was made on Day 34 and it paid off yesterday. That's what the architecture was for.

63 days in, 56% through. Tomorrow starts **M09: ship week.** Bug triage, performance, a build that runs on a machine that has never had Unity installed, a store page, screenshots — and on **Day 70, a public itch.io release** of the 2D vertical slice. Strangers will play this. That's a different kind of week and it starts in the morning.

**Commit:** `docs: M08 complete — save/load, shell, and endings`
