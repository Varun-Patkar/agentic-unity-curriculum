# M14 · Narrative in 3D + Kokoro VO

**Days 99–105 · 17–23 Dec 2026 · project: `Hearthfall3D`**

> Every line of narrative logic you need already exists. The dialogue graph, the conditions, the deferred ConsequenceEngine, the quests, Enid's letters, the conscience ledger, the `EndingSelector` — all of it is in `Hearthfall.Core`, unit-tested, and completely unaware that the game is now in three dimensions. That was the entire bet you made in M03 and this is the week it pays out.
>
> So this milestone is not systems work. It is presentation, framing, voice, and restraint. You are learning to point a camera at a conversation, put a subtitle under it, and hold a shot long enough to hurt.

**You end holding:** conversations that are framed like a film, voiced, with the three endings staged.

---

## Day 99 — Dialogue cameras: framing a conversation without a cutscene tool
**Thu 17 Dec · 60 min**

**Objective:** Talk to an NPC and the camera moves to a real over-the-shoulder angle, then blends back to gameplay when the conversation ends.

**Why:** In 2D, a conversation was a panel. In 3D, an unframed conversation is two people standing in a field while text appears — and it reads as cheap instantly. A camera that knows where to stand is 80% of the production value.

### Concepts (10 min)
- **This is not a cutscene.** No Timeline, no animated camera tracks, no keyframes. It is a virtual camera that positions itself relative to two transforms and blends in. Timeline exists, it is genuinely good for authored sequences, and it is out of scope for this game.
- **Over-the-shoulder framing:** camera sits behind and slightly above one speaker's shoulder, looking at the other's head. The listener's shoulder occupying a third of frame is what makes it read as a conversation rather than a security camera.
- **The 180-degree rule.** Draw an imaginary line through the two speakers. Keep the camera on one side of it. Cross it and the characters appear to swap sides between shots, and the player gets a lurch of disorientation they can't name.
- **Blending is Cinemachine's whole job.** Raise a dialogue camera's priority and the Brain interpolates. You do not write camera lerps in 2026.
- **The trap:** building a bespoke camera GameObject per conversation. You will have forty conversations. Build one rig that takes two transforms and positions itself.
- **Always give the player an exit.** A camera the player cannot escape is a camera that will eventually trap them in a wall.

### Build (40 min)
1. **Verify before you build:** Cinemachine 3.x renamed the core components (`CinemachineVirtualCamera` → `CinemachineCamera`, and the Brain and blend settings moved). Open Package Manager, check your installed Cinemachine version, and read the current docs for the exact component names. Do not trust a two-year-old tutorial here.
2. Create `DialogueCameraRig` as a prefab: one Cinemachine camera, low priority by default.
3. `Unity/Camera/DialogueCameraRig.cs` — a `Frame(Transform speaker, Transform listener)` method that positions the rig behind the listener's shoulder, aims at the speaker's head bone or a `HeadAnchor` child transform, and raises priority.
4. Compute the shoulder position from the speaker-to-listener vector plus a fixed lateral and vertical offset. Pick a side on first frame and **store it** — that's your 180-line, and re-deriving it per line is how you accidentally cross it.
5. Subscribe to the Core `ConversationStarted` / `ConversationEnded` events. Start frames the rig, end drops priority and lets the gameplay camera win.
6. Set the blend to EaseInOut, roughly 0.5–0.8 seconds. Faster feels like a cut, slower feels like a cutscene.
7. Add a raycast sanity check: if the framed position is inside geometry, pull the camera toward the listener until it isn't.
8. Test in the Hearthfall farmhouse (tight interior) and on the Wealdrun road (open). Interiors are where camera rigs die.

### Acceptance criteria
- [ ] Starting a conversation blends to an over-the-shoulder angle
- [ ] Ending a conversation blends back to gameplay control
- [ ] One rig prefab serves every conversation in the game
- [ ] The camera stays on one side of the 180-line for a whole conversation
- [ ] The rig does not clip into walls in the farmhouse interior
- [ ] Player input is locked to `UI` context while framed and restored on exit

### Failure modes
- **Camera snaps instead of blends** → the Brain's default blend is Cut, or the rig was already at high priority.
- **Camera inside the NPC's head** → you aimed at the root transform, not a head anchor. Add the anchor.
- **Characters swap sides between lines** → you're re-picking the shoulder each line. Latch it at conversation start.
- **Blend back leaves the player camera pointing at the sky** → the gameplay camera's rotation was never restored; let Cinemachine own it rather than writing to the transform.

**Stretch:** Add a second rig position — a wider two-shot — and switch to it for lines longer than ~120 characters. One extra angle buys a surprising amount of variety.

**Commit:** `feat: dialogue camera rig with over-the-shoulder framing`

---

## Day 100 — Porting the dialogue UI to 3D (and the UI Toolkit decision)
**Fri 18 Dec · 60 min**

**Objective:** The 2D dialogue panel, adapted for a 3D screen, running against the same Core dialogue runner.

**Why:** Day 75 proved a basic 3D dialogue UI works. Today it stops being a prototype and becomes the one the game ships with, with the framing camera behind it.

### Concepts (10 min)
- **The uGUI vs UI Toolkit decision, honestly:** UI Toolkit is the better long-term technology and Unity is clearly steering toward it. It is also a week of learning, and you have a working, styled uGUI dialogue panel from the 2D game that ports in about an hour. It is Day 100 of 112. **Stay on uGUI.** Write the reason down so future-you doesn't relitigate it at 11pm.
- **Screen real estate inverts in 3D.** In 2D the panel could own half the screen. In 3D the character is the performance — the UI has to get out of its way. Bottom third, maximum.
- **Letterboxing during conversation** — two black bars easing in — costs ten minutes and instantly signals "this is a scene". It also hides your camera's worst framing sins.
- **Portraits are now redundant.** The actual character is on screen, animated, looking at you. A portrait next to them is a competing image. Drop it; keep the speaker name.
- **Safe areas are real.** `Screen.safeArea` and a `CanvasScaler` set to Scale With Screen Size with a matched reference resolution. Test at 16:10 and ultrawide, not just 1920×1080.
- **The trap:** rebuilding the UI from scratch because the old one "was for 2D". It was never for 2D. It was for a `DialogueRunner`.

### Build (40 min)
1. Copy the dialogue canvas prefab from the 2D project into `Hearthfall3D`. Fix the missing script references — they'll point at your `Hearthfall.Core` assembly, which is shared, so most should resolve.
2. Re-anchor: dialogue text and speaker name in the bottom third, choices stacked above it, right-aligned.
3. Delete the portrait image and its layout slot. Widen the text.
4. Add a `LetterboxController` — two black `Image` bars that ease in over 0.3s on `ConversationStarted` and out on `ConversationEnded`. Same events as the camera rig.
5. Apply `Screen.safeArea` to a root `RectTransform` via a small `SafeAreaFitter` component. Set the `CanvasScaler` reference resolution and match mode; **verify the current recommended match value in the docs** rather than copying a number from memory.
6. Add a subtitle-style presentation mode: no panel background, centred text with a soft drop shadow, name in small caps above. Make it a serialized toggle — you'll want to compare them, and one of them is what ships.
7. Keep the typewriter reveal and the click-to-complete behaviour exactly as they are. Tomorrow it has to sync with audio, so don't touch its timing model yet.
8. Play three conversations at 1920×1080, 2560×1080, and a resized window.

### Acceptance criteria
- [ ] The 2D dialogue panel drives conversations in 3D against the unchanged Core runner
- [ ] UI occupies the bottom third and never covers the speaker's face
- [ ] Letterbox eases in and out with the conversation
- [ ] Layout is correct at 16:9, 16:10, and ultrawide
- [ ] Portrait removed; speaker name retained
- [ ] The uGUI decision is written down in `CONVENTIONS.md` with the reasoning

### Failure modes
- **Text is a wall of pixels at 4K** → `CanvasScaler` still on Constant Pixel Size.
- **Choices render behind the letterbox** → sibling order in the hierarchy; the bars must be below the text in the canvas.
- **Missing script references everywhere** → the 2D project referenced a Unity-side assembly you didn't port. Port the specific script, not the whole folder.
- **UI shows before the camera has blended** → delay the panel fade by the blend duration, or it appears over the gameplay shot.

**Stretch:** Fade the dialogue panel's alpha to 0 while a voiced line is playing and the subtitle is short. Trust the performance for a beat.

**Commit:** `feat: 3d dialogue ui with letterboxing`

---

## Day 101 — Wiring Kokoro VO: clips, timing, subtitles, and graceful fallback
**Sat 19 Dec · 60 min**

**Objective:** Key lines play voiced audio in sync with the typewriter reveal, and every unvoiced line behaves exactly as it does today.

**Why:** You can generate the clips in minutes. Getting them into Unity with the right import settings, keyed to the right nodes, and degrading cleanly is the part nobody writes a tutorial about.

### Concepts (10 min)
- **Import settings are the whole game here.** Voice is mono, so tick **Force To Mono** — stereo doubles memory for zero benefit on a dialogue line. Compression **Vorbis** at a modest quality for anything over a second or two.
- **Load Type matters more than compression.** `Decompress On Load` for short lines (fast to play, costs RAM), `Streaming` for anything long. Do **not** use `Compressed In Memory` for dialogue — it decodes on the audio thread and can hitch on the first frame of playback.
- **Key clips to node IDs, not to text.** `vo_<conversationId>_<nodeId>.wav`. The moment you rewrite a line, text-based keying silently breaks and audio-to-node keying doesn't.
- **Most lines will have no clip and that must be the normal path**, not an error case. You are voicing a handful of key lines. A missing clip is data, not a bug — do not log a warning for it.
- **Subtitles are always authoritative and always on screen.** Audio is decoration. If audio and text disagree, text wins, and the player never depends on hearing anything.
- **The trap:** driving conversation advance off `AudioSource.isPlaying`. Now an unvoiced line advances instantly and a voiced line can't be skipped. Advance is a player decision, always.

### Build (40 min)
1. `Assets/Audio/VO/` — drop in your generated WAVs using the naming convention. Select them all and set import settings once in the Inspector; **verify the current field names in the Audio Clip import docs**, as the importer UI has shifted between versions.
2. `Unity/Audio/VoiceBank.cs` — a `ScriptableObject` holding a serialized `List<VoiceEntry>` of `{ string nodeKey; AudioClip clip; }`, built into a `Dictionary` on `OnEnable`. Simple, inspectable, no Addressables.
3. A small editor button on the `VoiceBank` that scans `Assets/Audio/VO/` and fills the list from filenames. Ten minutes of editor tooling that saves you forty entries of manual dragging — this is exactly the kind of tool you're allowed to have written for you.
4. `Unity/Audio/DialogueVoicePlayer.cs` — subscribes to the runner's "line shown" event, looks up the key, and plays through a dedicated `AudioSource`. No entry found → return silently.
5. Route that `AudioSource` to a **Voice** group in an `AudioMixer`, separate from Music and SFX. Expose its volume so the options menu can reach it later.
6. Typewriter sync: if a clip exists, set the reveal duration to `clip.length * 0.85` so text finishes just before the audio does. If no clip, keep the existing fixed characters-per-second. One branch, two lines.
7. Skip handling: when the player clicks to complete the reveal, the audio keeps playing; when they advance to the next line, `Stop()` the source first. Two different actions, two different rules.
8. Test all four cases: voiced line played through, voiced line skipped mid-word, unvoiced line, and a voiced line interrupted by opening the journal.

### Acceptance criteria
- [ ] Voiced lines play in sync with the typewriter reveal
- [ ] Unvoiced lines behave identically to before, with no console noise
- [ ] Subtitles are always displayed regardless of audio
- [ ] Advancing a line stops the current clip cleanly, with no overlap
- [ ] VO routes through a dedicated mixer group with exposed volume
- [ ] Clips are mono, Vorbis, with a deliberate Load Type per length

### Failure modes
- **Two lines talking over each other** → you didn't `Stop()` on advance, or you're using `PlayOneShot` and can't stop it. Use `clip` + `Play()`.
- **First voiced line hitches** → `Compressed In Memory` load type, or the clip isn't preloaded. Switch to Decompress On Load.
- **Lookup misses everything** → filename case, or a stray `.wav` extension in the key. Normalise the key when you build the dictionary.
- **Audio continues after the conversation ends** → nothing is listening to `ConversationEnded`. Stop the source there too.
- **Text finishes long before the audio** → your characters-per-second is still fixed; the clip-length branch isn't being taken.

**Stretch:** Add a subtle mouth-flap: drive a blend shape or a small jaw-bone rotation from `AudioSource.GetOutputData` RMS. Crude lip-sync, twenty lines, disproportionate payoff.

**Commit:** `feat: kokoro vo playback with subtitle fallback`

---

## Day 102 — Quest log and markers in a 3D world
**Sun 20 Dec · 60 min**

**Objective:** Press J for the journal in 3D, and see a world-space objective marker that respects the tone you committed to on Day 46.

**Why:** The journal ports almost unchanged. The markers do not — a 2D marker was a sprite at a position, and a 3D marker has to deal with distance, occlusion, and the screen edge.

### Concepts (10 min)
- **Billboarding** means the marker always faces the camera. `transform.forward = camera.transform.forward` — copy the camera's forward, do not `LookAt` the camera position, or markers at the screen edge visibly skew.
- **Distance fade in both directions.** Fade in past a minimum distance (a marker on something two metres away is noise) and cap the maximum so you're not signposting across the whole valley.
- **Screen-edge clamping** needs a world-to-viewport conversion plus a behind-the-camera check. When `viewportPoint.z < 0` the target is behind you and the x/y are mirrored — miss this and off-screen markers appear on the wrong side. This is the single most common bug in every marker implementation ever written.
- **Occlusion is a design decision, not a technical one.** Marker visible through walls = never lost, slightly gamey. Occluded = grounded, occasionally lost. **Recommendation: occluded, with a faint ghosted version at low alpha through geometry.** You get both.
- **Your Day 46 position still stands:** verbal directions first, one marker on the current objective only, no compass, no minimap. 3D does not entitle you to a HUD.
- **The trap:** a marker per objective. Current objective only, or the tone you spent M13 building evaporates.

### Build (40 min)
1. Port the journal canvas prefab from the 2D project. Same panels, same tabs — Quests, Letters. It binds to `QuestLog` in Core and should need almost no change.
2. Verify J opens it, Esc closes it, the world pauses, and the input context switches. Confirm it is blocked during dialogue and during combat.
3. Port `ObjectiveMarker.cs` and `ObjectiveTarget.cs`. Move the marker to world space: a small quad or sprite renderer on a child object, billboarded in `LateUpdate`.
4. Distance fade: full alpha between 8m and 60m, easing to zero outside that range on both ends.
5. Screen-edge clamping: convert to viewport space, handle the `z < 0` mirror case explicitly, clamp to a margin inside the screen bounds, and rotate a small chevron toward the target.
6. Occlusion: a `Physics.Linecast` from camera to target against your environment layer. Hit → draw at 25% alpha with `ZTest Always`; clear → full alpha. **Verify the shader/material setup for depth-test override in URP** rather than assuming a Built-in-pipeline recipe works.
7. Place `ObjectiveTarget` components on the Hearthfall notice board, Osric, and the Vaskirk counting-house door.
8. Walk the village and the road with an active quest. Watch the marker behave at distance, behind buildings, and behind you.

### Acceptance criteria
- [ ] Journal opens and closes in 3D with the world paused and input context switched
- [ ] Letters tab renders Enid's received letters in full
- [ ] Exactly one marker is visible, on the current objective
- [ ] Marker fades correctly at both near and far distances
- [ ] Off-screen targets clamp to the correct screen edge, including behind the player
- [ ] Occlusion behaviour is deliberate and matches what you wrote down

### Failure modes
- **Marker appears on the opposite screen edge** → the `z < 0` case. Negate x and y when the target is behind the camera.
- **Marker rotates oddly near screen edges** → you used `LookAt(camera)` instead of copying camera forward.
- **Marker sits inside the NPC's chest** → offset the target anchor upward; add a `MarkerAnchor` child rather than hardcoding a Y value.
- **Journal list is stale** → it built once on `Awake`. Rebuild on open, same fix as Day 45.

**Stretch:** Add the "Endings seen" tab now, empty and greyed. Tomorrow it gets its first entry, and having the slot ready removes a whole task from Day 103.

**Commit:** `feat: 3d journal and world-space objective markers`

---

## Day 103 — Staging the three endings: camera, light, silence
**Mon 21 Dec · 60 min**

**Objective:** All three endings reachable, staged in 3D, each with its own camera, lighting, and — for one of them — no choice at all.

**Why:** `EndingSelector` is a pure function that already returns the right answer. Everything today is direction. This is the day the game becomes the thing you set out to make.

### Concepts (10 min)
- **Same geometry, different light.** Hearthfall's mesh does not change between the three endings. The Volume profile, the directional light angle and colour, the fog, and the props do. That's the cheapest devastating storytelling available to you and you built the level for it in M13.
- **Ending B's mechanic is the absence of a mechanic.** The game has asked the player what he wants for ten hours. Here it does not ask, because he already answered. No prompt, no choice list, no input except "continue".
- **Restraint is the direction.** No score screen. No villain reveal. No music sting. No stats. Let him sit in the room. Every instinct you have to add feedback here is wrong.
- **The conscience ledger is the payload.** The epilogue quotes his own `ConsciencePoint.Description` strings back at him, in order, with dates. "You named Tam Ferrier in the third week of Lent." A number could never do that; the specificity is the entire reason it lands.
- **Silence is a sound design choice.** Cut the ambient bed to near-nothing for B. Let a hearth crackle and nothing else.
- **Never show a morality meter.** Not now, not in the epilogue, not "for the completionists". Ever.

### Build (40 min)
1. Three scenes or three additive setup prefabs — `Ending_A`, `Ending_B`, `Ending_C` — each loading over the Hearthfall geometry.
2. Per-ending **Volume** profiles. **A:** warm, low afternoon sun, soft bloom, gentle vignette. **B:** desaturated, flat overcast, no bloom, colour grading pushed cold and slightly green. **C:** cold blue pre-dawn, low fog, high contrast. Verify current URP Volume override names in the docs — several were renamed as URP matured.
3. `Unity/Narrative/EndingStager.cs` — reads the ending from Core's `EndingSelector`, loads the matching setup, and drives a short sequence of camera holds. Core decides; Unity stages. Do not put a single ledger comparison in this script.
4. **Ending A:** the hearth, warm. Final shot is a slow push-in on the sword hung above the mantel. Hold four seconds after the last line. He is going to be poor and tired forever and the framing says it was worth it.
5. **Ending B:** interior, cold, still camera, no push. Present the ledger recital — one `ConsciencePoint.Description` per beat, slow, unhurried, no player input beyond continue. **The final choice UI is never instantiated.** Hold the last shot for six seconds of silence before the fade. Six seconds is longer than you think and shorter than it should be.
6. **Ending C:** exterior, morning, the road. A single tracking shot as he walks out of frame. Movement is the whole idea — A and B are static, C is not.
7. Record the ending in `GameState` and surface it in the journal's Endings-seen tab. Seen endings only; never hint at unseen ones.
8. Reach each ending with the Editor State Inspector's scenario-jump buttons. Watch all three back to back. **Sit through B without touching anything.**

### Acceptance criteria
- [ ] All three endings reachable and correctly selected by Core's `EndingSelector`
- [ ] Each ending has a distinct Volume profile and lighting setup over identical geometry
- [ ] Ending B presents no final choice and no score screen
- [ ] Ending B recites at least four specific conscience-ledger entries by description
- [ ] Endings-seen is recorded and shown in the journal
- [ ] No morality value is displayed anywhere in any of them

### Failure modes
- **Ending logic crept into `EndingStager`** → if the word `Conscience` appears in a Unity-side script, move it back to Core.
- **All three look the same** → you changed props but not the light. The light is the storytelling.
- **B feels like a bad end screen** → you added feedback. Remove the sting, remove the fade-to-red, remove the summary. Hold the shot.
- **Volume profile doesn't apply** → the Volume's layer mask doesn't match the camera's, or the profile is Global with priority 0.
- **Ledger entries read as a list of IDs** → your `Description` strings were written for debugging. Rewrite them as sentences. They are player-facing now.

**Stretch:** Write the four lines of Enid's third letter, corrupt version, and put it on the table in Ending B's room as a readable prop. It was homework from M06 and this is where it goes.

**Commit:** `feat: three endings staged in 3d`

---

## Day 104 — Acts I and II wired end to end
**Tue 22 Dec · 60 min**

**Objective:** Play from arriving in Hearthfall to leaving Vaskirk, in 3D, without touching the editor.

**Why:** You have proved every piece separately. Today is the first time the 3D game is a game, and the first time you find out what it actually feels like.

### Concepts (10 min)
- **Content work, not systems work.** Notice how fast this goes. That speed is the return on eleven milestones of architecture.
- **Location transitions in 3D** are scene loads or streamed sections, gated on quest state, with a confirmation on the irreversible ones. "Once you go, you go" still applies.
- **The Three Ledgers conversations are the spine.** Vance in the counting-house, Vance again on the Sheriff's behalf, Iselde at the Grain Hall. Each must be argued well enough that you're tempted while you're testing it.
- **Enid's letters need a delivery moment in 3D.** A courier, or a letter on your bunk at the barracks when you return. Not a popup. It costs ten minutes and it's the game's whole feedback channel.
- **Time the playthrough.** Act I should be 15–20 minutes, Act II longer. If Act I has grown to 40 minutes in the port, cut.
- **Note everything flat.** Same discipline as Day 48. Flat is the category you'll want to skip and the one that matters.

### Build (40 min)
1. Place the Act I conversations on the 3D Hearthfall NPCs: Osric, Enid, Cob. Same JSON, same Core, new bodies.
2. Wire the levy notice as a 3D interactable that starts "A Debt in Hearthfall".
3. The road out of the valley: a transition trigger gated on quest state, with the confirmation prompt.
4. Place the Vaskirk content — Bran at the barracks, Vance in the counting-house, Iselde at the Grain Hall — with their conversations and quest hooks.
5. Wire the three Ledger conversations, each with its honest alternative, and confirm the deferred consequences are queued in Core rather than applied immediately.
6. Letter delivery: a courier NPC or a bunk interactable that fires on the letter-received event, opening the journal to the Letters tab.
7. **Play the whole thing.** No editor, no scenario jumps. Time each act with a stopwatch.
8. Then use the Editor State Inspector scenario-jump buttons to verify each branch: zero ledgers, one, two, all three. Confirm the letter tones change accordingly.

### Acceptance criteria
- [ ] Acts I and II play start to finish in 3D with no editor intervention
- [ ] All three Ledger conversations are reachable, with honest alternatives
- [ ] Consequences queue in Core and do not resolve early
- [ ] All three of Enid's letters deliver with the correct tone per branch
- [ ] Both acts are timed and the numbers written into `PLAYTEST.md`
- [ ] Every flat moment is written down, unfixed

### Failure modes
- **A conversation never triggers** → the interactable's collider is on the wrong layer, or the trigger radius is smaller than your character controller. Check in Scene view with gizmos on.
- **Consequences fire immediately** → something is calling the ConsequenceEngine's resolve instead of enqueueing. Core bug, not a Unity one — the tests will find it.
- **Letters all arrive in the same tone** → the branch count isn't reaching the letter selector. Check in the State Inspector, not with `Debug.Log`.
- **Act I is 40 minutes now** → 3D traversal is slower than 2D. Shorten distances or increase move speed; do not add content.
- **The player is stranded in Vaskirk** → an exit gate whose quest condition can't be satisfied. Walk the quest states in the inspector.

**Stretch:** Play Act II taking every Ledger, then again taking none, back to back. If the clean run doesn't feel painfully poor, the temptation isn't working and the design needs a number changed.

**Commit:** `content: acts i and ii wired in 3d`

---

## Day 105 — BUFFER
**Wed 23 Dec**

- **Catch up.** If any of Days 99–104 slipped, this is where it lands.
- **Fix the flat list from yesterday.** You wrote it down; pick the two that matter most.
- **Polish the endings.** They are the last thing anyone experiences and the reason the game exists.
- **Write the remaining VO lines.** A handful more key moments, generated and imported, costs almost nothing now that the pipeline exists.
- **Rest.**

**This is the last buffer day in the entire curriculum.** There are no more. From Day 106 it is profile, playtest, fix, build, ship — seven consecutive days with a hard date at the end.

It is also two days before Christmas. **Take the 24th, 25th, and 26th off.** Slide the end date to 2 January and do not feel a thing about it. You have been at this for fifteen weeks, you have a 3D action-RPG with three endings and voiced dialogue, and the ship date is a tool you built to serve you, not the other way around. A rested week of shipping beats an exhausted one.

### Milestone review

Run `/review`. Ask specifically whether any ending logic has leaked out of `Hearthfall.Core` into the Unity staging scripts — the classic mistake here is `EndingStager` "just checking" a conscience value to decide which shot to hold.

### Where you are

**The story is in 3D and it ends three different ways.** Conversations are framed over the shoulder with letterboxing and voice. The journal and markers work in a world with walls in it. Acts I and II play end to end. And you can reach a room where the game stops asking you what you want, and reads your own decisions back to you, and holds the shot.

You are 105 days in, 94% through, and everything from here is finishing rather than building.

Tomorrow starts M15, the last one: profile it until it holds frame rate, playtest it properly one final time, fix what that finds, produce a real Windows build, write an itch.io store page that isn't embarrassing, and **ship on Day 112.**

**Commit:** `docs: M14 complete — narrative in 3d with vo and endings`
