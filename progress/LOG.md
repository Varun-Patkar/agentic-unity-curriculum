# LOG

One entry per session. Agents append; nobody edits history.

This file has one job beyond record-keeping: **on the day you want to quit, you read this from the top and discover you have built more than you remember.** Keep it honest, including the bad days — a log of only good days is worthless for that purpose.

---

## Entry template

```markdown
### Day N — YYYY-MM-DD — <milestone> — <title>

**Built:** what actually works now that didn't before
**Broke:** what fought back, and how it was resolved (or that it wasn't)
**Learned:** the one idea worth remembering from this hour
**Criteria:** N/M passed
**Commit:** `<sha or message>`
**Felt:** one honest word or line
```

Rest days and buffer days get entries too:

```markdown
### Day N — YYYY-MM-DD — BUFFER — rest
Took it. Streak intact.
```

---

## Sessions

<!-- Newest entries go at the BOTTOM. Append, never prepend. -->

### Day 1 — 2026-09-09 — M00 — Install audit and the Unity editor

**Built:** Created `Sandbox00` in Unity 6.6, toured the six editor windows, navigated the Scene view, set a Play Mode tint, moved a cube, and saved `SampleScene`.
**Broke:** Nothing. Deliberately changed the cube during Play Mode and watched Unity discard the change.
**Learned:** Play Mode runs temporary scene state; use it to inspect and tune, then make persistent edits outside it.
**Criteria:** 6/6 passed
**Commit:** `docs: day 1, unity version and project path recorded`
**Felt:** Interactive and companionable; better than consuming a block of content and taking a quiz afterward.

### Day 2 — 2026-09-10 — M00 — GameObjects, Components, Transforms

**Built:** Composed a cart from a parent, bed, and two wheels; moved it through its parent Transform; duplicated and independently reshaped a second cart; added a ground plane; and assigned `Interactable` and `Ground` classifications.
**Broke:** Nothing. The cart remained deliberately static; movement and input are not part of Day 2.
**Learned:** A GameObject is a scene identity and component container; components add capabilities, while parent Transforms define a shared local coordinate space.
**Criteria:** 5/5 passed
**Commit:** `docs: day 2 log`
**Felt:** Great and interactive, though I hoped to drive the cart; I understand that comes later.

### Day 3 — 2026-09-11 — M00 — Your first script and the MonoBehaviour lifecycle

**Built:** Created `Spinner` with an Inspector-controlled rotation speed, proved the startup callback order on two objects, compared frame-independent and frame-dependent rotation, and made Space pause and resume spinning.
**Broke:** Legacy `Input.GetKeyDown` threw an `InvalidOperationException` because the project used the new Input System exclusively; changed Active Input Handling to Both for today's legacy-input exercise.
**Learned:** Unity owns the loop and invokes exact-name callbacks; `Awake` runs once per component lifetime, `OnEnable` runs for each active period, and per-frame movement needs `Time.deltaTime`.
**Criteria:** 5/5 passed
**Commit:** `docs: day 3 log`
**Felt:** Fun; getting to make input visibly change something felt good.

### Day 4 — 2026-09-12 — M00 — Prefabs, instantiation, and the Project window

**Built:** Turned the spinning cube into a coloured prefab, proved asset propagation and instance overrides, spawned 50 copies at random positions through an Inspector-assigned prefab, and organised project assets under `_Project` without breaking references.
**Broke:** Nothing. The existing Day 2 carts and Day 3 comparison objects were removed after they had served their purpose.
**Learned:** Prefab assets are serialized templates; instances inherit asset changes except where a property has an explicit override, while `.meta` GUIDs preserve references when assets move.
**Criteria:** 5/5 passed
**Commit:** `docs: day 4 log`
**Felt:** Fifty synchronized red cubes felt a bit like the spinning cat meme.

### Day 5 — 2026-09-13 — M00 — Physics: Rigidbody, colliders, and the FixedUpdate rule

**Built:** Stacked a stable 15-box Rigidbody wall, launched a ball through it with an impulse applied in `FixedUpdate`, detected the ball through a trigger zone, and added a bouncy low-friction Physics Material.
**Broke:** Deliberately teleported the ball through the wall by changing its Transform, then disabled `Is Trigger` and watched the trigger become a solid invisible barrier; restored Rigidbody movement and trigger behavior afterward.
**Learned:** A Rigidbody gives physics ownership of movement; input belongs in `Update`, physics actions belong in `FixedUpdate`, and collision or trigger interaction requires a Rigidbody on at least one participating object.
**Criteria:** 5/5 passed
**Commit:** `docs: day 5 log`
**Felt:** Functionally similar to a game, though currently an 80s bowling game.

### Day 6 — 2026-09-14 — M00 — Build the toy, then export a real executable

**Built:** Turned the physics exercise into a mouse-aimed bowling toy with a red trajectory line, click-to-launch input, permanent fallen-box scoring and recolouring, an out-of-bounds loss state, R-to-restart scene reload, and a stable overview camera. Built, tested with Unity closed, and zipped a 1280×720 windowed Windows release.
**Broke:** The trajectory initially reused Line Renderer endpoint index 0 and rendered pink with an incompatible material; assigned endpoint index 1 correctly and switched the material to a URP-compatible shader.
**Learned:** A camera ray can intersect an invisible mathematical plane to turn a screen-space cursor into a world-space aim direction; a build is a separate deployed program whose executable depends on its adjacent data files.
**Criteria:** 5/5 passed
**Commit:** `docs: day 6 log — first build shipped`
**Felt:** Mouse control and the overview camera made it feel like a pretty good game.

### Day 7 — 2026-09-14 — BUFFER — polish

**Built:** Added a resolution-aware Canvas HUD with current and persistent best scores, a Reset Best button, separate game-over and victory overlays, and a full-platform out-of-bounds trigger. R reloads into a clean run, losses remain losses after a previous win, and knocking down all 15 boxes ends in victory.
**Broke:** The scoreboard initially rendered partly off-screen because its pivot remained centred; fixed the top-left pivot. The loaded high score initially shadowed the controller field, and the first out-of-bounds trigger was too small for the ground's doubled scale; fixed the assignment and expanded the catch volume.
**Learned:** Canvas anchors and pivots control responsive placement, UI event callbacks reference live scene components rather than script assets, and `PlayerPrefs` is sufficient for one disposable persistent integer but not a real save system.
**Criteria:** 5/5 passed
**Commit:** `docs: M00 complete — editor fundamentals`
**Felt:** Good enough for today; opening Unity and building a small scene now feels easy.

### Day 8 — 2026-09-15 — M01 — 2D URP project, sprites, and the 2D pipeline

**Built:** Created the Universal 2D `ChickenChase` project with a Unity-aware Git boundary, organised a `Game` scene under `_Project`, imported a peasant and three chickens at a consistent 16 PPU, extracted a fully opaque grass tile, framed the playfield with an orthographic camera, added sorting layers, and fitted every entity with a `BoxCollider2D`.
**Broke:** The first grass choices were transition tiles with transparent edges. After replacing them, the peasant still rendered far from its Transform because its custom pivot Y was accidentally set to 9 instead of 0; correcting the normalized bottom-centre pivot fixed it.
**Learned:** PPU controls world size, the pivot controls where sprite pixels sit relative to the Transform, and Sorting Layers control 2D draw order independently of physics Layers, Hierarchy order, and Z position.
**Criteria:** 5/5 passed
**Commit:** `46a1980` (`feat: 2d project setup with sprites and sorting layers`)
**Felt:** Fiddly while choosing grass and diagnosing the pivot, but the visible failures made the 2D conventions concrete.

### Re-plan — 2026-09-16 — End goal — Last Stop, Hollowbrook

**Built:** Replaced the medieval action-RPG destination with a modern supernatural-town choice RPG: two fully developed story endings, one first-playthrough secret ending where Mayor Vale escalates from annoyance to ejection for repeated dialogue skipping, and simple attack-and-dodge combat against one creature archetype.
**Broke:** The original premise was embedded throughout the milestone briefs rather than isolated to the story bible; retargeted all 16 milestones while preserving all 112 day numbers and the Unity learning sequence.
**Learned:** The dialogue-skip joke needs to be a tested Core rule with accessibility and repeat-playthrough exemptions, not a late presentation trick.
**Criteria:** Story bible, curriculum, and milestone consistency checks passed; Day 9 remains current.
**Commit:** `docs: retarget curriculum to Last Stop Hollowbrook`
**Felt:** The project now sounds like the game I actually want to finish.

### Day 9 — 2026-09-16 — M01 — The Input System, properly

**Built:** Created a `PlayerControls` Input Actions asset with normalized WASD and gamepad left-stick bindings, then moved the peasant through a `Rigidbody2D` at a tuned speed of 5.
**Broke:** Nothing. Solid chicken colliders block movement and the unbounded playfield allows leaving the screen; both are expected at this stage.
**Learned:** Gameplay code reads the abstract `Move` intent while Unity continuously resolves the bound device input; input is sampled in `Update` and consumed by physics in `FixedUpdate`.
**Criteria:** 5/5 passed; gamepad support was verified structurally because no controller was available.
**Commit:** `c96e592` (`feat: input system driven player movement`)
**Felt:** Cool to see Unity handle continuous input instead of manually moving something once per key press.

### Day 10 — 2026-09-17 — M01 — The loop: spawn, collect, score

**Built:** Turned the chicken into a trigger prefab, spawned three at varied positions away from the player, collected them through collision, scored through an event subscriber, spawned replacements through a second subscriber, and ended movement after a 60-second timer.
**Broke:** The timer initially checked for exact zero, which frame time can skip; changed the condition to `<= 0`. The event-driven split was initially introduced too quickly, then made concrete as one publisher with independent score and spawn subscribers.
**Learned:** A pickup can publish one fact without knowing its consumers; the manager scores it and the spawner replaces it independently, with subscriptions paired in `OnEnable` and `OnDisable`.
**Criteria:** 5/5 passed
**Commit:** `29b64ce` (`feat: collect loop with score and timer`)
**Felt:** The message-passing model clicked once collecting a chicken visibly caused both scoring and respawning.

### Day 11 — 2026-09-18 — M01 — UI: score, timer, game over

**Built:** Added a resolution-aware Canvas HUD with event-driven score and countdown text, plus a game-over overlay showing the final score and a Restart button that reloads a clean run.
**Broke:** Nothing. A five-second test duration made the game-over loop quick to verify before restoring the intended 60 seconds.
**Learned:** UI can subscribe to game-state events instead of polling every frame; anchors keep RectTransforms screen-relative as aspect ratios change.
**Criteria:** 5/5 passed
**Commit:** `3f65280` (`feat: score/timer hud and game over screen`)
**Felt:** Nice to revisit UI.

### Day 12 — 2026-09-19 — M01 — Juice: sound, particles, screenshake

**Built:** Added a pitch-varied chicken collect call, a self-cleaning collection particle burst, subtle decaying screenshake, score-text punch, chicken squash-to-zero animation, final-ten-second ticks, one-shot game-over audio, and alternating footsteps tuned to a 0.5-second interval.
**Broke:** The first chicken audio cut included an unwanted chirp and was recut to retain the middle and elegant final call. Footsteps initially continued after game over because disabled movement retained stale input; `IsMoving` now also requires the component to be active and enabled.
**Learned:** Small feedback layers make unchanged mechanics feel responsive; coroutines spread short animations across frames, while cached base transforms prevent shake drift.
**Criteria:** 5/5 passed
**Commit:** `6adc2d6` (`feat: audio, particles, and screenshake`)
**Felt:** The varied chicken call sounded natural, and the complete feedback stack made much more sense in motion.

### Day 13 — 2026-09-20 — M01 — Menus, scene flow, and a real build

**Built:** Added a title scene with Play and Quit, registered MainMenu and Game in the Windows Build Profile, added game-over navigation back to the title, and implemented an Escape/Resume pause menu that stops the timer, movement, and footsteps. Built, tested on a second PC without Unity, and zipped the 1280x720 windowed Windows release.
**Broke:** The first pause implementation toggled the game-over panel and omitted `Time.timeScale`; after fixing both, movement input could still trigger footsteps while paused, so the audio update now explicitly ignores paused time.
**Learned:** Scene loading destroys scene-owned state, Build Profiles define the executable entry point, and `Time.timeScale = 0` stops scaled simulation but does not stop `Update` or input callbacks.
**Criteria:** 5/5 passed
**Commit:** `52b7236` (`feat: menus, scene flow, and v1 build`)
**Felt:** Great. A full game was created by me.

### Day 14 — 2026-09-21 — BUFFER — polish

**Built:** Added four screen-relative collider walls that keep the player inside the orthographic camera view, and updated chicken spawning to maintain configurable separation from the player and other live chickens with bounded retries.
**Broke:** The boundary parent initially had a large world-space offset; after resetting it, the bottom wall faced inward and the side-wall width and height calculations were swapped. The chicken distance calculation also needed an explicit `Vector3`-to-`Vector2` cast.
**Learned:** Orthographic camera bounds convert viewport constraints into world-space colliders, while randomized placement needs candidate validation and a finite retry budget.
**Criteria:** 2/2 passed
**Commit:** `35f25c8` (`fix: constrain player and chicken spawns`)
**Felt:** Two lingering gameplay annoyances are now gone.

### Day 15 — 2026-09-22 — M02 — Project setup, done right

**Built:** Created the durable Universal 2D `Hollowbrook` project with a Unity-aware Git boundary, tracked `.meta` files, the full `_Project` folder structure, Force Text serialization, the new Input System, a saved town-square scene, and a project-owned 32 PPU placeholder for Alex Reed. Documented scale, naming, and attribution conventions from the first asset.
**Broke:** Changing Alex's import mode from Multiple to Single invalidated the originally dragged sub-sprite; dragging the corrected single sprite back into the scene restored it.
**Learned:** Unity asset references depend on GUIDs stored in `.meta` files, while `Library/` is generated state; preserving one and ignoring the other is the core Git boundary.
**Criteria:** 7/7 passed
**Commit:** `5fe6772` (`chore: initial hollowbrook project setup`)
**Felt:** Not artsy, but Alex is recognizably standing in the real project and the foundation is clean.

### Day 16 — 2026-09-23 — M02 — Input Actions for a real game

**Built:** Created Gameplay, Dialogue, and empty-for-now UI maps with keyboard and gamepad bindings and generated C# controls. Wrote a plain C# router that switches contexts; a temporary Rigidbody2D movement component moves Alex at equal cardinal and diagonal speed. J enters Dialogue and stops movement; Escape restores Gameplay movement.
**Broke:** Diagonal movement was initially faster; clamping input magnitude to one fixed it. No Console errors at the end.
**Learned:** Action maps separate physical keys from context-specific intent; one router owns map lifetime while the Unity component owns movement.
**Criteria:** 5/5 passed; gamepad bindings verified structurally, not with a physical controller.
**Commit:** `92b6c54` (`feat: input action maps with context routing`)
**Felt:** Good; compartmentalizing responsibilities made the pieces feel separate and clear.

### Day 17 — 2026-09-24 — M02 — A top-down controller that feels good

**Built:** Renamed the temporary movement script to `PlayerMovement`; Alex now accelerates and decelerates through Rigidbody2D velocity, retains the last non-zero facing direction, visibly flips left/right, and uses Rigidbody2D interpolation. Gameplay-to-Dialogue switching still stops movement; Escape restores it. Chose speed 5, acceleration 20, deceleration 30.
**Broke:** Nothing reported. The script's `.meta` GUID survived the rename and the scene reference remains intact.
**Learned:** Sample input per frame, approach target velocity on the physics tick, and interpolate the Rigidbody for smooth rendering between ticks.
**Criteria:** 4/5 confirmed; played roughly 5-10 minutes, so the minimum ten-minute tuning criterion is not confirmed.
**Commit:** `d676b20` (`feat: tuned top-down player controller`)
**Felt:** The default tuning felt good; Alex now starts and stops gradually.

**Follow-up (2026-09-24):** Played longer after closeout; total tuning time exceeded ten minutes and the default values still felt good. Final criteria: 5/5 passed.

### Day 18 — 2026-09-25 — M02 — Tilemaps: painting Hollowbrook

**Built:** Imported and attributed Kenney's CC0 modern-city tileset, sliced it at 16x16 with one-pixel spacing, and set up a four-layer Tilemap on a half-unit Grid. Painted pavement, a road strip, and a brick block; merged obstacle collision now stops Alex.
**Broke:** Alex walked through the brick block despite a static composite collider. The Console was clear; inspecting Alex revealed a Rigidbody2D but no Collider2D. Adding a non-trigger Capsule Collider 2D fixed it.
**Learned:** A Rigidbody2D moves through physics, but each interacting object still needs a Collider2D shape; visible tiles alone do not imply physical contact.
**Criteria:** 4/5 passed; the layout does not yet read as a modern town square. Finish the visual pass before Day 19.
**Commit:** `61bbb7e` (`wip: day 18, tilemap collision works; town square unfinished`)
**Felt:** Painting got boring quickly; the collision diagnosis was a better use of the hour. Tired, stopping on time.

### Day 18 follow-up — 2026-09-26 — M02 — Finish the town square

**Built:** Delegated repetitive painting to a reusable Unity Editor painter. The one-screen square now has distinct Town Hall and diner footprints, signs, a paved plaza, a solid asphalt road with sparse markings and simpler sidewalk strips, parked-car shapes, and a treeline. Verified saved tile coverage and an unobstructed player spawn in a batch Editor run; reviewed Scene and Game view screenshots.
**Broke:** The first automated layout repeated edge and transparent marking sprites as fills, making the road striped and buildings outlined. Selected solid center tiles by inspecting sprite pixel data, removed unused generated tile assets, and re-ran the Unity scene audit.
**Learned:** An atlas's edge and center sprites are not interchangeable; check the actual scene at game scale before declaring an automated tile fill done.
**Criteria:** Day 18 now 5/5; existing obstacle collision and quick palette painting were verified on 2026-09-25. Camera follow remains scheduled for Day 20.
**Commit:** `81b7c2d` (`feat: paint hollowbrook town square`)
**Felt:** Automation was useful once the visual mistakes were caught in Game view; a fixed camera still feels limiting.
