# M14 · Narrative in 3D + Kokoro VO

**Days 99–105 · 17–23 Dec 2026 · project: `Hollowbrook3D`**

> Dialogue graphs, conditions, deferred consequences, quests, `Decisions`, `PreparedTown`, `BargainTerms`, and `PlaythroughHistory` already live in `Hollowbrook.Core`. This week ports their presentation and stages exactly three endings.

**You end holding:** filmically framed Hollowbrook conversations, the journal and quests in 3D, and complete staging for *Morning in Hollowbrook*, *The New Keeper*, and *Just Passing Through*.

---

## Day 99 — Dialogue cameras
**Thu 17 Dec · 60 min**

**Objective:** Conversations with Mayor Vale, Deputy Pike, Mara Bell, June Mercer, and Eli Reed use one reusable over-the-shoulder camera rig.

### Concepts (10 min)
- Use one Cinemachine rig parameterised by speaker/listener anchors, not one camera per conversation.
- Latch one side of the 180-degree line for the conversation.
- Blend by priority and recover gracefully when an interior angle is blocked.
- Core emits conversation events; Unity frames bodies in Town Hall, Town Square, the road, and the mine.

### Build (40 min)
1. Verify installed Cinemachine 3.x APIs against current docs.
2. Build one dialogue-camera prefab with reusable speaker/listener framing.
3. Subscribe to existing conversation start/end events.
4. Add collision fallback and head anchors to the modern cast.
5. Test Vale's Town Hall office, Mara's diner, June on Old Mine Road, and Eli in Mercer Mine.

### Acceptance criteria
- [ ] One rig frames every conversation
- [ ] Blends enter and leave dialogue cleanly
- [ ] Camera remains on one side of the 180-degree line
- [ ] Tight Town Hall framing avoids walls and faces
- [ ] Input contexts switch and restore correctly
- [ ] No narrative condition exists in camera code

**Commit:** `feat: frame Hollowbrook conversations in 3d`

---

## Day 100 — Port the dialogue UI to 3D
**Fri 18 Dec · 60 min**

**Objective:** The shipped dialogue UI runs against the same Core runner, adapted to cinematic 3D framing and multiple aspect ratios.

### Concepts (10 min)
- Keep uGUI this late in the schedule; port the working contract.
- Reduce the panel to the bottom third so performances remain visible.
- Keep speaker name, subtitles, choices, typewriter reveal, and accessibility settings authoritative.
- Portrait/expression keys still matter for Mayor skip escalation even if the standard layout favours animated faces.

### Build (40 min)
1. Port the 2D dialogue prefab and bind it to unchanged Core events.
2. Fit safe areas and test 16:9, 16:10, and ultrawide.
3. Add restrained letterboxing tied to conversations.
4. Preserve normal reveal, instant/reveal accessibility modes, and choice input.
5. Map Vale's neutral, annoyed, sharper, and furious presentation states to portrait and/or facial animation assets.

### Acceptance criteria
- [ ] Unchanged Core runner drives the 3D dialogue UI
- [ ] UI never covers the active speaker's face
- [ ] Choices and subtitles remain readable at tested aspect ratios
- [ ] Accessibility reveal modes behave exactly as in 2D
- [ ] Vale's irritation stages have distinct presentation assets
- [ ] uGUI decision is documented

**Commit:** `feat: port Hollowbrook dialogue UI to 3d`

---

## Day 101 — Kokoro VO and subtitle fallback
**Sat 19 Dec · 60 min**

**Objective:** Selected key lines play voice clips keyed by dialogue node while all unvoiced lines and subtitles remain first-class.

### Concepts (10 min)
- Key audio by conversation/node ID, never line text.
- Missing VO is normal; subtitles are always authoritative.
- Advance remains player-driven and must stop the previous clip cleanly.
- Voice is limited to important Vale, Pike, Mara, June, Eli, and Guest Below beats.

### Build (40 min)
1. Import mono, appropriately compressed clips and log sources/voice generation.
2. Build an inspectable `VoiceBank` keyed by node ID.
3. Subscribe a voice player to the runner's line event.
4. Sync reveal duration when a clip exists; preserve text-speed behaviour otherwise.
5. Route VO through a dedicated mixer group.
6. Test voiced, unvoiced, skipped, interrupted, and accessibility-instant lines.

### Acceptance criteria
- [ ] Voiced lines and subtitles remain synchronized
- [ ] Unvoiced lines produce no warning or behavioural change
- [ ] Advancing stops current VO without overlap
- [ ] Voice volume is separately configurable
- [ ] Accessibility text presentation never depends on audio
- [ ] Clips and tools are attributed honestly

**Commit:** `feat: add selective Kokoro VO with subtitle fallback`

---

## Day 102 — Port quests, journal, and markers
**Sun 20 Dec · 60 min**

**Objective:** Existing quests and journal run in 3D with one restrained marker for the current objective.

### Concepts (10 min)
- Port `QuestLog` bindings; do not redesign quest state in Unity.
- One marker, no minimap: verbal directions remain primary.
- Billboarding, distance fade, screen-edge handling, and deliberate occlusion make a 3D marker usable.
- The four central choices must resolve at their real places and cite later town reactions.

### Build (40 min)
1. Port the journal and bind active/completed/failed states to Core.
2. Add current-objective targets for Deputy Pike's files, Mercer Mine entrance, June Mercer, and the emergency siren.
3. Implement billboard, near/far fade, edge clamp, behind-camera correction, and occlusion treatment.
4. Verify quest updates and delayed town reactions across Town Square, Old Mine Road, Mercer Mine, and Town Hall.
5. Block journal opening during dialogue and active encounters.

### Acceptance criteria
- [ ] Existing quest state ports without Core changes
- [ ] Journal displays current and resolved objectives
- [ ] Exactly one current-objective marker appears
- [ ] Marker handles distance, occlusion, and behind-camera targets
- [ ] All four central choice locations are wired
- [ ] Town reactions cite decisions rather than a morality score

**Commit:** `feat: port Hollowbrook quests and journal to 3d`

---

## Day 103 — Stage exactly three endings
**Mon 21 Dec · 60 min**

**Objective:** All three story-bible endings are fully staged, with the early dialogue-skip route treated as a real ending.

### Concepts (10 min)
- Core selects endings from decisions, town preparation, bargain terms, and profile history; Unity only stages the result.
- *Morning in Hollowbrook* is costly and hopeful: damage, survivors, Eli alive but changed, and the diner reopening at dawn.
- *The New Keeper* is calm and compromised: Alex accepts the final offer and the Town Hall office mirrors Vale's opening shot.
- *Just Passing Through* is first-playthrough only. Fast/Instant text and accessibility full-line reveal are exempt, and completed profiles permanently disable punishment.

### Build (40 min)
1. **Mayor skip escalation:** stage neutral patience at counts 0–1; annoyed expression and “Am I keeping you?” at 2; sharper expression and a slow restart at 3; furious expression, final warning, and deputies entering at 4; rage, meeting termination, and ejection at 5.
2. Keep warning lines in the dialogue graph. Core emits irritation stage; Unity owns expression, voice, deputies, sound, and camera.
3. Stage Deputy Nora Pike plus a second deputy escorting Alex out of Town Hall, onto the last bus, with the driver warned not to stop again.
4. Roll *Just Passing Through* credits with serious statistics: distance travelled, mysteries solved `0`, townspeople saved `0`, and time in office `under three minutes`; unlock its badge and normal New Game.
5. Stage *Morning in Hollowbrook* across damaged Town Square at dawn, varying survivors and rebuilding details from `PreparedTown` and decisions.
6. Stage *The New Keeper* in Vale's office with Alex behind the same desk, varying the epilogue from `BargainTerms` and decisions as a newcomer enters.
7. Record endings in profile history and show only seen endings.
8. Run automated fairness checks: fully revealed advances, Fast/Instant text, accessibility reveal, and later playthroughs cannot trigger the skip ending.

### Acceptance criteria
- [ ] Exactly three endings are reachable and named correctly
- [ ] *Morning in Hollowbrook* has full dawn, survivor, Eli, and rebuilding staging
- [ ] *The New Keeper* has full bargain epilogue and mirrored Town Hall staging
- [ ] Vale visibly escalates annoyed → sharper → furious before ejection
- [ ] Deputies, bus, deadpan statistics, badge, and credits complete *Just Passing Through*
- [ ] Skip ending is first-playthrough only and accessibility-exempt
- [ ] No morality score or ending selection logic appears in Unity staging

**Commit:** `feat: stage all three Hollowbrook endings`

---

## Day 104 — Wire the full story end to end
**Tue 22 Dec · 60 min**

**Objective:** Play arrival, Mayor briefing, sibling trail, four central choices, town preparation, mine bargain, and either full ending without editor intervention; separately verify the early route.

### Concepts (10 min)
- This is content integration, not new systems work.
- Decisions and town preparation replace abstract conscience scoring.
- Bargain terms determine the texture of keeperhood, not whether a hidden morality threshold was met.
- Location transitions are short and quest-gated across the four reusable sets.

### Build (40 min)
1. Place Vale, Pike, Mara, June, Eli, and the Guest Below at their story-bible locations.
2. Wire the Mayor briefing and sibling trail through Town Square and Town Hall.
3. Wire Pike's files, the mine entrance, June's fate, and the emergency siren.
4. Connect Old Mine Road encounters and Mercer Mine investigation to the Guest's final bargain.
5. Play one uninterrupted full run and time each act.
6. Use scenario tools to verify contrasting `PreparedTown` and `BargainTerms` epilogues.
7. Start a fresh profile and verify the five-skip route through bus and credits.

### Acceptance criteria
- [ ] Full story plays start to finish without editor intervention
- [ ] Four central choices visibly affect later scenes
- [ ] Town preparation changes the final night and Morning epilogue
- [ ] Bargain terms change the Keeper epilogue
- [ ] All named cast and four locations serve their story-bible roles
- [ ] Early skip route works only on an eligible fresh profile
- [ ] Timings and flat moments are recorded

**Commit:** `content: wire Last Stop Hollowbrook end to end`

---

## Day 105 — BUFFER
**Wed 23 Dec**

- Finish integration or fix the two flattest story beats.
- Rehearse all three endings without skipping their holds and credits.
- Add VO only where it improves an already complete beat.
- Re-run skip fairness and full-playthrough tests.

### Milestone review

Run `/review`. Check for ending logic outside `Hollowbrook.Core`, a fourth implied ending, inaccessible skip punishment, or epilogues that cite scores instead of concrete decisions, prepared-town outcomes, and bargain terms.

### Where you are

The entire choice-driven supernatural story now plays in 3D. Its two full conclusions have complete staging, and its one committed joke escalates from Vale's annoyance to deputies, bus, credits, and a permanent profile badge without punishing accessibility or repeat players.

**Commit:** `docs: M14 complete — Hollowbrook narrative in 3d`
