# STATE

**Agents: read this first, write to it last. Keep it terse and true.**

```yaml
day: 1
date_of_day_1: 2026-09-10
last_session: null
days_missed_total: 0
projected_end: 2026-12-30

milestone: M00
milestone_title: Ground Zero — The Editor
milestone_day: 1        # 1-6 = content, 7 = buffer

active_project: null    # set on Day 1 -> D:\Projects\Unity Games\Sandbox00
unity_version: null     # record the exact version on Day 1
render_pipeline: null   # URP from M02 onward

status: not_started
```

## What exists

*Nothing yet. Day 1 changes that.*

## What is broken

*Nothing.*

## Parked

Things noticed but deliberately deferred. Revisit on buffer days.

*Empty.*

## Next action

**Day 1** — install check, create the first sandbox project, and learn the six windows that make up the Unity editor. Open `milestones/M00-ground-zero.md`.

---

### Format rules for agents

- Update **every** field in the YAML block at the end of every session. A stale `day` breaks `/session`.
- `What exists` is a bullet list of working features, newest last. It is the file he reads when he wants to quit — make it accurate and make it concrete.
- `What is broken` entries must include the day number they appeared: `- [D44] quest state saves but does not reload`.
- Never delete history from `LOG.md`. This file is the snapshot; that file is the record.
