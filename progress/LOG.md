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
