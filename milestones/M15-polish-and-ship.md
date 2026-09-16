# M15 · Polish, Optimise, Ship

**Days 106–112 · 24–30 Dec 2026 · project: `Hollowbrook3D`**

> Nothing new gets built this week. You are shipping a choice-driven supernatural RPG: four reusable modern locations, dialogue and quests driven by `Hollowbrook.Core`, one creature archetype with two variants, one improvised attack, one dodge, generous aim assist, and exactly three endings.
>
> **Days 106–108 are 24, 25 and 26 December.** Taking them off slides the end to 2 January 2027 without changing day order or scope.

**You end holding:** a released 3D version of *Last Stop, Hollowbrook* with your name on it.

---

## Day 106 — The Profiler
**Thu 24 Dec · 60 min**

**Objective:** Measure all four locations and representative dialogue/encounter frames, then fix the two largest verified costs.

### Concepts (10 min)
- Establish CPU- versus GPU-bound before choosing a fix.
- Look for per-frame allocation, repeated lookups, realtime shadows, transparent overdraw, and imported mesh outliers.
- Static batching and occlusion suit Town Square and Town Hall; do not assume they help open road sections.
- A spike is more disruptive than a modestly low average.

### Build (40 min)
1. Capture Town Square, Town Hall, Old Mine Road, and Mercer Mine in Unity 6's Profiler.
2. Capture a dialogue frame and a two-variant encounter frame.
3. Record CPU/GPU status, top five costs, allocations, draw calls, and expensive assets.
4. Fix only the top two measured costs.
5. Re-capture and keep a change only if the numbers improve.
6. Set target frame rate and v-sync deliberately.

### Acceptance criteria
- [ ] Four locations and representative gameplay states are captured
- [ ] CPU/GPU bottleneck and top costs are documented
- [ ] Two fixes are verified by before/after measurements
- [ ] No per-frame allocation remains in obvious hot paths
- [ ] Target frame rate and v-sync are explicit
- [ ] Fixes do not alter Core behaviour

**Commit:** `perf: profile Hollowbrook and fix measured costs`

---

## Day 107 — Full playtest, every path
**Fri 25 Dec · 60 min**

**Objective:** Produce a timestamped broken/confusing/flat list without fixing anything.

### Concepts (10 min)
- One uninterrupted run proves pacing; scenario jumps prove branch coverage.
- Test concrete decisions, prepared-town outcomes, and bargain terms rather than a hidden score.
- The early ending requires its own fresh first-playthrough profile.
- Console warnings count as findings.

### Build (40 min)
1. Fresh profile: complete one uninterrupted full run to *Morning in Hollowbrook* or *The New Keeper*.
2. Use scenario tools to reach and watch the other full ending.
3. Fresh eligible profile: repeatedly skip Vale's briefing and watch annoyance, warning, deputies, ejection, bus, statistics, credits, and badge.
4. Verify Fast/Instant text, accessibility reveal, and a profile with a completed ending cannot trigger *Just Passing Through*.
5. Fight both variants using only the attack, dodge, and automatic facing; include one death.
6. Test every cast member, location, quest transition, save/load boundary, and VO fallback.
7. Categorise and timestamp everything; fix nothing.

### Acceptance criteria
- [ ] One genuine full-story run is complete
- [ ] *Morning in Hollowbrook* and *The New Keeper* are both observed
- [ ] *Just Passing Through* is observed on an eligible fresh profile
- [ ] Skip-ending fairness exemptions are verified
- [ ] Both creature variants and all four locations are covered
- [ ] Findings and console warnings are recorded
- [ ] Nothing is fixed today

**Commit:** `docs: record Hollowbrook v1 playtest findings`

---

## Day 108 — Fix pass 1: ship blockers
**Sat 26 Dec · 60 min**

**Objective:** Fix or design around anything that prevents a player reaching any of the three endings.

### Concepts (10 min)
- A blocker stops completion or convincingly makes the game appear broken.
- Prioritise softlocks, save/profile corruption, unreachable endings, then critical confusion.
- Cut content before growing systems during ship week.
- Run branch tests after each fix and stop at 40 minutes.

### Build (40 min)
1. Classify every finding BLOCKER / KNOWN / v1.1.
2. Fix softlocks, save/history corruption, and ending eligibility first.
3. Verify both full endings and the first-run-only route after each relevant fix.
4. Design around any blocker that cannot be fixed in fifteen minutes.
5. Stop at 40 minutes and write concise known issues.

### Acceptance criteria
- [ ] Every finding is classified
- [ ] All blockers are fixed or designed around
- [ ] Automated playthrough and ending-fairness tests are green
- [ ] One uninterrupted run succeeds after fixes
- [ ] Known issues are honest and shippable
- [ ] Fixing stops at the time limit

**Commit:** `fix: Hollowbrook v1 ship blockers`

---

## Day 109 — Fix pass 2, settings, and rebinding
**Sun 27 Dec · 60 min**

**Objective:** Resolve the top confusing findings and verify persistent display, audio, accessibility, camera, and input settings.

### Concepts (10 min)
- Use the Input System's current interactive rebinding API and persist override JSON.
- Offer resolution, fullscreen, v-sync, modest quality presets, and mixer controls.
- Keep subtitles and full-line accessibility reveal independent of VO.
- Camera sensitivity and invert Y matter for both exploration and short encounters.

### Build (40 min)
1. Fix the top three confusing findings.
2. Verify display and quality settings persist and apply on startup.
3. Verify Master, Music, SFX, Voice, and Ambience controls.
4. Verify text speed, subtitles, accessibility reveal, marker mode, camera sensitivity, and invert Y.
5. Rebind every action, support cancel/reset, persist overrides, and restart twice.
6. Re-run Mayor skip fairness tests after changing text/input settings.

### Acceptance criteria
- [ ] Top three confusing findings are fixed
- [ ] Display, quality, and audio settings persist
- [ ] Camera and accessibility settings persist
- [ ] Every action can be rebound, cancelled, and reset
- [ ] Rebinds survive restart
- [ ] No accessibility setting can trigger the early ending

**Commit:** `feat: finish settings and input rebinding`

---

## Day 110 — Build it, then test the build
**Mon 28 Dec · 60 min**

**Objective:** A clean Windows build of `Last Stop, Hollowbrook` runs away from the Unity project through every ending route.

### Concepts (10 min)
- Editor-only APIs, scene lists, content paths, and save locations commonly fail only in builds.
- Use `Application.persistentDataPath` for saves and profile history.
- Product identity is `Last Stop, Hollowbrook`; project/repository identity remains `Hollowbrook3D`.
- Build size should reflect four compact sets and limited VO, not forgotten source assets.

### Build (40 min)
1. Set product name `Last Stop, Hollowbrook`, company, icon, and version `1.0.0`.
2. Verify Unity 6 Build Profiles and scene order.
3. Build to `Builds/Last-Stop-Hollowbrook-v1.0-win64/`.
4. Run from an unrelated clean path.
5. Complete one full ending; use fresh/profile save fixtures to verify the other full ending and early route.
6. Relaunch and verify save, settings, bindings, ending history, badge, JSON, and VO.
7. Test another resolution or machine.

### Acceptance criteria
- [ ] Versioned Windows build runs from a clean path
- [ ] All current content loads outside the editor
- [ ] Both full endings work in the build
- [ ] Early route and later-playthrough lockout work in the build
- [ ] Save/profile/settings survive relaunch
- [ ] Build size is understood

**Commit:** `build: verify Last Stop Hollowbrook v1.0.0`

---

## Day 111 — Store page, screenshots, and a gif
**Tue 29 Dec · 60 min**

**Objective:** A draft itch.io page accurately presents a short choice-driven supernatural RPG.

### Concepts (10 min)
- Lead with title, premise, and one moving image.
- Show choices, cast, ordinary town spaces, supernatural mine pressure, and restrained combat.
- Describe exactly three endings without spoiling the early route's trigger.
- State honest playtime, accessibility, controls, asset/VO credits, content warnings, and known issues.

### Build (40 min)
1. Use `Last Stop, Hollowbrook` as the title and write a literal one-line premise.
2. Capture a short gif centred on dialogue choice or town reaction, not extended fighting.
3. Capture Town Square, Town Hall, Old Mine Road, Mercer Mine, a conversation, and one simple encounter.
4. Describe four major choices, two full conclusions, and one secret first-run ending.
5. State that combat is one improvised attack, one dodge, aim assist, and one creature with two variants.
6. Add controls, accessibility, credits, attributions, content warnings, known issues, and playtime.
7. Upload the zip and leave the page as draft.

### Acceptance criteria
- [ ] Page calls the game a choice-driven supernatural RPG
- [ ] Gif foregrounds story or consequence
- [ ] Screenshots cover all four modern locations
- [ ] Scope and playtime are honest
- [ ] Exactly three endings are promised
- [ ] Combat description matches the shipped simple system
- [ ] Credits and accessibility details are present

**Commit:** `docs: stage Last Stop Hollowbrook itch page`

---

## Day 112 — SHIP DAY
**Wed 30 Dec · 60 min**

**Objective:** Publish the verified build, record the release, and stop.

### Ship it (30 min)
1. Rebuild from a clean committed state; make no feature changes afterward.
2. Verify launch, content, save/profile history, and one representative route from a clean folder.
3. Upload the final zip and set the itch.io page Public.
4. Download from a private browser session and verify the archive.
5. Tag the release: `git tag -a v1.0 -m "Last Stop, Hollowbrook 1.0 — 3D release"` and push it.
6. Link the Day 70 2D release to the 3D release and send the new URL individually.
7. Update the required progress and release documentation, then read the full log.

### Acceptance criteria
- [ ] Final clean build is verified
- [ ] Public itch.io download works as a stranger
- [ ] Git tag `v1.0` is pushed
- [ ] 2D and 3D release pages link to each other
- [ ] Release is sent to named people
- [ ] Progress state is complete and both URLs are recorded

### Where you are

You shipped the same choice-driven supernatural RPG twice: first in 2D, then as `Hollowbrook3D`. `Hollowbrook.Core` survived the presentation rewrite and still owns dialogue, quests, four central decisions, town preparation, bargain terms, profile history, exactly three endings, and the small M07 combat rules.

The 3D release has Town Square, Old Mine Road, Mercer Mine, and a Town Hall office that opens the story and mirrors its compromised conclusion. Alex has one improvised attack, one dodge, generous automatic facing, and encounters one creature archetype in two data-tuned forms. The story, not combat progression, is the game.

Read `progress/LOG.md`, write the final entry, and close the project in a committable state.

**Commit:** `docs: M15 complete — Last Stop Hollowbrook v1.0 shipped`
