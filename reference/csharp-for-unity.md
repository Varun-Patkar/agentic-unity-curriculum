# C# for Unity

You know C#. This file is only the places where Unity's C# diverges from the backend C# you already write. Nothing else.

## The MonoBehaviour lifecycle

Not `Main()`. A set of callbacks Unity invokes by reflection if you define them. They are not overrides, they are not in an interface, and misspelling one fails silently.

| Callback | When | Use it for |
|---|---|---|
| `Awake()` | Object created, before anything else | Self-setup. Cache your own components. |
| `OnEnable()` | Every time the object is enabled | Subscribe to events |
| `Start()` | Before first frame, after all `Awake`s | Talk to *other* objects — they're all initialised by now |
| `Update()` | Every frame, variable rate | Input, animation, game logic |
| `FixedUpdate()` | Fixed timestep (default 50Hz) | **All physics. No exceptions.** |
| `LateUpdate()` | After all `Update`s | Cameras, anything that must react to this frame's movement |
| `OnDisable()` | Object disabled | Unsubscribe. Always pair with `OnEnable`. |
| `OnDestroy()` | Object destroyed | Cleanup |

**The rule that avoids a whole class of bugs:** `Awake` for yourself, `Start` for everyone else.

## Time

```csharp
transform.position += velocity * Time.deltaTime;   // Update — frame-rate independent
rb.MovePosition(rb.position + v * Time.fixedDeltaTime);  // FixedUpdate
```

Anything per-frame that isn't multiplied by `Time.deltaTime` behaves differently on a 144Hz monitor than on 60Hz. This is the most common bug in beginner Unity code and it's invisible on your own machine.

## Serialization — the big one

Unity's serializer is how the Inspector, prefabs, and scene files work. It is **not** `System.Text.Json` and its rules are surprising:

**Serialized:** public fields · `[SerializeField] private` fields · most primitives · `List<T>` · arrays · `[Serializable]` classes and structs · `UnityEngine.Object` references

**NOT serialized:** properties (even `public int X { get; set; }`) · `Dictionary<,>` · interfaces · `static` · `readonly` · abstract types by reference · null-but-typed custom classes (they become a default instance, never null)

Consequences you will hit:

```csharp
[SerializeField] private Rigidbody2D _body;   // right: private, still editable
public float Speed;                            // works, but leaks encapsulation everywhere
public int Score { get; set; }                 // NOT serialized. Silently zero.
public Dictionary<string,int> Flags;           // NOT serialized. Silently empty.
```

For dictionaries: serialize a `List<KeyValuePair-ish struct>` and rebuild the dictionary in `Awake`, or keep the data in Core and load it from JSON. From M03 onward you will do the latter, which sidesteps this entirely.

`[SerializeReference]` handles polymorphism, and you'll want it for dialogue nodes on Day 30 — it's the thing that lets a `List<DialogueNode>` actually hold subclasses.

## Null is a lie

`UnityEngine.Object` overloads `==` so that a destroyed object compares equal to `null` while the C# reference is still alive. Two consequences:

```csharp
if (thing != null)          // works, but is a surprisingly expensive overloaded operator
if (thing is not null)      // does NOT use the overload — will be TRUE for destroyed objects
thing?.DoStuff();           // same problem. Avoid ?. on UnityEngine.Object.
```

Use `!= null` for Unity objects. Use pattern matching freely for your own Core types.

## Performance in `Update`

`Update` runs 60+ times a second on potentially hundreds of objects. Things that are free in a request handler are not free here:

- **`GetComponent`, `Find`, `FindObjectOfType` in `Update`** — cache in `Awake`. Always.
- **Allocating in `Update`** — every `new`, every LINQ chain, every string concat feeds the GC, and a GC spike is a visible stutter.
- **`Camera.main`** is a `Find` under the hood in older versions. Cache it.
- **String building for UI** — build only when the value changes, not every frame.

LINQ is fine in Core, in setup, and in editor tooling. Keep it out of per-frame paths.

## Coroutines

Cooperative scheduling, driven by Unity's loop. Not threads.

```csharp
IEnumerator Attack() {
    _state = Windup;
    yield return new WaitForSeconds(0.15f);
    _state = Active;
    yield return new WaitForSeconds(0.10f);
    _state = Recovery;
}
```

- `StartCoroutine` returns a handle; keep it if you need to stop it.
- **A coroutine dies if its GameObject is disabled.** This causes genuinely baffling bugs.
- `new WaitForSeconds(...)` allocates — cache the instance if it's in a hot loop.
- `async`/`await` works but doesn't respect Play Mode exit, doesn't respect `Time.timeScale`, and will happily keep running after you stop the editor. Prefer coroutines for gameplay timing; `UniTask` exists if you later want awaitables that behave.

## Structure

- **Composition, not inheritance.** A deep `MonoBehaviour` hierarchy is the classic backend-dev mistake here. Many small components on one GameObject beats one `Character : Entity : Thing`.
- **`ScriptableObject` for shared config on disk.** Read-mostly data, edited in the Inspector, referenced by many objects. It is *not* for game logic — see M03.
- **Events over polling.** `UnityEvent` for designer-visible wiring, plain C# `event`/`Action` for code-to-code. From M03, Core raises plain C# events and Unity subscribes.

## The Core rule (M03 onward)

```csharp
// Hearthfall.Core — this file must never contain the string "UnityEngine"
public sealed class DialogueRunner { ... }
```

An assembly definition makes the compiler enforce this. If it compiles, the port to 3D works. That is the entire bet the curriculum makes, and it is why M10–M12 fit into six weeks instead of sixteen.
