# M13 · World Building & Art Direction

**Days 92–98 · 10–16 Dec 2026 · project: `Hearthfall3D`**

> You are not an artist and this week does not require you to become one. It requires you to become a *director* — someone who picks free assets that agree with each other, points a light at them, and grades the result. Coherence beats quality every time: five mediocre models under one lighting setup and one colour grade read as a style, while five gorgeous models from five artists read as an asset flip.
>
> The two biggest levers you own are Day 95 and Day 96, and neither involves downloading a single mesh. Lighting and post-processing will do more for how your game looks than any model you could find or make. Everything before those two days exists to give the light something to fall on.
>
> Ship order stays what it has been all curriculum: greybox first, dressing second, pretty last.

**You end holding:** three locations that read as one coherent, gritty world — built from free assets, unified by light.

---

## Day 92 — Greybox first: blocking out space before a single pretty asset
**Thu 10 Dec · 60 min**

**Objective:** Hearthfall, the Wealdrun and Greyhold blocked out in untextured grey boxes, walkable end to end, with encounter positions marked.

**Why:** A fun grey level is a game. A beautiful level that plays badly is wallpaper, and by the time you notice, you've spent three days dressing something you now have to tear up.

### Concepts (10 min)
- **Blockout is a gameplay document, not an art document.** You're deciding traversal time, sightlines, chokepoints, and where the player's eye goes. Grey is a feature — it stops you judging the space on how it looks.
- **Scale against the character, not against your intuition.** Your controller is roughly 1.8 units tall. Doorways ~2.2, ceilings ~3, a village street ~6–8 wide. Keep a capsule of the right height in the scene as a physical ruler.
- **Time your traversal.** Walking the length of the Wealdrun should take a specific number of seconds and you should decide that number now. "It feels long" during the blockout means it is unbearable dressed.
- **Landmarks beat corridors.** The player should always be able to see something they're heading toward. Block those in first, then fill the space between them.
- **ProBuilder is the right tool and may need installing.** It's a package — **Window > Package Manager**, and verify the current name and install path against Unity's docs rather than trusting a 2021 tutorial. If it fights you, scaled cubes are genuinely fine.
- **Beginner trap: dressing as you block.** The moment you place a nice-looking barrel, you stop editing the space around it. Nothing textured goes in today.

### Build (40 min)
1. Three new scenes: `Hearthfall_Blockout`, `Wealdrun_Blockout`, `Greyhold_Blockout`. One flat grey material on everything.
2. Drop your player prefab into each so you can walk them immediately. Keep a 1.8-unit reference capsule parked beside the origin.
3. **Hearthfall:** eight to ten building volumes around a central open space — the well, the notice board, the farm at one end, the road out at the other. Every building is a box for now.
4. **Wealdrun:** a path with curves and elevation change, not a straight tube. Two widenings for the combat encounters, and the tollgate as a hard pinch point.
5. **Greyhold:** an approach, a gate, and three interior rooms. Deliberately larger and taller than anything in the valley.
6. Walk each space and **time it with a stopwatch**. Write the numbers in `LEVELS.md`. Hearthfall corner to corner should be under 30 seconds.
7. Place empty GameObjects named `Encounter_01`, `Encounter_02`, `NPC_Osric`, etc. — the blockout is where those positions get decided.
8. Bake a NavMesh over each and confirm the enemies from Day 89 can path the whole space.

### Acceptance criteria
- [ ] Three blockout scenes exist, entirely untextured
- [ ] All three are walkable end to end with no falling through the floor
- [ ] Traversal times recorded in `LEVELS.md`
- [ ] Doorways and ceilings verified against the 1.8-unit character
- [ ] Encounter and NPC positions marked with named empties
- [ ] NavMesh bakes cleanly in all three

### Failure modes
- **Everything feels cramped or cavernous** → you built to the top-down Scene view, not to eye level. Judge only from Play mode.
- **The road is boring** → it's straight and flat. Curve it and change the elevation; both are free.
- **Player clips through walls** → box colliders scaled but not regenerated, or the collider is on the parent and the mesh on the child.
- **NavMesh has holes** → geometry isn't marked Navigation Static, or the agent radius doesn't fit your doorways.

**Stretch:** Walk each space with the camera at a fixed height and screenshot the three best sightlines. Those become your composition targets for the rest of the week.

**Commit:** `feat: greybox blockouts for hearthfall, wealdrun and greyhold`

---

## Day 93 — Building Hearthfall: kitbash discipline and the 80/20 of set dressing
**Fri 11 Dec · 60 min**

**Objective:** Hearthfall dressed with real assets, reading as a poor Weald hamlet, at a frame rate you can live with.

**Why:** Kitbashing badly is how free assets look free. Kitbashing with one rule — one primary pack, everything else an accent — is how they look intentional.

### Concepts (10 min)
- **Pick ONE primary pack and commit.** Quaternius' medieval/nature CC0 sets are the best fit for Hearthfall. That pack sets your polygon density, your proportions, and your silhouette language. Every other source is an *accent* — a hero prop here, a texture there — and must be outnumbered heavily.
- **The 80/20 of set dressing:** silhouette first (roofline variety is what makes a village read as a village), repetition with variation (same house, rotated, scaled 0.9–1.1, different roof), clutter concentrated at the edges of spaces, and negative space kept genuinely empty. Walkable middle, dense corners.
- **Prefab your assembled buildings.** House + roof + chimney + door + a barrel becomes one prefab. Place the prefab five times, vary it, and use prefab variants for the different ones. This is the difference between a 40-minute village and a four-hour one.
- **Static batching and occlusion culling.** Anything that never moves gets the **Static** flag in the Inspector — that's most of the world. Occlusion culling stops Unity rendering the far side of the village through a wall. **Verify the current occlusion bake window and workflow against Unity's docs**; the panel has moved between versions.
- **Watch poly counts as you import.** A 200k-triangle "medieval barrel" from Sketchfab is real and will cost you. Check before importing, note it in `LEVELS.md`, and drop anything absurd.
- **`ATTRIBUTIONS.md` gets a line the moment you import — not later.** Reconstructing 200 files' licences in January is genuinely miserable and you cannot ship without it.

### Build (40 min)
1. Choose your primary pack. Import it. Add its row to `ATTRIBUTIONS.md` immediately.
2. Expect imported materials to render **magenta** in URP. `Edit > Rendering > Materials > Convert Selected...`. This will happen with every pack; stop being surprised by it.
3. Check import scale — 1 unit = 1 metre. Fix it in the model's import settings, never with a scaled transform.
4. Build **one** house prefab properly: walls, roof, door, chimney, one or two props. Get it right once.
5. Replace the Hearthfall building boxes with instances of that prefab. Rotate, scale slightly, swap roofs. Make two variants for the farm and the mill.
6. Dress the centre: the well, the notice board, a cart, a fence line, a woodpile. Keep the middle of the square walkable and empty.
7. Mark everything immobile as Static. Bake occlusion culling.
8. Open the **Game view Stats** overlay and note your triangle count and batch count in `LEVELS.md`. You want a number to compare against in M15.

### Acceptance criteria
- [ ] One primary asset pack chosen and dominant in the scene
- [ ] Hearthfall dressed and still walkable, with the blockout gameplay intact
- [ ] At least one reusable building prefab with variants
- [ ] No magenta materials anywhere
- [ ] Static flags set and occlusion culling baked
- [ ] `ATTRIBUTIONS.md` updated for every asset imported today
- [ ] Triangle and batch counts recorded

### Failure modes
- **The village looks like a shop window** → too many different sources. Cut the accents until one pack clearly dominates.
- **Everything is identical** → repetition without variation. Rotate and scale; it costs seconds.
- **Frame rate collapses after dressing** → one scanned prop with an insane poly count, or nothing is marked Static. Check the Stats overlay before blaming the whole scene.
- **Models arrive tiny or enormous** → import scale, not transform scale. Fix at the source.
- **You can no longer walk where you could yesterday** → props got colliders. Strip colliders from decorative clutter.

**Stretch:** Build a `RandomYRotation` editor script that jitters rotation and scale on selected objects. Tooling is explicitly allowed and this one pays for itself in twenty minutes.

**Commit:** `feat: hearthfall dressed with kitbashed assets`

---

## Day 94 — The Wealdrun and Greyhold
**Sat 12 Dec · 60 min**

**Objective:** The road dressed as a long cold corridor that doesn't feel like a corridor, and Greyhold dressed as the heroic fantasy game this one isn't.

**Why:** These two locations do opposite jobs. The Wealdrun has to feel like distance and exposure; Greyhold has to feel like the adventure you were sold. Building them on the same day makes the contrast deliberate.

### Concepts (10 min)
- **A linear road hides its linearity with curves, elevation and landmarks.** You can't see the end, you can always see the next thing, and the thing you passed ten seconds ago is still visible behind you. That's the entire trick.
- **Sparse is a style, not laziness.** The Wealdrun is meant to be empty. Bare trees, a broken cart, a milestone, a shrine nobody tends. Emptiness with three good objects in it reads as desolation; emptiness with nothing reads as unfinished.
- **The tollgate is a set piece with gameplay in it.** It's a pinch point, it's where a combat encounter or a bribe conversation happens, and it should be visible from a long way off so the player has time to feel about it.
- **Greyhold deliberately looks like a different game.** Bigger stone, taller arches, banners, braziers, dressed columns. It is the one place where high fantasy is allowed, because the story needs it to be a lie. Let yourself enjoy it — it's the only day this week you can.
- **Reuse modular pieces aggressively.** A wall segment, a corner, an arch, a stair, a floor tile, a pillar. Six pieces build an entire keep, and snapping them with grid snap (hold Ctrl while dragging, and check the grid size in the Scene view's grid settings) is far faster than freehand.
- **Beginner trap: dressing the road for twenty minutes per hundred metres.** Dress a 30-metre section properly, then repeat and vary it. Nobody stands still on a road.

### Build (40 min)
1. Import a nature/forest pack and a stone/dungeon pack. Both go into `ATTRIBUTIONS.md` before you place anything.
2. **Wealdrun:** line the path with trees and rock formations that block sightlines around each curve. Keep the ground plane varied.
3. Place three landmarks along the road, spaced so one is always visible: the shrine, the broken cart, the tollgate.
4. Build the **tollgate**: a gatehouse, a barrier, a brazier, a couple of crates. Confirm the encounter position from Day 92 still works as a fight space.
5. Dress one 30-metre road section to a standard you're happy with, prefab it, then repeat it along the path with rotation and prop variation.
6. **Greyhold:** assemble the approach, gate and three rooms from six modular pieces with grid snapping. Taller and grander than anything in the valley.
7. Add the fantasy signals to Greyhold only: banners, braziers, a throne or altar, worked stone floors.
8. Walk both. Re-bake the NavMesh. Confirm the enemies still path, then check the Stats overlay again.

### Acceptance criteria
- [ ] The Wealdrun curves and changes elevation; you can't see the far end from the near end
- [ ] Three spaced landmarks, one always visible
- [ ] The tollgate is a readable set piece with a working encounter space
- [ ] Greyhold is visibly grander and built from reused modular pieces
- [ ] NavMesh re-baked and enemies path both spaces
- [ ] `ATTRIBUTIONS.md` current

### Failure modes
- **The road still feels like a tube** → the tree line is a perfectly parallel wall. Break it, and vary the path width.
- **Modular pieces have visible seams** → grid snapping is off, or the pieces come from packs with different grid sizes. Pick one pack for the keep.
- **Greyhold looks like the village with more rocks** → not enough vertical scale. Double the ceiling height and add a stair.
- **NavMesh breaks after dressing** → trees and rocks aren't marked Navigation Static, or props are blocking the path invisibly.

**Stretch:** Add a single distant silhouette visible from the road — a ruined tower, a hanging tree — that you never let the player reach. Cheap, and it makes the world feel like it continues.

**Commit:** `feat: wealdrun and greyhold environments`

---

## Day 95 — Lighting: the single biggest visual lever you own
**Sun 13 Dec · 60 min**

**Objective:** The same geometry, transformed — change nothing but the light and watch a grey scene become a place.

**Why:** This is the most important day of the milestone. A default Unity scene looks like a default Unity scene because of one directional light at 50 degrees, pure white, at intensity 1. Moving it is free and changes everything.

### Concepts (10 min)
- **The sun is an angle and a colour, and both are dramatic choices.** A high white sun reads as "Unity default project". A low, warm, strongly angled sun casts long shadows, rakes across surfaces, and reads as cinematic. Get the directional light down to 15–25 degrees of elevation and warm it.
- **Shadows do the modelling.** Your assets are low-poly and cheap; long raking shadows give them form your meshes don't have. This is the single highest-return adjustment available to you.
- **Realtime vs baked.** Realtime lights update but cost frame time and give you no bounce. **Baked lightmapping** pre-computes light and shadow into textures — it costs bake time and gives you soft indirect light for free. For static village geometry, bake. **Bake times can be long; start a bake and go get a drink**, and don't bake three times in a row while iterating.
- **Baking needs two things from you:** geometry marked as **Lightmap Static** (you did most of this on Day 93), and sane **lightmap UVs** — enable *Generate Lightmap UVs* in the model import settings for anything that comes out blotchy.
- **Light probes are how moving characters get baked light.** Without them, your player walks through a beautifully lit village lit as if by nothing. Place a probe group covering the walkable area, denser where light changes sharply.
- **Shadow distance and cascades.** Shadows are only rendered out to a distance you set. Too short and shadows visibly pop in ahead of you; too long and they get soft and expensive. **The lighting and shadow settings live in the URP Asset and the Universal Renderer, and their names and locations have moved between URP versions — open the docs for your exact version rather than guessing at a field name.**
- **An HDRI skybox is a two-minute upgrade.** Poly Haven's CC0 HDRIs give you a sky and matching ambient light in one asset.

### Build (40 min)
1. Open Hearthfall. Before touching anything, screenshot it. You want the before.
2. Drop the directional light to a low elevation, rotate it so shadows rake across the square, and warm its colour toward amber. Adjust intensity down. **Only the light — change nothing else.**
3. Screenshot again. Compare. This is the lesson; take the thirty seconds.
4. Download a matching HDRI from Poly Haven, make it your skybox material, and set ambient lighting to sample it. Log it in `ATTRIBUTIONS.md`.
5. Confirm your static flags, then bake lightmaps for Hearthfall. Set a modest lightmap resolution first — high resolution is for the final bake, not for iteration.
6. Add a **Light Probe Group** over the walkable area. Play, and confirm the character picks up the environment's light as they move.
7. Add local lights with intent: a warm point light in a doorway, a brazier, a lantern. Two or three, not twenty.
8. Set shadow distance and cascades so shadows don't visibly pop in at your normal walking speed. Check the exact setting names in the docs.

### Acceptance criteria
- [ ] Before and after screenshots of the same scene, light-only change
- [ ] Directional light is low, angled and coloured — not white at 50 degrees
- [ ] An HDRI skybox is in use and logged in `ATTRIBUTIONS.md`
- [ ] Hearthfall's lightmaps bake without blotching or seams
- [ ] A light probe group exists and the player visibly picks up local light
- [ ] Shadows don't pop in within normal viewing distance

### Failure modes
- **Baked lighting looks blotchy or seamy** → lightmap resolution too low, or missing generated lightmap UVs on the model.
- **Bake takes forever or never finishes** → resolution far too high for iteration, or you've marked the whole world static including things that shouldn't be.
- **The character looks flat and unlit while the world looks great** → no light probes.
- **Everything went dark after baking** → objects are static but their light isn't set to Baked or Mixed, or ambient is sampling nothing.
- **Shadows visibly appear a few metres ahead of you** → shadow distance is too short. Raise it, then re-check frame time.

**Stretch:** Do the whole light-only exercise a second time on Greyhold — cold, hard, top-down, contrasty. Same geometry, entirely different feeling, and it proves the point.

**Commit:** `feat: lighting pass and baked lightmaps for hearthfall`

---

## Day 96 — Fog, post-processing, colour grading: manufacturing the gritty look
**Mon 14 Dec · 60 min**

**Objective:** Three per-location Volume profiles that make the valley cold, Vaskirk warm, and Greyhold contrasty — and blend between them as you walk.

**Why:** "Gritty" is not a modelling decision. It is desaturation, crushed blacks, cool shadows and a bit of grain, and it is applied in about eight sliders.

### Concepts (10 min)
- **URP post-processing runs through the Volume framework.** A **Global Volume** holds a Volume Profile; the profile holds overrides. **Where these components live and what the overrides are called has moved between URP versions — open the URP docs for your version before hunting through menus.** You also need post-processing enabled on the camera and in the renderer.
- **Colour grading is where "gritty" is actually manufactured.** Reduce saturation. Crush the blacks slightly with lift/gamma/gain or the shadows/midtones/highlights trackballs. Push shadows cool and highlights warm — that split is most of the KCD look on its own.
- **Fog does two jobs.** *Height fog* fills the valley floor and separates your buildings from each other. *Distance fog* hides the edge of your world, which means you don't have to build past it. That's a scoping tool, not just an effect.
- **Bloom sparingly, vignette gently, grain lightly.** All three at obvious strength reads as amateur. Set each one until you notice it, then halve it.
- **Ambient occlusion** adds contact shadows where objects meet, which is exactly where cheap assets look like they're floating. High return, one override.
- **Different profiles per location is the whole point.** Vaskirk warmer, richer, more saturated — it should look *better*, because the theme needs the city to look like it's worth it. The Weald cold and desaturated. Greyhold high-contrast. That's the story doing visual work.
- **Local volumes blend.** A volume with **Is Global** off, a box collider and a blend distance will fade its profile in as the player approaches. That's how you cross from the cold road into the warm city without a cut.

### Build (40 min)
1. Confirm post-processing is enabled on the camera and in the renderer. Verify the setting names against your URP version's docs.
2. Create three profiles: `VP_Weald`, `VP_Vaskirk`, `VP_Greyhold`, stored in `Assets/Settings/`.
3. **Weald:** desaturate noticeably, cool the shadows, warm the highlights slightly, crush blacks a little. Add height fog at ground level and distance fog tuned so the far edge of your world disappears.
4. **Vaskirk:** the same grade, warmer and less desaturated. Side by side with the Weald it should look like somewhere you'd want to be.
5. **Greyhold:** raise contrast, cool it hard, drop the fog density and let the shadows go deep.
6. Add ambient occlusion, a light vignette, subtle film grain and restrained bloom to all three. Set each to where you notice it, then halve it.
7. Place a **local volume** with a box collider at the approach to the tollgate so the grade shifts as you travel. Set a blend distance and walk it.
8. Screenshot all three locations under their profiles. Put them side by side. If one doesn't belong, it's the grade, not the models.

### Acceptance criteria
- [ ] Post-processing verified enabled and working in URP
- [ ] Three distinct Volume profiles saved as assets
- [ ] Vaskirk visibly warmer and richer than the Weald in a side-by-side
- [ ] Height fog and distance fog both in use in the valley
- [ ] At least one local volume blends smoothly as you walk into it
- [ ] Bloom, vignette and grain all subtle enough to be deniable

### Failure modes
- **No visible effect from the volume** → post-processing off on the camera, the volume's layer mask doesn't match, or the profile isn't assigned.
- **The scene turns into a milky grey soup** → fog density too high, or you desaturated and lifted blacks at the same time. Pick one.
- **Everything glows** → bloom threshold too low. Raise the threshold before touching intensity.
- **The grade snaps instead of blending** → blend distance is zero, or the trigger collider isn't a trigger.
- **Frame rate drops sharply** → ambient occlusion and high-quality bloom are the usual suspects. Check them first.

**Stretch:** Author a fourth profile for Hearthfall in Act III — colder, greyer, lower sun. You'll use it tomorrow.

**Commit:** `feat: per-location volume profiles and colour grading`

---

## Day 97 — Ambience and audio-visual coherence · dressing Hearthfall for Act III
**Tue 15 Dec · 60 min**

**Objective:** Every location has a soundscape that agrees with its picture — and Hearthfall exists in two versions, same geometry, different world.

**Why:** Audio is half of atmosphere and costs a fraction of the effort. And the Act III Hearthfall is the single most efficient piece of storytelling in this game: you change the light, the weather and eight props, and the player's stomach drops.

### Concepts (10 min)
- **Ambience is layers, not a track.** A quiet bed (wind), a mid layer (birds, a river, distant work), and occasional one-shots (a dog, a hammer, a crow). Three layers at different volumes read as a place; one looping track reads as a menu.
- **Audio and visuals must sell the same thing.** Cold desaturated valley plus cheerful birdsong reads as a bug. If the picture says winter, the audio says wind and nothing else.
- **Spatial vs 2D sound.** Wind is 2D and everywhere. The river, the forge, the well are 3D sources with a rolloff, and walking past them is what makes the space feel dimensional.
- **Reverb zones.** Greyhold's interior should sound like stone. A reverb zone on the interior with a hall preset costs one component and does more than any amount of mixing.
- **The Act III trick: same geometry, changed light, weather, and props.** You do not rebuild Hearthfall. You duplicate the scene, swap the Volume profile for the cold one, drop the sun lower and greyer, add rain or snow particles, and change the props — boarded windows, a cart gone, a fresh grave, doors closed. Eight props carry it.
- **Coherence pass: judge from screenshots, not from Play mode.** Play mode flatters everything because you're moving. Static screenshots side by side is where mismatched assets confess.

### Build (40 min)
1. Source ambience from Freesound — wind, birds, river, distant village work, crows. Filter by licence and log every file in `ATTRIBUTIONS.md` as you download.
2. Layer three ambient sources per location. Wind as a 2D loop; river and forge as 3D sources with tuned rolloff.
3. Set up an AudioMixer with Master / Music / SFX / Ambience groups. You need this for the settings menu in M15 anyway; do it once, now.
4. Add a reverb zone to Greyhold's interior. Walk in and out and confirm you hear the change.
5. **Act III Hearthfall:** duplicate the scene as `Hearthfall_ActIII`. Do not touch the geometry.
6. Apply the cold Volume profile, drop the sun lower and grey it, add rain or a light snow particle system, and kill the birdsong — leave wind only.
7. Change eight props and no more: board a window, remove the cart, unlit the forge, close the doors, add a grave. Then walk it. If it doesn't land, the light is still too warm.
8. **Coherence pass.** Screenshot Hearthfall, the Wealdrun, Vaskirk, Greyhold and Act III Hearthfall. Put all five side by side. Anything that doesn't belong gets fixed with the grade first, the light second, the model last.

### Acceptance criteria
- [ ] Three ambient layers per location, with 3D sources where appropriate
- [ ] An AudioMixer with Master / Music / SFX / Ambience groups
- [ ] A reverb zone that is audible on entering Greyhold
- [ ] `Hearthfall_ActIII` exists with identical geometry and changed light, weather and props
- [ ] Act III Hearthfall reads as the same place, changed — not as a different place
- [ ] Five screenshots compared side by side and the worst mismatch fixed

### Failure modes
- **Ambience sounds like one loop** → all three layers at the same volume, or the same length. Vary both, and add one-shots.
- **3D sound is inaudible or omnipresent** → the rolloff curve. Logarithmic, with a max distance you actually tested.
- **Act III Hearthfall looks like a different village** → you moved geometry. Revert it; the whole effect depends on recognition.
- **The Act III change doesn't hit** → not enough contrast in the light, or too many props changed. Fewer, more pointed changes land harder.
- **Audio clips at loud moments** → everything routed to Master at full volume. That's what the mixer groups are for.

**Stretch:** Add footstep sounds that vary by surface using a tag on the ground material. Small, and it is the single most-heard sound in your game.

**Commit:** `feat: ambient audio and act iii hearthfall`

---

## Day 98 — BUFFER
**Wed 16 Dec**

- **Catch up.** If Days 92–97 spilled — and lighting or dressing usually does — this is where it lands.
- **Keep dressing the world.** This is a genuinely enjoyable and entirely legitimate use of the day. You've spent thirteen milestones on systems; spending an hour making the tollgate look right is earned, and the result is what ends up in your screenshots.
- **Fix the worst mismatch** from yesterday's side-by-side. There's always one location that doesn't belong yet.
- **Re-bake lighting at final resolution** on whichever scene you've stopped iterating on. Start it and go do something else.
- **Write the Act II Vaskirk conversations** if you're behind on content — M14 starts tomorrow and it's narrative work.
- **Rest.** This is the last buffer day before the final content push. Days 99 onward are narrative, VO and ship prep, and they run without a break of this kind. Use it.

### Milestone review

Run `/review`. Ask specifically about **frame time and poly budget**: which scene is heaviest, whether any imported asset is absurdly high-poly for what it is, and whether static batching and occlusion culling are actually doing anything. M15 has a dedicated profiling day and you want the list of suspects written down before you get there, not discovered on Day 108.

### Where you are

**You have a world.** Not a greybox with a character in it — three dressed locations, lit deliberately, graded to agree with each other, with sound that matches the picture. Hearthfall exists twice, and the second one is going to hurt people.

You are 98 days in, 87% through, and the hardest remaining problems are all ones you already know how to solve. Everything from here is content and polish on systems that work.

Tomorrow: narrative in 3D. Dialogue cameras that frame a conversation instead of pointing a third-person camera at a face, Kokoro voice-over on the lines that matter, and the three endings staged in the world you just built. The Act III Hearthfall you made today is where two of them happen.

**Commit:** `docs: M13 complete — world building and art direction`
