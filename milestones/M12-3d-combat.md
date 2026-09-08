# M12 · 3D Combat — Witcher-lite

**Days 85–91 · 3–9 Dec 2026 · project: `Hearthfall3D`**

> Every rule you need this week already exists and is already tested. Health, stamina, attack timings as data, the pure damage resolver, the combat state machine — all of it shipped in M07 and all of it comes across to `Hearthfall3D` unchanged. You are not building combat this week. You are building a *body* for combat that already works.
>
> That is a far better position than almost anyone building an action game from scratch. It means every hour goes into feel — animation, timing, camera, impact — instead of into arguing with yourself about damage formulas. The second time Core pays off, and it pays off bigger than the first.
>
> Scope stays deliberately tight: light, heavy, dodge, lock-on, three enemies. No parry, no counters, no skill tree, no weapon variety. This is a game about a letter from your sister.

**You end holding:** light, heavy, dodge, stamina, lock-on, and three enemies who make you use all of it.

---

## Day 85 — Light attack, combo windows, and input buffering that feels generous
**Thu 3 Dec · 60 min**

**Objective:** Press attack, watch a rooted, weighted swing land on a dummy — and chain a second swing by pressing again during recovery.

**Why:** The rules from Day 51 run untouched. Today is about hanging an animated body on them without introducing a single frame of input lag.

### Concepts (10 min)
- **Core already owns the timing.** `CombatMachine.Tick(Time.deltaTime)` drives windup → active → recovery exactly as it did in 2D. The Animator is a *listener*, not a source of truth. If you find yourself reading `Animator` state to decide game logic, you've inverted it.
- **Root motion on attacks, in-place on locomotion.** You set this up on Day 82. The attack animation carries the character forward; Core carries the timing. They must agree — if the clip is longer than windup+active+recovery, either retime the clip's speed multiplier or retune the JSON. Retune the JSON.
- **Combo window = a press during recovery.** You already have the buffer. A combo is the buffered input resolving into a *different* attack definition instead of the same one. `light_1 → light_2 → light_3`, then back to idle.
- **Input buffering ~0.2s is the difference between "responsive" and "unfair".** A press 0.15s too early should still land. Players do not perceive their own early presses; they perceive your game dropping inputs.
- **Beginner trap: Exit Time.** Unity ticks `Has Exit Time` on new transitions by default. On an attack transition it means "wait for the current clip to finish before responding" — which is input lag you will spend forty minutes blaming on your own code. Uncheck it and set Transition Duration low (0.05–0.1s) on all attack transitions.
- **Animation events are for presentation only.** Footstep sounds, VFX spawn points, weapon trail on/off. Never "now deal damage" — Core decides that.

### Build (40 min)
1. Import three light-attack clips from Mixamo (or one, repeated with a mirror). Set them Humanoid, root motion enabled, in the Model import tab.
2. Add `light_1`, `light_2`, `light_3` to `attacks.json` in Core — same schema as M07, different timings. Nothing new in C#.
3. In the Animator, add an `Attack` sub-state machine with the three clips. Transitions driven by an `AttackIndex` int and an `Attack` trigger. **Uncheck Has Exit Time on every transition into an attack.**
4. `Unity/Combat/PlayerCombat3D.cs` — subscribes to the Attack action, calls `TryStartAttack`, ticks the machine, sets Animator parameters on `AttackPhaseChanged`. That is the entire file.
5. Extend Core's buffered input to carry a *next attack id*. During recovery of `light_1`, a press queues `light_2`. If the buffer expires, chain resets to `light_1`.
6. Weapon hitbox: a trigger collider on the sword bone, disabled by default, enabled on the `Active` phase exactly as in Day 52. Same one-hit-per-swing `HashSet`.
7. Put a static dummy with a `Combatant` and a hurtbox in the scene. Hit it. Watch health fall in the Inspector, once per swing.
8. Tune the buffer to 0.2s, then spam attack for a minute. If any press feels dropped, it's Exit Time or the buffer, in that order.

### Acceptance criteria
- [ ] Attack timings still come from `attacks.json`; no new numbers in the MonoBehaviour
- [ ] Three-hit chain works when pressing during recovery
- [ ] Chain resets to hit one after the buffer expires
- [ ] A press up to 0.2s early still fires
- [ ] Root motion moves the character forward on the swing
- [ ] Each swing damages the dummy exactly once

### Failure modes
- **Attacks feel a beat late** → Has Exit Time is on, or Transition Duration is 0.25s. Both, usually.
- **Character slides during the swing** → root motion is off on the clip, or Apply Root Motion is off on the Animator.
- **Chain fires all three instantly** → you're queuing every press. One buffered timestamp, most recent wins.
- **Hitbox never triggers** → the sword collider needs a Rigidbody somewhere in the hierarchy (kinematic is fine), and the layer matrix must intersect.

**Stretch:** Give `light_3` a longer recovery and slightly more damage. Costs one JSON edit, and the chain suddenly has a decision in it.

**Commit:** `feat: 3d light attack with combo chain and input buffering`

---

## Day 86 — Heavy attack and stamina: the same Core rules, a new body
**Fri 4 Dec · 60 min**

**Objective:** Hold the attack button for a slower, heavier swing — and run out of stamina doing it.

**Why:** Stamina is the actual combat mechanic. Everything else is decoration on the question "can you afford this swing?"

### Concepts (10 min)
- **`Hold` is an Input System interaction, not your code.** On the Attack action, add a `Hold` interaction (default ~0.4s) alongside the default tap. The callback tells you which interaction fired. You built this as a Day 16 stretch; if you skipped it, today is the day.
- **The stamina numbers are already in Core.** `Stamina` with `RegenPerSecond` and `RegenDelay` shipped on Day 50. The heavy attack's cost is a field on its `AttackDefinition`. **Do not re-declare any of these as serialized fields on the MonoBehaviour** — that is the exact regression this curriculum exists to prevent, and it is the one you'll be tempted by because the Inspector is convenient.
- **The regen delay is what creates rhythm.** Spend, then wait ~1s before regen starts. Without the delay, stamina is a soft suggestion; with it, over-committing has a real cost.
- **Being locked out is the mechanic, not a bug.** When you can't afford a swing, the game should visibly refuse: no animation, a dull sound, a stamina bar flash. Silence reads as a dropped input.
- **Heavy = slow telegraph, big payoff.** Roughly triple the windup, double the damage, triple the cost. The player is committing hard, and the enemy gets a real window to punish.
- **Beginner trap:** driving the stamina bar from `Update` polling. Subscribe to a Core stamina-changed event, or read once per frame in a dedicated view component — but never let the UI own the value.

### Build (40 min)
1. In the Input Actions asset, add a `Hold` interaction to the Attack action. Verify the interaction list in Unity 6's Input System UI before assuming names — this UI has moved between versions.
2. Add `heavy_1` to `attacks.json`: windup ~0.45s, higher damage, higher stamina cost, higher knockback.
3. In `PlayerCombat3D`, branch on the interaction that fired. Tap → light chain. Hold → `TryStartAttack(heavy_1)`.
4. Import a heavy overhead clip, wire it into the Animator alongside the light chain, same Exit Time discipline.
5. `Unity/UI/StaminaBar.cs` — reads `Combatant.Stamina` from Core, fills a URP UI Image. Add a delayed "ghost" fill behind it so spend is legible.
6. Wire the refusal case: `TryStartAttack` returning false plays a short dull thud and flashes the bar red. No animation, no state change.
7. Tune: two heavies should nearly empty the bar. Light attacks should cost enough that mashing runs you dry in about five swings.
8. Fight the dummy for five minutes attacking only when you can afford it. If you never think about stamina, the costs are too low.

### Acceptance criteria
- [ ] Hold produces a heavy attack, tap produces a light one, from one action
- [ ] Every stamina and damage value is read from Core; none are serialized on a MonoBehaviour
- [ ] Regen starts after a delay, not immediately
- [ ] An unaffordable attack is visibly and audibly refused
- [ ] Two heavies nearly empty the bar
- [ ] Existing M07 stamina tests still pass unchanged

### Failure modes
- **Hold and tap both fire** → the default tap interaction is still implicit. Add explicit `Press` and `Hold` interactions and branch on `context.interaction`.
- **Heavy fires on button-down** → you're reading `performed` on the wrong interaction. Hold performs at the threshold, not at press.
- **Stamina bar lags a frame behind** → you're updating in `LateUpdate` after the tick, or vice versa. Pick one order.
- **You never run out** → regen delay is zero, or you copied the 2D values without accounting for a slower 3D fight.

**Stretch:** A charge-up visual — the weapon glows or the character leans as the hold threshold approaches. It makes the heavy feel deliberate rather than accidental.

**Commit:** `feat: heavy attack via hold interaction and stamina gating`

---

## Day 87 — The dodge roll: i-frames, commitment, and the cancel rules
**Sat 5 Dec · 60 min**

**Objective:** A rolling dodge that moves you a real distance, costs stamina, makes you briefly untouchable, and cannot be taken back.

**Why:** The dodge is the mechanic that turns a damage race into a conversation. In 3D it also becomes your primary movement tool, which raises the stakes on getting it right.

### Concepts (10 min)
- **I-frames are a Core combat state, not a Unity bool.** `Combatant.IsInvincible` already exists from Day 53, computed from elapsed dodge time against `IFrameStart`/`IFrameEnd`. `CombatResolver` already returns `WasDodged`. You are wiring, not writing.
- **Never disable the hurtbox collider to fake invincibility.** You lose the ability to distinguish "dodged" from "missed", and the dodge-success feedback is half the reason the mechanic feels good.
- **Commitment: a roll cannot be cancelled.** Locked in for the full duration. That's the price of a bad read, and it's what makes reading the enemy matter.
- **The cancel rules, decided explicitly: a roll may cancel attack *recovery*, but not windup or active.** Recommended, and here's why — a player who whiffs a heavy and can bail out with a roll feels skilled; one who eats a hit they saw coming because they were locked in feels cheated. Windup and active stay committed so attacking still has a cost. Write it in `CONVENTIONS.md`; it's the same rule as Day 53 and you will second-guess it.
- **Root motion carries the roll.** Let the animation move you and Core own the duration. Do not add velocity on top of root motion — you'll get a roll that travels twice as far as the animation suggests and clips through geometry.
- **The classic bug: i-frames that outlast the animation.** The clip is 0.9s, your JSON says duration 0.6s, and for 0.3s you're standing upright and untouchable. Match the numbers or set the clip's speed multiplier so it fits.

### Build (40 min)
1. Import a forward roll clip. Humanoid, root motion on. Check its actual length in the Inspector and write it down.
2. Reconcile `dodge.json` (from Day 53) against that length — adjust the clip speed multiplier so the animation and the Core duration agree exactly.
3. Wire the Dodge action to `TryDodge(direction)`. Direction is the camera-relative move input, or facing if there's no input — same convention as your Day 78 controller.
4. Rotate the character to the dodge direction *before* the roll starts, so root motion travels where the player asked.
5. Implement the cancel rules: `TryDodge` succeeds from `Idle` and attack `Recovery`, fails from `Windup` and `Active`.
6. During the roll, disable the normal locomotion path entirely — the `CharacterController` should be driven by root motion only, or you'll fight yourself.
7. Debug visualisation: a bright material swap or a trail renderer active **only during the i-frame window**, not the whole roll. You must be able to see the real window while tuning.
8. Have the dummy swing at you on a loop. Roll through it ten times. If you can't reliably dodge, widen the window before you touch anything else.

### Acceptance criteria
- [ ] Roll duration in Core matches the clip length on screen
- [ ] I-frames cover only part of the roll and are visible while active
- [ ] A roll in progress cannot be cancelled by any input
- [ ] A roll can cancel attack recovery but not windup or active
- [ ] Cancel rules are written in `CONVENTIONS.md`
- [ ] Two rolls leave stamina nearly empty

### Failure modes
- **You're invincible while standing still** → i-frame window extends past the animation. Reconcile the durations.
- **Roll travels twice as far as it looks** → you're applying movement *and* root motion. Pick root motion.
- **Roll goes through walls** → `CharacterController` movement must go through `Move()`, never `transform.position`. Root motion should be routed through `OnAnimatorMove` into `Move()`.
- **Roll always goes forward** → you rotated after starting instead of before, or you're using facing instead of input direction.
- **Dodge feels mandatory-spammable** → cost too low or regen too fast. It's a stamina problem, not a dodge problem.

**Stretch:** A perfect-dodge reward — rolling within 0.1s of an enemy's active phase refunds half the stamina and briefly slows time. Cheap, and it gives skilled play something to chase.

**Commit:** `feat: dodge roll with i-frames and cancel rules`

---

## Day 88 — Lock-on targeting, and the camera work it demands
**Sun 6 Dec · 60 min**

**Objective:** Press lock-on, the camera frames an enemy, your character strafes around it, and you can switch targets.

**Why:** In third person, lock-on is what makes combat aimable at all. It is also 80% camera work, which is why it gets a whole day.

### Concepts (10 min)
- **Target acquisition: nearest enemy within a cone in front of the camera.** Not nearest in world space — the player expects "the one I'm looking at". Cone half-angle around 40°, max range around 15m, plus a line-of-sight raycast so you can't lock through a wall.
- **The camera is the hard part.** Locked and free cameras have different jobs: free follows your input, locked frames *two* things — you and the target. A Cinemachine **Target Group** containing both, driven by a second virtual camera, is the standard solution.
- **Cinemachine 3.x renamed everything.** `CinemachineVirtualCamera` became `CinemachineCamera`, the component menu paths moved, and Target Group setup changed. **Verify against the current Cinemachine documentation before wiring** — guessing here costs you fifteen minutes of your sixty.
- **Blend between cameras by priority, not by teleporting.** Raise the locked camera's priority on engage; Cinemachine handles the blend. A 0.3–0.5s ease is enough. Instant cuts are nauseating.
- **Locomotion changes shape when locked.** You stop turning to face movement and start strafing — which is exactly what the 2D blend tree from Day 80 exists for. Forward/right input feeds X and Y; the character faces the target.
- **Beginner trap:** forgetting to drop the lock. On target death, on exceeding max range, and on manual toggle. A camera locked to a corpse is a bug report.

### Build (40 min)
1. `Unity/Combat/LockOnController.cs` — finds candidates via `Physics.OverlapSphere` on the enemy layer, filters by cone angle against the camera forward, filters by line of sight, sorts by angle, returns the best.
2. Bind a Lock-On action (middle mouse / right stick click). Toggle, not hold — holding a button for a whole fight is unpleasant on a controller.
3. Create a `CinemachineTargetGroup` at runtime containing the player and the current target, weighted roughly 1 : 1 with a small radius on each.
4. A second Cinemachine camera following the target group. Lower priority by default; raised on lock. Verify the component names and priority API against current docs.
5. On lock: switch the locomotion to strafe mode — character rotation drives toward the target, movement feeds the Day 80 2D blend tree.
6. Target switching: right-stick flick or mouse delta past a threshold picks the next candidate in that screen direction. Add a small cooldown or it cycles wildly.
7. Drop the lock on target death, on distance > max range plus a hysteresis margin, and on toggle. All three.
8. A reticle on the locked target — a simple world-space sprite that faces the camera. Small, subtle, and it removes all ambiguity about what you're hitting.

### Acceptance criteria
- [ ] Lock-on picks the enemy you're looking at, not the nearest one behind you
- [ ] The locked camera frames both player and target
- [ ] Blending between free and locked cameras is smooth, not a cut
- [ ] Locomotion strafes while locked, using the 2D blend tree
- [ ] Target switching works and doesn't cycle uncontrollably
- [ ] Lock drops on death, on distance, and on toggle

### Failure modes
- **Camera whips violently** → target group radius is too small, or the blend time is near zero. Both are quick fixes.
- **Locks on through walls** → no line-of-sight raycast, or the raycast is hitting the enemy's own collider. Mask it.
- **Character faces the target but moves the wrong way** → your strafe input is still camera-relative. While locked it should be target-relative.
- **Camera stays locked after the enemy dies** → subscribe to the death event, not just a distance check.
- **Cinemachine component not found** → 3.x naming. Check the docs rather than guessing at the old name.

**Stretch:** Slightly widen the camera FOV and pull back when locked onto the heavy enemy. One line, and big enemies immediately feel bigger.

**Commit:** `feat: lock-on targeting with cinemachine target group`

---

## Day 89 — Enemy AI in 3D: NavMesh, approach, attack, reposition, back off
**Mon 7 Dec · 60 min**

**Objective:** Three enemy types that notice you, path to you, telegraph, swing, and back off — each fighting differently.

**Why:** The bandit brain from Day 54 is pure C# in Core and comes across untouched. All that changes is how the intent becomes movement: NavMesh instead of a Rigidbody2D nudge.

### Concepts (10 min)
- **NavMesh moved.** In Unity 6 the navigation tooling lives in the **AI Navigation** package and is not installed by default. Open **Window > Package Manager**, install it, and **verify the exact package name and the current baking workflow against Unity's docs** — the surface changed significantly from the legacy built-in system and stale tutorials will send you in circles.
- **The brain decides, the agent executes.** `BanditBrain.Tick(distance, state, stamina)` returns `MoveToward`, `MoveAway`, `Attack`, or `Wait` — exactly as in 2D. The Unity layer turns that intent into `NavMeshAgent.SetDestination` calls. Same eight states: Idle → Aggro → Approach → Windup → Attack → Recover → Reposition → Stagger → Dead.
- **`NavMeshAgent` and `CharacterController` will fight each other.** Both want to own position. The clean split: let the **agent drive position**, disable agent rotation (`updateRotation = false`), and let your code rotate the transform toward the player. Do not put both on the same object expecting harmony.
- **Telegraphing is fairness, and it's harder in 3D** because the enemy may be behind you or off-camera. Windups need to be longer than the 2D version — start around 0.7s — and readable from the silhouette.
- **Three enemies, three ranges profiles.** Bandit: aggressive humanoid, moderate everything. Wolf: fast, low health, circles at a preferred range instead of charging straight in. Heavy: slow, high health, huge telegraph, big damage. Same brain, three `EnemyDefinition` JSON entries. That's the whole difference.
- **You cannot debug what you cannot see.** The floating state label from Day 54 comes with you — it's the difference between a glance and a twenty-minute mystery.

### Build (40 min)
1. Install the AI Navigation package. Add a `NavMeshSurface` to your greybox environment and bake. Confirm the blue mesh appears in the Scene view.
2. `Unity/Combat/EnemyController3D.cs` — holds a `BanditBrain`, a `NavMeshAgent`, and a `CombatMachine`. Ticks the brain, converts intent to destination, calls `TryStartAttack`.
3. Set `agent.updateRotation = false`. Rotate the transform toward the player yourself, at a capped turn speed — capped rotation is what makes a heavy enemy feel heavy.
4. Feed agent velocity into the Animator's locomotion blend tree so the enemy's feet match its movement.
5. Author three `EnemyDefinition` entries in JSON: `bandit`, `wolf`, `heavy`. Vary aggro range, preferred range, attack range, reaction delay, turn speed, health, and attack id. No new C#.
6. Wolf behaviour: set preferred range wider than attack range so `Reposition` circles rather than retreats. Verify it visibly orbits you.
7. Debug label above each enemy showing state and phase timer. `#if UNITY_EDITOR` around `Handles.Label` — it breaks builds otherwise.
8. Fight one of each. Note which one you beat by dodging and which by out-trading. Out-trading means the telegraph is too short.

### Acceptance criteria
- [ ] AI Navigation package installed and a navmesh baked over the greybox
- [ ] Enemies path around obstacles rather than pushing into them
- [ ] The brain is still pure Core C# with no Unity types
- [ ] Three enemy types differ only by JSON data
- [ ] Current state is visible above each enemy in the Scene view
- [ ] All three are beatable by dodging their telegraphs

### Failure modes
- **Enemy vibrates or stutters** → `NavMeshAgent` and `CharacterController` both writing position, or overlapping range thresholds. Add hysteresis to ranges; pick one position owner.
- **Enemy slides without animating** → agent velocity isn't feeding the Animator, or the locomotion blend tree parameter is wrong.
- **Enemy won't move at all** → it's off the navmesh. Check it's within the baked surface and that the agent radius fits the geometry.
- **Enemies clump into one blob** → agent avoidance priority is identical on all of them. Vary it.
- **`Handles.Label` breaks the build** → Editor-only API. Guard it.

**Stretch:** Give the wolf a lunge attack with a longer travel distance. One attack definition, and the fast enemy suddenly requires a different dodge timing than the bandit.

**Commit:** `feat: 3d enemy ai on navmesh with three enemy types`

---

## Day 90 — Hit reactions, VFX, hitstop, camera shake. Make it hurt.
**Tue 8 Dec · 60 min**

**Objective:** The same fight as yesterday, except every landed hit has weight.

**Why:** Highest value-per-minute day of the milestone. No rules change today and the game will feel three times better by the end of the hour.

### Concepts (10 min)
- **Hitstop is the single most effective trick in action games.** Freeze both combatants for 0.05–0.08s on impact. Scale it with damage, cap it around 0.1s. It reads as weight and it costs almost nothing.
- **Freeze the combatants, not the world.** `Time.timeScale = 0` stops your UI, your particles, and your camera. Skip `Tick` on the two involved combatants and zero their movement instead.
- **Hit reactions go on an upper-body layer.** You built the avatar mask on Day 79 — a stagger clip on that layer plays over running legs, so the enemy flinches without stopping. Full-body stagger is for heavy hits only.
- **Cinemachine Impulse for shake.** An Impulse Source on the attacker, an Impulse Listener on the camera. Verify the 3.x component names against current docs. Amplitude tiny — then halve it, and halve it again for hits you *take*, or the player can't see to recover.
- **Layer the feedback on one event.** Hitstop + knockback + stagger + VFX + sound + shake all fire from your existing Core `DamageDealt` event, orchestrated by one component. One place to tune, one place to disable.
- **The rule for today: build it all, then halve it.** You will overdo it, because you tune while staring at one hit in isolation. Every one of these effects is more tasteful at 50% of what felt right during tuning.

### Build (40 min)
1. `Unity/Combat/HitFeedback3D.cs` — one component subscribing to `DamageDealt`, orchestrating everything below.
2. **Hitstop:** freeze timer on both combatants; controllers skip `Tick` and zero velocity until it expires. Scale with damage.
3. **Hit reaction:** trigger a stagger clip on the upper-body layer via the Day 79 avatar mask. Add a full-body stagger reserved for heavy hits above a poise threshold.
4. **Knockback:** apply displacement along the attacker's forward *after* hitstop ends — simultaneous knockback and freeze cancel each other visually. Move via `NavMeshAgent.Move` or `CharacterController.Move`, never `transform.position`.
5. **VFX:** a small particle burst at the contact point — sparks for weapon-on-weapon, a darker burst for flesh. Spawn from an animation event if the timing needs to be frame-exact; otherwise from the damage event.
6. **Sound:** one impact clip, `pitch = Random.Range(0.9f, 1.1f)`. A second, duller clip for hits you take so you can tell what happened without looking.
7. **Camera shake:** Cinemachine Impulse, different amplitude for dealing versus taking.
8. **Death:** on `IsDead`, play a death clip, disable the agent and colliders, drop the lock-on, destroy after a delay. Defer the destroy — never do it inside the damage callback.
9. **Halve everything.** Then fight for five minutes and tune only what still bothers you.

### Acceptance criteria
- [ ] Hitstop fires on every landed hit and freezes only the combatants
- [ ] Upper-body stagger plays without stopping enemy locomotion
- [ ] Knockback applies after hitstop, not during
- [ ] Impact sound plays with randomised pitch
- [ ] Camera shake is present and smaller for hits you take
- [ ] Enemy death drops lock-on and cleans up without a null reference

### Failure modes
- **The whole game stutters** → you used `Time.timeScale`. Freeze the combatants.
- **Stagger cancels the enemy's movement entirely** → the clip is playing on the base layer, not the masked upper-body layer.
- **Camera shake makes it unplayable** → it's at 100%. Halve it. Twice.
- **Null reference on kill** → you destroyed the GameObject mid-callback, or lock-on is still holding the corpse. Defer the destroy and drop the lock first.
- **Effects feel cheap and loud** → all at 100%, as instructed against. Halve them.

**Stretch:** A weapon trail renderer enabled only during the active phase. Purely cosmetic, disproportionately good, and it makes the swing arc readable.

**Commit:** `feat: 3d hit feedback, hitstop, and camera shake`

---

## Day 91 — BUFFER
**Wed 9 Dec**

- **Catch up.** If any of Days 85–90 spilled, this is where it lands.
- **Tune the feel.** Fight all three enemies twenty times, adjust one number at a time, write down what you changed. **Spending the entire buffer day on combat feel is a completely legitimate and high-value use of it** — the difference between combat that's technically correct and combat that's fun is entirely in these numbers, and you will not get a better opportunity.
- **Place two real encounters** in the greybox world. Two fights is the whole budget; make them different.
- **Fix the camera.** Whatever's still wrong with lock-on, it's the camera. It always is.
- **Rest.**

### Milestone review

Run `/review`. Ask specifically whether any combat **rule** has been duplicated in the Unity layer instead of read from Core — a stamina cost re-declared as a serialized field, a timing baked into an animation clip length, a damage number in a MonoBehaviour. This is the classic regression when porting to a new view, and it's easy to do accidentally because the Inspector is so convenient. Anything that leaked is work you'll do twice.

### Where you are

You have a 3D fight. Light chains into a combo, heavy commits you to something you have to mean, the roll gets you out of trouble if you read it right, stamina says no when you're greedy, lock-on keeps the camera honest, and three enemies force three different answers.

And you built it in six days over a rules engine you wrote in October and did not touch this week. That's the whole thesis of this curriculum proving itself for the second time — the first was dialogue, this was combat, and both times the expensive part was already done.

Tomorrow: world building and art direction. Greybox to a coherent, gritty world — and the thing that will surprise you is that lighting will do more for how your game looks than any model you download.

**Commit:** `docs: M12 complete — 3d combat`
