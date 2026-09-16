# M04 · Dialogue

**Days 29–35 · 8–14 Oct 2026 · project: `Hollowbrook`**

> Story-heavy was your pillar, and this is the week it starts existing. By Sunday you walk into Town Hall, press E, and Mayor Vale briefs Alex on Eli's disappearance — with a portrait, a name, and text that types itself out at a pace you chose.
>
> The runner lives in Core. The pretty part lives in Unity. On Day 75 you'll run this exact conversation in a 3D scene without touching the runner.

**You end holding:** a working dialogue system with portraits and typewriter text, driven by data you can edit without recompiling.

---

## Day 29 — Designing the dialogue graph
**Thu 8 Oct · 60 min**

**Objective:** A node/edge model in Core that can express everything Hollowbrook needs — including the Mayor's warning branches — and nothing it doesn't.

**Why:** Designing this yourself, rather than importing Ink or Yarn, means you understand every branch condition when you're debugging a broken path in November. It also means the model fits *your* game exactly.

### Concepts (10 min)
- **A conversation is a directed graph.** Nodes are lines or choices; edges are transitions, sometimes conditional.
- **Node types you actually need:** `Line` (someone says something, then continues), `Choice` (player picks, each option leads somewhere), `Effect` (set a flag, record a decision or consequence — no text), `End`.
- **Keep the node set small.** Every type you add is a case in the runner, the loader, the validator, and the UI. Four types is enough for a 15-hour RPG; you can add a fifth when you genuinely need it.
- **Conditions are data, not code.** `flag_set:eli_clue_found`, `irritation_at_least:annoyed`. A tiny expression language beats hardcoded `if`s because content authors (you, in November, tired) shouldn't recompile to change a gate.
- **Why not Ink/Yarn?** They're excellent and you should look at them after this project. Today, building it teaches you the shape of the problem, and the shape is what you need to debug branching narrative.

### Build (40 min)
1. `Core/Dialogue/DialogueNode.cs` — abstract base with `NodeId Id`. Subclasses: `LineNode` (speaker, text, next), `ChoiceNode` (prompt, `List<ChoiceOption>`), `EffectNode` (effects, next), `EndNode`.
2. `Core/Dialogue/ChoiceOption.cs` — text, `NodeId Target`, optional `Condition`, optional `List<Effect>`.
3. `Core/Dialogue/Conversation.cs` — an id, an entry node id, and a `Dictionary<NodeId, DialogueNode>`.
4. `Core/Dialogue/Condition.cs` — for today, an abstract type with `FlagSetCondition`, `FlagNotSetCondition`, `IrritationStageCondition`, and an `AllOf`/`AnyOf` composite. Each has `bool Evaluate(GameState state)`.
5. `Core/Dialogue/Effect.cs` — same shape: `SetFlagEffect`, `RecordDecisionEffect`, `ScheduleConsequenceEffect`. Each has `void Apply(GameState state)`.
6. **Write one conversation entirely in C#** as a test fixture — the start of Mayor Vale's opening briefing, with explicit warning nodes reachable by irritation stage. Ugly to author, but it proves the model.
7. Test: the graph resolves, every `next` points at a node that exists, conditions evaluate correctly.
8. **Sketch the graph on paper first**, then check your code matches. If the paper version needed a node type you didn't build, decide honestly whether you need it.

### Acceptance criteria
- [ ] Four node types in Core, no Unity references
- [ ] Conditions and effects are data types with `Evaluate`/`Apply`
- [ ] A conversation can be constructed in code and validated
- [ ] A test proves conditional branching works
- [ ] You can draw the model on paper in under a minute

### Failure modes
- **Over-designing** — no variables system, no functions, no scripting language. Four node types.
- **Putting text formatting in the model** — Core stores the string. Rich text, colour, and speed are the view's problem.
- **Conditions as `Func<GameState,bool>`** — tempting and clean, but not serializable, so your dialogue can't live in a file. Data types.

**Stretch:** Add a `Condition` that checks the decision ledger for a specific past choice. That lets later conversations cite what Alex actually did without introducing a score.

**Commit:** `feat(core): dialogue graph model`

---

## Day 30 — Dialogue as data: format, loader, validator
**Fri 9 Oct · 60 min**

**Objective:** Conversations live in files you can edit in VS Code, and a broken one tells you exactly what's wrong.

**Why:** By November you'll have hundreds of nodes. Authoring them in C# would be miserable and recompiling to fix a typo would be worse.

### Concepts (10 min)
- **JSON is the pragmatic choice.** Diffable in git, editable anywhere, trivially parseable, and portable to the 3D project with zero changes.
- **Polymorphic deserialization** is the one wrinkle — a `List<DialogueNode>` holding subclasses needs a type discriminator. `System.Text.Json` supports this with a custom converter, or use a `"type": "line"` field and a manual factory. Manual is boring and completely reliable.
- **Validate at load, loudly.** Dangling node references, unreachable nodes, missing speakers, duplicate IDs. A good validator turns a 40-minute November debugging session into a 5-second error message.
- **Where do files live?** `Assets/_Project/Content/Dialogue/*.json`, loaded via `Resources`, `StreamingAssets`, or (best) a Unity-side loader that reads `TextAsset`s and hands strings to Core. Core takes a string, never a path — that keeps it portable.

### Build (40 min)
1. Design the JSON shape. Keep it readable — you're going to hand-write a lot of it:
   ```json
   { "id": "vale_briefing", "entry": "start", "nodes": [
     { "id": "start", "type": "line", "speaker": "mayor_vale",
       "text": "Alex. I wish you had come home under better circumstances.", "next": "reply" },
     { "id": "reply", "type": "choice", "options": [
       { "text": "Tell me what happened to Eli.", "target": "eli" },
       { "text": "Start with the mine.", "target": "mine" }
     ]}
   ]}
   ```
2. `Core/Dialogue/DialogueLoader.cs` — `Conversation Load(string json)`. Core takes the string. It never knows about files.
3. Handle the polymorphic node types via the `type` discriminator.
4. `Core/Dialogue/ConversationValidator.cs` — returns a list of problems: dangling targets, unreachable nodes, duplicate IDs, unknown speaker IDs, choice nodes with zero options.
5. `Unity/Content/DialogueRepository.cs` — loads every `TextAsset` in the dialogue folder, passes the text to Core, caches the results, logs validation errors on load.
6. Port yesterday's Mayor Vale briefing to JSON. Include graph nodes for counts 2–5: annoyed interruption, deliberate restart, unmistakable final warning, and ejection into `JustPassingThrough`. Delete the C# version.
7. **Deliberately break the JSON** — point a `next` at a node that doesn't exist. Confirm the validator names it precisely. That error message is a gift to future you.
8. Write tests: valid JSON loads, invalid JSON reports the right problem.

### Acceptance criteria
- [ ] Conversations load from JSON into Core types
- [ ] Core's loader takes a string, never a file path
- [ ] Validator catches dangling refs, duplicates, unreachable nodes, and unknown speakers
- [ ] Mayor Vale's briefing and all warning nodes are authored in JSON
- [ ] Validation errors appear in the Unity Console at load
- [ ] Tests cover both the happy path and each validation failure

### Failure modes
- **Polymorphic deserialization "just doesn't work"** → it needs explicit help. Manual factory on the `type` field is fine and takes ten minutes.
- **File not found in a build** → `Resources`/`StreamingAssets` paths behave differently in builds. That's why Unity loads and Core parses.
- **Silent load failure** → never swallow a parse exception. Fail loud, fail at startup.

**Stretch:** Write a JSON Schema for your format and wire it up in VS Code. You get autocomplete and inline validation while authoring dialogue. This is a genuinely large quality-of-life win for a small time cost.

**Commit:** `feat: json dialogue authoring with validation`

---

## Day 31 — The dialogue runner and Mayor patience
**Sat 10 Oct · 60 min**

**Objective:** A runner in Core that walks a conversation start to finish — driven entirely by a test, with no Unity involved.

**Why:** The runner is the brain. Building it headless first means it's correct before it's pretty, and it means every branch is testable in milliseconds.

### Concepts (10 min)
- **The runner is a state machine over the graph.** Current node, plus operations: `Start(conversationId)`, `Advance()`, `Choose(index)`, `ReportEarlyAdvance()`, `IsFinished`.
- **It pushes, the view pulls — or it raises events.** Prefer events: `LineShown(speaker, text)`, `ChoicesOffered(list)`, `ConversationEnded`. Consistent with Day 26.
- **`EffectNode`s are traversed, not displayed.** The runner should apply their effects and continue automatically until it reaches something the player must see.
- **Conditions filter choices at presentation time**, not authoring time. A choice whose condition is false is either hidden or shown greyed — decide which, per option. (Greyed-out-but-visible is a powerful narrative tool: *"[Requires Pike's files]"* tells the player what evidence they lack.)
- **Guard against infinite loops.** A cycle of `EffectNode`s with no visible node will hang your game. Cap traversal depth and throw.

### Build (40 min)
1. `Core/Dialogue/DialogueRunner.cs`. Constructor takes `GameState` and a conversation source.
2. `Start(ConversationId)` → resolve entry, traverse effects, raise the first `LineShown` or `ChoicesOffered`.
3. `Advance()` → move to `next`, traverse effects, raise the next event or `ConversationEnded`.
4. `Choose(int index)` → apply the option's effects, jump to its target, continue.
5. Filter choices through their conditions. Return both available and locked options so the view can decide how to render them.
6. **Loop guard:** if traversal passes 100 nodes without producing a visible one, throw with the node IDs involved.
7. `Core/Dialogue/MayorPatience.cs` owns `EarlyAdvanceCount` and emits an irritation stage: neutral at 0–1, annoyed at 2, deliberate restart at 3, final warning at 4, ejection at 5. It only activates for the opening briefing and only when `PlaythroughHistory.IsEarlyEndingEligible` is true.
8. **Test it hard.** Drive every branch, report early advances, verify each stage and warning node, and prove count 5 triggers `JustPassingThrough` only before any completed ending.

### Acceptance criteria
- [ ] Runner walks a conversation entirely in Core
- [ ] Effect nodes apply and traverse without display
- [ ] Choices are filtered by condition, with locked ones distinguishable
- [ ] Loop guard throws with useful information
- [ ] A test drives a full multi-branch conversation
- [ ] Core owns early-advance count and irritation stage
- [ ] Warning responses are graph nodes, not UI strings

### Failure modes
- **Runner reaching into the view** → it raises events and nothing else.
- **Effects applying twice** → advancing re-applies the current node. Apply on *entry* to a node, exactly once.
- **A choice's effects not applying** → option effects and node effects are different things; make sure both are handled.

**Stretch:** Add conversation *history* — every node visited, in order. That's your debug trace, and on Day 62 it's what lets an epilogue reference something from Act I.

**Commit:** `feat(core): dialogue runner with conditional choices`

---

## Day 32 — The dialogue UI
**Sun 11 Oct · 60 min**

**Objective:** A dialogue panel with speaker name, portrait, body text, and a choice list, driven by runner events.

**Why:** The moment the game stops being a tech demo. Also: this UI is the thing your player stares at for most of Hollowbrook, so it's worth caring about.

### Concepts (10 min)
- **The view subscribes; it never queries.** `LineShown` → render. `ChoicesOffered` → build buttons. The view holds no dialogue state of its own.
- **Layout Groups** (Vertical/Horizontal Layout Group + Content Size Fitter) auto-arrange the choice list. They're fiddly, they fight you, and they're still better than manual positioning.
- **Pooling choice buttons.** Instantiating and destroying buttons per node works and is fine at this scale; know that pooling exists.
- **Input context switching** — this is where `InputRouter` from Day 16 earns its keep. Entering dialogue calls `EnableDialogue()`; leaving calls `EnableGameplay()`.
- **Design for the worst case:** the longest line, the most choices, the longest choice text. Anchor and size for that, not for your test line.

### Build (40 min)
1. Build the panel: a bottom-anchored container, portrait image (left), speaker name, body text, and a vertical container for choices. Anchor properly — you learned this on Day 11.
2. `Unity/UI/DialogueView.cs` — subscribes to the runner's events, populates the fields, shows and hides the panel.
3. Portraits: resolve `CharacterId` → `Sprite` via the `ContentDatabase` from Day 27. **Core supplied only an ID.** Notice how clean that boundary feels.
4. Choice buttons from a prefab, instantiated into the layout group, wired to `runner.Choose(i)`.
5. Locked choices: render greyed and non-interactable, with the requirement in brackets. This is a narrative device, not just UI state.
6. Wire the input context switch on enter/exit.
7. Play it. Start Mayor Vale's briefing. This is a real moment — sit with it for a second.
8. Test the worst case: a 200-character line and five choices. Fix what breaks.

### Acceptance criteria
- [ ] Panel shows speaker, portrait, and text from runner events
- [ ] Choices render as buttons and selecting one advances correctly
- [ ] Locked choices are visible but disabled, with the requirement shown
- [ ] Player movement is disabled during dialogue and restored after
- [ ] Layout survives long text and five choices
- [ ] The view holds no dialogue state

### Failure modes
- **Choice buttons stack on top of each other** → no Layout Group, or Content Size Fitter misconfigured.
- **Buttons unclickable** → no EventSystem · panel behind something · Raycast Target off.
- **Portrait missing** → the ID isn't in the content database. Your Day 27 validator should have caught this.
- **Player walks around during dialogue** → input context not switched.
- **Text overflows the box** → auto-size, or a scroll rect, or a shorter line. Usually a shorter line.

**Stretch:** Add a subtle panel slide-in and fade. Ten lines with a coroutine and it dramatically raises the perceived quality of the whole game.

**Commit:** `feat: dialogue ui with portraits and choices`

---

## Day 33 — Typewriter text, early advances, and pacing
**Mon 12 Oct · 60 min**

**Objective:** Text that types out at a pace you tuned, reveals fully on first input, advances on the second, and reports only genuine early advances to Core.

**Why:** Pacing is a gameplay system in a story-heavy game. This is where a conversation stops reading like a webpage and starts reading like a scene.

### Concepts (10 min)
- **Typewriter = reveal characters over time.** With TextMeshPro, set the full string once and animate `maxVisibleCharacters` — far better than string concatenation, which allocates on every single character.
- **Two-stage input is the standard, and it matters:** first press completes the line instantly, second press advances. During Vale's first-playthrough briefing only, that first press reports one early advance to Core.
- **Punctuation pauses.** A longer beat on `.` `,` `—` `…` is a handful of lines of code and it is most of the difference between "text appearing" and "someone speaking".
- **Speed must be a setting.** Slow / Normal / Fast / Instant and accessibility reveal-complete are presentation choices. Fast, Instant, or accessibility-driven completion never reports an early advance.
- **`WaitForSecondsRealtime`** in your coroutine, so typing still works if you ever pause with `timeScale`.

### Build (40 min)
1. `Unity/UI/TypewriterText.cs` — takes a string, sets it on the TMP component, sets `maxVisibleCharacters = 0`, and increments it on a coroutine.
2. Serialized `_charactersPerSecond`. Tune it in Play Mode with real dialogue text. It'll be slower than you expect.
3. Punctuation pauses: a small dictionary of char → extra delay. `.` and `?` and `!` long, `,` medium, `—` and `…` long.
4. Two-stage input: `Advance` while actively revealing → jump to full and notify the runner that a reveal-state early advance occurred. `Advance` when complete → next node without incrementing anything.
5. A "continue" indicator — a small blinking arrow that only appears when the line is fully revealed. Tiny detail, big readability win.
6. Add text speed to a settings object (Slow / Normal / Fast / Instant) plus an accessibility reveal-complete path. Neither may call `ReportEarlyAdvance`.
7. Route irritation stage back to the graph so counts 2–4 insert Vale's escalating warning nodes and count 5 starts `JustPassingThrough`. The final warning must display fully before another early advance can eject Alex.
8. Add EditMode tests proving: Instant and accessibility reveal never count; fully visible advances and choices never count; counts map to the right warning nodes; after any completed ending, repeated reveal presses never punish the player.

### Acceptance criteria
- [ ] Text types character by character with no per-frame allocation
- [ ] Punctuation produces audible/visible pauses
- [ ] First input completes the line, second advances
- [ ] Speed setting exists and works, including Instant
- [ ] Continue indicator appears only when the line is complete
- [ ] Only reveal-state player advances increment `EarlyAdvanceCount`
- [ ] Automated tests cover every fairness rule and the unmistakable final warning

### Failure modes
- **Text flickers or reflows as it types** → you're rebuilding the string. Use `maxVisibleCharacters`.
- **Rich text tags visible** → `maxVisibleCharacters` handles tags correctly; string slicing does not. Another reason to use it.
- **One press advances two stages** → your input fires in both paths in one frame. Consume the input, or gate on reveal state.
- **Instant mode irritates Vale** → presentation completion is incorrectly calling the player-advance path. Keep those APIs separate.
- **Typing freezes when paused** → use `WaitForSecondsRealtime`.

**Stretch:** Support inline pause and speed tags in dialogue text. A small parser hands you directorial control without changing the fairness rules.

**Commit:** `feat: typewriter text with pacing and skip`

---

## Day 34 — Interactables: walking up and talking
**Tue 13 Oct · 60 min**

**Objective:** Walk to Mayor Vale in Town Hall, see a prompt, press E, and have the opening briefing start.

**Why:** This closes the loop between the world and the story. It's also the interaction system you'll use for doors, chests, signs, and every NPC in the game — and you'll rebuild exactly this in 3D on Day 76.

### Concepts (10 min)
- **The general problem:** what is the player able to interact with *right now*, and which one if there are several?
- **2D approach:** a trigger collider on the player detects `IInteractable` components in range; pick the nearest, or the one most aligned with facing.
- **Facing matters.** Standing between two NPCs should talk to the one you're looking at. You tracked facing on Day 17 for exactly this.
- **The prompt is a contract.** If a prompt shows, the interaction must work. A prompt that appears and does nothing is worse than no prompt.
- **`IInteractable` in Unity, not Core.** Interaction is a *view* concern — Core just needs to be told "start conversation X".

### Build (40 min)
1. `Unity/Interaction/IInteractable.cs` — `string PromptText { get; }`, `void Interact()`, `Transform Transform { get; }`.
2. `Unity/Interaction/PlayerInteractor.cs` — a trigger collider maintaining a set of in-range interactables, selecting the best candidate each frame (nearest, weighted by facing alignment), and calling `Interact()` on the `Interact` action.
3. `Unity/Interaction/DialogueInteractable.cs` — holds a conversation ID, calls into the dialogue runner.
4. World-space prompt UI: a small "E — Talk" above the selected interactable. A world-space canvas or a sprite; either is fine.
5. Put Mayor Vale in Town Hall with a portrait, a collider, and his briefing. **Ask about Eli.**
6. Add a second interactable close by (a sign, a well) and confirm that facing selects the right one.
7. Handle edge cases: interactable destroyed while in range · player enters dialogue while in range of two things · walking away mid-prompt.
8. Disable the interactor while dialogue is active.

### Acceptance criteria
- [ ] Prompt appears when in range of an interactable
- [ ] Facing correctly selects between two nearby interactables
- [ ] E starts Mayor Vale's briefing
- [ ] Prompt hides during dialogue and while out of range
- [ ] No errors when an interactable is destroyed while in range
- [ ] Interaction is entirely in the Unity layer

### Failure modes
- **Trigger doesn't detect** → no Rigidbody2D on the player · `Is Trigger` unchecked · layers not set to collide (Day 19's matrix).
- **Prompt sticks after walking away** → `OnTriggerExit2D` not handled, or the object was destroyed without leaving the set.
- **Wrong NPC selected** → facing weight too low relative to distance. Tune it.
- **Interaction fires repeatedly** → the button is being read as held rather than pressed.

**Stretch:** Add an outline or brightness tint on the currently-selected interactable. Clearer than a prompt alone, and it scales to a crowded scene where three things are in range.

**Commit:** `feat: interaction system with dialogue triggers`

---

## Day 35 — BUFFER
**Wed 14 Oct**

- **Catch up** on anything unfinished — this milestone has a lot of moving parts.
- **Write dialogue.** Finish Mayor Vale's briefing and five interruption responses from `reference/story-bible.md` §5. Your system can handle them now.
- **Polish the UI.** Fonts, spacing, the portrait frame. A dialogue box you like looking at is worth an hour.
- **Story homework:** Eli's reason for entering the mine and the Guest's best argument are due in M05. If either is vague, decide it now.
- **Rest.**

### Milestone review

Run `/review`. Ask specifically whether any dialogue *logic* has leaked into `DialogueView` — the classic mistake is the view deciding which node comes next.

### Where you are

You have a working, data-driven, tested dialogue system with portraits, pacing, and conditional choices — built by you, understood by you, and portable to 3D.

**Tomorrow is the pillar.** M05 is the reason this game exists: choices that matter, consequences that arrive late, and a gut-punch you'll have authored yourself. Everything so far has been building the machine that makes it possible.

**Commit:** `docs: M04 complete — dialogue system`
