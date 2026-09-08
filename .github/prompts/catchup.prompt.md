---
description: 'I missed days. Re-plan the curriculum without guilt or lost work.'
mode: agent
---

# Catch up

He missed days. This is expected, it is planned for, and your first job is to make the re-entry cost as close to zero as possible.

**Say nothing about the gap beyond acknowledging it in half a sentence.** No "no worries!", no "life happens", no discussion of motivation unless he raises it. The fastest route back is straight past it.

## Step 1 — Reconstruct reality

1. Read `progress/STATE.md` and the last three entries of `progress/LOG.md`.
2. Ask him **one** question: *"How many days, and is the project in the state STATE.md describes?"*
3. If the gap is longer than a week, also check `git log` in the Unity project — what he actually committed is more reliable than what he remembers.

## Step 2 — Pick the re-entry mode

| Gap | Mode |
|---|---|
| **1–3 days** | Absorb it. Buffer days exist for this. Resume at the current day, no re-plan. Just re-orient him for 2 minutes on where he left off. |
| **4–10 days** | Slide the calendar. Same day number, new dates. Then run a **re-entry session** (below) before resuming normal content. |
| **11–30 days** | Slide the calendar and **re-scope**. Look at the remaining milestones and cut, in this order: stretch goals → the second and third enemy types → the journal UI → the third ending. Never cut the branching-choice pillar or the ship day. |
| **Over 30 days, or "I've lost the thread"** | Offer a restart at the current *milestone* boundary, not at Day 1. His code and his understanding both still exist. Rebuilding the last milestone deliberately is often faster and more motivating than archaeology. |

Tell him which mode you picked and why. Let him override.

## Step 3 — The re-entry session (any gap over 3 days)

Do not resume the syllabus cold. Spend one full session doing this instead — it is not lost time, it is the thing that makes the next twenty sessions possible:

1. **Open the project and press Play.** Whatever happens, happens. Just look at it.
2. **He gives you a two-minute tour** of the code he wrote, from memory, in his own words. Where he goes vague is exactly where you should spend the next ten minutes.
3. **Fix or park everything `STATE.md` lists as broken.** Enter the session at a green state or an honestly-labelled red one.
4. **One small, satisfying win.** Add something visible in 20 minutes. Momentum is regained through the eyes, not through reading.
5. **Rewrite `STATE.md` from scratch** to describe what is actually true now.

## Step 4 — Re-plan the calendar

Update `progress/STATE.md`:

```
day: <N>
date_of_day_1: 2026-09-10
adjusted_start: <new anchor date>
projected_end: <recomputed>
days_missed_total: <running count>
scope_cuts: <list, if any>
```

Do **not** rewrite the milestone files to compress content. Slide dates, cut scope explicitly, or split a day into two — but keep the briefs intact so the sequence stays coherent.

## Step 5 — Ask one honest question

*"Was it time, or was it that something in the project stopped being fun?"*

Then actually act on the answer:

- **Time** → nothing is wrong with the plan. Slide it. Consider whether 1 hour should become 40 minutes for a while; a shorter session he actually does beats an hour he skips.
- **Stuck on something** → run `/unstuck` on it now, this session. That unresolved thing is a wall between him and the project and it will not get smaller.
- **Bored** → the sequence is wrong for him, not the project. Look ahead and pull something exciting forward — combat feel, art direction, the first real dialogue branch — and push the plumbing back. Tell him you're doing it.
- **Overwhelmed** → cut scope out loud and in writing, right now. Half of Hearthfall shipped beats all of it abandoned, and he needs to hear that as a decision rather than a concession.

## Never

- Do not guilt him, even gently, even as a joke.
- Do not suggest starting the curriculum over from Day 1. Ever.
- Do not silently compress the plan to hide the gap. Recalculated dates are honest; a plan that pretends is a plan he'll stop trusting.
