# M15 · Polish, Optimise, Ship

**Days 106–112 · 24–30 Dec 2026 · project: `Hearthfall3D`**

> Nothing new gets built this week. Everything this week is measurement, triage, and packaging — the unglamorous work that separates "a project on my hard drive" from "a game people played". You did this once already on Day 70 with 20 minutes of 2D content. This time it's a 3D action-RPG with three locations, three enemy types, voiced dialogue and three endings, and the difference is mostly in the profiler and the build.
>
> **Days 106–108 are 24, 25 and 26 December.** If you want them off, take them. You will have earned three days by then and you will not be a worse developer for spending Christmas as a person. Slide the end date to 2 January 2027, keep the day order exactly as written, and don't renegotiate anything else. That's the whole conversation.

**You end holding:** a released 3D action-RPG with your name on it.

---

## Day 106 — The Profiler
**Thu 24 Dec · 60 min**

**Objective:** A written list of what is actually slow in your game, measured, with the top two fixed.

**Why:** You have opinions about what's slow. They are almost certainly wrong, and every hour spent optimising the wrong thing is an hour of your last week gone.

### Concepts (10 min)
- **Measure, then fix. Never the reverse.** The classic trap is the "obvious" optimisation — pooling something that allocates twice a minute while a single realtime shadow casts on 400 objects.
- **CPU-bound or GPU-bound first.** They have completely different fixes. The Profiler's CPU and Rendering modules answer this in about thirty seconds; everything else follows from the answer.
- **The real culprits in a project like yours:** `GetComponent` or `Find` called in `Update`; per-frame allocations producing GC spikes (string concatenation in UI, LINQ in hot paths, `new` in an update loop); realtime shadows from too many lights; overdraw from transparent particles and fog cards; imported assets at absurd poly counts because a free pack shipped them that way.
- **The three cheap structural wins:** static batching for anything that never moves, occlusion culling so Greyhold isn't rendering while you're in the Wealdrun, and LODs on the expensive props.
- **Set a target frame rate and mean it.** 60 on your machine, and decide your vsync position. A game with no target has no definition of "fast enough" and you will optimise forever.
- **A spike is worse than a low average.** Players feel the hitch, not the number.

### Build (40 min)
1. Open the Profiler (`Window > Analysis > Profiler` — **verify this path against current Unity 6 docs before hunting for it**). Attach to a Play Mode session and record 30 seconds in each of the three locations.
2. Determine CPU-bound vs GPU-bound. Write the answer down before you touch anything.
3. Sort the CPU hierarchy by self-time. Screenshot or note the top five entries. Do the same for the GC Alloc column — anything allocating per frame is a candidate.
4. Open the Frame Debugger (`Window > Analysis > Frame Debugger` — verify) during a busy combat frame. Count draw calls and note what's issuing them. Batch-breaking materials show up here immediately.
5. Fix the top two named items only. Cache component references in `Awake`, hoist allocations out of `Update`, and turn off shadow casting on lights that don't need it.
6. Mark all static geometry as **Static** in the Inspector and bake occlusion culling for at least one location.
7. Set `Application.targetFrameRate` and your vsync setting deliberately, in one place, at startup.
8. Re-record. Confirm the numbers moved. If they didn't, revert the change — an unmeasured "optimisation" is just risk.

### Acceptance criteria
- [ ] Profiler captures recorded for all three locations
- [ ] CPU-bound vs GPU-bound established and written down
- [ ] Top five CPU costs and any per-frame allocations documented
- [ ] Two measured fixes applied, each verified by a re-capture
- [ ] Static flags set and occlusion culling baked for one location
- [ ] Target frame rate set explicitly in code

### Failure modes
- **Frame rate worse in the Profiler** → the Profiler costs frames. Compare like with like, or use Deep Profile only for finding names, never for numbers.
- **Everything is "Other"** → you need Deep Profile for one run to get real names, then turn it back off.
- **Fixed something and nothing changed** → you weren't bound by it. Revert and go back to the capture.
- **Bake made it worse** → occlusion culling on wide-open terrain costs more than it saves. It's for interiors and dense geometry.

**Stretch:** Add LOD groups to your five most expensive props. Free assets rarely ship with them and it's often the largest single GPU win available.

**Commit:** `perf: profiler pass and two measured fixes`

---

## Day 107 — Full playtest, every path
**Fri 25 Dec · 60 min**

**Objective:** A complete, categorised written record of everything wrong with the game — with nothing fixed.

**Why:** This is the last honest look you get. What you observe today decides what ships on Wednesday, and observation is destroyed the moment you start fixing.

### Concepts (10 min)
- **Play as a player.** You know the route that works. Take the stupid one deliberately — refuse quests, talk to everyone twice, attack the wrong people, walk into geometry.
- **Three categories, same as Day 67:** *broken* (errors, softlocks, unreachable content) · *confusing* (I didn't know what to do) · *flat* (it worked and I felt nothing).
- **"Flat" is the category you'll skip and the one that matters most.** In a game whose entire pillar is deferred consequence, an ending that doesn't land is a ship-blocker wearing a mood as a disguise.
- **One genuine unbroken run beats five scenario jumps.** The Editor State Inspector buttons are for coverage; a real start-to-finish playthrough is for pacing, and only the real run tells you where it drags.
- **Timestamp everything.** "Lost at 34 minutes in the Wealdrun" is actionable. "The middle drags" is not.
- **The Console counts.** Warnings you've scrolled past since November are candidates today.

### Build (40 min)
1. Fresh save, main menu, no scene-jumping. **One complete run, start to an ending, uninterrupted.** Note as you go in `PLAYTEST.md`, categorised and timestamped. Fix nothing.
2. Use the Editor State Inspector scenario buttons from Day 40 to force the ledger states for the other two endings. Play each ending from its last checkpoint and confirm it fires, stages correctly, and reads.
3. Combat pass: fight all three enemy types. Check lock-on with two enemies, dodge cancels, stamina exhaustion, and what happens when you die.
4. Dialogue pass: at least one conversation per major NPC, one with VO and one without, checking framing, subtitles and the fallback path.
5. Break it on purpose: pause mid-attack, open the journal mid-conversation, save and reload inside a fight, alt-tab during a cutscene camera.
6. Read the Console after every run. Every warning gets a line in `PLAYTEST.md` or an explicit "known and ignored".
7. Run the full `PlaythroughDriver` suite. Core failing on the last week is rare and very loud when it happens.
8. Sort the list by category. **Do not triage today.** That's tomorrow, with a clearer head.

### Acceptance criteria
- [ ] One genuine uninterrupted start-to-finish playthrough completed
- [ ] All three endings reached and observed
- [ ] All three enemy types fought, including a death
- [ ] `PLAYTEST.md` updated, categorised, timestamped
- [ ] At least one *flat* entry per act
- [ ] Console warnings triaged into the file; automated suite green
- [ ] Nothing was fixed today

### Failure modes
- **You fixed something mid-run** → the run is contaminated. Note it and restart the pass.
- **No *flat* entries** → you weren't honest. There are always some.
- **Only reached one ending** → the scenario buttons exist for exactly this. Use them.

**Stretch:** Get one other human to play twenty minutes while you watch in silence. Their hesitations are the highest-value data available to you this week.

**Commit:** `docs: full playtest findings for v1.0`

---

## Day 108 — Fix pass 1: ship-blockers
**Sat 26 Dec · 60 min**

**Objective:** Every ship-blocker fixed, and everything else consciously written down and left alone.

**Why:** There are more problems than hours. Picking the right ones is the entire difference between shipping Wednesday and not shipping.

### Concepts (10 min)
- **A ship-blocker is one thing:** it stops a player finishing, or it makes them believe the game is broken. That's the whole definition. Ugly is not a blocker. Unbalanced is not a blocker. A weightless heavy attack is not a blocker.
- **The trap is fixing the easy ones.** Six satisfying five-minute fixes feel like a great hour and leave the softlock in the Wealdrun untouched. Sort by severity. Never by effort.
- **You have explicit permission to ship with known bugs.** Every game ever released has. Recorded and shipped beats hidden and delayed, and an honest `KNOWN-ISSUES.md` on the store page buys goodwill that a discovered bug destroys.
- **Budget before you start.** 40 minutes of fixing, hard stop. Reality arrives at 40 whether or not you're ready.
- **Every fix risks a regression.** Commit per fix and run the suite after each, not at the end.
- **Fix causes, not symptoms — but only today.** A hack that ships beats a refactor that doesn't. Flag it in the file and move on.

### Build (40 min)
1. Mark every item in `PLAYTEST.md` as **BLOCKER** / **KNOWN** / **v1.1**. Ten minutes, and be brutal — expect fewer than eight blockers.
2. Sort the blockers: softlocks and unreachable endings first, save/load corruption second, confusion third, load-bearing story beats that fell flat fourth.
3. Fix in that order. One commit per fix.
4. Run the `PlaythroughDriver` suite after every fix. If a fix breaks something else, revert it and demote the item to KNOWN.
5. Any blocker you cannot fix in fifteen minutes gets *designed around* instead — block the route, remove the interactable, cut the line. Cutting is a legitimate fix.
6. At 40 minutes: **stop fixing.** Everything left is now KNOWN, by definition.
7. Write `KNOWN-ISSUES.md`: what it is, how to reproduce, whether there's a workaround. Short and honest. This goes on the itch.io page.
8. Move the v1.1 items into `SCOPE.md` so they survive past Day 112.

### Acceptance criteria
- [ ] Every playtest item classified BLOCKER / KNOWN / v1.1
- [ ] All blockers fixed or designed around, each in its own commit
- [ ] `KNOWN-ISSUES.md` exists and is honest
- [ ] Automated playthrough suite green
- [ ] One uninterrupted end-to-end run after the fixes
- [ ] You stopped fixing at 40 minutes

### Failure modes
- **Twenty blockers** → your definition drifted. Re-read it; most of those are KNOWN.
- **You're refactoring** → not this week. `SCOPE.md`, and back away.
- **A fix broke two other things** → revert. Late-week fixes have a bad risk profile and you have no time to absorb a cascade.

**Stretch:** Add a version string to the main menu corner from a single constant. You will want it the first time someone reports a bug against a build you no longer have.

**Commit:** `fix: v1.0 ship blockers`

---

## Day 109 — Fix pass 2, settings, and rebinding
**Sun 27 Dec · 60 min**

**Objective:** The second tier of fixes done, and a settings screen that lets a player actually configure the game — including rebinding every key.

**Why:** Resolution, quality, volume and rebinding are not decoration. They're the difference between "unplayable on my setup" and "played it", and rebinding specifically is the single highest-impact accessibility feature you can add in an hour.

### Concepts (10 min)
- **Second-tier fixes are the confusing ones.** Broken is gone; today you fix the moments where a player didn't know what to do.
- **Resolution and fullscreen** go through `Screen.SetResolution` and `Screen.fullScreenMode`, populated from `Screen.resolutions`. Offer borderless as well as exclusive fullscreen — alt-tabbing out of exclusive fullscreen is where builds go to hang.
- **Quality presets** are `QualitySettings` plus your URP asset's render scale, shadow distance and shadow resolution. Low / Medium / High is enough; a slider per setting is a v1.1 fantasy.
- **The Input System has a built-in interactive rebinding API** — `PerformInteractiveRebinding` — and it genuinely exists, so do not write your own. **Verify the exact API surface against current Input System docs before you use it**; it has moved between versions.
- **Persisting rebinds is `SaveBindingOverridesAsJson` / `LoadBindingOverridesFromJson`.** One string in your settings file. This is the part everyone forgets and it's five lines.
- **Every setting must persist and must apply on load**, not just when changed. A settings screen that forgets is worse than none.

### Build (40 min)
1. Fix the top three *confusing* items from `PLAYTEST.md`. Usually a clearer prompt, a marker that wasn't showing, or a conversation that needed one directional line.
2. Settings — Display: resolution dropdown from `Screen.resolutions`, fullscreen mode dropdown, vsync toggle.
3. Settings — Graphics: Low/Medium/High preset applying `QualitySettings` and URP render scale and shadow distance together.
4. Settings — Audio: Master, Music, SFX, UI sliders into the AudioMixer groups, log-converted as on Day 65, persisted.
5. Settings — Gameplay: text speed, marker mode (On / Objective-only / Off), and subtitles on/off with the VO fallback respecting it.
6. Settings — Controls: a rebinding row per action. Click, press a key, it rebinds. Handle cancel (Esc), duplicate-binding warning, and a "Reset to defaults" button.
7. Save the binding override JSON alongside the rest of your settings and load it during startup, before the first scene needs input.
8. Restart the game and confirm every single setting survived. Then rebind three keys, restart, and confirm again.

### Acceptance criteria
- [ ] Top three confusing items fixed
- [ ] Resolution, fullscreen mode and vsync all work and persist
- [ ] Three graphics presets with a visible difference between them
- [ ] Audio sliders control the mixer groups and persist
- [ ] Every gameplay action can be rebound, cancelled, and reset to defaults
- [ ] Rebinds survive a restart via `SaveBindingOverridesAsJson`
- [ ] Settings apply on load, not only on change

### Failure modes
- **Rebind captures the mouse or Esc immediately** → exclude those control paths in the rebinding operation. It's a builder option, not a bug.
- **Rebinds don't persist** → you saved the action asset, not the override JSON. Only the JSON survives a build.
- **Resolution dropdown lists forty entries** → deduplicate by width/height, ignoring refresh rate.
- **Quality preset changes nothing visible** → you set `QualitySettings` but not the URP asset. In URP most of the levers live on the pipeline asset.
- **Settings apply but the game still starts wrong** → nothing calls the apply path at startup.

**Stretch:** A camera sensitivity slider and an invert-Y toggle. Two floats, and their absence is the most common complaint on any third-person game's store page.

**Commit:** `feat: full settings screen with input rebinding`

---

## Day 110 — Build it, then test the build
**Mon 28 Dec · 60 min**

**Objective:** A Windows build that runs correctly from a clean folder on a machine that has never seen Unity.

**Why:** The build is a different program from the editor. Every serious "it worked on my machine" story in games starts with someone who only ever tested in Play Mode.

### Concepts (10 min)
- **The editor lies in specific, predictable ways.** It runs faster in some places and slower in others, it keeps assets alive that a build strips, and it resolves paths that don't exist once packaged.
- **Editor-only code paths ship as gaps.** Anything inside `#if UNITY_EDITOR`, anything using `AssetDatabase`, anything your Editor State Inspector was quietly doing — none of it exists in the build.
- **File paths are the classic build bug.** `Application.dataPath` differs, `Resources` and `StreamingAssets` behave differently, and `Application.persistentDataPath` is the only correct home for saves.
- **The scene list is ordered and index 0 launches.** A black-screen build is almost always a missing scene or a scene at the wrong index. Use **Build Profiles** in Unity 6 (`File > Build Profiles` — this replaced Build Settings; **verify the current path rather than guessing**).
- **Your unsigned exe will trigger SmartScreen.** This is normal for indie releases, it is not fixable for free, and it belongs as a one-line note on your store page.
- **Build size matters to whether people download it.** Uncompressed textures and stray scratch folders under `Assets/` are usually the whole story.

### Build (40 min)
1. Player Settings: product name `Hearthfall`, company name, version `1.0.0`, icon from your key art, default resolution and windowed mode set deliberately.
2. Build Profiles: confirm every scene is present, MainMenu at index 0, nothing orphaned. Build to `Builds/Hearthfall-v1.0-win64/`.
3. Read the build log for the size breakdown. If anything unexpected is large — a 4K texture set, a scratch art folder, uncompressed audio — fix the import settings, not the exe.
4. **Copy the whole folder to an unrelated path** with no Unity project nearby. A temp directory, a USB stick, anywhere else. Run it there.
5. Fresh save, full playthrough to an ending. Watch for missing dialogue, missing VO, missing textures, and anything that loaded fine in the editor.
6. Test save and load *in the build*, then quit and relaunch and load again. Confirm the save landed in `persistentDataPath`.
7. Test the settings screen in the build, including rebinding and every fullscreen mode. Alt-tab in each of them.
8. If you have a second machine, run it there. If not, run it with a different display resolution and a controller unplugged.

### Acceptance criteria
- [ ] Windows build produced, versioned `1.0.0`
- [ ] Build runs from a clean unrelated path, start to an ending
- [ ] Save/load verified in the build across a full relaunch
- [ ] Settings and rebinding verified in the build
- [ ] Build size understood and nothing unexpected is inflating it
- [ ] Tested on a second machine or a second resolution

### Failure modes
- **Black screen on launch** → wrong scene at index 0, or no camera in it.
- **"Failed to load data file"** → the exe was moved without its `_Data` folder.
- **Dialogue JSON or VO missing** → loaded through a path that only resolves in the editor. `StreamingAssets` or an addressable-equivalent, not `AssetDatabase`.
- **Saves don't persist** → you wrote to `dataPath` instead of `persistentDataPath`.
- **Runs at 400 fps and combat timing is wrong** → something is frame-rate dependent that should be using `deltaTime` or `FixedUpdate`.
- **Windows blocks it** → SmartScreen on an unsigned exe. Expected. Note it on the page.

**Stretch:** Build a second time with the Development Build flag off *and* on, and keep the development build locally. If a bug report arrives after launch, that build has a console.

**Commit:** `build: v1.0.0 windows build verified clean`

---

## Day 111 — Store page, screenshots, and a gif
**Tue 29 Dec · 60 min**

**Objective:** A finished itch.io page, staged as a draft, that would make a stranger download the game.

**Why:** The build decides whether they finish it. The page decides whether they start. You get about eight seconds.

### Concepts (10 min)
- **Eight seconds is the budget:** title, tagline, one moving image. Everything else is for people already interested.
- **Motion beats stills.** A 30-second gif at the top of the page outperforms every screenshot you could take. Capture with the Unity Recorder package or OBS — both work; the Recorder gives you clean framerate control, OBS is faster to set up.
- **Screenshots are casting, not documentation.** Faces, text and action. Four to six, at 1920×1080, no debug overlays, no editor chrome.
- **An honest description outperforms a hyped one at this scale.** Say what it is, say how long it takes, say what it isn't. Nobody browsing itch.io is expecting a AAA release; they're deciding whether to spend forty minutes.
- **Attributions are not optional.** Every free asset came with a licence. `ATTRIBUTIONS.md` has been accumulating since Day 65 — today it goes on the page.
- **Content warnings cost nothing and matter to real people.** Violence, and whatever else your Act III actually does.

### Build (40 min)
1. Title and tagline. One line, no genre-speak, no colon-subtitle if you can help it. It should say what happens, not what genre it is.
2. Capture the gif: 30 seconds, one continuous slice, ideally a fight that ends *or* a conversation with a real choice on screen. Trim hard. Loop cleanly. Keep it under 5 MB or the page will crawl.
3. Screenshots — one per location (Hearthfall, the Wealdrun, Vaskirk or Greyhold), one conversation with framing and subtitles visible, one mid-fight, one of the journal. Four to six total.
4. Description: what the game is, the three-ledger premise without spoiling it, playtime (be honest — an hour is an hour), three endings, and the fact that it's a solo project built in sixteen weeks.
5. Controls section, including the fact that everything is rebindable. Note controller support if you have it, and say so plainly if you don't.
6. Credits and attributions — every asset pack, every sound, every font, with author and licence, straight from `ATTRIBUTIONS.md`. Mention the Kokoro VO explicitly and honestly.
7. Content warnings, a "known issues" section pulled from `KNOWN-ISSUES.md`, and the SmartScreen note.
8. Pricing: free, or free with a "name your own price" donate option. Set the thumbnail. Upload the zip, flag it as Windows and executable, and **leave the page as a draft.**

### Acceptance criteria
- [ ] Title, tagline and thumbnail set
- [ ] A 30-second gif at the top of the page, under 5 MB, looping cleanly
- [ ] 4–6 screenshots covering three locations, a conversation and a fight
- [ ] Honest description with playtime and scope stated
- [ ] Controls, credits, attributions, content warnings and known issues all present
- [ ] Zip uploaded, platform and executable flags set, page still a draft

### Failure modes
- **Gif is 40 MB** → too long, too high a resolution, or too many colours. 720p, 15 fps, 20 seconds.
- **Screenshots look empty** → you shot terrain. Shoot faces and text.
- **Page reads like a press release** → cut every adjective and see what's left. Usually it's better.
- **You forgot an attribution** → check every folder under `Assets/` against the file. Do it now, not after a licence complaint.

**Stretch:** Write a 150-word devlog post about the sixteen weeks and schedule it with the release. It's the single best way for anyone to find a page with no following behind it.

**Commit:** `docs: itch.io page for v1.0 staged`

---

## Day 112 — SHIP DAY
**Wed 30 Dec · 60 min**

**Objective:** The page goes live. The game has a URL. Then you read the log and stop.

### Ship it (30 min)

1. **Final build.** Rebuild from a clean state so the shipped binary matches the committed code exactly. No code changes after this point — none, not even the small one you're thinking about.
2. **Verify it from a clean folder.** Fresh path, no Unity nearby, fresh save. Launch, play far enough to trust it, quit. If anything is wrong you rebuild; you do not patch and hope.
3. **Upload the final zip to itch.io**, replacing yesterday's. Windows, executable flagged, price free or name-your-price.
4. **Finalise the page.** Read every word once more as a stranger. Fix the two typos that are definitely in there.
5. **Set it Public.** Not restricted, not draft. Public. Then open the URL in a private browser window and download it as a stranger would — confirm the zip is intact and the page reads correctly to someone who knows nothing about you or the project.
6. **Tag the release:** `git tag -a v1.0 -m "Hearthfall 1.0 — 3D release"` and push the tag. This commit is now a permanent, reachable point in the history.
7. **Update the Day 70 page.** Go back to the 2D slice on itch.io and add a line at the top linking to the 3D release. The two are the same game, fourteen weeks apart, and that comparison is genuinely interesting to anyone who finds either one.
8. **Send it to people.** Individually, by name, not as a broadcast. Include the three from Day 70 — they get to see what happened next.

### Acceptance criteria
- [ ] Final build produced from a clean state and verified on an unrelated path
- [ ] itch.io page is **public** and the download works from a private browser session
- [ ] Git tag `v1.0` created and pushed
- [ ] The Day 70 2D page links to the 3D release
- [ ] Sent individually to named people
- [ ] `progress/LOG.md` has a real final entry and `progress/STATE.md` says `complete`
- [ ] `README.md` carries both URLs

### Then: read the log (20 min)

Open `progress/LOG.md` and read it from Day 1. All of it, in order, without skipping.

You will not remember most of it. You will not remember the evening the assembly definition wouldn't compile and you nearly stopped, or the week the dialogue system existed only as a passing unit test with nothing on screen, or which day the first attack connected. It's all in there, in your own words, written by someone who at the time did not know whether this was going to work.

That's the point of the file, and today is the day it was written for.

Then write the last entry.

### Where you are

One hundred and twelve days ago you were a backend engineer who had never shipped a game and had tried and abandoned this twice before.

You now have two published games. That is the smaller of the two outcomes.

The larger one is what's underneath them: **`Hearthfall.Core`** — a narrative engine with no `using UnityEngine` anywhere in it, under test, that ran unmodified when you replaced its entire presentation layer. Branching dialogue as data with a validator. A consequence engine with deferred effects, so a choice in Vaskirk arrives as a letter from Enid four scenes later. Three ledgers the player never sees, gating three endings. Quests as a state machine with real failure states. Save/load with versioning you wrote before you needed it. Combat with light, heavy, dodge, stamina and lock-on, against three enemies who make you use all of it. Three locations with deliberate art direction and lighting you chose. Voiced dialogue with graceful fallback. A settings screen with full key rebinding.

And the thing that isn't in any file: on Day 30 a compile error you didn't understand would have cost you an evening. It doesn't any more. You have a debugging playbook, a profiler you know how to read, and the specific skill of getting yourself unstuck — which is the exact capability whose absence ended both previous attempts.

The 2D slice took 56 days. The 3D game took 42, and it's bigger. That gap is `Hearthfall.Core`, and it's the most useful thing in the repository.

### What's next

No wrong answers here. Honest takes on each:

- **Keep extending Hearthfall.** Act III properly, more quests, more of Greyhold. You have the systems and content is now fast. The risk is that a shipped game with no deadline quietly becomes an unshipped one again.
- **Rebuild it, better.** You know exactly what you'd do differently now, and you'd be right about most of it. Genuinely valuable, and also the most seductive way to avoid making something new.
- **Learn the thing you deliberately skipped.** Addressables, UI Toolkit, shader graph, Blender, DOTS. Each is a real gap and each is now a weekend rather than a mountain, because you have a real project to apply it to.
- **Enter a game jam.** **This is the recommendation.** One weekend, a theme you didn't choose, a hard deadline, and a finished thing at the end. Everything this curriculum taught you about scope-cutting and shipping is precisely the jam skill set, and 48 hours will teach you more than another 112 days of solo work.
- **Stop.** You set out to learn Unity and ship a game. You shipped two. Walking away now is a completed project, not an abandoned one — and knowing the difference is worth something on its own.

Whatever you pick: `progress/LOG.md` stays. Next time you start something hard, read it.

**Commit:** `docs: M15 complete — Hearthfall v1.0 shipped`
