# M01 · First Blood — A Complete Tiny Game

**Days 8–14 · 17–23 Sep 2026 · project: `D:\Projects\Unity Games\ChickenChase`**

> **Chicken Chase.** You are a peasant. Chickens have escaped. You have 60 seconds. That is the entire design and you are not allowed to expand it.
>
> This is still throwaway, but it is throwaway that **finishes**. Most people who "learn Unity" never once take something from empty project to zipped build with a menu and a win state. After this week you'll have done it, and every future project inherits that confidence.

**You end holding:** a complete, built, playable game with a menu, sound, and a score. Small and silly and *done*.

**The one rule this week:** when you think of a feature, write it on a list and do not build it. The list is the deliverable, not the features.

---

## Day 8 — 2D URP project, sprites, and the 2D pipeline
**Thu 17 Sep · 60 min**

**Objective:** A 2D scene with a peasant sprite and a few chickens, correctly sized and correctly sorted.

**Why:** 2D in Unity is 3D with an orthographic camera and a lot of conventions. The conventions are what bite, and they all bite on the first day.

### Concepts (10 min)
- **2D is 3D with the lights off.** Orthographic camera, sprites on quads, Z used for sorting rather than depth.
- **Pixels Per Unit (PPU)** — maps sprite pixels to world units. **Pick one number for the whole project.** Mismatched PPU is why your sprites are inexplicably different sizes.
- **Sorting Layer + Order in Layer** decide 2D draw order. Not Z position, not the Hierarchy. This is a separate system from Layers and from Tags.
- **Filter Mode**: Point for pixel art, Bilinear for painted art. Getting this backwards makes pixel art blurry or painted art crunchy.
- **Pivot** — the sprite's origin. For a character, bottom-centre is usually right, so "position" means "where their feet are".

### Build (40 min)
1. New project, **Universal 2D** template, at `D:\Projects\Unity Games\ChickenChase`. `git init` in it immediately, with a Unity `.gitignore` (`Library/`, `Temp/`, `Logs/`, `Builds/`, `obj/`, `UserSettings/` — **not** `.meta` files).
2. Get sprites. Either grab a CC0 pack (Kenney) or make three quick ones yourself — a peasant, a chicken, a grass tile. 32×32 is plenty. See `reference/asset-pipeline.md`.
3. Import them. Set **Texture Type: Sprite (2D and UI)**, PPU **32**, Filter Mode to match your art style, Compression **None**.
4. Place a peasant and three chickens in the scene. Set the Camera's orthographic **Size** so the play area fills the Game view — size is *half the vertical height in world units*, which is the thing nobody tells you.
5. **Sorting.** Add Sorting Layers (`Project Settings > Tags and Layers`): `Background`, `Ground`, `Entities`, `UI`. Assign them. Deliberately put the peasant behind the grass, see the symptom, fix it.
6. Add a Box Collider 2D to the peasant and each chicken. **Note the `2D` suffix** — 2D and 3D physics are entirely separate systems that cannot interact.
7. Set the background colour on the camera to something that isn't Unity blue.

### Acceptance criteria
- [ ] 2D URP project created, git initialised with a correct `.gitignore` (`.meta` files tracked)
- [ ] Sprites imported with deliberate PPU, filter, and compression settings
- [ ] Camera framing looks intentional
- [ ] Sorting Layers exist and the peasant renders in front of the ground
- [ ] Every entity has a `Collider2D`

### Failure modes
- **Sprites are enormous or microscopic** → PPU mismatch. Fix on the import settings, never with the Transform scale.
- **Pixel art is blurry** → Filter Mode is Bilinear.
- **Sprite disappears behind another** → sorting layer / order in layer.
- **Everything is dark** → you used the 3D template. 2D URP handles lighting differently; check you're on Universal 2D.

**Stretch:** Add a 2D Light (`GameObject > Light > 2D`) and a Global Light. URP 2D lighting is genuinely lovely and takes five minutes.

**Commit:** `feat: 2d project setup with sprites and sorting layers`

---

## Day 9 — The Input System, properly
**Fri 18 Sep · 60 min**

**Objective:** The peasant moves with WASD *and* a gamepad, from one Input Actions asset, with zero `Input.GetKey` anywhere.

**Why:** You'll use the same setup in Hollowbrook for the next 100 days. Learning the modern system now means never having to unlearn the old one.

### Concepts (10 min)
- **Why the new system exists:** the old `Input` class hardcodes devices, can't rebind at runtime, and can't handle multiple players. The new one separates *what the player is trying to do* from *what button they pressed*.
- **Action Asset → Action Map → Action → Binding.** An Action is `Move` or `Attack`. A Binding is `WASD` or `left stick`. The map groups them by context (`Gameplay`, `UI`, `Dialogue`) — and you'll use those contexts heavily in M04.
- **Action types**: `Value` (continuous, like movement), `Button` (discrete press), `Pass Through`.
- **Two ways to read it**: the `PlayerInput` component with callbacks (fast to set up), or a generated C# class (more control). Use `PlayerInput` today.

### Build (40 min)
1. Confirm the **Input System** package is installed (`Window > Package Manager`). Set `Project Settings > Player > Active Input Handling` to **Input System Package (New)** — Unity will ask to restart. Do it.
2. Create an **Input Actions** asset: `Assets/_Project/Input/PlayerControls.inputactions`.
3. Add a `Gameplay` map with a `Move` action (Type: Value, Control Type: Vector2). Add a **2D Vector composite** binding for WASD, and a second binding for the gamepad left stick.
4. Tick **Generate C# Class** on the asset, or add a `PlayerInput` component to the peasant referencing the asset — pick one and be consistent.
5. Write `PlayerMovement.cs`. It reads the `Move` vector and moves a `Rigidbody2D`. **Read input in `Update`, apply movement in `FixedUpdate`** — the same pattern as Day 5.
6. Set the peasant's `Rigidbody2D` to **Gravity Scale 0** (top-down) and **Freeze Rotation Z**. Both of these are non-obvious and both are required.
7. Tune the speed in Play Mode until it feels right. Write the number down.
8. If you have a gamepad, test it. It should just work, with no code changes. That's the whole point of the system.

### Acceptance criteria
- [ ] Peasant moves on WASD
- [ ] The same code works on a gamepad with no changes (or you understand why it would)
- [ ] Input is read in `Update`, movement applied in `FixedUpdate`
- [ ] No `Input.GetKey` / `Input.GetAxis` anywhere in the project
- [ ] Diagonal movement isn't faster than straight movement (normalise it)

### Failure modes
- **Nothing happens** → Active Input Handling still on Old · action map not enabled · `PlayerInput` not referencing the asset · the callback method name doesn't match.
- **Peasant falls off screen** → Gravity Scale isn't 0.
- **Peasant spins on collision** → Freeze Rotation Z.
- **Diagonal is ~1.41× faster** → normalise the input vector.
- **Movement is jittery** → moving in `Update`, or using `transform.position` on a Rigidbody.

**Stretch:** Add acceleration and friction instead of instant velocity. Compare. Instant feels arcade-y and snappy; accelerated feels weighty. Note which you prefer — you'll make this exact decision again on Day 17.

**Commit:** `feat: input system driven player movement`

---

## Day 10 — The loop: spawn, collect, score
**Sat 19 Sep · 60 min**

**Objective:** Chickens spawn at random positions, you collect them by touching them, and the score goes up.

**Why:** This is the smallest complete gameplay loop that exists. Everything else is decoration.

### Concepts (10 min)
- **Triggers for pickups.** Overlap, don't block. `OnTriggerEnter2D`.
- **Who owns the score?** Not the player, not the chicken. A `GameManager`. Today a simple singleton is fine — and by M03 you'll know exactly why it's the wrong long-term answer.
- **`Destroy()` happens at the end of the frame**, not immediately. Guard against double-collection.
- **Events over polling.** The chicken raises "I was collected"; the manager listens. Better than the chicken reaching into the manager, and it's the pattern the entire back half of the curriculum is built on.

### Build (40 min)
1. Make the chicken a **prefab**.
2. `ChickenSpawner.cs`: spawn N chickens at random points inside a defined rectangle, avoiding the player's start position.
3. Chicken collider → **Is Trigger** ✓. `Chicken.cs` handles `OnTriggerEnter2D`, checks the tag, raises a static `event Action OnCollected`, and destroys itself.
4. `GameManager.cs`: subscribes in `OnEnable`, unsubscribes in `OnDisable` (**always pair these** — this is the leak that bites everyone), increments score, `Debug.Log`s it.
5. Add a 60-second timer in the manager. When it hits zero, log "game over" and stop the player.
6. Spawn a new chicken each time one is collected, so it doesn't run dry.
7. Play it for two minutes. Is it fun? Probably mildly. Note the *one* change that would help most. Don't build it yet.

### Acceptance criteria
- [ ] Chickens spawn at varied positions each run
- [ ] Touching one collects it and increments the score
- [ ] Score is logged and the count is correct (no double-counting)
- [ ] A 60-second timer ends the game
- [ ] Event subscriptions are paired `OnEnable`/`OnDisable`

### Failure modes
- **Trigger doesn't fire** → no `Rigidbody2D` on either object · `Is Trigger` unchecked · wrong signature (`Collider2D`, not `Collider`).
- **Score jumps by 2** → collected twice before `Destroy` took effect. Add a `_collected` guard flag.
- **Chickens spawn off-screen** → your spawn rectangle doesn't match the camera's orthographic bounds.
- **`MissingReferenceException` after a chicken is destroyed** → an event handler on a dead object. That's what `OnDisable` unsubscription is for.

**Stretch:** Make chickens flee from the player. Twenty lines, and it triples how fun the game is — a useful lesson about where value actually comes from.

**Commit:** `feat: collect loop with score and timer`

---

## Day 11 — UI: score, timer, game over
**Sun 20 Sep · 60 min**

**Objective:** On-screen score and countdown, plus a game-over panel with a restart button, that all scale correctly when you resize the window.

**Why:** UI is where a prototype becomes a game. It's also where Unity is at its most fiddly, so meeting it on something disposable is a gift to yourself.

### Concepts (10 min)
- **Canvas** — the root of all UI. Render modes: **Screen Space – Overlay** (99% of cases), Screen Space – Camera, World Space (for floating damage numbers later).
- **Canvas Scaler** — set to **Scale With Screen Size**, reference resolution 1920×1080. Without this, your UI is the wrong size on every monitor but yours. This is the setting everyone forgets exactly once.
- **RectTransform** — not a Transform. Anchors and pivots. **The anchor is what confuses everybody**: it defines what the element is positioned *relative to* as the parent resizes.
- **TextMeshPro**, not the legacy Text. Unity will offer to import TMP essentials — accept.
- **uGUI vs UI Toolkit** — you're using uGUI (the GameObject-based one). UI Toolkit is newer and more like the web; you'll evaluate it on Day 100.

### Build (40 min)
1. `GameObject > UI > Canvas`. Set the Canvas Scaler to Scale With Screen Size, 1920×1080, Match 0.5.
2. Add TMP text for score (top-left) and timer (top-centre). **Anchor them to those corners**, then resize the Game view and confirm they stay put. If they drift, your anchors are wrong.
3. `UIController.cs` subscribes to the manager's score/time changes and updates the text. **Only update text when the value changes**, not every frame — string allocation per frame is a real cost and a good habit to form now.
4. Build a Game Over panel: a semi-transparent background image, a final-score text, and a Restart button. Disable it at start.
5. Wire the button to `SceneManager.LoadScene(SceneManager.GetActiveScene().name)`. Crude, effective, and fine for today.
6. Show the panel when the timer hits zero, and stop player input.
7. Resize the Game view to several aspect ratios (16:9, 4:3, ultrawide). Fix anything that breaks.

### Acceptance criteria
- [ ] Score and timer display and update correctly
- [ ] UI stays anchored at every aspect ratio you test
- [ ] Game over panel appears at zero and restarts on click
- [ ] Text updates only on change, not per frame
- [ ] Player can't move after game over

### Failure modes
- **UI is enormous / invisible** → Canvas Scaler set to Constant Pixel Size.
- **UI drifts when resizing** → anchors are in the middle rather than on the corner.
- **Button does nothing** → no EventSystem in the scene (Unity usually adds one with the Canvas, but not always) · the button is behind another element · Raycast Target is off.
- **Text shows as blocks** → TMP essentials weren't imported.

**Stretch:** Animate the score text — a quick scale punch on increment. Two lines with a coroutine, and it makes collection feel dramatically better. That's your first taste of juice.

**Commit:** `feat: score/timer hud and game over screen`

---

## Day 12 — Juice: sound, particles, screenshake
**Mon 21 Sep · 60 min**

**Objective:** Collecting a chicken *feels good*. Same mechanics as yesterday, dramatically better game.

**Why:** This is the highest value-per-minute day in the first two weeks. Game feel is not polish applied at the end — it's the actual product, and you need to believe that viscerally rather than intellectually.

### Concepts (10 min)
- **Feedback on every action.** Every input the player makes should produce visible, audible, or kinetic response within a frame or two. Silence reads as "broken".
- **The juice stack**: sound · particles · scale punch · screenshake · hitstop · colour flash. Each is small. Together they're transformative.
- **`AudioSource` vs `AudioListener`** — sources emit, one listener (on the camera) receives. `PlayOneShot` for overlapping SFX.
- **Pitch variation.** Randomising pitch ±10% on a repeated sound is the difference between "a game" and "a machine gun of the same wav file". It costs one line.
- **Restraint.** Juice is seasoning. Twenty simultaneous effects is noise.

### Build (40 min)
1. Get 3–4 sounds — collect, timer-tick, game-over, footstep. Freesound, Kenney, or jsfxr (see `reference/asset-pipeline.md`).
2. `AudioManager.cs` with a `PlayOneShot(clip, pitchVariation)` helper. Randomise pitch on collect.
3. Particle System burst on collect — a few feathers or a puff. Make it a prefab, instantiate at the chicken's position, and have it self-destruct.
4. **Screenshake.** A coroutine on the camera that offsets its position by decaying random amounts over ~0.15s. Keep it subtle — you'll want it 3× bigger than it should be, so build it, then halve it.
5. Scale punch on the score text (or reuse yesterday's stretch).
6. Chicken squash-and-stretch before it disappears: scale up briefly, then down to zero, then destroy.
7. **Now play it back to back with yesterday's build.** Same mechanics. Notice how much better it is. Sit with that — it's the most useful thing you'll learn this month.

### Acceptance criteria
- [ ] Collecting plays a pitch-varied sound
- [ ] A particle burst fires at the collection point and cleans itself up
- [ ] Screenshake fires and decays, and it's subtle
- [ ] At least two elements animate on collection
- [ ] Playing it is noticeably more satisfying than yesterday

### Failure modes
- **No sound** → no AudioListener (it lives on the Main Camera) · volume 0 · clip not assigned · Play On Awake fighting you.
- **Screenshake never stops / drifts** → you're accumulating offsets instead of offsetting from a cached base position.
- **Particles persist forever** → Stop Action not set to Destroy, and no cleanup.
- **It feels like too much** → it is. Halve everything. Twice.

**Stretch:** Add **hitstop** — freeze `Time.timeScale` to 0 for ~0.05s on collection, then restore. Barely perceptible consciously; enormous felt impact. This is the single most-used trick in action games and you'll use it again on Day 90.

**Commit:** `feat: audio, particles, and screenshake`

---

## Day 13 — Menus, scene flow, and a real build
**Tue 22 Sep · 60 min**

**Objective:** Title screen → game → game over → back to title. Built, zipped, and runnable on someone else's machine.

**Why:** Scene management and a working build are the last two things between "a project" and "a game". After today you've shipped one.

### Concepts (10 min)
- **`SceneManager.LoadScene`** — by name (readable) or index (fragile). Use names, and add every scene to Build Profiles.
- **Scenes are torn down completely on load.** Everything not marked `DontDestroyOnLoad` dies. Persistent managers need explicit handling — and singleton-across-scenes is genuinely awkward, which you'll solve properly on Day 59.
- **Additive loading** exists (`LoadSceneMode.Additive`) for keeping UI or managers loaded. Useful later, overkill today.
- **`Time.timeScale`** — 0 pauses everything physics- and time-based. Beware: it also freezes any coroutine using `WaitForSeconds` (use `WaitForSecondsRealtime` for UI animations during a pause).

### Build (40 min)
1. New scene `MainMenu`. Title text, Play button, Quit button. Give it a background and 60 seconds of visual care — first impressions are cheap here.
2. `MainMenuController.cs`: Play loads `Game`; Quit calls `Application.Quit()` (which does nothing in the editor — that's expected, log alongside it).
3. Add both scenes to `File > Build Profiles`, with `MainMenu` first.
4. Game over panel gets a "Main Menu" button alongside Restart.
5. Add a pause: Escape toggles `Time.timeScale` between 0 and 1 and shows a panel. Make sure the timer respects it.
6. Player Settings: product name `Chicken Chase`, company name, an icon if you have one, default windowed 1280×720.
7. **Build.** Close Unity. Play the whole flow: menu → game → pause → game over → menu → quit.
8. Zip it. Put it somewhere you'll still find it in December.

### Acceptance criteria
- [ ] Full flow works in the **build**, not just the editor: menu → game → pause → game over → menu → quit
- [ ] Both scenes in Build Profiles, `MainMenu` at index 0
- [ ] Pause actually pauses (timer stops, player stops)
- [ ] Build runs on a machine without Unity installed
- [ ] It's zipped and stored

### Failure modes
- **"Scene couldn't be loaded"** → not in Build Profiles, or the name is misspelled.
- **Quit does nothing** → you're in the editor. Correct behaviour.
- **Pause menu is frozen** → your UI animation uses `WaitForSeconds` and `timeScale` is 0. Use `WaitForSecondsRealtime`.
- **Build has a black screen** → wrong scene at index 0, or no camera in it.

**Stretch:** Persist the high score with `PlayerPrefs`. Crude, dreadful for real save data, perfect for this. You'll do it properly on Day 57.

**Commit:** `feat: menus, scene flow, and v1 build`

---

## Day 14 — BUFFER · and give it to one human
**Wed 23 Sep**

Options:

- **Catch up** on anything unfinished.
- **Ship it.** Send the zip to exactly one person and *watch them play it* — in person or on a call. Do not explain the controls. Do not defend anything. Just watch and take notes. **You will learn more in five minutes of this than in five hours of solo playtesting**, and the discomfort is the point.
- **Polish** one thing that bothers you.
- **Rest.**

### Milestone review

Run `/review` on `ChickenChase`. It's throwaway code, so this is the cheapest possible place to be told hard truths about your Unity habits. Pay attention to what it says about where you put game logic — M03 is coming and this is your warning shot.

### Where you are

Fourteen days in, you have **shipped a game**. Not a tutorial you followed — a thing you built, from empty project to zipped build with a menu, sound, and a win state. Most people who set out to learn Unity never do this once.

The training wheels come off tomorrow. Day 15 starts `Hollowbrook`, and everything you write from here survives to December.

**Commit:** `docs: M01 complete — first game shipped`
