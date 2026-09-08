---
description: 'Critique what I built — architecture, Unity idiom, and whether it survives the 3D port.'
mode: agent
---

# Review

Tear apart what he built. He is a senior engineer; treat the review at that level. Vague encouragement is worse than useless here — it costs him the chance to fix something cheap while it is still cheap.

## Scope

Ask first, unless he already said: **today's work**, **this milestone**, or **the whole project**? Default to the milestone if he ran this at a milestone boundary.

Then read the code. Actually read it — do not review from memory of the conversation.

## The five lenses

Go through all five. Report findings grouped by severity at the end, not as you go.

### 1. Does it survive the port?
**This is the highest-value lens in the entire curriculum.** Milestone 10 rebuilds the presentation layer in 3D against the *same* core. Every piece of game logic that has leaked into a MonoBehaviour is a thing he pays for twice.

- Does anything in `Hearthfall.Core` reference `UnityEngine`? That's a hard failure.
- Is there game logic — rules, state transitions, conditions, resolution — sitting in a MonoBehaviour that should be in Core?
- Could he run this logic in a console app and get the same answers? If not, why not?
- Are the Unity classes doing three legitimate jobs only: **read input**, **render state**, **forward events**?

### 2. Is it idiomatic Unity?
He is fluent in C# and will reach for backend patterns that are wrong here. Common ones:

- Reinventing something Unity gives free (Cinemachine, the Input System, the Animator, `ScriptableObject` for config, the Job System).
- Over-abstracting: interfaces and DI containers where a `[SerializeField]` reference and a plain class would do.
- Under-abstracting: 400-line `PlayerController` doing movement, combat, input, and animation.
- `GetComponent`, `Find`, `FindObjectOfType`, or allocations inside `Update`.
- `Update` where `FixedUpdate` (physics) or an event (state change) belongs.
- Singletons everywhere because DI felt unavailable.

### 3. Is it correct?
- Null-reference risk from unassigned Inspector references.
- Frame-rate dependence — anything per-frame that isn't multiplied by `Time.deltaTime`.
- Physics done outside `FixedUpdate`.
- State machines that can reach a state with no exit.
- Save data that will not survive a schema change. (Ask: what happens when you add a field next week?)

### 4. Does it feel good?
Only from M02 onward, and it matters more than he thinks. Game feel is not polish, it is the product.

- Input latency: does the character respond on the frame the button goes down?
- Are transitions instant where they should be, and eased where they should be?
- Is there feedback on every player action — visual, audio, or motion?
- Would he enjoy holding this for ten minutes, or is he just verifying it functions?

### 5. Is it maintainable at day 112?
- Any file over ~300 lines is a smell. Name it.
- Naming: does a class name say what it does, or what it is made of?
- Is content (dialogue, quests, stats) data he can edit without recompiling? By M05 it must be.
- Is there a test he could have written cheaply and didn't?

## Output format

```
## Verdict
<one honest sentence>

## Must fix (breaks something, or will cost you at M10)
- <finding> — <why it matters> — <the fix, in one line>

## Should fix (cheap now, expensive later)
- ...

## Consider (taste, not correctness)
- ...

## Genuinely good
- <specific things he did right — only real ones, and say why they're right>

## Next action
<the single highest-value change, small enough to do in 20 minutes>
```

## Rules

- **Be specific.** "Extract the state machine from `PlayerController` into Core" — not "consider improving separation of concerns".
- **Ground everything in a consequence.** Every finding names what it will cost and when.
- **Praise only what is genuinely good, and say why.** He will discount all of your praise if any of it is padding.
- **Do not fix anything.** This prompt produces findings. He fixes them, or asks you to help with a specific one.
- **Cap "Must fix" at three items.** If there are more, the top three are the ones he can actually do.
