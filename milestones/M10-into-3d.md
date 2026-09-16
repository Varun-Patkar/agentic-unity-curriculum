# M10 · Into 3D — New View, Same Core

**Days 71–77 · 19–25 Nov 2026 · project: `Hollowbrook3D`**

> Two days ago you shipped the 2D version of *Last Stop, Hollowbrook*. Today the presentation layer starts again, but the expensive part does not: `Hollowbrook.Core`, its tests, and its content remain the source of truth.
>
> This week is the proof of that architecture. By Day 75, Mayor Vale's briefing runs in a 3D Town Hall greybox from the same dialogue graph, and the decisions, quests, and town reactions built in 2D still work without being redesigned.

**You end holding:** a 3D scene running Hollowbrook's current dialogue, choice, quest, and consequence content against an unchanged shared Core.

---

## Day 71 — New 3D URP project, and importing Core untouched
**Thu 19 Nov · 60 min**

**Objective:** A fresh `Hollowbrook3D` project where `Hollowbrook.Core` compiles and its full EditMode test suite is green with zero rule changes.

**Why:** This is the falsifiable test of M03. A clean project exposes presentation leaks before six weeks of 3D work grow around them.

### Concepts (10 min)
- **New project, not conversion.** Keep the shipped 2D game shippable and create a clean Universal 3D project.
- **One Core, two views.** Reference one versioned `Hollowbrook.Core`; never maintain copied source trees.
- **Core content travels with Core.** Dialogue graphs, the four central choices, quests, town reactions, and ending/profile rules are not 2D assets.
- **The boundary remains enforced.** `Hollowbrook.Core` keeps zero `using UnityEngine`; 3D positions, physics, cameras, and animation stay in the Unity assembly.

### Build (40 min)
1. Unity Hub → New Project → **Universal 3D**, name `Hollowbrook3D`, path `D:\Projects\Unity Games\Hollowbrook3D`.
2. Initialise git and commit the empty project with a Unity `.gitignore`.
3. Put `Hollowbrook.Core` and its tests in one shared package or repository, referenced by both Hollowbrook projects.
4. Confirm the 2D project still compiles and runs.
5. Resolve asmdef references in `Hollowbrook3D`; keep `noEngineReferences` enabled for Core.
6. Run all Core EditMode tests inside the new project.
7. Load and validate the current content: Mayor briefing, four central choices, quests, delayed town reactions, and all three ending/profile rules.

### Acceptance criteria
- [ ] `Hollowbrook3D` exists as a URP project under git
- [ ] Both projects reference one `Hollowbrook.Core`
- [ ] The 2D project still compiles and runs
- [ ] Core's EditMode tests are green in `Hollowbrook3D`
- [ ] Current dialogue, choice, quest, reaction, and ending content validates
- [ ] Zero Core rule changes, or every unavoidable portability fix is documented

### Failure modes
- **Tests do not appear** → verify the test asmdef and package test settings.
- **Core namespace cannot be found** → add the assembly definition reference; do not copy source files into `Assets`.
- **Content loads only in the editor** → remove `AssetDatabase` assumptions and use the shared content-loading contract.

**Commit:** `chore: create Hollowbrook3D with shared core`

---

## Day 72 — Scene fundamentals: units, scale, and materials
**Fri 20 Nov · 60 min**

**Objective:** A correctly scaled Town Hall test space with floor, walls, doorway, ramp, steps, lighting, and valid URP materials.

**Why:** Physics, animation, navigation, and camera tuning all assume consistent metric scale.

### Concepts (10 min)
- **1 Unity unit = 1 metre.** Keep a 1.8 m human reference in every blockout.
- **Greybox is intentional.** It tests space before downloaded assets make poor dimensions expensive to change.
- **Magenta means shader mismatch.** Convert imported materials to URP rather than treating it as a modelling problem.
- **Fix scale at import.** Do not compensate for a wrongly scaled character with Transform scale.

### Build (40 min)
1. Create `Assets/_Project/Scenes/TownHall_Greybox.unity`.
2. Block a compact municipal office, corridor, exterior step, two ramps, stairs, and a narrow doorway.
3. Add a 1.8 m reference capsule and verify human-scale doors and furniture volumes.
4. Create simple URP/Lit materials with enough contrast to read edges.
5. Set a deliberate directional-light angle and inspect the Lighting window.
6. Deliberately reproduce and fix one magenta material.

### Acceptance criteria
- [ ] Town Hall greybox exists and reads at human scale
- [ ] Ramps, steps, doorway, and collision test spaces are present
- [ ] Materials render correctly in URP
- [ ] Lighting is deliberate rather than default
- [ ] Character-scale objects have no Transform scale workaround

### Failure modes
- **Space feels wrong** → judge from Play mode beside the reference capsule.
- **Everything is black or flat** → inspect the light direction, shadows, and environment lighting.
- **The player fits only after scaling** → fix the environment or model import scale.

**Commit:** `feat: block out Town Hall at metric scale`

---

## Day 73 — A stable third-person controller
**Sat 21 Nov · 60 min**

**Objective:** Alex walks, runs, and turns camera-relative, handles Town Hall steps and ramps, and remains grounded without jitter.

**Why:** This modern choice RPG needs reliable traversal more than physics spectacle.

### Concepts (10 min)
- **Use `CharacterController`.** It provides predictable slopes, steps, and collision for authored third-person movement.
- **Movement is camera-relative.** Transform the Input System move vector by camera yaw.
- **Rotation and movement are separate.** Smoothly face travel direction during ordinary locomotion.
- **Gravity is manual.** Accumulate vertical velocity and apply one `CharacterController.Move` per frame.
- **Traversal remains presentation code.** No movement types or positions enter Core.

### Build (40 min)
1. Port the Input System action maps: Gameplay, Dialogue, and UI; add `Look` for mouse and right stick.
2. Create Alex's temporary capsule with a `CharacterController` at human scale.
3. Implement camera-relative movement, smoothed turning, grounded handling, and gravity.
4. Add walk/run speeds and tune against the greybox.
5. Test the shallow ramp, steep ramp, steps, doorway, wall contact, and ledge.

### Acceptance criteria
- [ ] Movement stays camera-relative while the camera turns
- [ ] Turning is smooth and ordinary traversal feels controlled
- [ ] Gravity and grounded handling are stable
- [ ] Steps work, steep slopes fail cleanly, walls do not jitter
- [ ] Core contains no movement or physics logic

### Failure modes
- **Movement stays world-relative** → camera yaw was not applied.
- **Wall jitter** → movement is split across update loops or `Move` is called twice.
- **Grounding flickers** → reinforce `isGrounded` with a short ground probe and a small downward velocity.

**Commit:** `feat: camera-relative third-person movement`

---

## Day 74 — Cinemachine in 3D: follow, orbit, and collision
**Sun 22 Nov · 60 min**

**Objective:** A third-person camera with mouse/stick orbit, useful damping, vertical limits, and geometry avoidance.

**Why:** The camera is the player's main way of reading navigation, interaction, and short encounters.

### Concepts (10 min)
- **Cinemachine describes camera intent.** The Main Camera is the output; Cinemachine cameras supply behaviours.
- **Version names move.** Verify installed Cinemachine 3.x component names against current documentation.
- **Follow a head-height target.** Do not aim at Alex's feet or inherit body rotation into the orbit target.
- **Sensitivity and invert Y are settings.** Expose them now for M15.

### Build (40 min)
1. Install Cinemachine and record the installed major version.
2. Add a head-height `CameraTarget` child whose rotation is decoupled from Alex's body.
3. Configure a Cinemachine camera to follow and orbit the target.
4. Bind `Look`, clamp vertical orbit, and expose sensitivity/invert Y.
5. Add current-version deocclusion support and test every Town Hall corner.
6. Tune damping while walking and running.

### Acceptance criteria
- [ ] Camera orbits with mouse and right stick
- [ ] Vertical orbit is clamped
- [ ] Walls pull the camera in without clipping
- [ ] Sensitivity and invert Y are configurable
- [ ] Movement remains camera-relative throughout an orbit

### Failure modes
- **Documented component is absent** → check whether the source describes Cinemachine 2.x.
- **Camera and body spin together** → decouple the follow target's rotation.
- **Judder** → align controller and Cinemachine update timing.

**Commit:** `feat: add Cinemachine third-person camera`

---

## Day 75 — The proof: Mayor Vale's briefing in 3D
**Mon 23 Nov · 60 min**

**Objective:** Walk Alex into a Town Hall greybox and run Mayor Silas Vale's complete opening briefing from unchanged Core content.

**Why:** This proves the shipped narrative game is being presented again, not rewritten as a new game.

### Concepts (10 min)
- **Core remains closed for feature work.** `GameRoot` composes existing systems; the 3D UI observes their events.
- **The content is broader than one conversation.** The briefing must connect to the same decision ledger, four central choices, quests, prepared-town flags, bargain terms, delayed town reactions, and profile history.
- **The skip mechanic keeps its contract.** Core owns `EarlyAdvanceCount` and irritation stage. Unity only presents Vale's expression and later staging.
- **uGUI is still valid.** A screen-space dialogue canvas can subscribe to the unchanged runner.

### Build (40 min)
1. Port the Unity composition root and adapters into the `Hollowbrook.Unity` assembly.
2. Rebuild the dialogue canvas with speaker, body text, choices, and typewriter reveal.
3. Place temporary modern civilian stand-ins for Alex and Mayor Vale in Town Hall.
4. Start the real Mayor briefing through the normal dialogue runner and unchanged graph.
5. Take a branch and inspect resulting decisions/flags in the existing state tool.
6. Load the quest catalog and verify references for the deputy's files, mine entrance, June Mercer, and emergency siren choices.
7. Trigger one delayed town reaction through a test scenario.
8. Run the Core EditMode suite again.

### Acceptance criteria
- [ ] `GameRoot` constructs existing Core systems in the 3D scene
- [ ] Mayor Vale's briefing runs end to end from unchanged content
- [ ] Choices branch and update the decision ledger visibly
- [ ] Quest and delayed town-reaction content resolves through existing Core rules
- [ ] Dialogue input context enters and exits correctly
- [ ] Core tests remain green with no 3D rule fork

### Failure modes
- **A Unity type is requested by Core** → restore the adapter boundary instead of accepting the leak.
- **Skip count changes for accessibility reveal** → the Unity adapter is reporting the wrong input event; Core's fairness tests should catch it.
- **Portrait or expression is missing** → map content keys to Unity assets; do not put assets in Core.

**Commit:** `feat: run Mayor briefing in 3d on unchanged core`

---

## Day 76 — Interaction in 3D
**Tue 24 Nov · 60 min**

**Objective:** Alex reliably selects and interacts with the intended nearby person or object, with readable prompts and line-of-sight checks.

**Why:** The same intent-selection problem from 2D now includes camera direction and occlusion.

### Concepts (10 min)
- **Gather, then rank.** Use a nearby overlap for candidates and score by camera angle plus distance.
- **Use layers and line of sight.** The player and environment must not become accidental interactables.
- **Add hysteresis.** The current target keeps a small bonus so prompts do not flicker.
- **Interaction forwards IDs.** Unity chooses the viewed object; Core decides what that ID permits.

### Build (40 min)
1. Create an `Interactable` layer and modern Town Hall test objects.
2. Add an `Interactable` adapter exposing a Core ID and display verb.
3. Gather candidates with `Physics.OverlapSphereNonAlloc` and rank by camera-forward angle and distance.
4. Reject blocked targets with a line-of-sight raycast.
5. Add hysteresis and a camera-facing world prompt.
6. Test Vale and two nearby props, including blocked and behind-camera cases.

### Acceptance criteria
- [ ] Nearby candidates are ranked by camera-relative intent
- [ ] Layer masks exclude Alex and the environment
- [ ] Line of sight prevents interaction through walls
- [ ] Prompt faces the camera and does not flicker
- [ ] Interaction forwards an ID and does not duplicate Core conditions

### Failure modes
- **Nothing is detected** → build the mask from bits, not a raw layer index.
- **Prompt flickers** → retain a current-target scoring bonus.
- **Conversation availability differs from 2D** → remove Unity-side eligibility logic.

**Commit:** `feat: add camera-weighted 3d interaction`

---

## Day 77 — BUFFER
**Wed 25 Nov**

- Catch up on controller, camera, or package sharing.
- Re-run the Mayor briefing and one quest/town-reaction scenario without opening a Core source file.
- Greybox the Town Square threshold outside Town Hall for M11 NPC placement.
- Tune traversal and camera comfort before animation sits on top of them.

### Milestone review

Run `/review`. Ask whether any Unity type leaked into `Hollowbrook.Core`, whether Unity duplicated a dialogue/quest condition, and whether all current Hollowbrook content still validates from the shared source.

### Where you are

Seven days after the 2D release, Alex can move through a 3D Town Hall, speak with Mayor Vale, make existing choices, start existing quests, and observe existing town consequences. The rendering dimension changed; the game rules did not.

Tomorrow the stand-ins become modern Hollowbrook residents: civilian Alex, Mayor Vale, Deputy Pike, Mara Bell, June Mercer, and Eli Reed.

**Commit:** `docs: M10 complete — Hollowbrook enters 3d`
