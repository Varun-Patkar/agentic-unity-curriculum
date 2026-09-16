# CURRICULUM — 112 Days

**10 Sep 2026 → 30 Dec 2026 · 1 hour/day · 16 milestones × 7 days**

Six content days, then a buffer day. Every milestone. The buffer day is not optional slack you should feel bad about using — it is the load-bearing beam that makes a 16-week plan survivable.

| | |
|---|---|
| **Days 1–14** | Throwaway. Learn the editor, finish a tiny game. Nothing here carries forward — that's the point. |
| **Days 15–70** | Build the 2D vertical slice of *Last Stop, Hollowbrook*. Ship it. |
| **Days 71–112** | Rebuild the presentation layer in 3D over the *same* core. Ship that. |

**Two release days: Day 70 and Day 112.** Both public, both on itch.io.

> **Holiday note:** M15 crosses Christmas. If you take 24–26 Dec off, take them — you'll have earned three days by then. Slide the end to 2 Jan and don't renegotiate anything else.

---

## Phase 1 — Throwaway (Days 1–14)

### [M00 · Ground Zero — The Editor](milestones/M00-ground-zero.md)
**Days 1–7 · 10–16 Sep · project: `Sandbox00`**
*You end holding: a physics toy you built and exported, and an editor that no longer intimidates you.*

| Day | Date | |
|---|---|---|
| 1 | Thu 10 Sep | Install audit, first project, and the six windows that are the whole editor |
| 2 | Fri 11 Sep | GameObjects, Components, Transforms — composition with a serializer bolted on |
| 3 | Sat 12 Sep | Your first script: the MonoBehaviour lifecycle and why `Update` is not `main()` |
| 4 | Sun 13 Sep | Prefabs, instantiation, and the Project window as a filesystem |
| 5 | Mon 14 Sep | Physics: Rigidbody, colliders, triggers, and the `FixedUpdate` rule |
| 6 | Tue 15 Sep | Build the toy — knock a tower down, then export your first `.exe` |
| 7 | Wed 16 Sep | **BUFFER** |

### [M01 · First Blood — A Complete Tiny Game](milestones/M01-first-blood.md)
**Days 8–14 · 17–23 Sep · project: `ChickenChase`**
*You end holding: a finished, built, playable game. Small, silly, and complete — which is more than most people who "learn Unity" ever get.*

| Day | Date | |
|---|---|---|
| 8 | Thu 17 Sep | New 2D URP project — sprites, pixels-per-unit, and the 2D pipeline |
| 9 | Fri 18 Sep | The Input System: actions, maps, bindings, and reading them in code |
| 10 | Sat 19 Sep | Spawning, collecting, scoring — the loop |
| 11 | Sun 20 Sep | UI: score, timer, game-over screen |
| 12 | Mon 21 Sep | Juice: sound effects, particles, screenshake — why it suddenly feels like a game |
| 13 | Tue 22 Sep | Menus, scene loading, and a real distributable build |
| 14 | Wed 23 Sep | **BUFFER** — or send it to one person and watch them play it |

---

## Phase 2 — The 2D Vertical Slice (Days 15–70)

Project: **`Hollowbrook`**. Everything from here compounds. Nothing gets thrown away.

### [M02 · Movement & Feel](milestones/M02-movement-and-feel.md)
**Days 15–21 · 24–30 Sep**
*You end holding: Alex, walking around Hollowbrook's town square, with a camera that behaves.*

| Day | Date | |
|---|---|---|
| 15 | Thu 24 Sep | Project setup done right — URP 2D, folder conventions, git, `.gitignore` |
| 16 | Fri 25 Sep | Input Actions properly: one asset, many devices, no `Input.GetKey` anywhere |
| 17 | Sat 26 Sep | Top-down controller with real feel — accel, friction, and why raw input feels bad |
| 18 | Sun 27 Sep | Tilemaps: painting Hollowbrook, rule tiles, and the tile palette |
| 19 | Mon 28 Sep | Collision, layers, and solving the 2D sorting problem |
| 20 | Tue 29 Sep | Cinemachine — a camera that follows without making you seasick |
| 21 | Wed 30 Sep | **BUFFER** |

### [M03 · The Core — Engine-Agnostic Game Logic](milestones/M03-the-core.md)
**Days 22–28 · 1–7 Oct**
*You end holding: a `Hollowbrook.Core` assembly with zero `using UnityEngine`, under test. **This milestone is why the 3D half fits in six weeks.***

| Day | Date | |
|---|---|---|
| 22 | Thu 1 Oct | Assembly definitions — building the wall, and making the compiler enforce it |
| 23 | Fri 2 Oct | Modelling world state as plain C#: who you are, where you are, what's true |
| 24 | Sat 3 Oct | The flag system and decision ledger — a list of choices, not a score |
| 25 | Sun 4 Oct | EditMode tests: proving game logic without ever pressing Play |
| 26 | Mon 5 Oct | The bridge — how Unity observes Core without Core knowing Unity exists |
| 27 | Tue 6 Oct | ScriptableObjects as *content*, never as logic — and where the line is |
| 28 | Wed 7 Oct | **BUFFER** |

### [M04 · Dialogue](milestones/M04-dialogue.md)
**Days 29–35 · 8–14 Oct**
*You end holding: walk up to an NPC, press E, and have a conversation with portraits and typewriter text.*

| Day | Date | |
|---|---|---|
| 29 | Thu 8 Oct | Designing the dialogue graph — nodes, edges, and why not to use an existing tool yet |
| 30 | Fri 9 Oct | Authoring dialogue as data: the file format, the loader, the validation |
| 31 | Sat 10 Oct | The dialogue runner — advancing nodes, in Core, tested headlessly |
| 32 | Sun 11 Oct | The UI: panel, speaker name, portrait, body text |
| 33 | Mon 12 Oct | Typewriter text, skip-to-full, and why pacing is a gameplay system |
| 34 | Tue 13 Oct | Interactables — proximity, prompts, and meeting Mayor Vale |
| 35 | Wed 14 Oct | **BUFFER** |

### [M05 · Choice & Consequence — *the pillar*](milestones/M05-choice-and-consequence.md)
**Days 36–42 · 15–21 Oct**
*You end holding: choices that branch, conditions that gate, and consequences that arrive three scenes later.*

| Day | Date | |
|---|---|---|
| 36 | Thu 15 Oct | Choice nodes and conditions — gating on flags without hardcoding anything |
| 37 | Fri 16 Oct | The consequence engine: immediate effects, and **deferred** effects |
| 38 | Sat 17 Oct | Authoring Hollowbrook's four central choices |
| 39 | Sun 18 Oct | Delayed town reactions — diegetic feedback without a morality meter |
| 40 | Mon 19 Oct | Debug tooling: a custom Editor window that shows you every flag |
| 41 | Tue 20 Oct | Automated playthroughs — walking every branch in a unit test |
| 42 | Wed 21 Oct | **BUFFER** |

### [M06 · Quests & Journal](milestones/M06-quests-and-journal.md)
**Days 43–49 · 22–28 Oct**
*You end holding: Act I, playable start to finish, with a journal that tracks it.*

| Day | Date | |
|---|---|---|
| 43 | Thu 22 Oct | A quest is a state machine — model it in Core |
| 44 | Fri 23 Oct | Objectives, triggers, and completion conditions |
| 45 | Sat 24 Oct | The journal UI — active, completed, and the failed ones |
| 46 | Sun 25 Oct | Markers, and solving "I don't know where to go" without a minimap |
| 47 | Mon 26 Oct | Wiring Act I as real content — welcome back to Hollowbrook |
| 48 | Tue 27 Oct | Playtest Act I end to end. Write the bug list. Fix the top three. |
| 49 | Wed 28 Oct | **BUFFER** |

### [M07 · Combat, 2D Edition](milestones/M07-combat-2d.md)
**Days 50–56 · 29 Oct – 4 Nov**
*You end holding: one attack, one dodge, and one creature that is readable and fair.*

| Day | Date | |
|---|---|---|
| 50 | Thu 29 Oct | Combat resolution in Core — health, damage, and combat states as data |
| 51 | Fri 30 Oct | The attack: windup, active, recovery, and input buffering |
| 52 | Sat 31 Oct | Hitboxes and hurtboxes — the difference, and why it matters |
| 53 | Sun 1 Nov | The dodge, invincibility frames, and commitment |
| 54 | Mon 2 Nov | The creature — one state machine with two tuned variants |
| 55 | Tue 3 Nov | Hit feel: hitstop, knockback, flash, sound. The 20% that is 80% of combat. |
| 56 | Wed 4 Nov | **BUFFER** |

### [M08 · Save/Load & The Game Shell](milestones/M08-save-and-shell.md)
**Days 57–63 · 5–11 Nov**
*You end holding: a game you can quit and return to, with a menu, pause screen, and three endings wired to decisions and profile history.*

| Day | Date | |
|---|---|---|
| 57 | Thu 5 Nov | Serializing Core state — and versioning it before you need to |
| 58 | Fri 6 Nov | Save slots, autosave, load, and the "what belongs in a save" question |
| 59 | Sat 7 Nov | Scene flow and a game manager that survives scene loads |
| 60 | Sun 8 Nov | Main menu, pause, settings — the shell that makes it a product |
| 61 | Mon 9 Nov | The ending gate: main choice outcomes plus the first-run skip ending |
| 62 | Tue 10 Nov | Writing all three epilogues — two full conclusions and one deadpan ejection |
| 63 | Wed 11 Nov | **BUFFER** |

### [M09 · Ship the 2D Slice](milestones/M09-ship-the-2d-slice.md)
**Days 64–70 · 12–18 Nov**
*You end holding: **a published game with a URL.** Mid-point win. This is the day the whole thing stops being a tutorial.*

| Day | Date | |
|---|---|---|
| 64 | Thu 12 Nov | Content pass — fill every gap, cut every "TODO" that isn't shippable |
| 65 | Fri 13 Nov | Audio pass: music, ambience, UI sounds, and mixing |
| 66 | Sat 14 Nov | Art pass — AI + paint sprites and portraits, made consistent |
| 67 | Sun 15 Nov | Playtest properly. Watch someone else play it. Write everything down. |
| 68 | Mon 16 Nov | Fix the list. Ruthless triage — ship-blockers only. |
| 69 | Tue 17 Nov | Build, package, and write the itch.io page |
| 70 | Wed 18 Nov | **SHIP DAY** — publish it, send it to three people, close the laptop |

---

## Phase 3 — The 3D Game (Days 71–112)

Same `Hollowbrook.Core`. New everything else.

### [M10 · Into 3D — New View, Same Core](milestones/M10-into-3d.md)
**Days 71–77 · 19–25 Nov**
*You end holding: a 3D scene running your dialogue system off code you did not modify. The moment M03 pays for itself.*

| Day | Date | |
|---|---|---|
| 71 | Thu 19 Nov | New 3D URP project — and importing Core with zero changes |
| 72 | Fri 20 Nov | 3D scene fundamentals: units, scale, gravity, materials, why everything is grey |
| 73 | Sat 21 Nov | A third-person character controller that isn't a physics accident |
| 74 | Sun 22 Nov | Cinemachine in 3D — follow, orbit, and camera collision |
| 75 | Mon 23 Nov | **The proof**: run a Hollowbrook conversation in 3D against untouched Core |
| 76 | Tue 24 Nov | Interaction in 3D — raycasts, triggers, and "what am I looking at?" |
| 77 | Wed 25 Nov | **BUFFER** |

### [M11 · Characters & Animation](milestones/M11-characters-and-animation.md)
**Days 78–84 · 26 Nov – 2 Dec**
*You end holding: a character that walks, runs, idles, and turns — and NPCs that don't look dead.*

| Day | Date | |
|---|---|---|
| 78 | Thu 26 Nov | The Mixamo pipeline: rig, import, humanoid avatars, retargeting |
| 79 | Fri 27 Nov | The Animator — states, transitions, parameters, and layers |
| 80 | Sat 28 Nov | Blend trees, and making locomotion that doesn't ice-skate |
| 81 | Sun 29 Nov | Root motion vs in-place: what each is for, and when each ruins your day |
| 82 | Mon 30 Nov | Animation events — driving gameplay from specific frames |
| 83 | Tue 1 Dec | NPCs: idles, look-at, and standing somewhere believable |
| 84 | Wed 2 Dec | **BUFFER** |

### [M12 · Simple Combat in 3D](milestones/M12-3d-combat.md)
**Days 85–91 · 3–9 Dec**
*You end holding: one readable attack, one dodge, generous aim assist, and one creature archetype with two variants.*

| Day | Date | |
|---|---|---|
| 85 | Thu 3 Dec | Port the single attack with generous input buffering |
| 86 | Fri 4 Dec | Aim assist and clear attack telegraphs |
| 87 | Sat 5 Dec | The dodge roll: i-frames, commitment, and the cancel rules |
| 88 | Sun 6 Dec | Camera framing for short encounters, without lock-on mode |
| 89 | Mon 7 Dec | Creature AI in 3D — NavMesh, approach, attack, reposition |
| 90 | Tue 8 Dec | Hit reactions, VFX, hitstop, camera shake. Make it *hurt*. |
| 91 | Wed 9 Dec | **BUFFER** |

### [M13 · World Building & Art Direction](milestones/M13-world-and-art-direction.md)
**Days 92–98 · 10–16 Dec**
*You end holding: a town square, mine road, and mine that read as one coherent supernatural place.*

| Day | Date | |
|---|---|---|
| 92 | Thu 10 Dec | Greybox first: blocking out space before a single pretty asset |
| 93 | Fri 11 Dec | Building Hollowbrook — kitbash discipline and the 80/20 of set dressing |
| 94 | Sat 12 Dec | The old mine road and Mercer Mine |
| 95 | Sun 13 Dec | Lighting — the single biggest visual lever you own |
| 96 | Mon 14 Dec | Fog, post-processing, colour grading: manufacturing the gritty look |
| 97 | Tue 15 Dec | Ambience and audio-visual coherence · dressing Hollowbrook for the last night |
| 98 | Wed 16 Dec | **BUFFER** |

### [M14 · Narrative in 3D + Kokoro VO](milestones/M14-narrative-in-3d.md)
**Days 99–105 · 17–23 Dec**
*You end holding: conversations that are framed like a film, voiced, with the three endings staged.*

| Day | Date | |
|---|---|---|
| 99 | Thu 17 Dec | Dialogue cameras — framing a conversation without a cutscene tool |
| 100 | Fri 18 Dec | Porting the dialogue UI to 3D (and the UI Toolkit vs uGUI decision) |
| 101 | Sat 19 Dec | Wiring Kokoro VO: clips, timing, subtitles, and graceful fallback |
| 102 | Sun 20 Dec | Quest log and markers in a 3D world |
| 103 | Mon 21 Dec | Staging the three endings — camera, light, silence |
| 104 | Tue 22 Dec | Acts I and II wired end to end |
| 105 | Wed 23 Dec | **BUFFER** |

### [M15 · Polish, Optimise, Ship](milestones/M15-polish-and-ship.md)
**Days 106–112 · 24–30 Dec**
*You end holding: a released 3D choice-driven supernatural RPG with your name on it.*

| Day | Date | |
|---|---|---|
| 106 | Thu 24 Dec | The Profiler — find what's actually slow, not what you assume is |
| 107 | Fri 25 Dec | Full playtest. Every path. Write down everything. |
| 108 | Sat 26 Dec | Fix pass 1 — ship-blockers |
| 109 | Sun 27 Dec | Fix pass 2 + settings, resolution, key rebinding |
| 110 | Mon 28 Dec | Build it, then test *the build* — it is a different program |
| 111 | Tue 29 Dec | Store page, screenshots, a 30-second gif |
| 112 | Wed 30 Dec | **SHIP DAY** |

---

## The two days you will want to quit

Around **Day 30** (the Core milestone feels abstract and nothing looks different on screen) and around **Day 75** (the 3D transition, when everything you were good at stops applying).

Both are normal. Both are in `AGENTS.md` under §9. When you hit them, open `progress/LOG.md` and read what you actually built. It will be more than you remember. Then take a buffer day and come back.
