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
