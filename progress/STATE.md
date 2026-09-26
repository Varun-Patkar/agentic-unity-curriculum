# STATE

**Agents: read this first, write to it last. Keep it terse and true.**

```yaml
day: 19
date_of_day_1: 2026-09-09
last_session: 2026-09-26
days_missed_total: 0
projected_end: 2026-12-29

milestone: M02
milestone_title: Movement & Feel
milestone_day: 5        # 1-6 = content, 7 = buffer

active_project: D:\Projects\Unity Games\Hollowbrook
unity_version: 6000.6.0f1
render_pipeline: URP

status: ready
```

## What exists

- `Sandbox00` Universal 3D project opens without Console errors.
- `SampleScene` is saved under `Assets/_Project/Scenes/`.
- Play Mode tint is set and Windows Build Support is installed.
- A ground plane uses the `Ground` layer.
- `Spinner` rotates a cube at an Inspector-set speed and toggles on Space.
- Active Input Handling is set to Both for the Day 3 legacy-input exercise.
- `SpinnerCube` is a coloured prefab whose asset changes propagate to scene and runtime instances; instance overrides can be identified and reverted.
- `Spawner` creates 50 spinning prefab instances at random positions from an Inspector-assigned reference.
- Project-owned scenes, scripts, prefabs, and materials are organised under `Assets/_Project/` without broken references.
- A stable 15-box Rigidbody wall collapses when struck by a physics-launched ball.
- `Launcher` captures input in `Update` and applies an impulse in `FixedUpdate`.
- `LaunchTrigger` reports entering objects without blocking them, and `BouncyBall` gives the projectile low-friction, high-bounce contact behavior.
- The bowling toy aims from the mouse through a red trajectory line and launches on left click.
- Fallen boxes score once, turn red, and reset with the full scene when R is pressed; an out-of-bounds ball reports the loss and restart instruction.
- A 1280×720 windowed Windows build runs from `Builds/v1/` with Unity closed, and the complete build folder is zipped for distribution.
- A resolution-aware HUD shows current and persistent best scores, with a working Reset Best button on terminal screens.
- A full-platform out-of-bounds trigger shows Game Over, while knocking down all 15 boxes shows You Win; R starts a clean run.
- M00 exit check passed: opening Unity and building a small scene now feels easy.
- `ChickenChase` is a clean Universal 2D project with a Unity-aware `.gitignore` and tracked `.meta` files.
- The `Game` scene has an intentionally framed orthographic playfield, a peasant, three chickens, and coherent 16 PPU pixel art.
- `Background`, `Ground`, `Entities`, and `UI` Sorting Layers exist; entities render over the ground without relying on Z position.
- Every entity has a `BoxCollider2D`, and Play Mode runs without Console errors.
- The post-M01 curriculum now targets *Last Stop, Hollowbrook*: a modern supernatural choice RPG with two full-story endings, one first-playthrough dialogue-skip ending, and deliberately simple combat.
- The peasant moves smoothly at speed 5 through a `Rigidbody2D`, using one `Move` action bound to normalized WASD and the gamepad left stick with no legacy input calls.
- Three collectible chicken prefabs spawn at varied positions away from the player, score exactly once through an event, and immediately respawn elsewhere.
- A 60-second game timer logs game over and disables player movement when it expires; event subscriptions are paired with unsubscriptions.
- A resolution-aware HUD shows event-driven score and countdown updates; game over reveals the final score and a Restart button that starts a clean run.
- Chicken collection now combines pitch-varied audio, a self-cleaning particle burst, subtle decaying screenshake, score-text punch, and squash-to-zero animation.
- Alternating footsteps play only while movement is enabled; the final ten seconds tick down and game over plays once.
- A title menu starts the game and quits the Windows build; game over can return to the title, and Escape/Resume pauses the timer, movement, and footsteps.
- `MainMenu` and `Game` are registered in that order in the Windows Build Profile; the 1280x720 windowed build passed the full flow on a second PC without Unity and is stored as a ZIP.
- Screen-relative collider walls keep the player inside the orthographic camera view at different aspect ratios.
- Chicken spawning keeps a configurable minimum distance from the player and other live chickens, with a bounded retry count.
- `Hollowbrook` is a clean Universal 2D project with a Git boundary, durable `_Project` folder structure, Force Text serialization, visible tracked `.meta` files, and the new Input System active.
- `Hollowbrook_TownSquare` opens without Console errors and contains a 32 PPU project-owned placeholder sprite for Alex Reed; project conventions and asset attribution are documented.
- `PlayerControls` has Gameplay, Dialogue, and empty-for-now UI action maps, with keyboard and gamepad bindings and a committed generated C# class.
- Plain C# `InputRouter` owns map switching; a temporary Rigidbody2D movement component moves Alex at equal cardinal and diagonal speed. J switches to Dialogue and stops movement; Escape returns to Gameplay and restores it.
- `PlayerMovement` accelerates to speed 5 at rate 20 and stops at rate 30; last non-zero facing direction is retained, the sprite flips left/right, and Rigidbody2D interpolation is enabled. Dialogue still stops movement.
- `Hollowbrook_TownSquare` has four tilemaps (`Ground`, `Detail`, `Obstacles`, `Above`) with a Kenney CC0 tileset at 32 PPU on a half-unit Grid; pavement, a road strip, and a brick block are painted.
- `Obstacles` has a merged Tilemap/Composite Collider 2D on a static Rigidbody2D; Alex's Capsule Collider 2D stops him at the brick block.
- Day 18's town square now reads as a modern block with distinct Town Hall and diner footprints, a central plaza, solid asphalt with sparse lane markings, parked-car shapes, and a treeline; a reusable Editor painter produced the scene through Unity APIs.

## What is broken

*Nothing.*

## Parked

Things noticed but deliberately deferred. Revisit on buffer days.

- [D8] Chicken trigger bounds remain generous; tune them on a buffer day.
- [D12] Audio source URLs, authors, and licences are unknown; verify `ATTRIBUTIONS.md` before distribution.
- [D18] Art is intentionally placeholder-scale; consider improving the parked cars and sidewalk on a later polish day. The fixed camera crops at narrow Free Aspect; Day 20 covers camera follow and bounds.

## Next action

**Day 19** — design collision layers, Y-sorting, and player colliders following the M02 brief.

---

### Format rules for agents

- Update **every** field in the YAML block at the end of every session. A stale `day` breaks `/session`.
- `What exists` is a bullet list of working features, newest last. It is the file he reads when he wants to quit — make it accurate and make it concrete.
- `What is broken` entries must include the day number they appeared: `- [D44] quest state saves but does not reload`.
- Never delete history from `LOG.md`. This file is the snapshot; that file is the record.
