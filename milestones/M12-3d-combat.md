# M12 · Simple Combat in 3D

**Days 85–91 · 3–9 Dec 2026 · project: `Hollowbrook3D`**

> Combat supports the mystery; it is not a progression system. Reuse M07's health, damage, attack phases, dodge state, input buffering, and creature brain unchanged wherever possible.
>
> The complete scope is one improvised melee attack, one dodge, generous automatic facing, and one creature state machine with two data-tuned variants. Two or three short encounters are enough for the whole game.

**You end holding:** one readable attack, one committed dodge, generous aim assist, and one creature archetype with two fair variants.

---

## Day 85 — Port the single attack with generous input buffering
**Thu 3 Dec · 60 min**

**Objective:** Alex swings one improvised melee weapon, hits a test creature once, and benefits from the M07 input buffer.

**Why:** The 3D layer gives an existing Core action animation, collision, and sound; it does not invent a second combat model.

### Concepts (10 min)
- Core remains canonical for health, damage, windup, active, recovery, and buffered input.
- The Animator observes Core phases; it never decides when damage is legal.
- One attack definition and one full-body animation are sufficient.
- Root displacement, if used, goes through `CharacterController.Move`.

### Build (40 min)
1. Wire the Attack action to the unchanged M07 `TryStartAttack` path.
2. Retarget the single improvised swing from M11 and reconcile its playback speed with Core timing data.
3. Drive Animator state from Core phase changes with reactive transitions.
4. Attach one trigger hitbox to the improvised weapon and enable it only during Active.
5. Preserve M07's one-hit-per-action tracking.
6. Test early presses and tune the existing buffer to feel generous without adding another move.

### Acceptance criteria
- [ ] Exactly one player attack is available
- [ ] Timing and damage come from existing Core data
- [ ] An early press inside the buffer still starts the attack
- [ ] The hitbox is active only during Core's Active phase
- [ ] One swing damages one target once
- [ ] M07 combat tests pass unchanged

### Failure modes
- **Attack feels delayed** → inspect transition exit time and transition duration before changing Core.
- **Double damage** → clear and enforce the per-action hit set.
- **Animation and hit disagree** → retime the clip to the canonical Core phases.

**Commit:** `feat: port single improvised attack to 3d`

---

## Day 86 — Automatic facing, aim assist, and telegraphs
**Fri 4 Dec · 60 min**

**Objective:** Pressing attack gently turns Alex toward the nearest visible threat in a generous forward cone, while creature attacks clearly announce themselves.

**Why:** Short encounters should be readable without asking the player to manage a targeting mode.

### Concepts (10 min)
- **Aim assist is a moment, not a mode.** Acquire only when an attack starts; ordinary movement and camera control remain unchanged.
- Rank visible creatures by angle first and distance second, inside a short range and broad forward cone.
- Rotate Alex before the committed swing, capped so targets behind Alex are not selected magically.
- Telegraphs use pose, sound, and timing; they are fairness, not decoration.

### Build (40 min)
1. Gather nearby creatures on the combat layer when Attack is pressed.
2. Reject dead, occluded, distant, and behind-camera candidates.
3. Score the remaining candidates and rotate Alex toward the best one before starting the attack.
4. Add a subtle presentation cue on the selected threat during the attack only.
5. Give the creature a readable windup pose and audio cue driven by its Core state.
6. Test no target, one target, two targets, a blocked target, and a target near the cone edge.

### Acceptance criteria
- [ ] Attack start turns Alex toward a sensible visible threat
- [ ] No persistent targeting mode or strafe state exists
- [ ] Occluded and rear targets are rejected
- [ ] Player camera control remains free
- [ ] Creature windup is readable with sight or sound
- [ ] Aim selection changes no Core combat rule

### Failure modes
- **Alex snaps across the scene** → cap range, cone, and turn angle.
- **Attacks choose through walls** → use a line-of-sight query against environment layers.
- **Camera starts steering itself** → remove target persistence; assist belongs only to attack start.

**Commit:** `feat: add generous attack-facing assist`

---

## Day 87 — The dodge: i-frames, commitment, and cancel rules
**Sat 5 Dec · 60 min**

**Objective:** Alex performs one directional dodge with the unchanged M07 invulnerability window and explicit commitment rules.

**Why:** A dodge gives the short fights one defensive decision without introducing a resource economy.

### Concepts (10 min)
- Core owns dodge duration and invulnerability timing.
- Keep the hurtbox present so feedback can distinguish a successful dodge from a miss.
- The dodge cannot cancel itself. Preserve M07's existing attack-cancel rule rather than designing a new one.
- Apply animation displacement through collision-aware movement.

### Build (40 min)
1. Retarget the M11 dodge clip and match its playback to Core duration.
2. Wire Dodge to the existing Core action with camera-relative input direction or current facing.
3. Rotate before the action starts and disable ordinary locomotion for its duration.
4. Display a temporary editor-only cue for the true invulnerability window.
5. Loop a creature test attack and verify the timing repeatedly.

### Acceptance criteria
- [ ] One dodge action exists with no resource bar
- [ ] On-screen duration matches Core timing
- [ ] Invulnerability covers only the authored portion
- [ ] Movement respects collision
- [ ] The action is committed and cannot cancel itself
- [ ] Existing M07 dodge tests pass unchanged

### Failure modes
- **Alex is invulnerable while standing** → animation and Core duration disagree.
- **Travel is doubled** → code movement and root displacement are both active.
- **Walls are crossed** → route displacement through the controller.

**Commit:** `feat: port simple dodge to 3d`

---

## Day 88 — Camera framing for short encounters
**Sun 6 Dec · 60 min**

**Objective:** The free camera keeps Alex and nearby threats readable during a short encounter without switching control schemes.

**Why:** Hollowbrook needs a camera that helps, not a separate combat camera the player must operate.

### Concepts (10 min)
- Preserve the same free orbit from exploration.
- During an encounter, modestly widen field of view or adjust follow distance based on nearby visible threats.
- Blend changes through Cinemachine priority or lens settings; never teleport the camera.
- Release encounter framing promptly when no active threat remains.

### Build (40 min)
1. Define encounter-active presentation from nearby aggroed creatures, not player input.
2. Blend to a slightly wider framing that keeps ground and telegraphs visible.
3. Retain full orbit control and collision avoidance.
4. Return smoothly to exploration framing after the encounter.
5. Test tight mine-road clearings, walls, two variants, and a defeated final threat.

### Acceptance criteria
- [ ] Encounter framing keeps Alex and active threats readable
- [ ] Player retains camera control throughout
- [ ] Exploration movement does not change into strafing
- [ ] Framing blends rather than cuts
- [ ] Camera returns after the encounter
- [ ] Walls still trigger deocclusion

### Failure modes
- **Camera hunts between threats** → frame the encounter area, not a continuously changing individual target.
- **Exploration framing never returns** → clear the encounter state on death and disengage.
- **The player loses orientation** → reduce the framing change and keep orbit input authoritative.

**Commit:** `feat: frame short encounters with free camera`

---

## Day 89 — One creature brain, two variants, NavMesh execution
**Mon 7 Dec · 60 min**

**Objective:** One creature state machine notices Alex, navigates around obstacles, telegraphs, attacks, recovers, and behaves as two variants through data only.

**Why:** The M07 brain already decides intent. Unity only translates that intent into 3D navigation and animation.

### Concepts (10 min)
- Install Unity 6's current AI Navigation package and verify its baking workflow.
- Core decides Idle, Aggro, Approach, Windup, Attack, Recover, Reposition, and Dead; `NavMeshAgent` executes movement.
- One archetype, two data profiles: for example a cautious weaker variant and a tougher territorial variant.
- Variant differences belong in definition data: health, speed, preferred range, windup, damage, and reaction delay.

### Build (40 min)
1. Add a `NavMeshSurface` to the greybox and bake walkable areas.
2. Adapt the unchanged M07 creature brain to a `NavMeshAgent` and Animator.
3. Give navigation position ownership; cap and drive visual rotation explicitly.
4. Create two data entries for the same creature prefab and state machine.
5. Add editor-only state labels and test pathing around walls and through both encounter clearings.
6. Verify each variant is readable and beatable with the same attack and dodge.

### Acceptance criteria
- [ ] NavMesh is baked and creatures route around obstacles
- [ ] One Core creature state machine drives both variants
- [ ] Variants differ only through data and presentation
- [ ] Windup and recovery are readable for both
- [ ] State labels expose current state during tuning
- [ ] No Unity navigation type enters Core

### Failure modes
- **Creature vibrates** → two components own position or thresholds lack hysteresis.
- **Creature does not move** → verify it and its destination are on the baked surface.
- **Variants require separate controllers** → move the difference back into definition data.

**Commit:** `feat: wire creature variants to NavMesh`

---

## Day 90 — Hit reactions, VFX, sound, and hitstop
**Tue 8 Dec · 60 min**

**Objective:** Every landed or avoided hit communicates clearly through restrained reaction, effect, sound, and camera feedback.

**Why:** Feedback makes a simple ruleset legible; it does not need more moves.

### Concepts (10 min)
- Drive all feedback from existing Core damage and dodge results.
- Freeze only involved combatants for a very short hitstop; do not change global time.
- Use a readable full-body creature reaction, small contact VFX, impact audio, and restrained Cinemachine impulse.
- Accessibility requires important hit and dodge information not rely on colour or sound alone.

### Build (40 min)
1. Subscribe one feedback adapter to Core combat results.
2. Add brief local hitstop, then collision-aware knockback.
3. Trigger a creature reaction and Alex damage reaction.
4. Spawn one reusable impact effect and play pitch-varied audio.
5. Add a smaller camera impulse for damage taken than damage dealt.
6. Add distinct visual feedback for a successful dodge.
7. Test death cleanup and encounter-camera release.

### Acceptance criteria
- [ ] Landed hits visibly and audibly register
- [ ] Hitstop freezes only involved combatants
- [ ] Knockback respects collision and navigation
- [ ] Successful dodges have distinct feedback
- [ ] Camera impulse remains readable and restrained
- [ ] Death cleanup produces no stale encounter reference

### Failure modes
- **Whole game freezes** → remove global time scaling.
- **Effects obscure telegraphs** → reduce size, duration, and opacity.
- **Death throws a null reference** → defer cleanup and release observers first.

**Commit:** `feat: add simple combat hit feedback`

---

## Day 91 — BUFFER: tune and integrate the encounters
**Wed 9 Dec**

- Place two or three short mandatory encounters across Old Mine Road and Mercer Mine.
- Fight both variants repeatedly and change one data value at a time.
- Confirm every encounter supports the mystery's pacing rather than becoming a gate.
- Cut an encounter before adding another move or creature.
- Re-run all M07 combat tests and one end-to-end gameplay pass.

### Acceptance criteria
- [ ] The game contains only two or three short mandatory encounters
- [ ] Both variants use the same creature prefab and Core state machine
- [ ] One attack, one dodge, and automatic attack-facing are enough to finish every encounter
- [ ] Encounter tuning is stored in data
- [ ] Combat never blocks dialogue, quest, save, or scene transitions after it ends
- [ ] M07 Core rules and tests remain unchanged wherever possible

### Milestone review

Run `/review`. Search for duplicated timings, damage, dodge rules, or variant branches in MonoBehaviours. Confirm the combat surface has not grown beyond the story bible.

### Where you are

Alex can survive Hollowbrook's few dangerous moments with one improvised swing and one committed dodge. The camera and automatic facing reduce friction, while a single creature brain produces two readable variants through data. Combat supports the investigation and then gets out of its way.

**Commit:** `docs: M12 complete — simple combat in 3d`
