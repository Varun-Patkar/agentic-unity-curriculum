# STATE

**Agents: read this first, write to it last. Keep it terse and true.**

```yaml
day: 6
date_of_day_1: 2026-09-09
last_session: 2026-09-13
days_missed_total: 0
projected_end: 2026-12-29

milestone: M00
milestone_title: Ground Zero — The Editor
milestone_day: 6        # 1-6 = content, 7 = buffer

active_project: D:\Projects\Unity Games\Sandbox00
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

## What is broken

*Nothing.*

## Parked

Things noticed but deliberately deferred. Revisit on buffer days.

*Empty.*

## Next action

**Day 6** — turn the milestone exercises into a small toy, then make a Windows build.

---

### Format rules for agents

- Update **every** field in the YAML block at the end of every session. A stale `day` breaks `/session`.
- `What exists` is a bullet list of working features, newest last. It is the file he reads when he wants to quit — make it accurate and make it concrete.
- `What is broken` entries must include the day number they appeared: `- [D44] quest state saves but does not reload`.
- Never delete history from `LOG.md`. This file is the snapshot; that file is the record.
