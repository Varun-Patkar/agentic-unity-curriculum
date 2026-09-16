# M02 · Movement & Feel

**Days 15–21 · 24–30 Sep 2026 · project: `D:\Projects\Unity Games\Hollowbrook`**

> **This is the real one.** Everything you write from today survives to December. Delete `Sandbox00` — you don't need it any more, and deleting it is a small ceremony worth performing.
>
> The goal this week is unglamorous and enormously important: a character who moves *well*, in a place that exists, with a camera that doesn't annoy you. Player movement is the thing your player does for ten straight hours. If it feels bad, nothing else you build can save it.

**You end holding:** Alex, walking around Hollowbrook's town square, with a camera that behaves.

---

## Day 15 — Project setup, done right
**Thu 24 Sep · 60 min**

**Objective:** `Hollowbrook` exists, is on git, has a folder structure you won't fight, and has one sprite in it.

**Why:** You'll live here for 97 days. An hour spent on conventions today is worth ten hours in November. It's also the single most boring day of the curriculum, so let's do it properly and never think about it again.

### Concepts (10 min)
- **What to commit in a Unity project:** `Assets/` (including every `.meta`), `Packages/`, `ProjectSettings/`. Never `Library/`, `Temp/`, `Logs/`, `obj/`, `Builds/`.
- **`.meta` files are load-bearing.** They contain the GUID every reference points at. Ignoring them means every prefab reference breaks on another machine — including your own after a clean checkout.
- **Git and Unity:** set `Asset Serialization Mode` to **Force Text** and `Version Control Mode` to **Visible Meta Files** (`Project Settings > Editor`). Text serialization makes diffs readable and merges possible. Both are the default in modern Unity — verify anyway.
- **Large binary assets** will bloat the repo. Git LFS is the answer if it becomes a problem; for a project this size, probably not.
- **Folder naming with `_Project`** keeps your work above imported packages alphabetically. Small thing, constant payoff.

### Build (40 min)
1. New project, **Universal 2D**, at `D:\Projects\Unity Games\Hollowbrook`.
2. `git init`. Write `.gitignore` (Unity's official template is a good base). Verify `.meta` files are **not** ignored — this is the mistake that costs a weekend.
3. Create the folder structure:
   ```
   Assets/_Project/
     Art/{Sprites,Portraits,UI,Tiles}
     Audio/{SFX,Music,VO}
     Code/{Core,Unity,Editor}
     Content/{Dialogue,Quests}
     Input/  Materials/  Prefabs/  Scenes/  Settings/
   ```
4. `Project Settings > Editor`: confirm Force Text and Visible Meta Files.
5. Create `Assets/_Project/Scenes/Hollowbrook_TownSquare.unity` and make it your working scene.
6. Import the Input System package and set Active Input Handling to the new one (restart when prompted). Do it now so you never have to again.
7. Drop in a placeholder sprite for Alex Reed. PPU 32 (or 16 — decide now, write it in a `CONVENTIONS.md` in the project, never change it).
8. Create `ATTRIBUTIONS.md` in the project root. Add your first asset to it. Start this habit on day one; reconstructing it in December is genuinely miserable.
9. **First commit:** `chore: initial hollowbrook project setup`.

### Acceptance criteria
- [ ] Project opens clean, no Console errors
- [ ] `git status` is clean after commit and **includes `.meta` files**
- [ ] Folder structure created
- [ ] Force Text + Visible Meta Files confirmed
- [ ] Input System active, editor restarted
- [ ] `CONVENTIONS.md` records PPU, unit scale, and naming rules
- [ ] `ATTRIBUTIONS.md` exists with one entry

### Failure modes
- **Thousands of files staged** → `Library/` isn't ignored.
- **Prefab references break after a fresh clone** → `.meta` files were ignored. Fix the `.gitignore` and re-add them; this only gets worse with time.
- **Input System restart loop** → let Unity restart properly, don't cancel it.

**Stretch:** Set up a GitHub remote (private). A repo that exists in exactly one place on one drive is a repo you can lose.

**Commit:** `chore: initial hollowbrook project setup`

---

## Day 16 — Input Actions for a real game
**Fri 25 Sep · 60 min**

**Objective:** A `PlayerControls` asset with three action maps — `Gameplay`, `Dialogue`, `UI` — and code that switches between them.

**Why:** Context switching is the part everyone skips and everyone regrets. When you're in a conversation on Day 34, WASD must not move Alex. Building maps now means that's already solved.

### Concepts (10 min)
- **Action maps are contexts**, and only one (or a deliberate few) should be enabled at a time. This is the whole reason the system exists.
- **`Move` in `Gameplay` and `Navigate` in `Dialogue` can share the same physical keys** and mean different things. That's the abstraction working.
- **Generated C# class vs `PlayerInput` component.** The generated class gives explicit control over enabling/disabling maps, which is what you want here. Prefer it for Hollowbrook.
- **Interactions and Processors** — Tap, MultiTap; Invert, Normalize, Scale. Use them only when an action's meaning requires them.

### Build (40 min)
1. Create `Assets/_Project/Input/PlayerControls.inputactions`. Tick **Generate C# Class**.
2. **Gameplay map:** `Move` (Value/Vector2, WASD composite + left stick), `Interact` (Button, E + South button), `Attack` (Button, Mouse Left + West button), `Dodge` (Button, Space + East button), `Journal` (Button, J + Select), `Pause` (Button, Esc + Start).
3. **Dialogue map:** `Advance` (Space/Mouse Left/South), `Navigate` (Vector2, WS + stick), `Select` (Enter/South), `Skip` (Esc).
4. **UI map:** leave mostly to Unity's defaults for now.
5. Write `InputRouter.cs` — a single class owning the generated controls instance, with `EnableGameplay()`, `EnableDialogue()`, `EnableUI()`, each disabling the others. **Everything in the game asks this one class**; nothing else touches the input asset.
6. Wire `Move` into a temporary movement script and prove it works.
7. Prove the switching: bind a key to toggle to the Dialogue map and confirm WASD stops moving Alex.
8. Add `[SerializeField]` nothing here — this is a plain class, and that's deliberate.

### Acceptance criteria
- [ ] Three action maps exist with the bindings above
- [ ] `InputRouter` is the only thing that enables or disables maps
- [ ] Switching to Dialogue stops gameplay movement
- [ ] Both keyboard and gamepad bindings exist for every gameplay action
- [ ] Generated C# class is committed

### Failure modes
- **Two maps active at once** → you enabled without disabling. That's what the router is for.
- **Input keeps firing after switching** → you cached a callback reference; unsubscribe on disable.
- **Generated class is missing** → tick Generate C# Class and hit Apply on the asset.

**Stretch:** Add a second keyboard binding for `Interact`, then verify both bindings drive the same action without code changes.

**Commit:** `feat: input action maps with context routing`

---

## Day 17 — A top-down controller that feels good
**Sat 26 Sep · 60 min**

**Objective:** Movement you'd be happy to do for ten hours. Acceleration, friction, and a number you tuned by feel rather than guessed.

**Why:** This is the thing the player does constantly. It is worth an entire session and it is worth revisiting on buffer days.

### Concepts (10 min)
- **Raw input feels bad.** Instant full speed on press and instant stop on release reads as "cheap". Acceleration and deceleration curves are what make movement feel like a *body*.
- **Accel vs decel should differ.** Faster deceleration than acceleration feels responsive; the reverse feels like ice.
- **Normalise diagonals** or diagonal movement is 41% faster.
- **`Rigidbody2D.velocity` vs `MovePosition` vs `AddForce`.** For a top-down character: set velocity directly, or `MovePosition` for kinematic. Forces give you the least control.
- **Tuning by feel, not by theory.** Change the number in Play Mode, play for 30 seconds, change it again. Do not calculate.

### Build (40 min)
1. `PlayerMovement.cs` on Alex, with a `Rigidbody2D` (Gravity Scale 0, Freeze Rotation Z, Interpolate **on** — interpolation smooths physics-rate movement to frame rate and is the difference between smooth and subtly juddery).
2. Serialized fields: `_maxSpeed`, `_acceleration`, `_deceleration`. Read input in `Update`, apply in `FixedUpdate`, `Vector2.MoveTowards` the current velocity toward the target.
3. Normalise the input vector.
4. **Tune in Play Mode.** Spend a genuine ten minutes on this. Try max speed 3, 5, 8. Try instant accel vs slow. Find what makes Alex feel grounded rather than like a spaceship.
5. Track a `FacingDirection` — you need it for interaction (Day 34) and attacks (Day 51). Store the last non-zero input direction.
6. Add a simple sprite flip or four-direction sprite swap so facing is visible.
7. **Write the final numbers into `CONVENTIONS.md`** so a future you doesn't wonder why 4.7.

### Acceptance criteria
- [ ] Movement accelerates and decelerates, tuned by feel
- [ ] Diagonal isn't faster than cardinal
- [ ] Facing direction is tracked and visible on the sprite
- [ ] Rigidbody Interpolate is on
- [ ] You spent at least 10 minutes actually playing and adjusting

### Failure modes
- **Slides forever** → deceleration too low, or you never zero the velocity.
- **Feels laggy** → acceleration too low, or Interpolate off, or you're moving in `FixedUpdate` but reading a stale input.
- **Micro-jitter** → Interpolate off · camera following in `Update` instead of `LateUpdate` (tomorrow's problem, but note it now).
- **Sticks on walls** → friction on the collider. Use a Physics Material 2D with zero friction.

**Stretch:** Add footstep sounds on a distance-travelled interval rather than a timer. Distance-based feels correct at every speed; timer-based never does.

**Commit:** `feat: tuned top-down player controller`

---

## Day 18 — Tilemaps: painting Hollowbrook
**Sun 27 Sep · 60 min**

**Objective:** A recognisable town square — pavement, roads, storefronts, trees, and a boundary — painted rather than assembled from individual sprites.

**Why:** Hand-placing sprites doesn't scale past about thirty of them. Tilemaps are how 2D worlds are actually built, and today Hollowbrook becomes a place instead of a concept.

### Concepts (10 min)
- **Grid → Tilemap → Tile.** The Grid defines cell size, the Tilemap holds placements, Tiles are assets referencing sprites.
- **Multiple Tilemaps as layers** — `Ground`, `Detail`, `Obstacles`, `Above`. Separate sorting orders and separate collision. This is the standard structure and you should adopt it immediately.
- **Tile Palette** (`Window > 2D > Tile Palette`) — your brush. Learn the shortcuts: B brush, E erase, U box fill, I picker.
- **Rule Tiles** (2D Tilemap Extras package) auto-select the right sprite based on neighbours. The difference between painting a path in 20 seconds and 20 minutes. Install the package.
- **Tilemap Collider 2D + Composite Collider 2D** — the composite merges thousands of tile colliders into a few polygons. Massive performance difference, and without it collisions catch on invisible tile seams.

### Build (40 min)
1. Get a modern small-town tileset from a CC0 source or make a tiny placeholder set — 16×16 or 32×32, matching your PPU. Log it in `ATTRIBUTIONS.md`.
2. Slice it: Sprite Mode **Multiple**, Sprite Editor → Slice → Grid By Cell Size.
3. `GameObject > 2D Object > Tilemap > Rectangular`. Add four child tilemaps: `Ground` (order 0), `Detail` (1), `Obstacles` (2), `Above` (10).
4. Open the Tile Palette, create a palette, drag your sliced sprites in.
5. **Paint Hollowbrook's Town Square.** Not the whole town — one screen's worth. Pavement, a road, Town Hall and diner footprints, parked-car shapes, and a treeline boundary. Twenty minutes, no perfectionism.
6. Add `Tilemap Collider 2D` + `Composite Collider 2D` (Used By Composite ✓ on the tilemap collider, Rigidbody2D set to **Static**) on `Obstacles`.
7. Walk around. Bump into things. Fix the places where collision feels wrong.
8. Install **2D Tilemap Extras** and convert your path to a Rule Tile if time allows.

### Acceptance criteria
- [ ] Four-layer tilemap structure with sensible sorting orders
- [ ] A screen of Hollowbrook exists and reads as a modern town square
- [ ] Obstacles block the player via a Composite Collider
- [ ] You can paint new tiles in under ten seconds
- [ ] Tileset logged in `ATTRIBUTIONS.md`

### Failure modes
- **Tiles have thin gaps between them** → texture bleeding. Fix with padding/extrusion in the sprite import, or a pixel-perfect camera.
- **Player catches on invisible edges** → Composite Collider not set up, or Used By Composite unticked.
- **Tiles render behind the player inconsistently** → sorting order on the tilemap renderer.
- **Palette is empty** → drag the *sliced sub-sprites*, not the parent texture.

**Stretch:** Add a `Above` layer for tree canopies the player walks behind, at a sorting order above the player. It's the cheapest possible depth illusion and it looks great.

**Commit:** `feat: hollowbrook town square tilemap with collision`

---

## Day 19 — Collision, layers, and the sorting problem
**Mon 28 Sep · 60 min**

**Objective:** A layer scheme you designed on purpose, and a player who correctly renders in front of and behind things as they walk.

**Why:** The Y-sorting problem is *the* 2D top-down problem. Solve it once, properly, today — and never think about it again.

### Concepts (10 min)
- **Three unrelated systems that sound identical:** *Layers* (physics + camera culling), *Tags* (string labels), *Sorting Layers* (2D draw order).
- **The physics collision matrix** (`Project Settings > Physics 2D`) — which layers can touch which. Design it deliberately: `Player`, `Enemy`, `PlayerHitbox`, `EnemyHitbox`, `Interactable`, `Obstacle`, `Trigger`. You'll thank yourself in M07.
- **Y-sorting:** in a top-down view, things lower on screen are nearer the camera and must draw in front. Unity does this for you: set **Transparency Sort Mode** to *Custom Axis* `(0, 1, 0)` in `Project Settings > Graphics`, and put everything on the same sorting layer.
- **Pivot placement matters** — Y-sorting uses the transform position, so a character's pivot should be at their feet.

### Build (40 min)
1. `Project Settings > Tags and Layers`: define the layers above. Assign them to existing objects.
2. `Project Settings > Physics 2D`: **untick everything, then tick only what must collide.** Player↔Obstacle, Player↔Interactable, Player↔EnemyHitbox, Enemy↔Obstacle, PlayerHitbox↔Enemy. Not Player↔PlayerHitbox. Not Enemy↔Enemy (decide, and know why).
3. `Project Settings > Graphics` → Transparency Sort Mode **Custom Axis**, Custom Axis `(0, 1, 0)`.
4. Put the player, NPCs, trees, and buildings on the same sorting layer (`Entities`).
5. Set every character sprite's **pivot to bottom-centre** in the Sprite Editor.
6. Walk behind a tree, then in front of it. It should just work. If it doesn't, the pivot or the sorting layer is wrong.
7. Add a second collider on the player: a small **feet** collider for movement (blocked by obstacles) and a larger **body** trigger for interaction. Two colliders on one object is normal and useful.
8. Document the layer scheme in `CONVENTIONS.md`.

### Acceptance criteria
- [ ] Layer scheme defined and documented
- [ ] Physics matrix set deliberately, not left at "everything collides"
- [ ] Y-sorting works: player renders in front when below, behind when above
- [ ] Character pivots at the feet
- [ ] Player has separate movement and interaction colliders

### Failure modes
- **Sorting doesn't change with Y** → objects are on different sorting layers, or Transparency Sort Mode wasn't set, or their Order in Layer is set explicitly.
- **Sorting is inverted** → custom axis is `(0,-1,0)`.
- **Player gets stuck on corners** → box collider on the feet; try a capsule or circle.
- **Collision silently stops working** → check the matrix. This will happen to you and this is where to look.

**Stretch:** Make the tree canopy fade to 50% alpha when the player is behind it. Trigger + sprite alpha lerp. Small, and it reads as care.

**Commit:** `feat: layer scheme and y-sorting`

---

## Day 20 — Cinemachine: a camera that doesn't annoy you
**Tue 29 Sep · 60 min**

**Objective:** A camera that follows Alex smoothly, has a dead zone, and never shows the void beyond the map edge.

**Why:** A bad camera makes good movement feel bad. This is one hour that improves every subsequent hour of the project.

### Concepts (10 min)
- **Cinemachine** is a virtual-camera system: virtual cameras describe *intent*, and a brain on the real camera blends between them. You'll use it for dialogue framing on Day 99 too.
- **Dead zone** — a region where the target can move without the camera moving. Essential in top-down; without it, tiny movements swing the whole screen and induce nausea.
- **Damping** — how fast the camera catches up. Too low: rigid. Too high: seasick.
- **Confiner** — clamps the camera to a polygon so it never shows past the level bounds.
- **Cameras update in `LateUpdate`**, after everything has moved. Cinemachine handles this; hand-rolled cameras usually don't, which is why they judder.

### Build (40 min)
1. Install **Cinemachine** from the Package Manager. *(Verify the current version and component names — Cinemachine 3.x renamed several things from 2.x. Have your agent check rather than guess.)*
2. Add a Cinemachine Camera to the scene, set Follow to the player. Note the brain component that appears on the Main Camera.
3. Configure the framing: set a **dead zone** and tune **damping**. Play, walk around, adjust in Play Mode.
4. Add a **Confiner 2D** with a Polygon Collider 2D drawn around the Town Square bounds. Walk to the edge; the camera should stop while the player keeps going.
5. Set the orthographic size to frame the square well. Consider a **Pixel Perfect Camera** component if you're using pixel art — it eliminates shimmer, at the cost of some camera-smoothness flexibility. Decide deliberately.
6. **Screenshake infrastructure:** add a Cinemachine Impulse Source to the player and an Impulse Listener on the camera. Trigger it on a keypress to test. You'll use this constantly from M07 onward, so build the plumbing now.
7. Play for five minutes. Adjust until you stop noticing the camera. That's the goal — a camera you notice is a camera that's wrong.

### Acceptance criteria
- [ ] Camera follows with a dead zone and tuned damping
- [ ] Camera never shows past the village boundary
- [ ] Movement is smooth with no judder at any point
- [ ] Impulse (shake) infrastructure is in place and testable
- [ ] You genuinely stopped noticing the camera while playing

### Failure modes
- **Camera judders** → target moving in `FixedUpdate` while camera updates per frame; enable Rigidbody Interpolate (you did on Day 17 — verify).
- **Pixel art shimmers while moving** → needs Pixel Perfect Camera, and PPU must match your assets exactly.
- **Confiner does nothing** → the polygon collider must be on a separate object, and shape must be assigned to the confiner.
- **Camera won't follow** → Follow target not assigned, or there's no Cinemachine Brain on the Main Camera.

**Stretch:** Add a second virtual camera that zooms in slightly when the player stands still for 3 seconds, and blend between them. It costs almost nothing and it's the kind of quiet touch that makes a game feel considered.

**Commit:** `feat: cinemachine follow camera with confiner and impulse`

---

## Day 21 — BUFFER
**Wed 30 Sep**

- **Catch up** on anything unfinished.
- **Polish the feel.** Go back to Day 17's numbers. Play for 15 minutes and tune. This is never wasted time.
- **Paint more of Hollowbrook.** Purely enjoyable, genuinely useful.
- **Story homework:** confirm whether `Last Stop, Hollowbrook` stays as the title. Record the decision in `reference/story-bible.md`.
- **Rest.**

### Milestone review

Run `/review` scoped to this milestone. Pay particular attention to anything it says about where logic lives — **tomorrow you build the wall**, and it's much easier if `PlayerMovement` isn't already doing six jobs.

### Heads up: the hard part starts tomorrow

M03 is the least visually rewarding week in the entire curriculum. You will write code for six days and the game will look **exactly the same** on Day 28 as it does today.

It is also the milestone the whole plan is built on. Everything you write next week is what makes M10 a six-week job instead of a sixteen-week one, and it's the reason you'll still be able to reason about this codebase in December.

Six days. The screen doesn't change. Trust it.

**Commit:** `docs: M02 complete — movement, world, camera`
