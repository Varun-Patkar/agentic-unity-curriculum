# M10 · Into 3D — New View, Same Core

**Days 71–77 · 19–25 Nov 2026 · project: `Hearthfall3D`**

> Two days ago you shipped a game. Today you start again in a project with nothing in it, and almost everything you got good at over the last ten weeks — tilemaps, sprite sorting, 2D colliders, `Vector2` — stops applying. The curriculum predicted this week would be the low point, and it was right to.
>
> Here's the thing that makes it survivable: the half of your game that took the most thought is already done and does not care what dimension you render in. `Hearthfall.Core` compiles against nothing. On Day 75 you'll watch it run a full conversation in a 3D scene without a single line changed. Everything before that day is scaffolding to make that moment possible.
>
> This week is view code. View code is fast.

**You end holding:** a 3D scene running your dialogue system off code you did not modify. The moment M03 pays for itself.

---

## Day 71 — New 3D URP project, and importing Core untouched
**Thu 19 Nov · 60 min**

**Objective:** A fresh `Hearthfall3D` project where `Hearthfall.Core` compiles and its full test suite runs green — with zero edits to Core.

**Why:** This is the falsifiable test of M03. Either the wall held or it didn't, and you find out in the first hour rather than the third week.

### Concepts (10 min)
- **New project, not a conversion.** The 2D game is shipped and should stay shippable. Converting it means fighting 2D physics settings, sprite-configured URP assets, and a scene full of things you're about to delete. A clean Universal 3D template is cheaper.
- **Core must live in exactly one place.** Two copies diverge, silently, and you find out when a bug you fixed in one project reappears in the other.
- **Two sharing strategies, honestly:** a **git submodule** (single source of truth, but every Core change is two commits and a pointer bump, and you will occasionally push a project referencing a commit nobody else has) versus **a shared local folder plus a symlink or Unity's local package** (simpler day to day, but it isn't versioned with either game). Submodule is the right answer if you can tolerate the friction; you already know git well enough that you can.
- **Unity can reference code outside `Assets/`** via a local package (`Packages/manifest.json` with a `file:` path). This is the cleanest mechanism and it keeps Core out of both projects' asset trees.
- **JSON content is Core's, not Unity's.** Dialogue, quests, and letters travel with Core, not with the 2D project's `StreamingAssets`.
- **The trap:** "just one small change to Core to make it fit". That is not a fit problem, it is a design leak, and it means something Unity-shaped got into your rules in October.

### Build (40 min)
1. Create the project: Unity Hub → New Project → **Universal 3D** template, name `Hearthfall3D`, path `D:\Projects\Unity Games\Hearthfall3D`. *(Template names shift between Hub versions — if you see "3D (URP)" or "Universal 3D Core", it's that one.)*
2. `git init`, commit the empty project with a Unity `.gitignore` before you touch anything else.
3. Move `Hearthfall.Core` and `Hearthfall.Core.Tests` out of the 2D project into their own repo at `D:\Projects\Unity Games\Hearthfall.Core`. Add it back to *both* projects as a submodule under `Packages/` or a local package path.
4. Verify the 2D project still compiles and still ships. Do this now, not later.
5. Get the asmdefs resolving in `Hearthfall3D`. `noEngineReferences` stays true.
6. `Window > General > Test Runner` → EditMode → run everything. **It should be green on the first run.**
7. Bring the JSON content across — dialogue graphs, quest definitions, Enid's letters — and run the loader and validator against them from a test.
8. **If you had to edit anything in Core, write down exactly what and why in `progress/STATE.md`.** That note is a design bug report on your October self.

### Acceptance criteria
- [ ] `Hearthfall3D` exists, URP, under git, first commit made
- [ ] Core lives in one place and is referenced by both projects
- [ ] The 2D project still compiles and runs
- [ ] Core's EditMode tests run green inside `Hearthfall3D`
- [ ] JSON content loads and validates in the new project
- [ ] Zero lines of Core changed — or a written note explaining each one

### Failure modes
- **Unity ignores the folder** → code outside `Assets/` only compiles via a package with a valid `package.json`. Check the manifest path and the name field.
- **Tests don't appear** → the test asmdef needs `UnityEngine.TestRunner` and the "Test Assemblies" flag; a package needs a `Tests` folder marked accordingly.
- **Submodule looks empty after clone** → `git submodule update --init`. It bites everyone once.
- **"The type or namespace `Hearthfall.Core` could not be found"** → the Unity asmdef in the new project doesn't reference it yet. Inspector → Assembly Definition References.

**Stretch:** Add a `dotnet test` run of Core outside Unity entirely, via a plain `.csproj` referencing the same sources. Fast feedback and a very concrete proof of portability.

**Commit:** `chore: new 3d project with shared core`

---

## Day 72 — Scene fundamentals: units, scale, and why everything is grey
**Fri 20 Nov · 60 min**

**Objective:** A greybox test space at correct human scale — floor, wall, ramp, steps — lit, with materials that aren't magenta.

**Why:** Every 3D problem you'll hit in the next six weeks is downstream of scale. Get it wrong now and gravity feels like the moon, and Mixamo characters arrive the size of a house on Day 78.

### Concepts (10 min)
- **1 unit = 1 metre.** Unity's physics constants, `Physics.gravity` at `-9.81`, and every humanoid animation asset assume this. It is a convention, not enforced, and breaking it is the single most expensive beginner mistake in 3D.
- **Human reference: ~1.8 units tall.** A default capsule is 2 units. Build your greybox against that: doorways ~2.2, steps ~0.2 rise, waist-high wall ~1.
- **Everything is grey because URP's default lit material is white with no albedo texture, under one directional light.** That's correct, not broken. Greybox first, art later — it's the standard workflow, not a compromise.
- **Magenta means shader mismatch.** A material built for the Built-in pipeline in a URP project renders magenta. Fix: `Window > Rendering > Render Pipeline Converter`, or change the material's shader to `Universal Render Pipeline/Lit`.
- **The directional light is your sun.** Rotation is the time of day; the skybox both draws the sky and provides ambient light. Change the sun angle and the whole scene's mood moves.
- **Scale on transforms is a lie you'll regret.** Scaling a GameObject to fit is fine for greybox blocks and quietly wrong for characters and physics. Fix scale at import.

### Build (40 min)
1. New scene, `Assets/_Project/Scenes/Greybox.unity`. Delete nothing yet — keep the default camera and Directional Light.
2. Floor: `GameObject > 3D Object > Plane`, scaled 5×1×5 (a Plane is 10 units across, so that's 50m). Note the units as you go.
3. Build the test space with cubes: a wall you can't cross, a ramp at roughly 20°, another at ~45°, a short stair of four steps at 0.2 rise, and a narrow gap. These exist specifically to break tomorrow's controller.
4. Drop a capsule at the origin as the human reference. Confirm it's 2 units and everything you built reads at the right size against it.
5. Create two materials in `Assets/_Project/Art/Materials/` — a mid-grey floor and a slightly darker wall — using `Universal Render Pipeline/Lit`. Just enough contrast to see edges.
6. Rotate the Directional Light to a low, warm angle. Look at `Window > Rendering > Lighting` and find the skybox and ambient settings. *(Menu paths under Window > Rendering moved in Unity 6 — if it isn't where I said, check the current docs rather than hunting.)*
7. Deliberately break one material: assign a Built-in Standard shader to it, see the magenta, then fix it. Thirty seconds now saves fifteen minutes in December.
8. Save the scene, commit.

### Acceptance criteria
- [ ] A greybox scene with floor, wall, two ramps, steps, and a gap
- [ ] Everything is at metric scale against a 1.8–2 unit reference
- [ ] Materials use URP shaders and the scene reads clearly
- [ ] The directional light is deliberately placed, not default
- [ ] You have seen the magenta shader-mismatch and fixed it
- [ ] No GameObject in the scene has a non-uniform character-scale hack

### Failure modes
- **Everything is black** → no light, or the light is pointing up, or the object's normals are inverted.
- **Scene looks flat and washed out** → all one material, one light angle, no shadow. Move the sun.
- **Objects fall through the floor at speed** → colliders and fixed timestep, not scale. Note it; Day 73 covers it.
- **You scaled the player capsule to fit the doorway** → wrong direction. Fix the doorway.

**Stretch:** Add a second scene that's just a flat infinite plane, for testing movement without the greybox in the way. You'll use it more than you expect.

**Commit:** `feat: greybox test scene at metric scale`

---

## Day 73 — A third-person controller that isn't a physics accident
**Sat 21 Nov · 60 min**

**Objective:** A character that walks, runs, and turns relative to the camera, handles the ramps and steps, and doesn't jitter.

**Why:** Movement feel is the thing players judge in the first five seconds. In 3D it's also where the most confidently wrong tutorial code on the internet lives.

### Concepts (10 min)
- **`CharacterController` vs Rigidbody, honestly.** `CharacterController` is a kinematic capsule with built-in slope limits, step offset, and a `Move` that resolves collisions for you. You get predictable, jitter-free movement and no free physics — no knockback, no being pushed by objects, no forces. Rigidbody gives you real physics and a permanent tuning problem. **For a third-person action game with a designed feel, use `CharacterController`.** You can fake knockback later with a scripted impulse; you cannot easily un-jitter a Rigidbody controller.
- **Camera-relative movement is the thing beginners get wrong.** Pressing forward should move the character away from the camera, not along world +Z. Transform the input vector by the camera's yaw before you use it.
- **Rotation is separate from movement.** The character moves in the input direction and *turns toward* it over time. `Mathf.SmoothDampAngle` on the Y euler angle gives you a turn that reads well; snapping looks robotic.
- **Gravity is manual.** `CharacterController` does not apply it. Accumulate a vertical velocity every frame, reset it to a small negative value when grounded (not zero — zero makes ground checks flicker), and feed it into the same `Move` call.
- **Ground check.** `controller.isGrounded` is correct only immediately after a `Move`. It is unreliable on slopes and edges; a short `SphereCast` down from the capsule base is the standard reinforcement.
- **All of this is view code.** None of it goes anywhere near `Hearthfall.Core`. Movement is input and physics, exactly as you wrote in `CONVENTIONS.md` on Day 26.

### Build (40 min)
1. Bring the Input System package in and port your `InputRouter` from the 2D project. Same Gameplay / Dialogue / UI action maps — they were designed to be view-agnostic and this is the payoff.
2. Change the Gameplay `Move` action to a 2D vector on WASD and left stick (unchanged), and add a `Look` action for mouse delta and right stick. `Look` goes unused until tomorrow.
3. Player GameObject: capsule mesh child, `CharacterController` component. Set Height 1.8, Radius 0.3, Center Y 0.9, Slope Limit 45, Step Offset 0.25.
4. `Unity/Player/PlayerLocomotion.cs`. Read `Move`, build a world-space direction from the camera's yaw, compute a target angle with `Mathf.Atan2`, smooth-damp the rotation, and move along the *rotated* forward.
5. Add gravity as a separate accumulated vertical component, applied in the same `controller.Move` call. One `Move` per frame, never two.
6. Add walk and run speeds behind a `Sprint` action. Tune them against the greybox: crossing 50m should feel like a walk, not a commute.
7. Test against every obstacle you built yesterday: the 20° ramp (should walk up), the 45° (should refuse or slide), the steps (should climb without a jump), the gap (should not pass).
8. Tune for ten minutes. Acceleration, turn speed, run multiplier. This is the whole day's real work.

### Acceptance criteria
- [ ] Character moves camera-relative — forward is away from the camera, always
- [ ] Turning is smoothed, not snapped
- [ ] Gravity is applied manually and the character stays grounded on slopes
- [ ] Steps are climbed, the steep ramp is refused, the gap is not crossed
- [ ] Walk and run both feel deliberate
- [ ] No movement logic anywhere near Core

### Failure modes
- **Character moves in world directions regardless of camera** → you skipped the yaw transform. The single most common 3D beginner bug.
- **Jitter or vibration against walls** → two `Move` calls in one frame, or movement in `FixedUpdate` while the camera updates in `LateUpdate`.
- **Falls slowly, floats, moon gravity** → scale is wrong, or you're multiplying gravity by `deltaTime` once instead of twice (velocity accumulates by `g·dt`, displacement by `v·dt`).
- **`isGrounded` flickers on flat ground** → you reset vertical velocity to exactly 0. Use about `-2`.
- **Sinks through the floor when standing still** → the capsule's Center doesn't match its Height.

**Stretch:** Add a dodge — a short, fixed-duration burst along the current input direction with movement locked. It's the seed of the M12 combat system and it's ten lines today.

**Commit:** `feat: camera-relative third-person controller`

---

## Day 74 — Cinemachine in 3D: follow, orbit, and collision
**Sun 22 Nov · 60 min**

**Objective:** An orbiting third-person camera with mouse and stick control, sensible damping, and no clipping through walls.

**Why:** The camera is half of "feel" in a 3D game and all of "can the player see what's happening". Hand-rolling one is a week of work; Cinemachine is an hour.

### Concepts (10 min)
- **Cinemachine is a camera *behaviour* system.** Your actual `Camera` becomes a dumb output driven by a Cinemachine Brain; virtual cameras describe intent and the Brain blends between them.
- **Cinemachine 3.x renamed most of 2.x.** `CinemachineVirtualCamera` became `CinemachineCamera`; the Body/Aim stage components were renamed and reorganised; `CinemachineFreeLook`'s three-rig model was replaced by an orbital component with a follow target. **Almost every tutorial you find will be 2.x.** Before you follow anything, check the version in Package Manager and read the current Cinemachine documentation for component names. Do not let me guess this for you.
- **What you want structurally:** a follow target that is a child of the player at roughly head height, an orbital rig around it, and a "look at" aim. Never point the camera at the player's feet.
- **Damping is the entire feel dial.** Too little and the camera is nauseating; too much and it lags behind the action. Different values on each axis is normal.
- **Camera collision** is a separate concern — Cinemachine's Deocclusion / Collider extension pulls the camera in when geometry intrudes. Without it, you'll see the inside of walls constantly.
- **Sensitivity and invert-Y are settings, not constants.** Bake them in as serialized fields today and route them to the options menu when M15 gets there. Someone will need invert-Y, and that someone might be you.

### Build (40 min)
1. Install Cinemachine via Package Manager. **Note the major version** — everything below assumes 3.x.
2. Add a Cinemachine Brain to your Main Camera (Cinemachine adds it automatically when you create a camera; confirm it's there).
3. Create an empty `CameraTarget` as a child of the player at about Y = 1.6, and don't rotate it with the character body.
4. Create a Cinemachine camera, set Follow and Look At to `CameraTarget`, and add the orbital positioning component. *(Exact component name and menu path: verify against the installed version's docs.)*
5. Wire the `Look` input action to the orbit's horizontal and vertical inputs. Cinemachine 3.x has an Input Axis Controller component that binds to Input System actions — use it rather than driving the camera from your own script.
6. Clamp vertical orbit to roughly -30° to 60°. Add `mouseSensitivity` and `invertY` serialized fields, applied to the axis values.
7. Add the deocclusion/collider extension. Walk the player behind the greybox wall and confirm the camera pulls in instead of clipping through.
8. Tune damping until following the run feels neither seasick nor sluggish. Then walk the whole greybox once and fix whatever annoyed you.

### Acceptance criteria
- [ ] Camera orbits with mouse and right stick
- [ ] Vertical orbit is clamped and doesn't flip
- [ ] Camera never enters geometry
- [ ] Sensitivity and invert-Y are serialized settings
- [ ] Movement is still camera-relative and consistent while orbiting
- [ ] Following the character at run speed feels controlled

### Failure modes
- **Tutorial components don't exist** → 2.x tutorial, 3.x package. Check the docs for the current name. Expect this several times today.
- **Camera and character fight each other, spinning** → the camera target is a child that rotates with the body while the body turns toward the camera direction. Decouple the target's rotation.
- **Judder while moving** → Cinemachine's update method versus your controller's update timing. There's a Brain setting for this; try Late Update / Smart Update.
- **Camera snaps violently when passing a corner** → deocclusion damping is too aggressive, or the collider extension's smoothing time is zero.
- **Look input does nothing** → the action map isn't enabled, or `Look` is bound to a Vector2 while the axis controller expects individual axes.

**Stretch:** Add a second Cinemachine camera framed for conversations — tighter, over the shoulder — and blend to it manually. You'll need exactly this tomorrow.

**Commit:** `feat: cinemachine third-person orbit camera`

---

## Day 75 — The proof: a Hearthfall conversation in 3D
**Mon 23 Nov · 60 min**

**Objective:** Walk up to a capsule named Osric in a grey box and have the same conversation, driven by the same Core, from the same JSON.

**Why:** Because on Day 22 you spent a week building something that changed nothing on screen, on the promise of today. Today is the invoice being paid.

### Concepts (10 min)
- **Nothing new gets built in Core today.** If you find yourself opening a Core file, stop and ask what the view is trying to push down that it shouldn't.
- **`GameRoot` ports as-is.** It's a composition root — it constructs `GameState`, the systems, the event bus, and the content catalog. None of that knows about dimensions.
- **`DialogueRunner` already returns everything the UI needs:** speaker, line, available choices. Your 2D UI subscribed to that. Your 3D UI subscribes to the identical events.
- **uGUI screen-space canvases work identically in 3D.** A screen-space dialogue box in a 3D game is not a compromise; it's what most third-person RPGs do. Reuse your prefab structure.
- **The input context switch is already solved.** Gameplay → Dialogue on start, back on end, through the same `InputRouter`. This is why the action maps were split in M02.
- **What you're actually testing:** whether the M03 wall was real or aspirational. Nothing else today matters.

### Build (40 min)
1. Copy `GameRoot.cs` and the dialogue UI scripts from the 2D project into `Hearthfall3D`'s `Hearthfall.Unity` assembly. Fix only what genuinely doesn't compile — expect that to be sprite and 2D-collider references, nothing more.
2. Put a `GameRoot` in the greybox scene. Confirm on Play that `CoreEventLogger` prints startup events to the Console.
3. Rebuild the dialogue canvas: screen-space overlay, speaker name, portrait image, body text with your typewriter effect, a vertical choice list. Same hierarchy as the 2D prefab.
4. Stand a capsule in the greybox with a `CharacterId` of Osric on it. Give it a temporary trigger collider and a "press E" that just calls `DialogueRunner.Start(conversationId)` — proper interaction is tomorrow's job.
5. Load Osric's real conversation JSON. **The same file, unmodified.**
6. Play it. Walk over, press E, read the lines, take a branch, see a flag set in the Console.
7. Open the State Inspector window from `Hearthfall.Editor` — port it if you haven't — and watch the flags and the Conscience ledger change while you talk.
8. Run the full EditMode suite once more. Green.
9. **Then stop and look at it for a minute.** A grey capsule in a grey box is having a real conversation with branching and consequence because you spent a week in October making rules that don't care about pixels. That is not a small thing and it will not feel like this again.

### Acceptance criteria
- [ ] `GameRoot` runs in the 3D scene and constructs Core
- [ ] A conversation runs end to end from unmodified JSON
- [ ] Choices branch and set flags, visible in the State Inspector
- [ ] Input context switches to Dialogue and back
- [ ] Core's test suite is green in `Hearthfall3D`
- [ ] Core has still had zero lines changed since Day 71

### Failure modes
- **You edited a Core file** → find out why. A `Sprite`, a `Vector3`, or a `MonoBehaviour` sneaking in is a design leak, and today is the cheapest day of the whole project to fix it.
- **Portraits are missing** → portrait *keys* are in Core, sprites are in the Unity content database. The keys came across; the assets didn't. Copy them.
- **Dialogue advances on every frame** → input reading in `Update` without a press check, or the Gameplay map is still enabled underneath.
- **Text renders behind the world** → Canvas render mode is World Space or Screen Space - Camera with a bad plane distance. Overlay for now.
- **Nothing happens on E** → the trigger collider needs `Is Trigger`, and one of the two objects needs a Rigidbody for trigger events to fire at all.

**Stretch:** Blend to the over-the-shoulder conversation camera from yesterday's stretch when dialogue starts, and back when it ends. Two lines, and the difference in presence is enormous.

**Commit:** `feat: dialogue running in 3d on unmodified core`

---

## Day 76 — Interaction in 3D: raycasts, triggers, and "what am I looking at?"
**Tue 24 Nov · 60 min**

**Objective:** Reliable interaction with the nearest sensible target, with a world-space prompt that always faces the camera.

**Why:** This is the direct 3D analogue of Day 34's facing-weighted selection. The problem is the same — *which of these things does the player mean?* — but in 3D the player's intent is expressed by the camera as much as the body.

### Concepts (10 min)
- **Two candidate approaches.** A **camera-forward raycast** ("what am I looking at") is precise and matches player intent in a third-person game, but fails on small targets and when the camera is orbited away. A **sphere overlap around the player** ("what's near me") never misses but has no sense of intent. Best answer: **overlap to gather candidates, then score them by angle to camera-forward.** Same shape as your 2D facing weight, one dimension richer.
- **`Physics.OverlapSphere` vs `Physics.SphereCast` vs `Raycast`.** Overlap gathers everything in a radius; SphereCast is a thick ray that tolerates aim error; Raycast is a line. Know which question each answers.
- **Layer masks are not optional.** Without one, your raycast hits the player's own capsule on frame one and nothing else ever. Put interactables on an `Interactable` layer and mask to it.
- **The prompt must face the camera.** A world-space canvas rendered flat in the scene is readable from exactly one angle. Billboarding — rotating the prompt each `LateUpdate` to match the camera's forward — fixes it. `LateUpdate`, so the camera has already moved.
- **Candidate churn is the real bug.** Two objects scoring nearly identically make the prompt flicker between them. Add hysteresis: the current target keeps a small scoring bonus until something clearly beats it.
- **Interaction is view.** It resolves *which* Core entity the player means and then calls into Core with an ID. It never decides what interacting does.

### Build (40 min)
1. Create an `Interactable` layer. Move the Osric capsule onto it.
2. `Unity/Interaction/Interactable.cs` — a component exposing a Core-side ID, a display verb ("Talk to"), and an `Interact()` that forwards to `GameRoot`.
3. `Unity/Interaction/InteractionDetector.cs` on the player. Each frame: `Physics.OverlapSphere` at ~2.5m masked to `Interactable`, gather candidates.
4. Score each candidate: dot product of the camera's flattened forward against the direction to the candidate, weighted against distance. Highest score wins. Reject anything behind the player entirely.
5. Add a line-of-sight check — a `Raycast` from the camera target to the candidate against the environment layer, so you can't talk through the greybox wall.
6. Add hysteresis so the selection doesn't flicker between two close candidates.
7. `Unity/UI/WorldPrompt.cs` — a world-space canvas above the current target showing "E — Talk to Osric", billboarded in `LateUpdate`, fading in and out rather than popping.
8. Test the awkward cases deliberately: two capsules 1m apart, a capsule behind a wall, a capsule directly behind you, orbiting the camera while standing still between two of them.

### Acceptance criteria
- [ ] Candidates are gathered by overlap and ranked by camera-relative intent
- [ ] Layer mask excludes the player and the environment
- [ ] Line of sight is required — no talking through walls
- [ ] The prompt faces the camera from every angle
- [ ] Selection doesn't flicker between two close targets
- [ ] Interaction forwards an ID to Core and decides nothing itself

### Failure modes
- **Nothing is ever detected** → layer mask built from a layer *index* instead of `1 << index`. Everyone does this once.
- **The player detects itself** → the player's collider is on the interactable layer, or the mask is `~0`.
- **Prompt is mirrored or upside down** → billboarding by copying camera rotation instead of using `transform.forward = cam.forward`. Try both; one reads correctly.
- **Prompt lags a frame behind the camera** → billboard in `LateUpdate`, after Cinemachine has run.
- **Detection cost climbs with NPC count** → `OverlapSphereNonAlloc` and a fixed buffer. Not urgent at three capsules; note it.

**Stretch:** Add a subtle outline or emissive tint on the current target instead of relying on the prompt alone. A URP renderer feature or a scaled inverted-hull; either is a good 20 minutes.

**Commit:** `feat: 3d interaction with camera-weighted target selection`

---

## Day 77 — BUFFER
**Wed 25 Nov**

- **Catch up.** Day 73 and Day 74 both routinely overrun; if either is half-done, finish it here.
- **Tune the controller and camera.** Feel is worth a whole buffer day and this is the best one you'll get before the animation work lands on top of it.
- **Greybox more space** — a village square, a doorway, an interior. M11's NPCs need somewhere to stand.
- **Fix the Core sharing setup** if the submodule is annoying you. Better to sort it now than at Day 100.
- **Rest.** You shipped a game seven days ago and started a new project. That's a lot of week.

### Milestone review

Run `/review`. Ask two specific questions: **has anything new leaked into Core this week** — a `Vector3`, a position, a "3D" anything — and **does any of the new view code duplicate logic Core already has?** The second is the sneakier failure: an interaction script that decides whether a conversation is available, when `DialogueRunner` already knows.

### Where you are

Seven days ago you had a shipped 2D game and no 3D experience at all. Right now you have a third-person character with a camera that doesn't make people ill, walking around a greybox and holding a branching conversation driven by an assembly you have not opened since October.

This was the week the curriculum warned you about — the one where everything you'd got good at stopped applying and quitting looked reasonable. It's behind you, and the reason it only took a week instead of a month is sitting in `Hearthfall.Core`, exactly where you left it. That week in October that changed nothing on screen just bought you the entire back half of this project.

Tomorrow the capsule stops being a capsule. Mixamo, humanoid rigs, the Animator, and blend trees — the week where it starts looking like a game instead of a physics demo.

**Commit:** `docs: M10 complete — into 3d, same core`
