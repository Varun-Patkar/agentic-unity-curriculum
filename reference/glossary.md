# Glossary

Unity and gamedev vocabulary, defined for someone who already writes software. Skip anything you already know.

**Agents: append new terms the first time they come up in a session, with the day number.**

---

## Core model

**GameObject** — an entity. An ID, a name, a transform, and a bag of components. Has no behaviour of its own.

**Component** — behaviour or data attached to a GameObject. Composition, enforced by the engine.

**MonoBehaviour** — the base class for components you write. Unity calls its lifecycle methods by reflection.

**Transform** — position, rotation, scale, and the parent/child hierarchy. Every GameObject has exactly one.

**Prefab** — a serialized GameObject template stored as an asset. A prototype. Instances can override individual fields and can be re-synced to the asset.

**ScriptableObject** — a serializable asset that isn't in a scene. Config, content, shared data. **Not for game logic** — see M03.

**Scene** — a serialized collection of GameObjects. Roughly a level, though you can load several additively at once.

**Asset** — anything under `Assets/`. Each has a `.meta` file holding the GUID that every reference points at. Commit the `.meta` files.

---

## Loop and timing

**Frame / tick** — one iteration. `Update` runs once per frame at a variable rate.

**`Time.deltaTime`** — seconds since the last frame. Multiply anything per-frame by it or your game runs at a different speed on different hardware.

**`FixedUpdate`** — fixed timestep (default 50Hz), decoupled from frame rate. All physics goes here.

**Coroutine** — cooperative scheduling via `IEnumerator`. Unity resumes it on a schedule you `yield` for. Not a thread. Dies if its GameObject is disabled.

---

## Physics

**Rigidbody** — makes an object physics-simulated. Collisions require one on at least one of the two objects.

**Collider** — the shape used for collision. Independent of the visual mesh.

**Trigger** — a collider that detects overlap without blocking. Fires `OnTriggerEnter`, not `OnCollisionEnter`.

**Raycast** — fire a line, ask what it hit. The workhorse of interaction, aiming, ground checks, and "what am I looking at".

**Layer** — a category for physics filtering and camera culling. Distinct from **Tag** (a string label) and **Sorting Layer** (2D render order). Three different systems that sound identical.

---

## Rendering

**URP** — Universal Render Pipeline. The default for most projects. Fast, cross-platform, good post-processing. Materials from the Built-In pipeline render **magenta** in it.

**Shader** — the program that decides a pixel's colour. **Material** — an instance of a shader with specific values.

**Draw call** — one instruction to the GPU. Fewer is faster. Batching and atlases reduce them.

**Sprite** — a 2D image with import settings. **Pixels Per Unit** maps its pixels to world units; keep it constant across the project.

**Sorting Layer / Order in Layer** — 2D draw order. The answer to "why is my character behind the grass".

**Post-processing / Volume** — screen-space effects: bloom, colour grading, vignette, depth of field. Where most of your art direction actually lives (M13).

**Global Illumination / baking** — precomputing bounced light. Slow to bake, cheap at runtime.

---

## Animation

**Rig** — the skeleton. **Humanoid** rigs use a standard bone map, which is what makes retargeting Mixamo animations onto any character possible.

**Animator Controller** — a state machine over animation clips, driven by parameters.

**Blend Tree** — smoothly mixes clips by a parameter. Idle → walk → run from one float.

**Root motion** — movement driven by the animation itself rather than by code. Looks better, harder to control.

**Animation Event** — a callback fired at a specific frame. This is how a sword swing knows when to become dangerous.

**IK** — inverse kinematics. Placing a hand or foot at a target and solving the joints backwards.

**NavMesh / NavMeshAgent** — baked walkable surface plus pathfinding. How enemies get to you.

---

## Design vocabulary

**Game feel / juice** — the layer of feedback (hitstop, shake, particles, sound, squash) that makes an action satisfying. Not polish. It *is* the product.

**Hitstop / hit freeze** — a few frames of frozen time on impact. The cheapest, most effective way to make a hit feel heavy.

**I-frames** — invincibility frames. The window during a dodge where you can't be hit. The whole reason dodging is a skill.

**Input buffering** — accepting a button press slightly before the game can act on it, then acting when it can. The difference between "responsive" and "unfair".

**Coyote time** — a grace period after leaving a ledge where a jump still works. Same family of ideas.

**Windup / active / recovery** — the three phases of an attack. Readability lives in windup, danger in active, commitment in recovery.

**Greybox / blockout** — a level built from untextured primitives to test how it plays before it looks like anything.

**Vertical slice** — a small piece of the game with every system present and finished to shipping quality. What you build by Day 70 and again by Day 112.

**Diegetic** — existing inside the fiction. A letter from your sister is diegetic feedback; a red morality bar is not.

---

## Curriculum-specific

**Core** — `Hearthfall.Core`, the assembly with zero `using UnityEngine`. All game rules live here.

**View** — the Unity layer. Reads input, renders Core state, forwards events. Two of these will exist by Day 112: 2D and 3D.

**The Three Ledgers** — the three profitable, corrosive choices in Vaskirk that determine Ending B. See `story-bible.md`.

**Conscience ledger** — a list of decisions with weights, not a score. It exists so the epilogue can cite specific choices back at the player.

---

<!-- Agents: append new terms below, format: **Term** — definition. *(D42)* -->
