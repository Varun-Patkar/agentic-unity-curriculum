# Asset Pipeline

You are one person with one hour a day. **Assets are the single most common way solo projects die** — not because they're hard, but because they're infinite. This file is a set of constraints designed to stop that.

## The three rules

1. **Greybox first, always.** Build every space out of untextured boxes and make it *play* well before it looks like anything. A fun grey level is a game. A beautiful level that plays badly is a wallpaper.
2. **Coherence beats quality.** Five mediocre assets under one lighting setup, one colour grade, and one silhouette language read as a *style*. Five gorgeous assets from five artists read as a Unity asset flip. **Lighting and post-processing do more for your look than any model you will ever download.** That's why M13 exists.
3. **Timebox brutally.** If a texture has eaten 30 minutes, it's placeholder now and you move on. Ship the game, then make it pretty. Almost nobody does it the other way and finishes.

---

## 2D (Milestones 01–09)

### Sprites and tiles
- **Kenney** (kenney.nl) — CC0, enormous, consistent. Your default for prototyping.
- **itch.io asset packs** — filter free + CC0. Excellent pixel-art tilesets.
- **OpenGameArt** — inconsistent quality, generous licences. Check each licence individually.

### Making your own (your stated preference)
The workflow that works:

1. Generate a base image with AI at high resolution — reference a concrete style ("dark medieval, muted ochre and grey, painted, no outline").
2. Downscale hard. Most AI art improves dramatically at 64×64 or 128×128 because the small stuff resolves into shape.
3. Clean it up in Paint / Krita / Aseprite: fix the silhouette, cut the background, unify the palette.
4. **Force a shared palette across everything.** This one step does most of the coherence work. Pick 16–24 colours and quantise every sprite to them.

For **portraits** (dialogue UI, M04) AI is genuinely excellent — one consistent prompt skeleton, swap the subject, then run every output through the same colour grade.

### Technical settings that matter
- **Pixels Per Unit** — pick one number (e.g. 16 or 32) and never deviate. Mismatched PPU is why your sprites are different sizes for no reason.
- **Filter Mode: Point (no filter)** for pixel art, **Bilinear** for painted art. Point + painted art looks crunchy; bilinear + pixel art looks like mud.
- **Compression: None** for pixel art. Artefacts are visible at small sizes.
- **Sprite Atlases** for anything you have a lot of — cuts draw calls.

---

## 3D (Milestones 10–15)

### Characters and animation — the highest-risk area
**Plan: Mixamo.** Free, auto-rigs a humanoid mesh, huge library of animations, exports FBX straight into Unity's Humanoid rig. It removes rigging and animation authoring from your project entirely, which is the only reason M11–M12 fit into six weeks.

> **Day 78 first task: verify Mixamo is still operating and still free.** Adobe has periodically signalled changes. Do not assume — check before you plan around it.
>
> **If Mixamo is gone or paywalled**, in order of preference:
> 1. **Unity Asset Store** — free "Starter Assets: Third Person" includes a rigged character with locomotion animations. Enough for the whole game.
> 2. **Quaternius** (quaternius.com) — CC0 rigged low-poly characters with animations.
> 3. **Kenney's animated characters** — CC0, blocky, would force a stylised art direction. Fine, but decide deliberately.
> 4. **Rokoko / ActorCore / Reallusion free tiers** — check current terms.
> 5. **Blender + Rigify** — only if you want to spend two weeks learning it. You don't, this time.

### Environment
- **Poly Haven** — CC0 HDRIs, textures, and models. The HDRIs alone will transform your lighting.
- **Quaternius** — CC0 low-poly medieval/nature packs. Perfect fit for Hearthfall.
- **Kenney** — CC0, consistent, and consistency is the whole game.
- **Sketchfab** — filter to **CC0 or CC-BY** and check each model's licence individually. Great for hero props (a cart, a shrine, a gate). Check the poly count before importing; a 2M-triangle scanned model will ruin your frame time.
- **AmbientCG** — CC0 PBR materials.

### Paid, if you ever want to
**Synty POLYGON** packs are the standard answer for solo devs who want instant coherence. One pack ≈ one visual identity. Not required, but if you find yourself fighting mismatched free assets in December, this is the £30 that fixes it.

### AI-generated 3D
Treat as a **bonus, never a dependency**. Meshy, Tripo, Rodin, and similar produce usable *props* (barrels, crates, shrines, statues) and unusable *characters* (topology and rigging are still bad). Where AI genuinely excels in 3D:
- **Textures** — infinite, tileable, style-matched
- **Skyboxes** — Blockade Labs' Skybox AI is excellent and takes two minutes
- **Concept art** — to lock your art direction before you build anything
- **UI, icons, portraits, and the itch.io page art**

### Import settings that matter
- **Scale.** 1 Unity unit = 1 metre. Blender exports and some Sketchfab models come in at 100× or 0.01×. Fix on import, not with a scaled transform, or physics will behave insanely.
- **Read/Write Enabled: off** unless you need mesh access — it doubles memory.
- **Mesh Compression** on for large scene props.
- **Rig type: Humanoid** for anything you want to retarget Mixamo animations onto. Generic for everything else.
- **Materials.** Imported models arrive with Built-In-pipeline materials and render **magenta** in URP. `Edit > Rendering > Materials > Convert Selected...` — expect this every single time and stop being alarmed by it.

---

## Audio

- **Freesound** — CC0/CC-BY sound effects. Filter by licence.
- **Kenney audio packs** — CC0 UI and impact sounds.
- **Pixabay / Incompetech** — free music. Check attribution requirements.
- **sfxr / jsfxr / ChipTone** — generate retro SFX in seconds.
- **Audacity** — pitch-shift and layer two free sounds and they stop sounding like free sounds.

**Voice: Kokoro TTS.** You already have this. The Unity side (Day 101) is: export per-line WAVs → name them by dialogue node ID → load by ID → play alongside the typewriter → subtitle stays authoritative if the clip is missing. Always design for the clip being absent; you will not voice every line.

---

## Licensing hygiene

Keep `ATTRIBUTIONS.md` in each Unity project from day one. Every asset gets a line: **what it is, where it came from, the licence, the author.** Add it the moment you import, not the week you publish — reconstructing this in December from 200 files is genuinely miserable, and you can't ship without it.

```markdown
| Asset | Source | Licence | Author |
|---|---|---|---|
| medieval_village_pack | quaternius.com | CC0 | Quaternius |
| footstep_gravel_01.wav | freesound.org/s/12345 | CC-BY 4.0 | username |
```
