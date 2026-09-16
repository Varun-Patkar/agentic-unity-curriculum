# Story Bible — *Last Stop, Hollowbrook*

> **Status: DRAFT SCAFFOLD.** The premise and scope are locked; names and individual lines can change without changing the systems.
>
> **Working title:** `Last Stop, Hollowbrook`.

---

## 1. The one-line pitch

*You return to a painfully ordinary small town after your sibling disappears, discover the Mayor has kept Hollowbrook safe through a bargain with something beneath it, and decide whether to destroy the bargain or inherit it — unless the Mayor throws you out before the story starts.*

## 2. Tone and promise

This is a sincere, cliché supernatural mystery with one committed joke. It uses familiar pieces on purpose: the missing sibling, the suspicious Mayor, the diner that knows everyone, the abandoned mine, the local cult, and the ancient thing under the town.

The game is not a parody. Outside the skip ending, characters take the danger seriously and choices have lasting consequences. The story stays small enough to finish and gives both main endings room to breathe.

Three rules:

1. **No morality meter.** Record decisions and cite them back through dialogue and epilogues.
2. **The Mayor's bargain genuinely protected people.** Destroying it is not an obviously correct button.
3. **The joke ending is staged like a real ending.** It gets escalation, a consequence, an epilogue, and credits rather than a throwaway game-over message.

## 3. The three endings

There are exactly three endings. Two conclude the full story. One is an early first-playthrough secret ending.

### Ending A — "Morning in Hollowbrook"
**Break the bargain · enough townspeople prepared or rescued**

You destroy the thing beneath the mine and end the bargain. The supernatural protection around Hollowbrook disappears with it. The final night damages the town, but the people you warned survive and begin rebuilding without sacrifices, missing-person cover-ups, or a Mayor deciding who counts as an acceptable loss.

Your sibling comes home changed but alive. The Mayor either helps evacuate the town or dies defending the bargain, based on how you treated him during the investigation. At dawn, the diner opens using a camping stove. The town is frightened, ordinary, and finally its own.

*Tone: costly, hopeful, practical. Freedom does not repair the buildings for them.*

### Ending B — "The New Keeper"
**Preserve the bargain · accept the final offer**

You learn that destroying the thing will also remove the boundary keeping worse things away. You take the Mayor's place as Hollowbrook's keeper, save your sibling, and preserve the town exactly as visitors expect to find it: quiet streets, cheap coffee, no violent crime, and one disappearance every few years.

The epilogue shows how earlier choices determine your version of the bargain. People you trusted may help choose the next name, leave town forever, or become your first enemies. The final image mirrors the opening meeting: you sit behind the Mayor's desk while a newcomer enters with questions.

*Tone: calm, compromised, ominous. You saved everyone currently in the room.*

### Ending C — "Just Passing Through"
**First playthrough only · repeatedly skip the Mayor's briefing**

The Mayor begins politely, notices the interruptions, becomes visibly annoyed, and finally loses his temper. Two deputies escort you out of Town Hall, put you back on the last bus, and warn the driver not to stop in Hollowbrook again.

The credits recap your adventure with absolute seriousness: distance travelled, mysteries solved (`0`), townspeople saved (`0`), and time in office (`under three minutes`). After the credits, the title screen unlocks a small badge and a normal New Game. This ending never triggers after any completed ending, so repeat players can move quickly without punishment.

*Tone: escalating deadpan. The Mayor is ridiculous, but the game keeps a straight face.*

---

## 4. The choice ledger

The systems live in the engine-agnostic Core assembly and are never shown as scores.

**`Decisions`** — a list of significant choices, each with a choice ID, description, and flags. Endings and epilogues cite concrete decisions rather than a morality total.

**`PreparedTown`** — flags for people warned, routes opened, and evidence shared. Ending A reads these to decide who survives the final night and what remains standing.

**`BargainTerms`** — flags recording compromises: accepted protection, concealed evidence, promised a replacement, or refused every offer. Ending B reads these to show what kind of keeper the player becomes.

**`PlaythroughHistory`** — whether any ending has been completed and which endings have been seen. The dialogue-skip ending is eligible only when no ending has ever been completed in the profile.

## 5. The Mayor's patience — the signature mechanic

The opening briefing introduces Mayor Silas Vale, establishes the mystery, and quietly watches how the player treats the conversation.

### What counts as a skip

A skip is an Advance input while typewriter text is still revealing. The first press completes the current line as normal and increments `EarlyAdvanceCount` for this conversation.

These do **not** count:

- advancing after the full line is visible
- selecting a dialogue choice
- setting text speed to Fast or Instant
- using an accessibility option that reveals complete lines
- advancing quickly on later playthroughs

### Escalation

| Count | Mayor response | Presentation |
|---|---|---|
| 0–1 | Continues normally | Neutral portrait and voice |
| 2 | "Am I keeping you?" | Annoyed portrait; one inserted line |
| 3 | Restarts the interrupted sentence slowly | Sharper expression; longer pause |
| 4 | Final warning | Furious portrait; deputies enter the background |
| 5 | Ends the meeting | Rage animation, deputies eject the player, Ending C begins |

The warning lines are part of the dialogue graph, not hardcoded in the UI. Core owns the count and emits an irritation stage; Unity owns portraits, animation, sound, and camera work.

### Fairness rules

- The final warning must be unmistakable before the ending can trigger.
- The count resets if the player reloads a save from before the meeting.
- Once any ending has been completed, the profile permanently disables punishment and the first press only reveals text.
- Automated tests prove Instant text and fully revealed advances cannot trigger Ending C.

## 6. The central choices

The full run has four major choices. Each is simple to understand and changes visible details in both main endings.

| Choice | Safe-looking option | Risky-looking option | Later consequence |
|---|---|---|---|
| The deputy's files | Give the evidence to reporter Mara | Return it to Mayor Vale | More people learn the truth, or the evacuation stays orderly |
| The mine entrance | Seal it after rescuing Eli | Leave it open to investigate deeper | Fewer creatures escape, or you learn the bargain's real terms |
| The marked resident | Hide June from the town | Hand her to the Mayor for protection | June aids the final ritual, escapes, or becomes part of the bargain |
| The emergency siren | Warn everyone immediately | Keep silent to avoid panic | A messy evacuation, or more people caught unaware |

No option is labelled good or evil. The player should understand the immediate trade-off even when the delayed result is uncertain.

## 7. Cast

| Character | Role | The thing that makes them memorable |
|---|---|---|
| **Alex Reed** | Player character | Returned only to find younger sibling Eli; practical, not chosen or magical |
| **Eli Reed** | Missing sibling | Left clues because nobody in authority would admit what was happening |
| **Mayor Silas Vale** | Town authority and current keeper | Polished civic patience over a spectacular temper; believes the bargain is necessary |
| **Deputy Nora Pike** | Guard-equivalent and reluctant ally | Enforces Vale's orders but keeps copies of everything |
| **Mara Bell** | Diner owner and local reporter | Knows every rumour and verifies them before repeating them |
| **June Mercer** | Resident marked by the bargain | Funny, frightened, and unwilling to become a noble sacrifice |
| **The Guest Below** | Supernatural force | Speaks using familiar voices and never states a direct lie |

## 8. Places

| Place | Function | Note for the art-direction milestone (M13) |
|---|---|---|
| **Town Square** | Hub: Town Hall, diner, sheriff's office | One compact street reused by day, dusk, and the final night |
| **Old Mine Road** | Travel and simple combat encounters | A short wooded route with two encounter clearings |
| **Mercer Mine** | Investigation and finale | Reuse modular tunnels; lighting does most of the storytelling |
| **Town Hall** | Opening briefing and Ending B mirror scene | The same office frames the Mayor and later the player |

## 9. Three acts, mapped to the build

| Act | Content | Built in |
|---|---|---|
| **I — Welcome Home** | Arrival, Mayor briefing, sibling's trail, first town choices | M04–M06 (2D) · M14 (3D) |
| **II — What Keeps Us Safe** | Mine investigation, creature encounters, the bargain revealed | M05–M09 (2D) · M12–M14 (3D) |
| **III — The Last Night** | Prepare the town, confront Vale and the Guest, choose the town's future | M08–M09 (2D) · M14 (3D) |

## 10. Combat scope

Combat supports the mystery; it is not a progression system.

- one improvised melee attack
- one dodge
- one creature archetype with two data-tuned variants
- health, damage, windup, recovery, and clear hit feedback
- two or three short mandatory encounters in the entire game

No stamina, combo chain, heavy attack, lock-on mode, weapon inventory, skill tree, loot, boss phases, or combat upgrades. In 3D, generous aim assist turns the character toward the nearest visible threat before an attack.

## 11. Scope guardrails

**In:** one hub, one road, one mine · 4 major choices · exactly 3 endings · simple attack and dodge · one enemy archetype · save/load with decision history · dialogue UI with portraits · short journal · limited voiced lines · full game shell.

**Out:** open world · medieval kingdom · elaborate combat · inventory economy · crafting · romance · full voice acting · procedural levels · a fourth ending · another town.

## 12. Narrative homework

- [ ] Decide whether `Last Stop, Hollowbrook` stays as the title. (by M02)
- [ ] Write Mayor Vale's complete opening briefing and five interruption responses. (by M04)
- [ ] Decide exactly why Eli entered the mine alone. (by M05)
- [ ] Write the Guest's best argument for preserving the bargain. (by M05)
- [ ] Decide the personal cost Mayor Vale already paid to become keeper. (by M06)
- [ ] Write the final images for both full-story endings. (by M08)
- [ ] Write the deadpan statistics and final title card for Ending C. (by M08)
