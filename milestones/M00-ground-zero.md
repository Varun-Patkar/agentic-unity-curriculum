# M00 · Ground Zero — The Editor

**Days 1–7 · 10–16 Sep 2026 · project: `D:\Projects\Unity Games\Sandbox00`**

> **This milestone is throwaway and that is the point.** You are not building anything you keep. You are making the editor stop being a wall of unfamiliar panels. Six days from now you will open Unity and know where things are, which is the single thing that separates people who start from people who continue.
>
> Delete `Sandbox00` on Day 15 without a second thought.

**You end holding:** a physics toy you built and exported as a real `.exe`, and an editor that no longer intimidates you.

---

## Day 1 — Install audit, first project, and the six windows
**Thu 10 Sep · 60 min**

**Objective:** Unity opens a project you created, you can name all six main windows, and you have made a cube move by dragging it.

**Why:** Every previous attempt at this died in setup. Today setup ends, permanently, in under twenty minutes.

### Concepts (10 min)
- **Unity Hub vs Unity Editor** — the Hub is a launcher and version manager. Multiple editor versions coexist; a project is pinned to one.
- **What a "project" is on disk**: `Assets/` (yours, committed), `Packages/` (a manifest, committed), `ProjectSettings/` (committed), `Library/` (a build cache — never committed, always safe to delete).
- **Templates** — Unity 6 offers Universal 2D, Universal 3D, HDRP, and others. "Universal" means URP. You'll use URP for everything in this curriculum.

### Build (40 min)
1. **Audit the install.** Open Unity Hub. Note the exact editor version — write it into `progress/STATE.md` under `unity_version`. Confirm the **Windows Build Support** module is installed (Hub → Installs → the gear icon → Add Modules). You need this on Day 6; finding out then is worse.
2. **Create `Sandbox00`** at `D:\Projects\Unity Games\Sandbox00` using the **Universal 3D** template. It will take a few minutes to open. That's normal, it always will be.
3. **Tour the six windows.** Find each one, and say out loud what it does: Hierarchy · Scene · Game · Inspector · Project · Console. Read `reference/unity-editor-map.md` alongside this — it's short.
4. **Set the Play Mode tint now.** `Edit > Preferences > Colors > Playmode tint`. Pick something obnoxious. This prevents a specific, guaranteed, infuriating loss of work later.
5. **Make a cube.** `GameObject > 3D Object > Cube`. Move it with the toolbar gizmos (W/E/R = move/rotate/scale). Watch the Transform values change in the Inspector as you drag.
6. **Navigate the Scene view** until it's automatic: right-drag to look, WASD to fly while right-dragging, scroll to zoom, **F to frame the selected object**. F is the one you'll use a thousand times.
7. **Press Play.** Nothing happens. Notice the tint. Move the cube while playing, press Stop, and watch your change vanish. Now you've seen it.
8. **Save the scene** (`Ctrl+S`) into `Assets/Scenes/`.

### Acceptance criteria
- [ ] `Sandbox00` opens from Unity Hub without errors in the Console
- [ ] You can name all six windows and what each is for, without looking
- [ ] You can frame, orbit, and fly to any object in the Scene view without thinking
- [ ] Play Mode tint is set and you have personally watched a Play Mode change get discarded
- [ ] Windows Build Support is confirmed installed
- [ ] `progress/STATE.md` records the exact Unity version and project path

### Failure modes
- **Project takes 10 minutes to open** → normal on first open, it's compiling shaders and building the Library cache. Not a problem.
- **Console has warnings on a fresh project** → normal. Warnings are not errors. Only red matters.
- **Can't find a menu path** → Unity 6 moved several. Tell your agent; have it verify against current docs rather than guess.

**Stretch:** Add a Directional Light and a Plane, and drop the cube onto it from height (you'll need a Rigidbody — that's Day 5, but poke at it).

**Commit:** *(no code yet — commit the curriculum repo)* `docs: day 1, unity version and project path recorded`

---

## Day 2 — GameObjects, Components, Transforms
**Fri 11 Sep · 60 min**

**Objective:** Build a small scene by composition, and be able to explain why Unity has no `Character` base class.

**Why:** This is Unity's central design idea. Getting it on day 2 stops you writing the deep inheritance hierarchy that every backend developer writes on day 20 and deletes on day 40.

### Concepts (10 min)
- **A GameObject has no behaviour.** It's an ID, a name, a Transform, and a list of components. All behaviour is components.
- **This is composition over inheritance, enforced by the engine.** Where you'd reach for `class Enemy : Character : Entity`, Unity wants `GameObject + Rigidbody + Collider + Health + EnemyAI`.
- **The Transform hierarchy is not the scene graph you might expect** — parenting affects position/rotation/scale and lifetime, nothing else. There's no inheritance of behaviour down the tree.
- **Local vs world space.** A child's Transform values are relative to its parent. This trips everyone once.

### Build (40 min)
1. Create an empty GameObject (`GameObject > Create Empty`), name it `Cart`. **Reset its Transform** (Inspector → the three-dot menu on Transform → Reset). Empties at odd positions cause hours of confusion.
2. Build a cart out of primitives parented under it: a stretched cube for the bed, two cylinders rotated for wheels. Rotate and scale them into place.
3. **Move and rotate the `Cart` parent.** Watch the children follow. Note that the children's *local* Transform values don't change.
4. Add components to the cart bed via **Add Component**: a `Rigidbody`, then remove it. Add a `Light`, look at what appears in the Inspector, remove it. You're learning that components are just data+behaviour you bolt on.
5. **Tags and Layers.** `Project Settings > Tags and Layers`. Add a tag `Interactable` and a layer `Ground`. Assign them to something. Understand these are two unrelated systems.
6. **Make a second cart** by copy-paste. Notice you now have two independent copies and changing one doesn't change the other. That problem is what prefabs solve — tomorrow's-ish.
7. Organise: put things in `Assets/Scenes/`, name every GameObject properly. `GameObject (17)` is how scenes become unmanageable.

### Acceptance criteria
- [ ] A parented multi-part object exists, and moving the parent moves the whole thing
- [ ] You can explain the difference between a GameObject and a Component in one sentence each
- [ ] You have added and removed at least three different component types
- [ ] You know the difference between a Tag, a Layer, and a Sorting Layer
- [ ] Nothing in the Hierarchy is named `GameObject`

### Failure modes
- **Child object flies off when parented** → the parent's Transform wasn't reset, or had a non-uniform scale. Non-uniform scale on a parent distorts children and is a genuine source of pain later.
- **Rotating a cylinder makes it look wrong** → you rotated on the wrong axis. Use the gizmo, not typed values, until it's intuitive.

**Stretch:** Look at the scene file in a text editor. It's YAML. Every GameObject and component you made is in there, referenced by GUID. Understanding that scenes are just serialized data demystifies a lot.

**Commit:** `docs: day 2 log`

---

## Day 3 — Your first script: the MonoBehaviour lifecycle
**Sat 12 Sep · 60 min**

**Objective:** A script you wrote makes an object move, rotate, and respond to a key — and you know exactly which callback runs when.

**Why:** You know C#. What you don't know is *when Unity calls your code*, and that's the entire difference. Today converts your existing fluency into Unity fluency.

### Concepts (10 min)
- **`Update()` is not `main()`.** It's a callback Unity invokes ~60 times a second, by reflection, if a method with that exact name exists. Misspell it and it silently never runs.
- **The lifecycle order** — `Awake` → `OnEnable` → `Start` → `Update`×n → `OnDisable` → `OnDestroy`. **The rule: `Awake` for yourself, `Start` for everyone else.**
- **`Time.deltaTime`.** Anything per-frame must be multiplied by it. Without it, your game runs at a different speed on a 144Hz monitor. This bug is invisible on your machine and it is in almost everyone's first project.
- **`[SerializeField] private`** — the idiom. Editable in the Inspector, still encapsulated. `public` fields work too and leak your API to the whole project.

Read `reference/csharp-for-unity.md` sections "The MonoBehaviour lifecycle" and "Time" — they're short and they're today's material.

### Build (40 min)
1. Create `Assets/Scripts/Spinner.cs` (right-click in Project → Create → **MonoBehaviour Script**). **The class name must match the filename.**
2. Write `Spinner`: a `[SerializeField] private float _degreesPerSecond = 90f;` and an `Update` that rotates the transform. You type this. `transform.Rotate(...)` is the call you want.
3. Attach it to a cube. Press Play. Adjust `_degreesPerSecond` **while playing** to find a speed you like. Note the value. Stop. Set it properly.
4. **Prove the lifecycle to yourself.** Add `Debug.Log` to `Awake`, `Start`, `OnEnable`, and the first frame of `Update`. Put the script on two objects. Read the Console and confirm the ordering.
5. **Prove `Time.deltaTime` matters.** Write a second rotation that *doesn't* use it. Then change the frame rate: `Application.targetFrameRate = 10;` in `Start`. Watch the two objects diverge. This is the lesson.
6. Add a key check in `Update` to toggle spinning. Use the legacy `Input.GetKeyDown(KeyCode.Space)` **today only** — you'll replace it with the Input System on Day 9 and you should feel why.
7. Deliberately misspell `Update` as `Updat`. Observe that Unity says nothing at all. Fix it. Remember this feeling; it's a whole class of bug.

### Acceptance criteria
- [ ] A cube spins at a speed you set in the Inspector
- [ ] Console output proves you understand `Awake`/`OnEnable`/`Start`/`Update` ordering
- [ ] You have seen frame-rate dependence with your own eyes
- [ ] Space toggles the spin
- [ ] You have experienced a silently-not-running misspelled callback

### Failure modes
- **"The script can't be added because it doesn't derive from MonoBehaviour"** → class name doesn't match filename, or there's a compile error blocking everything. Check the Console.
- **`_degreesPerSecond` isn't in the Inspector** → missing `[SerializeField]`, or Unity hasn't recompiled. Look bottom-right for the spinner.
- **Changes to the Inspector value don't persist** → you set them in Play Mode.

**Stretch:** Add a `Bobber` component that moves the object up and down with `Mathf.Sin(Time.time * frequency) * amplitude`. Put both components on one cube. That's composition, working.

**Commit:** `docs: day 3 log`

---

## Day 4 — Prefabs, instantiation, and the Project window
**Sun 13 Sep · 60 min**

**Objective:** Spawn fifty objects at runtime from one template, change the template, and watch all fifty change.

**Why:** Prefabs are how every creature, every prop, and every NPC in Hollowbrook will exist. They're also the thing that fixes the copy-paste problem you hit on Day 2.

### Concepts (10 min)
- **A prefab is a serialized GameObject template stored as an asset.** A prototype, in the pattern sense.
- **Prefab asset vs prefab instance.** Edits to the asset propagate to all instances. Edits to an instance become *overrides* on that instance only, shown in bold in the Inspector. This distinction causes more confusion than any other single Unity concept.
- **Prefab Mode** — double-click a prefab asset to edit it in isolation.
- **`Instantiate(prefab, position, rotation)`** — spawn a copy at runtime. `Destroy(go)` removes it (at the end of the frame, not immediately).
- **`.meta` files.** Every asset has one, holding the GUID that all references point at. Delete a `.meta` and every reference to that asset breaks. Commit them.

### Build (40 min)
1. Build a small object worth spawning — a cube with your `Spinner` on it and a coloured material (`Create > Material`, set Base Map colour, drag it on).
2. **Drag it from the Hierarchy into the Project window.** That's it — that's a prefab. Delete the scene copy.
3. Drag the prefab back into the scene three times. Open the prefab asset (double-click), change its colour or spin speed, exit Prefab Mode. **All three instances change.**
4. Now change one *instance*'s spin speed in the Inspector. Note the bold field and the left-margin marker — that's an override. Right-click the field → Revert. Understand this deeply; it will confuse you again in November otherwise.
5. Write `Spawner.cs`: a `[SerializeField] private GameObject _prefab;`, a count, and a `Start` that `Instantiate`s them at random positions in a radius. Assign the prefab in the Inspector.
6. Press Play. Fifty spinning cubes. Now edit the prefab asset while it's playing and watch them all respond.
7. **Organise the Project window.** Make `Assets/_Project/{Prefabs,Materials,Scripts,Scenes}`. The underscore keeps your folders sorted above imported packages. Move things. Note that Unity fixes all the references automatically — that's the `.meta` GUIDs working.

### Acceptance criteria
- [ ] A prefab asset exists and multiple instances are in the scene
- [ ] Editing the asset changes every instance
- [ ] You can create, spot, and revert an instance override
- [ ] `Spawner` creates N instances at runtime from an Inspector-assigned prefab
- [ ] Project folders are organised and nothing broke when you moved files

### Failure modes
- **`NullReferenceException` in `Spawner`** → the `_prefab` field is `None` in the Inspector. Look at it. This will be your most common error for the next month.
- **Edits to the prefab don't affect instances** → you're editing an instance, not the asset. Check the header of the Inspector.
- **Objects spawn on top of each other** → you passed the same position each time.

**Stretch:** Spawn on a keypress instead of in `Start`, and `Destroy` the oldest when you exceed a cap. That's object pooling's problem statement, which you'll meet properly much later.

**Commit:** `docs: day 4 log`

---

## Day 5 — Physics: Rigidbody, colliders, and the FixedUpdate rule
**Mon 14 Sep · 60 min** *(leave day — 2 hours available if you want them)*

**Objective:** A tower of boxes that collapses convincingly, a ball you can launch into it, and a trigger zone that logs when something enters.

**Why:** Physics is where Unity feels like a game engine for the first time. It's also where the largest cluster of beginner bugs lives, and you're going to meet all of them today deliberately rather than at 11pm in November.

### Concepts (10 min)
- **Rigidbody = "the physics engine owns this object's movement."** Once you add one, writing `transform.position` fights the simulation and breaks collision detection.
- **Collider = the shape used for collisions.** Independent of the visual mesh. A complex mesh with a simple box collider is normal and correct.
- **Collision needs a Rigidbody on at least one of the two objects.** Two static colliders do nothing. This is the #1 "why doesn't my collision work".
- **`FixedUpdate` runs at a fixed timestep** (default 50Hz), decoupled from frame rate. **All forces, velocities, and Rigidbody movement go here.** Physics in `Update` produces jitter and inconsistency.
- **Triggers** (`Is Trigger` checked) detect overlap without blocking. They fire `OnTriggerEnter`, not `OnCollisionEnter`. Mixing these up is guaranteed at least once.
- **Kinematic** Rigidbodies are moved by you, not by forces, but still participate in collision detection.

### Build (40 min)
1. Ground plane. Stack a tower of ~15 cubes, each with a Rigidbody and a Box Collider. Press Play — it should settle, not explode. If it explodes, they're intersecting.
2. Make a sphere with a Rigidbody. Write `Launcher.cs` that applies `AddForce` on a keypress — **in `FixedUpdate`**, gated by a flag set in `Update`. (Input is read in `Update`; physics is applied in `FixedUpdate`. Get this pattern into your hands now; you'll use it for every attack in M07 and M12.)
3. Knock the tower down. Spend a minute enjoying it — this is the first time it feels like an engine.
4. **Deliberately break it:** move the sphere with `transform.position += ...` instead of physics. Watch it tunnel straight through the tower. Revert. Now you know what that symptom means.
5. **Trigger zone:** an empty GameObject with a Box Collider, `Is Trigger` checked. Add a script with `OnTriggerEnter(Collider other)` that logs `other.name`. Roll the ball through it.
6. **Deliberately break that too:** uncheck `Is Trigger` and watch `OnTriggerEnter` stop firing while `OnCollisionEnter` starts. Both symptoms are now familiar.
7. Physics Materials: create one with low friction and high bounciness, apply it to the sphere. Play with the numbers in Play Mode.

### Acceptance criteria
- [ ] A tower collapses when struck and doesn't explode at rest
- [ ] The launch force is applied in `FixedUpdate`, with input read in `Update`
- [ ] A trigger zone logs entry by name
- [ ] You have personally seen: tunnelling from transform-movement, and a trigger that stops firing when `Is Trigger` is unchecked
- [ ] You can state the rule "collision needs a Rigidbody on at least one object" from memory

### Failure modes
- **Tower explodes on Play** → colliders overlapping at start. Space them by a hair.
- **Ball goes through the floor at speed** → set Collision Detection to Continuous on the Rigidbody.
- **`OnTriggerEnter` never fires** → no Rigidbody on either object · `Is Trigger` unchecked · layers not set to collide · method signature is wrong (`Collider` in 3D, `Collider2D` in 2D).
- **Movement is jittery** → physics in `Update`.

**Stretch (2hr day):** Add a "reset tower" key that destroys everything and re-spawns it from a prefab. That combines Day 4 and Day 5, and it's genuinely the first thing that resembles a game system.

**Commit:** `docs: day 5 log`

---

## Day 6 — Build the toy, then export a real executable
**Tue 15 Sep · 60 min** *(leave day — 2 hours available if you want them)*

**Objective:** A tiny complete toy — aim, launch, knock over the tower, see a score, press R to reset — exported as a `.exe` that runs when Unity is closed.

**Why:** Building is where a lot of people discover their project doesn't actually work. Doing it on Day 6, on something disposable, means the first time you meet build problems it costs you nothing.

### Concepts (10 min)
- **A build is a different program.** Different frame rate, no editor-only code, no `#if UNITY_EDITOR` paths, real file system. Things break here that never broke in the editor.
- **`File > Build Profiles`** (Unity 6; older versions call it Build Settings) — controls which scenes are included and in what order. **A scene not in the list does not exist in the build.**
- **Player Settings** — company name, product name, resolution, icon. Where a project starts feeling like a product.
- The build output is a folder, not a single file. The `.exe` needs its `_Data` folder beside it.

### Build (40 min)
1. Tidy the toy into something with a beginning and an end: aim with the mouse, launch on click, count fallen boxes as score, R to reset.
2. Score with a `Debug.Log` for now — UI is Day 11 and is not today's problem.
3. Set Player Settings: company name, product name `Sandbox Toy`, and a default windowed resolution of 1280×720. Windowed matters — a fullscreen build with a bug is genuinely annoying to escape.
4. `File > Build Profiles` → Windows → **add the open scene to the list** → Build. Output to `D:\Projects\Unity Games\Sandbox00\Builds\v1\`. **Never build into `Assets/`.**
5. **Close Unity entirely.** Run the `.exe`. Play your game.
6. Note anything that behaves differently from the editor. Anything at all. That difference is the lesson of the day.
7. Zip the build folder. That's a distributable game, and you made it on day six.

### Acceptance criteria
- [ ] The toy has a clear interaction loop and a reset
- [ ] A Windows build exists in `Builds/v1/`
- [ ] **The `.exe` runs with Unity closed**
- [ ] Product name and resolution are set deliberately
- [ ] You noted at least one editor-vs-build difference (or confirmed there were none)

### Failure modes
- **Build fails: "no scenes"** → you didn't add the scene in Build Profiles.
- **Build fails with compile errors** → the editor tolerates some things builds don't. Read the first error.
- **Black screen on launch** → no camera in the scene, or the wrong scene is index 0.
- **Build is enormous** → normal. A minimal Unity build is tens of MB.
- **Windows SmartScreen warns on the exe** → expected for unsigned executables. Not a problem.

**Stretch (2hr day):** Add a second scene (a title screen) and `SceneManager.LoadScene` between them. Rebuild. You now understand the scene list.

**Commit:** `docs: day 6 log — first build shipped`

---

## Day 7 — BUFFER
**Wed 16 Sep**

First buffer day. Pick one, genuinely:

- **Catch up** — anything from Days 1–6 that's unfinished or broken.
- **Polish** — make the toy feel better. Add a trail to the ball, a camera shake on impact, a sound. Feel is a skill and it starts now.
- **Explore** — an hour of aimless poking. Open the Package Manager and look at what's available. Try a ProBuilder shape. Make a particle system. This is how the editor becomes yours.
- **Rest** — close the laptop. Log it. Streak intact.

**Before you move on**, be honest about one thing: *can you open Unity and build a small scene without feeling lost?* If not, spend today on that, and take the extra day. M01 is much more fun when the editor isn't fighting you.

**Milestone review:** run `/review` if you want your Day 6 toy critiqued. It's throwaway code, so it's a free place to hear hard things.

**Commit:** `docs: M00 complete — editor fundamentals`
