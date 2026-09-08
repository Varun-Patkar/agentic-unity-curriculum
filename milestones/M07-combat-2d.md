# M07 · Combat, 2D Edition

**Days 50–56 · 29 Oct – 4 Nov 2026 · project: `Hearthfall`**

> Combat is the part you said you cared least about, so this week is deliberately small: one attack, one dodge, one enemy. Small is the point. A three-button fight that feels good beats a six-button fight that feels like typing.
>
> Everything that decides *what happens* goes in Core — damage, stamina, timings, state transitions. Everything that decides *how it feels* goes in Unity. That split is what lets M12 rebuild this in 3D over the same rules file rather than starting again.

**You end holding:** an attack that connects, a dodge that saves you, and an enemy that fights back.

---

## Day 50 — Combat resolution in Core: damage, stamina, and combat states as data
**Thu 29 Oct · 60 min**

**Objective:** A combat model in Core that can resolve a full exchange — swing, hit, damage, stamina drain, death — with no Unity in the room.

**Why:** On Day 71 a 3D project imports this assembly unchanged. If a single timing number lives in a MonoBehaviour today, you re-tune the entire game in December.

### Concepts (10 min)
- **Combat is a state machine plus arithmetic.** `Idle · Windup · Active · Recovery · Dodging · Staggered · Dead`. Every rule in this milestone is a transition or a subtraction.
- **Timings are data, not code.** Windup 0.15s, active 0.10s, recovery 0.25s — fields on an `AttackDefinition`, loaded from JSON like your dialogue. Hardcoded `yield return new WaitForSeconds(0.15f)` is the trap, and it is the single most common way combat becomes untunable.
- **Damage resolution is a pure function.** `Resolve(attacker, defender, attack) → DamageResult`. No mutation inside, no events raised inside. The caller applies the result. Pure functions are trivially testable and trivially portable.
- **Stamina is the whole difficulty dial.** Attack costs, dodge costs, regen rate, and a regen delay after spending. Four numbers control how the game feels; keep them together and keep them in data.
- **Core owns time via an explicit tick.** `combat.Tick(deltaTime)`. Unity calls it from `Update`. A test calls it in a loop with `1/60f`. Same code path, and you can simulate a fight in a millisecond.
- **Beginner trap:** putting `Health` on a MonoBehaviour "because it's on the object". Health is a rule. The MonoBehaviour holds a reference to it and draws a bar.

### Build (40 min)
1. `Core/Combat/CombatState.cs` — the enum above.
2. `Core/Combat/Health.cs` — `Current`, `Max`, `IsDead`, `TakeDamage(int)`, `Heal(int)`. Clamp both ends.
3. `Core/Combat/Stamina.cs` — `Current`, `Max`, `RegenPerSecond`, `RegenDelay`, `TrySpend(float)` returning bool, `Tick(float)`.
4. `Core/Combat/AttackDefinition.cs` — id, damage, stamina cost, `WindupSeconds`, `ActiveSeconds`, `RecoverySeconds`, knockback force, hitbox offset and size. Pure data, no behaviour.
5. `Core/Combat/CombatResolver.cs` — the pure function. Takes attacker stats, defender stats, and the attack; returns a `DamageResult` (damage dealt, was blocked, was dodged, killed).
6. `Core/Combat/Combatant.cs` — health, stamina, current state, facing, and the current attack. This is what both the player and the bandit will own.
7. Author `attacks.json` with one entry: `light_slash`. Load it through your existing JSON pipeline.
8. Tests: damage reduces health · lethal damage sets `IsDead` · stamina blocks an attack you can't afford · stamina regens after the delay, not during it · resolving against a dead defender does nothing.

### Acceptance criteria
- [ ] `Hearthfall.Core` still has zero `using UnityEngine` — check the file, don't assume
- [ ] All attack timings live in JSON, not in C#
- [ ] `CombatResolver` is a pure function with no side effects
- [ ] Stamina has a regen delay separate from its regen rate
- [ ] A full exchange can be simulated in a test with no Play Mode
- [ ] Six or more tests cover damage, death, and stamina

### Failure modes
- **Timings hardcoded in a coroutine** → the exact thing this day exists to prevent. Fix it now while there is one attack.
- **`Resolve` raises events itself** → it's no longer pure and no longer testable in isolation. Return the result, let the caller publish.
- **Health goes negative and the UI bar inverts** → clamp in `TakeDamage`, not in the view.

**Stretch:** Add a `poise` value — damage above a threshold staggers, below it doesn't. One field, and it's what makes heavy attacks feel heavy on Day 55.

**Commit:** `feat(core): combat state, health, stamina, and attack data`

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
2. `TryStartAttack` returns false if not in `Idle` or `Recovery`-with-buffer, or if stamina is insufficient. Spend stamina on entering windup, not on the button press.
3. Phase timer: windup elapses → `Active` · active elapses → `Recovery` · recovery elapses → `Idle`. Raise `AttackPhaseChanged` on each transition.
4. Buffering: `BufferedInputSeconds` on the machine. A press during recovery stores a timestamp; on reaching `Idle`, if the buffer is fresh, start immediately.
5. `Unity/Combat/PlayerCombat.cs` — subscribes to the Attack action from your `Gameplay` map, calls `TryStartAttack`, calls `Tick(Time.deltaTime)` in `Update`, and nothing else.
6. Visualise without art: tint the player sprite per phase (yellow windup, white active, grey recovery) so you can *see* the machine running. Real animation comes later.
7. Lock movement or scale it down during windup and active — commitment must be legible in the body, not just the rules.
8. Tests: attack can't start without stamina · phases advance in order at the right times · a press during recovery fires on exit · a press 1s before exit does not.

### Acceptance criteria
- [ ] Attack phases are driven by `Tick`, not a coroutine
- [ ] Sprite visibly changes colour through all three phases
- [ ] Stamina is spent once, on entering windup
- [ ] A press inside the buffer window fires; one outside it is dropped
- [ ] Movement is restricted during the attack
- [ ] Tests step the machine through a full attack deterministically

### Failure modes
- **Attack feels laggy** → windup too long, or no buffering. Try 0.12s windup and a 0.2s buffer before touching anything else.
- **Spamming attack locks the character** → you're queuing every press instead of keeping the most recent one. Store one timestamp, not a list.
- **Phases drift over time** → you're resetting the timer instead of carrying the remainder across the transition.

**Stretch:** A two-hit combo — a press during recovery of hit one chains to a second attack definition instead of returning to idle. Same buffer, one extra branch.

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
7. Clear the hit set on entering windup. Not on exiting active — buffered combos will bite you.
8. Put a placeholder enemy in the scene with health and a hurtbox. Hit it. Watch the health value fall in the Inspector, once per swing.

### Acceptance criteria
- [ ] Hitboxes and hurtboxes are separate objects on separate layers
- [ ] The hitbox collider is only enabled during the active phase
- [ ] The hitbox flips with facing direction
- [ ] Each swing damages a given target exactly once
- [ ] Damage resolution happens in Core; Unity only forwards the collision
- [ ] Attacking into empty air costs stamina and does nothing else

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

**Objective:** A dodge that costs stamina, moves you, makes you briefly untouchable, and cannot be taken back.

**Why:** The dodge is what makes combat a skill instead of a damage race. It's the single mechanic that decides whether your fights are interesting.

### Concepts (10 min)
- **I-frames are a window, not the whole dodge.** A 0.4s dodge might have i-frames from 0.05s to 0.25s. The startup and the tail are vulnerable — that's what makes timing matter. Full-duration invincibility means dodging is always correct and never interesting.
- **Invincibility is a Core flag, checked in the resolver.** Not a disabled collider. If you disable the hurtbox you can't distinguish "dodged" from "missed", and you lose the dodge-success feedback that makes the mechanic feel good.
- **Commitment: a dodge cannot be cancelled.** You're locked in for the duration. The stamina cost plus the recovery tail is the price of a bad read.
- **Decide the cancel rules explicitly and write them down.** Recommended: **you may dodge-cancel attack recovery, but not windup or active.** That gives skilled players an escape hatch without removing the cost of committing to a swing. Whatever you pick, write it in `CONVENTIONS.md` — this is a rule you will second-guess in December.
- **Stamina is the real limiter.** If you can dodge four times in a row you will never learn to read the enemy. Cost it so that two dodges leave you nearly empty.
- **Beginner trap:** implementing the dodge as a velocity change in `Update` and wondering why distance varies with framerate. Movement goes through the Rigidbody2D, and duration comes from Core.

### Build (40 min)
1. `Core/Combat/DodgeDefinition.cs` — duration, stamina cost, distance, `IFrameStart`, `IFrameEnd`. Data, in JSON, next to your attacks.
2. Add `Dodging` handling to `CombatMachine`: `TryDodge(direction)` checks state and stamina, spends, enters `Dodging`, runs the phase timer.
3. Add `IsInvincible` to `Combatant`, computed from elapsed dodge time against the i-frame window. `CombatResolver` returns `WasDodged` and zero damage when it's true.
4. Implement the cancel rules from your concept decision — `TryDodge` succeeds from `Idle` and from attack `Recovery`, fails from `Windup` and `Active`.
5. `Unity/Combat/PlayerCombat.cs` — bind the existing Dodge action from your `Gameplay` map. Direction is the current move input, or facing if there is no input.
6. Movement: on entering `Dodging`, apply the displacement over the duration via the Rigidbody2D. Interpolate rather than teleport, or you'll pass through walls.
7. Feedback: tint the sprite or drop its alpha during the i-frame window specifically — you must be able to *see* when you're actually invincible while tuning.
8. Tests: dodging with no stamina fails · damage during i-frames returns `WasDodged` · damage in the dodge's startup still lands · dodge cannot start during windup.

### Acceptance criteria
- [ ] Dodge duration, cost, and i-frame window all live in JSON
- [ ] I-frames cover only part of the dodge and this is visible on screen
- [ ] A dodge in progress cannot be cancelled by any input
- [ ] The cancel rules are implemented and written in `CONVENTIONS.md`
- [ ] Two dodges leave stamina nearly empty
- [ ] Tests cover dodged, not-dodged, and refused-for-stamina

### Failure modes
- **Dodge distance varies with framerate** → you're applying force per frame instead of interpolating over a fixed duration.
- **Dodging through walls** → move via the Rigidbody2D and keep collision on, or sweep the path. Never set `transform.position`.
- **Dodge feels useless** → i-frame window too small or starts too late. Widen it before you shorten the enemy's windup.
- **Dodge feels mandatory-spammable** → stamina cost too low, or regen too fast.

**Stretch:** A perfect-dodge reward — dodging within the first 0.1s of an enemy's active phase refunds half the stamina. Almost free to add, and it gives skilled play something to chase.

**Commit:** `feat: dodge with invincibility frames and commitment`

---

## Day 54 — Enemy #1: a bandit as a state machine you can debug
**Mon 2 Nov · 60 min**

**Objective:** A bandit on the Wealdrun that notices you, closes distance, telegraphs, swings, and can be beaten by dodging.

**Why:** An enemy is the first thing in this project that acts on its own. It's also the first thing that will misbehave in a way you can't step through — unless you build it so you can see its mind.

### Concepts (10 min)
- **AI here is one explicit state machine, no more.** `Idle · Approach · Windup · Attack · Recover · Reposition · Stagger · Dead`. Behaviour trees, utility AI, and GOAP are all real and all wrong for one bandit.
- **The AI decides; the combat machine executes.** The bandit's brain calls `TryStartAttack` exactly like your input handler does. Same machine, same rules, same tests. This is the payoff for keeping Core engine-free.
- **Telegraphing is fairness.** The enemy's windup must be long enough to react to — start around 0.5s, roughly triple the player's. If the player can't see the swing coming, dodging is a coin flip and the whole milestone is wasted.
- **Ranges, plural.** Aggro range (notice you), preferred range (hover here), attack range (swing). Three floats, in data, and they define the enemy's personality entirely.
- **Reposition is what makes it feel alive.** After attacking, back off briefly before closing again. Without it the bandit welds itself to your face and the fight becomes a mash.
- **You cannot debug what you cannot see.** A floating label showing the current state and timer turns "why did it do that" from a twenty-minute mystery into a glance.

### Build (40 min)
1. `Core/Combat/AI/BanditBrain.cs` — pure C#. Input: distance to target, own state, own stamina, elapsed timer. Output: an intent (`MoveToward`, `MoveAway`, `Attack`, `Wait`). No Unity types; direction is a float or your own `Vec2`.
2. `Core/Combat/AI/EnemyDefinition.cs` — health, aggro range, preferred range, attack range, reaction delay, attack id. JSON, alongside the player's.
3. `Unity/Combat/BanditController.cs` — reads world distance, ticks the brain, converts the intent into Rigidbody2D movement and `TryStartAttack` calls. That's the entire Unity side.
4. Aggro: below aggro range, transition to `Approach` — but with a reaction delay, so it doesn't snap the instant you cross a line.
5. Stagger: on taking damage, interrupt into `Stagger` for a short duration. Interrupting a windup is the reward for punishing a telegraph.
6. Death: on `IsDead`, disable colliders and the brain, play a placeholder fade, and destroy after a delay. Do not destroy inside the damage callback — you'll null-reference the collision that's still resolving.
7. Debug label: `OnDrawGizmos` with `Handles.Label` for the current state, plus wire circles for the three ranges (verify `Handles` needs an Editor-only guard — it does; wrap it in `#if UNITY_EDITOR`).
8. **Fight it.** Ten times. Note whether you win by dodging or by out-trading. If it's out-trading, the windup is too short or your recovery is too forgiving.

### Acceptance criteria
- [ ] Bandit brain is pure C# in Core with no Unity references
- [ ] All eight states are reachable and the current one is visible in the Scene view
- [ ] Aggro, preferred, and attack ranges are drawn as gizmos
- [ ] The enemy telegraphs long enough that dodging is reliably possible
- [ ] Taking damage during windup staggers and cancels the attack
- [ ] You beat it by dodging, not by trading hits

### Failure modes
- **Bandit jitters between two states** → your range thresholds overlap. Add hysteresis: leave a state at a slightly different distance than you enter it.
- **Bandit glued to your face** → no reposition state, or preferred range equals attack range.
- **`Handles.Label` breaks the build** → Editor-only API. `#if UNITY_EDITOR` around it.
- **Null reference on kill** → you destroyed the GameObject mid-collision-callback. Defer it.
- **Fight is unwinnable** → almost always the telegraph. Double the windup and re-test before touching damage numbers.

**Stretch:** A second attack in the bandit's set with a different windup, chosen at random. One extra definition, and suddenly the player has to actually read the animation rather than counting.

**Commit:** `feat: bandit enemy with debuggable state machine`

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
8. **Cut everything in half.** Then fight the bandit for five minutes and tune only what still bothers you.

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
- **Knockback launches the bandit across the room** → the impulse is a force, not a velocity, and mass matters. Tune against the actual Rigidbody2D mass.
- **Nothing feels different** → check the event is actually firing. It usually is, and the values are just too small to perceive.

**Stretch:** A directional hit-spark that orients to the swing angle. Purely cosmetic, disproportionately good, and it's ten minutes.

**Commit:** `feat: hitstop, knockback, flash, and impact feedback`

---

## Day 56 — BUFFER
**Wed 4 Nov**

- **Catch up.** If any of Days 50–55 spilled, this is where it lands.
- **Tune.** Fight the bandit twenty times. Adjust one number at a time and write down what you changed.
- **Place two real encounters** on the Wealdrun from the story bible — the road is where combat lives, and two fights is the entire budget.
- **Story homework:** ten lines of Bran's dialogue were due this milestone. He teaches you to fight, so write him now while combat is fresh in your hands.
- **Rest.**

### Milestone review

Run `/review`. Ask specifically whether any combat *rule* has leaked into a MonoBehaviour — a timing, a damage number, a state transition. Day 71 imports Core unchanged, and anything that leaked is work you'll do twice.

### Where you are

You have a fight. It has a windup you can read, a dodge that rewards timing, an enemy whose mind you can see in the Scene view, and a hit that lands with weight. It took six days and it is entirely built on a rules engine with no engine in it.

That's the whole trick of this milestone: combat is small, and it's small on purpose, because your game is about a letter from your sister and not about a sword. But it feels good, and "small but feels good" is exactly what you aimed at.

Tomorrow: save/load and the game shell — the unglamorous week that turns a project into something a stranger can actually start, play, and quit.

**Commit:** `docs: M07 complete — 2d combat`
