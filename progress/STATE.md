# STATE

**Agents: read this first, write to it last. Keep it terse and true.**

```yaml
day: 4
date_of_day_1: 2026-09-09
last_session: 2026-09-11
days_missed_total: 0
projected_end: 2026-12-29

milestone: M00
milestone_title: Ground Zero — The Editor
milestone_day: 4        # 1-6 = content, 7 = buffer

active_project: D:\Projects\Unity Games\Sandbox00
unity_version: 6000.6.0f1
render_pipeline: URP

status: ready
```

## What exists

- `Sandbox00` Universal 3D project opens without Console errors.
- `SampleScene` contains a cube and is saved under `Assets/Scenes/`.
- Play Mode tint is set and Windows Build Support is installed.
- `SampleScene` contains two independently editable carts, each composed from a parent, bed, and two wheel children.
- A ground plane uses the `Ground` layer, and the original cart uses the `Interactable` tag.
- `Spinner` rotates a cube at an Inspector-set speed and toggles on Space; a second component demonstrates frame-dependent movement.
- Lifecycle logs on two objects prove `Awake`/`OnEnable`/`Start`/first-`Update` ordering, and Active Input Handling is set to Both for the Day 3 legacy-input exercise.

## What is broken

*Nothing.*

## Parked

Things noticed but deliberately deferred. Revisit on buffer days.

*Empty.*

## Next action

**Day 4** — turn the spinning cube into a prefab, explore asset-versus-instance edits and overrides, then write a spawner that instantiates multiple copies.

---

### Format rules for agents

- Update **every** field in the YAML block at the end of every session. A stale `day` breaks `/session`.
- `What exists` is a bullet list of working features, newest last. It is the file he reads when he wants to quit — make it accurate and make it concrete.
- `What is broken` entries must include the day number they appeared: `- [D44] quest state saves but does not reload`.
- Never delete history from `LOG.md`. This file is the snapshot; that file is the record.
