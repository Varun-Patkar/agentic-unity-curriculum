# M05 · Choice & Consequence — *the pillar*

**Days 36–42 · 15–21 Oct 2026 · project: `Hollowbrook`**

> This is the week Hollowbrook becomes the game you described rather than a demo with dialogue in it.
>
> By Wednesday you'll have four choices threaded through Hollowbrook's mystery, each with an understandable immediate trade-off and a visible consequence later in town. You'll also have a test that plays every branch in under a second.

**You end holding:** the consequence engine, Hollowbrook's four central choices authored, and delayed town reactions that land.

**Read `reference/story-bible.md` §4–§6 before Day 36.** This milestone implements it directly.

---

## Day 36 — Choice nodes and conditions
**Thu 15 Oct · 60 min**

**Objective:** Choices that appear, hide, or lock based on world state — authored in data, with no hardcoded `if` anywhere.

**Why:** A gate you have to recompile to change is a gate you'll stop adjusting. Data-driven conditions are what make it possible to actually iterate on your narrative in November.

### Concepts (10 min)
- **Three ways to handle a failed condition, and they're narratively different:**
  - **Hidden** — the player never knows it existed. Use for choices that would confuse.
  - **Locked and visible** — *"[Requires Pike's files] Show Mara the evidence."* Powerfully tells the player what they have not discovered.
  - **Available but different** — the same choice with different text based on state. The most sophisticated and the most work.
- **Condition composition.** `AllOf`, `AnyOf`, `Not` over primitives gives you everything without a scripting language.
- **Author-time validation beats runtime discovery.** A choice whose condition can *never* be true is a content bug. Your validator should find it.
- **Keep the primitive set small:** flag set/unset, quest in state, decision recorded, preparation flag set, bargain term set. Five primitives cover this game.

### Build (40 min)
1. Extend `Core/Dialogue/Condition.cs`: add `NotCondition`, `AllOfCondition`, `AnyOfCondition`, `DecisionRecordedCondition`, `PreparedTownCondition`, and `BargainTermCondition`.
2. Add a `ChoiceVisibility` enum to `ChoiceOption`: `Hidden` or `LockedVisible`, with an optional `LockedText` explaining the requirement.
3. Update the runner to return three groups: available, locked-visible (with reason), hidden (excluded entirely).
4. Update `DialogueView` to render locked options greyed with the reason in brackets.
5. Extend the JSON format for conditions:
   ```json
   { "text": "Show Mara the copied files.", "target": "publish",
     "condition": { "type": "flagSet", "value": "pike_files_found" },
     "visibility": "lockedVisible", "lockedText": "Pike's files" }
   ```
6. Extend the validator: warn on a choice node where *every* option could be unavailable simultaneously — that's a soft-lock waiting to happen.
7. Author a real conditional choice: give Deputy Pike's files to Mara, or return them to Mayor Vale. Hide the evidence-specific wording until the files have actually been found.
8. Test each branch by manipulating `GameState` directly. No Play Mode needed — this is why M03 exists.

### Acceptance criteria
- [ ] Composite conditions work (`AllOf`, `AnyOf`, `Not`)
- [ ] Hidden vs locked-visible both render correctly
- [ ] Locked choices show their requirement
- [ ] Validator warns on potential dead-end choice nodes
- [ ] The deputy's-files choice is authored in JSON with both routes
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

**Objective:** An effect system where a choice made during the investigation changes Hollowbrook scenes later.

**Why:** **This is the single most important system in your game.** Immediate consequences are transactions. *Delayed* consequences are drama.

### Concepts (10 min)
- **Immediate effects** apply when the choice is made: a flag changes, a decision is recorded, preparation or bargain terms are updated.
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
6. **Author the proof:** giving Pike's files to Mara schedules `town_knows_truth` when Alex next enters Town Square. Write the test. Watch it pass.
7. Pending consequences must serialize — they're part of the save. Note this for Day 57.
8. Test: schedule, don't trigger, assert nothing happened. Then trigger. Assert it fired exactly once.

### Acceptance criteria
- [ ] Immediate and deferred effects both work
- [ ] Four trigger types implemented
- [ ] A dialogue choice can schedule a deferred consequence from JSON
- [ ] Consequences fire exactly once and are removed
- [ ] Every pending consequence knows which choice created it
- [ ] The files-to-Mara → informed-town chain is tested end to end

### Failure modes
- **Consequence fires twice** → not removed after firing, or `Evaluate` called from two places for one event.
- **Consequence never fires** → the trigger never occurs. This is why the debug window exists.
- **Ordering matters and isn't deterministic** → if two consequences fire on the same trigger, define the order explicitly. Non-determinism in narrative is a nightmare to debug.

**Stretch:** Add a `ConsequenceChain` — a consequence that schedules another consequence. That's how one choice ripples across three acts. Use sparingly; it is very easy to make unfollowable.

**Commit:** `feat(core): deferred consequence engine`

---

## Day 38 — Authoring Hollowbrook's four central choices
**Sat 17 Oct · 60 min**

**Objective:** Four understandable decisions authored and wired, each changing both immediate state and a later scene or epilogue.

**Why:** This is the content that makes the theme work. Today your systems stop being systems and start being *your game*.

### Concepts (10 min)
- **No option is labelled good or evil.** The immediate trade-off is clear; the delayed result is uncertain but fair in hindsight.
- **Vale's case must be credible.** The bargain genuinely protected Hollowbrook. Returning evidence, sealing the mine, or avoiding panic can all be defensible choices.
- **Each choice records a `Decision`, updates `PreparedTown` or `BargainTerms` where appropriate, and schedules a deferred consequence.** The machinery is small; the writing is everything.

### Build (40 min)
1. Author `choice_deputy_files.json`: give the evidence to reporter Mara, or return it to Mayor Vale. Later: broad public warning versus a more orderly evacuation.
2. Author `choice_mine_entrance.json`: seal the entrance after rescuing Eli, or leave it open to investigate deeper. Later: fewer creatures escape versus learning the bargain's real terms.
3. Author `choice_marked_resident.json`: hide June from the town, or hand her to Vale for protection. Later: June aids the final ritual, escapes, or becomes part of the bargain.
4. Author `choice_emergency_siren.json`: warn everyone immediately, or stay silent to avoid panic. Later: a messy evacuation versus more people caught unaware.
5. Add typed IDs and concrete descriptions to `Decisions`; update `PreparedTown` and `BargainTerms` only where the fiction supports it; schedule each delayed reaction.
6. **Playtest both routes through all four choices in the Console.** The immediate argument for each side must make sense without exposing ending logic.
7. Write decision descriptions as plain facts suitable for later dialogue and epilogues: *"Alex gave Pike's files to Mara Bell."*

### Acceptance criteria
- [ ] All four central choices authored in JSON with both routes
- [ ] Each records a decision and schedules a delayed consequence
- [ ] Preparation and bargain-term flags match the story bible
- [ ] Every immediate trade-off is understandable without a morality label
- [ ] Decision descriptions read well when cited later
- [ ] Both options in every choice are defensible

### Failure modes
- **One option is obviously correct** → strengthen the immediate cost or Vale's argument. This is a writing problem, not a code problem.
- **A choice only changes an ending flag** → add a visible town reaction before the finale.
- **The player cannot explain the trade-off** → rewrite the scene before adding more consequences.

**Stretch:** Let Deputy Pike notice who received her files. One restrained line can make the town feel observant without showing a notification.

**Commit:** `content: four central hollowbrook choices authored`

---

## Day 39 — Delayed town reactions
**Sun 18 Oct · 60 min**

**Objective:** The four central choices produce delayed, diegetic reactions in Hollowbrook before their ending consequences resolve.

**Why:** Consequences feel earned when the town changes before the epilogue. Reactions turn stored state into observed cause and effect without exposing a morality meter.

### Concepts (10 min)
- **Diegetic feedback.** People gather outside the diner, deputies redirect traffic, claw marks appear near the road, June is absent from a familiar spot. No `reputation changed` notification.
- **Selection, not generation.** Author small scene variants keyed to concrete decisions. Do not build a procedural town simulator.
- **React before resolving.** A reaction shows that the world noticed; the final consequence still belongs to the last night and epilogue.
- **Use multiple channels:** changed NPC lines, props, crowd placement, radio chatter, and journal wording.

### Build (40 min)
1. `Core/Consequence/TownReaction.cs` — reaction id, source decision id, trigger, and presentation keys for dialogue/props/crowd state.
2. Extend the consequence engine to emit `TownReactionAvailable` while keeping the content keys in Core and presentation details in Unity.
3. Author at least one delayed reaction for each central choice: Mara's story spreads; the mine road changes; June's status is noticed; the siren changes crowd behaviour.
4. `Unity/Narrative/TownReactionPresenter.cs` applies the scene variant by key when Town Square or Old Mine Road loads.
5. Store fired reaction IDs in `GameState` so save/load cannot replay or forget them.
6. Add short NPC lines that cite the concrete event without explaining its hidden value.
7. **Playtest opposite routes.** The town should visibly differ before the finale while both paths remain coherent.

### Acceptance criteria
- [ ] All four central choices have at least one delayed town reaction
- [ ] Reactions cite decisions, never a hidden score
- [ ] At least two presentation channels are used
- [ ] Fired reaction IDs persist in `GameState`
- [ ] Reactions fire once and survive save/load
- [ ] Opposite routes produce visibly different but coherent town states

### Failure modes
- **NPC explains the system** → delete that sentence. Let the scene show the consequence.
- **Reaction fires immediately** → schedule it at a later location or quest transition.
- **Reaction repeats on every load** → persist its fired ID and make application idempotent.

**Stretch:** Add local radio chatter that changes after the siren choice. Audio makes the same state change feel town-wide.

**Commit:** `content: delayed hollowbrook town reactions`

---

## Day 40 — Debug tooling: a state inspector
**Mon 19 Oct · 60 min**

**Objective:** A custom Editor window showing flags, decisions, preparation, bargain terms, profile history, and pending consequences live, with the ability to set state directly.

**Why:** You are about to have dozens of interacting flags and delayed consequences. Debugging that by playing is unaffordable at one hour a day. **This tool is why you'll still be able to work on this in December.**

### Concepts (10 min)
- **Editor windows** — `EditorWindow` in an Editor-only assembly (that's what `Hollowbrook.Editor` from Day 22 is for). IMGUI is old and ugly and takes twenty minutes to learn; UI Toolkit is nicer and takes longer. **Use IMGUI today** — this tool needs to exist, not to be beautiful.
- **Reading live state** requires finding the `GameRoot` in the scene during Play Mode.
- **Write access is the killer feature.** Setting a flag from the window lets you jump to any narrative state instantly instead of replaying twenty minutes.
- **Time-travel debugging, cheap version:** a "snapshot" button that dumps `GameState` to JSON, and a "restore" that loads it back. Two hours of narrative testing saved per use.

### Build (40 min)
1. `Editor/HollowbrookStateWindow.cs` — `EditorWindow` with `[MenuItem("Hollowbrook/State Inspector")]`.
2. Sections: **Flags**, **Decisions**, **Prepared Town**, **Bargain Terms**, **Playthrough History**, **Pending Consequences**, and **Quests** (empty until M06).
3. `Repaint()` on update so it's live during Play Mode.
4. **Write access:** toggle flags, add/remove test decisions, edit preparation and bargain terms, and set profile eligibility.
5. Snapshot/restore buttons that serialize `GameState` to a file in `Temp/`.
6. **Scenario buttons** — "Prepared break-bargain run", "Compromised keeper run", and "First-run skip eligible". One click each.
7. Use it immediately: give Mara the files and watch the pending reaction appear in the window.

### Acceptance criteria
- [ ] Editor window shows decisions, preparation, bargain terms, profile history, and pending consequences live
- [ ] Narrative state can be changed from the window
- [ ] Snapshot and restore work
- [ ] At least three scenario jump buttons exist
- [ ] Lives in the Editor-only assembly and doesn't ship in a build
- [ ] You used it to verify a deferred consequence was scheduled

### Failure modes
- **Window is blank in Play Mode** → can't find `GameRoot`, or you're not repainting.
- **Editor code breaks the build** → it's in the wrong assembly. Editor asmdef, Editor platform only.
- **Spending the whole hour making it pretty** → don't. It's a diagnostic tool. Ugly and working beats pretty and unfinished.

**Stretch:** Add a live view of the current dialogue node ID and the conversation history stack. When a branch goes wrong, this tells you exactly where you are in the graph.

**Commit:** `tools: hollowbrook state inspector window`

---

## Day 41 — Automated playthroughs
**Tue 20 Oct · 60 min**

**Objective:** Tests that play your entire game — every branch — in under a second, and assert the right ending is reached.

**Why:** You have three endings and a branching middle. Manually verifying that a specific combination of choices produces `NewKeeper` takes twenty minutes and you'd have to do it after every content change. This makes it instant.

### Concepts (10 min)
- **Because Core is engine-free, a "playthrough" is just a sequence of method calls.** No Play Mode, no scene, no rendering.
- **A `PlaythroughDriver` test helper** wraps the runner: `Talk("vale_briefing").ChooseText("Eli").Go("old_mine_road")`. Fluent, readable, and it becomes documentation of your narrative.
- **Test the paths that matter**, not all $2^n$ combinations: the three canonical endings, each central choice in isolation, and every known soft-lock risk.
- **Assert on outcomes, not implementation:** which ending, which decisions, preparation flags, bargain terms, and town reactions.
- **This catches content regressions.** Change one JSON file in November, run the suite, and know instantly whether you broke a branch three acts away.

### Build (40 min)
1. `Core.Tests/PlaythroughDriver.cs` — fluent wrapper over runner, consequence engine, and state. Methods: `Talk(id)`, `Choose(index)`, `ChooseText(substring)` (more readable and survives reordering), `EnterLocation(id)`, `AdvanceTime(seconds)`.
2. Write the **prepared break-bargain** test: warn or rescue enough townspeople, make the final break choice, assert the `MorningInHollowbrook` stub and the expected survivor reactions.
3. Write the **keeper** test: accept the final offer, assert `NewKeeper` and that recorded bargain terms shape the projected epilogue.
4. Write the **first-run skip** test: trigger five valid early advances during Vale's briefing, assert `JustPassingThrough`; repeat with completed profile history and assert no ejection.
5. Write a **soft-lock guard**: assert no reachable choice node exists where all options can be simultaneously unavailable.
6. Write a **content integrity test**: every conversation file loads and validates. This one will save you repeatedly.
7. Run the suite. Under a second. **Deliberately break a JSON target and watch the exact test fail.**

### Acceptance criteria
- [ ] `PlaythroughDriver` exists and reads clearly
- [ ] All three canonical endings, including skip eligibility, are covered by tests
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
- **Write.** More dialogue and reaction variants. Your systems are ready and content is now the bottleneck — that's a good place to be.
- **Balance.** Play opposite routes back to back. Are both choices defensible, and are the delayed differences legible?
- **Story homework:** lock Eli's reason for entering the mine and the Guest's argument for preserving the bargain.
- **Rest.**

### Milestone review

Run `/review`. Ask it specifically: *is the consequence chain traceable?* If you can't answer "why did this fire?" from the debug window in under thirty seconds, that's a must-fix.

### Where you are — read this bit

You have built the thing you said you wanted.

Four choices across a town and mine. Each defensible. Each recording a concrete decision that changes Hollowbrook later without a morality popup. And a test suite that verifies all of it in under a second.

**That is a real narrative engine.** Most people who set out to make a choice-driven RPG never build one, because they get stuck on combat or art or the perfect inventory system. You built the pillar first, which is the correct order and almost nobody does it.

The rest of this project is putting a game around it.

**Commit:** `docs: M05 complete — choice and consequence`
