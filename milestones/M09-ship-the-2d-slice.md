# M09 · Ship the 2D Slice

**Days 64–70 · 12–18 Nov 2026 · project: `Hollowbrook`**

> Most people who set out to learn Unity never ship anything. They accumulate systems, get bored, and start a new project with better architecture. You are seven days from not being one of them.
>
> Nothing you build this week is a new system. This week is content, audio, art, playtesting, triage, and a build — and the actual skill being taught is **cutting scope**. A vertical slice ships because you decided what wasn't in it, not because you finished everything.
>
> There is no buffer day this milestone. Day 70 replaces it: a build-and-upload day is low cognitive load and doubles as consolidation, which is exactly what a buffer is for.

**You end holding:** a published game with a URL. Mid-point win. This is the day the whole thing stops being a tutorial.

---

## Day 64 — Content pass: fill every gap
**Thu 12 Nov · 60 min**

**Objective:** No dead ends, no placeholder text, no unreachable content. Every path a player can take reaches an actual ending.

**Why:** A missing sound is a shrug. A conversation that stops mid-sentence with no way out is a refund. Content holes are the only bug class that reads as "abandoned".

### Concepts (10 min)
- **A vertical slice is complete along one axis, not all of them.** Short is fine. Ugly is fine. Broken is not.
- **Three buckets, and everything goes in exactly one:** *ship-blocker* (breaks a playthrough), *cut* (delete it today, including the content that references it), *defer to v0.2* (write it down, walk away).
- **Cutting means deleting, not disabling.** A disabled thing that half-exists will confuse you in December. Cut it, and let git remember it.
- **Every dialogue node needs an exit.** Branching content rots silently: you add a choice, never write its consequence, and the node dangles. Nothing warns you.
- **The trap: writing new content today.** You'll want to. New content creates new holes and this is a hole-closing day. If you write, you write only what closes a gap.

### Build (40 min)
1. `git grep -n "TODO\|FIXME\|HACK\|PLACEHOLDER"` across the project. Every hit gets a bucket, written into `SCOPE.md`. Nothing is unassigned.
2. Cut first, while you're honest. Delete the deferred features *and* the dialogue lines, quests, and flags that reference them.
3. Write a validator in Core: walk every dialogue graph and assert that every node either has choices or terminates, that every choice target exists, and that every referenced flag is set somewhere. Run it as a test.
4. Fix everything it finds. This will be more than you expect and it's the highest-value 15 minutes of the week.
5. Same treatment for quests: every quest must be completable *and* failable from a real path, and the journal must render both.
6. Verify **exactly three** endings are reachable: `MorningInHollowbrook`, `NewKeeper`, and first-run-only `JustPassingThrough`. Assert there is no fourth ID or fallback.
7. Run the full `PlaythroughDriver` suite from Day 41, with one complete driver run per ending. The skip run must use five valid reveal-state early advances; add negative tests for Instant reveal and completed profile history.
8. Anything still unshippable at 50 minutes goes into the *cut* bucket. Not *defer*. **Cut.**
9. Audit combat content: one creature archetype, two data variants, one player attack, one dodge. Delete accidental extra enemies or combat systems.

### Acceptance criteria
- [ ] `SCOPE.md` exists; every TODO is bucketed as blocker / cut / defer
- [ ] Cut content is deleted, including everything referencing it
- [ ] The dialogue validator runs as a test and passes
- [ ] No dialogue node dead-ends; no choice points at a missing node
- [ ] Exactly three endings reachable and verified by tests, including the skip ending
- [ ] One creature archetype only, with two data variants
- [ ] Full `PlaythroughDriver` suite green

### Failure modes
- **Validator finds 40 problems** → normal. Fix by frequency, not by file order.
- **Cutting one thing breaks four others** → that's the coupling telling you it was the right cut.
- **You spent the hour writing new dialogue** → re-read the objective. Holes, not content.

**Stretch:** Make the validator fail the build, not just the test run. Content bugs should be as loud as compile errors.

**Commit:** `content: close every gap and cut the rest`

---

## Day 65 — Audio pass
**Fri 13 Nov · 60 min**

**Objective:** Music, ambience, and UI sound throughout, routed through a mixer your settings sliders actually control.

**Why:** Audio is the cheapest perceived-quality upgrade in games, and a silent menu feels broken in a way players can't articulate.

### Concepts (10 min)
- **An `AudioMixer` is a routing graph with exposed parameters.** Groups: Master → Music, SFX, UI. Your Day 60 settings sliders have been controlling nothing until now; today they get something.
- **Volume sliders are logarithmic.** `SetFloat(param, Mathf.Log10(v) * 20f)` with a guard for zero. A linear slider on a dB parameter feels dead for the top 80% and then falls off a cliff.
- **Music must not restart on scene load.** One persistent audio object, crossfade on transitions. This is the most-noticed audio bug in student projects.
- **Ambience is a loop, not an event.** Traffic and diner ventilation in Town Square, insects and trees on Old Mine Road, drips and low resonance in Mercer Mine. Quiet enough that the player doesn't notice it until you mute it.
- **UI sound is what makes menus feel responsive.** Hover, click, back, open, close. Five short sounds, and the whole shell stops feeling like a prototype.
- **Mix at low volume.** If it sounds good quiet, it sounds good loud. The reverse is never true.

### Build (40 min)
1. Create an `AudioMixer` (`Window > Audio > Audio Mixer` — verify this path in Unity 6). Add Music, SFX, and UI groups under Master. Expose each group's volume parameter and rename them clearly.
2. Route every `AudioSource` in the project to the correct group. Any source not on a group is a bug.
3. Wire the settings sliders to `AudioMixer.SetFloat` with the log conversion, and persist the values with the rest of your settings.
4. `MusicManager` — persistent across scenes, `Play(track, fadeSeconds)`, crossfades, never restarts the same track. A restrained Town Square theme, tension for Old Mine Road/Mercer Mine, and deliberate ending cues.
5. Ambience loops for Town Square, Old Mine Road, and Mercer Mine, started by location transition and faded rather than cut.
6. UI sounds on hover, click, back, journal open/close, dialogue advance. Pitch-vary the dialogue blip or it becomes a woodpecker.
7. Source from Freesound, Kenney, Incompetech. **Add every single one to `ATTRIBUTIONS.md` as you import it**, not on Day 69.
8. Play Act I end to end with headphones. Balance until nothing makes you flinch and nothing disappears.

### Acceptance criteria
- [ ] AudioMixer with Master/Music/SFX/UI, every source routed
- [ ] Settings sliders change volume, log-scaled, and persist across restarts
- [ ] Music survives scene loads and crossfades on location change
- [ ] Ambience loops in at least two locations
- [ ] Menus, journal, and dialogue all have UI sound
- [ ] `ATTRIBUTIONS.md` lists every audio file with source, licence, and author

### Failure modes
- **Slider does nothing** → parameter not exposed, or you're setting the group name instead of the exposed parameter name.
- **Slider is silent for most of its travel** → linear value into a dB field. Log-convert it.
- **Music restarts on every load** → your manager isn't persistent, or `Play` doesn't check the current track.
- **Everything clips** → mixer levels stacking. Pull group volumes down, not clips up.

**Stretch:** Duck music by 6 dB while dialogue is open. One mixer snapshot, and it makes conversations feel like they matter.

**Commit:** `feat: audio mixer, music, ambience, and ui sound`

---

## Day 66 — Art pass: make it coherent
**Sat 14 Nov · 60 min**

**Objective:** The game looks like one game made by one person, not five asset packs in a trenchcoat.

**Why:** Coherence beats quality, always. Five mediocre sprites under one palette read as a style; five gorgeous ones from five sources read as an asset flip.

### Concepts (10 min)
- **Pick 16–24 colours and quantise everything to them.** This single step does most of the coherence work, and it works even on art you didn't make.
- **The AI workflow:** generate high-res with a concrete style prompt → downscale hard (64×64 or 128×128, where the mush resolves into shape) → clean the silhouette in Paint/Krita → quantise to the palette.
- **Portraits: one prompt skeleton, swap the subject.** Same framing, same lighting, same era, same colour grade over every output. Consistency is what sells a cast.
- **Silhouette first.** If you can't identify a sprite as a black shape, no amount of interior detail saves it.
- **Timebox brutally. Placeholder is a legitimate outcome.** A consistent grey box is better than a beautiful sprite that doesn't match anything next to it.
- **Don't touch import settings you already settled.** PPU stays where it is. Filter Mode stays where it is.

### Build (40 min)
1. Lock the palette. Build it as a small PNG in `Assets/_Project/Art/palette.png` and commit it. Every future asset gets quantised to this file.
2. Audit what exists: list every sprite, tile, portrait, and UI element, and mark each *fine* / *fix* / *replace*. Ten minutes, no more.
3. Fix the top five by visibility. The player looks at the character, the dialogue box, the portraits, the tileset, and the title screen. In that order.
4. Portraits for Alex Reed, Eli Reed, Mayor Vale, Deputy Pike, Mara Bell, and June Mercer through one prompt skeleton, downscaled, cleaned, quantised, and run through the same colour grade. The Guest Below may remain an obscured environmental presence.
5. Quantise every existing sprite to the palette — Krita's colour-to-alpha and index-mode conversion, or an Aseprite palette apply. Bulk operation, not per-sprite art.
6. One title-screen image. This is the thumbnail on your itch.io page and it does more work than any in-game asset.
7. Add a URP 2D global light and location-specific accents: civic warmth in Town Square, sparse road lighting, and unnatural mine light. Lighting does more for coherence than redrawing anything.
8. **Stop at 50 minutes.** Whatever isn't done stays placeholder and ships that way.

### Acceptance criteria
- [ ] A committed palette file, and everything visible quantised to it
- [ ] Hollowbrook's principal human cast has coherent portraits
- [ ] Title screen art exists
- [ ] The five highest-visibility assets are *fine*, not *fix*
- [ ] `ATTRIBUTIONS.md` updated for anything downloaded
- [ ] You stopped on time

### Failure modes
- **Two hours on one portrait** → this is the failure mode of the whole week. Timer on.
- **Palette applied and everything looks muddy** → too few colours, or no value range. Check you have real darks and real lights.
- **Sprites changed size** → you edited resolution, not just colour. PPU is per-asset; don't fix it with Transform scale.
- **Pixel art went blurry** → something got re-imported with Bilinear filtering.

**Stretch:** Use subtle grading to distinguish ordinary Town Square, uneasy Old Mine Road, and Mercer Mine without making them look like different games.

**Commit:** `art: shared palette, portraits, and title screen`

---

## Day 67 — Playtest properly
**Sun 15 Nov · 60 min**

**Objective:** A written, categorised list of everything wrong with the game — and not one line of it fixed today.

**Why:** You did this on Day 48 with 20 minutes of content. There's now an hour of it, and the observations you make today determine what ships on Wednesday.

### Concepts (10 min)
- **Play as a player, not as the developer.** You know the route that works. Take the stupid one on purpose.
- **Three categories:** *broken* (errors, softlocks, unreachable) · *confusing* (I didn't know what to do) · *flat* (it worked and I felt nothing).
- **Write everything down before fixing anything.** The moment you start fixing, you stop testing, and you'll never get back to the second half.
- **"Flat" is the category you'll skip and the one that matters most.** Especially on the endings — if Ending B doesn't land, that's a ship-blocker dressed as a mood.
- **The highest-value item in this milestone: watch one other human play it, in silence.** Say nothing. Not when they miss the door, not when they don't see the notice board, not when they ask what a button does. Their confusion is data and your explanation destroys it.
- **Note timestamps.** "Bored at 12 minutes" is more actionable than "the middle drags".

### Build (40 min)
1. Fresh save, no editor. **Play the actual build if you have one** — or the editor from the main menu, never from mid-scene.
2. Full playthrough, ~20 minutes, optimal route. Note as you go, in `PLAYTEST.md`, categorised, timestamped. Fix nothing.
3. Second run, ~10 minutes: take every wrong turn, refuse every quest, talk to everyone twice, open the journal at every wrong moment.
4. Third pass: drive all three endings through focused saves or test scenarios. Confirm the skip ending works only on an eligible first profile and the two full endings resolve their authored decisions.
5. Check the Console after every run. Warnings you've been ignoring for three weeks are now candidates.
6. **Get one human to play it while you watch.** Housemate, partner, colleague on a call sharing their screen. Ten minutes is plenty. Say nothing.
7. Write down every place they hesitated. Hesitation is the bug; what they said afterwards is commentary.
8. Sort the whole list by category. Do not triage yet — that's tomorrow, with a clearer head.

### Acceptance criteria
- [ ] Three of your own playthroughs, with different intent
- [ ] `PLAYTEST.md` updated, categorised, timestamped
- [ ] At least one *flat* entry per act — if there are none, you weren't honest
- [ ] One other human played it and you stayed quiet
- [ ] Every hesitation they had is written down
- [ ] Nothing was fixed today

### Failure modes
- **You fixed things mid-test** → the test is over. Note it and restart the pass.
- **You explained the controls** → you just deleted your onboarding data. Next person, hands off.
- **Everything is "confusing"** → you're too close to it. Their list is worth more than yours; weight it accordingly.

**Stretch:** Record the other person's screen and your own reaction. Watching yourself watch them is uncomfortable and extremely instructive.

**Commit:** `docs: playtest findings for the 2d slice`

---

## Day 68 — Fix the list
**Mon 16 Nov · 60 min**

**Objective:** Every ship-blocker fixed. Everything else written into `KNOWN-ISSUES.md` and consciously left alone.

**Why:** Triage is the skill. There are more problems than hours, and picking the right ones is the entire difference between shipping Wednesday and not shipping.

### Concepts (10 min)
- **A ship-blocker is one thing:** it stops a player finishing, or it makes them think the game is broken. That's the whole definition. Ugly is not a blocker. Unbalanced is not a blocker.
- **You have explicit permission to ship with known bugs.** Every game ever released has. Recorded and shipped beats hidden and delayed.
- **The trap is fixing the easy ones.** Six satisfying 5-minute fixes feel like a great hour and leave the softlock in. Sort by severity, then work down. Never sort by effort.
- **Budget the hour before you start.** 40 minutes of fixing, hard stop. Reality checks in at 40 whether you're ready or not.
- **Every fix risks a regression.** Run the `PlaythroughDriver` suite after each one, not at the end.
- **Fix causes, not symptoms, but only today.** A hack that ships is worth more than a refactor that doesn't — flag it in the file and move on.

### Build (40 min)
1. Take `PLAYTEST.md` and mark every item: **BLOCKER** / **KNOWN** / **v0.2**. Ten minutes, and be brutal — expect fewer than eight blockers.
2. Sort the blockers by severity. Softlocks and unreachable endings first. Confusion second. Flat-but-load-bearing story beats third.
3. Fix them in that order. Commit after each one, individually.
4. Run the automated playthrough suite after every fix.
5. At 40 minutes: **stop fixing.** Whatever is left is now KNOWN.
6. Write `KNOWN-ISSUES.md`: what it is, how to reproduce, and whether there's a workaround. Honest and short. This file goes on the itch.io page.
7. Move the v0.2 items into `SCOPE.md` so they survive into the 3D half.
8. One final clean playthrough. If it completes without an intervention, you're shipping.

### Acceptance criteria
- [ ] Every playtest item classified BLOCKER / KNOWN / v0.2
- [ ] All blockers fixed, each in its own commit
- [ ] `KNOWN-ISSUES.md` exists and is honest
- [ ] Automated playthrough suite green
- [ ] One uninterrupted end-to-end playthrough completed
- [ ] You stopped fixing at 40 minutes

### Failure modes
- **Twenty blockers** → your definition has drifted. Re-read it. Most of those are KNOWN.
- **A fix broke something else** → this is why you commit per fix. Revert, and demote the item to KNOWN.
- **You're refactoring** → not today. Note it in `SCOPE.md` and back away.

**Stretch:** Add a version string to the main menu corner, read from a single constant. You will need it the first time someone reports a bug against a build you no longer have.

**Commit:** `fix: ship blockers for v0.1`

---

## Day 69 — Build, package, and write the page
**Tue 17 Nov · 60 min**

**Objective:** A zipped Windows build that runs on a clean path, and an itch.io page written and staged as a draft.

**Why:** The build is 20 minutes. The page is 40, and it's the part that decides whether anyone plays it.

### Concepts (10 min)
- **Build Profiles** (`File > Build Profiles` in Unity 6 — this replaced Build Settings; verify the exact path before you go hunting). Scene list, platform, and per-profile Player Settings live here.
- **The scene list is ordered and index 0 is what launches.** MainMenu first. A black-screen build is almost always this.
- **Player Settings are the shopfront**: product name, company, icon, default resolution, windowed, and turning off the resolution dialog if it's still there.
- **Test the build, not the editor.** Copy it somewhere with no project nearby. Editor-only bugs and build-only bugs are both real and they're different sets.
- **Windows zip, not WebGL.** WebGL builds are real extra work: compression settings, browser audio quirks, longer build times, and a separate round of bugs. You have one hour. Ship the zip; WebGL is a v0.2 line.
- **Your page needs to survive eight seconds of attention.** Title, tagline, one screenshot. Everything else is for people already interested.

### Build (40 min)
1. Player Settings: product name `Last Stop, Hollowbrook`, company name, version `0.1.0`, an icon from your title art, default 1280×720 windowed.
2. Build Profiles: confirm every scene is listed, MainMenu at index 0, nothing orphaned. Build to `Builds/Hollowbrook-v0.1-win64/`.
3. **Copy the build folder somewhere unrelated** — a temp directory, or a USB stick — and run it there. Play to an ending. This catches the missing-data-folder class of bug.
4. Take 3–4 screenshots at 1920×1080: dialogue with a portrait, the journal, a choice moment, the village. No debug overlays, no editor chrome.
5. Zip the build folder (the `.exe` and its `_Data` folder together, or it won't run).
6. Write the itch.io page as a draft: **title**, **tagline** (one line, no genre-speak), a short honest description, **controls**, and an explicit line that this is a vertical slice — roughly 20–30 minutes, three endings, part of a longer project.
7. Link `KNOWN-ISSUES.md` content into a short "Known issues" section. Honesty buys goodwill; discovered bugs cost it.
8. Upload the zip, set the platform to Windows, mark it as executable, and **leave the page as a draft**. Tomorrow it goes live.

### Acceptance criteria
- [ ] Windows build produced, versioned `0.1.0`
- [ ] Build runs from a clean, unrelated path, start to an ending
- [ ] Product name, icon, and default resolution all set deliberately
- [ ] 3–4 clean screenshots
- [ ] itch.io draft page written: title, tagline, description, controls, slice disclaimer, known issues
- [ ] Zip uploaded, page still a draft

### Failure modes
- **Black screen on launch** → wrong scene at index 0, or no camera in it.
- **"Failed to load data file"** → you zipped the `.exe` alone. It needs the whole folder.
- **Missing dialogue or portraits in the build** → assets loaded from a path that only exists in the editor, or not in a `Resources`/addressable-equivalent location.
- **Build is 2 GB** → uncompressed textures, or the whole art scratch folder shipped. Check what's under `Assets/`.
- **Screenshots look flat** → shoot the moments with faces and text in them, not empty terrain.

**Stretch:** Write a 30-second GIF of one dialogue exchange for the top of the page. Motion outperforms every static screenshot.

**Commit:** `build: v0.1 windows build and itch page draft`

---

## Day 70 — SHIP DAY
**Wed 18 Nov · 60 min**

**Objective:** The page goes live. The game has a URL. Then you stop.

This is also this milestone's buffer. A build-and-upload day is low cognitive load and doubles as consolidation, so there's no separate rest day — today is both.

### Ship it (30 min)

1. **Final build.** Rebuild from a clean state so the shipped binary matches the committed code exactly. No last-minute code changes after this point.
2. **Verify it on a clean path.** Fresh folder, no Unity nearby. Launch, play to an ending, quit. If anything is wrong, you rebuild — you do not patch and hope.
3. **Upload the final zip to itch.io**, replacing yesterday's. Windows, executable flagged, price free or name-your-price.
4. **Set the page to Public.** Not restricted, not draft. Public. Copy the URL somewhere you'll find it.
5. **Open the URL in a private browser window** and download it as a stranger would. Confirm the zip is intact and the page reads correctly to someone who knows nothing.
6. **Tag the release:** `git tag -a v0.1-2d -m "2D vertical slice"` and push the tag. This commit is now a permanent, reachable point in the project's history.
7. **Send it to three specific people.** Not a broadcast post — three names, individually. One who plays games, one who doesn't, and one who will tell you the truth. Ask for one sentence back.

### Then stop (30 min)

Genuinely stop. The half-hour is not for adding anything.

1. Write a real `progress/LOG.md` entry. Not "shipped it". What you built over ten weeks, what surprised you, what you'd do differently, and how today feels. You'll want this on day 95.
2. Update `progress/STATE.md`: current day 70, milestone M09 complete, released `v0.1-2d`, the itch URL, and the contents of `SCOPE.md` under `parked`.
3. Add the URL to the top of `README.md`.
4. Close the editor. Take the rest of the hour off. If you feel the urge to fix one small thing, write it in `SCOPE.md` instead — it's a v0.2 item now and it'll still be there tomorrow.

### Where you are

Seventy days in, roughly 62% through, and **you have shipped a game**. It is on the internet, it has a URL, and strangers can download it. That is not a tutorial outcome.

Specifically: an engine-agnostic `Hollowbrook.Core` with no `using UnityEngine` anywhere in it. Branching dialogue with portraits, accessible typewriter text, and Mayor Vale's first-run patience system. Four central choices with delayed town reactions. `Where Is Eli?`, clues, and a journal. One improvised attack, one dodge, and one creature archetype with two variants. Save/load with decision and profile history. A full game shell. Exactly three endings, including the skip ending.

Tomorrow you start the 3D half, and parts of it will feel like starting over. Cameras, lighting, animation, navigation, physics in three dimensions — all new, and some of it will be humbling after ten weeks of feeling competent.

But `Hollowbrook.Core` does not care what dimension it's rendered in. That was the whole point of M03. **Day 75 is the payoff**: a 3D scene, a camera behind Alex's shoulder, and your dialogue system running — the same code, untouched, not one line changed.

### Acceptance criteria
- [ ] Final build produced from a clean state and verified on an unrelated path
- [ ] itch.io page is **public** and the download works from a private browser session
- [ ] Git tag `v0.1-2d` created and pushed
- [ ] Sent to three named people with a request for one sentence each
- [ ] `progress/LOG.md` has a real entry, not a one-liner
- [ ] `progress/STATE.md` and `README.md` updated with the release and the URL
- [ ] You stopped working

**Commit:** `docs: M09 complete — v0.1 2d slice shipped`
