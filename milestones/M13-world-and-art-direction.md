# M13 · World Building & Art Direction

**Days 92–98 · 10–16 Dec 2026 · project: `Hollowbrook3D`**

> Direct one coherent modern supernatural small town from free assets. One dominant contemporary pack, restrained accents, consistent scale, lighting, and colour will beat a folder of unrelated high-detail models.

**You end holding:** Town Square, Old Mine Road, Mercer Mine, and a reusable Town Hall interior that read as one ordinary town facing an extraordinary night.

---

## Day 92 — Greybox the four reusable sets
**Thu 10 Dec · 60 min**

**Objective:** All four locations blocked at human scale, walkable, timed, and marked for story and encounter beats.

### Concepts (10 min)
- Blockout decides traversal, sightlines, reuse, and pacing before art.
- Town Square is one compact street containing Town Hall, diner, and sheriff's office.
- Old Mine Road is short, wooded, and contains two encounter clearings.
- Mercer Mine reuses modular tunnels; Town Hall reuses one office for opening and Ending B.

### Build (40 min)
1. Create blockouts for `TownSquare`, `OldMineRoad`, `MercerMine`, and `TownHallInterior`.
2. Keep a 1.8 m reference and test every doorway in Play mode.
3. Keep Town Square compact; reuse it for day, dusk, and final night.
4. Curve the road and mark two readable clearings.
5. Build the mine from repeatable tunnel, corner, junction, and chamber modules.
6. Frame Vale's desk so the same office can later frame Alex as the new keeper.
7. Mark NPC, choice, quest, and encounter anchors; bake NavMesh surfaces.

### Acceptance criteria
- [ ] Four untextured blockouts exist and are walkable
- [ ] Traversal times are recorded
- [ ] Town Square contains the three civic storefronts
- [ ] Road has two clear encounter spaces
- [ ] Mine and Town Hall visibly reuse modular pieces
- [ ] NavMesh bakes cleanly in combat spaces

**Commit:** `feat: greybox Hollowbrook locations`

---

## Day 93 — Dress Town Square and Town Hall
**Fri 11 Dec · 60 min**

**Objective:** A present-day small-town hub and municipal interior built from a disciplined free-asset palette.

### Concepts (10 min)
- Choose one primary modern small-town/suburban pack; everything else is an accent.
- Search for storefronts, municipal furniture, diner props, road markings, utility poles, bins, noticeboards, and parked cars.
- Prefer CC0 or clearly licensed free assets from Kenney, Quaternius, Poly Haven, Unity Asset Store, or verified Sketchfab sources.
- Prefab repeated storefront and office assemblies; record attribution on import.

### Build (40 min)
1. Choose and log one dominant contemporary environment pack.
2. Replace Town Square boxes while preserving paths and sightlines.
3. Make Town Hall, diner, and sheriff's office readable from silhouette and signage.
4. Dress Vale's office with desk, civic seal, filing cabinets, visitor chairs, and deputy entrance route.
5. Create reusable storefront and office prefabs.
6. Fix URP materials and import scale; mark static geometry appropriately.

### Acceptance criteria
- [ ] Town Square reads as a current small town
- [ ] Town Hall office supports the briefing and mirrored ending shot
- [ ] One primary asset language dominates
- [ ] Reusable prefabs replace repeated assemblies
- [ ] No magenta materials or transform-scale fixes remain
- [ ] Every imported asset is attributed

**Commit:** `feat: dress Town Square and Town Hall`

---

## Day 94 — Old Mine Road and Mercer Mine
**Sat 12 Dec · 60 min**

**Objective:** A wooded modern access road and abandoned industrial mine that support investigation and short encounters.

### Concepts (10 min)
- Curves, elevation, tree lines, and utility landmarks hide a linear route.
- Modern mine language is chain-link, warning signs, timber supports, corrugated sheds, work lights, rails, cable, and abandoned equipment.
- Sparse set dressing is deliberate when every prop supports navigation or story.
- Reuse tunnel modules aggressively; lighting will carry most of the mine's escalation.

### Build (40 min)
1. Import and attribute compatible free nature and industrial-mine packs.
2. Dress the road with woods, drainage, utility poles, barriers, and an abandoned vehicle.
3. Preserve both encounter clearings and line-of-sight escape routes.
4. Build Mercer Mine entrance around the Mercer name and current closure evidence.
5. Assemble tunnels and finale chamber from the Day 92 module set.
6. Re-bake navigation and test both creature variants.

### Acceptance criteria
- [ ] Road curves and changes elevation without becoming confusing
- [ ] Two encounter clearings remain readable
- [ ] Mine reads as abandoned local industry, not a fantasy dungeon
- [ ] Modular tunnels carry the full route
- [ ] NavMesh and collision survive dressing
- [ ] Attribution remains current

**Commit:** `feat: dress Old Mine Road and Mercer Mine`

---

## Day 95 — Lighting the ordinary and the supernatural
**Sun 13 Dec · 60 min**

**Objective:** Reused geometry reads differently by day, dusk, final night, and underground through light alone.

### Concepts (10 min)
- Direction, colour, shadow, and contrast establish time and danger.
- Bake static sets; use light probes for Alex and NPCs.
- Town Square needs believable civic light sources: streetlights, diner spill, office fluorescents.
- Mercer Mine uses work lights and darkness to guide the route.

### Build (40 min)
1. Capture before images of all four locations.
2. Establish daylight Town Square, dusk road, practical-lit Town Hall, and guided mine lighting.
3. Add an attributed HDRI where useful.
4. Bake modest-resolution lightmaps and place probe groups.
5. Create final-night light variants without moving base geometry.
6. Tune shadow distance against normal camera range.

### Acceptance criteria
- [ ] Before/after captures prove lighting changes the same sets
- [ ] Practical light sources fit a modern town
- [ ] Baked lighting has no major seams
- [ ] Characters receive environment light through probes
- [ ] Mine route remains readable without flattening its darkness
- [ ] Final-night variants reuse base geometry

**Commit:** `feat: light Hollowbrook locations`

---

## Day 96 — Fog, post-processing, and colour continuity
**Mon 14 Dec · 60 min**

**Objective:** Restrained Volume profiles make all four sets coherent while separating ordinary day, warning dusk, mine descent, and final night.

### Concepts (10 min)
- Use URP Volumes for grading, fog, ambient occlusion, restrained bloom, vignette, and grain.
- Fog hides world edges and builds depth; it must not erase navigation.
- Keep Hollowbrook recognisably ordinary. Supernatural pressure arrives through controlled contrast and colour shifts.
- Verify exact Unity 6/URP controls against installed-version documentation.

### Build (40 min)
1. Create profiles for Town Day, Road Dusk, Mine, and Final Night.
2. Keep skin tones and dialogue framing readable under every profile.
3. Use distance fog on the road and selective haze in the mine.
4. Blend local profiles at Town Hall and mine thresholds.
5. Compare screenshots side by side and fix the worst mismatch with grade, then light, then asset choice.

### Acceptance criteria
- [ ] Four restrained profiles are saved as assets
- [ ] Every location still belongs to the same visual world
- [ ] Local transitions blend smoothly
- [ ] Fog supports scope and depth without hiding objectives
- [ ] Dialogue faces remain readable
- [ ] Effects remain subtle at gameplay speed

**Commit:** `feat: grade Hollowbrook across story states`

---

## Day 97 — Ambience and the last night
**Tue 15 Dec · 60 min**

**Objective:** Each location has a layered soundscape, and Town Square transforms for the last night through state-dependent light, weather, props, and town reactions.

### Concepts (10 min)
- Layer quiet beds, local 3D sources, and occasional one-shots.
- Town prep and earlier decisions should alter visible details: warning activity, open routes, boarded windows, deputies, and evacuees.
- Reuse the same Town Square set; recognition gives the change weight.
- Use an AudioMixer with Master, Music, SFX, Voice, and Ambience groups.

### Build (40 min)
1. Source and attribute town, woods, road, office, and mine ambience.
2. Add spatial diner, traffic, fluorescent hum, woods, and mine sounds where appropriate.
3. Configure the AudioMixer and a mine reverb zone.
4. Build a final-night setup over Town Square without changing base geometry.
5. Bind prepared-town and decision flags to a small set of prop and NPC variations.
6. Compare all locations and story states in stills and one continuous playthrough.

### Acceptance criteria
- [ ] Every location has layered, appropriate ambience
- [ ] Mixer groups support M15 settings
- [ ] Mercer Mine has audible spatial transition/reverb
- [ ] Final-night Town Square is recognisably the same place
- [ ] Town preparation and decisions alter visible details
- [ ] Worst audio-visual mismatch is fixed

**Commit:** `feat: add Hollowbrook ambience and final-night state`

---

## Day 98 — BUFFER
**Wed 16 Dec**

- Finish the weakest of the four sets before adding props elsewhere.
- Re-bake final lighting only after geometry stops moving.
- Recheck free-asset licences and modern visual consistency.
- Walk the full Town Square → Old Mine Road → Mercer Mine route and revisit Town Hall.

### Milestone review

Run `/review`. Check frame time, poly outliers, static batching, occlusion, attribution, traversal, and whether any asset pulls the game away from contemporary small-town supernatural fiction.

### Where you are

Hollowbrook now has one compact hub, one short road, one modular mine, and one Town Hall office reused for the opening and a final mirror. The world is small on purpose, coherent through direction rather than budget, and ready for its dialogue, quests, and three endings.

**Commit:** `docs: M13 complete — Hollowbrook world and art direction`
