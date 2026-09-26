# Debugging Playbook

**This is a living document.** Every time you lose more than 15 minutes to something, add it here. By Day 112 this file is the most valuable thing in the repo, because it's the record of how *you* specifically get stuck.

Agents: append to the right section when a bug is resolved. Include the symptom, the cause, and the tell.

---

## The sequence

Never skip to the fix. Run this in order — it is faster than guessing, every time.

1. **Read the *first* error in the Console**, not the last. Later errors are usually consequences.
2. **State the expectation.** Expected X, got Y. Half of all bugs dissolve here because the expectation was wrong.
3. **Identify the layer**: code · scene wiring · component config · physics/layers · editor state.
4. **`Debug.Log` at the top of the method.** Is it even running? Answer that before theorising.
5. **Find the smallest repro.** Empty scene, two objects.
6. **`git diff`.** What changed since it worked?

---

## Symptom → cause

### "NullReferenceException"
The single most common Unity error. In order of likelihood:
- A `[SerializeField]` reference is **`None`** in the Inspector. Look at it.
- You renamed the field, so Unity dropped the link. `[FormerlySerializedAs]` prevents this.
- `GetComponent<T>()` returned null — the component isn't on *that* object.
- You're in `Awake` talking to another object that hasn't `Awake`d yet. Move it to `Start`.
- The object was `Destroy`ed and you kept the reference.

### "My script isn't running"
- Not attached to any GameObject in the scene.
- The GameObject or the component is disabled.
- Method name misspelled — `Upate`, `start`, `FixedUpdated`. Unity binds by name and **fails silently**.
- Unity didn't compile. Check the Console and the spinner bottom-right.
- The class name doesn't match the filename. Unity requires this for MonoBehaviours.

### "Collisions don't work"
- Both objects need enabled Colliders: a Rigidbody2D alone has no collision shape. On Day 18 Alex had a Rigidbody2D but no Collider2D and walked through a solid Tilemap Collider 2D.
- An Editor script that paints a Tilemap and immediately saves may leave a Composite Collider with old geometry: process pending Tilemap Collider 2D changes and regenerate the composite before saving. Check its generated outline paths, then test by walking into a new obstacle in Play Mode.
- **At least one of the two objects needs a Rigidbody.** Two colliders alone do nothing.
- `Is Trigger` is checked → `OnTriggerEnter` fires, `OnCollisionEnter` does not.
- Layers aren't set to collide: `Project Settings > Physics` (or `Physics 2D`) collision matrix.
- 2D vs 3D mismatch — a `Collider2D` will never talk to a `BoxCollider`.
- Moving via `transform.position` instead of the Rigidbody → you teleport through things.
- Object is moving too fast between frames → set Collision Detection to Continuous.

### "Movement is jittery / inconsistent / too fast"
- Physics in `Update` instead of `FixedUpdate`.
- Missing `Time.deltaTime`. It'll feel right on your machine and wrong on everyone else's.
- Camera following in `Update` instead of `LateUpdate`.
- Fighting yourself: both a Rigidbody force *and* a transform write.

### "Everything is magenta"
Shader/pipeline mismatch. A Built-In material in a URP project. `Edit > Rendering > Materials > Convert Selected Built-in Materials to URP`. Happens on every model import — it is routine, not a disaster.

### "It works in Scene view but not Game view"
- Camera isn't looking at it, or it's outside the frustum.
- It's on a layer the camera's Culling Mask excludes.
- 2D: wrong sorting layer / order in layer / Z position. It's rendering, just behind something.
- UI: it's outside the Canvas, or the Canvas render mode is wrong.

### "My changes disappeared"
You made them in Play Mode. They're gone. Set an aggressive Play Mode tint in `Edit > Preferences > Colors` so this never happens twice.

### "The build behaves differently from the editor"
Genuinely a different program. Usual causes:
- Scene not added in `File > Build Profiles`.
- `Resources`/`StreamingAssets` path assumptions that only hold in the editor.
- `#if UNITY_EDITOR` code doing something load-bearing.
- Editor-only defaults, or a file path that doesn't exist on a clean machine.
- Different frame rate exposing a `Time.deltaTime` bug you had all along.

Always test the build before you ship (Day 110 exists for exactly this).

### "Animation doesn't play / character T-poses"
- Animator Controller not assigned, or the parameter name is misspelled (bound by string).
- Transition condition never becomes true — open the Animator **while in Play Mode** and watch it. This window is the debugger.
- Rig type is Generic when it needs to be Humanoid.
- Avatar not configured or not matching the mesh.
- Transition has Exit Time on when it should be immediate — that's your input lag.

### "Unity itself is broken"
In escalating order:
1. Re-import the asset (right-click > Reimport).
2. Close Unity, delete `Library/`, reopen. Safe. Slow. Fixes a startling amount.
3. `git status` — did you accidentally delete `.meta` files?
4. Revert to the last good commit and redo the change deliberately.

---

## Tools

| Tool | Use |
|---|---|
| **Console** | Turn OFF Collapse when hunting — it hides 400 identical errors as one |
| `Debug.Log` | Still the best debugger. Log *values*, not "here 1". |
| `Debug.DrawRay` / `DrawLine` | Visualise raycasts and directions in Scene view. Transforms raycast debugging. |
| **Inspector in Play Mode** | Select the object and watch fields change live |
| **Animator window in Play Mode** | Watch state transitions happen |
| **Profiler** | Only when you have a measured problem (M15) |
| **Breakpoints** | Attach VS Code / Rider to Unity. Works properly. Underused. |
| **Unity MCP / CLI** | Reading console output and scene state without alt-tabbing. Inspection only. |

---

## Bugs I actually hit

*Agents: append here. Newest at the bottom. Format:*

```markdown
### [Day N] Short symptom
**Cause:** what it actually was
**Tell:** the thing that would have found it in 30 seconds
```

<!-- entries below -->
