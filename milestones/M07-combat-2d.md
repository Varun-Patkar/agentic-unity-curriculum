# M07 · Combat, 2D Edition

**Days 50–56 · 29 Oct – 4 Nov 2026 · project: `Hollowbrook`**

> Combat supports the mystery, so this week is deliberately small: one improvised melee attack, one dodge, one creature archetype with two data-tuned variants. Small is the point.
>
> Everything that decides *what happens* goes in Core — health, damage, timings, state transitions. Everything that decides *how it feels* goes in Unity. That split is what lets M12 rebuild this in 3D over the same rules file rather than starting again.

**You end holding:** one attack that connects, one dodge that saves you, and one readable creature that fights back in two tuned variants.

---

## Day 50 — Combat resolution in Core: health, damage, and combat states as data
**Thu 29 Oct · 60 min**

**Objective:** A combat model in Core that can resolve a full exchange — windup, hit, damage, recovery, death — with no Unity in the room.

**Why:** On Day 71 a 3D project imports this assembly unchanged. If a single timing number lives in a MonoBehaviour today, you re-tune the entire game in December.

### Concepts (10 min)
- **Combat is a state machine plus arithmetic.** `Idle · Windup · Active · Recovery · Dodging · Dead`. Every rule in this milestone is a transition or a subtraction.
- **Timings are data, not code.** Windup 0.15s, active 0.10s, recovery 0.25s — fields on an `AttackDefinition`, loaded from JSON like your dialogue. Hardcoded `yield return new WaitForSeconds(0.15f)` is the trap, and it is the single most common way combat becomes untunable.
- **Damage resolution is a pure function.** `Resolve(attacker, defender, attack) → DamageResult`. No mutation inside, no events raised inside. The caller applies the result. Pure functions are trivially testable and trivially portable.
- **Core owns time via an explicit tick.** `combat.Tick(deltaTime)`. Unity calls it from `Update`. A test calls it in a loop with `1/60f`. Same code path, and you can simulate a fight in a millisecond.
- **Beginner trap:** putting `Health` on a MonoBehaviour "because it's on the object". Health is a rule. The MonoBehaviour holds a reference to it and draws a bar.

### Build (40 min)
1. `Core/Combat/CombatState.cs` — the enum above.
2. `Core/Combat/Health.cs` — `Current`, `Max`, `IsDead`, `TakeDamage(int)`, `Heal(int)`. Clamp both ends.
3. `Core/Combat/AttackDefinition.cs` — id, damage, `WindupSeconds`, `ActiveSeconds`, `RecoverySeconds`, knockback force, hitbox offset and size. Pure data, no behaviour.
4. `Core/Combat/CombatResolver.cs` — the pure function. Takes attacker stats, defender stats, and the attack; returns a `DamageResult` (damage dealt, was dodged, killed).
5. `Core/Combat/Combatant.cs` — health, current state, facing, and the current attack. This is what Alex and the creature both own.
6. Author `attacks.json` with exactly one player entry: `improvised_swing`. Load it through your existing JSON pipeline.
7. Author one creature attack definition. It uses the same timing model; the two variants tune data, not move sets.
8. Tests: damage reduces health · lethal damage sets `IsDead` · dodge can return zero damage · resolving against a dead defender does nothing · timings deserialize correctly.

### Acceptance criteria
- [ ] `Hollowbrook.Core` still has zero `using UnityEngine` — check the file, don't assume
- [ ] All attack timings live in JSON, not in C#
- [ ] `CombatResolver` is a pure function with no side effects
- [ ] A full exchange can be simulated in a test with no Play Mode
- [ ] Six or more tests cover damage, death, dodge resolution, and timing data

### Failure modes
- **Timings hardcoded in a coroutine** → the exact thing this day exists to prevent. Fix it now while there is one attack.
- **`Resolve` raises events itself** → it's no longer pure and no longer testable in isolation. Return the result, let the caller publish.
- **Health goes negative and the UI bar inverts** → clamp in `TakeDamage`, not in the view.

**Stretch:** Add a test-data builder for attacks so timing edge cases stay readable.

**Commit:** `feat(core): combat state, health, damage, and attack data`

---

## Day 51 — The attack: windup, active, recovery, and input buffering
**Fri 30 Oct · 60 min**

**Objective:** Press attack, watch the character wind up, swing, and recover — driven entirely by a Core state machine you can step through in a test.

**Why:** Three-phase attacks are why fights have rhythm. One-frame instant attacks are why bad combat feels like clicking a spreadsheet.

### Concepts (10 min)
- **The three phases have jobs.** Windup telegraphs (fairness), active is the only window that can hurt anyone (commitment), recovery is your punishment for missing (risk). Cut any one and combat stops being a conversation.
- **Commitment is the design.** Once windup starts you are in it. If the player can cancel freely, there is no cost to attacking and no reason to ever dodge.
- **Input buffering** — accept an attack press up to ~0.2s before the current attack's recovery ends, and fire it the moment you're free. Without it the game feels unresponsive and the player blames themselves. With it, they feel skilled. This is the highest ratio of "feel" to "lines of code" in the milestone.
- **Core owns the timer.** `Tick(deltaTime)` advances a phase timer and transitions state. Not a coroutine. A coroutine can't be unit-tested, can't be replayed deterministically, and can't come with you to 3D.
- **Coroutines still have a place** — they're fine for pure presentation (fade a sprite, play a sound after a beat). Just never for rules.
- **Beginner trap:** driving animation from the state machine *and* the state machine from animation events. Pick one direction. Core decides; animation follows.

### Build (40 min)
1. `Core/Combat/CombatMachine.cs` — holds a `Combatant`, exposes `TryStartAttack(AttackDefinition)`, `Tick(float)`, and `CurrentState`/`PhaseProgress` for the view.
2. `TryStartAttack` starts only from `Idle`; during late recovery it stores one buffered request. There is no attack resource or alternate attack.
3. Phase timer: windup elapses → `Active` · active elapses → `Recovery` · recovery elapses → `Idle`. Raise `AttackPhaseChanged` on each transition.
4. Buffering: `BufferedInputSeconds` on the machine. A press during recovery stores a timestamp; on reaching `Idle`, if the buffer is fresh, start immediately.
5. `Unity/Combat/PlayerCombat.cs` — subscribes to the Attack action from your `Gameplay` map, calls `TryStartAttack`, calls `Tick(Time.deltaTime)` in `Update`, and nothing else.
6. Visualise without art: tint the player sprite per phase (yellow windup, white active, grey recovery) so you can *see* the machine running. Real animation comes later.
7. Lock movement or scale it down during windup and active — commitment must be legible in the body, not just the rules.
8. Tests: phases advance in order at the right times · a press during the buffer window fires on exit · an early press is dropped · repeated presses never queue more than one attack.

### Acceptance criteria
- [ ] Attack phases are driven by `Tick`, not a coroutine
- [ ] Sprite visibly changes colour through all three phases
- [ ] Exactly one player attack definition exists
- [ ] A press inside the buffer window fires; one outside it is dropped
- [ ] Movement is restricted during the attack
- [ ] Tests step the machine through a full attack deterministically

### Failure modes
- **Attack feels laggy** → windup too long, or no buffering. Try 0.12s windup and a 0.2s buffer before touching anything else.
- **Spamming attack locks the character** → you're queuing every press instead of keeping the most recent one. Store one timestamp, not a list.
- **Phases drift over time** → you're resetting the timer instead of carrying the remainder across the transition.

**Stretch:** Expose the buffer window in the debug overlay and tune it against real input.

**Commit:** `feat: three-phase attack with input buffering`

---

## Day 52 — Hitboxes and hurtboxes: the difference, and why it matters
**Sat 31 Oct · 60 min**

**Objective:** Your swing actually connects with something, once, and takes health off it.

**Why:** This is the day combat stops being a state machine with a colour change and starts being a fight.

### Concepts (10 min)
- **Hitbox ≠ hurtbox.** A **hitbox** is *dangerous* — it belongs to the attacker, exists only during the active phase, and hurts what it overlaps. A **hurtbox** is *vulnerable* — it belongs to the defender and exists almost always. Conflating them is the classic beginner error and it produces enemies that damage you by standing near you.
- **Layers do the filtering, not code.** `PlayerHitbox` collides with `EnemyHurtbox` and nothing else. You built the collision matrix on Day 19; this is what it was for. Filtering with `CompareTag` inside `OnTriggerEnter2D` works and is slower and worse.
- **Trigger colliders, enabled by phase.** The hitbox collider is disabled in windup and recovery, enabled in active. Unity fires `OnTriggerEnter2D` when a trigger is enabled inside an overlap — which is exactly the behaviour you want here.
- **One hit per swing.** Track a `HashSet<Combatant>` of things already hit this swing, cleared on entering windup. Without it, a target that lingers in the hitbox takes damage every frame and dies instantly.
- **Facing determines placement.** Your controller already tracks facing from Day 15 — offset the hitbox along it. A hitbox that only points right is the bug you'll hit first.
- **Unity's collider is a query, Core does the maths.** `OnTriggerEnter2D` should do one thing: forward "these two combatants touched" into Core.

### Build (40 min)
1. Add a child GameObject `Hitbox` under the player with a `BoxCollider2D`, `Is Trigger` on, on layer `PlayerHitbox`, collider disabled by default.
2. Add a child `Hurtbox` under the player and the enemy prefab with a `BoxCollider2D`, `Is Trigger` on, on the matching hurtbox layer.
3. Verify the collision matrix in **Edit > Project Settings > Physics 2D** (verify this path in Unity 6) — hitbox layers must intersect only opposing hurtbox layers, never each other, never the environment.
4. `Unity/Combat/Hitbox.cs` — holds the owning `Combatant`, subscribes to `AttackPhaseChanged`, enables the collider on `Active` and disables it on exit. Positions itself from the attack definition's offset, rotated by facing.
5. `Unity/Combat/Hurtbox.cs` — exposes its owning `Combatant` so the hitbox can find the target through `GetComponent`.
6. On `OnTriggerEnter2D`: if the target is already in the swing's hit set, return. Otherwise call `CombatResolver.Resolve`, apply the result, add to the set, publish a `DamageDealt` event on the Core bus.
7. Clear the hit set on entering windup, so each new swing can hit once.
8. Put a placeholder enemy in the scene with health and a hurtbox. Hit it. Watch the health value fall in the Inspector, once per swing.

### Acceptance criteria
- [ ] Hitboxes and hurtboxes are separate objects on separate layers
- [ ] The hitbox collider is only enabled during the active phase
- [ ] The hitbox flips with facing direction
- [ ] Each swing damages a given target exactly once
- [ ] Damage resolution happens in Core; Unity only forwards the collision
- [ ] Attacking into empty air completes the same committed timing and does nothing else

### Failure modes
- **Nothing connects** → one of: collider isn't a trigger, layers don't intersect in the matrix, or neither object has a Rigidbody2D. Trigger callbacks need a Rigidbody2D on at least one side.
- **The target dies in one swing** → no hit set, so you're dealing damage every physics frame the collider overlaps.
- **You damage yourself** → your own hitbox and hurtbox layers intersect in the matrix. Uncheck it.
- **Hits register in windup** → the collider was already enabled in the prefab. Disable it in the prefab, not just at runtime.

**Stretch:** Draw the hitbox with `OnDrawGizmos` — green when disabled, red when active. Twenty lines, and it turns every future hitbox bug into a five-second diagnosis.

**Commit:** `feat: hitbox and hurtbox collision with one-hit-per-swing`

---

## Day 53 — The dodge, invincibility frames, and commitment
**Sun 1 Nov · 60 min**

**Objective:** A dodge that moves you, makes you briefly untouchable, and cannot be taken back.

**Why:** The dodge is what makes combat a skill instead of a damage race. It's the single mechanic that decides whether your fights are interesting.

### Concepts (10 min)
- **I-frames are a window, not the whole dodge.** A 0.4s dodge might have i-frames from 0.05s to 0.25s. The startup and the tail are vulnerable — that's what makes timing matter. Full-duration invincibility means dodging is always correct and never interesting.
- **Invincibility is a Core flag, checked in the resolver.** Not a disabled collider. If you disable the hurtbox you can't distinguish "dodged" from "missed", and you lose the dodge-success feedback that makes the mechanic feel good.
- **Commitment: a dodge cannot be cancelled.** You're locked in for the duration. The vulnerable startup and recovery tail are the price of a bad read.
- **Decide the cancel rules explicitly and write them down.** Recommended: **you may dodge-cancel attack recovery, but not windup or active.** That gives skilled players an escape hatch without removing the cost of committing to a swing. Whatever you pick, write it in `CONVENTIONS.md` — this is a rule you will second-guess in December.
- **Beginner trap:** implementing the dodge as a velocity change in `Update` and wondering why distance varies with framerate. Movement goes through the Rigidbody2D, and duration comes from Core.

### Build (40 min)
1. `Core/Combat/DodgeDefinition.cs` — duration, distance, recovery, `IFrameStart`, `IFrameEnd`. Data, in JSON, next to your attack.
2. Add `Dodging` handling to `CombatMachine`: `TryDodge(direction)` checks state, enters `Dodging`, and runs the phase timer. It cannot restart until dodge recovery completes.
3. Add `IsInvincible` to `Combatant`, computed from elapsed dodge time against the i-frame window. `CombatResolver` returns `WasDodged` and zero damage when it's true.
4. Implement the cancel rules from your concept decision — `TryDodge` succeeds from `Idle` and from attack `Recovery`, fails from `Windup` and `Active`.
5. `Unity/Combat/PlayerCombat.cs` — bind the existing Dodge action from your `Gameplay` map. Direction is the current move input, or facing if there is no input.
6. Movement: on entering `Dodging`, apply the displacement over the duration via the Rigidbody2D. Interpolate rather than teleport, or you'll pass through walls.
7. Feedback: tint the sprite or drop its alpha during the i-frame window specifically — you must be able to *see* when you're actually invincible while tuning.
8. Tests: damage during i-frames returns `WasDodged` · damage in startup still lands · dodge cannot start during windup · repeated dodge input during dodge/recovery is ignored.

### Acceptance criteria
- [ ] Dodge duration, distance, recovery, and i-frame window all live in JSON
- [ ] I-frames cover only part of the dodge and this is visible on screen
- [ ] A dodge in progress cannot be cancelled by any input
- [ ] The cancel rules are implemented and written in `CONVENTIONS.md`
- [ ] Dodge cannot be repeated until its recovery completes
- [ ] Tests cover dodged, not-dodged, and refused-during-commitment

### Failure modes
- **Dodge distance varies with framerate** → you're applying force per frame instead of interpolating over a fixed duration.
- **Dodging through walls** → move via the Rigidbody2D and keep collision on, or sweep the path. Never set `transform.position`.
- **Dodge feels useless** → i-frame window too small or starts too late. Widen it before you shorten the enemy's windup.
- **Dodge feels mandatory-spammable** → recovery is too short or enemy tracking is too forgiving.

**Stretch:** Draw the i-frame window on the same timeline overlay as windup, active, and recovery.

**Commit:** `feat: dodge with invincibility frames and commitment`

---

## Day 54 — One creature: a state machine with two tuned variants
**Mon 2 Nov · 60 min**

**Objective:** A creature on Old Mine Road that notices Alex, closes distance, telegraphs, attacks, and can be beaten by reading and dodging it.

**Why:** An enemy is the first thing in this project that acts on its own. It's also the first thing that will misbehave in a way you can't step through — unless you build it so you can see its mind.

### Concepts (10 min)
- **AI here is one explicit state machine, no more.** `Idle · Approach · Windup · Attack · Recover · Reposition · Dead`. Behaviour trees, utility AI, and GOAP are all real and all wrong for one creature.
- **The AI decides; the combat machine executes.** The creature brain calls `TryStartAttack` exactly like your input handler does. Same machine, same rules, same tests.
- **Telegraphing is fairness.** The creature's windup must be long enough to react to — start around 0.5s, roughly triple the player's. If the player can't see the attack coming, dodging is guesswork and the whole milestone is wasted.
- **Ranges, plural.** Aggro range (notice you), preferred range (hover here), attack range (swing). Three floats, in data, and they define the enemy's personality entirely.
- **Reposition is what makes it feel alive.** After attacking, back off briefly before closing again. Without it the creature welds itself to Alex and the fight becomes a mash.
- **You cannot debug what you cannot see.** A floating label showing the current state and timer turns "why did it do that" from a twenty-minute mystery into a glance.

### Build (40 min)
1. `Core/Combat/AI/CreatureBrain.cs` — pure C#. Input: distance to target, own state, elapsed timer. Output: an intent (`MoveToward`, `MoveAway`, `Attack`, `Wait`). No Unity types.
2. `Core/Combat/AI/CreatureDefinition.cs` — health, movement speed, aggro range, preferred range, attack range, reaction delay, attack id. Author exactly two data variants: a slower road creature and a tougher mine creature using the same brain, visuals, and attack shape.
3. `Unity/Combat/CreatureController.cs` — reads world distance, ticks the brain, converts intent into Rigidbody2D movement and `TryStartAttack` calls. That's the entire Unity side.
4. Aggro: below aggro range, transition to `Approach` — but with a reaction delay, so it doesn't snap the instant you cross a line.
5. Taking damage reduces health and triggers view feedback, but does not introduce a second crowd-control system.
6. Death: on `IsDead`, disable colliders and the brain, play a placeholder fade, and destroy after a delay. Do not destroy inside the damage callback.
7. Debug label: current state and timer, plus wire circles for the three ranges. Guard Editor-only APIs.
8. **Fight both variants.** They should feel different through data while remaining unmistakably the same archetype.

### Acceptance criteria
- [ ] Creature brain is pure C# in Core with no Unity references
- [ ] All seven states are reachable and the current one is visible in the Scene view
- [ ] Aggro, preferred, and attack ranges are drawn as gizmos
- [ ] The enemy telegraphs long enough that dodging is reliably possible
- [ ] Two data variants use the same brain, visual archetype, and attack shape
- [ ] You beat it by reading and dodging, not by trading hits

### Failure modes
- **Creature jitters between two states** → your range thresholds overlap. Add hysteresis: leave a state at a slightly different distance than you enter it.
- **Creature glued to Alex** → no reposition state, or preferred range equals attack range.
- **`Handles.Label` breaks the build** → Editor-only API. `#if UNITY_EDITOR` around it.
- **Null reference on kill** → you destroyed the GameObject mid-collision-callback. Defer it.
- **Fight is unwinnable** → almost always the telegraph. Double the windup and re-test before touching damage numbers.

**Stretch:** Tune the mine variant using only health, movement, and timing data. Do not add another behaviour or attack.

**Commit:** `feat: creature archetype with two data variants`

---

## Day 55 — Hit feel: hitstop, knockback, flash, sound
**Tue 3 Nov · 60 min**

**Objective:** The same fight as yesterday, except now landing a hit is satisfying.

**Why:** This is the highest value-per-minute day of the entire milestone. Nothing you did on Days 50–54 changes today, and the game will feel three times better by the end of the hour.

### Concepts (10 min)
- **Hitstop is the big one.** Freeze both parties for 0.05–0.08s on impact. It reads as weight and impact, it costs almost nothing, and every fighting game and action game you've enjoyed does it. Scale it with damage.
- **Freeze the combatants, not the world.** `Time.timeScale = 0` is the lazy version — it stops your UI, your particles, and everything else. Better: skip `Tick` on the two involved combatants for the duration.
- **Layer the feedback.** Hitstop + knockback + white flash + sound + screenshake fire *together* on one event. Individually each is subtle; stacked, they're a punch. Drive them all from your existing `DamageDealt` event.
- **Pitch variation on sound.** The same clip at pitch `Random.Range(0.9f, 1.1f)` stops sounding like a machine gun after three hits. Two lines, enormous difference.
- **Screenshake must be tiny.** You already have Cinemachine Impulse from M02. Set the amplitude at what feels right, then halve it. Then halve it again for hits you *take*, or the player can't see to recover.
- **The rule for today: build it all, then cut it in half.** Every one of these effects is more tasteful at 50% of the value that felt right while you were tuning it, because you tune while staring at one hit in isolation.

### Build (40 min)
1. `Unity/Combat/HitFeedback.cs` — one component subscribing to `DamageDealt` on the Core bus, orchestrating everything below. One place to tune, one place to disable.
2. **Hitstop:** on hit, set a freeze timer on both combatants; their controllers skip `Tick` and zero velocity until it expires. Duration scaled by damage, capped around 0.1s.
3. **Knockback:** apply the impulse from the `AttackDefinition` along the attacker's facing, via the Rigidbody2D, *after* hitstop ends. Simultaneous knockback and freeze cancel each other visually.
4. **White flash:** swap the sprite's material to an unlit white one for ~0.08s, or drive a `_FlashAmount` property if your URP shader has one (verify the property name in your sprite shader before wiring it).
5. **Sound:** one impact clip, played with randomised pitch. A second, duller clip for a hit you take, so you can tell what happened without looking.
6. **Screenshake:** fire your existing Cinemachine Impulse Source. Different amplitude for dealing versus taking.
7. **Hit spark:** a small particle burst at the contact point, or a two-frame sprite. Skip damage numbers — they're wrong for this game's tone.
8. **Cut everything in half.** Then fight the creature for five minutes and tune only what still bothers you.

### Acceptance criteria
- [ ] Hitstop fires on every landed hit and freezes only the combatants
- [ ] Knockback applies after hitstop, not during
- [ ] Sprites flash white on taking damage
- [ ] Impact sound plays with randomised pitch
- [ ] Screenshake is present and smaller for hits you take
- [ ] All of it is driven from one event and one component

### Failure modes
- **The whole game stutters** → you used `Time.timeScale`. Freeze the combatants instead.
- **Effects feel cheap and loud** → they're all at 100%. Halve them, as instructed, and again if needed.
- **Flash never returns to normal** → you cached the wrong original material, or an early return skipped the restore. Restore in a `finally` or on a hard timer.
- **Knockback launches the creature across the room** → the impulse is a force, not a velocity, and mass matters. Tune against the actual Rigidbody2D mass.
- **Nothing feels different** → check the event is actually firing. It usually is, and the values are just too small to perceive.

**Stretch:** A directional hit-spark that orients to the swing angle. Purely cosmetic, disproportionately good, and it's ten minutes.

**Commit:** `feat: hitstop, knockback, flash, and impact feedback`

---

## Day 56 — BUFFER
**Wed 4 Nov**

- **Catch up.** If any of Days 50–55 spilled, this is where it lands.
- **Tune.** Fight both creature variants. Adjust one number at a time and write down what changed.
- **Integrate two or three short encounters** across Old Mine Road and Mercer Mine. Tie each to exploration or an Eli clue; this is the entire combat budget.
- **Content pass:** make sure combat never blocks the four central choices or creates a second enemy type by accident.
- **Rest.**

### Milestone review

Run `/review`. Ask specifically whether any combat *rule* has leaked into a MonoBehaviour — a timing, a damage number, a state transition. Day 71 imports Core unchanged, and anything that leaked is work you'll do twice.

### Where you are

You have a fight. It has a windup you can read, a dodge that rewards timing, one creature whose mind you can see in the Scene view, and a hit that lands with weight. Two data variants cover every encounter.

That's the whole trick of this milestone: combat is small because the game is about Eli, Hollowbrook, and the bargain beneath the mine. It feels good without becoming a progression system.

Tomorrow: save/load and the game shell — the unglamorous week that turns a project into something a stranger can actually start, play, and quit.

**Commit:** `docs: M07 complete — 2d combat`
