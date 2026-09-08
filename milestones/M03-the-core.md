# M03 · The Core — Engine-Agnostic Game Logic

**Days 22–28 · 1–7 Oct 2026 · project: `Hearthfall`**

> **The screen will not change this week.** On Day 28 the game looks exactly as it did on Day 21. That is expected, it is fine, and it is the price of the entire back half of this curriculum.
>
> What changes is that by Friday your game's rules live in a place Unity cannot reach into, cannot break, and cannot slow down. On **Day 71** you'll create a fresh 3D project, import this assembly without editing a single line, and watch it work. That moment is what this week buys.

**You end holding:** a `Hearthfall.Core` assembly with zero `using UnityEngine`, under test, that a console app could run.

**If you only remember one sentence from this curriculum:** *game rules are not Unity's business.*

---

## Day 22 — Assembly definitions: building the wall
**Thu 1 Oct · 60 min**

**Objective:** Three assemblies — `Hearthfall.Core`, `Hearthfall.Unity`, `Hearthfall.Editor` — where the compiler physically prevents Core from referencing UnityEngine.

**Why:** Discipline you have to remember is discipline you will lose in November at 11pm. Make the compiler hold the line for you.

### Concepts (10 min)
- **Assembly Definition (`.asmdef`)** — a file that turns a folder into its own C# assembly. Without them, all your code compiles into one giant `Assembly-CSharp.dll`.
- **Two payoffs:** enforced dependency direction (an assembly can only see what it explicitly references), and much faster compile times (only changed assemblies rebuild).
- **`noEngineReferences: true`** — the flag that makes `using UnityEngine` a **compile error** in Core. This is the whole trick.
- **Dependency direction:** `Unity → Core`. Never the reverse. Core does not know Unity exists.
- **Test assemblies** need references to `nunit.framework` and the Unity test framework, and must be marked as test assemblies.

### Build (40 min)
1. In `Assets/_Project/Code/Core/`, create an Assembly Definition named `Hearthfall.Core`. In its Inspector, **untick "Auto Referenced"** is not what you want — instead find and enable **"No Engine References"**. *(The exact label has moved between versions; if you can't find it, edit the `.asmdef` JSON directly and add `"noEngineReferences": true`.)*
2. In `Assets/_Project/Code/Unity/`, create `Hearthfall.Unity`, and add `Hearthfall.Core` to its Assembly Definition References.
3. Move your existing scripts (`PlayerMovement`, `InputRouter`, etc.) into `Code/Unity/`. Everything should still compile.
4. **Prove the wall works.** Create a throwaway file in Core with `using UnityEngine;` at the top. It must fail to compile. Read the error. Delete the file. *This is the most important thirty seconds of the week.*
5. Create `Assets/_Project/Code/Editor/` with `Hearthfall.Editor` (Include Platforms → **Editor only**), referencing both others.
6. Create `Assets/_Project/Code/Core.Tests/` with a test assembly definition referencing `Hearthfall.Core`, `nunit.framework.dll`, and `UnityEngine.TestRunner`. *(Easiest route: `Window > General > Test Runner` → EditMode → "Create EditMode Test Assembly Folder", then add the Core reference.)*
7. Write one trivial passing test (`Assert.AreEqual(2, 1+1)`) and run it. Green light in the Test Runner.
8. Note your compile times before and after. They should be noticeably better.

### Acceptance criteria
- [ ] Four assemblies exist: Core, Unity, Editor, Core.Tests
- [ ] `using UnityEngine` in Core is a **compile error** — you verified this personally
- [ ] Existing gameplay still works
- [ ] One test runs green in the Test Runner
- [ ] Dependency direction is Unity → Core, never the reverse

### Failure modes
- **Everything breaks with "type not found"** → an asmdef is missing a reference. Add it in the Inspector.
- **Tests won't compile** → the test asmdef is missing `nunit.framework.dll` or isn't flagged as a test assembly.
- **Can't find "No Engine References"** → edit the `.asmdef` JSON in a text editor. It's a small, readable file.
- **Third-party packages break** → they may need explicit references. Add them.

**Stretch:** Create a tiny .NET console project referencing the Core source files directly and print something. Overkill today, but seeing your game logic run with no Unity in the process is genuinely clarifying.

**Commit:** `chore: assembly definitions with engine-free core`

---

## Day 23 — Modelling world state as plain C#
**Fri 2 Oct · 60 min**

**Objective:** A `GameState` in Core that models everything true about a playthrough — and nothing about how it's displayed.

**Why:** This is the object your saves serialize, your endings read, your tests drive, and your 3D game consumes untouched. Get its shape right and the next 90 days are downhill.

### Concepts (10 min)
- **Separate what is true from how it is shown.** "Osric's health is 40" is state. "The health bar is red" is view. The line between them is the entire architecture.
- **This is not new to you** — it's a domain model, and you've written a hundred. The only unusual constraint is that it must be serializable and testable with no framework.
- **Model the nouns of your design doc**, not the nouns of your scene. `Ledger`, `Flag`, `QuestState`, `Coin` — not `PlayerGameObject`.
- **Avoid premature interfaces.** Concrete classes now; extract abstractions when a second implementation actually exists.
- **Immutability where cheap.** Records and read-only collections make bugs impossible rather than unlikely.

### Build (40 min)
1. `Core/State/GameState.cs`: the root aggregate. Fields for `Flags`, `ConscienceLedger`, `Coin`, `Quests`, `CurrentLocationId`, `PlayerName`, `PlaythroughSeconds`.
2. `Core/State/Coin.cs`: current amount and a `LifetimeEarned` that never decreases. *(Per `story-bible.md`, the ending cares what you **took**, not what you still have — this field is load-bearing for Ending B.)*
3. `Core/World/LocationId.cs` and `Core/World/CharacterId.cs` — strongly-typed IDs, not raw strings. A `readonly record struct` wrapping a string works well and eliminates a whole class of typo bug.
4. `Core/State/GameStateFactory.cs`: `NewGame()` returning a valid starting state. Testable, deterministic.
5. **No behaviour on the state objects yet.** Just shape. Rules come tomorrow.
6. Write tests: a new game has zero coin, no flags, and a valid location.
7. **Draw the model on paper.** Genuinely — five minutes with a pen. If you can't draw it clearly, it isn't clear.

### Acceptance criteria
- [ ] `GameState` exists in Core with no UnityEngine anywhere
- [ ] Strongly-typed IDs instead of raw strings
- [ ] `NewGame()` produces a valid state
- [ ] Tests pass
- [ ] You can explain every field's purpose without hesitating

### Failure modes
- **Sneaking `Vector2` into Core** → `UnityEngine.Vector2` is banned. Write your own two-float struct if you need one, or reconsider whether position belongs in Core at all. (It mostly doesn't — Core cares about *which location*, not *which pixel*.)
- **Over-modelling** — you don't need an `IEntity` hierarchy. YAGNI, hard.
- **Under-modelling** — if you write `Dictionary<string, object>`, stop.

**Stretch:** Make `GameState` a `record` and see how the value semantics feel for save/load and testing.

**Commit:** `feat(core): game state model`

---

## Day 24 — Flags and the Conscience ledger
**Sat 3 Oct · 60 min**

**Objective:** The system that makes branching narrative possible — and the ledger that lets the epilogue quote your own decisions back at you.

**Why:** This is the mechanical heart of Hearthfall. Everything in M05 sits on top of what you build today.

### Concepts (10 min)
- **A flag is a fact about the playthrough.** `met_osric`, `took_the_assize`, `tam_named`. Set once, read everywhere, never guessed at.
- **Typed flags beat magic strings.** A `FlagId` record struct plus a static registry of known flags gives you compile-time safety and a definitive list of what exists.
- **Why not just booleans on classes?** Because you need to enumerate them (debug window, Day 40), serialize them (Day 57), and query them from data-driven conditions (Day 36).
- **The Conscience ledger is a `List<ConsciencePoint>`, not an `int`.** Each entry: the choice ID, a weight, a short human-readable description, and when it happened. This matters enormously — an int can tell you *how bad* you were; a ledger can tell the epilogue *that you named Tam Ferrier in the third week of Lent.* **That specificity is the entire reason Ending B lands.** See `reference/story-bible.md` §4.
- **Never show the ledger to the player.** No meter, no bar, no notification. The theme dies the moment it becomes a number to optimise.

### Build (40 min)
1. `Core/State/FlagId.cs` — a `readonly record struct` wrapping a string.
2. `Core/State/Flags.cs` — the store. `Set`, `Clear`, `IsSet`, `All`. Back it with a `HashSet<FlagId>`.
3. `Core/State/KnownFlags.cs` — static readonly declarations for every flag you know you need so far. Add to it as you go; it becomes your narrative index.
4. `Core/State/ConsciencePoint.cs` — `ChoiceId`, `Weight` (int, signed), `Description` (string), `AtSeconds`.
5. `Core/State/ConscienceLedger.cs` — `Record(point)`, `Total`, `EntriesWithWeightBelow(n)`, `Contains(choiceId)`. `Total` exists for the ending gate; the entries exist for the epilogue text.
6. Wire both into `GameState`.
7. **Tests, and take these seriously:** setting a flag twice is idempotent · clearing works · ledger total sums correctly · entries preserve order · a specific choice can be found by ID.
8. Add three real flags from the story bible: `took_the_assize`, `named_tam_ferrier`, `sold_the_harvest`.

### Acceptance criteria
- [ ] Flags are typed, not raw strings
- [ ] `KnownFlags` lists every flag currently in use
- [ ] Conscience ledger stores structured entries, not a bare int
- [ ] The three Ledger flags from the story bible exist
- [ ] Tests cover set/clear/idempotency and ledger ordering
- [ ] Nothing anywhere displays conscience to the player

### Failure modes
- **Reaching for magic strings** — `Flags.IsSet("met_osrik")` compiles and is silently always false. That's exactly what typed IDs prevent.
- **Making the ledger an int "for now"** → you will not go back, and Ending B will be a number comparison instead of a moment.
- **Putting flag *logic* in the flag store** — the store stores. Rules live in the systems that consume it.

**Stretch:** Add `FlagSetAt` timestamps. "You've been in Vaskirk for 40 days and haven't written home" is a genuinely great line the system can now generate.

**Commit:** `feat(core): flag system and conscience ledger`

---

## Day 25 — EditMode tests: proving it without pressing Play
**Sun 4 Oct · 60 min**

**Objective:** A test suite that runs in under a second and proves your game rules work, without ever entering Play Mode.

**Why:** You already know this is valuable — you write backend tests. What's new is how *unusually* valuable it is here, because the alternative is manually replaying a 20-minute narrative branch to check one condition.

### Concepts (10 min)
- **EditMode vs PlayMode tests.** EditMode runs instantly in a plain .NET context — perfect for Core. PlayMode boots the engine and is slow; reserve it for genuine integration checks.
- **This is why the wall exists.** Core has no `MonoBehaviour`, so it needs no engine, so tests are instant.
- **Test the rules, not the framework.** Don't test that `List.Add` works. Test that taking the Assize sets the flag, records the ledger entry, and increases lifetime coin.
- **Deterministic time and randomness.** Inject them. Anything in Core that reads a clock or an RNG directly is untestable, and you will need both.
- **Test names as documentation** — `TakingTheAssize_RecordsConscienceCostAndRaisesLevy` tells you what the game does.

### Build (40 min)
1. Structure the test project to mirror Core: `Core.Tests/State/`, `Core.Tests/World/`.
2. Write a `GameStateBuilder` test helper — fluent, e.g. `A.GameState().WithFlag(x).WithCoin(50).Build()`. This will save you hours over the next ten weeks; build it now while it's cheap.
3. Write real tests for Day 23–24's code: state factory validity, flag idempotency, ledger ordering and totals, lifetime coin never decreasing.
4. **Introduce `IClock`** in Core with a `SystemClock` and a `FakeClock`. Replace any direct time access. Same for `IRandom` if anything needs randomness.
5. Run the whole suite. It should take well under a second. Notice how different that is from testing by playing.
6. Deliberately break something in Core and watch the right test fail. Fix it.
7. Add a `dotnet`-style convention note to `CONVENTIONS.md`: every Core rule ships with a test, no exceptions.

### Acceptance criteria
- [ ] Test suite runs green in under one second
- [ ] `GameStateBuilder` helper exists and is used
- [ ] Time and randomness are injected, not accessed directly
- [ ] Tests are named as behaviour descriptions
- [ ] You've seen a deliberate break produce a targeted failure

### Failure modes
- **Tests don't appear in the Test Runner** → asmdef isn't marked as a test assembly, or the folder is outside the test asmdef.
- **Tests are slow** → something pulled in Unity. Check your references; that's a wall breach.
- **Testing implementation instead of behaviour** → if renaming a private method breaks a test, the test is wrong.

**Stretch:** Set up a GitHub Action that runs the Unity test suite on push. More work than it sounds like (Unity licensing in CI), but genuinely satisfying, and it's a real buffer-day project.

**Commit:** `test(core): test harness, builders, and injected clock`

---

## Day 26 — The bridge: how Unity observes Core
**Mon 5 Oct · 60 min**

**Objective:** Core raises events. Unity subscribes and renders. Core still doesn't know Unity exists.

**Why:** This is the actual interface between your two worlds. Get it right and the 3D port is a matter of writing new listeners. Get it wrong and you'll be untangling it in December.

### Concepts (10 min)
- **Direction of knowledge:** Unity knows about Core. Core knows nothing. Every arrow points one way.
- **Plain C# events, not `UnityEvent`.** Core can't reference `UnityEngine`, which conveniently forces the right choice.
- **Event payloads must be Core types.** `CoinChanged(int newAmount, int delta)` — never a Unity type, never a GameObject.
- **Who owns the `GameState` instance?** One Unity object — a `GameRoot` MonoBehaviour that constructs Core at startup and hands references down. This is composition root / DI, a pattern you already know. It is *not* a global singleton, even though Unity culture will push you toward one.
- **Subscribe in `OnEnable`, unsubscribe in `OnDisable`. Always pair them.** An unpaired subscription is a leak and a `MissingReferenceException` waiting to happen.

### Build (40 min)
1. `Core/Events/GameEvents.cs` — a class Core systems raise through. `event Action<FlagId> FlagSet`, `event Action<int,int> CoinChanged`, `event Action<ConsciencePoint> ConscienceRecorded`.
2. Wire the existing systems to raise them. Test that a flag set raises exactly one event with the right payload.
3. `Unity/GameRoot.cs` — a MonoBehaviour that on `Awake` builds the `GameState`, the systems, and the event bus, and exposes them. **One instance, in the scene, referenced by whoever needs it.**
4. `Unity/Debug/CoreEventLogger.cs` — subscribes to everything and `Debug.Log`s it. Crude, and it will be your primary window into Core for the next month.
5. Prove the loop: a temporary key that grants 10 coin in Core → event fires → Unity logs it. Small, but that's the whole architecture demonstrated end to end.
6. **Audit every existing script** for game logic that should have moved. Player movement staying in Unity is correct — that's input and physics, not rules.
7. Write down the rule in `CONVENTIONS.md`: *Unity classes may read input, render state, and forward events. Nothing else.*

### Acceptance criteria
- [ ] Core raises typed events with Core-only payloads
- [ ] `GameRoot` constructs Core once and hands out references
- [ ] A Unity component logs Core events
- [ ] All subscriptions are paired `OnEnable`/`OnDisable`
- [ ] Core still has zero UnityEngine references
- [ ] The one-way rule is written down

### Failure modes
- **Reaching for a singleton** → tempting, and it will make M10 harder and testing worse. Pass references from `GameRoot`.
- **`MissingReferenceException` on scene change** → unpaired subscription.
- **Wanting to pass a `Transform` through an event** → that's the wall telling you the design is wrong. Pass an ID; let Unity look up the object.

**Stretch:** Add an `IGameLog` interface in Core with a Unity implementation that writes to the Console. Now Core can log without knowing what logging is — a clean, tiny example of the whole pattern.

**Commit:** `feat: core event bus and unity game root`

---

## Day 27 — ScriptableObjects as content, never as logic
**Tue 6 Oct · 60 min**

**Objective:** Author content in the Inspector, convert it to Core types at load, and keep Core ignorant that ScriptableObjects exist.

**Why:** SOs are genuinely excellent for authoring and genuinely terrible as a place to put rules. Almost every Unity tutorial gets this wrong and it will not survive your 3D port.

### Concepts (10 min)
- **A ScriptableObject is an asset with serialized data.** Great for editing content in the Inspector with validation, references, and previews.
- **The trap:** "ScriptableObject architecture" tutorials put *behaviour and runtime state* in them. Then your rules are Unity types, and they can't be tested, and they can't be ported. Don't.
- **The pattern that works:** SO is an **authoring format**. At load, map it to a plain Core type. Core sees only the plain type.
  ```
  CharacterDefinitionSO  --(mapper)-->  Core.CharacterDefinition
  ```
- **Alternative: JSON.** Even more portable, better diffs, no Inspector nicety. Perfectly valid; you'll use JSON for dialogue on Day 30 and SOs for characters. Knowing *why* you'd pick each is today's real lesson.
- **SOs share state across the whole project.** Mutating one at runtime persists in the editor between plays and does not persist in a build. This inconsistency has cost people entire weekends.

### Build (40 min)
1. `Core/Content/CharacterDefinition.cs` — plain C#. `CharacterId`, display name, portrait *key* (a string, not a Sprite — Core can't hold a Sprite).
2. `Unity/Content/CharacterDefinitionSO.cs` — a `ScriptableObject` with the same fields plus the actual `Sprite`, and a `ToCore()` method.
3. `Unity/Content/ContentDatabase.cs` — a MonoBehaviour or SO holding lists of every definition SO, with `BuildCoreCatalog()` returning Core types and `GetPortrait(CharacterId)` for the view layer.
4. Author real content: Osric, Enid, Vance, Bran, Iselde, Corvin, Tam. Names and portrait keys from `reference/story-bible.md` §7. Placeholder portraits are fine.
5. `GameRoot` builds the catalog on `Awake` and hands it to Core.
6. Test that Core resolves a character by ID and gets a valid definition — **with no SO involved in the test.** That's the proof the separation is real.
7. **Write down the rule:** *SOs and JSON are authoring formats. Core consumes plain types. Nothing in Core knows how content was authored.*

### Acceptance criteria
- [ ] Core content types are plain C# with no Unity references
- [ ] SOs exist purely to author and map
- [ ] Seven characters authored
- [ ] Core resolves characters by ID
- [ ] Core tests run without touching any SO
- [ ] Portrait sprites live only on the Unity side

### Failure modes
- **Putting a `Sprite` field in a Core type** → the compiler stops you. Use a string key and resolve it in the view.
- **Mutating SOs at runtime** → editor-persistent, build-transient. Copy the data out first.
- **Wondering "why not just use the SO everywhere?"** → because on Day 71 you'll build a new project and want this content to come along without dragging Unity's asset system through your rules.

**Stretch:** Write a small editor validator (in `Hearthfall.Editor`) that flags any character SO with a missing portrait or duplicate ID. Content validation tooling pays for itself dramatically once you have 60 dialogue nodes.

**Commit:** `feat: scriptableobject content authoring with core mapping`

---

## Day 28 — BUFFER
**Wed 7 Oct**

- **Catch up** — this milestone is dense; using the whole day here is normal and expected.
- **Refactor** — now that the wall exists, is there logic still sitting in a MonoBehaviour that shouldn't be? Move it.
- **Write more tests.** Cheap now, invaluable in M05.
- **Story homework:** `reference/story-bible.md` wants the father's specific pride and Vance's best argument by M05. Today's a fine day for one of them.
- **Rest.**

### Milestone review

Run `/review` and specifically ask it to check the wall: is there anything in Core that shouldn't be, and is there anything in Unity that should be in Core? **This is the highest-value review of the entire curriculum.** Take its "must fix" list seriously.

### What you just did

The game looks identical to a week ago and you have done the most important work of the project.

You now have a testable, portable, engine-independent model of Hearthfall's rules. Tomorrow you start building dialogue on top of it, and in ten weeks you'll import this exact assembly into a 3D project and it will simply work.

That is not a normal thing for someone eight days into their first real Unity project to have. Most codebases never get there at all.

**Commit:** `docs: M03 complete — engine-agnostic core`
