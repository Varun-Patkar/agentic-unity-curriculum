# M06 · Quests & Journal

**Days 43–49 · 22–28 Oct 2026 · project: `Hearthfall`**

> Quests are the structure that turns "a village with conversations in it" into "a game with a shape". By Monday, Act I is playable start to finish: you arrive in Hearthfall, learn the valley is in trouble, talk to your father, and leave for Vaskirk — with a journal that tracks it.
>
> This is also the first week you'll play your own game and feel it working.

**You end holding:** Act I, playable end to end, with a quest system and a journal.

---

## Day 43 — A quest is a state machine
**Thu 22 Oct · 60 min**

**Objective:** A quest model in Core that can express Hearthfall's real quests, including failure and mutually exclusive outcomes.

**Why:** "Quest system" sounds big and is actually small if you model it as a state machine and refuse to over-build. The over-building is what kills people here.

### Concepts (10 min)
- **A quest is a state machine with objectives.** States: `Unavailable → Available → Active → (Completed | Failed | Abandoned)`. That's enough for a 15-hour RPG.
- **Objectives are sub-states**, each with its own completion condition. A quest completes when its required objectives do.
- **Optional and hidden objectives.** Optional ones affect the reward; hidden ones reveal on discovery. Both are cheap and both add a lot.
- **Failure must be a real state.** In a game about consequence, a quest you can fail — or lock yourself out of — is a feature. Design for it from the start; retrofitting failure is painful.
- **Quests live in Core.** They're rules. The journal that displays them is view.
- **Resist the urge to build a quest editor today.** JSON, like dialogue. Tooling on a buffer day if you genuinely need it.

### Build (40 min)
1. `Core/Quests/QuestState.cs` — the enum above.
2. `Core/Quests/Objective.cs` — id, description, `Condition` for completion, `IsOptional`, `IsHidden`, current state.
3. `Core/Quests/Quest.cs` — id, title, description, objectives, `Condition` to become available, completion effects, failure condition.
4. `Core/Quests/QuestLog.cs` — the runtime collection in `GameState`. `Start`, `Complete`, `Fail`, `Abandon`, queries by state.
5. `Core/Quests/QuestSystem.cs` — on each relevant game event, re-evaluate availability, objective completion, and failure. Raise `QuestStarted`, `ObjectiveCompleted`, `QuestCompleted`, `QuestFailed`.
6. Reuse your `Condition` types from M04/M05 for objectives — that reuse is the payoff of building conditions as data.
7. Author one real quest in JSON: **"A Debt in Hearthfall"** — the valley owes the levy, your father won't ask for help, you learn you need to leave.
8. Test: quest becomes available on a flag, objectives complete in order, quest completes, effects fire.

### Acceptance criteria
- [ ] Quest state machine in Core with all six states
- [ ] Objectives support optional and hidden
- [ ] Failure is a real, reachable state
- [ ] Quest conditions reuse the M05 condition types
- [ ] One real quest authored in JSON
- [ ] Tests cover start, objective completion, completion, and failure

### Failure modes
- **Building a general-purpose quest engine** → you need seven quests, not seven hundred. YAGNI hard.
- **Quests that can't fail** → in this game specifically, that's a design mistake.
- **Objective conditions re-evaluated every frame** → evaluate on events. It's cheap and it's deterministic.

**Stretch:** Add mutually exclusive quests — accepting one makes another `Unavailable` permanently. Powerful, and it costs one field.

**Commit:** `feat(core): quest state machine`

---

## Day 44 — Objectives, triggers, and completion
**Fri 23 Oct · 60 min**

**Objective:** Quests that actually respond to play — talking to someone, reaching somewhere, having something — with no polling.

**Why:** Yesterday was the model. Today it connects to the game, and quests start progressing because of what you do.

### Concepts (10 min)
- **Event-driven, not polled.** The quest system listens; it doesn't ask. Same architecture as everything else.
- **The events that matter:** flag set, location entered, conversation completed, item acquired, enemy defeated (stubbed until M07), coin threshold.
- **Ordered vs unordered objectives.** Most are unordered — "talk to these three people, in any order". Some are sequential. Support both; sequential is just an objective whose condition includes the previous one.
- **Auto-start and auto-complete** need care. A quest that starts and completes in the same frame produces two notifications and no gameplay. Guard it.
- **Quest state must be in `GameState`** for saves and for the ending gate.

### Build (40 min)
1. Wire `QuestSystem` to the Core event bus from Day 26. Every relevant event triggers re-evaluation.
2. Add `ObjectiveCompleted` and `QuestCompleted` events with payloads the UI can render.
3. Add quest-related effect types so dialogue can drive quests: `StartQuestEffect`, `CompleteObjectiveEffect`, `FailQuestEffect`. Now a conversation can advance a quest directly from JSON.
4. Add a `QuestInStateCondition` so dialogue can gate on quest state. Both directions now work — this is the piece that makes quests and dialogue feel integrated rather than parallel.
5. Guard the same-frame start-and-complete case.
6. Author the objectives for "A Debt in Hearthfall": talk to Osric · talk to Enid · visit the levy notice · decide to leave.
7. Play it in the Console. Watch objectives complete as you talk to people.
8. Test the whole quest through the `PlaythroughDriver` from Day 41.

### Acceptance criteria
- [ ] Quest system is event-driven, never polled
- [ ] Dialogue can start quests and complete objectives from JSON
- [ ] Dialogue can gate on quest state
- [ ] Ordered and unordered objectives both work
- [ ] "A Debt in Hearthfall" progresses correctly through play
- [ ] Covered by a playthrough test

### Failure modes
- **Objective completes before it's visible** → check availability ordering; a hidden objective completing silently is confusing.
- **Two notifications for one objective** → the event fires from two paths, or you didn't guard re-evaluation.
- **Quest doesn't progress** → check the debug window (Day 40) rather than adding `Debug.Log`s. That's what you built it for.

**Stretch:** Add objective *counters* — "collect 3 of 5". One int and a lot of use.

**Commit:** `feat: event-driven quest progression`

---

## Day 45 — The journal UI
**Sat 24 Oct · 60 min**

**Objective:** Press J. See your quests, your objectives, and Enid's letters. Close it. Nothing breaks.

**Why:** The journal is the player's model of your story. In a game with no map markers and no waypoints, it's how they know what they're doing.

### Concepts (10 min)
- **Full-screen menus** — pause the world, switch the input context, and give the player a clear way out. Same pattern as dialogue.
- **Tabs**: Quests · Letters · (later) Endings-seen. Simple, and it grows.
- **A list plus a detail pane** is the standard layout. Selecting a quest shows its objectives and description.
- **Completed and failed quests must be visible.** In a game about consequence, the record of what you failed is content.
- **Scroll rects.** Fiddly. Get the Content Size Fitter and the layout group right and they behave; get them wrong and content vanishes or scrolls infinitely.

### Build (40 min)
1. A full-screen journal canvas, hidden by default. J toggles it, Esc closes it.
2. Left pane: a scrollable quest list, grouped Active / Completed / Failed, built from `QuestLog`.
3. Right pane: selected quest's title, description, and objectives — completed ones struck through, hidden ones not shown at all.
4. Second tab: letters received, re-readable in full.
5. Input context switch to `UI` on open, back to `Gameplay` on close. Pause the world.
6. Style it. Parchment, a period-appropriate font, restrained. This screen will appear in your itch.io screenshots.
7. Handle the empty state — "No active quests" beats a blank panel.
8. Test opening the journal mid-dialogue (should be blocked) and mid-combat (later — but decide the rule now).

### Acceptance criteria
- [ ] J opens the journal, Esc closes it, the world pauses
- [ ] Active, completed, and failed quests all visible and grouped
- [ ] Selecting a quest shows objectives with completion state
- [ ] Letters tab shows received letters in full
- [ ] Input context switches correctly both ways
- [ ] Empty states handled

### Failure modes
- **Scroll content invisible or unscrollable** → Content Size Fitter / layout group misconfiguration. Common; check the Fitter is on the Content object, not the Viewport.
- **Journal opens during dialogue** → gate it on the current input context.
- **World keeps running underneath** → `Time.timeScale` or an explicit pause flag. Pick one and use it consistently.
- **List doesn't refresh** → you built it once on `Awake`. Rebuild on open, or subscribe to quest events.

**Stretch:** Add a subtle "new" indicator on the journal icon when a quest updates. Small, and it removes a whole category of "did I miss something?".

**Commit:** `feat: journal ui with quests and letters`

---

## Day 46 — "Where do I go?" without a minimap
**Sun 25 Oct · 60 min**

**Objective:** The player always knows what to do next — without a compass, a quest arrow, or a minimap.

**Why:** Player confusion is the quietest way to lose someone. And "grounded, KCD-adjacent" means you can't just bolt on a floating waypoint — you have to solve it with design.

### Concepts (10 min)
- **The problem:** the player knows the objective text and doesn't know where the thing is.
- **The spectrum**, most to least intrusive: compass arrow → minimap marker → world-space marker → NPC dialogue directions → environmental signposting (light, paths, architecture).
- **Grounded games earn their tone by sitting at the bottom of that list.** KCD gives you a map and directions and expects you to think. That's a design position, and it's consistent with your fiction.
- **The compromise that works:** clear *verbal* directions in dialogue, plus a subtle world-space marker on the immediate target that fades once you've seen it. Diegetic-ish, and nobody gets lost.
- **The real fix is often level design.** If the path to the notice board reads visually, no marker is needed.

### Build (40 min)
1. Decide your position and write it in `CONVENTIONS.md`. Recommended: **verbal directions + subtle world-space marker on the current objective only**, no compass, no minimap.
2. `Unity/UI/ObjectiveMarker.cs` — a world-space indicator over the current objective's target, visible at a distance, fading as you approach.
3. Register objective targets: an `ObjectiveTarget` component with an objective ID, found at runtime.
4. Off-screen indication: clamp the marker to the screen edge when the target is off-view.
5. A toggle in settings (Markers: On / Objective-only / Off) — costs nothing, respects players who want the harder version, and it's a genuine accessibility feature.
6. **Rewrite one conversation to include real directions.** "Past the mill, on the notice board by the well." Better than any marker.
7. Playtest Act I as if you'd never seen it. Note every moment of hesitation. Those are your bugs.

### Acceptance criteria
- [ ] A written position on navigation in `CONVENTIONS.md`
- [ ] Objective marker shows the current target and fades on approach
- [ ] Off-screen targets are indicated at the screen edge
- [ ] A settings toggle exists
- [ ] At least one conversation gives real spoken directions
- [ ] You played Act I and noted every hesitation

### Failure modes
- **Markers on everything** → visual noise, and it undermines the tone. Current objective only.
- **Marker on a target across the map** → distance-cap it, or it's useless.
- **Marker persists after completion** → subscribe to `ObjectiveCompleted`.

**Stretch:** A hand-drawn map item in the journal. Static image, marked locations, no player dot. Extremely on-theme and mostly an art task.

**Commit:** `feat: objective markers and navigation`

---

## Day 47 — Wiring Act I
**Mon 26 Oct · 60 min**

**Objective:** Act I as real, connected content — arrive, meet the family, understand the trouble, choose to leave.

**Why:** Six milestones of systems, and today they become a story. This is the first day Hearthfall is a *game* rather than a set of features.

### Concepts (10 min)
- **Act I's job:** make the family *people*, not backstory. The ending only lands if the player liked them. That's the whole design requirement.
- **Establish the stakes without a cutscene.** The levy notice, Osric's refusal to ask for help, Enid actually running the farm. Show through interaction.
- **The leaving must be a choice**, and the player should feel slightly reluctant. If they're eager to leave, Act III has nothing to work with.
- **Keep it short.** 15–20 minutes of play. Long enough to care, short enough to replay when testing.
- **This is content work, not systems work.** Notice how much faster it goes now that the systems exist.

### Build (40 min)
1. Lay out the village: the farm, the well, the notice board, the mill, the road out. Use your tilemap from Day 18.
2. Place Osric, Enid, and Cob with their conversations.
3. Author the real conversations. **Osric's specific pride** — the thing he will not do — is your homework from M03. Use it here; it's Ending B's load-bearing beam.
4. The levy notice as an interactable that starts the quest.
5. The road out as a location transition, gated on the quest state, with a confirmation ("Once you go, you go").
6. Wire the objective markers.
7. **Play the whole act.** Time it. Note everything that's flat.
8. One environmental detail that pays off in Act III: something in the house you'll see again, changed.

### Acceptance criteria
- [ ] Act I plays start to finish without intervention
- [ ] Three family members have real conversations
- [ ] The quest tracks it and the journal shows it
- [ ] Leaving is a deliberate, confirmed choice
- [ ] It takes 15–20 minutes
- [ ] One detail is planted for Act III

### Failure modes
- **The family is exposition** → if they only explain the plot, rewrite them. They need at least one thing that's just character.
- **Act I is 45 minutes** → cut. You'll replay this dozens of times.
- **The player has no reason to leave** → the stakes aren't material enough. Make the levy concrete and the number specific.

**Stretch:** A small optional interaction — help Cob with something, or don't. Sets a flag. Pays off in exactly one line in the epilogue. This is the kind of thing players remember.

**Commit:** `content: act i wired end to end`

---

## Day 48 — Playtest Act I properly
**Tue 27 Oct · 60 min**

**Objective:** A written bug list from a real playtest, and the top three fixed.

**Why:** Playtesting is a skill and most developers do it badly — they play to verify, not to observe. Doing it deliberately now, on 20 minutes of content, teaches you the habit before you have three hours of it.

### Concepts (10 min)
- **Play as a player, not as the developer.** Don't take the route you know works. Take the stupid route on purpose.
- **Write everything down, immediately**, without fixing anything. Fixing mid-playtest destroys the playtest.
- **Three categories:** *broken* (errors, softlocks, unreachable content) · *confusing* (I didn't know what to do) · *flat* (it worked and I felt nothing).
- **"Flat" is the most valuable category and the one you'll want to skip.** It's where the game is technically fine and emotionally dead.
- **Triage ruthlessly.** Broken first, confusing second, flat third — except when a flat moment is a story beat that has to land, in which case it's first.

### Build (40 min)
1. **Play Act I three times**, ~7 minutes each: once optimally, once taking every wrong turn, once trying to break it (talk to everyone twice, open the journal mid-dialogue, walk into everything).
2. Write everything in `PLAYTEST.md` in the project, categorised. Do not fix anything yet.
3. Check the Console after each run for warnings you've been ignoring.
4. Triage. Pick the top three.
5. Fix them.
6. Re-run the automated playthrough tests to confirm nothing regressed.
7. Anything not fixed goes into `progress/STATE.md` under `parked`, or into `PLAYTEST.md` as a known issue. Nothing gets forgotten silently.

### Acceptance criteria
- [ ] Three playtests completed with different approaches
- [ ] `PLAYTEST.md` exists, categorised
- [ ] Top three issues fixed
- [ ] Console is clean of warnings you understand and haven't addressed
- [ ] Automated tests still pass
- [ ] Unfixed issues recorded, not forgotten

### Failure modes
- **Fixing while testing** → you'll stop testing. Separate the passes.
- **No "flat" entries** → you weren't honest, or you weren't paying attention. There are always some.
- **Fixing the easy ones instead of the important ones** → triage exists for exactly this temptation.

**Stretch:** Get someone else to play it. Watch silently. Say nothing, even when they miss the obvious thing. *Especially* then — that's the most valuable data you'll get all month.

**Commit:** `fix: act i playtest issues`

---

## Day 49 — BUFFER
**Wed 28 Oct**

- **Catch up.**
- **Fix more of the playtest list.**
- **Write more content** — Act II conversations, minor choices, Bran's dialogue (due by M07, which starts tomorrow).
- **Polish Act I.** It's the first thing anyone plays. It deserves an extra hour.
- **Rest.**

### Milestone review

Run `/review`. Ask specifically whether quest logic has leaked into the journal UI — the classic mistake is the UI deciding whether an objective is complete.

### Where you are

**Act I is playable.** You can start the game, meet your family, understand why you have to leave, and leave — with quests tracking it and a journal recording it.

You are 49 days in, roughly 44% through, and you have a working narrative game. The pillar is built. The story has begun.

Tomorrow: combat. It's the part you said you cared least about, which means the plan keeps it small and makes it feel good — and "small but feels good" is a much better target than "deep but janky".

**Commit:** `docs: M06 complete — quests and act i`
