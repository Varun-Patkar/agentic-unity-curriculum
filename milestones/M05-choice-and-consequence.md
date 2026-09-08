# M05 · Choice & Consequence — *the pillar*

**Days 36–42 · 15–21 Oct 2026 · project: `Hearthfall`**

> This is the week Hearthfall becomes the game you described rather than a demo with dialogue in it.
>
> By Wednesday you'll have three choices in Vaskirk that are individually rational, well-argued, and profitable — and each of which quietly sets a flag that resolves hours later in a valley the player wasn't thinking about. You'll have Enid's letters. And you'll have a test that plays every branch of your game in under a second.

**You end holding:** the consequence engine, the Three Ledgers authored, and delayed feedback that lands.

**Read `reference/story-bible.md` §4–§6 before Day 36.** This milestone implements it directly.

---

## Day 36 — Choice nodes and conditions
**Thu 15 Oct · 60 min**

**Objective:** Choices that appear, hide, or lock based on world state — authored in data, with no hardcoded `if` anywhere.

**Why:** A gate you have to recompile to change is a gate you'll stop adjusting. Data-driven conditions are what make it possible to actually iterate on your narrative in November.

### Concepts (10 min)
- **Three ways to handle a failed condition, and they're narratively different:**
  - **Hidden** — the player never knows it existed. Use for choices that would confuse.
  - **Locked and visible** — *"[50 coin] Pay the toll."* Powerfully tells the player what their situation costs them.
  - **Available but different** — the same choice with different text based on state. The most sophisticated and the most work.
- **Condition composition.** `AllOf`, `AnyOf`, `Not` over primitives gives you everything without a scripting language.
- **Author-time validation beats runtime discovery.** A choice whose condition can *never* be true is a content bug. Your validator should find it.
- **Keep the primitive set small:** flag set/unset, coin at least, conscience below/above, quest in state, ledger contains choice. Six primitives cover a 15-hour RPG.

### Build (40 min)
1. Extend `Core/Dialogue/Condition.cs`: add `NotCondition`, `AllOfCondition`, `AnyOfCondition`, `ConscienceBelowCondition`, `LedgerContainsCondition`.
2. Add a `ChoiceVisibility` enum to `ChoiceOption`: `Hidden` or `LockedVisible`, with an optional `LockedText` explaining the requirement.
3. Update the runner to return three groups: available, locked-visible (with reason), hidden (excluded entirely).
4. Update `DialogueView` to render locked options greyed with the reason in brackets.
5. Extend the JSON format for conditions:
   ```json
   { "text": "Pay the toll.", "target": "through",
     "condition": { "type": "coinAtLeast", "value": 50 },
     "visibility": "lockedVisible", "lockedText": "50 coin" }
   ```
6. Extend the validator: warn on a choice node where *every* option could be unavailable simultaneously — that's a soft-lock waiting to happen.
7. Author a real conditional choice: the tollgate on the Wealdrun. Pay 50 coin, or take the long way, or (if you know Tam) mention his name.
8. Test each branch by manipulating `GameState` directly. No Play Mode needed — this is why M03 exists.

### Acceptance criteria
- [ ] Composite conditions work (`AllOf`, `AnyOf`, `Not`)
- [ ] Hidden vs locked-visible both render correctly
- [ ] Locked choices show their requirement
- [ ] Validator warns on potential dead-end choice nodes
- [ ] The tollgate choice is authored in JSON with three conditional routes
- [ ] Every branch is covered by a test

### Failure modes
- **Every choice is locked** → dead end. This is why the validator matters.
- **Condition JSON is unreadable** → your discriminator design is wrong. Fix it now, not in November.
- **Locked choices spoil the story** → be deliberate about which are hidden. Sometimes not-knowing is the point.

**Stretch:** Add a condition that checks *how long ago* a flag was set. "You told me that a month ago" is a line only a timestamped flag system can deliver — and you added timestamps as a Day 24 stretch.

**Commit:** `feat(core): composite conditions and choice visibility`

---

## Day 37 — The consequence engine: immediate and deferred
**Fri 16 Oct · 60 min**

**Objective:** An effect system where a choice made in Vaskirk fires a consequence in Hearthfall three hours later.

**Why:** **This is the single most important system in your game.** Immediate consequences are transactions. *Delayed* consequences are drama.

### Concepts (10 min)
- **Immediate effects** apply when the choice is made: coin changes, flag set, ledger entry recorded.
- **Deferred effects** are scheduled against a trigger and fire later. The trigger can be: entering a location, a quest reaching a state, another flag being set, or elapsed play time.
- **This is a rules engine, and it's the shape you already know** — a queue of pending consequences, evaluated against events. Nothing exotic.
- **Consequences must be *findable* when you're debugging.** Every pending consequence records which choice created it. On Day 40 you'll build a window that shows exactly that, and you will use it constantly.
- **The design rule from the story bible:** *foreseeable in hindsight, invisible in the moment.* The consequence must be a fair result of the choice, but the player must not be doing the arithmetic at the time.

### Build (40 min)
1. `Core/Consequence/PendingConsequence.cs` — `SourceChoiceId`, `Trigger`, `List<Effect>`, `Description` (for debugging and for your own sanity).
2. `Core/Consequence/ConsequenceTrigger.cs` — `OnEnterLocation(LocationId)`, `OnFlagSet(FlagId)`, `OnQuestState(QuestId, state)`, `AfterSeconds(float)`.
3. `Core/Consequence/ConsequenceEngine.cs` — holds the pending list, exposes `Schedule(consequence)` and `Evaluate(gameEvent)`. Fires matching consequences, applies their effects, removes them, raises `ConsequenceFired`.
4. Add a `ScheduleConsequenceEffect` to the effect types, so **a dialogue choice can schedule a future consequence directly from JSON.** That's the whole mechanism, in one effect type.
5. Wire `Evaluate` into the events that matter — location changes, flag sets, quest transitions.
6. **Author the proof:** taking the Assize in Vaskirk schedules `levy_raised` on entering Hearthfall in Act III. Write the test. Watch it pass.
7. Pending consequences must serialize — they're part of the save. Note this for Day 57.
8. Test: schedule, don't trigger, assert nothing happened. Then trigger. Assert it fired exactly once.

### Acceptance criteria
- [ ] Immediate and deferred effects both work
- [ ] Four trigger types implemented
- [ ] A dialogue choice can schedule a deferred consequence from JSON
- [ ] Consequences fire exactly once and are removed
- [ ] Every pending consequence knows which choice created it
- [ ] The Assize → raised levy chain is tested end to end

### Failure modes
- **Consequence fires twice** → not removed after firing, or `Evaluate` called from two places for one event.
- **Consequence never fires** → the trigger never occurs. This is why the debug window exists.
- **Ordering matters and isn't deterministic** → if two consequences fire on the same trigger, define the order explicitly. Non-determinism in narrative is a nightmare to debug.

**Stretch:** Add a `ConsequenceChain` — a consequence that schedules another consequence. That's how one choice ripples across three acts. Use sparingly; it is very easy to make unfollowable.

**Commit:** `feat(core): deferred consequence engine`

---

## Day 38 — Authoring the Three Ledgers
**Sat 17 Oct · 60 min**

**Objective:** The Assize, the Name, and the Contract — authored, wired, profitable, and quietly catastrophic.

**Why:** This is the content that makes the theme work. Today your systems stop being systems and start being *your game*.

### Concepts (10 min)
- **The rule that makes this land:** each choice must be **individually rational**. If the player can identify the corrupt option as corrupt in the moment, you've written a morality test, not a temptation.
- **Vance must be right.** The old tax rolls *are* decades out of date. Accuracy *is* defensible. He is not lying to you once, and he genuinely likes you. Re-read your homework paragraph from M03's buffer day — if it doesn't convince *you*, it won't convince a player.
- **The honest alternative must cost something real.** If refusing is free, refusing is not a choice. The player must feel poorer for the next two hours.
- **Each Ledger sets a flag, records conscience, adds coin, and schedules a deferred consequence.** Four effects. That's it. The machinery is small; the writing is everything.

### Build (40 min)
1. Author `vaskirk_vance_assize.json`. Vance's argument in full. Three options: take it (coin + flag + conscience −3 + schedule `levy_raised`), refuse (a smaller honest job, less coin), or defer (return later — the delay itself is characterful).
2. Author `vaskirk_the_name.json`. The Sheriff wants who's running grain past the tollgate. You know it's Tam. He *is* a criminal. Options: name him (large one-time coin + `tollgate_closed`), lie (risk), stay silent (nothing, and you stay poor).
3. Author `vaskirk_grain_contract.json`. Iselde's forward contract. Genuinely defensible — guaranteed prices *do* protect farmers. Options: broker it (coin + `harvest_sold`), decline, or negotiate a fairer price (less coin, half the conscience cost — reward the player for engaging rather than just refusing).
4. Add all flags to `KnownFlags`, all conscience weights to the ledger, all deferred consequences to Act III triggers.
5. **Playtest all three in the Console.** Take all of them. Then refuse all of them. Note how poor you feel in the second run — if you don't feel it, the coin numbers are wrong.
6. Balance the coin. Full corruption should be roughly **2–3× the honest run.** Enough to be genuinely tempting, not so much that refusing feels absurd.
7. Write the three conscience entry descriptions carefully. **These strings appear in Ending B, quoted back at the player.** They should be plain, factual, and devastating: *"You surveyed the Weald and took a surveyor's share."*

### Acceptance criteria
- [ ] All three Ledgers authored in JSON with full dialogue
- [ ] Each sets a flag, records conscience, grants coin, and schedules a deferred consequence
- [ ] Each has a genuine, costly honest alternative
- [ ] Full corruption is 2–3× the honest run in coin
- [ ] Conscience descriptions read well as epilogue quotes
- [ ] You personally found at least one of them tempting

### Failure modes
- **The corrupt option is obviously evil** → rewrite the NPC's argument. This is a writing problem, not a code problem, and it's the whole ballgame.
- **Refusing costs nothing** → then there's no choice. Make the honest path materially poorer.
- **The player can take all three with no friction** → consider making them slightly exclusive, or having one NPC comment on another. Friction creates awareness.

**Stretch:** Have Bran notice. One line, after the second Ledger: *"You're getting good at this."* No judgement, no mechanical effect. It will sit with the player for hours.

**Commit:** `content: the three ledgers authored`

---

## Day 39 — Enid's letters
**Sun 18 Oct · 60 min**

**Objective:** Three letters that arrive during Act II, whose tone is selected by how many Ledgers you've taken.

**Why:** The cheapest, highest-impact narrative system in the game. It's the entire consequence-feedback channel, it's pure text, and it's the thing that makes the ending *earned* rather than *sprung*.

### Concepts (10 min)
- **Diegetic feedback.** Not "reputation decreased". A letter from your sister that mentions food less than the last one did. See `reference/story-bible.md` §6.
- **They never accuse.** The player must connect it themselves. The moment Enid says "the taxes went up", you've turned subtext into a notification and lost everything.
- **Selection, not generation.** Three letter slots × three tone variants = nine short pieces of text. Don't build a templating system; write nine letters.
- **The tell is structural**: length, subject matter, and sign-off. Corrupt run letter 3 is four lines, mentions no food, and signs off differently. That's it. That's the whole design.
- **Delivery timing:** on entering Vaskirk's inn, or after N quest completions. Make it feel like the world's rhythm, not a scheduled event.

### Build (40 min)
1. `Core/Narrative/Letter.cs` — id, sender, body, arrival trigger, tone variant.
2. `Core/Narrative/LetterSystem.cs` — on trigger, select the variant by counting set Ledger flags (0 → warm, 1–2 → strained, 3 → cold) and raise `LetterArrived`.
3. **Write nine letters.** This is the work. Take the full 25 minutes on it. Letter 1 warm is easy; letter 3 cold should be four lines and take you longest.
4. `Unity/UI/LetterView.cs` — a distinct presentation from dialogue. A parchment panel, different font, no portrait, no typewriter (or a much slower one). It should feel like reading, not talking.
5. Schedule deliveries: letter 1 on arriving in Vaskirk, letter 2 after the first Ledger opportunity has passed, letter 3 before departing for Act III.
6. Store received letters in `GameState` so the journal can re-read them (Day 45).
7. **Playtest both extremes.** Clean run, then corrupt run. If the cold letter 3 doesn't make you uncomfortable, rewrite it.

### Acceptance criteria
- [ ] Nine letters written, three per slot
- [ ] Tone selected by Ledger count, never stated explicitly
- [ ] No letter mentions taxes, tollgates, or contracts directly
- [ ] Letters present differently from dialogue
- [ ] Received letters persist in `GameState`
- [ ] The corrupt letter 3 is four lines and one is a lie

### Failure modes
- **Enid explains the consequence** → delete that sentence. Every time.
- **Letters feel like quest updates** → they're not. No objectives, no calls to action. Just a sister writing.
- **Player misses them** → letters must be unmissable at arrival, and re-readable in the journal.

**Stretch:** Have the *player* be able to write back, briefly, with 2–3 options. Sending money home is a choice that costs coin and buys nothing mechanical. Watch how differently that plays.

**Commit:** `content: enid's letters with tone variants`

---

## Day 40 — Debug tooling: a state inspector
**Mon 19 Oct · 60 min**

**Objective:** A custom Editor window showing every flag, coin, the conscience ledger, and every pending consequence — live, with the ability to set state directly.

**Why:** You are about to have dozens of interacting flags and delayed consequences. Debugging that by playing is unaffordable at one hour a day. **This tool is why you'll still be able to work on this in December.**

### Concepts (10 min)
- **Editor windows** — `EditorWindow` in an Editor-only assembly (that's what `Hearthfall.Editor` from Day 22 is for). IMGUI is old and ugly and takes twenty minutes to learn; UI Toolkit is nicer and takes longer. **Use IMGUI today** — this tool needs to exist, not to be beautiful.
- **Reading live state** requires finding the `GameRoot` in the scene during Play Mode.
- **Write access is the killer feature.** Setting a flag from the window lets you jump to any narrative state instantly instead of replaying twenty minutes.
- **Time-travel debugging, cheap version:** a "snapshot" button that dumps `GameState` to JSON, and a "restore" that loads it back. Two hours of narrative testing saved per use.

### Build (40 min)
1. `Editor/HearthfallStateWindow.cs` — `EditorWindow` with `[MenuItem("Hearthfall/State Inspector")]`.
2. Sections: **Coin** (current + lifetime), **Flags** (all set flags, with a toggle each), **Conscience** (ledger entries with weights and descriptions), **Pending Consequences** (source choice, trigger, description), **Quests** (empty until M06).
3. `Repaint()` on update so it's live during Play Mode.
4. **Write access:** toggle any flag, set coin, clear the ledger.
5. Snapshot/restore buttons that serialize `GameState` to a file in `Temp/`.
6. **Scenario buttons** — this is the highest-value part: "Jump to Act III, clean run", "Jump to Act III, all three Ledgers taken", "Jump to Act III, mixed". One click each. You will press these hundreds of times over the next ten weeks.
7. Use it immediately: take the Assize, and watch the pending consequence appear in the window. That's the moment the tool proves itself.

### Acceptance criteria
- [ ] Editor window shows flags, coin, ledger, and pending consequences live
- [ ] Flags can be toggled and coin set from the window
- [ ] Snapshot and restore work
- [ ] At least three scenario jump buttons exist
- [ ] Lives in the Editor-only assembly and doesn't ship in a build
- [ ] You used it to verify a deferred consequence was scheduled

### Failure modes
- **Window is blank in Play Mode** → can't find `GameRoot`, or you're not repainting.
- **Editor code breaks the build** → it's in the wrong assembly. Editor asmdef, Editor platform only.
- **Spending the whole hour making it pretty** → don't. It's a diagnostic tool. Ugly and working beats pretty and unfinished.

**Stretch:** Add a live view of the current dialogue node ID and the conversation history stack. When a branch goes wrong, this tells you exactly where you are in the graph.

**Commit:** `tools: hearthfall state inspector window`

---

## Day 41 — Automated playthroughs
**Tue 20 Oct · 60 min**

**Objective:** Tests that play your entire game — every branch — in under a second, and assert the right ending is reached.

**Why:** You have three endings and a branching middle. Manually verifying that a specific combination of choices produces Ending B takes twenty minutes and you'd have to do it after every content change. This makes it instant.

### Concepts (10 min)
- **Because Core is engine-free, a "playthrough" is just a sequence of method calls.** No Play Mode, no scene, no rendering.
- **A `PlaythroughDriver` test helper** wraps the runner: `Talk("osric_farewell").Choose(0).Choose(2).Go("vaskirk")`. Fluent, readable, and it becomes documentation of your narrative.
- **Test the paths that matter**, not all 2^n combinations: the three canonical endings, each Ledger in isolation, and every known soft-lock risk.
- **Assert on outcomes, not implementation:** which ending, which flags, what Enid's letter 3 says.
- **This catches content regressions.** Change one JSON file in November, run the suite, and know instantly whether you broke a branch three acts away.

### Build (40 min)
1. `Core.Tests/PlaythroughDriver.cs` — fluent wrapper over runner, consequence engine, and state. Methods: `Talk(id)`, `Choose(index)`, `ChooseText(substring)` (more readable and survives reordering), `EnterLocation(id)`, `AdvanceTime(seconds)`.
2. Write the **clean run** test: refuse all three Ledgers, reach Act III, assert `Ending.SetTheSwordDown`, assert the family flags are healthy, assert Enid's letter 3 is the warm variant.
3. Write the **full corruption** test: take all three, assert `Ending.TheTithe`, assert `levy_raised`, `tollgate_closed`, `harvest_sold` all fired, assert the cold letter.
4. Write the **mixed** test: take one, assert `Ending.PeasantKnight`.
5. Write a **soft-lock guard**: assert no reachable choice node exists where all options can be simultaneously unavailable.
6. Write a **content integrity test**: every conversation file loads and validates. This one will save you repeatedly.
7. Run the suite. Under a second. **Deliberately break a JSON target and watch the exact test fail.**

### Acceptance criteria
- [ ] `PlaythroughDriver` exists and reads clearly
- [ ] All three canonical endings are covered by tests
- [ ] Every conversation validates in a test
- [ ] Soft-lock guard passes
- [ ] Suite runs in under a second
- [ ] A deliberate content break produces a named, obvious failure

### Failure modes
- **Tests break every time you edit dialogue** → you're asserting on node IDs. Assert on outcomes and use `ChooseText` instead of indices.
- **The driver becomes as complex as the game** → keep it thin. It calls the same APIs the view does.
- **Endings don't exist yet** → they're Day 61. Write the tests against a stub `EndingSelector` today; it'll make Day 61 trivial.

**Stretch:** A random-walk fuzzer that makes random choices 1000 times and asserts it always reaches *some* ending and never throws. It will find a soft-lock you didn't know about.

**Commit:** `test(core): automated branch playthroughs`

---

## Day 42 — BUFFER
**Wed 21 Oct**

- **Catch up.**
- **Write.** More dialogue, more letters, the minor choices from the story bible checklist. Your systems are ready and content is now the bottleneck — that's a good place to be.
- **Balance.** Play the corrupt run and the clean run back to back. Does refusing hurt enough? Does taking feel good enough?
- **Story homework:** Enid's cold letter 3 and the other 3–4 minor choices are both due around now.
- **Rest.**

### Milestone review

Run `/review`. Ask it specifically: *is the consequence chain traceable?* If you can't answer "why did this fire?" from the debug window in under thirty seconds, that's a must-fix.

### Where you are — read this bit

You have built the thing you said you wanted.

Three choices in a city. Each defensible. Each profitable. Each setting a flag that fires in a valley hours later, in a letter that never accuses you of anything. A ledger that will quote your own decisions back at you in an epilogue. And a test suite that verifies all of it in under a second.

**That is a real narrative engine.** Most people who set out to make a choice-driven RPG never build one, because they get stuck on combat or art or the perfect inventory system. You built the pillar first, which is the correct order and almost nobody does it.

The rest of this project is putting a game around it.

**Commit:** `docs: M05 complete — choice and consequence`
