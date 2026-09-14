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

### Day 2 — 2026-09-10 — M00 — GameObjects, Components, Transforms

**Built:** Composed a cart from a parent, bed, and two wheels; moved it through its parent Transform; duplicated and independently reshaped a second cart; added a ground plane; and assigned `Interactable` and `Ground` classifications.
**Broke:** Nothing. The cart remained deliberately static; movement and input are not part of Day 2.
**Learned:** A GameObject is a scene identity and component container; components add capabilities, while parent Transforms define a shared local coordinate space.
**Criteria:** 5/5 passed
**Commit:** `docs: day 2 log`
**Felt:** Great and interactive, though I hoped to drive the cart; I understand that comes later.

### Day 3 — 2026-09-11 — M00 — Your first script and the MonoBehaviour lifecycle

**Built:** Created `Spinner` with an Inspector-controlled rotation speed, proved the startup callback order on two objects, compared frame-independent and frame-dependent rotation, and made Space pause and resume spinning.
**Broke:** Legacy `Input.GetKeyDown` threw an `InvalidOperationException` because the project used the new Input System exclusively; changed Active Input Handling to Both for today's legacy-input exercise.
**Learned:** Unity owns the loop and invokes exact-name callbacks; `Awake` runs once per component lifetime, `OnEnable` runs for each active period, and per-frame movement needs `Time.deltaTime`.
**Criteria:** 5/5 passed
**Commit:** `docs: day 3 log`
**Felt:** Fun; getting to make input visibly change something felt good.

### Day 4 — 2026-09-12 — M00 — Prefabs, instantiation, and the Project window

**Built:** Turned the spinning cube into a coloured prefab, proved asset propagation and instance overrides, spawned 50 copies at random positions through an Inspector-assigned prefab, and organised project assets under `_Project` without breaking references.
**Broke:** Nothing. The existing Day 2 carts and Day 3 comparison objects were removed after they had served their purpose.
**Learned:** Prefab assets are serialized templates; instances inherit asset changes except where a property has an explicit override, while `.meta` GUIDs preserve references when assets move.
**Criteria:** 5/5 passed
**Commit:** `docs: day 4 log`
**Felt:** Fifty synchronized red cubes felt a bit like the spinning cat meme.

### Day 5 — 2026-09-13 — M00 — Physics: Rigidbody, colliders, and the FixedUpdate rule

**Built:** Stacked a stable 15-box Rigidbody wall, launched a ball through it with an impulse applied in `FixedUpdate`, detected the ball through a trigger zone, and added a bouncy low-friction Physics Material.
**Broke:** Deliberately teleported the ball through the wall by changing its Transform, then disabled `Is Trigger` and watched the trigger become a solid invisible barrier; restored Rigidbody movement and trigger behavior afterward.
**Learned:** A Rigidbody gives physics ownership of movement; input belongs in `Update`, physics actions belong in `FixedUpdate`, and collision or trigger interaction requires a Rigidbody on at least one participating object.
**Criteria:** 5/5 passed
**Commit:** `docs: day 5 log`
**Felt:** Functionally similar to a game, though currently an 80s bowling game.

### Day 6 — 2026-09-14 — M00 — Build the toy, then export a real executable

**Built:** Turned the physics exercise into a mouse-aimed bowling toy with a red trajectory line, click-to-launch input, permanent fallen-box scoring and recolouring, an out-of-bounds loss state, R-to-restart scene reload, and a stable overview camera. Built, tested with Unity closed, and zipped a 1280×720 windowed Windows release.
**Broke:** The trajectory initially reused Line Renderer endpoint index 0 and rendered pink with an incompatible material; assigned endpoint index 1 correctly and switched the material to a URP-compatible shader.
**Learned:** A camera ray can intersect an invisible mathematical plane to turn a screen-space cursor into a world-space aim direction; a build is a separate deployed program whose executable depends on its adjacent data files.
**Criteria:** 5/5 passed
**Commit:** `docs: day 6 log — first build shipped`
**Felt:** Mouse control and the overview camera made it feel like a pretty good game.
