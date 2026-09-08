# M11 · Characters & Animation

**Days 78–84 · 26 Nov – 2 Dec 2026 · project: `Hearthfall3D`**

> Animation is the single biggest lever on perceived quality in 3D. A capsule that slides is a prototype; a person who leans into a turn is a game — same code, same camera, same greybox. Nothing else you do this year buys as much for as little.
>
> The reason this fits in one week is that you are not authoring animation. You are downloading a library of it and learning the machine that plays it back: avatars, controllers, blend trees, root motion, events. That machine is the actual subject. The clips are just data going into it.

**You end holding:** a character that walks, runs, idles, and turns — and NPCs that don't look dead.

---

## Day 78 — The Mixamo pipeline: rig, import, humanoid avatars, retargeting
**Thu 26 Nov · 60 min**

**Objective:** A rigged humanoid character in `Hearthfall3D`, playing a Mixamo idle, at the correct scale, not magenta.

**Why:** Everything this week depends on having clips and a rig. Get the import pipeline right once today and the other six days are about behaviour rather than about fighting FBX.

### Concepts (10 min)
- **Humanoid is an abstraction layer.** Unity maps a skeleton's bones onto a standard humanoid **Avatar** — hips, spine, arms, legs. Any clip authored against that abstraction plays on any rig that maps to it. That is retargeting, and it is why one animation library can dress every character in your game.
- **Generic vs Humanoid.** Humanoid for people, Generic for a door or a cart. Humanoid costs a small runtime remap and buys you the entire library.
- **In-place vs root motion downloads.** Mixamo offers both. In-place clips animate the legs and leave the root still; root-motion clips move the root. You want both — Day 81 explains which goes where.
- **Scale is not cosmetic.** 1 Unity unit = 1 metre. Mixamo FBX frequently arrives at 100× and needs a **0.01** scale factor on import. Fix it on the importer, never with a scaled transform, or the `CharacterController` and physics will misbehave in ways that look like code bugs.
- **The magenta trap.** Imported materials are Built-In pipeline and render magenta in URP. Expected, not broken.

### Build (40 min)
1. **First: verify Mixamo.** Open mixamo.com, sign in with an Adobe ID, confirm auto-rigging and animation downloads still work and are still free. Adobe has signalled changes before and this plan is stale the moment they land. If it is gone or paywalled, stop and take the fallback ladder in `reference/asset-pipeline.md` — Unity's free **Starter Assets: Third Person** first, then Quaternius, Kenney, Rokoko/ActorCore free tiers, Blender+Rigify last. The rest of the week works identically on any Humanoid rig.
2. Pick a character from Mixamo's library — do not upload a custom mesh today. Something plainly medieval-peasant-adjacent. Name it `Wat`.
3. Download animations: **idle**, **walk**, **run**, **turn left/right**, plus two combat swings for M12. Format FBX for Unity, skin off for clips, 30fps. Grab idle/walk/run **in-place**, and the swings **with** root motion.
4. Import into `Assets/Characters/Wat/`. On the mesh FBX: Inspector > **Rig** tab > Animation Type **Humanoid**, Avatar Definition *Create From This Model*, Apply.
5. Click **Configure...** and check the bone map. Green everywhere, T-pose sane. This window is the only place you will ever debug a bad rig.
6. Fix scale on the **Model** tab if the character is 100 m tall. Convert materials: select them and use Unity 6's URP material upgrader — **verify the current menu path in the URP docs rather than trusting a remembered one**; it has moved between versions.
7. Drop `Wat` in the scene next to your capsule. Add an `Animator`, assign a controller with the idle clip, press Play.
8. **Prove retargeting:** import a second Mixamo character as Humanoid and play the *same* idle clip on it. If it works, you understand what Avatars are for.

### Acceptance criteria
- [ ] Mixamo's current status verified and written into `progress/STATE.md`
- [ ] `Wat` imported as **Humanoid** with a valid, green Avatar configuration
- [ ] Character stands ~1.8 m tall in the scene, no scaled transform
- [ ] Materials render correctly in URP, not magenta
- [ ] Idle plays in Play mode
- [ ] The same clip plays correctly on a second, different character
- [ ] Every download logged in `ATTRIBUTIONS.md`

### Failure modes
- **Character is enormous or ant-sized** → Scale Factor on the Model tab, usually 0.01. Not the transform.
- **Character is a magenta blob** → Built-In materials in URP. Convert; don't panic.
- **Limbs twisted or T-posing at runtime** → bad Avatar map. Reopen Configure and fix the bone assignments.
- **Animation plays but the character floats or sinks** → the mesh's pivot isn't at the feet, or root motion is on and you don't want it yet.
- **Mixamo won't auto-rig your custom mesh** → it needs a clean, single, closed humanoid mesh in something near an A-pose. Use a library character today.

**Stretch:** Download five more idle variations now while you have Mixamo open. Day 83 needs them and downloading is not a skill worth relearning twice.

**Commit:** `feat: humanoid character import pipeline`

---

## Day 79 — The Animator: states, transitions, parameters, and layers
**Fri 27 Nov · 60 min**

**Objective:** An Animator Controller that switches `Wat` between idle and walk because of your actual input, not because you clicked something.

**Why:** The Animator Controller is a state machine over clips. You have written a dozen state machines in your life — this one just has a graph editor bolted on and communicates over strings, which is where all the pain lives.

### Concepts (10 min)
- **It is a state machine and nothing more.** States hold clips (or blend trees, tomorrow). Transitions have conditions. Parameters are the inputs. Your Core state machines from M07 are strictly more sophisticated than this thing.
- **Parameters are bound by string.** `Float`, `Int`, `Bool`, `Trigger`. `animator.SetFloat("Speed", v)` with a typo does not throw, does not warn, does nothing at all. **Cache `Animator.StringToHash("Speed")` in a static readonly int and use the hash overloads** — faster, and a typo becomes one wrong thing in one place instead of five.
- **Trigger vs Bool.** A Trigger is a Bool that auto-resets when consumed. Use Triggers for one-shot actions, Bools for sustained states. Mixing them up produces attacks that fire twice or never.
- **Exit Time is a lie you tell yourself.** *Has Exit Time* means "wait until the current clip reaches X% before transitioning". Correct for locomotion loops. Catastrophic for a combat move — it is literally input lag with a checkbox. Turn it off for anything reactive.
- **Transition Duration** cross-fades between clips. 0.1–0.25 s for locomotion, near-zero for attacks.
- **Layers + Avatar Masks** let an upper-body animation play over lower-body locomotion. You will need this in M12 for swinging while walking. Know it exists; don't build it today.

### Build (40 min)
1. `Assets/Animation/WatLocomotion.controller`. Open the **Animator** window (Window > Animation > Animator — confirm the path in Unity 6 if it isn't where you expect).
2. Add a `Float` parameter `Speed` and a `Bool` `IsGrounded`.
3. Two states: `Idle` (default, orange) and `Walk`. Assign the clips. Set both to Loop Time on the clip's import Inspector — Mixamo clips often import unlooped.
4. Transitions both ways on `Speed > 0.1` and `Speed < 0.1`. Has Exit Time **off**, duration ~0.15 s.
5. `Unity/Player/PlayerAnimationDriver.cs` — **you write it.** It reads the movement values your Day 73 controller already computes and pushes them into the Animator. It owns no gameplay logic; it is a one-way adapter, state → animation. Cache the parameter hashes.
6. Wire it to the existing controller. The controller must not know the Animator exists; the driver reads from it.
7. Play. Walk around. Watch the character transition.
8. **The debugging move that matters:** with the game running, keep the Animator window open and select `Wat`. The active state glows and a progress bar shows playback. Nothing else tells you the truth this quickly.

### Acceptance criteria
- [ ] Controller with `Idle` and `Walk`, both looping
- [ ] Transitions driven by a `Speed` parameter from real input
- [ ] Parameter names accessed via cached hashes, not inline strings
- [ ] Has Exit Time off on both transitions
- [ ] The movement controller has no reference to `Animator`
- [ ] You have watched state changes live in the Animator window during Play

### Failure modes
- **Nothing animates, no errors** → parameter name typo, or the Animator has no Avatar assigned. The silent one. Check the Animator window in Play mode first.
- **Character T-poses on transition** → a state with no clip assigned, or a clip that failed to import.
- **Animation stutters between two states** → your threshold has no hysteresis. Use `> 0.1` / `< 0.05`, not the same number both ways.
- **Transition feels delayed** → Has Exit Time is on, or the duration is too long.

**Stretch:** Add a second layer with an upper-body Avatar Mask and play a "carrying" idle over walking. Five minutes, and it's the M12 groundwork.

**Commit:** `feat: locomotion animator controller`

---

## Day 80 — Blend trees, and making locomotion that doesn't ice-skate
**Sat 28 Nov · 60 min**

**Objective:** Idle → walk → run as one continuous blend, with feet that appear to be doing the moving.

**Why:** Discrete states pop. Blend trees are how every third-person game you've played gets smooth locomotion, and ice-skating is the single most common tell of an amateur 3D build.

### Concepts (10 min)
- **A blend tree is one state containing several clips**, weighted by a parameter. 1D: one axis, usually speed. It replaces your Idle/Walk pop with a continuous surface.
- **Normalise your parameter.** Feed the tree `0..1`, not raw m/s. Then thresholds are meaningful and swapping a clip doesn't require retuning. Idle at 0, walk at ~0.5, run at 1.
- **Ice-skating is a units mismatch.** The walk clip was authored at some real-world speed. If you move the `CharacterController` faster than the clip's feet cycle, the feet slide. Two honest fixes: **(a)** set `Animator.speed` (or a per-clip multiplier) so playback matches actual ground speed, or **(b)** drive movement from the animation via root motion. Pick (a) for now; Day 81 covers (b).
- **Damp the parameter.** Snapping `Speed` from 0 to 1 in one frame makes a character teleport into a sprint. `SetFloat(hash, target, dampTime, Time.deltaTime)` has damping built in — ~0.1 s is a good start.
- **2D blend trees** take two parameters — forward and strafe — and blend eight directional clips. You need this for lock-on strafing in M12. Build the 1D version today and know the 2D one is a superset.
- **Compute speed on the horizontal plane.** Include Y and gravity makes you "run" while falling.

### Build (40 min)
1. Replace `Idle` and `Walk` with a single **Blend Tree** state (right-click > Create State > From New Blend Tree). Blend Type: **1D**, parameter `Speed`.
2. Add three motions: idle at 0.0, walk at 0.5, run at 1.0. Confirm all three loop.
3. Scrub the preview slider in the Inspector. If walk→run pops, your thresholds don't match the clips' actual speeds — move the thresholds, not the clips.
4. In your driver, normalise: horizontal velocity magnitude ÷ your run speed, clamped `0..1`. Walk should land near 0.5 — if it doesn't, your controller's walk/run ratio is the thing that's wrong.
5. Apply damping on `SetFloat`. Tune until acceleration feels weighty rather than mushy. This is a *feel* number; expect five minutes of fiddling and stop at ten.
6. Kill the ice-skating: measure the walk clip's authored speed (walk with `Animator.speed = 1`, time the distance the feet imply vs the metres you cover), then scale playback to match. Approximate is fine — the eye forgives a lot, but not 30%.
7. Turn on Gizmos and check `Speed` in the Animator window while moving. The value should ramp, not jump.
8. Add the turn: feed the controller's rotation delta into a `Turn` float now if you have it, or note it for the stretch.

### Acceptance criteria
- [ ] One 1D blend tree replaces the discrete idle/walk states
- [ ] `Speed` is normalised `0..1` and computed on the horizontal plane only
- [ ] Damping applied; the parameter ramps smoothly
- [ ] Walk and run both look like the feet are driving the motion
- [ ] No visible pop anywhere across the speed range
- [ ] You can explain, out loud, why ice-skating happens

### Failure modes
- **Feet slide badly** → playback speed doesn't match ground speed. Scale one to the other.
- **Character runs on the spot** → normalised speed is pinned at 1, or your divisor is wrong.
- **Blend looks like two clips fighting** → the clips have mismatched foot phase. Mixamo clips generally align; mixed-source clips often don't.
- **Speed spikes when falling** → you included the Y component.
- **Everything is mush** → damp time too high. 0.1 s, not 0.5.

**Stretch:** Convert to a 2D Freeform Directional tree with forward/strafe now, while the context is loaded. M12 will want it and it's cheaper today than in three weeks.

**Commit:** `feat: locomotion blend tree`

---

## Day 81 — Root motion vs in-place: what each is for, and when each ruins your day
**Sun 29 Nov · 60 min**

**Objective:** A written, deliberate policy — in-place locomotion, root-motion attacks — implemented and proven with one root-motion clip.

**Why:** This is the decision that quietly determines how the rest of your 3D game feels. Getting it wrong shows up in M12 as attacks that look weightless or a character that won't respond to input.

### Concepts (10 min)
- **In-place:** the clip animates limbs, the root stays put, *your code* moves the transform. Fully controllable, deterministic, testable. Can skate.
- **Root motion:** the clip's root bone displacement drives the transform. Unity extracts it and applies it each frame. Looks correct by construction — zero foot slide, real weight — and you have given control of your position to an artist in Sunnyvale in 2019.
- **The trade-off in one line:** root motion looks right and is hard to control; in-place is controllable and can look wrong.
- **The policy, and it's the standard one: in-place for locomotion, root motion for attacks and dodges.** Locomotion needs to answer to the stick every frame. An attack is a committed action with a fixed displacement — the lunge *should* come from the swing, and it's exactly what makes a hit feel like it has mass.
- **`Apply Root Motion`** is a toggle on the Animator component, and a per-clip *Root Transform* setting on the import Inspector (bake into pose for position/rotation). Both matter and they interact.
- **`OnAnimatorMove()`** is the override: implement it and Unity hands you `animator.deltaPosition` / `deltaRotation` to apply yourself. This is how you feed root motion into a `CharacterController` rather than letting it stomp the transform — which is what you need, because you have a `CharacterController`, not a Rigidbody.

### Build (40 min)
1. Write the policy in `CONVENTIONS.md` before touching anything: *locomotion in-place, attacks and dodges root motion, applied through `OnAnimatorMove` into the `CharacterController`.*
2. Confirm your locomotion clips are in-place: import Inspector > Animation tab > **Root Transform Position (XZ)** > *Bake Into Pose* ticked. Verify the exact label in the Unity 6 docs if it differs from this — the import UI has changed wording across versions.
3. Leave `Apply Root Motion` **off** on the Animator for now. Confirm locomotion is unchanged.
4. Add one of your Mixamo combat swings as a state, entered by a `Trigger`. Import it with root motion **not** baked into pose on XZ, so displacement survives.
5. Implement `OnAnimatorMove()` on a component you write: take `animator.deltaPosition`, pass it to `CharacterController.Move`, apply `animator.deltaRotation` to the transform. Gate it — only apply when the current state is flagged as root-motion.
6. Trigger the swing while standing. The character should step into it and stay where the animation put him.
7. Trigger it while running. Decide the rule now: attacks stop locomotion. Write it down. M12 depends on it.
8. Break it deliberately once — turn `Apply Root Motion` on globally and watch locomotion stop responding to input. Understanding this failure now saves you an hour in December.

### Acceptance criteria
- [ ] Policy written in `CONVENTIONS.md`
- [ ] Locomotion clips confirmed in-place; movement still fully input-driven
- [ ] One attack clip applies root motion through `OnAnimatorMove`
- [ ] Root motion is gated per-state, not global
- [ ] `CharacterController.Move` receives the displacement — the transform is not written directly
- [ ] You have seen and can describe the failure mode of global root motion

### Failure modes
- **Character won't move at all** → `Apply Root Motion` on with in-place clips. The clips have no displacement, so neither do you.
- **Character drifts while idling** → an idle clip with a few centimetres of unbaked root translation. Bake it into pose.
- **Attack teleports or rubber-bands** → you wrote the transform directly while the `CharacterController` also moved. One of them, not both.
- **Root motion ignores collision** → you bypassed the `CharacterController`. Always route through `Move`.
- **Rotation goes wild during attacks** → Root Transform *Rotation* not baked on a clip that turns.

**Stretch:** Add a dodge-roll with root motion and a `Trigger`. It's twenty minutes now and it's half of a Day-88 brief.

**Commit:** `feat: root motion policy for attacks`

---

## Day 82 — Animation events: driving gameplay from specific frames
**Mon 30 Nov · 60 min**

**Objective:** Footsteps that fire on foot contact, and an attack clip that reports its own windup/active/recovery boundaries — reconciled against Core's timing data.

**Why:** This is the hinge between animation and gameplay, and it's the one place this week where you can accidentally create a second source of truth for combat rules you already wrote and tested.

### Concepts (10 min)
- **An animation event calls a method by name** on a component on the same GameObject, at a specific frame. Set them in the **Animation** window (not the Animator window) with the clip selected, or in the model importer's Animation tab for read-only FBX clips.
- **The silent-failure trap.** A typo in the method name, a wrong signature, or the component sitting on a child instead of the animated root, and the event fires into nothing. Sometimes you get a console warning. Sometimes you get silence. Assume silence.
- **The supported signatures are narrow:** no parameters, or exactly one `int`, `float`, `string`, or `Object`. Not two. Not a struct. Design around it.
- **Timing is normalised to the clip**, so `Animator.speed` changes when events fire. Relevant the moment you add a slowed heavy attack.
- **The architectural problem, and it is the point of today.** Core already owns attack timings as data — windup, active, recovery — from M07, and those are unit-tested. If animation events *also* declare when the hitbox opens, you have two sources of truth that will drift, and the one that drifts is the one nobody tested.
- **Pick one direction and write it down.** Either **(a)** Core's timings are canonical and the animation is authored/retimed to match — better for balance, testability, and reusing your 2D tuning; or **(b)** the animation is canonical and you measure its event frames and feed those numbers *into* Core's data as authored values. Both are defensible. (a) is the recommendation, because your Core tests are the asset you actually have.

### Build (40 min)
1. Decide (a) or (b). Write it in `CONVENTIONS.md` with one sentence of reasoning. This is today's most important deliverable and it takes four minutes.
2. `Unity/Characters/AnimationEventRelay.cs` — **you write it.** It lives on the animated root, receives events, and re-raises them as C# events or forwards to Core. Nothing else in your codebase implements event-named methods. One choke point, one place to check for typos.
3. Add footstep events to the walk and run clips at each foot contact. Select the clip, open the **Animation** window, scrub to contact, add the event. If the clip is inside a read-only FBX, use the importer's Animation tab — verify the current Unity 6 workflow rather than guessing.
4. Play a placeholder sound. Vary pitch slightly per step so it doesn't sound like a metronome.
5. On the attack clip, add `OnAttackActiveStart` and `OnAttackActiveEnd` events at the visually correct frames of the swing.
6. **Measure them.** Note the timings in seconds. Compare against the windup/active/recovery numbers in Core's attack data from M07.
7. Reconcile per your policy: retime the animation (or its playback speed) to Core's numbers, or update Core's data to the measured values and re-run the M07 tests. Do not leave both.
8. Prove the failure mode: rename one event to `OnAttackActveStart` and confirm exactly how loudly Unity complains. Then fix it.

### Acceptance criteria
- [ ] A written source-of-truth decision in `CONVENTIONS.md`
- [ ] All animation events land on one relay component
- [ ] Footsteps fire on actual foot contact, with pitch variation
- [ ] Attack active-window events exist and are measured in seconds
- [ ] Core's attack timings and the animation agree, with one of them canonical
- [ ] M07's combat tests still pass
- [ ] You have seen what a typo'd event name does

### Failure modes
- **Event does nothing** → typo, unsupported signature, or the receiving component isn't on the object with the `Animator`. Check all three in that order.
- **Footsteps fire twice** → an event on both the loop's first and last frame. Delete one.
- **Events fire at the wrong moment after tuning** → normalised time; `Animator.speed` moved them.
- **Hitbox opens before the sword moves** → you eyeballed the frame. Scrub properly.
- **Two sets of timings drift apart** → exactly the thing today was for. Re-read your own convention.

**Stretch:** Add a `string`-parameter event for surface type and pick the footstep sound from it. One event, many surfaces, and M13's environment pass gets it free.

**Commit:** `feat: animation events for footsteps and hit windows`

---

## Day 83 — NPCs: idles, look-at, and standing somewhere believable
**Tue 1 Dec · 60 min**

**Objective:** Osric, Enid, and Cob standing in the 3D greybox, breathing, turning their heads when you approach, doing something that isn't waiting.

**Why:** A village of identical statues reads as a tech demo no matter how good your player character is. Three NPCs with varied idles and a head turn is the cheapest believability you will ever buy.

### Concepts (10 min)
- **Idle variation is the whole trick.** Identical looping idles on every NPC is the shop-window effect — the uncanny thing where a crowd is obviously one puppet copied. Different clips, different start offsets, different playback speeds. All three are nearly free.
- **Offset the phase.** Even with the same clip, `Animator.Play(stateHash, layer, Random.value)` starts each NPC at a random point in the loop. Two lines, enormous effect.
- **Look-at is presence.** A head that turns toward you is the difference between scenery and a person. Two routes: `Animator.SetLookAtPosition` / `SetLookAtWeight` inside **`OnAnimatorIK`** (requires IK Pass enabled on the Animator layer), or a manual bone rotation in `LateUpdate` after the animator has written its pose. IK is better; the manual version is fine and you already understand it.
- **Clamp and damp the look.** An NPC whose head rotates 170° to track you is horror, not warmth. Clamp to ~70°, lerp the weight in and out, drop it when the player is behind them.
- **Placement carries character.** Standing in an open square is the default and the worst option. Leaning on a post, kneeling at a crate, sitting on a step — the pose says who they are before they say a word. Mixamo has all of these.
- **Turn it off at distance.** IK, look-at, and per-frame logic on twelve NPCs across a village is real cost for something nobody can see. Distance-gate it.

### Build (40 min)
1. Download or reuse idle variations: a neutral idle, an arms-crossed idle, a leaning idle, a working/hammering idle, a sitting idle. Log them in `ATTRIBUTIONS.md`.
2. `Assets/Animation/NPCIdle.controller` — one state per idle variant, chosen by an `int` parameter set once on start. Keep it dumb.
3. `Unity/NPC/NPCPresence.cs` — **you write it.** On `Start`: pick the configured idle, randomise the phase offset, jitter `Animator.speed` by ±5%.
4. Place the Act I cast per the story bible. **Osric** working, back half-turned — he's the one who won't ask for help, so give him something to be busy with. **Enid** somewhere she can see the whole yard. **Cob** loitering near you. Save **Vance**, **Bran**, **Iselde**, **Corvin**, and **Tam** for the Vaskirk pass in M14, but stand one of them in the greybox today to check the pipeline scales.
5. Enable **IK Pass** on the base layer of the NPC controller. Implement `OnAnimatorIK` with `SetLookAtWeight` (body/head/eyes weights) targeting the player's head height. Verify the current parameter order against the Unity 6 scripting reference — it's easy to get subtly wrong and hard to spot.
6. Clamp the yaw and lerp the weight from 0 to 1 as the player enters ~6 m and is within the front arc.
7. Distance-gate: beyond ~15 m, disable IK and drop the Animator's Culling Mode to cull off-screen updates.
8. Walk the village. Look for the thing that reads as fake — usually it's two NPCs blinking in perfect sync, or a look-at that snaps.

### Acceptance criteria
- [ ] At least three named NPCs placed with distinct idles and poses
- [ ] Phase offsets randomised — no two NPCs visibly in sync
- [ ] Head look-at tracks the player, clamped and damped
- [ ] Look-at disengages when the player is behind or far away
- [ ] IK and look-at disabled beyond a distance threshold
- [ ] Existing dialogue interaction still works on every NPC
- [ ] Every clip logged in `ATTRIBUTIONS.md`

### Failure modes
- **Heads snap or spin** → no clamp, or you're applying rotation before the Animator writes the pose. Manual bone work goes in `LateUpdate`.
- **`OnAnimatorIK` never fires** → IK Pass isn't ticked on the layer. The most common one.
- **Look-at does nothing despite firing** → `SetLookAtWeight` left at 0, or called before `SetLookAtPosition`.
- **Village looks robotic** → identical clips at identical phase. Offset them.
- **Frame rate drops with NPC count** → un-culled Animators. Set Culling Mode and distance-gate.

**Stretch:** Give one NPC a two-state routine — idle, then walk to a second point, then idle — on a timer. One NPC who *goes somewhere* makes the whole village feel inhabited.

**Commit:** `feat: npc idles and look-at`

---

## Day 84 — BUFFER
**Wed 2 Dec**

- **Catch up.** If root motion or animation events ate two days, this is where you land it.
- **Source more characters.** You have three NPCs and a cast of nine. Download and import the rest as Humanoid now, while the pipeline is muscle memory. It's the least demanding hour of the week and it unblocks M14.
- **Polish locomotion.** Tune damping, transition durations, and playback speed until walking around the greybox is genuinely pleasant. You'll do it for ten hours over the next month; it should feel good.
- **Story homework.** Ten lines of Bran's dialogue were due at M07 — if they're still blank, he's teaching you the sword in M12.
- **Rest.** You're eight days from the last combat build and the 3D game is real now.

### Milestone review

Run `/review`. Ask specifically whether animation state is being *driven from* Core state or *duplicated alongside* it — the failure mode is a `PlayerAnimationDriver` that has quietly started deciding things, or an animation event that owns a combat timing Core also owns. One of those two has probably crept in.

### Where you are

**The capsule is a person.** He idles, walks, runs, turns, and swings with weight behind it. Three NPCs stand in your village doing something other than existing, and they look at you when you come near. Your Core assembly has not changed once this week — everything you built sits on top of it, which is exactly the point of having built it that way.

You are 84 days in, 75% through, with a shipped 2D game behind you and a 3D one that finally looks like a game rather than a physics demo.

Tomorrow: combat. Light, heavy, dodge, lock-on — rebuilt in 3D over the **same Core combat rules you wrote and tested in M07**. You are not designing combat next week. You are giving combat you already own a body.

**Commit:** `docs: M11 complete — characters and animation`
